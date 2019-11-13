---
layout: post
title:  "서버리스(serverless) 아키텍쳐 및 Lambda"
date:   2019-09-11 10:00
author: twosunny
tags:	[serverless,lambda,aws,azure,function,apigateway]
---

# 시작하며
날씨정보를 제공해 주는 웹 애플리케이션 혹은 앱을 만들다고 가정해보자.   
SNS 계정을 연동하여 그 날씨에 대한 사용자 리뷰도 필요하다고 한다.   
먼저 대략적으로 시스템을 구성해 보면 웹서버, 애플리케이션 서버, DB 서버, 캐시서버
등등이 구성될수 있다.

그 후, 각 서버 목적에 맞는 소프트웨어 및 툴을 설치해야 하고, 환경 세팅해야 하고, 네트웍 설정,   
보안 설정 등등 실제 코딩을 하기 전에 해야 할일들이 꽤 많다.   
또한 실제 개발이 들어가서 코딩을 할때에도 각 플랫폼, 언어에 맞는 설정 및 세팅등의 시간등 그 시간 또한   
무시할수 없는 시간들이다. 

만약 서버 구성등의 외부 환경은 신경을 거의 쓰지 않고, 비즈니스 로직에만 집중할수 있다면 개발 속도나 컨텐츠의   
생산성 또한 빨라질것 이다.

이번 포스팅에서는 이런 이유로, 서버가 없는?? 서버리스(serverless) 아키텍쳐 및 그 중심에 있는 AWS Lambda에 대해서
포스팅 하려고 한다. 

# Server less?
서버리스 아키텍쳐는 개인적으로 정의하자만, **서버를 고려하지 않고 백앤드 기능이나 Function을 구현하여 그것을 클라우드에 올려놓고
필요할때 호출하는 것** 이라고 정의하고 싶다. 

