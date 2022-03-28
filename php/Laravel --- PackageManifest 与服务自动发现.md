通过 composer 引入的第三方 Laravel 服务可以被自动发现，而实现它的主要核心类是 \Illuminate\Foundation\PackageManifest 。

### 发现步骤
1. 在自己编写的 Service 项目中的 composer.json 文件里，在 extra.laravel 节点下定义你想要发现的 serviceProvider，比如 sanctum 里的定义：
```json
 "extra": {
    "laravel": {
        "providers": [
            "Laravel\\Sanctum\\SanctumServiceProvider"
        ]
    }
},
```
2. 在 Laravel 主项目的 composer.json 中定义了一个在 dump-autoload 之后执行的一个 script
```json
"post-autoload-dump": [
    "Illuminate\\Foundation\\ComposerScripts::postAutoloadDump",
    "@php artisan package:discover --ansi"
],
```
3. package:discover 这个 command 就会去执行 PackageManifest 中的 build 方法，这个方法会根据 composer 生成的 installed.json 文件另外生成一个 bootstrap/cache/package.php，用于在运行的时候 require 进代码

