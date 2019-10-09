# 9.1 内部变量

## 内建变量

影响 Bash 脚本行为的变量。

### $BASH

Bash程序的路径。

```bash
bash$ echo $BASH
/bin/bash
```

### $BASH_ENV

这个环境变量会指向一个 Bash 启动文件，该文件在脚本被调用时会被读取。

### $BASH_SUBSHELL

该变量用于提示所处的 subshell 层级。这是在 Bash version 3 中被引入的新特性。

具体用法可以参考 [样例21-1](http://tldp.org/LDP/abs/html/subshells.html#SUBSHELL)。

### $BASHPID

当前 Bash 进程实例的进程ID号。虽然与 `$$` 变量不一样，但是通常它们会给出相同的结果。

```bash
bash4$ echo $$
11015


bash4$ echo $BASHPID
11015


bash4$ ps ax | grep bash4
11015 pts/2    R      0:00 bash4
```

然而...

```bash
#!/bin/bash4

echo "\$\$ outside of subshell = $$"                              # 9602
echo "\$BASH_SUBSHELL  outside of subshell = $BASH_SUBSHELL"      # 0
echo "\$BASHPID outside of subshell = $BASHPID"                   # 9602

echo

( echo "\$\$ inside of subshell = $$"                             # 9602
  echo "\$BASH_SUBSHELL inside of subshell = $BASH_SUBSHELL"      # 1
  echo "\$BASHPID inside of subshell = $BASHPID" )                # 9603
  #  注意 $$ 总是返回父进程的 PID。
```

### $BASH_VERSINFO[n]

这是一个6个元素的数组，其中包含了已经安装的 Bash 的版本信息。该变量与变量 `$BASH_VERSION` 类似，但是更加详细。

```bash
# Bash 版本信息：

for n in 0 1 2 3 4 5
do
  echo "BASH_VERSINFO[$n] = ${BASH_VERSINFO[$n]}"
done

# BASH_VERSINFO[0] = 3                      # 主版本号
# BASH_VERSINFO[1] = 00                     # 次版本号
# BASH_VERSINFO[2] = 14                     # 补丁号
# BASH_VERSINFO[3] = 1                      # 构建版本号
# BASH_VERSINFO[4] = release                # 发行状态
# BASH_VERSINFO[5] = i386-redhat-linux-gnu  # 架构
                                            # (与 $MACHTYPE 相同)
```

### $BASH_VERSION

已经安装的 Bash 的版本信息。

```bash
bash$ echo $BASH_VERSION
3.2.25(1)-release
```

```bash
tcsh% echo $BASH_VERSION
BASH_VERSION: Undefined variable.
```

利用 `$BASH_VERSION` 来判断运行的是哪个 shell 是一个不错的方法，因为变量 `$SHELL` 并不总是能够给出正确的答案。

### $CDPATH

变量指定 `cd` 命令可以搜索的路径，路径之间用冒号进行分隔。该变量的功能类似于指定可执行文件搜索路径的变量 `$PATH`。可以在本地文件 `~/.bashrc` 中设置该变量。

```bash
bash$ cd bash-doc
bash: cd : bash-doc: No such file or directory


bash$ CDPATH=/usr/share/doc
bash$ cd bash-doc
/usr/share/doc/bash-doc


bash$ echo $PWD
/usr/share/doc/bash-doc
```

### $DIRSTACK

指代目录栈中顶部的值[^1]，目录栈由命令 `pushd` 和 `popd` 控制。

该变量相当于命令 `dirs`，但是 `dirs` 命令会显示整个目录栈。

### $EDITOR

脚本所调用的默认编辑器，通常是 `vi` 或是 `emcas`。

### $EUID

有效用户ID。

有效用户ID（EUID）是指当前用户正在使用的用户ID，可以通过 `su` 命令修改。

{% hint style="warning" %}

`$EUID` 与 `$UID` 并不总是相同的。

{% endhint %}

### $FUNCNAME

当前运行函数的函数名。

``` bash
xyz23 ()
{
  echo "$FUNCNAME now executing."  # xyz2 now executing.
}

xyz23

echo "FUNCNAME = $FUNCNAME"        # FUNCNAME =
                                   # 如果在函数外则为空值。 
```

可以参考 [样例 A-50]()。

### $GLOBIGNORE

在[文件匹配](../part5/18_2_globbing.md)时所忽略的文件名模式列表。

### $GROUPS

当前用户所属的用户组。

该变量存储了当前用户所归属的用户组ID列表，是一个数组。内容与记录在文件 `/etc/passwd` 和文件 `/etc/group` 中的一致。

```bash
root# echo $GROUPS
0


root# echo ${GROUPS[1]}
1


root# echo ${GROUPS[5]}
6
```

### $HOME

当前用户的主目录，其值通常为 `/home/username` （参考 [样例 10-7]()）。

### $HOMENAME

系统启动的初始化脚本通过命令 `hostname` 给系统分配主机名。而函数 `gethostname()` 则是给 Bash 的内部变量 `$HOSTNAME` 赋值。可以参考 [样例 10-7]()。

### $HOSTTYPE

主机类型。

类似变量 [`$MACHTYPE`]()，用于识别系统硬件信息。

```bash
bash$ echo $HOSTTYPE
i686
```

### $IFS

内部字段分隔符。

该变量决定了 Bash 在解析字符串时如何去识别 [字段]() 或单词边界。

`$IFS` 的缺省值是空白符（空格，制表符以及换行符），但其可以被修改。例如你在处理逗号分隔的文件时可以将其设置为逗号。需要注意 [`$*`]() 使用保存在 `$IFS` 中的第一个字符。可以参考 [样例 5-1]()。

```bash
bash$ echo "$IFS"

(当 $IFS 设置为缺省值时，显示空行。)


bash$ echo "$IFS" | cat -vte
 ^I$
 $
(显示空白符：首先是一个空格，然后是 ^I [水平制表符]，
 然后是换行符，最后在末尾显示 "$"。)


bash$ bash -c 'set w x y z; IFS=":-;"; echo "$*"'
w:x:y:z
(从字符串中解析命令，然后将命令参数分配给位置参数。)
```

通过设置 `$IFS` 来忽略文件路径名中空格带来的影响。

```bash
IFS="$(printf '\n\t')"   # 按 David Wheeler 所述。
```

{% hint style="warning" %}

相比于其他字符，变量 `$IFS` 在处理空白符时有所不同。

#### 样例 9-1. $IFS 与空白符

```bash
#!/bin/bash
# ifs.sh


var1="a+b+c"
var2="d-e-f"
var3="g,h,i"

IFS=+
# 加号会被解析成分隔符。
echo $var1     # a b c
echo $var2     # d-e-f
echo $var3     # g,h,i

echo

IFS="-"
# 恢复对加号的默认解析。
# 现在减号会被解析成分隔符。
echo $var1     # a+b+c
echo $var2     # d e f
echo $var3     # g,h,i

echo

IFS=","
# 现在逗号会被解析成分隔符。
# 恢复对减号的默认解析。
echo $var1     # a+b+c
echo $var2     # d-e-f
echo $var3     # g h i

echo

IFS=" "
# 现在空格会被解析成分隔符。
# 逗号恢复成默认解析。
echo $var1     # a+b+c
echo $var2     # d-e-f
echo $var3     # g,h,i

# ======================================================== #

# 然而...
# $IFS 处理空白符的方式不同其他字符。

output_args_one_per_line()
{
  for arg
  do
    echo "[$arg]"
  done #  ^    ^   为了获得更好的视觉体验，把参数放到了括号里。
}

echo; echo "IFS=\"  \""
echo "-------"

IFS=" "
var=" a  b c   "
#    ^ ^^   ^^^
output_args_one_per_line $var  # output_args_one_per_line `echo " a  b c   "`
# [a]
# [b]
# [c]


echo; echo "IFS=:"
echo "-----"

IFS=:
var=":a::b:c:::"               # 与上面一样的模式，
#    ^ ^^   ^^^                #+ 仅仅是将 " " 替换成了 ":" ...
output_args_one_per_line $var
# []
# [a]
# []
# [b]
# [c]
# []
# []

# 注意那些“空的”括号。
# 同样的情况也会出现在 awk 命令所使用的 "FS" 字段分隔符中。


echo

exit
```

{% endhint %}

（非常感谢 Stéphane Chazelas 提供了上面的样例并做出的详细说明。）

也可以参考 [样例 16-41]()，[样例 11-8]() 和 [样例19-14]()，获取更多使用 `$IFS` 的技巧。

### $IGNOREEOF

忽略 EOF：用于指示 Shell 在注销前需要忽略多少个文件结束符(EOF，contrl-D)。

### $LC_COLLATE

经常会在文件 [`.bashrc`]() 或是文件 `/etc/profile` 中被设置。该变量控制文件名扩展和模式匹配中的排序顺序。如果设置不得当，`LC_COLLATE` 将会导致 [文件名匹配]() 中出现非预期结果。


{% hint style="info" %}

在 Bash 2.05 版本之后，文件名匹配在不再区分中括号中字母的大小写。例如 `ls [A-M]*` 将会同时匹配 `File1.txt` 和 `file1.txt` 两个文件。如果想要恢复成之前的模式，则需要在文件 `/etc/profile` 或文件 `~/.bashrc` 中通过语句 `export LC_COLLATE=C` 设置 `LC_COLLATE` 的值为 `C`。

{% endhint %}

### $LC_CTYPE

这个内部变量控制在 [文件匹配]() 和模式匹配中的字符解析行为。

### $LINENO

该变量记录了其在脚本中被使用时所处行的行号。该变量只有在被使用时才有意义，在调试过程中非常有用。

```bash
# *** 调试部分起始 ***
last_cmd_arg=$_  # 保存最后的命令。

echo "At line number $LINENO, variable \"v1\" = $v1"
echo "Last command argument processed = $last_cmd_arg"
# *** 调试部分终止 ***
```

### $MACHTYPE

设备类型。

识别系统硬件。

```bash
bash$ echo $MACHTYPE
i686
```

### $OLDPWD

上一个工作目录(OLD-Print-Working-Directory)，也就是之前所在的目录。

### $OSTYPE

操作系统类型。

```bash
bash$ echo $OSTYPE
linux
```

### $PATH

可执行文件搜索路径，其值通常包含 `/usr/bin`，`/usr/X11R6/bin/`，`/usr/local/bin` 等路径。

给定一个命令，shell就会自动从搜索路径包含的目录中利用哈希表搜索该可执行命令。而搜索路径就保存在 [环境变量]() `$PATH` 中，其中包含的一系列目录则通过冒号进行分隔。通常情况下，`$PATH` 会定义在文件 `/etc/profile` 或文件 [`~/.bashrc`]() 中（参考 [附录 H]()）。

```bash
bash$ echo $PATH
/bin:/usr/bin:/usr/local/bin/:/usr/X11R6/bin:/sbin:/usr/sbin
```

`PATH=${PATH}:/opt/bin` 表示添加目录 `/opt/bin` 到当前的搜索路径中。在脚本中可以通过这种方式临时添加目录到搜索路径。而当脚本结束时，`$PATH` 就会恢复到原始值（类似于脚本这样的子进程所作出的修改，不会影响到例如 shell 这样的父进程的环境）。

{% hint style="info" %}

基于安全考虑，通常在 `$PATH` 中会省略当前工作目录 `./`。

{% endhint %}

### $PIPESTATUS

该 [数组]() 变量保存了最后运行的前台 [管道]() 的 [退出状态(es)]()。

```bash
bash$ echo $PIPESTATUS
0

bash$ ls -al | bogus_command
bash: bogus_command: command not found
bash$ echo ${PIPESTATUS[1]}
127

bash$ ls -al | bogus_command
bash: bogus_command: command not found
bash$ echo $?
127
```

`$PIPESTATUS` 数组中的每一个元素都代表了该管道中相对应命令的退出状态。`$PIPESTATUS[0]` 表示管道中第一个命令的退出状态，`$PIPESTATUS[1]` 表示第二个命令的退出状态，以此类推。

{% hint style="warning" %}

在Bash 3.0 以下版本的登录shell中，变量 `$PIPESTATUS` 可能会包含一个不正确的 0 值。
 
```bash
tcsh% bash

bash$ who | grep nobody | sort
bash$ echo ${PIPESTATUS[*]}
0
```

如果脚本包含了上述代码，应该得到期望的输出是 0 1 0。

感谢 Wayne Pollock 指出了这个问题并提供了上述的样例。

{% endhint %}

{% hint style="info" %}

在某些场景下，`$PIPESTATUS` 变量将会产生非预期结果。

```bash
bash$ echo $BASH_VERSION
3.00.14(1)-release

bash$ ls | bogus_command | wc
bash: bogus_command: command not found
 0       0       0

bash$ echo ${PIPESTATUS[@]}
141 127 0
```

Chet Ramey 把上述非预期结果的原因归咎于 [`ls`]() 命令的行为。如果 `ls` 将结果输出到没有被读取的管道上，产生的 SIGPIPE 信号将会终止 `ls` 命令，同时其 [退出状态]() 从期望的 0 变为 141。而同样的情况也会发生在命令 `tr` 中。

{% endhint %}

{% hint style="info" %}

`$PIPESTATUS` 是一个易失的变量。该变量需要在目标管道执行完成后，且其他任何命令执行之前去捕获。

```bash
bash$ ls | bogus_command | wc
bash: bogus_command: command not found
 0       0                0

bash$ echo ${PIPESTATUS[@]}
0 127 0

bash$ echo ${PIPESTATUS[@]}
0
```

{% endhint %}

{% hint style="info" %}

在 `$PIPESTATUS` 不能给出所期望的信息的情况下，使用 [pipeline 选项]() 可能会有帮助。

{% endhint %}

### $PPID

一个进程的 `$PPID` 即该进程的父进程的进程ID(pid)。[^2]

可以与命令 [`pidof`]() 进行比较。

### $PROMPT_COMMAND

该变量存储在主提示符 `$PS1` 显示之前所需要执行的命令。

### $PS1

主提示符，即在命令行中显示的提示符。

### $PS2

次要提示符，当需要额外输入时出现的提示符。默认显示为 `>`。

### $PS3

三级提示符，显示在 `select` 循环中（参考 [样例 11-30]()）。

### $PS4

四级提示符，当使用 `-x [verbose trace]` [选项]() 调用脚本时显示的提示符。默认显示为 `+`。

其可以作为调试的辅助手段，把一些诊断信息显示在 `$PS4` 中可能会有帮助。

```bash
P4='$(read time junk < /proc/$$/schedstat; echo "@@@ $time @@@ " )'
# 根据 Erik Brandsberg 提供的建议。
set -x
# 可以在后面写各种命令...
```

### $PWD

工作目录（你当前所在的目录）。

该变量是内建命令 [`pwd`]() 的翻版。

```bash
#!/bin/bash

E_WRONG_DIRECTORY=85

clear # 清空屏幕。

TargetDirectory=/home/bozo/projects/GreatAmericanNovel

cd $TargetDirectory
echo "Deleting stale files in $TargetDirectory."

if [ "$PWD" != "$TargetDirectory" ]
then    # 小心不要偶然清空了错误的目录。
  echo "Wrong directory!"
  echo "In $PWD, rather than $TargetDirectory!"
  echo "Bailing out!"
  exit $E_WRONG_DIRECTORY
fi

rm -rf *
rm .[A-Za-z0-9]*    # 删除隐藏文件。
# rm -f .[^.]* ..?*   删除那些以多个点开头的文件。
# (shopt -s dotglob; rm -f *)   这样写也可以。
# 感谢 S.C. 提出这点。

#  文件名可以包含ASCII码中范围为 0-255 的所有字符，
#+ 除了字符 "/"。
#  删除以一些特殊字符开头的文件，例如 -
#+ 留作练习。（提示： rm ./-weirdname 或者 rm -- -weirdname）
result=$?   # 删除操作的结果。如果删除成功，值为0。

echo
ls -al              # 是不是还有剩余没有删除的文件？
echo "Done."
echo "Old files deleted in $TargetDirectory."
echo

# 如果有其他需要，在这里完成。

exit $result
```

### $REPLY

当没有给 [`read`]() 命令提供接收参数时的默认接收参数。该变量同样适用于 [`select`]() 菜单接收用户输入值的场景，需要注意的是，用户只需要输入菜单项的编号，而不需要输入完整的菜单项内容。

```bash
#!/bin/bash
# reply.sh

# REPLY 是 'read' 命令的默认接收参数。

echo
echo -n "What is your favorite vegetable? "
read

echo "Your favorite vegetable is $REPLY."
#  当且仅当 'read' 命令没有接收参数的时候，
#+ REPLY 才能保存最近一次 'read' 命令接收的值。

echo
echo -n "What is your favorite fruit? "
read fruit
echo "Your favorite fruit is $fruit."
echo "but..."
echo "Value of \$REPLY is still $REPLY."
#  因为变量 $fruit 接收了新一次 "read" 命令所读入的值，
#+ 所以 $REPLY 仍旧存储的是上一次接收的值。

echo

exit 0
```

### $SECONDS

该变量记录到目前为止脚本执行的时间，单位为秒。

```bash
#!/bin/bash

TIME_LIMIT=10
INTERVAL=1

echo
echo "Hit Control-C to exit before $TIME_LIMIT seconds."
echo

while [ "$SECONDS" -le "$TIME_LIMIT" ]
do   #   $SECONDS 是一个 shell 的内部变量。
  if [ "$SECONDS" -eq 1 ]
  then
    units=second
  else
    units=seconds
  fi
  
  echo "This script has been running $SECONDS $units."
  #  在一台性能较差或负载过重的设备上，
  #+ 这个脚本可能会偶尔跳过几个计数。
  sleep $INTERVAL
done

echo -e "\a"  # 发出蜂鸣声！

exit 0
```

### $SHELLOPTS

该只读变量记录了 shell 中已启用的 [选项]() 列表。

```bash
bash$ echo $SHELLOPTS
braceexpand:hashall:histexpand:monitor:history:interactive-comments:emacs
```

### $SHLVL

当前 shell 的层级，即嵌套了多少层 Bash [^3]。如果命令行的层级 `$SHLVL` 为 1，那么在其中执行的脚本层级则增加到 2。

{% hint style="info" %}

该变量 [不受 subshell 影响]()。当你需要指出嵌套了多少层 subshell 时，需要使用变量 [`$BASH_SUBSHELL`](#$BASH_SUBSHELL)。

{% endhint %}

### $TMOUT

如果 `$TMOUT` 被设为非 0 值 `time`，那么 shell 会在 `$time` 秒后超时，然后导致 shell 登出。

在 Bash 2.05b 版本之后，可以在脚本中将 `read` 命令与 `$TMOUT` 变量进行结合。

```bash
# 只能在 Bash 2.05b 及之后的版本中成功执行。

TMOUT=3    # 提示会在 3 秒后超时。

echo "What is your favorite song?"
echo "Quickly now, you only have $TMOUT seconds to answer!"
read song

if [ -z "$song" ]
then
  song="(no answer)"
  # 默认值。
fi

echo "Your favorite song is $song."
```

在脚本中，同样也存在其他一些实现超时功能的更复杂的方法。其中一个方法是设置一个循环的计时器，当脚本超时的时候，计时器会给脚本发送一个信号。同时，也需要一个处理信号的程序来 [捕获]()（参考 [样例 32-5]()）由循环计时器产生的中断。

#### 样例 9-2. 限时输入

```bash
#!/bin/bash
# timed-input.sh

# TMOUT=3    在新版本的 Bash 中起效。

TIMER_INTERRUPT=14
TIMELIMIT=3  # 在该实例中设置为 3 秒。
             # 同样可以设置成其他值。
             
PrintAnswer()
{
  if [ "$answer" = TIMEOUT ]
  then
    echo $answer
  else       # 不要混淆了这两个实例。
    echo "Your favorite veggie is $answer"
    kill $!  #  终止在后台运行的
             #+ 不再被需要的 TimerOn 函数。
             #  $! 代表最后一个在后台运行的作业的进程ID。
  fi

}
                
                
TimerOn()
{
  sleep $TIMELIMIT && kill -s 14 $$ &
  # 等待 3 秒，然后给脚本发送一个信号。
}


Int14Vector()
{
  answer="TIMEOUT"
  PrintAnswer
  exit $TIMER_INTERRUPT
}

trap Int14Vector $TIMER_INTERRUPT
# 我们的目的就是通过时间中断 (14) 终止程序。

echo "What is your favorite vegetable "
TimerOn
read answer
PrintAnswer


#  必须承认，这个实现限时输入的方法并不优雅。
#  但利用 "read" 命令的 "-t" 选项可以简化这个操作。
#  参考脚本 "t-out.sh"。
#  思考一下，如果不是对用户的单次输入时间进行限制，
#+ 而是对整个脚本的运行时间进行限制，应该怎么做？

#  如果你需要更优雅的写法 ...
#+ 可以考虑用 C 或者 C++ 来编写应用，
#+ 并使用其中包含的类似 'alarm' 或是 ‘setitimer' 等合适的库函数来实现计时。

exit 0
```

还有一种方法是使用 [`stty`]()。

#### 样例 9-3. 再来一次，限时输入

```bash
#!/bin/bash
# timeout.sh

#  该脚本由 Stephane Chazelas 编写，
#+ 并由本书作者修改。

INTERVAL=5                # 超时所需的时间间隔

timedout_read() {
  timeout=$1
  varname=$2
  old_tty_settings=`stty -g`
  stty -icanon min 0 time ${timeout}0
  eval read $varname      # 或者直接写成 read $varname
  stty "$old_tty_settings"
  # 参考 "stty" 的帮助页面 (man)。
}

echo; echo -n "What's your name? Quick! "
timedout_read $INTERVAL your_name

#  该脚本也许并不能在所有类型的终端上正常运行。
#  最大的超时时间间隔依赖于终端。
#+ （通常是 25.5 秒）。

echo

if [ ! -z "$your_name" ]  # 如果在超时前输入了姓名 ...
then
  echo "Your name is $your_name."
else
  echo "Timed out."
fi

echo

# 该脚本的计时行为与 "timed-input.sh" 中的计时行为有所不同，
# 该脚本的计时器会在每次按键后被重置。

exit 0
```

可能最简单的方法就是利用 [`read`]() 命令的 `-t` 选项。

#### 样例 9-4. 限时 read

```bash
#!/bin/bash
# t-out.sh [time-out]
# 从 "syngin seven" 的建议中所汲取的灵感，谢谢你们。


TIMELIMIT=4         # 4 秒

read -t $TIMELIMIT variable <&1
#                           ^^^
#  在这个实例中，只有 Bash 1.x 或 Bash 2.x 版本需要 "<&1"，
#  而在 Bash 3 及更高版本则不需要。

echo

if [ -z "$variable" ]  # 判断是否为空。
then
  echo "Timed out, variable still unset."
else
  echo "variable = $variable"
fi

exit 0
```

### $UID

用户 ID。

记录在文件 [`/etc/passwd`]() 中当前用户的用户标识号。

该 ID 表示的是当前用户的真实 ID，即使用户通过 `su` 命令临时切换至另一个用户，这个 ID 也不会改变。`$UID` 是一个只读变量，不能够被命令行或是脚本中的命令所修改，并与内建命令 [`id`]() 相对应。

#### 样例 9-5. 我是 root 用户吗？

```bash
#!/bin/bash
# am-i-root.sh:   我是否是 root 用户？

ROOT_UID=0   # Root 用户的 $UID 为 0。

if [ "$UID" -eq "$ROOT_UID" ]  # 只有真正的 "root" 用户才能经受得住考研。
then
  echo "You are root."
else
  echo "You are just an ordinary user (but mom loves you just the same)."
fi

exit 0


# ============================================================= #
# 下面的代码将不会被执行，因为脚本已经退出了。

# 另外一种判断是否是 root 用户的方法：

ROOTUSER_NAME=root

username=`id -nu`              # 或是...  username=`whoami`
if [ "$username" = "$ROOTUSER_NAME" ]
then
  echo "Rooty, toot, toot. You are root."
else
  echo "You are just a regular fella."
fi
```

还可以参考 [样例2-3]()。

{% hint style="info" %}

变量 `$ENV`，`$LOGNAME`，`$MAIL`，`$TERM`，`$USER` 以及 `$USERNAME` 并不是 Bash 的 [内建变量]()，而是在 [`Bash`]() 或系统的某个启动文件中，被设置而成的 [环境变量]()。代表当前用户登录 shell 名称的变量 `$SHELL` 是在文件 `/etc/password` 或是某个初始化脚本中被设定的，它也不是一个 Bash 的内建变量。

```bash
tcsh% echo $LOGNAME
bozo
tcsh% echo $SHELL
/bin/tcsh
tcsh% echo $TERM
rxvt

bash$ echo $LOGNAME
bozo
bash$ echo $SHELL
/bin/tcsh
bash$ echo $TERM
rxvt
```

{% endhint %}

## 位置参数

### \$0, \$1, \$2 等

位置参数。出现在从命令行传递给脚本、函数或是通过内建命令 [`set`]() 设置变量时（参考 [样例 4-5]() 或是 [样例 15-16]()）。

### $&#35; 

命令行参数[^4]或是位置参数的个数（参考 [样例 36-2]()）。

### $*

将所有的位置参数整合，视作一个单词。

{% hint style="info" %}

该参数必须是被引用的状态，`"$*"`。

{% endhint %}

### $@

该参数等同于 `$*`，但其中每个参数都是独立的被引用的字符串。也就是说，所有的参数都是被原封不动的进行传递，并没有被解析或是扩展。这意味着，参数列表中的每一个参数都被独立视为一个单词。

{% hint style="info" %}

同样，该参数必须是被引用的状态，`"$@"`。

{% endhint %}

#### 样例 9-6. 参数列表：利用 \$* 和 \$@ 列出参数

```bash
#!/bin/bash
# arglist.sh
# 在调用该脚本时需要跟上一些参数，例如 "one two three" ...

E_BADARGS=85

if [ ! -n "$1" ]
then
  echo "Usage: `basename $0` argument1 argument2 etc."
  exit $E_BADARGS
fi

echo

index=1          # 初始化计数器。

echo "Listing args with \"\$*\":"
for arg in "$*"  # 如果这里没有引用 "$*"，脚本将不会正常运行。
do
  echo "Arg #$index = $arg"
  let "index+=1"
done             # $* 将所有参数视作一个单词。
echo "Entire arg list seen as single word."

echo

index=1          # 重置计数器。
                 # 如果忘了这一步将会发生什么？
                 
echo "Listing args with \"\$@\":"
for arg in "$@"
do
  echo "Arg #$index = $arg"
  let "index+=1"
done             # $@ 将所有参数视作独立的单词。
echo "Arg list seen as separate words."

echo

index=1          # 重置计数器。

echo "Listing args with \$* (unquoted):"
for arg in $*
do
  echo "Arg #$index = $arg"
  let "index+=1"
done             # 未被引用的 $* 将所有参数视作独立的单词。
echo "Arg list seen as separate words."

exit 0
```

在 `shift` 命令执行后，`$@` 将会保留除了 `$1` 之外的剩余的命令行参数，而 `$1` 则会被丢弃。

```bash
#!/bin/bash
# 使用 ./scriptname 1 2 3 4 5 调用脚本

echo "$@"    # 1 2 3 4 5
shift
echo "$@"    # 2 3 4 5
shift
echo "$@"    # 3 4 5

# 每一次 "shift" 都会丢弃参数 $1。
# "$@" 则包含了剩余的所有参数。
```

参数 `$@` 也可被用作过滤 shell 脚本输入的工具。结构 `cat "$@"` 可以接受来自标准输入 `stdin` 的输入，也可以接受传递给脚本的参数中的文件中的输入。参考 [样例 16-24]() 和 [样例 16-25]()。

{% hint style="warning" %}

根据分隔符 [`$IFS`]() 设置的不同，`$*` 和 `$@` 有时会出现不一致或非预期行为。

{% endhint %}

#### 样例 9-7. \$* 和 \$@ 的不一致行为

```bash
#!/bin/bash

#  Bash 的内部变量 "$*" 和 "$@" 拥有不稳定的行为，
#+ 这些行为是否出现通常依赖于它们是否是被引用的状态。
#  下面的代码会演示在分词和换行时，这些变量所会出现的一些不一致的处理方式。


set -- "First one" "second" "third:one" "" "Fifth: :one"
# 设置脚本参数，$1, $2, $3 等等。

echo 

echo 'IFS unchanged, using "$*"'
c=0
for i in "$*"               # 被引用状态。
do echo "$((c+=1)): [$i]"   # 这一行在下面所有的例子中都保持不变。
                            # 输出参数。
done
echo ---

echo 'IFS unchanged, using $*'
c=0
for i in $*                 # 未被引用状态。
do echo "$((c+=1)): [$i]"
done
echo ---

echo 'IFS unchanged, using "$@"'
c=0
for i in "$@"
do echo "$((c+=1)): [$i]"
done
echo ---

echo 'IFS unchanged, using $@'
c=0
for i in $@
do echo "$((c+=1)): [$i]"
done
echo ---

IFS=:
echo 'IFS=":", using "$*"'
c=0
for i in "$*"
do echo "$((c+=1)): [$i]"
done
echo ---

echo 'IFS=":", using $*'
c=0
for i in $*
do echo "$((c+=1)): [$i]"
done
echo ---

var=$*
echo 'IFS=":", using "$var" (var=$*)'
c=0
for i in "$var"
do echo "$((c+=1)): [$i]"
done
echo ---

echo 'IFS=":", using $var (var=$*)'
c=0
for i in $var
do echo "$((c+=1)): [$i]"
done
echo ---

var="$*"
echo 'IFS=":", using $var (var="$*")'
c=0
for i in $var
do echo "$((c+=1)): [$i]"
done
echo ---

echo 'IFS=":", using "$var" (var="$*")'
c=0
for i in "$var"
do echo "$((c+=1)): [$i]"
done
echo ---

echo 'IFS=":", using "$@"'
c=0
for i in "$@"
do echo "$((c+=1)): [$i]"
done
echo ---

echo 'IFS=":", using $@'
c=0
for i in $@
do echo "$((c+=1)): [$i]"
done
echo ---

var=$@
echo 'IFS=":", using $var (var=$@)'
c=0
for i in $var
do echo "$((c+=1)): [$i]"
done
echo ---

echo 'IFS=":", using "$var" (var=$@)'
c=0
for i in "$var"
do echo "$((c+=1)): [$i]"
done
echo ---

var="$@"
echo 'IFS=":", using "$var" (var="$@")'
c=0
for i in "$var"
do echo "$((c+=1)): [$i]"
done
echo ---

echo 'IFS=":", using $var (var="$@")'
c=0
for i in $var
do echo "$((c+=1)): [$i]"
done

echo

# 尝试在 ksh 或是 zsh -y 下执行这个脚本。

exit 0

#  这个样例是由 Stephane Chazelas 所编写，
#+ 由本书作者轻微改动。
```

{% hint style="info" %}

`$@` 和 `$*` 仅在被双引号引用时才会表现出不同。

{% endhint %}

#### 样例 9-8. 当 $IFS 为空时 \$* 和 \$@ 的表现

```bash
#!/bin/bash

#  如果 $IFS 被设置为空，
#+ 那么 "$*" 和 "$@" 将不会像期望的那样输出位置参数。

mecho ()       # 输出位置参数。
{
echo "$1,$2,$3";
}


IFS=""         # 设置为空。
set a b c      # 位置参数。

mecho "$*"     # abc,,
#                   ^^
mecho $*       # a,b,c

mecho $@       # a,b,c
mecho "$@"     # a,b,c

#  当 $IFS 为空时 $* 和 $@ 的行为
#+ 依赖于 Bash 或是 sh 所运行的版本。
#  因此不宜在脚本中使用这个“特性”。


# 感谢 Stephane Chazelas。

exit
```

## 其他特殊参数

### $-

使用 [`set`]() 命令设置的脚本标记。参考 [样例 15-16]()。

{% hint style="warning" %}

这个参数最开始是从 ksh 引入到 Bash中的。但很遗憾的是，该参数在 Bash 脚本中并不能可靠地运行。该参数可能的一个用法是用于 [自检脚本是否可交互]()。

{% endhint %}

### $!

运行在后台的最后一个任务的 [进程ID]()。

```bash
LOG=$0.log

COMMAND1="sleep 100"

echo "Logging PIDs background commands for script: $0" >> "$LOG"
# 这样就可以监控命令，并在必要的时候终止它们。
echo >> "$LOG"

# 记录命令。

echo -n "PID of \"$COMMAND1\":  " >> "$LOG"
${COMMAND1} &
echo $! >> "$LOG"
# "sleep 100" 的 PID 是 1506

# 感谢 Jacques Lederer 提出的该建议。
```

将 `$!` 用于控制任务：

```bash
possibly_hanging_job & { sleep ${TIMEOUT}; eval 'kill -9 $!' &> /dev/null; }
# 强制终止一个出错的程序。
# 非常有用，例如可以用在启动脚本中。

# 感谢 Sylvain Fourmanoit 提出的这个利用变量 "$!" 的创造性用法。
```

也可以这么使用：

```bash
# 该样例由 Matthew Sage 编写。
# 本书经授权后使用。

TIMEOUT=30   # 以秒为单位的超时时间值。
count=0

possibly_hanging_job & {
        while ((count < TIMEOUT )); do
                eval '[ ! -d "/proc/$!" ] && ((count = TIMEOUT))'
                # 当前运行进程的详细信息都可以在 /proc 中找到。
                # "-d" 用于测试进程是否存在（即在 /proc 文件夹下该进程的文件夹是否存在）。
                # 我们在等待出问题的任务出现。
                ((count++))
                sleep 1
        done
        eval '[ -d "/proc/$!" ] && kill -15 $!'
        # 如果被挂起的任务正在运行就终止它。
}

#  -------------------------------------------------------------- #

#  然而，如果另外一个进程在 "hanging_job" 之后开始运行
#+ 该函数可能不能正常运行 ...
#  在那种情况下，一个非我们预期的任务会被终止。
#  Ariel Meragelman 提出了如下的解决方案。

TIMEOUT=30
count=0

possibly_hanging_job & {
    while ((count < TIMEOUT )); do
            eval '[ !-d "/proc/$lastjob" ] && ((count = TIMEOUT))'
            lastjob=$!
            ((count++))
            sleep 1
    done
    eval '[ -d "/proc/$lastjob" ] && kill -15 $lastjob'
}

exit
```

### $_

该变量被设置为上一个执行的命令的最后一个参数。

#### 样例 9-9. 下划线变量

```bash
#!/bin/bash

echo $_              #  /bin/bash
                     #  仅通过调用 /bin/bash 执行该脚本。
                     #  注意这个结果会根据脚本如何被调用
                     #+ 而有所不同。

du >/dev/null        #  这样命令就不会在命令行上有任何输出。
echo $_              #  du

ls -al >/dev/null    #  这样命令就不会在命令行上有任何输出。
echo $_              #  -al (最后一个参数)

:
echo $_              #  :
```

### $?

命令、[函数]() 或是脚本自身的 [退出状态]()。参考 [样例 24-7]()。

### $$

脚本自身的进程 ID[^5]。该变量 `$$` 通常在脚本构建独有的临时文件时被使用（参考 [样例 32-6]()，[样例 16-31]()，以及 [样例 15-27]()）。该方法通常比调用 [`mktemp`]() 命令更简单。

## 注记

{% hint style="info" %}
栈寄存器是一段连续的内存空间，在该空间中，存入（压栈）的值是以倒序的方式取出（出栈）的。最后一个存入的值被最先取出。其通常又被称为后进先出(LIFO)或是下堆栈。
{% endhint %}

{% hint style="info" %}
当前运行脚本的进程 ID 就是 `$$`。
{% endhint %}

{% hint style="info" %}
类似于 [递归]()。在本文中，嵌套是指代一种模式被嵌入在一种更大的模式中。在 1913 年出版的韦伯斯特大辞典中用一种更加优雅的方式解释了什么是嵌套：“一组按体积大小排列的盒子、箱子或是类似的东西，它们中的每一个都被放入到另一个更大的箱子中。(A collection of boxes, cases, or the like, of graduated size, each put within the one next larger.)”。
{% endhint %}

{% hint style="info" %}
术语“变量(argument)”和“参数(parameter)”通常情况下是可以互相交换使用的。在本书中，它们具有相同的含义：传入脚本或函数的变量。
{% endhint %}

{% hint style="info" %}
在 subshell 中运行的脚本，`$$` [返回脚本的进程 ID]() 而非 subshell 的。
{% endhint %}

[^1]: Footnotes placeholder
[^2]: Footnotes placeholder
[^3]: Footnotes placeholder
[^4]: Footnotes placeholder
[^5]: Footnotes placeholder

