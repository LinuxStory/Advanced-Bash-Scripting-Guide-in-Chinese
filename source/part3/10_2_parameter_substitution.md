# 10.2 参数替换

参数替换用来处理或扩展变量。

### `${parameter}`

等同于 `$parameter`，是变量 parameter 的值。在一些特定的环境下，只允许使用不易混淆的 `${parameter}` 形式。

可以用于连接变量与字符串。

```bash
your_id=${USER}-on-${HOSTNAME}
echo "$your_id"
# 
echo "Old \$PATH = $PATH"
PATH=${PATH}:/opt/bin  # 在脚本执行过程中临时在 $PATH 中加入 /opt/bin。
echo "New \$PATH = $PATH"
```

### `${parameter-default}, ${parameter:-default}`

在没有设置变量的情况下使用缺省值。

```bash
var1=1
var2=2
# 没有设置 var3。

echo ${var1-$var2}   # 1
echo ${var3-$var2}   # 2
#           ^          注意前面的 $ 前缀。



echo ${username-`whoami`}
# 如果变量 $username 没有被设置，输出 `whoami` 的结果。
```

> ![note](http://tldp.org/LDP/abs/images/note.gif) `${parameter-default}` 与 `${parameter:-default}` 的作用几乎相同，唯一不同的情况就是当变量 parameter 已经被声明但值为空时。

```bash
#!/bin/bash
# param-sub.sh

# 无论变量的值是否为空，其是否已被声明决定了缺省设置的触发。

username0=
echo "username0 has been declared, but is set to null."
echo "username0 = ${username0-`whoami`}"
# 将不会输出 `whoami` 的结果。

echo

echo username1 has not been declared.
echo "username1 = ${username1-`whoami`}"
# 将会输出 `whoami` 的结果。

username2=
echo "username2 has been declared, but is set to null."
echo "username2 = ${username2:-`whoami`}"
#                            ^
# 因为这里是 :- 而不是 -，所以将会输出 `whoami` 的结果。
# 与上面的 username0 比较。


# 

# 再来一次：

variable=
# 变量已被声明，但其值为空。

echo "${varibale-0}"    # 没有输出。
echo "${variable:-1}"   # 1
#               ^

unser variable

echo "${variable-2}"    # 2
echo "${variable:-3}"   # 3

exit 0
```

当传入的命令行参数的数量不足时，可以使用这种缺省参数结构。

```bash
DEFAULT_FILENAME=generic.data
filename=${1:-$DEFAULT_FILENAME}
# 如果没有其他特殊情况，下面的代码块将会操作文件 "generic.data"。
# 代码块开始
# ...
# ...
# ...
# 代码块结束



# 摘自样例 "hanoi2.bash"：
DISKS=${1:-E_NOPARAM}   # 必须指定碟子的个数。
#  将 $DISKS 设置为传入的第一个命令行参数，
#+ 如果没有传入第一个参数，则设置为 $E_NOPARAM。
```

可以查看 [样例 3-4](http://tldp.org/LDP/abs/html/special-chars.html#EX58)，[样例 31-2](http://tldp.org/LDP/abs/html/zeros.html#EX73) 和 [样例 A-6](http://tldp.org/LDP/abs/html/contributed-scripts.html#COLLATZ)。

可以同 [使用与链设置缺省命令行参数](http://tldp.org/LDP/abs/html/list-cons.html#ANDDEFAULT) 做比较。

### `${parameter=default}, ${parameter:=default}`

在没有设置变量的情况下，将其设置为缺省值。

两种形式的作用几乎相同，唯一不同的情况与上面类似，就是当变量 parameter 已经被声明但值为空时。[^1]

```bash
echo ${var=abc}   # abc
echo ${vat=xyz}   # abc
# $var 已经在第一条语句中被赋值为 abc，因此第二条语句将不会改变它的值。
```

### `${parameter+alt_value}, ${parameter:+alt_value}`

如果变量已被设置，使用 alt_value，否则使用空值。

两种形式的作用几乎相同，唯一不同的情况就是当变量 parameter 已经被声明但值为空时，看下面的例子。

```bash
echo "###### \${parameter+alt_value} ########"
echo

a=${param1+xyz}
echo "a = $a"      # a =

param2=
a=${param2+xyz}
echo "a = $a"      # a = xyz

param3=123
a=${param3+xyz}
echo "a = $a"      # a = xyz

echo
echo "###### \${parameter:+alt_value} ########"
echo

a=${param4:+xyz}
echo "a = $a"      # a =

param5=
a=${param5:+xyz}
echo "a = $a"      # a =
# 不同于 a=${param5+xyz}

param6=123
a=${param6:+xyz}
echo "a = $a"      # a = xyz
```

### `${parameter?err_msg}, ${parameter:?err_msg}`

如果变量已被设置，那么使用原值，否则输出 err_msg 并且终止脚本，返回 [错误码](http://tldp.org/LDP/abs/html/exit-status.html#EXITSTATUSREF) 1。

两种形式的作用几乎相同，唯一不同的情况与上面类似，就是当变量 parameter 已经被声明但值为空时。

样例 10-7. 如何使用变量替换和错误信息

```bash
#!/bin/bash

# 检查系统环境变量。
# 这是一种良好的预防性维护措施。
# 如果控制台用户的名称 $USER 没有被设置，那么主机将不能够识别用户。

: ${HOSTNAME?} ${USER?} ${HOME?} ${MAIL?}
  echo
  echo "Name of the machine is $HOSTNAME."
  echo "You are $USER."
  echo "Your home directory is $HOME."
  echo "Your mail INBOX is located in $MAIL."
  echo
  echo "If you are reading this message,"
  echo "critcial environmental variables have been set."
  echo
  echo
  
# ------------------------------------------------------

# ${variablename?} 结构统一可以检查脚本中的变量是否被设置。

ThisVariable=Value-of-ThisVariable
# 顺带一提，这个字符串的值可以被设置成名称中不可以使用的禁用字符。
: ${ThisVariable?}
echo "Value of ThisVariable is $ThisVariable."

echo; echo


: ${ZZXy23AB?"ZZXy23AB has not been set."}
# 因为 ZZXy23AB 没有被设置，所以脚本会终止同时显示错误消息。

# 你可以指定错误消息。
# : ${variablename?"ERROR MESSAGE"}


# 与这些结果相同:  dummy_variable=${ZZXy23AB?}
#                 dummy_variable=${ZZXy23AB?"ZZXy23AB has not been set."}
#
#                 echo ${ZZXy23AB?} >/dev/null

# 将上面这些检查变量是否被设置的方法同 "set -u" 作比较。



echo "You will not see this message, because script already terminated."

HERE=0
exit $HERE   # 将不会从这里退出。

#  事实上，这个脚本将会返回退出码（echo $?）1。
```

样例 10-8. 参数替换与 "usage" 消息

```bash
#!/bin/bash
# usage-message.sh

: ${1?"Usage: $0 ARGUMENT"}
# 如果命令行参数缺失，脚本将会在这里结束，并且返回下面的错误信息。
#    usage-message.sh: 1: Usage: usage-message.sh ARGUMENT

echo "These two lines echo only if command-line parameter given."
echo "command-line parameter = \"$1\""

exit 0  # 仅当命令行参数存在是才会从这里退出。

# 在传入和未传入命令行参数的情况下查看退出状态。
# 如果传入了命令行参数，那么 "$?" 的结果是0。
# 如果没有，那么 "$?" 的结果是1。
```

参数替换用来处理或扩展变量。下面的表达式是对 `expr` 处理字符串的操作的补足（查看样例 16-9）。这些特殊的表达式通常养来解析文件的路径名。

### 变量长度 / 删除子串

#### `${#var}`

字符串的长度（`$var` 中字符的个数）。对任意 [数组](http://tldp.org/LDP/abs/html/arrays.html#ARRAYREF) array，`${#array}` 返回数组中第一个元素的长度。

> ![note](http://tldp.org/LDP/abs/images/note.gif) 以下情况例外：
>
> * `${#*}` 和 `${#@}` 返回位置参数的个数。
> * 任意数组 array，`${#array[*]}` 和 `${#array[@]}` 返回数组中元素的个数。

样例 10-9. 变量长度

```bash
#!/bin/bash
# length.sh

E_NO_ARGS=65

if [ $# -eq 0 ]  # 脚本必须传入参数。
then
  echo "Please invoke this script with one or more command-line arguments."
  exit $E_NO_ARGS
fi

var01=abcdEFGH28ij
echo "var01 = ${var01}"
echo "Length of var01 = ${#var01}"
# 现在我们尝试加入空格。
var02="abcd EFGH28ij"
echo "var02 = ${var02}"
echo "Length of var02 = ${#var02}"

echo "Number of command-line arguments passed to script = ${#@}"
echo "Number of command-line arguments passed to script = ${#*}"

exit 0
``` 

#### `${var#Pattern}, ${var##Pattern}`

`${var#Pattern}` 删除 `$var` 前缀部分匹配到的最短长度的 `$Pattern`。

`${var##Pattern}` 删除 `$var` 前缀部分匹配到的最长长度的 `$Pattern`。

摘自 [样例 A-7](http://tldp.org/LDP/abs/html/contributed-scripts.html#DAYSBETWEEN) 的例子：

```bash
# 函数摘自样例 "day-between.sh"。
# 删除传入的参数中的前缀0。

strip_leading_zero () #  删除传入参数中可能存在的
{                     #+ 前缀0。
  return=${1#0}       #  "1" 代表 "$1"，即传入的参数。
}                     #  从 "$1" 中删除 "0"。
```

下面是由 Manfred Schwarb 提供的上述函数的改进版本：

```bash
strip_leading_zero2 () # 删除前缀0，
{                      # 否则 Bash 会将其解释为8进制数。
  shopt -s extglob     # 启用扩展通配特性。
  local val=${1##+(0)} # 使用本地变量，匹配前缀中所有的0。
  shopt -u extglob     # 禁用扩展通配特性。
  _strip_leading_zero2=${var:-0}
                       # 如果输入的为0，那么返回 0 而不是 ""。
```

另外一个样例：

```bash
echo `basename $PWD`        # 当前工作目录的目录名。
echo "${PWD##*/}"           # 当前工作目录的目录名。
echo
echo `basename $0`          # 脚本名。
echo $0                     # 脚本名。
echo "${0##*/}"             # 脚本名。
echo
filename=test.data
echo "${filename##*.}"      # data
                            # 文件扩展名。
```

#### `${var%Pattern}, ${var%%Pattern}`

`${var%Pattern}` 删除 `$var` 后缀部分匹配到的最短长度的 `$Pattern`。

`${var%%Pattern}` 删除 `$var` 后缀部分匹配到的最长长度的 `$Pattern`。

在 Bash 的 [第二个版本](http://tldp.org/LDP/abs/html/bashver2.html#BASH2REF) 中增加了一些额外的选择。

样例 10-10. 参数替换中的模式匹配

```bash
#!/bin/bash
# patt-matching.sh

# 使用 # ## % %% 参数替换操作符进行模式匹配

var1=abcd12345abc6789
pattern1=a*c  # 通配符 * 可以匹配 a 与 c 之间的任意字符

echo
echo "var1 = $var1"           # abcd12345abc6789
echo "var1 = ${var1}"         # abcd12345abc6789
                              # （另一种形式）
echo "Number of characters in ${var1} = ${#var1}"
echo

echo "pattern1 = $pattern1"   # a*c  (匹配 'a' 与 'c' 之间的一切)
echo "--------------"
echo '${var1#$pattern1}  =' "${var1#$pattern1}"    #         d12345abc6789
# 匹配到首部最短的3个字符                                   abcd12345abc6789
#             ^                                           |-|
echo '${var1##$pattern1} =' "${var1##$pattern1}"   #                  6789
# 匹配到首部最长的12个字符                                  abcd12345abc6789
#             ^                                           |----------|

echo; echo; echo

pattern2=b*9            # 匹配 'b' 与 '9' 之间的任意字符
echo "var1 = $var1"     # 仍旧是 abcd12345abc6789
echo
echo "pattern2 = $pattern2"
echo "--------------"
echo '${var1%pattern2}  =' "${var1%$pattern2}"     #     abcd12345a
# 匹配到尾部最短的6个字符                                  abcd12345abc6789
#             ^                                                    |----|
echo '${var1%%pattern2} =' "${var1%%$pattern2}"    #     a
# 匹配到尾部最长的12个字符                                 abcd12345abc6789
#             ^                                           |-------------|

# 牢记 # 与 ## 是从字符串左侧开始，
#      % 与 %% 是从右侧开始。

echo

exit 0
```

样例 10-11. 更改文件扩展名：

```bash
#!/bin/bash
# rfe.sh: 更改文件扩展名。
#
#         rfe old_extension new_extension
#
# 如：
# 将当前目录下所有 *.gif 文件重命名为 *.jpg，
#         rfe gif jpg


E_BADARGS=65

case $# in
  0|1)             # 竖线 | 在这里表示逻辑或关系。
  echo "Usage: `basename $0` old_file_suffix new_file_suffix"
  exit $E_BADARGS  # 如果只有0个或1个参数，那么退出脚本。
  ;;
esac


for filename in *.$1
# 遍历以第一个参数作为后缀名的文件列表。
do
  mv $filename ${filename%$1}$2
  # 删除文件后缀名，增加第二个参数作为后缀名。
done

exit 0
```

### 变量扩展 / 替换子串

下面这些结构采用自 ksh。

#### `${var:pos}`

扩展为从偏移量 pos 处截取的变量 var。

#### `${var:pos:len}`

扩展为从偏移量 pos 处截取变量 var 最大长度为 len 的字符串。

#### `${var/Pattern/Replacement}`

替换 var 中第一个匹配到的 Pattern 为 Replacement。

如果 Replacement 被省略，那么匹配到的第一个 Pattern 将被替换为空，即删除。

#### `${var//Pattern/Replacement}`

全局替换。替换 var 中所有匹配到的 Pattern 为 Replacement。

跟上面一样，如果 Replacement 被省略，那么匹配到的所有 Pattern 将被替换为空，即删除。

样例 10-12. 使用模式匹配解析任意字符串

```bash
#!/bin/bash

var1=abcd-1234-defg
echo "var1 = $var1"

t=${var1#*-*}
echo "var1 (with everything, up to and including first - stripped out) = $t"
#  t=${var1#*-} 效果相同，
#+ 因为 # 只匹配最短的字符串，
#+ 并且 * 可以任意匹配，其中也包括空字符串。
# （感谢 Stephane Chazelas 指出这一点。）

t=${var##*-*}
echo "If var1 contains a \"-\", returns empty string...   var1 = $t"


t=${var1%*-*}
echo "var1 (with everything from the last - on stripped out) = $t"

echo

# -------------------------------------------
path_name=/home/bozo/ideas/thoughts/for.today
# -------------------------------------------
echo "path_name = $path_name"
t=${path_name##/*/}
echo "path_name, stripped of prefixes = $t"
# 在这里与 t=`basename $path_name` 效果相同。
#  t=${path_name%/}; t=${t##*/}  是更加通用的方法，
#+ 但有时仍旧也会出现问题。
#  如果 $path_name 以换行结束，那么 `basename $path_name` 将会失效，
#+ 但是上面这种表达式却可以。
# （感谢 S.C.）

t=${path_name%/*.*}
# 同 t=`dirname $path_name` 效果相同。
echo "path_name, stripped of suffixes = $t"
# 在一些情况下会失效，比如 "../", "/foo////", # "foo/", "/"。
#  在删除后缀时，尤其是当文件名没有后缀，目录名却有后缀时，
#+ 事情会变的非常复杂。
# （感谢 S.C.）

echo

t=${path_name:11}
echo "$path_name, with first 11 chars stripped off = $t"
t=${path_name:11:5}
echo "$path_name, with first 11 chars stripped off, length 5 = $t"

echo

t=${path_name/bozo/clown}
echo "$path_name with \"bozo\" replaced by \"clown\" = $t"
t=${path_name/today/}
echo "$path_name with \"today\" deleted = $t"
t=${path_name//o/O}
echo "$path_name with all o's capitalized = $t"
t=${path_name//o/}
echo "$path_name with all o's deleted = $t"

exit 0
```

#### `${var/#Pattern/Replacement}`

替换 var 前缀部分匹配到的 Pattern 为 Replacement。

#### `${var/%Pattern/Replacement}`

替换 var 后缀部分匹配到的 Pattern 为 Replacement。

样例 10-13. 在字符串首部或尾部进行模式匹配

```bash
#!/bin/bash
# var-match.sh:
# 演示在字符串首部或尾部进行模式替换。

v0=abc1234zip1234abc    # 初始值。
echo "v0 = $v0"         # abc1234zip1234abc
echo

# 在字符串首部进行匹配
v1=${v0/#abc/ABCDEF}    # abc1234zip123abc
                        # |-|
echo "v1 = $v1"         # ABCDEF1234zip1234abc
                        # |----|
                        
# 在字符串尾部进行匹配
v2=${v0/%abc/ABCDEF}    # abc1234zip123abc
                        #              |-|
echo "v2 = $v2"         # abc1234zip1234ABCDEF
                        #               |----|
                        
echo

#  --------------------------------------------
#  必须在字符串的最开始或者最末尾的地方进行匹配，
#+ 否则将不会发生替换。
#  --------------------------------------------
v3=${v0/#123/000}       # 虽然匹配到了，但是不在最开始的地方。
echo "v3 = $v3"         # abc1234zip1234abc
                        # 没有替换。
v4=${v0/%123/000}       # 虽然匹配到了，但是不在最末尾的地方。
echo "v4 = $v4"         # abc1234zip1234abc
                        # 没有替换。

exit 0
```

#### `${!varprefix*}, ${!varprefix@}`

匹配先前声明过所有以 varprefix 作为变量名前缀的变量。

```bash
# 这是带 * 或 @ 的间接引用的一种变换形式。
# 在 Bash 2.04 版本中加入了这个特性。

xyz23=whatever
xyz23=

a=${!xyz*}         #  扩展为声明变量中以 "xyz"
# ^ ^   ^           + 开头变量名。
echo "a = $a"      #  a = xyz23 xyz24
a=${!xyz@}         #  同上。
echo "a = $a"      #  a = xyz23 xyz24

echo "---"

abc23=something_else
b=${!abc*}
echo "b = $b"      #  b = abc23
c=${!b}            #  这是我们熟悉的间接引用的形式。
echo $c            #  something_else
```

[^1]: 如果在非交互的脚本中，`$parameter` 为空，那么程序将会终止，并且返回 [错误码 127](http://tldp.org/LDP/abs/html/exitcodes.html#EXITCODESREF)（意为“找不到命令”）。
