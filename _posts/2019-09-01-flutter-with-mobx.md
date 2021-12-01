---
layout: post
title:  "mobx를 이용한 flutter 상태 관리"
date:   2019-09-05 00:00
author: q_lazzarus
tags:	[flutter, dart, mobx, state, reactive, android, ios, iphone, google, hybrid]
---

## 들어가며

상태 관리란 무엇일까요? 위키 설명을 따르자면 텍스트 필드 같은 여러개의 UI 컨트롤의 상태를 관리하는 것을 의미합니다. 

예를 들자면, 회원 가입창에 이메일이 유효하면 전송 버튼이 활성화되고 유효하지 않으면 단추가 비활성화되는것 같은 상황은 상태에 따른 UI 컨트롤이 관리되는 경우입니다.

## 그래서 뭐가 다른건데?

사실 이래선 기존이랑 뭐가 다른지 알 수가 없습니다. 여기서는 flutter 를 예를 들어보도록 하겠습니다. flutter의 구성은 widget 들의 조합입니다. 최상위 root widget 에서 하위 widget 을 포함하는 전형적인 tree 구조를 가집니다.

어디선가 비슷하죠?  
react 나 vue.js 같은 frontend framework 가 연상됩니다.  
각 ui widget 들은 다른 widget 들간의 연결은 느슨하게 유지됩니다.
데이터는 bloc 패턴과 같은 stream 에 대한 event 로 리스닝 받고 있고요.  
(자세한 내용은 bloc pattern 이나 reactive programming 개념은 검색 해보세요!) 

![출처: https://www.instiz.net/name_enter/60415754](/files/posts/201909/69ac2650175cc118f093d921e6ad4faa.jpg)
> 버스 타요~


## mobx 로 가즈아

mobx 는 react 계열의 인기있는 상태 관리 라이브러리입니다.  
개인적으로 mobx 를 좋아하기 때문에 flutter 에 mobx 라이브러리 나와서 한번 다루어 보도록 하겠습니다.

자세한 내용은 [React에서 Mobx 경험기 (Redux와 비교기)](http://woowabros.github.io/experience/2019/01/02/kimcj-react-mobx.html) 를 참조하세요.

여기에서는 [mobx.pub](https://mobx.pub) 문서를 참조하였습니다.

## 라이브러리 설치

pubspec.yaml 파일에 참조 라이브러리를 설치합니다.   
여기서는 버전 정보를 일부러 건너 뛰었습니다. [pub.dev](https://pub.dev/) 에서 최신 버전을 참조해서 가져오세요.

```yaml
dependencies:
  mobx: ^0.3.7
  flutter_mobx: ^0.3.2
```

다음에는 dev_dependencies 를 추가합니다.

```yaml
dependencies:
  build_runner: ^1.6.7
  mobx_codegen: 0.3.7
```

아래 명령어로 패키지를 설치합니다.

```bash
flutter packages get
```

## store 를 추가합시다!

여기서는 간단한 counter 샘프을 만들어 보도록 하겠습니다.  
store 를 추가하기 위해 counter.dart 파일을 만들어 보겠습니다.

```dart
import 'package:mobx/mobx.dart';

// Include generated file
part 'counter.g.dart';

// This is the class used by rest of your codebase
class Counter = _Counter with _$Counter;

// The store-class
abstract class _Counter with Store {
  @observable
  int value = 0;

  @action
  void increment() {
    value++;
  }
}
```

추상 클래스 _Counter 는 store 를 포함하고 있으며,  
store 관련 코드는 모두 여기 클래스에 담게 됩니다.  
데이터는 observable annotation 으로 작성하며  
데이터를 조작하는 코드는 action annotation 으로 작성합니다.  
(더 많은 예제는 [mobx](https://mobx.js.org/) 공식 문서를 참조하세요.)

기존 js 에서와 달리 build_runner 로 통해 코드를 생성되게 됩니다.
(webpack 인가봐요)
part 지시자를 통해서 추상 클래스가 변활될 파일들을 지정하게 됩니다.

아래 명령어를 통해 counter.g.dart 파일을 생성합니다.

```bash
flutter packages pub run build_runner build
```

## widget 에 store 를 연결합시다.

```dart
import 'counter.dart'; // Import the Counter

final counter = Counter(); // Instantiate the store
```

생성된 카운터 클래스를 가져오고 선언합니다.

```dart
class MyHomePage extends StatelessWidget {
  const MyHomePage();

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('MobX Counter'),
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: <Widget>[
            Text(
              'You have pushed the button this many times:',
            ),
            // Wrapping in the Observer will automatically re-render on changes to counter.value
            Observer(
              builder: (_) => Text(
                    '${counter.value}',
                    style: Theme.of(context).textTheme.display1,
                  ),
            ),
          ],
        ),
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: counter.increment,
        tooltip: 'Increment',
        child: Icon(Icons.add),
      ),
    );
  }
}
```

참조하시면 StatefuleWidget 을 더이상 사용하지 않는 것을 볼 수 있습니다.
state 는 Counter store 에 보관되며, Observer Widget 를 통해 값을 렌더링하는 것이죠.

FloatingActionButton widget 클릭시 counter.increment action 을 호출하게 됩니다.

## 이제 끝이에요!

와 간단하게 counter 앱이 만들어졌어요!

![출처: https://mobx.pub](
https://mobx.pub/static/img/mobx-counter.aa2d8b3d.gif)

## 후기

이번 내용은 뭔가 깊이가 없어서 아쉽습니다.
다음번에는 좀 더 쉽게 더 재미있게 풀어서 연재해보도록 하겠습니다.

감사합니다.


## 출처 및 참고
 * [state managent](https://en.wikipedia.org/wiki/State_management)
 * [Understanding state management, and why you never will](https://medium.com/flutter/understanding-state-management-and-why-you-never-will-dd84b624d0e)
 * [Reactive Programming - Streams - BLoC](https://www.didierboelens.com/2018/08/reactive-programming---streams---bloc/)
 * [React에서 Mobx 경험기 (Redux와 비교기)](http://woowabros.github.io/experience/2019/01/02/kimcj-react-mobx.html)
 * [mobx.pub](https://mobx.pub)