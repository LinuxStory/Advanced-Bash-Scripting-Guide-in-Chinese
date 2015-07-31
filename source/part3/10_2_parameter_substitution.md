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





[^1]: 如果在非交互的脚本中，`$parameter` 为空，那么程序将会终止，并且返回 [错误码 127](http://tldp.org/LDP/abs/html/exitcodes.html#EXITCODESREF)（意为“找不到命令”）。
