---
layout: post
title:  "Typescript에 대하여"
date:   2019-09-25 10:00
author: maan
tags:	[typescript, script, 웹트랜드]
---

## 시작하며
JEST, TypeScript, GraphQL, React Hooks, Serverless를 2019년에 알아야한 최신 웹트렌드로 소개 하는 영상을 보다가
typescript에 대해 궁금해서 찾아 보았습니다.
해당 영상은 Typescript가 뜨고 있는 이유를 다음과 같이 설명합니다.

첫번째는 마소에서 만들었고, 두번째는 마소에서 만들었기 때문에 요새 많이들 쓰고 있는 vscode랑 호환이 잘된다고 합니다.

그리고 당연히 마소에서 만들었으니... 마소에서 사용할 것이고, 페이팔에서도 사용 중이라고 합니다.
<br><br>
## Typescript 란?
Typescript는 대규모 프로젝트에 javascript가 적합하지 않다는 문제점을 보완하기 위해 만들어진 언어로 자바스크립트(ES5)의 Superset(상위확장)입니다.

Microsoft에서 2012년 발표한 오픈소스이고 정적 타이핑을 지원하며 ES6(ECMAScript 2015)의 클래스, 모듈 등과 ES7의 Decorator 등을 지원한다고 합니다.

![출처: https://poiemaweb.com/typescript-introduction](/files/posts/typescript-superset.png)

> Typescript의 목표는 "클래스, 모듈, 정적 타입과 같은 것들을 통해 JavaScript를 강화하는 것" 이며, open-standard 나 크로스 플랫폼에 대한 이점을 포기하지 않는 것이고, 그 결과로 "JavaScript 개발 범위의 어플리케이션을 위한 언어"가 되었고, JavaScript 의 상위언어집합(superset)이 되었습니다.
 
 <br><br>
## Typescript 특징
### 컴파일 언어
javascript는 인터프리터 언어입니다. 코드를 생성 하고 실행이 되어야지만 오류가 있는지 확인 할 수 있지요. 

하지만 Typescript의 경우 코드를 컴파일할 때 구문 오류를 발견하면 컴파일 오류를 발생시킵니다.

컴파일 후 동일한 이름의 스크립트 파일이 생성 되는데, 오류가 발생해도 js 파일을 생성은 해 줍니다. 

이 파일은 문제가 있는 파일이다 라고 경고 해주는 역할을 해 주는 것이지요!


### type지정 언어
Typescript를 쉽게 설명하자면 javascript에 type을 지정해서 사용 한다는 것입니다.

javascript의 var, let, const 대신 string이나 number 같은 자료형을 지정해 줌으로써 컴파일시 안정성을 확보해 줍니다.

```javascript
function sum(a, b){
    return a+b;
}
```

위와 같은 코드가 있을때 javascript의 경우 형이 지정되어 있지 않기 때문에 a, b에 어떤 변수를 넣어도 연산이 됩니다.

```javascript
sum(1,2); //연산 결과 3
sum('a','b') //연산 결과 ab
```

위 코드를 타입 스크립트로 변경 하면 아래와 같습니다.

```typescript
function sum(a: number, b: number) {
  return a + b;
}

sum('x', 'y');
// error TS2345: Argument of type '"x"' is not assignable to parameter of type 'number'.
```

타입스크립트의 경우 처음부터 자료형을 설정해 주기 때문에 js의 경우 처럼 문자열로 변수를 지정하면 컴파일시 오류 발생 시킵니다.

이렇게 타입을 명시해 줌으로써 가독성을 높혀 주고 프로그램의 의도를 확실히 해 줄 수 있는 장점이 있습니다. 예외 처리에 대한 설정 또한 불필요게 되지요!!<br>

간혹 실무에서 json 값으로 받아 오는 숫자형 데이터 연산시 parseInt로 형변환 해주지 않으면 1+2 연산 값이 12가 되어 버리는 현상이 나타 나는데 
type을 지정해서 사용해 주면 이런 오류는 좀 줄어 들지 않을까 생각 됩니다.

#### Typescript 기본 타입
Typescript는 부울 (Boolean), 숫자형 (Number), 문자열 (String), 배열 (Array), 튜플 (Tuple), 열거 (Enum), Any, Void, Never 등의 타입을 지정할 수 있습니다.

1. 부울 (Boolean), 숫자형 (Number), 문자열 (String)<br>
TypeScript는 10진수 및 16진수와 함께 ECMAScript2015에 도입된 2진수 및 8진수 문자를 지원합니다.
문자열의 경우 템플릿 문자열을 사용 할 수도 있는데 이 문자열은 백틱 / 백 쿼트 (` ) 문자로 감싸져 있으며 포함된 표현식은 ${ 표현식 } 형식입니다. 
```typescript
let isDone: boolean = false;
let decimal: number = 6;
let hex: number = 0xf00d;
let binary: number = 0b1010;
let octal: number = 0o744;
let color: string = "blue";
let fullName: string = `Bob Bobbington`;
let sentence: string = `Hello, my name is ${ fullName }.`
```

2. 배열 (Array)<br>
배열은 제네릭 형태로도 선언 가능합니다.
```typescript
let list: number[] = [1, 2, 3];
let list: Array<number> = [1, 2, 3];
```

3. 튜플 (Tuple)<br>
서로 다른 타입의 배열을 만들때 사용 하고, 지정한 자리에는 꼭 그 타입이 선언되어야 합니다.
```typescript
// 튜플 타입 선언
let x: [string, number];
// 초기화
x = ["hello", 10]; // 좋아요
// 부정확한 초기화
x = [10, "hello"]; // 오류 - 지정한 타입과 다른 값이 들어 감.
```

4. 열거 (Enum)<br>
Enum 타입이 사용 가능 한데 자바에서 사용 하듯이 동일하게 사용됩니다.<br>
```typescript
enum Color {Red, Green, Blue}
let c: Color = Color.Green;
//보통 0부터 시작 하지만 1부터 시작하도록 설정
enum Color {Red=1, Green, Blue}
let c: Color = Color.Green;
//숫자값 지정도 가능
enum Color {Red=2, Green=3, Blue=6}
let c: Color = Color.Green;
```

5. Any<br>
어플리케이션을 작성할 때 알지 못하는 변수의 타입을 설명해야 할 수도 있습니다.
모든 타입을 받을 수 있도록 선언하는 타입입니다.
```typescript
let notSure: any = 4;
notSure = "문자열일수도 있다";
notSure = false; // 좋아요, 확실한 boolean
let list: any[] = [1, true, "free"];
list[1] = 100;
```

### 객체지향프로그래밍 지원
클래스, 인터페이스, 상속등을 지원합니다.
<br><br>


## 간단한 설치 방법
npm을 위해 노드를 설치 해야 하지만 전 npm이 설치 되어 있으니 생략합니다..

> npm install -g typescript

전 그냥 위와 같이 설정 했는데요 -D면 devDependency로 설치 되고 -g는 global로 설치가 되는 옵션이니 참고 바랍니다.

typescript 설치후 아래와 같은 예제 파일을 만들어 봤습니다.
#### Typescript 파일 작성
```typescript
class Coffes {
    name : string;
    price : number;

    constructor(name, price) {
        this.name = name;
        this.price = price;
    }

}

function coffeeInfo(name : string, price : number){
    return "name : " + name + ", price : " + price;
}
```
간단하게 클래스와 생성자 , 메소드가 있는 예제를 만들었습니다.

#### 컴파일
> tsc typescriptTest.ts

파일은 .ts 로 만들었지만 내용은 javascript로 이루어져 있습니다.

컴파일이 완료 되면 같은 이름의 js 파일이 생성 되는 걸 볼 수 있습니다.
###### 생성된 화면

![출처: 내가 한 캡처](/files/posts/190926_typescript2.jpg)

``` javascript
//컴파일 된 내용
var Coffes = /** @class */ (function () {
    function Coffes(name, price) {
        this.name = name;
        this.price = price;
    }
    return Coffes;
}());
function coffeeInfo(name, price) {
    return "name : " + name + ", price : " + price;
}

```

# 정리
지금까지 typescript에 대해 수박겉핥기로 알아 봤는데요~

정리 해보면 typescript는 javascript와 비슷하여 접근성이 좋고, 타입을 지정하여 좀 더 가독성 있고 사용하는 변수의 의도를 명확 하게 할 수 있다는 점이 장점으로 꼽힙니다. 

꼭 이걸 사용 해서 뭔가를 하겠다는 생각이 아직 들지는 않지만... 혹시 나중에 써 볼 수도 있지 않을까요? 



## 참고 사이트
- https://www.youtube.com/watch?v=QyxES-SUq_E
- https://poiemaweb.com/typescript-introduction 
- https://han41858.tistory.com/14
- https://velog.io/@dongwon2/TypeScript%EB%A5%BC-%EC%8B%9C%EC%9E%91%ED%95%98%EA%B8%B0-%EC%A0%84%EC%97%90-%EC%9D%B4%EC%A0%95%EB%8F%84%EB%8A%94-%ED%95%B4%EC%A4%98%EC%95%BC%EC%A7%80
- https://proimaginer.tistory.com/17
- https://typescript-kr.github.io/pages/tutorials/TypeScript%20in%205%20minutes.html