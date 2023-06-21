# 16.4 文本处理命令

> **处理文本和文本文件的命令**

### sort

这是给文本排序的实用程序，通常用作管道的过滤器。该命令按照升序或降序给*文本流*或文件排序，或者根据各种键或字符位置排序。使用`-m`选项，它会合并预排序(presorted)的输入文件。*info手册*列出了该命令的许多功能和选项。你可以参阅[样例 11-10](https://tldp.org/LDP/abs/html/loops1.html#FINDSTRING)，[样例 11-11](https://tldp.org/LDP/abs/html/loops1.html#SYMLINKS)和[样例 A-8](https://tldp.org/LDP/abs/html/contributed-scripts.html#MAKEDICT)。

### tsort

即*拓扑排序*，会成对读取空格分隔的字符串并根据输入模式进行排序。**tsort**最初目的是在UNIX的“古早”版本中对过时的*ld*链接器的依赖项列表进行排序。

*tsort*的结果通常与上述标准的**sort**命令执行的结果明显不同。

### uniq

此过滤器从已排序的文件中删除重复的行。通常能够在与[sort](https://tldp.org/LDP/abs/html/textproc.html#SORTREF)结合的管道中看到。

```shell
cat list-1 list-2 list-3 | sort | uniq > final.list
# 连接这些列表文件，
# 给它们排序,
# 删除重复行,
# 最后将结果写入输出文件。
```

`-c`选项会以出现次数在输入文件的每行前添加前缀。这是一个很有用的选项。

```shell
bash$ cat testfile
This line occurs only once.
 This line occurs twice.
 This line occurs twice.
 This line occurs three times.
 This line occurs three times.
 This line occurs three times.


bash$ uniq -c testfile
      1 This line occurs only once.
       2 This line occurs twice.
       3 This line occurs three times.


bash$ sort testfile | uniq -c | sort -nr
      3 This line occurs three times.
       2 This line occurs twice.
       1 This line occurs only once.
```

`sort INPUTFILE | uniq -c | sort -nr`命令字符串会在`INPUTFILE`文件上*生成频率列表* (**sort**的`-nr`选项会导致反向数值排序)。此模板可用于分析日志文件和字典列表，以及需要检查文档的词汇结构的任何地方。

**样例 16-12. 词频分析**

```shell
#!/bin/bash
# wf.sh: 文本文件上的粗词频分析。
# 这是"wf2.sh"脚本的更高效的一个版本。


# 在命令行检查输入文件。
ARGS=1
E_BADARGS=85
E_NOFILE=86

if [ $# -ne "$ARGS" ]  # 检查是否给脚本传递了正确数量的参数。
then
  echo "Usage: `basename $0` filename"
  exit $E_BADARGS
fi

if [ ! -f "$1" ]       # 检查文件是否存在。
then
  echo "File \"$1\" does not exist."
  exit $E_NOFILE
fi



########################################################
# main ()
sed -e 's/\.//g'  -e 's/\,//g' -e 's/ /\
/g' "$1" | tr 'A-Z' 'a-z' | sort | uniq -c | sort -nr
#                           =========================
#                                   出现频率

#  过滤掉句点和逗号，
#  并将单词之间的空格更改为换行，
#  然后将字符转换为小写，
#  最后添加计数和数字排序前缀。

#  Arun Giridhar建议将如上修改为:
#  . . . | sort | uniq -c | sort +1 [-f] | sort +0 -nr
#  这增加了一个辅助排序键，
#  相等出现的词汇按字母顺序排序。

#  正如他如此解释:
#  “这实际上是基数排序，
#  首先是最低有效列
#  (单词或字符串，可选择不区分大小写)，
#  最后是最高有效列 (频率)。”

#  正如Frank Wang解释，以上等价于
#       . . . | sort | uniq -c | sort +0 -nr
# 以下代码也同样有效
#       . . . | sort | uniq -c | sort -k1nr -k
########################################################

exit 0

# 练习:
# ---------
# 1) 添加 “sed” 命令以过滤掉其他标点符号，例如分号。
# 2) 修改脚本，以同时过滤掉多个空格和其他空格标志。
```

```shell
bash$ cat testfile
This line occurs only once.
 This line occurs twice.
 This line occurs twice.
 This line occurs three times.
 This line occurs three times.
 This line occurs three times.


bash$ ./wf.sh testfile
      6 this
       6 occurs
       6 line
       3 times
       3 three
       2 twice
       1 only
       1 once
```

### expand, unexpand

**expand**过滤器将制表符转换为空格。它经常用于[管道(pipe)](https://tldp.org/LDP/abs/html/special-chars.html#PIPEREF)中。

**unexpand**过滤器将空格转换为制表符。起着和**expand**相反的作用。

### cut

这是从文件中提取[字段](https://tldp.org/LDP/abs/html/special-chars.html#FIELDREF)的工具。它类似于[awk](https://tldp.org/LDP/abs/html/awk.html#AWKREF)中的`print $ N`命令集，但作用更有限。在脚本中使用*cut*可能比*awk*更简单。特别重要的是`-d` (分隔符) 和`-f `(字段说明符) 选项。

使用**cut**获得已挂载文件系统的列表:

```shell
cut -d ' ' -f1,2 /etc/mtab
```

使用**cut**列出操作系统和内核版本：

```shell
uname -a | cut -d" " -f1,3,11,1
```

使用**cut**从电子邮件文件夹中提取消息头（message header）:

```shell
bash$ grep '^Subject:' read-messages | cut -c10-80
Re: Linux suitable for mission-critical apps?
 MAKE MILLIONS WORKING AT HOME!!!
 Spam complaint
 Re: Spam complaint
```

使用**cut**解析文件：

```shell
# 列出所有在/etc/passwd中的用户。

FILENAME=/etc/passwd

for user in $(cut -d: -f1 $FILENAME)
do
  echo $user
done

# 感谢Oleg Philon提出的建议。
```

<code>**cut -d ' ' -f2,3 filename**</code> 与 <code>**awk -F'[ ]' '{ print \$2, \$3 }' filename**</code> 等价。

>  ![note](http://tldp.org/LDP/abs/images/note.gif)甚至可以将换行符指定为分隔符。诀窍就是实际在命令序列中键入一个换行符(**RETURN**)。

```shell
bash$ cut -d'
 ' -f3,7,19 testfile
This is line 3 of testfile.
 This is line 7 of testfile.
 This is line 19 of testfile.
```

感谢Jaka Kranjc指出。

你也可以参阅[样例 16-48](https://tldp.org/LDP/abs/html/mathc.html#BASE)。

### paste

用于将不同文件合并到单个、多列文件中的工具。与[cut](https://tldp.org/LDP/abs/html/textproc.html#CUTREF)结合使用，对于创建系统日志文件很有用。

```shell
bash$ cat items
alphabet blocks
 building blocks
 cables

bash$ cat prices
$1.00/dozen
 $2.50 ea.
 $3.75

bash$ paste items prices
alphabet blocks $1.00/dozen
 building blocks $2.50 ea.
 cables  $3.75
```

### join

该命令类似于**paste**，有着特殊的用途。这个强大的实用程序允许以有意义的方式合并两个文件，这实质上创建了简单版本的关系数据库。

**join**命令仅对两个文件进行操作，但是仅将具有公共标记[字段](https://tldp.org/LDP/abs/html/special-chars.html#FIELDREF) (通常是数字标签) 的那些行粘贴在一起，然后将结果写入`标准输出(stdout)`。要加入的文件应根据标记的字段进行排序，以使配对正常工作。

```shell
File: 1.data

100 Shoes
200 Laces
300 Socks
```

```shell
File: 2.data

100 $40.00
200 $1.00
300 $2.00
```

```shell
bash$ join 1.data 2.data
File: 1.data 2.data

 100 Shoes $40.00
 200 Laces $1.00
 300 Socks $2.00
```

> ![note](http://tldp.org/LDP/abs/images/note.gif)标记字段在输出中只出现一次。

### head

将文件的起始传递给`标准输出(stdout)`。默认为`10`行，但可以指定不同的数字。该命令有许多有趣的选项。

**样例 16-13. 哪些文件是脚本？**

```shell
#!/bin/bash
# script-detector.sh: 检测目录内的脚本。

TESTCHARS=2    # 测试前2个字符。
SHABANG='#!'   # 以"sha-bang"开头的脚本。

for file in *  # 遍历当前目录中的所有文件。
do
  if [[ `head -c$TESTCHARS "$file"` = "$SHABANG" ]]
  #      head -c2                      #!
  #  “head” 的 “-c” 选项输出指定数量的字符，而不是行 (默认值)。
  then
    echo "File \"$file\" is a script."
  else
    echo "File \"$file\" is *not* a script."
  fi
done

exit 0

#  练习:
#  ---------
#  1) 修改此脚本，以将扫描脚本的目录 (而不仅仅是当前工作目录) 作为可选参数。
#
#  2) 就目前而言，该脚本会“误报”Perl，awk和其他脚本语言脚本。
#     尝试纠正这个。
```

**样例 16-14. 生成10位随机数**

```shell
#!/bin/bash
# rnd.sh: 输出10位随机数

# Stephane Chazelas所写的脚本.

head -c4 /dev/urandom | od -N4 -tu4 | sed -ne '1s/.* //p'


# =================================================================== #

# 分析
# --------

# head:
# -c4 选项读取前4个字符。

# od:
# -N4 选项限制输出为4个字符。
# -tu4 选项选择无符号十进制格式进行输出。

# sed: 
# -n 选项，结合"s"命令的"p"标志
# 仅输出匹配的行。



# 该脚本的作者解释了'sed'所做的动作，如下所示。

# head -c4 /dev/urandom | od -N4 -tu4 | sed -ne '1s/.* //p'
# ----------------------------------> |

# 假设到达"sed"的输出为       --------> |
# 0000000 1198195154\n

#  sed开始读取字符串: 0000000 1198195154\n。
#  这里它发现了一个换行符，
#  所以它准备处理第一行 (0000000 1198195154)。
#  它查看其<范围><动作>。第一个且唯一一个是

#   范围         动作
#   1         s/.* //p

#  行号在范围内，因此它执行如下操作:
#  尝试替换行中以空格结尾的最长字符串
#  ("0000000 ")不带任何东西(//)，如果成功，则打印结果
#  (这里"p" 是"s" 命令的标志, 这与"p"命令有所不同)。

#  现在，sed已经准备好继续读取输入。（请注意，在继续之前，
#  ，如果没有通过-n选项，则sed会再次打印该行）。

#  现在，sed读取其余字符，并找到文件的结尾。
#  现在可以处理其第二行(由于是最后一行，因此也编号为 “$”)。
#  它似乎与任何<range>均不匹配，所以工作结束。

#  sed命令简要来说：
#  "仅在第一行, 删除最右边的任何字符，
#  然后打印内容"

#  更好的解决方法是：
#           sed -e 's/.* //;q'

# 这里，有两个<范围><动作> (也可以写成
#           sed -e 's/.* //' -e q):

#          范围                动作
#   nothing (matches line)   s/.* //
#   nothing (matches line)   q (退出)

#  在这里，sed只读取它的第一行输入。
#  它执行这两个操作，并在退出之前 (由于 “q” 操作) 打印行 (替换)，因为未传递 “-n” 选项。

# =================================================================== #

# 与上述单行脚本相比，一个更简单的替代方案为:
#           head -c4 /dev/urandom| od -An -tu4

exit
```

你也可以参阅[样例 16-39](https://tldp.org/LDP/abs/html/filearchiv.html#EX52)。

### tail

将文件的末尾传递给`标准输出(stdout)`。默认为`10`行，但这可以通过`-n`选项进行更改。通常用于跟踪系统日志文件的更改，使用`-f`选项，它输出附加到文件的行。

**样例 16-15. 使用*tail*来监控系统日志**

```shell
#!/bin/bash

filename=sys.log

cat /dev/null > $filename; echo "Creating / cleaning out file."
#  如果文件不存在，则创建文件，
#  如果存在，则截断为0长度（清空该文件）。
#  : > filename   和   > filename 同样有效。

tail /var/log/messages > $filename  
# /var/log/messages必须具有全局读取权限，该命令才有效。

echo "$filename contains tail end of system log."

exit 0
```

{% hint style="info" %}

要列出文本文件的特定行，请通过[管道(pipe)](https://tldp.org/LDP/abs/html/special-chars.html#PIPEREF)将**head**的输出输送给**tail -n 1**。例如，<code>**head -n 8 database.txt | tail -n 1**</code>列出了文件<code>**database.txt**</code>的第8行。

将文本文件的给定块(block)设置为变量:

```shell
var=$(head -n $m $filename | tail -n $n)

# filename = 文件名
# m = 从文件的开头，行数到块的结尾
# n = 要设置变量的行数 (从块末尾修剪)
```

{% endhint %}

> ![note](http://tldp.org/LDP/abs/images/note.gif)新版本的**tail**弃用了旧版本的<strong>tail -\$LINES filename</strong>的用法。标准的<strong>tail -n \$LINES filename</strong>仍然是正确的。

你也可以参阅[样例 16-5](https://tldp.org/LDP/abs/html/moreadv.html#EX41)、[样例 16-39](https://tldp.org/LDP/abs/html/filearchiv.html#EX52)和[样例 32-6](https://tldp.org/LDP/abs/html/debugging.html#ONLINE)。

### grep

使用[正则表达式](https://tldp.org/LDP/abs/html/regexp.html#REGEXREF)的多用途文件搜索工具。它最初是古老的**ed**行编辑器中的命令/过滤器: `g/re/p` -- *global - regular expression - print*。

<h4>grep <code><em>pattern</em></code> [<code><em>file...</em></code>]</h4>

在目标文件中搜索*pattern*，其中*pattern*可以是文字文本或正则表达式。

```shell
bash$ grep '[rst]ystem.$' osinfo.txt
The GPL governs the distribution of the Linux operating system.
```

如果目标文件没有指定，**grep**会对`标准输出(stdout)`进行过滤，就像在[管道(pipe)](https://tldp.org/LDP/abs/html/special-chars.html#PIPEREF)中一样。

```shell
bash$ ps ax | grep clock
765 tty1     S      0:00 xclock
 901 pts/1    S      0:00 grep clock
```

`-i`选项为不区分大小写的搜索。

`-w`选项为全局匹配。

`-l`选项仅列出找到匹配的文件，而不列出匹配的行。

`-r` (递归) 选项搜索当前工作目录中的文件及其下方的所有子目录。

`-n`选项列出了匹配的行以及行号。

```shell
bash$ grep -n Linux osinfo.txt
2:This is a file containing information about Linux.
 6:The GPL governs the distribution of the Linux operating system.
```

`-v`（或`--invert-match`）选项*过滤掉*匹配项。

```shell
grep pattern1 *.txt | grep -v pattern2

# 匹配 "*.txt" 文件中包含 "pattern1" 的所有行，
# 但 ***不*** 包含 "pattern2"。
```

`-c` (`--count`) 选项给出匹配的数字计数，而不是实际列出匹配项。

```shell
grep -c txt *.sgml   # (在"*.sgml"文件中"txt"出现的次数)


#   grep -cz .
#            ^ 点
# 表示匹配"."，计数 (-c) 且以零数据分割(-z)的项
# 即非空的 (包含至少1个字符)。
# 
printf 'a b\nc  d\n\n\n\n\n\000\n\000e\000\000\nf' | grep -cz .     # 3
printf 'a b\nc  d\n\n\n\n\n\000\n\000e\000\000\nf' | grep -cz '$'   # 5
printf 'a b\nc  d\n\n\n\n\n\000\n\000e\000\000\nf' | grep -cz '^'   # 5
#
printf 'a b\nc  d\n\n\n\n\n\000\n\000e\000\000\nf' | grep -c '$'    # 9
# 默认情况下，换行符(\n)会将需要匹配的项目分开。
# 请注意，-z选项是GNU "grep"命令的特性。

# 感谢S.C.
```

`--color`（或者`--colour`）选项会用颜色在（控制台或*xterm*窗口）中标记匹配的字符串。由于*grep*会打印出包含匹配模式的整行，这会让你清晰地看到*哪个*被匹配到了。另请参阅`-o`选项，该选项仅显示行中匹配到的部分。

**样例 16-16. 打印出本地电子邮件中的*From*行**

```shell
#!/bin/bash
# from.sh

#  在Solaris、BSD等系统中模拟'from'使用工具。
#  输出电子邮件目录中所有邮件中的 “发件人(From)” 标题行。


MAILDIR=~/mail/*               #  不引用变量。为什么？
# 或许需要检查目录是否存在:   if [ -d $MAILDIR ] . . .
GREP_OPTS="-H -A 5 --color"    #  展示文件，展示额外上下文
                               #  并用颜色显示"from"。
TARGETSTR="^From"              #  行开头的"From"。

for file in $MAILDIR           #  不引用变量。
do
  grep $GREP_OPTS "$TARGETSTR" "$file"
  #    ^^^^^^^^^^              #  同样，不引用变量。
  echo
done

exit $?

#  你可能想要将该脚本的输出通过管道传递给'more'命令
#  或者重定向到文件. . . 
```

当在给定多个目标文件的情况下调用时，**grep**会显示是哪个文件包含匹配项。

```shell
bash$ grep Linux osinfo.txt misc.txt
osinfo.txt:This is a file containing information about Linux.
 osinfo.txt:The GPL governs the distribution of the Linux operating system.
 misc.txt:The Linux operating system is steadily gaining in popularity.
```

{% hint style="info" %}

要强制**grep**在仅搜索一个目标文件时显示文件名，只需将`/dev/null`作为第二个文件即可。

```shell
bash$ grep Linux osinfo.txt /dev/null
osinfo.txt:This is a file containing information about Linux.
 osinfo.txt:The GPL governs the distribution of the Linux operating system.
```

{% endhint %}

如果匹配成功，**grep**将返回0的[退出状态](https://tldp.org/LDP/abs/html/exit-status.html#EXITSTATUSREF)，因此这在脚本中的条件测试中很有用，尤其是与`-q`选项结合使用以取消输出。

```shell
SUCCESS=0                      # 如果grep查找成功
word=Linux
filename=data.file

grep -q "$word" "$filename"    #  "-q"选项会
                               #  导致什么都不在stdout中输出
if [ $? -eq $SUCCESS ]
# if grep -q "$word" "$filename" 可以替换 5 - 7 行。
then
  echo "$word found in $filename"
else
  echo "$word not found in $filename"
fi
```

[样例 32-6](https://tldp.org/LDP/abs/html/debugging.html#ONLINE)演示了如何使用**grep**命令在系统日志文件中搜索单词。

**样例 16-17. 在脚本中模仿*grep*命令**

```shell
#!/bin/bash
# grp.sh: 实现grep命令的基本功能。

E_BADARGS=85

if [ -z "$1" ]    # 检查脚本的参数。
then
  echo "Usage: `basename $0` pattern"
  exit $E_BADARGS
fi  

echo

for file in *     # 遍历$PWD中的全部文件
do
  output=$(sed -n /"$1"/p $file)  # 命令替换。

  if [ ! -z "$output" ]           # 如果"$output"没有引用会发生什么？
  then
    echo -n "$file: "
    echo "$output"
  fi              #  sed -ne "/$1/s|^|${file}: |p" 与上等效。

  echo
done  

echo

exit 0

# 练习:
# ---------
# 1) 如果在任何给定文件中有多个匹配项，则将换行符添加到输出中。
# 2) 添加特性。
```

**grep**如何同时搜索两个 (或多个) 单独的模式(pattern)？如果你希望**grep**显示一个文件中的所有行或同时包含 “pattern1” 和 “pattern2” 的文件，该怎么办？

一种方法是将**grep pattern1**的结果通过[管道(pipe)](https://tldp.org/LDP/abs/html/special-chars.html#PIPEREF)输送给**grep pattern2**。

例如，给定以下文件:

```shell
# Filename: tstfile

This is a sample file.
This is an ordinary text file.
This file does not contain any unusual text.
This file is not unusual.
Here is some text.
```

现在，让我们在这个文件中搜索*同时*包含"file"和"text"的行. . .

```shell
bash$ grep file tstfile
# Filename: tstfile
 This is a sample file.
 This is an ordinary text file.
 This file does not contain any unusual text.
 This file is not unusual.

bash$ grep file tstfile | grep text
This is an ordinary text file.
 This file does not contain any unusual text.
```

现在，让我们来玩一下*grep*吧~~

**样例 16-18. 纵横字谜解算器**

```shell
#!/bin/bash
# cw-solver.sh
# 这脚本实际上就是围绕一条命令的包装而已（46行）.

#  填字游戏和anagramming文字游戏求解器。
#  你知道你要找的单词中的 *一些* 字母，
#  所以你需要一个列表，
#  列出在给定的位置上所有有效的单词和已知的字母。
#          例如: w...i....n
#               1???5????10
# w在位置1, 3个未知字母, i在位置5, 4个未知字母, n在末尾。
# (请参阅脚本末尾的注释。)


E_NOPATT=71
DICT=/usr/share/dict/word.lst
#                    ^^^^^^^^   在这里寻找单词列表。
#  ASCII单词列表，一行一个字母。
#  如果你碰巧需要这么一个单词列表，
#  请下载作者的 “yawl” 单词列表包。
#  http://ibiblio.org/pub/Linux/libs/yawl-0.3.2.tar.gz
#  或者
#  http://bash.deta.in/yawl-0.3.2.tar.gz


if [ -z "$1" ]   #  如果没有单词模式指定为命令行参数 . . .
then             
  echo           #+ . . . 那么 . . .
  echo "Usage:"  #+ 打印使用信息。
  echo
  echo ""$0" \"pattern,\""
  echo "where \"pattern\" is in the form"
  echo "xxx..x.x..."
  echo
  echo "The x's represent known letters,"
  echo "and the periods are unknown letters (blanks)."
  echo "Letters and periods can be in any position."
  echo "For example, try:   sh cw-solver.sh w...i....n"
  echo
  exit $E_NOPATT
fi

echo
# ===============================================
# 这是所有工作完成的地方。
grep ^"$1"$ "$DICT"   # 没错，就一行代码！
#    |    |
# ^ 匹配输入字符串的开始位置。
# $ 匹配输入字符串的结束位置。

#  摘自 _Stupid Grep Tricks_, vol. 1,
#  《ABS指南》(本书)的作者会再去里面找找看还有什么好玩的
#   . . . 可能就这几天吧 . . .
# ===============================================
echo


exit $?  # 脚本到此停止执行。
# 如果生成的单词太多，
# 请将输出重定向到文件。

$ sh cw-solver.sh w...i....n

wellington
workingman
workingmen
```

**egrep** -- *扩展的grep* -- 与**grep -E**相同。这用起来有一点不一样，它支持扩展的[正则表达式](https://tldp.org/LDP/abs/html/regexp.html#REGEXREF)，令搜索更加地灵活。它同时也支持布尔 | (*or*)运算符。

```shell
bash $ egrep 'matches|Matches' file.txt
Line 1 matches.
 Line 3 Matches.
 Line 4 contains matches, but also Matches
```

**fgrep** -- *快速的grep* -- 与**grep -F**相同。它只做文字字符串搜索（没有[正则表达式](https://tldp.org/LDP/abs/html/regexp.html#REGEXREF)），通常会加快一点速度。

> ![note](http://tldp.org/LDP/abs/images/note.gif)在某些Linux发行版上，**egrep**和**fgrep**是**grep**的符号链接或者别名(aliase)，但分别使用`-E`和`-F`选项调用。

**样例 16-19. 在*Webster 1913字典*中查找定义**

```shell
#!/bin/bash
# dict-lookup.sh

#  此脚本在1913 Webster的词典中查找定义.
#  该公共领域词典可从多个站点下载，
#  比如Gutenberg项目(http://www.gutenberg.org/etext/247)。
#
#  在使用该脚本之前，
#  请将其从DOS格式转换成UNIX格式(行末尾只有LF)
#  将文件存储在普通的、未压缩的ASCII文本中。
#  将下面的DEFAULT_DICTFILE变量设置为路径/文件名。


E_BADARGS=85
MAXCONTEXTLINES=50                        # 要显示的最大行数。
DEFAULT_DICTFILE="/usr/share/dict/webster1913-dict.txt"
                                          # 默认的字典文件路径。
                                          # 有必要的话请重新设置。
#  注意:
#  ----
#  该特定版本的1913 Webster字典每个单词以大写字母开头
#  (单词剩下的部分为小写字母)。
#  只有条目的 *第一行* 以这种方式开始，
#  这就是下面的搜索算法起作用的原因。



if [[ -z $(echo "$1" | sed -n '/^[A-Z]/p') ]]
#  必须至少要指定要查找的单词，并且
#  它必须以大写字母开头。
then
  echo "Usage: `basename $0` Word-to-define [dictionary-file]"
  echo
  echo "Note: Word to look up must start with capital letter,"
  echo "with the rest of the word in lowercase."
  echo "--------------------------------------------"
  echo "Examples: Abandon, Dictionary, Marking, etc."
  exit $E_BADARGS
fi


if [ -z "$2" ]                            #  可以指定不同的字典
                                          #  来作为此脚本的参数。
then
  dictfile=$DEFAULT_DICTFILE
else
  dictfile="$2"
fi

# ---------------------------------------------------------
Definition=$(fgrep -A $MAXCONTEXTLINES "$1 \\" "$dictfile")
#                        展现形式定义为 "Word \..."
#
#  而且，是的，“fgrep” 足够快，甚至可以搜索非常大的文本文件。


# 现在，裁剪定义块。

echo "$Definition" |
sed -n '1,/^[A-Z]/p' |
#  从输出的第一行打印到下一个条目的第一行。

sed '$d' | sed '$d'
#  删除最后两行输出
#  (下一个条目的空行和第一行)。
# ---------------------------------------------------------

exit $?

# 练习:
# ---------
# 1)  修改脚本以接受任何类型的字母输入
#     （大写、小写、混合格式），并且将其转换为可接受的格式进行处理。
#
# 2)  将脚本转换为GUI应用程序，
#     使用诸如'gdialog'或'zenity'等工具 . . .
#     脚本将不再从命令行中获取参数。
#
# 3)  修改脚本以解析其他可用的公共领域词典之一，
#     例如美国人口普查局地名词典(U.S. Census Bureau Gazetteer)。
```

> ![note](http://tldp.org/LDP/abs/images/note.gif)另请参阅[样例 A-41](https://tldp.org/LDP/abs/html/contributed-scripts.html#QKY)，了解如何在大型文本文件上进行快速*fgrep*查找。

**agrep** (*近似的grep*) 将**grep**的能力扩展到近似匹配。搜索字符串可能与结果匹配的字符数不同。此实用程序不是核心Linux发行版的一部分。

{% hint style="info" %}

要搜索压缩文件，请使用**zgrep**、**zegrep**或**zfgrep**命令。这些也适用于非压缩文件，尽管比**grep**、**egrep**、**fgrep**慢。对于搜索一组有些是压缩文件有些不是的混合文件来说很方便。

搜索[bzip](https://tldp.org/LDP/abs/html/filearchiv.html#BZIPREF)压缩文件，请使用**bzgrep**。

{% endhint %}

### look

**look**命令像**grep**，但是在 “字典”，即已排序的单词列表上进行查找。默认情况下，**look**在/`usr/dict/words`中搜索匹配项，但也有可能会指定其他字典文件。

**样例 16-20. 检查列表中的单词是否有效**

```shell
#!/bin/bash
# lookup: 对数据文件中的每个单词进行字典查找。

file=words.data  # 将从中读取测试单词的数据文件。

echo
echo "Testing file $file"
echo

while [ "$word" != end ]  # 数据文件中的最后一个单词。
do               # ^^^
  read word      # 从数据文件中read，因为循环结束时的重定向。
  look $word > /dev/null  # 不想显示字典文件中的行。
  #  在文件/usr/share/dict/words中寻找单词
  #  (经常链接linux.words)。
  lookup=$?      # 'look'命令的退出状态。

  if [ "$lookup" -eq 0 ]
  then
    echo "\"$word\" is valid."
  else
    echo "\"$word\" is invalid."
  fi  

done <"$file"    # 重定向标准输入(stdin)到$file，所以'read'命令从这读取。

echo

exit 0

# ----------------------------------------------------------------
# 由于上面的"exit"命令，行以下的代码将无法执行。


# Stephane Chazelas提出了以下更简洁的替代方案:

while read word && [[ $word != end ]]
do if look "$word" > /dev/null
   then echo "\"$word\" is valid."
   else echo "\"$word\" is invalid."
   fi
done <"$file"

exit 0
```

### sed,awk

特别适合解析文本文件和命令输出的脚本语言。可以单独嵌入，也可以组合嵌入管道(pipe)和shell脚本中。

### [sed](https://tldp.org/LDP/abs/html/sedawk.html#SEDREF)

非交互式 “流编辑器”，允许在[批处理](https://tldp.org/LDP/abs/html/timedate.html#BATCHPROCREF)模式下使用许多**ex**命令。它在shell脚本中有许多用途。

### [awk](https://tldp.org/LDP/abs/html/awk.html#AWKREF)

可编程文件提取器和格式化处理器，适用于处理和/或提取结构化文本文件中的[字段](https://tldp.org/LDP/abs/html/special-chars.html#FIELDREF) (列)。它的语法类似于C语言。

### wc

*wc*命令对一个文件或者I/O流进行“词汇统计”：

```shell
bash $ wc /usr/share/doc/sed-4.1.2/README
13  70  447 README
[13 lines  70 words  447 characters]
```

`wc -w`仅给出词汇统计。

`wc -l`仅给出行数统计。

`wc -c`仅给出字节数统计。

`wc -m`仅给出字符数统计。

`wc -L`仅给出最长行的长度。

使用**wc**命令统计当前工作目录下有多少`.txt`文件：

```shell
$ ls *.txt | wc -l
#  只要 “*.txt” 文件的名称中没有嵌入换行符，就可以工作。

#  以下为其他的等效方案：
#      find . -maxdepth 1 -name \*.txt -print0 | grep -cz .
#      (shopt -s nullglob; set -- *.txt; echo $#)

#  感谢S.C.
```

使用`wc`统计所有文件名以d - h范围内的字母开头的文件大小

```shell
bash$ wc [d-h]* | grep total | awk '{print $3}'
71832
```

使用**wc**统计本书主源文件中"Linux"一词的数量。

```shell
bash$ grep Linux abs-book.sgml | wc -l
138
```

你也可以参阅[样例 16-39](https://tldp.org/LDP/abs/html/filearchiv.html#EX52)和[样例 20-8](https://tldp.org/LDP/abs/html/redircb.html#REDIR4)。

某些命令会使用**wc**的一些功能作为选项。

```shell
... | grep foo | wc -l
# 这类经常使用的结构可以更简洁地呈现。

... | grep -c foo
# 只需使用grep命令的 "-c" (或"--count") 选项即可。

# 感谢S.C.
```

### tr

字符翻译过滤器。

> ![note](https://tldp.org/LDP/abs/images/caution.gif)[必须酌情使用引号和/或括号](https://tldp.org/LDP/abs/html/special-chars.html#UCREF)。引号阻止shell重新解释**tr**命令序列中的特殊字符。括号应加上，以防止shell的扩展行为。

<code>**tr "A-Z" "*" <filename**</code>以及<code>**tr A-Z * \<filename**</code>两者都会将`filename`中的大写字母更改为星号（写入`标准输出(stdout)`）。在一些系统上，这可能会失效，但是<code>**tr A-Z '[\*\*]'**</code>仍然有效。

`-d`选项会删除范围内的字符。

```shell
echo "abcdef"                 # abcdef
echo "abcdef" | tr -d b-d     # aef


tr -d 0-9 <filename
# 删除文件"filename"中所有数字。
```

`--squeeze-repeats`(或者`-s`)选项删除除了第一个连续字符串的所有实例。这个选项在移除多余的[空格](https://tldp.org/LDP/abs/html/special-chars.html#WHITESPACEREF)时很有用。

```shell
bash$ echo "XXXXX" | tr --squeeze-repeats 'X'
X
```

`-c`“补码”选项将字符集*反转*来进行匹配。当使用该选项时，**tr**命令仅作用于与指定集*不*匹配的字符。

```shell
bash$ echo "acfdeb123" | tr -c b-d +
+c+d+b++++
```

请注意，**tr**命令仅识别[POSIX字符集](https://tldp.org/LDP/abs/html/x17129.html#POSIXREF)。[[1]](https://tldp.org/LDP/abs/html/textproc.html#FTN.AEN11502)

```shell
bash$ echo "abcd2ef1" | tr '[:alpha:]' -
----2--1*
```

**样例 16-21. *toupper*：将文本全部转换为大写字母**

```shell
#!/bin/bash
# 将文本全部转换为大写字母。

E_BADARGS=85

if [ -z "$1" ]  # 命令行参数的标准检查。
then
  echo "Usage: `basename $0` filename"
  exit $E_BADARGS
fi  

tr a-z A-Z <"$1"

# 效果同上，但使用POSIX字符集表示法:
#        tr '[:lower:]' '[:upper:]' <"$1"
# 感谢S.C.

#     甚至 . . .
#     cat "$1" | tr a-z A-Z
#     或其他更多的方式 . . .

exit 0

#  练习:
#  重写该脚本，提供将文件 *既* 可以转换为大写 *又*可以转换为小写的选项。
#  提示：你可以使用"select"或"case"命令。
```

**样例 16-22. lowercase: 将工作目录中所有文件名转为小写**

```shell
#!/bin/bash
#
#  将工作目录中所有文件名转为小写。
#
#  受到John Dubois所编写的脚本的启发，
#  并被Chet Ramey翻译成Bash脚本，
#  且由本书作者进行了相当大的简化。


for filename in *                # 遍历目录下所有文件。
do
   fname=`basename $filename`
   n=`echo $fname | tr A-Z a-z`  # 将名称转为小写。
   if [ "$fname" != "$n" ]       # 仅重命名名称不是小写的文件。
   then
     mv $fname $n
   fi  
done   

exit $?


# 该行以下代码不会被执行因为"exit"命令。
#--------------------------------------------------------#
# 若想执行以下命令，请删除本脚本以上的代码。

# 上面的脚本对包含空格或换行符的文件名不起作用。
# 因此Stephane Chazelas therefore建议使用如下的替换方案：


for filename in *    # 没有必要使用basename命令，
                     # 因为 "*" 不会返回任何包含 "/" 的文件。
do n=`echo "$filename/" | tr '[:upper:]' '[:lower:]'`
#                             POSIX字符集表示方法。
#                    添加了斜线，因此不会通过命令替换删除尾随换行符。
   # 变量替换：
   n=${n%/}          # 从文件名中删除上面添加的拖尾斜杠。
   [[ $filename == $n ]] || mv "$filename" "$n"
                     # 检查文件名是不是本来就是小写。
done

exit $?
```

**样例 16-23. *du*：将DOS文本文件转换为UNIX文本文件**

```shell
#!/bin/bash
# Du.sh: DOS转UNIX文本文件转化器。

E_WRONGARGS=85

if [ -z "$1" ]
then
  echo "Usage: `basename $0` filename-to-convert"
  exit $E_WRONGARGS
fi

NEWFILENAME=$1.unx

CR='\015'  # 回车。
           # 015是CR的八进制ASCII码。
           # DOS文本文件中的行以CR-LF结尾。
           # UNIX文本文件中的行仅以LF结尾。

tr -d $CR < $1 > $NEWFILENAME
# 删除CR并将其写入一个新的文件。

echo "Original DOS text file is \"$1\"."
echo "Converted UNIX text file is \"$NEWFILENAME\"."

exit 0

# 练习：
# --------
# 将上面的脚本改为从UNIX转换为DOS。
```

**样例 16-24. *rot13*：超弱加密**

```shell
#!/bin/bash
# crypto-quote.sh: 加密名言

#  使用简单的单表替换算法加密名言。
#  结果类似于在周日报纸开头、结尾板块看到的"Crypto Quote"数独一样。

key=ETAOINSHRDLUBCFGJMQPVWZYXK
# “键” 只不过是一个扰动的字母表。
# 改变“键”即改变了加密方法。

# 'cat “$@”构造从标准输入(stdin)或文件获取输入。
# 如果使用标准输入(stdin)，需要键入Control-D来终止输入。
# 否则，指定文件名作为命令行参数。

cat "$@" | tr "a-z" "A-Z" | tr "A-Z" "$key"
#        |  转为大写       |     加密       
# 对于小写、大写和混合格式的名言均有效。
# 非字母的字符在传递时不发生改变。


# 你可以使用类似如下的输入来尝试本脚本：
# "Nothing so needs reforming as other people's habits."
# --Mark Twain
#
# 输入如下：
# "CFPHRCS QF CIIOQ MINFMBRCS EQ FPHIM GIFGUI'Q HETRPQ."
# --BEML PZERC

# 要反转加密，请执行以下操作:
# cat "$@" | tr "$key" "A-Z"


#  一个平均12岁的孩子只用铅笔和纸就可以解出
#  这个头脑简单的密码。

exit 0

#  练习：
#  --------
#  修改这个脚本使得它既能加密和解密，
#  取决于其命令行参数。
```

当然，*tr*会使其*代码混淆(code obfuscation)*。

```shell
#!/bin/bash
# jabh.sh

x="wftedskaebjgdBstbdbsmnjgz"
echo $x | tr "a-z" 'oh, turtleneck Phrase Jar!'

# 摘自百科全书《Just another Perl hacker》段落。
```

{% hint style="info" %}

***tr*变体**

**tr**实用工具有两个历史性变体。BSD版本不使用引号(`tr a-z A-Z`)，但是SysV版本使用(`tr '[a-z]' '[A-Z]'`)。**tr**的GNU版本类似于BSD版本。

{% endhint %}

### fold

将输入包装为每行为指定宽度的过滤器。`-s`选项特别有用，该选项在单词空格处中断行 (请参阅[样例 16-26](https://tldp.org/LDP/abs/html/textproc.html#EX50)和[样例 A-1](https://tldp.org/LDP/abs/html/contributed-scripts.html#MAILFORMAT))。

### fmt

简单的文件格式化程序，用作管道中的过滤器，以 “包装” 长文本输出。

**样例 16-26. 格式化文件列表**

```shell
#!/bin/bash

WIDTH=40                    # 40列宽。

b=`ls /usr/local/bin`       # 得到一个文件列表...

echo $b | fmt -w $WIDTH

# 也可以这样操作
#    echo $b | fold - -s -w $WIDTH

exit 0
```

你也可以参阅[样例 16-5](https://tldp.org/LDP/abs/html/moreadv.html#EX41)。

{% hint style="info" %}

Kamil Toman的**par**实用工具是**fmt**的强大替换品，你可以从[http://www.cs.berkeley.edu/~amc/Par/](http://www.cs.berkeley.edu/~amc/Par/)进行下载。

{% endhint %}

### col

这个命名令人困惑的过滤器从输入流中删除反向换行符。它还会尝试用等效的tab替换空格。**col**的主要用途是过滤某些文本处理实用程序 (例如**groff**和**tbl**) 的输出。

### column

列格式化程序。此过滤器通过在适当的位置插入tab来将列表类型的文本输出转换为 “格式美观” 的表格。

**样例 16-27. 使用*column*命令来格式化一个目录列表**

```shell
#!/bin/bash
# colms.sh
# 对"column"命令man手册中的示例文件进行了较小的修改。


(printf "PERMISSIONS LINKS OWNER GROUP SIZE MONTH DAY HH:MM PROG-NAME\n" \
; ls -l | sed 1d) | column -t
#         ^^^^^^           ^^

#  管道中的"sed 1d"删除了输出的第一行，
#  第一行内容为"total        N",
#  其中"N"是"ls -l"命令所找到文件的总数。

# "column"命令的-t选项打印了一个漂亮的表格。

exit 0
```

### colrm

> ![note](https://tldp.org/LDP/abs/images/caution.gif)如果文件包含tab或不可打印的字符，这可能会导致不可预测的行为。在这种情况下，请考虑在管道(pipe)中**colrm**命令之前使用[expand](https://tldp.org/LDP/abs/html/textproc.html#EXPANDREF)命令或**unexpand**命令。

### nl

行号筛选器：<code>**nl filename**</code>将`filename`输出到`标准输出(stdout)`，但在每个非空白行的开头插入连续的数字。如果省略文件名，则对`标准输入(stdin)`进行操作。

`nl`命令的输出与`cat -b`非常类似，因为默认情况下`nl`命令不列出空白行。

**样例 16-28. *nl*：一个自编号脚本**

```shell
#!/bin/bash
# line-number.sh

# 该脚本将自己打印两次到标准输出(stdout)，并带有行号.

echo "     line number = $LINENO" # 'nl'命令会认为这是第四行
#                                   (nl不会标记空白行).
#                                   'cat -n'会准确地显示它是第6行。

nl `basename $0`

echo; echo  # 现在，来试试看'cat -n'

cat -n `basename $0`
# 区别就是'cat -n'会标记空白行。
# 注意'nl -ba'也有相同的效果。

exit 0
# -----------------------------------------------------------------
```

### pr

打印格式过滤器。它会给文件（或`标准输出(stdout)`）划分成适合硬拷贝打印或在屏幕上查看的部分并标页数。有各种选项如行和列操作、连接行、设置边距、编号行、添加页眉和合并文件等。**pr**命令结合了**nl**、**paste**、**column**和**expand**的功能。

<code>**pr -o 5 --width=65 fileZZZ | more**</code>给予`fileZZZ`一个很好的分页视图，边距设置为5和页宽设置为65。

一个特别有用的选项是`-d`，强制双倍行距 (效果与**sed -G**相同)。

### gettext

GNU **gettext**软件包是一组实用程序，用于将程序的文本输出[本地化](https://tldp.org/LDP/abs/html/localization.html)并翻译成外语。虽然最初用于C程序，但现在支持许多编程和脚本语言。
<strong>gettext</strong><em>程序</em>在shell脚本上运行。请参阅`info手册`。

### msgfmt

用于生成二进制消息目录(binary message catalog)的程序。它用于[本地化](https://tldp.org/LDP/abs/html/localization.html)。

### iconv

用于将文件转换为不同编码 (字符集) 的实用程序。它主要用于[本地化](https://tldp.org/LDP/abs/html/localization.html)。

```shell
# 将字符串由UTF-8转为UTF-16并且将它传递给书单
function write_utf8_string {
    STRING=$1
    BOOKLIST=$2
    echo -n "$STRING" | iconv -f UTF8 -t UTF16 | \
    cut -b 3- | tr -d \\n >> "$BOOKLIST"
}

#  摘自Peter Knowles'的"booklistgen.sh"脚本
#  将文件转为Sony Librie/PRS-50X格式。
#  (http://booklistgensh.peterknowles.com)
```

### recode

请将其视为上述**iconv**命令的更高级版本。这个非常通用的实用程序，用于将文件转换为不同的编码方案。请注意，*recode*不是标准Linux安装的一部分。

### TeX, gs

**TeX**和**Postscript**是用于准备打印或格式化视频显示器拷贝件的文本标记语言。

TeX是Donald Knuth精心设计的文本格式系统。使用其中之一标记语言编写一个shell脚本通常很方便，因为所有选项和参数已传递并得到了封装。

<strong>Ghostscript</strong> (<em>gs</em>) 是GPL-ed Postscript解释器。

### texexec

用于处理*TeX*和*pdf*文件的实用程序。可以在许多Linux发行版的`/usr/bin`目录下中找到，它实际上是一个[shell包装器](https://tldp.org/LDP/abs/html/wrapper.html#SHWRAPPER)，它调用[Perl](https://tldp.org/LDP/abs/html/wrapper.html#PERLREF)来调用*Tex*。

```shell
texexec --pdfarrange --result=Concatenated.pdf *pdf

#  将当前工作目录中的所有pdf文件
#  合并到文件Concatenated.pdf中. . .
#  (--pdfarrange选项会重新编号pdf文件。 你也可以参阅 --pdfcombine。)
#  可以对上述命令行进行参数化并放入shell脚本中。
```

### enscript

将纯文本文件转化为Postscript的实用工具。

例如，**enscript filename.txt -p filename.ps**会产生一个Postscript输出文件`filename.ps`

### groff, tbl, eqn

另一种文本标记和格式化显示语言是**groff**。这是古早的UNIX **roff/troff**显示和排版包的增强GNU版本。[man手册](https://tldp.org/LDP/abs/html/basic.html#MANREF)使用**groff**。

**tbl**表处理工具被认为是**groff**的一部分，因为它的功能是将表标记(table markup)转换为**groff**命令。

**eqn**公式处理工具同样是**groff**的一部分，其功能是将方程标记(equation markup)转换为**groff**命令。

**样例 16-29. *manview*：查看格式化的man手册**

```shell
#!/bin/bash
# manview.sh: 格式化man页面的源以供查看。

#  当撰写man页面源时这个脚本比较有用。
#  它可以让你在工作的同时即时查看中间结果。

E_WRONGARGS=85

if [ -z "$1" ]
then
  echo "Usage: `basename $0` filename"
  exit $E_WRONGARGS
fi

# ---------------------------
groff -Tascii -man $1 | less
# 来自groff的man页面。
# ---------------------------

#  如果操作页面包含表格和/或方程式，
#  则上述代码将显示。
#  下一行同样可以处理这项事情。
#
#   gtbl < "$1" | geqn -Tlatin1 | groff -Tlatin1 -mtty-char -man
#
#   感谢S.C.

exit $?   # 你也可以参阅"maned.sh"脚本。
```

你也可以参阅[样例 A-39](https://tldp.org/LDP/abs/html/contributed-scripts.html#MANED)。

### lex, yacc

**lex**词法分析器产生用于模式匹配的程序。这已被Linux系统上的非自有程序**flex**所取代。

**yacc**实用程序基于一组规范创建解析器。这已被Linux系统上的非自有程序**bison**所取代。

## 注记

[[1]](https://tldp.org/LDP/abs/html/textproc.html#AEN11502)这仅适用于**tr**的GNU版本，而不是商业UNIX系统上经常安装的通用版本。
