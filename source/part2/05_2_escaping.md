# 5.2 转义

转义是一种引用单字符的方法。通过在特殊字符前加上转义符 `\` 来告诉shell按照字面意思去解释这个字符。

> ![notice](http://tldp.org/LDP/abs/images/caution.gif) 需要注意的是，在一些特定的命令和工具，比如 `echo` 和 `sed` 中，转义字符通常会起到相反的效果，即可能会使得那些字符产生特殊含义。

在 `echo` 与 `sed` 命令中，转义字符的特殊含义

### \n

换行（line feed）。

### \r

回车（carriage return）。

### \t

水平制表符。

### \v

垂直制表符。

### \b

退格。

### \a

警报、响铃或闪烁。

### \0xx

ASCII码的八进制形式，等价于 `0nn`，其中 `nn` 是数字。

> ![important](http://tldp.org/LDP/abs/images/important.gif) 在 `$' ... '` 字符串扩展结构中可以通过转义八进制或十六进制的ASCII码形式给变量赋值，比如 `quote=$'\042'`。

样例 5-2. 转义字符

```bash
#!/bin/bash
# escaped.sh: 转义字符

##############################################
### 首先让我们先看一下转义字符的基本用法。 ###
##############################################

# 转义新的一行。
# ------------

echo ""

echo "This will print
as two lines."
# This will print
# as two lines.

echo "This will print \
as one line."
# This will print as one line.

echo; echo

echo "============="


echo "\v\v\v\v"      # 按字面意思打印 \v\v\v\v
# 使用 echo 命令的 -e 选项来打印转义字符。
echo "============="
echo "VERTICAL TABS"
echo -e "\v\v\v\v"   # 打印四个垂直制表符。
echo "=============="

echo "QUOTATION MARK"
echo -e "\042"       # 打印 " （引号，八进制ASCII码为42）。
echo "=============="



# 使用 $'\X' 这样的形式后可以不需要加 -e 选项。

echo; echo "NEWLINE and (maybe) BEEP"
echo $'\n'           # 新的一行。
echo $'\a'           # 警报（响铃）。
                     # 根据不同的终端版本，也可能是闪屏。

# 我们之前介绍了 $'\nnn' 字符串扩展，而现在我们要看到的是...

# ============================================ #
# 自 Bash 第二个版本开始的 $'\nnn' 字符串扩展结构。
# ============================================ #

echo "Introducing the \$\' ... \' string-expansion construct . . . "
echo ". . . featuring more quotation marks."

echo $'\t \042 \t'   # 在制表符之间的引号。
# 需要注意的是 '\nnn' 是一个八进制的值。

# 字符串扩展同样适用于十六进制的值，格式是 $'\xhhh'。
echo $'\t \x22 \t'  # 在制表符之间的引号。
# 感谢 Greg Keraunen 指出这些。
# 在早期的 Bash 版本中允许使用 '\x022' 这样的形式。

echo


# 将 ASCII 码字符赋值给变量。
# -----------------------
quote=$'\042'        # 将 " 赋值给变量。
echo "$quote Quoted string $quote and this lies outside the quotes."

echo

# 连接多个 ASCII 码字符给变量。
triple_underline=$'\137\137\137'  # 137是 '_' ASCII码的八进制形式
echo "$triple_underline UNDERLINE $triple_underline"

echo

ABC=$'\101\102\103\010'           # 101，102，103是 A, B, C 
                                  # ASCII码的八进制形式。
echo $ABC

echo

escape=$'\033'                    # 033 是 ESC 的八进制形式
echo "\"escape\" echoes an $escape"
                                  # 没有可见输出

echo

exit 0
```

下面是一个更加复杂的例子：

样例 5-3. 检测键盘输入

```bash
#!/bin/bash
# 作者：Sigurd Solaas，作于2011年4月20日
# 授权在《高级Bash脚本编程指南》中使用。
# 需要 Bash 版本高于4.2。

key="no value yet"
while true; do
  clear
  echo "Bash Extra Keys Demo. Keys to try:"
  echo
  echo "* Insert, Delete, Home, End, Page_Up and Page_Down"
  echo "* The four arrow keys"
  echo "* Tab, enter, escape, and space key"
  echo "* The letter and number keys, etc."
  echo
  echo "    d = show date/time"
  echo "    q = quit"
  echo "================================"
  echo
  
  # 将独立的Home键值转换为数字7上的Home键值：
  if [ "$key" = $'\x1b\x4f\x48' ]; then
   key=$'\x1b\x5b\x31\x7e'
   #   引用字符扩展结构。
  fi
  
  # 将独立的End键值转换为数字1上的End键值：
  if [ "$key" = $'\x1b\x4f\x46' ]; then
   key=$'\x1b\x5b\x34\x7e'
  fi
  
  case "$key" in
   $'\x1b\x5b\x32\x7e')  # 插入
    echo Insert Key
   ;;
   $'\x1b\x5b\x33\x7e')  # 删除
    echo Delete Key
   ;;
   $'\x1b\x5b\x31\x7e')  # 数字7上的Home键
    echo Home Key
   ;;
   $'\x1b\x5b\x34\x7e')  # 数字1上的End键
    echo End Key
   ;;
   $'\x1b\x5b\x35\x7e')  # 上翻页
    echo Page_Up
   ;;
   $'\x1b\x5b\x36\x7e')  # 下翻页
    echo Page_Down
   ;;
   $'\x1b\x5b\x41')  # 上箭头
    echo Up arrow
   ;;
   $'\x1b\x5b\x42')  # 下箭头
    echo Down arrow
   ;;
   $'\x1b\x5b\x43')  # 右箭头
    echo Right arrow
   ;;
   $'\x1b\x5b\x44')  # 左箭头
    echo Left arrow
   ;;
   $'\x09')  # 制表符
    echo Tab Key
   ;;
   $'\x0a')  # 回车
    echo Enter Key
   ;;
   $'\x1b')  # ESC
    echo Escape Key
   ;;
   $'\x20')  # 空格
    echo Space Key
   ;;
   d)
    date
   ;;
   q)
    echo Time to quit...
    echo
    exit 0
   ;;
   *)
    echo Your pressed: \'"$key"\'
   ;;
  esac
  
  echo
  echo "================================"
  
  unset K1 K2 K3
  read -s -N1 -p "Press a key: "
  K1="$REPLY"
  read -s -N2 -t 0.001
  K2="$REPLY"
  read -s -N1 -t 0.001
  K3="$REPLY"
  key="$K1$K2$K3"
  
