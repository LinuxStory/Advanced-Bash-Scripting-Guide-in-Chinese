# 16.3 时间/日期命令

## 时间/日期和计时命令

### date

只需调用，**date**就会将日期和时间打印到`标准输出(stdout)`。这个命令有趣的地方在于它的格式和解析选项。

**样例 16-10. 使用*date*命令**

```shell
#!/bin/bash
# 练习'date'命令

echo "The number of days since the year's beginning is `date +%j`."
# 需要一个前导的'+'来调用格式。
# %j给出了年初以来的天数。

echo "The number of seconds elapsed since 01/01/1970 is `date +%s`."
#  %s给出了自从"UNIX时期(UNIX epoch)"开始至今已过的秒数，
#  但是这有什么用呢？

prefix=temp
suffix=$(date +%s)  # “日期”的“+%s”选项是GNU特有的。
filename=$prefix.$suffix
echo "Temporary filename = $filename"
#  它非常适合创建 “唯一且随机” 的临时文件名，
#  甚至比使用$$还好.

# 你可以阅读'date' man手册，了解更多格式选项。
exit 0
```

`-u`选项给出了UTC时间 (世界标准时间)。

```shell
bash$ date
Fri Mar 29 21:07:39 MST 2002



bash$ date -u
Sat Mar 30 04:07:42 UTC 2002
```

此选项便于计算不同日期之间的时间。

**样例 16-11. *日期*计算**

```shell
#!/bin/bash
# date-calc.sh
# 作者: Nathan Coulter
# 仅在本书中允许使用（谢谢）！

MPHR=60    # 每小时的分钟数。
HPD=24     # 每天的小时数。

diff () {
        printf '%s' $(( $(date -u -d"$TARGET" +%s) -
                        $(date -u -d"$CURRENT" +%s)))
#                       %d = 当前月份中的第几天。
}


CURRENT=$(date -u -d '2007-09-01 17:30:24' '+%F %T.%N %Z')
TARGET=$(date -u -d'2007-12-25 12:30:00' '+%F %T.%N %Z')
# %F = 完整日期, %T = %H:%M:%S, %N = 纳秒, %Z = 时区。

printf '\nIn 2007, %s ' \
       "$(date -d"$CURRENT +
        $(( $(diff) /$MPHR /$MPHR /$HPD / 2 )) days" '+%d %B')" 
#       %B = name of month                ^ halfway
printf 'was halfway between %s ' "$(date -d"$CURRENT" '+%d %B')"
printf 'and %s\n' "$(date -d"$TARGET" '+%d %B')"

printf '\nOn %s at %s, there were\n' \
        $(date -u -d"$CURRENT" +%F) $(date -u -d"$CURRENT" +%T)
DAYS=$(( $(diff) / $MPHR / $MPHR / $HPD ))
CURRENT=$(date -d"$CURRENT +$DAYS days" '+%F %T.%N %Z')
HOURS=$(( $(diff) / $MPHR / $MPHR ))
CURRENT=$(date -d"$CURRENT +$HOURS hours" '+%F %T.%N %Z')
MINUTES=$(( $(diff) / $MPHR ))
CURRENT=$(date -d"$CURRENT +$MINUTES minutes" '+%F %T.%N %Z')
printf '%s days, %s hours, ' "$DAYS" "$HOURS"
printf '%s minutes, and %s seconds ' "$MINUTES" "$(diff)"
printf 'until Christmas Dinner!\n\n'

#  练习:
#  --------
#  重写diff()函数使其能够接受传参，
#  而不是使用全局变量。
```

*date*命令有相当多的*输出*选项。例如，`%N`给出当前时间的纳秒部分。该命令的一个有趣的用途是可以生成随机整数。

```shell
date +%N | sed -e 's/000$//' -e 's/^0//'
           ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
#  如果存在，去除前后的零。
#  生成的整数的长度取决于去除了多少个零。

# 115281032
# 63408725
# 394504284
```

还有远远不止这些选项（尝试执行**man date**）

```shell
date +%j
# 输出今天在当年中的位置（自1月1日以来经过的天数）

date +%k%M
# 作为一个单独的数字字符串，以24小时制输出当前小时和分钟。


# 'TZ'参数允许覆盖默认时区。
date                 # Mon Mar 28 21:42:16 MST 2005
TZ=EST date          # Mon Mar 28 23:42:16 EST 2005
# 感谢Frank Kannemann and Pete Sjoberg的点子.


SixDaysAgo=$(date --date='6 days ago')
OneMonthAgo=$(date --date='1 month ago')  # 4周前(不是一个月！)
OneYearAgo=$(date --date='1 year ago')
```

