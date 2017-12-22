# 使用视图

视图表示应用程序的用户界面。视图通常是带有嵌入式PHP代码的HTML文件，它们执行与数据表示相关的任务。视图处理向web浏览器或从你的应用程序请求数据的其他工具提供数据的工作。

`Phalcon\Mvc\View` 和 `Phalcon\Mvc\View\Simple` 负责管理你的MVC应用的视图层。

## 集成视图与控制器

当一个特定的控制器完成它的周期时，Phalcon会自动将执行传递给视图组件。视图组件将在视图文件夹中查找一个文件夹，该文件夹与执行的最后一个控制器的名称相同，然后是与执行的最后一个操作同名的文件。例如，如果请求的URL是 http://127.0.0.1/blog/posts/show/301，Phalcon将解析URL如下所示：

| 服务器地址      | 127.0.0.1 |
| ---------- | --------- |
| Phalcon目录  | blog      |
| Controller | posts     |
| Action     | show      |
| Parameter  | 301       |

分派器将查找一个`PostsController`和他的操作`showAction`。 一个简单的控制器文件如下：

```php
<?php

use Phalcon\Mvc\Controller;

class PostsController extends Controller
{
    public function indexAction()
    {

    }

    public function showAction($postId)
    {
        // Pass the $postId parameter to the view
        $this->view->postId = $postId;
    }
}
```

`setVar()`方法允许我们根据需要创建视图变量，这样它们就可以在视图模板中使用。上面的例子演示了如何将`$postId`参数传递给各自的视图模板。


## 分层渲染

`Phalcon\Mvc\View` 支持文件的层次结构，是Phalcon中用于视图渲染的默认组件。这个层次结构允许通用布局(通常使用的视图)，以及用于定义各自的视图模板的控制器。

该组件默认使用PHP自身作为模板引擎，因此视图应该有.phtml扩展名。如果视图目录是 *app/views*，那么视图组件将自动找到这3个视图文件。


| 名称    | 文件                            | 描述                                       |
| ----- | ----------------------------- | ---------------------------------------- |
| 操作视图  | app/views/posts/show.phtml    | 这是与操作相关的视图，只有在执行show操作时才会显示它。            |
| 控制器布局 | app/views/layouts/posts.phtml | 这是与控制器相关的视图，它只会显示在控制器“posts”内执行的每个操作。在布局中实现的所有代码都将被用于该控制器中的所有操作。 |
| 主布局   | app/views/index.phtml         | 这是一个主要的操作，它将显示在应用程序中执行的每个控制器或操作。         |

你不需要实现上面提到的所有文件。 `Phalcon\Mvc\View` 会转移到文件层次结构中的下一级视图。如果所有三个视图文件都被实现，它们将被处理如下：

```php
<!-- app/views/posts/show.phtml -->

<h3>This is show view!</h3>

<p>I have received the parameter <?php echo $postId; ?></p>
```

```php
<!-- app/views/layouts/posts.phtml -->

<h2>This is the "posts" controller layout!</h2>

<?php echo $this->getContent(); ?>
```

```php
<!-- app/views/index.phtml -->
<html>
    <head>
        <title>Example</title>
    </head>
    <body>

        <h1>This is main layout!</h1>

        <?php echo $this->getContent(); ?>

    </body>
</html>
```

注意方法`$this->getContent()`被调用的行。 这个方法指导 `Phalcon\Mvc\View` 在那里注入层次结构执行的之前的视图的内容。对于上例，输出将是：

请求生成的HTML将会是：

```php
<!-- app/views/index.phtml -->
<html>
    <head>
        <title>Example</title>
    </head>
    <body>

        <h1>This is main layout!</h1>

        <!-- app/views/layouts/posts.phtml -->

        <h2>This is the "posts" controller layout!</h2>

        <!-- app/views/posts/show.phtml -->

        <h3>This is show view!</h3>

        <p>I have received the parameter 101</p>

    </body>
</html>
```


### 使用模板

