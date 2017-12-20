# 教程: Vökuró

Vökuró是另一个你可以用来学习更多Phalcon框架知识的样例应用。Vökuró是一个小网站，这个网站展示了如何实现安全特性和用户及权限的管理。你可以从[Github](https://github.com/phalcon/vokuro) 克隆他的代码。

## 项目结构

一旦你克隆了这个项目，在文档根目录你将看到如下结构：

```bash
vokuro/
    app/
        config/
        controllers/
        forms/
        library/
        models/
        views/
    cache/
    public/
        css/
        img/
    schemas/
```

这个项目遵循了与INVO相似的结构。一旦你在浏览器中打开了这个应用`http://localhost/vokuro`，你就会看到这样的内容：

![](/images/content/tutorial-vokuro-1.png)

这个应用分为两部分，访问者可以通过前端注册服务，在后端，具管理员身份的用户可以管理已注册用户。前端和后端被组合在一个模块中。

## 加载类和依赖

这个项目使用 `Phalcon\Loader` 来加载模型、控制器和表单等。在项目中使用[composer](https://getcomposer.org/)来加载项目的依赖。因此，执行Vökuró前的第一件事就是通过`composer`安装它的依赖。假设你已经正确安装了它，在控制台输入下面的命令：

```bash
cd vokuro
composer install
```

Vökuró 使用Swift发送邮件来确认用户注册， `composer.json` 如下：

```json
{
    "require" : {
        "php": ">=5.5.0",
        "ext-phalcon": ">=3.0.0",
        "swiftmailer/swiftmailer": "^5.4",
        "amazonwebservices/aws-sdk-for-php": "~1.0"
    }
}
```

现在，有一个叫`app/config/loader.php`的文件，里面设置好了所有自动加载的素材。在文件的末尾你可以看到composer的自动加载程序被包含了进来，这使应用可以自动加载所下载依赖中的任何类。

```php
<?php

// ...

// Use composer autoloader to load vendor classes
require_once BASE_PATH . '/vendor/autoload.php';
```

此外，与INVO不同的是，Vökuró为控制器和模型使用了命名空间，构建一个项目时建议这样做。这样，自动加载器看起来与我们之前看到的略有不同(`app/config/loader.php`)：

```php
<?php

use Phalcon\Loader;

$loader = new Loader();

$loader->registerNamespaces(
    [
        'Vokuro\Models'      => $config->application->modelsDir,
        'Vokuro\Controllers' => $config->application->controllersDir,
        'Vokuro\Forms'       => $config->application->formsDir,
        'Vokuro'             => $config->application->libraryDir,
    ]
);

$loader->register();

// ...
```

我们使用`registerNamespaces()`而不是`registerDirectories()`，每个命名空间指向一个在配置文件中定义的目录(app/config/config.php)。例如，命名空间`Vokuro\Controllers`指向`app/controllers`，因此应用程序中所需的所有类都需要它的定义：

```php
<?php

namespace Vokuro\Controllers;

class AboutController extends ControllerBase
{
    // ...
}
```


## 注册

首先，让我们查看在Vökuró里用户是如何注册的。当一个用户点击`Create an Account`按钮，控制器SessionController被调用，行为`signup`被执行：

```php
<?php

namespace Vokuro\Controllers;

use Vokuro\Forms\SignUpForm;

class SessionController extends ControllerBase
{
    public function signupAction()
    {
        $form = new SignUpForm();

        // ...

        $this->view->form = $form;
    }
}
```

这个行为简单地传递一个`SignUpForm`表单实例给视图，这个视图被渲染为允许用户输入登录细节：

```twig
{{ form('class': 'form-search') }}

    <h2>
        Sign Up
    </h2>

    <p>{{ form.label('name') }}</p>
    <p>
        {{ form.render('name') }}
        {{ form.messages('name') }}
    </p>

    <p>{{ form.label('email') }}</p>
    <p>
        {{ form.render('email') }}
        {{ form.messages('email') }}
    </p>

    <p>{{ form.label('password') }}</p>
    <p>
        {{ form.render('password') }}
        {{ form.messages('password') }}
    </p>

    <p>{{ form.label('confirmPassword') }}</p>
    <p>
        {{ form.render('confirmPassword') }}
        {{ form.messages('confirmPassword') }}
    </p>

    <p>
        {{ form.render('terms') }} {{ form.label('terms') }}
        {{ form.messages('terms') }}
    </p>

    <p>{{ form.render('Sign Up') }}</p>

    {{ form.render('csrf', ['value': security.getToken()]) }}
    {{ form.messages('csrf') }}

    <hr>

{{ endForm() }}
```