你也可以参见[样例 3-4](https://tldp.org/LDP/abs/html/special-chars.html#EX58)和[样例 A-43](https://tldp.org/LDP/abs/html/contributed-scripts.html#STOPWATCH)。

### zdump

时区转储: 打印指定时区的时间。

```shell
bash$ zdump EST
EST  Tue Sep 18 22:09:22 2001 EST
```

### time

输出执行命令的详细计时统计信息。

`time ls -l /`会给出类似这样的信息：

```shell
real    0m0.067s
 user    0m0.004s
 sys     0m0.005s
```

另请参阅上一节中的非常相似的[times](https://tldp.org/LDP/abs/html/x9644.html#TIMESREF)命令。

> ![note](http://tldp.org/LDP/abs/images/note.gif)从Bash的[2.0版](https://tldp.org/LDP/abs/html/bashver2.html#BASH2REF)开始，**time**变成了shell保留字，在管道(pipeline)中的行为略有改变。

### touch

这是一个用于将文件的访问/修改时间更新为当前系统时间或其他指定时间的实用程序，对于创建新文件也很有用。假设`zzz`以前不存在，命令<code>touch zzz</code>将创建一个零长度的新文件，名为`zzz`。以这种方式对空文件进行时间标记对于存储日期信息很有用，例如在跟踪项目的修改时间方面。

> ![note](http://tldp.org/LDP/abs/images/note.gif)（对于普通文件）**touch**命令等效于：`: >> newfile`或者`>> newfile`

{% hint style="info" %}

在执行[cp-u](https://tldp.org/LDP/abs/html/basic.html#CPREF) (复制/更新) 之前，使用**touch**命令来更新那些你不希望覆盖的文件的时间戳(time stamp)。

例如，如果目录`/home/bozo/tax_audit`包含文件`spreadsheet-051606.data`、`spreadsheet-051706.data`和`spreadsheet-051806.data`，然后执行**touch spreadsheet*.data**会保护这些文件在执行**cp -u /home/bozo/financial_info/spreadsheet*data /home/bozo/tax_audit**时不会被相同文件名文件覆盖。

{% endhint %}

### at

**at**作业控制命令在指定时间执行一组给定的命令。从表面上看，它类似于[cron](https://tldp.org/LDP/abs/html/system.html#CRONREF)，但是，**at**主要用于一次性执行命令集。

**at 2pm January 15**提示在该时点执行一组命令。这些命令需要与shell脚本兼容，出于所有实际目的，用户一次输入一行可执行shell脚本。键入[Ctl-D](https://tldp.org/LDP/abs/html/special-chars.html#CTLDREF)终止。

使用`-f`选项或输入重定向 (<)，**at**从文件中读取命令列表。这个文件是一个可执行的shell脚本，当然，它应该是非交互式的。在文件中包含[run-parts](https://tldp.org/LDP/abs/html/extmisc.html#RUNPARTSREF)命令以执行一组不同的脚本是一个非常明智的选择。

```shell
bash$ at 2:30 am Friday < at-jobs.list
job 2 at 2000-10-27 02:30
```

### batch

**batch**作业控制命令类似于**at**命令，但当系统负载降至低于.8时，它会运行一个命令列表。类似于**at**，它可以使用`-f`选项从文件里读取命令。

| *批处理*的概念可以追溯到大型计算机时代。这意味着在没有用户干预的情况下运行一组命令。 |
| ------------------------------------------- |

### cal

将格式整齐的月历传递给`标准输出(stdout)`。包含今年以及过去和未来几年。

### sleep

该命令是*等待循环*的shell等效项。它暂停指定的秒数，什么也不做。它可以用于定时或在后台运行的进程中，经常用于检查特定事件 (轮询)中，如[样例 32-6](https://tldp.org/LDP/abs/html/debugging.html#ONLINE)所示。

```shell
sleep 3     # 暂停3秒。
```

> ![note](http://tldp.org/LDP/abs/images/note.gif)**sleep**命令默认计时单位为秒，但是分钟、小时、日也可以指定。

```shell
sleep 3 h   # 暂停3小时！
```

> ![note](http://tldp.org/LDP/abs/images/note.gif)对于定时间隔运行命令，[watch](https://tldp.org/LDP/abs/html/system.html#WATCHREF)命令可能是相较于**sleep**命令更好的选择。

### usleep

*Microsleep*（在希腊语中，这个*u*可以读作*mu*，或者是前缀*micro*）。该命令与上述**sleep**相同，但是它是以微秒的间隔时间"sleeps"。它可以用于细粒度的计时，以及以非常频繁的间隔轮询运行中的进程。

```shell
usleep 30   # 暂停30微秒。
```

该命令是红帽系统*initscripts / rc-scripts*包中的一部分。

> ![note](https://tldp.org/LDP/abs/images/caution.gif)**usleep**命令不能提供特别准确的定时，因此不适用于精确的定时循环。

### hwclock,clock

**hwclock**命令访问或调整机器的硬件时钟(hardware clock)。有些选项需要*root*权限。在启动时，`/etc/rc.d/rc.sysinit`启动文件使用**hwclock**命令从硬件时钟中设置系统时间。

**clock**命令是**hwclock**的同义词(synonym)。
