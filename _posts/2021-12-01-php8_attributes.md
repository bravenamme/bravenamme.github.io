---
layout: post
title:  "php8 에 도입되는 annotation (attributes)"
date:   2021-12-01 10:00
author: q_lazzarus
tags:	[php, annotation]
---

## php8 에 도입되는 attributes
php 8 부터는 attrubutes 라는 기능을 사용할 수 있습니다. 다른 많은 언어에서는 annontation 이라고 불리는 것인데, 이 attributes (aka. annotation) 은 앞서, rman 님께서 작성해주신 [Annotation 이해하기](/2021/11/10/Annotation/) 을 참조해주세요.

(이미 php8 소개하는 많은 블로그들이 있었서 늦은감이 있네요...)

일단 여기서는 어떻게 사용하는지, 또 어떻게 커스텀 attributes 를 만드는지 등에 대해서 다뤄 보겠습니다.

## 개요
먼저 attribute 가 작성된 예제를 올려봅니다.

```php
use \Support\Attributes\ListensTo;

class ProductSubscriber
{
    #[ListensTo(ProductCreated::class)]
    public function onProductCreated(ProductCreated $event) { /* … */ }

    #[ListensTo(ProductDeleted::class)]
    public function onProductDeleted(ProductDeleted $event) { /* … */ }
}
```

뒷부분에 실제 사용하는 다른 코드를 보여드리겠습니다만. 이 코드가 attirbutes 를 가장 설명하기 좋은 예라고 생각합니다.

