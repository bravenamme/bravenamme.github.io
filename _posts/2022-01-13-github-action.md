---
layout: post
title:  "Github Action Basic"
date:   2022-01-12 20:00
author: firepizza
tags:	[github]
---

> github 에서 제공하는 공식 hello-world 튜토리얼을 진행하면서 적는 일지.

[https://lab.github.com/githubtraining/github-actions:-hello-world](https://lab.github.com/githubtraining/github-actions:-hello-world) 

### 배우게 될 것

- workflow file을 구성하는 방법
- 실행가능한 script를 추가하는 방법
- workflow와 action blocks를 만드는 방법
- workflow를 실행시키는 방법
- workflow log를 확인하는 방법

---

> Github action은 무엇인가?
> 
- 테스트 자동화 (CI)
- CD
- github의 이슈, @멘션, 라벨 등을 이용한 workflow trigger
- 코드리뷰 진행
- 브랜치 관리
- 이슈, PR관리

라고 한다..

---

> Actions and Workflows
> 
- 액션과 워크플로우에 대한 설명
- 깃헙액션에는 2개의 컴포넌트가 있는데 그것이 바로
    - **action** 그 자체
    - action을 이용하는 **workflow**
- workflow는 여러개의 action을 가질 수 있고,  action은 그 자체의 목적이 있다.

### Action의 종류

- container actions
    - Docker **container actions**는 github에서 호스팅한 linux 환경에서 실행되는 실행환경이다.
- javascript actions
    - github-action code를 환경에서 떼어서(decouple)  빠르게 실행하고 의존성을 관리하도록 한다.

의 2가지로 구분될 수 있다.

---

> STEP 1: Add a Dockerfile
> 
- 여기서 만드는 action은 도커 컨테이너를 이용하므로 dockerfile이 필요하다.

<aside>
💡 **action-a/Dockerfile** 을 작성하고, 깃헙 repo에 PR을 열어보자

</aside>

```docker
FROM debian:9.5-slim

ADD entrypoint.sh /entrypoint.sh
RUN chmod +x /entrypoint.sh
ENTRYPOINT ["/entrypoint.sh"]
```

- PR 여는 방법
    - 새 branch를 생선한다. (**first-action 브랜치**)
    - action-a 디렉토리를 생성하고, Dockerfile을 작성한다.
    - 커밋을 진행하고 repo에 push한다.
    - main 브랜치에 대해 PR을 연다.
- 위에서 /entrypoint.sh 스크립트는 Docker에서 실행될 것이며, 여기에는 action이 실제로 하고자 하는게 명세될 것임.

---

> STEP 2: Add an entrypoint script
> 
- action이 Docker를 이용해 실행시킬 entrypoint.sh 파일을 실제로 repo에 만들어야 한다.

<aside>
💡 **entrypoint.sh** 파일을 작성하고 first-action 브랜치에 푸시한다.

</aside>

```docker
#!/bin/sh -l
sh -c "echo Hello world my name is $INPUT_MY_NAME"
```

- 이제 이 스크립트를 실행시키기 위해서는 docker 를 트리거할 수 있는 github-action이 필요하다.

---

> STEP 3: Add an action metadata file
> 
- input 파라미터를 MY_NAME 값을 읽기위해 이용한다
- 아래와 같은 파일을 action-a/action.yml 로 추가한다

```yaml
name: "Hello Actions"
description: "Greet someone"
author: "octocat@github.com"

inputs:
  MY_NAME:
    description: "Who to greet"
    required: true
    default: "World"

runs:
  using: "docker"
  image: "Dockerfile"

branding:
  icon: "mic"
  color: "purple"
```

---

> STEP 4: Start your workflow file
> 
- workflow 는 .github/workflows 디렉토리 아래에 main.yml 로 정의된다.
- workflow는 특정 이벤트에 기반해 실행되며, 이 실습에서는 push 이벤트에 대해 실행되도록 해 본다.

- .github/workflows/main.yml 파일 생성

```yaml
name: A workflow for my Hello World file
on: push
```

- name: A workflow for my Hello World file
    - 각 워크플로에 대해 특정한 이름을 정해줄 수 있다.
    - 이 값은 action탭이나 각 PR에서 확인할 수 있다.
    - 특정 repo안에 여러개의 workflow가 있는 경우 유용하다.
- on: push
    - 이 워크플로가 언제 실행될 것인지를 지칭
    - 이 경우는 push event가 발생하는 경우이다.
    

<aside>
💡 - workflows
    - jobs
        -steps

의 구조로 이루어져 있다.

</aside>

- 이번에는 action을 실행시키는 job을 만들어 보자.
- actions는 다음과 같은 곳에서 이용될 수 있다.
    - 같은 repo
    - 다른 public repo
    - docker container image
- 여기서는 이 repo에서 실행시켜 본다.

---

> STEP 5: Run an action from your workflow file
> 
- 위에서 만들었던 .github/workflows/main.yml 파일에 job 명세를 추가해보자

```yaml
jobs:
  build:
    name: Hello world action
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v1
      - uses: ./action-a
        with:
          MY_NAME: "Mona"
```

- jobs
    - workflow 실행의 기본 단위(component)
- build
    - 이 job에 붙이는 식별자(identifier)
    - 자유롭게 줄수 있는 이름인 듯 (?)
- name
    - 이 job의 이름. 워크플로 실행시 github에서 확인되는 이름이다
    
    ![이런식으로](/files/posts/20220113/github-action-01.png)
    
    이런식으로
    
- runs-on
    - 이 job을 실행할 머신의 종류
    - github-hosted runner일 수도 있고, self-hosted runner일 수도 있다.
- steps
    - job을 수행하기 위한 linear sequence
- uses: actions/checkout@v1
    - **checkout** 이라는 이름의 community action에게 이 repo에 접근할 수 있도록 해 준다.
    - 즉 저 공개된 action을 이용한다는 뜻으로 보임
- uses: ./action-a
    - 상대경로로 우리가 만들어둔 action-a 디렉토리의 action을 이용하겠다
    
    <aside>
    💡 이때 상대경로가 지금 이 workflow를 실행하는 main.yml의 경로로부터 시작되는게 아니라 root에서 job이 실행된다는 것으로 보인다.
    
    </aside>
    
- with
    - runtime에 이용할 입력변수를 지정한다.
    - 이 경우에는 MY_NAME 이라는 변수에 “Mona” 라는 값을 할당했다.

> **총정리**
> 
- github-action은 github repo에 발생하는 특정 event를 감지해 workflow를 실행 시키는 일련의 작업을 말한다.
- Github-Actions는 일종의 상품명 같은 느낌이고 실제적인 action으로 볼 수 있는 구체적은 항목들은 workflow라고 부른다.
- workflow의 구성은 .github/workflows/ 디렉토리에 정의할 수 있으며, 지금까지는 main.yml 파일을 통해 정의했으나 복수개의 파일로도 가능할 것 같다. (아직 모름)
- workflow는 복수개의 job 들로 이루어져 있다.
- job은 복수개의 step들로 이루어져 있다.
- 위 실습의 최종 결과는 다음과 같다

![무엇이든 push가 발생하면 workflow가 실행된다.](/files/posts/20220113/github-action-02.png)


![Hello world action 이라는 이름의 job 실행, checkout과 action-a의 step이 실행된다.](/files/posts/20220113/github-action-03.png)


![docker 컨테이너를 이용해 entrypoint.sh 파일이 실행된 모습. 실행변수로 넘긴 Mona도 잘 적용 되었다.](/files/posts/20220113/github-action-04.png)