模板是可以用来共享公共视图代码的视图。它们作为控制器布局，所以您需要将它们放置在布局目录中。

模板可以在布局前渲染 (使用 `$this->view->setTemplateBefore()`) 或者可以在布局后渲染 (使用 `this->view->setTemplateAfter()`)。下例中，模板 (`layouts/common.phtml`) 在控制器布局后渲染 (`layouts/posts.phtml`):

```php
<?php

use Phalcon\Mvc\Controller;

class PostsController extends Controller
{
    public function initialize()
    {
        $this->view->setTemplateAfter('common');
    }

    public function lastAction()
    {
        $this->flash->notice(
            'These are the latest posts'
        );
    }
}
```

```php
<!-- app/views/index.phtml -->
<!DOCTYPE html>
<html>
    <head>
        <title>Blog's title</title>
    </head>
    <body>
        <?php echo $this->getContent(); ?>
    </body>
</html>
```

```php
<!-- app/views/layouts/common.phtml -->

<ul class='menu'>
    <li><a href='/'>Home</a></li>
    <li><a href='/articles'>Articles</a></li>
    <li><a href='/contact'>Contact us</a></li>
</ul>

<div class='content'><?php echo $this->getContent(); ?></div>
```

```php
<!-- app/views/layouts/posts.phtml -->

<h1>Blog Title</h1>

<?php echo $this->getContent(); ?>
```

```php
<!-- app/views/posts/last.phtml -->

<article>
    <h2>This is a title</h2>
    <p>This is the post content</p>
</article>

<article>
    <h2>This is another title</h2>
    <p>This is another post content</p>
</article>
```

The final output will be the following:

```php
<!-- app/views/index.phtml -->
<!DOCTYPE html>
<html>
    <head>
        <title>Blog's title</title>
    </head>
    <body>

        <!-- app/views/layouts/common.phtml -->

        <ul class='menu'>
            <li><a href='/'>Home</a></li>
            <li><a href='/articles'>Articles</a></li>
            <li><a href='/contact'>Contact us</a></li>
        </ul>

        <div class='content'>

            <!-- app/views/layouts/posts.phtml -->

            <h1>Blog Title</h1>

            <!-- app/views/posts/last.phtml -->

            <article>
                <h2>This is a title</h2>
                <p>This is the post content</p>
            </article>

            <article>
                <h2>This is another title</h2>
                <p>This is another post content</p>
            </article>

        </div>

    </body>
</html>
```

如果我们使用 `$this->view->setTemplateBefore('common')`，这将是最后的输出：

```php
<!-- app/views/index.phtml -->
<!DOCTYPE html>
<html>
    <head>
        <title>Blog's title</title>
    </head>
    <body>

        <!-- app/views/layouts/posts.phtml -->

        <h1>Blog Title</h1>

        <!-- app/views/layouts/common.phtml -->

        <ul class='menu'>
            <li><a href='/'>Home</a></li>
            <li><a href='/articles'>Articles</a></li>
            <li><a href='/contact'>Contact us</a></li>
        </ul>

        <div class='content'>

            <!-- app/views/posts/last.phtml -->

            <article>
                <h2>This is a title</h2>
                <p>This is the post content</p>
            </article>

            <article>
                <h2>This is another title</h2>
                <p>This is another post content</p>
            </article>

        </div>

    </body>
</html>
```


### 控制渲染级别

如上面所见， `Phalcon\Mvc\View` 支持视图分层。你可能需要控制视图组件生成的渲染级别， 方法 `Phalcon\Mvc\View::setRenderLevel()` 提供了这个功能。

此方法可以从控制器或高级视图层调用，以干扰渲染过程。

```php
<?php

use Phalcon\Mvc\View;
use Phalcon\Mvc\Controller;

class PostsController extends Controller
{
    public function indexAction()
    {

    }

    public function findAction()
    {
        // This is an Ajax response so it doesn't generate any kind of view
        $this->view->setRenderLevel(
            View::LEVEL_NO_RENDER
        );

        // ...
    }

    public function showAction($postId)
    {
        // Shows only the view related to the action
        $this->view->setRenderLevel(
            View::LEVEL_ACTION_VIEW
        );
    }
}
```

