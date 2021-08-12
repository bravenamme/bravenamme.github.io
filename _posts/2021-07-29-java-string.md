---
layout: post
title:  String , StringBuffer, StringBuilder
date:   2021-07-29 10:00
author: maan
tags:	[java]
---

# String

자바에서 문자열을 다룰 때  주로 String 객체를 사용합니다.   String 객체는 한번 생성되면 할당된 공간이 변하지 않는 불변의 속성을 가집니다.

```java
String str = "Hello World!";
str += " maan";
```

String은 내부 값을 수정할 수 없기 때문에 + 연산자를 사용하면 새로운 객체를 만들어 리턴시켜 줍니다. str객체에  Hello World! 를 선언 한 후 maan이라는 단어는 + 연산을 통하여 추가 하면 str 객체는 "Hello World! maan" 이라는 값을 가지고 있는 새로운 객체를 생성하게 되고 기존의 객체는 가비지([Garbage](https://bravenamme.github.io/2019/08/25/java-gc/))로 있다가 사라지게 됩니다.

그래서 문자열 연산 작업이 많은 경우, 계속해서 새로운 객체를 생성하기 때문에 메모리 부족 현상이 나타 날 수가 있습니다.

![String str = "Hello Wolrd!"](/files/posts/202107/maan_2108121.png)

![문자열추가](/files/posts/202107/maan_2108122.png)

# StringBuffer,StringBuilder

두 클래스는 내부 Buffer에 문자열을 저장해두고 그 안에서 추가, 수정, 삭제 작업을 할 수 있도록 설계되어 있습니다. 그렇기에 String처럼 새로운 객체를 만들지 않고도 문자열을 수정할 수 있습니다.

하지만 StringBuffer나 StringBuilder는 초기에 Buffer 크기를 설정해줘야 하고  문자열 수정을 할 경우에도 정해진 버퍼의 크기를 수정에 맞춰서 변경하는 내부 연산이 필요 합니다.

그렇기 때문에 문자열 추가 변경 작업이 많지 않은 경우엔 고정된 크기를 가지고 있는 String을 사용 하는 편이 더 좋습니다.

StringBuffer와 StringBuilder는 동일한 메소드를 사용합니다.

두 클래스의 큰 차이점은 StringBuffer는 동기화를 지원하고 StringBuilder는 동기화를 지원하지 않는 다는 것입니다. 그래서 StringBuffer는 다중쓰레드 환경에서 안전하고 StringBuilder는 다중쓰레드 환경에서 사용 할 수는 없지만 동기화를 위한 작업이 없기 때문에 단일 쓰레드 환경에서 더 빠르게 동작합니다. 

싱글쓰레드 환경에서의 개발이라면 StringBuilder를 멀티쓰레드 환경이라면 동기화처리가된 StringBuffer를 사용하면 됩니다.

# 문자열 비교

String의 경우 equals 메소드를 이용하여 비교를 하는데요. StringBuffer와  StringBuilder는 동일하게 equals 메소드를 그대로 이용하는 경우 원하는 값이 나오지 않습니다. 

```java
StringBuffer sb1 = new StringBuffer("Hello World!");
StringBuffer sb2 = new StringBuffer("Hello World!");

if(sb1.equals(sb2)){
    System.out.println(true);
}else{
    System.out.println(false);
} //false

//문자열로 변환후 비교 해야 함.
if(sb1.toString().equals(sb2.toString())){
    System.out.println(true);
}else{
    System.out.println(false);
} //true
```

# 주요 메소드

## append()

문자열을 연결할때 사용합니다.

```java
StringBuffer sb = new StringBuffer("Hello");

System.out.println("처음 상태 : " + sb);
System.out.println("문자열 연결 : " + sb.append("World"));

//결과
//처음 상태 : Hello
//문자열 연결 : HelloWorld
```

## insert()

원하는 위치에 문자열을 추가 합니다.

```java
StringBuffer sb = new StringBuffer("Hello");

System.out.println("처음 상태 : " + sb);
System.out.println("문자열 추가 : " + sb.insert(2,"ll"));

//결과
//처음 상태 : Hello
//문자열 추가 : Hellllo
```

## capacity()

length()는 실제 문자열의 길이를 보여주고 capacity()는 현재 버퍼의 용량을 보여 줍니다.

```java
StringBuffer sb = new StringBuffer("Hello");

System.out.println("처음 상태 : " + sb);
System.out.println("문자열의 길이 : " + sb.length() +", " +"용량의 크기 : " + sb.capacity());

System.out.println("문자열 연결 : " + sb.append("World"));
System.out.println("문자열의 길이 : " + sb.length() +", " +"용량의 크기 : " + sb.capacity());
//처음 잡혀 있던 용량이 이미 실제 용량보다 큰 21이었기 때문에 길이가 늘어 나도 용량은 변경되지 않음.

//결과
//처음 상태 : Hello
//문자열의 길이 : 5, 용량의 크기 : 21
//문자열 연결 : HelloWorld
//문자열의 길이 : 10, 용량의 크기 : 21

StringBuffer sb3 = new StringBuffer(10); //버퍼 용량을 10으로 지정.
System.out.println("문자열의 길이 : " + sb3.length() +", " +"용량의 크기 : " + sb3.capacity());
sb3.append("Hello World");
System.out.println("문자열의 길이 : " + sb3.length() +", " +"용량의 크기 : " + sb3.capacity());
//처음 지정된 버퍼 용량이 추가된 문자보다 작았기 때문에 용량이 늘어남.

//결과
//문자열의 길이 : 0, 용량의 크기 : 10
//문자열의 길이 : 11, 용량의 크기 : 22
```

## reverse()

문자열을 뒤집을 때 사용합니다.

```java
StringBuffer sb = new StringBuffer("Hello");
System.out.println("문자열 역순 변경 : " + sb.reverse());

//결과
//문자열 역순 변경 : olleH
```

## delete(), deleteCharAt()

문자열이나 문자를 삭제 할 때 사용합니다.

```java
StringBuffer sb = new StringBuffer("HelloWorld");
System.out.println("문자열 삭제 : " + sb.delete(5,10));
System.out.println("한문자 삭제 : " + sb.deleteCharAt(4));

// 결과
//문자열 삭제 : Hello
//특정 문자 삭제 : Hell
```

# 성능비교

String은 연산을 할 때마다 새로운 문자열 객체를 생성하기 때문에 수행 속도가 매우  느리고 StringBuffer의 경우 동기화 기능으로 인해 상대적으로 StringBuilder 보다 느립니다.

![출처:https://madplay.github.io/post/difference-between-string-stringbuilder-and-stringbuffer-in-java](/files/posts/202107/maan_2108123.png)


# 결론

자바 1.5 부터는 성능 향상을 위해 String 클래스도 내부 연산에서 StringBuilder를 사용 하도록 되어 있다고 합니다. 

단순하게 간단한 문자열추가 정도만 있다면 String을 사용하고 반복문 사용이나 문자열 추가, 수정, 삭제가 빈번하게 일어 나는 경우는 StringBuffer, StringBuilder를 사용 하면 됩니다.

![끝](/files/posts/202107/maan_2108124.png)

# 참고 사이트

[https://coding-factory.tistory.com/546](https://coding-factory.tistory.com/546)

[https://ifuwanna.tistory.com/221](https://ifuwanna.tistory.com/221)

[https://myhappyman.tistory.com/151](https://myhappyman.tistory.com/151)

성능비교 - [https://madplay.github.io/post/difference-between-string-stringbuilder-and-stringbuffer-in-java](https://madplay.github.io/post/difference-between-string-stringbuilder-and-stringbuffer-in-java)