对 rebinding 方法的作用不太理解，所以在网上搜到了几篇有用的文章：

* [Laravel 核心——服务容器的细节特性](https://learnku.com/articles/4870/the-laravel-core-the-details-of-the-service-container)
* [Laravel Container (容器) 深入理解 (下)](https://blog.csdn.net/andong154564667/article/details/80737928)
* [Digging in to Laravel's IoC Container](https://code.tutsplus.com/tutorials/digging-in-to-laravels-ioc-container--cms-22167)

rebinding 方法更像是定义一个事件，对某个 abstruct 发生重新绑定时触发的一个事件。可以通过官方的 unit test 看出来：

```php
public function testReboundListeners()
{
    unset($_SERVER['__test.rebind']);
    $container = new Container;
    $container->bind('foo', function () {
        //
    });
    $container->rebinding('foo', function () {
        $_SERVER['__test.rebind'] = true;
    });
    $container->bind('foo', function () {
        //
    });
    $this->assertTrue($_SERVER['__test.rebind']);
}
```
它的作用就像上面[这篇文章](https://code.tutsplus.com/tutorials/digging-in-to-laravels-ioc-container--cms-22167)说的，可能我有多种 fuel 供给 car，那我需要每次更换 fuel 的具体实现后，就要重新调用 car 的 setFuel 方法。使用 rebinding 就可以自动在重新 bind fuel 的时候调用，不用在业务逻辑中调用。

它也可以用来扩展自身的行为，比如 laravel 自身的 AuthServiceProvider 里有如下代码：
```php
protected function registerRequestRebindHandler()
{
    $this->app->rebinding('request', function ($app, $request) {
        $request->setUserResolver(function ($guard = null) use ($app) {
            return call_user_func($app['auth']->userResolver(), $guard);
        });
    });
}
```
如果你打算更换掉 laravel 里的 request，那你 bind 自定义的 Request 的时候，它就会自动调用这个方法。

但容器扩展自身行为还有一个 extend 方法可以用，rebinding 我想应该更多用于单例的扩展，因为单例 resolve 后，要更改只能通过重新 bind 的形式。