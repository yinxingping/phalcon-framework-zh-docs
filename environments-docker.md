# 概述

Phalcon Compose是一个社区驱动的在Docker上运行的样板开发环境。它的目的是使其能够更容易地引导Phalcon应用程序，并在开发或生产环境中运行它们。


## 依赖

要在机器上运行这个栈，你至少需要：
* 操作系统：Windows, Linux, 或OS X
* [Docker Engine](https://docs.docker.com/installation/) >= 1.10.0
* [Docker Compose](https://docs.docker.com/compose/install/) >= 1.6.2


## 服务

包含的服务如下：

| 服务名称          | 描述                                     |
| ------------- | -------------------------------------- |
| mongo         | MongoDB服务器容器                           |
| postgres      | PostgreSQL服务器容器                        |
| mysql         | MySQL数据库容器                             |
| phpmyadmin    | MySQL和MariaDB的一个web接口                  |
| memcached     | Memcached服务器容器                         |
| queue         | Beanstalk队列容器                          |
| aerospike     | Aerospike - 为Flash和Ram优化的可靠、高性能、分布式数据库 |
| redis         | Redis数据库容器                             |
| app           | PHP 7, Apache 2 and Composer容器         |
| elasticsearch | Elasticsearch是一个使数据更易搜索的强有力的开源搜索和分析引擎  |


## 安装

### 使用Composer (推荐)

使用Composer，你可以如下创建一个新项目：

```bash
$ composer create-project phalcon/compose --prefer-dist <folder name>
```

你的输出应该类似这里：

```php
Example
 Installing phalcon/compose (version)
  - Installing phalcon/compose (version)
    Loading from cache

Created project in folderName
> php -r "copy('variables.env.example', 'variables.env');"
Loading composer repositories with package information
Updating dependencies (including require-dev)
Nothing to install or update
Generating autoload files
```


### 使用 Git

另一种初始化你的应用的方法是使用Git

```bash
$ git clone git@github.com:phalcon/phalcon-compose.git
```

>
> 确保将`variables.env.example`拷贝到`variable.env`并调整文件里的设置
>

将你的Phalcon应用放到`application`文件夹。


## 配置

在你的`/etc/hosts`里添加 `phalcon.local` (或你喜欢的主机名，如下所示：

```bash
$ 127.0.0.1 www.phalcon.local phalcon.local
```


## 用法

现在，你可以构建、创建、启动和将容器附加到应用程序的环境中。要构建容器，在项目根内使用以下命令进行：

```bash
$ docker-compose build
```

To start the application and run the containers in the background, use following command inside project root:
要启动应用程序并在后台运行容器，请在项目根内使用以下命令：

这里你可以使用`-p <my-app>`参数来指定你喜欢的项目名称

```bash
$ docker-compose up -d
```

现在使用Phalcon开发工具来在应用容器建立你的项目吧。

Replace project in **<project_app_1>** with the name of your project/directory (shown in the output of `docker-compose up -d`)
用你的项目/目录的名称（在`docker-compose up -d`的输出中显示）代替掉**<project_app_1>**中的项目

```bash
$ docker exec -t <project_app_1> phalcon project application simple
```

现在你可以启动你的引用，在浏览器里访问 `http://phalcon.local` (或你上面选择的主机名称)了。


## 设置

如果你的应用使用了一个文件缓存或把日志写到文件里，你可以如下设置你的缓存和日志文件夹：

| Directory | Path             |
| --------- | ---------------- |
| Cache     | `/project/cache` |
| Logs      | `/project/log`   |


## 日志

对于大多数容器，你可以使用`docker logs <container_name>`命令在你的主机机器中访问日志。


## 环境变量

您可以通过编辑`variables.env`文件，将多个环境变量从外部文件传递到服务容器。


### Web环境

| 环境变量                 | 描述               | 默认值             |
| -------------------- | ---------------- | --------------- |
| `WEB_DOCUMENT_ROOT`  | web服务器的文档根(容器内部) | /project/public |
| `WEB_DOCUMENT_INDEX` | Index document   | index.php       |
| `WEB_ALIAS_DOMAIN`   | 域别名              | *.vm            |
| `WEB_PHP_SOCKET`     | PHP-FPM socket地址 | 127.0.0.1:9000  |
| `APPLICATION_ENV`    | 应用环境             | development     |
| `APPLICATION_CACHE`  | 应用的缓存目录(容器内部)    | /project/cache  |
| `APPLICATION_LOGS`   | 应用的日志目录(容器内部)    | /project/logs   |


### phpMyAdmin变量

| 环境变量               | 描述                                       | 默认值     |
| ------------------ | ---------------------------------------- | ------- |
| `PMA_ARBITRARY`    | 当设置为1时允许连接到服务器                           | 1       |
| `PMA_HOST`         | 定义MySQLO服务器的地址/主机名称                      | mysql   |
| `PMA_HOSTS`        | 定义逗号分隔的MySQL服务器的地址/主机名。仅当`PMA_HOST`为空时才使用 |         |
| `PMA_PORT`         | 定义MySQL服务器的端口                            | 3306    |
| `PMA_VERBOSE`      | 定义MySQL服务器的详细名称                          |         |
| `PMA_VERBOSES`     | 定义逗号分隔的MySQL服务器的详细名称。仅当`PMA_VERBOSE` 为空时使用 |         |
| `PMA_USER`         | 定义用于配置身份验证方法的用户名                         | phalcon |
| `PMA_PASSWORD`     | 定义用于配置身份验证方法的密码                          | secret  |
| `PMA_ABSOLUTE_URI` | 完全限定的路径(例如:https://pma.example.net/)，反向代理使phpMyAdmin可用 |         |

*另请参阅*
* https://docs.phpmyadmin.net/en/latest/setup.html#installing-using-docker
* https://docs.phpmyadmin.net/en/latest/config.html#config
* https://docs.phpmyadmin.net/en/latest/setup.html


## Xdebug远程调试 (PhpStorm)

出于调试目的你可以通过传递必要的参数（见`variables.env`）来设置Xdebug：

| 环境变量                         | 描述         | 默认值    |
| ---------------------------- | ---------- | ------ |
| `XDEBUG_REMOTE_HOST`         | `php.ini`项 | 你的主机IP |
| `XDEBUG_REMOTE_PORT`         | `php.ini`项 | 9000   |
| `XDEBUG_REMOTE_AUTOSTART`    | `php.ini`项 | Off    |
| `XDEBUG_REMOTE_CONNECT_BACK` | `php.ini`项 | Off    |

*注意* 你可以用如下命令找到你的本地IP：

**Linux/macOS**

```bash
$ ifconfig en1 | grep inet | awk '{print $2}' | sed 's/addr://' | grep .
```

**Windows**

```dos
> ipconfig
```


## 故障排除

### 启动或链接错误

如果你碰到任何启动问题，你可以尝试重构应用容器。数据不会丢失，这是一个安全的重置：

```bash
docker-compose stop
docker-compose rm --force app
docker-compose build --no-cache app
docker-compose up -d
```

### 完全重置

要重置所有容器，删除所有数据(mysql，elasticsearch等)，但不要在应用程序文件夹中删除项目文件：

```bash
docker-compose stop
docker-compose rm --force
docker-compose build --no-cache
docker-compose up -d
```

### 更新依赖

有时，基本镜像(例如，`phalconphp/php-apache:ubuntu-16.04)更新了，而Phalcon依赖这些镜像。因此，你需要更新它们，这样做总是一件好事，以确保您拥有最新的可用功能。需要对这些映像的依赖容器进行更新和重建：

```bash
docker pull mongo:3.4
docker pull postgres:9.5-alpine
docker pull mysql:5.7
docker pull phpmyadmin/phpmyadmin:4.6
docker pull memcached:1.4-alpine
docker pull phalconphp/beanstalkd:1.10
docker pull aerospike:latest
docker pull redis:3.2-alpine
docker pull elasticsearch:5.2-alpine
docker pull phalconphp/php-apache:ubuntu-16.04
```

Linux/MacOS用户可以用 `make`执行这个任务

```bash
$ make pull
```

然后你必须重置所有容器，删除所有数据，重新构建服务并重新启动应用程序。

Linux/MacOS用户可以使用`make`执行这个任务:

```bash
$ make reset
```