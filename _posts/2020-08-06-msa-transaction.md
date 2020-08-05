---
layout: post
title:  "MSA 트랜잭션"
date:   2020-08-06 12:00
author: chyusee
---

## MSA Transaction 

최근 MSA 관련하여 스터디를 하던 중 만일 2개 이상의 서비스에서 데이터를 생성해야할 경우,

하나의 서비스에서만 데이터 생성에 성공하고, 다른 하나가 서비스 장애가 생겨, 생성에 실패할 경우 어떻게 처리해야하지?

라는 의문에서 관련 처리 방법을 찾아보게 되었습니다.

MSA 트랜잭션으로 검색해 볼 경우 크게 Two Phase Commit, Saga Pattern 대한 결과를 쉽게 찾을 수 있습니다.


### Two Phase Commit

Two Phase Commit 방식에서는 Transaction Coordinator가 각 서비스의 Commit. Rollback 을
제어하는 형태로 트랜잭션을 관리합니다.

![예시](/files/posts/20200807/001.png)

쇼핑몰을 예시로 들면 고객이 결재를 완료 했을 경우
1. 재고관리서비스에서는 재고를 차감합니다.
2. 배송 서비스에서는 배송정보를 생성합니다.
3. 주문정보의 상태를 발송으로 변경합니다.

![1. Prepare](/files/posts/20200807/002.png)

Two Phase Commit 방식에서는 우선 각 데이터 Prepare 상태로 처리 합니다.
각 서비스에서 처리가 완료 되었을 경우 Transaction Coordinator에 Commit 준비 완료 메세지를 보냅니다.

![3. Commit](/files/posts/20200807/003.png)

Commit 준비가 완료되면 각 서비스에서 Coomit을 진행합니다.

![4. Error](/files/posts/20200807/004.png)

만일 위와 같이 배송정보 생성 중 오류가 발생할 경우 Transaction Coordinator 실패 메세지를 보냅니다.

![5. Rollback](/files/posts/20200807/005.png)

Transaction Coordinator에서 각 서비스에 Rollback 하도록 지시합니다.

Two Phase Commit의 경우 NoSQL에서는 지원을 하지 않으며, 교착상태가 발생할 우려가 있습니다.


### Choreography-based SAGA

SAGA 패턴은 크게 Choreography, Orchestration 두 개로 구분됩니다.

![6. Choreography-based](/files/posts/20200807/006.png)

Choreography-based SAGA는 위와 같이 재고 차감이 완료되면 배송서비스에 이벤트를 전달하여,

배송서비스에서 배송정보를 생성 후, 주문서비스에 이벤트를 전달하여 주문정보를 변경하는

순차적으로 이벤트로가 전달되면서 트랙잭션이 관리되는 방식입니다.

![7. Choreography-based](/files/posts/20200807/007.png)

만일 위와 같이 배송정보 생성 중 오류가 발생할 경우, 배송에서는 재고관리 서비스로

재고 복원 이벤트를 전달하게 되며, 재고 복원이 만료되면 결재서비스로 결재 취소 이벤트를 전달하여

결재가 취소되는 방식으로 관리됩니다.

### Orchestration-based SAGA





참조
https://cla9.tistory.com/22
https://www.howtodo.cloud/microservice/2019/06/19/microservice-transaction.html
https://microservices.io/patterns/data/saga.html