可用的渲染级别有：

| 类常量                     | 描述                             |  顺序  |
| ----------------------- | ------------------------------ | :--: |
| `LEVEL_NO_RENDER`       | 指出避免生成任何呈现                     |      |
| `LEVEL_ACTION_VIEW`     | 生成与操作相关的视图的呈现                  |  1   |
| `LEVEL_BEFORE_TEMPLATE` | 生成优先于控制器布局的模板的呈现               |  2   |
| `LEVEL_LAYOUT`          | 生成控制器布局的呈现                     |  3   |
| `LEVEL_AFTER_TEMPLATE`  | 生成控制器布局后模板的呈现                  |  4   |
| `LEVEL_MAIN_LAYOUT`     | 生成主布局的呈现，文件`views/index.phtml` |  5   |


### 禁用渲染级别

你可以永久或临时禁用渲染级别。如果在整个应用程序中不使用该级别，则可能永久禁用该级别。

```php
<?php

use Phalcon\Mvc\View;

$di->set(
    'view',
    function () {
        $view = new View();

        // Disable several levels
        $view->disableLevel(
            [
                View::LEVEL_LAYOUT      => true,
                View::LEVEL_MAIN_LAYOUT => true,
            ]
        );

        return $view;
    },
    true
);
```

或者在应用的某些部分临时禁用：

```php
<?php

use Phalcon\Mvc\View;
use Phalcon\Mvc\Controller;

class PostsController extends Controller
{
    public function indexAction()
    {

    }

    public function findAction()
    {
        $this->view->disableLevel(
            View::LEVEL_MAIN_LAYOUT
        );
    }
}
```


### 选择视图

如上面所提到的，当 `Phalcon\Mvc\View` 被 `Phalcon\Mvc\Application` 管理时，视图渲染与最后执行的控制器和操作相关。你可以使用 `Phalcon\Mvc\View::pick()` 覆盖它：

```php
<?php

use Phalcon\Mvc\Controller;

class ProductsController extends Controller
{
    public function listAction()
    {
        // Pick 'views-dir/products/search' as view to render
        $this->view->pick('products/search');

        // Pick 'views-dir/books/list' as view to render
        $this->view->pick(
            [
                'books',
            ]
        );

        // Pick 'views-dir/products/search' as view to render
        $this->view->pick(
            [
                1 => 'search',
            ]
        );
    }
}
```


### 禁用视图

如果你的控制器在视图中没有产生任何输出(或者甚至没有输出语句)，那么你可以禁用视图组件，避免不必要的处理：

```php
<?php

use Phalcon\Mvc\Controller;

class UsersController extends Controller
{
    public function closeSessionAction()
    {
        // Close session
        // ...

        // Disable the view to avoid rendering
        $this->view->disable();
    }
}
```

或者，你可以返回`false`来产生同样的效果：

```php
<?php

use Phalcon\Mvc\Controller;

class UsersController extends Controller
{
    public function closeSessionAction()
    {
        // ...

        // Disable the view to avoid rendering
        return false;
    }
}
```

你可以返回一个`response`对象来避免人工禁用视图：

```php
<?php

use Phalcon\Mvc\Controller;

class UsersController extends Controller
{
    public function closeSessionAction()
    {
        // Close session
        // ...

        // A HTTP Redirect
        return $this->response->redirect('index/index');
    }
}
```


## 简单渲染

`Phalcon\Mvc\View\Simple`是`Phalcon\Mvc\View`的替代组件。 它保留了`Phalcon\Mvc\View`的大部分理念，但缺乏文件的层次结构，事实上，这是它的主要特征。

该组件允许开发人员控制视图何时呈现以及它的位置。此外，该组件还可以利用诸如`Volt`等模板引擎中可用的视图继承。

必须在服务容器中替换默认组件:

