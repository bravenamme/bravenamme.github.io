---
layout: post
title:  "Hello World"
date:   2019-07-11 10:00
author: maan
tags:	[java]
---

## 서론 
어떤 주제로 포스팅의 처음을 써 볼까 한참 고민 하다가 처음이니 익숙한 언어에 대해 써 보기로 했다. 
현재 7년된 서비스를 운영하고 있는데 실무에 바쁘다는 핑계로 ... 아직 jdk6으로 개발 된 사이트이다... ㅜㅜ

오래된 서비스를 운영 하다 보니 여러 가지 문제가 발생하고 있어서 업그레이드를 해 보려 하는데..
사실 java만 업그레이드 뿐만 아니라 부가적으로 웹서버, spring 버전업등의 작업이 함께 이루어 져야 하니 이 작업이 만만치 않다. 

그래서 일단은 자기 계발을 너무 안 한 것에 대한 반성으로 java 8에 대해 살펴 보기로 했다. 

## jdk8특징
jdk8이 최신은 아니지만 아직은 제일 많이 사용 되고 있다. 
jdk8의 주목할 만한 특징은 람다식표현, 스트림 API, 날짜 API의 변경등이 있다. 


### 날짜 API 
기존에 날짜 계산이 힘들었음
```java 
    SimpleDateFormat sdf = new SimpleDateFormat("yyyyMMdd");
    Date today = new Date ( );
    Calendar cal = Calendar.getInstance ( );
    cal.setTime ( today );// 오늘로 설정.
    Calendar cal2 = Calendar.getInstance ( );
    cal2.set ( 2019, 6, 1 );

    System.out.println ( (Integer.parseInt(sdf.format(cal.getTime())) - Integer.parseInt(sdf.format(cal2.getTime()))) );
```
간단히 된다!
```java
   Duration.between(LocalDateTime.of(2019, 7, 1, 0, 0), LocalDateTime.of(2019, 7, 10, 0, 0)).toDays()
```

감동
jdk8이 처음 나왔을때는 jpa 버전과 맞지 않아 timestamp 값이 binary 코드로 저장되는 이슈가 있어, 컨버터 클래스를 구현해 줬었는데
최근에 수정 되었다고 한다.
이건 나중에 spring 공부 하면서 다시 해봐야 겠다.

### 람다표현식
람다가 아무리 봐도 익숙해 지지 않는다.
핵 편하고 가독성이 좋다고 하는데 아직은 많이 써 보지 않아서 인지 보기 좋지가 않다.



