# 使用视图

Views represent the user interface of your application. Views are often HTML files with embedded PHP code that perform tasks related solely to the presentation of the data. Views handle the job of providing data to the web browser or other tool that is used to make requests from your application.

视图表示应用程序的用户界面。视图通常是带有嵌入式PHP代码的HTML文件，它们执行与数据表示相关的任务。视图处理向web浏览器或从你的应用程序请求数据的其他工具提供数据的工作。

`Phalcon\Mvc\View` 和 `Phalcon\Mvc\View\Simple` 负责管理你的MVC应用的视图层。

## 集成视图与控制器

Phalcon automatically passes the execution to the view component as soon as a particular controller has completed its cycle. The view component will look in the views folder for a folder named as the same name of the last controller executed and then for a file named as the last action executed. For instance, if a request is made to the URL *http://127.0.0.1/blog/posts/show/301*, Phalcon will parse the URL as follows:
当一个特定的控制器完成它的周期时，Phalcon会自动将执行传递给视图组件。视图组件将在视图文件夹中查找一个文件夹，该文件夹与执行的最后一个控制器的名称相同，然后是与执行的最后一个操作同名的文件。例如，如果请求的URL是 http://127.0.0.1/blog/posts/show/301，Phalcon将解析URL如下所示：

| 服务器地址    | 127.0.0.1 |
| ----------------- | --------- |
| Phalcon目录 | blog      |
| Controller        | posts     |
| Action            | show      |
| Parameter         | 301       |

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


| 名称 | 文件 | 描述 |
| ----------------- | ----------------------------- | ---------------------------------------- |
| 操作视图 | app/views/posts/show.phtml    | 这是与操作相关的视图，只有在执行show操作时才会显示它。 |
| 控制器布局 | app/views/layouts/posts.phtml | 这是与控制器相关的视图，它只会显示在控制器“posts”内执行的每个操作。在布局中实现的所有代码都将被用于该控制器中的所有操作。 |
| 主布局  | app/views/index.phtml         | 这是一个主要的操作，它将显示在应用程序中执行的每个控制器或操作。 |

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

| 类常量  | 描述 | 顺序 |
| ----------------------- | ---------------------------------------- | :---: |
| `LEVEL_NO_RENDER`       | 指出避免生成任何呈现 |       |
| `LEVEL_ACTION_VIEW`     | 生成与操作相关的视图的呈现 |   1   |
| `LEVEL_BEFORE_TEMPLATE` | 生成优先于控制器布局的模板的呈现 |   2   |
| `LEVEL_LAYOUT`          | 生成控制器布局的呈现 |   3   |
| `LEVEL_AFTER_TEMPLATE`  | 生成控制器布局后模板的呈现 |   4   |
| `LEVEL_MAIN_LAYOUT`     | 生成主布局的呈现，文件`views/index.phtml` |   5   |


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

`Phalcon\Mvc\View\Simple` is an alternative component to `Phalcon\Mvc\View`. It keeps most of the philosophy of `Phalcon\Mvc\View` but lacks of a hierarchy of files which is, in fact, the main feature of its counterpart.

This component allows the developer to have control of when a view is rendered and its location. In addition, this component can leverage of view inheritance available in template engines such as `Volt` and others.

The default component must be replaced in the service container:

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

Automatic rendering must be disabled in `Phalcon\Mvc\Application` (if needed):

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

To render a view it's necessary to call the render method explicitly indicating the relative path to the view you want to display:

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

This is different to `Phalcon\Mvc\View` who's `render()` method uses controllers and actions as parameters:

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

<a name='using-partials'></a>

## 使用局部渲染

Partial templates are another way of breaking the rendering process into simpler more manageable chunks that can be reused by different parts of the application. With a partial, you can move the code for rendering a particular piece of a response to its own file.

One way to use partials is to treat them as the equivalent of subroutines: as a way to move details out of a view so that your code can be more easily understood. For example, you might have a view that looks like this:

```php
<div class='top'><?php $this->partial('shared/ad_banner'); ?></div>

<div class='content'>
    <h1>Robots</h1>

    <p>Check out our specials for robots:</p>
    ...
</div>

<div class='footer'><?php $this->partial('shared/footer'); ?></div>
```

The `partial()` method does accept a second parameter as an array of variables/parameters that only will exists in the scope of the partial:

```php
<?php $this->partial('shared/ad_banner', ['id' => $site->id, 'size' => 'big']); ?>
```

<a name='value-transfer'></a>

## 从控制器给视图传递值

`Phalcon\Mvc\View` is available in each controller using the view variable (`$this->view`). You can use that object to set variables directly to the view from a controller action by using the `setVar()` method.

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

A variable with the name of the first parameter of `setVar()` will be created in the view, ready to be used. The variable can be of any type, from a simple string, integer etc. variable to a more complex structure such as array, collection etc.

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

<a name='caching-fragments'></a>

## 缓存视图片段

Sometimes when you develop dynamic websites and some areas of them are not updated very often, the output is exactly the same between requests. `Phalcon\Mvc\View` offers caching a part or the whole rendered output to increase performance.

