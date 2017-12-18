# 概述

[WampServer](http://www.wampserver.com/en/) 是一个Windows web开发环境。它允许你使用Apache2、PHP和Mysql数据库创建web应用程序。以下是关于如何在Windows的WampServer环境安装Phalcon的详细说明，强力推荐使用最新的WampServer。

## 下载正确的Phalcon版本

---

WAMP有32和64位版本，你可以从下载区下载适合你的WAMP的Phalcon DLL。

下载Phalcon库后你将看到类似以下的zip文件：

![](/assets/import9.png)

从压缩包中提取可以得到Phalcon DLL：

![](/assets/import10.png)

拷贝文件 _**php\_phalcon.dll**_ 到PHP扩展文件夹。如果WAMP安装在_** C:\wamp**_ 文件夹，扩展要求在 _**C:\wamp\bin\php\php5.5.12\ext**_ （假设你的WAMP安装了PHP 5.5.12）：

![](/assets/import11.png)

编辑 _**php.ini**_ 文件，它位于_**C:\wamp\bin\php\php5.5.12\php.ini**_，可以用Notepad或类似程序编辑。我们推荐Nodepad++以避免行尾问题。在文件的末尾添加：

```
extension=php_phalcon.dll
```

并保存。

![](/assets/import12.png)

`C:\wamp\bin\apache\apache2.4.9\bin\php.ini `下的_** **_`php.ini`_** **_也要编辑，在文件的末尾添加：

```
extension=php_phalcon.dll
```

并保存。

重启Apache Web服务器，在系统托盘上单击 `WampServer`_** **_图标，从弹出菜单中选择_** **_`Restart All Services`。查看托盘图标将再次变为绿色。

![](/assets/import13.png)

打开浏览器导航到 [http://localhost，将会出现WAMP欢迎页面。检查\_\*\*](http://localhost，将会出现WAMP欢迎页面。检查_**) extensions loaded \*\*\_部分以确认phalcon已经加载。

![](/assets/import14.png)

恭喜你！你现在正在使用Phalcon。

