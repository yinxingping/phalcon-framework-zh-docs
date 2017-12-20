# 教程—基础

本教程中，我们将带你从头开始构建一个简单的注册表单应用程序。下面的指南将向你介绍Phalcon框架的设计方面。

如果你想尽快开始，你可以跳过这一章，用我们的[开发工具](devtools-usage.md)自动创建一个Phalcon项目。\(如果你没有经验并碰到困难，推荐你返回这里\)

使用这个指南最好的方法就是跟着做，试着去享受乐趣。您可以在[这里](https://github.com/phalcon/tutorial)获得完整的代码。如果你对某些地方感到不舒服请访问我们的[异议](https://phalcon.link/discord)或[论坛](https://phalcon.link/forum)。   


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

注意：你将不会看到 vendor 目录，因为Phalcon的核心依赖都通过你安装的扩展加载到内存里了。如果你缺失了那一步没有安装Phalcon扩展，继续前请[返回](installation.md)去完成安装。

如果这一切对你来说都是全新的，那么推荐你安装[Phalcon开发工具](devtools-installation.md)，因为它利用PHP内置的服务器让你的应用运行起来，而无需通过在项目的根目录添加 .htrouter来配置web服务器。

如果你想使用Nginx，Apache或Cherokee，[这里](webserver-setup.md)是一些额外的设置。


## 引导

---

你需要创建的第一个文件是引导文件。该文件充当应用程序的入口点和配置。在这个文件中，你可以实现组件的初始化以及应用程序的行为。

这个文件处理3件事情：

* 注册自动加载组件
* 配置服务并将它们注册到依赖注入上下文中
* 解决应用程序的HTTP请求

#### 自动加载器

自动加载器利用一个通过Phalcon扩展运行的兼容[PSR-4](http://www.php-fig.org/psr/psr-4/)的文件加载器，通常会把应用的控制器和模型加入到自动加载器中。你可以注册目录，它将在应用程序的命名空间中搜索文件。\(如果你想阅读其他使用自动加载器的方式，你可以先看[这里](loader.md)\)

首先，让我们注册我们的应用的控制器和模型目录。别忘了从 [Phalcon\Loader](api/Phalcon_Loader.md) 包含加载器。

**public/index.php**

```php
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

由于Phalcon是松耦合框架，服务由框架依赖管理器注册，所以它们可以被自动注入到IoC容器包裹着的组件和服务中。你将会经常碰到依赖注入的术语 **DI**，依赖注入和控制反转（**IoC**）听起来可能是一个复杂的特性，但在Phalcon里，它们非常简单和实用。Phalcon的IoC容器由以下概念组成：

* 服务容器（Service Container）：我们在全局存储应用程序所需服务的一个“包”
* 服务或组件（Service or Component）：要被注入组件库的数据处理对象

如果你仍然对细节感兴趣，请阅读 [Martin Fowler](https://martinfowler.com/articles/injection.html) 的文章。

每当框架需要一个组件或服务时，它就会使用一个约定的名称向容器请求服务。别忘记引入 `Phalcon\Di` 来设置服务容器。

服务可以用几种方式注册，但对于我们这个教程，我们将使用一个[匿名函数](http://php.net/manual/zh/functions.anonymous.php) 。

#### 工厂默认

[Phalcon\Di\FactoryDefault](api/Phalcon_Di_FactoryDefault.md) 是 [Phalcon\Di](api/Phalcon_Di.md) 的一个变体。为了让事情更容易，它会自动注册Phalcon带的大部分组件。我们推荐你手动注册你的服务，但使用这个可以帮忙降低使用依赖管理的门槛。后面，一旦你对这个概念更熟悉了，就可以手动指定。

**public/index.php**

```php
<?php

use Phalcon\Di\FactoryDefault;

// ...

// Create a DI
$di = new FactoryDefault();
```

在接下来的部分中，我们注册了 “view" 服务来指出框架在哪个目录下查找视图文件。由于视图与类不对应，它们不能用自动加载器来处理。

**public/index.php**

```php
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

接下来，我们注册一个基础URI，以便所有由Phalcon产生的URI都包含前面设置的"tutorial"文件夹。当我们后面用类 [Phalcon\Tag](api/Phalcon_Tag.md) 生成链接时这会很重要。

**public/index.php**

```php
<?php

use Phalcon\Mvc\Url as UrlProvider;

// ...

// Setup a base URI so that all generated URIs include the "tutorial" folder
$di->set(
    'url',
    function () {
        $url = new UrlProvider();
        $url->setBaseUri('/');
        return $url;
    }
);
```

#### 处理应用请求

在这个文件的最后部分，我们找到 `Phalcon\Mvc\Application`。它的目的是初始化请求环境，路由进来的请求，然后转发任何发现的操作（actions）；它聚合任何响应，并在程序完成时返回它们。

**public/index.php**

```php
<?php

use Phalcon\Mvc\Application;

// ...

$application = new Application($di);
$response = $application->handle();
$response->send();
```

#### 把以上所有代码放一起

**public/index.php**

```php
<?php

use Phalcon\Loader;
use Phalcon\Mvc\View;
use Phalcon\Mvc\Application;
use Phalcon\Di\FactoryDefault;
use Phalcon\Mvc\Url as UrlProvider;

// Define some absolute path constants to aid in locating resources
define('BASE_PATH', dirname(__DIR__));
define('APP_PATH', BASE_PATH . '/app');

// Register an autoloader
$loader = new Loader();

$loader->registerDirs(
    [
        APP_PATH . '/controllers/',
        APP_PATH . '/models/',
    ]
);

$loader->register();

// Create a DI
$di = new FactoryDefault();

// Setup the view component
$di->set(
    'view',
    function () {
        $view = new View();
        $view->setViewsDir(APP_PATH . '/views/');
        return $view;
    }
);

// Setup a base URI so that all generated URIs include the "tutorial" folder
$di->set(
    'url',
    function () {
        $url = new UrlProvider();
        $url->setBaseUri('/');
        return $url;
    }
);

$application = new Application($di);

try {
    // Handle the request
    $response = $application->handle();

    $response->send();
} catch (\Exception $e) {
    echo 'Exception: ', $e->getMessage();
}
```

如同你看到的，引导文件非常短，我们不需要包含任何额外的文件。祝贺你用不到30行代码创建了一个灵活的MVC应用程序。


## 创建控制器

---

默认Phalcon会寻找一个叫`IndexController`的控制器，当在请求中没有添加控制器（controller）或操作（action）时它就是开始点（如 [http://localhost:8000/](http://localhost:8000)）。一个`IndexController`和它的`IndexAction`应该像下面的例子：

**app/controllers/IndexController.php**

```php
<?php

use Phalcon\Mvc\Controller;

class IndexController extends Controller
{
    public function indexAction()
    {
        echo '<h1>Hello!</h1>';
    }
}
```

控制器类名必须带后缀"Controller"，并且控制器操作（action）名必须带后缀"Action"。如果你从浏览器访问这个应用，你应该看到类似这里的内容：

![](https://docs.phalconphp.com/images/content/tutorial-basic-1.png)

恭喜你，Phalcon已经玩起来了！


## 将输出发送到视图

---

将输出从控制器发送到屏幕上是必要的，但这并不理想，因为MVC社区中的大多数纯粹主义者都将证明这一结果。所有的内容都必须传递到负责在屏幕上输出数据的视图。Phalcon会在一个以最后执行的控制器命名的目录中查找与最后执行的操作同名的视图。在我们的事例中：

**app/views/index/index.phtml**

```php
<?php echo "<h1>Hello!</h1>";
```

我们的控制器现在有一个空的操作定义：

**app/controllers/IndexController.php**

```php
<?php

use Phalcon\Mvc\Controller;

class IndexController extends Controller
{
    public function indexAction()
    {

    }
}
```

浏览器输出应该是一样的。当Action执行结束时 `Phalcon\Mvc\View` 静态组件被自动创建。学习关于视图的更多用法请点[这里](views.md)。


## 设计一个注册表单

---

现在让我们改变 index.phtml 视图文件，增加链接到一个叫做“signup"的新控制器，目的是在我们的应用里允许用户注册。

**app/views/index/index.phtml**

```php
<?php

echo "<h1>Hello!</h1>";

echo PHP_EOL;

echo PHP_EOL;

echo $this->tag->linkTo(
    "signup",
    "Sign Up Here!"
);
```

生成的HTML代码显示了一个链接到新控制器的HTML标签"a"：

**app/views/index/index.phtml Rendered**

```html
<h1>Hello!</h1>

<a href="/signup">Sign Up Here!</a>
```

我们使用类 [Phalcon\Tag](api/Phalcon_Tag.md) 生成标记。这是一个实用程序类，它允许我们用框架约定来构建HTML标记。由于这个类也是在DI中注册的一个服务，所以我们使用 `$this->tag` 来访问它。

你可以在[这里](tag.md)找到更详细的关于HTML生成的文章。

![](https://docs.phalconphp.com/images/content/tutorial-basic-2.png)

这里是Signup控制器的代码：

**app/controllers/SignupController.php**

```php
<?php

use Phalcon\Mvc\Controller;

class SignupController extends Controller
{
    public function indexAction()
    {

    }
}
```

空的 indexAction 直接传递带表单定义的视图：

**app/views/signup/index.phtml**

```php
<h2>Sign up using this form</h2>

<?php echo $this->tag->form("signup/register"); ?>

    <p>
        <label for="name">Name</label>
        <?php echo $this->tag->textField("name"); ?>
    </p>

    <p>
        <label for="email">E-Mail</label>
        <?php echo $this->tag->textField("email"); ?>
    </p>

    <p>
        <?php echo $this->tag->submitButton("Register"); ?>
    </p>

</form>
```

在你的浏览器上将显示如下：

![](https://docs.phalconphp.com/images/content/tutorial-basic-3.png)

[Phalcon\Tag](api/Phalcon_Tag.md) 也提供了有用的方法来构建表单元素。

例如，`Phalcon\Tag::form()`方法只接收一个参数，一个指向应用中controller/action的相对URI.

通过点击“Register"按钮，你会看到从框架抛出一个异常，指出我们在控制器"signup"中缺少"register"操作。我们的 `public/index.php` 文件抛出这个异常：

```
Exception: Action "register" was not found on handler "signup"
```

实现这个方法可以移除这个异常：

**app/controllers/SignupController.php**

```php
<?php

use Phalcon\Mvc\Controller;

class SignupController extends Controller
{
    public function indexAction()
    {

    }

    public function registerAction()
    {

    }
}
```

如果你再次点击“Register"按钮，你会看到一个空白页。用户输入的name和emal应该保存在一个数据库里。根据MVC指南，数据库交互必须通过模型进行，以确保干净的面向对象的代码。

## 创建模型

---

对于PHP来说，Phalcon带来了第一个用C语言写的ORM，它简化了开发的复杂性。

在创建我们的第一个模型之前，我们需要创建一个数据库表，以便将它映射。一个存储注册用户的简单表可以这样创建：

**create\_users\_table.sql**

```sql
CREATE TABLE `users` (
    `id`    int(10)     unsigned NOT NULL AUTO_INCREMENT,
    `name`  varchar(70)          NOT NULL,
    `email` varchar(70)          NOT NULL,

    PRIMARY KEY (`id`)
);
```

模型应该位于 app/models 目录。映射到"users"表的模型：

**app/models/Users.php**

```php
<?php

use Phalcon\Mvc\Model;

class Users extends Model
{
    public $id;
    public $name;
    public $email;
}
```

## 建立数据库连接

---

为了使用数据库连接，然后通过我们的模型访问数据，我们需要在引导过程中指定它。数据库连接只是我们的应用程序拥有的另一个服务，它可以用于几个组件：

**public/index.php**

```php
<?php

use Phalcon\Db\Adapter\Pdo\Mysql as DbAdapter;

// Setup the database service
$di->set(
    'db',
    function () {
        return new DbAdapter(
            [
                'host'     => '127.0.0.1',
                'username' => 'root',
                'password' => 'secret',
                'dbname'   => 'tutorial1',
            ]
        );
    }
);
```

有了正确的数据库参数，我们的模型就可以工作并与应用程序的其余部分进行交互。

## 使用模型存储数据

---

**app/controllers/SignupController.php**

```php
<?php

use Phalcon\Mvc\Controller;

class SignupController extends Controller
{
    public function indexAction()
    {

    }

    public function registerAction()
    {
        $user = new Users();

        // Store and check for errors
        $success = $user->save(
            $this->request->getPost(),
            [
                "name",
                "email",
            ]
        );

        if ($success) {
            echo "Thanks for registering!";
        } else {
            echo "Sorry, the following problems were generated: ";

            $messages = $user->getMessages();

            foreach ($messages as $message) {
                echo $message->getMessage(), "<br/>";
            }
        }

        $this->view->disable();
    }
}
```

在registerAction的开头，我们从User类创建一个空的user对象，该用户类管理用户的记录。类的公共属性映射到数据库中users表的字段。在新记录中设置相关值，并调用save\(\)将在数据库中存储该记录的数据。save\(\)方法返回一个布尔值，该值表示数据的存储是否成功。

ORM自动地转义输入以阻止SQL注入，因此我们只需要将请求传递给save\(\)方法。

在定义为not null\(required\) 的字段中会自动进行额外的验证。如果我们在注册表单中没有输入任何必需的字段，我们的屏幕将会是这样的：

![](https://docs.phalconphp.com/images/content/tutorial-basic-4.png)

## 结语

---

正如你所看到的，使用Phalcon来构建应用程序是很容易的。事实上，Phalcon从扩展运行可以大大减少项目的占用，同时也大大提高了项目的性能。

如果你已经准备好学习更多的内容，请阅读下面的 [REST教程](tutorial-rest.md)。

