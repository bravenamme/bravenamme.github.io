---
layout: post
title:  "GitLab CI/CD Pipeline 구성과 .gitlab-ci.yml"
date:   2020-11-09 10:00
author: juhyun10
tags:	[devops,gitlab,gitlab-runner,ci,cd]
---


이 글은 GitLab CI/CD Pipeline 과 구성파일인 .gitlab-ci.yml 에 대해 설명합니다.

>***GitLab CI/CD Pipeline 구성***<br />
>- CI/CD 파이프라인
>- 수동 배포 설정
>- 수동 job 실행 시 변수 설정
>- CI/CD 파이프라인 그룹화
>- 파이프라인 아키텍처
>   - 파이프라인 아키텍처 - Basic
>   - 파이프라인 아키텍처 - DAG (Directed Acyclic Graph)
>   - 파이프라인 아키텍처 - Child/Parent Pipelines
>- 효율적인 파이프라인 구성
>- 파이프라인 스케쥴링
>
>***.gitlab-ci.yml 에 대하여***<br />
>- .gitlab-ci.yml
>- job 구성 요소
>- 전역 파라미터
>   - stages
>   - include
>       - include:local
>   - script
>       - script: before_script, after_script
>       - script:  Multi-line commands
>   - stage
>   - extends
>   - rules
>   - needs
>   - allow_failure
>   - when

## CI/CD 파이프라인

파이프라인은 `Jobs` 과 `Stages` 로 구성된다.

>**Jobs**<br />
>수행할 작업을 정의
>예를 들면 코드를 컴파일 하거나 테스트하는 작업
>
>**Stages**<br />
>jobs 를 실행할 시기를 정의

`Jobs` 는 Runner 에 의해 실행되는데 같은 `Stage` 내의 여러 개의 `Jobs` 가 병렬로 실행된다.

하나의 `Stage` 가 성공적으로 종료되면 파이프라인은 다음 `Stage` 로 넘어가고,<br />
만약 `Stage` 안의 `Job` 이 실패하면 다음 `Stage` 는 실행되지 않고 파이프라인이 종료된다.

파이프 라인과 구성 job 과 stage 는 각 프로젝트의 CI/CD 파이프라인 구성 파일인 .`gitlab-ci.yml` 에 정의되는데
뒷 부분에 .gitlab-ci.yml 에 대해 설명한다.

---

## 수동 배포 설정

파이프라인은 자동으로 실행시킬 수 있지만 프로덕션에 배포할 때는 수동 작업이 필요하다.
`when:manual` 설정을 통해 프로덕션 단계에서는 수동으로 배포할 수 있다.

```yaml
stages:
  - test
  - build
  - deploy

test:
  stage: test
  script: echo "Running tests"

build:
  stage: build
  script: echo "Building the app"

deploy_staging:
  stage: deploy
  script:
    - echo "Deploy to staging server"
  only:
    - master

deploy_prod:
  stage: deploy
  script:
    - echo "Deploy to production server"
  when: manual
  only:
    - master
```

