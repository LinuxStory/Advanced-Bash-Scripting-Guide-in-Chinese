# 附录 F. I/O 与 I/O 重定向详细介绍

一个指令会预期前三个文件描述符可用。第一个， *fd 0* (标准输入， `stdin`) 用于读取。其他两个 (*fd 1*, `stdout` 和 *fd 2*, `stderr`) 用于写入。

每个指令都关联着一个 `stdin`, `stdout` 和 `stderr` 。**`ls 2>&1`** 意味着临时将 ls 的 `stderr` 连接到与 Shell 的 `stdout` 一样的「资源」。

按照惯例，指令从 fd 0 (`stdin`) 读取输入，正常输出打印到 fd 1 (`stdin`)，错误输出打印到 fd 2 (`stderr`)。如果这三个文件描述符有一个没被打开，你可能会碰上问题：

```bash
bash$ cat /etc/passwd >&-
cat: standard output: Bad file descriptor
```

例如，当 **xterm** 运行，它首先自我初始化。在运行用户的 Shell 之前， **xterm** 会打开终端设备 (/dev/pts/<n> 或类似) 三次。

此时 Bash 继承这三个文件描述符，且每个 Bash 运行的指令 (子进程) 因此继承它们，除非你重定向它们。重定向意味着重设一个文件描述符到另一个文件 (或者管道，或任何有权限的东西)。文件描述符可以本地重设 (对于一个指令、指令组、子 Shell、while / case / for 循环等)，或者，对余下的 Shell 会话全局重设 (通过 exec)。

`ls > /dev/null` 意味着在其文件描述符 1 连接到 `/dev/null` 的情况下运行 **ls** 。

```bash
bash$ lsof -a -p $$ -d0,1,2
COMMAND PID     USER   FD   TYPE DEVICE SIZE NODE NAME
 bash    363 bozo        0u   CHR  136,1         3 /dev/pts/1
 bash    363 bozo        1u   CHR  136,1         3 /dev/pts/1
 bash    363 bozo        2u   CHR  136,1         3 /dev/pts/1


bash$ exec 2> /dev/null
bash$ lsof -a -p $$ -d0,1,2
COMMAND PID     USER   FD   TYPE DEVICE SIZE NODE NAME
 bash    371 bozo        0u   CHR  136,1         3 /dev/pts/1
 bash    371 bozo        1u   CHR  136,1         3 /dev/pts/1
 bash    371 bozo        2w   CHR    1,3       120 /dev/null


bash$ bash -c 'lsof -a -p $$ -d0,1,2' | cat
COMMAND PID USER   FD   TYPE DEVICE SIZE NODE NAME
 lsof    379 root    0u   CHR  136,1         3 /dev/pts/1
 lsof    379 root    1w  FIFO    0,0      7118 pipe
 lsof    379 root    2u   CHR  136,1         3 /dev/pts/1


bash$ echo "$(bash -c 'lsof -a -p $$ -d0,1,2' 2>&1)"
COMMAND PID USER   FD   TYPE DEVICE SIZE NODE NAME
 lsof    426 root    0u   CHR  136,1         3 /dev/pts/1
 lsof    426 root    1w  FIFO    0,0      7520 pipe
 lsof    426 root    2w  FIFO    0,0      7520 pipe
```

## 练习

分析以下脚本。

```bash
#! /usr/bin/env bash

mkfifo /tmp/fifo1 /tmp/fifo2
while read a; do echo "FIFO1: $a"; done < /tmp/fifo1 & exec 7> /tmp/fifo1
exec 8> >(while read a; do echo "FD8: $a, to fd7"; done >&7)

exec 3>&1
(
 (
  (
   while read a; do echo "FIFO2: $a"; done < /tmp/fifo2 | tee /dev/stderr \
   | tee /dev/fd/4 | tee /dev/fd/5 | tee /dev/fd/6 >&7 & exec 3> /tmp/fifo2

   echo 1st, to stdout
   sleep 1
   echo 2nd, to stderr >&2
   sleep 1
   echo 3rd, to fd 3 >&3
   sleep 1
   echo 4th, to fd 4 >&4
   sleep 1
   echo 5th, to fd 5 >&5
   sleep 1
   echo 6th, through a pipe | sed 's/.*/PIPE: &, to fd 5/' >&5
   sleep 1
   echo 7th, to fd 6 >&6
   sleep 1
   echo 8th, to fd 7 >&7
   sleep 1
   echo 9th, to fd 8 >&8

  ) 4>&1 >&3 3>&- | while read a; do echo "FD4: $a"; done 1>&3 5>&- 6>&-
 ) 5>&1 >&3 | while read a; do echo "FD5: $a"; done 1>&3 6>&-
) 6>&1 >&3 | while read a; do echo "FD6: $a"; done 3>&-

rm -f /tmp/fifo1 /tmp/fifo2


# 对每个指令和子 Shell，猜猜哪个 fd 指向什么。
# 祝你好运！

exit 0
```
