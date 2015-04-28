# 第二章 和Sha-Bang（#!）一起出发

> Shell编程声名显赫
>
> —— Larry Wall

### 本章目录
- [2.1 调用一个脚本](02_1_invoking_the_script.md)
- [2.2 小试牛刀](02_2_preliminary_exercises.md)

---

一个最简单的脚本其实就是将一系列的系统命令存储在文件中。最起码，它帮你省下了重复敲这一些命令的工作。

样例 2-1. *cleanup*：清理`/var/log`目录下的日志文件

	# Cleanup
	# 使用root权限运行
	
	cd /var/log
	cat /dev/null > messages
	cat /dev/null > wtmp
	echo "Log files cleaned up."
	
脚本仅仅是一些可以很容易地从终端或者控制台中调用的一行接一行的命令罢了，并没有什么奇怪的。将命令放在脚本中的好处就是让你不用再一遍遍重复的输入这些命令。脚本是一个程序、一款工具，它们可以很容易的被修改或者为特定的应用做定制。

样例 2-2. *cleanup*：改进的清理脚本

	#!/bin/bash
	# Bash脚本标准起始行。
	
	# Cleanup, version 2
	
	# 使用root权限运行。
	# 在这里插入代码用来打印错误信息，并且在未使用root权限时退出。
	
	LOG_DIR=/var/log
	# 变量比硬编码（hard-coded）要更合适一些
	cd $LOG_DIR
	
	cat /dev/null > messages
	cat /dev/null > wtmp
	
	
	echo "Logs cleaned up."
	
	exit # 这是正确终止脚本的方式。
	     # 不带参数的exit返回的是上一条命令的结果。
	     
现在让我们看一个真正意义上的脚本，而且我们可以做的更多。

样例 2-3. *cleanup*：改良、通用版

	#!/bin/bash
	# Cleanup, version 3
	
	# 注意：
	# --------
	# 这个脚本使用了许多后面才会涉及到的特性。
	# 当你阅读完整本书的一半以后，就没有任何困难了。
	
	
	LOG_DIR=/var/log
	ROOT_UID=0     # UID为0的用户才拥有root权限。
	LINES=50       # 缺省需要保存messages文件的行数。
	E_XCD=86       # 无法切换工作目录的错误码。
	E_NOTROOT=87   # 非root权限用户运行的错误码。
	
	
	
	# 使用root权限运行。
	if [ "$UID" -ne "$ROOT_UID" ]
	then
	  echo "Must be root to run this script."
	  exit $E_NOTROOT
	fi
	
	if [ -n "$1" ]
	# 测试命令行参数（保存的行数）是否为空
	then
	  lines=$1
	else
	  lines=$LINES # 使用缺省设置
	fi
	
	
	#  Stephane Chazelas建议使用下面的方法检查命令行参数，
	#+ 但是这已经超出了这个阶段教程的范围。
	#
	#    E_WRONGARGS=85  # Non-numerical argument (bad argument format).
	#    case "$1" in
	#    ""      ) lines=50;;
	#    *[!0-9]*) echo "Usage: `baseman $0` lines-to-cleanup";
	#     exit $E_WRONGARGS;;
	#    *       ) lines=$1;;
	#    esac
	#
	#* 在第十一章“循环与分支”中会对此作详细的阐述。
	
	
	cd $LOG_DIR
	
	if [ `pwd` != "$LOG_DIR" ]  # or   if [ "$PWD" != "$LOG_DIR" ]
	                            # Not in /var/log?
	then
	  echo "Can't change to $LOG_DIR"
	  exit $E_XCD
	fi  # 在清理日志之前，重复确认是否在正确的工作目录下。
	
	# 更高效的方法是：
	#
	# cd /var/log || {
	#   echo "Cannot change to necessary directory." > &2
	#   exit $E_XCD;
	# }
	
	
	
	
	tail -n $lines messages > msg.temp # 保存messages的最后一部分
	mv mesg.temp messages              # 替换原来的messages达到清理的目的
	
	#  cat /dev/null > messages
	#* 我们不再需要使用这个方法了，使用上面的方法更安全
	
	cat /dev/null > wtmp  #  ': > wtmp' 与 '> wtmp' 都有同样的效果
	echo "Log files cleaned up."
	#  注意在/var/log目录下其他的日志文件将不会被这个脚本清除
	
	exit 0
	#  返回0表示脚本运行成功

因为你也许并不希望清空全部的系统日志，所以这个脚本保留了messages日志的最后一部分。随着后面部分的学习，你将会不断的发现提高上述脚本效率的方法。

***

