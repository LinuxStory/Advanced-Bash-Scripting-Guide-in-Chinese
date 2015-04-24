# 第三章 特殊字符

是什么让一个字符特殊呢？如果一个字符不仅具有字面意义，而且具有元意（meta-meaning），我们称它为特殊字符。特殊字符同命令和关键词（keywords）一样，是bash脚本的组成部分之一。

你在脚本和其他地方都能够找到特殊字符。

### \# ###

注释。如果脚本的开头是#（除了#!），那么代表这一行是注释，它将不会被执行。

    # 这是一行注释
    
注释也可能会在一行命令结束之后出现。

    echo "A comment will follow." # 这儿可以写注释
    #                            ^ 注意在#之前有一个空格

注释也可以出现在一行开头的一系列空白符（whitespace）之后。

	    # 这个注释前面存在一个制表符（tab）
	    
注释甚至也可以嵌入管道（pipe）之中。

	initial=( `cat "$startfile" | sed -e '/#/d' | tr -d '\n' |\
	# 删除所有带'#'注释符号的行
	           sed -e 's/\./\. /g' -e 's/_/_ /g'` )
	# 摘录自脚本 life.sh

![notice](http://tldp.org/LDP/abs/images/caution.gif) 命令不能接在同一行的注释之后。没有任何一种方法可以结束注释，因此为了让新的命令能够执行，在输入下一个命令时应先另起一行。

![extra](http://tldp.org/LDP/abs/images/note.gif) 当然，在`echo`语句中被引用中或被转义的#并不会被认为是注释。同样的，在某些参数代换结构和常数表达式中的#也不会被认为是注释。

	echo "The # here does not begin a comment."
	echo 'The # here does not begin a comment.'
	echo The \# here does not begin a comment.
	echo The # here begins a comment.
	
	echo ${PATH#*:}       # 参数代换而非注释
	echo $(( 2#101011 ))  # 进制转换而非注释
	
	# 感谢S.C.
	
标准的引用和转义符（" ' \）转义了#。

一些模式匹配操作符同样使用了#。

### ;

命令分隔符[分号]。允许在同一行内放置两条或更多的命令。

	echo hello; echo there
	
	if [ -x "$filename" ]; then    #  注意在分号以后有一个空格
	#+                   ^^
	  echo "File $filename exists."; cp $filename $filename.bak
	else   #                       ^ ^
	  echo "File $filename not found."; touch $filename
	fi; echo "File test complete."
	
注意在有时候";"需要被转义。

### ;;

`case`条件语句终止符[双分号]。

	case "$variable" in
	  abc)  echo "\$variable = abc" ;;
	  xyz)  echo "\$variable = xyz" ;;
	esac
	
### ;;&, ;&

`case`条件语句终止符（需Bash 4版本及以上）。

### .

句点命令[句点]。等价于`source`命令（查看样例 15-22）。这是一个bash的内建命令。

### .

句点可以作为文件名的一部分。它是隐藏文件的文件名前缀，使用不带参数的`ls`命令时不会直接显示出来。

	bash$ touch .hidden-file
	bash$ ls -l
	total 10
	 -rw-r--r--    1 bozo      4034 Jul 18 22:04 data1.addressbook
	 -rw-r--r--    1 bozo      4602 May 25 13:58 data1.addressbook.bak
	 -rw-r--r--    1 bozo       877 Dec 17  2000 employment.addressbook
	
	
	bash$ ls -al
	total 14
	 drwxrwxr-x    2 bozo  bozo      1024 Aug 29 20:54 ./
	 drwx------   52 bozo  bozo      3072 Aug 29 20:51 ../
	 -rw-r--r--    1 bozo  bozo      4034 Jul 18 22:04 data1.addressbook
	 -rw-r--r--    1 bozo  bozo      4602 May 25 13:58 data1.addressbook.bak
	 -rw-r--r--    1 bozo  bozo       877 Dec 17  2000 employment.addressbook
	 -rw-rw-r--    1 bozo  bozo         0 Aug 29 20:54 .hidden-file
	 
当句点在目录中时，单个句点代表当前的工作目录，两个句点代表上级目录。

	bash$ pwd
	/home/bozo/projects
	
	bash$ cd .
	bash$ pwd
	/home/bozo/projects
	
	bash$ cd ..
	bash$ pwd
	/home/bozo/
	
句点通常代表文件移动的目的地（目录），在下文代表的是当前目录。

	bash$ cp /home/bozo/current_work/junk/* .
	
> 复制所有的“垃圾文件”到当前目录

### .

句点匹配符。句点在正则表达式中，意味着匹配一个任意字符。

### "

部分引用[双引号]。在字符串中保留大部分的特殊字符。详细内容查看[第五章](05_quoting.md)

### '

全引用[单引号]。在字符串中保留所有的特殊字符。是部分引用的强化版。详细内容查看[第五章](05_quoting.md)

### ,

逗号运算符。逗号运算符[^1]将一系列的算术表达式串联在一起。所有的算术表达式都会被执行，但只有最后一个被计算的表达式的值将会被返回。

	let "t2 = ((a = 9, 15 / 3))"
	# 设定 "a = 9" 与 "t2 = 15 / 3"
	
逗号运算符也可以用来连接字符串。

	for file in /{,usr/}bin/*calc
	#             ^    在 /bin 与 /usr/bin 目录中
	#+                 找到所有的以"calc"结尾的可执行文件
	do
	        if [ -x "$file" ]
	        then
	          echo $file
	        fi
	done
	
	# /bin/ipcalc
	# /usr/bin/kcalc
	# /usr/bin/oidcalc
	# /usr/bin/oocalc
	
	
	# 感谢Rory Winston指出以上这些
	
### ,, ,

在参数代换中进行小写字母转换（Bash 4中新增）。

### \

转义符[反斜杠]。单字符的引用机制。

`\x`转义了字符x。转义达到了引用的x的目的，等价于'x'。转义符也可以用来引用"与'，可以让他们显式的表达。

查看第五章了解更加深入的转义的解释。

### /

文件名路径分隔符[正斜杠]。它分割了文件名的各个部分（比如 `/home/bozo/projects/Makefile`）

它也在算术运算中充当除法运算符。

### `

命令替换符。`` `command` ``结构可以使得命令的输出结果赋值给一个变量。通常也称作后引号（backquotes）或反引号（backticks）。

### :

空命令[冒号]。它在shell中等价于"NOP"（即no op，空操作），同时也被认为是shell内建命令true的同义词。它本身是Bash的内建命令，返回值是true（0）。

	:
	echo $?   # 0

它在死循环中的情况：

	while :
	do
	   operation-1
	   operation-2
	   ...
	   operation-n
	done
	
	# 等价于
	#    while true
	#    do
	#      ...
	#    done

它在 `if/then` 中作为占位符的情况：

	if condition
	then :   # 什么都不做，只是建立一个分支
	else
	   take-some-action
	fi

它在二元操作中作为占位符的情况，查看样例 8-2或缺省参数部分。

	: ${username=`whoami`}
	# ${username=`whoami`}   如果没有:就会报错
	#                        除非 "username" 是一个命令或内建命令
	
	: ${1?"Usage: $0 ARGUMENT"}     # 摘自样例脚本 "usage-message.sh"

查看样例 19-10可以了解空命令在here document中作为占位符的情况。

使用参数代换给字符串变量赋值（查看样例 10-7）。

	: ${HOSTNAME?} ${USER?} ${MAIL?}
	#  如果其中一个或多个必要的环境变量没有被设置
	#+ 那么将会打印错误

查看变量扩展或者字串替换部分了解空命令在其中的作用。

与`>`重定向操作符结合，可以在不改变权限的情况下清空文件。如果文件不存在，那么将会创建这个文件。

	: > data.xxx   # 文件 "data.xxx" 已经被清空
	
	# 与 cat /dev/null >data.xxx 作用相同
	# 但是这个操作并不会产生一个新的进程，因为 ":" 是一个内建命令。
	
可以查看样例 16-15。

与`>>`重定向操作符结合将不会清空任何已经存在的文件（`: >> target_file`）。但是在文件不存在的情况下，将会创建这个文件。

![extra](http://tldp.org/LDP/abs/images/note.gif) 以上只适用于普通文件，不适用于管道、符号链接和特定的文件。

它也可以用来作为一行注释的开头，尽管我们并不推荐这么做。使用 # 可以使解释器对这一行不进行错误检测，所以几乎所有都可以出现在注释中。但是下面的情况却不是这样的：

	: 这一行注释将会产生一个错误，( if [ $x -eq 3] )。

它也可以作为一个域的分隔符，比如在`/etc/passwd`和 `$PATH` 变量中。
	
	bash$ echo $PATH
	/usr/local/bin:/bin:/usr/bin:/usr/X11R6/bin:/sbin:/usr/sbin:/usr/games

将冒号作为函数名称也是可行的。

	:()
	{
	  echo "The name of this function is "$FUNCNAME" "
	  # 为什么要使用冒号作为函数名称？
	  # 这是一种混淆代码的方式。
	}
	
	:
	
	# 函数名称是 :
	
这并不具有可移植性，并且也不推荐使用。事实上，在Bash的最近的发行版中已经禁用了这种用法。但还可以使用下划线_来替代。

冒号也可以作为非空函数的占位符。

	not_empty ()
	{
	  :
	} # 含有空指令，因此这并不是一个空函数。
	
### !

反转（或否定）测试操作符的意义或返回状态[感叹号]。! 操作符反转已执行的命令的返回状态（查看样例 6-2）。它同时可以反转测试操作符的意义，例如可以将相等（=）反转成不等（!=）。它是一个Bash关键词。

在一些特殊的场景下，它也会出现在间接变量引用中。

在另外一些特殊场景下，即在命令行下可以使用 ! 调用Bash的查看历史机制（查看附录 L）。需要注意的是在脚本中，这个机制是被禁用的。

### *

通配符[星号]。它在进行文件名匹配（globbing）时扩展文件名。如果它独立出现，则匹配该目录下的所有文件名。

	bash$ echo *
	abs-book.sgml add-drive.sh agram.sh alias.sh

它在正则表达式中表示任意多个（包括0）字符。

### *

算术运算符。在进行算术运算时，* 代表乘法运算。

** 双星号可以表示乘方运算或扩展文件匹配。

### ?

测试操作符。在一些特定的语句中，? 表示一个条件测试。

在一个双圆括号结构中，? 可以表示一个类似C语言风格的三元运算符的一个组成部分。[^2]

`condition?result-if-true:result-if-false`

	(( var0 = var1<98?9:21 ))
	#
	
	# if [ "$var1" -lt 98 ]
	# then
	#   var0=9
	# else
	#   var0=21
	# fi

在参数代换表达式中，? 用来测试一个变量是否已经被赋值。

### ?

通配符。它在进行文件名匹配（globbing）时以单字符通配符扩展文件名。在扩展正则表达式中表示匹配一个单字符。

### $

用来进行变量替换（即变量的内容）。

	var1=5
	var2=23skidoo
	
	echo $var1     # 5
	echo $var2     # 23skidoo

如果在变量名前有 $，则代表这个变量的值。

### $

行结束符。在正则表达式中，"$"匹配字符串的结束位置。

### ${}

参数代换。

### $'...'

引用字符串扩展。这个结构将一个或多个转义的八进制或十六进制的值转换成ASCII[^3]或Unicode字符。

### $*, $@

位置参数。

### $?

返回状态变量。这个变量保存一个命令、一个函数或该脚本自身的返回状态。

### $$

进程ID变量。这个变量保存该运行脚本的进程ID[^4]。

### ()

命令组。

`(a=hello; echo $a)`

> ![notice](http://tldp.org/LDP/abs/images/important.gif) 通过括号执行一系列命令会产生一个子shell（subshell）。
> >在括号中的变量，即在子shell中的变量，在脚本的其他部分是不可见的。父进程脚本不能访问子进程（子shell）中所创建的变量。
> >
	a=123
	( a=321; )
> >	
	echo "a = $a"   # a = 123
	# 在括号中的 "a" 看起来像一个局部变量。
	
数组初始化。

`Array=(element1 element2 element3)`


[^1]: 操作符（operator）用来执行表达式（operation）。最常见的例子就是算术运算符+ - * /。在Bash中，操作符和关键字的概念有一些重叠。