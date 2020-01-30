---
layout: post
title:  "처음 해본 Kotlin"
date:   2020-01-29 10:00
author: kapjong
tags:	[android, kotlin, Jetbrain, apply, let, also]
---

# 처음 해본 Kotlin
2019년말 처음으로 실무에 Kotlin 개발을 진행 해보았고 좋았던 점들을 포함 Kotlin에 대한 소개를 진행 하고자 합니다.  
기본 적은 내용은 <b>[Android 개발을 수주해서 Kotlin을 제대로 써봤더니 최고였다!](https://qiita.com/omochimetaru/items/98e015b0b694dd97f323)</b> 포스팅 내용을 참고 하였으며  
필자에 주관적인 견해가 많이 있으며, 시대가 지남에 따라 사실과 다를수 있습니다.  

## Kotlin 언어 주변 환경
취미로 개발하는 것과는 달리 실무 개발의 경우 유명하지 않은 언어는 개발이 중단되거나 나중에 언어가 없어지는 위험 부담이 있습니다.  
Kotlin은 아직 까지 C, Java 등등 메이져 프로그램 언어에 비해 보편적이지는 않았지만,  
2019년 구글 I/O Kotlin은 안드로이드 공식 언어로 지정 되어 앞으로가 더 밝은 상황 입니다.    
아직 까지는 Kotlin은 모바일 개발에 최적화 되어 있지만 훗날 Java 대안 프로그램 Top3에 들것일라는 확신이 들었습니다.  

### 누가 만들었을까?
개발을 주도하고 있는 업체는 Jetbrains입니다.  
Jetbrains는 IntelliJ 툴을 개발하고 판매하는 회사로 유명합니다.  
Java 개발툴을 개발하고 있는 수준이기 때문에, 컴파일러와 관련된 기술력이나 프로그래밍 언어에 대한 이해는 상당할 것이라고 생각합니다.    
Android의 개발 환경이 Eclipse + ADT에서 Android Studio로 전환된지 오래지만,     
이 Android Studio는 IntelliJ를 Android 개발을 위하여 고친 버전입니다.  

### 이러다 Jetbrains도 구글에 먹히는 거 아닌가?
![](/files/posts/202001/eat.gif)
안드로이드 개발과 Kotlin에 유착 관계가 높아 지니 이런 생각이 들었지만  
[2017년도 official 이지만 아직까지 젯브레인 인수에 대한 이슈는 없습니다.](https://blog.jetbrains.com/kotlin/2017/05/kotlin-on-android-now-official/)  
<br>
젯브레인 공식 블로그에서 구글에 인수계획은 전혀 없으며 다른 도구 개발에도 영향이 없을 것이라고 밝혔습니다.  
또한 Kotlin 릴리즈 사이클도 안드로이드나 안드로이드 스튜디오와 별도로 계속 독립적으로 가져갈 것이라고 하였습니다.    
다만 안드로이드 스튜디오용 플러그인을 만드는 일에 대해서는 젯브레인이 안드로이드 스튜디오팀과 협업하며 이어간다고 하였습니다.    

### Kotlin 좋으다
개발자에게 좋은 프로그램 언어란 무었일까?  
<ol>
  <li>안정적 구동&컴파일 환경과 기본으로 하며</li>
  <li>함수 사용이 편하고</li>
  <li>유용한 라이브러리가 많이 있고</li>
  <li>디버깅이 쉽고</li>
  <li>불필요한 코딩량이 적고</li>
  <li>에러 헨들링 편하고</li>
  <li>유지 보수가 편한 프로그램 언어 등등</li>
</ol>
물론 다른 이유에 좋은 프로그램 언어도 있겠지만 Kotlin은 위 장잠들을 모두 충족 시켜 주었습니다.  
![](/files/posts/202001/201472831417162399.jpg)

## Kotlin 표준함수 몇 가지 소개
Kotlin 스터디 시작 하고 블로그 서핑을 하고 있을때 이런 문구가 있었습니다,  
### "run, with, apply, let, also 함수를 잘 활용 하면 Kotlin은 50% 먹고 들어 간다"  
Kotlin에 경우 기본적으로 람다식 헨들링이 이루어 진 이후 객체 반환, 객체 추론이 가능 한 점을 이용 하여  
apply, let, also 함수를 사용 하여, 코드를 보다 효율적이고 유연 관리합니다.

#### apply 
<ol>
  <li>반복되는 코드를 줄이기 위한 함수</li>
  <li>람다식이 끝난 시점에 사용 가능</li>
  <li>수정된 수신자 객체가 반환 됨</li>
</ol>

```java
// java식 표현
val menuFile = File("menu-file.txt")
menuFile.setReadable(true)
menuFile.setWritable(true)
menuFile.setExecutable(true)

// Kotlin식 표현
val menuFile = File("menu-file.txt").apply {
    setReadable(true)
    setWritable(true)
    setExecutable(true)
}
```

#### let 
<ol>
  <li>null 가능 타입에 대한 처리</li>
  <li>람다식이 끝난 시점에 사용 가능 하며</li>
  <li>수신자 객체가 반환 됨</li>
  <li>? 연산자를 같이 사용해 주세요~</li>
</ol> 

```java
    // java 형식 표현    
    fun formatGreeting(vipGuest: String?): String{
        return if (vipGuest != null) {
            "오랫만입니다, $vipGuest . 테이블이 준비되어 있으니 들어오시죠."
        } else {
            "저희 술집에 오신 것을 환영합니다. 곧 자리를 마련해 드리겠습니다."
        }
    }
    
    // Kotlin ?.let 사용
    fun formatGreeting(vipGuest: String?): String{
        return vipGuest?.let {
            "오랫만입니다, $it . 테이블이 준비되어 있으니 들어오시죠."
            } ?: "저희 술집에 오신 것을 환영합니다. 곧 자리를 마련해 드리겠습니다."
    }
```

#### also 
<ol>
  <li>apply 비슷</li>
  <li>람다의 결과 대신 수신자 객체 변화 없이 반환함</li>
  <li>데이터 초기화 등에 사용 용이</li>
</ol>
   
```java
    // java 형식 표현  
    class Book(val author: Person) {
        init {
          requireNotNull(author.age)
          print(author.name)
        }
    }

    // Kotlin also 사용
    class Book(author: Person) {
        val author = author.also {
          requireNotNull(it.age)
          print(it.name)
        }
    }
```

# 결론
실무에 적용해본 apply, let, also 함수 사용 효과는 서비스 출시 이후에 장점이 보이기 시작 합니다.  
let 사용으로 NullpointerException 줄었으며  
개발 시는 몰랐지만 시간이 지난 후 코드 가독성이 좋아져 유지보수 시 장점을 보였습니다.  
하지만 run, with, apply, let, also 함수 중 2개 이상 중첩 으로 사용 하니 코드 가독성이 매우 떨어 지는 현상도 확인 하였습니다.  

---
## 참고
 * [Android 개발을 수주해서 Kotlin을 제대로 써봤더니 최고였다!](https://qiita.com/omochimetaru/items/98e015b0b694dd97f323)
 * [젯브레인 공식 블로그](https://blog.jetbrains.com/kotlin/2017/05/kotlin-on-android-now-official/)
 * [빅너드랜치의 코틀린 프로그래밍](https://github.com/Jpub/BNR_Kotlin)