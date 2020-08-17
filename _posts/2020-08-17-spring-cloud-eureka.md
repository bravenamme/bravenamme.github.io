---
layout: post
title:  "Spring Cloud(2) - Spring Cloud Eureka"
date:   2020-08-16 10:00
author: juhyun10
tags:	[MSA,spring-cloud-eureka,ribbon,feign]
---

# 시작하며
이 포스트는 MSA를 보다 편하게 도입할 수 있도록 해주는 스프링 클라우드 프로젝트 중 Spring Cloud Eureka 에 대해 기술한다.
관련 소스는 [github/juhyun10](https://github.com/juhyun10/msa-springcloud) 를 참고바란다.

>[1.Spring Cloud Config Server - 환경설정 외부화 및 중앙 집중화](https://bravenamme.github.io/2020/08/16/spring-cloud-config-server/)<br />
>***2.Eureka - Service Registry & Discovery***<br />
> 3.Zuul - Proxy & API Gateway<br />
> 4.Ribbon - Load Balancer<br />

Spring Cloud Config Server 에 대한 자세한 내용은 [여기](https://bravenamme.github.io/2020/08/16/spring-cloud-config-server/)에서 확인이 가능하다.


# 1. Service Registry & Discovery (서비스 등록 및 발견)
클라우드가 아닌 환경에서 서비스의 발견은 로드 밸런서로 해결이 되었다.
이 로드 밸런서가 서비스에 대한 프록시 역할을 하므로 서비스에 매핑된 정보가 있어야 하는데 이 매핑 규칙을 수동으로 정의해야 하기 때문에 인프라스트럭처의
복잡성이 높아진다. (새로운 서비스를 인스턴스 시작 시점이 아닌 수동으로 등록)

MSA로 설계된 환경에서 중앙 집중식 인프라 스트럭처는 확장성에 제한이 있다.
예를 들어 많은 수의 마이크로 서비스가 있다면 서버와 인스턴스의 갯수를 동적으로 조절할 상황이 발생할 수 있는데
이 경우 예측을 통해 서버의 URL을 정적으로 지정하는 것은 부적합하다. 
 
마이크로서비스가 자신의 서비스를 동적으로 등록하여 스스로 라이프 사이클을 관리할 수 있게 하고, 서비스가 등록되면 서비스 탐색 대상에 포함되어 발견되게 함으로써
최대한 수동 작업을 피하고 자동화하는 것이 좋다.

이러한 부분을 지원하기 위해 MSA 환경에서는 **서비스 디스커버리 메커니즘**을 사용한다.
> - Peer to Peer
>   - 서비스 디스커버리 클러스터의 각 노드는 서비스 인스턴스의 상태를 공유
> - 부하 분산
>   - 서비스 디스커버리는 요청을 동적으로 분산
>   - 정적이며 수동으로 관리되는 ***로드 밸런서가 서비스 디스커버리로 대체됨***
> - 회복성
>   - 서비스 디스커버리 클라이언트는 서비스 정보를 로컬에 캐시하여 서비스 디스커버리 서비스가 가용하지 않을 때에도 로컬 캐시에 저장된 정보를 기반으로 동작
> - 장애 내성
>   - 서비스 디스커버리는 서비스 인스턴스의 비정상을 탐지하고 가용서비스 목록에서 인스턴스 제거 

**서비스 디스커버리 메커니즘**을 구현하려면 아래 4가지 개념을 먼저 이해해야 한다.
> - 서비스 동적 등록
>   - 서비스를 서비스 디스커버리 에이전트에 어떻게 등록하는가?
> - 서비스 동적 발견
>   - 서비스 클라이언트가 어떻게 서비스 정보를 검색하는가?
> - 정보 공유
>   - 서비스 정보를 어떻게 공유하는가?
> - 상태 모니터링
>   - 서비스가 자신의 가용 여부를 어떻게 전달하는가?

## 1.1. 서비스 동적 등록 및 정보 공유
서비스 인스턴스가 시작될 때 자신의 물리적 위치와 경로, 포트를 서비스 레지스트리에 등록한다.
각 인스턴스에는 고유 IP 주소와 포트가 있지만 동일한 서비스 ID로 등록한다. (=서비스 ID는 동일한 서비스 인스턴스 그룹을 식별하는 키)
서비스는 1개의 서비스 디스커버리 인스턴스에만 등록하며, 서비스가 등록되고 나면 서비스 디스커버리는 Peer to Peer로 클러스터에 있는 다른 노드에 전파한다.
  
## 1.2. 서비스 동적 발견
서비스 레지스트리에서 현재 가용중인 필요한 서비스를 호출할 수 있게 해준다.
즉, 정적으로 서비스 URL을 설정하고 관리하는 대신 서비스 레지스트리를 통해 그때그때 사용 가능한 URL을 발견할 수 있다.
또한 빠른 처리를 위해 인스턴스들은 주기적으로 레지스트리 데이터를 로컬에 캐시하여, 서비스 호출시마다 캐싱된 데이터에서 위치 정보를 검색한다.
서비스 호출이 실패히면 로컬에 있던 캐시는 무효화되고 레지스트리로부터 새로 정보를 받아온다.

## 1.3. 상태 모니터링
인스턴스들은 주기적으로 자신의 상태를 레지스트리에 알린다.
레지스트리는 이 요청이 몇 번 오지 않으면 가용할 수 없는 서비스로 간주하여 레지스트리에서 해당 서비스를 제외시킨다.

설명만 들었을 땐 개념이 확 와닿지 않을수도 있는데, 
위 내용은 오늘 다룰 Eureka의 동작 원리와 정확히 일치하므로 지금 당장 이해가 되지 않는다고 걱정할 필요는 없다.

이제 Eureka에 대해 알아본 후 실제 구축을 해보도록 하자.


# 2. Eureka
Eureka 는 자가 등록, 동적 발견 및 부하 분산을 담당하며 위의 서비스 디스커버리 패턴을 구현할 수 있도록 도와준다.
클라이언트 측 부하 분산을 위해 내부적으로 Ribbon 을 사용하므로 이미 Ribbon 을 사용하고 있다면 Ribbon 은 제거해도 좋다.

Eureka는 서버 컴포넌트(이하 유레카 서버)와 클라이언트 컴포넌트(이하 유레카 클라이언트)로 이루어져 있다.

- 유레카 서버
    - 모든 마이크로서비스가 자신의 가용성을 등록하는 레지스트리
    - 등록되는 정보는 서비스 ID와 URL이 포함되는데 마이크로서비스는 유레카 클라이언트를 이용해서 이 정보를 유레카 서버에 등록
    
    
- 유레카 클라이언트
    - 등록된 마이크로서비스를 호출해서 사용할 때 유레카 클라이언트를 이용해서 필요한 서비스를 발견
    - 유레카 서버는 유레카 서버인 동시에 서로의 상태를 동기화하기 서로를 바라보는 유레카 클라이언트이기도 함 (=다른 유레카 서버와 상태를 동기화함)<br />
      이 점은 유레카 서버의 고가용성을 위해 여러 대의 유레카 서버 운영 시 유용함

아래 Eureka 동작 흐름을 살펴보자.

![유레카 동작 흐름](/files/posts/20200816/eureka.png)

★ 근데 리본이 있는데 로드밸런서가 있어야 하나...?

>서비스 부트스트래핑 시점에 각 마이크로 서비스는 유레카 서버에 서비스 ID와 URL 등의 정보를 등록한 후 30초 간격으로 ping을 날려 자신의 가용성을 알림<br />
>이 때 ping 요청이 몇 번 오지 않으면 가용불가한 서비스로 간주되어 레지스트리에서 제외됨<br /><br />
>유레카 클라이언트는 유레카 서버로부터 레지스트리 정보를 읽어와 로컬에 캐싱하고, 이 캐싱된 정보는 30초마다 갱신됨
>(새로 가져온 정보와 현재 캐싱된 정보의 차이를 가져오는 방식)<br /><br />
>다른 마이크로서비스 종단점 접속 시도 시 유레카 클라이언트는 요청된 서비스 ID를 기준으로 현재 사용가능한 서비스 목록을 제공하고,
>리본은 해당 서비스 인스턴스들에게 요청을 분산함<br />


# 3. 유레카 구축
이전 포스팅인 [컨피그 서버](https://bravenamme.github.io/2020/08/16/spring-cloud-config-server/)를 진행했다면
아래 구성도가 이미 로컬에 셋팅되어 있을 것이다.

![컨피그 서버](/files/posts/20200808/config.png)

기존에 진행한 컨피그 서버에 유레카를 추가하면 아래와 같은 구성도가 된다.
언뜻 보면 복잡해 보이지만 회색 음영된 부분은 컨피그 서버 구축 시 이미 구성된 부분으로 더 이상 신경쓰지 않아도 된다. 

![컨피그 서버 + 유레카](/files/posts/20200816/config_eureka.png)

이 포스팅은 내용을 이해하기 위한 것이기 때문에 유레카 서버를 클러스터 모드가 아닌 독립 설치형 모드로 진행할 것이다.
따라서 위 구성도에선 유레카 서버 != 유레카 클라이언트 이다. 

유레카 서버 구축에 앞서 다른 서비스 ID를 가진 마이크로서비스가 필요하니 회원 마이크로서비스를 만들었던 것과 동일한 방식으로
이벤트 마이크로서비스를 미리 만들어두자.<br />
잘 안되는 분들은 [여기](https://github.com/juhyun10/msa-springcloud/commit/57dfc09b2888b98e718502c0ddcb5195703fa7a4)를 참고하세요.

## 3.1. 유레카 서버 구축
새로운 스트링부트 프로젝트 생성 후 Config Client, Eureka Server, Actuator Dependency 를 추가한다.

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-config</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-bus-amqp</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-rsa</artifactId>
</dependency>
```

유레카 서버도 컨피그 서버를 사용하므로 bootstrap.yaml 을 생성해 준 후 아래와 같이 구성을 설정한다.

```yaml
# eurekaserver > application.yaml
server:
  port: 8761          # 유레카 서버가 수신 대기할 포트

# eurekaserver > bootstrap.yaml
spring:
  application:
    name: eurekaserver    # 서비스 ID (컨피그 클라이언트가 어떤 서비스를 조회하는지 매핑)
  profiles:
    active: default         # 서비스가 실행할 기본 프로파일
  cloud:
    config:
      uri: http://localhost:8889  # 컨피그 서버 위치
```

유레카 서버로 지정하기 위해 부트스트래핑 클래스에 `@EnableEurekaServer` 애노테이션을 추가한다.

```java
@EnableEurekaServer
@SpringBootApplication
public class EurekaserverApplication {
    public static void main(String[] args) {
        SpringApplication.run(EurekaserverApplication.class, args);
    }
}
``` 

컨피그 저장소에 유레카 서버에 대한 설정 정보를 셋팅한다.

![컨피그 저장소 디렉토리 구조](/files/posts/20200816/folder.png)

컨피그 서버의 bootstrap.yaml 에 유레카 구성정보 폴더 경로를 추가한다.

```yaml
# configserver > bootstrap.yaml
spring:
  application:
    name: configserver
  cloud:
    config:
      server:
        git:
          uri: https://github.com/juhyun10/config-repo.git
          username: juhyun10
          password: '{cipher}f38ff3546220bbac52d81c132916b1b1fd7c3cfdcfdf408760d1c4bf0b4ee97c'
          search-paths: member-service, event-service, eurekaserver    # 구성 파일을 찾을 폴더 경로
        encrypt:
          enabled: false
```

컨피그 저장소의 유레카 서버 설정을 한다.

```yaml
# config-repo > eurekaserver.yaml
your.name: "EUREKA DEFAULT"
spring:
  rabbitmq:
    host: localhost
    port: 5672
    username: guest
    password: '{cipher}17b3128621cb4e71fbb5a85ef726b44951b62fac541e1de6c2728c6e9d3594ec'
management:
  endpoints:
    web:
      exposure:
        include: "*"
  endpoint:
    shutdown:
      enabled: true
eureka:
  client:
    register-with-eureka: false           # 유레카 서비스에 (자신을) 등록하지 않는다. (클러스터 모드가 아니므로)
    fetch-registry: false                 # 레지스트리 정보를 로컬에 캐싱하지 않는다. (클러스터 모드가 아니므로)
  server:
    wait-time-in-ms-when-sync-empty: 5    # 서버가 요청을 받기 전 대기할 초기 시간 (5ms, 운영 환경에선 삭제 필요)
```

```shell
C:\> mvn clean install
C:\configserver\target>java -jar configserver-0.0.1-SNAPSHOT.jar
C:\eurekaserver\target>java -jar eurekaserver-0.0.1-SNAPSHOT.jar
```

컨피그 서버와 유레카 서버를 띄웠다면 [http://localhost:8761/](http://localhost:8761/) 에 접속하여 유레카 콘솔 화면을 확인해보자.

![유레카 콘솔화면](/files/posts/20200816/eureka_console.png)


콘솔의 "Instances currently registered with Eureka"를 보면 아직 아무런 인스턴스도 등록되어 있지 않다고 나오는데
아직 아무런 유레카 클라이언트가 실행되지 않았기 때문이다.

이제 유레카 서버에 서비스를 동적으로 등록해보자. 

## 3.2. 유레카 클라이언트 구축 (유레카 서버에 서비스 동적 등록)

## 3.3. 서비스 검색 (Feign 사용)

# 참고 사이트
* [스프링 마이크로서비스 코딩공작소](https://thebook.io/006962/)
* [https://docs.spring.io/spring-cloud-netflix/docs/2.2.4.RELEASE/reference/html/](https://docs.spring.io/spring-cloud-netflix/docs/2.2.4.RELEASE/reference/html/)
* [https://coe.gitbook.io/guide/service-discovery/eureka_2](https://coe.gitbook.io/guide/service-discovery/eureka_2)
* 