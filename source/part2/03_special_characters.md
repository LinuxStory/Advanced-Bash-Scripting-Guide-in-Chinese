# 第三章 特殊字符

是什么让一个字符变得*特殊*呢？如果一个字符不仅具有*字面*意义，而且具有*元意（meta-meaning）*，我们就称它为特殊字符。特殊字符同命令和关键词（keywords）一样，是bash脚本的组成部分。

你在脚本或其他地方都能够找到特殊字符。

### \# ###

注释符。如果一行脚本的开头是#（除了#!），那么代表这一行是注释，不会被执行。

```bash
# 这是一行注释
```
    
注释也可能会在一行命令结束之后出现。

```bash
echo "A comment will follow." # 这儿可以写注释
#                            ^ 注意在#之前有空格
```

注释也可以出现在一行开头的空白符（whitespace）之后。

```bash
	# 这个注释前面存在着一个制表符（tab）
```
	    
注释甚至可以嵌入到管道命令（pipe）之中。

```bash
initial=( `cat "$startfile" | sed -e '/#/d' | tr -d '\n' |\
# 删除所有带'#'注释符号的行
           sed -e 's/\./\. /g' -e 's/_/_ /g'` )
# 摘录自脚本 life.sh
```

> ![notice](http://tldp.org/LDP/abs/images/caution.gif) 命令不能写在同一行注释之后。因为没有任何方法可以结束注释(仅支持单行注释)，为了让新命令正常执行，另起一行写吧。

> ![extra](http://tldp.org/LDP/abs/images/note.gif) 当然，在`echo`语句中被引用或被转义的#不会被认为是注释。同样，在某些参数替换式或常量表达式中的#也不会被认为是注释。

```bash
echo "The # here does not begin a comment."
echo 'The # here does not begin a comment.'
echo The \# here does not begin a comment.
echo The # here begins a comment.

echo ${PATH#*:}       # 参数替换而非注释
echo $(( 2#101011 ))  # 进制转换而非注释

# 感谢S.C.
```
	
因为引用符和转义符（" ' \）转义了#。

一些模式匹配操作同样使用了#。

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
	
注意有时候";"需要被转义才能正常工作。

### ;;

`case`条件语句终止符[双分号]。

```bash
case "$variable" in
  abc)  echo "\$variable = abc" ;;
  xyz)  echo "\$variable = xyz" ;;
esac
```
	
### ;;&, ;&

`case`条件语句终止符（Bash4+ 版本）。

### .

句点命令[句点]。等价于`source`命令（查看样例 15-22）。这是一个bash的内建命令。

### .

句点可以作为文件名的一部分。如果它在文件名开头，那说明此文件是隐藏文件。使用不带参数的`ls`命令不会显示隐藏文件。

```
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
``` 
当句点出现在目录中时，单个句点代表当前工作目录，两个句点代表上级目录。

```
bash$ pwd
/home/bozo/projects

bash$ cd .
bash$ pwd
/home/bozo/projects

bash$ cd ..
bash$ pwd
/home/bozo/
```
	
句点通常代表文件移动的目的地（目录），下式代表的是当前目录。

```
bash$ cp /home/bozo/current_work/junk/* .
``` 
	
> 复制所有的“垃圾文件”到`当前目录`

### .

句点匹配符。在*正则表达式*中，点号意味着匹配任意单个字符。

### "

部分引用[双引号]。在字符串中保留大部分特殊字符。详细内容将在[第五章](05_quoting.md)介绍。

### '

全引用[单引号]。在字符串中保留所有的特殊字符。是部分引用的强化版。详细内容将在[第五章](05_quoting.md)介绍。

### ,

逗号运算符。逗号运算符[^1]将一系列的算术表达式串联在一起。算术表达式依次被执行，但只返回最后一个表达式的值。

```bash
let "t2 = ((a = 9, 15 / 3))"
# a被赋值为9，t2被赋值为15 / 3
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
# /usr/bin/oocalc

# 感谢Rory Winston提供的执行结果
```

### ,, ,

在参数替换中进行小写字母转换（Bash4 新增）。

### \

转义符[反斜杠]。转义某字符的标志。

`\x`转义了字符x。双引号""内的X与单引号内的X具有同样的效果。
转义符也可以用来转义"与'，使它们表达其字面含义。

第五章将更加深入的解释转义字符。

### /

文件路径分隔符[正斜杠]。起分割路径的作用。（比如 `/home/bozo/projects/Makefile`）

它也在算术运算中充当除法运算符。

### `

命令替换符。`` `command` ``结构可以使得命令的输出结果赋值给一个变量。通常也被称作后引号（backquotes）或反引号（backticks）。

### :

空命令[冒号]。它在shell中等价于"NOP"（即no op，空操作）与shell内建命令true有同样的效果。它本身也是Bash的内建命令之一，返回值是true（0）。

```bash
:
echo $?   # 返回0
```

在无限循环中的应用：

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

可在 `if/then` 中充当占位符：

```bash
if condition
then :   # 什么都不做，跳出判断执行下一条语句
else
   take-some-action
fi
```

在二元操作中作占位符: 查看*样例 8-2*或*默认参数*部分。

```bash
: ${username=`whoami`}
# ${username=`whoami`}   如果没有:就会报错
#                        除非 "username" 是系统命令或内建命令

: ${1?"Usage: $0 ARGUMENT"}     # 摘自样例脚本 "usage-message.sh"
```

查看*样例 19-10*了解空命令在here document中作为占位符的情况。

使用参数替换为字符串变量赋值（查看*样例 10-7*）。

```bash
: ${HOSTNAME?} ${USER?} ${MAIL?}
#  如果其中一个或多个必要的环境变量没有被设置
#  将会打印错误
```

查看*变量扩展*或*字符串替换*章节了解空命令在其中的作用。

与`>`重定向操作符结合，可以在不改变文件权限的情况下清空文件。如果文件不存在，那么将创建这个文件。

```bash
: > data.xxx   # 文件 "data.xxx" 已被清空

# 与 cat /dev/null >data.xxx 作用相同
# 但是此操作不会产生一个新进程，因为 ":" 是shell内建命令。
```
也可查看*样例 16-15*。

与`>>`重定向操作符结合，将不会清空任何已存在的文件（`: >> target_file`）。如果文件不存在，将创建这个文件。

> ![extra](http://tldp.org/LDP/abs/images/note.gif) 以上操作仅适用于普通文件，不适用于管道、符号链接和特殊文件。

空命令可以用来作为一行注释的开头，尽管我们并不推荐这么做。使用 # 可以使解释器关闭该行的错误检测，所以几乎所有的内容都可以出现在注释#中。使用空命令却不是这样的：

```bash
: 这一行注释将会产生一个错误，( if [ $x -eq 3] )。
```

:也可以作为一个域分隔符，比如在`/etc/passwd`和 `$PATH` 变量中。
```
bash$ echo $PATH
/usr/local/bin:/bin:/usr/bin:/usr/X11R6/bin:/sbin:/usr/sbin:/usr/games
```

将冒号作为函数名也是可以的。

```bash
:()
{
  echo "The name of this function is "$FUNCNAME" "
  # 为什么要使用冒号作函数名？
  # 这是一种混淆代码的方法......
}

:

# 函数名是 :
```
这种写法并不具有可移植性，也不推荐使用。事实上，在Bash的最近的版本更新中已经禁用了这种用法。但我们还可以使用*下划线 _*来替代。

冒号也可以作为非空函数的占位符。

```bash
not_empty ()
{
  :
} # 含有空指令，这并不是一个空函数。
```
	
### !

取反（或否定）操作符[感叹号]。! 操作符反转已执行的命令的返回状态（查看*样例 6-2*）。它同时可以反转测试操作符的意义，例如可以将相等（=）反转成不等（!=）。它是一个Bash关键词。

在一些特殊场景下，它也会出现在间接变量引用中。

在另外一些特殊场景下，即在命令行下可以使用 ! 调用Bash的历史记录（*附录 L*）。需要注意的是，在脚本中，这个机制是被禁用的。

### *

通配符[星号]。在文件匹配（globbing）操作时扩展文件名。如果它独立出现，则匹配该目录下的所有文件。

```
bash$ echo *
abs-book.sgml add-drive.sh agram.sh alias.sh
```

在*正则表达式*中表示匹配任意多个（包括0）前个字符。

### *

算术运算符。在进行算术运算时，表示乘法运算。

** 双星号可以表示乘方运算或扩展文件匹配。

### ?

测试操作符[问号]。在一些特定的语句中，? 表示一个条件测试。

在一个双圆括号结构中，? 可以表示一个类似C语言风格的三元（trinary）运算符的一个组成部分。[^2]

`condition?result-if-true:result-if-false`

```bash
(( var0 = var1<98?9:21 ))
#不要加空格，紧挨着写

#等价于
# if [ "$var1" -lt 98 ]
# then
#   var0=9
# else
#   var0=21
# fi
```

在参数替换表达式中，? 用来测试一个变量是否已经被赋值。

### ?

通配符。它在进行文件匹配（globbing）时以单字符通配符扩展文件名。
在*扩展正则表达式*中匹配一个单字符。

### $
取值符号[钱字符]，用来进行变量替换（即取出变量的内容）。

```bash
var1=5
var2=23skidoo

echo $var1     # 5
echo $var2     # 23skidoo
```

如果在变量名前有 $，则表示此变量的值。

### $

行结束符[EOF]。
在*正则表达式*中，$ 匹配行尾字符串。

### ${}

参数替换。

### $'...'

引用字符串扩展。这个结构将转义八进制或十六进制的值转换成ASCII[^3]或Unicode字符。

### $*, $@ ###

位置参数。

### $?

返回状态变量。此变量保存一个命令、一个函数或该脚本自身的返回状态。

### $$

进程ID变量。此变量保存该运行脚本的进程ID[^4]。

### ()

命令组。

`(a=hello; echo $a)`

![notice](http://tldp.org/LDP/abs/images/important.gif) 通过括号执行一系列命令会产生一个子shell（subshell）。
括号中的变量，即在子shell中的变量，在脚本的其他部分是不可见的。父进程脚本不能访问子进程（子shell）所创建的变量。

```bash
a=123
( a=321; )

echo "a = $a"   # a = 123
# 在括号中的 "a" 就像个局部变量。
```
	
数组初始化。

```bash
Array=(element1 element2 element3)
``` 

### {xxx,yyy,zzz,...}

花括号扩展结构。

```bash
echo \"{These,words,are,quoted}\"   # " 将作为单词的前缀和后缀
# "These" "words" "are" "quoted"


cat {file1,file2,file3} > combined_file
# 将 file1, file2 与 file3 拼接在一起后写入 combined_file 中。

cp file22.{txt,backup}
# 将 "file22.txt" 拷贝为 "file22.backup"
```

这个命令可以作用于花括号内由逗号分隔的文件描述列表。[^5] 文件名扩展（匹配）作用于大括号间的各个文件。

![notice](http://tldp.org/LDP/abs/images/caution.gif) 除非被引用或被转义，否则空白符不应在花括号中出现。

```bash
echo {file1,file2}\ :{\ A," B",' C'}
file1 : A file1 : B file1 : C file2 : A file2 : B file2 : C
```

### {a..z}

扩展的花括号扩展结构。

```bash
echo {a..z} #  a b c d e f g h i j k l m n o p q r s t u v w x y z
# 输出 a 到 z 之间所有的字母。
	
echo {0..3} # 0 1 2 3
# 输出 0 到 3 之间所有的数字。


base64_charset=( {A..Z} {a..z} {0..9} + / = )
# 使用扩展花括号初始化一个数组。
# 摘自 vladz 编写的样例脚本 "base64.sh"。
```

Bash第三版中引入了 {a..z} 扩展的花括号扩展结构。

### {}

代码块[花括号]，又被称作内联组（inline group）。它实际上创建了一个匿名函数（anonymous function），即没有名字的函数。但是，不同于那些“标准”函数，代码块内的变量在脚本的其他部分仍旧是可见的。
```
bash$ { local a;
              a=123; }
bash: local: can only be used in a
function
```

```bash
a=123
{ a=321; }
echo "a = $a"   # a = 321   (代码块内赋值)

# 感谢S.C.
```

代码块可以经由I/O重定向进行输入或输出。

**样例 3-1. 代码块及I/O重定向**

```bash
#!/bin/bash
# 读取文件 /etc/fstab

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

# 你知道如何解析剩下的行吗？
# 提示：使用 awk 或...
# Hans-Joerg Diers 建议：使用Bash的内建命令 set。
```

**样例 3-2. 将代码块的输出保存至文件中**

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

# rpm各项参数的具体含义可查看man文档

exit 0
```
![extra](http://tldp.org/LDP/abs/images/note.gif) 与由圆括号包裹起来的命令组不同，由花括号包裹起来的代码块不产生子进程。[^6]
 
也可以使用非标准的 for 循环语句来遍历代码块。

### {}

文本占位符。在 `xargs -i` 后作为输出的占位符使用。

```bash
ls . | xargs -i -t cp ./{} $1
#            ^^         ^^

# 摘自 "ex42.sh" (copydir.sh)
```

### {} \;

路径名。通常在 `find` 命令中使用，但这不是shell的内建命令。

> 定义：路径名是包含完整路径的文件名，例如`/home/bozo/Notes/Thursday/schedule.txt`。我们通常又称之为绝对路径。

![extra](http://tldp.org/LDP/abs/images/note.gif) 在执行`find -exec`时最后需要加上`;`，但是分号需要被转义以保证其不会被shell解释。

### [ ]

测试。在 [ ] 之间填写测试表达式。值得注意的是，[ 是shell内建命令 `test` 的一个组成部分，而不是外部命令 `/usr/bin/test` 的链接。

### [[ ]]

测试。在 [[ ]] 之间填写测试表达式。相比起单括号测试 （[ ]），它更加的灵活。它是一个shell的关键字。

详情查看*关于 [[ ]] 结构的讨论*。

### [ ]

数组元素。在数组中，可以使用中括号的偏移量来用来访问数组中的每一个元素。

```bash
Array[1]=slot_1
echo ${Array[1]}
```

### [ ]

字符集、字符范围。
在*正则表达式*中，中括号用来匹配指定字符集或字符范围内的任意字符。

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

详情查看*关于 (( ... )) 结构的讨论*。

### > &> >& >> < <>

重定向。

`scriptname >filename` 将脚本 *scriptname* 的输出重定向到 *filename* 中。如果文件存在，那么覆盖掉文件内容。

`command &>filename` 将命令 *command* 的标准输出(stdout) 和标准错误输出(stderr) 重定向到 *filename*。

![extra](http://tldp.org/LDP/abs/images/note.gif) 重定向在用于清除测试条件的输出时特别有效。例如测试一个特定的命令是否存在。

```
bash$ type bogus_command &>/dev/null


bash$ echo $?
1
```

或写在脚本中：

```bash
command_test () { type "$1" &>/dev/null; }
#                                      ^
 
cmd=rmdir            # 存在的命令。
command_test $cmd; echo $?   # 返回0


cmd=bogus_command    # 不存在的命令。
command_test $cmd; echo $?   # 返回1
```


`command >&2` 将命令的标准输出重定向至标准错误输出。

`scriptname >>filename` 将脚本 *scriptname* 的输出追加到 *filename* 文件末尾。如果文件不存在，那么将创建这个文件。

`[i]<>filename` 打开文件 *filename* 用来读写，并且分配一个文件描述符*i*指向它。如果文件不存在，那么将创建这个文件。

进程替换：
`(command)>`
`<(command)`

在某些情况下， "<" 与 ">" 将用作字符串比较。

在另外一些情况下， "<" 与 ">" 将用作数字比较。详情查看*样例 16-9*。

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

*正则表达式*中的单词边界（word boundary）。
```
bash$ grep '\<the\>' textfile
```

### |

管道（pipe）。管道可以将上一个命令的输出作为下一个命令的输入，或者直接输出到shell中。管道是一种可以将一系列命令连接在一起的绝妙方式。

```bash
echo ls -l | sh
#  将 "echo ls -l" 的结果输出到shell中，
#  与直接输入 "ls -l" 的结果相同。


cat *.lst | sort | uniq
# 将所有后缀名为 lst 的文件合并后排序，接着删掉所有重复行。
```

> 管道是一种在进程间通信的典型方法。它将一个进程的输出作为另一个进程的输入。举一个经典的例子，像 `cat` 或者 `echo` 这样的命令，可以通过管道将它们产生的数据流导入到过滤器（filter）中。过滤器是可以用来处理输入流的命令。[^7]
>
> `cat $filename1 $filename2 | grep $search_word`
> 
> 查看[UNIX FAQ第三章](http://www.faqs.org/faqs/unix-faq/faq/part3/)获取更多关于使用UNIX管道的信息。

命令的输出同样可以通过管道输入到脚本中。

```bash
#!/bin/bash
# uppercase.sh : 将所有输入变成大写

tr 'a-z' 'A-Z'
#  为了防止产生单字符文件名，
#  必须使用单引号引用字符范围。

exit 0
```

现在，让我们将 `ls -l` 的输出通过管道导入到脚本中。

```
bash$ ls -l | ./uppercase.sh
 -RW-RW-R--    1 BOZO  BOZO       109 APR  7 19:49 1.TXT
 -RW-RW-R--    1 BOZO  BOZO       109 APR 14 16:48 2.TXT
 -RW-R--R--    1 BOZO  BOZO       725 APR 20 20:56 DATA-FILE
```

![extra](http://tldp.org/LDP/abs/images/note.gif) 在管道中，每一个进程的输出必须作为下个进程的输入被正确读入，如果不这样，数据流会被阻塞（block），管道就无法按照预期正常工作。

```bash
cat file1 file2 | ls -l | sort
# "cat file1 file2" 的输出会消失。
```

管道是在一个子进程中运行的，因此它并不能修改父进程脚本中的变量。
 
```bash
variable="initial_value"
echo "new_value" | read variable
echo "variable = $variable"     # variable = initial_value
```
如果管道中的任意一个命令意外中止了，管道将会提前中断，我们称其为*管道破裂*(Broken Pipe)。出现这种情况，系统将发送一个 `SIGPIPE` 信号。

### >|

强制重定向。即使在 `noclobber` 选项被设置的情况下，重定向也会覆盖已存在的文件。

### ||

或（OR）逻辑运算符。在测试结构中，任意一个测试条件为真，整个表达式为真。返回 0（成功标志位）。

### &

后台运行操作符。如果命令后带&，那么此命令将转至后台运行。

```
bash$ sleep 10 &
[1] 850
[1]+  Done                    sleep 10
```

在脚本中，命令甚至循环都可以在后台运行。

**样例 3-3. 在后台运行的循环**

```bash
#!/bin/bash
# background-loop.sh

for i in 1 2 3 4 5 6 7 8 9 10            # 第一个循环
do
  echo -n "$i "
done & # 这个循环在后台运行。
       # 有时会在第二个循环结之后才执行此后台循环。

echo   # 此'echo' 有时不显示

for i in 11 12 13 14 15 16 17 18 19 20   # 第二个循环
do
  echo -n "$i "
done

echo   # 此'echo' 有时不显示

# ======================================================

# 脚本期望输出结果：
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
# 前台循环抢占（preempt）了后台循环。

exit 0

#  Nasimuddin Ansari 建议：在第6行和第14行的
#  echo -n "$i " 后增加 sleep 1，
#  会得到许多有趣的结果。
```

![notice](http://tldp.org/LDP/abs/images/caution.gif) 脚本在后台执行命令时可能因为等待键盘事件被挂起。幸运的是，有一套方案可以解决这个问题。

### &&

与（AND）逻辑操作符。在测试结构中，所有测试条件都为真，表达式才为真，返回 0（成功标志位）。

### -

选项与前缀。它可以作为命令的选项标志，也可以作为一个操作符的前缀，也可以作为在参数代换中作为默认参数的前缀。

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

双横线一般作为命令长选项的前缀。

`sort --ignore-leading-blanks`

双横线与Bash内建命令一起使用时，意味着该命令选项的结束。

![info](http://tldp.org/LDP/abs/images/tip.gif) 下面提供了一种删除文件名以横线开头文件的简单方法。
 
```
bash$ ls -l
-rw-r--r-- 1 bozo bozo 0 Nov 25 12:29 -badname


bash$ rm -- -badname

bash$ ls -l
total 0
```
双横线通常也和 `set` 连用。

`set -- $variable`（查看*样例 15-18*）。

### -

重定向输入输出[短横线]。

```
bash$ cat -
abc
abc

...

Ctl-D
```

在这个例子中，`cat -` 输出由键盘读入的标准输入(stdin) 到 标准输出(stdout)。但是在真实应用的 I/O 重定向中是否有使用 '-'？

```bash
(cd /source/directory && tar cf - . ) | (cd /dest/directory && tar xpvf -)

# 将整个文件树从一个目录移动到另一个目录。
# 感谢 Alan Cox <a.cox@swansea.ac.uk> 所作出的部分改动

# 1) cd /source/directory
#    工作目录定位到文件所属的源目录
# 2) &&
#    "与链"：如果 'cd' 命令操作成功，那么执行下一条命令
# 3) tar cf - .
#    'tar c' (create 创建) 创建一份新的档案
#    'tar f -' (file 指定文件) 在 '-' 后指定一个目标文件作为输出
#    '.' 代表当前目录
# 4) |
#    通过管道进行重定向
# 5) ( ... )
#    在建立的子进程中执行命令
# 6) cd /dest/directory
#    工作目录定位到目标目录
# 7) &&
#    与 2) 相同
# 8) tar xpvf -
#    'tar x' 解压档案
#    'tar p' (preserve 保留) 保留档案内文件的所有权及文件权限
#    'tar v' (verbose 冗余) 发送全部信息到标准输出
#    'tar f -' (file 指定文件) 在 '-' 后指定一个目标文件作为输入
#
#    注意 'x' 是一个命令，而 'p', 'v', 'f' 是选项。

# 干的漂亮！


# 更加优雅的写法是:
#   cd source/directory
#   tar cf - . | (cd ../dest/directory; tar xpvf -)
#
# 同样可以写成:
#   cp -a /source/directory/* /dest/directory
# 或:
#   cp -a /source/directory/* /source/directory/.[^.]* /dest/directory
# 可以在源目录中有隐藏文件时使用
```

```bash
bunzip2 -c linux-2.6.16.tar.bz2 | tar xvf -
#  --未解压的 tar 文件--        | --将解压出的 tar 传递给 "tar"--
#  如果不使用管道让 "tar" 处理 "bunzip2" 得到的文件，
#  那么就需要使用单独的两步来完成。
#  目的是为了解压 "bzipped" 压缩的内核源代码。
```

下面的例子中，"-" 并不是一个Bash的操作符，它仅仅是 `tar`, `cat` 等一些特定UNIX命令中将结果输出到标准输出的选项。

```
bash$ echo "whatever" | cat -
whatever 
```

当需要文件名的时候，- 可以用来代替某个文件而重定向到标准输出（通常出现在 `tar cf` 中）或从 *stdin* 中接受数据。这是一种在管道中使用面向文件（file-oriented）工具作为过滤器的方法。

```
bash$ file
Usage: file [-bciknvzL] [-f namefile] [-m magicfiles] file...
```

单独执行 `file` 命令，将会得到一条错误信息。

在命令后增加一个 "-" 可以得到一个更加有用的结果。它会使得shell暂停等待用户输入。

```
bash$ file -
abc
standard input:              ASCII text


bash$ file -
#!/bin/bash
standard input:              Bourne-Again shell script text executable
```
	
现在命令能够接受标准输入并且处理它们了。

"-" 能够通过管道将标准输出重定向到其他命令中。这就可以做到像在某个文件前添加几行这样的事情。

使用 `diff` 比较两个文件的部分内容：

```bash
grep Linux file1 | diff file2 -
```

最后介绍一个使用 - 的 `tar` 命令的实际案例。

**样例 3-4. 备份最近一天修改过的所有文件**

```bash
#!/bin/bash

#  将当前目录下24小时之内修改过的所有文件备份成一个
#  "tarball" (经 tar 打包`与 gzip 压缩) 文件

BACKUPFILE=backup-$(date +%m-%d-%Y)
#                 在备份文件中嵌入时间
#                 感谢 Joshua Tschida 提供的建议

archive=${1:-$BACKUPFILE}
#  如果没有在命令行中特别制定备份格式，
#  那么将会默认设置为 "backup-MM-DD-YYYY.tar.gz"。

tar cvf - `find . -mtime -1 -type f -print` > $archive.tar
gzip $archive.tar
echo "Directory $PWD backed up in archive file \"$archive.tar.gz\"."

#  Stephane Chazeles 指出如果目录中有非常多的文件，
#  或文件名中包含空白符时，上面的代码会运行失败。

# 他建议使用以下的任意一种方法：
# -------------------------------------------------------------------
#   find . -mtime -1 -type f -print0 | xargs -0 tar rvf "$archive.tar"
#   使用了 GNU 版本的 "find" 命令。


#   find . -mtime -1 -type f -exec tar rvf "$archive.tar" '{}' \;
#   兼容其他的 UNIX 发行版，但是速度会比较慢
# -------------------------------------------------------------------


exit 0
```

![notice](http://tldp.org/LDP/abs/images/caution.gif) 以 "-" 开头的文件在和"-" 重定向操作符一起使用时可能会导致一些问题。因此合格的脚本必须首先检查这种情况。如果遇到，就需要给文件名加一个合适的前缀，比如 `./-FILENAME, $PWD/-FILENAME` 或者`$PATHNAME/-FILENAME` 。

如果变量的值以 '-' 开头，也可能会造成类似问题。
 
```bash
var='-n'
echo $var
# 等同于 "echo -n"，不会输出任何东西。
```

### -

先前的工作目录。使用 `cd -` 命令可以返回先前的工作目录。它实际上是使用了 `$OLDPWD` 环境变量。

 ![notice](http://tldp.org/LDP/abs/images/caution.gif) 不要将这里的 "-" 与先前的 "-" 重定位操作符混淆。"-" 的具体含义需要根据上下文来解释。

### -

减号。算术运算符中的减法标志。

### =

等号。赋值操作符。

```bash
a=28
echo $a   # 28
```

在一些情况下，"=" 可以作为字符串比较操作符。

### +

加号。加法算术运算。

在一些情况下，+ 是作为正则表达式中的一个操作符。

### +

选项操作符。作为一个命令或过滤器的选项标记。

特定的一些指令和内建命令使用 + 启用特定的选项，使用 - 禁用特定的选项。在参数代换中，+ 是作为变量扩展的备用值（alternate value）的前缀。

### %

取模。取模操作运算符。

```bash
let "z = 5 % 3"
echo $z  # 2
```

在另外一些情况下，% 是一种模式匹配的操作符。

### ~

主目录[波浪号]。它相当于内部变量 `$HOME`。`~bozo` 是 bozo 的主目录，执行 `ls ~bozo` 将会列出他的主目录中内容。`~/` 是当前用户的主目录，执行 `ls ~/` 将会列出其中所有的内容。
```
bash$ echo ~bozo
/home/bozo

bash$ echo ~
/home/bozo

bash$ echo ~/
/home/bozo/

bash$ echo ~:
/home/bozo:

bash$ echo ~nonexistent-user
~nonexistent-user
```
	
### ~+

当前工作目录。它等同于内部变量 `$PWD`。

### ~-

先前的工作目录。它等同于内部变量 `$OLDPWD`。

### =~

*正则表达式*匹配。将在 *Bash version 3 *章节中介绍。

### ^

行起始符。在正则表达式中，"^" 代表一行文本的开始。

### ^, ^^

参数替换中的大写转换符（在Bash第4版新增）。

### 控制字符

改变终端或文件显示的一些行为。一个控制符是由 *CONTRL + key* 组成的（同时按下）。控制字符同样可以通过转义以八进制或十六进制的方式显示。

控制符不能在脚本中使用。

#### Ctrl-A

移动光标至行首。

#### Ctrl-B

非破坏性退格（即不删除字符）。

#### Ctrl-C

中断指令。终止当前运行的任务。

#### Ctrl-D

登出shell（类似 `exit`）

键入 `EOF`（end-of-file，文件终止标记），中断 *stdin* 的输入。

当你在终端或 *xterm* 窗口中输入字符时，`Ctl-D` 将会删除光标上的字符。当没有字符时，`Crl-D` 将会登出shell。在 *xterm* 中，将会关闭整个窗口。

#### Ctrl-E

移动光标至行末。

#### Ctrl-F

光标向前移动一个字符。

#### Ctrl-G

响铃`BEL`。在一些老式打字机终端上，将会响铃。而在 *xterm* 中，将会产生“哔”声。

#### Ctrl-H

抹除（破坏性退格）。退格删除前面的字符。

```bash
#!/bin/bash
# 在字符串中嵌入 Ctrl-H

a="^H^H"                  # 两个退格符 Ctrl-H
                          # 在 vi/vim 中使用 Ctrl-V Ctrl-H 来键入
echo "abcdef"             # abcdef
echo
echo -n "abcdef$a "       # abcd f
#                ^              ^ 末尾有空格退格两次的结果
echo
echo -n "abcdef$a"        # abcdef
#                                ^ 末尾没有空格时为什么退格无效了？
                          # 并不是我们期望的结果。
echo; echo

# Constantin Hagemeier 建议尝试一下：
# a=$'\010\010'
# a=$'\b\b'
# a=$'\x08\x08'
# 但是这些并不会改变结果。

########################################

# 现在来试试这个。

rubout="^H^H^H^H^H"       # 5个 Ctrl-H

echo -n "12345678"
sleep 2
echo -n "$rubout"
sleep 2
```

#### Ctrl-I

水平制表符。

#### Ctrl-J

另起一行（换行）。在脚本中，你也可使用八进制 '\012' 或者十六进制 '\x0a' 来表示。

#### Ctrl-K

垂直制表符。

当你在终端或 *xterm* 窗口中输入字符时，`Ctrl-K` 将会删除光标上及其后的所有字符。而在脚本中，`Ctrl-K` 的作用有些不同。具体查看下方 Lee Lee Maschmeyer 写的样例。

#### Ctrl-L

清屏、走纸。在终端中等同于 `clear` 命令。在打印时，`Ctrl-L` 将会使纸张移动到底部。

#### Ctrl-M

回车（CR）。

```bash
#!/bin/bash
# 感谢 Lee Maschmeyer 提供的样例。

read -n 1 -s -p \
$'Control-M leaves cursor at beginning of this line. Press Enter. \x0d'
           # '0d' 是 Control-M 的十六进制的值
echo >&2   # '-s' 参数禁用了回显，所以需要显式的另起一行。

read -n 1 -s -p $'Control-J leaves cursor on next line. \x0a'
           # '0a' 是 Control-J 换行符的十六进制的值
echo >&2

###

read -n 1 -s -p $'And Control-K\x0bgoes straight down.'
echo >&2   # Control-K 是垂直制表符。

# 一个更好的垂直制表符例子是：

var=$'\x0aThis is the bottom line\x0bThis is the top line\x0a'
echo "$var"
#  这将会产生与上面的例子类似的结果。但是
echo "$var" | col
#  这却会使得右侧行高于左侧行。
#  这也解释了为什么我们需要在行首和行尾加上换行符
#  来避免显示的混乱。

# Lee Maschmeyer 的解释：
# --------------------------
#  在第一个垂直制表符的例子中，垂直制表符使其
#  在没有回车的情况下向下打印。
#  这在那些不能回退的设备上，例如 Linux 的终端才可以。
#  而垂直制表符的真正目的是向上而非向下。
#  它可以用来在打印机中用来打印上标。
#  col 工具可以用来模拟真实的垂直制表符行为。

exit 0
```

#### Ctrl-N

在命令行历史记录中调用下一条历史命令[^8]。

#### Ctrl-O

在命令行中另起一行。

#### Ctrl-P

在命令行历史记录中调用上一条历史命令。

#### Ctrl-Q

恢复（XON）。

终端恢复读入 *stdin*。

#### Ctrl-R

在命令行历史记录中进行搜索。

#### Ctrl-S

挂起（XOFF）。

终端冻结 *stdin*。（可以使用 `Ctrl-Q` 恢复）

#### Ctrl-T

交换光标所在字符与其前一个字符。

#### Ctrl-U

删除光标所在字符之前的所有字符。
在一些情况下，不管光标在哪个位置，`Ctrl-U` 都会删除整行文字。

#### Ctrl-V

输入时，使用 `Ctrl-V` 允许插入控制字符。例如，下面两条语句是等价的：

```bash
echo -e '\x0a'
echo <Ctl-V><Ctl-J>
```

`Ctrl-V` 在文本编辑器中特别有用。

#### Ctrl-W

当你在终端或 *xterm* 窗口中输入字符时，`Ctrl-W` 将会删除光标所在字符之前到其最近的空白符之间的所有字符。
在一些情况下，`Ctrl-W` 会删除到之前最近的非字母或数字的字符。

#### Ctrl-X

在一些特定的文本处理程序中，剪切高亮文本并复制到剪贴板（clipboard）。

#### Ctrl-Y

粘贴之前使用 `Ctrl-U` 或 `Ctrl-W` 删除的文字。

#### Ctrl-Z

暂停当前运行的任务。

在一些特定的文本处理程序中是替代操作。

在 MSDOS 文件系统中作为 `EOF`（end-of-file，文件终止标记）。

### 空白符

作为命令或变量之间的分隔符。空白符包含空格、制表符、换行符或它们的任意组合。[^9]在一些地方，比如变量赋值时，空白符不应该出现，否则会造成语法错误。

空白行在脚本中不会有任何实际作用，但是可以划分代码，使代码更具可读性。

特殊变量 `$IFS` 是作为一些特定命令的输入域（field）分隔符，默认值为空白符。

> 定义：域是字符串中离散的数据块。使用空白符或者指定的字符（通常由 `$IFS` 决定）来分隔临近域。在一些情况下，域也可以被称作记录（record）。

如果想在字符串或者变量中保留空白符，请引用。

UNIX 过滤器可以使用 POSIX 字符类 `[:space:]` 来寻找和操作空白符。


[^1]: 操作符（operator）用来执行表达式（operation）。最常见的例子就是算术运算符+ - * /。在Bash中，操作符和关键字的概念有一些重叠。
[^2]: 它更被人熟知的名字是三元（ternary）操作符。但是读起来不清晰，而且容易令人混淆。trinary 是一种更加优雅的写法。
[^3]: 美国信息交换标准代码（American Standard Code for Information Interchange）。这是一套可以由计算机存储和处理的7位（bit）字符（包含字母、数字和一系列有限的符号）编码系统。
[^4]: 进程标识符（PID），是分配给正在运行进程的唯一数字标识。可以使用 `ps` 命令查看进程的 PID。<br \>定义：进程是正在执行的命令或程序，通常也称作任务。
[^5]: 由shell来执行大括号扩展操作。命令本身是在扩展的基础上进行操作的。
[^6]: 例外：作为管道的一部分的大括号中的代码块可能会运行在子进程中。<br \><pre>ls | { read firstline; read secondline; }<br/>#  错误。大括号中的代码块在子进程中运行，<br />#+ 因此 "ls" 命令输出的结果不能传递到代码块中。<br/>echo "First line is $firstline; second line is $secondline"  # 无效。<br/><br/># 感谢 S.C.</pre>
[^7]: 正如在古代催情剂（philtre）被认为是一种能引发神奇变化的药剂一样，UNIX 中的过滤器（filter）也是有类似的作用的。<br/>（如果一个程序员做出了一个能够在 Linux 设备上运行的 "love philtre"，那么他将会获得巨大的荣誉。）
[^8]: Bash将之前在命令行中执行过的命令存储在缓存（buffer）中，或者一块内存区域里。可以使用内建命令 `history` 来查看。
[^9]: 换行符本身也是一个空白符。因此这就是为什么仅仅包含一个换行符的空行也被认为是空白符。

