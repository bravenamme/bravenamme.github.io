---
layout: post
title:  "Flutter 개발 환경 설정"
date:   2020-07-21 10:00
author: tbzmtb
---

## Flutter 란?


지난번에 Flutter 에 대한 간략한 소개를 보고 급 관심이 생겨서 이번에 개발환경 설정까지 진행해 보았다.
소개글을 보실분은 다음 링크로 ...
<https://bravenamme.github.io/2020/07/01/flutter/>

## Flutter 개발 환경 설정

운영체제 : macOS (64bit)

저장 공간 : 2.8GB (IDE나 개발도구 용량은 별도)

도구 : bash, curl git2.x. mkdir, rm, unzip, which(음?)

SDK를 다운로드 후 아래 명령어를 실행한다.

    cd ~/development
    unzim ~/Downloads/flutter_macos_1.17.5-stable.zip

path 설정
    export PATH="$PATH:'pwd'/flutter/bin"

바이너리 다운로드
    flutter precache

Doctor 실행
    flutter doctor

이 명령은 환경을 체크하고 보고서를 터미널에 보여준다. 
(이로 인해 모듈간의 의존성에 대한 부분을 개발자가 일일히 기억해야할 필요가 없다)



### Xcode 설정

    sudo xcode-select --switch /Applications/Xcode.app/Contents/Developer
    sudo xcodebuild -runFistLaunch

시뮬레이터 실행

    open -a Simulator

Flutter 앱 실행

    flutter create my_app
    cd my_app
    flutter run

CocoaPods 설치
    sudo gem install cocoapods
    pod setup

### 에디터 설정
공식문서에는 안드로이드 스튜디오나, 인텔리j, 비주얼 스튜디오 코드를 추천하고있다.
개인 취향에 맞는 에디터를 사용하면 될 것 같다.

### 외부 패키지 이용하기
안드로이드 스튜디오를 이용중이라면 편집기 화면에서 pubspec.yaml 수정한다.
그리고 packages get 를 클릭하면 끝이다.

### 빌드
리액트 네이티브를 다룰때만 하더라도 안드로이드 스튜디오에서는 아이폰 단말기에 직접 빌드가 안되었는데 .. 플루터는 가능하다.
단말기를 연결하면 아래와 같이 프로비져닝을 안드로이드 스튜디오에서 체크한다.
![](/files/posts/20200728/1.png)


안드로이드 스튜디오에서 아이폰 단말기를 선택한 모습니다. 
여기에서 바로 안드로이드 단말기, 아이폰 단말기, 에뮬레이터 등 자유롭게 빌드가 가능하니 너무나도 좋다!.
![](/files/posts/20200728/2.png)

또한 기본적으로 각 컴포넌트는 머터리얼 UI를 제공한다.
과거에 IOS에서는 머터리얼 컴포넌트를 하나 하나 직접 임포트 해서 직접 구현해야 햇지만 Flutter 를 사용한다면 
그럴 필요가 없다.

![](/files/posts/20200728/3.png)

### 결론
하이브리드를 개인적으로 지향하지는 않지만 플루터로 한번쯤을 앱을 제작해보고 싶다는 생각이 든다.
과연 네이티브보다 얼마나 효율적일지 단점은 무엇인지.. 다음시간에 포스팅 하도록 해보겠다.


