---
layout: post
title:  No more "MVC"
date:   2019-06-17 10:00
author: sunyung
tags: [ios, mvc, mvvm, viper, ribs]
---

개발자라면 누구나 디자인 패턴에 대해서 한번쯤은 생각해 보았을 것이다.
여러 디자인 패턴을 간략하게 설명하고 앱개발에 유용한 VIPER 아키텍처에 대해서 소개해 보려고한다.

## 인트로
요즘 여러 회사들의 개발자 블로그를 염탐하는 맛에 빠져있다. 유용한 정보도 많고 다른 회사 개발자의 노오오력(?)을 엿볼 수 있어서 정서에 도움이 되는 것 같다.
그러다가 문득  "No more MVC" 라는 흥미로운 제목의 블로그를 보았다. 응? 더이상 MVC 방식으로 개발하지 말라고....? (사실 잘 지키고 있지도 않았다.)


## MVC
![출처:https://developer.apple.com](/files/posts/0617_mvc.png)
Controller는 Model 데이터를 가져와 View에 표시한다. Model이 변경되면 Controller는 View를 업데이트 하거나 새로 만든다. 하지만 IOS는 특정 프레임워크를 사용해야 하므로, MVC패턴을 적용하기가 어렵다.
또한 앱에 기능이 복잡해질수록 Controller의 비중이 점점 커지는 Massive ViewController 형태로 가까워져간다.
ViewController는 View Life Cycle과 매우 강하게 연결되어 있으므로 이것을 확실하게 분리할 수 없고, 재 사용할 수도 없기 때문에 View도 역시 재 사용할 수 없으며, Model 만 재사용 가능한 형태로 남게되는데, 이것은 결국 리소스를 낭비하고 같은 코드가 여러곳에 존재하게 되는 경우를 발생 시킨다.


## MVVM
IOS에서 가장 인기 있는 패턴이 MVVM이다. 가장 중요한 점이 UIViewController 와 UIView가 따로 분류된다는 점이다. 이는 또한 View Model을 테스트 할 수 있으므로 MVVM은 상당히 쓸만한 패턴이라고 할수 있다. View Model은 ViewController 로딩과 의존성이 없으므로 비즈니스 로직을 테스트 할 수 있다.)

다만 MVVM은 바인딩을 도와주는 라이브러리를 함께 사용하지 않으면 많은 기반 코드를 작성해야 한다는 점이다. ReactiveCocoa 와  Rx(Reactive Extentions) 등은 멋진 라이브러리지만 사용법이 쉽지 않아서 익숙해지기까지 상당한 시간이 필요하다. (개발자의 요구에 미치지 못해서 많은 부분 시간을 소비 해야 할 수 도 있다.)

MVVM에는 라우터 역할에 해당하는 것이 없기 때문에 ViewController 와 어떻게 바인딩 할지에 대한 문제가 여전히 남아있다 . WWDC에서 Apple 은 라우터가 없다면, 의존성 주입을 사용하고 권장을 했다.

그러나 많은 의존성이 포함된 뷰 컨트롤러의 코드는 가독성이 매우 떨어진다. 소스 코드를 봐도 어떤 것이 실제로 필요한 것인지 파악하기 힘들 것이다. 뷰 컨트롤러 소스에서 데이터베이스도 사용하고, 이미지 프로바이더도 사용하고, logger도 사용하고, 그 밖에 다른 것들도 사용하거나 실제로 사용하는 것이 없을 수도 있다.

라우터가 없는 MVVM의 단점은 불필요한 의존성이 발생한다는 점이다. 자신이 뷰 모델에 포함된 경우를 제외하고는 뷰 모델이 다른 뷰 모델에 대해 알 필요가 없다.
코드 재사용성도 낮고, if 문이 많이 사용된 스파게티 코드가 된다.

## VIPER

