# 教程—基础

在本教程中，我们将带你从头开始创建一个简单的注册表单应用程序。下面的指南将向你介绍Phalcon框架的设计方面。



如果你只是想开始，你可以跳过这一章，用我们的[开发工具](https://docs.phalconphp.com/en/3.2/devtools-usage)自动创建一个Phalcon项目。\(如果你没有经验碰到困难时，推荐你回到这里\)



## 文件结构

---

Phalcon的一个关键特征是它的松耦合，你可以为特定的应用程序创建便利的目录结构。也就是说，当你和其他人合作时保持一致性是有帮助的。所以本教程使用一个“标准”结构，如果你过去使用过其他MVC框架的话，会感觉如同在家一样自在。

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

注意：你将不会看到 vendor 目录，因为Phalcon的核心依赖都通过你安装的扩展加载到内存里了。如果你缺失了那一步没有安装Phalcon扩展，继续前请[返回](https://docs.phalconphp.com/en/3.2/installation)去完成安装。