`Phalcon\Mvc\View` integrates with `Phalcon\Cache` to provide an easier way to cache output fragments. You could manually set the cache handler or set a global handler:

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

When we do not define a key to the cache, the component automatically creates one using an [MD5](http://php.net/manual/en/function.md5.php) hash of the name of the controller and view currently being rendered in the format of `controller/view`. It is a good practice to define a key for each action so you can easily identify the cache associated with each view.

When the View component needs to cache something it will request a cache service from the services container. The service name convention for this service is `viewCache`:

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

<div class="alert alert-warning">
    <p>
        The frontend must always be <a href="/[[language]]/[[version]]/api/Phalcon_Cache_Frontend_Output">Phalcon\Cache\Frontend\Output</a> and the service <code>viewCache</code> must be registered as always open (not shared) in the services container (DI).
    </p>
</div>

When using views, caching can be used to prevent controllers from needing to generate view data on each request.

To achieve this we must identify uniquely each cache with a key. First we verify that the cache does not exist or has expired to make the calculations/queries to display data in the view:

```php
<?php

use Phalcon\Mvc\Controller;

class DownloadController extends Controller
{
    public function indexAction()
    {
        // Check whether the cache with key 'downloads' exists or has expired
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

The [PHP alternative site](https://github.com/phalcon/php-site) is an example of implementing the caching of fragments.

<a name='template-engines'></a>

## 模板引擎

Template Engines help designers to create views without the use of a complicated syntax. Phalcon includes a powerful and fast templating engine called `Volt`. `Phalcon\Mvc\View` allows you to use other template engines instead of plain PHP or Volt.

Using a different template engine, usually requires complex text parsing using external PHP libraries in order to generate the final output for the user. This usually increases the number of resources that your application will use.

If an external template engine is used, `Phalcon\Mvc\View` provides exactly the same view hierarchy and it's still possible to access the API inside these templates with a little more effort.

This component uses adapters, these help Phalcon to speak with those external template engines in a unified way, let's see how to do that integration.

<a name='custom-template-engine'></a>

### 创建你自己的模板引擎适配器

There are many template engines, which you might want to integrate or create one of your own. The first step to start using an external template engine is create an adapter for it.

A template engine adapter is a class that acts as bridge between `Phalcon\Mvc\View` and the template engine itself. Usually it only needs two methods implemented: `__construct()` and `render()`. The first one receives the `Phalcon\Mvc\View` instance that creates the engine adapter and the DI container used by the application.

The method `render()` accepts an absolute path to the view file and the view parameters set using `$this->view->setVar()`. You could read or require it when it's necessary.

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

You can replace the template engine completely or use more than one template engine at the same time. The method `Phalcon\Mvc\View::registerEngines()` accepts an array containing data that define the template engines. The key of each engine is an extension that aids in distinguishing one from another. Template files related to the particular engine must have those extensions.

The order that the template engines are defined with `Phalcon\Mvc\View::registerEngines()` defines the relevance of execution. If `Phalcon\Mvc\View` finds two views with the same name but different extensions, it will only render the first one.

If you want to register a template engine or a set of them for each request in the application. You could register it when the view service is created:

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

There are adapters available for several template engines on the [Phalcon Incubator](https://github.com/phalcon/incubator/tree/master/Library/Phalcon/Mvc/View/Engine)


## 视图中注入服务

Every view executed is included inside a `Phalcon\Di\Injectable` instance, providing easy access to the application's service container.

The following example shows how to write a jQuery [ajax request](http://api.jquery.com/jQuery.ajax/) using a URL with the framework conventions. The service `url` (usually `Phalcon\Mvc\Url`) is injected in the view by accessing a property with the same name:

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

<a name='stand-along'></a>

## 独立组件

All the components in Phalcon can be used as *glue* components individually because they are loosely coupled to each other:


### 分层渲染

Using `Phalcon\Mvc\View` in a stand-alone mode can be demonstrated below:

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

A short syntax is also available:

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

<a name='stand-alone-simple-rendering'></a>

### 简单渲染

Using `Phalcon\Mvc\View\Simple` in a stand-alone mode can be demonstrated below:

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

`Phalcon\Mvc\View` and `Phalcon\Mvc\View\Simple` are able to send events to an `EventsManager` if it is present. Events are triggered using the type `view`. Some events when returning boolean false could stop the active operation. The following events are supported:

| Event Name       | Triggered                                | Can stop operation? |
| ---------------- | ---------------------------------------- | :-----------------: |
| beforeRender     | Triggered before starting the render process |         Yes         |
| beforeRenderView | Triggered before rendering an existing view |         Yes         |
| afterRenderView  | Triggered after rendering an existing view |         No          |
| afterRender      | Triggered after completing the render process |         No          |
| notFoundView     | Triggered when a view was not found      |         No          |

The following example demonstrates how to attach listeners to this component:

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

The following example shows how to create a plugin that cleans/repair the HTML produced by the render process using [Tidy](http://www.php.net/manual/en/book.tidy.php):

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