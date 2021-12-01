---
layout: post
title:  "php 로 IntegerArray 구현하기"
date:   2019-11-27 23:52
author: q_lazzarus
tags:	[php, iterator, array, integer]
---

기본적으로 PHP 의 배열은 일반적인 ArrayList 구현이 아니라, Hash Table 입니다.  
그러다보니 php 개발자들은 배열을 배열처럼 쓰지 않고 Hash Table 처럼 이용하는 분들이 많습니다.  
(나쁜거 아니에요!)

```php
$a = ['q_lazzarus' => '킹왕짱'];
echo $a['q_lazzarus'];
```

다시 기초로 돌아가자면, array 는 동일한 자료구조의 반복 입니다.

![array data structure](https://media.geeksforgeeks.org/wp-content/uploads/array-2.png)

메모리 단위에서 생각해보면 동일한 크기의 방이 주루룩 있는 구조이죠.

![](https://cdn.pixabay.com/photo/2016/11/06/19/57/hotel-1803960_960_720.jpg)

!!

그렇다면, string 하나에 integer 몰빵해서 넣으면 되자너?

![profit!!](https://www.meme-arsenal.com/memes/ae6ae7737442185976d1e1cc19ea4f4f.jpg)

## 구현해보자 !
5개의 원소가 있는 배열이 있다고 가정하고 데이터를 읽는 간단한 함수를 만들어 보겠습니다.
```php
function array_get($i) {
    $data = '12345';
    return substr($data, $i, 1);
}

echo array_get(3);
```
```
3
```

짜잔! 이렇게 string 변수를 참조하여, 값을 리턴하는 간단한 코드가 만들어졌습니다.  
하지만 당연하게도 이건 못 씁니다.   
왜냐하면 원소의 데이터가 0~9 까지만 허용하는 말도 안되는 구현이기 때문입니다.  

## 자료형 ??
가만히 다시 생각해 봅시다. 설마 integer 데이터를 메모리에 string 처럼 저장하지 않겠죠...  
보통의 경우 integer 자료형은 4 Bytes 를 차지하고 있습니다.

| Data Type              | Size (in bytes) | Range                           |
| ---------------------- | --------------- | ------------------------------- |
| short int              | 2               | -32,768 to 32,767               |
| unsigned short int     | 2               | 0 to 65,535                     |
| unsigned int           | 4               | 0 to 4,294,967,295              |
| int                    | 4               | -2,147,483,648 to 2,147,483,647 |
| long int               | 4               | -2,147,483,648 to 2,147,483,647 |
| unsigned long int      | 4               | 0 to 4,294,967,295              |
| long long int          | 8               | -(2^63) to (2^63)-1             |
| unsigned long long int | 8               | 0 to 18,446,744,073,709,551,615 |
| singed char            | 1               | -128 to 127                     |
| unsigned char          | 1               | 0 to 255                        |

호오 그렇다면? 데이터를 변환해서 넣도록 하겠습니다.  
다행히 php 에서는 [pack](https://www.php.net/manual/en/function.pack.php) 이라는 함수가 이러한 변환을 편하게 도와줍니다.

```php
$data = str_repeat(pack('I', null), 5);

function array_get($i) {
    global $data;
    $binary = substr($data, $i * 4, 4);
    $unpack = unpack('I', $binary);
    return $unpack[1];
}

function array_set($i, $value) {
    global $data;
    $binary = pack('I', $value);
    $data = substr_replace($data, $binary, $i * 4, 4);
}

array_set(3, 65535);
echo array_get(3);
```
```
65535
```

오 이번에는 10 을 넘어서 심지어 65535 도 잘 나오고 있습니다.  
하지만 아직 문제가 있습니다. 

```php
echo array_get(2);
```
```
0
```

아니 어떻게 된 일이죠?? 분명히 공간만 만들어주기 위해서 null 을 넣어주었는데?

사실은 integer 자료형에서는 null 은 없습니다.  
왜냐하면 null 은 null 이기 때문입니다. (자료형도 달라요)  
그렇다보니 저희가 만든 함수에서는 null 은 처리가 불가능 합니다.  
그럼 어떻게 해야할까요?

![](https://i.pinimg.com/236x/58/47/f7/5847f7d07e2d99eda5f80bfa36901bad--kermit-the-frog-random-stuff.jpg)

## 꼼수 ??

생각해보면 데이터를 넣을때 한칸씩 미루면 되지 않을까요?
```php
$data = str_repeat(pack('I', 0), 5);

function array_get($i) {
    global $data;
    $binary = substr($data, $i * 4, 4);
    $unpack = unpack('I', $binary);
    $result = $unpack[1];
    if (0 === $result) {
        return null;
    }
    return $result - 1;
}

function array_set($i, $value) {
    global $data;
    if (null !== $value) {
        $value = $value + 1;
    }
    $binary = pack('I', $value);
    $data = substr_replace($data, $binary, $i * 4, 4);
}

var_dump(array_get(3));
```
```
NULL
```

이렇게 되면 데이터 제한이 4,294,967,295 에서 하나 소모가 되지만,  
이 정도면 괜찮은 것 같습니다.

## 실제 배열처럼 바꿔 보아요

어느정도 된 것 같습니다. 하지만 이것을 실제 배열 처럼 쓰기에는 아직 부족합니다.  
그래서 php 에서는 Array 구현하기 위한 [interface](https://www.php.net/manual/en/class.arrayaccess.php) 를 제공합니다.

```php
ArrayAccess {
    /* Methods */
    abstract public offsetExists ( mixed $offset ) : bool
    abstract public offsetGet ( mixed $offset ) : mixed
    abstract public offsetSet ( mixed $offset , mixed $value ) : void
    abstract public offsetUnset ( mixed $offset ) : void
}
```

클래스를 마치 php 기본 배열 처럼 접근 가능하게 하는 구현체이며  
제한적인 의미로 accessor overloading 입니다.

```php
$obj = new obj;
$obj[] = 'hello';
$obj[] = 'world';

echo $obj[0] . ' ' . $obj[1];
```
```
hello world
```

## 실제 구현은?

```php
class FixedUnsignedIntegerArray implements Iterator, ArrayAccess, Countable
{
    const BINARY_FORMAT = 'I';
    private $position = 0;
    private $length = 0;
    private $binaryLength = 0;
    private $data = null;
    public function __construct($length)
    
    {
        $null = $this->convertBinary(null);
        $this->position = 0;
        $this->length = $length;
        // because binary length is machine dependency
        $this->binaryLength = strlen($null);
        $this->data = str_repeat($null, $length);
    }
    
    private function getEntry($position)
    {
        $position = $position * $this->binaryLength;
        $unpack = unpack(self::BINARY_FORMAT, substr($this->data, $position, $this->binaryLength));
        $result = false !== $unpack ? $unpack[1] : null;
        if (0 === $result) {
            return null;
        } else {
            return $result - 1;
        }
    }
    
    private function setEntry($position, $value)
    {
        $position = $position * $this->binaryLength;
        $this->data = substr_replace($this->data, $this->convertBinary($value), $position, $this->binaryLength);
    }
    
    private function convertBinary($value)
    {
        if (null !== $value) {
            $value = $value + 1;
        }
        return pack(self::BINARY_FORMAT, $value);
    }
    
    public function rewind()
    {
        $this->position = 0;
    }
    
    public function current()
    {
        return $this->getEntry($this->position);
    }
    
    public function key()
    {
        return $this->position;
    }
    
    public function next()
    {
        $this->position = $this->position + 1;
    }

    public function valid()
    {
        return $this->offsetExists($this->position);
    }
    
    public function offsetSet($offset, $value)
    {
        if (is_integer($offset) && $offset < $this->length && $offset >= 0) {
            $this->setEntry($offset, $value);
        } else {
            throw new InvalidArgumentException('overflow array offset');
        }
    }
    
    public function offsetExists($offset)
    {
        return ($offset < $this->length && $offset >= 0);
    }
    
    public function offsetUnset($offset)
    {
        $this->setEntry($offset, null);
    }
    
    public function offsetGet($offset)
    {
        return $this->getEntry($offset);
    }
    
    public function count()
    {
        return $this->length;
    }
}
```

[pack](https://www.php.net/manual/en/function.pack.php) 함수의 설명을 다시 보자면,  환경에 따라 다른 byte 수가 나올 수 있습니다.
> unsigned integer (machine dependent size and byte order)

따라서 integer byte 수를 저희가 알고 있어야 잘못된 주소를 참조하지 않도록 하겠습니다.  
실제 구현에서는 [Iterator](https://www.php.net/manual/en/class.arrayaccess.php), [ArrayAccess](https://www.php.net/manual/en/class.arrayaccess.php), [Countable](https://www.php.net/manual/en/class.countable.php) 을 구현하여 좀 더 효용성을 높였습니다.

## 성능 테스트!
이제 구현했으니, 성능을 테스트 해보겠습니다.

테스트는 다음과 같이 진행하였습니다.
1. 10000 크기의 고정 배열을 만들어서 랜덤으로 데이터를 넣었을때 메모리 사용량을 체크 하였습니다.
2. 100번씩 반복문을 실행하여, 실행 시간의 평균을 체크 하였습니다.

```php
define('ARRAY_LENGTH', 10000);

$start_memory = memory_get_usage();
$vanilla = [];
for ($i = 0; $i < ARRAY_LENGTH; $i++) {
    $vanilla[] = mt_rand(0, 255);
}

$end_memory = memory_get_usage() - $start_memory;
echo "legacy array memory usage : {$end_memory}\n";

$start = microtime(true);
$result = 0;
for ($i = 0; $i < ARRAY_LENGTH; $i++) {
    $result += $vanilla[$i];
}

$time_elapsed_secs = microtime(true) - $start;
echo "for iterator legacy array : {$time_elapsed_secs}\n";

$start_memory = memory_get_usage();
$entries = new \Monoless\Arrays\FixedUnsignedIntegerArray(ARRAY_LENGTH);
for ($i = 0; $i < ARRAY_LENGTH; $i++) {
    $entries[$i] = mt_rand(0, 255);
}

$end_memory = memory_get_usage() - $start_memory;
echo "fixed unsigned integer array memory usage : {$end_memory}\n";

$time_total = 0;
$interval = 100;
for ($j = 0; $j < $interval; $j++) {
    $start = microtime(true);

    $result = 0;
    for ($i = 0; $i < ARRAY_LENGTH; $i++) {
        $result += $entries[$i];
    }

    $time_elapsed_secs = microtime(true) - $start;
    $time_total += $time_elapsed_secs;
}

$time_average = $time_total / $interval;
echo "iterator speed average : {$time_average}\n";
```

```
legacy array memory usage : 528440
for iterator legacy array : 0.00035810470581055
fixed unsigned integer array memory usage : 41072
iterator speed average : 0.0068418407440186
```

메모리 사용량은 약 90% 개선이 되었으나 속도는 기존 배열을 이길 수가 없었네요...

![오늘도 망했어요...](https://jjalbang.today/files/jjalboxthumb/2018/01/102_6895.jpg)

역시 기존껄 쓰는거 좋은거라고 배우고... 오늘도 맥주와 치킨을 시키며 잡니다.  
그래도 제 삽질이 좋다면, [github](https://github.com/q_lazzarus/arrays) 와 이 [패키지](https://packagist.org/packages/q_lazzarus/arrays)를 참고하세요

## 후일담...

혹시나 싶어서 php://memory 에 fwrite / fseek 를 구현해봤는데 더 느렸습니다...
```
for iterator legacy array : 0.0013089179992676
string speed average : 0.0084279131889343
php://memory speed average : 0.0098107981681824
```

### 참조 사이트
- https://www.php.net/manual/en/language.types.array.php
- https://www.geeksforgeeks.org/array-data-structure/
- https://www.geeksforgeeks.org/c-data-types/