---
layout: post
title:  "svn 사용자를 위한 git 안내서"
date:   2022-01-24 00:00
author: q_lazzarus
tags:	[git, svn]
---

## 들어가며
여태까지 많은 git 안내서가 있지만, 이번에는 subversion (aka: svn) 사용자를 위한 타겟으로 글을 작성해보겠습니다.

![subversion](https://upload.wikimedia.org/wikipedia/commons/thumb/2/22/Apache_Subversion_logo.svg/1200px-Apache_Subversion_logo.svg.png)
![git](https://media.vlpt.us/images/_seeul/post/a13ec304-4219-49f9-b294-145e79459532/img.jpeg)

## git 의 주요한 특징
먼저 git 은 subversion 과 달리 로컬에서도 원격에서도 저장이 됩니다. (원격 저장소를 여러개 두는 것도 가능하지만 여기서 다루지는 않겠습니다.)
그러다보니 commit 만 하고 끝이지 않냐? 라는 관성 때문에 많이들 고생하시고 계십니다..
이번은 주요한 소스 올리기와 공유 에 중점으로 다뤄보겠습니다.

## 데이터 받아오기
svn 의 checkout 명령어 처럼 git 의 저장소를 가져오는 명령어는 clone 입니다.

로컬 저장소를 복제(clone)하려면 아래 명령을 실행하세요.

```bash
$ git clone /로컬/저장소/경로
```

원격 서버의 저장소를 복제하려면 아래 명령을 실행하세요.

```bash
$ git clone 사용자명@호스트:/원격/저장소/경로
```

## 데이터 가져오기
svn 의 update 명령어 처럼 git 의 저장소에서 데이터를 가져오는 명령어는 pull 입니다.

```bash
$ git pull
```

## 데이터 저장하기
svn 의 add / commit 명령어 처럼 git 의 저장소로 데이터를 저장하는 명령어는 git 도 동일합니다.

> 하지만 svn 과 다르게 한가지 동작이 더 필요합니다

기존 svn 에서는 다음의 명령어이면 서버로 업로드하는것이 끝나지만

```bash
$ svn add * --force
$ svn commit -m "add files"
```

git 에서는 다음의 명령어이면 **로컬 저장소** 로 저장이 됩니다.

```bash
$ git add *
$ git commit -m "add files"
```

이후로 서버에 업로드 하기 위해서는 다음의 명령어가 더 필요합니다!

```bash
$ git push origin master
```

## 그리고 나머지들

git 은 svn 과 달리 merge / branch 생성이 비교적 쉽습니다. 하지만 이 단계에서 이야기할 것은 아닌 것 같습니다.

Ref.
* [git 이해하기](https://bravenamme.github.io/2021/09/01/Git/)
* [git tutorial](https://bravenamme.github.io/2019/06/11/git-tutorial/)
* [git - 간편 안내서](https://rogerdudler.github.io/git-guide/index.ko.html)
* [svn 사용자의 git 사용 후기](https://www.abel9999.com/2020/05/svn-git.html)