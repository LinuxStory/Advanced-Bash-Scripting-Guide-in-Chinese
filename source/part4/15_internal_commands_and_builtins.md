# 第十五章 内建命令

内建命令 `builtin` 是包含在 Bash 工具集中的命令，又写作 `built in`。使用内建命令通常出于性能原因或是需要直接存取 shell 内部变量的考量。内建命令执行速度比外部命令的执行速度快，因为外部命令通常需要派生[^1]出一个单独的进程执行。

--- 

命令或 shell 本身启动或生成一个子进程用于执行任务的操作被称为派生 `forking`。新生成的进程被称作子进程，而派生出子进程的进程被称作父进程。当子进程在执行任务时，父进程也仍在运行。

需要注意的是，父进程可以获取子进程的进程ID，并能传递参数给子进程，但反之则不行。[该机制会产生一些难以捉摸的问题。](https://tldp.org/LDP/abs/html/gotchas.html#PARCHILDPROBREF)

**样例 15-1. 脚本生成多个自身实例**

```bash
#!/bin/bash
# spawn.sh


PIDS=$(pidof sh $0)  # 脚本的多个实例的进程ID。
P_array=( $PIDS )    # 将进程ID放置到数组中（为什么？）。
echo $PIDS           # 显示父进程和子进程的进程ID。
let "instances = $(#P_array[*]} - 1"  # 元素数量减1。
                                      # 为什么要减1？
echo "$instances instance(s) of this script running."
echo "[Hit Ctl-C to exit.]"; echo


sleep 1              # 闲置等待。
sh $0                # 再运行一次。

exit 0               # 这不是必须的；脚本永远不会执行到这里。
                     # 为什么不是必须的？

#  在键入 Ctl-C 退出脚本后，
#+ 是否所有生成的脚本实例都会终止？
#  如果是，为什么？

# 注意：
# ----
# 注意不要长时间运行这个脚本。
# 它最终会占用大量的系统资源。

#  你认为让脚本生成大量的自身实例
#+ 是否是一个可取的编写脚本的技巧？
#  为什么？
```

通常来说，Bash 的内建命令不会在脚本中通过派生子进程来执行。而在脚本中调用外部系统命令或筛选器通常需要派生子进程。

--- 

一些内建命令可能与系统命令重名，但是这些都是在 Bash 内部重新实现后的命令。例如 Bash 中的 `echo` 命令与系统命令 `/bin/echo` 的功能基本一致，但是它们本质上并不一样。

```bash
#!/bin/bash

echo "This line uses the \"echo\" builtin."
/bin/echo "This line uses the /bin/echo system command."
```

关键词 `keyword` 是保留使用的词汇、标记或运算符。关键词在 shell 中具有特殊意义，是 shell 语法的组成部分。例如 `for`，`while`，`do` 和 `!` 都是关键词。关键词与 [内建命令](https://tldp.org/LDP/abs/html/internal.html#BUILTINREF) 相似，它们都是硬编码到 Bash 中的。但是关键词本身不是命令，而是构成命令的子单位[[2]](https://tldp.org/LDP/abs/html/internal.html#FTN.AEN8650)。

## 输入与输出 I/O

### echo

打印一个表达式或变量到标准输出 `stdout`（参考 [样例 4-1]()）。

```bash
echo Hello
echo $a
```

`echo` 命令需要 `-e` 选项来打印转义字符。参考 [样例 5-2]()。

通常情况下，每一个 `echo` 命令都会在最后打印一个终端换行符，但使用 `-n` 选项可以禁止这个行为。

{% hint style="info" %}

`echo` 可通过管道被用于给一系列命令提供值。

```bash
if echo "$VAR" | grep -q txt   # if [[ $VAR = *txt* ]]
then
  echo "$VAR contains the substring sequence \"txt\""
fi
```

{% endhint %}

{% hint style="info" %}

`echo` 与 [命令替换]() 结合可以用于给变量赋值。

``a=`echo "HELLO" | tr A-Z a-z` ``

参考 [样例 16-22]()，[样例 16-3]()，[样例 16-47]() 和 [样例 16-48]()。

{% endhint %}

需要注意的是 ``echo `command``` 会删除 `command` 中生成的所有换行符。

[\$IFS](https://tldp.org/LDP/abs/html/internalvariables.html#IFSREF)（内部字段分隔符）变量通常包含 \n（换行符） 作为其[空白](https://tldp.org/LDP/abs/html/special-chars.html#WHITESPACEREF)字符集之一。因此，Bash将换行符把*command*的输出分割为**echo**命令参数。然后**echo**以空格分割，输出这些参数。

```shell
bash$ ls -l /usr/share/apps/kjezz/sounds
-rw-r--r--    1 root     root         1407 Nov  7  2000 reflect.au
 -rw-r--r--    1 root     root          362 Nov  7  2000 seconds.au




bash$ echo `ls -l /usr/share/apps/kjezz/sounds`
total 40 -rw-r--r-- 1 root root 716 Nov 7 2000 reflect.au -rw-r--r-- 1 root root ...
```

那么，我们如何在[echo输出](https://tldp.org/LDP/abs/html/internal.html#ECHOREF)的字符串中嵌入换行符呢？

```shell
# 嵌入一个换行符？
echo "Why doesn't this string \n split on two lines?"
# 不会分割。

# 让我们尝试一些其他的。

echo

echo $"A line of text containing
a linefeed."
# 打印不同的两行(嵌入换行符)。
# 但是，变量前缀的"$"符号真的必需吗？

echo

echo "This string splits
on two lines."
# 其实，并不需要"$"。
echo
echo "---------------"
echo

echo -n $"Another line of text containing
a linefeed."
# 打印不同的两行(嵌入换行符)。
# 即便是 -n 参数也无法取消这里的换行符。

echo
echo
echo "---------------"
echo
echo

# 然而，以下这行并没有实现预期的效果。
# 为什么？提示：赋值给一个变量。
string1=$"Yet another line of text containing
a linefeed (maybe)."

echo $string1
# 又一行包含换行的文本(也许)。
#      ^^^
# 换行符变成了一个空格。

# 感谢Steve Parker指出。
```

> ![note](https://tldp.org/LDP/abs/images/note.gif)该命令是一个shell内建程序，尽管二者行为类似，但是与`/bin/echo`不同。
> 
> ```shell
> bash$ type -a echo
> echo is a shell builtin
>  echo is /bin/echo
> ```

### printf

**printf**，即格式化打印，该命令是增强版的**echo**。它是*C*语言`printf()`库函数的有限变体，并且语法有些不同。

**printf** *format-string*... *parameter*...

这是`/bin/printf`或`/usr/bin/printf`命令的Bash内置版本。请参见（系统命令的）**printf** [man手册](https://tldp.org/LDP/abs/html/basic.html#MANREF)以获得更深入的内容。

> ![note](https://tldp.org/LDP/abs/images/caution.gif)老版本的Bash可能不支持**printf**。

**样例 15-2. *printf*在起作用**

```shell
#!/bin/bash
# printf示例。

declare -r PI=3.14159265358979     # 只读变量，即常量。
declare -r DecimalConstant=31373

Message1="Greetings,"
Message2="Earthling."

echo

printf "Pi to 2 decimal places = %1.2f" $PI
echo
printf "Pi to 9 decimal places = %1.9f" $PI  # 它甚至准确地舍入了。

printf "\n"                                  # 打印一个换行符，
                                             # 等价于'echo' . . .

printf "Constant = \t%d\n" $DecimalConstant  # 插入一个缩进符(\t)。

printf "%s %s \n" $Message1 $Message2

echo

# ==========================================#
# C函数的模拟，sprint()。
# 加载带有格式化字符串的变量。

echo 

Pi12=$(printf "%1.12f" $PI)
echo "Pi to 12 decimal places = $Pi12"      # 舍入错误！

Msg=`printf "%s %s \n" $Message1 $Message2`
echo $Msg; echo $Msg

#  碰巧的是，"sprintf"函数现在可以
#  当做Bash的可加载模块进行访问，
#  但这不可移植。

exit 0
```

**printf**的一个实用场景是格式化错误信息。

```shell
E_BADDIR=85

var=nonexistent_directory

error()
{
  printf "$@" >&2
  # 格式化所传递的位置参数，并将其发送到标准错误(stderr)。
  echo
  exit $E_BADDIR
}

cd $var || error $"Can't cd to %s." "$var"

# 感谢S.C.
```

另请参阅[样例 36-17](https://tldp.org/LDP/abs/html/assortedtips.html#PROGRESSBAR)。

### read

从`标准输入(stdin)`“读取”变量值，即以交互式获取键盘的输入。`-a`参数可以使**read**可以获取数组变量（参见[样例 27-6](https://tldp.org/LDP/abs/html/arrays.html#EX67)）。

**样例 15-3. 变量赋值，使用*read***

```shell
#!/bin/bash
# “读取”变量。

echo -n "Enter the value of variable 'var1': "
# echo命令的-n参数用于防止换行。

read var1
# 注意在var1前没有'$'符号，因为它需要被设置。

echo "var1 = $var1"


echo

# 单个 'read' 语句可以设置多个变量。
echo -n "Enter the values of variables 'var2' and 'var3' "
echo -n "(separated by a space or tab): "
read var2 var3
echo "var2 = $var2      var3 = $var3"
#  如果你仅输入一个值，
#  那么其他变量将仍保持未设置的状态(null)。

exit 0
```

当**read**命令没有相关联的变量接收输入时，它将把输入传递给指定的变量[\$REPLY](https://tldp.org/LDP/abs/html/internalvariables.html#REPLYREF)。

**样例 15-4. 当*read*没有变量时会发生什么**

```shell
#!/bin/bash
# read-novar.sh

echo

# -------------------------- #
echo -n "Enter a value: "
read var
echo "\"var\" = "$var""
# 这里所有都在预期之中。
# -------------------------- #

echo

# ------------------------------------------------------------------- #
echo -n "Enter another value: "
read           #  没有提供给read的变量，那么...
               #  输入给'read'的内容赋值给默认变量，$REPLY。
var="$REPLY"
echo "\"var\" = "$var""
# 这等价于第一个代码块。
# ------------------------------------------------------------------- #

echo
echo "========================="
echo


#  但是，这表明即使以常规方式 “读取” 变量后，
#  $REPLY也可用。

# ================================================================= #

#  在一些情况下，你可能希望放弃第一次所读取的值。
#  在这种情况下，简单地忽略$REPLY变量即可。

{ # 代码块。
read            # 第一行，需要被丢弃。
read line2      # 第二行，存入变量中。
  } <$0
echo "Line 2 of this script is:"
echo "$line2"   #   # read-novar.sh
echo            #   #!/bin/bash  一行被丢弃。

# 另请参阅soundcard-on.sh脚本。

exit 0
```

通常，输入 \ 会忽略输入**read**的换行符。`-r`选项会使输入的 \ 进行字面转义。

**样例 15-5. 多行输入至*read***

```shell
#!/bin/bash

echo

echo "Enter a string terminated by a \\, then press <ENTER>."
echo "Then, enter a second string (no \\ this time), and again press <ENTER>."

read var1     # 当读取$var1时，"\"忽略换行符。
              #     first line \
              #     second line

echo "var1 = $var1"
#     var1 = first line second line

#  对于以 “\” 结尾的每一行，
#  你都会在下一行出现提示，以继续将字符输入到var1中。

echo; echo

echo "Enter another string terminated by a \\ , then press <ENTER>."
read -r var2  # -r选项会使 "\" 以字面读取。
              #     first line \

echo "var2 = $var2"
#     var2 = first line \

# 数据输入以第一个 <ENTER> 终止。

echo 

exit 0
```

**read**命令有一些有趣的选项，允许在不敲击**ENTER**键的情况下输出提示甚至读取击键。

```shell
# 在不敲击ENTER的情况下读取击键。

read -s -n1 -p "Hit a key " keypress
echo; echo "Keypress was "\"$keypress\""."

# -s 选项表示不要输出输入。
# -n N 选项表示仅接受N个字符输入。
# -p 选项表示在读取输入之前输出以下提示。

# 使用这些选项比较棘手，因为它们需要按照正确的顺序。
```

**read**的`-n`选项同样支持检测**箭头键**以及其他某些不常用的键。

**样例 15-6. 检测箭头键**

```shell
#!/bin/bash
# arrow-detect.sh: 检测到箭头键，以及其他键。
# 感谢Sandro Magi告诉我怎么实现。

# --------------------------------------------
# 按键生成的字符代码。
arrowup='\[A'
arrowdown='\[B'
arrowrt='\[C'
arrowleft='\[D'
insert='\[2'
delete='\[3'
# --------------------------------------------

SUCCESS=0
OTHER=65

echo -n "Press a key...  "
# 如果按下了上面未列出的键，可能还需要按ENTER键。
read -n3 key                      # 读取三个字符。

echo -n "$key" | grep "$arrowup"  #检查是否检测到字符代码。
if [ "$?" -eq $SUCCESS ]
then
  echo "Up-arrow key pressed."
  exit $SUCCESS
fi

echo -n "$key" | grep "$arrowdown"
if [ "$?" -eq $SUCCESS ]
then
  echo "Down-arrow key pressed."
  exit $SUCCESS
fi

echo -n "$key" | grep "$arrowrt"
if [ "$?" -eq $SUCCESS ]
then
  echo "Right-arrow key pressed."
  exit $SUCCESS
fi

echo -n "$key" | grep "$arrowleft"
if [ "$?" -eq $SUCCESS ]
then
  echo "Left-arrow key pressed."
  exit $SUCCESS
fi

echo -n "$key" | grep "$insert"
if [ "$?" -eq $SUCCESS ]
then
  echo "\"Insert\" key pressed."
  exit $SUCCESS
fi

echo -n "$key" | grep "$delete"
if [ "$?" -eq $SUCCESS ]
then
  echo "\"Delete\" key pressed."
  exit $SUCCESS
fi


echo " Some other key pressed."

exit $OTHER

# ========================================= #

#  Mark Alexander想出了上述脚本的简化版 (谢谢!)。
#  它消除了对grep的需要。

#!/bin/bash

  uparrow=$'\x1b[A'
  downarrow=$'\x1b[B'
  leftarrow=$'\x1b[D'
  rightarrow=$'\x1b[C'

  read -s -n3 -p "Hit an arrow key: " x

  case "$x" in
  $uparrow)
     echo "You pressed up-arrow"
     ;;
  $downarrow)
     echo "You pressed down-arrow"
     ;;
  $leftarrow)
     echo "You pressed left-arrow"
     ;;
  $rightarrow)
     echo "You pressed right-arrow"
     ;;
  esac

exit $?

# ========================================= #

# Antonio Macchi有一个更简洁的替代方案。

#!/bin/bash

while true
do
  read -sn1 a
  test "$a" == `echo -en "\e"` || continue
  read -sn1 a
  test "$a" == "[" || continue
  read -sn1 a
  case "$a" in
    A)  echo "up";;
    B)  echo "down";;
    C)  echo "right";;
    D)  echo "left";;
  esac
done

# ========================================= #

#  练习:
#  --------
#  1) 添加对"Home"、"End"、"PgUp"和"PgDn"键的检测。
```

> ![note](https://tldp.org/LDP/abs/images/note.gif)**read**的`-n`选项会不检测**ENTER**（新行）键。

**read**的`-t`选项允许定时输入（参阅[样例 9-4](https://tldp.org/LDP/abs/html/internalvariables.html#TOUT)和[样例 A-41](https://tldp.org/LDP/abs/html/contributed-scripts.html#QKY)）。

`-u`选项采用目标文件的[文件描述符](https://tldp.org/LDP/abs/html/io-redirection.html#FDREF)。

**read**命令还可以从[重定向](https://tldp.org/LDP/abs/html/io-redirection.html#IOREDIRREF)到`标准输入(stdin)`的文件中 “读取” 其变量值。如果文件包含多行，则仅将第一行分配给变量。如果**read**具有多个参数，则每个变量都会被分配一个连续的[空格描述](https://tldp.org/LDP/abs/html/special-chars.html#WHITESPACEREF)字符串。请注意！

**样例 15-7. 使用[文件重定向](https://tldp.org/LDP/abs/html/io-redirection.html#IOREDIRREF)的*read***

```shell
#!/bin/bash

read var1 <data-file
echo "var1 = $var1"
# var1设置为输入文件"data-file"的整个第一行

read var2 var3 <data-file
echo "var2 = $var2   var3 = $var3"
# 注意这里"read"的反直觉行为。
# 1) 倒回输入文件的开头。
# 2) 现在每个变量将设置为相应的字符串，
#    以空白符分割，而不是文本的一整行。
# 3) 最后一个变量获取该行的余数。
# 4) 如果要设置的变量比文件第一行的空格终止的字符串多，
#    则多余的变量保持为空。

echo "------------------------------------------------"

# 如何用循环解决上述问题：
while read line
do
  echo "$line"
done <data-file
# 感谢Heiner Steven指出。

echo "------------------------------------------------"

# 如果您不希望默认值为空格，
# 请使用$IFS(内部字段分隔符变量)将输入行拆分后传递给read。

echo "List of all users:"
OIFS=$IFS; IFS=:       # /etc/passwd使用":"作为字段分隔符。
while read name passwd uid gid fullname ignore
do
  echo "$name ($fullname)"
done </etc/passwd   # I/O重定向。
IFS=$OIFS              # 恢复原先的$IFS。
# 这代码片段由Heiner Steven所写。



#  如果将$IFS变量设置在循环体中，
#  则无需将原始$IFS存储在临时变量中。
#  感谢Dim Segebart指出。
echo "------------------------------------------------"
echo "List of all users:"

while IFS=: read name passwd uid gid fullname ignore
do
  echo "$name ($fullname)"
done </etc/passwd   # I/O重定向。

echo
echo "\$IFS still $IFS"

exit 0
```

> ![note](https://tldp.org/LDP/abs/images/note.gif)将输出[管道传输](https://tldp.org/LDP/abs/html/special-chars.html#PIPEREF)给*read*，再使用[echo](https://tldp.org/LDP/abs/html/internal.html#ECHOREF)来设置变量[会失败](https://tldp.org/LDP/abs/html/gotchas.html#BADREAD0)。然而，将输出管道传输给[cat](https://tldp.org/LDP/abs/html/basic.html#CATREF)*看起来*奏效。

```shell
cat file1 file2 |
while read line
do
echo $line
done
```

然而，正如Bjön Eriksson展示的：

**样例 15-8. 从管道中读取发生的问题**

```shell
#!/bin/sh
# readpipe.sh
# 该样例由Bjon Eriksson提供。

### shopt -s lastpipe

last="(null)"
cat $0 |
while read line
do
    echo "{$line}"
    last=$line
done

echo
echo "++++++++++++++++++++++"
printf "\nAll done, last: $last\n" #  如果你取消第5行的注释，
                                   #  那么这行的输出会改变。
                                   #  (Bash，大于等于4.2版本)

exit 0  # 代码的末尾。
        # 脚本的 (部分) 输出如下。
        # 'echo'的输出部分由括号括起来。

#############################################

./readpipe.sh 

{#!/bin/sh}
{last="(null)"}
{cat $0 |}
{while read line}
{do}
{echo "{$line}"}
{last=$line}
{done}
{printf "nAll done, last: $lastn"}


All done, last: (null)

变量 (last) 设置在循环/子shell内，
但其值不会在循环外持续存在。
```

*gendiff*脚本（通常可以在许多Linux发行版上的`/usr/bin`目录下找到）将[find](https://tldp.org/LDP/abs/html/moreadv.html#FINDREFhttps://tldp.org/LDP/abs/html/moreadv.html#FINDREF)的输出通过管道传输到*while read*结构中。

```shell
find $1 \( -name "*$2" -o -name ".*$2" \) -print |
while read f; do
. . .
```

{% hint style="info" %}

可以将文本*粘贴*到*read*的输入字段中（但不能多行！）。请参阅[样例A-38](https://tldp.org/LDP/abs/html/contributed-scripts.html#PADSW)。

{% endhint %}

## 文件系统命令

### cd

熟悉的**cd**跳转目录命令可以在脚本中使用，命令执行后将跳转至指定的目录中。

```shell
(cd /source/directory && tar cf - . ) | (cd /dest/directory && tar xpvf -)
```

[来自Alan Cox所写[先前引用](https://tldp.org/LDP/abs/html/special-chars.html#COXEX)的样例]

**cd**的`-P（物理）`选项用于忽略符号链接。

<strong>cd -</strong>将跳转至[\$OLDPWD](https://tldp.org/LDP/abs/html/internalvariables.html#OLDPWD)，即之前的工作目录。

> ![note](https://tldp.org/LDP/abs/images/caution.gif)当带有两个正斜杠时，**cd**命令将不能正常工作。
> 
> ```shell
> bash$ cd //
> bash$ pwd
> //
> ```
> 
> 当然，输出应该是 / 。这是命令行和脚本都存在的问题。

### pwd

打印工作目录。该命令给出了用户（或者脚本）当前的目录（参阅[样例 15-9](https://tldp.org/LDP/abs/html/internal.html#EX37)）。其效果等同于读取内置变量[$PWD](https://tldp.org/LDP/abs/html/internalvariables.html#PWDREF)的值。

### pushd, popd, dirs

这个命令集是一种为工作目录添加书签的机制，通过一种有序的方式在目录间来回移动。将目录名推入堆栈用于跟踪。该命令选项允许对目录堆栈进行各种操作。

**pushd dir-name**将路径<em>`dir-name`</em>推送到目录堆栈上（堆栈的*顶部*），同时将当前工作目录更改为<em>`dir-name`</em>。

**popd**从目录堆栈中删除(弹出)顶层目录路径名，同时将当前工作目录更改为现在位于堆栈*顶层*的目录。

**dirs**列出目录堆栈的内容（与[$DIRSTACK](https://tldp.org/LDP/abs/html/internalvariables.html#DIRSTACKREF)变量进行比较）。成功执行的**pushd**或**popd**将自动调用**dirs**。

需要对当前工作目录进行各种更改而不硬编码目录名更改的脚本可以很好地利用这些命令。请注意，隐式`$DIRSTACK`数组变量（可从脚本中访问）保存目录堆栈的内容。

**样例 15-9. 更改当前的工作目录**

```shell
#!/bin/bash

dir1=/usr/local
dir2=/var/spool

pushd $dir1
# 将自动执行"dirs"(将目录堆栈列表输出到标准输出)。
echo "Now in directory `pwd`." # 使用反引号括起来的'pwd'。

# 现在，在目录"dir1"中做一些事情。
pushd $dir2
echo "Now in directory `pwd`."

# 现在，在目录"dir2"中做一些事情。
echo "The top entry in the DIRSTACK array is $DIRSTACK."
popd
echo "Now back in directory `pwd`."

# 现在，在目录"dir1"中再做一些事情。
popd
echo "Now back in original working directory `pwd`."

exit 0

# 如果你不执行'popd' -- 然后退出该脚本会发生什么呢？
# 你会处在哪个目录下呢？为什么？
```

## 变量命令

### let

**let**命令用于对变量执行*算术*运算。[[3]](https://tldp.org/LDP/abs/html/internal.html#FTN.AEN9009)在许多情况下，它充当[expr](https://tldp.org/LDP/abs/html/moreadv.html#EXPRREF)的一个简洁版本。

**样例 15-10. 让*let*来做算术运算**

```shell
#!/bin/bash

echo

let a=11            # 等价于'a=11'
let a=a+5           # 等价于let "a = a + 5"
                    # (双引号和空格可以使其更易于阅读。)
echo "11 + 5 = $a"  # 16

let "a <<= 3"       # 等价于let "a = a << 3"
echo "\"\$a\" (=16) left-shifted 3 places = $a"
                    # 128

let "a /= 4"        # 等价于let "a = a / 4"
echo "128 / 4 = $a" # 32

let "a -= 5"        # 等价于let "a = a - 5"
echo "32 - 5 = $a"  # 27

let "a *=  10"      # 等价于let "a = a * 10"
echo "27 * 10 = $a" # 270

let "a %= 8"        # 等价于let "a = a % 8"
echo "270 modulo 8 = $a  (270 / 8 = 33, remainder $a)"
                    # 6


# "let"允许C风格的操作符吗？
# 是的，正如(( ... ))双括号结构可以。

let a++             # C风格(后置)增加。
echo "6++ = $a"     # 6++ = 7
let a--             # C风格减少。
echo "7-- = $a"     # 7-- = 6
# 当然，++a等也是允许的 . . .
echo


# 三元运算符。

# 参见上方代码，$a=6。
let "t = a<7?7:11"   # True
echo $t  # 7

let a++
let "t = a<7?7:11"   # False
echo $t  #     11

exit
```

> ![note](https://tldp.org/LDP/abs/images/caution.gif)在特定的上下文中，*let*命令可以返回一个惊人的[退出状态](https://tldp.org/LDP/abs/html/exit-status.html#EXITSTATUSREF)。

```shell
# Evgeniy Ivanov指出:

var=0
echo $?     # 0
            # 预期中。

let var++
echo $?     # 1
            # 命令执行成功，但为什么不是$?=0 ???
            # 简直不可理喻！

let var++
echo $?     # 0
            # 预期中。


# 同样的 . . .

let var=0
echo $?     # 1
            # 命令执行成功，但为什么不是$?=0 ???

#  然而，正如Jeff Gorak指出，
#  这是"let"设计规范的一部分 . . .
#  "如果最后一个参数值为0，let返回1;
#  否则let返回0。"['help let']
```

### eval

<strong>`eval arg1 [arg2] ... [argN]`</strong>

组合表达式或表达式列表中的参数并对其*求值*。表达式中的任何变量都将被引申含义。最终结果是将**字符串转换成命令**。

{% hint style="info" %}

**eval**命令可用于从命令行或在脚本中生成代码。

{% endhint %}

```shell
bash$ command_string="ps ax"
bash$ process="ps ax"
bash$ eval "$command_string" | grep "$process"
26973 pts/3    R+     0:00 grep --color ps ax
 26974 pts/3    R+     0:00 ps ax
```

每次调用*eval*都会强制对其参数重新*转义*。

```shell
a='$b'
b='$c'
c=d

echo $a             # $b
                    # 第一层。
eval echo $a        # $c
                    # 第二层。
eval eval echo $a   # d
                    # 第三层。

# 感谢E. Choroba.
```

**样例 15-11. 展示*eval*的效果**

```shell
#!/bin/bash
# 练习"eval" ...

y=`eval ls -l`  #  类似于y=`ls -l`
echo $y         #  但是删除了换行符，因为"echo"变量没有被括起来。
echo
echo "$y"       #  当变量被括起来时，换行符仍旧保留。

echo; echo

y=`eval df`     # 类似于y=`df`
echo $y         # 但是删除了换行符。

#  当不保留LF时，可能更加容易来解析输出，
#  可以使用诸如"awk"的实用工具。

echo
echo "==========================================================="
echo

eval "`seq 3 | sed -e 's/.*/echo var&=ABCDEFGHIJ/'`"
# var1=ABCDEFGHIJ
# var2=ABCDEFGHIJ
# var3=ABCDEFGHIJ

echo
echo "==========================================================="
echo


# Now, showing how to do something useful with "eval" . . .
# 现在，展示如何使用"eval"做一些有用的事情 . . .
# (感谢您，E. Choroba!)

version=3.4     #  我们能在一个命令中把版本分成
                #  主版本和小版本吗？
echo "version = $version"
eval major=${version/./;minor=}     #  将version中的'.'替换为';minor='
                                    #  该替换产生了'3; minor=4'
                                    #  所以eval执行的结果为minor=4, major=3
echo Major: $major, minor: $minor   #  Major: 3, minor: 4
```

**样例 15-12. 使用*eval*从变量中进行筛选**

```shell
#!/bin/bash
# arr-choice.sh

#  向函数传递参数来
#  从一组变量中选择一个特定的变量。

arr0=( 10 11 12 13 14 15 )
arr1=( 20 21 22 23 24 25 )
arr2=( 30 31 32 33 34 35 )
#       0  1  2  3  4  5      序号 (从0开始标号)


choose_array ()
{
  eval array_member=\${arr${array_number}[element_number]}
  #                 ^       ^^^^^^^^^^^^
  #  使用eval来构建变量名，
  #  在这个特定的场景中，是数组名。

  echo "Element $element_number of array $array_number is $array_member"
} #  可以重写函数，从而接受更多参数。

array_number=0    # 第一个数组。
element_number=3
choose_array      # 13

array_number=2    # 第三个数组。
element_number=4
choose_array      # 34

array_number=3    # 空数组 (array3没有分配空间)
element_number=4
choose_array      # (null)

# 感谢Antonio Macchi指出。
```

**样例 15-13. *输出命令行参数***

```shell
#!/bin/bash
# echo-params.sh

# 请使用一些命令行参数来调用该脚本。
# 例如：
#     sh echo-params.sh first second third fourth fifth

params=$#              # 命令行参数的序号。
param=1                # 从第一个命令行参数开始。

while [ "$param" -le "$params" ]
do
  echo -n "Command-line parameter "
  echo -n \$$param     #  仅给出变量的 *名称*。
#         ^^^          #  $1, $2, $3, 等等。
                       #  为什么？
                       #  \$ 转义了第一个 "$"
                       #  所以它能够以文本输出。
                       #  并且 $param 解除了与 "$param" 的关联 . . .
                       #  . . . 如预期一般。
  echo -n " = "
  eval echo \$$param   #  给出变量的 *值*。
# ^^^^      ^^^        #  "eval"强制将 \$$ 中的 *赋值号* 
                       #  作为间接变量引用。

(( param ++ ))         # 继续下一个。
done

exit $?

# =================================================

$ sh echo-params.sh first second third fourth fifth
Command-line parameter $1 = first
Command-line parameter $2 = second
Command-line parameter $3 = third
Command-line parameter $4 = fourth
Command-line parameter $5 = fifth
```

**样例 15-14. 强制注销**

```shell
#!/bin/bash
# 杀死ppp进程来强制注销。
# 当然，是拨号连接。

# 脚本需要被root用户运行。

SERPORT=ttyS3
#  取决于硬件甚至是内核版本，
#  您机器上的调制解调器端口可能不同 --
#  /dev/ttyS1 或 /dev/ttyS2。


killppp="eval kill -9 `ps ax | awk '/ppp/ { print $1 }'`"
#                     -------- ppp的进程ID -------  

$killppp                     # 这个变量现在是一条命令。


# 以下的操作必须由root用户完成。

chmod 666 /dev/$SERPORT      # 恢复r+w权限，不然呢？
#  由于在ppp上执行SIGKILL更改了串行(serial)端口的权限，
#  我们需要将权限恢复到以前的状态。

rm /var/lock/LCK..$SERPORT   # 删除串行(serial)端口锁文件。为什么？

exit $?

# 练习：
# ---------
# 1) 让脚本检查是否是root用户在调用它。
# 2) 在试图终止进程之前，检查要终止的进程是否正在运行。 
# 3) 基于“fuser”编写此脚本的替代版本：
#       if [ fuser -s /dev/modem ]; then . . .
```

**样例 15-15. *rot13*的一个版本**

```shell
#!/bin/bash
# 使用"eval"的"rot13"版本。
# 请对比样例"rot13.sh"。

setvar_rot_13()              # "rot13"倒频。
{
  local varname=$1 varvalue=$2
  eval $varname='$(echo "$varvalue" | tr a-z n-za-m)'
}


setvar_rot_13 var "foobar"   # 通过rot13运行"foobar"。
echo $var                    # sbbone

setvar_rot_13 var "$var"     # 通过rot13运行"sbbone"。
                             # 返回原值。
echo $var                    # foobar

# 该样例由Stephane Chazelas所写。
# 由本文档作者修改。

exit 0
```

这是另一个使用*eval*来*计算*复杂表达式的例子，这个例子来自于早期YongYe[俄罗斯方块游戏脚本](https://github.com/yongye/shell/blob/master/Tetris_Game.sh)。

```shell
eval ${1}+=\"${x} ${y} \"
```

[样例 A-53](https://tldp.org/LDP/abs/html/contributed-scripts.html#SAMORSE)使用*eval*将[数组](https://tldp.org/LDP/abs/html/arrays.html#ARRAYREF)元素转化为命令列表。

<em>**eval**</em>命令出现在[间接引用](https://tldp.org/LDP/abs/html/ivr.html#IVRREF)的旧版本中。

```shell
eval var=\$$var
```

{% hint style="info" %}

*eval*命令可用于[参数化*大括号拓展*](https://tldp.org/LDP/abs/html/bashver3.html#BRACEEXPREF3)。

{% endhint %}

> ![note](https://tldp.org/LDP/abs/images/caution.gif)使用**eval**命令可能具有风险，如果存在合理的替代方案，通常应该避免使用。**eval $COMMANDS**会执行*COMMANDS*的内容，其中可能包含诸如<strong>rm -rf *</strong>这样令人不快的“惊喜”。对陌生人编写的陌生代码进行**eval**是非常危险的。

### set

**set**命令用于更改内部脚本变量/选项的值。其中一个用途是切换[选项标志](https://tldp.org/LDP/abs/html/options.html#OPTIONSREF)，来帮助确定脚本的行为。另一个应用是重置脚本认为是命令结果的[位置参数](https://tldp.org/LDP/abs/html/internalvariables.html#POSPARAMREF)（<strong>set \`command \`</strong>）。之后，脚本可以解析命令输出的[字段](https://tldp.org/LDP/abs/html/special-chars.html#FIELDREF)。

**样例 15-6. 使用带有位置参数的*set***

```shell
#!/bin/bash
# ex34.sh
# 脚本 "set-test"

# 请使用三个命令行参数来调用该脚本，
# 例如，"sh ex34.sh one two three"。

echo
echo "Positional parameters before  set \`uname -a\` :"
echo "Command-line argument #1 = $1"
echo "Command-line argument #2 = $2"
echo "Command-line argument #3 = $3"


set `uname -a` # 设置命令`uname -a`输出的位置参数。

echo
echo +++++
echo $_        # +++++
# 脚本中设置的标志。
echo $-        # hB
#                反常行为？
echo

echo "Positional parameters after  set \`uname -a\` :"
# $1, $2, $3, 等。 重新初始化`uname -a`的结果。
echo "Field #1 of 'uname -a' = $1"
echo "Field #2 of 'uname -a' = $2"
echo "Field #3 of 'uname -a' = $3"
echo \#\#\#
echo $_        # ###
echo

exit 0
```

还有更多位置参数的故事，等你去探索！

**样例 15-17. 反转位置参数**

```shell
#!/bin/bash
# revposparams.sh: 反转位置参数。
# 该脚本由Dan Jacobson所写，由本书作者对代码进行美化。

set a\ b c d\ e;
#     ^      ^     转义的空格
#       ^ ^        未转义的空格
OIFS=$IFS; IFS=:;
#              ^   保存旧IFS并且设置一个新的。

echo

until [ $# -eq 0 ]
do          #      单步执行位置参数。
  echo "### k0 = "$k""     # 执行前
  k=$1:$k;  #      将每个位置参数附加到循环变量。
#     ^
  echo "### k = "$k""      # 执行后
  echo
  shift;
done

set $k  #  设置新的位置变量。
echo -
echo $# #  位置变量的数量。
echo -
echo

for i   #  省略"in list"结构会将变量 -- i --
        #  作为位置参数。
do
  echo $i  # 展示新的位置参数。
done

IFS=$OIFS  # 恢复IFS。

#  问题：
#  是否有必要设置一个新的IFS，即内部字段分隔符，
#  来使此脚本正常工作？
#  如果不设置会怎样？请尝试一下。
#  并且，为什么要在第17行要使用 -- 冒号 -- 新的IFS
#  来附加到循环变量。
#  此目的究竟为何？

exit 0

$ ./revposparams.sh

### k0 = 
### k = a b

### k0 = a b
### k = c a b

### k0 = c a b
### k = d e c a b

-
3
-

d e
c
a b
```

没有任何选项或参数来调用**set**,，仅会列出了所有已初始化的[环境变量](https://tldp.org/LDP/abs/html/othertypesv.html#ENVREF)和其他变量。

```shell
bash$ set
AUTHORCOPY=/home/bozo/posts
 BASH=/bin/bash
 BASH_VERSION=$'2.05.8(1)-release'
 ...
 XAUTHORITY=/home/bozo/.Xauthority
 _=/etc/bashrc
 variable22=abc
 variable23=xzy
```

使用带有 -- 参数的**set**会显式地将变量值赋给位置参数。如果 -- 后没有跟任何变量，则会*取消设置*位置参数。

**样例 15-18. 重新赋值位置参数**

```shell
#!/bin/bash

variable="one two three four five"

set -- $variable
# 将位置参数设置为"$variable"的值。

first_param=$1
second_param=$2
shift; shift        # Shift前两个位置参数。
# shift 2             同样有效。
remaining_params="$*"

echo
echo "first parameter = $first_param"             # one
echo "second parameter = $second_param"           # two
echo "remaining parameters = $remaining_params"   # three four five

echo; echo

# 再一次。
set -- $variable
first_param=$1
second_param=$2
echo "first parameter = $first_param"             # one
echo "second parameter = $second_param"           # two

# ======================================================

set --
# 如果未指定变量，则取消设置位置参数。

first_param=$1
second_param=$2
echo "first parameter = $first_param"             # (空值)
echo "second parameter = $second_param"           # (空值)

exit 0
```

另请参阅[样例 11-2](https://tldp.org/LDP/abs/html/loops1.html#EX22A)和[样例 16-56](https://tldp.org/LDP/abs/html/extmisc.html#EX33A)。

### unset

**unset**命令会删除一个shell变量，有效地将其设为空值。请注意该条命令不会影响位置参数。

```shell
bash$ unset PATH

bash$ echo $PATH

bash$ 
```

**样例 15-19. “Unset”一个变量**

```shell
#!/bin/bash
# unset.sh: 取消设置一个变量。

variable=hello                       #  初始化。
echo "variable = $variable"

unset variable                       #  取消设置。
                                     #  在该特定的上下文中，
                                     #  与 "variable= " 具有相同效果
echo "(unset) variable = $variable"  #  $variable为空。

if [ -z "$variable" ]                #  尝试测试一下字符串长度。
then
  echo "\$variable has zero length."
fi

exit 0
```

> ![note](https://tldp.org/LDP/abs/images/note.gif)在大多数情况下，*未声明*的变量和*被unset*的变量是等效的。但是，[${parameter:-default}](https://tldp.org/LDP/abs/html/parameter-substitution.html#UNDDR)参数替换构造可以区分两者。

### export

**export** [[4]](https://tldp.org/LDP/abs/html/internal.html#FTN.AEN9199) 命令用于为正在运行的脚本或shell的所有子进程提供可用的变量。**export**命令的一个重要用途是在[启动文件](https://tldp.org/LDP/abs/html/files.html#FILESREF1)中，来进行初始化并使后续用户进程可访问[环境变量](https://tldp.org/LDP/abs/html/othertypesv.html#ENVREF)。

> ![note](https://tldp.org/LDP/abs/images/caution.gif)不幸的是，没有办法将导出的变量重新导回父进程中，即调用该命令的脚本或shell中。

**样例 15-20. 使用*export*来将变量传递给嵌入的*awk*脚本**

```shell
#!/bin/bash

#  "列累加器"脚本(col-totaler.sh) 的另一版本，
#  它将目标文件中的指定列 (数字) 相加。
#  这需要使用环境将脚本变量传递给 'awk' . . .
#  并将awk脚本放在变量中。

ARGS=2
E_WRONGARGS=85

if [ $# -ne "$ARGS" ] # 检查命令行参数数量是否正确。
then
   echo "Usage: `basename $0` filename column-number"
   exit $E_WRONGARGS
fi

filename=$1
column_number=$2

#===== 到此为止，与原脚本相同 =====#

export column_number
# 将列号导出到环境，以进行检索。

# -----------------------------------------------
awkscript='{ total += $ENVIRON["column_number"] }
END { print total }'
# 没错，awk脚本中可以使用变量。
# -----------------------------------------------

# 现在，跑一下awk脚本。
awk "$awkscript" "$filename"

# 感谢Stephane Chazelas.

exit 0
```

{% hint style="info" %}

可以执行**export var1=xxx**来初始化和导出变量，其具有相同的效果。

但是，正如Greg Keraunen指出的那样，在某些情况下，设置变量再导出，还是一步到位会有一个不同的效果。

```shell
bash$ export var=(a b); echo ${var[0]}
(a b)



bash$ var=(a b); export var; echo ${var[0]}
a
```

{% endhint %}

> ![note](https://tldp.org/LDP/abs/images/note.gif)导出的变量可能需要进行特殊处理。请参见[样例 M-2](https://tldp.org/LDP/abs/html/sample-bashrc.html#BASHPROF)。

**declare, typeset**

[declare](https://tldp.org/LDP/abs/html/declareref.html)和[typeset](https://tldp.org/LDP/abs/html/declareref.html)命令指定并且/或限制变量的特性。

### readonly

与[declare -r](https://tldp.org/LDP/abs/html/declareref.html)相同，将变量设置为只读，或者实际上设置为常量。如果尝试修改该变量将失败，并显示错误信息。这是shell针对*C*语言**const**类型限定符的模拟。

### getopts

这个强大的工具会解析传递给脚本的命令行参数。这是*C*程序员所熟悉的[getopt](https://tldp.org/LDP/abs/html/extmisc.html#GETOPTY)外部命令和*getopt*库函数的Bash模拟。它允许多个选项[[5]](https://tldp.org/LDP/abs/html/internal.html#FTN. AEN9289)和关联参数传递和联结到脚本（例如<code>**scriptname -abc -e /usr/local**</code>）。

**getopts**构造使用了两个隐式变量。其中`$OPTIND`是参数指针（选项索引），`$OPTARG` （选项参数）是附加到选项的（可选）参数。在声明中，选项名后面的冒号会将该选项标记为关联参数。

**getopts**构造通常打包在[while循环](https://tldp.org/LDP/abs/html/loops1.html#WHILELOOPREF)中，该循环一次仅处理一个选项和参数，然后递增隐式`$ OPTIND`变量来指向下一个。

> ![note](https://tldp.org/LDP/abs/images/note.gif)
> 
> 1. 从命令行传递到脚本的参数必须在前面加上破折号（-）。前缀的 - 使得**getopts**将命令行参数识别为*选项*。实际上，**getopts**不会在没有前缀 - 的情况下处理参数，并且当缺乏参数时终止处理选项。
> 
> 2. **getopts**模板与标准[while循环](https://tldp.org/LDP/abs/html/loops1.html#WHILELOOPREF)略有不同，因为它缺少条件括号。
> 
> 3. **getopts**构造可以完全替代传统[getopt](https://tldp.org/LDP/abs/html/extmisc.html#GETOPTY)外部命令，甚至好得多。

```shell
while getopts ":abcde:fg" Option
# 初始的声明。
# a, b, c, d, e, f 和 g 是期望的选项(标志)。
# 选项 'e' 的 : 说明他需要同时携带一个参数进行传递。
do
  case $Option in
    a ) # 用变量 'a' 做点什么。
    b ) # 用变量 'b' 做点什么。
    ...
    e)  # 用变量 'e' 做点什么，并且带上$OPTARG，
        # 即与选项 'e' 一起传递的关联变量。
    ...
    g ) # 用变量 'g' 做点什么。
  esac
done
shift $(($OPTIND - 1))
# 将参数指针移至下一个。

# 所有这些并不像看起来那么复杂 <grin>。
```

**样例 15-21. 使用*getopts*来读取传递给脚本的选项 / 参数**

```shell
#!/bin/bash
# ex33.sh: 练习getopts和OPTIND
#          该脚本在Bill Gradwohl的建议下，与10/09/03修改。

# 在这里，我们将观察 "getopts" 是如何处理脚本的命令行参数的。
# 参数被解析为 "选项" (标志)和关联参数。

# 尝试使用以下的方式调用脚本：
#   'scriptname -mn'
#   'scriptname -oq qOption' (qOption可以是一些任意字符串。)
#   'scriptname -qXXX -r'
#
#   'scriptname -qr'
#       - 非预期的结果，将"r"作为选项"q"的参数
#   'scriptname -q -r' 
#       - 非预期的结果，效果同上
#   'scriptname -mnop -mnop'  - 非预期的结果
#   (OPTIND在指出选项来自何处时是不可靠的。)
#  如果一个选项需要一个参数 ("flag:")，
#  那么它就会抓住这个参数，不管其后是什么。

NO_ARGS=0 
E_OPTERROR=85

if [ $# -eq "$NO_ARGS" ]    # 没有命令行参数来调用脚本？
then
  echo "Usage: `basename $0` options (-mnopqrs)"
  exit $E_OPTERROR          # 退出并且解释使用方法。
                            # Usage: scriptname -options
                            # 请注意：破折号(-) 是必需的。
fi  


while getopts ":mnopq:rs" Option
do
  case $Option in
    m     ) echo "Scenario #1: option -m-   [OPTIND=${OPTIND}]";;
    n | o ) echo "Scenario #2: option -$Option-   [OPTIND=${OPTIND}]";;
    p     ) echo "Scenario #3: option -p-   [OPTIND=${OPTIND}]";;
    q     ) echo "Scenario #4: option -q-\
                  with argument \"$OPTARG\"   [OPTIND=${OPTIND}]";;
    #  请注意选项 'q' 必须有一个关联参数，
    #  否则它将落入默认选项。
    r | s ) echo "Scenario #5: option -$Option-";;
    *     ) echo "Unimplemented option chosen.";;   # 默认。
  esac
done

shift $(($OPTIND - 1))
#  递减参数指针，使其指向下一个参数。
#  $1现在将引用命令行中提供的第一个非选项参数(如果存在)。

# --> (译者注：有点不好理解，huh.试试看取消以下三行的注释。)
# --> echo "Now, I have no options!! HaHa~~"
# --> echo "My \$1 is" $1
# --> echo "My args are" $@

exit $?

#  正如Bill Gradwohl指出，
#  “getopts机制使我们可以这样写: scriptname -mnop -mnop，
#  但是没有任何可靠的方法可以通过使用OPTIND来区分参数来自何处。”
#  但是，总是有解决方法。
```

## 脚本行为控制命令

### **source**, . （[点](https://tldp.org/LDP/abs/html/special-chars.html#DOTREF)命令）

从命令行调用时，此命令将执行脚本。在脚本中，<code>**source file-name**</code>会读取文件`file-name`。*source*一个文件（点命令）会将代码*导入*脚本，并附加到脚本中（与*C*程序中的<strong>`#include`</strong>指令相同的效果）。最终结果与脚本中实际存在的 “source” 代码行相同。这在多个脚本中使用通用数据文件或函数库的情况下很有用。

**样例 15-22. "Include"一个数据文件**

```shell
#!/bin/bash
#  请注意该样例必须被bash解释器调用，即bash ex38.sh
#  不是sh ex38.sh！

. data-file    # 加载一个数据文件。
# 等效于"source data-file"，但是更加通用。

#  文件"data-file"必须存在于当前的工作目录下，
#  因为它被其basename所引用。

# 现在，让我们引用该文件中的一些数据。

echo "variable1 (from data-file) = $variable1"
echo "variable3 (from data-file) = $variable3"

let "sum = $variable2 + $variable4"
echo "Sum of variable2 + variable4 (from data-file) = $sum"
echo "message1 (from data-file) is \"$message1\""
#                                      转义符
echo "message2 (from data-file) is \"$message2\""

print_message This is the message-print function in the data-file.


exit $?
```

上面的[样例 15-22](https://tldp.org/LDP/abs/html/internal.html#EX38)中的文件`data-file`。必须位于相同目录下。

```shell
# 这是被一个脚本所读取的数据文件。
# 这种类型的文件可能包含变量、函数等。
# 它会被脚本中的 'source' 或 '.' 命令所读取。

# 让我们初始化一些变量吧。

variable1=23
variable2=474
variable3=5
variable4=97

message1="Greetings from *** line $LINENO *** of the data file!"
message2="Enough for now. Goodbye."

print_message ()
{   # 输出任何传递给它的消息。

  if [ -z "$1" ]
  then
    return 1 # 错误，如果缺少参数。
  fi

  echo

  until [ -z "$1" ]
  do             # 逐步传递给函数的参数。
    echo -n "$1" # 一次输出一个参数，取消换行符。
    echo -n " "  # 单词间插入空格。
    shift        # 下一个。
  done  

  echo

  return 0
}
```

如果*被source*的文件本身是可执行脚本，则它将运行，然后将控制权返回给调用它的脚本。为此目的，*被source*的可执行脚本可以使用[return](https://tldp.org/LDP/abs/html/complexfunct.html#RETURNREF)来返回。

参数可以 (可选) 作为[位置参数](https://tldp.org/LDP/abs/html/othertypesv.html#POSPARAMREF1)传递给*被source*的文件。

脚本甚至有可能自己*source*自己，尽管这似乎没有任何实际应用。

**样例 15-23. 一个source自己的（无用）脚本**

```shell
#!/bin/bash
# self-source.sh: 一个“递归地” source 自己的脚本。
# 来自于 "Stupid Script Tricks," 第II卷。

MAXPASSCNT=100    # 最大执行次数。

echo -n  "$pass_count  "
#  在第一次执行时，只会输出两个空格，
#  因为$pass_count尚未初始化。

let "pass_count += 1"
#  假设未初始化的变量$pass_count在第一次可以递增。
#  适用于Bash和pdksh，
#  但是它依赖于不可移植的（并且可能是危险的）行为。
#  最好是在递增之前将$pass_count初始化为0。

while [ "$pass_count" -le $MAXPASSCNT ]
do
  . $0   # 脚本会"source"自己，而不是调用自己。
         # ./$0（这将是真正的递归）在这里不起作用。为什么？
done  

#  这里所发生的实际上不是递归，
#  因为脚本有效地“扩展”了自己，
#  即每次通过“while”循环
#  并且在每个第19行的"source"处
#  均生成一个新的代码段。
#
#  当然，脚本会将每个新"source"脚本的"#!"行视为注释，
#  而不是一个新脚本的起点。

echo

exit 0   # 效果就是是从1数到100。
         # 令人印象深刻。

# 练习：
# --------
# 写一个利用该小技巧的脚本，来实际做些有用的事情。
```

### exit

无条件终止脚本。[[6]](https://tldp.org/LDP/abs/html/internal.html#FTN.AEN9393)**exit**命令可以选择接受一个整数作为参数，该参数会作为脚本的[退出状态](https://tldp.org/LDP/abs/html/exit-status.html#EXITSTATUSREF)返回给shell。除了最简单的脚本之外，最好用**exit 0**来结束脚本，这表示运行成功。

> ![note](https://tldp.org/LDP/abs/images/note.gif)如果脚本由于不携带参数的**exit**终止，则脚本的退出状态是脚本中执行的最后一个命令的退出状态，不取决于**exit**。等效于**exit**。

> ![note](https://tldp.org/LDP/abs/images/note.gif)**exit**命令也可以用于终止[子shell](https://tldp.org/LDP/abs/html/subshells.html#SUBSHELLSREF)。

### exec

这个shell内置命令用指定的命令替换当前进程。通常，当shell遇到命令时，它会[派生出](https://tldp.org/LDP/abs/html/internal.html#FORKREF)一个子进程来实际执行该命令。当使用**exec**内置命令时，shell不会fork，并且*exec*所携带的命令会替换shell。因此，当在脚本中使用时，它会在**exec**所携带的命令终止时强制退出脚本。[[7]](https://tldp.org/LDP/abs/html/internal.html#FTN.AEN9425)

**样例 15-24. *exec*的效果**

```shell
#!/bin/bash

exec echo "Exiting \"$0\" at line $LINENO."   # 在这里退出脚本。
# $LINENO是一个内部Bash变量，设置为命令所在的行号。

# ----------------------------------
# 以下几行永远不会执行

echo "This echo fails to echo."

exit 99                       #  脚本不会在这里退出。
                              #  脚本终止后请使用 'echo $?'
                              #  来检查退出码。
                              #  它 *不会是* 99。
```

**样例 15-25. 一个*exec*自己的脚本**

```shell
#!/bin/bash
# self-exec.sh

# 注意：将该脚本的权限设为 555 或者 755，
#       然后执行 ./self-exec.sh 或者 sh ./self-exec.sh 进行调用。

echo

echo "This line appears ONCE in the script, yet it keeps echoing."
echo "The PID of this instance of the script is still $$."
#     演示子shell没有派生。

echo "==================== Hit Ctl-C to exit ===================="

sleep 1

exec $0   #  产生与该脚本完全相同的另一个实例来替换上一个。

echo "This line will never echo!"  # 为什么不会？

exit 99                            # 并不会在这里退出！
                                   # 退出码不可能是99！
```

**exec**还用于[重新分配文件描述符](https://tldp.org/LDP/abs/html/x17974.html#USINGEXECREF)。例如，<strong>`exec <zzz-file`</strong>会使用文件`zzz-file`替换`标准输入(stdin)`。

> ![note](https://tldp.org/LDP/abs/images/note.gif)find命令的`-exec`选项和shell内置命令**exec**<em>`不是`</em>一个东西。

### shopt

此命令允许动态更改*shell选项* （参见[示例 25-1](https://tldp.org/LDP/abs/html/aliases.html#AL)和[示例 25-2](https://tldp.org/LDP/abs/html/aliases.html#UNAL)）。它经常出现在Bash[启动文件](https://tldp.org/LDP/abs/html/files.html#FILESREF1)中，但在脚本中也有其用途。需要[Bash 2.0](https://tldp.org/LDP/abs/html/bashver2.html#BASH2REF)及以上版本。

```shell
shopt -s cdspell
# 允许 "cd" 忽略对目录名称较小的拼写错误
# 选项 -s 设置， -u 取消设置。

cd /hpme  # Oops! '/home'敲错了。
pwd       # /home
          # shell纠正了拼写错误。
```

### caller

将**caller**命令放在一个[函数](https://tldp.org/LDP/abs/html/functions.html#FUNCTIONREF)中会将该函数中*caller*的状态输出到`标准输出(stdout)`。

```shell
#!/bin/bash

function1 ()
{
  # 位于 function1 () 中。
  caller 0   # 告诉我。
}

function1    # 脚本的第9行。

# 9 main test.sh
# ^                 函数被调用的行号。
#   ^^^^            从脚本的 "main" 部分调用。
#        ^^^^^^^    所调用的脚本名。

caller 0     # 无效，因为它位于函数外。
```

**caller**命令还可以从另一个[被source](https://tldp.org/LDP/abs/html/internal.html#SOURCEREF)的脚本中获取*caller*信息。类似于函数，这是一个“子例程调用（suroutine call）”。

在调试的时候你会发现这个命令很有用。

## 命令

### true

一个返回成功（0）[退出状态码](https://tldp.org/LDP/abs/html/exit-status.html#EXITSTATUSREF)的命令，别无他用。

```shell
bash$ true
bash$ echo $?
0
```

```shell
# 无限循环
while true   # 别名为 ":"
do
   operation-1
   operation-2
   ...
   operation-n
   # 需要一个退出循环的方式，否则脚本会挂起。
done
```

### false

一个返回失败[退出状态码](https://tldp.org/LDP/abs/html/exit-status.html#EXITSTATUSREF)的命令，别无他用。

```shell
bash$ false
bash$ echo $?
1
```

```shell
# 测试 "false" 
if false
then
  echo "false evaluates \"true\""
else
  echo "false evaluates \"false\""
fi
# false evaluates "false"


# while "false" 循环(空循环)
while false
do
   # 以下的代码不会执行。
   operation-1
   operation-2
   ...
   operation-n
   # 无事发生！
done   
```

### type [cmd]

类似于外部命令[which](https://tldp.org/LDP/abs/html/filearchiv.html#WHICHREF)，**type cmd**会标识“cmd”。与**which**不同，**type**是Bash内置命令。**type**的`-a`选项会标识<em>`关键字`</em>和<em>`内置程序`</em>，并定位具有相同名称的系统命令，非常有用。

```shell
bash$ type '['
[ is a shell builtin
bash$ type -a '['
[ is a shell builtin
 [ is /usr/bin/[


bash$ type type
type is a shell builtin
```

**type**命令在用来[测试特定命令是否存在](https://tldp.org/LDP/abs/html/special-chars.html#DEVNULLREDIRECT)这方面非常有用。

### hash [cmds]

在shell<strong>*哈希表*</strong>[[8]](https://tldp.org/LDP/abs/html/internal.html#FTN.AEN9591)中记录指定命令的*路径*名，这样shell或脚本在后续调用这些命令时就不需要搜索[$PATH](https://tldp.org/LDP/abs/html/internalvariables.html#PATHREF)。当不带参数调用**hash**时，它将仅仅列出已经被散列的命令。`-r`选项重置哈希表。

### bind

**bind**内建命令会显示或修改*readline*[[9]](https://tldp.org/LDP/abs/html/internal.html#FTN.AEN9621)绑定键。

### help

获取shell内置命令的简短摘要。与[whatis](https://tldp.org/LDP/abs/html/filearchiv.html#WHATISREF)相对应，但用于内置命令。在Bash[第4版](https://tldp.org/LDP/abs/html/bashver4.html#BASH4REF)中，*help*信息获得了极大的扩充。

```shell
bash$ help exit
exit: exit [n]
    Exit the shell with a status of N.  If N is omitted, the exit status
    is that of the last command executed.
```

## 注记

[[1]](https://tldp.org/LDP/abs/html/internal.html#AEN8607)正如Nathan Coulter所指出的，“虽然分叉一个进程是一个低成本的操作，但在新分叉的子进程中执行一个新程序会增加额外的开销。”

[[2]](https://tldp.org/LDP/abs/html/internal.html#AEN8650)但是，[time](https://tldp.org/LDP/abs/html/timedate.html#TIMREF)命令是一个例外，它在Bash官方文档中作为关键字（“保留字”）列出。

[[3]](https://tldp.org/LDP/abs/html/internal.html#AEN9009)请注意<em>let[不能用于设置**字符串型**变量](https://tldp.org/LDP/abs/html/gotchas.html#LETBAD)</em>。

[[4]](https://tldp.org/LDP/abs/html/internal.html#AEN9199)**Export**信息是为了使其在可用于更广的上下文中。另请参见[作用域](https://tldp.org/LDP/abs/html/subshells.html#SCOPEREF)。

[[5]](https://tldp.org/LDP/abs/html/internal.html#AEN9289)*选项*是充当标志的参数，用于打开或关闭脚本行为。与特定选项相关联的参数说明该选项（标志）所对应的脚本行为发生或不发生。

[[6]](https://tldp.org/LDP/abs/html/internal.html#AEN9393)从技术上讲，**exit**仅仅终止正在运行的进程（或者shell），而**不是父进程**。

[[7]](https://tldp.org/LDP/abs/html/internal.html#AEN9425)除非**exec**用于[重新声明文件描述符](https://tldp.org/LDP/abs/html/x17974.html#USINGEXECREF)。

[[8]](https://tldp.org/LDP/abs/html/internal.html#AEN9591)

*哈希*是一种为存储在表中的数据创建查找关键字的方法。*数据本身*被“打乱”以创建密钥，这属于许多简单的数学*算法*（方式或解决办法）中的一种。

*哈希*的一个优点是速度快。缺点是可能会发生*冲突*——一个键映射到多个数据项。

关于*哈希*的其他样例，你可以参阅[样例 A-20](https://tldp.org/LDP/abs/html/contributed-scripts.html#HASHLIB)和[样例 A-21](https://tldp.org/LDP/abs/html/contributed-scripts.html#HASHEXAMPLE)。

[[9]](https://tldp.org/LDP/abs/html/internal.html#AEN9621)Bash使用*readline*库在交互式shell中读取输入。
