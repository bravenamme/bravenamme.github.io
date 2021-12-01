---
layout: post
title:  "jekyll 블로그 wsl2 + docker + vscode 에서 작업하기"
date:   2020-07-15 16:08
author: q_lazzarus
tags:	[jekyll, wsl2, docker, windows, linux, vscode]
---

## 계기

 깃헙 블로그를 작성하기 위해서는 로컬 pc 에서 ruby 를 설치하고 jekyll 환경 구성을 해야 합니다.  
오늘 포스팅을 작성할려고 했으나, windows 를 다시 깔아버려서 세팅하기가 너무 귀찮더라구요...  
이번 기회에 저처럼 자주 이러저리 옮겨다니는 사람들을 위해 이번 내용을 작성하게 되었습니다.

![](/files/posts/202007/lazy.png)

우선 저는 windows 10 에 wsl2 와 docker 를 설치하여 사용하였습니다.


## wsl2 사용하기

### wsl2 란?

지난 5월 윈도우 Windows 10의 대규모 업데이트가 있었습니다. 이번 업데이트는 2019년 11월 이후 첫 대규모 업데이트로 사용자를 위한 다양한 기능이 추가되었을 뿐만 아니라,  
Microsoft 에서 미리 예고한 대로 WSL2 (Windows Subsystem for Linux 2) 를 포함하고 있습니다.

* [The Windows Subsystem for Linux BUILD 2020 Summary](https://devblogs.microsoft.com/commandline/the-windows-subsystem-for-linux-build-2020-summary/)

간략히 이야기 드리자면, 윈도우즈에서 경량 가상화 기술을 사용해 리눅스를 구동할 수 있도록 도와주는 기능입니다. 

WSL 2 를 설치하려면 먼저 Windows 10 2004 를 업데이트해야 합니다.  
현재 설치여부는 Windows Key + R 을 눌러서 실행창에 winver 를 넣고 버전을 확인해 보면 됩니다.

![](/files/posts/202007/winver.png)

### wsl2 활성화 하기

2004 업데이트를 설치했다면 PowerShell 을 관리자로 열고 다음 명령을 실행해서 WSL 을 활성화해줍니다.

```bash
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
```

설치 완료를 위해서 재부팅을 해줍니다.

### wsl2 기본 버젼으로 설정

새 Linux 배포를 설치할 때 Powershell에서 다음 명령을 실행하여 WSL 2를 기본 버전으로 설정합니다.  
똑같이 파워쉘을 띄워서 아래 명령어를 입력해줍시다.

```bash
wsl --set-default-version 2
```

### linux 배포판 선택하기

각종 배포판을 지원하긴 하지만, 무난한 Ubuntu 18.04 LTS를 설치 해봅니다.  
(다른 배포판들은 [여기에서](https://docs.microsoft.com/en-us/windows/wsl/install-win10) 확인 가능합니다.)

microsoft store 에서 ubuntu 를 검색해서 설치합니다.

![](/files/posts/202007/ubuntu_18_04_lts.png)

### 배포판 실행하기

설치한 배포판을 실행하면 설치하는데 시간이 조금 소요됩니다.

![](/files/posts/202007/ubuntu_install.png)

아래 명령어를 이용해서 wsl 버젼을 확인합니다.

```
PS C:\Users\ecst> wsl --list --verbose
  NAME                   STATE           VERSION
* Ubuntu-18.04           Running         2
```

## docker 사용하기

(https://docs.docker.com/docker-for-windows/wsl-tech-preview/)

위의 링크를 참고하여 윈도우용 docker desktop을 설치 후  
General에서 Enable the experimental WSL 2 based engine를 체크하여주고  
Resources > WSL Integration에서 설치한 리눅스 버전을 체크해주시면 됩니다.

![](/files/posts/202007/docker_general.png)

![](/files/posts/202007/docker_wsl.png)

## jekyll docker 설정하기

docker-compose.yaml 파일을 만들어 봅니다.

```yaml
version: "3.3"

services:
  blog:
    image: jekyll/jekyll:latest
    command: jekyll serve --force_polling --drafts --livereload --trace
    ports:
      - "4000:4000"
    volumes:
      - ".:/srv/jekyll"
```

docker-compose는 서비스에 필요한 docker 컨테이너를 한번에 실행해주는 프로그램 입니다.

다음은 작성된 코드의 의미입니다.

- version : docker-compose의 버전을 의미
- services : docker-compose는 여러개의 컨테이너를 띄울 수 있음. 하단 블록에 기입
  - blog : 실행할 컨테이너의 이름
    - image : docker 이미지
    - command : 컨테이너가 run 하면 실행할 명령
    - ports : {local_port}:{container_port} , 로컬 4000번 port를 컨테이너의 4000번 포트와 매핑
    - volumes : {local_dir}:{container_dir} , 현재 경로를 컨테이너의 /srv/jekyll 경로에 mount 하겠다는 뜻

이 작성된 코드로 아래 명령어를 내리면 자동으로 container 가 실행됩니다.

```bash
docker-compose up
```

![](/files/posts/202007/docker-compose.png)

정상적으로 완료가 되면 자동으로 빌드되는 것을 확인할 수 있습니다

![](/files/posts/202007/docker-result.png)

## 추가) vscode 연동

![](/files/posts/202007/docker-dashboard.png)

위는 docker dashboard 에서 확인된 container 입니다. vscode 로 열기를 선택하시면
해당 container 에 remote wsl 로 접속을 확인할 수 있습니다!
