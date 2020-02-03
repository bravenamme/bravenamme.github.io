---
layout: post
title:  "자바스크립트 콜백지옥 탈출기 - 3.async/await"
date:   2020-02-03 10:00
author: firepizza
tags:	[javascript, promise, async, es6, es8]
---

# async/await

지금까지 콜백지옥을 탈출하기 위해 Promise를 사용하여 적극적으로 해결해 보았습니다.

지난시간에는 제가 마주했던 트위치 API를 사용한 팔로우/언팔로우 기능을 Promise를 이용해 구현했는데요<br/>
마지막에 우리는 한가지 의문을 갖게 되었습니다. 바로 아래와 같은 코드로 마치 동기처리를 하듯이 비동기처리를 순차처리할 수 없을까? 하는 것입니다.

```javascript
const userId = getUserId()
const isFollowing = getIsFolowing(userId)
toggleFollow(isFollowing)

const msg = isFollowing ? '언팔로우 성공' : '팔로우 성공'
console.log(msg)
```

그러나 아시다시피 `getUserId` 라는 함수는 내부적으로 트위치 API서버와 비동기 통신을 하고 있고,<br/>
`getIsFollowing`과 `toggleFollow` 역시 마찬가지로 비동기 통신을 하고 있습니다.
이런 상황에서 위와 같은 처리를 한다면 `userId`가 `undefined`인 채로 다음코드가 실행될 것이고, 내부에서 `userId`를 사용해야 하는 함수는 오류가 발생할 것이며, `isFollowing`이 `undefined`인 채로 진행되고, ......

생각만 해도 끔찍한 결과가 진행될 것임을 알 수 있습니다.

그렇다면 어떻게 이 비동기호출들을 블로킹하며 진행할 수 있을까? 에 대한 답을 ES8에서 제시하고 있습니다. 그것은 바로,

> `async / await`

라는 2개의 키워드가 도입된 것인데요.<br/>
먼저 자세히 알아보기에 앞서 MDN문서를 통해 `async` 키워드의 정의를 가볍게 살펴보겠습니다.

> async 함수에는 await식이 포함될 수 있습니다. 이 식은 async 함수의 실행을 일시 중지하고 전달 된 Promise의 해결을 기다린 다음 async 함수의 실행을 다시 시작하고 완료후 값을 반환합니다.<br/>
await 키워드는 async 함수에서만 유효하다는 것을 기억하십시오. async 함수의 본문 외부에서 사용하면 SyntaxError가 발생합니다.

---

너무 길어서 왠지 복잡한 것 같지만 문장을 잠시 나누어 보자면 이렇게 볼 수 있겠네요.
- 이 식(await)은 async 함수의 실행을 일시 **중지**하고
- 전달된 Promise의 해결을 **기다린** 다음
- async 함수의 실행을 **다시 시작**하고 완료후 값을 반환합니다.

즉,
- 중지하고
- 기다린 다음
- 다시 시작

한다고 요약할 수 있겠습니다.

어떤가요? 우리가 위에서 고민했던 순차적인 비동기호출을 동기적으로 수행할 수 있을 것만 같은 느낌이 들지 않나요?

그렇다면 한번 이전 코드에 async/await 구문을 적용시켜 보도록 하겠습니다.


```javascript
function getUserId () {
  return 
    axios.get('/user')
         .then(r => { return r.userId });
}

function getIsFollowing (userId) {
  return 
    axios.get(`/follow/${userId}/channel/ABC`)
         .then(r => { return r.isFollowing })
}

function toggleFollow (isFollowing, userId) {
  const method = isFollowing ? 'delete' : 'put'
  const msg = isFollowing ? '언팔로우 성공' : '팔로우 성공'
  
  return
    axios.[method](`/follow/${userId}/channel/ABC`)
         .then(r => {
           return { r, msg }
         })
}


async function execute() {
  const userId = await getUserId()
  const isFollowing = await getIsFollowing(userId)
  const result = await toggleFollow(isFollowing)
  console.log(result.msg)
}

execute()
```

어떤가요? 보기에도 훨씬 간단하고 직관적인 코드가 되었습니다.

이처럼 async/await 구문을 활용하면 가독성 높은 코드를 생산할 수 있다는 장점이 있지만, 반대로 async/await에 익숙하지 않은 개발자와 협업하는 경우에는 오히려 생산성을 해칠 수 있습니다.

개발을 진행하기에 앞서 어떤 컨벤션을 가지고 진행할 것인지를 먼저 정하고 진행하는 것이 좋을 것 같다는 생각이 드네요^^

