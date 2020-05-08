# 第二十一章 子shell

运行一个shell脚本会启动一个新的进程，即*子shell*。

**定义**： 一个*子shell*是由一个shell（或*shell脚本*）触发的[子进程](http://tldp.org/LDP/abs/html/othertypesv.html#CHILDREF2)。

一个子shell是命令处理器（-- 在终端或者*xtrem*窗口给出提示符的*shell*）的一个独立的例子。正如你的命令在命令行提示符处被理解执行一样，一个脚本[批处理](http://tldp.org/LDP/abs/html/timedate.html#BATCHPROCREF)一组命令。每一个shell脚本运行实际上是[父](http://tldp.org/LDP/abs/html/internal.html#FORKREF)shell的一个支线进程（*子进程*）。

一个shell脚本可以自己启动多个子进程。这些子进程使得脚本进行并行处理，实际上是多个支线任务同事进行。

```bash
#!/bin/bash
# subshell-test.sh

(
# 在圆括号内，因此是一个子shell . . .
while [ 1 ]   # 无限循环.
do
  echo "Subshell running . . ."
done
)

#  脚本会永远运行，或者至少直到由Ctl-C终止。

exit $?  # 脚本结束 （但是永远无法到达这里）。



现在，运行这个脚本：
sh subshell-test.sh

另外，在脚本运行的同时， 从另一个xterm运行：
ps -ef | grep subshell-test.sh

UID       PID   PPID  C STIME TTY      TIME     CMD
500       2698  2502  0 14:26 pts/4    00:00:00 sh subshell-test.sh
500       2699  2698 21 14:26 pts/4    00:00:24 sh subshell-test.sh

          ^^^^

分析：
PID 2698, 脚本, 启动 PID 2699, 子shell.

注释: “UID ...”这一列可以通过“grep”命令筛去，但是由于说明的目的而显示在这里。
```

一般来说，脚本的一个[外部命令](http://tldp.org/LDP/abs/html/external.html#EXTERNALREF)会使得子进程产生[分叉](http://tldp.org/LDP/abs/html/internal.html#FORKREF)，[^1] 但是一个Bash内建命令不会如此。

**在圆括号内的命令列**

(命令1; 命令1; 命令3; ...)

    放在圆括号内的一列命令作为子shell运行。

子shell的变量*不能*被这个子shell内代码区块之外的部分看见。这些变量不能被[父进程](http://tldp.org/LDP/abs/html/internal.html#FORKREF)中调用，也不能被启动次子shell的shell调用。这些变量实际上是*子进程*的[局部变量](http://tldp.org/LDP/abs/html/localvar.html#LOCALREF)。

**例21-1.子shell的变量范围**

```bash
#!/bin/bash
# subshell.sh

echo

echo "We are outside the subshell."
echo "Subshell level OUTSIDE subshell = $BASH_SUBSHELL"
# Bash, 版本3，增加新变量                 $BASH_SUBSHELL 。
echo; echo

outer_variable=Outer
global_variable=
#  定义全局变量来”存储“子shell变量值。

(
echo "We are inside the subshell."
echo "Subshell level INSIDE subshell = $BASH_SUBSHELL"
inner_variable=Inner

echo "From inside subshell, \"inner_variable\" = $inner_variable"
echo "From inside subshell, \"outer\" = $outer_variable"

global_variable="$inner_variable"   #  这会允许”输出“ 一个子shell变量吗？
)

echo; echo
echo "We are outside the subshell."
echo "Subshell level OUTSIDE subshell = $BASH_SUBSHELL"
echo

if [ -z "$inner_variable" ]
then
  echo "inner_variable undefined in main body of shell"
else
  echo "inner_variable defined in main body of shell"
fi

echo "From main body of shell, \"inner_variable\" = $inner_variable"
#  $inner_variable 会显示为空白 （未初始化） 
#+ 因为定义在子shell的变量是“局部变量”。
#  有办法改正这一点吗？
echo "global_variable = "$global_variable""  # 为什么这不行？

echo

# =======================================================================

# 另外 ...

echo "-----------------"; echo

var=41                                                 # 全局变量。

( let "var+=1"; echo "\$var INSIDE subshell = $var" )  # 42

echo "\$var OUTSIDE subshell = $var"                   # 41
# 子shell内的变量操作，即使是对全局变量，不影响变量在子shell外的值！


exit 0

#  问题：
#  --------
#  一旦执行一个子shell，
#+ 是否有办法再次进入这个子shell以便修改或调用子shell的变量？ 
```

同样参看 [$BASHPID](http://tldp.org/LDP/abs/html/internalvariables.html#BASHPIDREF) 和 [例34-2](http://tldp.org/LDP/abs/html/gotchas.html#SUBPIT)。

**定义**： 变量的*范围*是指其有意义的上下文内容，在此变量*值*可以被引用。比如说，[局部变量](http://tldp.org/LDP/abs/html/localvar.html#LOCALREF1)的范围只在函数、代码区块或子shell内的相应定义范围内，而*全局*变量的范围则是其出现的整个脚本区域。

内部变量 [\$BASH_SUBSHELL](http://tldp.org/LDP/abs/html/internalvariables.html#BASHSUBSHELLREF) 指出一个子shell的嵌套层级时，而变量 [\$SHLVL](http://tldp.org/LDP/abs/html/internalvariables.html#SHLVLREF) 指示在子shell内*不变*的层级。

```bash
echo " \$BASH_SUBSHELL outside subshell       = $BASH_SUBSHELL"           # 0
  ( echo " \$BASH_SUBSHELL inside subshell        = $BASH_SUBSHELL" )     # 1
  ( ( echo " \$BASH_SUBSHELL inside nested subshell = $BASH_SUBSHELL" ) ) # 2
# ^ ^                          ***  嵌套   ***                        ^ ^

echo

echo " \$SHLVL outside subshell = $SHLVL"       # 3
( echo " \$SHLVL inside subshell  = $SHLVL" )   # 3 (不变！)
```

子shell内的路径改变不会带入到父shell中。

**例21-2. 列出用户信息**

```bash
#!/bin/bash
# allprofs.sh: 打印所有用户信息.

# 此脚本作者 Heiner Steven，由文件作者修改。

FILE=.bashrc  #  包含用户信息的文件是".profile"的原始脚本。

for home in `awk -F: '{print $6}' /etc/passwd`
do
  [ -d "$home" ] || continue    # 如果没有home目录，到下一个。
  [ -r "$home" ] || continue    # 如果没有读取权限，到下一个。
  (cd $home; [ -e $FILE ] && less $FILE)
done

# 脚本终止时， 不需要使用命令'cd'回到初始目录，因为'cd $home'只在子shell发生。

exit 0
```

一个子shell可以用来为一个命令组设定一个“特定环境”。

```bash
命令1
命令2
命令3
(
  IFS=:
  PATH=/bin
  unset TERMINFO
  set -C
  shift 5
  命令4
  命令5
  exit 3 # 只退出子shell！
)
# 父shell不受影响， 且环境保留。
命令6
命令7
```
从这里可以看出，命令 [exit](http://tldp.org/LDP/abs/html/internal.html#EXITREF) 只终止正在运行的子shell，并不终止父shell或脚本。

这样的“特定环境”的一个应用是检查一个变量是否被定义。
```bash
if (set -u; : $variable) 2> /dev/null
then
  echo "Variable is set."
fi     #  变量已在当前脚本被设定， 
       #+ 或者变量是一个Bash内部变量，
       #+ 或者变量在环境变量中（在export命令后）。

# 也可以写成  [[ ${variable-x} != x || ${variable-y} != y ]]
# 或者       [[ ${variable-x} != x$variable ]]
# 或者       [[ ${variable+x} = x ]]
# 或者       [[ ${variable-x} != x ]]
```

另一个应用是检查一个锁定文件。
```bash
if (set -C; : > lock_file) 2> /dev/null
then
  :   # lock_file不存在：没有用户运行此脚本
else
  echo "Another user is already running that script."
exit 65
fi

#  代码段作者 Stéphane Chazelas,
#+ 修改者 Paulo Marcel Coelho Aragao。
```

+

多个进程可以在不同子shell内并行执行。这样就可以将一个复杂的任务分解成多个子部分同时处理。

**例21-3. 在子shell中运行并行进程**

```bash
	(cat list1 list2 list3 | sort | uniq > list123) &
	(cat list4 list5 list6 | sort | uniq > list456) &
	# 同时合并和排列两组列表。
	# 在后台运行以确保并行执行。
	#
	# 同样效果如下
	#   cat list1 list2 list3 | sort | uniq > list123 &
	#   cat list4 list5 list6 | sort | uniq > list456 &
	
	wait   # 在子shell结束前不执行之后命令。
	
	diff list123 list456
```

向子shell的I/O重定向使用管道算符"|"，正如 ls -al | (命令)

在花括号间的代码块不会启动一个子shell。

{ 命令1； 命令2； 命令3； ...命令N； }

```bash
var1=23
echo "$var1"   # 23

{ var1=76; }
echo "$var1"   # 76
```

### **Notes**

[^1] 和 [exec](http://tldp.org/LDP/abs/html/internal.html#EXECREF) 命令一起触发的外部命令（通常）不会分叉一个子进程 / 子shell