```php
<?php

use Phalcon\Mvc\View\Simple as SimpleView;

$di->set(
    'view',
    function () {
        $view = new SimpleView();

        $view->setViewsDir('../app/views/');

        return $view;
    },
    true
);
```

`Phalcon\Mvc\Application`里的自动渲染必须禁用 (若需要):

```php
<?php

use Exception;
use Phalcon\Mvc\Application;

try {
    $application = new Application($di);

    $application->useImplicitView(false);

    $response = $application->handle();

    $response->send();
} catch (Exception $e) {
    echo $e->getMessage();
}
```

为了渲染一个视图，需要显式调用渲染方法指出你要显示的视图的相对路径：

```php
<?php

use Phalcon\Mvc\Controller;

class PostsController extends Controller
{
    public function indexAction()
    {
        // Render 'views-dir/index.phtml'
        echo $this->view->render('index');

        // Render 'views-dir/posts/show.phtml'
        echo $this->view->render('posts/show');

        // Render 'views-dir/index.phtml' passing variables
        echo $this->view->render(
            'index',
            [
                'posts' => Posts::find(),
            ]
        );

        // Render 'views-dir/posts/show.phtml' passing variables
        echo $this->view->render(
            'posts/show',
            [
                'posts' => Posts::find(),
            ]
        );
    }
}
```

这不同于 `Phalcon\Mvc\View`，它的 `render()`方法使用控制器和操作作为参数：

```php
<?php

$params = [
    'posts' => Posts::find(),
];

// Phalcon\Mvc\View
$view = new \Phalcon\Mvc\View();
echo $view->render('posts', 'show', $params);

// Phalcon\Mvc\View\Simple
$simpleView = new \Phalcon\Mvc\View\Simple();
echo $simpleView->render('posts/show', $params);
```


## 使用局部渲染

局部模板是将呈现过程分解为更易于管理的块的另一种方法，这些块可以被应用程序的不同部分重用。有了局部模板，你可以移动渲染响应特定部分的代码到它自己的文件。

使用局部模板的一种方法是把它们当作子程序的等价物：作为一种将细节从视图中移出的方法，这样你的代码就可以更容易地被理解。例如，你可能有一个这样的视图：

```php
<div class='top'><?php $this->partial('shared/ad_banner'); ?></div>

<div class='content'>
    <h1>Robots</h1>

    <p>Check out our specials for robots:</p>
    ...
</div>

<div class='footer'><?php $this->partial('shared/footer'); ?></div>
```

`partial()` 方法接受作为变量数组的第二个参数，这些变量只会存在于局部模板范围内：

```php
<?php $this->partial('shared/ad_banner', ['id' => $site->id, 'size' => 'big']); ?>
```

## 从控制器给视图传递值

`Phalcon\Mvc\View`在每个控制器中可以通过视图变量 `$this->view`使用。你可以使用这个对象，从一个控制器操作用`setVar()`直接设置变量给视图。

```php
<?php

use Phalcon\Mvc\Controller;

class PostsController extends Controller
{
    public function indexAction()
    {

    }

    public function showAction()
    {
        $user  = Users::findFirst();
        $posts = $user->getPosts();

        // Pass all the username and the posts to the views
        $this->view->setVar('username', $user->username);
        $this->view->setVar('posts', $posts);

        // Using the magic setter
        $this->view->username = $user->username;
        $this->view->posts    = $posts;

        // Passing more than one variable at the same time
        $this->view->setVars(
            [
                'username' => $user->username,
                'posts'    => $posts,
            ]
        );
    }
}
```

带有`setVar()`第一个参数名称的变量将在视图中创建，准备好被使用。该变量可以是任意类型的，从简单的字符串、整数等变量到更复杂的结构，如数组、集合等等。

```php
<h1>
    {{ username }}'s Posts
</h1>

<div class='post'>
<?php

    foreach ($posts as $post) {
        echo '<h2>', $post->title, '</h2>';
    }

?>
</div>
```


## 缓存视图片段