이 문법이 여러분들이 기대했던 것이 아닐 수도 있습니다. 보통 @ 혹은 @: /* */  같은 주석을 선호할 수 도 있습니다. 사실 이런 문법에 대한 논쟁들은 php rfc 에 대한 토론을 읽어보길 권장 드립니다. [PHP RFC: Shorter Attribute Syntax](https://externals.io/message/110640)

일단 코드에 집중 하도록 하겠습니다. 저 **ListensTo** 는 어떻게 작동할가요?

이제 보여 드릴 custom attributes 는 **#[Attributes]** 라는 attribute 가 선언된 간단한 클래스 입니다. 이 기본 attribute 는 원래 PhpAttribute 라고 RFC 에서 결정하였지만, 나중에 다른 토론을 통해 변경 되었습니다.  [PHP RFC: Attribute Amendments](https://wiki.php.net/rfc/attribute_amendments)

custom attribute 는 다음과 같이 작성됩니다.

```php
#[Attribute]
class ListensTo
{
    public string $event;

    public function __construct(string $event)
    {
        $this->event = $event;
    }
}
```

간단하죠? custom attribute 를 작성하신다면 목표를 한정해서 작성하세요. attribute 는 클래스와 메소드에 메타데이터를 추가하기 위한 것이며, 그 이상은 아닙니다.

예를 들어 parameter 입력 유효성 검사에 사용할 수 없습니다.

즉 attribute 내에서 메소드에 전달된 parameter 에는 접근할 수 없습니다.

원래는 이 동작을 허용하는 이전 RFC 가 있었지만 이후 논의를 통해 제외 되었습니다.

아까 event subscriber 예제를 다시 보자면, 여전히 meta data 를 읽고 어딘가의 subscriber 들에게 전달하여야 합니다. 

좀 더 내용을 추가하기 위해, 지루한 boilerplate 코드를 몇개 작성해봅시다.

```php
class EventServiceProvider extends ServiceProvider
{
    // In real life scenarios, 
    //  we'd automatically resolve and cache all subscribers
    //  instead of using a manual array.
    private array $subscribers = [
        ProductSubscriber::class,
    ];

    public function register(): void
    {
        // The event dispatcher is resolved from the container
        $eventDispatcher = $this->app->make(EventDispatcher::class);

        foreach ($this->subscribers as $subscriber) {
            // We'll resolve all listeners registered 
            //  in the subscriber class,
            //  and add them to the dispatcher.
            foreach (
                $this->resolveListeners($subscriber) 
                as [$event, $listener]
            ) {
                $eventDispatcher->listen($event, $listener);
            }       
        }       
    }
}
```

**[$event, $listener]** 문법이 친숙하지 않을 수 있겠지만, 코드 생산성을 높이기 문법입니다. (배열 구조 분해 / ES6 를 생각해보세요!)

이제 resolveListeners 를 한번 보실까요!

```php
private function resolveListeners(string $subscriberClass): array
{
    $reflectionClass = new ReflectionClass($subscriberClass);

    $listeners = [];

    foreach ($reflectionClass->getMethods() as $method) {
        $attributes = $method->getAttributes(ListensTo::class);
        
        foreach ($attributes as $attribute) {
            $listener = $attribute->newInstance();
            
            $listeners[] = [
                // The event that's configured on the attribute
                $listener->event,
    
                // The listener for this event 
                [$subscriberClass, $method->getName()],
            ];
        }
    }

    return $listeners;
}
```

**ReflectionMethod::getAttributes()** 를 통해 주석 문자열을 parsing 하는 것보다 메타 데이터를 쉽게 읽을 수 있는 것을 확인할 수 있습니다. 

> 역주 : php 8 이전에는 annotation 을 문법적으로 지원하지 못해, php 소스코드의 주석을 string 으로 parsing 해서 구현하였습니다. ㅎㄷㄷ [ReflectionProperty::getDocComment](https://www.php.net/manual/en/reflectionproperty.getdoccomment)

좀 어려운 포인트가 두가지 있는데요, 한번 정리해보겠습니다.

먼저 **$attribute->newInstance()** 호출이 있습니다. 이 코드는 실제로 우리가 작성한 custom attribute 를 인스턴스화되는 장소입니다. 우리가 작성한 subscriber 클래스의 attribute 에 나열된 매개 변수를 사용하여 생성자에 전달합니다.

즉, 기술적으로는 custom attribute 에 매개 변수를 전달할 필요가 없습니다. 물론 **$attribute->getArguments()** 를 직접 호출할 수 있습니다. 또한 클래스를 인스턴스화한다는 것은 원하는 방식으로 구문 분석 입력을 생성할 수 있다는 것을 의미합니다. 대체로 **newInstance()**를 사용하여 속성을 인스턴스화하는 것이 좋습니다.

두번째로 **ReflectionMethod::getAttributes()** 를 이용한 메쏘드의 모든 attributes 를 반환하는 함수의 사용입니다. 두가지 parameter 를 사용하여 반환값 필터링할 수 있습니다. 자세한건 아래 문서를 확인해주세요. [ReflectionFunctionAbstract::getAttributes](https://www.php.net/manual/en/reflectionfunctionabstract.getattributes.php)

이 필터링을 이해하려면 먼저 attributes 에 대해서 알아야 할 것이 한 가지 더 있습니다. method 뿐만 아니라 class, property 또는 constant에 여러 attribute 을 추가할 수 있다는 것입니다.

아래는 class 에 선언된 예제입니다.

```php
#[Route(Http::POST, '/products/create'), Autowire,]
class ProductsCreateController
{
    public function __invoke() { /* … */ }
}
```

이를 염두에 두시고 **Reflection::getAttributes()** 가 배열을 반환하는 이유가 명확하므로 결과를 필터링하는 방법을 살펴보겠습니다.

여기에서는 controller 의 route 를 parsing 하는것에 촛점을 맞추고
Route attribute 에 대해서만 관심을 가지도록 하겠습니다. 해당 클래스를 필터로 쉽게 전달 할 수 있습니다.

```php
$attributes = $reflectionClass->getAttributes(Route::class);
```

두번째 parameter는 필터링이 수행되는 방식을 변경합니다. 지정된 인터페이스를 구현하는 모든 속성을 반환하도록 하는  ReflectionAttribute::IS_INSTANCEOF 를 전달할 수 있습니다.

예를 들어 여러 attribute 에 의존하는 컨테이너 정의를 구문을 parsing 한다고 가정한다면...

```php
$attributes = $reflectionClass->getAttributes(
    ContainerAttribute::class, 
    ReflectionAttribute::IS_INSTANCEOF
);
```

좀 더 풀어서 쉽게 설명드리자면... class 아래의 모든 하위 method, property, constants 의 attribute 를 가져올때 선언하면 됩니다.

```php
$r_atts = $rc->getAttributes(SomeAttribute::class, 0); // 0 is default, just given class
echo json_encode(array_map(fn(ReflectionAttribute $r_att) => $r_att->getName(), $r_atts)), PHP_EOL;

$r_atts = $rc->getAttributes(SomeAttribute::class, 2); // given class and children classes
echo json_encode(array_map(fn(ReflectionAttribute $r_att) => $r_att->getName(), $r_atts)), PHP_EOL;
```

## Real World

그렇다면 실제 활용할 수 있는 예제는 무엇일까요?

laravel 을 예를 들면, php7 까지의 laravel 에서 route 를 등록하기 위해 route 에 대한 설정값이 존재하였습니다. [라우팅](https://laravel.kr/docs/8.x/routing)

routes 라는 디렉토리의 web.php / api.php 에 연결할 컨트롤러를 등록하는 형식이었죠.
아래와 같이요.

```php
Route::get('user/profile', [UserProfileController::class, 'show'])->name('profile');
```

하지만 php attributes 에 대한 도입으로 이런 설정 파일을 따로 작성할 필요가 없어졌습니다. [laravel-route-attributes](https://github.com/spatie/laravel-route-attributes)

```php
use Spatie\RouteAttributes\Attributes\Get;

class UserProfileController
{
    #[Get('user/profile')]
    public function show()
    {

    }
}
```

설정 파일을 통하지 않고, 컨트롤러에 attribute 작성하는 것 만으로도 충분하게 되었죠.

이상입니다.

Ref.
* [PHP 8: Attributes](https://stitcher.io/blog/attributes-in-php-8)
* [laravel-route-attributes](https://github.com/spatie/laravel-route-attributes)
* [PHP: PHP 8 기능 정리 및 요약 - 개발자 정상우](https://pronist.tistory.com/60)
* [PHP 소식 - Attribute](https://velog.io/@qroffle/PHP-%EC%86%8C%EC%8B%9D-Attribute)