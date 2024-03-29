---
layout: post
title:  "개발 비용은 왜이렇게 비싼걸까?"
date:   2021-09-07 10:00
author: kapjong
tags:   [experience]

---
아직까지 연락하고 지내는 친한 대학교 선배님이 한 분 계십니다.<br>
동일한 전자 계산 학부를 졸업하여 IT 생태는 조금 이해하고 계시기는 하나<br>
전공과 무관한 직업을 선택하셔서 실제 IT 업무는 진행해 보지 못하신 분이 있습니다.<br>

>흔히 보는 앱과 웹사이트같이 똑같이 만들면 되는데<br>
>이미 개발되어 있는 코드를 모듈 조립하듯 개발하면 되지 않아?<br>
>소프트웨어 개발 비용은 왜 이렇게 비싼 걸까?<br>

이런저런 사례로 설명을 드렸지만, 명쾌하게 설명을 못 한 것 같아<br>
이번 포스팅은 작성 후 선배님께 한번 공유드릴 생각으로 작성했습니다.<br>

## 지금 시대 개발자
특히 필자 같은 경우라면 인터넷 없이는 개발하기 힘듭니다.
- 검색해서 코드를 찾고 복사해서 붙여 넣기를 합니다.
- 다른 사람이 만들어 놓은 모듈을 통째로 프로젝트에 적용하기도 하고요
- 구글 레퍼런스 사이트를 찾아서 새로 적용된 기술에 대한 이해를 합니다.<br>

지금에 개발은 이렇게 하지 않고서는 개발이 어려울 정도로 고도로 복잡하며<br>
새로운 정보와 기술이 빠른 속도로 나오고 있습니다.

## 잠시 옜날 이야기
![대한민국 1호 컴퓨터 만든 이만영 박사](/files/posts/20210907/img_2.jpg)

소프트웨어가 일반적인 재화 집, 차, 공산품 등을 만드는 일과 가장 차별되는 점은<br>
소프트웨어는 결과물을 100% 동일하게 복사가 가능하다는 점입니다.<br>

컵을 대량 생산한다고 했을 때는 거푸집을 사용하고 그 거푸집에 내구성에는 한계가 있습니다.
하지만 소프트웨어는 컴퓨터와 전기만 있다면 동일한 결과를 무한대로 복사할 수 있다는 점입니다.

여기서 무한대로 복사할 때 발생하는 이익을 누가? 어떻게? 가져가느냐는<br>
컴퓨터가 처음 만들어졌을 때부터 계속 논쟁이 있는 부분입니다.

### [GUN 선언](https://www.gnu.org/gnu/manifesto.ko.html) 
'GNU 선언문'은 카피레프트 운동 선언문입니다.<br>
1989년 GNU GPL 소프트웨어 라이선스가 원조 역할을 한 카피레프트는 <br>
'자유롭게 가져다 쓰되, 도움을 받아서 개발한 소프트웨어는 똑같이 공유한다'라는 기본 철학을 담고 있습니다.<br>
![리처드 스톨만. [사진=미국 지디넷]](/files/posts/20210907/img_1.jpg)

### [오픈소스 운동](https://ko.wikipedia.org/wiki/%EC%98%A4%ED%94%88_%EC%86%8C%EC%8A%A4)
소스(source)는 정보와 지식의 원천입니다.<br>
오픈소스(open source) 운동은 개발자가 개발한 소프트웨어를 무료로 공개하고<br>
소프트웨어를 움직이는 소스 코드 자체도 무료로 공유하자는 생각으로 시작되었습니다.<br>

이런 철학적인 고민을 해준 선구자들 덕분에<br>
>개발자는 인터넷을 찾아보고 코드를 복사 & 붙여 넣기에 죄책감을 덜 느끼셔도 됩니다.

## 소프트웨어의 모듈화
웹 프로그램 입문 시 php 서적을 여러 번 정독하고 목차를 외워 찾아보며 개발한 기억이 있습니다.<br>

![](/files/posts/20210907/img_3.jpg)

은행권 프로젝트에 소속되어 있을 때는 인터넷이 불가능한 환경에 코드 복사&붙여 넣기 없이 날 코딩을 한 적도 있습니다.<br>
소프트웨어는 무에서 유를 창조하는 것이 개발자에 미덕이었던 시대도 있었지만<br>

지금 시대에서 생각하기도 싫은 개발 방법입니다.

### 소프트웨어 공학과 닮은 건축 공학
이미 만들어진 재료(모듈)를 사용해서 통합하는 비슷한 사레로
집 짓기를 예로 들어 볼까요? <br>
단독 주택을 만든다는 가정을 하고 생각해 보면
- 설계도가 있어야 하고 (요건 정의서, 기획서, 설계서)
- 토목 공사를 해야 하며 (서버 인프라 작업)
- 기둥 및 벽채 공사(프로젝트 프레임 워크 작업)
- 목공, 전기, 수도, 가스 공사 (모듈 통합 적용 작업)
- 창틀, 문, 미장, 도배, 마감, 조경 공사 (디자인 적용 작업)
- 관공서 검수 (QA)
- 유지 보수 (유지 보수)

### 모듈화가 잘되어 있는 기능들
 - 로그인 인증 기능
 - 우편번호 찾기 기능
 - 문자 인증
 - 카드 결제 
 - 지도
 - 기타 등등

## 결론
솔루션을 구입해서 사용하는 경우를 제외하고<br>
이미 만들어진 재료를 가지고 통합 것은 정말 어렵고 비용이 많이 들어가는 일입니다.<br>
모듈을 사용하고 통합하는 데 드는 일은 건축, 소프트웨어 개발 시 사람의 노동력이 높게 책정됩니다.<br>
