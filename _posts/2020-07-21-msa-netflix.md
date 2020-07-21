---
layout: post
title:  "MSA and Netflix OSS"
date:   2020-07-21 10:00
author: twosunny
tags:	[msa,netflix,eureka,ribbon,apigateway,zuul,hystrix]
---

## 시작하며
Microservice Architecture(이하 MSA) 는 구체적으로 언제부터 핫?해 졌는지는 몰라도,  
적어도 최근 5년간 가장 핫하게 발전된 아키텍쳐이다. 
하지만, MSA를 실 업무에 적용하기엔 검토해야 할 것들이 꽤 많이 있다. 레거시코드와의 혼재성, 
fail over, 각 모듈간 통신 등등 간단한 기술만은 결코 아니다. 따라서 현업에서 충분한 기술 검토 및 적용 타당성을 신중히 하여 판단해야 한다.  
그럼에도 불구하고 모듈들을 작게 쪼개고 권한/임무를 쪼갬으로써 얻은 운영상의 이익은 보기보다 꽤 크다.
MSA는 일정에 맞춰 빨리 개발하는게 결코 결코 목적이 아닌, 긴 시간 효율적으로 운영/개발에 필요한 아키텍쳐이다.

MSA를 구현하기 위해서 여러 방법들이 존재하겠지만, 가장 대중적이라 할수 있는 넷플릭스에서 공개한 Netflix OSS(Open Source Software) 를 통해 살펴보고자 한다.  


## Netflix OSS(Open Source Software)
'Netflixed’라는 신조어가 있다. 직역하면 ‘넷플릭스 당하다’인데, 미국 실리콘밸리에선 기존 비즈니스 모델이 붕괴되었을 때  
이 말을 사용한다.    
Netflix는 실제로 여러가지 혁신을 하였는데, 그중에서 기존 Legacy 시스템은 MSA로 모두 전환하여 운영/개발 상의 효율성을   
극대화 하였다.    
2007년 심각한 데이터베이스 손상으로 3일간 서비스 장애를 겪은 넷플릭스는 신뢰성 높고 수평 확장이 가능한 클라우드  
시스템으로 이전할 필요성을 느꼈는데, 단순히 플랫폼 이전만으로는 기존의 문제점과 한계를 탈피할 수 없다고 판단한  
넷플릭스는 고가용성, 유연한 스케일링, 빠르고 쉬운 배포를 위해 MSA를 선택하였다고 한다. 그리고 MSA 전환을 위한  
기술들을 도입하여 무려 7년에 걸쳐 클라우드 환경으로 이전하였으며, 이 과정에서 넷플릭스가 경험한 노하우와 문제해결  
방법을 공유하기 위해 MSA 전환 기술을 오픈소스로 공개 하였다.      
이렇게 탄생한 넷플릭스 OSS는 MSA를 도입하려는 많은 사람에게 좋은 선택지가 되고 있다.

### Spring Cloud Netflix
![Spring Cloud Netflix](/files/posts/20200721/cloud.png)
* 출처 : https://dzone.com/articles/microservices-journey-from-netflix-oss-to-istio-se  

모든 Netflix 라이브러리 및 시스템(Netflix OSS) 2012 년경에 오픈 소스로 제공되었으며 오늘날까지도 커뮤니티에서  
여전히 사용되고 있다.    
2015 년 Spring Cloud Netflix는 1.0에 도달 하였으며, 이는 Netflix 내부 솔루션 대신 Spring Boot를  
사용하여 Netflix OSS 구성 요소를 함께 연결하려는 커뮤니티 노력이었으며, 시간이 지남에 따라 커뮤니티에서  
Netflix의 오픈 소스 소프트웨어를 채택하는 데 선호되는 방법이 되었다.   
Netflix는 2018 년부터 Spring Cloud Netflix 를 통해 커뮤니티의 기여를 활용하여 핵심 Java 프레임 워크로  
Spring Boot로 전환하고 있음을 발표하게 되었다.

결론은 Netflix OSS와 Spring Boot의 결합은 Netflix 외부에서 시작되었으며. 이제 그것을 Netflix 내부에서 수용하고 있다.