done

exit $?
```

还可以查看样例 37-1。

### \\"

转义引号，指代自身。

```bash
echo "Hello"                     # Hello
echo "\"Hello\" ... he said."    # "Hello" ... he said.
```

### \\$

转义美元符号（跟在 `\\$` 后的变量名将不会被引用）。

```bash
echo "\$variable01"           # $variable01
echo "The book cost \$7.98."  # The book cost $7.98.
```

### \\\\

转义反斜杠，指代自身。

```bash
echo "\\"  # 结果是 \

# 然而...

echo "\"   # 在命令行中会出现第二行并提示输入。
           # 在脚本中会出错。
           
# 但是...

echo '\'   # 结果是 \
```

> ![extra](http://tldp.org/LDP/abs/images/note.gif) 根据转义符所在的上下文（强引用、弱引用，命令替换或者在 here document）的不同，它的行为也会有所不同。
>
```bash
                      #  简单转义与引用
echo \z               #  z
echo \\z              # \z
echo '\z'             # \z
ehco '\\z'            # \\z
echo "\z"             # \z
echo "\\z"            # \z
>
                      #  命令替换
echo `echo \z`        #  z
echo `echo \\z`       #  z
echo `echo \\\z`      # \z
echo `echo \\\\z`     # \z
echo `echo \\\\\\z`   # \z
echo `echo \\\\\\\z`  # \\z
echo `echo "\z"`      # \z
echo `echo "\\z"`     # \z
>
                      # Here Document
cat <<EOF
\z
EOF                   # \z
>
cat <<EOF
\\z
EOF                   # \z
>
# 以上例子由 Stéphane Chazelas 提供。 
```
> 含有转义字符的字符串可以赋值给变量，但是仅仅将单一的转义符赋值给变量是不可行的。
> 
```bash
variable=\
echo "$variable"
# 这样做会报如下错误：
# tesh.sh: : command not found
# 单独的转义符不能够赋值给变量。
# 
#  事实上，"\" 转义了换行，实际效果是：
#+ variable=echo "$variable"
#+ 这是一个非法的赋值方式。
>
variable=\
23skidoo
echo "$variable"        # 23skidoo
                        # 因为第二行是一个合法的赋值，因此不会报错。
>
variable=\ 
#        \^    转义符后有一个空格
echo "$variable"        # 空格
>
variable=\\
echo "$variable"        # \
>
variable=\\\
echo "$variable"
# 这样做会报如下错误：
# tesh.sh: \: command not found
#
#  第一个转义符转转义了第二个，但是第三个转义符仍旧转义的是换行，
#+ 跟开始的那个例子一样，因此会报错。
>
variable=\\\\
echo "$variable"        # \\
                        # 第二个和第四个转义符被转义了，因此可行。
```

转义空格能够避免在命令参数列表中的字符分割问题。

```bash
file_list="/bin/cat /bin/gzip /bin/more /usr/bin/less /usr/bin/emacs-20.7"
# 将一系列文件作为命令的参数。

# 增加两个文件到列表中，并且列出整个表。
ls -l /usr/X11R6/bin/xsetroot /sbin/dump $file_list

echo "-------------------------------------------------------------------------"

# 如果我们转义了这些空格会怎样？
ls -l /usr/X11R6/bin/xsetroot\ /sbin/dump\ $file_list
# 错误：因为转义了两个空格，因此前三个文件被连接成了一个参数传递给了 'ls -l'
```

转义符也提供一种可以撰写多行命令的方式。通常，每一行是一个命令，但是转义换行后命令就可以在下一行继续撰写。

```bash
(cd /source/directory && tar cf - . ) | \
(cd /dest/directory && tar xpvf -)
# 回顾 Alan Cox 的目录树拷贝命令，但是把它拆成了两行。

# 或者你也可以：
tar cf - -C /source/directory . |
tar xpvf - -C /dest/directory
# 可以看下方的注释。
# （感谢 Stéphane Chazelas。）
```

> ![extra](http://tldp.org/LDP/abs/images/note.gif) 在脚本中，如果以 "|" 管道作为一行的结束字符，那么不需要加转义符 \ 也可以写多行命令。但是一个好的编程习惯就是在写多行命令的事后，无论什么情况都要在行尾加上转义符 \。

```bash
echo "foo
bar"
#foo
#bar

echo

echo 'foo
bar'    # 没有区别。
#foo
#bar

echo

echo foo\
bar     # 转义换行。
#foobar

echo

echo "foo\
bar"     # 没有区别，在弱引用中，\ 转义符仍旧转义了换行。
#foobar

echo

echo 'foo\
bar'     # 在强引用中，\ 就按照字面意思来解释了。
#foo\
#bar

# 由 Stéphane Chazelas 提供的例子。
```
