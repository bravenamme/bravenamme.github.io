---
layout: post
title:  "회원가입/로그인/보안 그리고 AWS cognito"
date:   2021-03-09 10:00
author: twosunny
tags:	[aws,awscognito,oauth,openconnectid]
---

## 들어가며
우리가 회원 인증 시스템을 처음부터 만든다고 가정해 보자.  
러프하게 기능을 나열해 봐도, 회원가입 / 로그인 / 비번 찾기 / 본인인증 / 쿠키 세션 토큰 관리 / 이중 인증 / 보안 등등등    
할일이 너무나 너무나 많다. 또한 로그인 및 가입은 회원을 소유한 어떤 사이트 건간에 핵심 기능이기 때문에  
확장성 및 특히 보안적인 부분에 굉장한 공을 들여야 한다.  
또한 설계 / 코딩적인 영역 뿐 아니라 DB나 인프라적인 부분도 필수적으로 같이 고민해야 함으로 제대로 만드려면,    
꽤 고난이도의 작업이 필요하다.  
  
물론 모두 직접 구현 할수도 있지만, 잘 구현된 서비스를 잘 활용하는것도 매우 좋은 방법임으로,       
회원 및 보안 기능들을 종합 선물 세트처럼 서비스 해주는 AWS cognito 에 대해서 알아보려고 한다.    
정말 쉽게 회원관련 기능들을 거의 모두 제공하며, 그냥 가져다 쓰면 된다....(물론 돈이...)  
커스터마이징이 힘든 부분이 있지만, 로그인 전후 / 회원가입 전후 등 특정 액션 사이사이 비즈니스 로직을    
추가하는건 가능하다.  


### aws cognito 
먼저 aws cognito에 대한 간략한 안내는 아래와 같다.(aws 가이드 발췌)    
> Amazon Cognito는 웹 및 모바일 앱에 대한 인증, 권한 부여 및 사용자 관리를 제공합니다.  
> 사용자는 사용자 이름과 암호를 사용하여 직접 로그인하거나 Facebook, Amazon, Google 또는 Apple 같은  
> 타사를 통해 로그인할 수 있습니다.
> Amazon Cognito의 두 가지 주요 구성 요소는 사용자 풀과 자격 증명 풀입니다.  
> 사용자 풀은 앱 사용자의 가입 및 로그인 옵션을 제공하는 사용자 디렉터리입니다.   
> 자격 증명 풀을 통해 기타 AWS 서비스에 대한 사용자 액세스 권한을 부여할 수 있습니다.   
> 자격 증명 풀과 사용자 풀을 별도로 또는 함께 사용할 수 있습니다.

여기서 사용자 풀과 자격 증명 풀 이렇게 두개 개념이 있는데, 사용자 풀은 쉽게 말해 회원의 정보를 저장하고 있고,  
이를 위한 회원가입/로그인 등을 기능을 제공한다.  
자격 증명 풀은 회원이 어떠한 서비스를 이용할때 엑세스 제어 권한(예를들어 결제 유제만 들어갈수 있는 페이지 라던지) 등을  
제어하는 영역이다.

![](/files/posts/20210309/cognito.png)   

비용은 MAU으로 산정되며 아래와 같다.  
![](/files/posts/20210309/cost.png)

### 실습
간단한 웹 UI를 통해 이메일 인증까지 필요한 회원가입 및 로그인을 구현해 보자.

사용자 풀관리 선택
![](/files/posts/20210309/1.png)

사용자 풀생성 선택
![](/files/posts/20210309/2.png)

설정을 순서대로 진행 선택
![](/files/posts/20210309/3.png)

로그인할때 어떠한 값을 사용할지 선택, 이메일 선택
![](/files/posts/20210309/4.png)

회원가입시 어떠한 값을 필수로 입력 받을지 선택, 이메일과 닉네임 선택 
![](/files/posts/20210309/5.png)

암호를 어떻게 입력 받을지 선택 
![](/files/posts/20210309/6.png)

멀티팩터 인증을 사용할건지 선택 
![](/files/posts/20210309/7.png)

회원가입시 어떠한 인증을 할것인지, 이메일 인증 선택 
![](/files/posts/20210309/8.png)

사용자 디바이스를 기억 할것인지, 아니요 선택 
![](/files/posts/20210309/9.png)

앱 클라이언트 추가, 아직 무언가 만든게 없음으로 패스 
![](/files/posts/20210309/10.png)

어떠한 트리거를 쓸지 선택  
예를 들어 회원가입전 로그인전에 어떠한 비즈니스 로직을 넣을지 선택
![](/files/posts/20210309/11.png)

자 이렇게 되면 사용자 풀이 간단히 생성되게 된다.  
주의할점은 필수 입력 값 이라던지, 이런 부분은 나중에 변경이 아예 안됌으로ㅠ  
초기 설계시 주의해서 설정 해야 한다.  

자 이제 실제 사용자 풀을 사용하기 위한 클라이언트(앱 이나 웹)를 생성해 보자 

화면 왼쪽에 앱 클라이언트 추가를 하고, 클라이언트 이름을 입력한다.  
여기서 엑서스 토큰 만료 시간등의 설정을 할수 있다. 그리고 생성!!  
생성되면 앱 클라이언트 ID가 할당이 된다.  
![](/files/posts/20210309/12.png)

자, 이제 실제 로그인이 가능하도록 도메인을 세팅해보자.  
왼쪽 메뉴에 도메인 선택 후 호스팅할 도메인을 입력해 준다.  
cognito에서 제공하는 호스팅을 선택할수 있고, 기존에 사용했던 도메인 연결도 가능하다.  
![](/files/posts/20210309/14.png)

자 도메인이 준비 되었다면, 로그인/로그아웃후 이동되는 callback 설정 및 oauth scope 설정을 한다.  
![](/files/posts/20210309/13.png)


드디어 실제 로그인/회원가입을 할수 있는 호스팅 UI가 뚝딱? 만들어 졌다.   
아래 URL로 접속 해보자.  
https://your_domain/login?response_type=code&client_id=your_app_client_id&redirect_uri=your_callback_url

짜잔!  
![](/files/posts/20210309/15.png)

회원가입 페이지
![](/files/posts/20210309/16.png)

실제로 회원가입을 해보자.  
시용자 풀 생성시 필수 입력으로 선택했던 부분이 실제로 반영되었다.  
![](/files/posts/20210309/17.png)

이메일 인증도 사용자 풀 생성시 설정 했음으로, 회원가입시 이렇게 이메일 인증을 하라고 나오게 된다.
![](/files/posts/20210309/18.png)


cognito에서 디폴트로 제공한 UI를 그대로 사용하면서, 테스트를 해보았지만  
도메인 지정 및 UI를 사용자 지정으로 할수도 있으며, 또한 앱에서도 동일한 방식으로 로그인/회원가입이  
가능하다.  


## 마치며
앞서 말한대로, 회원가입 및 로그인은 내용이 매우 방대하며 보안적인 요소들을 넣으려면  
굉장히 할일이 많다.  
물론 aws를 사용하려면 그만큼 돈이 들어가지만....빠르게 사이트를 구축하고 사용자 피드백을 받기 위해서는  
이미 집단 지성으로 구축해 놓은 좋은 솔루션들을 적절히 사용하는것도 방법이다.  


## 참고사이트 
https://docs.aws.amazon.com/ko_kr/cognito/latest/developerguide/cognito-user-pools-configuring-app-integration.html
https://docs.aws.amazon.com/ko_kr/cognito/latest/developerguide/what-is-amazon-cognito.html
 