### Eureka
간단히 말해 각각 쪼개진 서비스들의 주소들의 집합체 또는 서비스 resistry 리고 생각하면 된다.
예를 들어 서비스가 100개 있다고 가정하자. 100개의 서비스는 당연히 서로간 통신을 하고 있다.
이럴때 100개의 서비스를 관리하기 위헤서, 100개의 대한 도메인 또는 IP 정보와 포트 정보들을 일일히 다 알고 관리를 해야 한다.  
만약 서비스를 줄이거나, 또한 늘려야 하는 상황이면 다시 수동으로 작성/관리를 해줘야 하며, 특정 서비스에 장애가 났을때, 그 서비스를
수동으로 제외해줘야 한다. 말이 100개이지 엄청난 운영 리소스가 들어갈 것이다.  


Eureka는 이러한 서비스들의 목록들을 자동으로 관리해 준다고 생각하면 된다. 
서비스들은 서비스가 기동할때 Eureka 서버에에 자기 정보(IP 정보 등)을 노티하게 되며, Eureka는 등록된 서버들에게 일정 시간동안
heartbeat를 보내며, 서비스가 정상적인지 죽었는지 확인한다. 각 서비스들은 Eureka에서 정상 작동 서비스 목록들을 가져와서 사용하게 된다.

![eureka구조](/files/posts/20200721/eureka.png)
* 출처:https://tech.asimio.net/2017/03/06/Multi-version-Service-Discovery-using-Spring-Cloud-Netflix-Eureka-and-Ribbon.html

### Ribbon
클라이언트 사이드 로드 밸런서(Client Side Load Balancer)이다.  
시스템 부하 분산을 위해 L4 스위치등 하드웨어 장비를 통해 앞단에 두어 부하를 분산하게 되는데, Ribbon은  
클라이언트 소프트웨어로 트래픽을 분산/제어 할수 있다. 위에서 예시한것 처럼 100개의 서비스가 있는 환경에서  
트래픽 이슈로  서비스가 추가/삭제 된다면, L4와 같은 하드웨어에서도 똑같은 대응을 해야 하는데,   
그런일이 빈번하다면 꽤 고달퍼 질것이다.  
Ribbon은 물론 정적 서버 리스트를 가지고 부하 분산을 할 수도 있으며, Eureka와 연동하여, 동적으로 리스트를   
관리하면서 부한 분산이 가능하다. 부하 분산 방식도 라운드 로빈, 3회 연속 호출 실패 시 30초 동안 목록에서 제외하는   
Availability filtering 등 여러가지 설계가 가능하다.   

### API Gateway 그리고 Zuul
먼저 API Gateway에 대해 알아보자.  
MSA는 작은 단위의 서비스로 분리함에 따라 서비스의 복잡도를 줄일 수 있으며, 변경에 따른 영향도를 최소화하면서 개발과    
배포를 할 수 있다는 장점이 있다.  
하지만, 여러 서비스의 엔드포인트를 관리해야 하는 어려움이 있으며 각 서비스의 API에서 공통적으로 필요한 기능을   
중복으로  개발해야 하는 문제가 있다.  
API Gateway를 이용하면 통합적으로 엔드포인트와 REST API를 관리 할 수 있으며, 모든 클라이언트는 각 서비스의   
엔드포인트 대신 API Gateway로 요청을 전달한다.  
API Gateway는 즉 라우터 및 reverse proxy 기능뿐 아니라, 엔드포인트 서버에서 공통으로 필요한 인증, 접근 제어등  
공통적인 부분도 구현이 가능하다.
![apigateway](/files/posts/20200721/apigateway.jpg)
* 출처 : https://microservices.io/patterns/apigateway.html

zuul은 Netflix OSS에서 제공하는 api gateway의 구현체라고 보면 된다.     
![zuul](/files/posts/20200721/zuul.png)
* 출처 : https://github.com/Netflix/zuul/wiki/How-We-Use-Zuul-At-Netflix

zuul은 인증요구 식별, 라우팅, 스트레스 테스팅 등등 여러 기능이 있는데, 요청 온 request를 필터 / 라우팅 하는 그림은  
아래와 같이 표현될수 있다.
![zuul](/files/posts/20200721/filter.png)
* 출처 : https://netflixtechblog.com/announcing-zuul-edge-service-in-the-cloud-ab3af5be08ee

요청이 들어면 PRE Filter를 실행하고, ROUTING Filter에 의해 정해진 서버로 요청을 보낸다. 정해진 서버에서 응답이  
오면 POST Filter를 실행시킨다.  

