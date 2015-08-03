### 本节翻译进度 40%

---

# 11.1 循环

循环是当循环控制条件为真时，一系列命令迭代[^1]执行的代码块。

### for 循环

### `for arg in [list]`

这是 shell 中最基本的循环结构，它与C语言形式的循环有着明显的不同。

```bash
for arg in [list]
do
  command(s)...
done
```

> ![note](http://tldp.org/LDP/abs/images/note.gif) 在循环的过程中，`arg` 会从 `list` 中连续获得每一个变量的值。
>
```bash
for arg in "$var1" "$var2" "$var3" ... "$varN"
# 第一次循环中，arg = $var1
# 第二次循环中，arg = $var2
# 第三次循环中，arg = $var3
# ...
# 第 N 次循环中，arg = $varN
>
# 为了防止可能的字符分割问题，[list] 中的参数都需要被引用。
```

参数 list 中允许含有 [通配符](http://tldp.org/LDP/abs/html/special-chars.html#ASTERISKREF)。

如果 `do` 和 `for` 写在同一行时，需要在 list 之后加上一个分号。

`for arg in [list] ; do`

样例 11-1. 简单的 for 循环

```bash
#!/bin/bash
# 列出太阳系的所有行星。

for planet in Mercury Venus Earth Mars Jupiter Saturn Uranus Neptune Pluto
do
  echo $planet  # 每一行输出一个行星。
done

echo; echo

for planet in "Mercury Venus Earth Mars Jupiter Saturn Uranus Neptune Pluto"
    # 所有的行星都输出在一行上。
    # 整个 'list' 被包裹在引号中时是作为一个单一的变量。
    # 为什么？因为空格也是变量的一部分。
do
  echo $planet
done

echo; echo "Whoops! Pluto is no longer a planet!"

exit 0
```

[list] 中的每一个元素中都可能含有多个参数。这在处理参数组中非常有用。在这种情况下，使用 [`set`](http://tldp.org/LDP/abs/html/internal.html#SETREF) 命令（查看 [样例 15-16](http://tldp.org/LDP/abs/html/internal.html#EX34)）强制解析 [list] 中的每一个元素，并将元素的每一个部分分配给位置参数。

样例 11-2. `for` 循环 [list] 中的每一个变量有两个参数的情况

```bash
#!/bin/bash
# 让行星再躺次枪。

# 将每个行星与其到太阳的距离放在一起。

for planet in "Mercury 36" "Venus 67" "Earth 93" "Mars 142" "Jupiter 483"
do
  set -- $planet  #  解析变量 "planet"
                  #+ 并将其每个部分赋值给位置参数。
  # "--" 防止一些极端情况，比如 $planet 为空或者以破折号开头。
  
  # 因为位置参数会被覆盖掉，因此需要先保存原先的位置参数。
  # 你可以使用数组来保存
  #         original_params=("$@")
  
  echo "$1		$2,000,000 miles from the sum"
  #-------两个制表符---将后面的一系列 0 连到参数 $2 上。
done

# （感谢 S.C. 做出的额外注释。）

exit 0
```

一个单一变量也可以成为 `for` 循环中的 [list]。

样例 11-3. 文件信息：查看一个单一变量中含有的文件列表的文件信息

```bash
#!/bin/bash
# fileinfo.sh

FILES="/usr/sbin/accept
/usr/sbin/pwck
/usr/sbin/chroot
/usr/bin/fakefile
/sbin/badblocks
/sbin/ypbind"     # 你可能会感兴趣的一系列文件。
                  # 包含一个不存在的文件，/usr/bin/fakefile。
                  
echo

for file in $FILES
do

  if [ ! -e "$file" ]       # 检查文件是否存在。
  then
    echo "$file does not exist."; echo
    continue                # 继续判断下一个文件。
  fi
  
  ls -l $file | awk '{ print $8 "         file size: " $5 }'  # 输出其中的两个域。
  whatis `basename $file`   # 文件信息。
  # 脚本正常运行需要注意提前设置好 whatis 的数据。
  # 使用 root 权限运行 /usr/bin/makewhatis 可以完成。
  echo
done

exit 0
```

`for` 循环中的 [list] 可以是一个参数。

样例 11-4. 操作含有一系列文件的参数

```bash
#!/bin/bash

filename="*txt"

for file in $filename
do
 echo "Contents of $file"
 echo "---"
 cat "$file"
 echo
done
```

如果在匹配文件扩展名的 `for` 循环中的 [list] 含有通配符（* 和 ?），那么将会进行文件名扩展。

样例 11-5. 在 `for` 循环中操作文件

```bash
#!/bin/bash
# list-glob.sh: 通过文件名扩展在 for 循环中产生 [list]。
# 通配 = 文件名扩展。

echo

for file in *
#           ^  Bash 在检测到通配表达式时，
#+             会进行文件名扩展。
do
  ls -l "$file"  # 列出 $PWD（当前工作目录）下的所有文件。
  #  回忆一下，通配符 "*" 会匹配所有的文件名，
  #+ 但是，在文件名扩展中，他将不会匹配以点开头的文件。
  
  #  如果没有匹配到文件，那么它将会扩展为它自身。
  #  为了防止出现这种情况，需要设置 nullglob 选项。
  #+    (shopt -s nullglob)。
  #  感谢 S.C.
done

echo; echo

for file in [jx]*
do
  rm -f $file    # 删除当前目录下所有以 "j" 或 "x" 开头的文件。
  echo "Removed file \"$file\"".
done

echo

exit 0
```

如果在 `for` 循环中省略 `in [list]` 部分，那么循环将会遍历位置参数（`$@`）。[样例 A-15](http://tldp.org/LDP/abs/html/contributed-scripts.html#PRIMES) 中使用到了这一点。也可以查看 [样例 15-17](http://tldp.org/LDP/abs/html/internal.html#REVPOSPARAMS)。

样例 11-6. 缺少 `in [list]` 的 `for` 循环

```bash
#!/bin/bash

# 尝试在带参数和不带参数两种情况下调用这个脚本，观察发生了什么。

for a
do
 echo -n "$a "
done

#  缺失 'in list' 的情况下，循环会遍历 '$@'
#+（命令行参数列表，包括空格）。

echo

exit 0
```

可以在 `for` 循环中使用 [命令代换](http://tldp.org/LDP/abs/html/commandsub.html#COMMANDSUBREF) 生成 [list]。查看 [样例 16-54](http://tldp.org/LDP/abs/html/extmisc.html#EX53)，[样例 11-11](http://tldp.org/LDP/abs/html/loops1.html#SYMLINKS) 和 [样例 16-48](http://tldp.org/LDP/abs/html/mathc.html#BASE)。

样例 11-7. 在 `for` 循环中使用命令代换生成 [list]

```bash
#!/bin/bash
# for-loopcmd.sh: 带命令代换所生成 [list] 的 for 循环

NUMBERS="9 7 3 8 37.53"

for number in `echo $NUMBERS`  # for number in 9 7 3 8 37.53
do
  echo -n "$number "
done

echo
exit 0
```

下面是使用命令代换生成 [list] 的更加复杂的例子。

样例 11-8. 一种替代 `grep` 搜索二进制文件的方法

```bash
#!/bin/bash
# bin-grep.sh: 在二进制文件中定位匹配的字符串。

# 一种替代 `grep` 搜索二进制文件的方法
# 与 "grep -a" 的效果类似

E_BADARGS=65
E_NOFILE=66

if [ $# -ne 2 ]
then
  echo "Usage: `basename $0` search_string filename"
  exit $E_BADARGS
fi

if [ ! -f "$2" ]
then
  echo "File \"$2\" does not exist."
  exit $E_NOFILE
fi


IFS=$'\012'       # 按照 Anton Filippov 的意见应该是
                  # IFS="\n"
for word in $( strings "$2" | grep "$1" )
# "strings" 命令列出二进制文件中的所有字符串。
# 将结果通过管道输出到 "grep" 中，检查是不是匹配的字符串。
do
  echo $word
done

# 就像 S.C. 指出的那样，第 23-30 行可以换成下面的形式：
#    strings "$2" | grep "$1" | tr -s "$IFS" '[\n*]'


# 尝试运行脚本 "./bin-grep.sh mem /bin/ls"

exit 0
```

下面的例子同样展示了如何使用命令代换生成 [list]。

样例 11-9. 列出系统中的所有用户

```bash
#!/bin/bash
# userlist.sh

PASSWORD_FILE=/etc/passwd
n=1           # 用户数量

for name in $(awk 'BEGIN{fs=":"}{print $1}' < "$PASSWORD_FILE" )
# 分隔符 = :              ^^^^^^
# 输出第一个域                    ^^^^^^^^
# 读取密码文件 /etc/passwd                    ^^^^^^^^^^^^^^^^^
do
  echo "USER #$n = $name"
  let "n += 1"
done


# USER #1 = root
# USER #2 = bin
# USER #3 = daemon
# ...
# USER #33 = bozo

exit $?

# 讨论：
# -----
# 一个普通用户是如何读取 /etc/passwd 文件的？
# 提示：检查 /etc/passwd 的文件权限。
# 这算不算是一个安全漏洞？为什么？
```