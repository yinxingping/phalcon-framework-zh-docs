# Web服务器配置

为了让您的应用程序能够正常工作，您需要设置您的web服务器来正确处理重定向。对于流行的web服务器的设置说明如下：

## PHP-FPM

---

PHP-FPM（FastCGI进程管理器）通常用于处理PHP文件。现在，PHP-FPM与所有基于Linux的PHP发行版绑定在一起。



Windows中PHP-FPM通过PHP发行包中的文件_** php-cgi.exe **_提供。您可以用这个脚本启动它来帮助设置选项。Windows不支持unix套接字，因此这个脚本将在端口9000上以TCP模式启动 fast-cgi。



用以下内容创建 _**php-fcgi.bat **_：

```
@ECHO OFF
ECHO Starting PHP FastCGI...
set PATH=C:\PHP;%PATH%
c:\bin\RunHiddenConsole.exe C:\PHP\php-cgi.exe -b 127.0.0.1:9000
```

## PHP内置web服务器（用于开发人员）

---

为了加速让您的Phalcon应用程序在开发模式下运行起来，最简单的方法就是使用这个内置的web服务器。不要在生产环境下使用此服务器，后面为 [Nginx](https://docs.phalconphp.com/zh/3.2/webserver-setup#nginx) 和 [Apache](https://docs.phalconphp.com/zh/3.2/webserver-setup#apache) 提供的配置才是您所需要的。

### Phalcon配置

没有Apache或Nginx的条件下，为了让Phalcon需要的动态URI重写起作用，您可以使用后面的路由文件：[.htrouter.php](https://github.com/phalcon/phalcon-devtools/blob/master/templates/.htrouter.php)

如果您用[Phalcon-Devtools](https://docs.phalconphp.com/en/3.2/devtools-installation)创建了应用程序，这个文件应该已经在您项目的根目录下，您可以用下面的命令启动服务器：

```
$(which php) -S localhost:8000 -t public .htrouter.php
```

上面命令详解：

* $\(which php\)- 将插入PHP命令的绝对路径

* -S localhost:8000 - 用提供的 host:port 启动服务模式

* -t public - 定义服务器根目录，php需要路由请求到public目录下的静态资源\(assets\)如JS, CSS和images

* .htrouter.php - 将对每个请求进行评估的入口点

然后在浏览器中点击[http://localhost:8000/](http://localhost:8000/)来检查是否一切正常。

## Nginx

---

[Nginx](http://wiki.nginx.org/Main)是一个自由、开源、高性能的HTTP服务器和反向代理，以及一个IMAP/POP3代理服务器。与传统的服务器不同，Nginx不依赖线程来处理请求，相反，它使用了一种更具伸缩性的事件驱动（异步）架构。这种架构使用较小，但更重要的是、在负载下可预测的内存量。



Nginx + PHP-FPM + Phalcon 提供了一套强大的工具，为你的PHP应用程序提供最大的性能。

#### 安装Nginx

[NginX Offical Site](https://www.nginx.com/resources/wiki/start/topics/tutorials/install/)

#### Phalcon设置

你可以使用以下可能设置来配置Nginx+Phalcon：

```

```

#### 启动Nginx

通常从命令行启动nginx，但这取决于你的安装方法。

## Apache

---

Apache是一个多平台可用、流行且广为人知的Web服务器。

#### Phalcon设置

以下是用来配置 Apache+Phalcon 的可能设置。这些记录主要聚焦于允许使用友好URLs和路由组件的 mod\_rewrite 模块的设置。通常一个应用程序有以下结构：

```

```

#### Document root

这是最普通的情况，应用被安装在Document root下的任何目录。这种情况下，我们使用两个 .htaccess 文件，第一个用于隐藏应用代码，将所有请求导向到应用的document root（public/）。

> 注意：使用 .htaccess 文件要求你的apache安装有 “AllowOverride“ 选项设置。

```

```

第二个 .htaccess 文件位于 public/ 目录，重写所有的URIs到 public/index.php 文件：

```

```

#### Apache设置

如果你不想使用 .htaccess文件，你可以将这些设置移到apache的主设置文件：

```

```

#### Virtual Hosts

这个设置允许你将Phalcon应用程序安装到一个虚拟主机：

```

```