### hystrix
분산 환경에서는 불가피하게 많은 서비스 종속성 중 일부가 실패한다.  
Hystrix는 대기 시간 허용 오차 및 내결함성 논리를 추가하여 이러한 분산 서비스 간의 상호 작용을 제어하는 ​​데 도움이되는  
라이브러리이다. Hystrix는 서비스 간 액세스 지점을 격리하고 여러 서비스에서 계단식 오류를 방지하며 대체 옵션을  
제공하여 시스템의 전반적인 복원력을 향상시킬수 있다.  

예를 들어보자.  
각 서비스의 가동 시간이 99.99% 인 30 개의 서비스에 의존하는 응용 프로그램의 경우 다음과 같이 예상 할 수 있다.  

> 99.99 30 = 99.7 % 가동 시간  
> 10 억 개의 요청 중 0.3 % = 3,000,000 실패  
> 모든 종속성이 뛰어난 가동 시간을 갖더라도 2시간 이상의 가동 중지 시간

현실은 일반적으로 더 훨신 더 나쁘다....  
모든 종속성이 제대로 수행 되더라도 전체 시스템에 대해서 복원하는 로직을 설계하지 않으면 수십 개의 각 서비스에 대해     
0.01 %의 다운 타임이 전체적으로 영향을 줄 수 있다.

모든 것이 정상이면 요청 흐름은 다음과 같다.  
![서비스 정상 상황](/files/posts/20200721/hys1.png)
* 출처 : https://github.com/Netflix/Hystrix/wiki  


그 중 하나의 서비스가 장애가 발생한다고 가정하면,
![서비스 1개 장애](/files/posts/20200721/hys2.png)
* 출처 : https://github.com/Netflix/Hystrix/wiki   


대량의 트래픽으로 인해 어떠한 하나의 서비스가 지연 또는 장애가 나면, 모든 서버에서 모든 자원이  
몇 초 내에 포화 상태가 될 수 있다.  
네트워크를 통해 또는 클라이언트 라이브러리에 도달하여 네트워크 요청이 발생할 수있는 응용 프로그램의  
모든 지점은 잠재적인 오류의 원인이 되며, 장애보다 더 나쁜 이러한 응용 프로그램은 서비스 간 대기 시간을 늘려   
대기열, 스레드 및 기타 시스템 리소스등 시스템 전체에서 더 많은 계단식 오류를 발생시킨다.

![계단식 장애](/files/posts/20200721/hys4.png)
* 출처 : https://github.com/Netflix/Hystrix/wiki  


이러한 계단식 오류를 방지하기 위한 Circuit breaker 패턴을 구현한 라이브러리가 hystrix 이다.


![hystrix 동작](/files/posts/20200721/hy3.png)  
* 출처 : https://github.com/Netflix/Hystrix/wiki  

간단히 동작을 설명하자면, 어떠한 특정 서비스에서의 응답이 지연되어 정의된 임계값을 넘게 되면,  
기다리는 대신 사용자가 정의한 풀백(fallback) 메서드 실행하여 응답값을 클라이언트에게 전달하게 된다.  
그리고 새롭게 들어오는 다음 request는 장애가 난 서비스에 컨택하지 않고 정의된 fallback 메서드를 즉시 실행하며,  
장애가 복구되면 hystrix는 모니터링 하고 있다가 정상화된 서비스에 다시 연결시켜 준다.   


## 마치며 
MSA 구성은 기존 monolithic 구조와 비교해서 결코 심플하지 않다.  
하나의 서비스를 잘게 쪼갬으로써, 서비스간 복잡도가 증가 될수 있으머, 라우터, Circuit breaker,  
각 서비스들의 관리등 고려해야 할 것들이 기존보다 더 많아질수도 있다.  
그럼에도 불구하고 서비스들을 나누고 권한을 위임하면서, 고가용성, 유연한 스케일링, 빠르고 쉬운 배포 등의  
큰 장점들이 있기 때문에 협업 부서가 많거나 규모가 좀 있는 시스템이라면 충분히 고려해 볼 만한 가치가 있다.
  

## 참고 사이트
* https://www.samsungsds.com/global/ko/support/insights/msa_and_netflix.html
* https://dzone.com/articles/microservices-journey-from-netflix-oss-to-istio-se
* https://github.com/Netflix/zuul/wiki/How-We-Use-Zuul-At-Netflix
* https://meetup.toast.com/posts/201
* https://microservices.io/patterns/apigateway.html
* https://netflixtechblog.com/announcing-zuul-edge-service-in-the-cloud-ab3af5be08ee
* https://github.com/Netflix/Hystrix/wiki