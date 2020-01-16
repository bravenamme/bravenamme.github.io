---
layout: post
title:  Material UI For IOS
date:   2020-01-09 10:00
author: sunyung
---
## 머터리얼 디자인이란?
머터리얼 디자인이란 플랫 디자인의 장점을 살리면서도 빛에 따른 종이의 그림자 효과를 이용하여 입체감을 살리는 디자인 방식을 말한다. 2014년 구글이 안드로이드 스마트폰에 처음 적용하기 시작했다.
(플랫 디자인과 마찬가지로 최소한의 코딩만으로 가독상을 최대한 살리는 미니멀리즘을 추구한다.)

## 머터리얼 Github
![출처 : https://github.com/material-components/material-components-ios](/files/posts/github_material_git_home.png)

![출처 : https://github.com/material-components/material-components-web](/files/posts/github_material_git_home1.png)
4년 전부터 지속해오던 프로젝트이며, 지금도 계속 Develop 중이며 Document 나 Sameple 코드가 상당히 잘 정리되어있다.

<https://github.com/material-components/material-components-web>

## 왜 머터리얼 UI를 적용 해야하는가?

![출처 : https://github.com/material-components/material-components-ios](/files/posts/2020_01_09_image_01.png)
달라도 너무 다르다.

안드로이드 IOS 및 Web 에서 서로 다른 UI를 제공하면 사용자의 혼란이 있을수 있다. 예를 들어 다이얼로그의 "확인" 버튼이 안드로이드는 오른쪽인데 반해 IOS에서 왼쪽에 노출된다면 사용자는 헷갈릴 수 밖에없다.
최근 통계에 따르면 소셜 커머스의 경우 웹과 모바일에 동일한 UI를 적용했더니 매출이 더욱 증대 되었다고 한다.
또한 요즘에는 글로벌한 기업들이 Material Design을 많이 선호 하고 있으며, 그 안에서의 디자인의 포인트를 주는 것이 유행 처럼 번지고 있다.


##  머터리얼 컴포넌트 소개(웹)

![출처 : https://github.com/material-components/material-components-web](/files/posts/github_material_git_home2.png)

## 자 그럼 Slider를 하나 만들어 보자.

## Android

    private void setUpSlider(

    View view, @IdRes int switchId, @IdRes int sliderId, Slider.LabelFormatter labelFormatter) {
    
    final Slider slider = view.findViewById(sliderId);
  
    slider.setLabelFormatter(labelFormatter);
  
    SwitchCompat switchButton = view.findViewById(switchId);
  
    switchButton.setOnCheckedChangeListener(
  
      (buttonView, isChecked) -> slider.setEnabled(isChecked));
      
    switchButton.setChecked(true);
  
    }


## Swift

    override func viewDidLoad(){

    super.viewDidLoad()

    let slider = MDCSlider(frame : CGRect(x:50, y:50, width: 100, height: 27))

    slider.addTarget(self,

    action: #selecter(didChangeSliderValue(senderSLider:)),
    for: .valueChanged)

    view.addSubview(slider)

    }

## Objective-c

    -(void)viewDidLoad{

    MDCSlider *slider = [[MDCSlider alloc] initWithFrame:CGRectMake(50, 50, 100, 27)];

    [slider addTarget:self

    action:@selector(didChangeSliderValue:]

    forControlEvents:UIControlEventValueChanged];

    [self.view addSubview:slider];

    }

## Web

<https://material.io/develop/web/components/input-controls/sliders/>

## Flutter

<https://api.flutter.dev/flutter/material/Slider-class.html>

![짠 하고 슬라이더가 나온다](/files/posts/2020_01_09_image_04.png)


자 그럼 이제 아래 링크에서 시작해보자.!

<https://material-ui.com>



