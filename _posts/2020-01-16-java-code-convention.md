---
layout: post
title:  Java Code Convention
date:   2020-01-09 14:00
author: juhyun10
tags:	[java,coding-guide,code-convention]
---

> 본 code convention 은 Oracle Code Convention을 참고로 하되 약간 변형된 부분이 있음을 알려드립니다.<br />
> 예) 한 줄 주석과 꼬리주석의 사용법 

# WHY Code Conventions(코딩 규약)?
- 1인 개발보다는 여러 명이서 협력하는 경우가 더 많다.
- 소프트웨어를 직접 개발한 개발자가 해당 소프트웨어의 서비스가 종료될때까지 유지보수를 담당하는 경우는 드물다.
- 코딩 규약을 지키면 다른 개발자가 소스 코드를 보았을 때 더 빠른 시간안에 이해할 수 있도록 도와준다.


[1. 들여쓰기](#들여쓰기)<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[한 줄의 길이](#한-줄의-길이)<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[줄 나누기](#줄-나누기)<br />
[2. 주석](#주석)<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[블록 주석](#블록-주석)<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[한 줄 주석](#한-줄-주석)<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[꼬리 주석](#꼬리-주석)<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[줄 끝 주석](#줄-끝-주석)<br />
[3. Statements](#Statements)<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[복합문](#복합문)<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[if, if-else, if else-if else](#if,-if-else,-if-else-if-else)<br />
[4. White Space](#White-Space)<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[한 줄 띄우기](#한-줄-띄우기)<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[공백](#공백)<br />
[5. 기타](#기타)<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[괄호](#괄호)


# 1. 들여쓰기
들여쓰기는 4개의 빈 칸을 들여쓰기 단위로 사용한다.
## 한 줄의 길이
한 줄에 80자 이상 쓰는 것은 대부분의 터미널과 툴에서 다룰 수 없으므로 피한다.
## 줄 나누기
하나의 식이 한 줄에 들어가지 않을 때에는 아래 원칙에 따라 두 줄로 나눈다.<br />
&nbsp;&nbsp;&nbsp;&nbsp; - 앞줄과 같은 레벨의 식(expression)이 시작되는 새로운 줄은 앞 줄과 들여쓰기를 일치시킨다.<br />

```java
// 추천
int longAnswer = longNumber1 * (longNumber2 + longNumber2 - longNumber3)
                    + longNumber4 * longNumber5;
// 비추천
int longAnswer = longNumber1 * (longNumber2 + longNumber2
                                    - longNumber3) + longNumber4 * longNumber5;
```
&nbsp;&nbsp;&nbsp;&nbsp; - 위 원칙이 코드를 더 복잡하게 한다면 8개의 빈 칸을 사용하여 들여쓴다.

```java
public static synchorized veryLongMethod(String longArgument1,
        Object longObject1, String longArgument3, String longArgument4,
        String longArgument5) {
    // TODO : something..
}
```

## 메서드 본문 시작시의 줄바꿈
- 일반적으로 메서드 본문이 시작할 때 4개의 빈 칸을 사용하므로, 메서드 본문과 구분하기 위해서 줄을 나누는 경우의 들여쓰기는 8개의 빈 칸 원칙을 사용한다.

```java
// 좋지 않은 들여쓰기
if ((condition1 && condition2)
    || (condition3 && condition4)
    || !(condition5 && condition6)) {
    doSomething();      // 메서드 본문 시작이 명확하지 않다.
}
// 좋은 들여쓰기
if ((condition1 && condition2) || (condition3 && condition4)
        || !(condition5 && condition6)) {
    doSomething();
}
```

# 2. 주석
- 주석은 코드 자체만 가지고는 알 수 없는 추가적인 정보들을 제공하기 위하여 사용되어야 한다.
- 코드 상에 이미 표현되어 있는 중복 정보는 피해야 한다.<br /><br />
** 때로는 코드에 대한 주석이 많이 필요하다는 것은 코드의 품질이 좋지 않다는 것을 의미한다.
주석을 추가해야 한다고 느낄 때에는 코드를 좀 더 명확하게 작성하는 것을 고려해보는 것도 좋다.

## 블록 주석
블록 주석은 파일, 메서드 등의 설명을 제공할 때 사용된다.<br />
블록 주석은 다른 코드들과 구분하기 위하여 처음 한 줄은 비우고 사용한다.

```java
/*
* 여기서부터 블록 주석 시작
*/
```

## 한 줄 주석
짧은 주석은 뒤따라오는 코드와 동일한 들여쓰기를 하는 한 줄로 작성한다.
만약 주석이 한 줄에 다 들어가지 않는다면 위의 블록 주석 형식을 따라야 한다.

```java
if (condition) {
    // 이 조건을 만족하면 실행
    // TODO : something..
}
```

## 꼬리 주석
아주 짧은 주석이 필요한 경우 코드와 같은 줄에 주석을 작성한다.
하지만 실제 코드와는 구분될 수 있도록 충분히 멀리 떨어뜨려야 한다.

```java
if (condition) {
    a = true;       // 참인 경우
} else {
    a = false;      // 거짓인 경우
}
```

# 3. Statements
## 복합문
복합문은 중괄호 { statements } 로 둘러쌓여진 statements 이다.
- 둘러싸여진 statements 은 한 단계 더 들여쓰기를 한다.
- 여는 중괄호 { 는 한 칸 띄운 뒤 복합문을 시작하는 줄의 마지막에 위치한다.
- 닫은 중괄호 } 는 새로운 줄에 써야하고, 복합문의 시작과 같은 들여쓰기를 한다.

## if, if-else, if else-if else
if 문은 항상 중괄호를 사용한다.

```java
if (condition) {
    // TODO : something..
}
if (condition1) {
    // TODO : something..
} else if (condition2) {
    // TODO : something..
} else {
    // TODO : something..
}
// 아래와 같이 중괄호 {} 를 생략해서 사용하지 않는다.
if (condition) 
    // TODO : something..
if (condition) something..
```

# 4. While Space
## 한 줄 띄우기
한 줄을 띄우고 코드를 작성하면 논리적으로 관계가 있는 코드들을 쉽게 구분할 수 있기 때문에 코드의 가독성을 향상시킨다.
- 메서드들 사이에서
- 메서드 안에서의 지역 변수과 그 메서드의 첫 번째 문장 사이에서
- 블록 주석 또는 한 줄 주석 이전에
- 가독성을 향상시키기 위한 메서드 내부의 논리적인 섹션들 사이에서

## 공백
- 괄호와 함께 나타나는 키워드는 공백으로 나눈다.

```java
if (condition) {
    // TODO : do something...
}
```

- 메서드명과 메서드를 여는 괄호 사이에는 공백을 사용하지 않는다.<br />
이렇게 하는 것은 메서드 호출과 키워드를 구별하는데 도움을 준다.

```java
callMethod (arg1);      // 잘못된 사용법
callMethod(arg1);       // 올바른 사용법
 ```

- 공백은 인자리스트에서 콤마 이후에 사용한다.

```java
callMethod(arg1, arg2);
```

- 변수의 타입을 변환하는 cast 의 경우 공백으로 구분한다.

```java
callMethod((byte) num);
```

# 5. 기타
## 괄호
본인이 연산자 우선 순위를 확실하게 알고 있다고 할지라도, 다른 프로그래머를 위해 괄호를 사용한다.

```java
if (a == b && c == d)       // 잘못된 사용법
if ((a == b) && (c == d))   // 올바른 사용법
```

#### 참고사이트
- [Code Conventions for the Java Programming Language](https://www.oracle.com/technetwork/java/javase/documentation/codeconvtoc-136057.html){:target="_blank"}
- [Java Code Conventions](http://kwangshin.pe.kr/blog/java-code-conventions-%EC%9E%90%EB%B0%94-%EC%BD%94%EB%94%A9-%EA%B7%9C%EC%B9%99/){:target="_blank"}

