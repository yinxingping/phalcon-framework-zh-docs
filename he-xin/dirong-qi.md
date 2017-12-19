# 依赖注入/服务定位

## DI含义

---

下面的例子有点长，但是它试图解释为什么Phalcon要使用服务定位和依赖注入。首先，假设我们正在开发一个名为`SomeComponent`的组件，我们的组件有一个依赖项，它就是是与数据库的连接。

在第一个示例中，连接是在组件内部创建的。尽管这是一个完全有效的实现，但是我们不能更改连接参数或数据库系统的类型，因为组件只能按创建的那样工作。

```php
<?php

class SomeComponent
{
    /**
     * The instantiation of the connection is hardcoded inside
     * the component, therefore it's difficult replace it externally
     * or change its behavior
     */
    public function someDbTask()
    {
        $connection = new Connection(
            [
                'host'     => 'localhost',
                'username' => 'root',
                'password' => 'secret',
                'dbname'   => 'invo',
            ]
        );

        // ...
    }
}

$some = new SomeComponent();

$some->someDbTask();
```

为了解决这个问题，我们创建了一个 `setter` ，它在使用前从外部注入依赖。这也是一个有效的实现，但也有它的缺点：

```php
<?php

class SomeComponent
{
    private $connection;

    /**
     * Sets the connection externally
     *
     * @param Connection $connection
     */
    public function setConnection(Connection $connection)
    {
        $this->connection = $connection;
    }

    public function someDbTask()
    {
        $connection = $this->connection;

        // ...
    }
}

$some = new SomeComponent();

// Create the connection
$connection = new Connection(
    [
        'host'     => 'localhost',
        'username' => 'root',
        'password' => 'secret',
        'dbname'   => 'invo',
    ]
);

// Inject the connection in the component
$some->setConnection($connection);

$some->someDbTask();
```

现在，考虑我们在应用程序的不同部分中使用该组件，然后在将其传递给组件之前，需要多次创建连接。使用全局注册模式，我们可以在那里存储连接对象，并在需要时重用它。

```php
<?php

class Registry
{
    /**
     * Returns the connection
     */
    public static function getConnection()
    {
        return new Connection(
            [
                'host'     => 'localhost',
                'username' => 'root',
                'password' => 'secret',
                'dbname'   => 'invo',
            ]
        );
    }
}

class SomeComponent
{
    protected $connection;

    /**
     * Sets the connection externally
     *
     * @param Connection $connection
     */
    public function setConnection(Connection $connection)
    {
        $this->connection = $connection;
    }

    public function someDbTask()
    {
        $connection = $this->connection;

        // ...
    }
}

$some = new SomeComponent();

// Pass the connection defined in the registry
$some->setConnection(Registry::getConnection());

$some->someDbTask();
```

现在，让我们想象一下，我们必须在组件中实现两个方法，第一个总是需要创建一个新的连接，第二个总是需要使用一个共享连接：

```php
<?php

class Registry
{
    protected static $connection;

    /**
     * Creates a connection
     *
     * @return Connection
     */
    protected static function createConnection(): Connection
    {
        return new Connection(
            [
                'host'     => 'localhost',
                'username' => 'root',
                'password' => 'secret',
                'dbname'   => 'invo',
            ]
        );
    }

    /**
     * Creates a connection only once and returns it
     *
     * @return Connection
     */
    public static function getSharedConnection(): Connection
    {
        if (self::$connection === null) {
            self::$connection = self::createConnection();
        }

        return self::$connection;
    }

    /**
     * Always returns a new connection
     *
     * @return Connection
     */
    public static function getNewConnection(): Connection
    {
        return self::createConnection();
    }
}

class SomeComponent
{
    protected $connection;

    /**
     * Sets the connection externally
     *
     * @param Connection $connection
     */
    public function setConnection(Connection $connection)
    {
        $this->connection = $connection;
    }

    /**
     * This method always needs the shared connection
     */
    public function someDbTask()
    {
        $connection = $this->connection;

        // ...
    }

    /**
     * This method always needs a new connection
     *
     * @param Connection $connection
     */
    public function someOtherDbTask(Connection $connection)
    {

    }
}

$some = new SomeComponent();

// This injects the shared connection
$some->setConnection(
    Registry::getSharedConnection()
);

$some->someDbTask();

// Here, we always pass a new connection as parameter
$some->someOtherDbTask(
    Registry::getNewConnection()
);
```

