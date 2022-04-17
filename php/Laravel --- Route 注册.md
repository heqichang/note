Laravel 脚手架生成的项目文件中有 routes 文件夹，是存放路由定义的地方，而加载这些路由定义则是在 App\Providers\RouteServiceProvider 中的 boot 方法里。

在没有路由缓存的情况下，boot 里定义的路由会在该 ServiceProvider 下 boot 之后被加载。

注册路由用的 Route 这个 Facades 实际用的是 Illuminate\Routing\Router ，而不是这个命名空间下的 Route。

Router 类下有个 $routes，这个才是 app 注册的路由集合，它是 Illuminate\Routing\RouteCollection，最后寻找路由匹配也是调用这个类下的 match 方法

个人认为路由定义分为两种，一种就是单纯的定义 get、post、put、delete 等的纯路由，还有一种就是类似 domain、controller、middleware 等把各种属性组合在一起的组合路由。

纯路由会单纯的向 $routes 变量里添加新的路由定义，而组合路由则会创建一个 RouteRegistrar 对象，相当于给路由里创建了一层堆栈，这层堆栈保存了各种属性，比如 RouteServiceProvider 里这么定义的：

```php

Route::prefix('api')
    ->middleware('api')
    ->namespace($this->namespace)
```
其实相当于在 RouteRegistrar 里的 $attributes 属性里添加了
```php
$attributes = [
    'prefix' => 'api',
    'middleware' => 'api',
    'namespace' => $this->namespace,
];
```

还可以这么组合属性
```php
Route::prefix('api')->group(function() {
    Route::middleware('api')->group(function() {
        Route::namespace($this->namespace)->group(function() {

        });
    });
});

```
效果其实和上面一样，只不过 group 方法稍微复杂一点，它实际调用到了 Router 里的 group，把当前定义的属性都入栈到 Router 里，直接调用后出栈当前的属性。代码定义如下:

```php
public function group(array $attributes, $routes)
{
    foreach (Arr::wrap($routes) as $groupRoutes) {

        // $attributes 就是当前层次定义的一堆属性，比如 prefix、middleware 等
        // 这里会把这些属性加入到 Router 里的 $groupStack 属性里
        $this->updateGroupStack($attributes);
        
        // 执行 group 中定义的回调或者 require 一个路由定义的文件
        $this->loadRoutes($groupRoutes);

        // 把当前层定义的属性出栈
        array_pop($this->groupStack);
    }
}

```
