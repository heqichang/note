### 本地环境
* windows 10
* homestead 12.1
* phpstorm 2021.2.1

### 步骤
1. 现在版本的 homestead 已经安装配置好了 xdebug 了，不用做任何更改。
2. 在 phpstorm 的目录栏上点击 File -> Setting，然后在 PHP -> Servers 处添加一个 server，name 可以随便填，Host 和 Port 填你本机访问站点的地址，勾选下面的 use path mappings 选项，最后填好 Project files 映射关系就可以了。
3. 点开 phpstorm 的 toolbar 上 start listening for PHP Debug connections 的按钮（像个电话筒一样的），
4. 在 chrome 下载一个 xdebug 的扩展，它开启就相当于在我们的 cookies 里加入一个 XDEBUG_SESSION ，方便我们调试。

到此我们就配置完成了。