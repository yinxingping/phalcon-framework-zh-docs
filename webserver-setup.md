# Web服务器配置

为了让你的应用程序能够正常工作，你需要设置你的web服务器来正确处理重定向。对于流行的web服务器的设置说明如下：

## PHP-FPM

---

PHP-FPM（FastCGI进程管理器）通常用于处理PHP文件。现在，PHP-FPM与所有基于Linux的PHP发行版绑定在一起。

Windows中PHP-FPM通过PHP发行包中的文件`php-cgi.exe`提供。你可以用这个脚本启动它来帮助设置选项。Windows不支持unix套接字，因此这个脚本将在端口9000上以TCP模式启动 fast-cgi。

用以下内容创建 `php-fcgi.bat`：

```bat
@ECHO OFF
ECHO Starting PHP FastCGI...
set PATH=C:\PHP;%PATH%
c:\bin\RunHiddenConsole.exe C:\PHP\php-cgi.exe -b 127.0.0.1:9000
```

## PHP内置web服务器（用于开发人员）

---

为了加速让你的Phalcon应用程序在开发模式下运行起来，最简单的方法就是使用这个内置的web服务器。不要在生产环境下使用此服务器，后面为Nginx和Apache提供的配置才是你所需要的。

### Phalcon配置

没有Apache或Nginx的条件下，为了让Phalcon需要的动态URI重写起作用，你可以使用后面的路由文件：[.htrouter.php](https://github.com/phalcon/phalcon-devtools/blob/master/templates/.htrouter.php)

如果你用[Phalcon-Devtools](devtools-installation.md)创建了应用程序，这个文件应该已经在你项目的根目录下，你可以用下面的命令启动服务器：

```bash
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

    server {
        # Port 80 will require Nginx to be started with root permissions
        # Depending on how you install Nginx to use port 80 you will need
        # to start the server with `sudo` ports about 1000 do not require
        # root privileges
        # listen      80;

        listen        8000;
        server_name   default;

        ##########################
        # In production require SSL
        # listen 443 ssl default_server;

        # ssl on;
        # ssl_session_timeout  5m;
        # ssl_protocols  SSLv2 SSLv3 TLSv1;
        # ssl_ciphers  ALL:!ADH:!EXPORT56:RC4+RSA:+HIGH:+MEDIUM:+LOW:+SSLv2:+EXP;
        # ssl_prefer_server_ciphers   on;

        # These locations depend on where you store your certs
        # ssl_certificate        /var/nginx/certs/default.cert;
        # ssl_certificate_key    /var/nginx/certs/default.key;
        ##########################

        # This is the folder that index.php is in
        root /var/www/default/public;
        index index.php index.html index.htm;

        charset utf-8;
        client_max_body_size 100M;
        fastcgi_read_timeout 1800;

        # Represents the root of the domain
        # http://localhost:8000/[index.php]
        location / {
            # Matches URLS `$_GET['_url']`
            try_files $uri $uri/ /index.php?_url=$uri&$args;
        }

        # When the HTTP request does not match the above
        # and the file ends in .php
        location ~ [^/]\.php(/|$) {
            # try_files $uri =404;

            # Ubuntu and PHP7.0-fpm in socket mode
            # This path is dependent on the version of PHP install
            fastcgi_pass  unix:/var/run/php/php7.0-fpm.sock;

            # Alternatively you use PHP-FPM in TCP mode (Required on Windows)
            # You will need to configure FPM to listen on a standard port
            # https://www.nginx.com/resources/wiki/start/topics/examples/phpfastcgionwindows/
            # fastcgi_pass  127.0.0.1:9000;

            fastcgi_index /index.php;

            include fastcgi_params;
            fastcgi_split_path_info ^(.+?\.php)(/.*)$;
            if (!-f $document_root$fastcgi_script_name) {
                return 404;
            }

            fastcgi_param PATH_INFO       $fastcgi_path_info;
            # fastcgi_param PATH_TRANSLATED $document_root$fastcgi_path_info;
            # and set php.ini cgi.fix_pathinfo=0

            fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        }

        location ~ /\.ht {
            deny all;
        }

        location ~* \.(js|css|png|jpg|jpeg|gif|ico)$ {
            expires       max;
            log_not_found off;
            access_log    off;
        }
    }

#### 启动Nginx

通常从命令行启动nginx，但这取决于你的安装方法。

## Apache

---

Apache是一种流行的、众所周知的web服务器，可以在许多平台上使用。

#### Phalcon设置

以下是用来配置 Apache+Phalcon 的可能设置。这些记录主要聚焦于允许使用友好URLs和 [路由组件](routing.md) 的 `mod_rewrite` 模块的设置。通常一个应用程序有以下结构：

```
test/
  app/
    controllers/
    models/
    views/
  public/
    css/
    img/
    js/
    index.php
```

#### 文档根目录

这是最常见的情况，应用被安装在文档根下的任何目录中。在本例中，我们使用两个.htaccess文件，第一个用于隐藏应用代码，将所有请求转发到应用程序的文档根（public/）。

> 注意：使用 .htaccess 文件要求你的apache安装有 “AllowOverride“ 选项设置。

```
# test/.htaccess

<IfModule mod_rewrite.c>
    RewriteEngine on
    RewriteRule   ^$ public/    [L]
    RewriteRule   ((?s).*) public/$1 [L]
</IfModule>
```

第二个 .htaccess 文件位于 public/ 目录，重写所有的URI到 public/index.php 文件：

```
# test/public/.htaccess

<IfModule mod_rewrite.c>
    RewriteEngine On
    RewriteCond   %{REQUEST_FILENAME} !-d
    RewriteCond   %{REQUEST_FILENAME} !-f
    RewriteRule   ^((?s).*)$ index.php?_url=/$1 [QSA,L]
</IfModule>
```

#### Apache设置

如果你不想使用 .htaccess文件，你可以将这些设置移到apache的主设置文件：

```
<IfModule mod_rewrite.c>

    <Directory "/var/www/test">
        RewriteEngine on
        RewriteRule  ^$ public/    [L]
        RewriteRule  ((?s).*) public/$1 [L]
    </Directory>

    <Directory "/var/www/test/public">
        RewriteEngine On
        RewriteCond   %{REQUEST_FILENAME} !-d
        RewriteCond   %{REQUEST_FILENAME} !-f
        RewriteRule   ^((?s).*)$ index.php?_url=/$1 [QSA,L]
    </Directory>

</IfModule>
```

#### Virtual Hosts

第二种配置允许你将Phalcon应用程序安装到虚拟主机：

```
<VirtualHost *:80>

    ServerAdmin    admin@example.host
    DocumentRoot   "/var/vhosts/test/public"
    DirectoryIndex index.php
    ServerName     example.host
    ServerAlias    www.example.host

    <Directory "/var/vhosts/test/public">
        Options       All
        AllowOverride All
        Require       all granted
    </Directory>

</VirtualHost>
```

## Cherokee

---

[Cherokee](http://www.cherokee-project.com/) 是一个高性能的web服务器，它非常快速、灵活和易于配置。

#### Phalcon配置

Cherokee提供了一个友好的图形界面来配置web服务器上几乎所有可用的设置。

以root身份执行`/path-to-cherokee/sbin/cherokee-admin`进入cherokee管理员页面

![](https://docs.phalconphp.com/images/content/webserver-cherokee-1.jpg)

点击`vServers`创建一个虚拟主机，然后增加一个新的虚拟服务器：

![](https://docs.phalconphp.com/images/content/webserver-cherokee-2.jpg)

最近添加的虚拟服务器一定出现在屏幕的左侧栏中，在`Behaviors`选项卡中，你将看到该虚拟服务器的一组默认行为。点击`Rule Management`按钮，删除那些标签为`Directory/cherokee_thems`和 `Directory/icons` 的：

![](https://docs.phalconphp.com/images/content/webserver-cherokee-3.jpg)

使用向导添加 `PHP Language` 行为，这个行为允许你运行PHP应用程序：

正常情况下，这种行为不需要额外的设置。添加另一个行为，这一次在`Manual Configuration` 部分。在`Rule Type`下选择 `File Exists`，然后确保选项`Match any file为 enabled`：

![](https://docs.phalconphp.com/images/content/webserver-cherokee-4.jpg)

在 `Handler` 选项卡中选择`List & Send`作为`handler`：

![](https://docs.phalconphp.com/images/content/webserver-cherokee-5.jpg)

为了启用URL重写引擎，编辑 `Default` 行为，将 `handler` 改为 `Redirection`，然后增加下面的正则表达式到引擎`^\(.\*\)$`：

![](https://docs.phalconphp.com/images/content/webserver-cherokee-6.jpg)

最后，确保行为按照下列顺序排列：

![](https://docs.phalconphp.com/images/content/webserver-cherokee-8.jpg)

在浏览器里执行应用程序：

![](https://docs.phalconphp.com/images/content/webserver-cherokee-9.jpg)

