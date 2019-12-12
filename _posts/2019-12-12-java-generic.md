---
layout: post
title:  "Java 제네릭"
date:   2019-12-12 10:00
author: maan
tags:	[java]
---

# Java 제네릭

제네릭은 타입에 대해 파라미터 형식으로 보여 주며 컴파일시 구체적인 타입으로 결정되도록 하는 것을 말한다.  클래스나 메소드에 매개 변수의 타입이나 리턴 타입에 사용 할 수 있다.

제네릭이 나오기 전 Object 타입으로 아래와 같이 표기 했지만 이렇게 쓸 경우 get을 값을 받을 때 원하는 타입으로 형변환을 해줘야 하는 번거로움이 있다.

```java
    public class Box {
        private Object object;
    
        public void set(Object object) { this.object = object; }
        public Object get() { return object; }
    }
```

아래와 같이 제네릭 형식으로 클래스를 정의 할 수 있다. 

```java
    /**
     * Generic version of the Box class.
     * @param <T> the type of the value being boxed
     */
    public class Box<T> {
        // T stands for "Type"
        private T t;
    
        public void set(T t) { this.t = t; }
        public T get() { return t; }
    }
    
    Box<String> box1= new Box<String>(); 
    Box<Integer> box2= new Box<Integer>(); 
```

제네릭은 1.6부터 지원하며 1.7 부터는 앞에 선언된 타입으로 컴파일러에서 추측이 가능 하기 때문에 뒤에 생성자에서 제네릭  생략 가능하다.  이걸 다이아몬드 표현이라고 한다.
```java
    Box<Integer> integerBox = new Box<>();
```

만약 1.6 이하 프로젝트라면 ....

![도망쳐](/files/posts/201912/191212_generic_1.JPG)

## 제네릭을 사용 하면 좋은 점.

컴파일시에 제네릭을 사용하여 미리 타입을 지정해 줌으로써 프로그램 실행시 발생 할 수 있는 버그를 사전에 방지 할 수 있다.  그리고 제네릭을 사용 하면 타입 변환을 제거 할 수 있다.

아래 예를 들어 보면 .. 
```java
    List list = new ArrayList();
    
    list.add("hello");
    list.add("world");
    
    for(Object c : list){
        String a = (String) c;
        System.out.println(a);
    }
```

일반적인 방법으로 코드를 작성 했을 때 타입이 지정 되지 않았기 때문에 리스트에 있는 값을 String 타입으로 넣어 주려 할 때 강제로 String 타입으로 변환 하여 넣어 줘야 한다. 

타입을 지정하지 않았기 때문에 위의 list에 숫자형 데이터를 넣어도 문제가 없지만 실행시 Exception이 발생하게 된다.
```java
    List list = new ArrayList();
    
    list.add("hello");
    list.add("world");
    list.add(4);
    
    for(Object c : list){
        String a = (String) c;
        System.out.println(a);
    }
    
    //결과
    Exception in thread "main" java.lang.ClassCastException: java.lang.Integer cannot be cast to java.lang.String
    	at io.github.lunaelva.Main.main(Main.java:18)
```

하지만 제네릭을 사용 하여 타입을 지정해 주면 실행 전부터 에러를 발견 할 수 있고 ,형변환이 생기면 생길 수록 성능 저하가 일어 날 수 있는데 제네릭을 사용 하면 이런 불필요한 형변환을 없애 줄 수 있다. 아래는 제네릭으로 변경한 예시이다.

```java
    List<String> list = new ArrayList<>();
    
    list.add("hello");
    list.add("world");
    
    for(String c : list){
        System.out.println(c);
    }
```
![](/files/posts/201912/191212_generic_2.JPG)

String으로 지정된 list에 숫자형 데이터를 넣었을 때 에러 발생

![](/files/posts/201912/191212_generic_3.JPG)

컴파일시에도 에러 발생!

## 타입 파라미터 이름

- E - Element (배열이나 맵등의 컬렉션에 들어 가는 element)
- K - Key 
- N - Number
- T - Type
- V - Value
- S,U,V etc. - 2nd, 3rd, 4th types

이런게 있다 정도로만 알고 넘어 가자.

## 다중 타입 파라미터

제네릭 타입은 두 개 이상의 멀티 파라미터를 이용할 수 있다. 이 경우 각 타입 파라미터는 콤마로 구분한다.

```java
    public interface Pair<K, V> {
        public K getKey();
        public V getValue();
    }
    
    public class OrderedPair<K, V> implements Pair<K, V> {
    	private K key;
    	private V value;
     
    	public OrderedPair(K key, V value) {
    		this.key = key;
    		this.value = value;
    	}
     
    	public K getKey() { 
    		return key; 
    	}
    	
    	public V getValue() { 
    		return value; 
    	}
    }
    
    Pair<String, Integer> p1 = new OrderedPair<String, Integer>("Even", 8);
    Pair<String, String>  p2 = new OrderedPair<String, String>("hello", "world");
```

new OrderedPair <String, Integer> 코드는 K를 문자열로, V를 정수로 인스턴스화한다. 따라서 OrderedPair 생성자의 매개 변수 유형은 각각 String 및 Integer입니다. 오토 박싱에 의해 int 값이 자동으로 Integer 객체로 자동 변환 되는데 오토박싱이랑 기초 자료형을 대응되는 클래스 객체로 자동 변환해 주는 기능이다.

위의 선언도 다이아몬드로 변환시킬수 있다.

## 제한된 타입 파라미터

제네릭으로 사용될 타입 파라미터의 범위를 제한할 수 있는 방법이 있다.

