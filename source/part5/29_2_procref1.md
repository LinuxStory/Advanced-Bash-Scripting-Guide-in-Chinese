# 29.2 `/proc`

`/proc` 目录事实上是一个伪文件系统。`/proc` 中的文件反映了当前运行的系统以及内核进程和容器的信息和与它们相关的统计数据。

```console
bash$ cat /proc/devices
Character devices:
   1 mem
   2 pty
   3 ttyp
   4 ttyS
   5 cua
   7 vcs
  10 misc
  14 sound
  29 fb
  36 netlink
 128 ptm
 136 pts
 162 raw
 254 pcmcia

 Block devices:
   1 ramdisk
   2 fd
   3 ide0
   9 md



bash$ cat /proc/interrupts
           CPU0       
   0:      84505          XT-PIC  timer
   1:       3375          XT-PIC  keyboard
   2:          0          XT-PIC  cascade
   5:          1          XT-PIC  soundblaster
   8:          1          XT-PIC  rtc
  12:       4231          XT-PIC  PS/2 Mouse
  14:     109373          XT-PIC  ide0
 NMI:          0 
 ERR:          0


bash$ cat /proc/partitions
major minor  #blocks  name     rio rmerge rsect ruse wio wmerge wsect wuse running use aveq

    3     0    3007872 hda 4472 22260 114520 94240 3551 18703 50384 549710 0 111550 644030
    3     1      52416 hda1 27 395 844 960 4 2 14 180 0 800 1140
    3     2          1 hda2 0 0 0 0 0 0 0 0 0 0 0
    3     4     165280 hda4 10 0 20 210 0 0 0 0 0 210 210
    ...



bash$ cat /proc/loadavg
0.13 0.42 0.27 2/44 1119



bash$ cat /proc/apm
1.16 1.2 0x03 0x01 0xff 0x80 -1% -1 ?



bash$ cat /proc/acpi/battery/BAT0/info
present:                 yes
 design capacity:         43200 mWh
 last full capacity:      36640 mWh
 battery technology:      rechargeable
 design voltage:          10800 mV
 design capacity warning: 1832 mWh
 design capacity low:     200 mWh
 capacity granularity 1:  1 mWh
 capacity granularity 2:  1 mWh
 model number:            IBM-02K6897
 serial number:            1133
 battery type:            LION
 OEM info:                Panasonic
 
 
 
bash$ fgrep Mem /proc/meminfo
MemTotal:       515216 kB
 MemFree:        266248 kB
         
```

Shell脚本可以从 `/proc` 中的某些文件中提取数据。[^1]

```bash
FS=iso                       # 内核是否支持 ISO 文件系统？
grep $FS /proc/filesystems   # iso9660
```

```bash
kernel_version=$( awk '{ print $3 }' /proc/version )
```

```bash
CPU=$( awk '/model name/ {print $5}' < /proc/cpuinfo )

if [ "$CPU" = "Pentium(R)" ]
then
  run_some_commands
  ...
else
  run_other_commands
  ...
fi



cpu_speed=$( fgrep "cpu MHz" /proc/cpuinfo | awk '{print $4}' )

#  你的主机 CPU 的当前执行速度（以 MHZ 为单位）。
#  在笔记本上，根据电池或交流电源的使用情况，这会发生变化。
```

```bash
#!/bin/bash
# get-commandline.sh
# 获得进程的命令行参数。

OPTION=cmdline

# 识别 PID。
pid=$( echo $(pidof "$1") | awk '{ print $1 }' )
# 只获取                     ^^^^^^^^^^^^^^^^^^ 多个实例的第一个。

echo
echo "Process ID of (first instance of) "$1" = $pid"
echo -n "Command-line arguments: "
cat /proc/"$pid"/"$OPTION" | xargs -0 echo
#   格式化输出               ^^^^^^^^^^^^^^^
#   （感谢 Han Holl 修复问题！）

echo; echo


# 例如：
# sh get-commandline.sh xterm
```

```bash
devfile="/proc/bus/usb/devices"
text="Spd"
USB1="Spd=12"
USB2="Spd=480"


bus_speed=$(fgrep -m 1 "$text" $devfile | awk '{print $9}')
#                 ^^^^ Stop after first match.

if [ "$bus_speed" = "$USB1" ]
then
  echo "USB 1.1 port found."
  # Do something appropriate for USB 1.1.
fi
```

甚至有可能通过发送到 `/proc` 目录的命令来控制某些外围设备。

```console
root# echo on > /proc/acpi/ibm/light
```

这会打开某些型号 IBM/Lenovo Thinkpad 的 *Thinklight*。（可能不会在所有 Linux 发行版上生效。）

当然，在写入 `/proc` 时应谨慎。

`/proc` 目录包含一些不寻常的以数字为名的子目录。每一个名称都映射到当前运行的进程的进程 ID。在每一个子目录内，有一些文件保存着与对应进程有关的有用信息。`stat` 和 `status` 文件维护进程运行时的统计数据，`cmdline` 文件保存了进程被调用时的命令行参数，`exe` 文件是一个链接到调用进程的完整路径名称的符号链接。还有一些类似的文件，但前面这些是从编写脚本的角度来说最为感兴趣的。

**例 29-3. 找到与 PID 关联的进程**

