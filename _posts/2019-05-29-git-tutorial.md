---
layout: post
title:  "GIT tutorial"
date:   2019-05-29 10:00
author: moohyuk
tags:	[git, tutorial]
---
글 작성에 앞서 해당 POST 는 git 초보자를 위한 글 임을 밝힙니다.  
요즘 대세는 git 인데 우리는 여전히 svn 에 의존하고 있습니다. svn 은 개념적으로 접근이 쉽습니다. 글쓴이 또한 이런 쉬움이 제공하는 편안함에 변화와 트랜드를 등한시 하고 안주 하고 있었습니다.(비단 버전컨트롤에 해당 되는 이야기는 아닌것 같다. ㅜㅜ)  
소위 잘나가는 개발사와 개발자들은 git 으로 형상 관리를 하고 있습니다. 현장에서 git 을 쓰는데는 나름의 이유가 있을 것이고 이 글에서는 그 이유를 공유하고 초보자가 접근하기 쉽도록 간단한 기능을 정리해 보고자 합니다.
  
작성한 내용은 여러 웹사이트 및 블로그들을 읽어보며 정리한 내용 입니다.

## 장단점
### 장점
* 자신만의 commit history 관리가 가능하다. 
* commit 이 바로 서버에 영향을 미치지 않으며 원하는 커밋만 원격 저장소로 보낼 수 있다. 통합 관리자가 존재할 경우 통합관리자가 원하는 커밋만을 적용 할 수 있다.
* 서버 저장소와 개인의 저장소가 독립적인 commit history 를 갖음으로써 유연한 source 관리가 가능하며 이것이 가장 큰 장점 이다.


![git commit history 관리](/files/posts/ffm-git.png)
### 단점
* 개념적으로 어려우며 러닝커브가 높다
* 여러 개발자의 branch 를 관리하는 리소스 필요 (규모가 큰 프로젝트의 경우 통합 관리자 필요)


### stage, unstage 개념



### git 기본
* 처음 시작하기
  * git init //git 로컬 저장소 생성
  * git clone <로컬/저장소/경로>
  * git clone 사용자명@호스트: //원격/저장소/경로

* 추가와 확정
  * git add <파일 이름>
  * git add .
  * git commit -m "이번 확정본에 대한 설명"

* 변경 내용 발행
  * git push origin master

* 브랜치 변경하기 
  * git checkout <branch>
  * git remote add <원격 서버 주소> //로컬 저장소 원격 연결

* 브랜치 만들기
  * git branch <name> / git checkout <name>
  * git checkout -b <name>

* 브랜치 삭제
  * git branch -d <name>

* 브랜치 병합
  * git checkout master (병합할 베이스 저장소)
  * git merge <branch>

* 브랜치 목록
  * git branch 
  * git brnach -r


### git RESET, REVERT 차이



* ref.
  * [git-간편안내서][rogerdudler.github.io]  
  * [git-scm.com][git-scm]  
  * [seungzzang.blogspot.com][seungzzang]  



[rogerdudler.github.io]: http://rogerdudler.github.io/git-guide/index.ko.html
[git-scm]: https://git-scm.com
[seungzzang]: http://seungzzang.blogspot.com/2013/04/git-svn-svn-git.html