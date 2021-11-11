---
layout: post
title:  "Annotation 이해하기"
date:   2021-11-10 22:00
author: rman
tags:	[annotation]
---

## 들어가기 앞서

자바를 사용하며 봤었던 @Override 표시.
![override](/files/posts/20211111/override.png)

단순히 "부모에 정의된 메소드를 오버라이딩했나보다."  라며 넘어갔습니다.

어노테이션을 잘 모르는채로 지냈는데, 어느 순간부터 점점 보이기 시작합니다.


![entity](/files/posts/20211111/entity2.png)
![lombok](/files/posts/20211111/getter.png)


큰 의미 없었던 @기호 인줄 알았는데, 아니였습니다.


@Getter, @Setter를 쓰면  알아서 아래 코드를 생성시켜주고
![lombok2](/files/posts/20211111/getter2.png)

@Column 한다음 name에 값을 넣어주면,   테이블 칼럼명에 적히게 됩니다.
![entity](/files/posts/20211111/entity2.png)


이쯤되니 아무 의미 없는 표시가 아님이 느껴졌고, 제대로 공부해봐야겠다는 생각이 들었습니다.

<br/><br/>

## Annotation  

oracle 사이트에는  어노테이션에 대해 아래처럼 적혀있습니다.

![https://docs.oracle.com/javase/tutorial/java/annotations/](/files/posts/20211111/oracle_doc_annotation2.png)

이게 도대체 무슨 말이지..?  


![](/files/posts/20211111/ohno.png)


"프로그램에 대한 데이터를 제공한다"

>컴파일러라는 프로그램에게  "@Override 적힌 메소드는 오버라이딩 목적으로 쓴거야. 만약 개발자가 메소드 이름을 잘못썼다면 알려줘. "<br/>
>컴파일할때 @Getter, @Setter 들어간건 메소드 자동 생성해줘.

이처럼 특정 기능들이 동작할 수 있게 만들어주는 것이 어노테이션 인거죠.


<br/><br/>
```java
@Target({ElementType.FILED, ElementType.TYPE})
@Retention(RetentionPolicy.SOURCE)
public @interface Getter{

}
```

어노테이션의 생김새는 이렇습니다.
- @Target은 어노테이션 기호를 어디에 붙일 수 있는지를 뜻하고,
- @Retention은 어노테이션이 어느시점에 동작하는지를 뜻합니다.

예를 들어 위 Getter는 {ElementType.FILED, ElementType.TYPE} 이렇게 적혀있으니
필드영역에 적힐 수 있고,  클래스, 인터페이스 등에 적힐 수 있다는 뜻입니다.




![](/files/posts/20211111/e_getter_2.png)
![](/files/posts/20211111/e_getter_1.png)


(만약 Target에 적히지 않은 곳에 어노테이션을 쓰면 에러가 발생합니다.)

그럼, 이런 어노테이션을 이용해서 무엇을 할 수 있을까요?

## 어노테이션 활용

어노테이션은 비즈니스 로직에는 영향을 주지 않으면서, 특정 동작을 할 수 있게 만들어줍니다.

```java
@Target({ElementType.METHOD, ElementType.TYPE})
@Rentention(RetentionPolicy.RUNTIME)
public @interface AccessDeny {
}
```

```java

public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler){
    AccessDeny accessDeny = handler.getClass().getAnnotation(AccessDeny.class);

    if(accessDeny != null){
        return false;
    }
    
}
```

이런식으로 적으면, @AccessDeny 가 붙은 클래스나 메소드에 접근하는 요청이 들어왔을때 실행되지 않게 막을 수 있습니다.


또한 spring에서 제공하는 handlerMethodArgumentsResolver를 이용하면 파라미터에 어노테이션을 적은것만으로 조금 더 깔끔하게 만들수 있습니다.


```java
@Controller
public class TestController{
    @GetMapping("/")
    public String index(Model model) {
        SessionUser user = (SessionUser) httpSession.getAttribute("user");

        ~~
    }
}

```

```java
@Controller
public class TestController{
    @GetMapping("/")
    public String index(Model model, @LoginUser SessionUser user) {

        ~~
    }
}
```


```java
@Target(ElementType.PARAMETER)
@Rentention(RententionPolicy.RUNTIME)
public @interface LoginUser{

}
```

```java
public class LoginUserArgumentResolver implements HandlerMethodArgumentResolver {

    private final HttpSession httpSession;

    @Override
    public boolean supportsParameter(MethodParameter parameter) {
        return parameter.getParameterType().equals(LoginUser.class);
    }

    @Override
    public Object resolveArgument(MethodParameter parameter, ModelAndViewContainer mavContainer, NativeWebRequest webRequest, WebDataBinderFactory binderFactory) throws Exception {
        return httpSession.getAttribute("user");
    }
}
```



(스프링 부트와 AWS로 혼자 구현하는 웹서비스 책 참고)


## 마무리

어노테이션을 처음 접했을땐 '이런 애가 있나보다' 였는데,

알아갈수록 활용만 잘하면 정말 매력적이게 사용할 수 있겠구나란 생각이 들었습니다.

읽어주셔서 감사합니다.