脚本起始行的sha-bang（#!）[^1]告诉系统这个脚本文件需要使用指定的命令解释器来执行。#!事实上是一个占两字节[^2]的幻数（magic number）。幻数是一个特殊的标记用来标识特殊类型的文件，在这里则是标记可执行shell脚本（在终端中输入`man magic`来了解更多的信息）。紧随#!的是一个路径名。这个路径指向了用来解释脚本的程序，它可以是shell，可以是程序设计语言，也可以是实用程序。然后解释器从头（#!的下一行）开始执行整个脚本的命令，同时忽略注释。[^3]

	#!/bin/sh
	#!/bin/bash
	#!/usr/bin/perl
	#!/usr/bin/tcl
	#!/bin/sed -f
	#!/bin/awk -f
	
上面的每一条脚本起始行都调用了不同的解释器，比如`/bin/sh`调用了缺省的shell（Linux系统中默认是bash）[^4]。而使用大部分UNIX商业发行版中缺省的Bourne shell，即`#!/bin/sh`，可以以牺牲Bash的特性为代价，在非Linux的机器上运行。当然，脚本遵循POSIX[^5] sh标准。

需要注意的是#!后的路径名必须正确，否则当你运行脚本时只会得到一条错误信息，通常是"Command not found."[^6]

当脚本仅包含一些通用的系统命令而不使用shell内部指令时，可以省略#!。而第三个例子需要#!是因为当对变量赋值时，例如`lines=50`，是使用了与shell特性相关的结构[^7]。再重复一次，`#!/bin/sh`调用的是缺省的shell解释器，在Linux系统中默认是`/bin/bash`。

> 这个例子鼓励读者使用模块化的方式编写脚本，并在平时记录和收集一些在以后可能会用到的代码模板。最终你将会拥有一个相当丰富易用的代码库。下面的代码是用来测试脚本被调用时的参数的数量是否正确。

	E_WRONG_ARGS=85
	script_parameters="-a -h -m -z"
	#                  -a = all, -h = help等等
	
	if [ $# -ne $Number_of_expected_args ]
	then
	  echo "Usage: `basename $0` $script_parameters"
	  # `basename $0`是脚本的文件名
	  exit $E_WRONG_ARGS
	fi

大多数情况下，你会针对特定的任务编写一个脚本。本章的第一个脚本就是这样一个例子。然后你也许会泛化（generalize）脚本使其能够适应相似的任务，比如用变量代替硬编码，用函数代替重复的代码都可以达到目的。

[^1]: 在文献中更常见的形式是she-bang或者sh-bang。它们都来源于词汇sharp(#)和bang(!)的连接。
[^2]: 一些UNIX的衍生版（基于4.2 BSD）声称他们使用四字节的幻数，在#!后增加一个空格，即`#! /bin/sh`。而[Sven Mascheck](http://www.in-ulm.de/~mascheck/various/shebang/#details)指出这是虚构的。
[^3]: <p>命令解释器首先将会解释#!这一行，而因为#!以#打头，因此解释器将其视作注释。起始行作为调用解释器的作用已经完成了。</p><p>事实上即使脚本中含有不止一个#!,bash也会将其解释为注释。</p><pre>#!/bin/bash<br/><br/>echo "Part 1 of script."<br/>a=1<br/><br/>#!/bin/bash<br/># 这并不会启动新的脚本<br/><br/>echo "Part 2 of script."<br/>echo $a  # $a的值仍旧为1</pre>
[^4]: <p>这里允许使用一些技巧。</p><pre>#!/bin/rm<br/># 自我删除的脚本<br/><br/># 当你运行这个脚本，除了这个脚本本身消失以外并不会发生什么。<br/><br/>WHATEVER=85<br/><br/>echo "This line will never print (betcha!)."<br/><br/>exit $WHATEVER  # 这没有任何关系。脚本将不会从这里退出。<br/>                # 尝试在脚本终止后打印echo $a。<br/>                # 得到的值将会是0而不是85.</pre>当然你也可以建立一个起始行是`#!/bin/more`的README文件，并且使它可以执行。结果就是这个文件成为了一个可以打印本身的文件。（查看样例 19-3，使用`cat`命令的here document也许是一个更好的选择）
[^5]: 可移植操作系统接口（POSIX）尝试标准化类UNIX操作系统。POSIX规范可以在[Open Group site](http://www.opengroup.org/onlinepubs/007904975/toc.htm)中查看。
[^6]: 为了避免这种情况的发生，可以使用`#!/bin/env bash`作为起始行。这在bash不在`/bin`的UNIX系统中会有效果。
[^7]: 如果bash是系统的缺省shell，那么脚本并不一定需要#!作为起始行。但是当你在其他的shell中运行脚本，例如tcsh，则需要使用#!。