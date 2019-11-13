---
layout: post
title:  "Hello World"
date:   2019-07-11 10:00
author: maan
tags:	[java]
---

## 서론 
어떤 주제로 포스팅의 처음을 써 볼까 한참 고민 하다가 처음이니 익숙한 언어에 대해 써 보기로 했습니다.
현재 7년된 서비스를 운영하고 있는데 실무에 바쁘다는 핑계로 ... 아직 java6으로 개발 된 사이트입니다.... ㅜㅜ

오래된 서비스를 운영 하다 보니 여러 가지 문제가 발생하고 있어서 업그레이드를 해 보려 하는데..
사실 java만 업그레이드 뿐만 아니라 부가적으로 웹서버, spring 버전업등의 작업이 함께 이루어 져야 하니 이 작업이 만만치 않더라구요.

그래서 일단은 자기 계발을 너무 안 한 것에 대한 반성으로 java 8에 대해 살펴 보기로 했습니다.

## jdk8특징
jdk8이 최신은 아니지만 아직은 제일 많이 사용 되고 있습니다.
jdk8의 주목할 만한 특징은 람다식표현식, 스트림 API, 날짜 API의 변경 등이 있는데요. 
개인적으로 날짜 API가 너무 맘에 듭니다.


### 날짜 API 
기존에 날짜 계산을 하려면 달에서 -1도 해줘야 하고 set 해서 before나 after 함수를 사용 하기도 하고
[<a href="https://www.joda.org/joda-time/">joda-time</a>] api를 이용하는 경우도 많았습니다.

```java 
    SimpleDateFormat sdf = new SimpleDateFormat("yyyyMMdd");
    Date today = new Date ( );
    Calendar cal = Calendar.getInstance ( );
    cal.setTime ( today );// 오늘로 설정.
    Calendar cal2 = Calendar.getInstance ( );
    cal2.set ( 2019, 6, 1 );

    System.out.println ( (Integer.parseInt(sdf.format(cal.getTime()))
     - Integer.parseInt(sdf.format(cal2.getTime()))) );
```
가끔 이렇게도 하지요!(저는요...)


하지만 jdk8에서는 날짜 계산이 한 줄로 간단히 구현이 됩니다.
```java
   Duration.between(LocalDateTime.of(2019, 7, 1, 0, 0), LocalDateTime.of(2019, 7, 10, 0, 0)).toDays()
```
핵감동

자바8이 처음 나왔을때는 jpa 버전과 맞지 않아 timestamp 값이 binary 코드로 저장되는 이슈가 있어, 컨버터 클래스를 구현해 줬었는데
최근에 수정 되었다고 합니다.
이건 나중에 spring 공부 하면서 다시 해봐야 겠어요.

### 람다표현식
람다표현식은 자바8의 가장 큰 특징이라 할 수 있는데 표현을 간결하게 해주고 가독성을 높여 준다고 합니다.
익숙치 않은 문장이지만 아래와 같이 간단하게 변경 되어서 좋더라구요.

```java
    for(String str : strLists){
        System.out.println(str);
    }
```
```java
     strLists.forEach((str)->System.out.println(str));
```

### Stream API
자바8에 추가된 Stream API는 람다를 사용하여 자바 컬렉션을 조작 할 수 있게 해 줍니다.
```java
    List<String> strList =new ArrayList<>();
    strList.add("Americano");
    strList.add("Latte");
    strList.add("Cappuccino");

    Collections.sort(strList);
    for(String str : strList) {
        System.out.println(str);
    }
```
```java
     Stream<String> listStream = strList.stream();
     listStream.sorted().forEach(System.out::println);
```

실무에서 Category 라는 enum의 key 값으로 값을 가져 오는 함수인데 
좀 더 보기 편하게 바꿀 수 있습니다.
```java    
    Category cate = null;
    for(Category c : values()){
        if(c.ordinal() == no){
            cate =  c;
        }
    }
```

```java
    Arrays.stream(Category.values())
    .filter(c -> c.getNo() == no)
    .collect(Collectors.toList())
```


## 마치며 
정말 정말 핵 간단하게 java 8에 대해 알아 보았습니다.
다음엔 java8과 spring boot에 대해 공부해 보겠습니다.