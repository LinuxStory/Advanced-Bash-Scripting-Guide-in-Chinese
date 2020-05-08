# 19 嵌入文档
<blockquote class="blockquote-center">Here and now, boys.
&emsp;&emsp;&emsp;&emsp;--Aldous Huxley, Island</blockquote>

嵌入文档是一段有特殊作用的代码块，它用 [I/O 重定向](http://tldp.org/LDP/abs/html/io-redirection.html#IOREDIRREF) 在交互程序和交互命令中传递和反馈一个命令列表，例如 [ftp](http://tldp.org/LDP/abs/html/communications.html#FTPREF)，[cat](http://tldp.org/LDP/abs/html/basic.html#CATREF) 或者是 ex 文本编辑器

```
COMMAND <<InputComesFromHERE
...
...
...
InputComesFromHERE
```


嵌入文档用限定符作为命令列表的边界，在限定符前需要一个指定的标识符 `<<`，这会将一个程序或命令的标准输入(stdin)进行重定向，它类似 `交互程序 < 命令文件` 的方式，其中命令文件内容如下
```
command #1
command #2
...
```

嵌入文档的格式大致如下
```
interactive-program <<LimitString
command #1
command #2
...
LimitString
```

限定符的选择必须保证特殊以确保不会和命令列表里的内容发生混淆。

注意嵌入文档有时候用作非交互的工具和命令有着非常好的效果，例如 [wall](http://tldp.org/LDP/abs/html/system.html#WALLREF)

样例 19-1. broadcast: 给每个登陆者发送信息
```
#!/bin/bash

wall <<zzz23EndOfMessagezzz23
E-mail your noontime orders for pizza to the system administrator.
    (Add an extra dollar for anchovy or mushroom topping.)
# 额外的信息文本.
# 注意: 'wall' 会打印注释行.
zzz23EndOfMessagezzz23

# 更有效的做法是通过
#         wall < 信息文本
#  然而, 在脚本里嵌入信息模板不乏是一种迅速而又随性的解决方式.

exit
```

样例: 19-2. dummyfile：创建一个有两行内容的虚拟文件
```
#!/bin/bash

# 非交互的使用 `vi` 编辑文件.
# 仿照 'sed'.

E_BADARGS=85

if [ -z "$1" ]
then
  echo "Usage: `basename $0` filename"
  exit $E_BADARGS
fi

TARGETFILE=$1

# 插入两行到文件中保存
#--------Begin here document-----------#
vi $TARGETFILE <<x23LimitStringx23
i
This is line 1 of the example file.
This is line 2 of the example file.
^[
ZZ
x23LimitStringx23
#----------End here document-----------#

#  注意 "^" 对 "[" 进行了转义 
#+ 这段起到了和键盘上按下 Control-V <Esc> 相同的效果.

#  Bram Moolenaar 指出这种情况下 'vim' 可能无法正常工作
#+ 因为在与终端交互的过程中可能会出现问题.

exit
```

上述脚本实现了 `ex` 的功能, 而不是 `vi`. 嵌入文档包含了 `ex` 足够通用的命令列表来形成自有的类别, 所以又称之为 `ex` 脚本.
```
#!/bin/bash
#  替换所有的以 ".txt" 后缀结尾的文件的 "Smith" 为 "Jones"

ORIGINAL=Smith
REPLACEMENT=Jones

for word in $(fgrep -l $ORIGINAL *.txt)
do
  # -------------------------------------
  ex $word <<EOF
  :%s/$ORIGINAL/$REPLACEMENT/g
  :wq
EOF
  # :%s is the "ex" substitution command.
  # :wq is write-and-quit.
  # -------------------------------------
done
```

类似的 `ex 脚本` 是 `cat 脚本`.

样例 19-3. 使用 `cat` 的多行信息
```
#!/bin/bash

#  'echo' 可以输出单行信息,
#+  但是如果是输出消息块就有点问题了.
#   'cat' 嵌入文档却能解决这个局限.

cat <<End-of-message
-------------------------------------
This is line 1 of the message.
This is line 2 of the message.
This is line 3 of the message.
This is line 4 of the message.
This is the last line of the message.
-------------------------------------
End-of-message

#  替换上述嵌入文档内的 7 行文本
#+   cat > $Newfile <<End-of-message
#+       ^^^^^^^^^^
#+ 将输出追加到 $Newfile, 而不是标准输出.

exit 0


#--------------------------------------------
# 由于上面的 "exit 0"，下面的代码将不会生效.

# S.C. points out that the following also works.
echo "-------------------------------------
This is line 1 of the message.
This is line 2 of the message.
This is line 3 of the message.
This is line 4 of the message.
This is the last line of the message.
-------------------------------------"
# 然而, 文本可能不包括双引号除非出现了字符串逃逸.
```

`-` 的作用是标记了一个嵌入文档限制符 (<<-LimitString) ，它能抑制输出的行首的 `tab` (非空格). 这在脚本可读性方面可能非常有用.

样例 19-4. 抑制 tab 的多行信息
```
#!/bin/bash
# 和之前的样例一样, 但...

#  嵌入文档内的 '-' ，也就是 <<-
#+ 抑制了文档行首的 'tab',
#+ 但 *不是* 空格.

cat <<-ENDOFMESSAGE
	This is line 1 of the message.
	This is line 2 of the message.
	This is line 3 of the message.
	This is line 4 of the message.
	This is the last line of the message.
ENDOFMESSAGE
# 脚本的输出将左对齐.
# 行首的 tab 将不会输出.

# 上面 5 行的 "信息" 以 tab 开始, 不是空格.
# 空格不会受影响 <<- .

# 注意这个选项对 *内嵌的* tab 没有影响.

exit 0
```

嵌入文档支持参数和命令替换. 因此可以向嵌入文档传递不同的参数,变向的改其输出.

样例 19-5. 可替换参数的嵌入文档
```
#!/bin/bash
# 另一个使用参数替换的 'cat' 嵌入文档.

# 试一试没有命令行参数,   ./scriptname
# 试一试一个命令行参数,   ./scriptname Mortimer
# 试试用一两个单词引用命令行参数,
#                           ./scriptname "Mortimer Jones"

CMDLINEPARAM=1     #  Expect at least command-line parameter.

if [ $# -ge $CMDLINEPARAM ]
then
  NAME=$1          #  If more than one command-line param,
                   #+ then just take the first.
else
  NAME="John Doe"  #  Default, if no command-line parameter.
fi  

RESPONDENT="the author of this fine script"  
  

cat <<Endofmessage

Hello, there, $NAME.
Greetings to you, $NAME, from $RESPONDENT.

# 这个注释在输出时显示 (为什么?).

Endofmessage

# 注意输出了空行.
# 所以可以这样注释.

exit
```

这个包含参数替换的嵌入文档是相当有用的

样例 19-6. 上传文件对到 `Sunsite` 入口目录
```
#!/bin/bash
# upload.sh

#  上传文件对 (Filename.lsm, Filename.tar.gz)
#+ 到 Sunsite/UNC (ibiblio.org) 的入口目录.
#  Filename.tar.gz 是个 tarball.
#  Filename.lsm is 是个描述文件.
#  Sunsite 需要 "lsm" 文件, 否则将会退回给发送者


E_ARGERROR=85

if [ -z "$1" ]
then
  echo "Usage: `basename $0` Filename-to-upload"
  exit $E_ARGERROR
fi  


Filename=`basename $1`           # Strips pathname out of file name.

Server="ibiblio.org"
Directory="/incoming/Linux"
#  脚本里不需要硬编码,
#+ 但最好可以替换命令行参数.

Password="your.e-mail.address"   # Change above to suit.

ftp -n $Server <<End-Of-Session
# -n 禁用自动登录

user anonymous "$Password"       #  If this doesn't work, then try:
                                 #  quote user anonymous "$Password"
binary
bell                             # Ring 'bell' after each file transfer.
cd $Directory
put "$Filename.lsm"
put "$Filename.tar.gz"
bye
End-Of-Session

exit 0
```

在嵌入文档头部引用或转义"限制符"来禁用参数替换.原因是 `引用/转义` 限定符能有效的[转义](http://tldp.org/LDP/abs/html/escapingsection.html#ESCP)  "$", "`", 和 "\" 这些[特殊符号](http://tldp.org/LDP/abs/html/special-chars.html#SCHARLIST), 使他们维持字面上的意思. (感谢 Allen Halsey 指出这点.)

样例 19-7. 禁用参数替换
```
#!/bin/bash
#  A 'cat' here-document, but with parameter substitution disabled.

NAME="John Doe"
RESPONDENT="the author of this fine script"  

cat <<'Endofmessage'

Hello, there, $NAME.
Greetings to you, $NAME, from $RESPONDENT.

Endofmessage

#   当'限制符'引用或转义时不会有参数替换.
#   下面的嵌入文档也有同样的效果
#   cat <<"Endofmessage"
#   cat <<\Endofmessage



#   同样的:

cat <<"SpecialCharTest"

Directory listing would follow
if limit string were not quoted.
`ls -l`

Arithmetic expansion would take place
if limit string were not quoted.
$((5 + 3))

A a single backslash would echo
if limit string were not quoted.
\\

SpecialCharTest


exit
```

生成脚本或者程序代码时可以用禁用参数的方式来输出文本.

样例 19-8. 生成其他脚本的脚本
```
#!/bin/bash
# generate-script.sh
# Based on an idea by Albert Reiner.

OUTFILE=generated.sh         # Name of the file to generate.


# -----------------------------------------------------------
# '嵌入文档涵盖了生成脚本的主体部分.
(
cat <<'EOF'
#!/bin/bash

echo "This is a generated shell script."
#  注意我们现在在一个子 shell 内,
#+ 我们不能访问 "外部" 脚本变量.

echo "Generated file will be named: $OUTFILE"
#  上面这行并不能按照预期的正常工作
#+ 因为参数扩展已被禁用.
#  相反的, 结果是文字输出.

a=7
b=3

let "c = $a * $b"
echo "c = $c"

exit 0
EOF
) > $OUTFILE
# -----------------------------------------------------------

#  在上述的嵌入文档内引用'限制符'防止变量扩展

if [ -f "$OUTFILE" ]
then
  chmod 755 $OUTFILE
  # 生成可执行文件.
else
  echo "Problem in creating file: \"$OUTFILE\""
fi

#  这个方法适用于生成 C, Perl, Python, Makefiles 等等

exit 0
```

可以从嵌入文档的输出设置一个变量的值. 这实际上是种灵活的 [命令替换](http://tldp.org/LDP/abs/html/commandsub.html#COMMANDSUBREF).
```
variable=$(cat <<SETVAR
This variable
runs over multiple lines.
SETVAR
)

echo "$variable"
```

同样的脚本里嵌入文档可以作为函数的输入.

样例 19-9. 嵌入文档和函数
```
#!/bin/bash
# here-function.sh

GetPersonalData ()
{
  read firstname
  read lastname
  read address
  read city 
  read state 
  read zipcode
} # 可以肯定的是这应该是个交互式的函数, 但 . . .


# 作为函数的输入.
GetPersonalData <<RECORD001
Bozo
Bozeman
2726 Nondescript Dr.
Bozeman
MT
21226
RECORD001


echo
echo "$firstname $lastname"
echo "$address"
echo "$city, $state $zipcode"
echo

exit 0
```

可以这样使用: 作为一个虚构的命令接受嵌入文档的输出. 这样实际上就创建了一个 "匿名" 嵌入文档.

样例 19-10. "匿名" 嵌入文档
```
#!/bin/bash

: <<TESTVARIABLES
${HOSTNAME?}${USER?}${MAIL?}  # Print error message if one of the variables not set.
TESTVARIABLES

exit $?
```

- 上面技巧的一种变体允许 "可添加注释" 的代码块.

样例 19-11. 可添加注释的代码块
```
#!/bin/bash
# commentblock.sh

: <<COMMENTBLOCK
echo "This line will not echo."
这些注释没有 "#" 前缀.
则是另一种没有 "#" 前缀的注释方法.

&*@!!++=
上面这行不会产生报错信息,
因为 bash 解释器会忽略它.

COMMENTBLOCK

echo "Exit value of above \"COMMENTBLOCK\" is $?."   # 0
# 没有错误输出.
echo

#  上面的技巧经常用于工作代码的注释用作排错目的
#  这省去了在每一行开头加上 "#" 前缀,
#+ 然后调试完不得不删除每行的前缀的重复工作.
#  注意我们用了 ":", 在这之上，是可选的.

echo "Just before commented-out code block."
#  下面这个在双破折号之间的代码不会被执行.
#  ===================================================================
: <<DEBUGXXX
for file in *
do
 cat "$file"
done
DEBUGXXX
#  ===================================================================
echo "Just after commented-out code block."

exit 0



######################################################################
#  注意, 然而, 如果将变量中包含一个注释的代码块将会引发问题
#  例如:


#/!/bin/bash

  : <<COMMENTBLOCK
  echo "This line will not echo."
  &*@!!++=
  ${foo_bar_bazz?}
  $(rm -rf /tmp/foobar/)
  $(touch my_build_directory/cups/Makefile)
COMMENTBLOCK


$ sh commented-bad.sh
commented-bad.sh: line 3: foo_bar_bazz: parameter null or not set

# 有效的补救办法就是在 49 行的位置加上单引号，变为 'COMMENTBLOCK'.

  : <<'COMMENTBLOCK'

# 感谢 Kurt Pfeifle 指出这一点.
```

- 另一个漂亮的方法使得"自文档化"的脚本成为可能


样例 19-12. 自文档化的脚本
```
#!/bin/bash
# self-document.sh: self-documenting script
# Modification of "colm.sh".

DOC_REQUEST=70

if [ "$1" = "-h"  -o "$1" = "--help" ]     # 请求帮助.
then
  echo; echo "Usage: $0 [directory-name]"; echo
  sed --silent -e '/DOCUMENTATIONXX$/,/^DOCUMENTATIONXX$/p' "$0" |
  sed -e '/DOCUMENTATIONXX$/d'; exit $DOC_REQUEST; fi


: <<DOCUMENTATIONXX
List the statistics of a specified directory in tabular format.
---------------------------------------------------------------
The command-line parameter gives the directory to be listed.
If no directory specified or directory specified cannot be read,
then list the current working directory.

DOCUMENTATIONXX

if [ -z "$1" -o ! -r "$1" ]
then
  directory=.
else
  directory="$1"
fi  

echo "Listing of "$directory":"; echo
(printf "PERMISSIONS LINKS OWNER GROUP SIZE MONTH DAY HH:MM PROG-NAME\n" \
; ls -l "$directory" | sed 1d) | column -t

exit 0
```

使用 [cat script](http://tldp.org/LDP/abs/html/here-docs.html#CATSCRIPTREF) 是另一种可行的方法.
```
DOC_REQUEST=70

if [ "$1" = "-h"  -o "$1" = "--help" ]     # Request help.
then                                       # Use a "cat script" . . .
  cat <<DOCUMENTATIONXX
List the statistics of a specified directory in tabular format.
---------------------------------------------------------------
The command-line parameter gives the directory to be listed.
If no directory specified or directory specified cannot be read,
then list the current working directory.

DOCUMENTATIONXX
exit $DOC_REQUEST
fi
```

> 另请参阅 [样例 A-28](http://tldp.org/LDP/abs/html/contributed-scripts.html#ISSPAMMER2), [样例 A-40](http://tldp.org/LDP/abs/html/contributed-scripts.html#PETALS), [样例 A-41](http://tldp.org/LDP/abs/html/contributed-scripts.html#QKY), and [样例 A-42](http://tldp.org/LDP/abs/html/contributed-scripts.html#NIM) 更多样例请阅读脚本附带的注释文档.


- 嵌入文档创建了临时文件, 但这些文件在打开且不可被其他程序访问后删除.

```
bash$ bash -c 'lsof -a -p $$ -d0' << EOF
> EOF
lsof    1213 bozo    0r   REG    3,5    0 30386 /tmp/t1213-0-sh (deleted)
```

- 某些工具在嵌入文档内部并不能正常运行.

- 在嵌入文档的最后关闭限定符必须在起始的第一个字符的位置开始.行首不能是空格. 限制符后尾随空格同样会导致意想不到的行为.空格可以防止限制符被当做其他用途. [[1]](http://tldp.org/LDP/abs/html/here-docs.html#FTN.AEN17822)

```
#!/bin/bash

echo "----------------------------------------------------------------------"

cat <<LimitString
echo "This is line 1 of the message inside the here document."
echo "This is line 2 of the message inside the here document."
echo "This is the final line of the message inside the here document."
     LimitString
#^^^^限制符的缩进. 出错! 这个脚本将不会如期运行.

echo "----------------------------------------------------------------------"

#  这些评论在嵌入文档范围外并不能输出

echo "Outside the here document."

exit 0

echo "This line had better not echo."  # 紧跟着个 'exit' 命令.
```

- 有些人非常聪明的使用了一个单引号(!)做为限制符. 但这并不是个好主意

```
# 这个可以运行.
cat <<!
Hello!
! Three more exclamations !!!
!


# 但是 . . .
cat <<!
Hello!
Single exclamation point follows!
!
!
# Crashes with an error message.


# 然而, 下面这样也能运行.
cat <<EOF
Hello!
Single exclamation point follows!
!
EOF
# 使用多字符限制符更为安全.
```

为嵌入文档设置这些任务有些复杂, 可以考虑使用 `expect`, 一种专门用来和程序进行交互的脚本语言。

**Notes:**
&emsp;&emsp;除此之外, Dennis Benzinger 指出,  [使用 <<- 抑制 tab.](http://tldp.org/LDP/abs/html/here-docs.html#LIMITSTRDASH)
