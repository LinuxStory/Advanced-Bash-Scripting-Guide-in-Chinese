# 11.1 循环

循环是当循环控制条件为真时，一系列命令迭代[^1]执行的代码块。

### for 循环

### `for arg in [list]`

这是 shell 中最基本的循环结构，它与C语言形式的循环有着明显的不同。

```bash
for arg in [list]
do
  command(s)...
done
```

> ![note](http://tldp.org/LDP/abs/images/note.gif) 在循环的过程中，`arg` 会从 `list` 中连续获得每一个变量的值。
>
```bash
for arg in "$var1" "$var2" "$var3" ... "$varN"
# 第一次循环中，arg = $var1
# 第二次循环中，arg = $var2
# 第三次循环中，arg = $var3
# ...
# 第 N 次循环中，arg = $varN
>
# 为了防止可能的字符分割问题，[list] 中的参数都需要被引用。
```

参数 list 中允许含有 [通配符](http://tldp.org/LDP/abs/html/special-chars.html#ASTERISKREF)。

如果 `do` 和 `for` 写在同一行时，需要在 list 之后加上一个分号。

`for arg in [list] ; do`

样例 11-1. 简单的 for 循环

```bash
#!/bin/bash
# 列出太阳系的所有行星。

for planet in Mercury Venus Earth Mars Jupiter Saturn Uranus Neptune Pluto
do
  echo $planet  # 每一行输出一个行星。
done

echo; echo

for planet in "Mercury Venus Earth Mars Jupiter Saturn Uranus Neptune Pluto"
    # 所有的行星都输出在一行上。
    # 整个 'list' 被包裹在引号中时是作为一个单一的变量。
    # 为什么？因为空格也是变量的一部分。
do
  echo $planet
done

echo; echo "Whoops! Pluto is no longer a planet!"

exit 0
```

[list] 中的每一个元素中都可能含有多个参数。这在处理参数组中非常有用。在这种情况下，使用 [`set`](http://tldp.org/LDP/abs/html/internal.html#SETREF) 命令（查看 [样例 15-16](http://tldp.org/LDP/abs/html/internal.html#EX34)）强制解析 [list] 中的每一个元素，并将元素的每一个部分分配给位置参数。

样例 11-2. `for` 循环 [list] 中的每一个变量有两个参数的情况

```bash
#!/bin/bash
# 让行星再躺次枪。

# 将每个行星与其到太阳的距离放在一起。

for planet in "Mercury 36" "Venus 67" "Earth 93" "Mars 142" "Jupiter 483"
do
  set -- $planet  #  解析变量 "planet"
                  #+ 并将其每个部分赋值给位置参数。
  # "--" 防止一些极端情况，比如 $planet 为空或者以破折号开头。
  
  # 因为位置参数会被覆盖掉，因此需要先保存原先的位置参数。
  # 你可以使用数组来保存
  #         original_params=("$@")
  
  echo "$1		$2,000,000 miles from the sun"
  #-------两个制表符---将后面的一系列 0 连到参数 $2 上。
done

# （感谢 S.C. 做出的额外注释。）

exit 0
```

一个单一变量也可以成为 `for` 循环中的 [list]。

样例 11-3. 文件信息：查看一个单一变量中含有的文件列表的文件信息

```bash
#!/bin/bash
# fileinfo.sh

FILES="/usr/sbin/accept
/usr/sbin/pwck
/usr/sbin/chroot
/usr/bin/fakefile
/sbin/badblocks
/sbin/ypbind"     # 你可能会感兴趣的一系列文件。
                  # 包含一个不存在的文件，/usr/bin/fakefile。
                  
echo

for file in $FILES
do

  if [ ! -e "$file" ]       # 检查文件是否存在。
  then
    echo "$file does not exist."; echo
    continue                # 继续判断下一个文件。
  fi
  
  ls -l $file | awk '{ print $8 "         file size: " $5 }'  # 输出其中的两个域。
  whatis `basename $file`   # 文件信息。
  # 脚本正常运行需要注意提前设置好 whatis 的数据。
  # 使用 root 权限运行 /usr/bin/makewhatis 可以完成。
  echo
done

exit 0
```

`for` 循环中的 [list] 可以是一个参数。

样例 11-4. 操作含有一系列文件的参数

```bash
#!/bin/bash

filename="*txt"

for file in $filename
do
 echo "Contents of $file"
 echo "---"
 cat "$file"
 echo
done
```

如果在匹配文件扩展名的 `for` 循环中的 [list] 含有通配符（* 和 ?），那么将会进行文件名扩展。

样例 11-5. 在 `for` 循环中操作文件

```bash
#!/bin/bash
# list-glob.sh: 通过文件名扩展在 for 循环中产生 [list]。
# 通配 = 文件名扩展。

echo

for file in *
#           ^  Bash 在检测到通配表达式时，
#+             会进行文件名扩展。
do
  ls -l "$file"  # 列出 $PWD（当前工作目录）下的所有文件。
  #  回忆一下，通配符 "*" 会匹配所有的文件名，
  #+ 但是，在文件名扩展中，他将不会匹配以点开头的文件。
  
  #  如果没有匹配到文件，那么它将会扩展为它自身。
  #  为了防止出现这种情况，需要设置 nullglob 选项。
  #+    (shopt -s nullglob)。
  #  感谢 S.C.
done

echo; echo

for file in [jx]*
do
  rm -f $file    # 删除当前目录下所有以 "j" 或 "x" 开头的文件。
  echo "Removed file \"$file\"".
done

echo

exit 0
```

如果在 `for` 循环中省略 `in [list]` 部分，那么循环将会遍历位置参数（`$@`）。[样例 A-15](http://tldp.org/LDP/abs/html/contributed-scripts.html#PRIMES) 中使用到了这一点。也可以查看 [样例 15-17](http://tldp.org/LDP/abs/html/internal.html#REVPOSPARAMS)。

样例 11-6. 缺少 `in [list]` 的 `for` 循环

```bash
#!/bin/bash

# 尝试在带参数和不带参数两种情况下调用这个脚本，观察发生了什么。

for a
do
 echo -n "$a "
done

#  缺失 'in list' 的情况下，循环会遍历 '$@'
#+（命令行参数列表，包括空格）。

echo

exit 0
```

可以在 `for` 循环中使用 [命令代换](http://tldp.org/LDP/abs/html/commandsub.html#COMMANDSUBREF) 生成 [list]。查看 [样例 16-54](http://tldp.org/LDP/abs/html/extmisc.html#EX53)，[样例 11-11](http://tldp.org/LDP/abs/html/loops1.html#SYMLINKS) 和 [样例 16-48](http://tldp.org/LDP/abs/html/mathc.html#BASE)。

样例 11-7. 在 `for` 循环中使用命令代换生成 [list]

```bash
#!/bin/bash
# for-loopcmd.sh: 带命令代换所生成 [list] 的 for 循环

NUMBERS="9 7 3 8 37.53"

for number in `echo $NUMBERS`  # for number in 9 7 3 8 37.53
do
  echo -n "$number "
done

echo
exit 0
```

下面是使用命令代换生成 [list] 的更加复杂的例子。

样例 11-8. 一种替代 `grep` 搜索二进制文件的方法

```bash
#!/bin/bash
# bin-grep.sh: 在二进制文件中定位匹配的字符串。

# 一种替代 `grep` 搜索二进制文件的方法
# 与 "grep -a" 的效果类似

E_BADARGS=65
E_NOFILE=66

if [ $# -ne 2 ]
then
  echo "Usage: `basename $0` search_string filename"
  exit $E_BADARGS
fi

if [ ! -f "$2" ]
then
  echo "File \"$2\" does not exist."
  exit $E_NOFILE
fi


IFS=$'\012'       # 按照 Anton Filippov 的意见应该是
                  # IFS="\n"
for word in $( strings "$2" | grep "$1" )
# "strings" 命令列出二进制文件中的所有字符串。
# 将结果通过管道输出到 "grep" 中，检查是不是匹配的字符串。
do
  echo $word
done

# 就像 S.C. 指出的那样，第 23-30 行可以换成下面的形式：
#    strings "$2" | grep "$1" | tr -s "$IFS" '[\n*]'


# 尝试运行脚本 "./bin-grep.sh mem /bin/ls"

exit 0
```

下面的例子同样展示了如何使用命令代换生成 [list]。

样例 11-9. 列出系统中的所有用户

```bash
#!/bin/bash
# userlist.sh

PASSWORD_FILE=/etc/passwd
n=1           # 用户数量

for name in $(awk 'BEGIN{fs=":"}{print $1}' < "$PASSWORD_FILE" )
# 分隔符 = :              ^^^^^^
# 输出第一个域                    ^^^^^^^^
# 读取密码文件 /etc/passwd                    ^^^^^^^^^^^^^^^^^
do
  echo "USER #$n = $name"
  let "n += 1"
done


# USER #1 = root
# USER #2 = bin
# USER #3 = daemon
# ...
# USER #33 = bozo

exit $?

# 讨论：
# -----
# 一个普通用户是如何读取 /etc/passwd 文件的？
# 提示：检查 /etc/passwd 的文件权限。
# 这算不算是一个安全漏洞？为什么？
```

另外一个关于 [list] 的例子也来自于命令代换。

样例 11-10. 检查目录中所有二进制文件的原作者

```bash
#!/bin/bash
# findstring.sh
# 在指定目录的二进制文件中寻找指定的字符串。

directory=/usr/bin
fstring="Free Software Foundation"  # 查看哪些文件来自于 FSF。

for file in $( find $directory -type f -name '*' | sort )
do
  strings -f $file | grep "$fstring" | sed -e "s%$directory%%"
  #  在 "sed" 表达式中，你需要替换掉 "/" 分隔符，
  #+ 因为 "/" 是一个会被过滤的字符。
  #  如果不做替换，将会产生一个错误。（你可以尝试一下。）
done

exit $?

# 简单的练习：
# ----------
# 修改脚本，使其可以从命令行参数中获取 $directory 和 $fstring。
```

最后一个关于 [list] 和命令代换的例子，但这个例子中的命令是一个[函数](http://tldp.org/LDP/abs/html/functions.html#FUNCTIONREF)。

```bash
generate_list ()
{
  echo "one two three"
}

for word in $(generate_list)  # "word" 获得函数执行的结果。
do
  echo "$word"
done

# one
# two
# three
```

`for` 循环的结果可以通过管道导向至一个或多个命令中。

样例 11-11. 列出目录中的所有符号链接。

```bash
#!/bin/bash
# symlinks.sh: 列出目录中的所有符号链接。

directory=${1-`pwd`}
# 如果没有特别指定，缺省目录为当前工作目录。
# 等价于下面的代码块。
# ---------------------------------------------------
# ARGS=1                 # 只有一个命令行参数。
#
# if [ $# -ne "$ARGS" ]  # 如果不是只有一个参数的情况下
# then
#   directory=`pwd`      # 设为当前工作目录。
# else
#   directory=$1
# fi
# ---------------------------------------------------

echo "symbolic links in directory \"$directory\""

for file in "$( find $directory -type 1 )"   # -type 1 = 符号链接
do
  echo "$file"
done | sort                                  # 否则文件顺序会是乱序。
#  严格的来说这里并不需要使用循环，
#+ 因为 "find" 命令的输出结果已经被扩展成一个单一字符串了。
#  然而，为了方便大家理解，我们使用了循环的方式。

#  Dominik 'Aeneas' Schnitzer 指出，
#+ 不引用 $( find $directory -type 1 ) 的话，
#  脚本将在文件名包含空格时阻塞。

exit 0


# --------------------------------------------------------
# Jean Helou 提供了另外一种方法：

echo "symbolic links in directory \"$directory\""
# 备份当前的内部字段分隔符。谨慎永远没有坏处。
OLDIFS=$IFS
IFS=:

for file in $(find $directory -type 1 -printf "%p$IFS")
do     #                              ^^^^^^^^^^^^^^^^
       echo "$file"
       done|sort

# James "Mike" Conley 建议将 Helou 的代码修改为：

OLDIFS=$IFS
IFS='' # 空的内部字段分隔符意味着将不会分隔任何字符串
for file in $( find $directory -type 1 )
do
  echo $file
  done | sort
  
#  上面的代码可以在目录名包含冒号（前一个允许包含空格）
#+ 的情况下仍旧正常工作。
```

只需要对上一个样例做一些小小的改动，就可以把在标准输出 `stdout` 中的循环 [重定向](http://tldp.org/LDP/abs/html/io-redirection.html#IOREDIRREF) 到文件中。

样例 11-12. 将目录中的所有符号链接保存到文件中。

```bash
#!/bin/bash
# symlinks.sh: 列出目录中的所有符号链接。

OUTFILE=symlinks.list

directory=${1-`pwd`}
# 如果没有特别指定，缺省目录为当前工作目录。


echo "symbolic links in directory \"$directory\"" > "$OUTFILE"
echo "---------------------------" >> "$OUTFILE"

for file in "$( find $directory -type 1 )"    # -type 1 = 符号链接
do
  echo "$file"
done | sort >> "$OUTFILE"                     # 将 stdout 的循环结果
#           ^^^^^^^^^^^^^                       重定向到文件。

# echo "Output file = $OUTFILE"

exit $?
```

还有另外一种看起来非常像C语言中循环那样的语法。你需要使用到 [双圆括号](http://tldp.org/LDP/abs/html/dblparens.html#DBLPARENSREF) 语法。

样例 11-13. C语言风格的循环

```bash
#!/bin/bash
# 用多种方式数到10。

echo

# 基础版
for a in 1 2 3 4 5 6 7 8 9 10
do
  echo -n "$a "
done

echo; echo

# +==========================================+

# 使用 "seq"
for a in `seq 10`
do
  echo -n "$a "
done

echo; echo

# +==========================================+

# 使用大括号扩展语法
# Bash 3+ 版本有效。
for a in {1..10}
do
  echo -n "$a "
done

echo; echo

# +==========================================+

# 现在用类似C语言的语法再实现一次。

LIMIT=10

for ((a=1; a <= LIMIT ; a++))  # 双圆括号语法，不带 $ 的 LIMIT
do
  echo -n "$a "
done                           # 从 ksh93 中学习到的特性。

echo; echo

# +==========================================+

# 我们现在使用C语言中的逗号运算符来使得两个变量同时增加。

for ((a=1, b=1; a <= LIMIT ; a++, b++))
do  # 逗号连接操作。
  echo -n "$a-$b "
done

echo; echo

exit 0
```

还可以查看 [样例 27-16](http://tldp.org/LDP/abs/html/arrays.html#QFUNCTION)，[样例 27-17](http://tldp.org/LDP/abs/html/arrays.html#TWODIM) 和 [样例 A-6](http://tldp.org/LDP/abs/html/contributed-scripts.html#COLLATZ)。

\---

接下来，我们将展示在真实环境中应用的循环。

样例 11-14. 在批处理模式下使用 `efax`

```bash
#!/bin/bash
# 传真（必须提前安装了 'efax' 模块）。

EXPECTED_ARGS=2
E_BADARGS=85
MODEM_PORT="/dev/ttyS2"   # 你的电脑可能会不一样。
#                ^^^^^       PCMCIA 调制解调卡缺省端口。

if [ $# -ne $EXPECTED_ARGS ]
# 检查是不是传入了适当数量的命令行参数。
then
   echo "Usage: `basename $0` phone# text-file"
   exit $E_BADARGS
fi


if [ ! -f "$2" ]
then
  echo "File $2 is not a text file."
  #     File 不是一个正常文件或者文件不存在。
  exit $E_BADARGS
fi


fax make $2              # 根据文本文件创建传真格式文件。

for file in $(ls $2.0*)  # 连接转换后的文件。
                         # 在参数列表中使用通配符（文件名通配）。
do
  fil="$fil $file"
done

efax -d "$MODEM_PORT"  -t "T$1" $fil   # 最后使用 efax。
# 如果上面一行执行失败，尝试添加 -o1。


#  S.C. 指出，上面的 for 循环可以被压缩为
#     efax -d /dev/ttyS2 -o1 -t "T$1" $2.0*
#+ 但是这并不是一个好主意。

exit $?   # efax 同时也会将诊断信息传递给标准输出。
```

> ![note](http://tldp.org/LDP/abs/images/note.gif) [关键字](http://tldp.org/LDP/abs/html/internal.html#KEYWORDREF) `do` 和 `done` 圈定了 for 循环代码块的范围。但是在一些特殊的情况下，也可以被 [大括号](http://tldp.org/LDP/abs/html/special-chars.html#CODEBLOCKREF) 取代。
> 
```bash
for((n=1; n<=10; n++))
# 没有 do！
{
  echo -n "* $n *"
}
# 没有 done！
>
>
# 输出：
# * 1 ** 2 ** 3 ** 4 ** 5 ** 6 ** 7 ** 8 ** 9 ** 10 *
# 并且 echo $? 返回 0，因此 Bash 并不认为这是一个错误。
>
>
echo
>
>
#  但是注意在典型的 for 循环 for n in [list] ... 中，
#+ 需要在结尾加一个分号。
>
for n in 1 2 3
{  echo -n "$n "; }
#               ^
>
>
# 感谢 Yongye 指出这一点。
```

### while 循环

`while` 循环结构会在循环顶部检测循环条件，若循环条件为真（ [退出状态](http://tldp.org/LDP/abs/html/exit-status.html#EXITSTATUSREF) 为0）则循环持续进行。与 [`for` 循环](http://tldp.org/LDP/abs/html/loops1.html#FORLOOPREF1) 不同的是，`while` 循环是在不知道循环次数的情况下使用的。

```bash
while [ condition ]
do
  command(s)...
done
```

在 `while` 循环结构中，你不仅可以使用像 `if/test` 中那样的 [括号结构](http://tldp.org/LDP/abs/html/testconstructs.html#TESTCONSTRUCTS1)，也可以使用用途更广泛的 [双括号结构](http://tldp.org/LDP/abs/html/testconstructs.html#DBLBRACKETS)（`while [[ condition ]]`）。

就像在 `for` 循环中那样，将 `do` 和循环条件放在同一行时需要加一个分号。

`while [ condition ] ; do`

在 `while` 循环中，括号结构 [并不是必须存在的](http://tldp.org/LDP/abs/html/loops1.html#WHILENOBRACKETS)。比如说 [`getopts` 结构](http://tldp.org/LDP/abs/html/internal.html#GETOPTSX)。

样例 11-15. 简单的 `while` 循环

```bash
#!/bin/bash

var0=0
LIMIT=10

while [ "$var0" -lt "$LIMIT" ]
#      ^                    ^
# 必须有空格，因为这是测试结构
do
  echo -n "$var0 "        # -n 不会另起一行
  #             ^           空格用来分开输出的数字。
  
  var0=`expr $var0 + 1`   # var0=$(($var0+1))  效果相同。
                          # var0=$((var0 + 1)) 效果相同。
                          # let "var0 += 1"    效果相同。
done                      # 还有许多其他的方法也可以达到相同的效果。

echo

exit 0
```

样例 11-16. 另一个例子

```bash
#!/bin/bash

echo
                               # 等价于：
while [ "$var1" != "end" ]     # while test "$var1" != "end"
do
  echo "Input variable #1 (end to exit) "
  read var1                    # 不是 'read $var1' （为什么？）。
  echo "variable #1 = $var1"   # 因为存在 "#"，所以需要使用引号。
  # 如果输入的是 "end"，也将会在这里输出。
  # 在结束本轮循环之前都不会再测试循环条件了。
  echo
done

exit 0
```

一个 `while` 循环可以有多个测试条件，但只有最后的那一个条件决定了循环是否终止。这是一种你需要注意到的不同于其他循环的语法。

样例 11-17. 多条件 `while` 循环

```bash
#!/bin/bash

var1=unset
previous=$var1

while echo "previous-variable = $previous"
      echo
      previous=$var1
      [ "$var1" != end ] # 记录下 $var1 之前的值。
      # 在 while 循环中有4个条件，但只有最后的那个控制循环。
      # 最后一个条件的退出状态才会被记录。
do
echo "Input variable #1 (end to exit) "
  read var1
  echo "variable #1 = $var1"
done

# 猜猜这是怎样实现的。
# 这是一个很小的技巧。

exit 0
```

就像 `for` 循环一样， `while` 循环也可以使用双圆括号结构写得像C语言那样（也可以查看[样例 8-5](http://tldp.org/LDP/abs/html/dblparens.html#CVARS)）。

样例 11-18. C语言风格的 `while` 循环

```bash
#!/bin/bash
# wh-loopc.sh: 在 "while" 循环中计数到10。

LIMIT=10                 # 循环10次。
a=1

while [ "$a" -le $LIMIT ]
do
  echo -n "$a "
  let "a+=1"
done                     # 没什么好奇怪的吧。

echo; echo

# +==============================================+

# 现在我们用C语言风格再写一次。

((a = 1))      # a=1
# 双圆括号结构允许像C语言一样在赋值语句中使用空格。

while (( a <= LIMIT ))   #  双圆括号结构，
do                       #+ 并且没有使用 "$"。
  echo -n "$a "
  ((a += 1))             # let "a+=1"
  # 是的，就是这样。
  # 双圆括号结构允许像C语言一样自增一个变量。
done

echo

# 这可以让C和Java程序猿感觉更加舒服。

exit 0
```

在测试部分，`while` 循环可以调用 [函数](http://tldp.org/LDP/abs/html/functions.html#FUNCTIONREF)。

```bash
t=0

condition ()
{
  ((t++))
  
  if [ $t -lt 5 ]
  then
    return 0  # true 真
  else
    return 1  # false 假
  fi
}

while condition
#     ^^^^^^^^^
#     调用函数循环四次。
do
  echo "Still going: t = $t"
done

# Still going: t = 1
# Still going: t = 2
# Still going: t = 3
# Still going: t = 4
```

> 和 [if 测试](http://tldp.org/LDP/abs/html/testconstructs.html#IFGREPREF) 结构一样，`while` 循环也可以省略括号。
>
```bash
while condition
do
  command(s) ...
done
```

在 `while` 循环中结合 [`read`](http://tldp.org/LDP/abs/html/internal.html#READREF) 命令，我们就得到了一个非常易于使用的 [`while read`](http://tldp.org/LDP/abs/html/internal.html#WHILEREADREF) 结构。它可以用来读取和解析文件。

```bash
cat $filename |    # 从文件获得输入。
while read line    # 只要还有可以读入的行，循环就继续。
do
  ...
done

# ==================== 摘自样例脚本 "sd.sh" =================== #

  while read value   # 一次读入一个数据。
  do
    rt=$(echo "scale=$SC; $rt + $value" | bc)
    (( ct++ ))
  done
  
  am=$(echo "scale=$SC; $rt / $ct" | bc)
  
  echo $am; return $ct   # 这个功能“返回”了2个值。
  # 注意：这个技巧在 $ct > 255 的情况下会失效。
  # 如果要操作更大的数字，注释掉上面的 "return $ct" 就可以了。
} <"$datafile"   # 传入数据文件。
```

> ![note](http://tldp.org/LDP/abs/images/note.gif) 在 `while` 循环后面可以通过 < 将标准输入 [重定位到文件](http://tldp.org/LDP/abs/html/redircb.html#REDIRREF) 中。
> `while` 循环同样可以 [通过管道](http://tldp.org/LDP/abs/html/internal.html#READPIPEREF) 传入标准输入中。

### until

与 `while` 循环相反，`until` 循环测试其顶部的循环条件，直到其中的条件为真时停止。

```bash
until [ condition-is-true ]
do
  commands(s)...
done
```

注意到，跟其他的一些编程语言不同，`until` 循环的测试条件在循环顶部。

就像在 `for` 循环中那样，将 `do` 和循环条件放在同一行时需要加一个分号。

`until[ condition-is-true ] ; do`

样例 11-19. `until` 循环

```bash
#!/bin/bash

END_CONDITION=end

until [ "$var1" = "$END_CONDITION" ]
# 在循环顶部测试条件。
do
  echo "Input variable #1 "
  echo "($END_CONDITION to exit)"
  read var1
  echo "variable #1 = $var1"
  echo
done

#                ---                   #

#  就像 "for" 和 "while" 循环一样，
#+ "until" 循环也可以写的像C语言一样。

LIMIT=10
var=0

until (( var > LIMIT ))
do  # ^^ ^     ^     ^^   没有方括号，没有 $ 前缀。
  echo -n "$var "
  (( var++ ))
done    # 0 1 2 3 4 5 6 7 8 9 10


exit 0
```

如何在 `for`，`while` 和 `until` 之间做出选择？我们知道在C语言中，在已知循环次数的情况下更加倾向于使用 `for` 循环。但是在Bash中情况可能更加复杂一些。Bash中的 `for` 循环相比起其他语言来说，结构更加松散，使用更加灵活。因此使用你认为最简单的就好。

[^1]: 迭代：重复执行一个或一组命令。通常情况下，会使用`while`或者`until`进行控制。
