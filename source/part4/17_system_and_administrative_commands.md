# 系统与管理命令

`/etc/rc.d`目录下的启动和关闭脚本演示了这些命令的大多数用法(和有用性)。这些通常由`root`用户调用，用于系统维护以及紧急文件系统修复。请谨慎使用，因为如果使用不当，其中一些命令可能会损坏您的系统。

## 用户和用户组

### users

显示所有已经登录的用户。大致相当与**who -q**。

### groups

列出当前用户及其所属的组。该命令对应于[\$GROUPS](https://tldp.org/LDP/abs/html/internalvariables.html#GROUPSREF)内部变量，但是仅仅给出了组名，没有组号。

```shell
bash$ groups
bozita cdrom cdwriter audio xgrp

bash$ echo $GROUPS
501
```

### chown, chgrp

**chown**命令用于更改一个或多个文件的所有权。这个命令非常有用，root用户可以使用它将文件所有权从一个用户转移到另一个用户。普通用户不能改变文件的所有权，即使是她自己的文件。[[1]](https://tldp.org/LDP/abs/html/system.html#FTN.AEN14695)

```shell
root# chown bozo *.txt
```

**chgrp**命令用于更改一个或多个文件的*用户组*所有权。若想成功执行这个操作，你必须是该文件的所有者以及目标用户组（或**root组**）的成员。

```shell
chgrp --recursive dunderheads *.data
#  "dunderheads"用户组现在拥有了所有"*.data"文件的权限。
#  "*.data"一路从$PWD目录向下检索（这就是所谓的"recursive"）。
```

### useradd, userdel

**useradd**管理命令将用户帐户添加到系统中，并为该特定用户创建主目录(如果指定了主目录)。相应的，**userdel**命令从系统[[2]](https://tldp.org/LDP/abs/html/system.html#FTN.AEN14727)中删除一个用户帐户，并删除相关的文件。

> ![note](https://tldp.org/LDP/abs/images/note.gif)**adduser**命令是**useradd**的同义词，通常就是指向它的符号链接。

### usermod

修改用户帐户。可以对给定用户帐户的密码、用户组、到期日期和其他属性进行更改。当使用此命令时，用户的密码可能会被锁定，因此具有禁用帐户的效果。

### groupmod

修改给定的组。可使用此命令更改用户组名和/或ID号。

### id

id命令列出了与当前进程相关联的真实有效的用户id和用户的组id。这是内部Bash变量[\$UID](https://tldp.org/LDP/abs/html/internalvariables.html#UIDREF)、[\$EUID](https://tldp.org/LDP/abs/html/internalvariables.html#EUIDREF)和[\$GROUPS](https://tldp.org/LDP/abs/html/internalvariables.html#GROUPSREF)的对应副本。

```shell
bash$ id
uid=501(bozo) gid=501(bozo) groups=501(bozo),22(cdrom),80(cdwriter),81(audio)

bash$ echo $UID
501
```

> ![note](https://tldp.org/LDP/abs/images/note.gif)**id**命令仅在*有效*id与*真实*id不同时才显示它们。

另请参阅[样例 9-5](https://tldp.org/LDP/abs/html/internalvariables.html#AMIROOT)。

### lid

*lid*(list ID)命令显示给定用户所属的用户组，或者属于给定用户组的用户。该命令只能由root用户调用。

```shell
root# lid bozo
 bozo(gid=500)


root# lid daemon
 bin(gid=1)
  daemon(gid=2)
  adm(gid=4)
  lp(gid=7)
```

### who

显示所有已登录系统的用户。

```shell
bash$ who
bozo  tty1     Apr 27 17:45
 bozo  pts/0    Apr 27 17:46
 bozo  pts/1    Apr 27 17:47
 bozo  pts/2    Apr 27 17:49
```

`-m`选项仅给出当前用户的详细信息。向**who**传递任意另外两个参数与**who -m**等价，即**who am i**以及**who The Man**。

```shell
bash$ who -m
localhost.localdomain!bozo  pts/2    Apr 27 17:49
```

**whoami**命令类似于**who -m**，但仅列出用户名。

```shell
bash$ whoami
bozo
```

### w

显示所有已登录的用户和属于他们的进程。这是**who**的扩展版本。**w**的输出可以管道传输给[grep](https://tldp.org/LDP/abs/html/textproc.html#GREPREF)来找到特定的用户和/或进程

```shell
bash$ w | grep startx
bozo  tty1     -                 4:22pm  6:41   4.47s  0.45s  startx
```

### logname

显示当前用户的登录名（你也可以在`/var/run/utmp`下找到）。它近似于以上所说的[whoami](https://tldp.org/LDP/abs/html/system.html#WHOAMIREF)。

```shell
bash$ logname
bozo

bash$ whoami
bozo
```

然而 ...

```shell
bash$ su
Password: ......

bash# whoami
root
bash# logname
bozo
```

> ![note](https://tldp.org/LDP/abs/images/note.gif)尽管**logname**打印出了已经登录的用户名，但是**whoami**给出运行当前进程的用户名。正如我们刚刚看到的，有时这些执行结果并不相同。

### su

作为另一个用户来运行这个程序或脚本。**su rjones**会作为*rjones*来开启一个shell。不带任何参数执行**su**默认切换到*root*用户。请参阅[样例 A-14](https://tldp.org/LDP/abs/html/contributed-scripts.html#FIFO)。

### sudo

作为*root*（或其他）用户运行一条命令。这可以运用在脚本中，从而允许*普通用户*来运行脚本中的命令。

```shell
#!/bin/bash

# 一些命令。
sudo cp /root/secretfile /home/bozo/secret
# 其他更多命令。
```

文件`/etc/sudoers`中记有所有允许调用**sudo**的用户名称。

### passwd

设置、改变或者管理一个用户的密码。
**passwd**命令可以在脚本中使用，但是*不推荐*使用。

**样例 17-1. 设置一个新密码**

```shell
#!/bin/bash
#  setnew-password.sh: 该脚本仅用于演示。
#                      实际运行这个脚本并不是一个好主意。
#  该脚本必须以root用户运行。

ROOT_UID=0         # Root用户的$UID为0.
E_WRONG_USER=65    # 不是root用户?

E_NOSUCHUSER=70
SUCCESS=0


if [ "$UID" -ne "$ROOT_UID" ]
then
  echo; echo "Only root can run this script."; echo
  exit $E_WRONG_USER
else
  echo
  echo "You should know better than to run this script, root."
  echo "Even root users get the blues... "
  echo
fi  


username=bozo
NEWPASSWORD=security_violation

# 检查bozo用户是否存在。
grep -q "$username" /etc/passwd
if [ $? -ne $SUCCESS ]
then
  echo "User $username does not exist."
  echo "No password changed."
  exit $E_NOSUCHUSER
fi  

echo "$NEWPASSWORD" | passwd --stdin "$username"
#  给予'passwd'命令'--stdin'选项
#  可以使其从标准输入(stdin)(或管道)中得到新密码。

echo; echo "User $username's password changed!"

# 在脚本中使用'passwd'命令市非常危险的。

exit 0
```

**passwd**命令的`-l`、`-u`和`-d`选项分别可以冻结、解锁以及删除用户的密码。只有*root用户*可以使用这些选项。

### ac

显示从`/var/log/wtmp`中读取的用户登录时间。这是GNU会计实用工具之一。

```shell
bash$ ac
        total       68.08
```

### last

从`/var/log/wtmp`中读取并列出最后登录的用户。这条命令也能够显示远程登陆的用户。
例如，显示最近几次系统重新启动的信息：

```shell
bash$ last reboot
reboot   system boot  2.6.9-1.667      Fri Feb  4 18:18          (00:02)    
 reboot   system boot  2.6.9-1.667      Fri Feb  4 15:20          (01:27)    
 reboot   system boot  2.6.9-1.667      Fri Feb  4 12:56          (00:49)    
 reboot   system boot  2.6.9-1.667      Thu Feb  3 21:08          (02:17)    
 . . .

 wtmp begins Tue Feb  1 12:50:09 2005
```

### newgrp

在不登出的情况下改变用户的*组ID*。这允许用户可以访问新组里的文件。由于用户可能同时是多个组的成员，因此该命令用途有限。

> ![note](https://tldp.org/LDP/abs/images/note.gif)Kurt Glaesemann指出，*newgrp*命令可能有助于设置用户写入文件的默认组权限。但是仅仅为此目的，[chgrp](https://tldp.org/LDP/abs/html/system.html#CHGRPREF)命令可能更为方便。

## 终端命令

### tty

输出当前用户终端的名称 (文件名)。请注意，每个单独的*xterm*窗口都算作不同的终端。

```shell
bash$ tty
/dev/pts/1
```

### stty

显示和/或更改终端设置。脚本中如果使用该复杂的命令可以控制终端行为和输出显示方式。请参阅info手册，并开展仔细的研究。

**样例 17-2. 设置一个*擦除*字符**

```shell
#!/bin/bash
# erase.sh: 当读取输入时，使用"stty"来设置一个擦除字符。

echo -n "What is your name? "
read name                      #  请尝试回车
                               #  来删除输入的字符
                               #  发现问题了吗？
echo "Your name is $name."

stty erase '#'                 #  设置"井号"(#)作为擦除字符。
echo -n "What is your name? "
read name                      #  尝试使用井号来删除最后一个输入的字符。
echo "Your name is $name."

exit 0

# 即便脚本已经执行结束，这个新的键值设置仍然有效。
# 练习：要怎样才能把擦除字符重置默认值呢？
```

**样例 17-3. *秘密的密码*：关闭终端输入显示**

```shell
#!/bin/bash
# secret-pw.sh: 秘密的密码
echo
echo -n "Enter password "
read passwd
echo "password is $passwd"
echo -n "If someone had been looking over your shoulder, "
echo "your password would have been compromised."

echo && echo  # 在一个&列表中输出两个换行符。


stty -echo    # 关闭屏幕输出。
#   也可以使用以下命令实现相同效果：
#   read -sp passwd
#   非常感谢Leigh James指出。

echo -n "Enter password again "
read passwd
echo
echo "password is $passwd"
echo

stty echo     # 恢复屏幕输出。

exit 0

# 请执行'info stty'以了解这个有用但棘手的命令。
```

**stty**的一种创造性用途是检测用户按键 (不键入**回车键**)。

**样例 17-4. 检测按键**

```shell
#!/bin/bash
# keypress.sh: 检测用户按键 (“热键”)。
echo

old_tty_settings=$(stty -g)   # 保存原有设置（为什么？）。
stty -icanon
Keypress=$(head -c1)          # 或者在非GNU系统上使用
                              # $(dd bs=1 count=1 2> /dev/null)

echo
echo "Key pressed was \""$Keypress"\"."
echo

stty "$old_tty_settings"      # 恢复原有设置。

# 感谢Stephane Chazelas。

exit 0
```

另请参阅[样例 9-3](https://tldp.org/LDP/abs/html/internalvariables.html#TIMEOUT)和[样例 A-43](https://tldp.org/LDP/abs/html/contributed-scripts.html#STOPWATCH)。

{% hint style="info" %}

#### 终端和模式

通常，终端以*规范*模式工作。当用户击中某个键时，生成的字符不会立即转到该终端中实际运行的程序。终端本地的缓冲区会对用户的击键(keystroke)进行缓存。当用户敲击**回车**键时，才会将所有存储的击键(keystroke)发送给正在运行的程序。终端内部甚至有一个基本的行编辑器(line editor)。

```shell
{% raw %}bash$ stty -a
speed 9600 baud; rows 36; columns 96; line = 0;
 intr = ^C; quit = ^\; erase = ^H; kill = ^U; eof = ^D; eol = <undef>; eol2 = <undef>;
 start = ^Q; stop = ^S; susp = ^Z; rprnt = ^R; werase = ^W; lnext = ^V; flush = ^O;
 ...
 isig icanon iexten echo echoe echok -echonl -noflsh -xcase -tostop -echoprt

{% endraw %}      
```

使用规范模式，可以重新定义本地终端线路编辑器的特殊键。

```shell
bash$ cat > filexxx
wha<ctl-W>I<ctl-H>foo bar<ctl-U>hello world<ENTER>
<ctl-D>
bash$ cat filexxx
hello world        
bash$ wc -c < filexxx
12        
```

尽管用户按了26个键，但控制终端的进程仅接收12个字符 (11个字母字符，加上1个换行符)。

在非规范 (“raw”) 模式下，每个击键 (包括特殊的编辑键，例如**ctl-H**) 都会立即向控制过程发送一个字符。

Bash提示符禁用了`icanon`和`echo`，因为它用自己更精细的编辑器代替了基本的终端行编辑器。例如，当你在Bash提示符中键入**ctl-A**，在终端上并没有输出{% raw %}^A{% endraw %}，但是Bash得到了一个**\\1**字符并翻译，将光标从行首向前移动一位。

*Stéphane Chazelas*

{% endhint %}

### setterm

设置某些终端属性。此命令将更改写入终端`标准输出(stdout)`的字符串行为。

```shell
bash$ setterm -cursor off
bash$
```

**setterm**命令可以在脚本中更改写入`标准输出(stdout)`的文本的外观，尽管肯定有[更好的工具](https://tldp.org/LDP/abs/html/colorizing.html#COLORIZINGREF)可实现此目的。

```shell
setterm -bold on
echo bold hello

setterm -bold off
echo normal hello
```

### tset

显示或者初始化终端设置。这是一个能力不强的**stty**版本。

```shell
bash$ tset -r
Terminal type is xterm-xfree86.
 Kill is control-U (^U).
 Interrupt is control-C (^C).
```

### setserial

设置或显示串行端口参数。此命令必须由*root*用户运行，通常可以在系统设置脚本中找到。

```shell
# 选自/etc/pcmcia/serial脚本:

IRQ=`setserial /dev/$DEVICE | sed -e 's/.*IRQ: //'`
setserial /dev/$DEVICE irq 0 ; setserial /dev/$DEVICE irq $IRQ
```

### getty, agetty

**getty**或**agetty**是终端初始化进程，并设置用户登录窗口。这些命令不在用户shell脚本中使用。他们脚本对应的命令是**stty**。

### mesg

允许或者禁用当前用户终端的写权限。禁用权限可以防止网络上的另一个用户将内容[写入](https://tldp.org/LDP/abs/html/communications.html#WRITEREF)这个终端。

> ![note](https://tldp.org/LDP/abs/images/tip.gif)设想一下，当你正在专心致志地编辑文本文件时突然跳出一条外卖广告是多么烦人。因此，在多用户网络中，当需要避免中断时，你或许希望禁用对终端的写访问。

### wall

这是“全部[写入](https://tldp.org/LDP/abs/html/communications.html#WRITEREF)”的首字母缩写，即向当前网络中每个终端的所有用户发送消息。这主要是一个系统管理员的工具，很有用，例如，当警告每个人系统将由于一个问题而即将关闭时(参见[样例 19-1](https://tldp.org/LDP/abs/html/here-docs.html#EX70))。

```shell
bash$ wall System going down for maintenance in 5 minutes!
Broadcast message from bozo (pts/1) Sun Jul  8 13:53:27 2001...

 System going down for maintenance in 5 minutes!
```

> ![note](https://tldp.org/LDP/abs/images/note.gif)如果一个特定的终端已经被**mesg**禁用了写权限，那么**wall**将不能向那个终端发送信息。

## 信息和统计命令

### uname

该命令将系统参数(操作系统、内核版本等)输出到`标准输出(stdout)`。当使用`-a`参数调用时，会给出详细的系统信息(参见[样例 16-5](https://tldp.org/LDP/abs/html/moreadv.html#EX41))。`-s`参数仅显示操作系统类型。

```shell
bash$ uname
Linux

bash$ uname -s
Linux


bash$ uname -a
Linux iron.bozo 2.6.15-1.2054_FC5 #1 Tue Mar 14 15:48:33 EST 2006
 i686 i686 i386 GNU/Linux
```

### arch

显示系统架构。与**uname -m**等价。请参阅[样例 11-27](https://tldp.org/LDP/abs/html/testbranch.html#CASECMD)。

```shell
bash$ arch
i686

bash$ uname -m
i686
```

### lastcomm

给出存储在`/var/account/pacct`文件中之前执行过命令的信息。命令名和用户名可以由选项指定。这是GNU会计工具之一。

### lastlog

列出所有系统用户最后一次的登陆时间。该命令参考`/var/log/lastlog`文件。

> ![note](https://tldp.org/LDP/abs/images/caution.gif)如果调用该命令的用户没有`var/log/lastlog`文件的读权限，那么将会失败。

### lsof

列出所有已打开的文件。这条命令会以表格的形式详细输出当前所有打开的文件，并给出这些文件的拥有者、大小、与其关联的进程等等。当然，**lsof**可以通过管道传输给[grep](https://tldp.org/LDP/abs/html/textproc.html#GREPREF)或[awk](https://tldp.org/LDP/abs/html/awk.html#AWKREF)来解析和分析它的结果。

```shell
bash$ lsof
COMMAND    PID    USER   FD   TYPE     DEVICE    SIZE     NODE NAME
 init         1    root  mem    REG        3,5   30748    30303 /sbin/init
 init         1    root  mem    REG        3,5   73120     8069 /lib/ld-2.1.3.so
 init         1    root  mem    REG        3,5  931668     8075 /lib/libc-2.1.3.so
 cardmgr    213    root  mem    REG        3,5   36956    30357 /sbin/cardmgr
 ...
```

**lsof**命令是一条有用的，复杂的命令工具。如果你无法取消挂载一个文件系统并且仍得到“该文件系统仍在使用中”的报错消息，那么执行*lsof*会有助于查找究竟是哪个在文件系统上的文件还处于打开状态。`-i`选项会列出网络套接字(socket)文件，因此这可以帮助追踪入侵或者黑客攻击。

```shell
bash$ lsof -an -i tcp
COMMAND  PID USER  FD  TYPE DEVICE SIZE NODE NAME
 firefox 2330 bozo  32u IPv4   9956       TCP 66.0.118.137:57596->67.112.7.104:http ...
 firefox 2330 bozo  38u IPv4  10535       TCP 66.0.118.137:57708->216.79.48.24:http ...
```

请参阅[样例 30-2](https://tldp.org/LDP/abs/html/networkprogramming.html#IPADDRESSES)了解**lsof**命令有效的使用方式。

### strace

系统跟踪:用于跟踪*系统调用*和信号的调试诊断工具。该命令和下面的**ltrace**命令对于诊断给定程序或软件包无法运行的原因非常有用 . . . 可能是因为缺少库或相关原因。

```shell
bash$ strace df
execve("/bin/df", ["df"], [/* 45 vars */]) = 0
 uname({sys="Linux", node="bozo.localdomain", ...}) = 0
 brk(0)                                  = 0x804f5e4

 ...
```

该条命令运行在Linux系统上，并且与Solaris系统上的**truss**命令等价。

### ltrace

库跟踪：用于跟踪给定命令所调用的*库调用*操作的调试诊断工具。

```shell
bash$ ltrace df
__libc_start_main(0x804a910, 1, 0xbfb589a4, 0x804fb70, 0x804fb68 <unfinished ...>:
 setlocale(6, "")                                 = "en_US.UTF-8"
bindtextdomain("coreutils", "/usr/share/locale") = "/usr/share/locale"
textdomain("coreutils")                          = "coreutils"
__cxa_atexit(0x804b650, 0, 0, 0x8052bf0, 0xbfb58908) = 0
getenv("DF_BLOCK_SIZE")                          = NULL

 ...
```

### nc

**nc** (*netcat*)实用程序是一个完整的工具包，用于连接和侦听TCP和UDP端口。它是一个有用的调试诊断工具，也是基于脚本的简单HTTP客户端及服务器中的一个组件。

```shell
bash$ nc localhost.localdomain 25
220 localhost.localdomain ESMTP Sendmail 8.13.1/8.13.1;
 Thu, 31 Mar 2005 15:41:35 -0700
```

一个现实生活中的[案例](https://tldp.org/LDP/abs/html/process-sub.html#NETCATEXAMPLE)。

**样例 17-5. 检查远程服务器的<em>ident</em>**

```shell
#! /bin/sh
## 用netcat复刻DaveG的“ident扫描”程序。哦我的上帝，他将被隔壁老奶奶的靴子狠狠地踢到屁股。
## 参数: 目标 端口 [端口 端口 端口 ...]
## 不区分标准输出(stdout)和标准错误(stderr)。
##
##  优点：运行速度比ident扫描慢，远程ident也产生了更少的报警信息，
##  并且仅命中你指定的几个已知守护程序端口。
##  缺点: 仅支持整型端口参数，输出困难。
##  并且当来自高源端口时，对于r服务不起作用。
# 脚本作者: Hobbit <hobbit@avian.org>
# 经许可在本书中使用。
# ---------------------------------------------------
E_BADARGS=65       # 需要至少两个参数。
TWO_WINKS=2        # 等待多长时间。
THREE_WINKS=3
IDPORT=113         # 身份验证 “tap ident” 端口。
RAND1=999
RAND2=31337
TIMEOUT0=9
TIMEOUT1=8
TIMEOUT2=4
# ---------------------------------------------------

case "${2}" in
  "" ) echo "Need HOST and at least one PORT." ; exit $E_BADARGS ;;
esac

# Ping他们一次，看看他们 *是否* 正在运行identd。
nc -z -w $TIMEOUT0 "$1" $IDPORT || \
{ echo "Oops, $1 isn't running identd." ; exit 0 ; }
#  -z 参数扫描监听的守护程序。
#     -w $TIMEOUT = 尝试连接的等待时间。

# 生成一个随机的基本端口。
RP=`expr $$ % $RAND1 + $RAND2`

TRG="$1"
shift

while test "$1" ; do
  nc -v -w $TIMEOUT1 -p ${RP} "$TRG" ${1} < /dev/null > /dev/null &
  PROC=$!
  sleep $THREE_WINKS
  echo "${1},${RP}" | nc -w $TIMEOUT2 -r "$TRG" $IDPORT 2>&1
  sleep $TWO_WINKS

# 这看起来像一个接吻脚本或者其他的 . . . ?
# 本书作者评论说：“实际上并没有那么糟糕 . . . 
#                             反而有点聪明”

  kill -HUP $PROC
  RP=`expr ${RP} + 1`
  shift
done

exit $?

#  注记：
#  -----

#  请尝试注释掉第30行，并以"localhost.localdomain 25"
#  为参数运行此脚本。

#  有关Hobbit更多的'nc'样例脚本，
#  请参阅文档：
#  位于/usr/share/doc/nc-X.XX/scripts目录下。
```

当然，在BitKeeper事件中还有Andrew Tridgell博士臭名昭著的单行脚本:

```shell
echo clone | nc thunk.org 5000 > e2fsprogs.dat
```

### free

以表格形式显示内存和缓存使用情况。此命令的输出可以使用[grep](https://tldp.org/LDP/abs/html/textproc.html#GREPREF)、[awk](https://tldp.org/LDP/abs/html/awk.html#AWKREF)或**Perl**进行解析。**procinfo**命令不仅显示**free**输出的所有信息，而且包括更多信息。

```shell
bash$ free
                total       used       free     shared    buffers     cached
   Mem:         30504      28624       1880      15820       1608       16376
   -/+ buffers/cache:      10640      19864
   Swap:        68540       3128      65412
```

显示未用的RAM内存：

```shell
bash$ free | grep Mem | awk '{ print $4 }'
1880
```

### procinfo

从[`/proc`伪文件系统](https://tldp.org/LDP/abs/html/devproc.html#DEVPROCREF)中提取并列出信息和统计信息。该命令给出了一个非常广泛和详细的列表。

```shell
bash$ procinfo | grep Bootup
Bootup: Wed Mar 21 15:15:50 2001    Load average: 0.04 0.21 0.34 3/47 6829
```

### lsdev

列出设备，即显示已安装的硬件。

```shell
bash$ lsdev
Device            DMA   IRQ  I/O Ports
 ------------------------------------------------
 cascade             4     2 
 dma                          0080-008f
 dma1                         0000-001f
 dma2                         00c0-00df
 fpu                          00f0-00ff
 ide0                     14  01f0-01f7 03f6-03f6
 ...
```

### du

递归显示 (磁盘) 文件使用情况。默认为当前工作目录，除非另有说明。

```shell
bash$ du -ach
1.0k    ./wi.sh
 1.0k    ./tst.sh
 1.0k    ./random.file
 6.0k    .
 6.0k    total
```

### df

以表格形式显示文件系统使用情况。

```shell
bash$ df
Filesystem           1k-blocks      Used Available Use% Mounted on
 /dev/hda5               273262     92607    166547  36% /
 /dev/hda8               222525    123951     87085  59% /home
 /dev/hda7              1408796   1075744    261488  80% /usr
```

### dmesg

将系统启动消息写入`标准输出(stdout)`。方便调试和确定安装了哪些设备驱动程序以及正在使用哪些系统中断。当然，**dmesg**的输出可以在脚本中使用[grep](https://tldp.org/LDP/abs/html/textproc.html#GREPREF)、[sed](https://tldp.org/LDP/abs/html/sedawk.html#SEDREF)或[awk](https://tldp.org/LDP/abs/html/awk.html#AWKREF)进行解析。

```shell
bash$ dmesg | grep hda
Kernel command line: ro root=/dev/hda2
 hda: IBM-DLGA-23080, ATA DISK drive
 hda: 6015744 sectors (3080 MB) w/96KiB Cache, CHS=746/128/63
 hda: hda1 hda2 hda3 < hda5 hda6 hda7 > hda4
```

### stat

给出指定文件 (甚至是目录或设备文件) 或文件集详细全面的*统计信息*。

```shell
bash$ stat test.cru
  File: "test.cru"
   Size: 49970        Allocated Blocks: 100          Filetype: Regular File
   Mode: (0664/-rw-rw-r--)         Uid: (  501/ bozo)  Gid: (  501/ bozo)
 Device:  3,8   Inode: 18185     Links: 1    
 Access: Sat Jun  2 16:40:24 2001
 Modify: Sat Jun  2 16:40:24 2001
 Change: Sat Jun  2 16:40:24 2001
```

如果目标文件不存在，**stat**会返回错误信息。

```shell
bash$ stat nonexistent-file
nonexistent-file: No such file or directory
```

在脚本中，你可以使用**stat**提取有关文件 (和文件系统) 的信息，并相应地设置变量。

```shell
#!/bin/bash
# fileinfo2.sh

# 采取来自Joël Bourquard 以及 . . .
# http://www.linuxquestions.org/questions/showthread.php?t=410766
# 等建议。

/www.linuxquestions.org/questions/
FILENAME=testfile.txt
file_name=$(stat -c%n "$FILENAME")   # 当然，等效于"$FILENAME"。
file_owner=$(stat -c%U "$FILENAME")
file_size=$(stat -c%s "$FILENAME")
#  当然，使用"ls -l $FILENAME"
#  再用sed解析要方便地多。
file_inode=$(stat -c%i "$FILENAME")
file_type=$(stat -c%F "$FILENAME")
file_access_rights=$(stat -c%A "$FILENAME")

echo "File name:          $file_name"
echo "File owner:         $file_owner"
echo "File size:          $file_size"
echo "File inode:         $file_inode"
echo "File type:          $file_type"
echo "File access rights: $file_access_rights"

exit 0

sh fileinfo2.sh

File name:          testfile.txt
File owner:         bozo
File size:          418
File inode:         1730378
File type:          regular file
File access rights: -rw-rw-r--
```

### vmstat

展示虚拟内存统计数据。

```shell
bash$ vmstat
   procs                      memory    swap          io system         cpu
 r  b  w   swpd   free   buff  cache  si  so    bi    bo   in    cs  us  sy id
 0  0  0      0  11040   2636  38952   0   0    33     7  271    88   8   3 89
```

### uptime

显示系统运行了多长时间，以及相关的统计信息。

```shell
bash$ uptime
10:28pm  up  1:57,  3 users,  load average: 0.17, 0.34, 0.27
```

> ![note](https://tldp.org/LDP/abs/images/note.gif)*负载均值*为1或更小表示系统将会立即处理进程。负载平均值大于1意味着进程正在排队。当负载平均值高于3 (在单核处理器上) 时，系统性能会大大降低。

### hostname

列出当前系统的主机名。该命令会在`/etc/rc.d`（`/etc/rc.d/rc.sysinit`或类似的文件）中设置主机名。该命令与**uname -n**等效，并且是内部变量[\$HOSTNAME](https://tldp.org/LDP/abs/html/internalvariables.html#HOSTNAMEREF)的对应项。

```shell
bash$ hostname
localhost.localdomain

bash$ echo $HOSTNAME
localhost.localdomain
```

与**hostname**类似的命令还有**domainname**、**dnsdomainname**、**nisdomainname**和**ypdomainname**命令。可以使用这些命令来显示或设置系统DNS或NIS/YP域名。**hostname**命令的各种选项也实现了这些功能。

### hostid

输出主机的32位十六进制数字标识符。

```shell
bash$ hostid
7f0100
```

> ![note](https://tldp.org/LDP/abs/images/note.gif)据称，该命令获取了特定系统的 “唯一” 序列号。某些产品注册程序使用此编号来标记特定的用户许可证。不幸的是，**hostid**仅返回十六进制机器网络地址，并对字节进行了转置。

在`/etc/hosts`文件中可以找到典型的非联网Linux主机的网络地址。

```shell
bash$ cat /etc/hosts
127.0.0.1               localhost.localdomain localhost
```

碰巧的是，如果转置<strong>`127.0.0.1`</strong>的字节，我们会得到<strong>`0.127.1.0`</strong>，转换为十六进制为<strong>`007f0100`</strong>，完全等同于上面的**hostid**返回的内容。在全世界，只有几百万台其他Linux机器具有相同的*hostid*。

### sar

**sar**命令(系统活动报告者) 提供了极详细的系统统计信息摘要。圣克鲁斯行动 (“旧” SCO) 在1999年6月将**sar**作为开源软件发布。

此命令不是基本Linux发行版的一部分，但可以从[Sebastien Godard](sebastien.godard@wanadoo.fr)编写的[sysstat实用程序](http://perso.wanadoo.fr/sebastien.godard/)包中获得。

```shell
bash$ sar
Linux 2.4.9 (brooks.seringas.fr)     09/26/03

10:30:00          CPU     %user     %nice   %system   %iowait     %idle
10:40:00          all      2.21     10.90     65.48      0.00     21.41
10:50:00          all      3.36      0.00     72.36      0.00     24.28
11:00:00          all      1.12      0.00     80.77      0.00     18.11
Average:          all      2.23      3.63     72.87      0.00     21.27

14:32:30          LINUX RESTART

15:00:00          CPU     %user     %nice   %system   %iowait     %idle
15:10:00          all      8.59      2.40     17.47      0.00     71.54
15:20:00          all      4.07      1.00     11.95      0.00     82.98
15:30:00          all      0.79      2.94      7.56      0.00     88.71
Average:          all      6.33      1.70     14.71      0.00     77.26
```

### readelf

显示指定*elf*二进制文件的信息和统计信息。该命令是*binutils*包的一部分。

```shell
bash$ readelf -h /bin/bash
ELF Header:
   Magic:   7f 45 4c 46 01 01 01 00 00 00 00 00 00 00 00 00 
   Class:                             ELF32
   Data:                              2's complement, little endian
   Version:                           1 (current)
   OS/ABI:                            UNIX - System V
   ABI Version:                       0
   Type:                              EXEC (Executable file)
   . . .
```

### size

<strong>size [/path/to/binary]</strong>命令给出二进制可执行文件或存档文件的段大小。这主要对程序员有用。

```shell
bash$ size /bin/bash
   text    data     bss     dec     hex filename
  495971   22496   17392  535859   82d33 /bin/bash
```

## 系统日志命令

### logger

将用户生成的消息附加到系统日志 (`/var/log/messages`)。您不必是*root*用户即可调用**logger**。

```shell
logger Experiencing instability in network connection at 23:10, 05/21.
# 现在，执行一下'tail /var/log/messages'。
```

通过在脚本中嵌入**logger**命令，可以将调试信息写入`/var/log/messages`。

```shell
logger -t $0 -i Logging at line "$LINENO".
# "-t"选项指定logger条目的标记。
# "-i"选项会记录进程ID。

# tail /var/log/message
# ...
# Jul  7 20:48:58 localhost ./test.sh[1712]: Logging at line 3.
```

### logrotate

此实用程序（日志轮转）管理系统日志文件，根据需要轮转、压缩、删除和/或通过电子邮件发送它们。这样可以防止`/var/log`被旧的日志文件弄得乱七八糟。通常[cron](https://tldp.org/LDP/abs/html/system.html#CRONREF)程序每天会运行**logrotate**。

在`/etc/logrotate.conf`中添加适当的条目可以管理个人日志文件以及系统范围的日志文件。

> ![note](https://tldp.org/LDP/abs/images/note.gif)Stefano Falsetto开发了[rottlog]([GNU Rot[t]log](http://www.gnu.org/software/rottlog/))，他认为这是**logrotate**的改进版本。

## 任务控制命令

### ps

进程统计：按所有者和PID (进程ID) 列出当前正在执行的进程。该命令经常带有`ax`或`aux`选项进行调用，并且可以管道传输到[grep](https://tldp.org/LDP/abs/html/textproc.html#GREPREF)或[sed](https://tldp.org/LDP/abs/html/sedawk.html#SEDREF)来搜索特定的进程(参阅[样例 15-14](https://tldp.org/LDP/abs/html/internal.html#EX44)和[样例 29-3](https://tldp.org/LDP/abs/html/procref1.html#PIDID))。

```shell
bash$  ps ax | grep sendmail
295 ?       S      0:00 sendmail: accepting connections on port 25
```

以图形化进程 “树” 格式显示系统进程：请执行**ps afjx**或**ps ax -- forest**。

### pgrep, pkill

将**ps**命令与[grep](https://tldp.org/LDP/abs/html/textproc.html#GREPREF)和[kill](https://tldp.org/LDP/abs/html/x9644.html#KILLREF)结合了起来。

```shell
bash$ ps a | grep mingetty
2212 tty2     Ss+    0:00 /sbin/mingetty tty2
 2213 tty3     Ss+    0:00 /sbin/mingetty tty3
 2214 tty4     Ss+    0:00 /sbin/mingetty tty4
 2215 tty5     Ss+    0:00 /sbin/mingetty tty5
 2216 tty6     Ss+    0:00 /sbin/mingetty tty6
 4849 pts/2    S+     0:00 grep mingetty


bash$ pgrep mingetty
2212 mingetty
 2213 mingetty
 2214 mingetty
 2215 mingetty
 2216 mingetty
```

请比较**pkill**和[killall](https://tldp.org/LDP/abs/html/x9644.html#KILLALLREF)命令的行为。

### pstree

以进程“树”格式列出当前正在执行的进程。`-p`选项显示了PID以及进程名称。

### top

实时更新显示大多数cpu密集型进程。`-b`选项以文本模式显示，因此可以解析命令输出或者用于脚本中。

```shell
bash$ top -b
  8:30pm  up 3 min,  3 users,  load average: 0.49, 0.32, 0.13
 45 processes: 44 sleeping, 1 running, 0 zombie, 0 stopped
 CPU states: 13.6% user,  7.3% system,  0.0% nice, 78.9% idle
 Mem:    78396K av,   65468K used,   12928K free,       0K shrd,    2352K buff
 Swap:  157208K av,       0K used,  157208K free                   37244K cached

   PID USER     PRI  NI  SIZE  RSS SHARE STAT %CPU %MEM   TIME COMMAND
   848 bozo      17   0   996  996   800 R     5.6  1.2   0:00 top
     1 root       8   0   512  512   444 S     0.0  0.6   0:04 init
     2 root       9   0     0    0     0 SW    0.0  0.0   0:00 keventd
   ...  
```

### nice

运行优先级已更改的后台作业。优先级从19(最低) 到-20(最高)。只有*root权限*可以设置负 (更高) 优先级。相关的命令是**renice**、**snice**和**skill**。

**renice**和**snice**命令会更改正在运行的一个或多个进程的优先级。

**skill**命令会向一个或多个进程发送[终止](https://tldp.org/LDP/abs/html/x9644.html#KILLREF)信号。

### nohup

即使在用户注销后，命令仍保持运行。该命令将作为前台进程运行，除非后面跟着&。如果在脚本中使用**nohup**，请考虑将其与[wait](https://tldp.org/LDP/abs/html/x9644.html#WAITREF)相结合，以避免创建*孤立*或[僵尸](https://tldp.org/LDP/abs/html/x9644.html#ZOMBIEREF)进程。

### pidof

标识正在运行的作业的*进程ID(PID)*。由于作业控制命令 (例如[kill](https://tldp.org/LDP/abs/html/x9644.html#KILLREF)和[renice](https://tldp.org/LDP/abs/html/system.html#NICE2REF)) 直接作用于进程的*PID*(而不是其名称)，因此有时有必要识别该*PID*。**pidof**命令近似对应于内部变量**\$PPID**。

```shell
bash$ pidof xclock
880
```

**样例 17-6. 在*pidof*协助下杀死一个进程**

```shell
#!/bin/bash
# kill-process.sh

NOPROCESS=2

process=xxxyyyzzz  # 使用不存在的一个进程。
# 仅用于示例...
# ... 不想用这个脚本杀死任何实际存在的进程。
#
# 例如，如果你想使用此脚本注销Internet，
#     process=pppd

t=`pidof $process`       # 找到$process的pid(进程id)。
# 这个'pid'需要被'kill' (不能通过程序名来'kill')。

if [ -z "$t" ]           # 如果进程不存在，'pidof' 返回null。
then
  echo "Process $process was not running."
  echo "Nothing killed."
  exit $NOPROCESS
fi  

kill $t                  # 可能需要'kill -9'来杀死阻塞的进程。

# 需要在这里检查一下，看看进程是否允许自己被杀死。
# 可能需要一个额外的" t=`pidof $process` " 或者 ...


# 整个脚本可以使用以下代码行来代替
#        kill $(pidof -x process_name)
# 或者
#        killall process_name
# 但这不具有启发性。

exit 0
```

### fuser

标识正在访问给定文件、文件集或目录的进程 (通过PID)。也可以使用`-k`选项进行调用，该选项会杀死这些进程。这对系统安全具有有趣的应用场景，尤其是用于防止未经授权的用户访问系统服务的脚本中。

```shell
bash$ fuser -u /usr/bin/vim
/usr/bin/vim:         3207e(bozo)



bash$ fuser -u /dev/null
/dev/null:            3009(bozo)  3010(bozo)  3197(bozo)  3199(bozo)
```

**fuser**的一个重要应用是在物理上插入或删除存储介质 (例如CD ROM磁盘或USB闪存驱动器) 时。有时，执行[umount](https://tldp.org/LDP/abs/html/system.html#UMOUNTREF)会失败，并报“设备忙”的错误信息。这意味着一些用户和/或进程正在访问设备。执行**fuser -um /dev/device_name**将真相大白，于是你可以杀死任何相关进程。

```shell
bash$ umount /mnt/usbdrive
umount: /mnt/usbdrive: device is busy



bash$ fuser -um /dev/usbdrive
/mnt/usbdrive:        1772c(bozo)

bash$ kill -9 1772
bash$ umount /mnt/usbdrive
```

使用`-n`选项调用的**fuser**命令可以标识进程访问的端口。当与[nmap](https://tldp.org/LDP/abs/html/system.html#NMAPREF)命令结合使用时特别有用。

```shell
root# nmap localhost.localdomain
PORT     STATE SERVICE
 25/tcp   open  smtp



root# fuser -un tcp 25
25/tcp:               2095(root)

root# ps ax | grep 2095 | grep -v grep
2095 ?        Ss     0:00 sendmail: accepting connections
```

### cron

管理程序调度程序，执行清理、删除系统日志文件以及更新slocate数据库等职责。这是[at](https://tldp.org/LDP/abs/html/timedate.html#ATREF)的*超级用户*版本 (尽管每个用户可能都有自己的`crontab`文件，可以使用**crontab**命令进行更改)。它作为[守护进程](https://tldp.org/LDP/abs/html/communications.html#DAEMONREF)运行，并从`/etc/crontab`执行计划条目。

> ![note](https://tldp.org/LDP/abs/images/note.gif)一些Linux的版本运行**crond**程序，这是Matthew Dillon的**cron**版本。

## 进程控制和启动命令

### init

**init**命令是所有进程的[父进程](https://tldp.org/LDP/abs/html/internal.html#FORKREF)。在启动的最后一步被调用，**init**从`/etc/inittab`中决定了系统的运行级别。被其别名**telinit**调用，并且仅由*root*用户调用。

### telinit

**init**的符号链接，这是一种改变系统运行级别的方法，通常用于系统维护或紧急文件系统修复。仅由*root*用户调用。这个命令可能非常危险——在使用之前一定要理解它！

### runlevel

显示当前和上一次运行级别，即系统是暂停模式(运行级别0)、单用户模式(1)、多用户模式(2或3)、X窗口模式(5)还是重新启动(6)。这个命令会访问`/var/run/utmp`文件。

### halt, shutdown, reboot

通常在断电前用于关闭系统的命令集。

> ![note](https://tldp.org/LDP/abs/images/warning.gif) 在一些Linux发行版上，**halt**命令拥有755的权限，所以它能够被非root用户调用。粗心地在终端或者脚本中执行**halt**会导致整个系统关闭！

### service

启动或者停止一个系统**服务**。在开机启动时，位于`/etc/init.d`或者`/etc/rc.d`中的启动脚本使用该命令来启动服务.

```shell
root# /sbin/service iptables stop
Flushing firewall rules:                                   [  OK  ]
 Setting chains to policy ACCEPT: filter                    [  OK  ]
 Unloading iptables modules:                                [  OK  ]
```

## 网络命令

### nmap

网络映射器和端口扫描器。此命令扫描服务器以定位开放的端口以及与这些端口相关联的服务。它还可以报告有关包过滤和防火墙的信息。这是一个重要的安全工具，用于锁定网络以防黑客攻击。

```shell
#!/bin/bash

SERVER=$HOST                           # localhost.localdomain (127.0.0.1).
PORT_NUMBER=25                         # SMTP 端口。

nmap $SERVER | grep -w "$PORT_NUMBER"  # 指定的端口开放了吗？
#              grep -w 仅匹配全词，
#              例如，这不会匹配到1025端口。
exit 0

# 25/tcp     open        smtp
```

### ifconfig

网络*接口配置*和调整的实用程序。

```shell
bash$ ifconfig -a
lo        Link encap:Local Loopback
           inet addr:127.0.0.1  Mask:255.0.0.0
           UP LOOPBACK RUNNING  MTU:16436  Metric:1
           RX packets:10 errors:0 dropped:0 overruns:0 frame:0
           TX packets:10 errors:0 dropped:0 overruns:0 carrier:0
           collisions:0 txqueuelen:0 
           RX bytes:700 (700.0 b)  TX bytes:700 (700.0 b)
```

**ifconfig**命令最常用于启动时设置接口，或者在重启时关闭接口。

```shell
# 代码摘自 /etc/rc.d/init.d/network

# ...

# 检查网络是否启动。
[ ${NETWORKING} = "no" ] && exit 0

[ -x /sbin/ifconfig ] || exit 0

# ...

for i in $interfaces ; do
  if ifconfig $i 2>/dev/null | grep -q "UP" >/dev/null 2>&1 ; then
    action "Shutting down interface $i: " ./ifdown $i boot
  fi
#  GNU特有的"grep"命令"-q"选项意味着“安静”，
#  即不产生任何输出。
#  因此，并没有必要将输出重定向至 /dev/null。

# ...

echo "Currently active devices:"
echo `/sbin/ifconfig | grep ^[a-z] | awk '{print $1}'`
#                            ^^^^^   它需要被括起来以防使用glob。
#  以下的代码同样有效：
#    echo $(/sbin/ifconfig | awk '/^[a-z]/ { print $1 })'
#    echo $(/sbin/ifconfig | sed -e 's/ .*//')
#    感谢S.C.所提供的额外注释。
```

另请参阅[样例 32-6](https://tldp.org/LDP/abs/html/debugging.html#ONLINE)。

### netstat

显示当前网络统计数据和信息，如路由表和活动的网络连接。这个实用程序访问`/proc/net`中的信息([第29章](https://tldp.org/LDP/abs/html/devproc.html))。参阅[样例 29-4](https://tldp.org/LDP/abs/html/procref1.html#CONSTAT)。

**netstat -r**等价于[route](https://tldp.org/LDP/abs/html/system.html#ROUTEREF)程序。

```shell
bash$ netstat
Active Internet connections (w/o servers)
 Proto Recv-Q Send-Q Local Address           Foreign Address         State      
 Active UNIX domain sockets (w/o servers)
 Proto RefCnt Flags       Type       State         I-Node Path
 unix  11     [ ]         DGRAM                    906    /dev/log
 unix  3      [ ]         STREAM     CONNECTED     4514   /tmp/.X11-unix/X0
 unix  3      [ ]         STREAM     CONNECTED     4513
 . . .
```

> ![note](https://tldp.org/LDP/abs/images/note.gif)**netstat -lptu**显示了正在监听端口的[套接字](https://tldp.org/LDP/abs/html/devref1.html#SOCKETREF)以及相关的进程。这对于确定计算机是否已被黑客攻击或威胁非常有用。

### iwconfig

这是用于配置无线网络的命令集。可以理解为无线版的**ifconfig**。

### ip

用于设置、更改和分析*IP*（互联网协议）网络和附加设备。该命令是*iproute2*软件包的一部分。

```shell
bash$ ip link show
1: lo: <LOOPBACK,UP> mtu 16436 qdisc noqueue 
     link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
 2: eth0: <BROADCAST,MULTICAST> mtu 1500 qdisc pfifo_fast qlen 1000
     link/ether 00:d0:59:ce:af:da brd ff:ff:ff:ff:ff:ff
 3: sit0: <NOARP> mtu 1480 qdisc noop 
     link/sit 0.0.0.0 brd 0.0.0.0


bash$ ip route list
169.254.0.0/16 dev lo  scope link
```

或者，在脚本中：

```shell
#!/bin/bash
# 由Juan Nicolas Ruiz编写。
# 感谢他的慷慨相赠。

# 启动（和停止）一个GRE隧道。


# --- start-tunnel.sh ---

LOCAL_IP="192.168.1.17"
REMOTE_IP="10.0.5.33"
OTHER_IFACE="192.168.0.100"
REMOTE_NET="192.168.3.0/24"

/sbin/ip tunnel add netb mode gre remote $REMOTE_IP \
  local $LOCAL_IP ttl 255
/sbin/ip addr add $OTHER_IFACE dev netb
/sbin/ip link set netb up
/sbin/ip route add $REMOTE_NET dev netb

exit 0  #############################################

# --- stop-tunnel.sh ---

REMOTE_NET="192.168.3.0/24"

/sbin/ip route del $REMOTE_NET dev netb
/sbin/ip link set netb down
/sbin/ip tunnel del netb

exit 0
```

### route

显示内核路由表的信息或对其进行更改。

```shell
bash$ route
Destination     Gateway         Genmask         Flags   MSS Window  irtt Iface
 pm3-67.bozosisp *               255.255.255.255 UH       40 0          0 ppp0
 127.0.0.0       *               255.0.0.0       U        40 0          0 lo
 default         pm3-67.bozosisp 0.0.0.0         UG       40 0          0 ppp0
```

### iptables

**iptables**命令集是一种数据包过滤工具，主要用于诸如设置网络防火墙之类的安全目的。这是一个复杂的工具，对其使用的详细解释超出了本文档的范围。可以从[Oskar Andreasson的教程](http://www.frozentux.net/iptables-tutorial/iptables-tutorial.html)开始了解。

另请参阅[关闭*iptables*](https://tldp.org/LDP/abs/html/system.html#IPTABLES01)和[样例 30-2](https://tldp.org/LDP/abs/html/networkprogramming.html#IPADDRESSES)。

### chkconfig

检查网络和系统配置。此命令会列出和管理在通过`/etc/rc?.d`目录来控制启动的网络和系统服务。

**chkconfig**最初是从IRIX继承到Red Hat Linux的端口，某些Linux版本的核心安装可能不包含该程序。

```shell
bash$ chkconfig --list
atd             0:off   1:off   2:off   3:on    4:on    5:on    6:off
 rwhod           0:off   1:off   2:off   3:off   4:off   5:off   6:off
 ...
```

### tcpdump

网络数据包 “嗅探器”。这是通过转储符合指定条件的数据包头的方法，来分析和网络流量故障定位的工具。

在主机*bozoville*和*caduceus*之间转储ip数据包流量:

```shell
bash$ tcpdump ip host bozoville and caduceus
```

当然，**tcpdump**的输出可以被我们之前讨论的[文本处理实用工具](https://tldp.org/LDP/abs/html/textproc.html#TPCOMMANDLISTING1)进行解析。

## 文件系统命令

### mount

挂载文件系统，通常是外部设备，如软盘或CDROM。文件`/etc/fstab`便捷地列出了可用文件系统、分区和设备的列表，以及包括可以自动或手动挂载的选项。文件`/etc/mtab`显示了当前挂载的文件系统和分区 (包括虚拟的，例如`/proc`)。

`mount -a`通过检索`/etc/fstab`列出所有已挂载的文件系统和分区，但带有`noauto`选项的文件系统和分区除外。在启动时，`/etc/rc.d` (`rc.sysinit`或类似的东西) 中的启动脚本调用此命令以挂载所有内容。

```shell
mount -t iso9660 /dev/cdrom /mnt/cdrom
# 挂载CD ROM。ISO 9660是一个标准的CD ROM文件系统。
mount /mnt/cdrom
# 简写，如果/mnt/cdrom在/etc/fstab列出。
```

多功能*mount*命令甚至可以在块设备上挂载普通文件，并且该文件将具有文件系统的性质。*Mount*通过将文件与[回环设备(loopback device)](https://tldp.org/LDP/abs/html/devref1.html#LOOPBACKREF)关联来完成此操作。这样做的一个应用场景是在将ISO9660文件系统映像刻录到CDR上之前，将其挂载并检查。[[3]](https://tldp.org/LDP/abs/html/system.html#FTN.AEN16255)

**样例 17-7. 检查一个CD映像**

```shell
# 作为root用户...

mkdir /mnt/cdtest  # 如果它不存在，请准备一个挂载点。

mount -r -t iso9660 -o loop cd-image.iso /mnt/cdtest   # 挂载映像。
#                  "-o loop" 选项等效于 "losetup /dev/loop0"
cd /mnt/cdtest     # 现在，检查一下映像。
ls -alR            # 在当前目录下，列出目录树中的文件。
                   # 等等。
```

### umount

卸载当前挂载的文件系统。在物理移除曾经挂载的软盘或CDROM磁盘之前，该设备必须使用**umount**取消挂载，否则可能会导致文件系统损坏。

```shell
umount /mnt/cdrom
# 你现在可以按下弹出按钮，安全地取出磁盘。
```

> ![note](https://tldp.org/LDP/abs/images/note.gif)如果已经安装有自动挂载实用程序，可以在访问或移除软盘或CDROM磁盘时自动挂载和卸载它们。然而，在带有可交换软驱和光驱的“多轴”笔记本电脑上，这可能会导致问题。

### gnome-mount

较新的Linux发行版已经弃用了**mount**和**umount**。取而代之的是用于可移动存储设备的命令行挂载工具。可以使用`-d`选项来挂载在`/dev`中列出的[设备文件](https://tldp.org/LDP/abs/html/devref1.html#DEVFILEREF)。

例如，要安装USB闪存驱动器:

```shell
bash$ gnome-mount -d /dev/sda1
gnome-mount 0.4


bash$ df
. . .
 /dev/sda1                63584     12034     51550  19% /media/disk
```

### sync

强制将所有更新的数据从缓冲区立即写入硬盘(使硬盘与缓冲区同步)。虽然并非绝对必要，但**sync**程序可以向系统管理员或用户保证，刚刚更改的数据将在突然断电后仍然存在。在过去，一个<code>**sync**;**sync**</code>(两次，只是为了确保万无一失)是系统重启前的一项有用的预防措施。

有时，你可能希望强制立即刷新缓冲区，比如当安全地删除一个文件时(参阅[样例 16-61](https://tldp.org/LDP/abs/html/extmisc.html#BLOTOUT))，或者当电源供应不足时。

### losetup

设置和配置[回环设备(loopback device)](https://tldp.org/LDP/abs/html/devref1.html#LOOPBACKREF)。

**样例 17-8. 在文件中创建一个文件系统**

```shell
SIZE=1000000  # 1 meg

head -c $SIZE < /dev/zero > file  # 创建一个指定大小的文件。
losetup /dev/loop0 file           # 将其设置为一个回环设备(loopback device)。
mke2fs /dev/loop0                 # 创建文件系统。
mount -o loop /dev/loop0 /mnt     # 挂载它。

# 感谢S.C.
```

### mkswap

创建一个swap分区或文件。交换区必须随后使用**swapon**命令进行激活。

### swapon, swapoff

挂载 / 卸载swap分区或文件。这些命令经常在启动和关机时起作用。

### mke2fs

创建一个Linux *ext2*文件系统。该命令必须被*root*用户所调用。

**样例 17-9. 添加一个新硬盘**

```shell
#!/bin/bash

# 给系统添加第二个硬盘。
# 软件配置。假设硬件已经安装。
# 该脚本由本书作者所著。
# 位于_Linux Gazette_中的issue #38，http://www.linuxgazette.com。

ROOT_UID=0     # 该脚本必须由root用户运行。
E_NOTROOT=67   # 非root用户错误退出码。

if [ "$UID" -ne "$ROOT_UID" ]
then
  echo "Must be root to run this script."
  exit $E_NOTROOT
fi  

# 非常小心地使用!
# 如果出现问题，你可能会擦除当前的文件系统。


NEWDISK=/dev/hdb         # 假设/dev/hdb空缺。请检查!
MOUNTPOINT=/mnt/newdisk  # 或者选择另外一个挂载点。


fdisk $NEWDISK
mke2fs -cv $NEWDISK1   # 检查坏块 (详细输出)。
#  注记：           ^     /dev/hdb1， *不是* /dev/hdb!
mkdir $MOUNTPOINT
chmod 777 $MOUNTPOINT  # 使所有用户都可以访问新硬盘。


# 现在，进行测试 ...
# mount -t ext2 /dev/hdb1 /mnt/newdisk
# 尝试创建一个目录。
# 如果没有问题，取消挂载，并且进行下一步骤。

# 最后一步：
# 将下一行添加至 /etc/fstab。
# /dev/hdb1  /mnt/newdisk  ext2  defaults  1 1

exit
```

另请参阅[样例 17-8](https://tldp.org/LDP/abs/html/system.html#CREATEFS)和[样例 31-3](https://tldp.org/LDP/abs/html/zeros.html#RAMDISK)。

### mkdosfs

创建一个DOS *FAT*文件系统。

### tune2fs

调整*ext2*文件系统。可用于更改文件系统参数，例如最大挂载数。该命令必须被*root*用户所调用。

> ![note](https://tldp.org/LDP/abs/images/warning.gif)这是一个极其危险的命令。使用它的风险将由你自己承担，因为你可能会无意中破坏你的文件系统。

### dumpe2fs

转储 (列表到`标准输出(stdout)`) 非常详细的文件系统信息。该命令必须被*root*用户所调用。

```shell
root# dumpe2fs /dev/hda7 | grep 'ount count'
dumpe2fs 1.19, 13-Jul-2000 for EXT2 FS 0.5b, 95/08/09
 Mount count:              6
 Maximum mount count:      20
```

### hdparm

列出或更改硬盘参数。此命令必须被*root*用户所调用。如果滥用，可能会非常危险。

### fdisk

创建或更改存储设备的分区表，通常是硬盘。该命令必须被*root*用户所调用。

> ![note](https://tldp.org/LDP/abs/images/warning.gif)使用该命令时要格外小心。如果哪里出现问题，你可能会破坏现有的文件系统。

### fsck, e2fsck, debugfs

文件系统检查、修复和调试命令集。

<strong>fsck：</strong>检查UNIX文件系统的前端（也能够被其他实用工具所调用）。实际的文件系统类型通常默认为*ext2*。

<strong>e2fsck：</strong>ext2文件系统检查器。

<strong>dubugfs：</strong>ext2文件系统调试器。这个多用途但危险的命令的用途之一是(试图)恢复被删除的文件。仅限老手使用！

> ![note](https://tldp.org/LDP/abs/images/caution.gif)以上这些命令需要被*root*用户所调用，如果滥用它们可能会对损坏甚至摧毁一个文件系统。

### badblocks

检查存储设备上的坏块(物理介质缺陷)。此命令在格式化新安装的硬盘或测试备份介质的完整性时非常有用。[[4]](https://tldp.org/LDP/abs/html/system.html#FTN.AEN16504)例如，通过执行**badblocks /dev/fd0**测试软盘。

**badblocks**命令可以以破坏性模式(覆盖所有数据)或以非破坏性只读模式调用。如果*root用户*拥有要测试的设备，通常情况下，*root*用户必须调用该命令。

### lsusb, usbmodules

**lsusb**命令列出所有usb(通用串行总线)总线和连接到它们的设备。

**usbmodules**命令输出所有已连接USB设备的驱动程序模块的信息。

```shell
bash$ lsusb
Bus 001 Device 001: ID 0000:0000  
 Device Descriptor:
   bLength                18
   bDescriptorType         1
   bcdUSB               1.00
   bDeviceClass            9 Hub
   bDeviceSubClass         0 
   bDeviceProtocol         0 
   bMaxPacketSize0         8
   idVendor           0x0000 
   idProduct          0x0000

   . . .
```

### lspci

列出目前所有的*pci*总线。

```shell
bash$ lspci
00:00.0 Host bridge: Intel Corporation 82845 845
 (Brookdale) Chipset Host Bridge (rev 04)
 00:01.0 PCI bridge: Intel Corporation 82845 845
 (Brookdale) Chipset AGP Bridge (rev 04)
 00:1d.0 USB Controller: Intel Corporation 82801CA/CAM USB (Hub #1) (rev 02)
 00:1d.1 USB Controller: Intel Corporation 82801CA/CAM USB (Hub #2) (rev 02)
 00:1d.2 USB Controller: Intel Corporation 82801CA/CAM USB (Hub #3) (rev 02)
 00:1e.0 PCI bridge: Intel Corporation 82801 Mobile PCI Bridge (rev 42)

   . . .
```

### mkbootdisk

创建一张引导软盘。当MBR(主引导记录)损坏，可以用它来启动系统。特别有趣的是`--iso`选项，它使用**mkisofs**创建一个可引导的*ISO9660*文件系统映像，并用于刻录可引导的CDR。

**mkbootdisk**命令实际上是一个由Erik Troan所写的Bash脚本，存放在`/sbin`目录下。

### mkisofs

创建适配CDR映像的*ISO9660*文件系统。

### chroot

更改 ROOT 目录。通常，命令是从相对于 / 的 [$PATH](https://tldp.org/LDP/abs/html/internalvariables.html#PATHREF) 中获取的，默认为*根目录*。该命令将*根*目录更改为另一个（同时也将工作目录更改到那里）。为了安全，该命令很有用。例如，当系统管理员希望将某些用户(如[远程登录](https://tldp.org/LDP/abs/html/communications.html#TELNETREF)的用户)限制在文件系统的安全部分时(这有时被称为将访客用户限制在“chroot监狱”中)。注意，在执行**chroot**命令之后，系统二进制文件的执行路径将不再有效。

执行**chroot /opt**会导致对`/usr/bin`的引用被转换为`/opt/usr/bin`。同样，`chroot /aaa/bbb /bin/ls`会将**ls**的实例重定向到<code>/aaa/bbb</code>作为基目录（base directory），而不是通常情况下的 / 目录。在用户的[~/.bashrc](https://tldp.org/LDP/abs/html/sample-bashrc.html)中添加一条<strong>alias XX 'chroot /aaa/bbb
 ls'</strong>有效地限制了她可以在文件系统的哪个部分运行命令“XX”。

**chroot**命令在使用紧急引导软盘(给`/dev/fd0`**chroot**)运行时也很方便，或者在从系统崩溃中恢复时作为**lilo**的一个选项。其他用途包括但不限于从不同的文件系统安装([rpm](https://tldp.org/LDP/abs/html/filearchiv.html#RPMREF)选项)或从CD ROM运行只读文件系统。该命令仅能被*root*用户所调用，并请小心使用。

> ![note](https://tldp.org/LDP/abs/images/caution.gif)可能需要将一些特定的系统文件拷贝到*chroot*改变后的目录下，因为不能再依赖正常的 \$PATH。

### lockfile

这个实用程序是**procmail**软件包([www.procmail.org](http://www.procmail.org))的一部分。该命令会创建一个*锁文件*，一个<strong>旗语(*semaphore*)</strong>来对文件、设备或资源进行访问控制。

| **定义：** *旗语*是一个标志或信号。（这种用法起源于铁路运输，用彩旗、灯笼或带条纹的活动臂*旗语*来表示某一特定轨道是否已被使用，因此不能供另一列火车使用。)UNIX进程可以检查*旗语*是否合适，以确定特定资源是否可用/可访问。 |
| --------------------------------------------------------------------------------------------------------------------- |

锁文件代表着该特定文件、设备或者资源正在被一个进程所占用（因此也是“忙”的）。锁文件的存在限制了其他进程的访问（或拒绝访问）。

```shell
lockfile /home/bozo/lockfiles/$0.lock
# 创建以脚本名称为前缀的写保护锁文件。

lockfile /home/bozo/lockfiles/${0##*/}.lock
# 由E. Choroba指出，以上代码的更安全的版本。
```

锁文件在如下的应用程序中使用，例如保护系统邮件文件夹不被多个用户同时更改，显示正在被访问的调制解调器(modem)端口，并显示有一个Firefox实例正在使用其缓存。脚本可能会检查某个进程创建的锁文件是否存在，以检查该进程是否正在运行。请注意，如果脚本尝试创建已经存在的锁定文件，则该脚本可能会被挂起。

通常，应用程序会在`/var/lock`目录中创建并检查锁文件。[[5]](https://tldp.org/LDP/abs/html/system.html#FTN.AEN16659)脚本可以通过以下方式来测试锁文件的存在性。

```shell
appname=xyzip
# 应用程序"xyzip"创建锁文件"/var/log/xyzip.lock"。

if [ -e "/var/lock/$appname.lock" ]
then   # 防止其他程序和脚本来
       # 访问xyzip使用的文件/资源。
  ...
```

### flock

**flock**远远不及**lockfile**那么有用。它在文件上设置 “咨询” 锁，然后在锁打开时执行命令。这是为了防止任何其他进程在指定命令完成之前就锁定该文件。

```shell
flock $0 cat $0 > lockfile__$0
#  当输出脚本本身到标准输出(stdout)时，
#  给上面的脚本上设置一个锁，
```

> ![note](https://tldp.org/LDP/abs/images/note.gif)不同于**lockfile**，**flock**<em>不会</em>自动创建一个锁文件。

### mknod

创建块或字符[设备文件](https://tldp.org/LDP/abs/html/devref1.html#DEVFILEREF) (在系统上安装新硬件时可能是必需的)。**MAKEDEV**实用程序几乎具有**mknod**的所有功能，并且更易于使用。

### MAKEDEV

创建设备文件的实用工具。必须由*root*用户运行，并且该程序位于`/dev`目录。这是**mknod**程序的一系列高级版本。

### tmpwatch

自动删除在指定时间内未被访问的文件。通常由[cron](https://tldp.org/LDP/abs/html/system.html#CRONREF)调用以删除陈旧的日志文件。

## 备份命令

### dump, restore

**dump**命令是一个精心设计的文件系统备份实用程序，通常在较大的系统安装和网络上使用。[[6]](https://tldp.org/LDP/abs/html/system.html#FTN.AEN16748)它读取原始磁盘分区，并以二进制格式写入备份文件。要备份的文件可能会保存到各种存储介质中，包括磁盘和磁带驱动器。**restore**命令用于恢复**dump**命令生成的备份文件。

### fdformat

在软盘 (<code>/dev/fd0*</code>) 上进行低阶格式化(low-level format)。

## 系统资源管理命令

### ulimit

设置系统资源使用的*上限*。通常在调用时加上`-f`选项，该选项用于设置文件大小限制 （如**ulimit -f 1000**将文件限制为最大1兆）。[[7]](https://tldp.org/LDP/abs/html/system.html#FTN.AEN16782)`-t`选项限制了核心转储(coredump)大小（(如**ulimit -c 0**禁止核心转储）。通常，将在`/etc/profile`和/或`~/.bash_profile`中设置**ulimit**的值 （参阅[附录H](https://tldp.org/LDP/abs/html/files.html)）。

> ![note](https://tldp.org/LDP/abs/images/important.gif)明智地使用**ulimit**可以保护系统免受可怕的*fork炸弹*的侵害。
> 
> ```shell
> #!/bin/bash
> # 该脚本仅用于说明。
> # 你需要自己承担运行该脚本的风险 -- 它会冻结你的系统。
> 
> while true  #  死循环。
> do
>   $0 &      #  这个脚本会调用它自己 . . .
>             #  fork无限次 . . .
>             #  直到系统冻结，因为所有资源耗尽。
> done        #  这是臭名昭著的 “巫师的应用” 场景。
> 
> exit 0      #  不会在此退出，因为该脚本永远不会终止。
> ```
> 
> `/etc/profile`中的**ulimit -Hu XX**（其中*XX*是用户进程限制）将在超过预设限制时中止此脚本。

### quota

显示用户或组磁盘配额。

### setquota

用命令行设置用户或组磁盘配额。

### umask

建立用户文件时预设的权限*掩码*。限制特定用户的默认文件属性。该用户创建的所有文件都将采用**umask**指定的属性。传递给**umask**的 (八进制) 值定义了*禁用*的文件权限。例如，**umask 022**确保新文件最多具有755权限 (777 NAND 022)。[[8]](https://tldp.org/LDP/abs/html/system.html#FTN.AEN16847)当然，用户以后可能会使用[chmod](https://tldp.org/LDP/abs/html/basic.html#CHMODREF)更改特定文件的属性。通常的做法是在`/etc/profile`和/或` ~/.bash_profile`中设置**umask**的值 （参阅[附录H](https://tldp.org/LDP/abs/html/files.html)）。

**样例 17-10. 使用*umask*命令来隐藏输出文件以防止窥探**

```shell
#!/bin/bash
# rot13a.sh: 与“rot13.sh”脚本相同，但将输出写入“安全”文件。

# 使用方式: ./rot13a.sh filename
# or     ./rot13a.sh <filename
# or     ./rot13a.sh and supply keyboard input (stdin)

umask 177               #  文件创建掩码。
                        #  由该文件创建出的脚本
                        #  将拥有600权限。

OUTFILE=decrypted.txt   #  输出到"decrypted.txt"的内容
                        #  将只能由脚本调用者(或root用户)
                        #  读/写。

cat "$@" | tr 'a-zA-Z' 'n-za-mN-ZA-M' > $OUTFILE 
#    ^^ 来自标准输入(stdin)或者文件的输入 ^^^^^^^^^^ 输出重定向到文件。

exit 0
```

### rdev

获取有关根设备、交换空间或视频模式的信息或对其进行更改。**rdev**的功能基本已由**lilo**顶替，但是**rdev**对于设置随机存储器(ram)磁盘仍然有用。如果滥用，这将会成为一个危险的命令。

## 模块管理命令

### lsmod

列出已经安装的内核模块。

```shell
bash$ lsmod
Module                  Size  Used by
 autofs                  9456   2 (autoclean)
 opl3                   11376   0
 serial_cs               5456   0 (unused)
 sb                     34752   0
 uart401                 6384   0 [sb]
 sound                  58368   0 [opl3 sb uart401]
 soundlow                 464   0 [sound]
 soundcore               2800   6 [sb sound]
 ds                      6448   2 [serial_cs]
 i82365                 22928   2
 pcmcia_core            45984   0 [serial_cs ds i82365]
```

> ![note](https://tldp.org/LDP/abs/images/note.gif)执行**cat /proc/modules**将会得到同样的信息。

### insmod

强制安装内核模块（请尽可能使用**modprobe**代替）。必须由*root*用户调用。

### rmmod

强制卸载内核模块。必须由*root*用户调用。

### modprobe

通常位于启动脚本中并自动调用的模块加载程序。必须由*root*用户调用。

### depmod

创建模块依赖文件，经常由启动脚本所调用。

### modinfo

输出一个可加载模块的信息。

```shell
bash$ modinfo hid
filename:    /lib/modules/2.4.20-6/kernel/drivers/usb/hid.o
 description: "USB HID support drivers"
 author:      "Andreas Gal, Vojtech Pavlik <vojtech@suse.cz>"
 license:     "GPL"
```

## 杂项命令

### env

运行包含有自定义[环境变量](https://tldp.org/LDP/abs/html/othertypesv.html#ENVREF)的程序或脚本（不改变整体系统环境）。<strong>`[varname=xxx]`</strong>允许在脚本运行期间更改环境变量<strong>`varname`</strong>。如果没有指定选项，该命令将列出所有环境变量设置。[[9]](https://tldp.org/LDP/abs/html/system.html#FTN.AEN16975)

> ![note](https://tldp.org/LDP/abs/images/note.gif)当shell或解释器的路径未知时，脚本的第一行（“sha-bang”行）可能会使用**env**。
> 
> ```shell
> #! /usr/bin/env perl
> 
> print "This Perl script will run,\n";
> print "even when I don't know where to find Perl.\n";
> 
> # 适用于可移植的跨平台脚本，
> # 其中Perl二进制文件可能不在预期的位置。
> # 感谢S.C.
> ```
> 
> 甚至 ...
> 
> ```shell
> #!/bin/env bash
> # 在$path环境变量中查询bash的位置。
> # 因此 ...
> # 该脚本可以执行那些Bash不在寻常地方的系统，比如说/bin。
> ```

### ldd

显示可执行文件的共享库(shared lib)依赖项。

```shell
bash$ ldd /bin/ls
libc.so.6 => /lib/libc.so.6 (0x4000c000)
/lib/ld-linux.so.2 => /lib/ld-linux.so.2 (0x80000000)
```

### watch

以指定的时间间隔重复地执行一条命令。

默认两秒的间隔，但可以通过`-n`选项进行更改。

```shell
watch -n 5 tail /var/log/messages
# 每5秒显示 /var/log/message 系统日志的末尾
```

>  ![note](https://tldp.org/LDP/abs/images/note.gif)不幸的是，无法通过[管道](https://tldp.org/LDP/abs/html/special-chars.html#PIPEREF)将**watch命令**的输出传输到[grep](https://tldp.org/LDP/abs/html/textproc.html#GREPREF)。

### strip

从可执行二进制文件中移除调试符号引用。这减小了二进制文件的大小，但是使调试变得不可能。

该命令经常出现在[Makefile](https://tldp.org/LDP/abs/html/filearchiv.html#MAKEFILEREF)中，但是很少出现在shell脚本中。

### nm

列出未被strip处理过的编译后二进制文件中的符号。

### xrandr

用于操作屏幕根窗口(root window)的命令行工具。

**样例 17-11. *背光*:更改（笔记本电脑）屏幕背光的亮度**

```shell
#!/bin/bash
# backlight.sh
# 发布时间 02dec2011

#  Fedora Core 16/17中的一个bug，它会弄乱键盘背光控制。
#  这个脚本是一个简单易用的变通方法，其本质上是xrandr的shell包装器。
#  相较于屏幕上的滑块和小部件，它提供了更多的控制内容。

OUTPUT=$(xrandr | grep LV | awk '{print $1}')   # 获取显示名称！@
INCR=.05      # 对于更细粒度的控制，请将INCR设置为.03或.02。

old_brightness=$(xrandr --verbose | grep rightness | awk '{ print $2 }')


if [ -z "$1" ]
then
  bright=1    # 如果没有命令行参数，将亮度设置为1.0(默认)。

  else
    if [ "$1" = "+" ]
    then
      bright=$(echo "scale=2; $old_brightness + $INCR" | bc)   # +.05

  else
    if [ "$1" = "-" ]
    then
      bright=$(echo "scale=2; $old_brightness - $INCR" | bc)   # -.05

  else
    if [ "$1" = "#" ]   # 输出当前的亮度；不改变它
    then
      bright=$old_brightness

  else
    if [[ "$1" = "h" || "$1" = "H" ]]
    then
      echo
      echo "Usage:"
      echo "$0 [No args]    Sets/resets brightness to default (1.0)."
      echo "$0 +            Increments brightness by 0.5."
      echo "$0 -            Decrements brightness by 0.5."
      echo "$0 #            Echoes current brightness without changing it."
      echo "$0 N (number)   Sets brightness to N (useful range .7 - 1.2)."
      echo "$0 h [H]        Echoes this help message."
      echo "$0 any-other    Gives xrandr usage message."

      bright=$old_brightness

  else
    bright="$1"

      fi
     fi
    fi
  fi
fi


xrandr --output "$OUTPUT" --brightness "$bright"   # 请参阅xrandr man手册。
                                                   # 使用root权限运行！
E_CHANGE0=$?
echo "Current brightness = $bright"

exit $E_CHANGE0


# ===========     或者  . . .   ==================== #

#!/bin/bash
# backlight2.sh
# 发布时间 20jun2012

#  Fedora Core 16/17中的一个bug，它会弄乱键盘背光控制。
#  这个脚本是一个简单易用的变通方法，可以替代backlight.sh。

target_dir=\
/sys/devices/pci0000:00/0000:00:01.0/0000:01:00.0/backlight/acpi_video0
# 硬件目录。

actual_brightness=$(cat $target_dir/actual_brightness)
max_brightness=$(cat $target_dir/max_brightness)
Brightness=$target_dir/brightness

let "req_brightness = actual_brightness"   # 请求的亮度。

if [ "$1" = "-" ]
then     # 降低一格亮度
  let "req_brightness = $actual_brightness - 1"
else
  if [ "$1" = "+" ]
  then   # 上升一格亮度。
    let "req_brightness = $actual_brightness + 1"
   fi
fi

if [ $req_brightness -gt $max_brightness ]
then
  req_brightness=$max_brightness
fi   # 不要超过硬件设计的最高亮度。

echo

echo "Old brightness = $actual_brightness"
echo "Max brightness = $max_brightness"
echo "Requested brightness = $req_brightness"
echo

# =====================================
echo $req_brightness > $Brightness
# 必须由root用户运行来使其生效。
E_CHANGE1=$?   # 成功吗？
# =====================================

if [ "$?" -eq 0 ]
then
  echo "Changed brightness!"
else
  echo "Failed to change brightness!"
fi

act_brightness=$(cat $Brightness)
echo "Actual brightness = $act_brightness"

scale0=2
sf=100 # 比例因子。
pct=$(echo "scale=$scale0; $act_brightness / $max_brightness * $sf" | bc)
echo "Percentage brightness = $pct%"

exit $E_CHANGE1
```

### rdist

远程分发客户端：同步、克隆或备份远程服务器上的文件系统。

## 注记

[[1]](https://tldp.org/LDP/abs/html/system.html#AEN14695)在Linux机器或具有磁盘配额的UNIX系统上就是这种情况。

[[2]](https://tldp.org/LDP/abs/html/system.html#AEN14727)如果需要被删除的特定用户处于登录状态，**userdel**命令将失败。

[[3]](https://tldp.org/LDP/abs/html/system.html#AEN16255)有关刻录CDR的更多详细信息，请参阅1999年10月期<em>[Linux Journal](http://www.linuxjournal.com/)</em>中Alex withers的文章 “[创建CD](http://www2.linuxjournal.com/lj-issues/issue66/3335.html)”。

[[4]](https://tldp.org/LDP/abs/html/system.html#AEN16504)**mke2fs**命令的`-c`参数还会检查是否有坏块。

[[5]](https://tldp.org/LDP/abs/html/system.html#AEN16659)因为只有*root*用户有`/var/lock`目录的写权限，普通用户的脚本不可以在此目录下设置锁文件。

[[6]](https://tldp.org/LDP/abs/html/system.html#AEN16748)单用户Linux系统的操作员通常更喜欢更简单的备份，例如**tar**。

[[7]](https://tldp.org/LDP/abs/html/system.html#AEN16782)从Bash的[版本4更新](https://tldp.org/LDP/abs/html/bashver4.html#BASH4REF)后，在[POSIX](https://tldp.org/LDP/abs/html/sha-bang.html#POSIX2REF)模式下，`-f`和`-c`选项的块大小为512。此外，还有两个新选项：`-b`表示[套接字(socket)](https://tldp.org/LDP/abs/html/devref1.html#SOCKETREF)缓冲区大小，`-T`表示*线程*数限制。

[[8]](https://tldp.org/LDP/abs/html/system.html#AEN16847)NAND是一个逻辑与-非运算符。它的作用有点类似于减法。

[[9]](https://tldp.org/LDP/abs/html/system.html#AEN16975)在Bash和其他Bourne shell派生中，可以在单命令环境下设置变量。

```shell
var1=value1 var2=value2 commandXXX
# $var1和$var2仅在环境'commandXXX'中生效。
```
