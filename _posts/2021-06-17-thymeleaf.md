---
layout: post
title:  "thymeleaf"
date:   2021-06-16 09:00
author: chyusee
---

# Thymeleaf
보안상 이슈로 프로젝트의 스프링 버전을 올려야하는 일이 발생하여 업그레이드를 진행하던 중<br>
사용 중이던 Velocity가 Spring Boot에서 지원을 종료하여 대체할 템플릿 엔진을 검토하게 된 일이 있었습니다.

가능하다면 React 로 컨버팅 했으면 했지만, 컨버팅 하는게 거의 신규 프로젝트 급의 공수가...

Spring Boot를 지원하는 템플릿 엔진 FreeMarker, Groovy, Thymeleaf, Mustache 등 이 있습니다.<br>
각 템플릿 엔진별로 장단점이 있지만, Spring 측에서는 Thymeleaf를 밀어주는 듯하여 선택하게 되었습니다.


## 설정
### Maven Dependencies
```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-thymeleaf</artifactId>
    </dependency>
</dependencies>
```

### Gradle Dependencies
```gradle
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-thymeleaf'
}
```

### application.properties
```properties
# thymeleaf 사용 여부
spring.thymeleaf.enabled=true
# template 경로 접두사
spring.thymeleaf.prefix=classpath:/templates/
# template 경로 접미사
spring.thymeleaf.suffix=.html
# cache 활성화 여부, 개발환경에서는 비 활성화
spring.thymeleaf.cache=true;
# template 인코딩
spring.thymeleaf.encoding=UTF-8
#기본 template 모드, TemplateMode에 정의 (HTML, XML, TEXT, JAVASCRIPT 등)
spring.thymeleaf.mode=HTML
# 렌더링 전에 template 존재 여부 확인 
spring.thymeleaf.check-template=true
# template 위치 존재 여부 확인
spring.thymeleaf.check-template-location=true
```

## 표준 표현식
### 메세지 표현식: #{...}
message properties 에 등록 된 메세지를 표시<br>
매개 변수를 추가하여 사용 가능
```properties
home.welcome=Welcome to our store
home.welcome.name=Welcome to our grocery store, {0}!
```
```html
<p th:utext="#{home.welcome}"></p>
<p th:utext="#{home.welcome.name(${user.name})}"></p>

<p> Welcome to our store</p>
<p> Welcome to our grocery store, Mr. Kim!</p>
```


### 변수 표현식: ${...}
view 영역으로 넘어온 객체를 표시
```java
model.addattribute("welcome", "Welcome to our fantastic grocery store!");
```
```html
<p th:text="#{welcome}">Welcome to our store!</p>
<p>Welcome to our fantastic grocery store!</p>
```

### 선택변수 표현식: *{...}
객체의 일부 프로퍼티를 표시
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
<!-- 위와 동일한 결과를 반환 -->
<div th:object="${home}">
    <span th:text="${home.address}">...</span>
    <span th:text="*{phone}">...</span>
</div>

<div>
    <span>Seoul, Republic of Korea</span>
    <span>000-0000-0000</span>
</div>
```

### 링크 URL 표현식: @{...}
링크 URL 표현식, 주로 th:href 와 함께 쓰임
```html
<a href="details.html"  th:href="@{http://localhost:8080/gtvg/order/details(orderId=${o.id})}">view</a>
<a href="details.html" th:href="@{/order/details(orderId=${o.id})}">view</a>
<a href="details.html" th:href="@{/order/{orderId}/details(orderId=${o.id})}">view</a>


<a href="http://localhost:8080/gtvg/order/details?orderId=3">view</a>
<a href="/gtvg/order/details?orderId=3">view</a>
<a href="/gtvg/order/3/details">view</a>
```

### 텍스트: th:text
변수 값을 태그의 텍스트로 출력, 기존 텍스트 값은 치환됨
인라인 표현식 [[...]] 형태로도 표현 가능
```properties
home.welcome=Welcome to our fantastic grocery store!
```
```html
<p th:text="#{home.welcome}">Welcome to our grocery store!</p>
<p>[[#{home.welcome}]]</p>
<p>Welcome to our fantastic grocery store!</p>
```

### 태그가 제거되지 않은 텍스트: th:utext
변수 값내의 태그가 변환되지 않은 텍스트로 출력<br>
th:text 의 경우 태그가 제거되어 출력 됨<br>
인라인 표현식 [(...)] 형태로도 표현 가능
```properties
home.welcome=Welcome to our <b>fantastic</b> grocery store!
```
```html
<p th:text="#{home.welcome}">Welcome to our grocery store!</p>
<p>Welcome to our &lt;b&gt;fantastic&lt;/b&gt; grocery store!</p>

<p th:utext="#{home.welcome}">Welcome to our grocery store!</p>
<p>[(#{home.welcome})]</p>
<p>Welcome to our <b>fantastic</b> grocery store!</p>
```

### Fragments: ~{...}
템플릿 레이아웃 구성시 사용
```html
<div th:insert="~{commons :: main}">...</div>

<div th:with="frag=~{footer :: #main/text()}">
  <p th:insert="${frag}">
</div>
```

### Literals
#### Text literals
문자열은 '...'로 정의
```html
<p>Now you are looking at a <span th:text="'working web application'">template file</span>.</p>
```

#### Number literals
숫자만 사용
```html
<p>The year is <span th:text="2013">1492</span>.</p>
<p>In two years, it will be <span th:text="2013 + 2">1494</span>.</p>
```

#### Boolean literals
true, false 로 구성
```html
<div th:if="${user.isAdmin()} == false">
```

#### Null literals
```html
<div th:if="${variable.something} == null">
```

### 제어문
#### if: th:if
 ```html
<p th:if="${account.isAdmin()}">관리자</p>
<a th:href="@{logout.html}" th:if="${account.isLogin()}">로그아웃</a>
 ```

#### else: th:unless
 ```html
<a th:href="@{logout.html}" th:if="${account.isLogin()}">로그아웃</a>
<a th:href="@{login.html}" th:unless="${account.isLogin()}">로그인</a>
 ```

#### switch case 
th:block 으로 감싼 후 switch, case 설정<br>
default 는 th:case="*" 형태로 사용
 ```html
<th:block th:switch="${account.grade}"> 
    <span th:case="admin">관리자</span> 
    <span th:case="tester">테스터</span> 
    <span th:case="*">유저</span> 
</th:block>
 ```

### 반복문: th:each
```html
<table>
    <tr>
        <th>NAME</th>
        <th>PRICE</th>
        <th>IN STOCK</th>
    </tr>
    <tr th:each="prod : ${prods}">
        <td th:text="${prod.name}">Onions</td>
        <td th:text="${prod.price}">2.41</td>
        <td th:text="${prod.inStock}? #{true} : #{false}">yes</td>
    </tr>
</table>
```


참고사이트: <br>
* https://www.thymeleaf.org/
* https://docs.spring.io/spring-boot/docs/current/reference/html/application-properties.html
* https://kururu.tistory.com/200
* https://noritersand.github.io/java/java-%ED%83%80%EC%9E%84%EB%A6%AC%ED%94%84-thymeleaf-%EA%B8%B0%EB%B3%B8