---
layout: post
title:  "GIT tutorial"
date:   2019-06-11 10:00
author: moohyuk
tags:	[git, git-flow]
---
작성에 앞서 해당 POST 는 git 초보자를 위한 글 임을 밝힙니다.  
요즘 대세는 git 인데 우리는 여전히 svn 에 의존하고 있습니다. svn 은 개념적으로 접근이 쉽습니다. 개발자는 늘 새로운것에 친숙해야 하는데 쉬움에 동반되는 편안함이 변화와 트랜드를 멀리하게 만드는 것은 아닐까요? (제 얘기 입니다. 비단 버전 컨트롤에 해당 되는 이야기는 아닌것 같지만...)  
소위 잘나가는 개발사와 개발자들은 git 으로 형상 관리를 하고 있습니다. 현장에서 git 을 쓰는데는 나름의 이유가 있을 것이고 이 글에서는 그 이유를 공유하고 초보자가 접근하기 쉽도록 간단한 기능을 정리해 보고자 합니다.
  
작성한 내용은 여러 웹사이트 및 블로그들을 읽어보며 정리한 내용 입니다.

## GIT 의 장단점
### 장점
* 자신만의 commit history 관리가 가능하다. 
* 작은 용량과 손쉬운 로컬 브랜치 생성.   
* commit 이 바로 서버에 영향을 미치지 않으며 원하는 커밋만 원격 저장소로 보낼 수 있다. 통합 관리자가 존재할 경우 통합관리자가 원하는 커밋만을 적용 할 수 있다.
* 서버 저장소와 개인의 저장소가 독립적인 commit history 를 갖음으로써 유연한 source 관리가 가능하며 이것이 가장 큰 장점 이다.


