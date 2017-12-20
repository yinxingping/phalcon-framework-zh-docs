# 要求

Phalcon需要PHP来运行。它的松耦合设计允许开发人员在没有附加扩展的情况下安装和使用它的功能。某些组件依赖于其他扩展。例如，使用数据库连接将需要 `phppdo`扩展。如果你的RDBMS是Mysql/MariaDb或Aurora数据库，那么你也需要`phpmysqlnd`扩展。类似地，要使用PostgreSql数据库时需要`php_pgsql`扩展。

## 硬件

在提供高性能的同时，Phalcon被设计为尽可能少的使用资源。虽然我们已经在各种低端环境中测试过了（比如0.25 RAM，0.5 CPU），但是你所选择的硬件将取决于你的应用程序需要。

我们的网站和博客（以及其他站点）托管在一个Amazon VM上，有512MB RAM和1个vCPU。

## 软件

* PHP &gt;= 5.5

> 你应该始终尝试使用最新版本的Phalcon和PHP来解决问题、增强安全性和性能。PHP 5.5将在不久的将来被弃用，而Phalcon 4 将只支持PHP 7。

### Phalcon 需要以下扩展来运行（最小）

* curl
* gettext
* gd2 （为使用 [Phalcon\Image\Adapter\Gd](https://docs.phalconphp.com/zh/3.2/api/Phalcon_Image_Adapter_Gd) 类）
* libpcre3-dev （Debian/Ubuntu），pcre-devel（CentOS），pcre（macOS）
* json
* mbstring
* pdo\_\*
* fileinfo
* openssl

### 根据应用程序的需要选择

* [PDO](http://php.net/manual/zh/book.pdo.php) 扩展以及相关RDBMS的具体扩展（如 [MySQL](http://php.net/manual/zh/ref.pdo-mysql.php)，[PostgreSql](http://php.net/manual/zh/ref.pdo-pgsql.php)等）

* [OpenSSL](http://php.net/manual/zh/book.openssl.php) 扩展

* [Mbstring](http://php.net/manual/zh/book.mbstring.php) 扩展

* [Memcache](http://php.net/manual/zh/book.memcache.php)，[Memcached](http://php.net/manual/zh/book.memcached.php) 或其他相关的缓存适配器，取决于你对缓存的使用情况


# 安装

由于Phalcon被编译为PHP扩展，所以它的安装与其他传统PHP框架有所不同。需要在web服务器上安装和加载一个模块。

## Linux

在Linux上安装Phalcon，你需要将我们的存储库添加到你的发行版中，然后安装它。

### 基于Deb的发行版（Debian，Ubuntu等）

#### 存储库安装

将存储库添加到你的发行版：

**稳定版**

```bash
curl -s https://packagecloud.io/install/repositories/phalcon/stable/script.deb.sh | sudo bash
```

**每晚构建版**

```bash
 curl -s https://packagecloud.io/install/repositories/phalcon/nightly/script.deb.sh | sudo bash
```

> 这只需要做一次，除非你的发行版本发生了变化，或者你想要从稳定版切换到最新构建版本。

#### Phalcon安装

要安装Phalcon，你需要在终端上执行以下命令：

**PHP 5.x**

```bash
sudo apt-get update
sudo apt-get install php5-phalcon
```

**PHP 7**

```bash
sudo apt-get update
sudo apt-get install php7.0-phalcon
```

#### 额外的PPAs

**Ondřej Surý**

如果你不想用位于 [packagecloud.io](https://packagecloud.io/phalcon) 的存储库，你可以随时使用 [Ondřej Surý](https://launchpad.net/~ondrej/+archive/ubuntu/php/) 提供的。

安装存储库：

```bash
sudo add-apt-repository ppa:ondrej/php
sudo apt-get update
```

然后Phalcon：

```bash
sudo apt-get install php-phalcon
```

### 基于RPM的发行版（CentOS，Fedora等）

#### 存储库安装

将存储库添加到你的发行版：

**稳定版**

```bash
curl -s https://packagecloud.io/install/repositories/phalcon/stable/script.rpm.sh | sudo bash
```

**每晚构建版**

```bash
curl -s https://packagecloud.io/install/repositories/phalcon/nightly/script.rpm.sh | sudo bash
```

> 这只需要做一次，除非你的发行版本发生了变化，或者你想要从稳定版切换到最新构建版本。

#### Phalcon安装

要安装Phalcon，你需要在终端上执行以下命令：

**PHP 5.x**

```bash
sudo yum update
sudo yum install php56u-phalcon
```

**PHP 7**

```bash
sudo yum update
sudo yum install php70u-phalcon
```

#### 额外的RPMs

**Remi**

[Remi Collet](https://github.com/remicollet) 为基于RPM的安装维护着一个优秀的存储库。你可以在 [这里](https://blog.remirepo.net/pages/Config-en) 找到关于如何为你的发行版启用它的说明

在那之后安装Phalcon是很容易的：

```bash
yum install php56-php-phalcon3
```

特定体系结构（X86/X64）和PHP（5.5、5.6、7.x）的版本都有提供。

## FreeBSD

已经为FreeBSD提供了移植版本。安装它你需要执行以下命令：

**pkg\_add**

```bash
pkg_add -r phalcon
```

**Source**

```bash
export CFLAGS="-O2 --fvisibility=hidden"

cd /usr/ports/www/phalcon

make install clean
```

## Gentoo

可以在这里找到安装Phalcon的说明 [https://github.com/smoke/phalcon-gentoo-overlay](https://github.com/smoke/phalcon-gentoo-overlay)

## macOS

在macOS系统中，你可以使用brew、macports或源代码来编译和安装这个扩展

#### 要求

* PHP 5.5.x/5.6.x/7.0.x/7.1.x 开发资源
* XCode

#### Brew

```bash
brew tap homebrew/homebrew-php
brew install php55-phalcon
brew install php56-phalcon
brew install php70-phalcon
brew install php71-phalcon
```

#### MacPorts

```bash
sudo port install php55-phalcon
sudo port install php56-phalcon
```

编辑php.ini文件，并在最后添加：

```ini
extension=php_phalcon.so
```

重启你的web服务器。

## Windows

要在Windows上使用Phalcon，你需要安装phalcon.dll。我们根据目标平台编译了几个DLL，你可以在我们的[download](https://phalconphp.com/en/download/windows)页面找到。

认清你的PHP安装和体系结构。如果你下载了错误的DLL，那么它就不会起作用。phpinfo\(\) 包含这些信息。在下面的示例中，我们将需要DLL的NTS版本：

![](https://docs.phalconphp.com/images/content/phpinfo-api.png)

可供使用的DLL有：

| Architecture | Version | Type                    |
| :----- | :--- | :---- |
| x64          | 7.x     | Thread safe             |
| x64          | 7.x     | Non Thread safe \(NTS\) |
| x86          | 7.x     | Thread safe             |
| x86          | 7.x     | Non Thread safe \(NTS\) |
| x64          | 5.6     | Thread safe             |
| x64          | 5.6     | Non Thread safe \(NTS\) |
| x86          | 5.6     | Thread safe             |
| x86          | 5.6     | Non Thread safe \(NTS\) |
| x64          | 5.5     | Thread safe             |
| x64          | 5.5     | Non Thread safe \(NTS\) |
| x86          | 5.5     | Thread safe             |
| x86          | 5.5     | Non Thread safe \(NTS\) |

编辑php.ini文件，并在最后添加：

```ini
extension=php_phalcon.dll
```

重启你的web服务器。

## 从源代码编译

从源代码编译类似于大多数环境\(Linux/macOS\)。

#### 要求

* PHP 5.5.x/5.6.x/7.0.x/7.1.x 开发资源
* GCC编译器（Linux/Solaris/FreeBSD）或XCode（macOS）
* re2c &gt;= 0.13
* libpcre-dev

你可以使用相关的包管理器在系统中安装这些包。下面是流行的linux发行版的说明：

**Ubuntu**

```bash
sudo apt-get install php5-dev libpcre3-dev gcc make
```

**Suse**

```bash
sudo zypper install php5-devel gcc make
```

**CentOS/Fedora/RHEL**

```bash
sudo yum install php-devel pcre-devel gcc make
```

#### 编译Phalcon

我们首先需要从Github的存储库克隆Phalcon

```bash
git clone https://github.com/phalcon/cphalcon
```

现在构建扩展

```bash
cd cphalcon/build
sudo ./install
```

你现在需要添加`extension=phalcon.so`到你的`php.ini`然后重启web服务器，以便加载扩展：

```ini
# Suse: Add a file called phalcon.ini in /etc/php5/conf.d/ with this content:
extension=phalcon.so

# CentOS/RedHat/Fedora: Add a file called phalcon.ini in /etc/php.d/ with this content:
extension=phalcon.so

# Ubuntu/Debian with apache2: Add a file called 30-phalcon.ini in /etc/php5/apache2/conf.d/ with this content:
extension=phalcon.so

# Ubuntu/Debian with php5-fpm: Add a file called 30-phalcon.ini in /etc/php5/fpm/conf.d/ with this content:
extension=phalcon.so

# Ubuntu/Debian with php5-cli: Add a file called 30-phalcon.ini in /etc/php5/cli/conf.d/ with this content:
extension=phalcon.so
```

## 高级编译

Phalcon自动检测你的体系结构，但是，你也可以指定为特定体系结构编译。

```bash
cd cphalcon/build

# One of the following:
sudo ./install --arch 32bits
sudo ./install --arch 64bits
sudo ./install --arch safe
```

如果自动安装失败了，你可以手动构建扩展：

```bash
git clone https://github.com/phalcon/cphalcon
# cd cphalcon/build/php5/32bits
cd cphalcon/build/php5/64bits

# NOTE: for PHP 7 you have to use
# cd cphalcon/build/php7/32bits
# or
# cd cphalcon/build/php7/64bits

make clean
phpize --clean

export CFLAGS="-O2 --fvisibility=hidden"
./configure --enable-phalcon

make
make install
```

如果你有特定PHP版本运行：

```bash
git clone https://github.com/phalcon/cphalcon
# cd cphalcon/build/php5/32bits
cd cphalcon/build/php5/64bits

# NOTE: for PHP 7 you have to use
# cd cphalcon/build/php7/32bits
# or
# cd cphalcon/build/php7/64bits

make clean
/opt/php-5.6.15/bin/phpize --clean

export CFLAGS="-O2 --fvisibility=hidden"
./configure --with-php-config=/opt/php-5.6.15/bin/php-config --enable-phalcon

make
make install
```

你现在需要添加`extension=phalcon.so`到你的`php.ini`然后重启web服务器，以便加载扩展。

你可以在web服务器根目录中创建一个小脚本，其中有以下内容：

```php
<?php

phpinfo();
```

然后在你的浏览器上加载它。应该有一个关于Phalcon的部分。如果没有，请确保你的扩展已被正确编译，你对 php.ini 进行了必要的更改，和你已经重新启动了web服务器。

你还可以从命令行检查你的安装：

```bash
php -r 'print_r(get_loaded_extensions());'
```

这将输出类似以下的内容：

```php
Array
(
    [0] => Core
    [1] => libxml
    [2] => filter
    [3] => SPL
    [4] => standard
    [5] => phalcon
    [6] => pdo_mysql
)
```

你也可以使用Cli查看已安装的模块：

```bash
php -m
```

> 注意：在一些基于Linux的系统中，你可能需要更改两个 php.ini 文件，一个用于你的web服务器\(apach/nginx\)，另一个用于CLI。如果只对web服务器加载了Phalcon，那么你将需要定位CLI php.ini 并为要加载的模块进行必要的添加。


