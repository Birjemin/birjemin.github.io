# xhprof的使用

## 简介
XHProf是一个分层PHP性能分析工具。
XHProf是一个分层PHP性能分析工具。它报告函数级别的请求次数和各种指标，包括阻塞时间，CPU时间和内存使用情况。一个函数的开销，可细分成调用者和被调用者的开销，XHProf数据收集阶段，它记录调用次数的追踪和包容性的指标弧在动态callgraph的一个程序。它独有的数据计算的报告/后处理阶段。在数据收集时，XHProfd通过检测循环来处理递归的函数调用，并通过给递归调用中每个深度的调用一个有用的命名来避开死循环。XHProf分析报告有助于理解被执行的代码的结构，它有一个简单的HTML的用户界面（ PHP写成的）。基于浏览器的性能分析用户界面能更容易查看，或是与同行们分享成果。也能绘制调用关系图。（来自百度百科）

## 前提
我的PHP版本是PHP7:

![PHP版本](http://upload.ouliu.net/i/20180203104935y3sca.jpeg)

而xhprof支持PHP7的库请在[longxinH-xhprof](https://github.com/longxinH/xhprof)查看。

![longxinH/xhprof](http://upload.ouliu.net/i/20180203101436v20oc.png)

备注：我之前在[phacility](https://github.com/phacility/xhprof)克隆的，不过安装失败，原因就是我的PHP版本是PHP7，而phacility版本的并不支持。（采坑呢~~）

## 安装步骤(当做一个php项目！clone别人php代码放哪个目录你自己定)
1. 编译安装
```
cd /Users/birjemin/Developer/Php
git clone https://github.com/longxinH/xhprof
cd xhprof/extension
phpize
./configure
make
make install
```
[编译成功之后](http://upload.ouliu.net/i/20180203103331j7i77.jpeg)

2. 配置文件
```
cd /usr/local/etc/php/7.0/conf.d
vim ext-xhprof.ini
```
添加内容（这个就是编译成功之后的路径,见上图）
```
[xhprof]
extension="/usr/local/Cellar/php70/7.0.14_7/lib/php/extensions/no-debug-non-zts-20151012/xhprof.so"
```

3. 重启php-fpm（视个人情况而定）
```
cd /usr/local/etc/php/7.0/
sudo killall php-fpm
sudo /usr/local/Cellar/php70/7.0.14_7/sbin/php-fpm -D
```

4. 查看是否安装成功

[php -m](http://upload.ouliu.net/i/20180203110939nd5nd.png)

[phpinfo](http://upload.ouliu.net/i/20180203111006ewlh9.png)

恭喜安装成功！

## 使用步骤
1. 如果使用呢？两种方法，第一种就是把xhprof_lib.php和xhprof_runs.php拷贝到项目中，第二种就是用绝对路径引入这两个文件，推荐第一种，这样别人clone的你的项目，不需要做啥。

2. 建立两个开启函数(isDev()是判断本地环境的意思，请视情况删除)

* 开启xhprof
```
function enableXhprof()
{
    isDev() && xhprof_enable(0, []);
}
```
* 运行xhprof(xhprof_lib.php和xhprof_runs.php可以拷贝到项目的，请确保include_once请够加载这两个文件)。
```
function disableXhprof()
{
    if (isDev()) {
        $xhprof_data = xhprof_disable();
        include_once app_path() . "/xhprof_lib.php";
        include_once app_path() . "/xhprof_runs.php";
        $xhprof_runs = new \XHProfRuns_Default();
        $run_id = $xhprof_runs->save_run($xhprof_data, "xhprof_foo");
        #echo $run_id;
    }
}
```

3. 可以在需要监听的接口中代码片段前面加上enableXhprof(),后面加上disableXhprof()就好了。接下来就可以分析这段代码了。（请确保这两个函数全局访问）

4. 将克隆的xprof配置虚拟主机，这个和你的项目一样的，就把xprof也当做一个项目，配置成浏览器可访问。比如我的配置：
* host: 127.0.0.1       local.xhprof.com
* nginx server conf
```
server {
  listen       80;
  server_name  local.xhprof.com;
  access_log   logs/xhprof.access.log  main;
  autoindex on;
  location / {
    root  /Users/birjemin/Developer/Php/xhprof;
    index  index.html index.htm index.php;
    try_files $uri $uri/ /index.php?$query_string;
  }
  location ~ \.php$ {
    root           /Users/birjemin/Developer/Php/xhprof;
    fastcgi_pass   127.0.0.1:9000;
    fastcgi_index  index.php;
    fastcgi_param  SCRIPT_FILENAME $document_root$fastcgi_script_name;
    fastcgi_intercept_errors    on;
    include        fastcgi_params;
  }
}
```
5. 重启nginx，浏览器访问local.xhprof.com看看能不能访问（没有目录？？你是不是浏览器无法访问目录？？权限没开。。自己配置一下）

[xhprof网站](http://upload.ouliu.net/i/20180203112523ab2do.png)

6. 在postman或者浏览器访问接口，转啊转，好了之后就可以去
`http://local.xhprof.com/xhprof_html/`查看了。（图我就不截了。。下次讲一下xgui的使用）

[示例](http://upload.ouliu.net/i/20180203112909b6hs7.png)

## 遇到的问题
1. php -v的版本和phpinfo()的版本不一致，如下图：

[php -v](http://upload.ouliu.net/i/20180203105336m46fk.jpeg)

[phpinfo](http://upload.ouliu.net/i/201802031054371bsd3.png)

解决方法：更改php cli的默认版本
* 我用的是bash
```
vim .bash_profile
```
* 添加两行
```
# PHP -v
export PATH="/usr/local/Cellar/php70/7.0.14_7/bin:$PATH"
```
* 生效
```
source .bash_profile
```

2. 编译成功，但是php -m 没有 xhprof。

xhprof支持PHP7的库请在[longxinH-xhprof](https://github.com/longxinH/xhprof)查看！！！！

## 备注
全程经历了2.5h吧~~~不过文章也写了2.5小时。哈哈哈哈~这个东西和xdebug不一样的，这个是分析性能瓶颈的，可以让你优化代码。xdebug主要用于调试，我下次写一个xdebug安装的过程。

## 参考
1. [https://www.jianshu.com/p/38e3ae81970c](https://www.jianshu.com/p/38e3ae81970c)
2. [https://juejin.im/post/5860d23f128fe10069e1cfbf](https://juejin.im/post/5860d23f128fe10069e1cfbf)
3. [http://blog.csdn.net/qq_28602957/article/details/72697901](http://blog.csdn.net/qq_28602957/article/details/72697901)
4. [https://baike.baidu.com/item/xhprof/2513363?fr=aladdin](https://baike.baidu.com/item/xhprof/2513363?fr=aladdin)