Ref.[http://seungzzang.blogspot.com][seungzzang]![fig1.git commit history 관리](/files/posts/ffm-git.png)  

### 단점
* 개념적으로 어려우며 러닝커브가 높다
* 여러 개발자의 branch 를 관리하는 리소스 필요 (규모가 큰 프로젝트의 경우 통합 관리자 필요)

***

## stage 와 unstage 그리고... commit
### stage 
저장소에서 변경된 파일을 커밋 대상에 포함되도록 예약하는 것을 스테이지라고 합니다. 커밋을 위해서는 파일을 stage 상태로 만들어야 합니다. 예를 들어 이클립스 git-plugin 의 경우 "add to index", bash 에서는 "git add <파일경로>" 를 통해 stage 상태로 변경 할 수 있으며 해당 파일은 스테이지 영역에 포함되게 됩니다.

### unstage
stage 상태의 파일을 스테이지 영역에서 제외하여 커밋 대상에 포함하지 않을 수 있습니다.

***

## 이력 되돌리기
작업을 하다보면 push 한 commit 을 되돌려야 하는 경우가 종종 발생 합니다. 개인 저장소에 이력을 되돌리는 것은 쉽습니다. 하지만 개인이 혼자 사용하는 저장소가 아닌 공용으로 사용하는 remote 저장소에 push 를 한 경우에는 어떻게 해야 할까요?

### RESET
히스토리를 남기고 싶지 않다면 RESET 을 사용하면 됩니다
 * git reset --hard {hash} (강제로 hash 값에 해당하는 커밋 내용을 제거 한다.)
 * git reset --hard HEAD^ (최신 commit id(HEAD) 를  이전으로 되돌린다.)

위 명령어를 통해 로컬 저장소의 커밋을 이전으로 되돌리고 나면 remote 의 커밋보다 이전 버전이 되기 때문에 remote 에 push 를 실행할 수 없습니다. 이 경우에 강제로 push 명령어를 사용해야 합니다. 
 * git push -force  

다만 위와같이 처리 할 경우 다른 사용자가 reset 이전의 버전을 내려받아 사용중이라면 소스 버전은 두개로 분리되게 됩니다. 해당 사용자는 여전히 reset 이전의 히스토리를 가지고 있기 때문에 소스를 병합할 때 어려움을 맞이하게 되며 이러한 사용자가 존재 하는지 확인할 방법이 없기 때문에 reset 을 사용할 때는 조심해야 합니다.
(해당 개발자의 local/master 의 HEAD 를 현재와 맞춰야 하는 상황이 발생)

Ref.[https://www.atlassian.com][atlassian]![fig2.git reset](/files/posts/reset.png)  

### REVERT
위와 같은 상황을 피하고 싶다면 revert 를 사용하면 됩니다.
 * git revert {되돌릴 commit hash}  
commit history 가 commit_a -> commit_b -> commit_c 와 같이 되어 있다고 가정 합니다.

이를 되돌리기 위해서는 역순으로 실행을 취소해야 합니다.
revert commit_c -> revert commit_b -> revert commit_a 
REVERT 명령어를 사용할 때는 위와 같이 되돌리고 싶은 commit 수 만큼 revert 를 추가해야 합니다. 다만 revert 를 위한 commit 의 갯수가 많을 경우 특정 커밋의 hash아닌 [되돌리기 위한 커밋 범위] 를 입력하여 처리도 가능 합니다.
 * git revert --no-commit HEAD~3.. # 또는 master~3..master


Ref.[https://www.atlassian.com][atlassian]![fig3.git revert](/files/posts/revert.svg)

***

## git-flow 전략이란?
git-flow 전략은 소프트웨어의 소스코드를 관리하고 출시하기 위한 '브랜치 관리 전략'(branch management strategy)입니다. 백문이 불여일견! 간략한 설명을 위해 우선 아래 이미지를 봅시다.

이미 앱 개발자들과는 많은 논의를 거친 이미지 입니다.
Ref.[https://nvie.com][git-branch-model]![fig4.git-flow 전략](/files/posts/gitflow.png)

### master branch(배포)
배포 되었거나 배포준비(production-ready)된 코드는 `origin/master` 로 관리 합니다.
git-flow 에서 master 브랜치에 `병합` 한다는 것은 새버전을 배포 한다는 것을 의미 합니다. master 브랜치에서 커밋될 때 git hook 스크립트를 걸어서 자동으로 빌드하여 운영서버로 배포하는 형식을 취하며 배포 방식은 프로젝트별로 차이가 있을 수 있으니 참고 바랍니다.

### develop branch(개발)
다음 배포를 위한 개발 코드는 `origin/develop` 로 관리 합니다. 프로젝트에 참여하는 개발자들이 함께 업무를 진행하는 브랜치이며 develop 브랜치의 코드가 안정화되고 배포할 준비가 되면 `master`로 `병합` 하고 배포버전으로 태그를 달 수 있습니다.

### feature, realease, hotfix branch(보조)
배포를 준비하고, 이미 배포한 제품이나 서비스의 버그를 빠르게 해결(hotfix) 해야 합니다. 이 모든 것을 동시에 진행하기 위해서 다양한 브랜치가 필요 합니다.

* ##### feature (기능 브랜치)  
특정 기능을 개발할 때 사용하는 브랜치 입니다. 요구사항에 의해 develop 브랜치에서 생성되며 요구사항 별로 하나씩 생성하여 기능 개발후 develop 브랜치로 머지할 수 있습니다.  
``` 
  시작브랜치: develop  
  병합대상 브랜치: develop
  브랜치이름 규칙: master, develop, release-, hotfix- 를 제외한 것  
```  

Ref.[https://nvie.com][git-branch-model]![fig5.feature branch](/files/posts/git-feature.png)

* ##### release (출시/QA 브랜치)  
`release` 브랜치는 실제 배포할 상태가 된 경우에 생성하는 브랜치 입니다. 해당 브랜치를 QA 에 전달해 최종 테스트를 수행할 수 있습니다. `master` 브랜치를 통해 배포 해야하므로 QA가 완료되면 `release`를 `master` 브랜치로 병합 합니다. (향후 관리를 위해 태그를 만들어 현재 병합되는 커밋을 가리킨다.) 이 때 배포된 기능에 반영될 수 있도록 `develop` 브랜치에도 함께 병합되어야 합니다.  
```
  시작브랜치: develop
  병합대상 브랜치: develop, master
  브랜치이름 규칙: release-*
``` 

* ##### hotfix 브랜치
미리 계획되지 않은 배포를 위한 브랜치 입니다. 기본적인 동작 방식은 `release`와 비슷 합니다. 운영 버전에 치명적인 버그 발생시 빠른 해결을 위해 `master` 브랜치에 만들어 둔 `tag`로 부터 `hotfix` 브랜치를 생성 합니다.  
``` 
  시작브랜치: master
  병합대상 브랜치: develop, master
  브랜치이름 규칙: hotfix-*
``` 

Ref.[https://nvie.com][git-branch-model]![fig6.hotfix branch](/files/posts/git-feature.png)

***  

### git 기본명령어
* 처음 시작하기
  * git init //git 로컬 저장소 생성
  * git clone <로컬/저장소/경로>
  * git clone 사용자명@호스트: //원격/저장소/경로

* 추가와 커밋
  * git status
  * git add <파일이름> 또는 git add .
  * git commit -m "이번 확정본에 대한 설명"

* 변경 내용 원격서버로 배포
  * git push origin master

* 브랜치 변경하기 
  * git checkout <브랜치명>
  * git remote add <원격서버주소> //로컬 저장소 원격 연결

* 브랜치 만들기
  * git branch <브랜치명> / git checkout <브랜치명>
  * git checkout -b <브랜치명>

* 브랜치 삭제
  * git branch -d <브랜치명>

* 브랜치 병합
  * git checkout master (병합할 베이스 저장소)
  * git merge <브랜치명>

* 브랜치 목록
  * git branch 
  * git brnach -r

***

### ref.
  * [http://rogerdudler.github.io][rogerdudler.github.io]  
  * [https://git-scm.com][git-scm]  
  * [http://seungzzang.blogspot.com][seungzzang]  
  * [https://www.atlassian.com][atlassian]  
  * [https://nvie.com][git-branch-model]
  * [https://gist.github.com/ihoneymon][ihoneymon]



[rogerdudler.github.io]: http://rogerdudler.github.io/git-guide/index.ko.html
[git-scm]: https://git-scm.com
[seungzzang]: http://seungzzang.blogspot.com/2013/04/git-svn-svn-git.html
[git-branch-model]: https://nvie.com/posts/a-successful-git-branching-model/
[atlassian]: https://www.atlassian.com/git/tutorials/resetting-checking-out-and-reverting
[ihoneymon]: https://gist.github.com/ihoneymon/a28138ee5309c73e94f9