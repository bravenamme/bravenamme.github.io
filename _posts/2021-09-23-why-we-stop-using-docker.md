---
layout: post
title:  "왜 우리가 이제 도커를 그만 써야 할까요?"
date:   2021-09-23 00:00
author: q_lazzarus
tags:	[docker]
---

# 들어가며

![docker!](https://d1.awsstatic.com/acs/characters/Logos/Docker-Logo_Horizontel_279x131.b8a5c41e56b77706656d61080f6a0217a3ba356d.png)

다들 아시고 계시다시피 도커는 이미 너무나 유명하고 인기 있는 도구이며 이미 수많은 개발자/엔지니어들이 사용하고 있습니다.
모르시는 분들을 위해 설명 하자면 배포를 쉽게 하기 위해 소프트웨어를 컨테이너화할 수 있는 도구입니다.

문제는 이제 도커가 유료화 됩니다. 이미 유료화를 발표 했습니다.

![docker subscription](https://miro.medium.com/max/1400/1*VVthxEXFGWojG2ypLG6P-Q.png)

새로운 가격 구조는 일반적인 유저에게는 여전히 공짜지만, 프로 유저는 한달에 5$ / 5명 이상의 팀에게는 한달에 7$ 를 가지고
이미 대규모 기업에게는 한달에 21$ 를 부담시키고 있습니다. (무려 인당)

지금 유료화하는것은 docker desktop 이며, 이미 docker hub 의 경우는 유료화된 상태입니다.

이번 내용에서는 유료화하는 docker desktop 에 대한 대응책을
소개하고자 합니다.

## Rancher Desktop
https://rancherdesktop.io/

랜처는 컨테이너와 쿠버네티스 클러스터를 관리, 배포하기에 최적화된 도구입니다.

최초버전에는 쿠버네티스 뿐만 아니라 다른 컨테이너 오케스트레이션들도 지원을 했지만 현재는 쿠버네티스만을 지원하고 있습니다.

## Red Hat CodeReady Containers
https://developers.redhat.com/products/codeready-containers/overview

레드햇 오픈시프트는 쿠버네티스 환경의 설정 및 관리를 자동화하는 기능을 통해 개발자가 클라우드 네이티브 기술을 보다 쉽게 사용하고 액세스하는 것을 목표로 한다고 업체 측은 설명했다. 개발자는 심층적인 쿠버네티스 전문 지식 없이도 차세대 엔터프라이즈 애플리케이션을 개발하는데 집중할 수 있습니다.

## 아니면 유료화 쓰기 혹은 무료 대상자 되기
실제 docker desktop 의 무료 대상자는 다음과 같습니다.

* 소규모 회사(직원 수 250명 미만 및 연간 매출 천만 달러 미만)
* 개인적인 사용
* 교육 및 학습(학업적 또는 전문적 환경에서 학생 또는 강사로서)
* 비상업적 오픈 소스 프로젝트<
* 2021년 8월 31일 이전에 다운로드한 Docker Desktop을 사용하는 경우

실제 사용에는 아직 문제가 없는 수준이며, docker-cli 는 유료화 대상이 아니라서 계속 사용이 가능합니다.

상기 위의 쿠버네티스 도입과 상관없이 간략하게 쓸꺼라면
docker-cli 를 익히고 저 무료 대상으로 사용하는게 현재로써는 
이게 가장 맞는 대안으로 보입니다.

Ref.

* [Why You Shouldn’t Use Docker Anymore](https://preettheman.medium.com/why-you-shouldnt-use-docker-anymore-47b7053559ce)
* [Goodbye Docker Desktop, Hello Minikube!](https://itnext.io/goodbye-docker-desktop-hello-minikube-3649f2a1c469)
* [향상된 보안을 약속하며 갑자기 유료 서비스로 전환한 도커](https://www.boannews.com/media/view.asp?idx=100402)
* https://www.bearpooh.com/92
* https://www.reddit.com/r/docker/comments/pfo6gm/docker_desktop_windows_alternatives/