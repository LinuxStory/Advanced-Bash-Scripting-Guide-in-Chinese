# 16.2 复杂命令

**高级用户需要掌握的命令**

### find

-exec `COMMAND` \;

对**find**匹配的每个文件执行`COMMAND`。命令序列以半角分号（;）终止（其中分号需要进行[转义](https://tldp.org/LDP/abs/html/escapingsection.html#ESCP)以确保shell从字面上将其传递给**find**，而不会将其解释为特殊字符）。

```shell
bash$ find ~/ -name '*.txt'
 /home/bozo/.kde/share/apps/karm/karmdata.txt
 /home/bozo/misc/irmeyc.txt
 /home/bozo/test-scripts/1.txt
```

如果`COMMAND`中包含{}，则**find**将用所选文件的完整路径来替换"{}"。

```shell
find ~/ -name 'core*' -exec rm {} \;
# 从用户主目录中删除所有核心转储文件。
```

```shell
find /home/bozo/projects -mtime -1
#                               ^   注意减号标志!
#  列出所有在/home/bozo/projects目录树下且
#  在昨日(当日-1日)修改过的文件。
#
find /home/bozo/projects -mtime 1
#  和上面一样，但是 *恰巧* 在一天前修改过的文件。
#  mtime = 目标文件的最后修改时间
#  ctime = 上次状态更改时间 (通过“chmod”或其他方式)
#  atime = 上次访问时间

DIR=/home/bozo/junk_files
find "$DIR" -type f -atime +5 -exec rm {} \;
#                          ^           ^^
#  大括号是由"find"输出的路径名的占位符。
#
#  删除在"/home/bozo/junk_files"下
#  *至少* 5日内没有访问过(加号 ... +5)的文件。
#
#  "-type filetype", filetype取以下值分别代表
#  f = 常规文件
#  d = 目录
#  l = 符号链接，等
#
#  (‘find’命令man手册和info手册有着完整详尽的选项列表)。
```

```shell
find /etc -exec grep '[0-9][0-9]*[.][0-9][0-9]*[.][0-9][0-9]*[.][0-9][0-9]*' {} \;

# 在/etc目录下找到所有IP地址(xxx.xxx.xxx.xxx)
# 有一些无关紧要的信息。它们能够被过滤掉吗？

# 可能的解决方案如下:

find /etc -type f -exec cat '{}' \; | tr -c '.[:digit:]' '\n' \
| grep '^[^.][^.]*\.[^.][^.]*\.[^.][^.]*\.[^.][^.]*$'
#
#  [:digit:]是POSIX 1003.2标准引入的字符类之一。

# 感谢Stéphane Chazelas。
```

> ![note](http://tldp.org/LDP/abs/images/note.gif)**find**命令的`-exec`选项请不要与shell内建命令[exec](https://tldp.org/LDP/abs/html/internal.html#EXECREF)混淆。

**样例 16-3. *Badname*, 消除当前目录中包含错误字符和[空格](https://tldp.org/LDP/abs/html/special-chars.html#WHITESPACEREF)的文件名。**

```shell
#!/bin/bash
# badname.sh
# 删除当前目录中包含错误字符的文件名。

for filename in *
do
  badname=`echo "$filename" | sed -n /[\+\{\;\"\\\=\?~\(\)\<\>\&\*\|\$]/p`
# badname=`echo "$filename" | sed -n '/[+{;"\=?~()<>&*|$]/p'` 同样有效。
# 删除包含这些污点的文件:     + { ; " \ = ? ~ ( ) < > & * | $
#
  rm $badname 2>/dev/null
#             ^^^^^^^^^^^ 抛弃错误信息.
done

# 现在，处理包含各种空白的文件。
find . -name "* *" -exec rm -f {} \;
# find命令找到的文件路径名将替换"{}"
# 从字面上，'\'确保';'解释该命令结束

exit 0

#---------------------------------------------------------------------
# 由于exit命令，在这条虚线下的命令将不会被执行。

# 上述脚本的替代方案:
find . -name '*[+{;"\\=?~()<>&*|$ ]*' -maxdepth 0 \
-exec rm -f '{}' \;
#  “-maxdepth 0”选项确保find命令不会搜索当前目录($PWD)以下的子目录。

# (感谢 S.C.)
```

**样例 16-4. 通过*inode*编号删除一个文件**

```shell
#!/bin/bash
# idelete.sh: 删除一个文件通过其inode编号。

#  当文件名以非法字符开头时这很有用，
#  比如?或-。

ARGCOUNT=1                      # 文件名参数必须传递给脚本.
E_WRONGARGS=70
E_FILE_NOT_EXIST=71
E_CHANGED_MIND=72

if [ $# -ne "$ARGCOUNT" ]
then
  echo "Usage: `basename $0` filename"
  exit $E_WRONGARGS
fi  

if [ ! -e "$1" ]
then
  echo "File \""$1"\" does not exist."
  exit $E_FILE_NOT_EXIST
fi  

inum=`ls -i | grep "$1" | awk '{print $1}'`
# inum = inode (索引节点) 文件数
# -----------------------------------------------------------------------
# 每个文件都有一个inode，一个用于保存其物理地址信息的记录。
# -----------------------------------------------------------------------

echo; echo -n "Are you absolutely sure you want to delete \"$1\" (y/n)? "
# rm命令的-v选项也会同样询问这个问题。
read answer
case "$answer" in
[nN]) echo "Changed your mind, huh?"
      exit $E_CHANGED_MIND
      ;;
*)    echo "Deleting file \"$1\".";;
esac

find . -inum $inum -exec rm {} \;
#                           ^^
#        大括号是"find"所输出文本的占位符。
echo "File "\"$1"\" deleted!"

exit 0
```

不使用`-exec`选项的**find**命令同样有效。

```shell
#!/bin/bash
#  寻找suid root文件。
#  一个奇怪的suid文件可能代表一个安全漏洞，
#  甚至是系统入侵。

directory="/usr/sbin"
# 您也可以尝试/sbin, /bin, /usr/bin, /usr/local/bin等等。
permissions="+4000"  # root用户suid (危险!)


for file in $( find "$directory" -perm "$permissions" )
do
  ls -ltF --author "$file"
done
```

您可以查看使用**find**命令的[样例 16-30](https://tldp.org/LDP/abs/html/filearchiv.html#EX48)，[样例 3-4](https://tldp.org/LDP/abs/html/special-chars.html#EX58)和[样例 11-10](https://tldp.org/LDP/abs/html/loops1.html#FINDSTRING)。它的[man手册](https://tldp.org/LDP/abs/html/basic.html#MANREF)提供了关于这个复杂且强大的命令的更多细节。

### xargs

这是将参数传递给命令的过滤器，同时也是一个用于组装命令本身的工具。它将数据流分解成足够小的块，以供过滤器和命令进行处理。可以将它视为[backquotes](https://tldp.org/LDP/abs/html/commandsub.html#BACKQUOTESREF)的强大替代品。在由于参数数量过多而导致的[命令替换](https://tldp.org/LDP/abs/html/commandsub.html#COMMANDSUBREF)失败的情况下，切换使用**xargs**通常有效。[[1]](https://tldp.org/LDP/abs/html/moreadv.html#FTN.AEN10465)通常，**xargs**从标准输入(`stdin`)或者管道(pipe)中读取数据，但它也可以从文件输出中读取。

**xargs**的默认命令是[echo](https://tldp.org/LDP/abs/html/internal.html#ECHOREF)。这意味着通过管道输入到**xargs**可能会去除换行符和其他空格字符。

```shell
bash$ ls -l
total 0
 -rw-rw-r--    1 bozo  bozo         0 Jan 29 23:58 file1
 -rw-rw-r--    1 bozo  bozo         0 Jan 29 23:58 file2



bash$ ls -l | xargs
total 0 -rw-rw-r-- 1 bozo bozo 0 Jan 29 23:58 file1 -rw-rw-r-- 1 bozo bozo 0 Jan...



bash$ find ~/mail -type f | xargs grep "Linux"
./misc:User-Agent: slrn/0.9.8.1 (Linux)
 ./sent-mail-jul-2005: hosted by the Linux Documentation Project.
 ./sent-mail-jul-2005: (Linux Documentation Project Site, rtf version)
 ./sent-mail-jul-2005: Subject: Criticism of Bozo's Windows/Linux article
 ./sent-mail-jul-2005: while mentioning that the Linux ext2/ext3 filesystem
 . . .
```

`ls | xargs -p -l gzip`[gzips](https://tldp.org/LDP/abs/html/filearchiv.html#GZIPREF)当前目录下的所有文件，在每次操作前均提示一下。

> ![note](http://tldp.org/LDP/abs/images/note.gif)请注意*xargs*命令按顺序处理传递给它的参数，*一次一个*。

```shell
bash$ find /usr/bin | xargs file
/usr/bin:          directory
 /usr/bin/foomatic-ppd-options:          perl script text executable
 . . .
```

{% hint style="info" %}

一个有趣的*xargs*选项是`-n NN`，它将传递参数的数量限制为*NN*。

`ls | xargs -n 8 echo` 该条命令以`8`列的形式列出在当前目录下的文件。

{% endhint %}

{% hint style="info" %}

另一个有用的选项是`-0`，结合`find -print0`或` grep -lZ`。这允许处理包含空格或引号的参数。

`find / -type f -print0 | xargs -0 grep -liwZ GUI | xargs -0 rm -f`

`grep -rliwZ GUI / | xargs -0 rm -f`*

以上两条命令都起到移除所有包含"GUI"的文件。*(感谢 S.C.)*

或者：

```shell
cat /proc/"$pid"/"$OPTION" | xargs -0 echo
#  格式化输出:               ^^^^^^^^^^^^^^^
#  来自Han Holl对"/dev and /proc"一章中
#  "get-commandline.sh"脚本的修正。
```

{% endhint %}

{% hint style="info" %}

*xargs*的`-P`选项允许并行(parallel)运行进程。这能够提升多核CPU计算机中的执行速度。

```shell
#!/bin/bash

ls *gif | xargs -t -n1 -P2 gif2png
# 将当前目录中的所有gif图像转换为png。

# 选项:
# =======
# -t    将命令输出到标准错误(stderr)。
# -n1   每个命令行最多1个参数。
# -P2   最多同时运行2个进程。

# 感谢Roberto Polli的启发。
```

{% endhint %}

**样例 16-5. Logfile: 使用*xargs*来监视系统日志**

```shell
#!/bin/bash

# 从/var/log/messages的末尾
# 来生成当前目录中的日志文件。

# 注意: 如果普通用户调用此脚本，
# 则/var/log/messages必须是全局可读的。
#         #root chmod 644 /var/log/messages

LINES=5

( date; uname -a ) >>logfile
# 时间和设备名
echo ---------------------------------------------------------- >>logfile
tail -n $LINES /var/log/messages | xargs | fmt -s >>logfile
echo >>logfile
echo >>logfile

exit 0

#  注意:
#  ----
#  正如Frank Wang指出，
#  源文件中不匹配的引号 (单引号或双引号) 可能会导致xargs无法识别

#
#  他建议第15行采用如下替换方案：
#  tail -n $LINES /var/log/messages | tr -d "\"'" | xargs | fmt -s >>logfile



#  练习:
#  --------
#  修改此脚本，以20分钟的间隔跟踪/var/log/messages中的更改。
#  提示：使用"watch"命令
```

[与**find**相同](https://tldp.org/LDP/abs/html/moreadv.html#CURLYBRACKETSREF)，大括号对用作替换文本的占位符。

**样例 16-6. 将当前目录下的文件复制到另一个目录**

```shell
#!/bin/bash
# copydir.sh

#  将当前目录($PWD)下的所有文件复制(详细的)
#  到命令行中指定的目录。

E_NOARGS=85

if [ -z "$1" ]   # 如果没有提供参数则退出。
then
  echo "Usage: `basename $0` directory-to-copy-to"
  exit $E_NOARGS
fi  

ls . | xargs -i -t cp ./{} $1
#            ^^ ^^      ^^
#  -t 是“详细的”(输出命令行到标准错误stderr)选项。
#  -i 是“替换字符串”选项。
#  {} 是输出文本的占位符。
#  这类似于在 “find”命令中使用的大括号对。

#
#  列出当前目录中的文件(ls .)，
#  将 “ls” 命令的输出作为参数传递给 “xargs” (-i -t选项)，
#  并且将这些参数({})复制(cp)到新的目录($1)。
#
#  最终结果完全等同于
#    cp * $1
#  除非某个文件名包含 “空格” 字符。

exit 0
```

**样例 16-7. 通过进程名杀死进程**

```shell
#!/bin/bash
# kill-byname.sh: 通过进程名杀死进程。
# 您可以与kill-process.sh脚本作比较。

#  例如，
#  试图执行"./kill-byname.sh xterm"
#  然后眼睁睁地看着你桌面上的xterm都云消雾散。

#  警告:
#  -------
#  这是一个相当危险的脚本。
#  不在意地(尤其是root用户)运行它可能会导致数据丢失和其他不良影响。

E_BADARGS=66

if test -z "$1"  # 你难道不提供命令行参数吗？
then
  echo "Usage: `basename $0` Process(es)_to_kill"
  exit $E_BADARGS
fi


PROCESS_NAME="$1"
ps ax | grep "$PROCESS_NAME" | awk '{print $1}' | xargs -i kill {} 2&>/dev/null
#                                                       ^^      ^^

# ---------------------------------------------------------------
# 注意:
# -i 是xargs命令的“替换字符串”选项。
# 大括号是替换的占位符。
# 2&>/dev/null丢弃不需要的错误消息。
#
# 可以把grep "$PROCESS_NAME"替换成pidof "$PROCESS_NAME"吗？
# ---------------------------------------------------------------

exit $?

#  “killall” 命令与该脚本具有相同的效果，
#  但是使用它并不具有教育意义。
```

**样例 16-8. 使用*xargs*进行词频分析**

```shell
#!/bin/bash
# wf2.sh: 文本文件的粗词频分析。

# 使用"xargs"将文本行分解为单个单词。
# 将此样例与稍后的"wf.sh"脚本进行比较。

# 在命令行检查输入文件。
ARGS=1
E_BADARGS=85
E_NOFILE=86

if [ $# -ne "$ARGS" ]
# 是否传递给脚本正确的参数数量？
then
  echo "Usage: `basename $0` filename"
  exit $E_BADARGS
fi

if [ ! -f "$1" ]       # 判断文件存在吗？
then
  echo "File \"$1\" does not exist."
  exit $E_NOFILE
fi



#####################################################
cat "$1" | xargs -n1 | \
#  列出文件，每行一个单词。
tr A-Z a-z | \
#  把大写字母转换为小写字母。
sed -e 's/\.//g'  -e 's/\,//g' -e 's/ /\
/g' | \
#  过滤掉句号和逗号，
#  并将单词之间的空格更改为换行，
sort | uniq -c | sort -nr
#  最后删除重复项，并前缀计数和数字排序。
#####################################################

#  该脚本与"wf.sh"做了相同的工作，
#  但是更麻烦了一点，它运行得更慢 (为什么？)。

exit $?
```

### expr

通用表达式计算器: 根据给定的表达式对参数进行连接和求值 (参数必须用空格分隔)。表达式可以是算术运算、比较运算、字符串运算或逻辑运算。

`expr 3 + 5`

返回8

`expr 5 % 3`

返回2

`expr 1 / 0`

返回错误信息，expr: division by zero

不允许非法算术运算。

`expr 5 \* 3`

返回15

在**expr**的算术表达式中使用乘法运算符(*)时，必须对其进行转义。

<code>y=\`expr $y + 1\`</code>

将一个变量递增，与<strong>`let y=y+1`</strong>和<strong>`y=((y+1))`</strong>等效。这是一个[算数扩展](https://tldp.org/LDP/abs/html/arithexp.html#ARITHEXPREF)的例子。

<code>**z=\`expr substr \$string \$position \$length\`**</code>

从\$position开始，提取\$length长度的子字符串。

**样例 16-9. 使用 *expr***

```shell
#!/bin/bash

# 演示'expr'命令的一些用途
# =======================================

echo

# 算数运算符
# ---------- ---------

echo "Arithmetic Operators"
echo
a=`expr 5 + 3`
echo "5 + 3 = $a"

a=`expr $a + 1`
echo
echo "a + 1 = $a"
echo "(incrementing a variable)"

a=`expr 5 % 3`
# 取模
echo
echo "5 mod 3 = $a"

echo
echo

# 逻辑运算符
# ------- ---------

#  真返回1，假返回0，
#  与正常的Bash惯例相反。

echo "Logical Operators"
echo

x=24
y=25
b=`expr $x = $y`         # 测试是否相等。
echo "b = $b"            # 0  ( $x -ne $y )
echo

a=3
b=`expr $a \> 10`
echo 'b=`expr $a \> 10`, therefore...'
echo "If a > 10, b = 0 (false)"
echo "b = $b"            # 0  ( 3 ! -gt 10 )
echo

b=`expr $a \< 10`
echo "If a < 10, b = 1 (true)"
echo "b = $b"            # 1  ( 3 -lt 10 )
echo
# 注意运算符的转义。

b=`expr $a \<= 3`
echo "If a <= 3, b = 1 (true)"
echo "b = $b"            # 1  ( 3 -le 3 )
# 还有一个 “\>=” 运算符 (大于或等于)。


echo
echo



# 字符串运算符
# ------ ---------

echo "String Operators"
echo

a=1234zipper43231
echo "The string being operated upon is \"$a\"."

# length: 字符串长度
b=`expr length $a`
echo "Length of \"$a\" is $b."

# index: 子字符串中与原字符串中匹配的第一个字符的位置。
b=`expr index $a 23`
echo "Numerical position of first \"2\" in \"$a\" is \"$b\"."

# substr: 提取子字符串,起始位置和长度需要指定
b=`expr substr $a 2 6`
echo "Substring of \"$a\", starting at position 2,\
and 6 chars long is \"$b\"."


#  "match"操作的默认行为是
#  从字符串的起始位置搜索匹配的字符串。
#  
#       使用正则表达式 ...
b=`expr match "$a" '[0-9]*'`               #  数值型字符计数。
echo Number of digits at the beginning of \"$a\" is $b.
b=`expr match "$a" '\([0-9]*\)'`           #  请注意，转义括号
#                   ==      ==             #  触发子字符串匹配。
echo "The digits at the beginning of \"$a\" are \"$b\"."

echo

exit 0
```

![note](https://tldp.org/LDP/abs/images/caution.gif)[: (*null*)](https://tldp.org/LDP/abs/html/special-chars.html#NULLREF) 运算符可以替代**match**。举个例子，<code>b=\`expr $a : [0-9]*\`</code>完全等价于上述列举的<code>b=\`expr match $a [0-9]*\`</code>。

```shell
#!/bin/bash

echo
echo "String operations using \"expr \$string : \" construct"
echo "==================================================="
echo

a=1234zipper5FLIPPER43231

echo "The string being operated upon is \"`expr "$a" : '\(.*\)'`\"."
#     转义括号为分组运算符。                                ==  ==

#       ***************************
#              转义括号
#            匹配子字符串
#       ***************************


#  如果没有转义括号 ...
#  那么'expr'会将字符串操作数转换为整数。

echo "Length of \"$a\" is `expr "$a" : '.*'`."   # 字符串长度

echo "Number of digits at the beginning of \"$a\" is `expr "$a" : '[0-9]*'`."

# ------------------------------------------------------------------------- #

echo

echo "The digits at the beginning of \"$a\" are `expr "$a" : '\([0-9]*\)'`."
#                                                             ==      ==
echo "The first 7 characters of \"$a\" are `expr "$a" : '\(.......\)'`."
#         =====                                          ==       ==
# 同样，转义的括号强制子字符串匹配。
#
echo "The last 7 characters of \"$a\" are `expr "$a" : '.*\(.......\)'`."
#         ====                  字符串运算符的末尾         ^^
#  (实际上，这意味着跳过一个或多个任何字符，
#  直到找到指定的子字符串。)

echo

exit 0
```

上面的脚本解释了**expr**是如何使用转义括号 -- \( ... \) -- 分组运算符与[正则表达式](https://tldp.org/LDP/abs/html/regexp.html#REGEXREF)解析一起使用来匹配字符串。以下是另一个 “现实生活”中的案例。

```shell
# 去除开头和结尾的空格。
LRFDATE=`expr "$LRFDATE" : '[[:space:]]*\(.*\)[[:space:]]*$'`

#  摘自Peter Knowles的"booklistgen.sh"脚本
#  该脚本用于将文件转换为Sony Librie/PRS-50X格式。
#  (http://booklistgensh.peterknowles.com)
```

[Perl](https://tldp.org/LDP/abs/html/wrapper.html#PERLREF)，[sed](https://tldp.org/LDP/abs/html/sedawk.html#SEDREF)，和[awk](https://tldp.org/LDP/abs/html/awk.html#AWKREF)具有更为优秀的字符串解析功能。脚本中简短的**sed**或者**awk**"子例程(subroutine)"(参见[章节 36.2](https://tldp.org/LDP/abs/html/wrapper.html))相比**expr**是更优的替代方案。

有关在字符串操作中使用**expr**的更多信息请参见[章节 10.1](https://tldp.org/LDP/abs/html/string-manipulation.html)。

## 注记

[[1]](https://tldp.org/LDP/abs/html/moreadv.html#AEN10465)即使不是绝对需要**xargs**，它也可以加快涉及多个文件的[批处理](https://tldp.org/LDP/abs/html/timedate.html#BATCHPROCREF)指令的执行速度。
