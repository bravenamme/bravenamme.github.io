---
layout: post
title:  "Web Server VS WAS"
date:   2021-07-08 10:00
author: ryan
tags:	[]
---

## Web Server 란?

웹브라우저로 부터 HTTP 요청을 받아 HTML 문서와 같은 정적 컨텐츠를 응답하는 프로그램 입니다.

정적 컨텐츠 요청시 html, image, css ... 등을 제공 할 수있습니다.

동적 컨텐츠 요청시 Web Application Server 로 전달하여 WAS 가 처리한 결과를 Response 합니다.

대표적인 Web Server 에는 Apache, NGINX 와 Windows 전용 WEB 서버인 IIS 가 있습니다

## 정적 컨텐츠 및 요청

Web Server 는 파일 경로의 이름을 요청 받아 경로와 일치하는 File Contents 를 반환합니다. 그래서 항상 동일한 페이지를 반환합니다.

요청 인자 값에 상관없이 달라지지 않는 컨텐츠를 말합니다. (Image, HTML, CSS, JavaScript).

![fig1. Static Pages](/files/posts/202107/fig1.png)
([https://has3ong.github.io/webwas/](https://has3ong.github.io/webwas/))

### **Nginx와 Apache의 관계**

- Apache 웹 서버는 현재 세계에서 가장 인기있는 웹 서버이며 오픈소스 입니다.
- Nginx는 새로운 시대의 요청에 부응해서 만들어진 웹서버 이며, 개발의 모든 목적이 "높은 성능"에 맞춰져 있습니다.

## Web Application Server(WAS) 란?

DB 조회나 다양혼 로직 처리를 요구하는 동적인 컨텐츠를 제공하기 위해 만든 프로그램 입니다.

HTML 만으로는 할 수 없는 데이터베이스 조회나 다양한 로직처리 같은 동적인 컨텐츠를 제공하기 위해 만들어진 어플리케이션 서버입니다. HTTP 를 통해 컴퓨터나 장치에 어플리케이션을 수행해주는 미들웨어라고도 합니다

Web Container / Servlet Container 라고도 불리며, Web Server + Web Container 의 역할을 수행합니다.

대표적인 WAS 에는 Tomcat, Jeus, JBoss, Web Sphere 등이 있습니다

## 동적 컨텐츠 및 요청

요청 인자의 내용에 맞게 동적인 File Contents 를 반환합니다.

예를 들어, Web Server 에 의해서 실행되는 프로그램을 통해서 만들어진 결괌루이나, WAS 위에서 돌아가는 Java Program 인 Servlet 을 이용합니다. 개발자는 이 Servlet 에 `doPost()` 나 `doGet()` 같은 메소드를 구현합니다.

![fig2. Dynamic Pages](/files/posts/202107/fig2.png)
([https://has3ong.github.io/webwas/](https://has3ong.github.io/webwas/))


## **Web Server 와 WAS 를 함께 쓰는 이유**

Web Server 와 WAS 를 서로 분리한 이유는 여러가지가 있습니다.

여러 장점과 더불어 서비스의 확장성과 안정성을 고려한다면 함께 사용하는 것이 유리 합니다. 

![fig3.Web server 와 WAS](/files/posts/202107/fig3.png)


### 보안

SSL / TLS 에 대한 암복호화 처리에 Web Server 를 사용합니다. 또한 Web Server 는 Proxy 같은 역할도 수행할 수 있습니다.

### 책임 분할을 통한 서버 부하 방지

WAS 는 DB 조회와 다양한 비즈니스 로직을 처리를 담당하는 동적 컨텐츠를 처리하고 단순한 정적 컨텐츠는 Web Server 에서 빠르게 처리하는게 좋습니다.

### 로드밸런싱을 통한 서버 부하 방지

Web Server 에서 Load Balancing 을 수행할 수 있습니다. 따라서 Fail Over, Fail Back 처리에 유용하여 무중단 배포를 위한 장애 극복에 쉽게 대응할 수 있습니다.

### Health check

서버에 주기적으로 HTTP 요청을 보내서 서버의 상태를 확인 할 수 있습니다.

## Web Server / WAS Architecture

![fig4. Web Service Architecture](/files/posts/202107/fig4.png)
([https://gmlwjd9405.github.io/2018/10/27/webserver-vs-was.html](https://gmlwjd9405.github.io/2018/10/27/webserver-vs-was.html))

![fig5. 3 Tier Web Server Architecture](/files/posts/202107/fig5.png)
Fig5. 3 Tier Web Server Architecture ([https://www.researchgate.net/figure/A-Typical-3-Tier-Server-Architecture-Tier-1-Web-Server-Tier-2-Application-Server-Tier_fig1_221147997](https://www.researchgate.net/figure/A-Typical-3-Tier-Server-Architecture-Tier-1-Web-Server-Tier-2-Application-Server-Tier_fig1_221147997))

1. Client 가 Web Server 로 HTTP Request 를 전송한다.
2. Web Server 는 Client 의 Request 를 WAS 로 전송한다.
3. WAS 는 관련된 Servlet 을 메모리에 올린다.
4. WAS는 web.xml을 참조하여 해당 Servlet에 대한 Thread를 생성한다.
5. HttpServletRequest 와 HttpServletResponse 객체를 생성하여 Servlet에 전달한다.
6. Thread 는 Servlet 의 `service()` 메서드를 호출한다.
7. `service()` 메서드는 요청에 맞게 `doGet()` 또는 `doPost()` 메서드를 호출한다.
    1. `protected doGet(HttpServletRequest request, HttpServletResponse response)`
8. WAS 는 해당 로직을 수행하다 DB 접근이 필요하면 Database 에 SQL Query 를 한다.
9. Database 는 SQL Query 에 따른 결과값을 Response 에 담아서 반환한다.
10. `doGet()` 또는 `doPost()` 메서드는 인자에 맞게 생성된 적절한 동적 페이지와 쿼리 결과를 Response 에 담아 WAS 에 전달한다.
11. WAS 는 Response 를 HttpResponse 형태로 바꾸어 Web Server 에 전달한다.
12. 생성된 Thread 를 종료하고, HttpServletRequest 와 HttpServletResponse 객체를 제거한다.
13. Web Server 는 HTTP Response 를 Client 에게 응답한다.


Ref.

- [우아한형제들 테코톡](https://www.youtube.com/watch?v=z5fUkck_RZM)
- [https://github.com/benelog/blog/issues/27](https://github.com/benelog/blog/issues/27)
- [DTO와-VO-의문에-대한-답변-DTO와-VO는-왜-혼용되는가](https://bperhaps.tistory.com/entry/DTO%EC%99%80-VO-%EC%9D%98%EB%AC%B8%EC%97%90-%EB%8C%80%ED%95%9C-%EB%8B%B5%EB%B3%80-DTO%EC%99%80-VO%EB%8A%94-%EC%99%9C-%ED%98%BC%EC%9A%A9%EB%90%98%EB%8A%94%EA%B0%80-DTO%EB%8A%94-%EC%99%9C-%EC%82%AC%EC%9A%A9%ED%95%98%EB%8A%94%EA%B0%80-VO%EB%8A%94-%EB%AC%B4%EC%97%87%EC%9D%B8%EA%B0%80)
- [https://hyeon9mak.github.io/DTO-vs-VO/](https://hyeon9mak.github.io/DTO-vs-VO/)