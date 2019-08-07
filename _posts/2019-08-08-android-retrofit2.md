---
layout: post
title:  "Retrofit2"
date:   2019-08-07 10:00
author: kapjong
tags:	[android, kotlin, retrofit2, okhttp, https, ssl, SSLProtocolException]
---

## 인트로
kotlin 개인 스터디 진행 중 어느 정도 기본 문법이 익숙해 지니 네트워크를 통한 데이터 취득을 해볼까?
하는 마음에 네트워크 통신 라이브러리 검토 중 문득 팀동료에 메세지가 생각났다! 이번 포스팅은 retrofit2에대한 간략한 소개를 진행 합니다.
![](/files/posts/201908/who_.png)

## retrofit2
retrofit2이란 okhttp + rxkotlin + json파싱 라이브러이가 혼합된 네트워크 통신 라이브러리 입니다.<br>
개발자가 네트워크 통신 시 신경 써야한 시스템 오류, 스레드 처리, 문자열 파싱 처리등에 편의성을 제공 하며,<br><font color='red'>REST api 통신에 특화 되어 있습니다.</font><br>
웹서핑 결과 kotlin 언어 도입과 마추어 retrofit2가 도입 되었으며 다수에 메이져 프로젝트에 쓰인다는 소문? 이 있어 구현 및 테스트를 진행 하였습니다.<br>

## [Square, Inc](https://squareup.com)
Square는 Twitter 제작자 Jack Dorsey 공동 창립한 모바일 환경 카드 리더기 회사 입니다.<br>
retrofit외 Square, Inc 에서는 다양 한 <b>[[오픈 소스 라이브러리]](https://github.com/search?l=Java&q=android&type=Repositories)</b>를 공유 하고 있습니다.<br>

## [Square Open Source](https://square.github.io/)
As a company built on open source, here are some of the internally-developed libraries we have contributed back to the community.<br>
우리 회사는 오픈 소스 기반에 회사이며, 자사에서 진행한 프로젝트 내부라이브러리 자원을 사회 공헌 차원으로 오픈 소스화 하여 운영 하고 있습니다.(구글 번역이 마음에 들지 않아 의역 했습니다.)<br>

## 구현
구현 방법 및 간단한 사용법에 대한 내용은 <b>[retrofit 공식 페이지](https://square.github.io/retrofit/)</b>에서 확인 할수 있습니다.<br>

간단한 kotlin 코드 구현 예)
```java
private fun getData() {
    compositeDisposable = CompositeDisposable()
    compositeDisposable.add(
        ApplicationTest.apiManager().characters.subscribeOn(Schedulers.io())
            .observeOn(AndroidSchedulers.mainThread())
            .subscribe(
                { characterData ->
                    // 서버 에러
                    characterData.error?.let { ErrorHandling.init(this, characterData.error) }

                    // 데이터 취득
                    characterData?.let { characterData(characterData) }
                },
                // 시스템 에러
                { error -> error?.let { ErrorHandling.init(this, error) }
            }
        )
    )
}

interface TestAPI {
    @get: GET("characters?apikey=XXXXXXXXXXXXXXX")
    val characters: Observable<Character>
}
```
인터 페이스 호출 시 어노테이션을 식 호출 적용 하여 직관 적인 유지보수 장점!

## 성능
retrofit2는 기본적으로 OkHttp를 네트워킹 계층으로 활용하며 그 위에 구축 됩니다.<br>
탁월한 네트워크 성능이 보장 됩니다. 아래는 속도 테스트 진행 포스팅 입니다.
![http://instructure.github.io/blog/2013/12/09/volley-vs-retrofit/](/files/posts/201908/tIdZkl3_.png)

## javax.net.ssl.SSLProtocolException
retrofit 가이드 페이지 및 구글링을 통해 구현이 했지만 앱 크래시가 발생 했다!!!
![](/files/posts/201908/boom_.gif)

원인은 테스트 진행한 open aip 서버에 보안 통신 https 가 적용 되어 있어서 였습니다!<br>
해결 방안은 <b>[[okhttp HTTPS 클라이언트 설정]](https://square.github.io/okhttp/https/)</b> 부분에 관한 내용이 상세하게 정리 되어 있습니다.<br>
```java
ConnectionSpec spec = new ConnectionSpec.Builder(ConnectionSpec.MODERN_TLS)
    .tlsVersions(TlsVersion.TLS_1_2)
    .cipherSuites(
          CipherSuite.TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256,
          CipherSuite.TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256,
          CipherSuite.TLS_DHE_RSA_WITH_AES_128_GCM_SHA256)
    .build();

OkHttpClient client = new OkHttpClient.Builder()
    .connectionSpecs(Collections.singletonList(spec))
    .build();
```
HTTPS 서버에 대한 연결을 협상 할 때 OkHttp는 통신 대상 서버 TLS 버전 과 암호 제품군 을 알아야 합니다.<br>
tlsVersions, cipherSuites 보안 통신 정보는 <b>[[www.ssllabs.com]](https://www.ssllabs.com/ssltest/analyze.html?d=github.com)</b> 에서 확인 가능 합니다.

## 마무리
<ol>
  <li>코틀린 개발 환경에 최고에 궁합</li>
  <li>탁월한 네트워크 성능</li>
  <li>구현 및 사용방법이 단순</li>
  <li>네트워크 스레드 처리에 신경 쓰지 않아도 됨</li>
  <li>REST api 통신에 특화 되어 있으며</li>
  <li>json 데이터 파싱도 해줘요!!</li>
</ol>
안드로이드 kotlin 개발환경에서 네트워크 모듈을 찾고 있다면 retrofit2를 추천 합니다.