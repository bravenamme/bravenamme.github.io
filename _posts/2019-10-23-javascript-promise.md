---
layout: post
title:  "Promise"
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

짐작하셨다시피 이 포스트는 위 3가지 주제를 가지고 3회에 걸쳐 연재할 예정입니다.!!
그렇다면 먼저, Promise를 사용해 콜백지옥에서 탈출해 봅시다.

# Promise
2015년 발표된 ECMAScript2015 (ES6)에서 최초 정의된 개념으로 비동기 연산이 종료된 이후의 결과값이나 실패 이유를 처리하기 위한 처리기를 연결할 수 있도록 하는 객체입니다.<br/>
(출처 : [MDN web docs](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Promise))


일반적으로 많은 js라이브러리에서 비동기함수에 대한 리턴값으로 `Promise`를 반환하기 때문에,
손쉽게 체이닝이 가능합니다.
```javascript
axios.get('')       // get()은 Promise객체를 리턴함.
    .then((res) => {})
    .catch((err) => {})
```

그러나 Promise를 좀 더 적극적으로 활용하기 위해 커스텀해서 사용하는 경우도 있습니다.
처음에 있었던 콜백장풍의 일부분을 Promise 패턴으로 바꿔 보았습니다.
```javascript
function foo () {
    return new Promise((resolve, reject) => {
        setTimeout(() => resolve('foo'), 3000);
    })
}

function bar () {
    return new Promise((resolve, reject) => {
        setTimeout(() => resolve('bar'), 2000);
    })
}

function baz () {
    return new Promise((resolve, reject) => {
        setTimeout(() => resolve('baz'), 1000);
    })
}

function run () {
    console.log('finish')
}


foo()
    .then((res) => {
        console.log(res)
        return bar()
    })
    .then((res) => {
        console.log(res)
        return baz()
    })
    .then((res) => {
        console.log(res)
        run()
    })
```

위 코드의 수행결과는 어떨까요 ??

정답은
```
finish
baz
bar
foo
```
가 아닌
```
foo
bar
baz
finish
```
가 됩니다.

마치 비동기블럭들이 동기적으로 실행되는 모습을 보이네요..
축하드립니다! 드디어 콜백헬에서 탈출하게 되었습니다.!!

