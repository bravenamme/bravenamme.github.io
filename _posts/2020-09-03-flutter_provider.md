---
layout: post
title:  "StatelessWidget, StatefulWidget, Provider"
date:   2020-09-03 10:00
author: kapjong
tags:   [flutter, provider]

---

# Application life cycle 관리
안드로이드 app, IOS app 모두 life cycle 관리가 필요합니다.<br>
Android는 코드에 기본이 되는 "extends Activity"가 그역활을 하고 있고<br>
Flutter는 상속 받아 만들 수 있는 StatelessWidget과 StatefulWidget가 life cycle 관리를 수행 합니다.

## 1, StatelessWidget
- 단 한번 만 Build 과정이 일어나고,
- 한번 그려진 화면은 계속 유지되며,
- 성능 상 장점이 생김니다.

## 2, StatefulWidget
- state를 포함한 Widgetd이며,
- setState 가 발생할때마다 다시 Build 과정이 일어고,
- 때문에, 동적 화면을 쉽게 구현이 가능합니다.

# Application 데이터 관리를 위한 Provider

## Provider가 주목받게된 배경

구글에서 Flutter 발표 시점에는 Bloc 패턴을 사용하길 권했습니다.<br>
Bloc 패턴은 UI와 데이터 처리 로직(비즈니스 로직)을 분리하는 방식입니다.<br>
### flutter UI 작성도 코드다 보니, 잘 관리 안 해주면 코드가 한 클래스에 너무 많이 몰리는 문제가 있습니다.

이를 해결하기 위해 나온 게 Bloc 패턴인데...<br>
BLoc 패턴은 사용하기 너무 어렵다는 여론이 많았습니다.<br>
단순한 로직을 짜려고 해도 최소 4개 정도는 클래스를 만들어야 합니다.
### 반면 Provider 패턴을 쓰면 데이터 공유나 로직의 분리를 좀 더 간단히 할 수 있습니다.

Provider는 Google IO (2019 Google IO) 에서 추천되면서 큰 주목을 받았습니다.<br>
커뮤니티에서 만든 플러그인인데 공식으로 승격 하였습니다.<br>
- [Flutter Package provider](https://pub.dev/packages/provider)

## Provider 패턴을 쓰는 이유는?
1, 관심사의 분리<br>
관심사의 분리는 디자인 원칙의 하나입니다, 보통 관심사는 어떤 코드가 하는 일을 말합니다.<br>
UI를 담당하는 코드, 네트워크를 담당하는 코드, 데이터를 담당하는 코드 등 코드를 역할에 따라 나눌 수 있죠.

>
Provider를 쓰는 이유는 관심사의 분리를 위해서입니다.

2, 데이터의 공유<br>
하나의 데이터를 여러 페이지에서 공유하고 싶을 때가 있습니다.<br>
로그인 사용자의 정보나, 여저가지 앱 환경 정보 등은 여러 페이지에서 쓰이죠.<br>
근데 페이지마다 인증 정보를 새로 불러온다면 앱이 복잡해지고, 비용도 많이 발생 합니다!
이럴 때 데이터 공유가 필요합니다.

>
Provider 패턴을 쓰면 데이터 공유를 쉽게 할 수 있습니다.

3, 좀 더 간결한 코드<br>
Bloc 패턴의 경우 클래스들을 역할 별로 나누는 데는 좋지만 코드 자체가 복잡해지는 경향이 있습니다.<br>
Provider 패턴을 쓰면 좀 더 적은 코드로 클래스들을 구분해서 쓸 수 있습니다.

>
구글에서도 중규모 프로젝트는 Provider 패턴을, 대규모 프로젝트는 Bloc 패턴을 추천하고 있습니다.

## 결론
1, react 경험상 앱 크로스 플랫폼은 UI 상태 변화에 대한 코드 작성이 조금은 난이도가 있을줄 알았으나<br>
Flutter에 경우 StatefulWidget 지원으로 기존 네이티브 코드와 동일한 난이도로 UI 상태 변화 코드 작성이 가능 하였습니다.

2, 하나에 Application 프로젝트가 시작될때 경험상 가장 신경 쓰이는 부분이 Application 데이터 공유 입니다.<br>
다음 Flutter를 사용한 프로젝트가 진행 된다면 Provider를 적극 활용할 예정 입니다.
