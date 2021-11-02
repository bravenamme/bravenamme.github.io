---
layout: post
title: "브라우저에서 비동기요청을 수행할 때 도움이 될 수도 있는 글 (api wrapping)"
date: 2021-11-04 10:00
author: firepizza
tags: [javascript]
---

## API 서버와의 통신

웹 개발 환경이 여러모로 다양해지면서 REST API 서버를 별도로 두고 클라이언트(브라우저)에서 특정 endpoint를 호출해 CRUD를 수행하는 모델이 속속 등장하고 있습니다.

이러한 개발을 진행하다 보면 자연스레 소위 AJAX(Asynchronous JavaScript and XML) 라고 불리우는 기법을 클라이언트단에서 활용하게 됩니다.

AJAX 를 이용하기 위해서는 여러가지 방법이 있을 수 있는데, 먼저 js 네이티브단에서 지원하는 XMLHttpRequest (XHR이라고도 불리는)객체를 활용한 방법이 있겠습니다.

## XMLHttpRequest

```javascript
const xhr = new XMLHttpRequest();
xhr.onreadystatechange = () => {
    // 콜백은 간략화
    if (xhr.status === 200) {
        console.log('ok!');
    }
    console.log('callback');
};
xhr.open('GET', '/someapi/foo/bar', true);
xhr.setRequestHeader('Content-Type', 'application/json');
xhr.send(JSON.stringify({foo:'bar'}}));
```

위와 같이 직접 XMLHttpRequest 객체를 이용해 AJAX 기법을 이용해 볼 수 있습니다.

그러나 직접 객체를 사용해 핸들링하는 방식은 여러모로 번거롭기도 하고, 가독성 측면에서도 떨어기 때문에 (콜백 함수 자체가 갖는 특성) 아마 이 방법 보다는 대부분 ajax를 jQuery에서 지원하는 내장함수를 이용해 접해 보았을 것으로 생각 됩니다.

## $.ajax

```javascript
$.ajax({
  type: "GET",
  url: "/someapi/foo/bar",
  async: true,
  beforeSend: function (xhr) {
    xhr.setRequestHeader("Content-Type", "application/json");
  },
  success: function () {
    console.log("ok!");
  },
  complete: function () {
    console.log("callback");
  },
});
```

위 문단에서 XHR 객체를 직접 이용해 보냈던 요청과 동일한 요청을 jQuery 의 ajax함수를 이용해 보내 보았습니다.

이렇게 2개를 함께 보니 \$.ajax() 함수가 내부에서 어떤식으로 동작할지가 어느정도 머릿속에 그려지지 않나요?

> 사실 엄밀히 말해서 jQuery는 내부적으로 XHR객체가 아닌 jqXHR 이라는 자체 모델을 이요하고 있긴 하지만 그 구성은 동일하게 이해할 수 있습니다.

저는 처음 프로그래밍을 배울 때 \$.ajax()를 습관적으로 이용 했기에 ajax통신 이라고 부르는 것이 \$.ajax가 보통명사화 된 것으로 오해 했었는데요.

ajax라는 것은 앞서 이야기 했던 통신 기법을 의미하는 명칭이며
\$.ajax가 있고 나서 ajax 라는 기법이 있었던게 아니라

AJAX 기법을 이용하기 쉽게 jQuery에서 제공해주는 함수 이름이 기법과 동일한 \$.ajax() 였다는 것을 이제는 알게 되었습니다. (말이 좀 헷갈리긴 하네요)

## 그 외의 ajax 통신을 돕는 것들

정확히 언제 AJAX 라는 기법이 생겼는지에 대해서는 자세히 알기 어려우나,
현 시대에 와서는 너무나 당연한 기법이 되어버렸는데요

그에 발맞추어 AJAX를 돕는 라이브러리나 native API단에서의 지원도 늘어 왔습니다.

대표적으로 ECMA6와 함께 도입된 fetch API를 예로 들어볼 수 있겠네요.

```javascript
const promiseObject = fetch("/someapi/foo/bar");
```

위 코드에서 `promiseObject`에는 Promise 객체가 담겨 있을 것이고,

then 체이닝을 통해 차근차근 XHR을 resolve해나갈 수 있도록 돕습니다.

[Promise에 대한 지난 포스트](https://bravenamme.github.io/2019/11/06/javascript-promise2/)

또한 npm이 득세하면서 몇년 전부터 시장의 강자로 떠오르고 있는 [axios](https://github.com/axios/axios)를 예로 들어볼 수도 있습니다. (개인적으로 가장 선호하는 라이브러리 이기도 합니다.)

## wrapper가 있는게 좋을까?

이 외에도 여러 AJAX 라이브러리들이 생산 되고 있으며, 경우에 따라 유연하에 필요한 스펙에 맞춘 라이브러리를 도입하거나 fetch API와 같은 native api를 이용하는 경우도 있을 것입니다.

그러나 우리가 모두 개발을 해 보아서 아시다시피 항상 요구사항은 바뀌기 마련이고 개발자는 그에 대해 유연하게 대처할 수 있는 준비를 해 둘 필요가 있었던 경험이 있으실 텐데요.

같은 맥락으로 ajax 통신을 전담하는 라이브러리에 대한 교체가 필요한 시기가 도래했다고 할때, 소스코드 곳곳에서 \$.ajax() 를 이용해 직접 이용 중이었다면 모든 소스를 찾아가며 수정이 필요하게 될 텐데요..

이런 악몽같은 일을 미연에 방지하기 위해 제가 제안드리는 방법은 AJAX Request를 담당하는 추상화 계층을 하나 만들어 내부 구현체를 쉽게 변경할 수 있도록 하자는 것 입니다.

> 마치 Hibernate로 내부가 구현된 레이어를 이용하지만 우리는 JPA(인터페이스)를 이용해 개발했다고 이야기 하는 것과 같은 느낌으로요

좀더 쉽게 이야기 하면 ajax 통신을 전담하는 모듈을 한번 감싸는 warpper 를 만들어 두는 것이라고 할 수 있겠네요.

이 단순한 이야기를 하기 위해 서론이 정말정말 길었네요 ㅎ

## 간단한 래퍼 예제

다음으로는 제가 주로 이용하는 ajax 통신 wrapper 구조를 간단히 소개하고 마치고자 합니다.

위 까지는 방법론적인 내용이었고, 실제 어떤 식으로 구현할지에 대해서는 사람에 따라 천차만별이며 해당 어플리케이션의 요구사항에 따라 스펙이 들쑥날쑥 할 수 있기 때문에 단순 참고용 으로만 봐 주세요

```javascript
// index.js

import axios from "axios";

// headers나 기타 공통 설정등이 필요하다면 이곳에서 한번에 세팅

const onNotFound = () {
    // 404일때 뭔가
}

const onServerError = () {
    // 500일때 뭔가
}

const onCatchProcess = (err) => {
    const {status, data} = err.response;
    if (status === 404) onNotFound();
    else (status === 500) onServerError();

    throw err.response;
}

export default ({ method = "GET", url, data, headers }) => {
  return axios({
    method,
    url,
    data,
  })
    .then((res) => res.data)
    .catch((err) => onCatchProcess(err));
};
```

위와 같은 Promise를 리턴하는 래퍼로 한번 감싸서 ajax 라이브러리를 직접 사용하지 않고도 충분히 ajax콜이 가능하도록 이용한다면 에러핸들링이나 공통처리 등도 수월해지고, 관리포인트도 줄어드는 장점이 있다고 생각 합니다.
