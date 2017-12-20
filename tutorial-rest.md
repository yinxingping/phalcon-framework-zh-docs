# 教程: 创建一个简单的REST API

在这个教程里，我们将解释如何创建一个简单的应用，这个应用提供了一个使用不同HTTP方法的[RESTful](http://en.wikipedia.org/wiki/Representational_state_transfer) API。

* `GET` 获取和搜索数据
* `POST` 增加数据
* `PUT` 更新数据
* `DELETE` 删除数据

## 定义API

这个API由下面的方法组成：

| 方法       | URL                      | 动作                   |
| -------- | ------------------------ | -------------------- |
| `GET`    | /api/robots              | 获取所有Robots           |
| `GET`    | /api/robots/search/Astro | 用"Astro"在名字中搜索robots |
| `GET`    | /api/robots/2            | 基于主键获取robots         |
| `POST`   | /api/robots              | 添加一个新robot           |
| `PUT`    | /api/robots/2            | 基于主键更新robots         |
| `DELETE` | /api/robots/2            | 基于主键删除robots         |


## 创建应用

因为这个应用是那么简单，因为我们开发时将不实现任何完全MVC环境。在这本例中，我们将使用一个[微应用](application-micro.md)来满足我们的目的。

下面的文件结构完全够用：

```php
my-rest-api/
    models/
        Robots.php
    index.php
    .htaccess
```

首先，我们需要一个`.htaccess`文件，里面包含了所有重写请求URI到`index.php`（应用入口点）的规则：

```apacheconfig
<IfModule mod_rewrite.c>
    RewriteEngine On
    RewriteCond %{REQUEST_FILENAME} !-f
    RewriteRule ^((?s).*)$ index.php?_url=/$1 [QSA,L]
</IfModule>
```

我们的代码块将被放到`index.php`中，文件创建如下：

```php
<?php

use Phalcon\Mvc\Micro;

$app = new Micro();

// Define the routes here

$app->handle();
```

现在我们将创建如我们上面定义的路由：

```php
<?php

use Phalcon\Mvc\Micro;

$app = new Micro();

// Retrieves all robots
$app->get(
    '/api/robots',
    function () {
        // Operation to fetch all the robots
    }
);

// Searches for robots with $name in their name
$app->get(
    '/api/robots/search/{name}',
    function ($name) {
        // Operation to fetch robot with name $name
    }
);

// Retrieves robots based on primary key
$app->get(
    '/api/robots/{id:[0-9]+}',
    function ($id) {
        // Operation to fetch robot with id $id
    }
);

// Adds a new robot
$app->post(
    '/api/robots',
    function () {
        // Operation to create a fresh robot
    }
);

// Updates robots based on primary key
$app->put(
    '/api/robots/{id:[0-9]+}',
    function ($id) {
        // Operation to update a robot with id $id
    }
);

// Deletes robots based on primary key
$app->delete(
    '/api/robots/{id:[0-9]+}',
    function ($id) {
        // Operation to delete the robot with id $id
    }
);

$app->handle();
```

每个路由都使用与HTTP方法相同的方法定义，我们传递一个路由模式作为第一个参数，然后是一个处理程序。在本例中，处理程序是一个匿名函数。下面的路由：`/api/robots/id:[0-9]+`，作为范例，显式地设置`id`参数必须有一个数字格式。

当一个已定义的路由匹配到了请求URL，然后应用执行对应的处理程序。

## 创建模型

我们的API提供了关于`robots`的信息，这些数据存储在一个数据库中。下面的模型允许我们以面向对象的方式访问该表。我们已经使用内置的验证器和简单的验证实现了一些业务规则。这样做可以让我们安心，节省了数据，满足了我们应用程序的需求。这个模型文件应该放在你的的`Models`文件夹中。

```php
<?php

namespace Store\Toys;

use Phalcon\Mvc\Model;
use Phalcon\Mvc\Model\Message;
use Phalcon\Mvc\Model\Validator\Uniqueness;
use Phalcon\Mvc\Model\Validator\InclusionIn;

class Robots extends Model
{
    public function validation()
    {
        // Type must be: droid, mechanical or virtual
        $this->validate(
            new InclusionIn(
                [
                    'field'  => 'type',
                    'domain' => [
                        'droid',
                        'mechanical',
                        'virtual',
                    ],
                ]
            )
        );

        // Robot name must be unique
        $this->validate(
            new Uniqueness(
                [
                    'field'   => 'name',
                    'message' => 'The robot name must be unique',
                ]
            )
        );

        // Year cannot be less than zero
        if ($this->year < 0) {
            $this->appendMessage(
                new Message('The year cannot be less than zero')
            );
        }

        // Check if any messages have been produced
        if ($this->validationHasFailed() === true) {
            return false;
        }
    }
}
```

现在，我们必须设置这个模型需要的连接并将它加载到我们的应用里[文件：`index.php`]：

```php
<?php

use Phalcon\Loader;
use Phalcon\Mvc\Micro;
use Phalcon\Di\FactoryDefault;
use Phalcon\Db\Adapter\Pdo\Mysql as PdoMysql;

// Use Loader() to autoload our model
$loader = new Loader();

$loader->registerNamespaces(
    [
        'Store\Toys' => __DIR__ . '/models/',
    ]
);

$loader->register();

$di = new FactoryDefault();

// Set up the database service
$di->set(
    'db',
    function () {
        return new PdoMysql(
            [
                'host'     => 'localhost',
                'username' => 'asimov',
                'password' => 'zeroth',
                'dbname'   => 'robotics',
            ]
        );
    }
);

// Create and bind the DI to the application
$app = new Micro($di);
```

## 获取数据

我们要实现的第一个`handler`，使用GET方法返回所有可用的robots。让我们要PHQL执行这个简单的查询，以JSON返回结果。[文件：`index.php`]

```php
<?php

// Retrieves all robots
$app->get(
    '/api/robots',
    function () use ($app) {
        $phql = 'SELECT * FROM Store\Toys\Robots ORDER BY name';

        $robots = $app->modelsManager->executeQuery($phql);

        $data = [];

        foreach ($robots as $robot) {
            $data[] = [
                'id'   => $robot->id,
                'name' => $robot->name,
            ];
        }

        echo json_encode($data);
    }
);
```

[PHQL](db-phql.md)，允许我们使用一种高级的、面向对象的SQL方言编写查询，它在内部转换为与我们正在使用的数据库系统一致的正确的SQL语句。在匿名函数中使用的子句`use`允许我们轻松地将一些变量从全局传递给局部。

通过名字处理程序搜索如下 [文件： `index.php`]：

```php
<?php

// Searches for robots with $name in their name
$app->get(
    '/api/robots/search/{name}',
    function ($name) use ($app) {
        $phql = 'SELECT * FROM Store\Toys\Robots WHERE name LIKE :name: ORDER BY name';

        $robots = $app->modelsManager->executeQuery(
            $phql,
            [
                'name' => '%' . $name . '%'
            ]
        );

        $data = [];

        foreach ($robots as $robot) {
            $data[] = [
                'id'   => $robot->id,
                'name' => $robot->name,
            ];
        }

        echo json_encode($data);
    }
);
```

通过字段`id`的搜索非常类似，本例中，我们也会通知robot是否找到 [文件：`index.php`]：

```php
<?php

use Phalcon\Http\Response;

// Retrieves robots based on primary key
$app->get(
    '/api/robots/{id:[0-9]+}',
    function ($id) use ($app) {
        $phql = 'SELECT * FROM Store\Toys\Robots WHERE id = :id:';

        $robot = $app->modelsManager->executeQuery(
            $phql,
            [
                'id' => $id,
            ]
        )->getFirst();



        // Create a response
        $response = new Response();

        if ($robot === false) {
            $response->setJsonContent(
                [
                    'status' => 'NOT-FOUND'
                ]
            );
        } else {
            $response->setJsonContent(
                [
                    'status' => 'FOUND',
                    'data'   => [
                        'id'   => $robot->id,
                        'name' => $robot->name
                    ]
                ]
            );
        }

        return $response;
    }
);
```


## 插入数据

把插入到请求体中的数据当作JSON字符串，我们还是用PHQL实现插入 [文件：`index.php`]：

```php
<?php

use Phalcon\Http\Response;

// Adds a new robot
$app->post(
    '/api/robots',
    function () use ($app) {
        $robot = $app->request->getJsonRawBody();

        $phql = 'INSERT INTO Store\Toys\Robots (name, type, year) VALUES (:name:, :type:, :year:)';

        $status = $app->modelsManager->executeQuery(
            $phql,
            [
                'name' => $robot->name,
                'type' => $robot->type,
                'year' => $robot->year,
            ]
        );

        // Create a response
        $response = new Response();

        // Check if the insertion was successful
        if ($status->success() === true) {
            // Change the HTTP status
            $response->setStatusCode(201, 'Created');

            $robot->id = $status->getModel()->id;

            $response->setJsonContent(
                [
                    'status' => 'OK',
                    'data'   => $robot,
                ]
            );
        } else {
            // Change the HTTP status
            $response->setStatusCode(409, 'Conflict');

            // Send errors to the client
            $errors = [];

            foreach ($status->getMessages() as $message) {
                $errors[] = $message->getMessage();
            }

            $response->setJsonContent(
                [
                    'status'   => 'ERROR',
                    'messages' => $errors,
                ]
            );
        }

        return $response;
    }
);
```


## 更新数据

数据更新与插入类似。作为参数传递的`id`指出哪个robot必须更新 [文件：`index.php`]：

```php
<?php

use Phalcon\Http\Response;

// Updates robots based on primary key
$app->put(
    '/api/robots/{id:[0-9]+}',
    function ($id) use ($app) {
        $robot = $app->request->getJsonRawBody();

        $phql = 'UPDATE Store\Toys\Robots SET name = :name:, type = :type:, year = :year: WHERE id = :id:';

        $status = $app->modelsManager->executeQuery(
            $phql,
            [
                'id'   => $id,
                'name' => $robot->name,
                'type' => $robot->type,
                'year' => $robot->year,
            ]
        );

        // Create a response
        $response = new Response();

        // Check if the insertion was successful
        if ($status->success() === true) {
            $response->setJsonContent(
                [
                    'status' => 'OK'
                ]
            );
        } else {
            // Change the HTTP status
            $response->setStatusCode(409, 'Conflict');

            $errors = [];

            foreach ($status->getMessages() as $message) {
                $errors[] = $message->getMessage();
            }

            $response->setJsonContent(
                [
                    'status'   => 'ERROR',
                    'messages' => $errors,
                ]
            );
        }

        return $response;
    }
);
```


## 删除数据

数据删除类似于数据更新。作为参数传递的`id`指出哪个robot必须删除 [文件：`index.php`]：

```php
<?php

use Phalcon\Http\Response;

// Deletes robots based on primary key
$app->delete(
    '/api/robots/{id:[0-9]+}',
    function ($id) use ($app) {
        $phql = 'DELETE FROM Store\Toys\Robots WHERE id = :id:';

        $status = $app->modelsManager->executeQuery(
            $phql,
            [
                'id' => $id,
            ]
        );

        // Create a response
        $response = new Response();

        if ($status->success() === true) {
            $response->setJsonContent(
                [
                    'status' => 'OK'
                ]
            );
        } else {
            // Change the HTTP status
            $response->setStatusCode(409, 'Conflict');

            $errors = [];

            foreach ($status->getMessages() as $message) {
                $errors[] = $message->getMessage();
            }

            $response->setJsonContent(
                [
                    'status'   => 'ERROR',
                    'messages' => $errors,
                ]
            );
        }

        return $response;
    }
);
```


## 测试我们的应用

我们使用 [curl](http://en.wikipedia.org/wiki/CURL) 来测试应用中的每一个路由，以验证操作是否正确。

获取所有的robots:

```bash
curl -i -X GET http://localhost/my-rest-api/api/robots

HTTP/1.1 200 OK
Date: Tue, 21 Jul 2015 07:05:13 GMT
Server: Apache/2.2.22 (Unix) DAV/2
Content-Length: 117
Content-Type: text/html; charset=UTF-8

[{"id":"1","name":"Robotina"},{"id":"2","name":"Astro Boy"},{"id":"3","name":"Terminator"}]
```

通过名字搜索robot：

```bash
curl -i -X GET http://localhost/my-rest-api/api/robots/search/Astro

HTTP/1.1 200 OK
Date: Tue, 21 Jul 2015 07:09:23 GMT
Server: Apache/2.2.22 (Unix) DAV/2
Content-Length: 31
Content-Type: text/html; charset=UTF-8

[{"id":"2","name":"Astro Boy"}]
```

通过id搜索robot:

```bash
curl -i -X GET http://localhost/my-rest-api/api/robots/3

HTTP/1.1 200 OK
Date: Tue, 21 Jul 2015 07:12:18 GMT
Server: Apache/2.2.22 (Unix) DAV/2
Content-Length: 56
Content-Type: text/html; charset=UTF-8

{"status":"FOUND","data":{"id":"3","name":"Terminator"}}
```

插入一个新robot:

```bash
curl -i -X POST -d '{"name":"C-3PO","type":"droid","year":1977}'
    http://localhost/my-rest-api/api/robots

HTTP/1.1 201 Created
Date: Tue, 21 Jul 2015 07:15:09 GMT
Server: Apache/2.2.22 (Unix) DAV/2
Content-Length: 75
Content-Type: text/html; charset=UTF-8

{"status":"OK","data":{"name":"C-3PO","type":"droid","year":1977,"id":"4"}}
```

尝试用一个已存在robot的名字来插入新robot：

```bash
curl -i -X POST -d '{"name":"C-3PO","type":"droid","year":1977}'
    http://localhost/my-rest-api/api/robots

HTTP/1.1 409 Conflict
Date: Tue, 21 Jul 2015 07:18:28 GMT
Server: Apache/2.2.22 (Unix) DAV/2
Content-Length: 63
Content-Type: text/html; charset=UTF-8

{"status":"ERROR","messages":["The robot name must be unique"]}
```

或者用一个未知的类型更新robot:

```bash
curl -i -X PUT -d '{"name":"ASIMO","type":"humanoid","year":2000}'
    http://localhost/my-rest-api/api/robots/4

HTTP/1.1 409 Conflict
Date: Tue, 21 Jul 2015 08:48:01 GMT
Server: Apache/2.2.22 (Unix) DAV/2
Content-Length: 104
Content-Type: text/html; charset=UTF-8

{"status":"ERROR","messages":["Value of field 'type' must be part of
    list: droid, mechanical, virtual"]}
```

最后，删除一个robot:

```bash
curl -i -X DELETE http://localhost/my-rest-api/api/robots/4

HTTP/1.1 200 OK
Date: Tue, 21 Jul 2015 08:49:29 GMT
Server: Apache/2.2.22 (Unix) DAV/2
Content-Length: 15
Content-Type: text/html; charset=UTF-8

{"status":"OK"}
```


## 结语

如我们所见，使用Phalcon的[微应用](application-micro.md)和[PHQL](db-phql.md)开发一个 [RESTful](http://en.wikipedia.org/wiki/Representational_state_transfer) API是非常容易的。