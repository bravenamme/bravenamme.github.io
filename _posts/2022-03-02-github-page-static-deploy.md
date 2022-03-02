---
layout: post
title:  "github action / github page 기능을 이용한 static deploy"
date:   2022-03-02 00:00
author: q_lazzarus
tags:	[github, page, ci, cd, svn]
---

## 들어가며
github page 기능을 이용해서 수많은 사람들이 블로그로 애용 했습니다.
그렇지만 나는 이미지 파일 하나 올릴껀데? jekyll 블로그를 써야해? 라는 사람이 있겠죠?
그런 사람을 위해서 준비했습니다!

## summary
webpack 을 이용해서 번들링할 예정이고, 
github action 을 통한 빌드와 배포 자동화를 진행할 예정입니다.

최근에 githuab pages 가 업데이트되어 별도의 설정 메뉴로 빠졌습니다!

![github pages](https://docs.github.com/assets/cb-70869/images/help/pages/publishing-source-drop-down.png)

이번 업데이트로 특정 branch 를 github page 로 지정 가능합니다. 아래와 같이 운영할 예정입니다.
 - gh-pages branch 를 github page 용도로 사용할 예정입니다.
 - main branch 는 소스코드를 저장할 용도입니다.

## 빌드를 설정하자
webpack 빌드 설정에 대해서는 이미 많이 다루고 있으니, 여기에서는 적지 않겠습니다. github action 을 통해서 빌드를 먼저 하도록 하겠습니다.

먼저 yml 파일을 생성합니다.
```bash
.github/workflows/deploy.yml
```

다음과 같이 작성할 예정입니다!
```yml
name: Build and deploy
on: 
  push: 
    branches: [ main ]
jobs: 
  build:
    runs-on: ubuntu-latest
    steps: 
      - uses: actions/checkout@main
      - name: set up node.js
        uses: actions/setup-node@main
        with:
          node-version: 16.x
      - name: cache node_modules
        uses: actions/cache@v1
        with: 
          path: node_modules
          key: ${{runner.OS}}-build-$${hashFiles('**/package-lock.json')}
          restore-keys: |
            ${{runner.OS}}-build-${{runner.OS}}-build
      - name: install devDependencies
        run: npm install
      - name: build
        run: npm run prod
      - name: deploy
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{secrets.GITHUB_TOKEN}}
          publish_branch: gh-pages
          publish_dir: ./dist
```

천천히 항목별로 설명 드리겠습니다.

```yml
name: Build and deploy
on: 
  push: 
    branches: [ main ]
```
main branch 를 통해서 pipeline 이 유발되도록 세팅했습니다.

```yml
jobs: 
  build:
    runs-on: ubuntu-latest
```
ubuntu 20.04 버젼 기반으로 pipeline 이 작동하기 위해서 명시합니다. [참고](https://github.com/actions/virtual-environments)

```yml
      - uses: actions/checkout@main
      - name: set up node.js
        uses: actions/setup-node@main
        with:
          node-version: 16.x
```
빌드를 위해서 소스코드 받아야 겠죠? checkout 구현을 위해서 steps 에 추가합니다. (actions/checkout@main)
또한 webpack 빌드를 위해서 node 를 설치합니다. (actions/setup-node@main) node-version 변수를 통해서 node 버젼 설정이 가능합니다.
지원하는 메이저 버젼은 여기에서 [참고](https://github.com/actions/setup-node) 하세요. 

```yml
      - name: cache node_modules
        uses: actions/cache@v1
        with: 
          path: node_modules
          key: ${{runner.OS}}-build-$${hashFiles('**/package-lock.json')}
          restore-keys: |
            ${{runner.OS}}-build-${{runner.OS}}-build
```

이 부분은 npm dependency module 설치를 캐쉬하여, gitlab action 에 실행시간을 줄여주는데 목적이 있습니다. [참고](https://github.com/actions/cache)

```yml
      - name: install devDependencies
        run: npm install
      - name: build
        run: npm run prod
```
이 부분은 실제 빌드를 위한 dependency 와 빌드 명령어를 쓰는데 목적이 있습니다.
(중요하지 않아 빠르게 넘어가겠습니다!)

```yml
      - name: deploy
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{secrets.GITHUB_TOKEN}}
          publish_branch: gh-pages
          publish_dir: ./dist
```
이 부분이 중요 키포인트 인데요. 실제 빌드가 완료된 결과물들을 github page로 배포하기 위한 스텝입니다. 

먼저 저 github_token 항목은 별도로 설정이 필요한 항목이 아닙니다!

> A GitHub Actions runner automatically creates a GITHUB_TOKEN secret to authenticate in your workflow. So, you can start to deploy immediately without any configuration.

github action 이 자동적으로 생성한다고 합니다

나머지 설정은 다음과 같습니다. (추가 옵션은 여기를 [참고](https://github.com/peaceiris/actions-gh-pages) 하세요)
 - publish_branch - 빌드할 결과물들을 올릴 branch 입니다. 여기에선 gh-pages 로
 - publish_dir - 빌드할 결과물들이 저장될 디렉토리입니다. 각자 webpack 설정에 따라서 변경하세요. 저는 dist 로 설정했습니다.


## github pages 설정하기

아까 공유 드렸던 내용대로 githuab pages 가 업데이트되어 별도의 설정 메뉴로 빠졌습니다!

![github pages](https://docs.github.com/assets/cb-70869/images/help/pages/publishing-source-drop-down.png)

해당 설정 페이지에서 branch 를 gh-pages 로 설정하고, 경로는 / 로 설정해주세요
(gh-branch 에서 / 디렉토리로 서비스될 예정입니다!)

Ref.
* [Configuring a publishing source for your GitHub Pages site](https://docs.github.com/en/pages/getting-started-with-github-pages/configuring-a-publishing-source-for-your-github-pages-site)
* [GitHub Actions Virtual Environments](https://github.com/actions/virtual-environments)
* [Checkout V3](https://github.com/actions/checkout)
* [setup-node](https://github.com/actions/setup-node)
* [cache](https://github.com/actions/cache)
* [peaceiris/actions-gh-pages](https://github.com/peaceiris/actions-gh-pages)
* [Github Actions를 이용하여 gh-pages 자동 배포하기](https://davidyang2149.dev/front-end/github-actions%EB%A5%BC-%EC%9D%B4%EC%9A%A9%ED%95%98%EC%97%AC-gh-pages-%EC%9E%90%EB%8F%99-%EB%B0%B0%ED%8F%AC%ED%95%98%EA%B8%B0/)
* [CI/CD ② GitHub Actions로 Vue.js 자동 배포](https://velog.io/@xxell-8/GitHub-Pages-%ED%99%9C%EC%9A%A9-GitHub-Actions%EB%A1%9C-Vue.js-%EC%9E%90%EB%8F%99-%EB%B0%B0%ED%8F%AC)