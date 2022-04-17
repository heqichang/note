真实加载的地方被定义在 Illuminate\Foundation\Application  的 registerConfiguredProviders 方法里，时间则是在 Kernel 调用 bootstrap 的时候。

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
        $this->app->register($provider);
    }

    // 记录延迟加载的 service
    $this->app->addDeferredServices($manifest['deferred']);
}
```