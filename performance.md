# 提高性能

获得更快的应用程序需要改进许多方面：服务器、客户端、网络、数据库、web服务器、静态源等等。在本章中，我们重点介绍可以提高性能的场景，以及如何检测应用程序中真的很慢的地方。


## 服务器上的Profile

每个应用程序都是不同的，定量Profile对于了解在哪里可以提高性能是很重要的。Profiling给了我们一个什么是慢、什么不是的真实的画面。Profile可以在一个请求和其他请求之间有所不同，因此重要的是要进行足够的度量以得出结论。

使用XDebug进行Profiling

[XDebug](http://xdebug.org/docs) 为profile PHP应用提供了一个更容易的方法，只要安装xdebug扩展并在php.ini中开启即可：

```ini
xdebug.profiler_enable = On
```

使用类似 [Webgrind](https://github.com/jokkedk/webgrind/) 这样的工具，你可以看到哪个函数/方法比其他的更慢：

![](https://docs.phalconphp.com/images/content/performance-webgrind.jpg)


### 使用Xhprof进行Profiling

[Xhprof](https://github.com/facebook/xhprof) 是另一个profile PHP应用的有趣的扩展。添加下面一行到引导文件的开始处：

```php
<?php

xhprof_enable(XHPROF_FLAGS_CPU + XHPROF_FLAGS_MEMORY);
```

然后在文件末尾保存profiled数据：

```php
<?php

$xhprof_data = xhprof_disable('/tmp');

$XHPROF_ROOT = '/var/www/xhprof/';
include_once $XHPROF_ROOT . '/xhprof_lib/utils/xhprof_lib.php';
include_once $XHPROF_ROOT . '/xhprof_lib/utils/xhprof_runs.php';

$xhprof_runs = new XHProfRuns_Default();
$run_id = $xhprof_runs->save_run($xhprof_data, 'xhprof_testing');

echo "http://localhost/xhprof/xhprof_html/index.php?run={$run_id}&source=xhprof_testing\n";
```

Xhprof 提供了一个内置的HTML查看器来分析profiled数据：

![](https://docs.phalconphp.com/images/content/performance-xhprof-2.jpg)

![](https://docs.phalconphp.com/images/content/performance-xhprof-1.jpg)


### SQL语句的Profiling

大多数数据库系统都提供了识别慢SQL语句的工具。为了提高服务器端的性能，检测和修复慢速查询是非常重要的。在Mysql案例中，你可以使用慢查询日志来了解什么SQL查询所花费的时间比预期的要多：

```ini
log-slow-queries = /var/log/slow-queries.log
long_query_time = 1.5
```


## 客户端的Profile

有时，我们可能需要改进静态元素的加载，比如图像、javascript和css，以提高性能。以下工具对于检测客户端的常见瓶颈非常有用：


### 使用 Chrome/Firefox浏览器Profile

大多数现代浏览器都有工具来profile页面加载时间。在Chrome中，你可以使用网页检查器来了解一个页面所需要的不同资源的加载时间：

![](https://docs.phalconphp.com/images/content/performance-chrome-1.jpg)

[Firebug](http://getfirebug.com/) 提供了一个类似的功能：

![](https://docs.phalconphp.com/images/content/performance-firefox-1.jpg)


### Yahoo! YSlow

[YSlow](http://developer.yahoo.com/yslow/) 基于一组[高性能网页规则](http://developer.yahoo.com/performance/rules.html)来分析网页并提出提高性能的建议。

![](https://docs.phalconphp.com/images/content/performance-yslow-1.jpg)


### 使用Speed Tracer Profile

[Speed Tracer](https://developers.google.com/web-toolkit/speedtracer/) 是一种帮助你识别和修复web应用程序中的性能问题的工具。它可以可视化从浏览器内部的低级工具点获取的度量，并在应用程序运行时对它们进行分析。Speed Tracer是一个Chrome扩展，可以在所有支持扩展的平台上运行(Windows和Linux)。

![](https://docs.phalconphp.com/images/content/performance-speed-tracer.jpg)


这个工具非常有用，因为它可以帮助您获得渲染整个页面所用的真正时间，包括HTML解析、Javascript评估和CSS样式。


## 使用近期的PHP版本

PHP每一天都变得更快，使用最新版本能提高应用程序的性能，也提高了Phalcon的性能。


## 使用PHP字节码缓存

[APC](http://php.net/manual/zh/book.apc.php) 与许多其他字节码缓存一样，帮助应用程序减少每次请求中读取、分词和解析PHP文件的开销。安装扩展之后，使用以下设置启用APC：

```ini
apc.enabled = On
```


## 在后台做阻塞性工作

处理一个视频、发送电子邮件、压缩文件或图像等，是必须在后台作业中处理的慢任务。有许多工具提供排队或消息系统，这些系统与PHP可以很好地协同工作：

* [Beanstalkd](http://kr.github.io/beanstalkd/)
* [Redis](http://redis.io/)
* [RabbitMQ](http://www.rabbitmq.com/)
* [Resque](https://github.com/chrisboulton/php-resque>)
* [Gearman](http://gearman.org/)
* [ZeroMQ](http://www.zeromq.org/)


## Google Page Speed

[mod_pagespeed](https://developers.google.com/speed/pagespeed/mod) 加速你的网站并缩减页面加载时间。这个开源的Apache HTTP服务器模块 (也可以 [ngx_pagespeed](https://developers.google.com/speed/pagespeed/ngx)用于nginx) 自动将web性能最佳实践应用到页面以及相关的资源(CSS、JavaScript、图像)，而不需要修改现有的内容或工作流。