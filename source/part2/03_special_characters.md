# 第三章 特殊字符

是什么让一个字符特殊呢？如果一个字符不仅具有字面意义，而且具有元意（meta-meaning），我们称它为特殊字符。特殊字符同命令和关键词（keywords）一样，是bash脚本的组成部分之一。

你在脚本和其他地方都能够找到特殊字符。

### \# ###

注释。如果脚本的开头是#（除了#!），那么代表这一行是注释，它将不会被执行。

```bash
# 这是一行注释
```
    
注释也可能会在一行命令结束之后出现。

```bash
echo "A comment will follow." # 这儿可以写注释
#                            ^ 注意在#之前有一个空格
```

注释也可以出现在一行开头的一系列空白符（whitespace）之后。

```bash
	# 这个注释前面存在一个制表符（tab）
```
	    
注释甚至也可以嵌入管道（pipe）之中。

```bash
initial=( `cat "$startfile" | sed -e '/#/d' | tr -d '\n' |\
# 删除所有带'#'注释符号的行
           sed -e 's/\./\. /g' -e 's/_/_ /g'` )
# 摘录自脚本 life.sh
```

> ![notice](http://tldp.org/LDP/abs/images/caution.gif) 命令不能接在同一行的注释之后。没有任何一种方法可以结束注释，因此为了让新的命令能够执行，在输入下一个命令时应先另起一行。

> ![extra](http://tldp.org/LDP/abs/images/note.gif) 当然，在`echo`语句中被引用中或被转义的#并不会被认为是注释。同样的，在某些参数代换结构和常数表达式中的#也不会被认为是注释。

```bash
echo "The # here does not begin a comment."
echo 'The # here does not begin a comment.'
echo The \# here does not begin a comment.
echo The # here begins a comment.

echo ${PATH#*:}       # 参数代换而非注释
echo $(( 2#101011 ))  # 进制转换而非注释

# 感谢S.C.
```
	
标准的引用和转义符（" ' \）转义了#。

一些模式匹配操作符同样使用了#。

### ;

命令分隔符[分号]。允许在同一行内放置两条或更多的命令。

```bash
echo hello; echo there

if [ -x "$filename" ]; then    #  注意在分号以后有一个空格
#+                   ^^
  echo "File $filename exists."; cp $filename $filename.bak
else   #                       ^^
  echo "File $filename not found."; touch $filename
fi; echo "File test complete."
```
	
注意在有时候";"需要被转义。

### ;;

`case`条件语句终止符[双分号]。

```bash
case "$variable" in
  abc)  echo "\$variable = abc" ;;
  xyz)  echo "\$variable = xyz" ;;
esac
```
	
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

```bash
let "t2 = ((a = 9, 15 / 3))"
# 设定 "a = 9" 与 "t2 = 15 / 3"
```
	
逗号运算符也可以用来连接字符串。

```bash
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
# /usr/bin/oocalcå


# 感谢Rory Winston指出以上这些
```

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

```bash
:
echo $?   # 0
```

它在死循环中的情况：

```bash
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
```

它在 `if/then` 中作为占位符的情况：

```bash
if condition
then :   # 什么都不做，只是建立一个分支
else
   take-some-action
fi
```

它在二元操作中作为占位符的情况，查看样例 8-2或缺省参数部分。

```bash
: ${username=`whoami`}
# ${username=`whoami`}   如果没有:就会报错
#                        除非 "username" 是一个命令或内建命令

: ${1?"Usage: $0 ARGUMENT"}     # 摘自样例脚本 "usage-message.sh"
```

查看样例 19-10可以了解空命令在here document中作为占位符的情况。

使用参数代换给字符串变量赋值（查看样例 10-7）。

```bash
: ${HOSTNAME?} ${USER?} ${MAIL?}
#  如果其中一个或多个必要的环境变量没有被设置
#+ 那么将会打印错误
```

查看变量扩展或者字串替换部分了解空命令在其中的作用。

与`>`重定向操作符结合，可以在不改变权限的情况下清空文件。如果文件不存在，那么将会创建这个文件。

```bash
: > data.xxx   # 文件 "data.xxx" 已经被清空

# 与 cat /dev/null >data.xxx 作用相同
# 但是这个操作并不会产生一个新的进程，因为 ":" 是一个内建命令。
```
	
可以查看样例 16-15。

与`>>`重定向操作符结合将不会清空任何已经存在的文件（`: >> target_file`）。但是在文件不存在的情况下，将会创建这个文件。

> ![extra](http://tldp.org/LDP/abs/images/note.gif) 以上只适用于普通文件，不适用于管道、符号链接和特定的文件。

它也可以用来作为一行注释的开头，尽管我们并不推荐这么做。使用 # 可以使解释器对这一行不进行错误检测，所以几乎所有都可以出现在注释中。但是下面的情况却不是这样的：

```bash
: 这一行注释将会产生一个错误，( if [ $x -eq 3] )。
```

它也可以作为一个域的分隔符，比如在`/etc/passwd`和 `$PATH` 变量中。
	
	bash$ echo $PATH
	/usr/local/bin:/bin:/usr/bin:/usr/X11R6/bin:/sbin:/usr/sbin:/usr/games

将冒号作为函数名称也是可行的。

```bash
:()
{
  echo "The name of this function is "$FUNCNAME" "
  # 为什么要使用冒号作为函数名称？
  # 这是一种混淆代码的方式。
}

:

# 函数名称是 :
```
	
这并不具有可移植性，并且也不推荐使用。事实上，在Bash的最近的发行版中已经禁用了这种用法。但还可以使用下划线_来替代。

冒号也可以作为非空函数的占位符。

```bash
not_empty ()
{
  :
} # 含有空指令，因此这并不是一个空函数。
```
	
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

```bash
(( var0 = var1<98?9:21 ))
#

# if [ "$var1" -lt 98 ]
# then
#   var0=9
# else
#   var0=21
# fi
```

在参数代换表达式中，? 用来测试一个变量是否已经被赋值。

### ?

通配符。它在进行文件名匹配（globbing）时以单字符通配符扩展文件名。在扩展正则表达式中表示匹配一个单字符。

### $

用来进行变量替换（即变量的内容）。

```bash
var1=5
var2=23skidoo

echo $var1     # 5
echo $var2     # 23skidoo
```

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
```bash
a=123
( a=321; )
> >
echo "a = $a"   # a = 123
# 在括号中的 "a" 看起来像一个局部变量。
```
	
数组初始化。

`Array=(element1 element2 element3)`

### {xxx,yyy,zzz,...}

大括号扩展结构。

```bash
echo \"{These,words,are,quoted}\"   # " 将会作为单词的前缀和后缀
# "These" "words" "are" "quoted"


cat {file1,file2,file3} > combined_file
# 将 file1, file2 与 file3 拼接在一起后写入 combined_file 中。

cp file22.{txt,backup}
# 将 "file22.txt" 拷贝为 "file22.backup"
```

这个命令可以作用于大括号内由逗号分隔的文件描述列表。[^5] 文件名扩展（匹配）作用于大括号间的文件描述。

> ![notice](http://tldp.org/LDP/abs/images/caution.gif) 除非空格被引用或被转义，否则不应该在大括号中出现。
>
```bash
echo {file1,file2}\ :{\ A," B",' C'}
file1 : A file1 : B file1 : C file2 : A file2 : B file2 : C
```

### {a..z}

扩展大括号扩展结构。

```bash
echo {a..z} #  a b c d e f g h i j k l m n o p q r s t u v w x y z
# 输出 a 到 z 之间所有的字母。
	
echo {0..3} # 0 1 2 3
# 输出 0 到 3 之间所有的数字。


base64_charset=( {A..Z} {a..z} {0..9} + / = )
# 使用扩展大括号扩展符初始化一个数组。
# 摘自由 vladz 编写的样例脚本 "base64.sh"。
```

Bash的第三个版本引入了 {a..z} 这样的扩展大括号扩展结构。

### {}

代码块[大括号]，又被称作内联组（inline group）。它实际上创建了一个匿名函数（anonymous function），即没有名字的函数。但是，不同于那些“标准”函数，代码块内的变量在脚本的其他部分仍旧是可见的。

	bash$ { local a;
	              a=123; }
	bash: local: can only be used in a
	function

```bash
a=123
{ a=321; }
echo "a = $a"   # a = 321   (代码块内所赋的值)

# 感谢S.C.
```

代码块可以经由I/O重定向进行输入或输出。

样例 3-1. 代码块及I/O重定向

```bash
#!/bin/bash
# 读取文件 /etc/fstab。

File=/etc/fstab

{
read line1
read line2
} < $File

echo "First line in $File is:"
echo "$line1"
echo
echo "Second line in $File is:"
echo "$line2"

exit 0

# 那么，你知道如何解析每一行不同的字段么？
# 提示：使用 hint 或者
# Hans-Joerg Diers 建议使用Bash的内建命令 set。
```

样例 3-2. 将代码块的输出保存至文件中

```bash
#!/bin/bash
# rpm-check.sh

# 查询一个rpm文件的文件描述、包含文件列表，以及是否可以被安装。
# 将输出保存至文件。
#
# 这个脚本使用代码块来描述。

SUCCESS=0
E_NOARGS=65

if [ -z "$1" ]
then
  echo "Usage: `basename $0` rpm-file"
  exit $E_NOARGS
fi  

{ # 代码块起始
  echo
  echo "Archive Description:"
  rpm -qpi $1       # 查询文件描述。
  echo
  echo "Archive Listing:"
  rpm -qpl $1       # 查询文件列表。
  echo
  rpm -i --test $1  # 查询是否可以被安装。
  if [ "$?" -eq $SUCCESS ]
  then
    echo "$1 can be installed."
  else
    echo "$1 cannot be installed."
  fi  
  echo              # 代码块结束。
} > "$1.test"       # 输出重定向至文件。

echo "Results of rpm test in file $1.test"

# 查看 rpm 的帮助获得解释。

exit 0
```
> ![extra](http://tldp.org/LDP/abs/images/note.gif) 与由圆括号包裹的命令组不同，由大括号包裹的代码块将不会产生一个子进程。[^6]
> 
> 可以使用非标准的 for 循环语句来遍历代码块。

### {}

文本占位符。在 `xargs -i` 后作为输出的占位符来使用。

```bash
ls . | xargs -i -t cp ./{} $1
#            ^^         ^^

# 摘自 "ex42.sh" (copydir.sh)
```

### {} \;

路径名。通常在 `find` 命令中使用，但这不是shell的内建命令。

> 定义：路径名是包含完整路径的文件名，例如`/home/bozo/Notes/Thursday/schedule.txt`。我们通常又称之为绝对路径。

> ![extra](http://tldp.org/LDP/abs/images/note.gif) 在执行`find -exec`时最后需要加上`;`，但是分号需要被转义以保证其不会被shell解释。

### [ ]

测试。在 [ ] 之间填写测试表达式。值得注意的是，[ 是shell内建命令 `test` 的一个组成部分，而不是外部命令 `/usr/bin/test` 的连接。

### [[ ]]

测试。在 [[ ]] 之间填写测试表达式。相比起单括号测试 （[ ]），它更加的灵活。它是一个shell的关键字。

详情查看关于 [[ ]] 结构的讨论。

### [ ]

数组元素。在数组中，可以使用中括号的偏移量来用来访问数组中的每一个元素。

```bash
Array[1]=slot_1
echo ${Array[1]}
```

### [ ]

字符集、字符范围。在正则表达式中，中括号用来匹配指定字符集或字符范围内的任意字符。

### $[ ... ]

整数扩展符。在 $[ ] 中可以计算整数的算术表达式。

```bash
a=3
b=7

echo $[$a+$b]   # 10
echo $[$a*$b]   # 21
```

### (( ))

整数扩展符。在 (( )) 中可以计算整数的算术表达式。

详情查看关于 (( ... )) 结构的讨论。

### > &> >& >> < <>

重定向。

`scriptname >filename` 将脚本 *scriptname* 的输出重定向到 *filename* 中。如果文件存在，那么覆盖掉文件内容。

`command &>filename` 将脚本 *scriptname* 的标准输出 *stdout* 和命令的错误输出 *stderr* 重定向到 *filename*。

> ![extra](http://tldp.org/LDP/abs/images/note.gif) 重定向在用于清除测试条件的输出时特别有效。例如让我们测试一个特定的命令是否存在。
>
	bash$ type bogus_command &>/dev/null
>
>
>
	bash$ echo $?
	1
> 或者写在脚本中：
>
```bash
command_test () { type "$1" &>/dev/null; }
#                                      ^
> 
cmd=rmdir            # 存在的命令。
command_test $cmd; echo $?   # 0
>
>
cmd=bogus_command    # 不存在的命令。
command_test $cmd; echo $?   # 1
```

`command >&2` 将命令的标准输出重定向至错误输出。

`scriptname >>filename` 将脚本 *scriptname* 的输出添加到 *filename* 中。如果文件不存在，那么创建这个文件。

`[i]<>filename` 打开文件 *filename* 用来读写，并且分配一个文件描述符 *i*。如果文件不存在，那么创建这个文件。

进程替换：

	(command)>
	<(command)

在某些情况下， "<" 与 ">" 将用作字符串比较。

在另外一些情况下， "<" 与 ">" 将用作数字比较。详情查看样例 16-9。

### <<

在here document中进行重定向。

### <<<

在here string中进行重定向。

### <, >

ASCII码比较。

```bash
veg1=carrots
veg2=tomatoes

if [[ "veg1" < "veg2" ]]
then
  echo "Although $veg1 precede $veg2 in the dictionary,"
  echo -n "this does not necessarily imply anything "
  echo "about my culinary preferences."
else
  echo "What kind of dictionary are you using, anyhow?"
fi
```

### \<, \>

正则表达式中的单词边界（word boundary）。

	bash$ grep '\<the\>' textfile

### |

管道（pipe）。管道可以将上一个命令的输出作为下一个命令的输入，或者直接输出到shell中。它是一种将一系列命令链接在一起的方法。

```bash
echo ls -l | sh
#  将 "echo ls -l" 的结果输出到shell中，
#+ 与直接输入 "ls -l" 的结果相同。


cat *.lst | sort | uniq
# 将所有后缀名为 lst 的文件合并后排序，接着删掉所有重复的行。
```

> 管道是一种在进程间通信的典型方法。它将一个进程的输出作为另一个进程的输入。举一个经典的例子，像 `cat` 或者 `echo` 这样的命令，可以通过管道将它们产生的数据流导入到过滤器（filter）中。过滤器是用来处理输入流的命令。
>
> `cat $filename1 $filename2 | grep $search_word`
> 
> 查看[UNIX FAQ第三章](http://www.faqs.org/faqs/unix-faq/faq/part3/)获得更多关于使用UNIX管道的信息。

命令的输出同样可以通过管道输入到脚本中。

```bash
#!/bin/bash
# uppercase.sh : 将所有输入变成大写

tr 'a-z' 'A-Z'
#  为了防止产生单个字符的文件名，
#+ 必须使用' '引用字符范围。

exit 0
```

现在，让我们将 `ls -l` 的输出通过管道导入到脚本中。

	bash$ ls -l | ./uppercase.sh
	-RW-RW-R--    1 BOZO  BOZO       109 APR  7 19:49 1.TXT
	-RW-RW-R--    1 BOZO  BOZO       109 APR 14 16:48 2.TXT
	-RW-R--R--    1 BOZO  BOZO       725 APR 20 20:56 DATA-FILE

> ![extra](http://tldp.org/LDP/abs/images/note.gif) 在管道中每一个进程的输入必须作为下一个进程的输入被读入，如果不是这样，那么数据流将会被阻塞（block），管道也将不会按照预期的那样工作。
> 
```bash
cat file1 file2 | ls -l | sort
# "cat file1 file2" 的输出将会消失。
```
> 管道是在一个子进程中运行的，因此它并不能修改脚本中的变量。
> 
```bash
variable="initial_value"
echo "new_value" | read variable
echo "variable = $variable"     # variable = initial_value
```

如果管道中的任意一个命令意外中止了，管道将会提前中断，我们称其为破损管道。在这种情况下，将会发送一个 `SIGPIPE` 信号。

### >|

强制重定向。即使在 `noclobber` 选项被设置的情况下，重定向也会覆盖已经存在的文件。

### ||

或（OR）逻辑运算符。在测试结构中，如果任意一个测试条件为真，则返回 0（成功标志位）。

### &

后台运行操作符。如果命令后有 &，将会使这个命令转至后台运行。

	bash$ sleep 10 &
	[1] 850
	[1]+  Done                    sleep 10

在脚本中，命令甚至循环都可以在后台运行。

样例 3-3. 在后台运行的循环

```bash
#!/bin/bash
# background-loop.sh

for i in 1 2 3 4 5 6 7 8 9 10            # 第一个循环
do
  echo -n "$i "
done & # 使这个循环在后台运行。
       # 有是会在第二个循环结束之后才运行。

echo   # 一些情况下 'echo' 将不会显示出来。

for i in 11 12 13 14 15 16 17 18 19 20   # 第二个循环
do
  echo -n "$i "
done

echo   # 一些情况下 'echo' 将不会显示出来。

# ======================================================

# 脚本期望的输出结果：
# 1 2 3 4 5 6 7 8 9 10
# 11 12 13 14 15 16 17 18 19 20

# 一些情况下可能会输出：
# 11 12 13 14 15 16 17 18 19 20
# 1 2 3 4 5 6 7 8 9 10 bozo $
# 第二个 'echo' 没有被执行，为什么？

# 另外一些情况下可能会输出：
# 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20
# 第一个 'echo' 没有被执行，为什么？

# 非常罕见的情况下，可能会输出：
# 11 12 13 1 2 3 4 5 6 7 8 9 10 14 15 16 17 18 19 20
# 前台的循环抢占（preempt）了后台的循环。

exit 0

#  Nasimuddin Ansari 建议在第6行和第14行的
#+ echo -n "$i " 后增加 sleep 1，
#+ 会得到许多有趣的结果。
```

> ![notice](http://tldp.org/LDP/abs/images/caution.gif) 脚本在后台执行命令可能会导致脚本等待键盘事件而被挂起。幸运的是，有一套解决方案可以解决这个问题。

### &&

与（AND）逻辑操作符。在测试结构中，如果所有测试条件都为真，才返回 0（成功标志位）。

### -

选项与前缀。它可以作为命令的选项标志，也可以作为一个操作符的前缀，也可以作为在参数代换中作为缺省参数的前缀。

`COMMAND -[Option1][Option2][..]`

`ls -al`

`sort -dfu $filename`

```bash
if [ $file1 -ot $file2 ]
then #      ^
  echo "File $file1 is older than $file2."
fi

if [ "$a" -eq "$b" ]
then #    ^
  echo "$a is equal to $b."
fi

if [ "$c" -eq 24 -a "$d" -eq 47 ]
then #    ^              ^
  echo "$c equals 24 and $d equals 47."
fi


param2=${param1:-$DEFAULTVAL}
#               ^
```

### --

双横线是作为命令的逐字长选项的前缀。

`sort --ignore-leading-blanks`

双横线与Bash内建命令一起使用时，意味着该命令选项的结束。

> ![info](http://tldp.org/LDP/abs/images/tip.gif) 下面提供了一种删除文件名以横线开头的文件的简单方法。
> 
	bash$ ls -l
	-rw-r--r-- 1 bozo bozo 0 Nov 25 12:29 -badname
>
>
	bash$ rm -- -badname
>
	bash$ ls -l
	total 0

双横线通常也和 `set` 连用。

`set -- $variable`（查看样例 15-18）。

### -

重定向输入输出[短横线]。

	bash$ cat -
	abc
	abc

	...

	Ctl-D

在这个例子中，`cat -` 输出由键盘读入的输入 *stdin* 到 *stdout*。但是在真实应用的 I/O 重定向中是否有使用 '-'？





[^1]: 操作符（operator）用来执行表达式（operation）。最常见的例子就是算术运算符+ - * /。在Bash中，操作符和关键字的概念有一些重叠。