有时当你开发一个动态网站时，它们的某些区域并不经常更新，不同请求的输出是相同的。`Phalcon\Mvc\View`提供了缓存部分或整个渲染输出来提升性能。

`Phalcon\Mvc\View`与`Phalcon\Cache`集成来提供一种缓存输入片段的更容易的方式。你可以手动设置缓存处理器或设置一个全局处理器：

```php
<?php

use Phalcon\Mvc\Controller;

class PostsController extends Controller
{
    public function showAction()
    {
        // Cache the view using the default settings
        $this->view->cache(true);
    }

    public function showArticleAction()
    {
        // Cache this view for 1 hour
        $this->view->cache(
            [
                'lifetime' => 3600,
            ]
        );
    }

    public function resumeAction()
    {
        // Cache this view for 1 day with the key 'resume-cache'
        $this->view->cache(
            [
                'lifetime' => 86400,
                'key'      => 'resume-cache',
            ]
        );
    }

    public function downloadAction()
    {
        // Passing a custom service
        $this->view->cache(
            [
                'service'  => 'myCache',
                'lifetime' => 86400,
                'key'      => 'resume-cache',
            ]
        );
    }
}
```

当我们没有定义缓存的键时，这个组件会自动用控制器的名字和当下正被渲染的视图，以`controller/view`的格式，使用[MD5](http://php.net/manual/en/function.md5.php) 哈希创建一个。为每个操作定义一个键，你因而可以很容易地分辨与每个视图相关的缓存，这是一个好的实践。

当视图组件想要缓存一些东西时，它将从服务容器中请求一个缓存服务。这个服务的服务名称约定是`viewCache`：

```php
<?php

use Phalcon\Cache\Frontend\Output as OutputFrontend;
use Phalcon\Cache\Backend\Memcache as MemcacheBackend;

// Set the views cache service
$di->set(
    'viewCache',
    function () {
        // Cache data for one day by default
        $frontCache = new OutputFrontend(
            [
                'lifetime' => 86400,
            ]
        );

        // Memcached connection settings
        $cache = new MemcacheBackend(
            $frontCache,
            [
                'host' => 'localhost',
                'port' => '11211',
            ]
        );

        return $cache;
    }
);
```

> 前端必须总是[Phalcon\Cache\Frontend\Output](api/Phalcon_Cache_Frontend_Output.md) 服务viewCache一定要在服务容器（DI）中注册为一直开放的（不是共享的）。
>

当我们使用视图时，缓存可以避免控制器对每一次请求都生成视图数据。

为了实现这一点，我们必须用一个键来标识每个缓存。首先，我们验证缓存不存在或已过期，此时需计算/查询在视图中显示的数据：

```php
<?php

use Phalcon\Mvc\Controller;

class DownloadController extends Controller
{
    public function indexAction()
    {
        // Check whether the cache with key 'downloads' exists or has expired
        // 这里好像有问题，应该是缓存不存在时请求数据库
        if ($this->view->getCache()->exists('downloads')) {
            // Query the latest downloads
            $latest = Downloads::find(
                [
                    'order' => 'created_at DESC',
                ]
            );

            $this->view->latest = $latest;
        }

        // Enable the cache with the same key 'downloads'
        $this->view->cache(
            [
                'key' => 'downloads',
            ]
        );
    }
}
```

[PHP替代站](https://github.com/phalcon/php-site)是一个实现片段缓存的例子。


## 模板引擎

模板引擎帮助设计人员在不使用复杂语法的情况下创建视图。Phalcon包括一个强大而快速的模板引擎，叫做`Volt`。`Phalcon\Mvc\View`允许你使用其他的模板引擎，来代替纯PHP或Volt。

使用不同的模板引擎，通常需要使用外部PHP库进行复杂的文本解析，以便为用户生成最终的输出。这通常会增加应用程序使用的资源数量。

如果使用了一个外部模板引擎，`Phalcon\Mvc\View`提供一个完全相同的视图层次结构，而且在这些模板中仍然可以付出多一点点的工作来访问API。

这个组件使用了适配器，这些适配器帮助Phalcon以统一的方式与外部模板引擎进行对话，让我们来看看如何进行集成。


### 创建你自己的模板引擎适配器

There are many template engines, which you might want to integrate or create one of your own. The first step to start using an external template engine is create an adapter for it.

A template engine adapter is a class that acts as bridge between `Phalcon\Mvc\View` and the template engine itself. Usually it only needs two methods implemented: `__construct()` and `render()`. The first one receives the `Phalcon\Mvc\View` instance that creates the engine adapter and the DI container used by the application.

The method `render()` accepts an absolute path to the view file and the view parameters set using `$this->view->setVar()`. You could read or require it when it's necessary.

有许多模板引擎，你可能希望集成或创建自己的模板引擎。开始使用外部模板引擎的第一步是为它创建一个适配器。

模板引擎适配器是一个类，它充当了在`Phalcon\Mvc\View`和模板引擎之间的桥梁。通常它只需要实现两个方法：`__construct()`和`render()`。第一个接受`Phalcon\Mvc\View`实例，通过这个实例创建引擎适配器和应用程序使用的DI容器。

方法`render()`接受视图文件的绝对路径，并使用`$this->view->setVar()`来设置视图参数。你可以在必要的时候读取或要求它。

```php
<?php

use Phalcon\DiInterface;
use Phalcon\Mvc\Engine;

class MyTemplateAdapter extends Engine
{
    /**
     * Adapter constructor
     *
     * @param \Phalcon\Mvc\View $view
     * @param \Phalcon\Di $di
     */
    public function __construct($view, DiInterface $di)
    {
        // Initialize here the adapter
        parent::__construct($view, $di);
    }

    /**
     * Renders a view using the template engine
     *
     * @param string $path
     * @param array $params
     */
    public function render($path, $params)
    {
        // Access view
        $view = $this->_view;

        // Access options
        $options = $this->_options;

        // Render the view
        // ...
    }
}
```


### 更换模板引擎

你可以彻底替换模板引擎，或者同时使用多个模板引擎。方法`Phalcon\Mvc\View::registerEngines()`接受包含定义模板引擎的数据的数组。每个引擎的键是一个有助于区分不同的引擎的扩展名。与特定引擎相关的模板文件必须有这些扩展名。

模板引擎的顺序使用`Phalcon\Mvc\View::registerEngines()`定义，它定义了执行的相关性。如果`Phalcon\Mvc\View`发现了两个使用同样名字但不同扩展名的视图，它将只渲染第一个。

如果你想为应用程序中的每个请求注册一个或一组模板引擎。你可以在创建视图服务时注册它：

```php
<?php

use Phalcon\Mvc\View;

// Setting up the view component
$di->set(
    'view',
    function () {
        $view = new View();

        // A trailing directory separator is required
        $view->setViewsDir('../app/views/');

        // Set the engine
        $view->registerEngines(
            [
                '.my-html' => 'MyTemplateAdapter',
            ]
        );

        // Using more than one template engine
        $view->registerEngines(
            [
                '.my-html' => 'MyTemplateAdapter',
                '.phtml'   => 'Phalcon\Mvc\View\Engine\Php',
            ]
        );

        return $view;
    },
    true
);
```

在 [Phalcon孵化器](https://github.com/phalcon/incubator/tree/master/Library/Phalcon/Mvc/View/Engine)提供了多种模板引擎的适配器。


## 视图中注入服务

每个执行的视图被包含在一个`Phalcon\Di\Injectable`实例里，提供对应用的服务容器的易访问性。


下面的例子展示了如何用框架约定的URL写一个jQuery [ajax request](http://api.jquery.com/jQuery.ajax/)。服务 `url` (通常是`Phalcon\Mvc\Url`) 通过访问一个同名属性被注入视图：

```js
<script type='text/javascript'>

$.ajax({
    url: '<?php echo $this->url->get('cities/get'); ?>'
})
.done(function () {
    alert('Done!');
});

</script>
```


## 独立组件

Phalcon中的所有组件都可以单独使用，因为它们之间是松耦合的：


### 分层渲染

以独立模式使用 `Phalcon\Mvc\View` 示范如下：:

```php
<?php

use Phalcon\Mvc\View;

$view = new View();

// A trailing directory separator is required
$view->setViewsDir('../app/views/');

// Passing variables to the views, these will be created as local variables
$view->setVar('someProducts', $products);
$view->setVar('someFeatureEnabled', true);

// Start the output buffering
$view->start();

// Render all the view hierarchy related to the view products/list.phtml
$view->render('products', 'list');

// Finish the output buffering
$view->finish();

echo $view->getContent();
```

可以使用一种简短的语法：

```php
<?php

use Phalcon\Mvc\View;

$view = new View();

echo $view->getRender(
    'products',
    'list',
    [
        'someProducts'       => $products,
        'someFeatureEnabled' => true,
    ],
    function ($view) {
        // Set any extra options here

        $view->setViewsDir('../app/views/');

        $view->setRenderLevel(
            View::LEVEL_LAYOUT
        );
    }
);
```

### 简单渲染

以独立模式使用 `Phalcon\Mvc\View\Simple`的示范如下：

```php
<?php

use Phalcon\Mvc\View\Simple as SimpleView;

$view = new SimpleView();

// A trailing directory separator is required
$view->setViewsDir('../app/views/');

// Render a view and return its contents as a string
echo $view->render('templates/welcomeMail');

// Render a view passing parameters
echo $view->render(
    'templates/welcomeMail',
    [
        'email'   => $email,
        'content' => $content,
    ]
);
```


## 视图事件

如果提供了`EventManager`，`Phalcon\Mvc\View`和`Phalcon\Mvc\View\Simple`能发送事件给它。事件用类型`view`触发。某些事件当返回布尔值`false`时能停止活动的操作。支持以下事件：

| 事件名称             | 被触发             | 能停止操作吗？ |
| ---------------- | --------------- | :-----: |
| beforeRender     | 开始渲染过程时触发       |   Yes   |
| beforeRenderView | 开始渲染一个已存在的视图前触发 |   Yes   |
| afterRenderView  | 渲染一个已存在的视图后触发   |   No    |
| afterRender      | 完成渲染过程后触发       |   No    |
| notFoundView     | 当找不到一个视图时触发     |   No    |

下面的例子示范了如何给这个组件绑定侦听器：

```php
<?php

use Phalcon\Events\Event;
use Phalcon\Events\Manager as EventsManager;
use Phalcon\Mvc\View;

$di->set(
    'view',
    function () {
        // Create an events manager
        $eventsManager = new EventsManager();

        // Attach a listener for type 'view'
        $eventsManager->attach(
            'view',
            function (Event $event, $view) {
                echo $event->getType(), ' - ', $view->getActiveRenderPath(), PHP_EOL;
            }
        );

        $view = new View();

        $view->setViewsDir('../app/views/');

        // Bind the eventsManager to the view component
        $view->setEventsManager($eventsManager);

        return $view;
    },
    true
);
```

下面的例子展示了如何创建一个插件，这个插件使用 [Tidy](http://www.php.net/manual/en/book.tidy.php)净化/修补渲染过程中生成的HTML：

```php
<?php

use Phalcon\Events\Event;

class TidyPlugin
{
    public function afterRender(Event $event, $view)
    {
        $tidyConfig = [
            'clean'          => true,
            'output-xhtml'   => true,
            'show-body-only' => true,
            'wrap'           => 0,
        ];

        $tidy = tidy_parse_string(
            $view->getContent(),
            $tidyConfig,
            'UTF8'
        );

        $tidy->cleanRepair();

        $view->setContent(
            (string) $tidy
        );
    }
}

// Attach the plugin as a listener
$eventsManager->attach(
    'view:afterRender',
    new TidyPlugin()
);
```