---
layout: post
title:  "equals() And hashCode()"
date:   2019-11-21 01:00
author: ryan
tags:	[java]
---

# 객체를 비교하는 방법

연산자(==), 'equals()', 'hashCode()'

> ## 연산자(==)

두 객체가 같은것을 가르킬 때 "true" 를 리턴.

== 연산자는 피연산자가 primitive type(int, double, boolean, ...)일 때는 값이 같은지 비교하고, 피연산자가 그 외 객체, reference type일 때 가리키는 주소가 같은지를 검사한다

아래의 예제는 연산자 비교 차이에 대한 이해를 돕기 위해 String 객체를 예로 들었다.

![fig1. String 은 객체로 연산자 비교를 한다](/files/posts/201911/fig1.png)

![fig2. 객체에 대한 연산 결과](/files/posts/201911/fig2.png)


*****

> ## equals()

equals는 내용이 같은지를 검사하는 메서드.

Primitive Type은 내용이 같은지 검사하고 Reference Type은 객체의 주소가 같은지 검사 한다.

'==' 연산자와 다른 점은 완전히 같은 객체를 가리키지 않아도 개발자가 true로 만들 수 있다

![fig3. equal 을 이용한 비교](/files/posts/201911/fig3.png)

![fig4. equal 의 결과](/files/posts/201911/fig4.png)



첫번째 equal 의 경우 주소도 같고 값도 같다. true 를 리턴한다.

두번째 equal 의 경우 주소는 다르지만 값이 같다. true 를 리턴한다.

아래의 예제의 경우 객체의 값이 같은데 어떤 결과가 노출되는지 확인해 보자.

![fig5. 예상과는 다른 상황](/files/posts/201911/fig5.png)


결과는 '==' 연산자와 equals() 모두 false 를 리턴 했다. 

자바는 해당 객체의 내용(값) 이 같은지 모르고 있었다. 개발자의 의도에 따라 name 만 같으면 같은 객체로 볼 수도 있고 경우에 따라 여러가지 경우가 존재하기 때문이다.

equals() 를 제대로 사용하기 위해서는 method override를 통해 개발자의 의도에 맞게 재정의가 필요하다. (IDE 지원도구 또는 lombok plug-in 을 통해 편하게 생성 가능하다.)

String 은 내부에서 equals() 를 override 하고 있어서 위의 결과가 다른 것이다.


*****

> ## hashCode()

메모리에서 가진 hash 주소 값을 반환한다. 주소의 hash 값을 통해 비교가 가능하다. 반환되는 hash 값이 같은 경우가 존재할 수도 있으므로 중복을 허용하지 않는 collection 과 함께 사용한다.

hashCode() 역시 override 를 통해 재정의가 필요하며 equals() 재정의시 같은 방식으로 함께 정의 해주어야 한다. 

equals() 로 같은 객체라면 반드시 hashCode() 도 같아야 한다.


*****

> ## 언제 equals 와 hashCode 를 사용(override)하면  좋을까?

Map은 get(key)를 통해 반복문 수행 없이 해당 key값이 등록되어 있는지 아닌지를 바로 확인 할 수 있습니다.  Map의 구현체인 HashMap이나 LinkedHashMap을 보면 바로 이 인스턴스의 hashCode 메소드 결과를 통해 key를 비교한다는 것을 확인 할 수 있습니다. 즉, 인스턴스의 hashCode 메소드 결과가 같다면 동일한 key로 간주하겠다는 것입니다

![](/files/posts/201911/fig6.png)
![fig6.7. Map 구현체 내부](/files/posts/201911/fig7.png)
 

다음 예제의 Entity 내부의 데이터를 특정 키를 기준으로 분류해야 하는 일이 있을때 Map 과 hashCode() 를 사용하여 이 문제를 해결할 수 있습니다. 

![](/files/posts/201911/fig8.png)
![](/files/posts/201911/fig9.png)
![fig8.9.10. 분류를 위한 테이블과 분류 로직](/files/posts/201911/fig10.png)
 

### reference
  * [http://jojoldu.tistory.com][jojoldu.tistory.com]  
  * [https://nesoy.github.io][nesoy.github.io]    

[jojoldu.tistory.com]: http://jojoldu.tistory.com
[nesoy.github.io]: https://nesoy.github.io
