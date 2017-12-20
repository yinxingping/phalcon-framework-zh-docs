# FAQ - 常问问题


## Phalcon是什么

Phalcon是一个作为C语言扩展提供的开源、全栈PHP框架。


## Phalcon是如何工作的

对于PHP或用PHP编写的项目来说，它并 *不是* 一个加速器。*(为类php语言生成高性能的C扩展请参阅[Zephir](https://github.com/phalcon/zephir))*

Phalcon是一个使用低级C语言实现其功能的框架。C扩展被编译为与你的PHP代码一起加载，提高应用程序的速度并降低开销。

#### Phalcon通过以下达成这样的目标：

- 利用原生编译的优势：生成一个[二进制可执行表示法](https://en.wikipedia.org/wiki/Machine_code)，处理器可以直接理解和执行代码，没有在虚拟机(VM)中运行字节码的开销。

- 减少内存占用：通过使用优化的特定C结构和静态类型C编译器来减少内存占用，比如GCC/CLANG/VCC。这些对代码的操作 [一些优化](https://en.wikipedia.org/wiki/Category:Compiler_optimizations)提高了性能。

- 将变量和数据存入堆栈的能力：这些通常有更高访问 [位置](https://en.wikipedia.org/wiki/Locality_of_reference) 。

- 分支预测更容易：因为它直接在用户代码上运行，而不是在VM实现上。* Mystical在[Stack Overflow](https://stackoverflow.com/a/11227902/1661465)上给出了一个很好的解释 *

- 直接访问内部结构和函数来减少 [计算开销](https://en.wikipedia.org/wiki/CPU-bound)。

- [使用Profile引导优化(PGO)](https://en.wikipedia.org/wiki/Profile-guided_optimization) 在现有执行Profile的基础上提高性能。

*按照 [维基百科](https://en.wikipedia.org/wiki/Profile-guided_optimization)，Profile引导优化 (PGO, 有时被宣称为pogo)，在计算机编程中是一个编译器优化技术，它使用profiling来提高运行时性能。*

Phalcon依赖于一些PHP的内部设计如内存管理、[垃圾回收](https://en.wikipedia.org/wiki/Garbage_collection_(computer_science))和其内部结构。这些方面的任何改进，对Phalcon的性能有着和对PHP一样的积极的影响。

credit:< https://github.com/andresgutierrez >


## 我怎么帮忙？

加入我们的[Discord](https://phalcon.link/discord)频道，在 [GitHub](https://github.com/phalcon)访问我们， 或浏览我们的网站 [https://phalconphp.com/](https://phalconphp.com/en/)。