그런데 진정 진짜로 레알로 서버가 없단 말인가!!
![출처:https://news.sbs.co.kr/news/endPage.do?news_id=N1005360798](/files/posts/nolran.jpg)

서버가 없는데 당연히 내가 구현한 로직들이 돌아가지 않을것이다.^^   
로직을 실행하는 서버는 존재하나 그 서버는 클라우드상에 구축되어 쉽게 말해 숨어? 있고,
그러한 숨은 서버는 실제 개발시 고려하지 않아도 된다.   
서버리스는 여러 개념(Paas, Baas, Faas)과 물려 있는데 이번 포스팅에서는 FaaS (Function as a Service) 에 해당하는 AWS Lambda를 활용하여
직접 구현해 보도록 하겠다.

**개념들을 정리하면**
### FaaS (Function as a Service)
>FaaS 는 프로젝트를 여러개의 함수로 쪼개서 (혹은 한개의 함수로 만들어서), 
>매우 거대하고 분산된 컴퓨팅 자원에 여러분이 준비해둔 함수를 등록하고, 
>이 함수들이 실행되는 횟수 (그리고 실행된 시간) 만큼 비용을 내는 방식.

### Lambda
>FaaS 서비스의 한 형태로, AWS 에서 제공

# AWS Lambda를 활용한 serverless 구현

먼저 FaaS 를 제공하는 곳은 대표적으로 아래 세곳이다.
* AWS Lambda
* Google Functions
* Azure Functions

이번 포스팅에선 Lambda를 활용하도록 하겠다.   
Lambda를 활용한 아키텍쳐 예시는 아래와 같다.

### 웹 애플리케이션 백앤드
![Lambda 서버리스 아키텍쳐
(출처:https://aws.amazon.com/ko/serverless/)](/files/posts/lambda.png)

### 소셜 미디어 앱에 대한 모바일 백엔드
![Lambda 서버리스 아키텍쳐
(출처:https://aws.amazon.com/ko/serverless/)](/files/posts/lambda2.png)

### 이미지 썸네일 생성
![Lambda 서버리스 아키텍쳐
(출처:https://aws.amazon.com/ko/serverless/)](/files/posts/lambdaw3.png)

자 구체적으로 Lambda를 구현해보자!

### 새로운 Lambda 만들기
[AWS Lambda Console](https://ap-northeast-2.console.aws.amazon.com/lambda/home?region=ap-northeast-2#/begin) 로 이동 후,
함수 생성을 클릭한다.
![함수 생성](/files/posts/aws1.png)

그 다음 새로 작성 선택후, 함수 이름에 hello_lambda, Rumtime에 Java 8 을 선택하였다.   
런타임은 개발하기 편한 프로그래밍 언어로 선택 가능하다.
![람다 새로 작성](/files/posts/aws2.png)

역할은 람다에 부여하고싶은 권한을 설정하게 된다. [기본 Lambda 권한을 가진 새 역할 생성] 선택후 Create Function!
![역할 설정](/files/posts/aws3.png)

만들어 졌다!!! 참 쉽죠잉?
![생성후 화면](/files/posts/aws4.png)

자, 이제 코드를 편집해보자...해보자..해보자...음????   
소스 편집기가 없다! 어디있지??? 아무리 찾아봐도 없다.. AWS 가이드 찾아보니.
>블루프린트는 Python 또는 Node.js로 작성된 샘플 코드를 제공합니다. 
>콘솔의 인라인 에디터를 사용하여 예제를 손쉽게 수정할 수 있습니다. 
>그러나 Java로 Lambda 함수의 코드를 작성하고 싶은 경우에는 블루프린트가 제공되지 않습니다. 
>또한 콘솔에서 Java 코드를 작성할 수 있는 인라인 에디터가 없습니다.
>
>즉, Java 코드를 작성하고 콘솔 밖에서 배포 패키지를 생성해야 한다는 뜻입니다. 
>배포 패키지를 생성하고 나면 콘솔을 사용하여 Lambda 함수를 생성하기 위해 AWS Lambda에 패키지를 업로드할 수 있습니다. 
>또한 콘솔을 사용하여 수동 호출을 통해 함수를 테스트할 수도 있습니다.

그렇다 java는 aws 에서 제공하는 인라인 에디터를 사용 할 수 없다는 뜻이다.
즉 밖에서 코드짜고 테스트 하고 lambda에 올려야 한다.  
귀찮으니, 런타임을 node로 바꿔주자! 오오 코드 편집기가 보인다!
![람다 코드 수정](/files/posts/aws5.png)

그럼 이제 http 요청이 오면 만든 함수를 호출하도록 해보자.   

구성 부분에서 [트리거 추가]를 클릭
![트리거 추가](/files/posts/aws6.png) 

[API 게이트웨이] 선택!
![API 게이트웨이 선택](/files/posts/aws7.png) 

그 다 보안은 [열기]로 선택. 즉, 모든 request가 접근 가능하게 된다. 테스트 용도이니 다 허용해 주자.   
그 다음 Add!
![API 게이트웨이 설정](/files/posts/aws8.png) 

함수 메인 화면으로 넘어가게 되며, API Gateway가 추가된 걸 볼수 있다.
![API 게이트웨이 설정 완료](/files/posts/aws9.png)

그리고 밑에 API 게이트웨이쪽에 상세정보를 보면 URL이 하나 보일 것이다.   
이 URL을 클릭하면 아까 만든 Lambda 함수가 호출된다. 클릭해보자!
![API 게이트웨이 URL](/files/posts/aws10.png)

짜잔~~~ 함수가 호출되어 response 된 걸 볼수 있다.
![API 게이트웨이 URL](/files/posts/aws11.png)

# 마치며
개인적으로는 복잡하고 많은 비즈니스 로직을 담고 있는 대형 애플리케이션에는 어울리지 않은듯 하지만,
왠만한 중소형 앱 구축이나 백단 로직에서 이루어지는 로그 분석, 이미지 인코딩 등 펑션별로 명확히 기능들을 나누어 
기존 애플리케이션과 융합하여 사용하면 꽤 좋은 개발 퍼포먼스가 이루어 질것 같다. 

## 참고 사이트
* https://aws.amazon.com/ko/serverless/
* https://velopert.com/3543
* https://velopert.com/3546





