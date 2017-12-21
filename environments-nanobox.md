# 概述

[Nanobox](https://nanobox.io) 是一个可移植的微型平台，用于开发和部署应用程序。在本地工作时，Nanobox使用Docker来配置一个特定需求的虚拟开发环境。当你准备好部署到实时服务器上时，Nanobox将会采用同样的环境，并在你的云服务提供商上进行选择，在那里你可以通过Nanobox仪表盘来管理和扩展你的应用程序。

在这篇文章中，我们将会在当地启动和运行一个全新的Phalcon应用程序，除了Nanobox之外，不需要安装任何东西。首先[创建一个免费的Nanbox账户](https://dashboard.nanobox.io/users/register)，然后[下载并安装Nanobox](https://dashboard.nanobox.io/download)。


## 创建一个新项目

创建一个项目文件夹并`cd`进入：

```bash
mkdir nanobox-phalcon && cd nanobox-phalcon
```

## 添加一个 `boxfile.yml`

Nanobox使用[`boxfile.yml`](https://docs.nanobox.io/boxfile/) 来构建和配置你的应用的运行时和环境。在你项目的根目录下创建一个`boxfile.yml`如下：

```yaml
run.config:
  engine: php
  engine.config:
    runtime: php-7.1
    document_root: public
    extensions:
      - phalcon
  extra_steps:
    - echo "alias phalcon=\'phalcon.php\'" >> /data/var/home/gonano/.bashrc
```

这告诉Nanobox：

- 使用PHP[引擎](https://docs.nanobox.io/engines/)，构建你的应用的运行时的一组脚本
- 使用PHP 7.1
- 设置Apache文档根到 `public`.
- 包含Phalcon扩展。 *Nanobox自带了少之又少的扩展，所以你可能需要包含其他扩展，更多相关信息可以浏览[这里](https://guides.nanobox.io/php/phalcon/php-extensions/).*
- 为Phalcon开发工具添加一个bash别名，那样你就可以直接使用 `phalcon`命令了


## 添加Phalcon开发工具到你的 `composer.json`

在你的项目根目录下创建一个 `composer.json`文件，并添加 `phalcon/devtools`包到你的开发需求项：

```json
{
    "require-dev": {
        "phalcon/devtools": "~3.0.3"
    }
}
```

> *注意* Phalcon开发工具的版本依赖于你正在使用的PHP版本。


## 启动Nanobox并生成一个新的Phalcon应用

从项目的根运行以下命令来启动Nanobox，并生成一个新的Phalcon应用。当Nanobox启动时，,PHP引擎将自动安装和启用Phalcon扩展。运行`composer install`，这将安装Phalcon开发工具，然后将你带到虚拟环境里的一个交互式控制台内。你的工作目录被安装到VM的`/app`目录中，因此在进行更改时，它们将在VM和本地工作目录中得到反映。

```bash
# start nanobox and drop into a nanobox console
nanobox run

# cd into the /tmp directory
cd /tmp

# generate a new phalcon app
phalcon project myapp

# change back to the /app dir
cd -

# copy the generated app into your project
cp -a /tmp/myapp/* .

# exit the console
exit
```


## 本地运行应用

在实际运行你的新Phalcon应用程序之前，我们建议使用Nanobox来添加一个DNS别名。这将为您的本地`hosts`文件添加一个指向您的dev环境的条目，并提供一种方便的方式从浏览器访问您的应用程序。

```bash
nanobox dns add local phalcon.dev
```

Nanobox提供了一个`php-server`助手脚本，它启动了Apache(或Nginx，取决于你的`boxfile.yml`配置)和PHP。当使用`nanobox run`命令时，它将启动本地开发环境，并立即运行你的应用。

```bash
nanobox run php-server
```

运行起来后，你可以在 [phalcon.dev](http://phalcon.dev) 访问你的应用。

## 检查环境

你的虚拟环境包含运行你的Phalcon应用所需的一切。你可以随意地四周逛逛就行。

```bash
# drop into a Nanobox console
nanobox run

# check the php version
php -v

# check that phalcon devtools are available
phalcon info

# check that your local codebase is mounted
ls

# exit the console
exit
```

## Phalcon和Nanobox

Nanobox给了你在一个隔离的虚拟环境中开发和运行你的Phalcon应用程序所需的一切。使用你项目中的`boxfile.yml`，协作者可以通过运行`nanobox run`，在几分钟内启动和运行你的应用。

Nanobo的[Phalcon快速入门](https://github.com/nanobox-quickstarts/nanobox-phalcon)，包括了这篇文章中介绍的所有内容。他们也有作为指南的[在Nanobox中使用Phalcon](https://guides.nanobox.io/php/phalcon/)。在将来的文章中，我们想覆盖在Nanobox里使用Phalcon的其他方面，包括添加和连接到一个数据库，部署Phalcon到生产环境等。如果你有兴趣 [让我们在Twitter上知道](http://twitter.com/nanobox_io)。