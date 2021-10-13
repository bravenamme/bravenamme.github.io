---
layout: post
title:  what's new in Swift UI 3.0
date:   2021-10-08 10:00
author: sunyung
---

## 들어가며
Swift UI가 처음 출시 했을때는 버그가 너무 많아서 실사용에 어려움이 많았다.
하지만 지금은 IOS15, iPadOS15, macOS12, watchOS12 에서만 사용이 가능하다는 단점을 뺀다면, 어느정도 개발도구로 사용하기에 장점이 많다고 생각한다.

그럼 이번에 출시된 Swift UI 3.0 에 주요 업데이트 내용을 살펴보자


## Markdown 정식 지원
Github Readme 에서 많이들 보셧을 법한 Markdown언어를 정식으로 지원한다
아래와 같이 SwiftUI Text에 Markdown 구문을 문자열로 추가할 수 있다.
~~~
Text("**Connect** on [Twitter](url_here)!")
~~~

![https://betterprogramming.pub/](/files/posts/2021_10_08_swiftUI01.png)

## 새로운 버튼 스타일
SwiftUI 의 버튼들이 더욱더 다양해졌다.

![https://betterprogramming.pub/](/files/posts/2021_10_08_swiftUI02.png)


## AsyncImage
지금까지는 ImageView에 URL Image 를 표현 하려면 Loader를 설정하고 비동기 작업을 수행해야만 했다.
하지만 이 과정에서 IOS 의 Table이나 Android의 Recycler View에서 함수 재호출 되면서 깜박이거나 하는 문제가 발생한다.
그래서 대부분은 이 문제를 캐시로 해결하는데 메모리 캐시나, 파일캐시등을 구현해놓은 외부 이미지 오픈소스를 사용해야만 했다.

하지만 SwiftUI3.0 에서는 외부 오픈소스 없이 간편하게 이 작업을 수행시킬 수 있다.
~~~
AsyncImage(url: URL(String: <hereUrl>)!)
~~~

![https://betterprogramming.pub/](/files/posts/2021_10_08_swiftUI03.gif)

또한 현재 오픈소스에서 지원하고 있는 transcation을 통한 애니메이션도 사용 가능하고, scale을 통해 확대 및 축소가 쉽게 제어할 수 있다.
(이제는 오픈소스에 의존하지 않아도 될 것 같다.)

## table 검색
기존 table에는 검색 기능이 없어서 개발자가 한땀 한땀 만들어야만 했다. 하지만 IOS15 에서 지원하는 searchable을 통해 navigationView 안에서 검색기능 UI를 제공할 수 있다. (안드로이드는 예전 부터 제공하던걸 이제서야 지원하다니 많이 늦은 느낌이다.)

![https://betterprogramming.pub/](/files/posts/2021_10_08_swiftUI04.gif)

## Pull to Refresh
pull to refresh 특허권을 가지고 있는 트위터가 2014년에 "모두 사용"으로 전환하면서 IOS에서는 refresh Control 이라는 이름으로 사용되었왔다.

하지만 refresh Control은 사용 방법이 직관적이지 않고, Custom 하기에 번거로운 측면이 있다. 또한 단말기의 네트워크가 소실 될때 간혹 무한루프에 빠지기도 한다.

이부분을 간단히 아래와 같은 코드로 작동 시킬 수 있고 커스텀도 매우 쉽다. 

![https://betterprogramming.pub/](/files/posts/2021_10_08_swiftUI05.gif)

## table item Swipe
Swipe 기능도 엄청나게 간단히 구현이 가능합니다.
~~~
List {
      ForEach (colors, id:\.id){ color in
          Text(color.name)
              .swipeActions{
                  Button(role: .destructive){
                      //do something
                  }label: {
                      Label("Delete", systemImage: "xmark.bin")
                  }

                  Button{
                      //do something
                  }label: {
                      Label("Pin", systemImage: "pin")
                  }
              }
      }
}
~~~

![https://betterprogramming.pub/](/files/posts/2021_10_08_swiftUI06.gif)

## 참고
 * [https://betterprogramming.pub](https://betterprogramming.pub/whats-new-in-swiftui3-ios15-fa0e0d62235b)
 * [What's new In SwiftUI 3.0](https://www.youtube.com/watch?v=SQE5DZDqASA)