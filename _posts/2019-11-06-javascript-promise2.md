---
layout: post
title:  "자바스크립트 콜백지옥 탈출기 - 2....."
date:   2019-11-06 10:00
author: firepizza
tags:	[javascript, promise, async]
---

# Promise 
지난 시간에는 콜백지옥 이라는 문제에 좌절했다가, Promise를 리턴하는 함수의 체이닝을 통해 문제를 해결해 보았었습니다. 그렇다면 이번에는 조금 더 적극적으로 Promise를 활용하는 방법에 대해 알아보겠습니다.

## 현실의 문제
최근에 트위치 API를 활용해 특정 스트리머를 팔로우/언팔로우 하는 기능을 개발했을 때 만났던 문제인데요, 어떻게 해결할 수 있을지와, Promise를 통해 해결한 방법을 소개해 볼까 합니다.

---
### **요구사항은 아래와 같았습니다.**
- ABC 라는 채널을 팔로우중인지 여부에 따라 팔로우/언팔로우 요청을 보내야 함 
  1. 팔로우여부를 먼저 알아내고..
  1. 해당 여부에 따라 각기 다른 API호출을 해야 함.
- 위에서 필요한 호출을 보내기 위해서는 로그인한 유저의 고유id가 필요함
  - 별도 API호출을 통해 획득

---
사용하는 API는 총 4가지 입니다.

편의상 API 호출에 필요한 몇가지 파라미터와 URL은 생략하겠습니다.
  1. 팔로우 여부를 확인한다. (GET /follow/유저고유ID/channel/ABC)
  1. 채널을 팔로우한다. (PUT /follow/유저고유ID/channel/ABC)
  1. 채널을 언팔로우한다. (DELETE /follow/유저고유ID/channel/ABC)
  1. 유저고유 ID를 얻는다. (GET /user)<br/>
    - 헤더에 로그인중인 유저의 OAuth Token을 담아야 함. (아래 예제에서는 담았다고 치겠습니다.)

---


단순하게 로직 흐름만을 생각한다면 아래와 같은 코드로 간단히 해결할 수 있을 것 같네요.
```javascript
axios.get('/user')  // 유저 고유 ID를 얻고
  .then((id) => {
    axios.get(`/follow/${id}/channel/ABC`)  // 얻은 고유ID를 통해 팔로우 여부를 확인하고
      .then((isFollowing) => {
        if (isFollowing) {
          axios.delete(`/follow/${id}/channel/ABC`) // 팔로우 중이라면 언팔로우한다.
            .then(() => {
              console.log('언팔로우 성공')
            })
        } else {
          axios.put(`/follow/${id}/channel/ABC`)    // 팔로우 중이 아니라면 팔로우한다.
            .then(() => {
              console.log('팔로우 성공')
            })
        }
      })
  })
```

그러나 짐작 하셨겠지만, 위 코드는 다양한 문제점을 내포하고 있는데요.
일단 제가 보기에는 이러한 문제가 있어 보이네요.
> 1. Promise를 사용하는 axios 라이브러리를 사용했음에도 가독성이 좋지 않다. (콜백지옥)
> 1. 비지니스 로직 흐름 사이에 다른 로직이 끼어들어야 하는 요구사항에 대응하기 어렵다. (유지보수 난이도 상승)
> 1. 위 예제에서는 생략했지만, 각각의 호출에서 발생하는 예외상황에 대한 각기 다른 처리가 필요하다면 가독성과 유지보수 문제는 더 심각해 진다.


## Promise를 최대한 활용해 리팩토링

이전 시간에 언급했던 Promise의 특징으로는
1. Promise는 연산 대리자이다.
1. 각 연산의 handler를 연결할 수 있도록 하고 있다.

가 있었는데요.

