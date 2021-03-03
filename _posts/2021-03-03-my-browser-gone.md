---
layout: post
title:  "내 탭이 날라가 버렸어!"
date:   2021-03-03 23:55
author: monoless
tags:	[thread, process, browser, chromium]
---

# 내 탭이 날라가버렸어!

안녕하세요 오늘 원래  [이전]([/2020/12/09/raycasting-pseudo-3d/)에 이야기했던 내용처럼
웹에서 3d 엔진 렌더링을 구현하려고 하였으나
무한 루프에 빠지는 바람에... 실패하였고.
왜 무한 루프에 빠졌는데도 제 브라우져의 남은 탭들은 멀쩡한가에 대해서 고찰하였습니다.

먼저 제가 작성한 멍청한 코드를 먼저 보고 가시겠습니다.

```typescript
// 멍청한 코드
while (1) {
   // do something stupid
   updateWorld();
   renderWorld();
}
```

자 이러면 진짜 멍때립니다. 어디서 이런 코드를 보겠나요?
이것은 역시나 제대로 알지도 못하고 짜면 이렇게 되는 것입니다.
(이 코드가 왜 망한 코드인지는 javascript single threaded 관련글들을 참조해주세요.)

![](/files/posts/202103/page_not_response.png)

![](https://static.tvtropes.org/pmwiki/pub/images/deja_q_hd_046_resized_6484.jpg)

지금도 브라우져는 무한 로딩 중인데 이 블로그를 쓰는 내용은 날아가지 않습니다.
다른 탭은 멈추지 않고 실행되고 있습니다.
어떻게 이렇게 될까요? 
그냥 제가 엄청 비싼 킹왕짱 컴퓨터를 가지고 있어서 그냥 무대포로 버티는걸까요?

아닙니다.

크롬 (chrominum) 은 개별 탭을 쓰레드 (thread) 가 아닌 프로세스 (process) 로 관리하고 있기 때문입니다.

![](/files/posts/202103/much_much_process.png)

프로세스는 어플리케이션의 실행 프로그램이며, 쓰레드는 프로세스 내부에 있습니다.

![](https://developers.google.com/web/updates/images/inside-browser/part1/process-thread.png?hl=ko)

즉 저희가 탭 하나당, 브라우져 프로그램이 하나씩 실행되는것과 같은 것 입니다.
예전 IE는 웹 서핑 도중 플래시 플레이어가 멈추면 플래시 플레이어를 다시 실행하기 위해 웹 브라우저를 종료 후 다시 시작해야 했습니다.

이 구조상 메모리를 많이 먹을 수밖에 없습니다...

![](https://tl360.b-cdn.net/wp-content/uploads/2016/11/Simple-Hacks-And-Best-Tools-To-Limit-Memory-Usage-In-Google-Chrome.jpg)

이미 죽어버린 프로세스 입니다.

![](https://developers.google.com/web/updates/images/inside-browser/part1/tabs.png?hl=ko)

그렇다고 해서 전부 각기 프로세스가 나눠지는것은 또 아니더라구요
가끔 생각해보니 탭 하나만 죽은게 아니라 모두 죽어버리는 일이 있었던것이 기억 났습니다.

![](https://developers.google.com/web/updates/images/inside-browser/part1/browserui.png?hl=ko)

아래 표는 크롬의 각 프로세스들이 어떤 역할을 하는지 설명합니다:

| -- | -- |
| 브라우저 | 주소 창, 뒤로 및 앞으로 이동 버튼을 포함한 어플리케이션의 "chrome" 부분을 제어합니다. 또한 네트워크 요청 및 파일 액세스와 같은 웹 브라우저의 권한이 부여된 보이지 않는 부분을 제어합니다. |
| 렌더러 | 웹사이트가 디스플레이 될 때 탭 안의 모든 것 담당. |
| 플러그인 | 플래시와 같은 웹사이트가 사용하는 모든 플러그인 담당. |
| GPU | 다른 프로세스와 분리된 GPU 작업을 제어합니다. GPU는 여러 앱의 요청을 제어하고 동일한 표면에 표시하기 때문에 다른 프로세스로 분리됩니다. |

오늘은 간단히 모던 브라우져 (chrominum) 에 대해서 수박 겉핱기식으로 알아보았습니다.

다음번에는 javascript single threaded 구조에 대해서 좀 더 공부하고 이것을 게임엔진에 녹여보도록 하겠습니다 ㅠ