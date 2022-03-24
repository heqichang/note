Pipeline 用来处理中间件逻辑的。它代码相对较短，公共方法也就几个，相对比较好理解。主要逻辑集中在 then 方法，难点可能就是需要了解 php 中 array_reduce 方法。

array_reduce 中第二个参数要求的是个 callable，格式声明如下：
```php
callback(mixed $carry, mixed $item): mixed
```

在 laravel 中这个 callback 是这样的
```php

return function ($stack, $pipe) {
    return function ($passable) use ($stack, $pipe) {
        // ...
        return handle($passable, $stack) // 伪代码
        // 这里调用的就是中间件中定义的 handle 方法
        // handle($request, Closure $next)
        // $stack 保留的是上一次得到的 function($passable)
        // 所以中间件中 $next 就是这个 $stack，所以这么调用 $next($request)
    };
};

```

所以最后就形成了类似如下的包裹关系，（伪代码）
```
function($request) {

    // 初始化 firstMiddleware
    firstMiddleware->handle($request, $next) {
        // first do something

        $next($request) {

            // 初始化 secondMiddleware
            secondMiddleware->handle($request, $next) {
                // second do something

                $next($request) {

                    // 初始化 thiredMiddleware
                    // ...
                }

                // second do something
            }
        }
        
        // first do something
    }
}
```

所以再看 then 代码就好理解了：
```php
public function then(Closure $destination)
{
    $pipeline = array_reduce(
        array_reverse($this->pipes()), $this->carry(), $this->prepareDestination($destination)
    );
    return $pipeline($this->passable);
}
```

\$pipeline 就是最顶层的 function($request) 了，而且为什么第一个参数要 array_reverse 也可以从包裹关系理解到了。

最后还能看出中间件执行顺序，第一个执行的中间件，前置操作先执行，而后置操作会最后执行。


参考资料：
[Pipeline Pattern in Laravel](https://dev.to/abrardev99/pipeline-pattern-in-laravel-278p)


