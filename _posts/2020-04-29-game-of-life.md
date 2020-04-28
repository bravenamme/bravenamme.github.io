---
layout: post
title:  "phaser 로 구현해 보는 콘웨이의 생명게임"
date:   2020-04-28 04:00
author: monoless
tags:	[phaser, typescript, game, life, cells]
---

# phaser 로 구현해 보는 콘웨이의 생명게임

## 존 호턴 콘웨이 교수님의 명복을 빕니다.

![John Horton Conway, 1937년 12월 26일 ~ 2020년 4월 11일](https://www.princeton.edu/sites/default/files/styles/scale_1440/public/images/2020/04/20090310_ConwayKochen_DJA_066-copy.jpg?itok=BbmXyoCQ)

먼저 이 자리를 빌어서 ‘마법적인 천재(Magical Genius)’로 불리는 영국 태생의 수학자 존 호턴 콘웨이   
프린스턴대 수학과 교수님의 명복을 빕니다.  
또한 코로나19로 희생되신 모든 이들의 명복을 빕니다.

1937년 영국 리버풀에서 태어난 콘웨이 교수는 유한군, 매듭이론, 조합론적 게임이론, 블록부호 등을 만들어낸 수학자입니다. 
교수님이 유명해진 것은 ‘라이프 게임(Life of Game)’이라는 개념을 고안하면서부터인데요,  
임의로 배열된 세포들이 기본 법칙에 의해 자동으로 생성·소멸하면서 삶과 죽음, 그리고 증식의 퍼즐을  
만들어낸다는 개념을 정리한 이 이론은 후에 컴퓨터과학에도 큰 영향을 미쳤습니다.  

콘웨이 교수님은 이러한 공로를 인정받아 1981년 영국왕립학회 회원으로 임명됐고 1985년 프린스턴대 교수로  
임명됐을 될 때는 총장으로부터 “20세기 가장 뛰어난 수학자 중 한 명”이라는 찬사를 받기도 했습니다.

오늘 이 시간은 [phaser](https://phaser.io/) 게임 프레임워크를 이용해서 ‘라이프 게임(Life of Game)’ 를 구현하는 것을  
목표를 하겠습니다.

## 라이프 게임(Life of Game) 이란

<video controls autoplay loop style="width: 100%; height: auto;">
  <source src="https://i.imgur.com/46B3SyZ.mp4" type="video/mp4">
</video>

바둑판처럼 정사각형의 여러 칸으로 나뉘어진 공간에서 한 칸에 한 마리씩 있는 세포들의 삶과 죽음이  
펼쳐지는 게임입니다.  
실제로는 개입할 수 있는게 없고 그냥 규칙에 따라 삶과 죽음이 일어나는 것을 재미있게 구경하시면 됩니다.

### 규칙

각 칸마다 존재하는 세포들은 인접한 8개의 이웃 세포 분포에 따라 생사가 결정 됩니다.  
기준은 다음과 같습니다.

1. 죽은 칸과 인접한 8칸 중 정확히 3칸에 세포가 살아 있다면 해당 칸의 세포는 그 다음 세대에 살아난다.
2. 살아있는 칸과 인접한 8칸 중 2칸 혹은 3칸에 세포가 살아 있다면 해당 칸의 세포는 살아있는 상태를  
유지한다.
3. 그 이외의 경우 해당 칸의 세포는 다음 세대에 고립돼 죽거나 혹은 주위가 너무 복잡해져서 죽는다.  
혹은 죽은 상태를 유지한다.

표로 정리하면 다음과 같습니다.

| 현 세대 | 인접 세포 갯수 | 다음 세대 |
| -- | -- | -- |
| LIVE | 0 | DEAD |
| LIVE | 1 | DEAD |
| LIVE | 2 | LIVE |
| LIVE | 3 | LIVE |
| LIVE | 4 | DEAD |
| LIVE | 5 | DEAD |
| LIVE | 6 | DEAD |
| LIVE | 7 | DEAD |
| LIVE | 8 | DEAD |
| DEAD | 0 | DEAD |
| DEAD | 1 | DEAD |
| DEAD | 2 | DEAD |
| DEAD | 3 | LIVE |
| DEAD | 4 | DEAD |
| DEAD | 5 | DEAD |
| DEAD | 6 | DEAD |
| DEAD | 7 | DEAD |
| DEAD | 8 | DEAD |

## 코드로 옮기기

### phaser scene system

> 전체 코드는 여기를 참조 하세요.  
> [https://github.com/monoless/lifegame](https://github.com/monoless/lifegame)

먼저 phaser 의 scene 시스템을 이해해야 합니다.

```typescript
class MainScene extends Phaser.Scene {
    constructor() {
        super(SCENE_KEY);
    }

    public init(): void {
        // init
    }

    public preload(): void {
        // preload
    }

    public create(): void {
        // create
    }

    public update(): void {
        // update
    }
}
```

각 메쏘드는 다음의 내용을 정의 합니다.

| 메쏘드명 | 역할 |
| -- | -- |
| init | scene 초기화 |
| preload | 에셋 (이미지/오디오 등) 호출 |
| create | 게임 오브젝트 생성시 |
| update | 게임 기동시 tick 단위 업데이트시 호출 |

실행 순서는 정리하자면 다음과 같습니다.

![scene 메쏘드 실행 순서](/files/posts/202004/diagram.png)

지금의 게임은 단순히 하나의 scene 을 가지고 하나의 object 만 다루도록 할 것 인데요.  
이번 글은 phaser 프레임워크의 전체적인 플로우를 다루도록 진행할 예정입니다.
디테일한 부분은 [예제](https://phaser.io/examples) 부분을 참조하세요

### 최초 세포 생성 해봅시다.

원래 라이프게임은 세포를 배치하고 지켜보는 재미이지만,  
제가 구현한 내용은 그냥 랜덤으로 배치하고 지켜보도록 간소화 시켰습니다.

create 메쏘드에서 최초 세포와 현재 세대를 정의 했습니다.  
표시할 수 있는 전체 세포를 기준으로 object 를 생성하였습니다.

또한 다음세대 메쏘드를 반복하여 실행하도록 하여 라이프게임의 규칙대로  
세포들의 탄생과 소멸을 정의하도록 하였습니다.

```typescript
    public create(): void {
        this.generation = 1; // 현재 세대를 정의
        Array.from(Array(this.stageSize).keys()).forEach(index => {
            // 각 픽셀단위로 cell 클래스를 생성하였습니다.
            // 해당 셀은 위치값과 live/dead 를 표기하는 값만 가집니다.
            this.cells.push(new Cell(this.numberToPosition(index)));
        });

        // this.nextGeneration 메쏘드를 타이머로 반복하게 실행하도록 하였습니다.
        this.timer = this.time.addEvent({
            delay: GameConfig.DELAY_INTERVAL,
            callback: this.nextGeneration,
            callbackScope: this,
            loop: true
        });
    }
```

### 세포들을 계속 그려줍시다.

> graphics property 는 프리젠테이션 레이어입니다.

여기서는 매 tick 마다 실행되는 메쏘드를 수정하여  
변경된 내역을 presentaion layer 로 그리도록 정의하겠습니다.

이러면 코드상에서는 조작하는 부분과 UI 그리는 부분을 따로 제어할 수 있습니다.

```typescript
    public update(): void {
        // 그려진 모든 버퍼를 지웁니다.
        this.graphics?.clear();
        this.cells.forEach(cell => {
            // 생존한 cell 들만 그려줍니다.
            cell.active && this.graphics?.fillRectShape(cell)
        });
    }
```

### 게임 규칙을 정의합니다.

여기서는 timer 로 호출하도록 설정된 nextGeneration 메쏘드를 정의하도록 하겠습니다.  
여기에서는 현재 세포의 생존유무와 주변 세포들의 존재유무로 점수로 매겨서  
다음세대에서도 생사 유무를 결정하는 메쏘드입니다.

```typescript
    private nextGeneration(): void {
        this.generation++; // 현재 세대를 정의
        Array.from(Array(this.stageSize).keys()).forEach(index => {
            // 이전 세대에서 살아남았는지
            const active = this.cells[index].active;
            // 주변 세포들이 존재하는지 판단해서 점수를 매깁니다.
            const score = this.neighborScore(index); 

            // 점수에 따른 세포들의 생존 유무
            if (active && (2 > score || 3 < score)) {
                this.cells[index].active = false;
            } else if (!active && 3 == score) {
                this.cells[index].active = true;
            }
        });
    }
```

### 이제 실행해봅니다!

<div style="position: relative; height: 0; padding-bottom: 56.25%; padding-top: 25px;">
<iframe src='https://www.svn.io/assets/lifegame/' style='position: absolute; top: 0; left: 0; width: 100%; height: 100%; border: 0;'></iframe>
</div>

![우와 대박](https://pbs.twimg.com/profile_images/804219262000693248/7SAdHjXD.jpg)

오늘도 고생하셨습니다.