到目前为止，我们已经看到了依赖注入如何解决了我们的问题。将依赖项作为参数传递，而不是在代码内部创建它们，这使我们的应用程序更易于维护和解耦。但是，从长期来看，这种依赖注入有一些缺点。

例如，如果组件有很多依赖项，我们将需要创建多个`setter`参数来传递依赖项，或者创建一个通过多个参数传递它们的构造器。另外，使用组件之前创建依赖项，每次，都使我们的代码不像我们希望的那样易于维护：

```php
<?php

// Create the dependencies or retrieve them from the registry
$connection = new Connection();
$session    = new Session();
$fileSystem = new FileSystem();
$filter     = new Filter();
$selector   = new Selector();

// Pass them as constructor parameters
$some = new SomeComponent($connection, $session, $fileSystem, $filter, $selector);

// ... Or using setters
$some->setConnection($connection);
$some->setSession($session);
$some->setFileSystem($fileSystem);
$some->setFilter($filter);
$some->setSelector($selector);
```

如果我们必须在应用程序的很多地方创建这个对象，将来，如果不需要任何依赖项时，我们需要遍历整个代码库，在注入代码的任何构造函数或`setter`中删除参数。为了解决这个问题，我们再次返回到一个全局注册表来创建组件。但是，在创建对象之前，它添加了一个新的抽象层：

```php
<?php

class SomeComponent
{
    // ...

    /**
     * Define a factory method to create SomeComponent instances injecting its dependencies
     */
    public static function factory()
    {
        $connection = new Connection();
        $session    = new Session();
        $fileSystem = new FileSystem();
        $filter     = new Filter();
        $selector   = new Selector();

        return new self($connection, $session, $fileSystem, $filter, $selector);
    }
}
```

现在，我们发现自己回到了起点，我们再次构建了组件内部的依赖关系！必须找到一个解决办法，使我们避免反复陷入不良实践。

解决这些问题的一种实用而优雅的方法是使用一个依赖容器。容器充当我们之前看到的全局注册表，将依赖容器作为一个桥梁来获得依赖项，使我们可以减少组件的复杂性：

```php
<?php

use Phalcon\Di;
use Phalcon\DiInterface;

class SomeComponent
{
    protected $di;

    public function __construct(DiInterface $di)
    {
        $this->di = $di;
    }

    public function someDbTask()
    {
        // Get the connection service
        // Always returns a new connection
        $connection = $this->di->get('db');
    }

    public function someOtherDbTask()
    {
        // Get a shared connection service,
        // this will return the same connection every time
        $connection = $this->di->getShared('db');

        // This method also requires an input filtering service
        $filter = $this->di->get('filter');
    }
}

$di = new Di();

// Register a 'db' service in the container
$di->set(
    'db',
    function () {
        return new Connection(
            [
                'host'     => 'localhost',
                'username' => 'root',
                'password' => 'secret',
                'dbname'   => 'invo',
            ]
        );
    }
);

// Register a 'filter' service in the container
$di->set(
    'filter',
    function () {
        return new Filter();
    }
);

// Register a 'session' service in the container
$di->set(
    'session',
    function () {
        return new Session();
    }
);

// Pass the service container as unique parameter
$some = new SomeComponent($di);

$some->someDbTask();
```

组件现在可以简单地访问它需要的服务，如果它不需要一个服务，该服务甚至不用初始化，节省资源。该组件现在已经高度解耦了，例如，我们可以替换连接创建的方式、它们的行为或它们的任何其他方面，而这不会影响组件。