상속 및 구현 관계를 이용해서 타입을 제한할수 있는데 상위 타입은 클래스 뿐만 아니라 인터페이스도 가능 하다. 단, 인터페이스라고 해서 extends 대신 implements를 사용하지 않는다.
```java
    public class Box<T> {
    
        private T t;          
    
        public void set(T t) {
            this.t = t;
        }
    
        public T get() {
            return t;
        }
    
        public <U extends Number> void inspect(U u){ 
    		//Number 타입 또는 하위 클래스 타입(Byte, Short, Integer, Long, Double)의 인스턴스만 가져야 한다.
            System.out.println("T: " + t.getClass().getName());
            System.out.println("U: " + u.getClass().getName());
        }    
```

```java
        Box<Integer> integerBox = new Box<Integer>();
        integerBox.set(new Integer(10));
        integerBox.inspect(1.1);        
    }
 ```   
![실행결과](/files/posts/201912/191212_generic_4.JPG)

```java
    integerBox.inspect("some text"); // error: this is still String!    
``` 
![실행결과](/files/posts/201912/191212_generic_5.JPG)

## 제네릭메소드

제네릭 메소드는 매개타입과 리턴타입으로 타입파라미터를 갖는 메소드를 말한다. 

> public <타입파라미터, ...> 리턴타입 메소드명(매개변수, ...) { ... }

```java
    public class Pair<K, V> {
        private K key;
        private V value;
    
        public Pair(K key, V value) {
            this.key = key;
            this.value = value;
        }
    
        public void setKey(K key) { this.key = key; }
        public void setValue(V value) { this.value = value; }
        public K getKey()   { return key; }
        public V getValue() { return value; }
    }
    
    public class Util {
        public static <K, V> boolean compare(Pair<K, V> p1, Pair<K, V> p2) {
            return p1.getKey().equals(p2.getKey()) &&
                    p1.getValue().equals(p2.getValue());
        }
    }
```

제네릭 메소드를 사용 하는 방법은 아래와 같이 두 가지 방법이 있다.
``` java
    Pair<Integer, String> p1 = new Pair<>(1, "apple");
    Pair<Integer, String> p2 = new Pair<>(2, "pear");
    boolean same = Util.<Integer, String>compare(p1, p2); // 명시적으로 타입 지정
    
    System.out.println(same); //false
    
    Pair<Integer, String> p3 = new Pair<>(3, "strawberry");
    Pair<Integer, String> p4 = new Pair<>(3, "strawberry");
    same = Util.compare(p3, p4); //파라미터 값으로 타입 추정
    
    System.out.println(same); //true
```

## 와일드카드

와일드 카드는 알 수 없는 타입을 말하며 <?>로 표시 한다.  와일드카드는 파라미터 값의 타입이 명확하지 않을 때 사용 할 수 있으며 매개변수, 필드 또는 로컬 변수의 타입, 때로는 반환 타입 같은 다양한 상황에서 사용 될수 있다. 제네릭 메소드 호출에 대한 형식 인수, 제네릭 클래스 인스턴스 생성, 또는 슈퍼타입으로 사용될 수 없다.

### Upper Bounded Wildcards(상위 클래스 제한)

상위 타입의 하위 타입을 타입으로 가질 수 있다. 아래 예제로 보면 Number의 하위 클래스만 받을수 있다.
``` java
    public static double sumOfList(List<? extends Number> list) {
        double s = 0.0;
        for (Number n : list)
            s += n.doubleValue();
        return s;
    }
    
    List<Integer> li = Arrays.asList(1, 2, 3);
    System.out.println("sum = " + Util.sumOfList(li)); //sum = 6.0
    
    List<Double> ld = Arrays.asList(1.2, 2.3, 3.5);
    System.out.println("sum = " + Util.sumOfList(ld)); //sum = 7.0
```

![실행결과](/files/posts/201912/191212_generic_6.JPG)
### Unbounded Wildcards(제한없음)

List<?>로 표현이 되며 모든 타입을 가지고 올 수 있다. 
```java
    public static void printList(List<?> list) {
        for (Object elem: list)
            System.out.print(elem + " ");
        System.out.println();
    }
    
    List<Integer> li = Arrays.asList(1, 2, 3);
    List<String>  ls = Arrays.asList("one", "two", "three");
    Util.printList(li);
    Util.printList(ls);
```

![실행결과](/files/posts/201912/191212_generic_7.JPG)

### Lower Bounded Wildcards(하위 클래스 제한)

<? super A>와 같이 물음표와 super 키워드로 정의합니다. upper bounded wildcard와는 반대로 지정된 타입의 그 상위타입만을 허용합니다. 예로 List<? super Integer> 로 정의 하면 Integer의 상위인 Number와 Object 가 사용 가능합니다.
```java
    public static void addNumbers(List<? super Integer> list) {
        for (int i = 1; i <= 10; i++) {
            list.add(i);
        }
        System.out.println(list.toString());
    }
    
    List<Integer> li = Arrays.asList(1, 2, 3);
    Util.addNumbers(li);
```

![실행결과](/files/posts/201912/191212_generic_8.JPG)
```java
    List<Number> li = new ArrayList<>();
    Number num1 = 1;
    Number num2 = 1.0;
    li.add(num1);
    li.add(num2);
    Util.addNumbers(li);
```

![실행결과](/files/posts/201912/191212_generic_9.JPG)

## 요약
제네릭을 사용 하면 컴파일시 에러를 발생시켜 코드 안정성을 확보 할 수 있고, 미리 타입을 지정하여 불필요한 형변환을 줄여 준다.
제네릭 메서드를 사용하면 하나의 메서드로 모든 타입에 대응 할 수 있게 만들어 지기 때문에 재사용성도 좋아 진다.



#### 참고 사이트
- https://docs.oracle.com/javase/tutorial/java/generics/methods.html
- https://www.youtube.com/watch?v=QmTn1nL2jyA
- https://offbyone.tistory.com/327 