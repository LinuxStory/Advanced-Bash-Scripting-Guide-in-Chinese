# 15.1 任务控制命令

以下某些作业控制命令将*作业标识符*作为参数。您可以参阅本章末尾的[表格](#biao-ge-151.-zuo-ye-biao-shi-fu)。

### jobs

列出在后台运行的作业，并给出该*作业ID*。在实际应用中，没有[ps](https://tldp.org/LDP/abs/html/system.html#PPSSREFhttps://tldp.org/LDP/abs/html/system.html#PPSSREF)有用。

> ![note](http://tldp.org/LDP/abs/images/note.gif)非常容易将*作业*和*进程*二者混淆。某些[内建命令](https://tldp.org/LDP/abs/html/internal.html#BUILTINREF) (例如**kill**，**disown**和**wait**) 既接受作业ID，又接受进程ID作为参数。[fg](https://tldp.org/LDP/abs/html/x9644.html#FGREF)、[bg](https://tldp.org/LDP/abs/html/x9644.html#BGREF)和**jobs**命令仅接受一个作业编号。

```shell
bash$ sleep 100 &
[1] 1384

bash $ jobs
[1]+  Running                 sleep 100 &
```

“1” 是作业ID (作业由当前shell维护)。“1384” 是[PID](https://tldp.org/LDP/abs/html/internalvariables.html#PPIDREF)或*进程ID号* (进程由系统维护)。如果要杀死这个作业/进程，可以使用**kill % 1**或**kill 1384**。

*感谢 S.C.*

### disown

从shell的活动作业表中删除作业。

### fg,bg

**fg**命令将在后台运行的作业切换到前台。**bg**命令重新启动一个挂起的作业，并在后台运行它。如果未指定作业ID，则**fg**或**bg**命令将作用于当前运行的作业。

### wait

暂停脚本执行，直到在后台运行的所有作业都终止，或者直到以作业ID或进程ID标识的指定后台作业终止。返回wait-for命令的[退出状态](https://tldp.org/LDP/abs/html/exit-status.html#EXITSTATUSREF)。

您可以使用**wait**命令来阻止脚本在后台作业完成执行之前退出 (这将创建一个可怕的[孤立进程](https://tldp.org/LDP/abs/html/x9644.html#ZOMBIEREF))。

**样例 15-26. 等待一个进程结束后再继续**

```shell
#!/bin/bash

ROOT_UID=0   # 只有UID是0的用户拥有root权限。
E_NOTROOT=65
E_NOPARAMS=66

if [ "$UID" -ne "$ROOT_UID" ]
then
  echo "Must be root to run this script."
  # “快跑吧，孩子，已经过了你的就寝时间了。”
  exit $E_NOTROOT
fi  

if [ -z "$1" ]
then
  echo "Usage: `basename $0` find-string"
  exit $E_NOPARAMS
fi


echo "Updating 'locate' database..."
echo "This may take a while."
updatedb /usr &     # 必须使用root权限运行。

wait
# 直到"updatedb"运行结束，不要运行这个脚本剩下部分的代码。
# 你希望在查找文件名之前更新数据库。

locate $1

# 没有"wait"命令，在更加糟糕的情况下，
# 这个脚本将会在"updatedb"仍在运行的情况下退出，
# 并且将它作为一个孤立进程。

exit 0
```

或者，**wait**可以将*作业标识符*作为一个参数，例如，`wait % 1`或`wait $ PPID`。 [[1]](https://tldp.org/LDP/abs/html/x9644.html#FTN.AEN9753)请参阅[作业id表](https://tldp.org/LDP/abs/html/x9644.html#JOBIDTABLE)。

{% hint style="info" %}

在脚本中，在后台运行带有 & 符号的命令可能会导致脚本挂起，直到命中**ENTER**。这似乎发生在原本写入`stdout`的命令中。对于程序员来说，这可能是一个很大的烦恼。

```shell
#!/bin/bash
# test.sh          

ls -l &
echo "Done."
```

```shell
bash$ ./test.sh
Done.
 [bozo@localhost test-scripts]$ total 1
 -rwxr-xr-x    1 bozo     bozo           34 Oct 11 15:09 test.sh
 _
```

正如Walter Brameld IV解释说：

据我所知，这样的脚本实际上并没有挂起。似乎他们这样做是因为后台命令在提示后将文本写入控制台。用户得到的印象是提示从未显示过。这是事件的顺序:

1. 脚本启动后台命令。

2. 脚本退出。

3. Shell输出提示。（译者注：即输出`Done`）

4. 后台命令继续运行并向控制台写入文本。（译者注：即`ls -l`命令的输出信息写入`标准输出(stdout)`）

5. 后台命令运行结束

6. 用户在脚本输出的末尾看不到提示，认为脚本挂起中。

在后台命令之后放置**wait**命令似乎可以解决此问题。

```shell
#!/bin/bash
# test.sh          

ls -l &
echo "Done."
wait
```

```shell
bash$ ./test.sh
Done.
 [bozo@localhost test-scripts]$ total 1
 -rwxr-xr-x    1 bozo     bozo           34 Oct 11 15:09 test.sh
```

将命令的输出[重定向](https://tldp.org/LDP/abs/html/io-redirection.html#IOREDIRREF)到文件甚至是`/dev/null`也可以解决此问题。

{% endhint %}

### suspend

该命令与**Control-Z**具有类似的效果，但它会挂起shell (shell的父进程应在适当的时间恢复它)。

### logout

退出登录的shell，可以有选择性地指定它的[退出状态](https://tldp.org/LDP/abs/html/exit-status.html#EXITSTATUSREF)。

### times

以以下形式展现执行命令时经过的系统时间的统计信息:

```shell
0m0.020s 0m0.020s
```

此功能的价值相对有限，因为它在配置文件和基准测试shell脚本中并不常见。

### kill

通过向进程发送适当的*终止*信号来强制终止进程 (参见[样例 17-6](https://tldp.org/LDP/abs/html/system.html#KILLPROCESS))。

**样例 15-27. 一个杀死自己的脚本**

```shell
#!/bin/bash
# self-destruct.sh

kill $$  # 这里，脚本杀死自己的进程。
         # 回想一下，"$$"指的是当前脚本的PID。

echo "This line will not echo."
# 相反，shell会将"终止"信息传递给标准输出(stdout)。

exit 0   # 正常地退出？非也！

#  在这脚本过早终止后，
#  它返回了一个怎样的退出状态？
#
# sh self-destruct.sh
# echo $?
# 143
#
# 143 = 128 + 15
#             终止信号
```

> ![note](http://tldp.org/LDP/abs/images/note.gif)`kill -l`列出了所有[信号](https://tldp.org/LDP/abs/html/debugging.html#SIGNALD)(与文件`/usr/include/asm/signal.h`一样)。`kill -9`是一个*有把握的kill*，通常会终止那些顽固地拒绝以简单的`kill`终止的进程。有些情况下，`kill -15`是生效的。*僵尸*进程是指子进程已经被终止，而[父进程](https://tldp.org/LDP/abs/html/internal.html#FORKREF)还没有（已经）被杀死，僵尸进程无法被登录用户杀死 -- 你不能杀死任何已经死去的东西 -- 但是，通常**init**会迟早清理它。

### killall

**killall**命令通过**名称**来杀死一个运行中的程序，而不是通过[进程ID](https://tldp.org/LDP/abs/html/special-chars.html#PROCESSIDREF)。如果某个特定命令的多个实例正在运行，则执行**killall**将终止**所有**实例。

>  ![note](http://tldp.org/LDP/abs/images/note.gif)这是影响脚本命令处理的三个shell指令之一。其他二者为[builtin](https://tldp.org/LDP/abs/html/x9644.html#BLTREF)和[enable](https://tldp.org/LDP/abs/html/x9644.html#ENABLEREF)。

### builtin

调用**builtin BUILTIN_COMMAND**将命令*BUILTIN_COMMAND*作为shell的[内建命令](https://tldp.org/LDP/abs/html/internal.html#BUILTINREF) 运行，暂时禁用具有相同名称的函数和外部系统命令。

### enable

该命令可以启用或禁用shell内建命令。例如，`enable -n kill`会禁用shell内建命令[kill](https://tldp.org/LDP/abs/html/x9644.html#KILLREF)，因此当Bash随后遇到`kill`命令时，它会调用外部命令`/bin/kill`。

*enable*命令的`-a`选项列出了所有的shell内建命令，并提示它们是否已启用。*enable*命令的`-f 文件名`选项可以从正确编译的对象文件中加载共享库 (DLL) 模块作为[内建命令](https://tldp.org/LDP/abs/html/internal.html#BUILTINREF)。 [[2]](https://tldp.org/LDP/abs/html/x9644.html#FTN.AEN9928)。

### autoload

这是*ksh*自动加载器的Bash入口。当**autoload**到位的情况下，带有*autoload*声明的函数将在第一次调用时从外部文件加载。[[3]](https://tldp.org/LDP/abs/html/x9644.html#FTN.AEN9949) 这节省了系统资源。

请注意，*autoload*不是核心Bash安装的一部分。它需要用*enable -f*加载 (见上文)。

### 表格 15-1. 作业标识符

| 符号表示 | 意义                          |
|:----:|:---------------------------:|
| %N   | 作业编号 [N]                    |
| %S   | 以字符串*S*开始的作业调用（命令行）         |
| %?S  | 包含字符串*S*的作业调用（命令行）          |
| %%   | “当前”作业（指最后一个在前台停止或在后台启动的作业） |
| %+   | “当前”作业（指最后一个在前台停止或在后台启动的作业） |
| %-   | 最后一个作业                      |
| $!   | 最后一个后台进程                    |

## 注记

{% hint style="info" %}

[[1]](https://tldp.org/LDP/abs/html/x9644.html#AEN9753)当然，这仅适用于子进程。

[[2]](https://tldp.org/LDP/abs/html/x9644.html#AEN9928)通常能够在`/usr/share/doc/bash-?.??/functions`目录下找到许多可加载的内建命令的C源码。

注意，**enable**命令的`-f`选项并非[可移植](https://tldp.org/LDP/abs/html/portabilityissues.html)到所有系统。

[[3]](https://tldp.org/LDP/abs/html/x9644.html#AEN9949)使用[typeset -fu](https://tldp.org/LDP/abs/html/declareref.html)可以达到与**autoload**相同的效果。

{% endhint %}
