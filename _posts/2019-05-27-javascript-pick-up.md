---
layout: post
title:  "ESnext, 내일의 자바스크립트"
date:   2019-05-30 10:00
author: jizero
tags:	[javascript,'ECMAscript','esnext']
---
<style>
#post-content p {margin: 10px 0 0 0;}
</style>

## 시작하며
이 이야기는 "ECMAscript 무엇이며, 표준은 어떻게 만들어지는가?" 라는 궁금증으로 부터 시작한다.
우선 위키에 정의된 ECMAscript란 이렇다.
>ECMA스크립트(ECMAScript, 또는 ES[1])는 Ecma 인터내셔널의 ECMA-262 기술 규격에 정의된 표준화된 스크립트 프로그래밍 언어이다. 자바스크립트를 표준화하기 위해 만들어졌고 지금도 자바스크립트가 제일 잘 알려져 있지만, 액션스크립트와 J스크립트 등 다른 구현체도 포함하고 있다.
Ecma 인터내셔널의 여러 기술 위원회(Technial Committee, 이하 TC) 중 TC39 라는 위원회가 이 명세를 관리한다.

JavaScript 표준을 제정하는 기술 그룹인 TC39
현재는 9th Edition - ECMAScript 2018 까지 나왔으며, 아직 출시되지 않은 ECMAScript의  준비 버전들 <code class="highlighter-rouge"> ESNext (또는 ES.Next) </code> 이라 부른다.
오늘은 정식 release를 기다리는 ESNext에 대해 살펴 볼 것이다.


## TC39 Process
ESNext 들은 TC39 프로세스라 불리는 과정을 통해 제정되는데,
진행 단계에 따라 허수아비 단계와 성숙한 1~4단계로 나눠진다. 

<img src="/files/posts/201905/pickme.png" style="max-width:100%;">


Stage 0: Strawman. 말 그대로 뼈대만 있는 상태다. 0단계 프로포절 제안에는 아무런 제약이 없다.

Stage 1: Proposal. 어느 정도 형태를 갖춘 상태다. 챔피언이라 불리는 프로포절 담당 TC39 위원회의 일원을 구해야 한다. 
이 외에도 제안의 실사용 예와 중요한 내부 알고리즘 등이 요구된다.

Stage 2: Draft. 조금 부족하지만, 형태를 거의 다 갖춘 상태다. 기초적인 프로포절에 대한 명세서가 요구된다. 

Stage 3: Candidate. 형태를 전부 갖춘 상태다. 완벽한 프로포절에 대한 명세서와 리뷰어, 명세서 작성자의 승인을 받아야 한다. 

Stage4: Finished. 다음 해 표준에 도입되기로 확정된 상태다. 


