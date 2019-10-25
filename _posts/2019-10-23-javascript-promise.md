---
layout: post
title:  "자바스크립트 콜백지옥 탈출기 - 1.영원한 사랑"
date:   2019-10-23 10:00
author: firepizza
tags:	[javascript, promise, async]
---


## Callback hell

비동기 호출이 자주 일어나는 프로그램을 짜다 보면 반드시 한번쯤은 마주할 수 밖에 없는 문제가 있습니다.
그것은 소위 콜백헬 이라고 불리는 것인데요.

![출처: https://adrianalonso.es/desarrollo-web/apis/trabajando-con-promises-pagination-promise-chain/](/files/posts/201910/promise_01.png)
> 콜백장풍을 받아라

---

저의 경우는 주로 다음과 같은 요구사항을 해결하려다 보면 어느순간 콜백헬이 만들어져 있었던 것 같습니다.
- 하나의 비동기 요청이 완료된 뒤, 완료로 인해 얻어진 값을 사용해 다음 비동기요청이 이루어짐.
- 여러 번의 비동기 호출이 이루어지는데 각 처리는 비동기로 이루어지나, 각 비동기호출간의 실행순서는 동기적이었으면 함. 

여러분은 어떤 경우에 이 친구를 만나보셨나요?
어찌되었던, 비동기 프로그래밍을 하다 보면 피할 수 없는 상황인 것은 분명해 보입니다.

프로그래밍의 기원 자체가 비효율적인 작업을 효율적으로 해결하기 위함이었던 것 처럼, 수많은 프로그래머들은

> 그렇다면 어떻게 이 상황을 효율적으로 처리할 수 있을까? 

라는 고민을 해왔을 것이고, 그 결과 다음과 같은 방법으로 조금 더 쉽게 해결할 수 있게 되었습니다.

---

- promise
- async/await
- generator (약간 다른 느낌이긴 하지만..)

---

이 포스트는 위 3가지 주제를 가지고 여러번에 나누어 연재할 예정입니다.!!

그렇다면 먼저, Promise를 사용해 콜백지옥에서 탈출해 봅시다.


# Promise
![출처: https://tartae.tistory.com/entry/%ED%95%91%ED%81%B4-%EC%98%81%EC%9B%90%ED%95%9C-%EC%82%AC%EB%9E%91-%EC%BD%94%EB%93%9C%EC%95%85%EB%B3%B4](/files/posts/201910/promise_03.jpg)
> 약속~ 해~ 줘~~~

2015년 발표된 ECMAScript2015 (ES6)에서 최초 정의된 개념으로 비동기 연산이 종료된 이후의 결과값이나 실패 이유를 처리하기 위한 처리기를 연결할 수 있도록 하는 객체입니다.<br/>
(출처 : [MDN web docs](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Promise))


일반적으로 많은 js라이브러리에서 비동기함수에 대한 리턴값으로 `Promise`를 반환하기 때문에, 손쉽게 사용이 가능합니다.
```javascript
axios.get('')       // get()은 Promise객체를 리턴함.
  .then((res) => {})
  .catch((err) => {})
```

## resolve, reject로 이루어진 내부 콜백

Promise 객체를 생성할 때 인자로 해당요청에 대한 콜백함수를 넘기게 됩니다.

콜백함수로 또다시 2개의 함수가 인자로 전달되는데 첫번쨰는 정상수행 후 실행될 <code>resolve</code>이고, 두번째는 실패 후 실행될 <code>reject</code> 입니다.

비동기 요청이 정상종료 되었는지 여부에 따라 <code>resolve</code>와 <code>reject</code>함수를 적절하게 실행함으로 인해 흐름을 제어할 수 있게 됩니다.

그렇다면 처음에 있었던 콜백장풍의 일부분을 Promise 패턴을 사용해 작성해 보겠습니다.

```javascript
function foo () {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      console.log('foo ok')
      resolve()
    }, 3000)
  })
}

function bar () {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      console.log('bar ok')
      resolve()
    }, 2000)
  })
}

function baz () {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      console.log('baz ok')
      resolve()
    }, 1000)
  })
}

function run () {
  console.log('finish')
}

// 결과물
foo()
  .then(() => {
    bar()
      .then(() => {
        baz()
          .then(() => {
            run()
          })
      })
  })
```

<img src="/files/posts/201910/promise_02.gif">

---

## Promise Hell

> 축하드립니다! 드디어 콜백헬에서 탈출하게 되었습니다.!!<br/>
그런데 이를 어쩌죠, Promise Hell 이 기다리고 있었네요 !

의도된대로 작동은 하고 있지만 위 코드는 뭔가 잘못되었다는 생각이 들 수 밖에 없을 것입니다.<br/>
정말 잘못되었기 때문이죠.

분명 Promise객체를 사용했고, 의도한대로 정상적으로 작동하는 하는데..<br/>
그렇다면 어디가 어떻게 잘못 되었을까요? 그리고 어떻게 해결할 수 있을까요?

MDN에서 제공하는 Promise의 정의를 다시 한번 살펴보면

>Promise는 프로미스가 생성될 때 꼭 알 수 있지는 않은 값을 위한 대리자로, 비동기 연산이 종료된 이후의 결과값이나 실패 이유를 처리하기 위한 처리기를 연결할 수 있도록 합니다. 프로미스를 사용하면 비동기 메서드에서 마치 동기 메서드처럼 값을 반환할 수 있습니다. 다만 최종 결과를 반환하지는 않고, 대신 프로미스를 반환해서 미래의 어떤 시점에 결과를 제공합니다.

라고 정의하고 있는데 중요한 부분은
- Promise는 대리자이며
- 결과나 실패를 처리하기 위한 처리기를 연결할 수 있도록 한다

는 부분이라고 할 수 있겠습니다.

- Promise는 실제 연산을 직접 처리해 주는 객체가 아니라, 해당 연산을 대리해(proxy for a value) 주는 객체 이며,
- 처리기(handler)를 연결할 수 있도록 하고 있다는 말이지요.

> 어떻게 처리기를 연결할 수 있을까?

![Promise의 연결(체이닝)](/files/posts/201910/promise_04.png)

위 도표에서 볼 수 있는 방법과 같이 함수에서 연산을 대리해 주는 Promise를 다시 반환해 그 처리를 다시한번 위임하는 방법은 어떨까요?

위의 Promise Hell을 다음과 같이 바꾸면 처리할 수 있을것만 같습니다.
```javascript
foo()
  .then(() => {
    return bar()
  })
  .then(() => {
    return baz()
  })
  .then(() => {
    run()
  })
```
<img src="/files/posts/201910/promise_05.gif">

짜잔 실행결과는 동일하지만 코드의 가독성이 좀 더 좋아졌고, Promise를 적극적으로 활용한 코드가 만들어졌습니다.!

처음에 보았던 콜백장풍과 비교해 볼 때, 훨씬 발전된 코드의 모습을 하고 있네요.
그리고 비동기로 작성된 함수들을 연이어 동기적으로 실행할 수 있게 되었습니다.

이러한 Promise의 특징을 활용하면 조금 더 효율적이고 가독성높은 코드를 만들 수 있게 되는데요. 다음 편에서는 지금보다 좀 더 적극적으로 Promise 패턴을 활용하는 방법에 대해 소개해 보도록 하겠습니다.


## 참고자료
- [https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise)
- [https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Promise](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Promise)
