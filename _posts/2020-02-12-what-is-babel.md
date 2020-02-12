---
layout: post
title:  "babel 이란 무엇인가?"
date:   2020-02-12 21:56
author: monoless
tags:	[javascript, webpack, babel, polyfill, esnext, typescript]
---

## 들어가며

 최근에 react 프로젝트와 typescript 프로젝트를 거치면서 webpack 을 자주 써보고 세팅해보게 되었습니다.  
처음에는 동작의 원리보다 `요즘 잘나가는 프론트엔드 개발 환경 만들기`라는 목표로 세팅 하였으나  
점점 처음부터 차근 차근 만지면서, 내가 이걸 몰랐구나 이게 이런 뜻이었구나 새삼 느끼게 되었습니다.  

![](https://thumbs.gfycat.com/DarkInsignificantBumblebee-size_restricted.gif)

webpack 에 대한 설명은 이미 많이 있으니 이번에는 babel 에 대해서 글을 적어보도록 하겠습니다.

## babel 이란?

 먼저 공식 사이트 소개를 가져오자면 아래와 같습니다.
> Babel is a JavaScript compiler.

그렇습니다. 바로 _`자바스크립트 컴파일러`_ 입니다. 왜 인터프리터 언어에 컴파일러가 필요하지? 라고 생각하시는 분들도 있으실 껍니다.  

정확히는 babel 은 javascript 로 결과물을 만들어주는 컴파일러입니다. _`소스 대 소스 컴파일러 (transpiler)`_  
라고 불립니다.

## 그렇다면 왜 javascript 로 변환하는 과정이 필요할까요?

### ESNext and Legacy... Legacy...
지금도 프론트엔드는 너무나도 빠르게 발전되고 있습니다.  
심지어 최신 브라우져 조차도 지원하지 못하는 문법과 여러 기술들이 출연하고 있습니다.  
> 관련되어서 아래 문서도 읽어보세요! [ESnext, 내일의 자바스크립트](/2019/05/30/javascript-pick-up/)
새로운 ESNext 문법을 기존의 브라우져에 사용하기 위해서도 babel 은 필수적입니다.

 슬프게도 아직도 많은 사람들이 예전 브라우져 예전 OS 를 사용하고 있습니다.  

![2019년 브라우져 점유율](http://cdn.bizwatch.co.kr/news/photo/2019/07/15/c46915b4abe464c554a4b0fb3ef115cd.jpg) [출처] [네이버 브라우저 '웨일' 점유율 4% 넘어섰다](http://news.bizwatch.co.kr/article/mobile/2019/07/15/0030)

 안드로이드는 더욱 더 충격적인데 아직 제 주변에서도 갤럭스 노트 2를 사용하시는 분들이 꽤 있더군요...

 ![2019년 안드로이드 버전별 점유율](/files/posts/202002/20200212_android_market_share.png) [출처] [Android 배포 대시보드](https://developer.android.com/about/dashboards)

이런 하위 호환성은 외면하기에는 쉽지 않습니다. (이 부분은 babel-polyfill 이라는 내용으로 추가로 다루어 보겠습니다.)

 
### New Language
현재 typescript 열풍이 거셉니다. [typescript](https://www.typescriptlang.org/) 는 Microsoft 에서 만든 언어로써  
강력한 정적 typing 과 es2015 기반으로하는 객체지향적 문법입니다.
> 관련되어서 아래 문서도 읽어보세요! [Typescript에 대하여](/2019/09/25/typescript/)

이미 npm 다운로드 트랜드 사이트에서는 폭발적인 인기를 볼 수 있습니다.

![최근 2년간 npm 다운로드 횟수](/files/posts/202002/20200212_npmtrends.png) [출처] [npmtrends.com](https://www.npmtrends.com/typescript-vs-coffee-script-vs-purescript-vs-flow-bin-vs-elm)

또한 기존 job recruit 사이트에서도 연봉이 많이 오른 것을 확인할 수 있습니다.
![typescript average salary](/files/posts/202002/20200212_typescript_salaries.png) [출처] [ziprecruiter.com](https://www.ziprecruiter.com/n/Typescript-Jobs-Near-Me?near_me_location=San-Francisco,CA)
![javascript average salary](/files/posts/202002/20200212_javascript_salaries.png) [출처] [ziprecruiter.com](https://www.ziprecruiter.com/n/Typescript-Jobs-Near-Me?near_me_location=San-Francisco,CA)

심지어 java salary 보다 ㅎㄷㄷ
![java average salary](/files/posts/202002/20200212_java_salaries.png) [출처] [ziprecruiter.com](https://www.ziprecruiter.com/n/Java-Developer-Jobs-Near-Me?near_me_location=San-Francisco,CA)

여튼 typescript 든 coffeescript 든 javascript 로의 compile 이 필수가 되어야 하며, 이를 담당하는게 babel 입니다.

## polyfill??

폴리필(polyfill) 은 개발자가 특정 기능이 지원되지 않는 브라우저를 위해 사용할 수 있는 코드 조각이나 플러그인을 말합니다.  
기술은 빨리 발전하지만, 오래된 브라우저나 아직 최신 브라우져에서 지원하지 않는 기능들을 하위 호환성위해서 작성하는 프로그램들을  
polyfill 이라고 칭합니다.

### babel-polyfill

babel 을 사용한다고 최신 함수를 사용할 수 있는 건 아닙니다.  
babel 은 문법을 변환하여 javascript 로 변환하는 transpiler 역할만 할 뿐입니다. 

앞에서 설명한대로 polyfill 은 프로그램이 처음에 시작될 때 지원하지 않는 기능들을 추가하는 것입니다.  
즉, babel 은 컴파일시에 실행되고 babel-polyfill 은 런타임에 실행되는 것입니다.

아래는 호환성을 지키기 위해 최근 브라우져 2가지의 버전만 지원하면서 IE의 경우 10버전 이하는 제외하는 설정을 한 것 입니다!

```json 
    {
      "presets": [
        ["env", {
          "targets": {
            "browsers": ["last 2 versions", "not ie <= 10"]
          }
        }]
      ]
    }
```

물론 실제 리얼월드에서는 이것 만으로 불충분하지만. 이게 어디입니까!

**Reference**
 - [네이버 브라우저 '웨일' 점유율 4% 넘어섰다](http://news.bizwatch.co.kr/article/mobile/2019/07/15/0030)
 - [Android 배포 대시보드](https://developer.android.com/about/dashboards)
 - [npmtrends.com](https://www.npmtrends.com/typescript-vs-coffee-script-vs-purescript-vs-flow-bin-vs-elm)
 - [typescript](https://www.typescriptlang.org/)
 - [ziprecruiter.com](https://www.ziprecruiter.com)