![프로덕션 환경에서는 수동 배포하도록 구성된 파이프라인<br />(출처 : https://docs.gitlab.com/ee/ci/environments/index.html#configuring-manual-deployments)](/files/posts/20201112/manual.jpg)


그 외에도 아래와 같은 `when:` 옵션이 있다. (default 는 on_success)

![job 실행 조건 when](/files/posts/20201112/when.jpg)

좀 더 상세한 설명은 [Configuring manual deployments](https://docs.gitlab.com/ee/ci/environments/index.html#configuring-manual-deployments) 를
참고하세요.

---

## 수동 job 실행 시 변수 설정

*사전 정의된 환경 변수는 [Predefined environment variables reference](https://docs.gitlab.com/ee/ci/variables/predefined_variables.html) 와 
[list-all-environment-variables](https://docs.gitlab.com/ee/ci/variables/README.html#list-all-environment-variables) 를 참고*

수동 job 실행 시 특정 변수를 추가하여 실행할 수 있는데 추가 변수는 실행하려는 수동 job 페이지와 .gitlab-ci.yml 에서 설정 가능하다.
고정된 값의 변수인 경우 .gitlab-ci.yml 에서 설정하여 수동이 아닌 job 에서도 사용 가능하다.

아래 내용 외 더 자세한 사항은 [.gitlab-ci.yml defined variables](https://docs.gitlab.com/ee/ci/variables/README.html#gitlab-ciyml-defined-variables) 를 참고하세요.

```yaml
stages:
  - build
  - test
  - deploy

variables:    # 전역 변수
  NAME: 'assu'

build-phase:      # 임의의 job 이름..
  stage: build
  tags:
    - ci-tag
  except:
    - dev
  before_script:
    - echo 'build-phase before script, except dev'
  script:
    - chcp 65001    # UTF-8 설정
    - echo 'build-phase script, except dev'
  after_script:
    - echo 'build-phase after_script, except dev'

test-phase:      # 임의의 job 이름
  stage: test
  tags:
    - ci-tag
  except:
    - dev
  before_script:
    - echo 'build-phase before script, except dev'
  script:
    - chcp 65001    # UTF-8 설정
    - echo 'build-phase script, except dev' $NAME
  after_script:
    - echo 'build-phase after_script, except dev'

deploy-phase:      # 임의의 job 이름
  stage: deploy
  tags:
    - ci-tag
  when: manual
  only:
    - master
  before_script:
    - echo 'deploy-phase before script, except dev'
  script:
    - chcp 65001    # UTF-8 설정
    - echo 'deploy-phase script, except dev'$NAME
  after_script:
    - echo 'deploy-phase after_script, except dev'
```

```shell
Running with gitlab-runner 13.6.0~beta.126.g34d8ad75 (34d8ad75)
  on ci-test xxxxxxxx
Preparing the "shell" executor
00:00
Using Shell executor...
Preparing environment
00:00
Running on LAPTOP-EGGD0QMM...
Getting source from Git repository
00:04
Fetching changes with git depth set to 50...
Reinitialized existing Git repository in C:/Gitlab-Runner/builds/xxxxxxxx/0/juhyun10.lee/test/.git/
Checking out 3c40ce99 as master...
git-lfs/2.11.0 (GitHub; windows amd64; go 1.14.2; git 48b28d97)
Skipping Git submodules setup
Executing "step_script" stage of the job script
00:00
$ echo 'deploy-phase before script, except dev'
deploy-phase before script, except dev
$ chcp 65001
Active code page: 65001
$ echo 'deploy-phase script, except dev'$NAME         # 변수 출력
deploy-phase script, except dev
assu            # 변수 출력
Running after_script
00:01
Running after script...
$ echo 'deploy-phase after_script, except dev'
deploy-phase after_script, except dev
Cleaning up file based variables
00:00
Job succeeded
```

수동 job 페이지와 .gitlab-ci.yml 에서 변수 동시 사용하는 경우는 아래와 같다.

```yaml
stages:
  - build
  - test
  - deploy

variables:    # 전역 변수
  NAME: 'assu'

build-phase:      # 임의의 job 이름..
  stage: build
  tags:
    - ci-tag
  except:
    - dev
  before_script:
    - echo 'build-phase before script, except dev'
  script:
    - chcp 65001    # UTF-8 설정
    - echo 'build-phase script, except dev'
  after_script:
    - echo 'build-phase after_script, except dev'

test-phase:      # 임의의 job 이름
  stage: test
  tags:
    - ci-tag
  except:
    - dev
  before_script:
    - echo 'build-phase before script, except dev'
  script:
    - chcp 65001    # UTF-8 설정
    - echo "build-phase script, except dev $NAME"   # 변수 출력 시엔 큰 따옴표로 묶어주어야 함
  after_script:
    - echo 'build-phase after_script, except dev'

deploy-phase:      # 임의의 job 이름
  stage: deploy
  tags:
    - ci-tag
  when: manual
  only:
    - master
  before_script:
    - echo 'deploy-phase before script, except dev'
  script:
    - chcp 65001    # UTF-8 설정.
    - echo "deploy-phase script, except dev $NAME $NICK1 $NICK2"    # NAME 은 전역 변수로 셋팅, 나머진 job 페이지에서 셋팅
  after_script:
    - echo 'deploy-phase after_script, except dev'
```

![수동 job 페이지 위치)](/files/posts/20201112/job.jpg)

![수동 job 페이지에서 변수 설정)](/files/posts/20201112/job2.jpg)


```shell
Running with gitlab-runner 13.6.0~beta.126.g34d8ad75 (34d8ad75)
  on ci-test xxxxxxxx
Preparing the "shell" executor
00:00
Using Shell executor...
Preparing environment
00:01
Running on LAPTOP-EGGD0QMM...
Getting source from Git repository
00:04
Fetching changes with git depth set to 50...
Reinitialized existing Git repository in C:/Gitlab-Runner/builds/xxxxxxxx/0/juhyun10.lee/test/.git/
Checking out 47bcb2a2 as master...
git-lfs/2.11.0 (GitHub; windows amd64; go 1.14.2; git 48b28d97)
Skipping Git submodules setup
Executing "step_script" stage of the job script
00:00
$ echo 'deploy-phase before script, except dev'
deploy-phase before script, except dev
$ chcp 65001
Active code page: 65001
$ echo "deploy-phase script, except dev $NAME $NICK1 $NICK2"
deploy-phase script, except dev assu TEST    # 변수 출력 (대소문자 가리지 않음)
Running after_script
00:01
Running after script...
$ echo 'deploy-phase after_script, except dev'
deploy-phase after_script, except dev
Cleaning up file based variables
00:00
Job succeeded
```

설정한 변수가 없어도 오류는 나지 않으며, 없는 변수를 매개 변수로 넘겨도 또한 오류는 발생하지 않는다.

## CI/CD 파이프라인 그룹화

비슷한 job 이 많은 경우 파이프라인 그래프가 길어져 보기 힘들어지는데 아래와 같은 방식으로 설정하면 비슷한 job 을 자동으로 그룹화할 수 있다.

![그룹화된 파이프라인<br />(출처 : https://docs.gitlab.com/ee/ci/pipelines/index.html)](/files/posts/20201112/group1.jpg)

- 슬래쉬 (/) 사용 : test 1/3, test 2/3, test 3/3
- 콜론 (:) 사용 : test 1:3, test 2:3, test 3:3
- 공백 ( ) 사용 : test 0 3, test 1 3, test 2 3

아래와 같이 구성 시 3개의 job 을 가진 *build ruby* 라는 그룹화된 파이프라인 확인이 가능하다.

```yaml
build ruby 1/3:
  stage: build
  script:
    - echo "ruby1"

build ruby 2/3:
  stage: build
  script:
    - echo "ruby2"

build ruby 3/3:
  stage: build
  script:
    - echo "ruby3"
```

![그룹화된 파이프라인<br />(출처 : https://docs.gitlab.com/ee/ci/pipelines/index.html)](/files/posts/20201112/group2.jpg)

---

## 파이프라인 아키텍처

파이프라인 아키텍쳐는 3가지로 분류할 수 있다.

- **Basic**<br />
    - 간단한 프로젝트에 적합
- **DAG (Directed Acyclic Graph)**<br />
    - 복잡한 프로젝트에 적합
- **Child/Parent Pipelines**<br />
    - 독립적으로 정의된 구성요소가 많은 프로젝트에 적합

---

### 파이프라인 아키텍처 - Basic

GitLab 에서 가장 간단한 구조의 파이프라인으로 빌드 stage 의 모든 항목을 동시에 실행하고, 
모든 작업이 완료되면 테스트 stage 의 모든 항목을 동일한 방식으로 실행한다.<br />
유지 관리는 쉽지만, 가장 효율적인 방법은 아니며 단계가 많아질수록 상당히 복잡해진다.

![파이프라인 - 기본 아키텍처](/files/posts/20201112/pipeline.jpg)

```yaml
stages:
  - build
  - test
  - deploy

build_a:
  stage: build
  tags:
    - ci-tag
  script:
    - echo "This job builds something."

build_b:
  stage: build
  tags:
    - ci-tag
  script:
    - echo "This job builds something else."

test_a:
  stage: test
  tags:
    - ci-tag
  script:
    - echo "This job tests something. It will only run when all jobs in the"
    - echo "build stage are complete."

test_b:
  stage: test
  tags:
    - ci-tag
  script:
    - echo "This job tests something else. It will only run when all jobs in the"
    - echo "build stage are complete too. It will start at about the same time as test_a."

deploy_a:
  stage: deploy
  tags:
    - ci-tag
  script:
    - echo "This job deploys something. It will only run when all jobs in the"
    - echo "test stage complete."

deploy_b:
  stage: deploy
  tags:
    - ci-tag
  script:
    - echo "This job deploys something else. It will only run when all jobs in the"
    - echo "test stage complete. It will start at about the same time as deploy_a."
```

---

### 파이프라인 아키텍처 - DAG (Directed Acyclic Graph)

DAG 는 기본 아키텍처보다 효율적이고 빠른 실행을 가능하게 해준다.<br />
`needs` 키워드를 통해 job 간의 종속성을 정의하여 사용한다.

예를 들면 아래의 예에서 build_a 와 test_a 가 build_b, test_b 보다 훨씬 빠르면 build_b 가 아직 실행 중이라도
GitLab 은 deploy_a 를 시작한다.

이는 독립적으로 배포 가능하지만 관련성 있는 마이크로서비스의 배포처럼 복잡한 배포 처리 시 특히 효율적이다.

좀 더 자세한 사항은 [Directed Acyclic Graph](https://docs.gitlab.com/ee/ci/directed_acyclic_graph/index.html) 를 참고하세요.

![파이프라인 - DAG 아키텍처](/files/posts/20201112/dag.jpg) 

```yaml
stages:
  - build
  - test
  - deploy

build_a:
  stage: build
  tags:
    - ci-tag
  script:
    - echo "This job builds something quickly."

build_b:
  stage: build
  tags:
    - ci-tag
  script:
    - echo "This job builds something else slowly."

test_a:
  stage: test
  tags:
    - ci-tag
  needs: [build_a]
  script:
    - echo "This test job will start as soon as build_a finishes."
    - echo "It will not wait for build_b, or other jobs in the build stage, to finish."

test_b:
  stage: test
  tags:
    - ci-tag
  needs: [build_b]
  script:
    - echo "This test job will start as soon as build_b finishes."
    - echo "It will not wait for other jobs in the build stage to finish."

deploy_a:
  stage: deploy
  tags:
    - ci-tag
  needs: [test_a]
  script:
    - echo "Since build_a and test_a run quickly, this deploy job can run much earlier."
    - echo "It does not need to wait for build_b or test_b."

deploy_b:
  stage: deploy
  tags:
    - ci-tag
  needs: [test_b]
  script:
    - echo "Since build_b and test_b run slowly, this deploy job will run much later."
```

---

### 파이프라인 아키텍처 - Child/Parent Pipelines

구성을 여러 파일로 분리 후 `trigger` 키워드를 통해 파이프라인을 구성하는 방법으로 구성 파일을 간단하게 유지 관리할 수 있다.

- **rules**<br />
    - 예를 들어 해당 영역이 변경된 경우만 하위 파이프라인을 트리거함
- **include**<br /> 
    - 외부의 yaml 파일을 포함시킴으로써 CI/CD 구성을 여러 파일로 나누어 긴 구성 파일의 가독성을 높임
    - 모든 프로젝트에 대한 전역 변수와 같은 중복 구성을 방지할 수 있음

좀 더 자세한 내용은 [rules](https://docs.gitlab.com/ee/ci/yaml/README.html#rules) 와
[include](https://docs.gitlab.com/ee/ci/yaml/README.html#include) 를 참고하세요.


![파이프라인 - Child/Parent Pipelines 아키텍처 (a 디렉토리 안의 내용이 변경된 경우)](/files/posts/20201112/trigger.jpg)

.gitlab-ci.yml 의 내용은 아래와 같다.
a 디렉토리 안의 내용이 변경되면 a 디렉토리에 위한 gitlab-ci.yml 을 실행한다. 

```yaml
stages:
  - triggers

trigger_a:
  stage: triggers
  trigger:
    include: a/.gitlab-ci.yml
  rules:
    - changes:
        - a/*

trigger_b:
  stage: triggers
  trigger:
    include: b/.gitlab-ci.yml
  rules:
    - changes:
        - b/*
```

/a/.gitlab-ci.yml 에 위치한 구성 파일이고, DAG 기법을 사용하기 위해 `needs` 키워드를 사용하였다.

```yaml
stages:
  - build
  - test
  - deploy

build_a:
  stage: build
  tags:
    - ci-tag
  script:
    - echo "This job builds something."

test_a:
  stage: test
  tags:
    - ci-tag
  needs: [build_a]
  script:
    - echo "This job tests something."

deploy_a:
  stage: deploy
  tags:
    - ci-tag
  needs: [test_a]
  script:
    - echo "This job deploys something."
```

/b/.gitlab-ci.yml 에 위치한 구성 파일이고, DAG 기법을 사용하기 위해 `needs` 키워드를 사용하였다.

```yaml
stages:
  - build
  - test
  - deploy

build_b:
  stage: build
  tags:
    - ci-tag
  script:
    - echo "This job builds something else."

test_b:
  stage: test
  tags:
    - ci-tag
  needs: [build_b]
  script:
    - echo "This job tests something else."

deploy_b:
  stage: deploy
  tags:
    - ci-tag
  needs: [test_b]
  script:
    - echo "This job deploys something else."
```

---

## 효율적인 파이프라인 구성

파이프라인을 효율적으로 구성하면 개발자의 시간을 절약할 수 있다.

- DevOps 프로세스 속도 향상
- 비용 절감
- 개발 피드백 루프 단축

비효율적인 파이프라인을 확인하는 가장 쉬운 지표는 job, stage 의 런타임 시간과 파이프라인 자체의 총 런타임 시간이다.<br />
파이프라인 총 런타임 시간은 아래 내용에 의해 크게 영향을 받는다.

- job 과 stage 의 총 갯수
- job 간의 종속성

여러 항목을 병렬로 실행하는 job 을 동일한 stage 에서 실행하여 전체 런타임을 줄일 수 있다. 단, 병렬 작업을 지원하기 위해
더 많은 Runner 가 동시에 실행이 되어야 한다.

[GitLab CI Pipelines Exporter](https://github.com/mvisonneau/gitlab-ci-pipelines-exporter) 을 사용하면 파이프라인의 상태 및
수행 시간을 확인할 수 있다.

또한 호스트 시스템의 [monitor CI runners](https://docs.gitlab.com/runner/monitoring/) 를 이용하거나 쿠버네티스를 이용하는 방법도 있다. 

---

## 파이프라인 스케쥴링

파이프라인은 일반적으로 push 가 발생하는 등 특정 조건이 충족할 경우 실행되는데 일정을 사용하여 특정 간격으로 파이프라인을 실행할 수도 있다.<br />
예를 들면 특정 브랜치에 대해서 매월 1일에 실행한다던가 매일 한번 실행 등의 조건으로 실행 가능하다.

GitLab UI 로 스케쥴링하는 것 외에도 [Pipeline schedules API](https://docs.gitlab.com/ee/api/pipeline_schedules.html) 를 사용하여
파이프라인 일정을 관리할 수 있다.

스케쥴링 파이프라인을 구성하기 위해선 스케쥴링 소유자는 대상 브랜치로의 merge 권한이 있어야 한다.

스케쥴링 구성은 GitLab 사이트의 CI/CD > Schedules 에서 설정할 수 있다.

***주의! CI 에 대한 스케쥴링이지 특정 비즈니스 API 를 주기적으로 호출하는 성격의 스케쥴링이 아니다.*** 

---

## .gitlab-ci.yml

GitLab CI/CD 파이프라인은 `.gitlab-ci.yml` 이라는 파일로 구성이 된다.<br />
이 `.gitlab-ci.yml`(이하 구성 파일) 파일은 아래 내용을 정의한다.

- 파이프라인의 구조와 순서를 정의
- GitLab Runner 를 사용하여 실행할 항목
- 특정 상황이 발생했을 때 실행할 항목. (예를 들면 프로세스가 성공 혹은 실패했을 때 어떠한 액션을 취할지)

좀 더 많은 예제는 [GitLab CI/CD Examples](https://docs.gitlab.com/ee/ci/examples/README.html) 를 참고하세요.<br />
복잡한 구조의 구성은 [여기](https://gitlab.com/gitlab-org/gitlab/blob/master/.gitlab-ci.yml) 를 참고하세요.

파이프라인 구성은 `job` 으로 시작하는데 이 `job` 은 구성 파일의 가장 기본적인 요소이다.

- **job**<br />
    - 실행되어야 하는 상황(특정 조건)을 제약 조건으로 정의
    - 최상위 요소이며, 반드시 script 를 포함해야 함
    - 정의할 수 있는 수의 제한 없음

```shell
job1:
  script: "execute-script-for-job1"

job2:
  script: "execute-script-for-job2"
```

위는 각 job 이 서로 다른 명령을 실행하는 두 개의 개별 job 이다.
이 job 은 Runner 에 의해 선택되어 Runner 안에서 실행된다. 

---

## 전역 파라미터

모든 job 에 공툥으로 사용되는 전역 파라미터는 `default` 키워드를 사용하여 설정 가능하다.

`default` 키워드 내 사용 가능한 job 파라미터는 아래와 같다.

- image
- services
- before_script
- after_script
- tags
- cache
- artifacts
- retry
- timeout
- interruptible

예를 들면 아래에서 ruby:2.5 는 rspec 2.6 job 을 제외한 모든 job 에 적용된다. 

```shell
default:
  image: ruby:2.5

rspec:
  script: bundle exec rspec

rspec 2.6:
  image: ruby:2.6
  script: bundle exec rspec
```

---

### stages

job 이 실행되는 단계를 의미한다.

동일한 stage 안에 있는 job 은 병렬적으로 실행되며, 다음 stage 의 job 은 이전 stage 가 성공적으로 완료된 후 실행된다.

```shell
stages:
  - build
  - test
  - deploy
```

build stage 의 모든 job 이 성공적으로 완료되면 test stage 의 job 이 실행되고, deploy stage 또한 마찬가지로 동작한다.

.gitlab-ci.yml 에 정의된 stage 가 없으면 기본적으로 build, test, deploy 가 사용된다.

---

### include

`include` 키워드를 사용하여 외부 yaml 파일을 포함할 수 있다.
CI/CD 구성을 여러 파일로 나누어 긴 구성 파일의 가독성을 높이고, 전역 기본 변수와 같은 중복 구성을 방지할 수 있다.

`include` 는 외부 yaml 파일의 확장자가 반드시 .yml 혹은 .yaml 이어야 한다.

- local
    - 로컬 프로젝트 repository 안의 파일을 포함
- file
    - 다른 프로젝트 repository 안의 파일을 포함
- remote
    - 원격 URL 의 파일을 포함 / 공개적으로 access 가능해야 함
- template 
    - GitLab 에서 제공하는 템플릿 포함

---

#### include:local
 
```shell
include:
  - local: '/templates/.gitlab-ci-template.yml'

include: '.gitlab-ci-production.yml'
```

그 외는 [include:file](https://docs.gitlab.com/ee/ci/yaml/README.html#includefile),
[include:remote](https://docs.gitlab.com/ee/ci/yaml/README.html#includeremote),
[include:template](https://docs.gitlab.com/ee/ci/yaml/README.html#includetemplate) 를 참고하세요.

include 의 자세한 예제는 [GitLab CI/CD include examples](https://docs.gitlab.com/ee/ci/yaml/includes.html) 에 있습니다.

---

### script

스크립트에 아래 특수 문자가 오는 경우는 따옴표로 묶어서 처리해야 한다.

```shell
:, {, }, [, ], ,, &, *, #, ?, |, -, <, >, =, !, %, @, `
```

스크립트 명령이 0 이 아닌 exit code 를 반환하는 경우 job 이 실패하는데 이럴 경우엔 변수에 exit code 를 저장하여 처리한다.

```shell
job:
  script:
    - false || exit_code=$?
    - if [ $exit_code -ne 0 ]; then echo "Previous command failed"; fi;
```


#### script - before_script, after_script

`before_script` 는 deploy job 을 포함하여 각 job 이 실행되기 전에 수행될 command 의 집합이고, 
`after_script` 는 실패한 job 을 포함하여 각 job 실행 후 수행될 command 의 집합이다.

또한 before/after script 모두 default 셋팅이 되어있어도 각 job 에서 오버라이드 가능하다.

```shell
default:
  before_script:
    - global before script

job:
  before_script:
    - execute this instead of global before script
  script:
    - my command
  after_script:
    - execute this after my script
```

좀 더 자세한 내용은 [YAML anchors for before_script and after_script](https://docs.gitlab.com/ee/ci/yaml/README.html#yaml-anchors-for-before_script-and-after_script)
을 참고하세요.

---

#### script - Multi-line commands

가독성을 높이기 위해 `|` 과 `>` 을 사용하여 긴 command 를 여러 줄로 분할하여 사용 가능하다.

*여러 명령이 하나의 command 로 구성된 경우 마지막 command 의 성공 여부만 리턴 (이전 command 의 실패는 버그로 인해 무시됨) 되므로,
각 명령을 별도 스크립트로 작성할 것*

```shell
build_a:
  stage: build
  tags:
    - ci-tag
  before_script:        # 둘 다 동일
    - |
      echo "First command line
      is split over two lines."
      echo "Second command line."
    - |
      echo "1First command line
      is split over two lines."

      echo "1Second command line."
```

위의 결과는 아래와 같다.
```shell
$ echo "First command line # collapsed multi-line command
First command line
is split over two lines.
Second command line.

$ echo "1First command line # collapsed multi-line command
1First command line
is split over two lines.
1Second command line.
```

```shell
build_a:
  stage: build
  tags:
    - ci-tag
  before_script:        # 결과 상이
    - >
      echo "First command line
      is split over two lines."
      echo "Second command line."   # echo 가 하나의 문자열로 출력
    - >
      echo "2First command line
      is split over two lines."

      echo "2Second command line."
```

위의 결과는 아래와 같다.
```shell
$ echo "First command line is split over two lines." echo "Second command line."
First command line is split over two lines.
echo
Second command line.

$ echo "2First command line is split over two lines." # collapsed multi-line command
2First command line is split over two lines.
2Second command line.
```

```shell
build_a:
  stage: build
  tags:
    - ci-tag
  before_script:        # 결과 상이
    - echo "First command line
      is split over two lines."
      echo "Second command line."       # echo 가 하나의 문자열로 출력

    - echo "3First command line
      is split over two lines."

      echo "3Second command line."
```

위의 결과는 아래와 같다.
```shell
$ echo "First command line is split over two lines." echo "Second command line."
First command line is split over two lines.
echo
Second command line.

$ echo "3First command line is split over two lines." # collapsed multi-line command
3First command line is split over two lines.
3Second command line.
```

---

### stage

Runner 는 기본적으로 한 번에 하나의 job 만 처리한다.

![하나의 stage 안의 여러 job 은 한 번에 하나씩 실행](/files/posts/20201112/parallel.jpg)

job 은 아래와 같은 상황에서 병렬적으로 실행된다.

- 다른 Runner 에서 실행
- Runner 의 `concurrent` 셋팅
 
Runner 의 `concurrent` 셋팅은 [runner global settings](https://docs.gitlab.com/runner/configuration/advanced-configuration.html#the-global-section) 를 참고하세요.


`.pre` 는 항상 파이프라인의 첫 번째로 실행되고, `.post` 는 항상 파이프라인의 마지막으로 실행된다.<br />
(.gitlab-ci.yml 에서 정의한 순서에 상관없음)

예를 들어 아래는 모두 동일한 순서로 실행된다.

```shell
stages:
  - .pre
  - a
  - b
  - .post


stages:
  - a
  - .pre
  - b
  - .post
```

### extends

템플릿 job 을 설정한 후 `extends` 키워드를 통해 상속받을 수 있다.<br />
이 방법은 [YAML anchors](https://docs.gitlab.com/ee/ci/yaml/README.html#anchors) 를 대체할 수 있을 뿐 아니라 더 유연하고 가독성이 좋다.

```shell
.build:
  stage: build
  tags:
    - ci-tag
  script: echo "job parent"

job_child_1:
  extends:
    - .build
  script: echo "job child 1"    # 오버라이드 가능

job_child_2:
  extends:
    - .build
```

다단계 상속도 가능하지만 최대 3 depth 이상의 상속은 피하는 것이 좋다.

`include` 와 `extends` 를 함께 사용하면 좀 더 유지 관리하기 편한 구조로 구성 파일을 관리할 수 있다.

```shellscript
# event-ci-template.yml

.event-template:
  tags:
    - ci-tag
  script:
    - echo event


# .gitlab-ci.yml
include: event-ci-template.yml

job1:
  extends: .event-template
  stage: build
```

---

### rules

***rules 는 only/except 보다 더 유연하고 강력한 솔루션이다. 파이프라인을 최대한 활용하려면 only/except 대신 rules 를 사용하는 것을 권장한다.***

`rules` 키워드는 파이프라인에 job 을 포함시키거나 제외시킬 때 사용할 수 있다.

단, `only/except` 키워드의 함께 사용할 수 없다. 동일한 job 에 두 키워드를 함께 사용 시 `key may not be used with rules` 오류가 발생한다.

`rules` 에서 허용하는 job 속성은 아래와 같다.

- `when`
    - 정의하지 않으면 기본값은 `when: on_success`
    - `when: delayed` 사용시엔 `start_in` 도 정의해야 함
- `allow_failure`
    - 정의하지 않으면 기본값은 `allow_failure: false`
    
```shell
job1:
  tags:
    - ci-tag
  stage: build
  script:
    - echo event
  rules:
    - if: '$CI_COMMIT_BRANCH == "master"'
      when: delayed
      start_in: '30 seconds'
      allow_failure: false
    - if: '$CI_COMMIT_BRANCH == "master22"'
      when: delayed
      start_in: '50 minutes'
      allow_failure: true
```    

```shell
job:
  script: "echo Hello, Rules!"
  rules:
    - if: ($CI_COMMIT_BRANCH == "master" || $CI_COMMIT_BRANCH == "develop") && $MY_VARIABLE
      when: manual
      allow_failure: true
```

위 예에서 조건에 맞는 경우 job 은 수동으로 실행시켜 주어야 하고, 성공 여부는 true 로 설정된다.

- [`if`](https://docs.gitlab.com/ee/ci/yaml/README.html#rulesif)
    - 조건에 따라 job 을 추가/제외할 때 사용
- [`change`](https://docs.gitlab.com/ee/ci/yaml/README.html#ruleschanges)
    - 변경된 파일에 따라 job 을 추가/제외할 때 사용
    - `only: changes` / `except: changes` 와 동일하게 동작
- [`exist`](https://docs.gitlab.com/ee/ci/yaml/README.html#rulesexists)
    - 특정 파일 존재 여부에 따라 job 을 추가/제외할 때 사용 


```shell
job:
  script: "echo Hello, Rules!"
  rules:
    - if: '$CI_PIPELINE_SOURCE == "merge_request_event"'
      when: never
    - if: '$CI_PIPELINE_SOURCE == "schedule"'
      when: never
    - when: on_success
```

위에서 merge request 인 경우 job 은 파이프라인에 추가되지 않는다.<br />
스케쥴링 파이프라인인 경우 job 은 파이프라인에 추가되지 않는다.<br />
그 외 경우의 job 은 파이프라인에 추가된다.

`rules:if` 에 `$CI_PIPELINE_SOURCE` 가 많이 사용되는데,
`$CI_PIPELINE_SOURCE` 의 값으로 사용되는 항목은 
`api`, `chat`, `external`, `external_pull_request_event`, `merge_request_event`,
`parent_pipeline`, `pipeline`, `push`, `schedule`, `trigger`, `web`, `webide` 가 있다.

각 항목의 상세 내용은 [Common if clauses for rules](https://docs.gitlab.com/ee/ci/yaml/README.html#common-if-clauses-for-rules) 를 참고해주세요.


---

### needs

`needs` 키워드는 [DAG](https://docs.gitlab.com/ee/ci/directed_acyclic_graph/index.html) 를 가능하게 해줌으로써 다른 작업을 기다리지 않고
여러 단계를 동시에 실행할 수 있도록 해준다.

```shell
linux:build:
  tags:
    - ci-tag
  stage: build
  script:
    - echo "linux:build"
mac:build:
  tags:
    - ci-tag
  stage: build
  script:
    - echo "mac:build"
lint:
  tags:
    - ci-tag
  stage: test
  needs: []
  script:
    - echo "lint"
linux:rspec:
  tags:
    - ci-tag
  stage: test
  needs: ["linux:build"]
  script:
    - echo "linux:rspec"
linux:rubocop:
  tags:
    - ci-tag
  stage: test
  needs: ["linux:build"]
  script:
    - echo "linux:rubocop"
mac:rspec:
  tags:
    - ci-tag
  stage: test
  needs: ["mac:build"]
  script:
    - echo "mac:rspec"
mac:rubocop:
  tags:
    - ci-tag
  stage: test
  needs: ["mac:build"]
  script:
    - echo "mac:rubocop"
production:
  tags:
    - ci-tag
  stage: deploy
  script:
    - echo "production"
``` 

위에서 lint 는 build 단계가 끝날때까지 기다리지 않고 즉시 실행되고, 나머진 needs 안의 작업이 끝난 후 실행된다.

---

### allow_failure 

나머지 작업에 영향을 주지 않고 job 이 실패하도록 할 때 사용한다.<br />
`rules` 를 사용하지 않을 때 `when:manual` 을 사용하는 수동 작업을 제외하고 기본값은 false 이다.<br />

예를 들어 아래에서 job1 과 job2 는 병렬로 실행되지만 job1 이 실패하더라고 `allow_failure:true` 이므로 다음 stage 가 실행된다.

```shell
job1:
  stage: test
  script:
    - execute_script_that_will_fail
  allow_failure: true

job2:
  stage: test
  script:
    - execute_script_that_will_succeed

job3:
  stage: deploy
  script:
    - deploy_to_staging
```

---

### when

- `on_success`
    - 이전 stage 의 모든 job 이 성공한 경우에만 job 을 실행
- `on_failure`
    - 이전 stage 의 job 이 하나 이상 실패한 경우에만 job 을 실행
- `always`
    - 이전 stage 의 job 상태에 상관없이 job 을 실행
- `manual`
    - job 을 수동으로 실행
- `delayed`
    - 특정 시간 후에 job 을 실행
- `never`
    - `rules` 와 함께 사용 시 job 을 실행하지 않음
    - `workflow:rules` 와 함께 사용 시 파이프라인을 실행하지 않음

```shell
stages:
  - build
  - cleanup_build
  - test
  - deploy
  - cleanup

build_job:
  stage: build
  script:
    - make build

cleanup_build_job:
  stage: cleanup_build
  script:
    - cleanup build when failed
  when: on_failure

test_job:
  stage: test
  script:
    - make test

deploy_job:
  stage: deploy
  script:
    - make deploy
  when: manual

cleanup_job:
  stage: cleanup
  script:
    - cleanup after jobs
  when: always
```

- *build_job* 이 실패한 경우에만 *cleanup_build_job* 실행
- job 의 성공 여부와 상관없이 항상 파이프라인의 마지막 단계로 *cleanup_job* 실행
- GitLab UI 에서 수동으로 실행할 때 *deploy_job* 실행

`when:manual` 은 프로덕션 환경에서의 배포 시 사용하는데 자세한 내요은 [Configuring manual deployments](https://docs.gitlab.com/ee/ci/environments/index.html#configuring-manual-deployments)
을 참고하세요.


---

### trigger

자식 파이프라인 생성은 하위 파이프라인 ci 구성이 포함된 yaml 파일의 경로를 지정하면 된다.<br />
단일 저장소이면서 특정 파일이 변경될 때만 파이프라인을 트리거하는 경우 `only: change` 가 유용하다.

좀 더 자세한 사항은 [Parent-child pipelines](https://docs.gitlab.com/ee/ci/parent_child_pipelines.html) 을 참고하세요.


---

## 참고 사이트 & 함께 보면 좋은 사이트
* [GitLab CI/CD](https://docs.gitlab.com/ee/ci/README.html)
* [CI/CD pipelines](https://docs.gitlab.com/ee/ci/pipelines/index.html)
* [GitLab CI/CD Pipeline Reference](https://docs.gitlab.com/ee/ci/yaml/README.html)
* [Configuring manual deployments(수동 배포)](https://docs.gitlab.com/ee/ci/environments/index.html#configuring-manual-deployments)
* [Pipeline Architecture](https://docs.gitlab.com/ee/ci/pipelines/pipeline_architectures.html)
* [Directed Acyclic Graph](https://docs.gitlab.com/ee/ci/directed_acyclic_graph/index.html)
* [rules](https://docs.gitlab.com/ee/ci/yaml/README.html#rules)
* [include](https://docs.gitlab.com/ee/ci/yaml/README.html#include)
* [GitLab CI Pipelines Exporter](https://github.com/mvisonneau/gitlab-ci-pipelines-exporter)
* [monitor CI runners](https://docs.gitlab.com/runner/monitoring/)
* [Pipeline schedules](https://docs.gitlab.com/ee/ci/pipelines/schedules.html)
* [Maximum artifacts size](https://docs.gitlab.com/ee/user/admin_area/settings/continuous_integration.html#maximum-artifacts-size)
* [Jobs artifacts administration](https://docs.gitlab.com/ee/administration/job_artifacts.html)
* [Triggering pipelines through the API](https://docs.gitlab.com/ee/ci/triggers/README.html)
* [CI/CD Settings](https://docs.gitlab.com/ee/ci/pipelines/settings.html)
* [GitLab CI/CD pipeline configuration reference](https://docs.gitlab.com/ee/ci/yaml/README.html)
* [Configuration parameters](https://docs.gitlab.com/ee/ci/yaml/README.html#configuration-parameters)
* [스크립트 컬러링](https://docs.gitlab.com/ee/ci/yaml/README.html#coloring-script-output)
* [yaml-multiline](https://yaml-multiline.info/)
* [사전 정의된 환경 변수](https://docs.gitlab.com/ee/ci/variables/predefined_variables.html)
* [$CI_PIPELINE_SOURCE value 종류](https://docs.gitlab.com/ee/ci/yaml/README.html#common-if-clauses-for-rules)