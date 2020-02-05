---
layout: post
title:  "java serializable"
date:   2020-02-05 10:00
author: ryan
tags:	[java, serializable]
---

## 직렬화란?

- 직렬화 : JVM의 메모리에 상주되어 있는 객체 데이터를 바이트 형태로 변환하는 직렬화 기술 (자바 내부에서 사용되는 객체/데이터를 외부 자바 시스템에서도 사용할 수 있도록 함)
- 역직렬화 : 직렬화 된 바이트 형태의 데이터를 객체로 변환해서 JVM 에서 사용 하도록 다시 객체로 변환하는 기술

![fig1. 직렬화/역직렬화](/files/posts/202002/20200206_fig1.png) [출처] [https://ktko.tistory.com](https://ktko.tistory.com/)

## 어떤 조건이 필요한가?

**직렬화** :

- "java.io.Serializable" 인터페이스를 상속받은 객체
- 기본형 타입(int, char, String, short, double, long, byte 등등)은 직렬화가 가능
- transient가 사용된 멤버는 전송되지 않는다
- 생성자나 메소드는 직렬화 대상에 속하지 않는다

**역직렬화** :

- 직렬화 대상 클래스가 클래스패스에 존재해야 하며, import 되어있어야 한다
- 자바 직렬화 대상 객체는 동일한 serialVersionUID 를 가지고 있어야 한다
`(private static final long serialVersionUID = 1L;)`

## 자바 직렬화는 왜, 어디서 사용하는가?

CSV, JSON, 프로토콜 버퍼 등은 대부분 데이터 교환 시 많이 사용 되지만 자바 직렬화 형태의 데이터 교환은 자바 시스템 간의 데이터 교환을 위해서 존재 한다. 
**`서블릿 세션 (Servlet Session) 그리고 캐시 (Cache) 에서도 사용 된다`**

![fig2. 자바 직렬화](/files/posts/202002/20200206_fig2.png) [출처] [http://woowabros.github.io/experience/2017/10/17/java-serialize.html](http://woowabros.github.io/experience/2017/10/17/java-serialize.html)

**자바 직렬화는 아래와 같은 장점이 있다...**
- 자바 시스템에서의 개발에 최적화되어 있다
- 복잡한 데이터 구조의 클래스의 객체라도 직렬화 기본 조건만 지키면  바로 직렬화 가능
- 데이터 타입이 자동으로 맞춰진다


## 방법

문자열 직렬화 방법 (익숙한 json 또는 CSV) 과 데이터 변환 및 전송 속도에 최적화하여 이진 직렬화 방법이 있다.

`자바에서 직렬화를 하기 위해서는 "java.io.ObjectOutputStream과 java.io.ObjectInputStream" 패키지를 사용한다`

```java 
out = new ObjectOutputStream(new FileOutputStream("tmp.dat"));
Person p1 = new Person("person1", 29);
Person p2 = new Person("person2", 31);
out.writeObject(p1); out.writeObject(p2);
...
//(중략)
```

## serialVersionUID를 선언해야 하는 이유

JVM은 직렬화와 역직렬화를 하는 시점의 클래스에 대한 버전 번호를 부여 한다. 

만약 그 시점에 클래스의 정의가 바뀌어 있다면 새로운 버전 번호를 할당 하고 직렬화할 때의 버전 번호와 역직렬화를 할 때의 버전 번호가 다르면 역직렬화가 불가능하게 될 수도 있다. 이런 문제를 해결하기 위해 SerialVerionUID 를 사용 한다. 

즉 serialVersionUID 값을 저장할 때 클래스 버전이 맞는지 확인하기 위한 용도이다. 만약 직렬화할 때 사용한 serialVersionUID의 값과 역직렬화 하기 위해 사용했던 serialVersionUID값이 다르다면 InvalidClassException이 발생할 수 있다.

## JPA Entity 에 Serializable 을 상속해야 할까?

직렬화는 JVM 메모리에 상주 되어 있는 객체 데이터를 그대로 영속화 할 때 사용 된다. 시스템이 종료 되더라도 없어지지 않고 영속화되어 네트워크로 전송도 가능 하다.

`“JSR 220: Enterprise JavaBeansTM,Version 3.0 Java Persistence API Version 3.0, Final Release May 2, 2006″에 따르면, 다음과 같이 기술하고 있다.`

`“If an entity instance is to be passed by value as a detached object (e.g., through a remote interface), the entity class must implement the Serializable interface.”`

따라서, 이 객체를 어딘가로 전송하거나 세션에 기록하거나 등등 정말 serialization을 위한 용도가 아니라면, Hibernate상에서는 굳이 Serailizable을 구현하지 않아도 된다는 뜻이지만, 사람들은 흔히 권하기를 그래도 웬만하면 Serializable을 구현하는 것이 좋은 습관일 것이라고 권한다.

예를 들어 서버가 다중화 되어 있고 세션 클러스터링 하는 환경에서 도메인 객체가 세션에 저장 된다면 해당 도메인 객체가 Serializable 인터페이스를 구현해야 정상적으로 세션을 저장/불러오기 할 수 있다.

**Reference**

- [자바 직렬화, 그것이 알고싶다. 훑어보기편](http://woowabros.github.io/experience/2017/10/17/java-serialize.html)
- [https://ktko.tistory.com](https://ktko.tistory.com/entry/JAVA-%EA%B0%9D%EC%B2%B4%EC%9D%98-%EC%A7%81%EB%A0%AC%ED%99%94Serializable-serialVersionUID)
- [https://ryan-han.com/](https://ryan-han.com/)
- [https://okky.kr/article/224715](https://okky.kr/article/224715)