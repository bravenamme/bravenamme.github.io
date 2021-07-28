---
layout: post
title: "java 커스텀 어노테이션"
date: 2021-06-24 10:00
author: firepizza
tags: [java, annotation]
---

# Annotation

- Annotation을 직역하면 주석 이라는 뜻인데, 간단히 말해 소스에 메타 데이터를 주입시켜주는 일종의 표식 정도로 생각하면 좋겠습니다.
- 스프링 으로 구성된 웹서버에서 흔히 사용하는
  > <code>@Controller</code>, <code>@Service</code>, <code>@Repository</code>
- 등을 예로 들 수 있겠네요.

- 물론 다양하고 광범위한 활용 방법이 있겠지만..<br/>
  직접 사용해 보았던 것을 소개하고, 방법을 이해하신 뒤에 응용해 보시는 편이 더 좋을 것 같습니다.!

## 어노테이션을 만들자

간단한 커스텀 어노테이션을 만들어 보겠습니다.

```java
// MyAnnotation.java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Inherited
@Documented
public @interface MyAnnotation {
}
```

- MyAnnotation 이라는 이름의 커스텀 어노테이션이 간단히 만들어졌습니다.
- 위에 여러 가지 메타 어노테이션들이 추가되어 있는데요 다음과 같은 의미를 갖고 있습니다.

- @Target(ElementType.TYPE) => 이 어노테이션은 TYPE선언시에 사용되어야 한다.
  > ElementType에는
  >
  > - PACKAGE (패키지에 사용)
  > - TYPE (타입(클래스)에 사용)
  > - CONSTRUCTOR (생성자에 사용)
  > - FIELD (필드에 사용)
  > - METHOD (메소드에 사용)<br/>
  > - ...
  > - 등이 있습니다.
- @Retention(RetentionPolicy.RUNTIME) => 이 어노테이션은 Runtime 시에도 살아 있어 참조가능하다.

  > RetentionPolicy에는
  >
  > - SOURCE (컴파일 시에만 사용 됨)
  > - CLASS (클래스 파일에는 포함되나, 런타임에 참조될 수는 없음)
  > - RUNTIME (클래스파일에도 포함되고, 런타임에도 reflectively하게 참조 가능함.)
  > - 이 있습니다.

- @Inherited 를 선언하면, 해당 어노테이션이 하위 클래스까지 적용되게끔 하며 (미 선언시 하위까지 전파되지는 않음)

- @Documented 를 선언하면, JavaDoc을 생성할 때 포함되게 됩니다.

## 만들긴 했는데 어떻게 쓸 수 있을까?

- 만드는 거야 간단하지만, 그렇다면 이걸 어떻게 사용할 수 있을까요?
- 제가 실무에서 사용 했던 방법은 Spring AOP와 결합시켜 AOP대상을 뽑아내서 선별적으로 적용하는 방법 이었습니다.

- 해당 내용을 그대로 기술하기엔 좀 그래서 (아시죠?).. 조금 단순화시킨 모델로 예를 들어 보겠습니다.
- 물론 지금 들게 되는 예시에서는 당연히 이런 처리보다 훨씬 간단하게 처리할 수 있는 방법이 있겠습니다.

- 예를 들어 이런 데이터모델이 있다고 가정 해 보겠습니다.

```java
public class Member {
  private String name;
  private int age;
  // getter, setter
}
```

- 그리고 age가 30 이상인 경우 name 앞에 접두사로 "[old]" 라는 문자열을 붙여주는 처리를 하고 싶다고 가정 해 보겠습니다.
- 앞서 말씀드렸듯이 물론 getName() 함수를 이용하거나 , setter를 사용해 age와 연동되는 name필드의 접두사 처리를 할 수도 있고, 접두사를 붙여주는 별도 함수를 통해 처리할 수도 있겠지만..
- 여기서는 한번 커스텀 어노테이션을 이용해 필드에 값을 주입해보도록 하겠습니다.

- 먼저 어노테이션이 필요하겠네요

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface AddPrefixOld {
}
```

- 그 다음 실제 사용하는 코드에서 어노테이션을 적용 하구요

```java
@AddPrefixOld
public Member someFunction() {
  Member member = doSomething();
  return member;
}
```

- pointcut을 통해 해당 함수를 캐치하게 한 뒤,
- 잡힌 pointcut에서 실행되는 advice 함수를 작성 해 봅니다.

```java
public Member someAdvice(JoinPoint jp) {
  
  // 리플렉션으로 AOP 대상 메소드 시그니쳐를 얻어 Annotation 보유 여부를 확인
  MethodSignature signature = (MethodSignature) jp.getSignature();
  Method method = signature.getMethod();
  AddPrefixOld annotation = method.getDeclaredAnnotation(AddPrefixOld.class);
  Member targetObj = jp.proceed();
  if (annotation != null) {
    // AddPrefixOld 어노테이션을 보유한 경우 특수 처리 추가
    if (targetObj.getAge() > 29) {
      targetObj.setName("[old]"+targetObj.getName());
    }
  }
  return targetObj;
}
```

- 이런 처리를 통해 이제 Member 객체를 리턴하는 모든 메소드에는 @AddPrefixOld 어노테이션을 추가하는 것 만으로 (Annotate 하는 것만으로) "[old]" 라는 접두사를 붙이는 공통 처리가 가능해졌습니다.
- 물론 해당 메소드 실행구간이 pointcut에 잡혀 있어야 겠지만요.
- 어노테이션을 활용할 수 있는 방법이야 무궁무진 하겠고, 생각하기 나름이겠지만<br/>
이런 방법으로도 사용해 본 적이 있다는 방법을 소개 해 드리고 싶었습니다.
- 지금 생각해보면 사실 올바른 사용법 인지에 대한 검증은 되어 있지 않으나.. 이것저것 해 보며 배울 수 있는 부분이 많았다고 생각 되어요.
- 물론 어렵겠지만 개발 하면서 돌아가네.. 하고 넘어가는 것도 좋지만 이렇게해볼까? 저렇게해볼까? 하는 고민을 하는 과정 또한 너무 재밌는 과정인 것 같아요

- 읽어주셔서 감사합니다.