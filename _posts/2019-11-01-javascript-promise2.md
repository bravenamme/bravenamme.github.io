---
layout: post
title:  "자바스크립트 콜백지옥 탈출기 - 2....."
date:   2019-11-01 10:00
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
    console.log('팔로우 성공')
  })
```
