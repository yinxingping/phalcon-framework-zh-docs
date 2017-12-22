# Volt: 模板引擎

Volt是一种以C语言为PHP编写的超快、设计友好的模板语言。它为你提供了一组帮助你轻松编写视图的助手。Volt与Phalcon的其他的组件是高度集成的，在应用中你可以像使用独立组件一样使用它。

![](https://docs.phalconphp.com/images/content/volt.jpg)

Volt由[Jinja](http://jinja.pocoo.org/)出品，最初由 [Armin Ronacher](https://github.com/mitsuhiko)创建。 因此，许多开发人员将熟练使用与类似的模板引擎相同的语法。 Volt的语法和特征被增强了，包含更多的元素，当然，还有开发人员使用Phalcon工作时已经习惯了的高性能。

## 介绍

Volt视图被编译为纯PHP代码，因此它们从根本上节约了人工写PHP代码的付出：

```twig
{# app/views/products/show.volt #}

{% block last_products %}

{% for product in products %}
    * Name: {{ product.name|e }}
    {% if product.status === 'Active' %}
       Price: {{ product.price + product.taxes/100 }}
    {% endif  %}
{% endfor  %}

{% endblock %}
```


## 激活Volt

与其他模板引擎一样，你可以在视图组件注册Volt，使用一个新的扩展名或重用标准的`.phtml`：

```php
<?php

use Phalcon\Mvc\View;
use Phalcon\Mvc\View\Engine\Volt;

// Register Volt as a service
$di->set(
    'voltService',
    function ($view, $di) {
        $volt = new Volt($view, $di);

        $volt->setOptions(
            [
                'compiledPath'      => '../app/compiled-templates/',
                'compiledExtension' => '.compiled',
            ]
        );

        return $volt;
    }
);

// Register Volt as template engine
$di->set(
    'view',
    function () {
        $view = new View();

        $view->setViewsDir('../app/views/');

        $view->registerEngines(
            [
                '.volt' => 'voltService',
            ]
        );

        return $view;
    }
);
```

使用标准的`.phtml` 扩展名：

```php
<?php

$view->registerEngines(
    [
        '.phtml' => 'voltService',
    ]
);
```

你不必在DI中指定Volt服务；也可以用默认设置使用Volt引擎：

```php
<?php

$view->registerEngines(
    [
        '.volt' => Phalcon\Mvc\View\Engine\Volt::class,
    ]
);
```

如果你不想作为服务重用Volt，你可以传递一个匿名函数而不是服务名称来注册引擎：

```php
<?php

use Phalcon\Mvc\View;
use Phalcon\Mvc\View\Engine\Volt;

// Register Volt as template engine with an anonymous function
$di->set(
    'view',
    function () {
        $view = new View();

        $view->setViewsDir('../app/views/');

        $view->registerEngines(
            [
                '.volt' => function ($view, $di) {
                    $volt = new Volt($view, $di);

                    // Set some options here

                    return $volt;
                }
            ]
        );

        return $view;
    }
);
```

Volt中可以使用下面的选项：

| 选项                  | 描述                                   | 默认值     |
| ------------------- | ------------------------------------ | ------- |
| `compiledPath`      | 一个可写的路径，编译后的PHP模板将保存在这里              | `./`    |
| `compiledExtension` | 添加到已编译PHP文件上的附加扩展名                   | `.php`  |
| `compiledSeparator` | 为了在编译目录创建一个单文件，Volt使用这个分隔符替换/和\目录分隔符 | `%%`    |
| `stat`              | Phalcon是不是必须检查模板文件和它的编译路径内容是否不同      | `true`  |
| `compileAlways`     | 告诉Volt模板是不是每次请求都要编译，或仅当它们改变时编译       | `false` |
| `prefix`            | 允许编译路径里的模板添加一个前缀                     | `null`  |
| `autoescape`        | 启动HTML的全局自动转义                        | `false` |


编译路径根据上面的选项生成，如果开发人员想要完全自由地定义编译路径，可以使用一个匿名函数来生成它，这个函数将接收视图目录中的模板的相对路径。下面的例子展示了如何动态更改编译路径：

```php
<?php

// Just append the .php extension to the template path
// leaving the compiled templates in the same directory
$volt->setOptions(
    [
        'compiledPath' => function ($templatePath) {
            return $templatePath . '.php';
        }
    ]
);

// Recursively create the same structure in another directory
$volt->setOptions(
    [
        'compiledPath' => function ($templatePath) {
            $dirName = dirname($templatePath);

            if (!is_dir('cache/' . $dirName)) {
                mkdir('cache/' . $dirName , 0777 , true);
            }

            return 'cache/' . $dirName . '/'. $templatePath . '.php';
        }
    ]
);
```


## 基本用法

视图由Volt代码、PHP和HTML组成，通过一组特殊的定界符进入Volt模式。`{%……%}`用于执行诸如for循环或赋值等语句，`{{ ... }}`将表达式的结果打印到模板中。

下面是一个最小化的模板，展示了一些基础知识：

```twig
{# app/views/posts/show.phtml #}
<!DOCTYPE html>
<html>
    <head>
        <title>{{ title }} - An example blog</title>
    </head>
    <body>

        {% if show_navigation %}
            <ul id='navigation'>
                {% for item in menu %}
                    <li>
                        <a href='{{ item.href }}'>
                            {{ item.caption }}
                        </a>
                    </li>
                {% endfor %}
            </ul>
        {% endif %}

        <h1>{{ post.title }}</h1>

        <div class='content'>
            {{ post.content }}
        </div>

    </body>
</html>
```

使用 `Phalcon\Mvc\View` 你可以从控制器传递变量到视图。上面的例子中，四个变量被传递给了视图：`show_navigation`, `menu`, `title` and `post`:

```php
<?php

use Phalcon\Mvc\Controller;

class PostsController extends Controller
{
    public function showAction()
    {
        $post = Post::findFirst();
        $menu = Menu::findFirst();

        $this->view->show_navigation = true;
        $this->view->menu            = $menu;
        $this->view->title           = $post->title;
        $this->view->post            = $post;

        // Or...

        $this->view->setVar('show_navigation', true);
        $this->view->setVar('menu',            $menu);
        $this->view->setVar('title',           $post->title);
        $this->view->setVar('post',            $post);
    }
}
```


## 变量

对象变量可能具有可以通过语法访问的属性：`foo.bar`。如果你正在传递数组，你必须使用方括号语法：`foo[bar]`

```twig
{{ post.title }} {# for $post->title #}
{{ post['title'] }} {# for $post['title'] #}
```


## 过滤器

可以使用过滤器格式化或修改变量。管道操作符`|`用于将过滤器应用到变量:

```twig
{{ post.title|e }}
{{ post.content|striptags }}
{{ name|capitalize|trim }}
```

下面是Volt中内置的过滤器列表：

| 过滤器                | 描述                                       |
| ------------------ | ---------------------------------------- |
| `abs`              | 应用PHP函数[abs](http://php.net/manual/zh/function.abs.php)到值 |
| `capitalize`       | 通过应用PHP函数[ucwords](http://php.net/manual/zh/function.ucwords.php)大写一个字符串值 |
| `convert_encoding` | 转换字符串编码                                  |
| `default`          | 在表达式为空（没定义或为false）时设置默认值                 |
| `e`                | 应用`Phalcon\Escaper->escapeHtml()`到值      |
| `escape`           | 应用`Phalcon\Escaper->escapeHtml()`到值      |
| `escape_attr`      | 应用`Phalcon\Escaper->escapeHtmlAttr()`到值  |
| `escape_css`       | 应用`Phalcon\Escaper->escapeCss()`到值       |
| `escape_js`        | 应用`Phalcon\Escaper->escapeJs()`到值        |
| `format`           | 使用[sprintf](http://php.net/manual/zh/function.sprintf.php)格式化一个字符串 |
| `json_encode`      | 将值转换为[JSON](http://php.net/manual/zh/function.json-encode.php)格式 |
| `json_decode`      | 将值从[JSON](http://php.net/manual/zh/function.json-encode.php)格式转换为PHP表示 |
| `join`             | 使用分隔符连接数组部分 [join](http://php.net/manual/zh/function.join.php) |
| `keys`             | 使用PHP函数[array_keys](http://php.net/manual/en/function.array-keys.php)返回数组的键数组 |
| `left_trim`        | 应用[ltrim](http://php.net/manual/zh/function.ltrim.php)PHP函数删除左边额外的空格 |
| `length`           | 计算字符串长度，或数组和对象里包含多少项                     |
| `lower`            | 小写一个字符串                                  |
| `nl2br`            | 使用`<br />`替换`\n`。使用PHP函数[nl2br](http://php.net/manual/zh/function.nl2br.php) |
| `right_trim`       | 应用PHP函数[rtrim](http://php.net/manual/zh/function.rtrim.php)删除右侧额外的空格 |
| `sort`             | 使用PHP函数 [asort](http://php.net/manual/zh/function.asort.php) 给数组排序 |
| `stripslashes`     | 应用PHP函数[stripslashes](http://php.net/manual/zh/function.stripslashes.php)删除转义后的引号 |
| `striptags`        | 应用PHP函数[striptags](http://php.net/manual/zh/function.striptags.php)删除HTML标签 |
| `trim`             |  应用PHP函数[trim](http://php.net/manual/zh/function.trim.php)删除额外的空格 |
| `upper`            | Change the case of a string to uppercase |
| `url_encode`       | Applies the [urlencode](http://php.net/manual/en/function.urlencode.php) PHP function to the value |

例如：

```twig
{# e or escape filter #}
{{ '<h1>Hello<h1>'|e }}
{{ '<h1>Hello<h1>'|escape }}

{# trim filter #}
{{ '   hello   '|trim }}

{# striptags filter #}
{{ '<h1>Hello<h1>'|striptags }}

{# slashes filter #}
{{ ''this is a string''|slashes }}

{# stripslashes filter #}
{{ '\'this is a string\''|stripslashes }}

{# capitalize filter #}
{{ 'hello'|capitalize }}

{# lower filter #}
{{ 'HELLO'|lower }}

{# upper filter #}
{{ 'hello'|upper }}

{# length filter #}
{{ 'robots'|length }}
{{ [1, 2, 3]|length }}

{# nl2br filter #}
{{ 'some\ntext'|nl2br }}

{# sort filter #}
{% set sorted = [3, 1, 2]|sort %}

{# keys filter #}
{% set keys = ['first': 1, 'second': 2, 'third': 3]|keys %}

{# join filter #}
{% set joined = 'a'..'z'|join(',') %}

{# format filter #}
{{ 'My real name is %s'|format(name) }}

{# json_encode filter #}
{% set encoded = robots|json_encode %}

{# json_decode filter #}
{% set decoded = '{'one':1,'two':2,'three':3}'|json_decode %}

{# url_encode filter #}
{{ post.permanent_link|url_encode }}

{# convert_encoding filter #}
{{ 'désolé'|convert_encoding('utf8', 'latin1') }}
```


## 注释

注释也可以用`{# ... #}`分隔符被加到模板中。它们里面的所有文本在最后输出时将被忽略掉：

```twig
{# note: this is a comment
    {% set price = 100; %}
#}
```


## 控制结构列表

Volt提供了一组强大的控制结构可以在模板里使用：


### For

按顺序对每一项进行循环。下面的例子展示了如何遍历一组‘robots'并打印出他/她的名字:

```twig
<h1>Robots</h1>
<ul>
    {% for robot in robots %}
        <li>
            {{ robot.name|e }}
        </li>
    {% endfor %}
</ul>
```

for循环也可以嵌套：

```twig
<h1>Robots</h1>
{% for robot in robots %}
    {% for part in robot.parts %}
        Robot: {{ robot.name|e }} Part: {{ part.name|e }} <br />
    {% endfor %}
{% endfor %}
```

你可以使用以下语法同PHP对应部分一样获得元素的`keys`:


```twig
{% set numbers = ['one': 1, 'two': 2, 'three': 3] %}

{% for name, value in numbers %}
    Name: {{ name }} Value: {{ value }}
{% endfor %}
```

可选择设置一个`if`判断：

```twig
{% set numbers = ['one': 1, 'two': 2, 'three': 3] %}

{% for value in numbers if value < 2 %}
    Value: {{ value }}
{% endfor %}

{% for name, value in numbers if name !== 'two' %}
    Name: {{ name }} Value: {{ value }}
{% endfor %}
```

如果在`for`中定义了`else`，那么迭代器中的表达式将被执行为0迭代:

```twig
<h1>Robots</h1>
{% for robot in robots %}
    Robot: {{ robot.name|e }} Part: {{ part.name|e }} <br />
{% else %}
    There are no robots to show
{% endfor %}
```

替代语法:

```twig
<h1>Robots</h1>
{% for robot in robots %}
    Robot: {{ robot.name|e }} Part: {{ part.name|e }} <br />
{% elsefor %}
    There are no robots to show
{% endfor %}
```


### 循环控制

`break` and `continue` 语句可以用于退出循环，或者在当前块中强制执行迭代:

```twig
{# skip the even robots #}
{% for index, robot in robots %}
    {% if index is even %}
        {% continue %}
    {% endif %}
    ...
{% endfor %}
```

```twig
{# exit the foreach on the first even robot #}
{% for index, robot in robots %}
    {% if index is even %}
        {% break %}
    {% endif %}
    ...
{% endfor %}
```


### If

与PHP一样，`if`语句检查一个表达式的值是真或假：

```twig
<h1>Cyborg Robots</h1>
<ul>
    {% for robot in robots %}
        {% if robot.type === 'cyborg' %}
            <li>{{ robot.name|e }}</li>
        {% endif %}
    {% endfor %}
</ul>
```

else字句也得到了支持：

```twig
<h1>Robots</h1>
<ul>
    {% for robot in robots %}
        {% if robot.type === 'cyborg' %}
            <li>{{ robot.name|e }}</li>
        {% else %}
            <li>{{ robot.name|e }} (not a cyborg)</li>
        {% endif %}
    {% endfor %}
</ul>
```

`elseif`控制流结构可以和if来模拟一个`switch`块:

```twig
{% if robot.type === 'cyborg' %}
    Robot is a cyborg
{% elseif robot.type === 'virtual' %}
    Robot is virtual
{% elseif robot.type === 'mechanical' %}
    Robot is mechanical
{% endif %}
```


### 循环上下文

在`for`循环中可以用一个特殊的变量来为你提供关于以下的信息

| 变量         | 描述                             |
| ---------------- | ---------------------------------------- |
| `loop.index`     | 以1为基准的循环的当前位置 |
| `loop.index0`    | 以0为基准的循环的当前位置 |
| `loop.revindex`  | 以1为基准的循环的最后位置 |
| `loop.revindex0` | 以0为基准的循环的最后位置 |
| `loop.first`     | 循环的第一个位置时为True         |
| `loop.last`      | 循环的最后一个位置时True        |
| `loop.length`    | 迭代项的数量  |

举例：

```twig
{% for robot in robots %}
    {% if loop.first %}
        <table>
            <tr>
                <th>#</th>
                <th>Id</th>
                <th>Name</th>
            </tr>
    {% endif %}
            <tr>
                <td>{{ loop.index }}</td>
                <td>{{ robot.id }}</td>
                <td>{{ robot.name }}</td>
            </tr>
    {% if loop.last %}
        </table>
    {% endif %}
{% endfor %}
```


## 赋值

变量可以在模板里使用指令`set`改变：

```twig
{% set fruits = ['Apple', 'Banana', 'Orange'] %}

{% set name = robot.name %}
```

在同一条指令中允许多个赋值：

```twig
{% set fruits = ['Apple', 'Banana', 'Orange'], name = robot.name, active = true %}
```

此外，你可以使用组合赋值运算符：

```twig
{% set price += 100.00 %}

{% set age *= 5 %}
```

可以使用如下运算符：

| 运算符 | 描述               |
| -------- | ------------------------- |
| `=`      | 标准赋值       |
| `+=`     | 加法赋值       |
| `-=`     | 减法赋值    |
| `\*=`    | 乘法赋值 |
| `/=`     | 除法赋值       |


## 表达式

Volt提供了一组基本的表达式支持，包括字面值和普通运算符。一个表达式可以使用`{{`和`}}`分隔符进行计算和输出:

```twig
{{ (1 + 1) * 2 }}
```

如果一个表达式需要计算但不需输出，可以使用`do`语句：

```twig
{% do (1 + 1) * 2 %}
```

### 字面值

支持下面的字面值：

| 字面值               | 描述                             |
| -------------------- | ---------------------------------------- |
| `'this is a string'` | 双引号和单引号之间的文本被作为字符串处理 |
| `100.25`             | 带小数点的数字被当作双精度/浮点数处理 |
| `100`                | 不带小数点的数字被当作整数处理 |
| `false`              | 常量'false'是布尔false |
| `true`               | 常量'true'是布尔值true |
| `null`               | 常量'null'是NULL       |


### 数组

无论你是用PHP 5.3还是 >=5.4，都可以通过将一列值放到中括号来创建数组：

```twig
{# Simple array #}
{{ ['Apple', 'Banana', 'Orange'] }}

{# Other simple array #}
{{ ['Apple', 1, 2.5, false, null] }}

{# Multi-Dimensional array #}
{{ [[1, 2], [3, 4], [5, 6]] }}

{# Hash-style array #}
{{ ['first': 1, 'second': 4/2, 'third': '3'] }}
```

花括号也能用来定义数组或哈希：

```twig
{% set myArray = {'Apple', 'Banana', 'Orange'} %}
{% set myHash  = {'first': 1, 'second': 4/2, 'third': '3'} %}
```


### 数学运算

你可以使用下面的运算符在模板中执行计算：

| 运算符 | 描述                             |
| :------: | ---------------------------------------- |
|   `+`    | 执行加法运算，`{{ 2 + 3 }}` 返回 5 |
|   `-`    | 执行减法运算， `{{ 2 - 3 }}` 返回 -1 |
|   `*`    | 执行乘法运算， `{{ 2 * 3 }}` 返回 6 |
|   `/`    | 执行除法运算， `{{ 10 / 2 }}` 返回 5 |
|   `%`    | 计算除法运算的余数， `{{ 10 % 3 }}` 返回 1 |


### 比较运算

下面是可用的比较运算符：

| Operator | Description                              |
| :------: | ---------------------------------------- |
|   `==`   | Check whether both operands are equal    |
|   `!=`   | Check whether both operands aren't equal |
|   `<>`   | Check whether both operands aren't equal |
|   `>`    | Check whether left operand is greater than right operand |
|   `<`    | Check whether left operand is less than right operand |
|   `<=`   | Check whether left operand is less or equal than right operand |
|   `>=`   | Check whether left operand is greater or equal than right operand |
|  `===`   | Check whether both operands are identical |
|  `!==`   | Check whether both operands aren't identical |


### 逻辑运算

Logic operators are useful in the `if` expression evaluation to combine multiple tests:

|  Operator  | Description                              |
| :--------: | ---------------------------------------- |
|    `or`    | Return true if the left or right operand is evaluated as true |
|   `and`    | Return true if both left and right operands are evaluated as true |
|   `not`    | Negates an expression                    |
| `( expr )` | Parenthesis groups expressions           |


### 其它运算符

Additional operators seen the following operators are available:

| Operator          | Description                              |
| ----------------- | ---------------------------------------- |
| `~`               | Concatenates both operands `{{ 'hello ' ~ 'world' }}` |
| `|`               | Applies a filter in the right operand to the left `{{ 'hello'|uppercase }}` |
| `..`              | Creates a range `{{ 'a'..'z' }}` `{{ 1..10 }}` |
| `is`              | Same as == (equals), also performs tests |
| `in`              | To check if an expression is contained into other expressions `if 'a' in 'abc'` |
| `is not`          | Same as != (not equals)                  |
| `'a' ? 'b' : 'c'` | Ternary operator. The same as the PHP ternary operator |
| `++`              | Increments a value                       |
| `--`              | Decrements a value                       |

The following example shows how to use operators:

```twig
{% set robots = ['Voltron', 'Astro Boy', 'Terminator', 'C3PO'] %}

{% for index in 0..robots|length %}
    {% if robots[index] is defined %}
        {{ 'Name: ' ~ robots[index] }}
    {% endif %}
{% endfor %}
```

<a name='tests'></a>

## 测试

Tests can be used to test if a variable has a valid expected value. The operator `is` is used to perform the tests:

```twig
{% set robots = ['1': 'Voltron', '2': 'Astro Boy', '3': 'Terminator', '4': 'C3PO'] %}

{% for position, name in robots %}
    {% if position is odd %}
        {{ name }}
    {% endif %}
{% endfor %}
```

The following built-in tests are available in Volt:

| Test          | Description                              |
| ------------- | ---------------------------------------- |
| `defined`     | Checks if a variable is defined (`isset()`) |
| `divisibleby` | Checks if a value is divisible by other value |
| `empty`       | Checks if a variable is empty            |
| `even`        | Checks if a numeric value is even        |
| `iterable`    | Checks if a value is iterable. Can be traversed by a 'for' statement |
| `numeric`     | Checks if value is numeric               |
| `odd`         | Checks if a numeric value is odd         |
| `sameas`      | Checks if a value is identical to other value |
| `scalar`      | Checks if value is scalar (not an array or object) |
| `type`        | Checks if a value is of the specified type |

More examples:

```twig
{% if robot is defined %}
    The robot variable is defined
{% endif %}

{% if robot is empty %}
    The robot is null or isn't defined
{% endif %}

{% for key, name in [1: 'Voltron', 2: 'Astroy Boy', 3: 'Bender'] %}
    {% if key is even %}
        {{ name }}
    {% endif %}
{% endfor %}

{% for key, name in [1: 'Voltron', 2: 'Astroy Boy', 3: 'Bender'] %}
    {% if key is odd %}
        {{ name }}
    {% endif %}
{% endfor %}

{% for key, name in [1: 'Voltron', 2: 'Astroy Boy', 'third': 'Bender'] %}
    {% if key is numeric %}
        {{ name }}
    {% endif %}
{% endfor %}

{% set robots = [1: 'Voltron', 2: 'Astroy Boy'] %}
{% if robots is iterable %}
    {% for robot in robots %}
        ...
    {% endfor %}
{% endif %}

{% set world = 'hello' %}
{% if world is sameas('hello') %}
    {{ 'it's hello' }}
{% endif %}

{% set external = false %}
{% if external is type('boolean') %}
    {{ 'external is false or true' }}
{% endif %}
```

<a name='macros'></a>

## 宏指令

Macros can be used to reuse logic in a template, they act as PHP functions, can receive parameters and return values:

```twig
{# Macro 'display a list of links to related topics' #}
{%- macro related_bar(related_links) %}
    <ul>
        {%- for link in related_links %}
            <li>
                <a href='{{ url(link.url) }}' title='{{ link.title|striptags }}'>
                    {{ link.text }}
                </a>
            </li>
        {%- endfor %}
    </ul>
{%- endmacro %}

{# Print related links #}
{{ related_bar(links) }}

<div>This is the content</div>

{# Print related links again #}
{{ related_bar(links) }}
```

When calling macros, parameters can be passed by name:

```twig
{%- macro error_messages(message, field, type) %}
    <div>
        <span class='error-type'>{{ type }}</span>
        <span class='error-field'>{{ field }}</span>
        <span class='error-message'>{{ message }}</span>
    </div>
{%- endmacro %}

{# Call the macro #}
{{ error_messages('type': 'Invalid', 'message': 'The name is invalid', 'field': 'name') }}
```

Macros can return values:

```twig
{%- macro my_input(name, class) %}
    {% return text_field(name, 'class': class) %}
{%- endmacro %}

{# Call the macro #}
{{ '<p>' ~ my_input('name', 'input-text') ~ '</p>' }}
```

And receive optional parameters:

```twig
{%- macro my_input(name, class='input-text') %}
    {% return text_field(name, 'class': class) %}
{%- endmacro %}

{# Call the macro #}
{{ '<p>' ~ my_input('name') ~ '</p>' }}
{{ '<p>' ~ my_input('name', 'input-text') ~ '</p>' }}
```

<a name='tag-helpers'></a>

## 使用标签助手

Volt is highly integrated with `Phalcon\Tag`, so it's easy to use the helpers provided by that component in a Volt template:

```twig
{{ javascript_include('js/jquery.js') }}

{{ form('products/save', 'method': 'post') }}

    <label for='name'>Name</label>
    {{ text_field('name', 'size': 32) }}

    <label for='type'>Type</label>
    {{ select('type', productTypes, 'using': ['id', 'name']) }}

    {{ submit_button('Send') }}

{{ end_form() }}
```

The following PHP is generated:

```php
<?php echo Phalcon\Tag::javascriptInclude('js/jquery.js') ?>

<?php echo Phalcon\Tag::form(array('products/save', 'method' => 'post')); ?>

    <label for='name'>Name</label>
    <?php echo Phalcon\Tag::textField(array('name', 'size' => 32)); ?>

    <label for='type'>Type</label>
    <?php echo Phalcon\Tag::select(array('type', $productTypes, 'using' => array('id', 'name'))); ?>

    <?php echo Phalcon\Tag::submitButton('Send'); ?>

{{ end_form() }}
```

To call a `Phalcon\Tag` helper, you only need to call an uncamelized version of the method:

| Method                           | Volt function        |
| -------------------------------- | -------------------- |
| `Phalcon\Tag::checkField`        | `check_field`        |
| `Phalcon\Tag::dateField`         | `date_field`         |
| `Phalcon\Tag::emailField`        | `email_field`        |
| `Phalcon\Tag::endForm`           | `end_form`           |
| `Phalcon\Tag::fileField`         | `file_field`         |
| `Phalcon\Tag::form`              | `form`               |
| `Phalcon\Tag::friendlyTitle`     | `friendly_title`     |
| `Phalcon\Tag::getTitle`          | `get_title`          |
| `Phalcon\Tag::hiddenField`       | `hidden_field`       |
| `Phalcon\Tag::image`             | `image`              |
| `Phalcon\Tag::javascriptInclude` | `javascript_include` |
| `Phalcon\Tag::linkTo`            | `link_to`            |
| `Phalcon\Tag::numericField`      | `numeric_field`      |
| `Phalcon\Tag::passwordField`     | `password_field`     |
| `Phalcon\Tag::radioField`        | `radio_field`        |
| `Phalcon\Tag::select`            | `select`             |
| `Phalcon\Tag::selectStatic`      | `select_static`      |
| `Phalcon\Tag::stylesheetLink`    | `stylesheet_link`    |
| `Phalcon\Tag::submitButton`      | `submit_button`      |
| `Phalcon\Tag::textArea`          | `text_area`          |
| `Phalcon\Tag::textField`         | `text_field`         |


## 函数

The following built-in functions are available in Volt:

| Name          | Description                              |
| ------------- | ---------------------------------------- |
| `content`     | Includes the content produced in a previous rendering stage |
| `get_content` | Same as `content`                        |
| `partial`     | Dynamically loads a partial view in the current template |
| `super`       | Render the contents of the parent block  |
| `time`        | Calls the PHP function with the same name |
| `date`        | Calls the PHP function with the same name |
| `dump`        | Calls the PHP function `var_dump()`      |
| `version`     | Returns the current version of the framework |
| `constant`    | Reads a PHP constant                     |
| `url`         | Generate a URL using the 'url' service   |

<a name='view-integrations'></a>

## 视图集成

Also, Volt is integrated with `Phalcon\Mvc\View`, you can play with the view hierarchy and include partials as well:

```twig
{{ content() }}

<!-- Simple include of a partial -->
<div id='footer'>{{ partial('partials/footer') }}</div>

<!-- Passing extra variables -->
<div id='footer'>{{ partial('partials/footer', ['links': links]) }}</div>
```

A partial is included in runtime, Volt also provides `include`, this compiles the content of a view and returns its contents as part of the view which was included:

```twig
{# Simple include of a partial #}
<div id='footer'>
    {% include 'partials/footer' %}
</div>

{# Passing extra variables #}
<div id='footer'>
    {% include 'partials/footer' with ['links': links] %}
</div>
```

<a name='view-integration-include'></a>

### Include

`include` has a special behavior that will help us improve performance a bit when using Volt, if you specify the extension when including the file and it exists when the template is compiled, Volt can inline the contents of the template in the parent template where it's included. Templates aren't inlined if the `include` have variables passed with `with`:

```twig
{# The contents of 'partials/footer.volt' is compiled and inlined #}
<div id='footer'>
    {% include 'partials/footer.volt' %}
</div>
```

<a name='view-integration-partial-vs-include'></a>

### Partial vs Include

Keep the following points in mind when choosing to use the `partial` function or `include`:

| Type       | Description                              |
| ---------- | ---------------------------------------- |
| `partial`  | allows you to include templates made in Volt and in other template engines as well |
|            | allows you to pass an expression like a variable allowing to include the content of other view dynamically |
|            | is better if the content that you have to include changes frequently |
| `includes` | copies the compiled content into the view which improves the performance |
|            | only allows to include templates made with Volt |
|            | requires an existing template at compile time |

<a name='template-inheritance'></a>

## 模板继承

With template inheritance you can create base templates that can be extended by others templates allowing to reuse code. A base template define *blocks* than can be overridden by a child template. Let's pretend that we have the following base template:

```twig
{# templates/base.volt #}
<!DOCTYPE html>
<html>
    <head>
        {% block head %}
            <link rel='stylesheet' href='style.css' />
        {% endblock %}

        <title>{% block title %}{% endblock %} - My Webpage</title>
    </head>

    <body>
        <div id='content'>{% block content %}{% endblock %}</div>

        <div id='footer'>
            {% block footer %}&copy; Copyright 2015, All rights reserved.{% endblock %}
        </div>
    </body>
</html>
```

From other template we could extend the base template replacing the blocks:

```twig
{% extends 'templates/base.volt' %}

{% block title %}Index{% endblock %}

{% block head %}<style type='text/css'>.important { color: #336699; }</style>{% endblock %}

{% block content %}
    <h1>Index</h1>
    <p class='important'>Welcome on my awesome homepage.</p>
{% endblock %}
```

Not all blocks must be replaced at a child template, only those that are needed. The final output produced will be the following:

```html
<!DOCTYPE html>
<html>
    <head>
        <style type='text/css'>.important { color: #336699; }</style>

        <title>Index - My Webpage</title>
    </head>

    <body>
        <div id='content'>
            <h1>Index</h1>
            <p class='important'>Welcome on my awesome homepage.</p>
        </div>

        <div id='footer'>
            &copy; Copyright 2015, All rights reserved.
        </div>
    </body>
</html>
```

<a name='template-inheritance-multiple'></a>

### 多重继承

Extended templates can extend other templates. The following example illustrates this:

```twig
{# main.volt #}
<!DOCTYPE html>
<html>
    <head>
        <title>Title</title>
    </head>

    <body>
        {% block content %}{% endblock %}
    </body>
</html>
```

Template `layout.volt` extends `main.volt`

```twig
{# layout.volt #}
{% extends 'main.volt' %}

{% block content %}

    <h1>Table of contents</h1>

{% endblock %}
```

Finally a view that extends `layout.volt`:

```twig
{# index.volt #}
{% extends 'layout.volt' %}

{% block content %}

    {{ super() }}

    <ul>
        <li>Some option</li>
        <li>Some other option</li>
    </ul>

{% endblock %}
```

Rendering `index.volt` produces:

```html
<!DOCTYPE html>
<html>
    <head>
        <title>Title</title>
    </head>

    <body>

        <h1>Table of contents</h1>

        <ul>
            <li>Some option</li>
            <li>Some other option</li>
        </ul>

    </body>
</html>
```

Note the call to the function `super()`. With that function it's possible to render the contents of the parent block. As partials, the path set to `extends` is a relative path under the current views directory (i.e. `app/views/`).

<div class="alert alert-warning">
    <p>
        By default, and for performance reasons, Volt only checks for changes in the children templates to know when to re-compile to plain PHP again, so it is recommended initialize Volt with the option <code>'compileAlways' => true</code>. Thus, the templates are compiled always taking into account changes in the parent templates.
    </p>
</div>


## 自动转义模式

You can enable auto-escaping of all variables printed in a block using the autoescape mode:

```twig
Manually escaped: {{ robot.name|e }}

{% autoescape true %}
    Autoescaped: {{ robot.name }}
    {% autoescape false %}
        No Autoescaped: {{ robot.name }}
    {% endautoescape %}
{% endautoescape %}
```


## 扩展Volt

Unlike other template engines, Volt itself is not required to run the compiled templates. Once the templates are compiled there is no dependence on Volt. With performance independence in mind, Volt only acts as a compiler for PHP templates.

The Volt compiler allow you to extend it adding more functions, tests or filters to the existing ones.


### 函数

Functions act as normal PHP functions, a valid string name is required as function name. Functions can be added using two strategies, returning a simple string or using an anonymous function. Always is required that the chosen strategy returns a valid PHP string expression:

```php
<?php

use Phalcon\Mvc\View\Engine\Volt;

$volt = new Volt($view, $di);

$compiler = $volt->getCompiler();

// This binds the function name 'shuffle' in Volt to the PHP function 'str_shuffle'
$compiler->addFunction('shuffle', 'str_shuffle');
```

Register the function with an anonymous function. This case we use `$resolvedArgs` to pass the arguments exactly as were passed in the arguments:

```php
<?php

$compiler->addFunction(
    'widget',
    function ($resolvedArgs, $exprArgs) {
        return 'MyLibrary\Widgets::get(' . $resolvedArgs . ')';
    }
);
```

Treat the arguments independently and unresolved:

```php
<?php

$compiler->addFunction(
    'repeat',
    function ($resolvedArgs, $exprArgs) use ($compiler) {
        // Resolve the first argument
        $firstArgument = $compiler->expression($exprArgs[0]['expr']);

        // Checks if the second argument was passed
        if (isset($exprArgs[1])) {
            $secondArgument = $compiler->expression($exprArgs[1]['expr']);
        } else {
            // Use '10' as default
            $secondArgument = '10';
        }

        return 'str_repeat(' . $firstArgument . ', ' . $secondArgument . ')';
    }
);
```

Generate the code based on some function availability:

```php
<?php

$compiler->addFunction(
    'contains_text',
    function ($resolvedArgs, $exprArgs) {
        if (function_exists('mb_stripos')) {
            return 'mb_stripos(' . $resolvedArgs . ')';
        } else {
            return 'stripos(' . $resolvedArgs . ')';
        }
    }
);
```

Built-in functions can be overridden adding a function with its name:

```php
<?php

// Replace built-in function dump
$compiler->addFunction('dump', 'print_r');
```

<a name='extending-filters'></a>

### 过滤器

A filter has the following form in a template: leftExpr|name(optional-args). Adding new filters is similar as seen with the functions:

```php
<?php

// This creates a filter 'hash' that uses the PHP function 'md5'
$compiler->addFilter('hash', 'md5');
```

```php
<?php

$compiler->addFilter(
    'int',
    function ($resolvedArgs, $exprArgs) {
        return 'intval(' . $resolvedArgs . ')';
    }
);
```

Built-in filters can be overridden adding a function with its name:

```php
<?php

// Replace built-in filter 'capitalize'
$compiler->addFilter('capitalize', 'lcfirst');
```

<a name='extending-extensions'></a>

### 扩展

With extensions the developer has more flexibility to extend the template engine, and override the compilation of a specific instruction, change the behavior of an expression or operator, add functions/filters, and more.

An extension is a class that implements the events triggered by Volt as a method of itself. For example, the class below allows to use any PHP function in Volt:

```php
<?php

class PhpFunctionExtension
{
    /**
     * This method is called on any attempt to compile a function call
     */
    public function compileFunction($name, $arguments)
    {
        if (function_exists($name)) {
            return $name . '('. $arguments . ')';
        }
    }
}
```

The above class implements the method `compileFunction` which is invoked before any attempt to compile a function call in any template. The purpose of the extension is to verify if a function to be compiled is a PHP function allowing to call it from the template. Events in extensions must return valid PHP code, this will be used as result of the compilation instead of the one generated by Volt. If an event doesn't return an string the compilation is done using the default behavior provided by the engine.

The following compilation events are available to be implemented in extensions:

| Event/Method        | Description                              |
| ------------------- | ---------------------------------------- |
| `compileFunction`   | Triggered before trying to compile any function call in a template |
| `compileFilter`     | Triggered before trying to compile any filter call in a template |
| `resolveExpression` | Triggered before trying to compile any expression. This allows the developer to override operators |
| `compileStatement`  | Triggered before trying to compile any expression. This allows the developer to override any statement |

Volt extensions must be in registered in the compiler making them available in compile time:

```php
<?php

// Register the extension in the compiler
$compiler->addExtension(
    new PhpFunctionExtension()
);
```


## 缓存视图片段

With Volt it's easy cache view fragments. This caching improves performance preventing that the contents of a block from being executed by PHP each time the view is displayed:

```twig
{% cache 'sidebar' %}
    <!-- generate this content is slow so we are going to cache it -->
{% endcache %}
```

Setting a specific number of seconds:

```twig
{# cache the sidebar by 1 hour #}
{% cache 'sidebar' 3600 %}
    <!-- generate this content is slow so we are going to cache it -->
{% endcache %}
```

Any valid expression can be used as cache key:

```twig
{% cache ('article-' ~ post.id) 3600 %}

    <h1>{{ post.title }}</h1>

    <p>{{ post.content }}</p>

{% endcache %}
```

The caching is done by the `Phalcon\Cache` component via the view component. Learn more about how this integration works in the section [Caching View Fragments](/[[language]]/[[version]]/views#caching-fragments).

<a name='services-in-templates'></a>

## 将服务注入到模板

If a service container (DI) is available for Volt, you can use the services by only accessing the name of the service in the template:

```twig
{# Inject the 'flash' service #}
<div id='messages'>{{ flash.output() }}</div>

{# Inject the 'security' service #}
<input type='hidden' name='token' value='{{ security.getToken() }}'>
```


## 独立组件

Using Volt in a stand-alone mode can be demonstrated below:

```php
<?php

use Phalcon\Mvc\View\Engine\Volt\Compiler as VoltCompiler;

// Create a compiler
$compiler = new VoltCompiler();

// Optionally add some options
$compiler->setOptions(
    [
        // ...
    ]
);

// Compile a template string returning PHP code
echo $compiler->compileString(
    "{{ 'hello' }}"
);

// Compile a template in a file specifying the destination file
$compiler->compileFile(
    'layouts/main.volt',
    'cache/layouts/main.volt.php'
);

// Compile a template in a file based on the options passed to the compiler
$compiler->compile(
    'layouts/main.volt'
);

// Require the compiled templated (optional)
require $compiler->getCompiledTemplatePath();
```

## 外部资源

* A bundle for Sublime/Textmate is available [here](https://github.com/phalcon/volt-sublime-textmate)
* [Album-O-Rama](https://album-o-rama.phalconphp.com) is a sample application using Volt as template engine, [Github](https://github.com/phalcon/album-o-rama)
* [Our website](https://phalconphp.com) is running using Volt as template engine, [Github](https://github.com/phalcon/website)
* [Phosphorum](https://forum.phalconphp.com), the Phalcon's forum, also uses Volt, [Github](https://github.com/phalcon/forum)
* [Vökuró](https://vokuro.phalconphp.com), is another sample application that use Volt, [Github](https://github.com/phalcon/vokuro)