axios를 사용한 비동기 호출 함수들 역시 [Promise를 리턴](https://github.com/axios/axios)하고 있기 때문에 위 특징을 활용해 리팩토링 해 보겠습니다.

>1.5버전 이상의 jQuery.ajax 에서는 [jqXHR](https://api.jquery.com/Types/#jqXHR) 객체를 사용하고 있으나 내부적으로 Promise를 사용하기 때문에 활용 가능함.

### 먼저 thenable 체인을 만들고.
```javascript
axios.get('/user')  // 유저 고유 ID를 얻고
  .then((id) => {
    return axios.get(`/follow/${id}/channel/ABC`)  // 얻은 고유ID를 통해 팔로우 여부를 확인하고
  })
  .then((isFollowing) => {
    if (isFollowing) {
      return axios.delete(`/follow/${id}/channel/ABC`) // 팔로우 중이라면 언팔로우한다.
    } else {
      return axios.put(`/follow/${id}/channel/ABC`)    // 팔로우 중이 아니라면 팔로우한다.
    }
  })
  .then(() => {
    console.log('팔로우/언팔로우 성공')
  })
```

### 가독성을 높이기 위해 함수단위로 리팩토링한다.
```javascript
function getUserId () {
  return axios.get('/user')
}

function getIsFollowing (userId) {
  return 
    axios.get(`/follow/${userId}/channel/ABC`)
      .then((isFollowing) => {
        return { isFollowing, userId }
      })
}

function toggleFollow (isFollowing, userId) {
  const method = isFollowing ? 'delete' : 'put'
  const msg = isFollowing ? '언팔로우 성공' : '팔로우 성공'
  return 
    axios.[method](`/follow/${userId}/channel/ABC`)
      .then((result) => {
        return { result, msg }
      })
}

getUserId()
  .then((id) => {
    return getIsFollowing(id)
  })
  .then((res) => {
    return toggleFollow(res.isFollowing, res.userId)
  })
  .then((res) => {
    console.log(`${res.msg} 성공`)
  })
```

위 2번의 리팩토링을 통해서, Promise를 사용해 체이닝을 하며 가독성 좋은 코드를 만들어 보았습니다.<br/>
그런데 코드를 자세히 들여다 보면 어딘가 부자연스러운? 어색한? 부분이 있는 것 같습니다. 여러분은 어느 부분이 어색하게 느껴지시나요?<br/>
<code>getIsFollowing()</code>함수를 다시 한번 봐 볼까요?
이름으로 볼 때 팔로우 상태를 얻는 역할만을 해야 할 것 같은 함수인데.. 내용을 들여다 보면 그렇지 않죠. 팔로우상태를 얻은 다음 그 결과와 함께 다음 호출에서 사용하기 위한 userId를 결과에 주입해 리턴하고 있습니다.
이번 예제에서는 바로 다음 then절에서 필요했기 때문에 간단해 보이지만, <code>getIsFollowing()</code> 이후에 userId가 필요없는 로직이 오거나 다음다음 then절에서 값이 필요하게 되는 경우를 상상해보면 복잡도가 점점 커진다는 걸 알 수 있습니다.

그렇다면 이러한 요구사항(특정 Promise의 결과값을 다른 then절에서 재사용하고 싶다)을 해결하기 위해서 이런 의문이 들 수 있습니다.

> 지나간 then절에서 사용된 Promise의 결과값은 미래에 나올 then절에서 재사용이 불가능한가?

정답은... 그렇습니다. 아쉽게도 ECMAscript6 스펙에서는 위와 같은 방식 또는 지나간 then절에서 얻어낸 결과값을 전역변수로 캐시해두고 다음 then절에서 사용할 수 밖에 없습니다.

예제에서는 간단한 데이터 하나만을 재사용하고 있기 때문에 저렇게 해도 되지 않나? 하는 생각이 들 수 있지만, 만약 전달해야 하는 데이터가 복잡하고 개수가 늘어난다면 어떨까요?

그러면 일단 현존하는 문법과 기술은 무시하고 아래 코드와 같은 방법으로 비동기처리를 동기처리할 수는 없을까요?

```javascript
const userId = getUserId()
const isFollowing = getIsFolowing(userId)
toggleFollow(isFollowing)

const msg = isFollowing ? '언팔로우 성공' : '팔로우 성공'
console.log(msg)
```

위와 같은 처리가 가능하다면 훨씬 가독성 좋고, 개발하기 수월한 코드가 될 것만 같은 생각에 행복해집니다.

제가 짧은기간이지만 주니어개발자로 개발을 하면서 깨달은 것 한가지를 소개해 드리고 싶은데요..

> 내가 불편하다/있었으면 좋겠다.. 라고 생각하는 것이 있다면, 그 생각을 내가 세계최초로 한 사람은 아닐 것이며, 해결방법은 있을 것이다.

마지막에 소개해드린 스타일로 처리하는 방법 역시 있고, 대중적이지는 않지만 조금씩 사용되고 있는 추세인 것 같습니다.

다음 시간에는 async/await 라는 주제를 가지고 위 트위치 팔로우 예제를 조금 더 개선해 보겠습니다!
