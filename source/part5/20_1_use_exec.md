# 20.1 使用 exec
一个 `exec < filename` 命令重定向了 `标准输入` 到一个文件。自此所有 `标准输入` 都来自该文件而不是默认来源(通常是键盘输入)。在使用 [sed](http://tldp.org/LDP/abs/html/sedawk.html#SEDREF) 和 [awk](http://tldp.org/LDP/abs/html/awk.html#AWKREF) 时候这种方式可以逐行读文件并逐行解析。

样例 20-1. 使用 `exec` 重定向 标准输入
```
#!/bin/bash
# 使用 'exec' 重定向 标准输入 .


exec 6<&0          # 链接文件描述符 #6 到标准输入.
                   # .

exec < data-file   # 标准输入被文件 "data-file" 替换

read a1            # 读取文件 "data-file" 首行.
read a2            # 读取文件 "data-file" 第二行

echo
echo "Following lines read from file."
echo "-------------------------------"
echo $a1
echo $a2

echo; echo; echo

exec 0<&6 6<&-
#  现在在之前保存的位置将从文件描述符 #6 将 标准输出 恢复.
#+ 且关闭文件描述符 #6 ( 6<&- ) 让其他程序正常使用.
#
# <&6 6<&-    also works.

echo -n "Enter data  "
read b1  # 现在按预期的，从正常的标准输入 "read".
echo "Input read from stdin."
echo "----------------------"
echo "b1 = $b1"

echo

exit 0
```

同理, `exec >filename` 重定向 标准输出 到指定文件. 他将所有的命令输出通常是 标准输出 重定向到指定的位置.

`exec N > filename` 影响整个脚本或当前 shell。[PID](http://tldp.org/LDP/abs/html/special-chars.html#PROCESSIDREF) 从重定向脚本或 shell 的那时候已经发生了改变. 然而 `N > filename` 影响的就是新派生的进程，而不是整个脚本或 shell。

样例 20-2. 使用 exec 重定向标准输出
```
#!/bin/bash
# reassign-stdout.sh

LOGFILE=logfile.txt

exec 6>&1           # 链接文件描述符 #6 到标准输出.
                    # 保存标准输出.

exec > $LOGFILE     # 标准输出被文件 "logfile.txt" 替换.

# ----------------------------------------------------------- #
# 所有在这个块里的命令的输出都会发送到文件 $LOGFILE.

echo -n "Logfile: "
date
echo "-------------------------------------"
echo

echo "Output of \"ls -al\" command"
echo
ls -al
echo; echo
echo "Output of \"df\" command"
echo
df

# ----------------------------------------------------------- #

exec 1>&6 6>&-      # 关闭文件描述符 #6 恢复 标准输出.

echo
echo "== stdout now restored to default == "
echo
ls -al
echo

exit 0
```

样例 20-3. 用 exec 在一个脚本里同时重定向 标准输入 和 标准输出
```
#!/bin/bash
# upperconv.sh
# 转化指定的输入文件成大写.

E_FILE_ACCESS=70
E_WRONG_ARGS=71

if [ ! -r "$1" ]     # 指定的输入文件是否可读?
then
  echo "Can't read from input file!"
  echo "Usage: $0 input-file output-file"
  exit $E_FILE_ACCESS
fi                   #  同样的错误退出
                     #+ 等同如果输入文件 ($1) 未指定 (为什么?).

if [ -z "$2" ]
then
  echo "Need to specify output file."
  echo "Usage: $0 input-file output-file"
  exit $E_WRONG_ARGS
fi


exec 4<&0
exec < $1            # 将从输入文件读取.

exec 7>&1
exec > $2            # 将写入输出文件.
                     # 假定输出文件可写 (增加检测?).

# -----------------------------------------------
    cat - | tr a-z A-Z   # 转化大写.
#   ^^^^^                # 读取标准输入.
#           ^^^^^^^^^^   # 写到标准输出.
# 然而标准输入和标准输出都会被重定向.
# 注意 'cat' 可能会被遗漏.
# -----------------------------------------------

exec 1>&7 7>&-       # 恢复标准输出.
exec 0<&4 4<&-       # 恢复标准输入.

# 恢复后, 下面这行会预期从标准输出打印.
echo "File \"$1\" written to \"$2\" as uppercase conversion."

exit 0
```

I/O 重定向是种明智的规避 [inaccessible variables within a subshell](http://tldp.org/LDP/abs/html/subshells.html#PARVIS) 问题的方法.

样例 20-4. 规避子 shell
```
#!/bin/bash
# avoid-subshell.sh
# Matthew Walker 的建议.

Lines=0

echo

cat myfile.txt | while read line;
                 do {
                   echo $line
                   (( Lines++ ));  #  递增变量的值趋近外层循环
                                   #  使用子 shell 会有问题.
                 }
                 done

echo "Number of lines read = $Lines"     # 0
                                         # 报错!

echo "------------------------"


exec 3<> myfile.txt
while read line <&3
do {
  echo "$line"
  (( Lines++ ));                   #  递增变量的值趋近外层循环.
                                   #  没有子 shell，就不会有问题.
}
done
exec 3>&-

echo "Number of lines read = $Lines"     # 8

echo

exit 0

# 下面的行并不在脚本里.

$ cat myfile.txt

Line 1.
Line 2.
Line 3.
Line 4.
Line 5.
Line 6.
Line 7.
Line 8.
```

