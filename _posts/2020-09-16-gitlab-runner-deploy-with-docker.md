---
layout: post
title:  "gitlab-runnner 를 활용한 docker 배포"
date:   2020-09-16 06:00
author: q_lazzarus
tags:	[git, gitlab, gitlab-runner, docker, deploy, ci, cd]
---

## 서론
jekyll 블로그를 사용하면서 github action 에 대해서 많이 부러움을 느꼈습니다.

![](/files/posts/20200916/2434BC4257A04CAC1D.jpg)

여기서는 gitlab 과 gitlab-runner (+ nexus) 를 설치하여 나도 자동적으로 빌드와 배포가 되도록 CI (Continuous Integration) / CD (Continuous Deployment) 를 구성 해보았습니다.

## Basic Concept

![발로 그린 diagram](/files/posts/20200916/diagram.png)

기본 컨셉은 다음과 같습니다! 코드를 보내면 자동으로 도커 이미지를 생성하고 배포 대상 서버에서 컨테이너를 실행!

1. gitlab 으로 코드를 커밋
2. build
  - CI/CD (gitlab runner) 가 실행됨
  - 코드를 도커 이미지 로 빌드하고 사설 도커 저장소 (이하 *nexus*) 로 배포 진행
3. deploy
  - 대상 원격지 서버로의 SSH 키 등록
  - 대상 원격지 서버에 SSH 명령어 전달 (*nexus* 로 특정 리비전을 가져와서 컨테이너를 실행)

## POC (Proof-Of-Concept)

### 먼저 설치할 것들

물론 구성을 위해서 아래의 솔루션들이 필요합니다. 이 부분 설치는 별도의 내용을 참조하세요!  
기본적으로 손쉬운 서버 설정을 위해서 빌드서버 / 배포서버 모두 *docker engine* 이 설치되어 있다고 가정합니다.

- gitlab 설치
- sonatype nexus3 설치

### sonatype nexus3 설정
아래 문서를 참고 하여, 설치된 sonaytype nexus3 에서 docker 저장소를 설정합니다. 

