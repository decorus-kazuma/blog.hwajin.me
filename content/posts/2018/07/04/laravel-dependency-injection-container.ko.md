+++
title = "라라벨의 의존성 주입 컨테이너"
date = "2018-07-04"
author = "Hwajin Lee"
cover = "https://private-user-images.githubusercontent.com/8151366/369705124-c6c4d0f1-940f-43c1-85e2-abf278445276.png?jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3MjcwMjUzMTYsIm5iZiI6MTcyNzAyNTAxNiwicGF0aCI6Ii84MTUxMzY2LzM2OTcwNTEyNC1jNmM0ZDBmMS05NDBmLTQzYzEtODVlMi1hYmYyNzg0NDUyNzYucG5nP1gtQW16LUFsZ29yaXRobT1BV1M0LUhNQUMtU0hBMjU2JlgtQW16LUNyZWRlbnRpYWw9QUtJQVZDT0RZTFNBNTNQUUs0WkElMkYyMDI0MDkyMiUyRnVzLWVhc3QtMSUyRnMzJTJGYXdzNF9yZXF1ZXN0JlgtQW16LURhdGU9MjAyNDA5MjJUMTcxMDE2WiZYLUFtei1FeHBpcmVzPTMwMCZYLUFtei1TaWduYXR1cmU9ZDI0MjZkZmI4MjlhOTliY2Q5ZTczNGU3MzgxZWZjZjIxZDM3MWY0NmY5Y2EwMjc0ZDllOWUwMmI5Y2E1MGNmYyZYLUFtei1TaWduZWRIZWFkZXJzPWhvc3QifQ.uM1QZPLqXs1MYDo2CP2i1C4b0EifBXeC-lrpRnAfrr0"
description = "Laravel Framework 의 근간인 Dependency Injection 을 담당하는 IoC Container 를 살펴봅니다."
+++
[Laravel](https://laravel.kr/) 의 제어의 역전(Inversion of Control) / 의존성 주입 (Dependency Injection) 컨테이너는 매우 강력한 기능입니다. 안타깝게도, 라라벨의 [공식 문서](https://laravel.kr/docs/5.6/)는 이 기능의 모든 면을 설명하고 있지 않습니다. 그런 이유로 저는 직접 이 기능들을 실험하여 본 문서를 작성했습니다. 이 문서는 [Laravel 5.4.26](https://github.com/laravel/framework/tree/5.4/src/Illuminate/Container) 버전을 기준으로 작성되었으며, 그 외 버전은 기능이 다를 수 있습니다. **이 문서는 번역되었습니다.** _번역한 문서는 [이곳](https://gist.github.com/davejamesmiller/bd857d9b0ac895df7604dd2e63b23afe)을 클릭하시면 이동하실 수 있습니다._


## 의존성 주입이란

이 문서에서는 의존성 주입과 제어의 역전 원칙에 대해 설명하지 않습니다. 이것들에 대해 익숙하지 않으시다면 [What is Dependency Injection?](http://fabien.potencier.org/what-is-dependency-injection.html) by Fabien Potencier ([Symfony](http://symfony.com/) framework 의 메인테이너) 가 작성한 글을 참고하실 수 있습니다.


## 컨테이너 접근

라라벨의 컨테이너 인스턴스에 접근하는 방법은 여러가지가 있습니다. 그러나 가장 간단한 방법은 `app()` 헬퍼 메소드를 호출하는 것입니다.

```php
$container = app();
```

이 시간에는 다른 방법에 대해서는 따로 서술하지 않겠습니다. 그 대신, 컨테이너 클래스에 초점을 맞추어 보도록 하겠습니다.

**Note:** 공식 문서에서는 `$container` 대신 `$this->app` 을 사용할 것입니다.

(* 라라벨 어플리케이션에서 이것은 실제로 Application 이라는 Container 의 서브 클래스 입니다. 그러나 이 글에서는 Container 메소드에 대해서만 설명할 것입니다.)



### 외부에서 Illuminate\Container 를 사용

라라벨 외부에서 컨테이너를 사용하고 싶으신 경우, [이것](https://packagist.org/packages/illuminate/container)을 설치하신 후, 아래와 같이 사용하실 수 있습니다.

```php
use Illuminate\Container\Container;

$container = Container::getInstance();
```



### 기본적인 사용법

가장 간단한 방법은 의존성 주입을 원하는 클래스의 생성자에 타입 힌트를 사용하는 것입니다

```php
class MyClass
{
    /**
     * @var AnotherClass
     */
    private $dependency;

    public function __construct(AnotherClass $dependency)
    {
        $this->dependency = $dependency;
    }
}
```



이후 `new MyClass` 를 사용하는 대신, 컨테이너의 `make()` 메소드를 사용합니다

```php
$instance = $container->make(MyClass::class);
```



컨테이너는 자동으로 의존성을 인스턴스화 할 것이며 이것은 실질적으로 아래와 동일합니다

```php
$instance = new MyClass(new AnotherClass());
```

`AnotherClass` 가 다른 의존성을 가진 경우 컨테이너는 재귀적으로 의존성을 해결할 것입니다.



### 실용적인 예제

여기 조금 더 실용적인 예제가 있습니다. (based on [PHP-DI docs](http://php-di.org/doc/getting-started.html))가 있습니다. - 회원 가입 기능에서 메일 기능을 분리하는 예제

```php
class Mailer
{
    public function mail($recipient, $content)
    {
        // Send an email to the recipient
        // ...
    }
}
```
```php
class UserManager
{
    private $mailer;

    public function __construct(Mailer $mailer)
    {
        $this->mailer = $mailer;
    }

    public function register($email, $password)
    {
        // Create the user account
        // ...

        // Send the user an email to say hello!
        $this->mailer->mail($email, 'Hello and welcome!');
    }
}
```
```php
use Illuminate\Container\Container;

$container = Container::getInstance();

$userManager = $container->make(UserManager::class);
$userManager->register('dave@davejamesmiller.com', 'MySuperSecurePassword!');
```


## 구현 객체에 인터페이스를 바인딩 하는 법

컨테이너를 이용하면 인터페이스와 구현체 (Impl)를 런타임시 쉽게 인스턴스화 할 수 있습니다

```php
interface MyInterface { /* ... */ }
interface AnotherInterface { /* ... */ }
```
이후, 이 인터페이스들을 구현하는 클래스를 작성합니다

```php
class MyClass implements MyInterface
{
    private $dependency;

    public function __construct(AnotherInterface $dependency)
    {
        $this->dependency = $dependency;
    }
}
```
그리고 `bind()` 메소드를 사용하여 각 인터페이스를 구현체 (class)에 매핑합니다

```php
$container->bind(MyInterface::class, MyClass::class);
$container->bind(AnotherInterface::class, AnotherClass::class);
```
마지막으로, `make()` 메소드에 클래스 이름이 아닌 인터페이스 이름을 전달합니다

```php
$instance = $container->make(MyInterface::class);
```
**Note:** 만약 인터페이스 바인딩을 잊으셨다면 치명적인 오류가 발생합니다.

```
Fatal error: Uncaught ReflectionException: Class MyInterface does not exist
```
왜냐하면 컨테이너는 `new MyInterface`, 즉 인터페이스를 인스턴스화 하고자 할 것이기 때문입니다. 이것은 올바른 클래스가 아니죠.

### 실용적인 예제

변경 가능한 캐시 레이어를 구성해보겠습니다.

```php
interface Cache
{
    public function get($key);
    public function put($key, $value);
}
```
```php
class RedisCache implements Cache
{
    public function get($key) { /* ... */ }
    public function put($key, $value) { /* ... */ }
}
```
```php
class Worker
{
    private $cache;

    public function __construct(Cache $cache)
    {
        $this->cache = $cache;
    }

    public function result()
    {
        // Use the cache for something...
        $result = $this->cache->get('worker');

        if ($result === null) {
            $result = do_something_slow();

            $this->cache->put('worker', $result);
        }

        return $result;
    }
}
```
```php
use Illuminate\Container\Container;

$container = Container::getInstance();
$container->bind(Cache::class, RedisCache::class);

$result = $container->make(Worker::class)->result();
 ```



## 추상 & 구상 클래스 바인딩

추상 클래스(abstract class) 또한 바인딩 할 수 있습니다

```php
$container->bind(MyAbstract::class, MyConcreteClass::class);
```

또는 추상 클래스를 서브 클래스로 대체할 수 있습니다

```php
$container->bind(MySQLDatabase::class, CustomMySQLDatabase::class);
```


## 커스텀 바인딩

만약 클래스가 추가적인 설정을 요구한다면 클래스의 이름 대신 `closure` 를 `bind()` 메소드의 두 번째 매개 변수로 넘길 수 있습니다.

```php
$container->bind(Database::class, function (Container $container) {
    return new MySQLDatabase(MYSQL_HOST, MYSQL_PORT, MYSQL_USER, MYSQL_PASS);
});
```

매 번 데이터베이스 인터페이스가 필요할 때 마다 새로운 MySQLDatabase 인스턴스가 생성되어 사용되며, 몇 가지 미리 지정된 구성값을 필요로 합니다. (단일 인스턴스를 공유하고자 한다면, 아래 _싱글톤_ 를 읽어주시기 바랍니다.)

클로저는 컨테이너 인스턴스를 첫 번째 매개 변수로 받으며, 필요한 경우 다른 클래스를 인스턴스화 하는 데 사용하실 수 있습니다

```php
$container->bind(Logger::class, function (Container $container) {
    $filesystem = $container->make(Filesystem::class);

    return new FileLogger($filesystem, 'logs/error.log');
});
```

클로저는 구상 클래스가 인스턴스화 하는 법을 따로 지정할 수도 있습니다

```php
$container->bind(GitHub\Client::class, function (Container $container) {
    $client = new GitHub\Client;
    $client->setEnterpriseUrl(GITHUB_HOST);
    return $client;
});
```



### 콜백 해결 [(컨테이너 이벤트)](https://laravel.kr/docs/5.6/container#container-events)

바인딩을 완벽하게 오버라이딩 하는 대신, `resolving()` 메소드를 이용하여 객체의 의존성을 해결할 때 콜백을 받으실 수 있습니다

```php
$container->resolving(GitHub\Client::class, function ($client, Container $container) {
    $client->setEnterpriseUrl(GITHUB_HOST);
});
```

콜백이 여러 개 존재한다면, 모든 콜백이 호출됩니다. 이 기능은 인터페이스나 추상 클래스에도 동일하게 동작합니다

```php
$container->resolving(Logger::class, function (Logger $logger) {
    $logger->setLevel('debug');
});

$container->resolving(FileLogger::class, function (FileLogger $logger) {
    $logger->setFilename('logs/debug.log');
});

$container->bind(Logger::class, FileLogger::class);

$logger = $container->make(Logger::class);
```

또한 모든 의존성 해결에 있어서 콜백을 받을 수도 있습니다 (제 생각에는 Logging / Debugging 에만 유용할 것 같습니다)

```php
$container->resolving(function ($object, Container $container) {
    // ...
});
```



### 클래스 확장 [(바인딩 확장)](https://laravel.kr/docs/5.6/container#extending-bindings)

`extend()` 메소드를 사용하여 클래스를 감싸고 다른 객체로 반환시킬 수 있습니다

```php
$container->extend(APIClient::class, function ($client, Container $container) {
    return new APIClientDecorator($client);
});
```



반환되는 객체는 여전히 같은 인터페이스를 구현해야 합니다. 그렇지 않으면 Type hinting 에서 에러가 발생할 것입니다.



## 싱글톤

자동 바인딩과 `bind()` 메소드를 이용하면 필요할 때마다 새로운 인스턴스가 생성됩니다 (또는 클로저가 호출됩니다). 하나의 인스턴스를 공유하고자 한다면 `bind()` 대신 `singleton()` 을 사용할 수 있습니다

```php
$container->singleton(Cache::class, RedisCache::class);
```

또는 클로저를 사용할 수도 있습니다

```php
$container->singleton(Database::class, function (Container $container) {
    return new MySQLDatabase('localhost', 'testdb', 'user', 'pass');
});
```

구상 클래스를 싱글톤으로 생성하고자 한다면, 두 번째 매개 변수를 제외하고 전달합니다

```php
$container->singleton(MySQLDatabase::class);
```



이 경우, 싱글톤 객체는 필요할 때 단 한 번 생성될 것입니다. 그리고 이후에는 이것을 재사용 할 것입니다. 재사용을 원하는 인스턴스가 이미 존재하는 경우, `instance()` 메소드를 사용하시기 바랍니다.

```php
$container->instance(Container::class, $container);
```


## 임의의 바인딩 이름

클래스나 인터페이스 이름 대신 임의의 문자열을 사용할 수 있습니다. 타입 힌트는 사용할 수 없으며, `make()` 메소드를 이용해야 합니다

```php
$container->bind('database', MySQLDatabase::class);

$db = $container->make('database');
```

인터페이스와 클래스 모두 짧은 이름 기능을 지원하려면 `alias()` 메소드를 사용해야 합니다

```php
$container->singleton(Cache::class, RedisCache::class);
$container->alias(Cache::class, 'cache');

$cache1 = $container->make(Cache::class);
$cache2 = $container->make('cache');

assert($cache1 === $cache2);
```



## 임의의 값 저장

또한 컨테이너를 임의의 값을 저장하기 위해 사용할 수도 있습니다 (e.g. 설정 데이터)

```php
$container->instance('database.name', 'testdb');

$db_name = $container->make('database.name');
```

이 방법은 배열 접근을 지원하고 있습니다. 조금 더 자연스러운 방식이죠

```php
$container['database.name'] = 'testdb';

$db_name = $container['database.name'];
```

클로저 바인딩과 같이 사용하면 왜 이것이 유용할 수 있는지 알 수 있습니다

```php
$container->singleton('database', function (Container $container) {
    return new MySQLDatabase(
        $container['database.host'],
        $container['database.name'],
        $container['database.user'],
        $container['database.pass']
    );
});
```

라라벨 자체는 컨테이너를 설정(구성)을 위해 사용하지 않습니다. 설정 파일은 [Config](https://laravel.kr/docs/5.6/configuration) 로 분리되어 있고 설정에 대한 부분은 [`Illuminate\Config\Repository`](https://github.com/laravel/framework/blob/5.6/src/Illuminate/Config/Repository.php) 클래스에 존재합니다. 그러나 [PHP-DI](http://php-di.org/doc/php-definitions.html#values) 는 아닙니다.

**Tip:** 배열 구문은 `make()` 메소드 대신 객체를 인스턴스화 할 때 사용할 수 있습니다

```php
$db = $container['database'];
```



## 함수와 메소드의 의존성 주입

우리는 지금까지 생성자를 통한 의존성 주입을 보았습니다. 그러나 라라벨은 함수에도 의존성 주입 기능을 제공합니다

```php
function do_something(Cache $cache) { /* ... */ }

$result = $container->call('do_something');
```

추가적인 매개 변수는 순서 또는 연관 배열로 전달할 수 있습니다

```php
function show_product(Cache $cache, $id, $tab = 'details') { /* ... */ }

// show_product($cache, 1)
$container->call('show_product', [1]);
$container->call('show_product', ['id' => 1]);

// show_product($cache, 1, 'spec')
$container->call('show_product', [1, 'spec']);
$container->call('show_product', ['id' => 1, 'tab' => 'spec']);
```

이 기능은 Callable 메소드에 모두 적용할 수 있습니다.

#### Closures

```php
$closure = function (Cache $cache) { /* ... */ };

$container->call($closure);
```

#### Static methods

```php
class SomeClass
{
    public static function staticMethod(Cache $cache) { /* ... */ }
}
```

```php
$container->call(['SomeClass', 'staticMethod']);
// or:
$container->call('SomeClass::staticMethod');
```

#### Instance methods

```php
class PostController
{
    public function index(Cache $cache) { /* ... */ }
    public function show(Cache $cache, $id) { /* ... */ }
}
```

```php
$controller = $container->make(PostController::class);

$container->call([$controller, 'index']);
$container->call([$controller, 'show'], ['id' => 1]);
```



### 인스턴스 메소드 호출을 위한 손쉬운 방법

클래스를 인스턴스화하고 바로 메소드를 호출하는 빠른 방법이 있습니다. `ClassName@methodName` 을 사용하는 것입니다

```php
$container->call('PostController@index');
$container->call('PostController@show', ['id' => 4]);
```

컨테이너는 클래스를 인스턴스화하는 것에 이 기능을 사용합니다. 이것은 다음을 의미합니다

- 의존성은 생성자를 통해 주입됩니다
- 클래스를 재사용하고자 할 때 싱글톤으로 정의할 수 있습니다
- 구상 클래스 대신 인터페이스 또는 임의의 이름을 사용할 수 있습니다

예를 들어, 아래와 같은 경우가 있습니다

```php
class PostController
{
    public function __construct(Request $request) { /* ... */ }
    public function index(Cache $cache) { /* ... */ }
}
```

```php
$container->singleton('post', PostController::class);
$container->call('post@index');
```

마지막으로, 세 번째 매개 변수로 "기본 메소드"를 전달할 수 있습니다. 만약 첫번째 매개 변수가 메소드가 따로 지정되어 있지 않은 클래스 이름인 경우, 기본 메소드가 대신 호출됩니다. 라라벨은 이 기능을 이용하여 [이벤트 핸들러](https://laravel.kr/docs/5.6/events#registering-events-and-listeners)를 구현합니다

```php
$container->call(MyEventHandler::class, $parameters, 'handle');

// 위와 같음
$container->call('MyEventHandler@handle', $parameters);
```



### 메소드 호출 바인딩

`bindMethod()` 메소드는 메소드 호출을 오버라이드 할 수 있습니다. 예를 들어, 추가적인 매개 변수를 아래와 같이 전달할 수 있습니다

```php
$container->bindMethod('PostController@index', function ($controller, $container) {
    $posts = get_posts(...);

    return $controller->index($posts);
});
```

이 기능은 기존의 메소드 대신 클로저를 호출하여 동작합니다

```php
$container->call('PostController@index');
$container->call('PostController', [], 'index');
$container->call([new PostController, 'index']);
```

그러나 `call()` 메소드에서 추가적인 매개 변수를 전달할 수는 없습니다

```php
// 사용할 수 없음
$container->call('PostController@index', ['Not used :-(']);
```



_**Notes:** 이 메소드는 [컨테이너 인터페이스](https://github.com/laravel/framework/blob/5.6/src/Illuminate/Contracts/Container/Container.php)에 존재하지 않습니다. [컨테이너 클래스](https://github.com/laravel/framework/blob/5.6/src/Illuminate/Container/Container.php)에만 존재합니다. [왜 매개 변수를 무시하는지 이유는 이곳에서 확인하실 수 있습니다](https://github.com/laravel/framework/pull/16800)_



## 문맥에 따른 조건적 바인딩

가끔 여러분은 각각의 클래스마다 다른 구현 객체를 전달하고자 할 수도 있습니다. 다음은 [라라벨 공식 문서](https://laravel.kr/docs/5.6/container#contextual-binding)에서 일부 수정된 예제입니다

```php
$container
    ->when(PhotoController::class)
    ->needs(Filesystem::class)
    ->give(LocalFilesystem::class);

$container
    ->when(VideoController::class)
    ->needs(Filesystem::class)
    ->give(S3Filesystem::class);
```

`PhotoController` 와 `VideoController` 는 `Filesystem` 인터페이스에 의존적일 수 있지만, 서로 다른 구현체를 받게 됩니다. `bind()` 와 같이 `give()` 에서도 클로저를 사용할 수 있습니다

```php
$container
    ->when(VideoController::class)
    ->needs(Filesystem::class)
    ->give(function () {
        return Storage::disk('s3');
    });
```

또는 이름(문자열)을 통해 의존성을 주입할 수도 있습니다

```php
$container->instance('s3', $s3Filesystem);

$container
    ->when(VideoController::class)
    ->needs(Filesystem::class)
    ->give('s3');
```



### 기본 타입 바인딩

여러분은 또한 기본 타입들(string, integer..) 를 `needs()` 메소드를 통해 주입할 수 있습니다 (인터페이스 대신). 그리고 `give()` 메소드를 통해 값들을 전달할 수 있습니다

```php
$container
    ->when(MySQLDatabase::class)
    ->needs('$username')
    ->give(DB_USER);
```

클로저를 통해 값이 필요할 때 까지 값을 가지고 오는 것을 늦출 수 있습니다

```php
$container
    ->when(MySQLDatabase::class)
    ->needs('$username')
    ->give(function () {
        return config('database.user');
    });
```

여기서 여러분은 클래스나 또는 의존성 이름을 사용할 수 없습니다 (e.g. `give('database.user')`). 왜냐하면, 이 값은 리터럴 값으로 반환되기 때문입니다. 대신 클로저를 사용할 수 있습니다

```php
$container
    ->when(MySQLDatabase::class)
    ->needs('$username')
    ->give(function (Container $container) {
        return $container['database.user'];
    });
```



## 태깅 [(Tagging)](https://laravel.kr/docs/5.6/container#tagging)

컨테이너를 사용하여 관련된 바인딩을 "태그"할 수 있습니다

```php
$container->tag(MyPlugin::class, 'plugin');
$container->tag(AnotherPlugin::class, 'plugin');
```

이후 태그가 지정된 모든 인스턴스를 배열로 받아올 수 있습니다

```php
foreach ($container->tagged('plugin') as $plugin) {
    $plugin->init();
}
```

`tag()`  메소드의 매개 변수는 배열도 사용할 수 있습니다.

```php
$container->tag([MyPlugin::class, AnotherPlugin::class], 'plugin');
$container->tag(MyPlugin::class, ['plugin', 'plugin.admin']);
```



## 재바인딩 (Rebinding)

_**Note:** 이 기능은 조금 더 고급 기능이며, 대부분의 경우 이 기능을 필요로 하지 않습니다. 원하신다면 이 부분을 넘기셔도 괜찮습니다!_

바인딩 또는 인스턴스가 사용된 후 변경될 때 `rebinding()` 콜백이 호출됩니다. 예를 들어 세션 클래스가 Auth 클래스에 의해 사용된 후 변경된다면, Auth 클래스는 변경 사실을 알아야 합니다

```php
$container->singleton(Auth::class, function (Container $container) {
    $auth = new Auth;
    $auth->setSession($container->make(Session::class));

    $container->rebinding(Session::class, function ($container, $session) use ($auth) {
        $auth->setSession($session);
    });

    return $auth;
});

$container->instance(Session::class, new Session(['username' => 'dave']));
$auth = $container->make(Auth::class);

echo $auth->username(); // dave
$container->instance(Session::class, new Session(['username' => 'danny']));

echo $auth->username(); // danny
```

재바인딩에 대해 더 자세한 내용은 [이곳](https://stackoverflow.com/questions/38974593/laravels-ioc-container-rebinding-abstract-types)과 [이곳](https://code.tutsplus.com/tutorials/digging-in-to-laravels-ioc-container--cms-22167)을 참고해 주시기 바랍니다.

### refresh()

이 기능을 더 쉽게 사용하고자 할 때, `refresh()` 메소드를 사용할 수 있습니다. 이것은 아래와 같은 일반적인 패턴을 다루고자 할 때 사용합니다

```php
$container->singleton(Auth::class, function (Container $container) {
    $auth = new Auth;
    $auth->setSession($container->make(Session::class));

    $container->refresh(Session::class, $auth, 'setSession');

    return $auth;
});
```

또한 이 기능은 이미 존재하는 인스턴스나 바인딩을 반환하므로, 아래와 같이 코드를 작성할 수도 있습니다

```php
$container->singleton(Session::class);

$container->singleton(Auth::class, function (Container $container) {
    $auth = new Auth;
    $auth->setSession($container->refresh(Session::class, $auth, 'setSession'));
    return $auth;
});
```

(개인적으로 저는 이 구문이 더 혼란스럽고 위의 좀 더 자세한 방법을 선호합니다)

_**Notes:** 이 메소드는 [컨테이너 인터페이스](https://github.com/laravel/framework/blob/5.6/src/Illuminate/Contracts/Container/Container.php)에 존재하지 않습니다. [컨테이너 클래스](https://github.com/laravel/framework/blob/5.6/src/Illuminate/Container/Container.php)에만 존재합니다._



## 생성자 매개 변수 오버라이딩

`makeWith()` 메소드는 생성자 매개 변수에 추가적인 값을 전달할 수 있습니다. 기존에 존재하는 인스턴스나 싱글톤을 무시하고 종속성을 주입하면서 다른 매개 변수로 클래스의 인스턴스를 생성할 때 유용하게 사용될 수 있습니다

```php
class Post
{
    public function __construct(Database $db, int $id) { /* ... */ }
}
```

```php
$post1 = $container->makeWith(Post::class, ['id' => 1]);
$post2 = $container->makeWith(Post::class, ['id' => 2]);
```

_**Note:** 라라벨 5.3 이하에서는 간단하게 `make($class, $pameters)` 로 사용했었습니다. 라라벨 5.4에서 이 기능은 삭제되어 `makeWith() ` 로 추가되었습니다. 그러나, 라라벨 5.5 에서는 다시 라라벨 5.3과 같이 사용할 수 있습니다._



## 그 외 메소드

그 외 유용하다고 생각하는 모든 메소드를 다루고자 합니다. 이 내용으로 충분하지 않다면 아래 퍼블릭 메소드를 참고하실 수 있습니다.

### bound()

`bound()` 메소드는 `bind()`, `singleton()`, `instance()`, `alias()` 메소드에 의해 클래스 또는 이름(문자열)이 바운딩 되었을 때 `true` 를 반환합니다

```php
if (! $container->bound('database.user')) {
    // ...
}
```

배열 접근 구문과 `isset()`를 사용할 수도 있습니다.

```php
if (! isset($container['database.user'])) {
    // ...
}
```

`unset()` 함수를 통해 제거될 수 있으며, 지정된 바인딩 / 인스턴스 / 별칭을 제거합니다

```php
unset($container['database.user']);
var_dump($container->bound('database.user')); // false
```



### bindIf()

`bindIf()` 는 바인딩이 존재하지 않는 경우에만 바인딩을 하는 메소드 입니다. 동작 자체는 `bind()` 와 동일하게 동작합니다. (위의 `bound()` 참고)

이것은 잠재적으로 사용자의 오버라이드를 허용하여 패키지의 기본 바인딩을 등록하게끔 사용될 수 있습니다.

```php
$container->bindIf(Loader::class, FallbackLoader::class);
```

그러나 `singletonIf()` 메소드는 없습니다. 대신, `bindIf($abstract, $concrete, true)` 를 통해 동일한 기능을 사용할 수 있습니다

```php
$container->bindIf(Loader::class, FallbackLoader::class, true);
```

또는 아래와 같은 방법도 가능합니다

```php
if (! $container->bound(Loader::class)) {
    $container->singleton(Loader::class, FallbackLoader::class);
}
```



### resolved()

`resolved()` 메소드는 의존성 문제가 이전에 해결된 경우 `true` 를 반환합니다.

```php
var_dump($container->resolved(Database::class)); // false
$container->make(Database::class);
var_dump($container->resolved(Database::class)); // true
```

저는 이 기능이 유용할 것인지 확신하지 못하겠습니다.. `unset()` 을 사용하면 재설정이 이루어지는데 말이죠 (위의 `bound()` 참고)

```php
unset($container[Database::class]);
var_dump($container->resolved(Database::class)); // false
```



### factory()

`factory()` 메소드는 매개 변수가 존재하지 않고 `make()` 를 호출하는 클로저를 반환합니다

```php
$dbFactory = $container->factory(Database::class);

$db = $dbFactory();
```

이 기능이 언제 쓸모가 있을 지 모르겠네요.



### wrap()

`wrap()` 메소드는 클로저가 실행될 때 종속성이 주입되도록 클로저를 래핑합니다. wrap 메소드는 매개 변수 배열을 사용할 수 있으며 반환된 클로저에는 매개 변수가 없습니다.

```php
$cacheGetter = function (Cache $cache, $key) {
    return $cache->get($key);
};

$usernameGetter = $container->wrap($cacheGetter, ['username']);

$username = $usernameGetter();
```

이 기능이 언제 쓸모가 있을 지 모르겠네요. (2)

_**Note:** 이 메소드는 [컨테이너 인터페이스](https://github.com/laravel/framework/blob/5.6/src/Illuminate/Contracts/Container/Container.php)에 존재하지 않습니다. [컨테이너 클래스](https://github.com/laravel/framework/blob/5.6/src/Illuminate/Container/Container.php)에만 존재합니다._



### afterResolving()

`afterResolving()` 은 `resolving()` 콜백 후에 `afterResolving()` 콜백을 호출한다는 점을 제외하면 `resolving()` 과 동일합니다. 이 기능이 언제 쓸모가 있을 지 모르겠네요.



### 마지막으로..

- `isShared()` : 주어진 타입이 공유된 싱글톤인지 인스턴스인지 알아냅니다.
- `isAlias()` : 주어진 문자열이 등록된 별칭(alias)인지 알아냅니다.
- `hasMethodBinding()` : 컨테이너가 주어진 바인딩을 가지고 있는지 알아냅니다.
- `getBindings()` : 등록된 모든 바인딩을 원시 배열의 형태로 가져옵니다.
- `getAlias($abstract)` : 기본 클래스와 바인딩 이름에 대한 별칭을 해결(Resolves) 합니다.
- `forgetInstance($abstract)` : 단일 인스턴스 객체(싱글톤)를 지웁니다.
- `forgetInstances()` 모든 인스턴스 객체를 지웁니다.
- `flush()` : 모든 바인딩과 인스턴스를 지우고 컨테이너를 리셋합니다 (effectively resetting the container).
- `setInstance()` : `getInstance()` 에서 사용된 인스턴스를 대체합니다. (Tip: `setInstance(null)` 을 이용해서 이것을 지우면 다음에 새로운 인스턴스가 생성됩니다.)

_**Note:** 마지막 부분의 메소드들은 [컨테이너 인터페이스](https://github.com/laravel/framework/blob/5.6/src/Illuminate/Contracts/Container/Container.php)의 일부가 아닙니다._



---

- 이 문서의 원본은 2017년 06월 15일에 DaveJamesMiller.com 에서 포스팅되었습니다.
- 이 문서는 원작자의 허가를 받아 번역되었습니다. Dave <[dave@davejamesmiller.com](mailto:dave%40davejamesmiller.com)>. 감사합니다.
- 이 문서는 [Hwajin Lee <<brightdelusion@gmail.com>>](https://github.com/hwajin-me) 가 번역하였습니다.