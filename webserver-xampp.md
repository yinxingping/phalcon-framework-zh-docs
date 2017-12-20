# 概述

[XAMPP](https://www.apachefriends.org/download.html) 是一个易于安装的Apache发行版，它包括了MySQL、PHP和Perl。一旦下载了XAMPP，你所有需要做的就是提取并启动它。以下是关于如何在Windows的XAMPP下安装Phalcon的详细说明，强力推荐使用最新的XAMPP。

## 下载正确的Phalcon版本

XAMPP总是发行Apache和PHP的32位版。你需要从下载区下载x86版的Windows Phalcon。

下载Phalcon库后你将看到类似以下的zip文件：

![](https://docs.phalconphp.com/images/content/webserver-xampp-1.png)

从压缩包中提取库得到Phalcon DLL：

![](https://docs.phalconphp.com/images/content/webserver-xampp-2.png)

复制`php\_phalcon.dll`到PHP扩展目录。如果你在`C:\xampp`文件夹安装了XAMPP，这个扩展需要在`C:\xampp\php\ext`里：

![](https://docs.phalconphp.com/images/content/webserver-xampp-3.png)

编辑 `C:\xampp\php\php.ini`，可以用Notepad或类似程序编辑它。我们推荐使用Notepad++以避免行尾问题。在文件末尾添加：

```ini
extension=php_phalcon.dll
```

并保存。

![](https://docs.phalconphp.com/images/content/webserver-xampp-4.png)

从XAMPP的控制中心重启Apache Web服务器，这将加载最新的PHP配置。

![](https://docs.phalconphp.com/images/content/webserver-xampp-5.png)

打开浏览器导航到 [http://localhost](http://localhost)，将出现XAMPP的欢迎页面。点击链接 `phpinfo()` ：

![](https://docs.phalconphp.com/images/content/webserver-xampp-6.png)

[phpinfo](http://php.net/manual/zh/function.phpinfo.php) 将在屏幕上输出关于当前PHP状态的大量信息。向下滚动，检查是否正确加载了该扩展。

![](https://docs.phalconphp.com/images/content/webserver-xampp-7.png)

如果你能在 phpinfo\(\) 的输出中看到phalcon版本，恭喜你！Phalcon已经玩起来了。

## 相关向导

* [一般安装](installation.md)
* [XAMPP安装](webserver-xampp.md)