[사내 Docker Registry 만들기 (Nexus3 기반)](https://velog.io/@king/private-docker-registry)

> nexus docker 로그인 연결을 그냥 admin 으로 진행 하는게 낫습니다.
> 
> 위 블로그의 내용대로 nexus 설정에 다음과 세팅이 필요합니다.
> - realm 에 docker bearer token realm 추가 필요
> - docker-hosted 에 api v1 + 포트 추가 (8082)

### gitlab-runner-network 추가
빌드서버에서 gitlab 만 네트워크를 분리하기 위하여 아래 명령어를 통해 네트워크를 추가합니다.  
각 컨테이너간 통신은 hostname 으로 호출하게 됩니다.

```bash
$ sudo docker network create gitlab-runner-net
62322b083c76808fd3502b7388c36e7dca5087059bf688434a7dc7573dba103b
```

### gitlab-runner 설정값 추가
gitlab runner 기본 설정값을 추가 합니다.

```bash
$ sudo mkdir -p /etc/srv/gitlab-runner
$ sudo touch /etc/srv/gitlab-runner/config.toml
```

### gitlab-runner / gitlab-dind 컨테이너 추가
gitlab runner 연결을 위해서 두가지 컨테이너를 추가합니다.

다음과 같은 프로세스로 runner 가 실행됩니다.

1. gitlab-runner 는 runner 가 실행시 gitlab-dind 컨테이너를 실행시키는 주체 입니다.
2. 실행된 gitlab-dind 컨테이너는 도커 이미지를 빌드를 합니다.

![docker-in-docker !?](https://i2.wp.com/www.docker.com/blog/wp-content/uploads/2013/08/docker-meme.jpg?fit=499%2C323&ssl=1)

```bash
$ sudo docker run -d \
--name gitlab-runner \
--restart always \
--network gitlab-runner-net \
-v /srv/gitlab-runner/config.toml:/etc/gitlab-runner/config.toml \
-e DOCKER_HOST=tcp://gitlab-dind:2375 \
gitlab/gitlab-runner:alpine
```
> 위에서 만들어둔 외부의 config.toml 파일을 매핑시켜 줍니다.  
> 추후 수정을 용이하게 하기 위함 입니다.

```bash
$ sudo docker run -d \
--name gitlab-dind \
--privileged \
--restart always \
--network gitlab-runner-net \
-v /var/lib/docker \
docker:18.09.9-dind
```

### gitlab-dind 컨테이너 수정
추가된 nexus 는 https 가 지원되지 않아 insecure registry 로 등록이 필요합니다.
*gitlab-dind 컨테이너*에 콘솔로 접속하여 수정을 진행합니다.

```bash
$ echo "{ \"insecure-registries\" : [\"nexus..somewhere.com:8082\"] }" > /etc/docker/daemon.json
```

### gitlab-runner 등록
이제 gitlab runner 를 gitlab 에 연동 되도록 등록해야 합니다.  

설정의 Runners 항목을 참조하여 등록 토큰 등을 설정합니다

![](/files/posts/20200916/gitlab_runner_token.png)

```bash
$ sudo docker run -it --rm \
 -v /srv/gitlab-runner/config.toml:/etc/gitlab-runner/config.toml \
 gitlab/gitlab-runner:alpine \
 register \
 --executor docker \
 --docker-image docker:18.09.9 \
 --docker-volumes /var/run/docker.sock:/var/run/docker.sock
[sudo] password for someone:
Runtime platform arch=amd64 os=linux pid=6 revision=a998cacd version=13.2.2
Running in system-mode.
Please enter the gitlab-ci coordinator URL (e.g. https://gitlab.com/):
http://somewhere.com:8000/
Please enter the gitlab-ci token for this runner:
[gitlab-ci-token]
Please enter the gitlab-ci description for this runner:
[d6cfe8946db3]: Docker Runner
Please enter the gitlab-ci tags for this runner (comma separated):
Registering runner... succeeded runner=Jxyieb2E
Please enter the executor: shell, ssh, virtualbox, docker+machine, custom, docker-ssh, parallels, dockerssh+machine, kubernetes, docker:
[docker]:
Please enter the default Docker image (e.g. ruby:2.6):
[docker:18.09.9]:
Runner registered successfully. Feel free to start it, but if it's running already the config should be
automatically reloaded!
```

정상적으로 추가되었다면 ci 설정에서 해당 runner 가 노출되게 됩니다.

### gitlab ci variables 등록

#### ssh 키 생성 (클라이언트)
ssh 키는 server 에 id / password 입력 없이 접속하기 위해서 생성합니다.
여기에서는 ssh private key 만 gitlab 에 저장할 예정이며, public key 는 서버로 전송할 예정입니다.

ssh-keygen 명령어로 ssh 키를 생성합니다.
```bash
$ ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa):
Created directory '/root/.ssh'.
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /root/.ssh/id_rsa.
Your public key has been saved in /root/.ssh/id_rsa.pub.
The key fingerprint is:
d0:82:24:8e:d7:f1:bb:9b:33:53:96:93:49:da:9b:e3 root@localhost
The key's randomart image is:
+---[RSA 2048]----+
|.*++ o.o. |
|.+B + oo. |
| +++ *+. |
| .o.Oo.+E |
| ++B.S. |
| o * =. |
| + = o |
| + = = . |
| + o o |
+----[SHA256]-----+
```

ssh-copy-id 명령어로 ssh 키를 대상 서버로 복사합니다.
```bash
$ ssh-copy-id someone@somewhere.com
```

ssh 연결을 확인해보면 key 없이 잘 접속 되는 것을 확인됩니다.
```bash
$ ssh someone@somewhere.com
```

private key 는 gitlab ci 용 변수로 담기 위해서 복사한 후 .ssh 폴더 자체를 삭제합니다. 
```bash
$ cat ~/.ssh/id_rsa
-----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAABlwAAAAdzc2gtcn
...
u+Hr4KoWBWAJVdAAAAEXJvb3RANmMwNzdhYzZkZDVjAQ==
-----END OPENSSH PRIVATE KEY-----
$ rm -rf ~/.ssh
```

#### sudo 등록 (서버)

보안상의 이유로 *배포할 서버*를 접근시에는 root 가 아니라 일반 사용자로 등록했습니다.  
다만 이럴 경우 일반 사용자가 docker 를 실행시킬 권한이 제한됩니다. (root 로 구동됨)

따라서 docker 명령어는 sudo 로 실행해야 하며, 이럴 경우 비밀번호 묻는 프롬프트가 나와서 ssh 명령어를 전달하는 입장에서는 제한이 됩니다.  
그러하므로 visudo 로 비밀번호를 묻지 않도록 제한할 예정입니다.

root 로 visudo 를 실행하여 docker 명령어만 패스워드 묻지 않도록 허용 합니다.
```
## Allow root to run any commands anywhere
root ALL=(ALL) ALL
someone ALL= NOPASSWD: /bin/docker
```

#### docker insecure registries 등록 (서버)
사설 docker registry 를 생성하였기 때문에 gitlab 과 배포할 서버 모두 insecure-registries 로 등록이 필요합니다.  
gitlab-dind 는 이미 등록이 완료되어서 여기에서는 배포할 서버에만 등록해줍니다.

/etc/docker/daemon.json 파일을 수정해서 아래의 경로를 추가해줍니다.

```json
{
 "insecure-registries" : ["nexus.somewhere.com:8082"]
}
```

#### 등록 (gitlab)
곧 작성할 gitlab-ci.yml 파일에 쓰일 변수들이 있습니다.  
yaml 파일에서 바로 정의될 변수도 있지만, 외부에서 쓰일 변수들 설정이 필요하여 gitlab ci 설정을 들어가도록 하겠습니다

설정 → CI / CD → Variables 순서대로 접근합니다.

![](/files/posts/20200916/ci_variables.png)

아래는 등록된 모습이며 각각 다음의 내용을 기술합니다.

| 타입 | 키 | 설명 | 예제 |
|------|---|------|------|
| Variables | DOCKER_PRIVATE_HOST | 도커 컨테이너 호스트입니다. nexus3 주소를 입력합니다. | nexus.somewhere.com:8082 |
| Variables | DOCKER_PRIVATE_PASSWORD | nexus 비밀번호 입니다. | nexus_password |
| Variables | DOCKER_PRIVATE_USER | nexus 아이디 입니다. | admin |
| Variables | PROD_SERVER_IP | 배포할 원격 호스트입니다. | deploy.com |
| Variables | PROD_SERVER_PORT | 접근할 원격 호스트 SSH 포트입니다. | 22 |
| Variables | PROD_SERVER_USER | 접근할 원격 호스트 유저입니다. | deployer |
| Variables | SSH_PRIVATE_KEY | 생성된 private key 입니다. | -----BEGIN OPENSSH ... 후략 |

### gitlab-ci.yml 파일 등록

gitlab ci 관련해서는 [이 문서](https://velog.io/@wickedev/Gitlab-CICD-%ED%8A%9C%ED%86%A0%EB%A6%AC%EC%96%BC-bljzphditt) 를 보시고,  
우선은 commit & push 시 발동하는 파이프라인을 작성하겠습니다.

해당 CI 파일은 두가지 스테이지 (build / deploy) 로 나누어져 있으며 각각 하는 역할은 다음과 같습니다.

- build - 도커 이미지 생성
- deploy - 원격지 서버로 배포

```yaml
image: monoless/ansible-docker:18.09.9-03
services:
 - name: monoless/ansible-docker:18.09.9-dind-03
 command: ["--insecure-registry=nexus.somwwhere.com:8082"]
stages:
 - build
 - deploy
variables:
 IMAGE_NAME: $DOCKER_PRIVATE_HOST/hello-world:$CI_PIPELINE_ID

before_script:
 - docker info

cache:
 paths:
 - node_modules/
build:
 stage: build
 script:
 - ls -al
 - docker container ls -a
 - docker build . -t $IMAGE_NAME
 - docker login -u $DOCKER_PRIVATE_USER -p $DOCKER_PRIVATE_PASSWORD $DOCKER_PRIVATE_HOST
 - docker push $IMAGE_NAME
 - docker images | grep 'somewhere.com'

deploy-image:
 stage: deploy
 before_script:
 - 'which ssh-agent || (apk update && apk add openssh-client)'
 - eval $(ssh-agent -s)
 - echo "$SSH_PRIVATE_KEY" | tr -d '\r' | ssh-add - > /dev/null
 - mkdir -p ~/.ssh
 - chmod 700 ~/.ssh
 - ssh-keyscan -p $PROD_SERVER_PORT $PROD_SERVER_IP >> ~/.ssh/known_hosts
 - chmod 644 ~/.ssh/known_hosts
 script:
 - ssh $PROD_SERVER_USER@$PROD_SERVER_IP -p$PROD_SERVER_PORT "sudo docker login \"${DOCKER_PRIVATE_HOST}\" -
u \"${DOCKER_PRIVATE_USER}\" -p \"${DOCKER_PRIVATE_PASSWORD}\" && sudo docker pull $IMAGE_NAME && sudo docker
stop \$(sudo docker ps -aq) && sudo docker rm \$(sudo docker ps -aq) && sudo docker run -d --restart=unlessstopped -p 5000:5000 $IMAGE_NAME"
```

gitlab-ci.yml 파일이 생성되면, 해당 저장소의 수정이 된 것으로 판단 자동적으로 파이프라인이 실행됩니다.

이후 정상적으로 배포된 docker image 와 원격지 서버를 확인 하였습니다.

![](/files/posts/20200916/successful_build.png)


## 결론

지금 작성한 gitlab-ci 코드를 통해서 지속적인 통합과 배포를 구성하였습니다.

이점으로는
1. 코드로 손쉬운 통합이 가능합니다. (다만 gitlab-runner 에 대한 어느정도 학습이 필요합니다.)
2. 솔직히 jenkins 보다는 쉬운거 같습니다....


아쉬운 점으로는
1. 테스트/스테이지 환경에 대한 부분은 브렌치를 통해서 구현이 필요할 것 같습니다.
2. 여러 서버로 배포시 대응이 필요합니다.