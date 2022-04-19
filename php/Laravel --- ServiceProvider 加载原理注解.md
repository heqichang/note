在 Illuminate\Foundation\Http\Kernel.php 文件里可以看到启动顺序：

```php 
protected $bootstrappers = [
    // ...
    // 这里注册启动 provider 的地方
    \Illuminate\Foundation\Bootstrap\RegisterProviders::class, // register
    \Illuminate\Foundation\Bootstrap\BootProviders::class, // boot
];
```

先深入注册代码，实际注册调用在 Illuminate\Foundation\Application 的 registerConfiguredProviders 方法里


```php
public function registerConfiguredProviders()
{
    // 读取 app.php 配置文件里定义的 service
    // 然后这里会先分类，让 Illuminate 空间下的 service 排在前
    $providers = Collection::make($this->make('config')->get('app.providers'))
                    ->partition(function ($provider) {
                        return str_starts_with($provider, 'Illuminate\\');
                    });

    // 然后插入第三方被自动发现的服务在 app 自定义的服务之前
    $providers->splice(1, 0, [$this->make(PackageManifest::class)->providers()]);
    (new ProviderRepository($this, new Filesystem, $this->getCachedServicesPath()))
                ->load($providers->collapse()->toArray());
}
```

然后来看下 ProviderRepository 的 load 方法

```php
public function load(array $providers)
{
    // 加载已缓存的 servers 文件
    $manifest = $this->loadManifest();
    
    // 如果没有缓存文件，或者缓存中的 providers 数组和代码配置中的不一致时会重新去生成这个缓存文件
    // 所以这里如果你只是改变 service 是否延迟加载的属性，它不会重新更新这个缓存文件 
    if ($this->shouldRecompile($manifest, $providers)) {
        $manifest = $this->compileManifest($providers);
    }
    
    // 注册事件触发的 service (延迟加载的一种)
    foreach ($manifest['when'] as $provider => $events) {
        $this->registerLoadEvents($provider, $events);
    }
    
    // 注册 service
    foreach ($manifest['eager'] as $provider) {
        // 这里面会调用到 provider 的 register() 方法
        $this->app->register($provider);
    }

    // 记录延迟加载的 service
    $this->app->addDeferredServices($manifest['deferred']);
}
```

注册完 provider 之后，下一步就是 boot。刚启动时只会 boot 标记为 eager 的 provider。

```php
这里要注意的一点是 Console 启动也会调用 provider 的 register 和 boot 方法，
所以最好不要在这两个方法里放入业务逻辑，因为 composer 执行的时候可能会报错，
因为它会去执行业务发现那个命令，会经过一次 laravel 的应用启动过程。
```


