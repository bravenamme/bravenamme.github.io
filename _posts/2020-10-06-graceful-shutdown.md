---
layout: post
title:  "Spring boot Graceful Shutdown"
date:   2020-10-06 10:00
author: twosunny
tags:	[springboot,shutdown,graceful,deployment]
---

## 시작하며
보통 애플리케이션에서 변경 사항이 있으면, 애플리케이션을 재시작 해야 변경사항이 반영된다.  
물론 배포 방식에 따라 Blue-Green 배포 형식을 따르게 되면, 이전 버전 앱을 굳이 죽일 필요는 없지만,  
Rolling 배포를 따르는 경우, 순차적으로 구버전의 앱을 죽이고, 신규 버전 앱을 띄어야 한다.  
  
구버전 앱을 죽일때 프로세스 자체를 바로 죽이게 되면, 현재 요청을 받아 처리중인 작업들이 모두 취소가 되기 때문에,  
복구 프로세스 확립등 할것이 너무나 많아 진다.  
예를들어 결재처리중인 애플리케이션이 갑자기 죽어 버린다면?(생각도 하기 싫다....)  

물론 애플리케이션 윗단에서, 종료 시그널이 오면 더이상 요청을 받지 않고, 이미 와 있는 요청들만  
처리 후 앱을 종료하면 되지만(AWS Beanstalk등), 앱 자체에서도 정상 종료 프로세스는 꼭 필요하다.  

이번 포스팅에서는 앱 프로세스를 죽일때 사용되는 Graceful Shutdown을 구현해 보도록 한다.    

## KILL
리눅스등 환경에서는 프로세스를 죽일때 kill 을 사용하는데, 옵션에 따라 동작이 상이하다.
> -15 : 정상 종료  
> -9 : 강제 종료

-9 옵션을 쓰게 되면 작업중인 데이터를 저장하지 않고, 즉시 종료하기 때문에 되도록 사용에 신중해야 하며,  
-15(기본옵션)는 자신이 하던 작업을 안전하게 종료하는 절차가 진행된다.  

때문에 kill -15 를 기본으로 테스트 해보도록 하자.  

## 애플리케이션 구성
매우 간단한 springboot 앱을 만들고, 요청이 오면 20초뒤 응답을 주는 controller를 만들자.  
```java
@RestController
@RequestMapping("/test")
public class TestController {

    @GetMapping("/process/{no}")
    public String process(@PathVariable int no) throws InterruptedException {

        System.out.println("Start Process No." + no);
        Thread.sleep(20000);
        System.out.println("End Process No." + no);
        
        return "shutdown!";
    }

}
``` 
http://localhost:8080/test/process/1 를 실행하면 아래와 같이 요청을 받은즉시 로그를 남기고,  
20초 뒤에 요청 작업이 완료 되면 로그를 남기게 하였다.  
```java
Start Process No.1
End Process No.1
```

## 테스트
앱 실행후, 작업 요청을 준 뒤 20초를 기다리지 않고 즉시 kill -15 를 날려보자.
```java  
Start Process No.1
2020-10-06 12:54:02.026  INFO 97162 --- [extShutdownHook] o.s.s.concurrent.ThreadPoolTaskExecutor  : 
Shutting down ExecutorService 'applicationTaskExecutor'
Disconnected from the target VM, address: '127.0.0.1:49878', transport: 'socket'

Process finished with exit code 143 (interrupted by signal 15: SIGTERM)
```
End Process 로그가 찍히기도 전에 앱이 종료되었다. 로그를 보면 앱에서는 kill 옵션 15를 수신하긴 하였지만,  
앱 입장에서는 그 이후에 어떻게 종료해야 하는지 전혀 모르기 때문에 그냥 종료가 진행된 것이다.  

## graceful 옵션 적용
spring boot 에서는 actuator를 사용하여 graceful shutdown을 구현하는 방식이 있는데,  
spring boot 2.3 버전 이후부터는 graceful shutdown을 지원하기 때문에 너무나 편하게 구현이 가능하다.  
(아....버전업을 해야하나.....)  
spring boot 내장 엔진인 tomcat, Jetty, Undertow, Netty 모두 적용 가능하다.
application.yml(또는 application.properties)에 아래 옵션을 추가해보자.
  
```java  
server:
  shutdown: graceful
```

그리고 다시 동일하게 테스트!

```java  
Start Process No.1
2020-10-06 13:03:01.542  INFO 98184 --- [extShutdownHook] o.s.b.w.e.tomcat.GracefulShutdown        : 
Commencing graceful shutdown. Waiting for active requests to complete
End Process No.1
2020-10-06 13:03:15.275  INFO 98184 --- [tomcat-shutdown] o.s.b.w.e.tomcat.GracefulShutdown        : 
Graceful shutdown complete
```
로그를 보면 처리 요청을 받은후에(Start Process 로그), 앱 종료 시그널이 수신 되었다.  
하지만 graceful shutdown 옵션이 있으니, 요청된 작업을 완료후에 End Process 로그가 찍혔고  
그 이후에 앱이 종료되는것을 볼수 있다.  

그렇다면, 요청1 --> 앱 종료(kill -15) --> 요청2 이렇게 되면 어떻게 될까?
결과는 위 테스트 결과와 동일하게 나온다. 즉 요청2는 네트워크 레벨에서 아예 차단 되어 앱에서 요청을 받지 못한다. 

![요청2 접속 불가](/files/posts/20201006/fail.png)



## 마치며 
애플리케이션 구동도 중요하지만, 안전한 종료야 말로 신규 버전을 배포 하기 위해서 필수적으로 진행해야 하는 부분이다.  
위 예시로 든 graceful shutdown은 tomcat 환경에서 이루어 졌으며, undertow 에서는 약간 동작이 다르니  
참고하여 구현하면 되겠다.  
  

## 참고 사이트
* https://www.baeldung.com/spring-boot-web-server-shutdown
