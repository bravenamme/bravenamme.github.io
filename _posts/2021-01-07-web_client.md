---
layout: post
title:  "WebClient"
date:   2021-01-07 09:00
author: chyusee
---


### WebClient?

일반적으로 Spring 에서 웹 클라인어트를 이용시에는 RestTemplate를 이용하고 있었습니다.

하지만 Spring 5 버전 부터는 유지관리 모드로 진행되며, WebClient 를 사용하는 것을 권장하고 있습니다.

[https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/client/RestTemplate.html]

WebClient와 RestTemplate의 차이를 간단히 설명하면

RestTemplate은 Blocking I/O 기반의 Synchronous API 

Non-Blocking I/O 기반의 Asynchronous API

입니다. 예를 들어 3초, 5초 가 걸리는 요청을 각각 호출할 경우

RestTemplate은 3초 요청의 응답 처리 후, 5초 요청의 응답을 처리하기 떄문에 총 8초가

WebClient는 각각 응답 처리를 하므로 총 5초가 소요됩니다.


### 의존성 설정
사용하기 앞서 아래와 같이 의존성을 설정합니다.
##### Maven
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-webflux</artifactId>
</dependency>
<dependency>
    <groupId>org.projectreactor</groupId>
    <artifactId>reactor-spring</artifactId>
    <version>1.0.1.RELEASE</version>
</dependency>
```
##### Gradle
```gradle
dependencies {
    compile 'org.springframework.boot:spring-boot-starter-webflux'
    compile 'org.projectreactor:reactor-spring:1.0.1.RELEASE'
}
```

### Instance 생성
WebClient를 사용하기 위해 아래와 같이 간단한 방법으로 생성이 가능합니다.
```java
// 기본 설정으로 생성
WebClient client1 = WebClient.create();

// URL값을 추가하여 생성
WebClient client2 = WebClient.create("http://localhost:8080");
```

WebClientBuilder 클래스를 사용하여 여러가지 설정하여 생성할 수도 있습니다.
```java
WebClient client3 = WebClient
  .builder()
    .baseUrl("http://localhost:8080") 
    .defaultCookie("cookieKey", "cookieValue")
    .defaultHeader(HttpHeaders.CONTENT_TYPE, MediaType.APPLICATION_JSON_VALUE) 
  .build();
```

##### Timeout 설정하기
TcpClient의 ChannelOption.CONNECT_TIMEOUT_MILLIS 값을 통해 Connection Timeout을 설정합니다. 
ReadTimeoutHandler 및 WriteTimeoutHandler를 사용하여 Read Timeout 및 Write Timeout을 설정할 수도 있습니다.
```java
TcpClient tcpClient = TcpClient
  .create()
  .option(ChannelOption.CONNECT_TIMEOUT_MILLIS, 5000)
  .doOnConnected(connection -> {
      connection.addHandlerLast(new ReadTimeoutHandler(5000, TimeUnit.MILLISECONDS));
      connection.addHandlerLast(new WriteTimeoutHandler(5000, TimeUnit.MILLISECONDS));
  });

WebClient client = WebClient.builder()
  .clientConnector(new ReactorClientHttpConnector(HttpClient.from(tcpClient)))
  .build();
```

### Request 요청
##### Method 설정
아래와 같은 방식으로 Http Method를 설정할 수 있습니다.
post, get, put, delete, patch등 모든 Http Method가 정의되어 있습니다
```java
WebClient.create().method(HttpMethod.POST);
WebClient.create().post();
WebClient.create().get();
```
##### URL 설정
아래와 갈이 URL 갑을 직접입력하거나 URI.class 를 이용하여 설정 가능합니다.
```java
WebClient.create().get().uri("/resource");
WebClient.create().post().uri(URI.create("/resource"));
```
그런 다음 필요한 경우 요청 본문, 콘텐츠 유형, 길이, 쿠키 또는 헤더를 설정할 수 있습니다.

예를 들어 요청 본문을 설정하려는 경우 BodyInserter로 작성하거나 이 작업을 게시자에게위임하는 두 가지 방법이 있습니다.
```java
WebClient.create()
  .method(HttpMethod.POST)
  .uri("/resource")
  .body(BodyInserters.fromPublisher(Mono.just("data")), String.class);

WebClient.create("http://localhost:8080")
  .post()
  .uri(URI.create("/resource"))
  .body(BodyInserters.fromObject("data"));
```

##### Parameter 추가
queryParam으로 전달
```java
WebClient.create().get()
	.uri(uriBuilder ->
		uriBuilder.path("/test")
			.queryParam("name", "value")
                        .build()
	).retrieve();
```
formData으로 전달
```java
WebClient.create().post()
	.body(BodyInserters.fromFormData("id","tester").with("password","1234").with(...))
	.exchange();

// MultiValueMap
MultiValueMap map = new LinkedMultiValueMap();
map.add("key1", "value1");
map.add("key2", "value2");

WebClient.create().post()
	.body(BodyInserters.fromFormData(params))
	.exchange();
```
객체로 전달
```java
WebClient.create().post()
	.bodyValue(loginInfo)
	.exchange();
```

##### 호출결과 받아오기
호출 결과를 가져오는 방법으로는 retrieve() 와 exchange()가 존재합니다.
retrieve : ResponseBody 값을 받아와서 bodyToMono()로 빠르게 처리 가능함
```java
Mono<User> result = webClient.post()
   .uri("/login")
   .body(BodyInserters.fromFormData("id","tester").with("password","1234"))
   .accept(MediaType.APPLICATION_JSON) 
   .retrieve() 
   .bodyToMono(User.class);
```
exchange : 상태값, Header 정보를 포함하여 결과를 받아옴
```java
Mono<User> result = webClient.post()
   .uri("/login")
   .body(BodyInserters.fromFormData("id","tester").with("password","1234"))
   .accept(MediaType.APPLICATION_JSON)
   .exchange()
   .flatMap(response -> 
     response.bodyToMono(User.class));
```

여기까지 간단한 WebClient 사용법을 알아보았습니다.

출처: 

[https://www.baeldung.com/spring-5-webclient]

[https://ict-nroo.tistory.com/119]

[https://akageun.github.io/2019/06/23/spring-webflux-4-webclient.html]

[https://medium.com/@odysseymoon/spring-webclient-%EC%82%AC%EC%9A%A9%EB%B2%95-5f92d295edc0]
