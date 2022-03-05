Laravel 从 Illuminate\Foundation\Application 类开始启动，它就继承自 Illuminate\Container\Container。


我认为它有如下作用：
### 1. 接口和接口实现类的绑定，用于解耦，参数注入，也方便单元测试  mock
---

在代码里有很多类似的接口绑定，比如在 bootstrap/app.php 文件里有如下代码：
```php
$app->singleton(
    Illuminate\Contracts\Http\Kernel::class,
    App\Http\Kernel::class
);
```
在 public/index.php 文件里可以发现有下面这个代码：
```php
$kernel = $app->make(Kernel::class);
```
注意到这里的 Kernel::class 使用的是一个接口，而不是一个具体的实现。

整个应用里面都是通过这种 bind，make 的统一方法完成类的初始化。控制器里构造函数或者方法也都可以依赖这种方式注入接口的实现，而不是引用具体的实现类。


比如我定义一个如下的接口，并且有个它的实现类：
```php
// ITest.php
namespace App\Http\Controllers;

Interface ITest
{
    public function printHello();
}

// TestA.php
namespace App\Http\Controllers;

class TestA implements ITest
{
    public function printHello()
    {
        echo "hello_a";
    }
}

```

然后我通过如下代码 bind 

```php
$app->bind(
    \App\Http\Controllers\ITest::class,
    \App\Http\Controllers\TestA::class
);
```

最后，我在控制器里可以直接这样用
```php
class TestController extends Controller
{
    public function show(ITest $test)
    {
        $test->printHello();
    }
}
```
能正确的在网页上输出 hello_a 。它也能注入到控制器的构造函数里。

当然不调用 bind ，也能注入 TestA ，但是定义方法参数的时候不能用接口，只能用具体的类来声明参数。

### 2. 更方便的全局调用以及单例管理
---
我觉得这个功能也是更彻底的解耦合，比如，只通过 app('files') 就能使用到 \Illuminate\Filesystem\Filesystem 这个具体类，而不是在代码里强依赖这个实现。而且 files 绑定的时候是用的 singleton 方法，所以无论你在哪里调用 app('files') 得到的都是同一个实例。
```php
$app->singleton(
    'test',
    \App\Http\Controllers\TestA::class
);
```
然后在控制器里可以这么使用
```php
class TestController extends Controller
{
    public function show()
    {
        $test = app('test');
        $test->printHello();
    }
}
```


### 其它一些方便的特性
---
* 不用进行 bind ，也可以用 make 去初始化任意类，如果之前 bind 过的类，还能帮你自动注入进去

```php
namespace App\Http\Controllers;

use Illuminate\Contracts\Foundation\Application;

class TestA
{
    protected Application $app;

    public function __construct(Application $app)
    {
        $this->app = $app;
    }
}
```

然后用容器 make 的方式去初始化类
```php
    public function show()
    {
        $test = app(TestA::class);
    }
```

* 一个类可能实现了多个接口，可以用 alias 来绑定多个接口

比如可以 Illuminate\Foundation\Application 类里可以看到 registerCoreContainerAliases 这个方法

拿其中 app 这个来说
```php
'app' => [self::class, \Illuminate\Contracts\Container\Container::class, \Illuminate\Contracts\Foundation\Application::class, \Psr\Container\ContainerInterface::class],

```

如果我定义的参数是 \Illuminate\Contracts\Container\Container 或者 \Illuminate\Contracts\Foundation\Application 等，注入的实例都将是 Illuminate\Foundation\Application