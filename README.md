용감한 남매들 Tech Blog
====================

### push 권한 요청
push 권한 요청 메일 전송
* 수신자 이메일 : singwoosun@gmail.com
* 메일 제목 : [github 권한 요청]bravenamme.github.io
* 메일 내용 : 자신의 giuhub 아이디 전송      


### 로컬 환경 세팅

1. [Jekyll] 설치 - Mac, Windows 모두 가능
2. <https://github.com/bravenamme/bravenamme.github.io> 소스 pull or clone
3. jekyll 실행하여 http://localhost:4000 확인

### post 작성
#### 작성자 프로필 등록
1. _authors 폴더에 본인의 닉네임(영문) 파일 생성

    ex) twosunny.md
    
2. 프로필 사진 추가

    /files/authors/ 폴더에 사진 파일 추가(사진은 되도록 100kb이하)
    
3. 위 1번에서 생성한 파일에 내용추가 (twosunny.md 파일 참고)

4. 작성한 프로필 파일 및 이미지 push


```console

name: twosunny --> 작성자 닉네임
title: 최우선 --> 작성자 이름
image: /files/authors/twosunny.jpg -- 본인의 프로필 사진 경로
```

#### post 작성
1. _posts 폴더에 파일 생성

    파일 생성 규칙
    yyyy-mm-dd-포스트제목(영문).md
    * ex) 2019-05-14-java-image-perpormance.md

2. 생성한 파일 편집

    제일 상단에 글 meta 정보 작성
    ```
    layout: post
    title:  "Java Image Performance" --> 글 제목
    date:   2019-05-14 10:00 --> 글 작성 시간
    author: twosunny --> 글 작성자
    tags:	[java,bufferedImage,pngEncoder] --> 태그
    ```

3. post 글 작성 (markdown 으로 작성)

4. image는 /files/posts/ 폴더에 저장

5. _posts에 작성한 글 및 이미지 파일 push

#### tag 파일 작성
태그를 글에서 자동으로 찾아서 매칭시키는건 jekyll 에서는 어려움이 있어,

post 작성시 tag를 활용하려면 tag파일을 생성해 주어야 한다.

예를들어 post 작성시 tag를 아래와 같이 설정했다고 하자.
```
tags:	[java,bufferedImage,pngEncoder]
```
그렇다면, _tags폴더에 java.md, bufferedImage.md, pngEncoder.md 3개 파일을 생성해준다,

파일 작성시 내용을 보면
```
name: java --> 태그 이름
title: 'java' --> 태그 이름
```
이렇게 작성해 준다.