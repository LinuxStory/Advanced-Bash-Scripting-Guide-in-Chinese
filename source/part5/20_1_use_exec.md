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
read b1  # Now "read" functions as expected, reading from normal stdin.
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
# Converts a specified input file to uppercase.

E_FILE_ACCESS=70
E_WRONG_ARGS=71

if [ ! -r "$1" ]     # Is specified input file readable?
then
  echo "Can't read from input file!"
  echo "Usage: $0 input-file output-file"
  exit $E_FILE_ACCESS
fi                   #  Will exit with same error
                     #+ even if input file ($1) not specified (why?).

if [ -z "$2" ]
then
  echo "Need to specify output file."
  echo "Usage: $0 input-file output-file"
  exit $E_WRONG_ARGS
fi


exec 4<&0
exec < $1            # Will read from input file.

exec 7>&1
exec > $2            # Will write to output file.
                     # Assumes output file writable (add check?).

# -----------------------------------------------
    cat - | tr a-z A-Z   # Uppercase conversion.
#   ^^^^^                # Reads from stdin.
#           ^^^^^^^^^^   # Writes to stdout.
# However, both stdin and stdout were redirected.
# Note that the 'cat' can be omitted.
# -----------------------------------------------

exec 1>&7 7>&-       # Restore stout.
exec 0<&4 4<&-       # Restore stdin.

# After restoration, the following line prints to stdout as expected.
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

