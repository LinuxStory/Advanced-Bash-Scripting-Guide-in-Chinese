# 29.1 `/dev`

`/dev` 目录包含硬件中可能存在或不存在的*物理设备*的条目[^1]。这些条目很恰当地被称为*设备文件*。作为一个例子，包含挂载了文件系统的硬盘分区在 `/dev` 中就有条目，如 [df](https://www.tldp.org/LDP/abs/html/system.html#DFREF) 所示。

```bash
bash$ df
Filesystem           1k-blocks      Used Available Use%  Mounted on
 /dev/hda6               495876    222748    247527  48% /
 /dev/hda1                50755      3887     44248   9% /boot
 /dev/hda8               367013     13262    334803   4% /home
 /dev/hda5              1714416   1123624    503704  70% /usr
```

其中，`/dev` 目录包含*回环*设备，例如 `/dev/loop0`。回环设备是是一种小花招，它允许像访问块设备一样访问普通文件。这就允许将整个文件系统挂载到单个大文件内[^2]。参见[例 17-8](https://www.tldp.org/LDP/abs/html/system.html#CREATEFS) 和[例 17-7](https://www.tldp.org/LDP/abs/html/system.html#ISOMOUNTREF)。

`/dev` 中的一些伪设备有其他的特殊用途，例如 [`/dev/null`](https://www.tldp.org/LDP/abs/html/zeros.html#ZEROSREF)、[`/dev/zero`](https://www.tldp.org/LDP/abs/html/zeros.html#ZEROSREF1)、[`/dev/urandom`](https://www.tldp.org/LDP/abs/html/randomvar.html#URANDOMREF)、`/dev/sda1`（硬盘分区）、`/dev/udp`（*用户数据报文*端口）和 [`/dev/tcp`](#DEVTCP)。

例如：

要手动挂载一个 USB 闪存，应将下面的行添加到 `/etc/fstab`。[^3]

```
/dev/sda1    /mnt/flashdrive    auto    noauto,user,noatime    0 0
```

（另请参见[例 A-23](https://www.tldp.org/LDP/abs/html/contributed-scripts.html#USBINST)）。

检查一个磁盘是否在 CD 刻录机中（软链接到 `/dev/hdc`）：

```bash
head -1 /dev/hdc


#  head: cannot open '/dev/hdc' for reading: No medium found
#  (No disc in the drive.)

#  head: error reading '/dev/hdc': Input/output error
#  (There is a disk in the drive, but it can't be read;
#+  possibly it's an unrecorded CDR blank.)   

#  Stream of characters and assorted gibberish
#  (There is a pre-recorded disk in the drive,
#+ and this is raw output -- a stream of ASCII and binary data.)
#  Here we see the wisdom of using 'head' to limit the output
#+ to manageable proportions, rather than 'cat' or something similar.


#  Now, it's just a matter of checking/parsing the output and taking
#+ appropriate action.
```

当对 `/dev/tcp/$host/$port` 伪设备文件执行命令时，Bash 打开一个到关联*套接字*的 TCP 连接。

> *套接字*（*socket*）是与特定 I/O 端口关联的通信节点。（这与连接线的*硬件接口*，或*插座*类似）。它允许同一个机器上、同一个网络的机器之间、不同网络的机器之间以及（当然）位于互联网不同位置的机器之间进行数据传输。

下面的示例假设有一个有效的互联网连接。

从 `nist.gov` 获取时间：

```bash
bash$ cat </dev/tcp/time.nist.gov/13
53082 04-03-18 04:26:54 68 0 0 502.3 UTC(NIST) *      
```

【Mark 贡献了此示例。】

将上述内容概括为一个脚本：

```bash
#!/bin/bash
# 本脚本必须以 root 权限运行

URL="time.nist.gov/13"

Time=$(cat </dev/tcp/"$URL")
UTC=$(echo "$Time" | awk '{print$3}')   # 第三个字段是 UTC (GMT) 时间。
# 练习：修改为不同的时区。

echo "UTC Time = "$UTC""
```

下载一个 URL：

```bash
bash$ exec 5<>/dev/tcp/www.net.cn/80
bash$ echo -e "GET / HTTP/1.0\n" >&5
bash$ cat <&5
```

【感谢 Mark 和 Mihai Maties。】

**<a name="DEVTCP">例 29-1. 使用 `/dev/tcp` 进行排错</a>**

```bash
#!/bin/bash
# dev-tcp.sh: /dev/tcp 重定向来验证互联网连接。

# Script by Troy Engel.
# Used with permission.
 
TCP_HOST=news-15.net       # 一个已知的垃圾邮件友好的 ISP。
TCP_PORT=80                # 端口 80 是 http。
  
# 尝试连接。（有些类似 “ping”……）
echo "HEAD / HTTP/1.0" >/dev/tcp/${TCP_HOST}/${TCP_PORT}
MYEXIT=$?

: <<EXPLANATION
如果 bash 以 --enable-net-redirections 编译，
那么它就可以为 TCP 和 UDP 重定向而使用特殊的字节设备。
这些重定向的用法和 STDIN/STDOUT/STDERR 一样。
/dev/tcp 设备的条目是 30,36：

  mknod /dev/tcp c 30 36

>摘自 bash 参考：
/dev/tcp/host/port
    如果 host 是合法的主机名或互联网地址，port 是整数端口号或服务名，Bash 会尝试打开到
对应套接字的 TCP 连接。
EXPLANATION

   
if [ "X$MYEXIT" = "X0" ]; then
  echo "Connection successful. Exit code: $MYEXIT"
else
  echo "Connection unsuccessful. Exit code: $MYEXIT"
fi

exit $MYEXIT
```

**例 29-2. 播放音乐**

```bash
#!/bin/bash
# music.sh

# Music without external files

# Author: Antonio Macchi
# Used in ABS Guide with permission.


#  /dev/dsp default = 8000 frames per second, 8 bits per frame (1 byte),
#+ 1 channel (mono)

duration=2000       # If 8000 bytes = 1 second, then 2000 = 1/4 second.
volume=$'\xc0'      # Max volume = \xff (or \x00).
mute=$'\x80'        # No volume = \x80 (the middle).

function mknote ()  # $1=Note Hz in bytes (e.g. A = 440Hz ::
{                   #+ 8000 fps / 440 = 16 :: A = 16 bytes per second)
  for t in `seq 0 $duration`
  do
    test $(( $t % $1 )) = 0 && echo -n $volume || echo -n $mute
  done
}

e=`mknote 49`
g=`mknote 41`
a=`mknote 36`
b=`mknote 32`
c=`mknote 30`
cis=`mknote 29`
d=`mknote 27`
e2=`mknote 24`
n=`mknote 32767`
# European notation.

echo -n "$g$e2$d$c$d$c$a$g$n$g$e$n$g$e2$d$c$c$b$c$cis$n$cis$d \
$n$g$e2$d$c$d$c$a$g$n$g$e$n$g$a$d$c$b$a$b$c" > /dev/dsp
# dsp = Digital Signal Processor

exit      # A "bonny" example of an elegant shell script!
```

### 注

[^1]:`/dev` 中的条目为物理和虚拟设备提供了挂载点。这些条目几乎不占用磁盘空间。某些设备，例如 `/dev/null`、`dev/zero` 和 `/dev/urandom` 是虚拟的。它们不是真实的物理设备，只以软件形式存在。

[^2]:与以*字符*为单位访问数据的*字符设备*相比，*块设备*以大段或*块*的形式读取和/或写入数据。块设备的例子有硬盘、CD 光盘以及闪存。字符设备的例子有键盘、调制解调器、声卡。

[^3]: 当然，挂载点 `/mnt/flashdrive` 必须存在。如果没有，那么以 *root* 执行 **mkdir /mnt/flashdrive**。要真正挂载磁盘，请使用这个命令：**mount /mnt/flashdrive**。新版本 Linux 发行版自动挂载闪存到 `/media` 目录而无需用户介入。