[Phalcon\Di](https://docs.phalconphp.com/zh/3.2/api/Phalcon_Di) 是一个实现依赖注入和服务定位的组件，它本身就是一个容器。

由于Phalcon是高度解耦的，因此，[Phalcon\Di](https://docs.phalconphp.com/zh/3.2/api/Phalcon_Di) 对这个框架不同组件的集成至关重要。开发人员还可以使用该组件来注入依赖关系，并管理应用程序中使用的不同类的全局实例。

基本上，这个组件实现了控制反转（[Inversion of Control](http://en.wikipedia.org/wiki/Inversion_of_control)）模式。应用这一点，对象不会使用`setter`或构造函数来接收它们的依赖项，而是请求依赖注入器服务。这降低了整体的复杂性，因为只有一种方法可以在组件中获得所需的依赖项。

此外，该模式增加了代码的可测试性，从而降低了错误的可能性。

## 在容器中注册服务

---

框架本身或开发人员可以注册服务。当组件a需要组件B\(或其类的一个实例\)来操作时，它可以从容器中请求组件B，而不是创建一个新的实例组件B。

这种工作方式给了我们很多优势：

* 我们可以很容易地将组件替换为自己或第三方创建的组件

* 我们可以完全控制对象初始化，允许我们在将这些对象交付到组件之前，根据需要设置这些对象

* 我们可以以结构化和统一的方式获得组件的全局实例

服务可以使用多种定义类型：

### 简单注册

如前面看到的，有多种方式来注册服务。这些我们称之为简单类型：

#### 字符串

这种类型要求一个合法的类名称，返回指定类的一个对象，如果这个类没有被加载，则会使用一个自动加载器实例化它。这种类型的定义不允许为类的构造器或参数指定参数：

```php
<?php

// Return new Phalcon\Http\Request();
$di->set(
    'request',
    'Phalcon\Http\Request'
);
```

#### 类实例

这种类型需要一个对象。由于对象不需要解析，它已经是对象了，所以我们可以说它实际上不是一个依赖注入，但是如果你想强制返回的依赖项始终是相同的对象/值，那么它是很有用的。

```php
<?php

use Phalcon\Http\Request;

// Return new Phalcon\Http\Request();
$di->set(
    'request',
    new Request()
);
```

#### 闭包/匿名函数

这种方法提供了更大的自由来构建依赖项，但是，完全不改变依赖项定义情况下，在外部更改一些参数是很困难的：

```php
<?php

use Phalcon\Db\Adapter\Pdo\Mysql as PdoMysql;

$di->set(
    'db',
    function () {
        return new PdoMysql(
            [
                'host'     => 'localhost',
                'username' => 'root',
                'password' => 'secret',
                'dbname'   => 'blog',
            ]
        );
    }
);
```

可以通过给闭包环境传递额外的变量来克服部分限制：

```php
<?php

use Phalcon\Config;
use Phalcon\Db\Adapter\Pdo\Mysql as PdoMysql;

$config = new Config(
    [
        'host'     => '127.0.0.1',
        'username' => 'user',
        'password' => 'pass',
        'dbname'   => 'my_database',
    ]
);

// Using the $config variable in the current scope
$di->set(
    'db',
    function () use ($config) {
        return new PdoMysql(
            [
                'host'     => $config->host,
                'username' => $config->username,
                'password' => $config->password,
                'dbname'   => $config->name,
            ]
        );
    }
);
```

你也可以使用 `get()` 方法访问其他的DI服务：

```php
<?php

use Phalcon\Config;
use Phalcon\Db\Adapter\Pdo\Mysql as PdoMysql;

$di->set(
    'config',
    function () {
        return new Config(
            [
                'host'     => '127.0.0.1',
                'username' => 'user',
                'password' => 'pass',
                'dbname'   => 'my_database',
            ]
        );
    }
);

// Using the 'config' service from the DI
$di->set(
    'db',
    function () {
        $config = $this->get('config');

        return new PdoMysql(
            [
                'host'     => $config->host,
                'username' => $config->username,
                'password' => $config->password,
                'dbname'   => $config->name,
            ]
        );
    }
);
```

### 复杂注册

如果需要更改服务的定义，而不需要实例化/解析服务，那么，我们需要使用数组语法来定义服务。使用数组定义定义服务可以稍微详细一点：

```php
<?php

use Phalcon\Logger\Adapter\File as LoggerFile;

// Register a service 'logger' with a class name and its parameters
$di->set(
    'logger',
    [
        'className' => 'Phalcon\Logger\Adapter\File',
        'arguments' => [
            [
                'type'  => 'parameter',
                'value' => '../apps/logs/error.log',
            ]
        ]
    ]
);

// Using an anonymous function
$di->set(
    'logger',
    function () {
        return new LoggerFile('../apps/logs/error.log');
    }
);
```

上面的两个服务注册都产生相同的结果。然而，数组定义允许在需要时更改服务参数：

```php
<?php

// Change the service class name
$di
    ->getService('logger')
    ->setClassName('MyCustomLogger');

// Change the first parameter without instantiating the logger
$di
    ->getService('logger')
    ->setParameter(
        0,
        [
            'type'  => 'parameter',
            'value' => '../apps/logs/error.log',
        ]
    );
```

除了使用数组语法之外，还可以使用三种类型的依赖注入：

#### 构造器注入

这种注入类型将依赖项/参数传递给类构造器。假设我们有下面的组件：

```php
<?php

namespace SomeApp;

use Phalcon\Http\Response;

class SomeComponent
{
    /**
     * @var Response
     */
    protected $response;

    protected $someFlag;

    public function __construct(Response $response, $someFlag)
    {
        $this->response = $response;
        $this->someFlag = $someFlag;
    }
}
```

服务可以这样注册：

```php
<?php

$di->set(
    'response',
    [
        'className' => 'Phalcon\Http\Response'
    ]
);

$di->set(
    'someComponent',
    [
        'className' => 'SomeApp\SomeComponent',
        'arguments' => [
            [
                'type' => 'service',
                'name' => 'response',
            ],
            [
                'type'  => 'parameter',
                'value' => true,
            ],
        ]
    ]
);
```

‘response'服务（[Phalcon\Http\Response](https://docs.phalconphp.com/zh/3.2/api/Phalcon_Http_Response)）被解析并作为构造器的第一个参数，而第二个参数是个布尔值（true），按它应该的方法传递。

#### Setter注入

类可以有注入可选依赖项的 `setter` 方法，我们之前的类可以修改为接受使用 `setter` 的依赖项：

```php
<?php

namespace SomeApp;

use Phalcon\Http\Response;

class SomeComponent
{
    /**
     * @var Response
     */
    protected $response;

    protected $someFlag;

    public function setResponse(Response $response)
    {
        $this->response = $response;
    }

    public function setFlag($someFlag)
    {
        $this->someFlag = $someFlag;
    }
}
```

带 setter 注入的服务可以如下注册：

```php
<?php

$di->set(
    'response',
    [
        'className' => 'Phalcon\Http\Response',
    ]
);

$di->set(
    'someComponent',
    [
        'className' => 'SomeApp\SomeComponent',
        'calls'     => [
            [
                'method'    => 'setResponse',
                'arguments' => [
                    [
                        'type' => 'service',
                        'name' => 'response',
                    ]
                ]
            ],
            [
                'method'    => 'setFlag',
                'arguments' => [
                    [
                        'type'  => 'parameter',
                        'value' => true,
                    ]
                ]
            ]
        ]
    ]
);
```

#### 属性注册

一个不太常见的策略是将依赖项或参数直接注入到类的公共属性中：

```php
<?php

namespace SomeApp;

use Phalcon\Http\Response;

class SomeComponent
{
    /**
     * @var Response
     */
    public $response;

    public $someFlag;
}
```

带属性注入的服务可以如下注册：

```php
<?php

$di->set(
    'response',
    [
        'className' => 'Phalcon\Http\Response',
    ]
);

$di->set(
    'someComponent',
    [
        'className'  => 'SomeApp\SomeComponent',
        'properties' => [
            [
                'name'  => 'response',
                'value' => [
                    'type' => 'service',
                    'name' => 'response',
                ],
            ],
            [
                'name'  => 'someFlag',
                'value' => [
                    'type'  => 'parameter',
                    'value' => true,
                ],
            ]
        ]
    ]
);
```

支持的参数类型包括以下：

| Type | Description | Example |
| :--- | :--- | :--- |
| parameter | Represents a literal value to be passed as parameter | `php['type' =>'parameter', 'value' =>1234]` |
| service | Represents another service in the service container | `php['type' =>'service', 'name' =>'request']` |
| instance | Represents an object that must be built dynamically | `php['type' =>'instance', 'className' =>'DateTime', 'arguments' => ['now']]` |

  
解析一个定义复杂的服务可能比前面的简单定义稍微慢一些。但是，这些提供了更健壮的定义和注入服务的方法。



通过允许混合不同类型的定义，每个人都可以根据应用程序的需要来决定什么是最合适的注册服务。

### 数组语法

数组也允许注册服务：

```php
<?php

use Phalcon\Di;
use Phalcon\Http\Request;

// Create the Dependency Injector Container
$di = new Di();

// By its class name
$di['request'] = 'Phalcon\Http\Request';

// Using an anonymous function, the instance will be lazy loaded
$di['request'] = function () {
    return new Request();
};

// Registering an instance directly
$di['request'] = new Request();

// Using an array definition
$di['request'] = [
    'className' => 'Phalcon\Http\Request',
];
```

在上面的示例中，当框架需要访问请求数据时，它将请求在容器中标识为“request”的服务。反过来，容器将返回所需服务的一个实例。开发人员可能最终会在需要的时候替换组件。



用于设置/注册服务的每个方法\(在上面的示例中演示\)都有其优点和缺点。这取决于开发人员和特定的需求，这些需求将指定使用哪一个。



通过字符串设置服务很简单，但是缺乏灵活性。使用数组设置服务提供了更大的灵活性，但是使代码更加复杂。lambda函数在这两者之间是一个很好的平衡，但是可能会导致比预期更多的维护。



[Phalcon\Di](https://docs.phalconphp.com/zh/3.2/api/Phalcon_Di) 为它所存储的每一个服务都提供了延迟加载的功能，除非开发人员选择直接实例化对象并将其存储在容器中，否则存储在该对象中的任何对象\(通过数组、字符串等\)将被延迟加载，也就是说只有在请求时才实例化。

### 从YMAL文件加载服务

这个特性可以让您在 `yaml` 文件中设置服务，或者只使用简单的php。例如，您可以使用 `yaml` 文件来加载服务：

```yaml
config:
  className: \Phalcon\Config
  shared: true
```

```php
<?php

use Phalcon\Di;

$di = new Di();
$di->loadFromYaml('services.yml');
$di->get('config'); // will properly return config service
```

## 解析服务

---

从容器中获取服务只是简单地调用“get”方法，将返回一个新的服务实例：

```php
$request = $di->get('request');
```

或调用魔术方法：

```php
$request = $di->getRequest();
```

或使用数组访问语法：

```php
$request = $di['request'];
```

通过给"get"方法添加一个数组参数将参数传递给构造器：

```php
<?php

// new MyComponent('some-parameter', 'other')
$component = $di->get(
    'MyComponent',
    [
        'some-parameter',
        'other',
    ]
);
```

### 事件

[Phalcon\Di](https://docs.phalconphp.com/zh/3.2/api/Phalcon_Di) 能发送事件给一个



