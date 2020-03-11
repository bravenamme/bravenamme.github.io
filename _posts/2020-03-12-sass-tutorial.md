---
layout: post
title:  Sass (Syntactically Awesome Style Sheets)
date:   2020-03-12 08:00
author: chyusee
---

# Sass
## [Sass](https://sass-lang.com/) (Syntactically Awesome Style Sheets)

CSS를 작성하다보면 길어질수록 단순 반복되는 부분이 많아지고, 구조 또한 복잡해져서 관리하기가 힘들어 집니다.

그런 CSS를 편리하기 사용하기 위해, 조건문, 반복문, 연산, 상속 등을 지원하는  CSS 전처리기가 등장하게 됩니다.

대표적으로 [Sass](https://sass-lang.com/), [LESS](http://lesscss.org/), [Stylus](https://stylus-lang.com/) 등이 있습니다.

그 중에서 Sass(SCSS) 사용법에 대해서 알아보자고 합니다.
<br>
<br>

## Sass와 SCSS 차이점
**Sass**
```sass
.wrap 
    width: 100px
    height: 200px
    .container
        position: relative
        margin: 0 auto
```
**SCSS**
```scss
.wrap {
    width: 100px;
    height: 200px;
    .container {
        position: relative;
        margin: 0 auto;
    }    
}
```
**CSS**
```css
.wrap {
    width: 100px;
    height: 200px;
}

.wrap .container {
    position: relative;
    margin: 0 auto;
}
```

위의 예제를 컴파일 했을때 동일한 CSS로 변환됩니다.<br>
문법적으로 SCSS쪽이 좀 더 익숙한 느낌입니다. 

<table style="width:100%;border:1px solid #000;text-align:center;">
    <tr>
        <th></th>
        <th>Sass</th>
        <th>Scss</th>
    </tr>
    <tr>
        <td>선언</td>
        <td>들여쓰기</td>
        <td>증괄호</td>
    </tr>
    <tr>
        <td>속성</td>
        <td>줄바꿈</td>
        <td>세미콜론</td>
    </tr>
</table>

## 기본문법
SCSS 기준으로 설명합니다.

### 1. 변수
변수로 값을 저장하여 재사용 할 수 있습니다.
**SCSS**
```scss
$font-stack:    Helvetica, sans-serif;
$primary-color: #333;

body {
    font: 100% $font-stack;
    color: $primary-color;
}
```
**CSS**
```css
body {
    font: 100% Helvetica, sans-serif;
    color: #333;
}
```
<br>

### 2. 중첩
계층 구조를 식별하기 편하게 작성 할 수 있습니다.
**SCSS**
```scss
nav {
    ul {
        margin: 0;
        padding: 0;
        list-style: none;
    }

  li { display: inline-block; }

  a {
        display: block;
        padding: 6px 12px;
        text-decoration: none;
    }
}
```
**CSS**
```css
nav ul {
    margin: 0;
    padding: 0;
    list-style: none;
}

nav li {
    display: inline-block;
}

nav a {
    display: block;
    padding: 6px 12px;
    text-decoration: none;
}
```
<br>

### 3. 모듈
CSS를 여러 개의 파일로 분할해서 사용 할 수 있습니다.
(1.23 버전 부터 지원)

**SCSS**
```scss
/* _base.scss */
$font-stack:    Helvetica, sans-serif;
$primary-color: #333;

body {
  font: 100% $font-stack;
  color: $primary-color;
}

/* styles.scss */
@use 'base';

.inverse {
  background-color: base.$primary-color;
  color: white;
}
```
**CSS**
```css
body {
  font: 100% Helvetica, sans-serif;
  color: #333;
}

.inverse {
  background-color: #333;
  color: white;
}
```
<br>

### 4. Mixin
Mixin을 사용하여 매크로처럼 값을 전달 받아 재사용할 수 있습니다.
**SCSS**
```scss
@mixin transform($property) {
    -webkit-transform: $property;
    -ms-transform: $property;
    transform: $property;
}
.box { @include transform(rotate(30deg)); }
```
**CSS**
```css
.box {
    -webkit-transform: rotate(30deg);
    -ms-transform: rotate(30deg);
    transform: rotate(30deg);
}
```

### 5. 상속
미리 선언된 CSS 속성을 상속받아서 사용할 수 있습니다.

**SCSS**
```scss
%message-shared {
    border: 1px solid #ccc;
    padding: 10px;
    color: #333;
}

%equal-heights {
    display: flex;
    flex-wrap: wrap;
}

.message {
    @extend %message-shared;
}

.success {
    @extend %message-shared;
    border-color: green;
}

.error {
    @extend %message-shared;
    border-color: red;
}

.warning {
    @extend %message-shared;
    border-color: yellow;
}
```
**CSS**
```css
.message, .success, .error, .warning {
    border: 1px solid #ccc;
    padding: 10px;
    color: #333;
}

.success {
    border-color: green;
}

.error {
    border-color: red;
}

.warning {
    border-color: yellow;
}
```
<br>

### 6. 연산자
CSS에서 수학 연산자를 이용 할 수 있게 해줍니다.

**SCSS**
```scss
.container {
    width: 100%;
}

article[role="main"] {
    float: left;
    width: 600px / 960px * 100%;
}

aside[role="complementary"] {
    float: right;
    width: 300px / 960px * 100%;
}
```
**CSS**
```css
.container {
    width: 100%;
}

article[role="main"] {
    float: left;
    width: 62.5%;
}

aside[role="complementary"] {
    float: right;
    width: 31.25%;
}
```
<br>

### 7. 조건문
@if, @else, @elseif 를 이용하여 조건문을 사용할 수 있습니다.

**SCSS**
```scss
@mixin triangle($size, $color, $direction) {
    height: 0;
    width: 0;

    border-color: transparent;
    border-style: solid;
    border-width: $size / 2;

     @if $direction == up {
        border-bottom-color: $color;
    } @else if $direction == right {
        border-left-color: $color;
    } @else if $direction == down {
        border-top-color: $color;
    } @else if $direction == left {
        border-right-color: $color;
    } @else {
        @error "Unknown direction #{$direction}.";
    }
}

.next {
    @include triangle(5px, black, right);
}
```
**CSS**
```css
.next {
    height: 0;
    width: 0;
    border-color: transparent;
    border-style: solid;
    border-width: 2.5px;
    border-left-color: black;
}
```
<br>

### 8. 반복문
@each, @for, @while 등의 반복문을 지원합니다.
<br>
#### 8-1. @each

**SCSS**
```scss
$sizes: 40px, 50px, 80px;

@each $size in $sizes {
    .icon-#{$size} {
        font-size: $size;
        height: $size;
        width: $size;
    }
}
```
**CSS**
```css
.icon-40px {
    font-size: 40px;
    height: 40px;
    width: 40px;
}

.icon-50px {
    font-size: 50px;
    height: 50px;
    width: 50px;
}

.icon-80px {
    font-size: 80px;
    height: 80px;
    width: 80px;
}
```

**SCSS - 맵형태의 데이터 이용시**
```scss
$icons: ("cat": "🐱", "dog": "🐶", "cow": "🐮");

@each $name, $glyph in $icons {
    .icon-#{$name}:before {
        display: inline-block;
        font-family: "Icon Font";
        content: $glyph;
    }
}
```
**CSS**
```css
@charset "UTF-8";
.icon-eye:before {
    display: inline-block;
    font-family: "Icon Font";
    content: "🐱";
}

.icon-start:before {
    display: inline-block;
    font-family: "Icon Font";
    content: "🐶";
}

.icon-stop:before {
    display: inline-block;
    font-family: "Icon Font";
    content: "🐮";
}
```

#### 8-2. @for

**SCSS**
```scss
$base-color: #036;

@for $i from 1 through 3 {
        ul:nth-child(3n + #{$i}) {
        background-color: lighten($base-color, $i * 5%);
    }
}
```
**CSS**
```css
ul:nth-child(3n + 1) {
    background-color: #004080;
}

ul:nth-child(3n + 2) {
    background-color: #004d99;
}

ul:nth-child(3n + 3) {
    background-color: #0059b3;
}
```

#### 8-3. @while
**SCSS**
```scss
@function scale-below($value, $base, $ratio: 1.618) {
    @while $value > $base {
    $value: $value / $ratio;
    }
    @return $value;
}

$normal-font-size: 16px;
sup {
    font-size: scale-below(20px, 16px);
}
```
**CSS**
```css
sup {
    font-size: 12.36094px;
}
```
<br>

여기까지 SCSS의 문법에 대해서 알아보았습니다.

##### 참조
https://sass-lang.com/documentation <br>
https://heropy.blog/2018/01/31/sass <br>