---
layout: post
title: "[JS] 행복한 신기능"
date: 2021-01-14 15:00
author: firepizza
tags: [javascript, ECMAscript]
---

### ES2020, ES2021

- 모던 스크립트 기법을 나누는 분기점이 되었던 ES6(ES2015)가 발표된지 어느덧 6년째가 되었습니다.

- 작년말에는 ES12(ES2021)가 발표되었고, 그간 업데이트 된 추가 기능 중 흥미로운 기능들이 있어 몇가지 소개하고자 합니다.

## Optional Chaining (?.)

- 예전에 웹뷰 기반의 앱 서비스를 하던 때 처음으로 iOS 개발자분과 협업을 할 수 있는 기회가 있었고, bridge통신이 중요했기에 서로 코드를 공유하며 일했던 적이 있었습니다.
- 그때 처음으로 swift 라는 언어를 접할 수 있었고, 당시 자바개발에만 푹 빠져있던 저에게 충격적으로 다가왔던 문법이 있었는데요, 그건 바로

> nil

이라는 용법이었습니다.

물론 자바에서도 Optional을 사용하거나 자바스크립트에서도 연산자를 사용하는 등

```javascript
const book = { author: { first: "fire", last: "pizza" } };
book.author && book.author.first; // 'fire'
book.publisher && book.publisher.address; // undefined
```

해결할 수 있는 방법은 있었지만, 구문뒤에 물음표 하나만을 붙여 null체크를 진행하던 방식이 너무 부러웠던 기억이 있습니다.

그리고 어느덧 javascript에서도 해당 기능이 추가되었는데요, 바로 Optional Chaining [?.] 이라고 불리는 구문이며 다음과 같이 사용할 수 있습니다.

```javascript
const book = { author: { first: "fire", last: "pizza" } };
book?.author?.first; // 'fire'
book?.publisher?.address; // undefined
```

더 놀라운 점은 Optional Chaining을 객체 뿐만 아니라 배열이나 함수에도 사용할 수 있다는 것인데요.

- 어찌보면 함수도 1급객체로 취급하는 자바스크립트에선 당연한 얘기일 수도 있지만요

```javascript
// 배열에서의 사용
const abcArr = ["a", "b", "c"];
abcArr?.[0]; // 'a'
defArr?.[0];

// 함수에서의 사용
const sayHello = () => consoloe("hello");
sayHello?.(); // 'hello'
sayHi?.(); // undefined
```

주의할 점은 이 용법은 ? 만이 아니라 ?. 이라는 것이고<br/>
배열과 함수를 참조/실행하기 전 단계에서 이 구문으로 검증해 사용해야 한다는 점 입니다!

## Nullish coalescing operator (??)

- 영어는 너무 길고 이걸 뭐라고 부르면 좋을런지... 쌍물음표?
- 값이 null이나 undefined 인 경우에만 falsy value로 인식하는 연산자인데요, 아래 예제를 보시면 바로 이해가 되실거에요.

```javascript
const nullVal = null;
const undefinedVal = undefined;
const falseVal = false;
const zeroVal = 0;

// before
nullVal || "okay"; // okay
undefinedVal || "okay"; // okay
falseVal || "okay"; // okay
zeroVal || "okay"; // okay
/* 이 경우에 0이나 false는 정상값으로 통과시키려고 하면 조금 화가 나려고 함*/

// after
nullVal ?? "okay"; // okay
undefinedVal ?? "okay"; // okay
falseVal ?? "okay"; // false
zeroVal ?? "okay"; // 0
/* 행복하다 */
```

- 0이나 false 값을 정상값으로 인식하기 위해 더욱 간단한 코딩이 가능해졌습니다.
- 쌍물음표를 활용해 행복한 코딩 하세요

## replaceAll()

- 드디어 나왔습니다.
- 습관적으로 손이 움직여 아래처럼 작업한 후에

```javascript
const str = "fire is pizza";
str.replaceAll("z", "t");
// Uncaught TypeError: str.replaceAll is not a function
```

- 아차 하고 아래와 같이 처리해본 적이 많으실 텐데요.

```javascript
const str = "fire is pizza";
str.replace(/z/gi, "t"); // fire is pitta
```

- String.prototype.replaceAll 함수의 추가로 인해 이제 가능해 졌습니다 !!!
- 많이 활용하셔서 행복한 코딩 하세요

출처:
[MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference)
