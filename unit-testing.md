# 概述

编写适当的测试可以帮助编写更好的软件。如果设置了适当的测试用例，你可以消除大多数功能性bug，并更好地维护你的软件。

## 集成Phalcon与PHPUnit

如果你还没有安装phpunit，你可以使用下面的composer命令安装：

```bash
composer require phpunit/phpunit:^5.0
```

或手动添加到`composer.json`：

```json
{
    "require-dev": {
        "phpunit/phpunit": "^5.0"
    }
}
```

一旦PHPUnit安装完成，就可以在项目的根目录下创建一个叫`tests`的目录：

    app/
    public/
    tests/


接下来，我们需要一个'helper'文件来引导应用进行单元测试。


## PHPUnit助手文件

需要一个助手文件来引导应用程序运行测试。我们准备了一个示例文件，把这个文件放到你的`tests/`目录并命名为`TestHelper.php`。

```php
<?php

use Phalcon\Di;
use Phalcon\Di\FactoryDefault;
use Phalcon\Loader;

ini_set("display_errors", 1);
error_reporting(E_ALL);

define("ROOT_PATH", __DIR__);

set_include_path(
    ROOT_PATH . PATH_SEPARATOR . get_include_path()
);

// Required for phalcon/incubator
include __DIR__ . "/../vendor/autoload.php";

// Use the application autoloader to autoload the classes
// Autoload the dependencies found in composer
$loader = new Loader();

$loader->registerDirs(
    [
        ROOT_PATH,
    ]
);

$loader->register();

$di = new FactoryDefault();

Di::reset();

// Add any needed services to the DI here

Di::setDefault($di);
```

你要测试你库中的任何组件，将它们添加到自动加载程序中，或者从主应用程序中使用自动加载程序。

为帮助你构建单元测试，我们制作了几个抽象类，你可以使用它们来引导单元测试本身。这些文件位于[Phalcon孵化器](https://github.com/phalcon/incubator)。

你可以将孵化器作为依赖添加并使用：

```bash
composer require phalcon/incubator
```

或者手动将它加到`composer.json`：

```json
{
    "require": {
        "phalcon/incubator": "^3.0"
    }
}
```

你也可以使用上面的仓库连接克隆这个仓库。


## The `phpunit.xml` file

现在，创建`phpunit.xml`文件如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>

<phpunit bootstrap="./TestHelper.php"
         backupGlobals="false"
         backupStaticAttributes="false"
         verbose="true"
         colors="false"
         convertErrorsToExceptions="true"
         convertNoticesToExceptions="true"
         convertWarningsToExceptions="true"
         processIsolation="false"
         stopOnFailure="false"
         syntaxCheck="true">

    <testsuite name="Phalcon - Testsuite">
        <directory>./</directory>
    </testsuite>
</phpunit>
```

修改`phpunit.xml`以适合你的需要并将它保存在`tests`里，这将可以运行`tests`目录下的任何测试。


## 单元测试示例

运行任何单元测试，你需要定义它们。自动加载程序将确保加载适当的文件，因此所有你需要做的就是创建这些文件，phpunit将为你运行测试。

这个示例不包含配置文件，然后大部分的测试用例都需要一个配置文件。你可以将它加到`DI`来取得`UnitTestCase`文件。

首先在你的`tests`目录创建一个叫`UnitTestCase.php`的基础单元测试：

```php
<?php

use Phalcon\Di;
use Phalcon\Test\UnitTestCase as PhalconTestCase;

abstract class UnitTestCase extends PhalconTestCase
{
    /**
     * @var bool
     */
    private $_loaded = false;

    public function setUp()
    {
        parent::setUp();

        // Load any additional services that might be required during testing
        $di = Di::getDefault();

        // Get any DI components here. If you have a config, be sure to pass it to the parent

        $this->setDi($di);

        $this->_loaded = true;
    }

    /**
     * Check if the test case is setup properly
     *
     * @throws \PHPUnit_Framework_IncompleteTestError;
     */
    public function __destruct()
    {
        if (!$this->_loaded) {
            throw new \PHPUnit_Framework_IncompleteTestError(
                "Please run parent::setUp()."
            );
        }
    }
}
```

使用命名空间来隔离你的单元测试一直时个好主意，对于这个测试我们创建命名空间`Test`。因此创建一个叫`tests\Test\UnitTest.php`文件：

```php
<?php

namespace Test;

/**
 * Class UnitTest
 */
class UnitTest extends \UnitTestCase
{
    public function testTestCase()
    {
        $this->assertEquals(
            "works",
            "works",
            "This is OK"
        );

        $this->assertEquals(
            "works",
            "works1",
            "This will fail"
        );
    }
}
```

现在，当你从`tests`目录用命令行执行`phpunit`命令时，你将得到下面的输出：

```bash
$ phpunit
PHPUnit 3.7.23 by Sebastian Bergmann.

Configuration read from /var/www/tests/phpunit.xml

Time: 3 ms, Memory: 3.25Mb

There was 1 failure:

1) Test\UnitTest::testTestCase
This will fail
Failed asserting that two strings are equal.
--- Expected
+++ Actual
@@ @@
-'works'
+'works1'

/var/www/tests/Test/UnitTest.php:25

FAILURES!
Tests: 1, Assertions: 2, Failures: 1.
```

现在你可以开始构建你自己的单元测试了。你可以查看[好指南](http://blog.stevensanderson.com/2009/08/24/writing-great-unit-tests-best-and-worst-practises/)。如果你不熟悉PHPUnit，我们也推荐你阅读PHPUnit文档。