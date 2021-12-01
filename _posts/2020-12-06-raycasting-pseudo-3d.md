---
layout: post
title:  "울펜슈타인3D 는 과연 어떻게 3d를 구현했을까요?"
date:   2020-12-09 13:00
author: q_lazzarus
tags:	[game, wolfenstain, raycasting, 3d]
---

1992년 5월 5일, *울펜슈타인 3D* 가 출시됩니다. 그전에도 1인칭 시점의 게임은 존재하였지만, 이 게임은 화려한 256 컬러 그래픽, 빠른 속도, 높은 프레임 레이트, 영리한 AI, 선명한 사운드 효과, 매력적인 음악을 가능하게 한 엔진으로 신선한 충격을 선사하였습니다.

*울펜슈타인3D* 은 과연 어떻게 3d를 구현했을까요?

[![Wolfenstein 3D Trailer](https://img.youtube.com/vi/7P_dic-pSKo/0.jpg)](https://www.youtube.com/watch?v=7P_dic-pSKo "Wolfenstein 3D Trailer")

*레이캐스팅* 이란 2차원 맵을 3차원 시점을 구현하기 위한 렌더링 테크닉입니다. 이전 시대의 PC 는 실시간으로 리얼한 3D 엔진을 구현하기에는 턱도 없이 느렸습니다. 그래서 *레이캐스팅* 은 첫번째 대안으로 제시되었습니다. 빠르거든요. 단순히 수직선에만 계산하면 되기 때문입니다. 물론 그 당시 이 기술로 유명한 게임은 *울펜슈타인 3D* 였습니다. 

![](https://www.permadi.com/tutorial/raycast/images/figure1.gif)

![raycasting](https://i.stack.imgur.com/AC2tt.gif)

사실 *레이캐스팅* 의 역사는 오래되었습니다. 이 용어는 1982년 스코트 로스의 *[구조적 입체 기하학](https://ko.wikipedia.org/wiki/%EA%B5%AC%EC%A1%B0%EC%A0%81_%EC%9E%85%EC%B2%B4_%EA%B8%B0%ED%95%98%ED%95%99)* 모델을 렌더링하기 위한 기법을 묘사하는 컴퓨터 그래픽스 논문에서 처음 사용되었습니다.

레이 캐스팅의 기본 개념은 다음과 같습니다. 지도는 2D 로 된 정사각형 구역이며, 각 정사각형은 0 (텅빈 공간) 과 정수로 표기 됩니다. (특정 색상이나 텍스쳐를 가진 벽)

아래 이미지를 보시면 노란색은 플레이어의 위치, 표시된 삼각형은 플레이어의 시야를 의미합니다. 플레이어 시야 끝에서 부터 끝까지 광선을 쏜다고 가정을 합시다. 이 광선은 벽에 부딪칠때까지 이동합니다. 만약 벽에 부딪쳤다면 벽까지 얼마나 걸렸는지 거리를 계산합니다. 그리고 계산된 거리는 화면에 그려질 하는 벽의 크기 (원근감) 를 나타냅니다. 거리가 멀다면 플레이어에게 멀리 있고 화면에 작게 나타나겠죠. 반대로 거리가 가깝다면 플레이어에게 가까이 있고 화면에 크게 나타납니다.

![](https://lodev.org/cgtutor/images/raycastgrid.gif)

당연히 사람이면 광선들이 벽에 닿는 곳을 바로 알 수 있겠지만, 컴퓨터는 한번의 연산으로는 벽이 닿는 곳을 파악할 수 없었습니다. 왜냐하면 광선을 쏘면서 벽까지 거리를 파악하는 방법은 광선을 쏘고 tick 마다 광선이 벽에 있는지 없는지 검사하는 방법으로 벽에 맞았냐 안맞았냐를 파악하고 있기 때문입니다.

이러한 방법 때문에 한 화면에 여러 광선들을 쏘아도 벽을 놓칠 가능성이 존재했습니다. 예를 들어 아래의 사진을 보시면 각 광선들을 쏘고 광선들이 벽에 있냐 아니냐 검사하는 tick 들은 빨간점으로 표기 됩니다. 따라서 실제 특정 각도에서 벽이 없다고 판단할 수 있는 것 입니다.

![](https://lodev.org/cgtutor/images/raycastmiss.gif)

다시 tick 을 줄여도 벽을 감지할 가능성은 올라가지만, 여전히 벽을 감지하지 못할 가능성이 존재합니다. 또한 효율적이지도 않구요.

![](https://lodev.org/cgtutor/images/raycastmiss2.gif)

이런 방식이라면 무한한 정밀도를 가지기 위해서, tick 수치를 아주 아주 낮게 설정하면 됩니다. 다만 이것도 말도 안되는게 무한한 계산이 필요합니다. 그 당시가 92년이라고 생각하셔야 합니다. 지금 여러분이 가지고 계신 스마트폰이 그때 당시 컴퓨터보다 성능이 더 뛰어나다구요. 물론 좀 더 좋은 방법이 있습니다. 그건 광선이 벽을 만나기 전 tick 에서 근처에 모든 면을 확인하는 것입니다. 각 정사각형을 너비 1 이라고 가정하고, 각 정사각형은 정수값이며 쏜 광선의 확인하는 tick 은 소수점을 가집니다. 이걸 아래 그림과 같이 반올림해서 확인하게 된다면 손쉽게 충돌 유무를 판단할 수 있습니다.

![](https://lodev.org/cgtutor/images/raycasthit.gif)

이 방식을 DDA (Digital Diferential Analysis) 이라고 부르며 일반적으로 정사각형 격자에서 광선에 맞은 정사각형을 찾기 위한 빠른 알고리즘입니다. 

다음시간에는 이 알고리즘을 기반으로 Javascript 로 간단한 3D 엔진을 작성해보도록 하겠습니다.

ref
 * https://en.wikipedia.org/wiki/Maze_War
 * https://en.wikipedia.org/wiki/First-person_shooter
 * https://en.wikipedia.org/wiki/Wolfenstein_3D
 * https://lodev.org/cgtutor/raycasting.html
 * https://ko.wikipedia.org/wiki/%EA%B4%91%EC%84%A0_%ED%88%AC%EC%82%AC
 * https://namu.wiki/w/%EB%A0%88%EC%9D%B4%EC%BA%90%EC%8A%A4%ED%8A%B8
 * https://www.permadi.com/tutorial/raycast/rayc1.html