(출처 : http://it.chosun.com/site/data/html_dir/2018/07/25/2018072501553.html)

아래의 단계를들 밟아가며, 올라가는데 stage-4는 나름 데뷔조(?)인 것이다.  별다른 이변이 없는 이상 다가오는 새 표준에 포함되어 발표된다.
stage-3은 데뷔직전 후보들인 것인데, stage-2, stage-1의 것들은 한 두 단계를 열심히 올라가야 데뷔조에 들어갈 수 있다.
자세한 과정은 [공식문서](https://tc39.github.io/process-document/) 에서 확인 할 수 있다. 



## ESnext 살펴보기 

2015년을 기점으로 매년 6월 새로운 ECMAScript 표준이 발표되는데, 당해 3월 전까지 4단계를 달성하고 3월 회의에서 최종 승인된 제안들이 새 표준에 포함된다.

[http://kangax.github.io/compat-table/esnext/](http://kangax.github.io/compat-table/esnext/) 사이트에서
ESnext을 확인해 볼 수 있다.

<img src="/files/posts/201905/33.JPG" style="max-width:100%;">



### stage중에 아주아주 주관적인  개인픽 을 선정해보았다. (2019.05 기준)

- static class fields 

static 클래스의 정적 메서드  키워드는를 지원하고있지만, 이제 그냥 프로퍼티 앞에서 사용이 가능해 질것으로 보인다.
(new를 붙여 인스턴스화하지 않아도 접근 가능.)

```javascript
class CustomDate {
  // ...
  static epoch = new CustomDate(0);
}
```
 [Stage 3 static class fields 자세히 확인하기](https://github.com/tc39/proposal-static-class-features/)


- throw expressions

throw expressions 문법을 추가하자는 것인데, 기존 exception 에러 형식을 따로 만들지 않아도 된다.
사용하기 한결 쉬워지는 제안이다.

```javascript
//Parameter initializers
function save(filename = throw new TypeError("Argument required")) {
}

//Conditional expressions
function getEncoder(encoding) {
  const encoder = encoding === "utf8" ? new UTF8Encoder() 
                : encoding === "utf16le" ? new UTF16Encoder(false) 
                : encoding === "utf16be" ? new UTF16Encoder(true) 
                : throw new Error("Unsupported encoding");
}

// logical
const id = value || throw new Error();
// nullish coalesce
const id = value ?? throw new Error();

```
[Stage 2 throw expressions 자세히 확인하기](https://github.com/tc39/proposal-throw-expressions)


- Optional Chaining for JavaScript

객체의 프로퍼티를 체크할때 한번 쯤은 <code>TypeError: Cannot set property 'name' of undefined </code> 오류를 본적이 있을 것이다. 이런 타입을 체크할때
"?." 을 사용하여 프로퍼티가 존재하는지 유무를 체크할 수 있다. 만약 체크하는게 함수 타입일 경우, null/undefined가 아니라면 함수를 실행시킨다.


```javascript
if (myForm.checkValidity?.() === false) { // skip the test in older web browsers
    // form validation fails
    return;
}
```
[Stage 1 Optional Chaining 자세히 확인하기](https://github.com/tc39/proposal-optional-chaining)



## 마치며
자바스크립트가 처음 접했을때만 하더라도 웹브라우저에서 보조역할을 하는 스크립트언어에 불과했지만 시간이 지나고 발전함에 따라 그 컨텐츠의 거의 모든 부분에 상호작용을 하며 큰 영향력을 차지하고 있다.  매년 6월 릴리스되는 명세와 함께 JavaScript는 계속해서 발전하고  영향력을 키워나갈 것으로 보인다. 앞으로도 ECMAscript 의 가는길을 함께 응원 할 것이다.


* ESnext는 출시될 다음 버전을 통틀어 부르기도 한다.
* 최종 결정권은 ECMA의 TC39라는 위원회에 있지만, 새로운 안은 누구나 만들어서 TC39에 제안할 수 있다. 
* 위에 ESnext 개인픽들은 현재 대부분이 브라우저에서 지원하지 않는다. npm @babel/preset-stage-(버전) 설치하여 미리 사용해 볼 수 있다.
* 곧 ES2019가 나올 예정이며 주기적으로 릴리즈 됨에 따라 버전의 의미보다는 <span style="color:red">지원여부</span> 의 영향이 커질 것 이다.


## 참고 사이트
[https://benmccormick.org/2015/09/14/es5-es6-es2016-es-next-whats-going-on-with-javascript-versioning](https://benmccormick.org/2015/09/14/es5-es6-es2016-es-next-whats-going-on-with-javascript-versioning) 
[https://www.freecodecamp.org/news/es5-to-esnext-heres-every-feature-added-to-javascript-since-2015-d0c255e13c6e/](https://www.freecodecamp.org/news/es5-to-esnext-heres-every-feature-added-to-javascript-since-2015-d0c255e13c6e/)<br/>
[http://it.chosun.com/site/data/html_dir/2018/07/25/2018072501553.html/](http://it.chosun.com/site/data/html_dir/2018/07/25/2018072501553.html)<br/>
[https://github.com/tc39/proposals](https://github.com/tc39/proposals.)