```bash
#!/bin/bash
# pid-identifier.sh:
# 给出与 PID 关联的进程的完整路径名称。

ARGNO=1  # 脚本预期的参数数量。
E_WRONGARGS=65
E_BADPID=66
E_NOSUCHPROCESS=67
E_NOPERMISSION=68
PROCFILE=exe

if [ $# -ne $ARGNO ]
then
  echo "Usage: `basename $0` PID-number" >&2  # 错误信息 >stderr.
  exit $E_WRONGARGS
fi  

pidno=$( ps ax | grep $1 | awk '{ print $1 }' | grep $1 )
# 在“ps”列表，第一个字段中检查 pid。
# 然后确认这是个真实的进程，而不是被这个脚本调用的进程。
# 最后的 “grep $1”排除了这种可能性。
#
#    pidno=$( ps ax | awk '{ print $1 }' | grep $1 )
#    也可以，如 Teemu Huovila 指出。

if [ -z "$pidno" ]  #  如果，在所有过滤之后，结果是长度为 0 的字符串，
then                #+ 那么就没有与给定 pid 对应的运行进程。
  echo "No such process running."
  exit $E_NOSUCHPROCESS
fi  

# 另一种方法：
#   if ! ps $1 > /dev/null 2>&1
#   then                # 没有与给定 pid 对应的运行进程。
#     echo "No such process running."
#     exit $E_NOSUCHPROCESS
#    fi

# 要简化整个过程，可以使用 “pidof”。


if [ ! -r "/proc/$1/$PROCFILE" ]  # 检查读权限。
then
  echo "Process $1 running, but..."
  echo "Can't get read permission on /proc/$1/$PROCFILE."
  exit $E_NOPERMISSION  # 普通用户无法访问 /proc 中的某些文件。
fi  

# 最后两条测试可以用下面替换：
#    if ! kill -0 $1 > /dev/null 2>&1 # '0' 不是一个信号，但
                                      # 会测试是否可以向进程
                                      # 发送信号。
#    then echo "PID doesn't exist or you're not its owner" >&2
#    exit $E_BADPID
#    fi



exe_file=$( ls -l /proc/$1 | grep "exe" | awk '{ print $11 }' )
# 或者      exe_file=$( ls -l /proc/$1/exe | awk '{print $11}' )
#
#  /proc/pid-number/exe 是一个链接到调用进程的完整路径名的符号链接。

if [ -e "$exe_file" ]  #  如果 /proc/pid-number/exe 存在，
then                   #+ 那么对应进程就存在。
  echo "Process #$1 invoked by $exe_file."
else
  echo "No such process running."
fi  


#  
#  这个复杂的脚本可以*几乎*用下面的命令替代
#       ps ax | grep $1 | awk '{ print $5 }'
#  但是，这不会生效……
#+ 因为“ps”的第 5 个字段是进程的 argv[0]，而不是可执行文件的路径。
#
# 不过，下面的方法都是可行的。
#       find /proc/$1/exe -printf '%l\n'
#       lsof -aFn -p $1 -d txt | sed -ne 's/^n//p'

# Stephane Chazelas 的补充评论。

exit 0
```

**例 29-4. 在线连接状态**

```bash
#!/bin/bash
# connect-stat.sh
#  注意这个脚本可能需要针对无线连接做修改才能工作。

PROCNAME=pppd        # ppp 进程
PROCFILENAME=status  # 在哪儿看
NOTCONNECTED=85
INTERVAL=2           # 每 2 秒更新。

pidno=$( ps ax | grep -v "ps ax" | grep -v grep | grep $PROCNAME |
awk '{ print $1 }' )

# 找到“pppd”，“ppp 守护进程”的进程号。
# 必须过滤出由查询本身生成的进程行。
#
#  不过，正如 Oleg Philon 指出的，这可以通过使用“pidof”很好地简化。
#  pidno=$( pidof $PROCNAME )
#
#  这个故事的寓意：
#+ 当一个命令序列变得过于复杂时，请寻找捷径。


if [ -z "$pidno" ]   # 如果没有 pid，那么进程不再运行。
then
  echo "Not connected."
# exit $NOTCONNECTED
else
  echo "Connected."; echo
fi

while [ true ]       # 无限循环，这里脚本可以优化。
do

  if [ ! -e "/proc/$pidno/$PROCFILENAME" ]
  # 当进程运行时，“status”文件也就存在了。
  then
    echo "Disconnected."
#   exit $NOTCONNECTED
  fi

netstat -s | grep "packets received"  # 获得一些连接统计数据。
netstat -s | grep "packets delivered"


  sleep $INTERVAL
  echo; echo

done

exit 0


# 目前而言，这个脚本必须使用 Control-C 终止。

#    练习：
#    ---------
#    优化脚本，在键盘敲击“q”时退出。
#    用其他的方式使脚本更为用户友好。
#    修复脚本，使其可以与无线/DSL 连接工作。
```

> 总的来说，向 `/proc` 中的文件*写入*是危险的，因为这会损坏文件系统或毁了机器。

### 注

[^1]: 某些系统命令，例如 [procinfo](https://www.tldp.org/LDP/abs/html/system.html#PROCINFOREF)、[free](https://www.tldp.org/LDP/abs/html/system.html#FREEREF)、[vmstat](https://www.tldp.org/LDP/abs/html/system.html#VMSTATREF), [lsdev](https://www.tldp.org/LDP/abs/html/system.html#LSDEVREF) 和 [uptime](https://www.tldp.org/LDP/abs/html/system.html#UPTIMEREF) 也能很好地做到。