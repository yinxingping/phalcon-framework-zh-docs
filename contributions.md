# 为Phalcon贡献

Phalcon是一个开源项目，很大程度上依赖于志愿者的努力。我们欢迎大家的贡献！

请大家花点时间检阅一下这份文件，以便使我们的贡献过程更容易、更有效。

遵循这些指导方针，允许更好的沟通，更快地解决问题，并推动项目向前发展。


## 贡献

为Phalcon做贡献应该按照[GitHub pull request](https://help.github.com/articles/using-pull-requests/) 的方式进行。每个pull request将由一个核心贡献者(允许合并pull request的人)评审。根据pull request的类型和内容，可以立即合并，如果需要澄清或者拒绝，就可以暂停。

请确保你将你的pull request送到正确的分支，并且你已经重新编写了你的代码。


## 问题和支持

> 我们仅接受Bug报告、新feature请求和GitHub中的pull requests。对于关于框架用法的问题或支持请求请访问[官方论坛](https://phalcon.link/forum)。


## Bug报告清单

- 提交bug报告之前，确保你使用的是最新版本的Phalcon。核心团队不处理老版本的Bug。
- 如果你发现了一个bug，添加相关信息对于重现它是非常必要的。能够重现一个bug可以大大减少调查和修复它的时间。这些信息应该以脚本、小应用程序或者甚至是一个失败的测试的形式出现。请检阅[提交可重复测试](https://github.com/phalcon/cphalcon/wiki/Submit-Reproducible-Test)的更多信息。
- 作为报告的一部分，请包含其他信息，如操作系统、PHP版本、文件系统、web服务器、内存等。
- 如果你提交一个[Segmentation Fault](https://en.wikipedia.org/wiki/Segmentation_fault)错误，我们将要求回溯。为了获得更多信息，请检阅下一段：生成回溯。

### 生成回溯

有时因为[Segmentation Fault](https://en.wikipedia.org/wiki/Segmentation_fault)错误，Phalcon 可能会破坏你的一些web服务器进程。请帮助我们找到问题的答案，在bug报告中添加一个崩溃回溯。

请按照下面的指南来理解如何生成回溯:

- [生成gdb回溯](https://bugs.php.net/bugs-generating-backtrace.php)
- [Win32上用编译器生成回溯](http://bugs.php.net/bugs-generating-backtrace-win32.php)
- [调试符号](https://github.com/oerdnj/deb.sury.org/wiki/Debugging-symbols)
- [构建PHP](http://www.phpinternalsbook.com/build_system/building_php.html)



## Pull Request清单

- 不要将你的 pull 请求提交给 `master` 分支。 从需要的分支中分支，如果需要，在提交你的pull request之前，将其rebase到适当的分支。如果它不干净地与master合并，你可能会被要求rebase你的变更
- 不要在pull request中添加子模块更新，例如`composer.lock`，除非它们是合并提交
- 添加修补bug或新feature的测试，更多信息请看我们的[测试指南](https://github.com/phalcon/cphalcon/blob/master/tests/README.md) 
- Phalcon是用[Zephir](https://zephir-lang.com/)编写的，请不要提交直接修改C生成文件的commits，或者那些commits的功能/补丁是用C编程语言实现的
- 确保你所编写的PHP代码符合[公认PHP标准](http://www.php-fig.org/psr/)的通用风格和编码标准
- 提交pull request之前请移除任何对`ext/kernel`、`.zep.c`、`.zep.h`文件的更改

在提交**新功能**之前，请在GitHub上打开一个作为新问题的[NFR](new-feature-request.md)，讨论包含这个新功能或核心扩展的变更带来的影响。一旦功能被批准，确保你的PR包含以下内容:

- `CHANGELOG.md`更新
- 单元测试
- 文档或用法示例


## 获取支持

如果你有关于如何使用Phalcon的问题，请查看[支持页面](https://phalconphp.com/support)。


## 请求Features

如果你有个修改或新feature，请填写[NFR](new-feature-request.md)。

感谢

Phalcon团队