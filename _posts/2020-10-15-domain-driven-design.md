---
layout: post
title:  "Domain Driven Design"
date:   2020-10-15 12:00
author: chyusee
---

## Domain Driven Design

MSA 설계에 관해서 알아보다보면, DDD(Domain Driven Design)가 많이 거론 되곤 합니다.

직역하면 **도메인 주도 설계** 라고하는데, 도메인??


### 도메인

DDD에서의 도메인은 개발자 입장에서 보자면 '개발해야 하는 것' 이라고 보는 편이 편합니다.

예를 들어 상품을 등록하고, 결재하고, 배송하는 쇼핑몰을 개발해야 한다고 할 때

예시에서는 쇼핑몰이 도메인이라고 지칭할 수 있을 것입니다. 

그리고 도메인에는 하위 도메인 개념도 존재하며, 

예시에서는 등록, 결재, 배송 등도 하위 도메인 개념으로 볼 수 있습니다.


### 도메인 모델 도출

도메인을 모델링 하기 위해서는 우선 요구사항에서 거기서 핵심 구성요소, 규칙, 구성을 찾아 

데이터, 메소드 등을 정의하는 작업이 필요합니다.

예를 들어 쇼핑몰의 리뷰 페이지의 요구사항들을 정리해보면

- 리뷰는 작성자 정보를 마스킹하여 보여줍니다.
- 리뷰는 구매 한 유저들만 작성할 수 있습니다.
- 리뷰 작성시 1~5사이의 평점을 매길 수 있습니다.
- 리뷰는 500자 글자 제한이 있습니다.

위의 예제에서 리뷰 도메인을 도출해 보면 아래와 같이 표현이 가능합니다.

```java
public class Review {
    
    int reviewerId; // 리뷰 작성 유저아이디 
    int reviewerName; // 리뷰 작성 유저명
    int grade; // 평점 
    String content; // 리뷰 내용
    
    boolean isOrderedUser(customerId) {} // 구매한 유저 유무
    boolean isLimitContentLength {} // 리뷰 글자수 제한 여부 확인
    ...
}
```


### 엔티티와 밸류
도출한 모델은 크게 엔티티와 밸류로 구분할 수 있습니다.

#### 엔티티
- 엔티티의 가장 큰 특징은 고유 식별자를 갖습니다.
- 엔티티 식별자는 바뀌지 않으며, 삭제될때까지 유지됩니다.
- 엔티티 식별자는 아래의 한 가지 방식으로 생성 됩니다.
    - 특정 규칙에 따라 생성
    - UUID 사용
    - 값을 직접 입력
    - 일련번호 사용 (시퀀스나 DB의 자동증가 컬럼 사용)

위의 예제를 기준으로 보면 리뷰들은 각자의 고유 일련번호를 가질 수 있습니다.
```java
public class Review {
    int id; // 리뷰 일련 번호
    int reviewerId; // 리뷰 작성 유저 아이디 
    int reviewerName; // 리뷰 작성 유저명
    ...
}
```

#### 밸류
- 밸류 타입은 개념적으로 완전한 하나를 표현할때 사용합니다.
- 밸류 타입은 반드시 여러개의 데이터를 가져야 하는 것은 아닙니다.<br>
  의미를 명확하게 하기 위해서도 사용 가능합니다.
- 밸류타입을 위한 기능을 추가 할 수 있습니다.
- 벨류 타입은 불변 타입으로 구현하여, 참조 투명성을 보장할 수 있도록 합니다.

예를 들면 위의 위의 리뷰 작성 유저정보는 User라는 밸류 타입으로 표현 할 수 있습니다.
```java
public class User {
    int userId; // 리뷰 작성 유저 아이디 
    int userName; // 리뷰 작성 유저명
    ...
}

public class Review {
    int id; // 리뷰 일련 번호
    User reviewer; // 리뷰 작성 유저 정보 
    ...
}
```


### 도메인 생성시 고려사항
- 도메인 모델에 set 메소드는 핵심 개념이나, 의도를 사라지게 하므로 를 넣지 않습니다.
- 도메인 정보가 불완전한 상태에서 사용되지 않도록 생성 시점에 필요한 데이터를 모두 전달 합니다.
- 도메인 용어를 사용하여 의미를 명확게 표현합니다.

```java
public class Review {
    int id; // 리뷰 일련 번호
    User reviewer; // 리뷰 작성 유저 정보 
    ...

    void setReviewer(User user);
    void setContent(String content);
    ...
}

Review review = new Review();
review.setReviewer(reviewer);

reviewRepository.save(review); // content가 빠진채 저장

Review review = new Review(reviewer, content, grade); // 생성자를 호출하는 시점에서 데이터를 검증
```


```java
public class Review {
    int state; // 0: 정상, 1: 삭제
}


public class ReviewState {
    NORMAL, DELETE; 
}

public class Review {
    ReviewState state;
}
```

### 아키텍처
아키텍처는 크케 표현, 응용, 도메인, 인프라스트럭처 4개의 영역으로 나눠집니다.
<table style="width:100%;border:1px solid #000;text-align:center;">
    <tr>
        <th>계층</th>
        <th>설명</th>
    </tr>
    <tr>
        <td>표현 or UI</td>
        <td>
            사용자의 요청을 전달 받아 응용 영역에 전달하고,<br>
            처리결과를 사용자에게 보여주는 역활을 합니다.
        </td>
    </tr>
    <tr>
        <td>응용</td>
        <td>
            사용자가 제공 할 기능을 구현 합니다.<br>
            업무로직을 직접 구현하는 것이 아니라, 도메인 계층을 조합하여 기능을 구현합니다.
        </td>
    </tr>
    <tr>
        <td>도메인</td>
        <td>도메인 모델을 구현합니다.</td>
    </tr>
    <tr>
        <td>인프라스트럭처</td>
        <td>메세징 송수신이나 RDBMS 연동 등을 처리합니다.</td>
    </tr>
</table>

![출처: https://www.programmersought.com/article/86083216834/](/files/posts/202010/1015_4layer.png)

- 각 계층은 하위계층으로만의 의존만 존재하고, 상위 계층으로는 존재하지 않습니다.
- 하위 계층으로 의존을 막기 위해 필요 시 DIP(Dependency Inversion Principle, 의존역전원칙)을 적용합니다.

다음에는 애그리거트, 리포지터리 구현등에 대해서 알아볼 예정입니다. 