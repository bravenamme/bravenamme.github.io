---
layout: post
title:  "command shell"
date:   2021-09-16 10:00
author: ryan
tags:	[]
---

## 터미널 명령어

개발 업무를 하다보면 피할 수 없는게 커멘드 쉘 프로그램 이다. "Bourne Shell", "Bash", "CMD", "PowerShell", "fish", "zsh" 등등 종류도 다양하다. 사용처 역시 웹개발, 원격서버 접속, GIT 명령어 수행, 자동화 도구 스크립트 작성 등 다양하다.

![fig1. 다양한 사용처](/files/posts/202109/fig1.png)

```java
젠킨스 배포 스크립트 예

#!/bin/sh
#/var/lib/jenkins/workspace/Kuzal/target/*.jar

target=/home/ec2-user/kuzalBE
jenkins=/var/lib/jenkins/workspace/Kuzal/target
filename=kuzalBE.jar

# 파일있으면 삭제 
echo "deleting ${filename}"
cd $target
if test -e $filename
then rm -f $filename
fi
echo "delete done"

#jenkins 로 배포된 파일 복사
echo "copying ${filename}"
cp $jenkins/*.jar $target/kuzalBE.jar
chmod 755 $target/kuzalBE.jar
sleep 10
echo "copy done"

#서비스 run되어있는것  삭제
echo "stop service"
sudo kill $(ps aux | grep 'java -jar kuzalBE.jar' | awk '{print $2}')
sleep 20

#서비스 실행
echo "start service"
sudo nohup java -jar $filename 2>> /dev/null >>/dev/null &
echo "started service"
```

### 유닉스 명령어를 윈도우에서 사용

개발자가 선호하는 OS 는 윈도우가 차지하는 비중이 꽤 높다. 맥에서는 자연스럽게 리눅스 명령어를 사용하지만 윈도우에선 기존에 Cygwin 이라는 프로그램을 깔아야만 리눅스 명령어 사용이 가능 했었는데 최신 버전의 윈도우에선 리눅스를 포함하고 있으며 WSL 설치를 통해 사용 가능하다.

![fig2. 유닉스 명령어의 윈도우 OS 에서 사용하기](/files/posts/202109/fig2.png)

이번 글에서는 쉘 프로그램을 왜 배워야 하는지 간단한 소개를 하려고 한다. 스택오버플로우 설문조사에서 인기있는 언어에서도 상위권을 차지한 바가 있다고 한다.
![fig3. 스택오버플로우 설문조사](/files/posts/202109/fig3.png)


### 유닉스쉘 명령어 소개

- **`man`** : Manual 의 약자이며 help 와 같은 기능을 수행한다.
- **`clear`** : 화면의 내용을 모두 지워 준다.
- **`pwd`** : Print Working Directory 의 약자이며 현재 내가 있는 곳의 전체 경로를 보여줌
- **`ls`** : List 의 약자이고 현재 경로의 폴더와 파일들의 목록을 볼 수 있음<br/>
![](/files/posts/202109/fig4.png)
> 1. `-l` : LONG 의 약어 이며 자세히 보기에 사용 된다.
> 2. `-a` : all 의 약어이며 숨겨진 파일까지 모두 확인할 수 있다.
> 3. `-la` 과 괕이 두가지 옵션을 붙여서 사용할 수 있음
- **`explorer`** : 현재 위치에서 윈도우즈 탐색기 켜기 (Powershell)
- **`cd`** : Change Directory 의 약자이며 경로를 변경할 때 사용한다.<br/>
![](/files/posts/202109/fig5.png)
> 1. **`.`**  : 현재경로 
> 2. **`..`** : 현재 경로의 상위 
> 3. **`~`** : 사용자의 최상위 경로 (HOME) 
> 4. **`-`** : 바로 이전경로로 이동

- **`find`** : 파일 시스템 내에서 특정 파일을 찾을 때 사용<br/>
![](/files/posts/202109/fig6.png)

- **`touch`** : 파일이 존재하지 않으면 생성하고, 존재하는 파일이면 해당 시간으로 업데이트됨<br/>
ex) ~/projects/shell > touch new_file1.txt
- **`cat`** : 파일명에 해당하는 내용을 확인할 수 있음 <br/>
![](/files/posts/202109/fig7.png)<br/>
- **`echo`** : 문자열 출력<br/>
![](/files/posts/202109/fig8.png)<br/>
- **`mkdir`** : Make Directory , 디렉토리 생성
- **`cp, mv, rm`** : 파일 또는 폴더의 복사, 이동 및 삭제<br/>
![](/files/posts/202109/fig9.png)<br/>
- **`grep`** : Global regular expression print<br/>
![](/files/posts/202109/fig10.png)

<!--
그 외..

 - **`exprot, env, unset`** <br/>
![](/files/posts/202109/fig11.png) <br/>
export 로 지정한 환경변수를 사용할때는 앞에 $를 붙여준다. 삭제시 "unset {지정이름}" 으로 사용 가능하다 

현업 개발자라면 한번쯤은 사용해 봤거나 앞으로 사용하게 될 것이다. 서버의 로그를 보는데 주로 사용했을 테지만 그 외에도 사용처가 다양하다. 위에 소개한 간단한 기능들을 익혀두고 사용에 익숙해 지는것이 필요하다.  
 -->



REF. <br/>
드림코딩 [https://www.youtube.com/channel/UC_4u-bXaba7yrRz_6x6kb_w](https://www.youtube.com/channel/UC_4u-bXaba7yrRz_6x6kb_w)<br/>
흥개발자 [https://taekwang.tistory.com/23][https://taekwang.tistory.com/23]<br/>