![원문 출처:https://medium.com/ios-os-x-development/ios-architecture-patterns-ecba4c38de52](/files/posts/0617_viper.png)

MVC 패턴을 대체하기 위해 만들어진 것으로 먼저 Entity는 Dumb(말을 하지 않는) Model이라고 불리운다. 이 모델 객체를 조작하는 것이 바로 Interactor 다. 어떤 행동에 따라서 모델 객체를 조작하는 로직이 담겨 있다. 작업이 완료되어도 View에 아무런 영향 없이 오로지 데이터 작업만 한다.

Presenter는 데이터를 Interactor에서 가져오고, 언제 View에 보여줄지 결정한다. View는 Presenter에서 어떻게 보여줘야 할지 요청대로 디스플레이하고, 사용자의 입력을 받으면 다시 Presenter로 넘긴다. Presenter는 View/ViewController, Interactor, Router와 상호작용한다. Interactor로부터 조작된 데이터를 가져오고, 디스플레이하기 위해 데이터들을 준비한 다음 View/ViewController에 보낸다.

Router는 화면 전환을 담당한다. Router는 화면 전환 애니메이션을 구현하고, View Controller를 생성하여 Presenter와 연결한다.

|---|:---:|
| `View` | Presenter의 요청대로 화면에 Display 하고, 사용자의 입력을 Presenter로 보내는 역할을 함 |
| `Interacter` | Use Case에 따라서 Entity 모델 객체를 조작하는 로직. (API로부터 Data를 받아 Model을 생성 |
| `Presenter` | Interactor로부터 데이터를 가져오고, View로 보내기 위해 데이터를 준비하여 “언제” View에 보여줄지를 결정함 |
| `Entity` | 모델 객체 |
| `Router` | Router는 화면 전환 애니메이션을 구현하고, View Controller를 생성하여 Presenter와 연결 (Navigation Logic을 담당) |

하지만 Viper를 제대로 사용하기 위해서는 사전에 많은 클래스를 생성해야 한다는 단점이 있다.
이러한 문제를 해결하기 위해 Viper 클래스를 자동으로 생성해주는 code generator  바로 Uber에서 만든 Ribs 이다.


## Ribs IOS, Android 샘플코드 분석 (feat. 반응형)
![출처:https://github.com/uber/RIBs](/files/posts/0617_tictactoe.png)

![출처:https://github.com/uber/RIBs](/files/posts/0617_and_tictactoe.png)

LoggedOutViewController 에서 “로그인” 버튼 액션을 받는다
LoggedOutPresentableListener 을 호출 한다.

이름을 받기전까지 버튼을 disable 하는 부분은 LoggedOutPresentableListener 프로토콜을 따르도록 LoggedOutInteractor를 수정을 한다.
그래서 결국 버튼을 터치한경우 LoggedOutViewController 리스터의 함수를 호출하면 LoggedOutInteractor의 로직을 따르게 된다.
LoggedOutInteractor 에 로그인 완료 리스너를 추가 한다. LoggedOutInteractor 에 이전에 작성한 login 함수의 구현을 변경하여 새로 선언 된 리스너 호출을 추가한다.
RootInteractor에서 프로토콜을 업데이트하고 RootInteractor 클래스에 didLogin함수를 업데이트 한다.


## 결론

Viper를 사용하는 사람들은 "자잘한 레고를 가지고 여럿이서 오랜시간에 걸쳐  엄청나게 큰 건물을 짖는 느낌"이라고 한다. 
개인적으로 지금 당장 앱에 Viper를 적용시키는 것은 무리가 있고, 대형 프로젝트가 아니라면 좀더 단순한 패턴을 고려하는게 좋다고 생각한다.

몇몇 사람들은 유지보수 비용이 엄청나게 들더라도, 적어도 미래의 언젠가는 Viper 패턴으로 이득을 볼 것이라고 생각하고 강행 하기도한다. 
하지만 왠지 "호미로 막을 것을 가래로 막느 것" 같은 느낌이 아니 아니 들 수 없다.

결국 아키텍처를 설계하는 문제는 완벽한 정답이란 없고, 프로젝트에 따라 상황에 맞게 저울질 하는게 중요하다고 생각한다.
