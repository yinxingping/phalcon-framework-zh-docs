# 要求

Phalcon需要PHP来运行。它的松耦合设计允许开发人员在没有附加扩展的情况下安装和使用它的功能。某些组件依赖于其他扩展。例如，使用数据库连接将需要 _**php**_**pdo **扩展。如果您的RDBMS是Mysql/MariaDb或Aurora数据库，那么您也需要** php**_**mysqlnd **_扩展。类似地，要使用PostgreSql数据库时需要_** php**_**\_pgsql **扩展。

## 硬件

---

在提供高性能的同时，Phalcon被设计为尽可能少的使用资源。虽然我们已经在各种低端环境中测试过了（比如0.25 RAM，0.5 CPU），但是您所选择的硬件将取决于您的应用程序需要。

我们的网站和博客（以及其他站点）托管在一个Amazon VM上，有512MB RAM和1个vCPU。

## 软件

---

* PHP &gt;= 5.5

> 您应该始终尝试使用最新版本的Phalcon和PHP来解决问题、增强安全性和性能。PHP 5.5将在不久的将来被弃用，而Phalcon 4 将只支持PHP 7。

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

* [PDO](http://php.net/manual/en/book.pdo.php) 扩展以及相关RDBMS的具体扩展（如 [MySQL](http://php.net/manual/en/ref.pdo-mysql.php)，[PostgreSql](http://php.net/manual/en/ref.pdo-pgsql.php)等）

* [OpenSSL](http://php.net/manual/en/book.openssl.php) 扩展

* [Mbstring](http://php.net/manual/en/book.mbstring.php) 扩展

* [Memcache](http://php.net/manual/en/book.memcache.php)，[Memcached](http://php.net/manual/en/book.memcached.php) 或其他相关的缓存适配器，取决于您对缓存的使用情况



## 

# 安装



