# 第一章 为什么使用shell编程

> No programming language is perfect. There is not even a single best language; there are only languages well suited or perhaps poorly suited for particular purposes.
> 没有任何一种程序设计语言是完美的，甚至没有一个最好的语言。只有在特定环境下适合的语言。
>
> -- Herbert Mayer

无论你是否打算真正编写shell脚本，只要你想要在一定程度上熟悉系统管理，了解掌握shell脚本的相关知识都是非常有必要的。例如Linux系统在启动的时候会执行`/etc/rc.d`目录下的shell脚本来恢复系统配置和准备服务。详细了解这些启动脚本对分析系统行为大有益处，何况，你很有可能会去修改它们呢。

编写shell脚本并不困难，shell脚本由许多小的部分组成，而其中只有数量相当少的与shell本身特性，操作和选项[^1]有关的部分才需要去学习。Shell语法非常简单朴素，很像是在命令行中调用和连接工具，你只需遵循很少一部分的"规则"就可以了。大部分短小的脚本通常在第一次就可以正常工作，即使是一个稍长一些的脚本，调试起来也十分简单。

> 在个人计算机发展的早期，BASIC语言让计算机专业人士能够在早期的微机上编写程序。几十年后，Bash脚本可以让所有仅对Linux或UNIX系统有初步了解的用户在现代计算机上做同样的事。
> 
> 我们现在已经可以做出一些又小又快的单板机，比如[树莓派](http://www.raspberrypi.org/)。Bash脚本提供了一种发掘这些有趣设备潜力的方式。

使用shell脚本构建一个复杂应用原型（prototype），不失为是一种虽有缺陷但非常快速的方式。在项目开发初期，使用脚本实现部分功能往往显得十分有用。在使用C/C++，Java，Perl或Python编写最终代码前，可以使用shell脚本测试，修补应用结构，提前发现重大缺陷。

Shell脚本与经典的UINX哲学相似，将复杂的任务划分为简单的子任务，将组件与工具连接起来。许多人认为比起新一代功能强大、高度集成的语言，例如Perl，shell脚本至少是一种在美学上更加令人愉悦的解决问题的方式，Perl试图做到面面俱到，但你必须强迫自己改变思维方式适应它。

Herbert Mayer曾说：“有用的语言需要数组、指针以及构建数据结构的通用机制”。如果依据这些标准，那shell脚本距“有用”还差得很远，甚至是“无用”的。

{% hint style="info" %}

什么时候不应该使用shell脚本

- 资源密集型的任务，尤其是对速度有要求（如排序、散列、递归[^2]等）
- 需要做大量的数学运算，例如浮点数运算，高精度运算或者复数运算（使用C++或FORTRAN代替）
- 有跨平台需求（使用C或者Java代替）
- 必须使用结构化编程的复杂应用（如变量类型检查、函数原型等）
- 影响系统全局的关键性任务
- 对安全性有高要求，需要保证系统的完整性以及阻止入侵、破解、恶意破坏
- 项目包含有连锁依赖关系的组件
- 需要大量的文件操作（Bash只能访问连续的文件，并且是以一种非常笨拙且低效的逐行访问的方式进行的）
- 需要使用多维数组
- 需要使用如链表、树等数据结构
- 需要产生或操作图像和图形用户接口（GUI）
- 需要直接访问系统硬件或外部设备
- 需要使用端口或套接字输入输出端口（Socket I/O）
- 需要使用库或旧程序的接口
- 私有或闭源的项目（Shell脚本直接将源代码公开，所有人都可以看到）

如果你的应用满足上述任意一条，你可以考虑使用更加强大的脚本语言，如Perl，Tcl，Python，Ruby等，或考虑使用编译型语言，如C，C++或Java等。即使如此，在开发阶段使用shell脚本建立应用原型也是十分有用的。

{% endhint %}

我们接下来将使用Bash。Bash是"Bourne-Again shell"的首字母缩略词[^3]，Bash来源于Stephen Bourne开发的Bourne shell（sh）。如今Bash已成为了大部分UNIX衍生版中shell脚本事实上的标准。本书所涉及的大部分概念在其他shell中也是适用的，例如Korn Shell，Bash从它当中继承了一部分的特性[^4]；又如C Shell及其变体（需要注意的是，1993年10月Tom Christiansen在[Usenet帖子](http://www.faqs.org/faqs/unix-faq/shell/csh-whynot/)中指出，因C Shell内部固有的问题，不推荐使用C Shell编程）

接下来的部分将是一些编写shell脚本的指导。这些指导很大程度上依赖于实例来阐述shell的特性。本书所有的例子都能够正常工作，并在尽可能的范围内进行过测试，其中的一部分已经运用在实际生产生活中。读者们可以使用这些在存档中的例子（文件名为`scriptname.sh`或`scriptname.bash`）[^5]，赋予它们可执行权限（`chmod u+rx scriptname`），然后执行它们看看会发生什么。如果[存档](http://bash.deta.in/abs-guide-latest.tar.bz2)不可用，读者朋友也可以从本书的HTML或者PDF版本中复制粘贴代码出来。需要注意的是，在部分例子中使用了一些暂时还未被解释的特性，这需要读者暂时跳过它们。

除特别说明，本书所有例子均由[本书作者](mailto:thegrendel.abs@gmail.com)编写。

> His countenance was bold and bashed not.
>
> —— Edmund Spenser

## 注记

{% hint style="info" %}
[1] 这些操作和选项被称为内建命令（builtin），是shell的内部特征。
{% endhint %}

{% hint style="info" %}
[2] 虽然递归可以在shell脚本中实现，但是它的效率很低且实现起来很复杂、不具有美感。
{% endhint %}

{% hint style="info" %}
[3] 首字母缩略词是由每一个单词的首字母拼接而成的易读的代替短语。这不是一个好习惯，通常会引起一些不必要的麻烦。
{% endhint %}

{% hint style="info" %}
[4] ksh88中的许多特性，甚至一些ksh93的特性都被合并到Bash中了。
{% endhint %}

{% hint style="info" %}
[5] 按照惯例，用户编写的Bourne shell脚本应该在文件名后加上`.sh`的扩展名。而那些系统脚本，比如在`/etc/rc.d`中的脚本通常不遵循这种规范。
{% endhint %}

[^1]: 这些操作和选项被称为内建命令（builtin），是shell的内部特征。
[^2]: 虽然递归可以在shell脚本中实现，但是它的效率很低且实现起来很复杂、不具有美感。
[^3]: 首字母缩略词是由每一个单词的首字母拼接而成的易读的代替短语。这不是一个好习惯，通常会引起一些不必要的麻烦。
[^4]: ksh88中的许多特性，甚至一些ksh93的特性都被合并到Bash中了。
[^5]: 按照惯例，用户编写的Bourne shell脚本应该在文件名后加上`.sh`的扩展名。而那些系统脚本，比如在`/etc/rc.d`中的脚本通常不遵循这种规范。
