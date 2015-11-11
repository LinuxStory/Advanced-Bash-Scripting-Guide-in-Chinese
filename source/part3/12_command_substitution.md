### 本章翻译进度 40%

---

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
