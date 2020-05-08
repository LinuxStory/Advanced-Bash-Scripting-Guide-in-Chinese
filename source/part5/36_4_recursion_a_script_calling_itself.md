# 36.4 递归：可以调用自己的脚本

脚本可以递归的调用自己吗？答案是肯定的。

## Example 36-10. 可以调用自己的脚本（但没什么实际用途）

```
#!/bin/bash
# recurse.sh

# 脚本可以调用自己吗？
# 其实是可以的。但这样有什么实际用途吗？
# （请往下看）

RANGE=10
MAXVAL=9

i=$RANDOM
let "i %= $RANGE"  # 在0到$RANGE - 1的范围内产生一个随机数

if [ "$i" -lt "$MAXVAL" ]
then
    echo "i = $i"
    ./$0           # 脚本进行递归调用（调用自己）
fi                 # 每次被调用的脚本做同样的事情，直到$i和$MAXVAL相等。 

# 如果使用“while”循环代替“if/then”语句会出问题。请试着解释为什么。

exit 0

# 笔记:
# ----
# 这个脚本文件必须有可执行权限。
# 即使使用“sh”命令调用，这脚本也可以执行。
# 请解释原因。
```

## Example 36-11. 一个有点用的调用自己的脚本

```
#!/bin/bash
# pb.sh: phone book

# 用于权限管理的脚本，由Rick Boivie编写。
# ABS作者稍有修改

MINARGS=1     # 需要至少一个参数
DATAFILE=./phonebook
              # 当前目录下必须存在名为“phonebook”的数据文件
PROGNAME=$0
E_NOARGS=70   # 没有错误

if [ $# -lt $MINARGS ]; then
    echo "Usage: "$PROGNAME" data-to-look-up"
    exit $E_NOARGS
fi      

if [ $# -eq $MINARGS ]; then
    grep $1 "$DATAFILE"
    # 如果$DATAFILE没有匹配则'grep'命令会报错。
else
    ( shift; "$PROGNAME" $* ) | grep $1
    # 脚本的递归调用
fi
exit 0        # 脚本结束 

# 下面是一些文件内容

# ------------------------------------------------------------------------
一个简单的"phonebook"数据文件:

John Doe        1555 Main St., Baltimore, MD 21228          (410) 222-3333
Mary Moe        9899 Jones Blvd., Warren, NH 03787          (603) 898-3232
Richard Roe     856 E. 7th St., New York, NY 10009          (212) 333-4567
Sam Roe         956 E. 8th St., New York, NY 10009          (212) 444-5678
Zoe Zenobia     4481 N. Baker St., San Francisco, SF 94338  (415) 501-1631
# ------------------------------------------------------------------------

$bash pb.sh Roe
Richard Roe     856 E. 7th St., New York, NY 10009          (212) 333-4567
Sam Roe         956 E. 8th St., New York, NY 10009          (212) 444-5678

$bash pb.sh Roe Sam
Sam Roe         956 E. 8th St., New York, NY 10009          (212) 444-5678

# 当对脚本传入多于一个参数时，脚本只显示包含所有参数的行
```

## Example 36-12. 另一个调用自己的脚本

```
#!/bin/bash
# usrmnt.sh, 由Anthony Richardson编写
# 在ABS Guide中用于权限管理

# usage:       usrmnt.sh
# description: 想使用挂载设备操作的用户，在/etc/sudoers文件中必须属于MNTUSERS组。

# ----------------------------------------------------------
# 这个脚本会返回加了sudo命令的自身。
# 如果一个有权限的用户，则只需要输入：
#   usermount /dev/fd0 /mnt/floppy
# 而不需要使用下面的方法：
#   sudo usermount /dev/fd0 /mnt/floppy

# 我对于所有需要sudo的脚本都使用了这个技术，因为我发现这让我感觉非常舒服。
# ----------------------------------------------------------

# 如果SUDO_COMMAND变量没有被设置，那证明没有使用sudo命令运行。这需要
# 再重新运行这个脚本，同时传递用户的ID和组ID...

if [ -z "$SUDO_COMMAND" ]
then
    mntusr=$(id -u) grpusr=$(id -g) sudo $0 $* # 译注：脚本调用自己，并且传递参数
    exit 0
fi

# 如果使用了sudo运行，就不会卡在这里了。
/bin/mount $* -o uid=$mntusr,gid=$grpusr

exit 0

# 附加说明：
# -------------------------------------------------

# 1) Linux系统允许/etc/fstab文件中列出的用户挂在移动存储设备。但在服务器上，我喜欢让更少的人访问移动存储。我发现使用sudo可以帮我做到。

# 2) 我还发现使用sudo比用组权限来实现让人感觉更加舒服。

# 3) 这种方法可以给任何有权限的人使用mount命令，所以要小心处理。
#    你也可以将这种技术用到比如mntfloppy，mntcdrom和mntsamba等脚本上来实现更优雅的挂载管理。
```

过多层次的递归调用会导致脚本的栈空间溢出，引起段错误（segfault）。
