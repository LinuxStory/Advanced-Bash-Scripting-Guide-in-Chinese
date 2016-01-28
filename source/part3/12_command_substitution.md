# 第十二章 命令替换

命令替换重新指定一个[^1]或多个命令的输出。其实就是将命令的输出导到另外一个地方[^2]。

命令替换的通常形式是（`` `...` ``），即用反引号引用命令。

```bash
script_name=`basename $0`
echo "The name of this script is $scirpt_name."
```

命令的输出可以作为另一个命令的参数，也可以赋值给一个变量。甚至在 [`for`](http://tldp.org/LDP/abs/html/loops1.html#FORLOOPREF1) 循环中可以用输出产生参数表。

```bash
rm `cat filename`   # "filename" 中包含了一系列需要被删除的文件名。
#
# S.C. 指出这样写可能会导致出现 "arg list too long" 的错误。
# 更好的写法应该是 xargs rm -- < filename
# （ -- 可以在 "filename" 文件名以 "-" 为开头时仍旧正常执行 ）

textfile_listing=`ls *.txt`
# 变量中包含了当前工作目录下所有的名为 *.txt 的文件。
echo $textfile_listing

textfile_listing2=$(ls *.txt)   # 命令替换的另一种形式。
echo $textfile_listing2
# 结果相同。

# 这样将一系列文件名赋值给一个单一字符串可能会出现换行。
#
# 而更加安全的方式是将这一系列文件存入数组。
#      shopt -s nullglob    # 设置后，如果没有匹配到文件，那么变量会被赋值为空。
#      textfile_listing=( *.txt )
#
# 感谢 S.C.
```

> ![note](http://tldp.org/LDP/abs/images/note.gif) 命令替换本质上是调用了一个 [子进程](http://tldp.org/LDP/abs/html/subshells.html#SUBSHELLSREF) 来执行。

> ![caution](http://tldp.org/LDP/abs/images/caution.gif) 命令替换有可能会出现 [字符分割](http://tldp.org/LDP/abs/html/quotingvar.html#WSPLITREF) 的情况。

> ```bash
> COMMAND `echo a b`     # 2个参数：a和b
>
> COMMAND "`echo a b`"   # 1个参数："a b"
>
> COMMAND `echo`         # 没有参数
> 
> COMMAND "`echo`"       # 一个空参数
> 
> 
> # 感谢 S.C.
> ```

> 但即使不存在字符分割的情况，使用命令替换也会出现丢失尾部换行符的情况。

> ```bash
> # cd "`pwd`"  # 你是不是认为这条语句在任何情况下都不会出现错误？
> # 但事实却不是这样的。
> 
> mkdir 'dir with trailing newline
> '
> 
> cd 'dir with trailing newline
> '
> 
> cd "`pwd`"  # Bash 会出现如下错误提示：
> # bash: cd: /tmp/file with trailing newline: No such file or directory
> 
> cd "$PWD"   # 这样写是对的。
> 
> 
> 
> 
> 
> old_tty_setting=$(stty -g)   # 保存旧的设置。
> echo "Hit a key "
> stty -icanon -echo           # 禁用终端的 canonical 模式。
>                              # 同时禁用 echo。
> key=$(dd bs=1 count=1 2> /dev/null)   # 使用 'dd' 获得键值。
> stty "$old_tty_setting"      # 恢复旧的设置。
> echo "You hit ${#key} key."  # ${#variable} 表示 $variable 中的字符个数。
> #
> # 除了按下回车键外，其余情况都会输出 "You hit 1 key."
> # 按下回车键会输出 "You hit 0 key."
> # 因为唯一的换行符在命令替换中被丢失了。
> 
> # 这段代码摘自 Stéphane Chazelas。
> ```

> ![caution](http://tldp.org/LDP/abs/images/caution.gif) 使用 `echo` 输出未被引用的命令代换的变量时会删掉尾部的换行。这可能会导致非常不好的情况出现。

> ```bash
> dir_listing=`ls -l`
> echo $dir_listing     # 未被引用
> 
> # 你希望会出现按行显示出文件列表。
> 
> # 但是，你却看到了：
> # total 3 -rw-rw-r-- 1 bozo bozo 30 May 13 17:15 1.txt -rw-rw-r-- 1 bozo
> # bozo 51 May 15 20:57 t2.sh -rwxr-xr-x 1 bozo bozo 217 Mar 5 21:13 wi.sh
> 
> # 所有换行都消失了。
> 
> 
> echo "$dir_listing"   # 被引用
> # -rw-rw-r--    1 bozo       30 May 13 17:15 1.txt
> # -rw-rw-r--    1 bozo       51 May 15 20:57 t2.sh
> # -rwxr-xr-x    1 bozo      217 Mar  5 21:13 wi.sh
> ```

你甚至可以使用 [重定向](http://tldp.org/LDP/abs/html/io-redirection.html#IOREDIRREF) 或者 [`cat`](http://tldp.org/LDP/abs/html/basic.html#CATREF) 命令把一个文件的内容通过命令代换赋值给一个变量。

```bash
variable1=`<file1`      # 将 "file1" 的内容赋值给 variable1。
variable2=`cat file2`   # 将 "file2" 的内容赋值给 variable2。
                        # 使用 cat 命令会开一个新进程，因此执行速度会比重定向慢。

# 需要注意的是，这些变量中可能包含一些空格或者控制字符。

# 无需显示的赋值给一个变量。
echo "` <$0`"           # 输出脚本自身。
```

```bash
#  摘录自系统文件 /etc/rc.d/rc.sysinit
#+ （Red Hat Linux 发行版）


if [ -f /fsckoptions ]; then
        fsckoptions=`cat /fsckoptions`
...
fi
#
#
if [ -e "/proc/ide/${disk[$device]}/media" ] ; then
             hdmedia=`cat /proc/ide/${disk[$device]}/media`
...
fi
#
#
if [ ! -n "`uname -r | grep -- "-"`" ]; then
       ktag="`cat /proc/version`"
...
fi
#
#
if [ $usb = "1" ]; then
    sleep 5
    mouseoutput=`cat /proc/bus/usb/devices 2>/dev/null|grep -E "^I.*Cls=03.*Prot=02"`
    kbdoutput=`cat /proc/bus/usb/devices 2>/dev/null|grep -E "^I.*Cls=03.*Prot=01"`
...
fi
```

> ![caution](http://tldp.org/LDP/abs/images/caution.gif) 尽量不要将一大段文字赋值给一个变量，除非你有足够的理由。也绝不要将一个二进制文件的内容赋值给一个变量。

> 样例 12-1. 蠢蠢的脚本
> 
> ```bash
> #!/bin/bash
> # stupid-script-tricks.sh: 不要在自己的电脑上尝试。
> # 摘自 "Stupid Script Tricks" 卷一。
> 
> exit 99  ### 如果你有胆，就注释掉这行。:)
> 
> dangerous_variable=`cat /boot/vmlinuz`   # 压缩的 Linux 内核。
> 
> echo "string-length of \$dangerous_variable = ${#dangerous_variable}"
> # $dangerous_variable 的长度为 794151
> # （更新版本的内核可能更大。）
> # 与 'wc -c /boot/vmlinuz' 的结果不同。
> 
> # echo "$dangerous_variable"
> # 不要作死。否则脚本会挂起。
> 
> 
> 
> # 将二进制文件的内容赋值给一个变量没有任何意义。
> 
> exit 0
> ```

> 尽管脚本会挂起，但并不会出现缓存溢出的情况。而这正是像 Bash 这样的解释型语言相比起编译型语言能够提供更多保护的一个例子。

命令替换允许将 [循环](http://tldp.org/LDP/abs/html/loops1.html#FORLOOPREF1) 的输出结果赋值给一个变量。这其中的关键在于循环内部的 [`echo`](http://tldp.org/LDP/abs/html/internal.html#ECHOREF) 命令。

样例 12-2. 将循环的输出结果赋值给变量

```bash
#!/bin/bash
# csubloop.sh: 将循环的输出结果赋值给变量。

variable1=`for i in 1 2 3 4 5
do
  echo -n "$i"                 #  在这里，'echo' 命令非常关键。
done`

echo "variable1 = $variable1"  # variable1 = 12345


i=0
variable2=`while [ "$i" -lt 10 ]
do
  echo -n "$i"                 # 很关键的 'echo'。
  let "i += 1"                 # i 自增。
done`

echo "variable2 = $variable2"  # variable2 = 0123456789

# 这个例子表明可以在变量声明时嵌入循环。

exit 0
```

> 命令替换能够让 Bash 做更多的事情。而这仅仅需要在书写程序或者脚本时将结果输出到标准输出 `stdout` 中，然后将这些输出结果赋值给变量即可。
> 
> ```c
> #include <stdio.h>
>
> /*  "Hello, world." C program  */
> 
> int main()
> {
>   printf( "Hello, world.\n" );
>   return (0);
> }
> ```
> 
> ```
> bash$ gcc -0 hello hello.c
> ```
> 
> ```bash
> #!/bin/bash
> # hello.sh
> 
> greeting=`./hello`
> echo $greeting
> ```
> 
> ```
> bash$ sh hello.sh
> Hello, world.
> ```



> ![note](http://tldp.org/LDP/abs/images/note.gif) 在命令替换中，你可以使用 `$(...)` 来替代反引号。
> 
> ```bash
> output=$(sed -n /"$1"/p $file)   # 摘自 "grp.sh"。
> 
> # 将文本文件的内容赋值给一个变量。
> File_contents1=$(cat $file1)
> File_contents2=$(<$file2)        # 这么做也是可以的。
> ```
> 
> `$(...)` 和反引号在处理双反斜杠上有所不同。
> 
> ```
> bash$ echo `echo \\`
> 
> 
> bash$ echo $(echo \\)
> \
> ```
> 
> `$(...)` 允许嵌套。[^3]
> 
> ```bash
> word_count=$( wc -w $(echo * | awk '{print $8}') )
> ```
> 
> 样例 12-3. 寻找变位词（anagram）
> 
> ```bash
> #!/bin/bash
> # agram2.sh
> # 嵌套命令替换的例子。
> 
> # 其中使用了作者写的工具包 "yawl" 中的 "anagram" 工具。
> # http://ibiblio.org/pub/Linux/libs/yawl-0.3.2.tar.gz
> # http://bash.deta.in/yawl-0.3.2.tar.gz
> 
> E_NOARGS=86
> E_BADARG=87
> MINLEN=7
> 
> if [ -z "$1" ]
> then
>   echo "Usage $0 LETTERSET"
>   exit $E_NOARGS         # 脚本需要命令行参数。
> elif [ ${#1} -lt $MINLEN ]
> then
>   echo "Argument must have at least $MINLEN letters."
>   exit $E_BADARG
> fi
> 
> 
> 
> FILTER='.......'         # 至少需要7个字符。
> #       1234567
> Anagrams=( $(echo $(anagram $1 | grep $FILTER) ) )
> #          $(     $(        嵌套命令集        ) )
> #        (              赋值给数组                )
> 
> echo
> echo "${#Anagrams[*]}  7+ letter anagrams found"
> echo
> echo ${Anagrams[0]}      # 第一个变位词。
> echo ${Anagrams[1]}      # 第二个变位词。
>                          # 以此类推。
> 
> # echo "${Anagrams[*]}"  # 将所有变位词在一行里面输出。
> 
> # 可以配合后面的数组章节来理解上面的代码。
> 
> # 建议同时查看另一个寻找变位词的脚本 agram.sh。
> 
> exit $?
> ```

以下是包含命令替换的样例：

1. [样例 11-8](http://tldp.org/LDP/abs/html/loops1.html#BINGREP)
2. [样例 11-27](http://tldp.org/LDP/abs/html/testbranch.html#CASECMD)
3. [样例 9-16](http://tldp.org/LDP/abs/html/randomvar.html#SEEDINGRANDOM)
4. [样例 16-3](http://tldp.org/LDP/abs/html/moreadv.html#EX57)
5. [样例 16-22](http://tldp.org/LDP/abs/html/textproc.html#LOWERCASE)
6. [样例 16-17](http://tldp.org/LDP/abs/html/textproc.html#GRP)
7. [样例 16-54](http://tldp.org/LDP/abs/html/extmisc.html#EX53)
8. [样例 11-14](http://tldp.org/LDP/abs/html/loops1.html#EX24)
9. [样例 11-11](http://tldp.org/LDP/abs/html/loops1.html#SYMLINKS)
10. [样例 16-32](http://tldp.org/LDP/abs/html/filearchiv.html#STRIPC)
11. [样例 20-8](http://tldp.org/LDP/abs/html/redircb.html#REDIR4)
12. [样例 A-16](http://tldp.org/LDP/abs/html/contributed-scripts.html#TREE)
13. [样例 29-3](http://tldp.org/LDP/abs/html/procref1.html#PIDID)
14. [样例 16-47](http://tldp.org/LDP/abs/html/mathc.html#MONTHLYPMT)
15. [样例 16-48](http://tldp.org/LDP/abs/html/mathc.html#BASE)
16. [样例 16-49](http://tldp.org/LDP/abs/html/mathc.html#ALTBC)

[^1]: 在命令替换中可以使用外部系统命令，[内建命令](http://tldp.org/LDP/abs/html/internal.html#BUILTINREF) 甚至是 [脚本函数](http://tldp.org/LDP/abs/html/assortedtips.html#RVT)。

[^2]: 从技术的角度来讲，命令替换实际上是获得了命令输出到标准输出的结果，然后通过赋值号将结果赋值给一个变量。

[^3]: 事实上，使用反引号进行嵌套也是可行的。但是 John Default 提醒到需要将内部的反引号进行转义。<pre>word_count=\` wc -w \\\`echo * | awk '{print $8}'\\\` \`</pre>