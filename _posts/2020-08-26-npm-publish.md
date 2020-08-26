---
layout: post
title:  "NPM 번외편 - npm에 내 라이브러리 등록하기"
date:   2020-08-26 15:00
author: firepizza
tags:	[javascript, npm]
---

# 나만의 라이브러리를 만들어 보자.

오픈소스... 라이브러리... 말은 많이 들어 봤지만.. 그런건 정말 대단한 슈퍼개발자들이나 하는 거 아닌가?
라는 생각을 어렴풋이 가지고 있었던 것 같다. <br/>
(물론 지금도 어느 정도는 벽을 느끼고 있지만.)

그런데 아래와 같은 경험을 하며 그 진입장벽중 꽤나 큰 부분이 허물어졌던(?) 기억이 있어, 이 글을 읽은 분들이라면 한번쯤은 도전해 보셨으면 하는 마음에 주제를 선정해 보았다.

- 개발을 하다 보니 유틸리티성을 띄는 js모듈을 별도로 만들어 작업하는 경우가 있었다.
- 그런데 이런 유틸성 모듈들은 또 다른 프로젝트에서도 사용하고 싶은 경우가 생겼다.
  - 이걸 복붙해 가자니 왠지 조금 허접해보이는 느낌도 든다. (물론 관리측면에서 어려움은 말할것도 없고)
- 그렇다면 이 모듈 자체를 별도 프로젝트로 빼내서 의존성을 두어 관리 하면 어떨까?
  - 한발 더 나아가 내가 만든 이 모듈을 npm install 해서 모두가 같이 사용할 수 있을까?


이러한 생각에서 시작되어 만들었던 프로젝트가 이름만 거창한 js-util 이라는 모듈이다.

[요기를 클릭](https://www.npmjs.com/package/firepizza-js-util)<br/>
(Weekly Downloads가 꾸준한 걸 보니, 이전 회사에서 열심히 사용해 주고 계신 것 같다. 감사합니다.)

자, 그럼 본격적으로 어떻게 이런 단순하고 별볼일 없는 라이브러리가 npmjs에 올라갔고, 전세계 어디서든 누구나 <code>npm install firepizza-js-util</code> 이라고만 치면 설치할 수 있는지 지금부터 알아보겠다.


## 1. 먼저 프로젝트를 생성한다.

- 앞서 소개한 npm을 이용해 배포하고자 하는 모듈의 boilerplate를 생성한다.

![package.json 생성](/files/posts/20200826/init.png)

- 그다음 간단히 작동하는 함수를 포함하는 index.js 파일을 만들어 준다.

```javascript
// index.js

const sayHello = (name) => console.log(name)

module.exports = { sayHello }
```

- npmjs에 노출되는 정보등을 추가하기 위해 pacakge.json 파일을 조금 수정한다.

![package.json 수정](/files/posts/20200826/packagejson.png)

> - name
>   - 이 모듈의 이름. npm install [...] 으로 사용된다.
> - version
>   - 현재 모듈의 배포버전.
> - description
>   - 이 모듈에 대한 설명
> - main
>   - 이 모듈을 다운받아 사용할 때 <code>require()</code><code>import</code> 등으로 로드시 불러올 진입점 파일.
> - scripts
>   - 이 모듈에 대한 script 명령어 선언부
> - author
>   - 이 모듈의 저자
> - license
>   - 라이센스 종류
> - keywords
>   - 이 모듈에 달리는 키워드. (태그 라고 봐도 된다.) npmjs에서 검색 시 활용된다.

> 더 다양한 설정타입에 대해서는 아래 사이트를 참조하면 좋다.<br/>
> [npmjs 가이드](https://docs.npmjs.com/creating-a-package-json-file)


## 2. 작성한 프로젝트를 npmjs에 배포.

- 배포하는 방법은 정말 간단하다. 


- 먼저 npmjs 사이트에 회원가입을 진행한다.
- npm CLI를 활용해 몇가지 명령을 실행할 수 있는데, 상세한 내용은 아래 사이트를 참조하면 된다.
  - [npm CLI 가이드](https://docs.npmjs.com/cli-documentation/)
- 이 가이드에서는 아래 2개 명령어만 사용할 예정이다.
  - <code>npm login</code>
  - <code>npm publish</code>  
- 해당 모듈의 프로젝트 루트에서 <code>npm login</code>명령어를 실행한다.

![npm login](/files/posts/20200826/npmlogin.png)

- 위와 같이 로그인이 되었다면 <code>npm publish</code>를 실행한다.

![npm publish](/files/posts/20200826/npmpub.png)

- 배포가 완료되었다. 아래 페이지에서 확인해 보자.

[배포된 모듈](https://www.npmjs.com/package/firepizza-say-hello)


## 3. 모듈 사용하기

- 실제로 모듈을 사용해 보자.
- 테스트 프로젝트를 하나 작성한 뒤 해당 모듈을 설치한다. <code>npm install firepizza-say-hello</code>

![install](/files/posts/20200826/install.png)

- 여타 npm모듈들과 마찬가지로 node_modules 폴더에 설치된 모습을 확인할 수 있다.

![잘 설치된 모듈](/files/posts/20200826/tree.png)

- 자 그럼 위에서 작성한 모듈을 사용하는 스크립트를 하나 작성해 실행해 보도록 하자.

```javascript
console.log('init index.js')

const helloModule = require('firepizza-say-hello')
helloModule.sayHello('firepizza')
```

![정상적으로 사용할 수 있다.!](/files/posts/20200826/result.png)

- 위와 같은 형태로 npmjs에 배포된 나만의 라이브러리를 불러와 사용할 수 있다.

별 것 아닌 라이브리러도 npmjs에 배포해서 다운받아 사용하곤 했는데,<br/>
이러한 일련의 과정을 혼자이긴 하지만 직접 수행 해 보는 것으로 인해<br/>
오픈소스 생태계에 한발짝...까진 아니어도 그쪽 세상의 공기를 조금이라도 맛본 느낌이 드는 부분이 있었다.<br/>

이렇게 하나씩 하나씩 경험해 나가다 보면 언젠간 우리도 슈퍼개발자들 처럼 거대 오픈소스 세계에서 함께 활동할 수 있지 않을까?