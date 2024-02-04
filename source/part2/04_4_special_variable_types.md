# 4.4 特殊的变量类型

### 局部变量

仅在代码块或函数中才可见的变量（参考函数章节的局部变量部分）。

### 环境变量

会影响用户及shell行为的变量。

> ![extra](http://tldp.org/LDP/abs/images/note.gif) 一般情况下，每一个进程都有自己的“环境”（environment），也就是一组该进程可以访问到的变量。从这个意义上来说，shell表现出与其他进程一样的行为。
> 
> 每当shell启动时，都会创建出与其环境对应的shell环境变量。改变或增加shell环境变量会使shell更新其自身的环境。*子进程*（由父进程执行产生）会继承*父进程*的环境变量。
> 
> ![notice](http://tldp.org/LDP/abs/images/caution.gif) 分配给环境变量的空间是有限的。创建过多环境变量或占用空间过大的环境变量有可能会造成问题。
> 
```
bash$ eval "`seq 10000 | sed -e 's/.*/export var&=ZZZZZZZZZZZZZZ/'`"
>
bash$ du
bash: /usr/bin/du: Argument list too long
```
> 注意，上面的"错误"已经在Linux内核版本号为2.6.23的系统中修复了。
> 
> （感谢 Stéphane Chazelas 对此问题的解释并提供了上面的例子。）

如果在脚本中设置了环境变量，那么这些环境变量需要被“导出”，也就是通知脚本所在的*环境*做出相应的更新。这个“导出”操作就是 `export` 命令。

> ![extra](http://tldp.org/LDP/abs/images/note.gif) 脚本只能将变量导出到子进程，即在这个脚本中所调用的命令或程序。在命令行中调用的脚本不能够将变量回传给命令行环境，即*子进程不能将变量回传给父进程*。
> 
> **定义：** 子进程（child process）是由另一个进程，即其父进程（parent process）所启动的子程序。

### 位置参数

从命令行中传递给脚本的参数[^1]：`$0, $1, $2, $3 ...`
即**命令行参数**。

`$0` 代表脚本名称，`$1` 代表第一个参数，`$2` 代表第二个，`$3` 代表第三个，以此类推[^2]。在 `$9` 之后的参数必须被包含在大括号中，如 `${10}, ${11}, ${12}`。

特殊变量 `$*` 与 `$@` 代表所有位置参数。

样例 4-5. 位置参数

```bash
#!/bin/bash

# 调用脚本时使用至少10个参数，例如
# ./scriptname 1 2 3 4 5 6 7 8 9 10
MINPARAMS=10

echo

echo "The name of this script is \"$0\"."
# 附带 ./ 代表当前目录
echo "The name of this script is \"`basename $0`\"."
# 除去路径信息（查看 'basename'）

echo

if [ -n "$1" ]              # 测试变量是否存在
then
 echo "Parameter #1 is $1"  # 使用引号转义#
fi

if [ -n "$2" ]
then
 echo "Parameter #2 is $2"
fi

if [ -n "$3" ]
then
 echo "Parameter #3 is $3"
fi

# ...

if [ -n "${10}" ]  # 大于 $9 的参数必须被放在大括号中
then
 echo "Parameter #10 is ${10}"
fi

echo "-----------------------------------"
echo "All the command-line parameters are: "$*""

if [ $# -lt "$MINPARAMS" ]
then
  echo
  echo "This script needs at least $MINPARAMS command-line arguments!"
fi

echo

exit 0
```

在位置参数中使用大括号助记符提供了一种非常简单的方式来访问传入脚本的最后一个参数。在其中会使用到间接引用。

```bash
args=$#           # 传入参数的个数
lastarg=${!args}
# 这是 $args 的一种间接引用方式

# 也可以使用:       lastarg=${!#}             (感谢 Chris Monson.)
# 这是 $# 的一种间接引用方式。
# 注意 lastarg=${!$#} 是无效的。
```

一些脚本能够根据调用时文件名的不同来执行不同的操作。要达到这样的效果，脚本需要检测 `$0`，也就是调用时的文件名[^3]。同时，也必须存在指向这个脚本所有别名的符号链接文件（symbolic links）。详情查看样例 16-2。

> ![info](http://tldp.org/LDP/abs/images/tip.gif) 如果一个脚本需要一个命令行参数但是在调用的时候却没有传入，那么这将会造成一个空变量赋值。这通常不是我们想要的。一种避免的方法是，在使用期望的位置参数时候，在赋值语句两侧添加一个额外的字符。

```bash
variable1_=$1_  # 而不是 variable1=$1
# 使用这种方法可以在没有位置参数的情况下避免产生错误。

critical_argument01=$variable1_

# 多余的字符可以被去掉，就像下面这样：
variable1=${variable1_/_/}
# 仅仅当 $variable1_ 是以下划线开头时候才会有一些副作用。
# 这里使用了我们稍后会介绍的参数替换模板中的一种。
# （将替换模式设为空等价于删除。）

# 更直接的处理方法就是先检测预期的位置参数是否被传入。
if [ -z $1 ]
then
  exit $E_MISSING_POS_PARAM
fi


#  但是，正如 Fabin Kreutz 指出的，
#+ 上面的方法会有一些意想不到的副作用。
#  更好的方法是使用参数替换：
#         ${1:-$DefaultVal}
#  详情查看第十章“操作变量”的第二节“变量替换”。
```

样例 4-6. *wh*, *whois* 域名查询

```bash
#!/bin/bash
# ex18.sh

# 在下面三个可选的服务器中进行 whois 域名查询：
# ripe.net, cw.net, radb.net

# 将这个脚本重命名为 'wh' 后放在 /usr/local/bin 目录下

# 这个脚本需要进行符号链接：
# ln -s /usr/local/bin/wh /usr/local/bin/wh-ripe
# ln -s /usr/local/bin/wh /usr/local/bin/wh-apnic
# ln -s /usr/local/bin/wh /usr/local/bin/wh-tucows

E_NOARGS=75


if [ -z "$1" ]
then
  echo "Usage: `basename $0` [domain-name]"
  exit $E_NOARGS
fi

# 检查脚本名，访问对应服务器进行查询。
case `basename $0` in    # 也可以写:    case ${0##*/} in
    "wh"       ) whois $1@whois.tucows.com;;
    "wh-ripe"  ) whois $1@whois.ripe.net;;
    "wh-apnic" ) whois $1@whois.apnic.net;;
    "wh-cw"    ) whois $1@whois.cw.net;;
    *          ) echo "Usage: `basename $0` [domain-name]";;
esac

exit $?
```

使用 `shift` 命令可以将全体位置参数向左移一位, 重新赋值。

`$1 <--- $2`, `$2 <--- $3`, `$3 <--- $4`，以此类推。

原先的 `$1` 将会消失，而 `$0`（脚本名称）不会有任何改变。如果你在脚本中使用了大量的位置参数，`shift` 可以让你不使用{大括号}助记法也可以访问超过10个的位置参数。

样例 4-7. 使用 `shift` 命令

```bash
#!/bin/bash
# shft.sh: 使用 `shift` 命令步进访问所有的位置参数。

# 将这个脚本命名为 shft.sh，然后在调用时跟上一些参数。
# 例如：
#    sh shft.sh a b c def 83 barndoor

until [ -z "$1" ]  # 直到访问完所有的参数
do
  echo -n "$1 "
  shift
done

echo               # 换行。

# 那些被访问完的参数又会怎样呢？
echo "$2"
# 什么都不会被打印出来。
# 当 $2 被移动到 $1 且没有 $3 时，$2 将会保持空。
# 因此 shift 是移动参数而非复制参数。

exit

#  可以参考 echo-params.sh 脚本，在不使用 shift 命令的情况下，
#+ 步进访问所有位置参数。
```

`shift` 命令也可以带一个参数来指明一次移动多少位。

```bash
#!/bin/bash
# shift-past.sh

shift 3    # 移动3位。
# 与 n=3; shift $n 效果相同。

echo "$1"

exit 0

# ======================== #


$ sh shift-past.sh 1 2 3 4 5
4

#  但是就像 Eleni Fragkiadaki 指出的那样，
#  如果尝试将位置参数（$#）传给 'shift'，
#  将会导致脚本错误的结束，同时位置参数也不会发送改变。
#  这也许是因为陷入了一个死循环...
#  比如：
#      until [ -z "$1" ]
#      do
#         echo -n "$1 "
#         shift 20    #  如果少于20个位置参数，
#      done           #+ 那么循环将永远不会结束。
#
#  当你不确定是否有这么多的参数时，你可以加入一个测试：
#      shift 20 || break
#               ^^^^^^^^
```

> ![extra](http://tldp.org/LDP/abs/images/note.gif) 使用 `shift` 命令同给函数传参相类似。详情查看样例 36-18。


[^1]: 函数同样也可以接受与使用位置参数。
[^2]: 是调用脚本的进程设置了 $0 参数。就是脚本的文件名。详情可以查看 `execv` 的使用手册。<br>在命令行中，$0 是shell的名称。<pre>bash$ echo $0<br>bash<br><br>tcsh% echo $0<br>tcsh</pre>
[^3]: 如果脚本被引用（sourced）执行或者被链接（symlinked）执行时会失效。安全的方法是检测变量 `$BASH_SOURCE`。

