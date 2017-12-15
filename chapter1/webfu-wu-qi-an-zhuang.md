# Web服务器配置

为了让Phalcon应用路由工作，你需要配置你的web服务器来正确地处理重定向。流行的web服务器配置指导如下：

> 注意：Windows、WAMP、XAMPP，Cherokee配置请参考英文文档

## PHP-FPM

---

PHP-FPM（FastCGI Process Manager）用于允许处理PHP文件。如今，PHP-FPM已经与所有基于Linux的PHP distributions绑定了。

## PHP内置Web服务器（供开发者使用）

---

为了加快让你的Phalcon应用程序在开发模式下运行起来，最容易的方法就是使用这个内置的Web服务器。不要在生产环境下使用这个服务器，后面为Nginx和Apache提供的配置才是你需要的。

#### Phalcon配置

没有Apache或Nginx的条件下，为了让Phalcon需要的动态URI重写起作用，你可以使用后面的路由文件：[.htrouter.php](https://github.com/phalcon/phalcon-devtools/blob/master/templates/.htrouter.php)

如果你用[Phalcon-Devtools](https://docs.phalconphp.com/en/3.2/devtools-installation)创建了应用程序，这个文件应该已经在你项目的根目录下，你可以用下面的命令启动服务器：

```

```

上面命令详解：

* $\(which php\)- 将插入PHP命令的绝对路径

* -S localhost:8000 - 用提供的 host:port 启动服务模式

* -t public - 定义服务器根目录，php需要路由请求到public目录下的静态资源\(assets\)如JS, CSS和images

* .htrouter.php - 计算每个请求的进入点

然后在浏览器中点击[http://localhost:8000/](http://localhost:8000/)来检查是否一切正常。

## Nginx

---

[Nginx](http://wiki.nginx.org/Main)是一个自由、开源、高性能的HTTP服务器和反向代理，也是一个IMAP/POP3代理服务器。不像传统的服务器，Nginx不靠线程处理请求，取而代之的，它使用更多扩展性的事件驱动（异步）架构。这种架构使用更小但更重要、可预计数量的内存。

Nginx + PHP-FPM + Phalcon 为你的PHP应用程序提供了一个强大的工具集，这个工具集可以提供最大的性能。

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



  


