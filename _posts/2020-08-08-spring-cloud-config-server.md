---
layout: post
title:  "Spring Cloud (1) - Spring Cloud Config Server"
date:   2020-08-08 10:00
author: juhyun10
tags:	[MSA,spring-cloud-config-server,spring-cloud-bus]
---

# 시작하며
이 포스트는 MSA를 보다 편하게 도입할 수 있도록 해주는 스프링 클라우드 프로젝트 중 Spring Cloud Config Server에 대해 기술한다.
관련 소스는 [github/juhyun10](https://github.com/juhyun10/msa-springcloud) 를 참고바란다.

앞으로 연재 방식으로 아래 컴포넌트들에 대해 포스팅을 할 예정이다.
>***Spring Cloud Config Server - 환경설정 외부화 및 중앙 집중화***<br />
>   - Spring Cloud Config Server
>   - 컨피그 서버 구축
>       - 컨피그 서버 셋업
>       - 저장소 구현
>   - 클라이언트에서 컨피그 서버 접근
>   - 컨피그 서버에서 환경설정 변경값 갱신
>   - 환경설정 변경 전파
>       - 환경설정 변경 전파 적용<br />
>
>Zuul - Proxy & API Gateway<br />
>Ribbon - Load Balancer<br />
>Eureka - Service Registry & Discovery

다양한 요청을 처리하는 마이크로서비스를 관리하기 위해서 스프링 부트 프레임워크가 제공하는 기능만으로는 충분하지 않다.
스프링 클라우드 프로젝트는 마이크로서비스 개발에 필요한 공통적인 패턴들을 모아서 사용하기 쉬운 스프링 라이브러리형태로 구현해서 제공한다.

스프링 클라우드 프로젝트(Netflix OSS)에 대한 전체적인 설명은 
[여기](https://bravenamme.github.io/2020/07/21/msa-netflix/)에서 개념 확인이 가능하다.


# Spring Cloud Config Server
Spring Cloud Config Server(이하 컨피그 서버)는 애플리케이션과 서비스의 모든 환경설정 속성 정보를 저장, 조회, 관리할 수 있게 해주는 외부화된 환경설정 서버이다.
* 환경 설정 속성 정보 : 데이터베이스, 미들웨어 접속 정보, 애플리케이션의 행동 양식을 정하는 메타데이터 등...<br />
이전 그리고 아직도 많은 방식으로 운영되고 있는 환경설정방식은 아래와 같다.
>모든 환경설정 파라미터를 프로젝트에 함께 패키징되는 application.properties 혹은 application.yaml 파일로 관리

이러한 방식의 단점은 환경설정 속성 정보가 변경되면 환경설정 파일이 애플리케이션에 함께 패키징되어 있기 때문에 애플리케이션 전체를 다시 빌드해야 한다는 점이다.

컨피그 서버는 애플리케이션의 빌드없이 환경 설정의 변경을 적용할 수 있도록 해준다.<br />
아래 컨피그 서버의 동작 흐름을 보자.

![컨피그서버 동작 흐름](/files/posts/20200808/config.png)

>마이크로서비스 인스턴스가 시작하면 필요한 환경설정 정보를 읽기 위해 컨피그 서버에 접근<br /><br />
>마이크로서비스는 성능 향상을 위해 컨피그 서버에서 읽어온 환경설정 정보를 로컬에 캐시<br /><br />
>컨피그 서버는 환경설정 정보가 변경되면 자신을 바라보는 모든 마이크로서비스에 변경 사항 전파<br /><br />
>마이크로서비스는 변경 사항을 로컬 캐시에 반영<br />

컨피그 서버는 환경설정 정보를 Git이나 SVN 같은 버전 관리 도구에 저장한다.<br />
원격 저장소가 아닌 파일 기반으로도 가능은 하지만 버전 관리, 운영상의 리스크 등을 고려하여 원격 저장소를 많이 이용한다.

스프링 부트와는 다르게 스프링 클라우드는 부트스트랩 컨텍스트를 이용한다.<br />
부트스트랩 컨텍스트는 메인 애플리케이션의 부모 컨텍스트 역할을 담당하며, 컨피그 서버에서 환경설정 정보를 읽어온다.<br />
부트스트랩 컨텍스트는 환경설정 정보를 bootstrap.yaml 또는 bootstrap.properties 에서 읽어오는데 application.* 파일을 bootstrap.* 로 이름 변경만 해주면 된다.
<br /><br />
 
<details markdown="1">
<summary>application.* VS bootstrap.* (Click!)</summary>
- 라이프사이클 상 application.* 이 로드되기 전에 bootstrap.* 이 로드됨<br />
(application.* 을 사용하기 전에 부모 ApplicationContext가 로드되는데 bootstrap.* 은 이 부모 ApplicationContext에 의해 로드됨)
- bootstrap.* 에는 환경설정 정보를 조회하기 위한 정보만 기재하고, 그 외의 정보는 application.* 을 이용
- bootstrap.*
   - 기본적으로 구성 서버의 위치(spring.cloud.config.uri)와 애플리케이션 이름(spring.application.name) 기재
   - 그 외 Eureka 서버 위치, 암호화/해독 관련 속성과 같은 다른 설정 포함 가능
   - 애플리케이션 기동 시 애플리케이션의 이름으로 컨피그 서버에 HTTP 호출을 하여 해당 애플리케이션 구성 정보검색
   - Spring Cloud를 사용하고 애플리케이션 환경설정이 원격 구성 서버(컨피그 서버)에 저장되어 있는 경우에만 필요
   - application.* 과 마찬가지로 bootstrap-dev.* 이런 식의 프로파일별 구성 가능
</details>

<details markdown="1">
<summary>*.properties VS *.yml VS *.yaml (Click!)</summary>
- [yaml FAQ](https://yaml.org/faq.html)에선 *.yaml을 공식 확장자라고 이야기 함.
- *.yml 이 있는 이유는 *.html VS *.htm 과 비슷한 이유일 것이라 추측<br />
   (MS-Dos 시절엔 파일의 확장자가 길이가 3자로 제한되었다고 하는데 
   그 시절의 영향인지 3글자 확장자 스타일을 고수하는 사람들때문에
   *.yml 이나 *.htm 같은 확장자가 존재하는 것으로 추정)
- *.yaml 사용 시 특징
   - `--` 를 이용하여 한 파일에서 모든 프로파일별 설정 정보 관리 가능<br />
     (하지만 설정 정보가 많아질 경우 오히려 더 불편하므로 개인적으론 프로파일별로 개별 파일 관리 선호)
   - 동일 key를 허용하지 않기 때문에 동일한 부모를 가진 key 끼리 모여있으므로 흐름 파악에 용이

![*.properties와 *.yaml 차이](/files/posts/20200808/yaml.png)
</details>

# 컨피그 서버 구축
## 1. 컨피그 서버 셋업
새로운 스트링부트 프로젝트 생성 후 Config Server Dependency 를 추가한다.
actuator는 서버 구동 확인용으로 사용할 예정이다.
actuator에 대한 간단한 설명은 이전 포스트인 [여기](https://bravenamme.github.io/2020/03/26/spring-actuator/)를 참고하길 바란다.

스트링부트의 버전은 2.3.2 이고, 스프링 클라우드의 버전은 Hoxton.SR6 이다.
스프링 클라우드 버전에 따른 스프링 부트 버전 선택은 [여기](https://spring.io/projects/spring-cloud)를 참고한다.

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.3.2.RELEASE</version>
  </parent>
...
<properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <java.version>11</java.version>
    <spring-cloud.version>Hoxton.SR6</spring-cloud.version>
</properties>
...
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-config-server</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```
## 2. 저장소(Git) 구현
지금은 로컬 파일 시스템 기반의 저장소로 연결하고 원격 저장소로의 연결은 마지막에 구현할 예정이다.
저장소로 사용할 폴더 생성 후 컨피그 서버가 방금 만든 저장소를 사용하도록 설정한다.
application.properties의 이름을 bootstrap.yaml 으로 변경 후 아래와 같이 설정한다.
윈도우 환경에서는 URL의 맨 끝에 `/` 를 추가해준다.

```yaml
# bootstrap.yaml
spring:
  application:
    name: configserver
  cloud:
    config:
      server:
        encrypt:
          enabled: false

# applicaton.yaml
server:
  port: 8889    # 컨피그 서버가 수신 대기하는 포트
spring:
  profiles:
    active: native
  cloud:
    config:
      server:
        native:
          search-locations: file:C:/myhome/03_Study/13_SpringCloud/assucloud/config-repo/member-service
management:
  endpoints:
    web:
      exposure:
        include: "*"
  endpoint:
    shutdown:
      enabled: true
```

부트스트랩 클래스에 `@EnableConfigServer` 애노테이션을 추가한다.
`@EnableConfigServer` 애노테이션은 서비스를 컨피스 서버 서비스로 사용 가능하게 한다.

아래 명령어를 통해 컨피그 서버를 시작한다.
```shell
mvn spring-boot:run
```
아래 주소로 접속하여 컨피그 서버가 구동중인지 확인할 수 있다.<br />
[http://localhost:8889/actuator](http://localhost:8889/actuator)

![컨피그 서버 구동 확인](/files/posts/20200808/actuator.png)


# 클라이언트에서 컨피그 서버 접근
위에서 컨피그 서버를 구성 후 웹 브라우저를 통해 접근하는 방법을 보았으니, 이제 마이크로서비스가 컨피그 클라이언트로서 컨피그 서버에
접근하도록 한다.

새로운 스트링 부트 프로젝트를 생성한다.
이 때 Config Client와 Actuator Dependency를 추가한다.
Actuator은 환경설정 정보 갱신 후 확인용도로 필요하다.

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-config</artifactId>
</dependency>
```

위에 언급했다시피 bootstrap.yaml 은 서비스 애플리케이션 이름, 애플리케이션 프로파일과 컨피그 서버에 접속할 수 있는 URI를 기입하고
application.yaml엔 로컬에 유지하고 싶은 구성정보를 기입한다.

```yaml
# bootstrap.yaml
spring:
  application:
    name: member-service    # 서비스 ID (컨피그 클라이언트가 어떤 서비스를 조회하는지 매핑)
  profiles:
    active: default         # 서비스가 실행할 기본 프로파일
  cloud:
    config:
      uri: http://localhost:8889  # 컨피그 서버 위치

# application.yaml
server:
  port: 8090
management:
  endpoints:
    web:
      exposure:
        include: "*"
  endpoint:
    shutdown:
      enabled: true
```

컨피그 서버와 정상적으로 통신하는지 확인하기 위하여 저장소를 아래와 같이 구성하여 확인해보자.

![회원서비스 컨피그 저장소](/files/posts/20200808/memberconfig.png)

![회원서비스 환경설정 파일](/files/posts/20200808/memberyaml.png)

```java
// 회원서비스 커스텀 컨피그 파일
@Component
public class CustomConfig {
    @Value("${your.name}")
    private String yourName;

    public String getYourName() {
        return yourName;
    }
}

// 회원서비스 RESTController
@RestController
@RequestMapping("/member")
public class MemberController {

    private CustomConfig customConfig;

    public MemberController(CustomConfig customConfig) {
        this.customConfig = customConfig;
    }

    @GetMapping(value = "name")
    public String getYourName(String nick) {
        return "Your name is " + customConfig.getYourName() + ", nickname is " + nick;
    }
}
```

`mvn spring-boot:run` 으로 컨피그 서버 기동 후 각각 아래의 주소로 접속하면 각 프로파일에 맞는 JSON 페리로드가 반환되는 것을 알 수 있다.

[http://localhost:8889/member-service/default/](http://localhost:8889/member-service/default/)

![디폴트 설정 파일](/files/posts/20200808/memberdefault.png)

[http://localhost:8889/member-service/dev](http://localhost:8889/member-service/dev)

![dev 설정 파일](/files/posts/20200808/memberdev.png)

Actuator 를 이용하여 현재 실행중인 환경 정보를 확인할 수 있다.
단, /env 엔 많은 정보가 노출되므로 운영시엔 비활성화하도록 한다. 

[http://localhost:8090/actuator/env](http://localhost:8090/actuator/env)

![Actuator](/files/posts/20200808/actuator2.png)

이제 컨피그 서버를 통해 전달받은 설정값이 마이크로서비스에서 정상적으로 사용되고 있는지 확인해보자.
![컨피그 서버로부터 전달받은 설정값](/files/posts/20200808/membername.png)


# 컨피그 서버에서 환경설정 변경값 갱신
저장소의 프로퍼티를 변경하면 컨피그 서버는 항상 최신 버전의 프로퍼티를 제공한다.
하지만 애플리케이션은 기동시에만 프로퍼티를 읽어오는데 이 때 actuator 의 `@RefreshScope` 를 사용하여
`/actuator/refresh` 엔드 포인트를 호출함으로써 애플리케이션 재기동없이 프로퍼티를 다시 읽어올 수 있다.

`@RefreshScope` 애노테이션은 실제 프로퍼티를 받아오는 클래스에 달아준다.
```java
@Component
@RefreshScope
public class CustomConfig {
    @Value("${your.name}")
    private String yourName;

    public String getYourName() {
        return yourName;
    }
}
```

이제 `member-service.yaml`의 프로퍼티 속성을 변경해보자.
```yaml
your.name: "ASSU ASSU DEFAULT Modify"
```

[http://localhost:8889/member-service/default/](http://localhost:8889/member-service/default/) 컨피그 서버엔 변경된 프로퍼티값이 바로 반영된 것을 확인할 수 있다.

하지만 [http://localhost:8090/member/name?nick=JU](http://localhost:8090/member/name?nick=JU) 을 호출해보면 변경 전 프로퍼티값이 노출되고 있다.

`/actuator/refresh` 종단점을 호출(POST 호출)하여 변경된 프로퍼티값으로 갱신 후 다시 확인해보면 마이크로서비스에서 변경된 프로퍼티값이 전달되고 있는 것을 확인할 수 있다.

![프로퍼티값 갱신](/files/posts/20200808/refresh.png)


![회원서비스에서의 확인](/files/posts/20200808/refresh2.png)


# 환경설정 변경 전파
위에 기술한 것처럼 `/actuator/refresh` 종단점을 호출하여 환경설정값을 갱신해도 되지만 인스턴스가 수가 많아지면
환경설정 정보가 변경될 때마다 인스턴수의 수만큼 종단점을 호출해줘야 한다.
이에 대한 해결 방법으로는 두 가지가 있다.

> 1. Eureka 엔진으로 모든 서비스 인스턴스를 조회한 후 `/actuator/refresh` 종단점을 직접 호출하는 간단한 스크립트 작성
> 2. 스프링 클라우드 버스와 RabbitMQ(메시지 브로커)를 통해 변경 내용을 broadcasting

1의 방법이 간단해 보이지만 여기선 2번의 내용으로 진행해 볼 것이다.

Spring Cloud Bus (이하 클라우드 버스) 는 현재 실행되고 있는 인스턴스의 수나 위치에 관계없이 환경설정 변경 내용이 모든 인스턴스에 적용되게 할 수 있다.
모든 인스턴스를 하나의 메시지 브로커에 연결하면 가능하다.

동작 흐름은 아래와 같다.

> 1. 각 인스턴스는 하나의 메시지 브로커를 통해 변경 이벤트 구독
> 2. 변경 이벤트가 발생하면 각 인스턴스는 변경된 환경설정 정보를 새로 읽어와서 로컬에 캐싱된 정보 갱신

어느 한 인스턴스의 `/actuator/bus-refresh` 종단점이 호출되면 클라우드 버스와 공통의 메시지 브로커를 통해 변경된 내용이 모두에게 전파되는 방식이다.

가장 많이 사용되고 있다는 RabbitMQ를 [AMQP(Advanced Message Queuing Protocol)](https://ko.wikipedia.org/wiki/AMQP) 메시지 브로커로 사용할 예정이다.

## 1. 환경설정 변경 전파 적용
[여기](http://www.rabbitmq.com/download.html) 에서 RabbitMQ를 다운로드 받은 후 클라우드 버스 Dependency를 추가한다.
                                                                                           
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-bus-amqp</artifactId>
</dependency>
```

컨피그 서버 내 member-service.yaml에 rabbitMQ 접속 정보를 셋팅한다.
```yaml
your.name: "ASSU ASSU DEFAULT"
spring:
  rabbitmq:
    host: localhost
    port: 5672
    username: guest
    password: guest
```

회원 서비스를 2개의 인스턴스로 띄운 후 컨피그 저장소의 프로퍼티값을 변경해보자.
```shell
C:\member-service> mvn clean install
C:\member-service\target>java -Dserver.port=8090 -jar member-service-0.0.1-SNAPSHOT.jar
C:\member-service\target>java -Dserver.port=8091 -jar member-service-0.0.1-SNAPSHOT.jar
```
 
```yaml
your.name: "ASSU ASSU DEFAULT Modify!!"
```

이후 특정 한 인스턴스의 `/actuator/bus-refresh` 종단점을 호출하여 변경된 환경설정값이 모든 인스턴스에 적용되는지 확인해보자.

![port 8090의 /actuator/bus-refresh 호출](/files/posts/20200808/bus.png)  


![port 8090 확인](/files/posts/20200808/8090.png) 


![port 8091 확인](/files/posts/20200808/8091.png) 

클라우드 버스 종단점(`/actuator/bus-refresh`) 은 메시지 브로커에게 내부적으로 메시지를 전송하는데,
이 메시지는 결국 모든 인스턴스가 각자의 환경설정 정보를 최신 내용으로 갱신할 수 있게 한다.
위에서 보다시피 port 8090의 `/actuator/bus-refresh` 종단점만 호출하면 같은 클라우드 버스에 연결되어 있는
port 8091 인스턴스까지 변경된 환경설정값이 갱신되는 것을 확인할 수 있다.


★정보보호
★git 

```shell
-- RabbitMQ 실행
rabbitmq_server-3.8.6\sbin> ./rabbitmq-server
``` 





 

 
# 마치며 
MSA 구성은 기존 monolithic 구조와 비교해서 결코 심플하지 않다.  
하나의 서비스를 잘게 쪼갬으로써, 서비스간 복잡도가 증가 될수 있으머, 라우터, Circuit breaker,  
각 서비스들의 관리등 고려해야 할 것들이 기존보다 더 많아질수도 있다.  
그럼에도 불구하고 서비스들을 나누고 권한을 위임하면서, 고가용성, 유연한 스케일링, 빠르고 쉬운 배포 등의  
큰 장점들이 있기 때문에 협업 부서가 많거나 규모가 좀 있는 시스템이라면 충분히 고려해 볼 만한 가치가 있다.
  

# 참고 사이트
* [https://perfectacle.github.io/2018/08/19/yaml/](https://perfectacle.github.io/2018/08/19/yaml/)
* [https://stackoverflow.com/questions/54929656/intellij-idea-not-showing-anything-endpoints-tab-failed-to-retrieve-applicatio](https://stackoverflow.com/questions/54929656/intellij-idea-not-showing-anything-endpoints-tab-failed-to-retrieve-applicatio)
* [Messaging with RabbitMQ](https://spring.io/guides/gs/messaging-rabbitmq/)
* [AMQP doc](https://docs.spring.io/spring-boot/docs/2.3.2.RELEASE/reference/htmlsingle/#boot-features-amqp)