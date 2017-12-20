# 可重现测试

如果你已经发现了一个bug，那么将相关的重现信息添加到你的问题中是很重要的，这样可以让我们重现bug并更快地修复它。如果你在Github上公开了这个应用程序，请提交存储库地址和问题描述。你还可以使用[Gist](https://gist.github.com/)来发布你想要与我们共享的任何代码。


## 创建一个小脚本

一个小单文件脚本通常是重现一个问题最好的办法：

```php
<?php

$di = new Phalcon\DI\FactoryDefault();

//Register your custom services
$di['session'] = function() {
    $session = new \Phalcon\Session\Adapter\Files();
    $session->start();
    return $session;
};

$di['cookies'] = function() {
    $cookies = new Phalcon\Http\Response\Cookies();
    $cookies->useEncryption(false);
    return $cookies;
};

class SomeClass extends \Phalcon\DI\Injectable
{
    public function someMethod()
    {
        $cookies = $this->getDI()->getCookies();
        $cookies->set("mycookie", "test", time() + 3600, "/");
    }
}

$c = new MyClass;
$c->setDI($di);
$c->someMethod();

$di['cookies']->send();

var_dump($_SESSION);
var_dump($_COOKIE);
```

根据你的应用程序，你可以使用这些框架来创建你自己的脚本并再现bug：

### 数据库

请记得把你如何注册数据库服务添加到脚本：

```php
<?php

$di = new Phalcon\DI\FactoryDefault();

$di->setShared('db', function () {
    return new \Phalcon\Db\Adapter\PDO\Mysql(array(
        'host' => '127.0.0.1',
        'username' => 'root',
        'password' => '',
        'dbname'   => 'test',
        'charset'  => 'utf8',
    ));
});

$result = $di['db']->query('SELECT * FROM customers');

```

### 单/多模块（single/modules）应用

请记得把你如何创建 Phalcon\Mvc\Application 实例添加到脚本：

```php
<?php

$di  = new \Phalcon\DI\FactoryDefault();

//other services

$app = new \Phalcon\Mvc\Application();
$app->setDi($di);

//register modules if any

echo $app->handle->getContent()

```

包含模型和控制器作为测试的部分：

```php
<?php

$di  = new \Phalcon\DI\FactoryDefault();

//other services

$app = new \Phalcon\Mvc\Application();
$app->setDi($di);

class IndexController extends Phalcon\Mvc\Controller
{
    public function indexAction() { 
          /* your content here */
    }
}

class Users extends Phalcon\Mvc\Model
{
}

echo $app->handle->getContent()

```

### 微（micro）应用

依据这个结构创建脚本：

```php
<?php

$di = new \Phalcon\DI\FactoryDefault();

$app = new \Phalcon\Mvc\Micro($di);

//define your routes here

$app->handle();
```


### ORM

你可以提供你自己的数据库模式，或使用任何phalcon测试[数据库](https://github.com/phalcon/cphalcon/tree/master/unit-tests/schemas)甚至更好。根据这个结构创建脚本：

```php
<?php

use Phalcon\DI;
use Phalcon\Events\Manager as EventsManager;
use Phalcon\Db\Adapter\Pdo\Mysql as Connection;
use Phalcon\Mvc\Model\Manager as ModelsManager;
use Phalcon\Mvc\Model\Metadata\Memory as ModelsMetaData;

$eventsManager = new EventsManager();

$di = new DI();

$connection = new Connection(array(
    "host" => "localhost",
    "username" => "root",
    "password" => "",
    "dbname" => "test"
));

$connection->setEventsManager($eventsManager);

$eventsManager->attach('db',
    function ($event, $connection) {
        switch ($event->getType()) {
            case 'beforeQuery':
                echo $connection->getSqlStatement(), "<br>\n";
                break;
        }
    }
);

$di['db'] = $connection;
$di['modelsManager'] = new ModelsManager();
$di['modelsMetadata'] = new ModelsMetadata();

if (!$connection->tableExists('user', 'test')) {
    $connection->execute('CREATE TABLE user (id integer primary key auto_increment, email varchar(120) not null)');
}

class User extends \Phalcon\Mvc\Model
{
    public $id;

    public $email;

    public static function myCustomUserCreator()
    {
        $newUser = new User();
        $newUser->email = 'test';
        if ($newUser->save() == false) {
            return false;
        }
        return $newUser->id;        
    }
}

echo User::myCustomUserCreator();
```