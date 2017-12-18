# 概述

[XAMPP](https://www.apachefriends.org/download.html) 是一个易于安装的Apache发行版，它包括了MySQL、PHP和Perl。一旦下载了XAMPP，你所有需要做的就是提取并启动它。以下是关于如何在Windows的XAMPP下安装Phalcon的详细说明，强力推荐使用最新的XAMPP。

## 下载正确的Phalcon版本

---

XAMPP总是发行Apache和PHP的32位版。你需要从下载区下载x86版的Windows Phalcon。

下载Phalcon库后你将看到类似以下的zip文件：

![](/assets/import15.png)

从压缩包中提取库得到Phalcon DLL：

![](/assets/import16.png)

复制 _**php\_phalcon.dll**_ 到PHP扩展目录。如果你在_** C:\xampp**_ 文件夹安装了XAMPP，这个扩展需要在 _**C:\xampp\php\ext**_里：

![](/assets/import17.png)

编辑 `C:\xampp\php\php.ini`，可以用Notepad或类似程序编辑它。我们推荐使用Notepad++以避免行尾问题。在文件为不添加：

```
extension=php_phalcon.dll
```

并保存。

![](/assets/import18.png)

从XAMPP的控制中心重启Apache Web服务器，这将加载最新的PHP配置。

![](/assets/import19.png)

打开浏览器导航到 [http://localhost](http://localhost)，将出现XAMPP的欢迎页面。点击链接 `phpinfo()` ：

![](/assets/import20.png)

[phpinfo](http://php.net/manual/en/function.phpinfo.php) 将在屏幕上输出关于当前PHP状态的大量信息。向下滚动，检查是否正确加载了该扩展。

![](/assets/import21.png)

如果你能在 phpinfo\(\) 的输出中看到phalcon版本，恭喜你！你现在正在使用Phalcon。

