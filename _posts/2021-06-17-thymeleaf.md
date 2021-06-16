---
layout: post
title:  "thymeleaf"
date:   2021-06-16 09:00
author: chyusee
---

# thymeleaf
Spring Boot를 지원하는 템플릿 엔진<br>
FreeMarker, Groovy, Thymeleaf, Mustache<br>
Spring 측에서는 Thymeleaf를 밀어주는 듯 


## 의존성추가
#### maven
```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```

#### gradle
```gradle
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-thymeleaf'
}
```


## 템플릿 형식
템플릿 확장자는 html 을 사용
html 형식이므로 웹 브라우저에서 바로 확인이 가능
기본 파일 위치는 src/main/resource/templete


## 표준 표현식
#### #{...}
메세지 표현식, message properties 내의 값을 출력 
```properties
home.welcome=Welcome to our grocery store, {0}!
```
```html
<p th:utext="#{home.welcome(${user.name})}">
  Welcome to our grocery store, Mr. Kim!
</p>
```


#### ${...}
변수 표현식
```java
model.addattribute("welcome", "Welcome to our fantastic grocery store!");
```
```html
<p th:text="#{welcome}">Welcome to our store!</p>
<p>Welcome to our fantastic grocery store!</p>
```

####*{...}</b>
선택변수 표현식
```java
Map<String, Object> home = new HashMap<>();
info.put("address", "Seoul, Republic of Korea");
info.put("phone", "000-0000-0000");
model.addattribute("home", home);
```
```html
<div th:object="${home}">
    <span th:text="*{address}">...</span>
    <span th:text="*{phone}">...</span>
</div>

<div>
    <span>Seoul, Republic of Korea</span>
    <span>000-0000-0000</span>
</div>
```

***@{...}***<br>
링크 URL 표현식


***th:text***<br>
변수 값을 태그의 텍스트로 출력
기존 텍스트 값은 치환됨

```html
<!-- .properties 내 home.welcome=Welcome to our fantastic grocery store! 이면 아래와 같이 출력 -->
<p th:text="#{home.welcome}">Welcome to our grocery store!</p>
<p>Welcome to our fantastic grocery store!</p>

```

***th:utext***<br>
변수 값을 태그의 이스케이프 되지 않은 텍스트로 출력

```html
<!-- .properties 내 home.welcome=Welcome to our <b>fantastic</b> grocery store! 이면 태그가 이스케이프되어 아래와 같이 출력-->
<p th:text="#{home.welcome}">Welcome to our grocery store!</p>
<p>Welcome to our &lt;b&gt;fantastic&lt;/b&gt; grocery store!</p>

<!-- th:utext를 사용 시 정상적으로 태그가 출력-->
<p th:utext="#{home.welcome}">Welcome to our grocery store!</p>
<p>Welcome to our <b>fantastic</b> grocery store!</p>

```



참고사이트: 

[https://www.thymeleaf.org/]
