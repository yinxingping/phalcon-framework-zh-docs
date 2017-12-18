# 教程—基础

本教程中，我们将带你从头开始构建一个简单的注册表单应用程序。下面的指南将向你介绍Phalcon框架的设计方面。

如果你想尽快开始，你可以跳过这一章，用我们的[开发工具](https://docs.phalconphp.com/en/3.2/devtools-usage)自动创建一个Phalcon项目。\(如果你没有经验并碰到困难，推荐你返回这里\)

## 文件结构

---

Phalcon的一个关键特征是它的松耦合，你可以为特定的应用程序创建惯例的目录结构。也就是说，当你和其他人合作时保持一致性是有帮助的。所以本教程使用一个“标准”结构，如果你过去使用过其他MVC框架的话，会感觉如同在家一样自在。

```
┗ tutorial
   ┣ app
   ┇ ┣ controllers
   ┇ ┇ ┣ IndexController.php
   ┇ ┇ ┗ SignupController.php
   ┇ ┣ models
   ┇ ┇ ┗ Users.php
   ┇ ┗ views
   ┗ public
      ┣ css
      ┣ img
      ┣ js
      ┗ index.php
```

注意：你将不会看到 vendor 目录，因为Phalcon的核心依赖都通过你安装的扩展加载到内存里了。如果你缺失了那一步没有安装Phalcon扩展，继续前请[返回](https://docs.phalconphp.com/en/3.2/installation)去完成安装。

如果这一切对你来说都是全新的，那么推荐你安装[Phalcon开发工具](https://docs.phalconphp.com/en/3.2/devtools-installation)，因为它利用PHP内置的服务器让你的应用运行起来，而无需通过在项目的根目录添加 .htrouter来配置web服务器。

如果你想使用Nginx，[这里](https://docs.phalconphp.com/en/3.2/webserver-setup#nginx)是一些额外的设置。

使用[这些](https://docs.phalconphp.com/en/3.2/webserver-setup#nginx)额外的设置，Apache也可以使用。

最后，如果你喜欢Cherokee使用[这里](https://docs.phalconphp.com/en/3.2/webserver-setup#cherokee)的设置。

## 引导

---

你需要创建的第一个文件是引导文件。该文件充当应用程序的入口点和配置。在这个文件中，你可以实现组件的初始化以及应用程序的行为。



这个文件处理3件事情：

* 注册自动加载组件
* 配置服务并将它们注册到依赖注入上下文中
* 解决应用程序的HTTP请求

#### 自动加载器

自动加载器利用一个通过Phalcon扩展运行的兼容[PSR-4](http://www.php-fig.org/psr/psr-4/)的文件加载器，通常会把应用的控制器和模型加入到自动加载器中。你可以注册目录，它将在应用程序的命名空间中搜索文件。\(如果你想阅读其他使用自动加载器的方式，你可以先看[这里](https://docs.phalconphp.com/en/3.2/loader#overview)\)



首先，让我们注册我们的应用的控制器和模型目录。别忘了从 [Phalcon\Loader](https://docs.phalconphp.com/en/3.2/api/Phalcon_Loader) 包含加载器。



**public/index.php**

```
<?php

use Phalcon\Loader;

// Define some absolute path constants to aid in locating resources
define('BASE_PATH', dirname(__DIR__));
define('APP_PATH', BASE_PATH . '/app');
// ...

$loader = new Loader();

$loader->registerDirs(
    [
        APP_PATH . '/controllers/',
        APP_PATH . '/models/',
    ]
);

$loader->register();
```

#### 依赖管理

由于Phalcon是松散耦合的，服务由框架依赖管理器注册，所以它们可以被自动注入到IoC容器包裹着的组件和服务中。你将会经常碰到依赖注入的术语 **DI**，依赖注入和控制反转（**IoC**）听起来可能是一个复杂的特性，但在Phalcon里，它们非常简单和实用。Phalcon的IoC容器由以下概念组成：

* 服务容器（Service Container）：我们在全局存储应用程序需要的服务的一个“包”
* 服务或组件（Service or Component）：要被注入组件库的数据处理对象

如果你仍然对细节感兴趣，请阅读 [Martin Fowler](https://martinfowler.com/articles/injection.html) 的文章。



每当框架需要一个组件或服务时，它就会使用一个约定的名称向容器请求服务。别忘记引入 **Phalcon\Di** 来设置服务容器。

服务可以用几种方式注册，但对于我们这个教程，我们将使用一个[匿名函数](http://php.net/manual/en/functions.anonymous.php) 。

#### 工厂默认

[Phalcon\Di\FactoryDefault](https://docs.phalconphp.com/en/3.2/api/Phalcon_Di_FactoryDefault) 是 [Phalcon\Di](https://docs.phalconphp.com/en/3.2/api/Phalcon_Di) 的一个变体。为了让事情更容易，它会自动注册Phalcon带的大部分组件。我们推荐你手动注册你的服务，但使用这个可以帮忙降低使用依赖管理的门槛。后面，一旦你对这个概念更熟悉了，就可以手动指定。



**public/index.php**

```
<?php

use Phalcon\Di\FactoryDefault;

// ...

// Create a DI
$di = new FactoryDefault();
```



在接下来的部分中，我们注册了 “view" 服务来指出框架在哪个目录下查找视图文件。由于视图与类不对应，它们不能用自动加载器来处理。



**public/index.php**

```
<?php

use Phalcon\Mvc\View;

// ...

// Setup the view component
$di->set(
    'view',
    function () {
        $view = new View();
        $view->setViewsDir(APP_PATH . '/views/');
        return $view;
    }
);
```



接下来，我们注册一个基础 URI





