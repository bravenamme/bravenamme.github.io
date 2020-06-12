---
proplayout: post
title:  Spring Cloud Feign
date:   2020-06-18 14:00
author: juhyun10
tags:	[springcloud-feign,MSA,feign]
---









# SpringCloud Feign 이란

MSA (MicroService Architecture) 대해 검토를 하다 보면 분산 시스템에 최적화된 여러 가지 라이브러리들이 소개가 되는데 그 중 하나인 Feign 에 대해 다룹니다.



- Feign 은 REST 기반 서비스 호출을 추상화해주는 Spring cloud Netflix 라이브러리

- Feign을 사용하면 웹 서비스 클라이언트를 보다 쉽게 작성 가능 (코드의 복잡성이 낮아짐)
- 선언적 방식으로 동작 (아래 예제를 통해 Feign 클라이언트 인터페이스 작성 및 호출 방법을 알 수 있습니다.)
  - 선언적 REST 서비스 인터페이스를 클라이언트 측에 작성
  - 이 인터페이스를 통해 REST api 호출
- Spring 이 런타임에 인터페이스의 구현체 제공
  - 개발자는 이 인터페이스의 구현 신경쓰지 않아도 됨



본 예제는 아래와 같은 흐름으로 진행이 됩니다.

![그림으로 이해하는 API 호출 흐름](/files/posts/202006/0618_흐름.JPG)



Provider (localhost:9090) 는 `member/{id}` 말고도 여러 API 들을 제공하는 API 서버입니다.

Consumer (localhost:8080) 는 그 API 들을 이용하는 입장으로서 회원 id 와 함께 Provider 의 `member/{id}` 를 호출하면 Provider는 해당 id 에 해당하는 회원의 이름을 return 합니다.




# SpringCloud Feign 적용해보기
### [Provider]

Provider 는 일반적인 API Application 으로 생성해주시면 됩니다.

```java
@RestController
@RequestMapping(value="/api/v1/provider", produces = MediaType.APPLICATION_JSON_VALUE)
public class ProviderController {
    @GetMapping("member/{id}")
    public String member(@PathVariable("id") int id) {
        // TODO: 회원 id 에 따른 이름 구하는 로직 구현
        String memberName = "LEE";
        return memberName;
    }
}
```

좀 더 자세한 소스는 [여기](https://github.com/juhyun10/feign.git)를 참고해주세요.



### [Consumer]

#### 프로젝트 생성

![springboot project 생성 - Consumer](/files/posts/202006/0618_프로젝트설정.JPG)

```xml
 <dependency>
     <groupId>org.springframework.cloud</groupId>
     <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```



#### 어노테이션 추가 (`@EnableFeignclients`)

main class에 `@EnableFeignclients` 어노테이션을 선언하여 Feign Client를 사용할 것을 알려줍니다.

```java
@EnableFeignClients
@SpringBootApplication
public class DemoApplication {
    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }
}
```



#### Client 작성 (인터페이스)

`@FeignClient` 안의 url 은 요청할(=Provider)의 url 을 넣어주고, 호출하고자 하는 API를 선언해줍니다.



```java
@FeignClient(name = "member-client", url = "http://localhost:9090/api/v1/provider/")
public interface ConsumerClient {
    @GetMapping("member/{id}")
    String member(@PathVariable("id") int id);
}
```





#### Feign Client 호출

이제 위에서 선언한 feign client를 통해 Provider의 GET member/{id} 를 호출해봅시다.

```java
@RestController
@RequestMapping(value="/api/v1/consumer", produces = MediaType.APPLICATION_JSON_VALUE)
public class ConsumerController {
    private ConsumerClient consumerClient;
    public ConsumerController(ConsumerClient consumerClient) {
        this.consumerClient = consumerClient;
    }

    @GetMapping("getMemberName")
    public String getMemberName(int id) {
        return consumerClient.member(id);
    }
}
```



![Consumer API 호출](/files/posts/202006/0618_api.JPG)





RestTemplate을 사용하게 될 경우 http client connection 설정, return 값에 대한 파싱 등 비즈니스 로직 외 셋팅해줘야 하는 것들이 많은 반면에 Feign을 사용하면 dependency 추가, 어노테이션 선언 그리고 호출하고자 하는 api를 인터페이스로 선언해주는 것만으로 REST API 호출이 가능합니다.



관련 소스는 [github/juhyun10](https://github.com/juhyun10/feign.git){:target="_blank"}  에서 확인하실 수 있습니다.



#### 참고사이트

- https://spring.io/projects/spring-cloud-openfeign
- https://woowabros.github.io/experience/2019/05/29/feign.html
- https://cloud.spring.io/spring-cloud-netflix/multi/multi_spring-cloud-feign.html

