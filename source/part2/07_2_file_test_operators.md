# 7.2 文件测试操作

下列每一个运算符在满足其下条件时，返回的结果为真。

### -e

检测文件是否存在

### -a

检测文件是否存在

等价于 `-e`。不推荐使用，已被弃用[^1]。

### -f

文件是常规文件(regular file)，而非目录或 [设备文件](http://tldp.org/LDP/abs/html/devref1.html#DEVFILEREF)

### -s

文件大小不为0

### -d

文件是一个目录

### -b

文件是一个 [块设备](http://tldp.org/LDP/abs/html/devref1.html#BLOCKDEVREF)

### -c

文件是一个 [字符设备](http://tldp.org/LDP/abs/html/devref1.html#CHARDEVREF)

```bash
device0="/dev/sda2"    # /   (根目录)
if [ -b "$device0" ]
then
  echo "$device0 is a block device."
fi

# /dev/sda2 是一个块设备。



device1="/dev/ttyS1"   # PCMCIA 调制解调卡
if [ -c "$device1" ]
then
  echo "$device1 is a character device."
fi

# /dev/ttyS1 是一个字符设备。
```

### -p

文件是一个 [管道设备](http://tldp.org/LDP/abs/html/special-chars.html#PIPEREF)

```bash
function show_input_type()
{
   [ -p /dev/fd/0 ] && echo PIPE || echo STDIN
}

show_input_type "Input"                           # STDIN
echo "Input" | show_input_type                    # PIPE

# 这个例子由 Carl Anderson 提供。
```

### -h

文件是一个 [符号链接](http://tldp.org/LDP/abs/html/basic.html#SYMLINKREF)

### -L

文件是一个符号链接

### -S

文件是一个 [套接字](http://tldp.org/LDP/abs/html/devref1.html#SOCKETREF)

### -t

文件（[文件描述符](http://tldp.org/LDP/abs/html/io-redirection.html#FDREF)）与终端设备关联

该选项通常被用于 [测试](http://tldp.org/LDP/abs/html/intandnonint.html#II2TEST) 脚本中的 `stdin [ -t 0 ]` 或 `stdout [ -t 1 ]` 是否为终端设备。

### -r

该文件对执行测试的用户可读

### -w

该文件对执行测试的用户可写

### -x

该文件可被执行测试的用户所执行

### -g

文件或目录设置了 set-group-id `sgid` 标志

如果一个目录设置了 `sgid` 标志，那么在该目录中所有的新建文件的权限组都归属于该目录的权限组，而非文件创建者的权限组。该标志对共享文件夹很有用。

### -u

文件设置了 set-user-id `suid` 标志。

一个属于 root 的可执行文件设置了 `suid` 标志后，即使是一个普通用户执行也拥有 root 权限[^2]。对需要访问硬件设备的可执行文件（例如 `pppd` 和 `cdrecord`）很有用。如果没有 `suid` 标志，这些可执行文件就不能被非 root 用户所调用了。

```
-rwsr-xr-t    1 root       178236 Oct  2  2000 /usr/sbin/pppd
```

设置了 `suid` 标志后，在权限中会显示 `s`。

### -k

设置了粘滞位(sticky bit)。

标志粘滞位是一种特殊的文件权限。如果文件设置了粘滞位，那么该文件将会被存储在高速缓存中以便快速访问[^3]。如果目录设置了该标记，那么它将会对目录的写权限进行限制，目录中只有文件的拥有者可以修改或删除文件。设置标记后你可以在权限中看到 `t`。

```
drwxrwxrwt    7 root         1024 May 19 21:26 tmp/
```

如果一个用户不是设置了粘滞位目录的拥有者，但对该目录有写权限，那么他仅仅可以删除目录中他所拥有的文件。这可以防止用户不经意间删除或修改其他人的文件，例如 `/tmp` 文件夹。（当然目录的所有者可以删除或修改该目录下的所有文件）

### -O

执行用户是文件的拥有者

### -G

文件的组与执行用户的组相同

### -N

文件在在上次访问后被修改过了

### f1 -nt f2

文件 f1 比文件 f2 新

### f1 -ot f2

文件 f1 比文件 f2 旧

### f1 -ef f2

文件 f1 和文件 f2 硬链接到同一个文件

### !

取反——对测试结果取反(如果条件缺失则返回真)。

样例 7-4. 检测链接是否损坏

```bash
#!/bin/bash
# broken-link.sh
# Lee bigelow <ligelowbee@yahoo.com> 编写。
# ABS Guide 经许可可以使用。

#  该脚本用来发现输出损坏的链接。输出的结果是被引用的，
#+ 所以可以直接导到 xargs 中进行处理 ：）
#  例如：sh broken-link.sh /somedir /someotherdir|xargs rm
#
#  更加优雅的方式：
#
#  find "somedir" -type 1 -print0|\
#  xargs -r0 file|\
#  grep "broken symbolic"|
#  sed -e 's/^\|: *broken symbolic.*$/"/g'
#
#  但是这种方法不是纯 Bash 写法。
#  警告：小心 /proc 文件下的文件和任意循环链接！
############################################


#  如果不给脚本传任何参数，那么 directories-to-search 设置为当前目录
#+ 否则设置为传进的参数
#####################

[ $# -eq 0 ] && directorys=`pwd` || directorys=$@


#  函数 linkchk 是用来检测传入的文件夹中是否包含损坏的链接文件，
#+ 并引用输出他们。
#  如果文件夹中包含子文件夹，那么将子文件夹继续传给 linkchk 函数进行检测。
#################

linkchk () {
    for element in $1/*; do
      [ -h "$element" -a ! -e "$element" ] && echo \"$element\"
      [ -d "$element" ] && linkchk $element
    # -h 用来检测是否是链接，-d 用来检测是否是文件夹。
    done
}

#  检测传递给 linkchk() 函数的参数是否是一个存在的文件夹，
#+ 如果不是则报错。
################
for directory in $direcotrys; do
    if [ -d $directory ]
        then linkchk $directory
        else
            echo "$directory is not a directory"
            echo "Usage $0 dir1 dir2 ..."
    fi
done

exit $?
```

[样例 31-1](http://tldp.org/LDP/abs/html/zeros.html#COOKIES)，[样例 11-8](http://tldp.org/LDP/abs/html/loops1.html#BINGREP)，[样例 11-3](http://tldp.org/LDP/abs/html/loops1.html#FILEINFO)，[样例 31-3](http://tldp.org/LDP/abs/html/zeros.html#RAMDISK)和[样例 A-1](http://tldp.org/LDP/abs/html/contributed-scripts.html#MAILFORMAT) 也包含了测试运算符的使用。

[^1]: 摘自1913年版本的韦氏词典<br><pre>Deprecate<br>...<br><br>To pray against, as an evil;<br>to seek to avert by prayer;<br>to desire the removal of;<br>to seek deliverance from;<br>to express deep regret for;<br>to disapprove of strongly.</pre>
[^2]: 注意使用 suid 的可执行文件可能会带来安全问题。suid 标记对 shell 脚本没有影响。
[^3]: 在 Linux 系统中，文件已经不使用粘滞位了, 粘滞位只作用于目录。

