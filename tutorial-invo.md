# 教程: INVO

在第二篇教程中，我们将解释一个更完整的应用程序，以便更好地理解使用Phalcon的方法。INVO是我们创建的示例应用程序之一。INVO是一个小型网站，允许用户生成发票，并执行其他任务，如管理客户和产品。你可以从 [Github](https://github.com/phalcon/invo)克隆它的代码。

INVO利用客户端框架[Bootstrap](http://gebootstrap.com/)开发完成。尽管应用没有生成实际的发票，但它仍然是一个展示框架如何工作的示例。


## 项目结构

一旦克隆这个项目，在项目的文档根将看到下面的结构：

```bash
invo/
    app/
        config/
        controllers/
        forms/
        library/
        logs/
        models/
        plugins/
        views/
    cache/
        volt/
    docs/
    public/
        css/
        fonts/
        js/
    schemas/
```

如你所知，Phalcon不为应用程序开发设置特定的文件结构。这个项目有一个简单的MVC结构和一个公共文档根。

一旦你在浏览器打开你的应用`http://localhost/invo`，你将看到如下内容：

![](https://docs.phalconphp.com/images/content/tutorial-invo-1.png)

应用程序分为两个部分：端和后端。前端是一个公共区域，游客可以在这里接收有关INVO的信息，并请求联系信息。后端是一个管理区域，注册用户可以管理他们的产品和客户。


## 路由

INVO使用内置的 [Router](/[[language]]/[[version]]/routing) 组件提供的标准路由。这些路由匹配如下模式：`/:controller/:action/:params`。这意味着URI的第一部分是控制器，第二部分是控制器操作，剩余部分是参数。

下面的路由 `/session/register`执行控制器 `SessionController`和它的操作`registerAction`。


## 配置

INVO有一个用于设置应用中一般参数的配置文件。这个文件位于 `app/config/config.ini`， 它在应用引导文件（`public/index.php`）非常靠前的几行被加载：

```php
<?php

use Phalcon\Config\Adapter\Ini as ConfigIni;

// ...

// Read the configuration
$config = new ConfigIni(
    APP_PATH . 'app/config/config.ini'
);

```

[Phalcon Config](config.md) (`Phalcon\Config`)允许我们以面向对象方式操作这个配置文件。本例中，我们使用ini配置文件，但Phalcon通过[adapters](config.md)也支持其他文件类型。这个配置文件包括以下设置：

```ini
[database]
host     = localhost
username = root
password = secret
name     = invo

[application]
controllersDir = app/controllers/
modelsDir      = app/models/
viewsDir       = app/views/
pluginsDir     = app/plugins/
formsDir       = app/forms/
libraryDir     = app/library/
baseUri        = /invo/
```

Phalcon没有任何预先定义的设置约定。节帮助我们按照适当的方式组织选项。在这个文件中有两个稍后将使用到的节：application和database。


## 自动加载器

在引导文件（`public/index`）出现的第二部分是自动加载器：

```php
<?php

/**
 * Auto-loader configuration
 */
require APP_PATH . 'app/config/loader.php';
```

自动加载器注册了一组目录，应用将在这些目录里查找它最终需要的类。

```php
<?php

$loader = new Phalcon\Loader();

// We're a registering a set of directories taken from the configuration file
$loader->registerDirs(
    [
        APP_PATH . $config->application->controllersDir,
        APP_PATH . $config->application->pluginsDir,
        APP_PATH . $config->application->libraryDir,
        APP_PATH . $config->application->modelsDir,
        APP_PATH . $config->application->formsDir,
    ]
);

$loader->register();
```

注意，以上代码注册了配置文件中定义的目录，唯一没被注册的目录是viewsDir，因为它包含HTML+PHP文件但没有类。也要注意，我们使用了一个叫APP_PATH的常量。这个常量在引导文件 (`public/index.php`) 中定义，允许我们引用项目的根目录：

```php
<?php

// ...

define(
    'APP_PATH',
    realpath('..') . '/'
);
```


## 注册服务

引导文件中需要的另一个文件是 (`app/config/services.php`)。这个文件允许我们组织INVO用到的服务。

```php
<?php

/**
 * Load application services
 */
require APP_PATH . 'app/config/services.php';
```

服务注册是通过延迟加载所需组件的闭包实现的：

```php
<?php

use Phalcon\Mvc\Url as UrlProvider;

// ...

/**
 * The URL component is used to generate all kind of URLs in the application
 */
$di->set(
    'url',
    function () use ($config) {
        $url = new UrlProvider();

        $url->setBaseUri(
            $config->application->baseUri
        );

        return $url;
    }
);
```

后面我们将深入讨论这个文件。


## 处理请求

如果我们跳到文件（`public/index.php`）的末尾，可以看到请求最后是由`Phalcon\Mvc\Application`处理的，它初始化并执行所需的一切来让应用运行：

```php
<?php

use Phalcon\Mvc\Application;

// ...

$application = new Application($di);

$response = $application->handle();

$response->send();
```

## 依赖注入

上面代码块的第一行中，Application类构造器接受变量`$di`作为参数。这个变量的目的是什么？Phalcon是一个高度解耦的框架，所以我们需要一个能让所有东西协同工作的组件，这个组件就是`Phalcon\Di`。它是一个服务容器，还执行依赖项注入和服务定位，在应用程序需要的时候实例化所有组件。

在容器中注册服务有很多方法。在INVO中，大多数服务都是使用匿名函数/闭包进行注册的。由于这一点，对象以一种惰性的方式被实例化，从而减少了应用程序所需要的资源。

例如，在下面的片段中，会话服务被注册。只有当应用程序需要访问会话数据时，才会调用匿名函数。

```php
<?php

use Phalcon\Session\Adapter\Files as Session;

// ...

// Start the session the first time a component requests the session service
$di->set(
    'session',
    function () {
        $session = new Session();

        $session->start();

        return $session;
    }
);
```

在这里，我们可以自由地更改适配器、执行额外的初始化以及更多的操作。注意，该服务是使用名称`session`注册的。这是一种约定，它允许框架在服务容器中标识活动服务。

一个请求可能使用许多服务，单独注册每个服务可能是一个非常麻烦的任务。由于这个原因，框架提供了一个名为`Phalcon\Di\FactoryDefault`的``Phalcon\Di`的变种，它的任务是注册所有服务以提供一个全栈框架。

```php
<?php

use Phalcon\Di\FactoryDefault;

// ...

// The FactoryDefault Dependency Injector automatically registers the
// right services providing a full-stack framework
$di = new FactoryDefault();
```

它使用框架提供的组件作为标准来注册大多数服务。如果我们需要覆盖某个服务的定义，我们可以像上面的`session`或`url`那样重新设置它。这就是变量`$di`存在的原因。


## 登录到应用

`log in`设施将允许我们在后端控制器上工作。后端控制器和前端控制器之间的分离只是逻辑上的。所有控制器都位于同一个目录(`app/controller/`)中。

要进入系统，用户必须有一个有效的用户名和密码。用户信息存储在数据库`invo`中的`users`表中。

在开始会话之前，我们需要在应用程序中配置好到数据库的连接。在服务容器中，建立了一个带连接信息的名为`db`的服务。与自动加载器一样，我们再次从配置文件中获取参数，以配置服务：

```php
<?php

use Phalcon\Db\Adapter\Pdo\Mysql as DbAdapter;

// ...

// Database connection is created based on parameters defined in the configuration file
$di->set(
    'db',
    function () use ($config) {
        return new DbAdapter(
            [
                'host'     => $config->database->host,
                'username' => $config->database->username,
                'password' => $config->database->password,
                'dbname'   => $config->database->name,
            ]
        );
    }
);
```

这里，我们返回一个MySQL连接适配器的实例，如果需要，你可以执行额外的操作，如添加一个日志记录器，一个分析器，或更改适配器，根据你的需要设置它。

下面的简单表单（`app/views/session/index.volt`）请求登录信息。我们已经删除了一些HTML代码以使示例更简洁：

```twig
{{ form('session/start') }}
    <fieldset>
        <div>
            <label for='email'>
                Username/Email
            </label>

            <div>
                {{ text_field('email') }}
            </div>
        </div>

        <div>
            <label for='password'>
                Password
            </label>

            <div>
                {{ password_field('password') }}
            </div>
        </div>

        <div>
            {{ submit_button('Login') }}
        </div>
    </fieldset>
{{ endForm() }}
```

我们开始用[Volt](volt.md)，而不是像以前的教程那样使用原始的PHP。Volt是由Jinjia出品的一款内置模版引擎，提供了一种更简单和友好的语法来创建模版。用不了多久你就会熟悉Volt了。

`SessionController::startAction`函数（`app/controllers/SessionController.php）的任务是验证表单中输入的数据，包括检查是否是数据库中一个合法的用户。

```php
<?php

class SessionController extends ControllerBase
{
    // ...

    private function _registerSession($user)
    {
        $this->session->set(
            'auth',
            [
                'id'   => $user->id,
                'name' => $user->name,
            ]
        );
    }

    /**
     * This action authenticate and logs a user into the application
     */
    public function startAction()
    {
        if ($this->request->isPost()) {
            // Get the data from the user
            $email    = $this->request->getPost('email');
            $password = $this->request->getPost('password');

            // Find the user in the database
            $user = Users::findFirst(
                [
                    "(email = :email: OR username = :email:) AND password = :password: AND active = 'Y'",
                    'bind' => [
                        'email'    => $email,
                        'password' => sha1($password),
                    ]
                ]
            );

            if ($user !== false) {
                $this->_registerSession($user);

                $this->flash->success(
                    'Welcome ' . $user->name
                );

                // Forward to the 'invoices' controller if the user is valid
                return $this->dispatcher->forward(
                    [
                        'controller' => 'invoices',
                        'action'     => 'index',
                    ]
                );
            }

            $this->flash->error(
                'Wrong email/password'
            );
        }

        // Forward to the login form again
        return $this->dispatcher->forward(
            [
                'controller' => 'session',
                'action'     => 'index',
            ]
        );
    }
}
```

为简单起见，我们使用[SHA1](http://php.net/manual/zh/function.sha1.php)在数据库中存储密码散列，然而，不推荐在真实的应用程序中使用这个算法，用[bcrypt](security.md)代替。

注意，在控制器中可以访问多个公共属性，比如：`$this->flash`、`$this->$request`或`$this->session`。这些是早先在在服务容器(`app/config/services.php`)中定义的服务。当第一次访问它们时，它们被作为控制器的一部分注入。这些服务是`shared`的，这意味着无论在哪里调用它们，我们总是访问相同的实例。例如，我们在这里调用`session`服务，然后将用户标识存储在变量`auth`中：

```php
<?php

$this->session->set(
    'auth',
    [
        'id'   => $user->id,
        'name' => $user->name,
    ]
);
```

这一节的另一个重要方面是如何验证用户是合法的，首先我们验证请求是否使用方法`POST`：

```php
<?php

if ($this->request->isPost()) {
    // ...
}
```

然后，我们从表单接收参数：

```php
<?php

$email    = $this->request->getPost('email');
$password = $this->request->getPost('password');
```

现在，我们必须验证是否有一个相同的用户名/邮箱和密码的用户：

```php
<?php

$user = Users::findFirst(
    [
        "(email = :email: OR username = :email:) AND password = :password: AND active = 'Y'",
        'bind' => [
            'email'    => $email,
            'password' => sha1($password),
        ]
    ]
);
```

注意“绑定参数”的使用，占位符`:email:`和`:password:`放在值应该在的位置，然后使用参数`bind`绑定值。这样就可以安全地替换那些列的值，而不会有SQL注入的风险。

如果用户是合法的，我们在会话中注册并将其转发给仪表板：

```php
<?php

if ($user !== false) {
    $this->_registerSession($user);

    $this->flash->success(
        'Welcome ' . $user->name
    );

    return $this->dispatcher->forward(
        [
            'controller' => 'invoices',
            'action'     => 'index',
        ]
    );
}
```

如果用户不存在，我们让用户回到显示表单的地方：

```php
<?php

return $this->dispatcher->forward(
    [
        'controller' => 'session',
        'action'     => 'index',
    ]
);
```


## 后端安全

后端是一个只有注册用户才能访问的私有区域。因此，必须检查只有注册用户才能访问这些控制器。如果你没有登录到应用程序，并且尝试访问，例如，产品控制器(它是私有的)，你将看到这样的屏幕：

![](https://docs.phalconphp.com/images/content/tutorial-invo-2.png)

每当有人试图访问任何控制器/操作时，应用程序就会验证当前角色(在会话中)是否有访问它的权限，否则它将显示类似上述的消息，并将流程转发到主页。

现在让我们看看应用程序是如何实现这一点的。首先要知道的是，有一个组件叫[Dispatcher](dispatcher.md)。它被告知由[Routing](routing.md)组件所找到的路由。然后，它负责加载适当的控制器并执行相应的操作。

通常，框架会自动创建分派器。在我们的例子中，我们希望在执行所需的操作之前执行一个验证，检查用户是否可以访问它。为了实现这一点，我们已经通过在引导程序中创建一个函数来替换组件：

```php
<?php

use Phalcon\Mvc\Dispatcher;

// ...

/**
 * MVC dispatcher
 */
$di->set(
    'dispatcher',
    function () {
        // ...

        $dispatcher = new Dispatcher();

        return $dispatcher;
    }
);
```

我们现在已经完全控制了应用程序中使用的分派器。框架中的许多组件触发事件，这些事件允许我们修改它们的内部操作流。由于依赖注入器组件充当组件的粘合剂，一个名为[EventsManager](events.md)的新组件允许我们拦截组件生成的事件，将事件路由到侦听器。

### 事件管理

[EventsManager](/events.md)允许我们给特定类型的事件附加侦听器。我们现在感兴趣的类型是“dispatch”。下面的代码过滤了分派器产生的所有事件：

```php
<?php

use Phalcon\Mvc\Dispatcher;
use Phalcon\Events\Manager as EventsManager;

$di->set(
    'dispatcher',
    function () {
        // Create an events manager
        $eventsManager = new EventsManager();

        // Listen for events produced in the dispatcher using the Security plugin
        $eventsManager->attach(
            'dispatch:beforeExecuteRoute',
            new SecurityPlugin()
        );

        // Handle exceptions and not-found exceptions using NotFoundPlugin
        $eventsManager->attach(
            'dispatch:beforeException',
            new NotFoundPlugin()
        );

        $dispatcher = new Dispatcher();

        // Assign the events manager to the dispatcher
        $dispatcher->setEventsManager($eventsManager);

        return $dispatcher;
    }
);
```

当一个叫`beforeExecuteRoute`被触发时，下面的插件将得到通知：

```php
<?php

/**
 * Check if the user is allowed to access certain action using the SecurityPlugin
 */
$eventsManager->attach(
    'dispatch:beforeExecuteRoute',
    new SecurityPlugin()
);
```

当`beforeException`被触发，另外一个插件将得到通知：

```php
<?php

/**
 * Handle exceptions and not-found exceptions using NotFoundPlugin
 */
$eventsManager->attach(
    'dispatch:beforeException',
    new NotFoundPlugin()
);
```

SecurityPlugin是一个位于 (`app/plugins/SecurityPlugin.php`)的类。这个类实现了方法 `beforeExecuteRoute`。这个名字与分派器里的一个事件的名字相同：

```php
<?php

use Phalcon\Events\Event;
use Phalcon\Mvc\User\Plugin;
use Phalcon\Mvc\Dispatcher;

class SecurityPlugin extends Plugin
{
    // ...

    public function beforeExecuteRoute(Event $event, Dispatcher $dispatcher)
    {
        // ...
    }
}
```

钩子事件总是接收两个参数。第一个参数包含所生成事件的上下文信息(`$event`)，第二个参数是产生事件本身的对象(`$dispatcher`)。插件继承`Phalcon\Mvc\User\Plugin`类并不是强制的，但通过这样做，他们可以更容易地访问应用中可用的服务。

现在，我们正在验证当前会话中的角色，检查用户是否使用ACL列表访问。如果用户没有访问权限，我们将重定向到主屏幕，如前所述:

```php
<?php

use Phalcon\Acl;
use Phalcon\Events\Event;
use Phalcon\Mvc\User\Plugin;
use Phalcon\Mvc\Dispatcher;

class SecurityPlugin extends Plugin
{
    // ...

    public function beforeExecuteRoute(Event $event, Dispatcher $dispatcher)
    {
        // Check whether the 'auth' variable exists in session to define the active role
        $auth = $this->session->get('auth');

        if (!$auth) {
            $role = 'Guests';
        } else {
            $role = 'Users';
        }

        // Take the active controller/action from the dispatcher
        $controller = $dispatcher->getControllerName();
        $action     = $dispatcher->getActionName();

        // Obtain the ACL list
        $acl = $this->getAcl();

        // Check if the Role have access to the controller (resource)
        $allowed = $acl->isAllowed($role, $controller, $action);

        if (!$allowed) {
            // If he doesn't have access forward him to the index controller
            $this->flash->error(
                "You don't have access to this module"
            );

            $dispatcher->forward(
                [
                    'controller' => 'index',
                    'action'     => 'index',
                ]
            );

            // Returning 'false' we tell to the dispatcher to stop the current operation
            return false;
        }
    }
}
```


### 取得ACL列表

在上面的示例中，我们使用方法`$this-getAcl()`获得了ACL。这种方法也被在Plugin中实现。现在，我们将逐步解释如何构建访问控制列表(ACL)：

```php
<?php

use Phalcon\Acl;
use Phalcon\Acl\Role;
use Phalcon\Acl\Adapter\Memory as AclList;

// Create the ACL
$acl = new AclList();

// The default action is DENY access
$acl->setDefaultAction(
    Acl::DENY
);

// Register two roles, Users is registered users
// and guests are users without a defined identity
$roles = [
    'users'  => new Role('Users'),
    'guests' => new Role('Guests'),
];

foreach ($roles as $role) {
    $acl->addRole($role);
}
```

现在，我们分别为每个区域定义了资源。控制器名称是资源，它们的操作是对资源的访问：

```php
<?php

use Phalcon\Acl\Resource;

// ...

// Private area resources (backend)
$privateResources = [
    'companies'    => ['index', 'search', 'new', 'edit', 'save', 'create', 'delete'],
    'products'     => ['index', 'search', 'new', 'edit', 'save', 'create', 'delete'],
    'producttypes' => ['index', 'search', 'new', 'edit', 'save', 'create', 'delete'],
    'invoices'     => ['index', 'profile'],
];

foreach ($privateResources as $resourceName => $actions) {
    $acl->addResource(
        new Resource($resourceName),
        $actions
    );
}



// Public area resources (frontend)
$publicResources = [
    'index'    => ['index'],
    'about'    => ['index'],
    'register' => ['index'],
    'errors'   => ['show404', 'show500'],
    'session'  => ['index', 'register', 'start', 'end'],
    'contact'  => ['index', 'send'],
];

foreach ($publicResources as $resourceName => $actions) {
    $acl->addResource(
        new Resource($resourceName),
        $actions
    );
}
```

ACL现在了解已存在的控制器及其相关操作。角色`users`可以访问前端和后端的所有资源，角色`Guests`只能进入公共区域：

```php
<?php

// Grant access to public areas to both users and guests
foreach ($roles as $role) {
    foreach ($publicResources as $resource => $actions) {
        $acl->allow(
            $role->getName(),
            $resource,
            '*'
        );
    }
}

// Grant access to private area only to role Users
foreach ($privateResources as $resource => $actions) {
    foreach ($actions as $action) {
        $acl->allow(
            'Users',
            $resource,
            $action
        );
    }
}
```


## 使用CRUD

后端通常提供表单以允许用户操作数据。继续对解释INVO，我们现在处理CRUD的创建，这是一个非常常见的任务，Phalcon将帮助你使用表单、验证器、分页器等等。

INVO里操作数据（公司，产品和产品类型）的大多数选项使用一个基本的、普通的[CRUD](http://en.wikipedia.org/wiki/Create,_read,_update_and_delete)(创建、读取、更新和删除)开发。每个CRUD包含以下文件：

```bash
invo/
    app/
        controllers/
            ProductsController.php
        models/
            Products.php
        forms/
            ProductsForm.php
        views/
            products/
                edit.volt
                index.volt
                new.volt
                search.volt
```

每个控制器有下列操作：

```php
<?php

class ProductsController extends ControllerBase
{
    /**
     * The start action, it shows the 'search' view
     */
    public function indexAction()
    {
        // ...
    }

    /**
     * Execute the 'search' based on the criteria sent from the 'index'
     * Returning a paginator for the results
     */
    public function searchAction()
    {
        // ...
    }

    /**
     * Shows the view to create a 'new' product
     */
    public function newAction()
    {
        // ...
    }

    /**
     * Shows the view to 'edit' an existing product
     */
    public function editAction()
    {
        // ...
    }

    /**
     * Creates a product based on the data entered in the 'new' action
     */
    public function createAction()
    {
        // ...
    }

    /**
     * Updates a product based on the data entered in the 'edit' action
     */
    public function saveAction()
    {
        // ...
    }

    /**
     * Deletes an existing product
     */
    public function deleteAction($id)
    {
        // ...
    }
}
```


## 搜索表单

每个CRUD都以搜索表单开始。这个表单显示了该表拥有的每个字段(产品)，允许用户为任何字段创建一个搜索条件。`products`表与`products_types`表有关系。在本例中，我们之前查询了该表中的记录，以处理该字段的搜索：

```php
<?php

/**
 * The start action, it shows the 'search' view
 */
public function indexAction()
{
    $this->persistent->searchParams = null;

    $this->view->form = new ProductsForm();
}
```

`ProductsForm`表单 (`app/forms/ProductsForm.php`)的实例被传递给视图。这个表单定义了用户可见的字段：

```php
<?php

use Phalcon\Forms\Form;
use Phalcon\Forms\Element\Text;
use Phalcon\Forms\Element\Hidden;
use Phalcon\Forms\Element\Select;
use Phalcon\Validation\Validator\Email;
use Phalcon\Validation\Validator\PresenceOf;
use Phalcon\Validation\Validator\Numericality;

class ProductsForm extends Form
{
    /**
     * Initialize the products form
     */
    public function initialize($entity = null, $options = [])
    {
        if (!isset($options['edit'])) {
            $element = new Text('id');
            $element->setLabel('Id');
            $this->add($element);
        } else {
            $this->add(new Hidden('id'));
        }

        $name = new Text('name');
        $name->setLabel('Name');
        $name->setFilters(
            [
                'striptags',
                'string',
            ]
        );
        $name->addValidators(
            [
                new PresenceOf(
                    [
                        'message' => 'Name is required',
                    ]
                )
            ]
        );
        $this->add($name);

        $type = new Select(
            'profilesId',
            ProductTypes::find(),
            [
                'using'      => [
                    'id',
                    'name',
                ],
                'useEmpty'   => true,
                'emptyText'  => '...',
                'emptyValue' => '',
            ]
        );

        $this->add($type);

        $price = new Text('price');
        $price->setLabel('Price');
        $price->setFilters(
            [
                'float',
            ]
        );
        $price->addValidators(
            [
                new PresenceOf(
                    [
                        'message' => 'Price is required',
                    ]
                ),
                new Numericality(
                    [
                        'message' => 'Price is required',
                    ]
                ),
            ]
        );
        $this->add($price);
    }
}
```

表单是使用面向对象的方案、基于[Forms](forms.md)组件提供的元素声明的。每个元素都遵循着几乎相同的结构：

```php
<?php

// Create the element
$name = new Text('name');

// Set its label
$name->setLabel('Name');

// Before validating the element apply these filters
$name->setFilters(
    [
        'striptags',
        'string',
    ]
);

// Apply this validators
$name->addValidators(
    [
        new PresenceOf(
            [
                'message' => 'Name is required',
            ]
        )
    ]
);

// Add the element to the form
$this->add($name);
```

这个表单里也用到其他元素：

```php
<?php

// Add a hidden input to the form
$this->add(
    new Hidden('id')
);

// ...

$productTypes = ProductTypes::find();

// Add a HTML Select (list) to the form
// and fill it with data from 'product_types'
$type = new Select(
    'profilesId',
    $productTypes,
    [
        'using'      => [
            'id',
            'name',
        ],
        'useEmpty'   => true,
        'emptyText'  => '...',
        'emptyValue' => '',
    ]
);
```

注意 `ProductTypes::find()` 包含需用`Phalcon\Tag::select()`填充到SELECT标签的数据。一旦表单被传给视图，它将被渲染并展现给用户：

```twig
{{ form('products/search') }}

    <h2>
        Search products
    </h2>

    <fieldset>

        {% for element in form %}
            <div class='control-group'>
                {{ element.label(['class': 'control-label']) }}

                <div class='controls'>
                    {{ element }}
                </div>
            </div>
        {% endfor %}



        <div class='control-group'>
            {{ submit_button('Search', 'class': 'btn btn-primary') }}
        </div>

    </fieldset>

{{ endForm() }}
```

这生成如下HTML：

```html
<form action='/invo/products/search' method='post'>

    <h2>
        Search products
    </h2>

    <fieldset>

        <div class='control-group'>
            <label for='id' class='control-label'>Id</label>

            <div class='controls'>
                <input type='text' id='id' name='id' />
            </div>
        </div>

        <div class='control-group'>
            <label for='name' class='control-label'>Name</label>

            <div class='controls'>
                <input type='text' id='name' name='name' />
            </div>
        </div>

        <div class='control-group'>
            <label for='profilesId' class='control-label'>profilesId</label>

            <div class='controls'>
                <select id='profilesId' name='profilesId'>
                    <option value=''>...</option>
                    <option value='1'>Vegetables</option>
                    <option value='2'>Fruits</option>
                </select>
            </div>
        </div>

        <div class='control-group'>
            <label for='price' class='control-label'>Price</label>

            <div class='controls'>
                <input type='text' id='price' name='price' />
            </div>
        </div>

        <div class='control-group'>
            <input type='submit' value='Search' class='btn btn-primary' />
        </div>

    </fieldset>

</form>
```

当提交表单时，根据用户输入的数据，在执行搜索的控制器中执行`search`操作。


## 执行搜索

`search`操作有两个行为。当通过POST访问时，它基于发送的数据执行一个搜索，但当通过GET访问时，它在分页器中移动当前页。对于区分HTTP方法，我们使用 [Request](request.md) 组件：

```php
<?php

/**
 * Execute the 'search' based on the criteria sent from the 'index'
 * Returning a paginator for the results
 */
public function searchAction()
{
    if ($this->request->isPost()) {
        // Create the query conditions
    } else {
        // Paginate using the existing conditions
    }

    // ...
}
```

借助`Phalcon\Mvc\Model\Criteria`的帮助，我们可以根据从表单发送的数据类型和值来智能地创建搜索条件:

```php
<?php

$query = Criteria::fromInput(
    $this->di,
    'Products',
    $this->request->getPost()
);
```

该方法验证哪些值不为空或NULL，并将它们纳入到创建搜索条件的考量：

* 如果字段数据类型是text或类text(char、varchar、text等)，它使用`like`操作符这样的SQL来过滤结果。
* 如果数据类型不是text或类text，它使用操作符`=`。

另外， `Criteria`忽略所有`$_POST`中与数据表字段不匹配的变量，且值使用`bound parameters`自动转义。

现在，我们存储控制器会话包产生的参数：

```php
<?php

$this->persistent->searchParams = $query->getParams();
```

会话包，是控制器中的一个特殊的属性，它使用会话服务在不同的请求中持续存在。当访问时，这个属性注入一个`Phalcon\Session\Bag`实例， 这个实例在每个控制器中是独立的。

然后，基于构建的参数我们执行查询：

```php
<?php

$products = Products::find($parameters);

if (count($products) === 0) {
    $this->flash->notice(
        'The search did not found any products'
    );

    return $this->dispatcher->forward(
        [
            'controller' => 'products',
            'action'     => 'index',
        ]
    );
}
```

如果搜索不返回任何产品，我们将用户再次转发到索引操作（index action）。让我们假设搜索返回了结果，然后创建一个分页器来轻松地导航它们：

```php
<?php

use Phalcon\Paginator\Adapter\Model as Paginator;

// ...

$paginator = new Paginator(
    [
        'data'  => $products,   // Data to paginate
        'limit' => 5,           // Rows per page
        'page'  => $numberPage, // Active page
    ]
);

// Get active page in the paginator
$page = $paginator->getPaginate();
```

最后，我们传递返回的页到视图：

```php
<?php

$this->view->page = $page;
```

在 (`app/views/products/search.volt`)视图里，我们遍历与当前页面对应的结果，向用户显示当前页面中的每一行：

```twig
{% for product in page.items %}
    {% if loop.first %}
        <table>
            <thead>
                <tr>
                    <th>Id</th>
                    <th>Product Type</th>
                    <th>Name</th>
                    <th>Price</th>
                    <th>Active</th>
                </tr>
            </thead>
            <tbody>
    {% endif %}

    <tr>
        <td>
            {{ product.id }}
        </td>

        <td>
            {{ product.getProductTypes().name }}
        </td>

        <td>
            {{ product.name }}
        </td>

        <td>
            {{ '%.2f'|format(product.price) }}
        </td>

        <td>
            {{ product.getActiveDetail() }}
        </td>

        <td width='7%'>
            {{ link_to('products/edit/' ~ product.id, 'Edit') }}
        </td>

        <td width='7%'>
            {{ link_to('products/delete/' ~ product.id, 'Delete') }}
        </td>
    </tr>

    {% if loop.last %}
            </tbody>
            <tbody>
                <tr>
                    <td colspan='7'>
                        <div>
                            {{ link_to('products/search', 'First') }}
                            {{ link_to('products/search?page=' ~ page.before, 'Previous') }}
                            {{ link_to('products/search?page=' ~ page.next, 'Next') }}
                            {{ link_to('products/search?page=' ~ page.last, 'Last') }}
                            <span class='help-inline'>{{ page.current }} of {{ page.total_pages }}</span>
                        </div>
                    </td>
                </tr>
            </tbody>
        </table>
    {% endif %}
{% else %}
    No products are recorded
{% endfor %}
```

在上面的例子中有很多值得详细说明的东西。首先，当前页面上的活动项是使用Volt的`for`来遍历的。Volt为PHP的`foreach`提供了更简单的语法。

```twig
{% for product in page.items %}
```

这与PHP里的用法是相同的：

```php
<?php foreach ($page->items as $product) { ?>
```

整个`for`块提供如下：

```twig
{% for product in page.items %}
    {% if loop.first %}
        Executed before the first product in the loop
    {% endif %}

    Executed for every product of page.items

    {% if loop.last %}
        Executed after the last product is loop
    {% endif %}
{% else %}
    Executed if page.items does not have any products
{% endfor %}
```

现在你可以回到视图，找出每个块都在做什么。`product`的每一个字段都有相应的输出：

```twig
<tr>
    <td>
        {{ product.id }}
    </td>

    <td>
        {{ product.productTypes.name }}
    </td>

    <td>
        {{ product.name }}
    </td>

    <td>
        {{ '%.2f'|format(product.price) }}
    </td>

    <td>
        {{ product.getActiveDetail() }}
    </td>

    <td width='7%'>
        {{ link_to('products/edit/' ~ product.id, 'Edit') }}
    </td>

    <td width='7%'>
        {{ link_to('products/delete/' ~ product.id, 'Delete') }}
    </td>
</tr>
```

就像我们之前看到的那样，使用`product.id`和在PHP中是一样的：`$product->id`，`product.name`也一样。其他字段的渲染则不同，作为示例，让我们专注于`product.productTypes.name`。要理解这部分，我们必须检查Products模型(`app/models/Products.php`)：

```php
<?php

use Phalcon\Mvc\Model;

/**
 * Products
 */
class Products extends Model
{
    // ...

    /**
     * Products initializer
     */
    public function initialize()
    {
        $this->belongsTo(
            'product_types_id',
            'ProductTypes',
            'id',
            [
                'reusable' => true,
            ]
        );
    }

    // ...
}
```

模型可能有一个叫做 `initialize()`的方法， 这个方法每次请求都仅被调用一次，它服务于ORM来初始化一个模型。在本例中，’Products'通过定义该模型与另一个名为'ProductTypes'的模型有一对多的关系来实现初始化。

```php
<?php

$this->belongsTo(
    'product_types_id',
    'ProductTypes',
    'id',
    [
        'reusable' => true,
    ]
);
```

也就是说，`Products`中的本地属性`product_types_id`与`ProductTypes`模型的属性`id`有一对多的关系。通过定义这种关系，我们可以使用以下方法来访问产品类型的名称：

```twig
<td>{{ product.productTypes.name }}</td>
```

字段 `price`使用Volt过滤器格式化化后输出：

```twig
<td>{{ '%.2f'|format(product.price) }}</td>
```

在普通PHP中，这将是：


```php
<?php echo sprintf('%.2f', $product->price) ?>
```

使用模型中实现的助手来打印出产品是否活跃：

```php
<td>{{ product.getActiveDetail() }}</td>
```

这个方法在模型中定义。


## 创建和更新记录

现在让我们看看CRUD是如何创建和更新记录的。从`new`和`edit`视图中，用户输入的数据被发送到`create`和`save`操作，这些操作分别执行`creating`和`updating`产品。

在创建案例中，我们恢复提交的数据并将它们分配给一个新产品实例：

```php
<?php

/**
 * Creates a product based on the data entered in the 'new' action
 */
public function createAction()
{
    if (!$this->request->isPost()) {
        return $this->dispatcher->forward(
            [
                'controller' => 'products',
                'action'     => 'index',
            ]
        );
    }

    $form = new ProductsForm();

    $product = new Products();

    $product->id               = $this->request->getPost('id', 'int');
    $product->product_types_id = $this->request->getPost('product_types_id', 'int');
    $product->name             = $this->request->getPost('name', 'striptags');
    $product->price            = $this->request->getPost('price', 'double');
    $product->active           = $this->request->getPost('active');

    // ...
}
```

还记得我们在产品表单中定义的过滤器吗？数据在被分配给对象`$product`之前被过滤。这种过滤是可选的；ORM还会转义输入数据，并根据字段类型执行额外的转换：

```php
<?php

// ...

$name = new Text('name');

$name->setLabel('Name');

// Filters for name
$name->setFilters(
    [
        'striptags',
        'string',
    ]
);

// Validators for name
$name->addValidators(
    [
        new PresenceOf(
            [
                'message' => 'Name is required',
            ]
        )
    ]
);

$this->add($name);
```

保存时，我们就会知道数据是否符合业务逻辑和在`ProductsForm`表单（`app/forms/ProductsForm.php`）中实现的验证规则：

```php
<?php

// ...

$form = new ProductsForm();

$product = new Products();

// Validate the input
$data = $this->request->getPost();

if (!$form->isValid($data, $product)) {
    $messages = $form->getMessages();

    foreach ($messages as $message) {
        $this->flash->error($message);
    }

    return $this->dispatcher->forward(
        [
            'controller' => 'products',
            'action'     => 'new',
        ]
    );
}
```

最后，如果表单没有返回任何验证消息，我们可以保存产品实例：

```php
<?php

// ...

if ($product->save() === false) {
    $messages = $product->getMessages();

    foreach ($messages as $message) {
        $this->flash->error($message);
    }

    return $this->dispatcher->forward(
        [
            'controller' => 'products',
            'action'     => 'new',
        ]
    );
}

$form->clear();

$this->flash->success(
    'Product was created successfully'
);

return $this->dispatcher->forward(
    [
        'controller' => 'products',
        'action'     => 'index',
    ]
);
```

现在，在更新产品的情况下，我们必须首先向用户提供当前已编辑的记录中的数据：

```php
<?php

/**
 * Edits a product based on its id
 */
public function editAction($id)
{
    if (!$this->request->isPost()) {
        $product = Products::findFirstById($id);

        if (!$product) {
            $this->flash->error(
                'Product was not found'
            );

            return $this->dispatcher->forward(
                [
                    'controller' => 'products',
                    'action'     => 'index',
                ]
            );
        }

        $this->view->form = new ProductsForm(
            $product,
            [
                'edit' => true,
            ]
        );
    }
}
```

通过传递模型作为第一个参数，将所找到的数据绑定到表单。感谢这一点，用户可以更改任何值，然后通过`save`操作将其发送回数据库：

```php
<?php

/**
 * Updates a product based on the data entered in the 'edit' action
 */
public function saveAction()
{
    if (!$this->request->isPost()) {
        return $this->dispatcher->forward(
            [
                'controller' => 'products',
                'action'     => 'index',
            ]
        );
    }

    $id = $this->request->getPost('id', 'int');

    $product = Products::findFirstById($id);

    if (!$product) {
        $this->flash->error(
            'Product does not exist'
        );

        return $this->dispatcher->forward(
            [
                'controller' => 'products',
                'action'     => 'index',
            ]
        );
    }

    $form = new ProductsForm();

    $data = $this->request->getPost();

    if (!$form->isValid($data, $product)) {
        $messages = $form->getMessages();

        foreach ($messages as $message) {
            $this->flash->error($message);
        }

        return $this->dispatcher->forward(
            [
                'controller' => 'products',
                'action'     => 'new',
            ]
        );
    }

    if ($product->save() === false) {
        $messages = $product->getMessages();

        foreach ($messages as $message) {
            $this->flash->error($message);
        }

        return $this->dispatcher->forward(
            [
                'controller' => 'products',
                'action'     => 'new',
            ]
        );
    }

    $form->clear();

    $this->flash->success(
        'Product was updated successfully'
    );

    return $this->dispatcher->forward(
        [
            'controller' => 'products',
            'action'     => 'index',
        ]
    );
}
```


## 用户组件

应用程序的所有UI元素和可视化样式都是通过[Bootstrap](http://getbootstrap.com/)实现的。一些元素，例如导航条会根据应用程序的状态变化。例如，在右上角，如果用户登录到应用程序，则链接`Log in/Sign Up`变为`Log out`。

该应用程序的这一部分是在组件元素(`app/library/Elements.php`)中实现的。

```php
<?php

use Phalcon\Mvc\User\Component;

class Elements extends Component
{
    public function getMenu()
    {
        // ...
    }

    public function getTabs()
    {
        // ...
    }
}
```

这个类继承了`Phalcon\Mvc\User\Component`。用这个类继承`Component`并不是强制的，但是它有助于更快地访问应用程序服务。现在，我们将在服务容器中注册我们的第一个用户组件:

```php
<?php

// Register a user component
$di->set(
    'elements',
    function () {
        return new Elements();
    }
);
```

作为视图中的控制器、插件或组件，该组件还可以访问在容器中注册的服务，并且只需访问与先前注册的服务相同的属性:

```twig
<div class='navbar navbar-fixed-top'>
    <div class='navbar-inner'>
        <div class='container'>
            <a class='btn btn-navbar' data-toggle='collapse' data-target='.nav-collapse'>
                <span class='icon-bar'></span>
                <span class='icon-bar'></span>
                <span class='icon-bar'></span>
            </a>

            <a class='brand' href='#'>INVO</a>

            {{ elements.getMenu() }}
        </div>
    </div>
</div>

<div class='container'>
    {{ content() }}

    <hr>

    <footer>
        <p>&copy; Company 2017</p>
    </footer>
</div>
```

重要的部分是：

```twig
{{ elements.getMenu() }}
```


## 动态改变标题

当你在一个选项和另一个选项间浏览时，会看到标题会动态地显示我们当前工作的位置。这是在每个控制器的初始化时实现的：

```php
<?php

class ProductsController extends ControllerBase
{
    public function initialize()
    {
        // Set the document title
        $this->tag->setTitle(
            'Manage your product types'
        );

        parent::initialize();
    }

    // ...
}
```

注意，方法`parent::initialize()`也被调用了，它给标题添加了更多的数据：

```php
<?php

use Phalcon\Mvc\Controller;

class ControllerBase extends Controller
{
    protected function initialize()
    {
        // Prepend the application name to the title
        $this->tag->prependTitle('INVO | ');
    }

    // ...
}
```

最后，在主视图（`app/views/index.volt`）中，标题被打印出来：

```php
<!DOCTYPE html>
<html>
    <head>
        <?php echo $this->tag->getTitle(); ?>
    </head>

    <!-- ... -->
</html>
```