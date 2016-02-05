# 第34章 陷阱
> Turandot: Gli enigmi sono tre, la morte una!
> Caleph: No, no! Gli enigmi sono tre, una la vita!

> ——Puccini

### 以下的做法（非推荐）将让你原本平淡无奇的生活激动不已。
- 将保留字或特殊字符声明为变量名。

```bash
case=value0       # 引发错误。
23skidoo=value1   # 也会引发错误。
# 以数字开头的变量名是被shell保留使用的。
# 试试_23skidoo=value1。以下划线开头的变量名就没问题.

# 然而 . . .   只用一个下划线作为变量名就不行。
_=25
echo $_           # $_是一个特殊变量, 代表最后一个命令的最后一个参数。
# 但是，_是一个有效的函数名！

xyz((!*=value2    # 引起严重的错误。
# Bash3.0之后，标点不能出现在变量名中。
```

- 使用连字符或其他保留字符来做变量名（或函数名）。

```bash
var-1=23
# 用 'var_1 代替。

function-whatever ()   # 错误
# 用 ‘function_whatever ()’ 代替。

 
# Bash3.0之后，标点不能出现在函数名中。
function.whatever ()   # 错误
# 用 ‘functionWhatever ()’ 代替。
```

- 让变量名与函数名相同。 这会使得脚本的可读性变得很差。

```bash
do_something ()
{
  echo "This function does something with \"$1\"."
}

do_something=do_something

do_something do_something

# 这么做是合法的，但会让人混淆。
```

- 不合时宜的使用[空白符][1]。与其他编程语言相比，Bash非常讲究空白符的使用。

```bash
var1 = 23   # ‘var1=23’才是正确的。
# 对于上边这一行来说，Bash会把“var1”当作命令来执行，
# “=”和“23”会被看作“命令”“var1”的参数。
	
let c = $a - $b   # ‘let c=$a-$b’或‘let "c = $a - $b"’才是正确的。

if [ $a -le 5]    # if [ $a -le 5 ]   是正确的。
#           ^^      if [ "$a" -le 5 ]   这么写更好。
                  # [[ $a -le 5 ]] 也行。
```

- 在[大括号包含的代码块][2]中，最后一条命令没有以[分号][3]结尾。

```bash
{ ls -l; df; echo "Done." }
# bash: syntax error: unexpected end of file

{ ls -l; df; echo "Done."; }
#                        ^     ### 最后的这条命令必须以分号结尾。
```

- 假定未被初始化的变量（赋值前的变量）被“清0”。事实上，未初始化的变量值为“null”，而不是0。

```bash
#!/bin/bash

echo "uninitialized_var = $uninitialized_var"
# uninitialized_var =

# 但是 . . .
# if $BASH_VERSION ≥ 4.2; then

if [[ ! -v uninitialized_var ]]
then
  uninitialized_var=0   # Initialize it to zero!
fi


```

- 混淆测试符号=和-ep。请记住，=用于比较字符变量，而-ep用来比较整数。

```bash
if [ "$a" = 273 ]      # $a是整数还是字符串？
if [ "$a" -eq 273 ]    # $a为整数。

# 有些情况下，即使你混用-ep和=，也不会产生错误的结果。
# 然而 . . .


a=273.0   # 不是一个整数。
	   
if [ "$a" = 273 ]
then
  echo "Comparison works."
else  
  echo "Comparison does not work."
fi    # Comparison does not work.

# 与a=" 273"和a="0273"相同。


# 类似的， 如果对非整数值使用“-ep”的话，就会产生问题。
	   
if [ "$a" -eq 273.0 ]
then
  echo "a = $a"
fi  # 产生了错误消息而退出。
# test.sh: [: 273.0: integer expression expected
```

- 误用了[字符串比较][4]操作符。

样例 34-1. 数字比较与字符串比较并不相同

```bash
#!/bin/bash
# bad-op.sh: 尝试一下对整数使用字符串比较。

echo
number=1

#  下面的"while循环"有两个过错误:
#+ 一个比较明显，而另一个比较隐蔽。

while [ "$number" < 5 ]    # 错！应该是:  while [ "$number" -lt 5 ]
do
  echo -n "$number "
  let "number += 1"
done  
#  如果试图运行这个错误的脚本，就会得到一个错误信息:
#+ bad-op.sh: line 10: 5: No such file or directory
#  在单中括号结构（[ ]）中，"<"必须被转义，
#+ 即便如此，比较两个整数仍是错误的。

echo "---------------------"

while [ "$number" \< 5 ]    #  1 2 3 4
do                          #
  echo -n "$number "        #  看起来好像可以工作，但是 . . .
  let "number += 1"         #+ 事实上是比较ASCII码，
  done                      #+ 而不是整数比较。

echo; echo "---------------------"

# 这么做会产生问题。比如:

lesser=5
greater=105

if [ "$greater" \< "$lesser" ]
then
  echo "$greater is less than $lesser"
fi                          # 105 is less than 5
#  事实上，在字符串比较中（按照ASCII码的顺序）
#+ "105"小于"5"。

echo

exit 0
```

- 试图用[let][5]来设置字符串变量。

```bash
let "a = hello, you"
echo "$a"   # 0
```

- 有时候在“test”中括号（[ ]）结构里的变量需要被引用起来（双引号）。如果不这么做的话，可能会引起不可预料的结果。请参考[例子 7-6][6]，[例子 16-5][7]，[例子 9-6][8]。

- [为防分隔][9]，用双引号引用一个包含空白符的变量。 有些情况下，这会产生[意想不到的后果][10]。

- 脚本中的命令可能会因为脚本宿主不具备相应的运行权限而导致运行失败。如果用户在命令行中不能调用这个命令的话，那么即使把它放到脚本中来运行，也还是会失败。这时可以通过修改命令的属性来解决这个问题，有时候甚至要给它设置suid位(当然, 要以root身份来设置)。

- 试图使用-作为作为重定向操作符（事实上它不是），通常都会导致令人不快的结果。

```bash
command1 2> - | command2
# 试图将command1的错误输出重定向到一个管道中 . . .
# . . . 不会工作。

command1 2>& - | command2  # 也没效果。

感谢，S.C。
```

- 使用[Bash 2.0或更高版本][11]的功能，可以在产生错误信息的时候，引发修复动作。但是比较老的Linux机器默认安装的可能是Bash 1.XX。

```bash
#!/bin/bash

minimum_version=2
# 因为Chet Ramey经常给Bash添加一些新的特征，
# 所以你最好将$minimum_version设置为2.XX，3.XX，或是其他你认为比较合适的值。
E_BAD_VERSION=80

if [ "$BASH_VERSION" \< "$minimum_version" ]
then
  echo "This script works only with Bash, version $minimum or greater."
  echo "Upgrade strongly recommended."
  exit $E_BAD_VERSION
fi

...
```

- 在非Linux机器上的[Bourne shell][12]脚本( **#!/bin/sh** )中使用Bash特有的功能，[可能会引起不可预料的行为][13]。Linux系统通常都会把**bash**别名化为**sh**，但是在一般的UNIX机器上却不一定会这么做。

- 使用Bash未文档化的特征，将是一种危险的举动。本书之前的几个版本就依赖一个这种“特征”，下面说明一下这个“特征”，虽然[exit][14]或[return][15]所能返回的最大正值为255，但是并没有限制我们使用负整数。不幸的是, Bash 2.05b之后的版本，这个漏洞消失了。请参考[例子 24-9][16]。

- 在某些情况下，会返回一个误导性的[退出状态][17]。[设置一个函数内的局部变量][18]或[分配一个算术值给一个变量][19]时，就有可能发生这种情况。

- [算术表达式的退出状态][20]不等同于一个错误代码。

```bash
var=1 && ((--var)) && echo $var
#        ^^^^^^^^^ 在这里，这个与列表返回错误代码1而终止。
#                     不会打印$var的值！
echo $?   # 1
```

- 一个带有DOS风格换行符(\r\n)的脚本将会运行失败，因为**#!/bin/bash\r\n**是不合法的，与我们所期望的**#!/bin/bash\n**不同，解决办法就是将这个脚本转换为UNIX风格的换行符。

```bash
#!/bin/bash

echo "Here"

unix2dos $0    # 脚本先将自己改为DOS格式。
chmod 755 $0   # 更改可执行权限。
               # 'unix2dos'会删除可执行权限

./$0           # 脚本尝试再次运行自己。
               # 但它作为一个DOS文件，已经不能运行了。

echo "There"

exit 0
```

- 以**#!/bin/sh**开头的Bash脚本，不能在完整的Bash兼容模式下运行。某些Bash特定的功能可能会被禁用。如果脚本需要完整的访问所有Bash专有扩展，那么它需要使用**#!/bin/bash**作为开头。

- 如果在[here document][21]中，[结尾的limit string之前加上空白字符][22]的话，将会导致脚本的异常行为。

- 在一个[输出被捕获][23]的函数中放置了不止一个echo语句。

```bash
add2 ()
{
  echo "Whatever ... "   # 删掉zhehan
  let "retval = $1 + $2"
    echo $retval
    }

    num1=12
    num2=43
    echo "Sum of $num1 and $num2 = $(add2 $num1 $num2)"

#   Sum of 12 and 43 = Whatever ... 
#   55

#        这些echo连在一起了。
```
这是[行不通][24]的。

- 脚本不能将变量export到它的[父进程][25](即调用这个脚本的shell)，或父进程的环境中。就好比我们在生物学中所学到的那样，子进程只会继承父进程, 反过来则不行。

```bash
WHATEVER=/home/bozo
export WHATEVER
exit 0
```
```bash
bash$ echo $WHATEVER
bash$
```

- 可以确定的是，即使回到命令行提示符，变量$WHATEVER仍然没有被设置。

- 在[子shell][26]中设置和操作变量之后，如果尝试在子shell作用域之外使用同名变量的话, 将会产生令人不快的结果。

样例 34-2. 子shell缺陷

```bash
#!/bin/bash
# 子shell中的变量缺陷。

outer_variable=outer
echo
echo "outer_variable = $outer_variable"
echo

(
# 开始子shell

echo "outer_variable inside subshell = $outer_variable"
inner_variable=inner  # Set
echo "inner_variable inside subshell = $inner_variable"
outer_variable=inner  # 会修改全局变量吗？
echo "outer_variable inside subshell = $outer_variable"

# 如果将变量‘导出’会产生不同的结果么？
#    export inner_variable
#    export outer_variable
# 试试看。

# 结束子shell
)

echo
echo "inner_variable outside subshell = $inner_variable"  # Unset.
echo "outer_variable outside subshell = $outer_variable"  # Unchanged.
echo

exit 0

# 如果你去掉第19和第20行的注释会怎样？
# 会产生不同的结果吗？
```

- 将echo的输出通过[管道][27]传递给[read][28]命令可能会产生不可预料的结果。在这种情况下，read命令的行为就好像它在子shell中运行一样。可以使用[set][29]命令来代替(就好像[例子15-18][30]一样)。

样例 34-3. 将echo的输出通过管道传递给read命令

```bash
#!/bin/bash
#  badread.sh:
#  尝试使用'echo'和'read'命令
#+ 非交互的给变量赋值。

#   shopt -s lastpipe

a=aaa
b=bbb
c=ccc

echo "one two three" | read a b c
# 尝试重新给变量a，b，和c赋值。

echo
echo "a = $a"  # a = aaa
echo "b = $b"  # b = bbb
echo "c = $c"  # c = ccc
# 重新赋值失败。

### 但如果 . . .
##  去掉第6行的注释:
#   shopt -s lastpipe
##+ 就能解决这个问题！
### 这是Bash 4.2版本的新特性。

# ------------------------------

# 试试下边这种方法。

var=`echo "one two three"`
set -- $var
a=$1; b=$2; c=$3

echo "-------"
echo "a = $a"  # a = one
echo "b = $b"  # b = two
echo "c = $c"  # c = three 
# 重新赋值成功。

# ------------------------------

#  也请注意，echo到'read'的值只会在子shell中起作用。
#  所以，变量的值*只*会在子shell中被修改。

a=aaa          # 重新开始。
b=bbb
c=ccc

echo; echo
echo "one two three" | ( read a b c;
echo "Inside subshell: "; echo "a = $a"; echo "b = $b"; echo "c = $c" )
# a = one
# b = two
# c = three
echo "-----------------"
echo "Outside subshell: "
echo "a = $a"  # a = aaa
echo "b = $b"  # b = bbb
echo "c = $c"  # c = ccc
echo

exit 0
```

事实上，也正如Anthony Richardson指出的那样，通过管道将输出传递到任何循环中, 都会引起类似的问题。

```bash
# 循环的管道问题。
#  这个例子由Anthony Richardson编写，
#+ 由Wilbert Berendsen补遗。


foundone=false
find $HOME -type f -atime +30 -size 100k |
while true
do
   read f
   echo "$f is over 100KB and has not been accessed in over 30 days"
   echo "Consider moving the file to archives."
   foundone=true
   # ------------------------------------
     echo "Subshell level = $BASH_SUBSHELL"
   # Subshell level = 1
   # 没错, 现在是在子shell中运行。
   # ------------------------------------
done
   
#  变量foundone在这里肯定是false，
#+ 因为它是在子shell中被设置为true的。
if [ $foundone = false ]
then
   echo "No files need archiving."
fi

# =====================现在，下边是正确的方法:=================

foundone=false
for f in $(find $HOME -type f -atime +30 -size 100k)  # 这里没使用管道。
do
   echo "$f is over 100KB and has not been accessed in over 30 days"
   echo "Consider moving the file to archives."
   foundone=true
done
   
if [ $foundone = false ]
then
   echo "No files need archiving."
fi

# ==================这里是另一种方法==================

#  将脚本中读取变量的部分放到一个代码块中，
#+ 这样一来，它们就能在相同的子shell中共享了。
#  感谢，W.B。

find $HOME -type f -atime +30 -size 100k | {
     foundone=false
     while read f
     do
       echo "$f is over 100KB and has not been accessed in over 30 days"
       echo "Consider moving the file to archives."
       foundone=true
     done

     if ! $foundone
     then
       echo "No files need archiving."
     fi
}
```

- 一个相关的问题：当你尝试将tail -f的stdout通过管道传递给[grep][31]时，会产生问题。

```bash
tail -f /var/log/messages | grep "$ERROR_MSG" >> error.log
#  “error.log”文件将不会写入任何东西。
#  正如Samuli Kaipiainen指出的那样，
#+ 这一结果是从grep的缓冲区输出的。
#  解决的办法就是把“--line-buffered”参数添加到grep中。
```

- 在脚本中使用“suid”命令是非常危险的，因为这会危及系统安全。[^suid]

- 使用shell脚本来编写CGI程序是值得商榷的。因为Shell脚本的变量不是“类型安全”的，当CGI被关联的时候，可能会产生令人不快的行为。此外，它还很难抵挡住“破解的考验”。

- Bash不能正确地处理[双斜线(//)字符串][32]。

- 在Linux或BSD上编写的Bash脚本，可能需要修改一下，才能使它们运行在商业的UNIX机器上。这些脚本通常都使用GNU命令和过滤工具，GNU工具通常都比一般的UNIX上的同类工具更加强大。这方面的一个非常明显的例子就是，文本处理工具[tr][33]。

- 遗憾的是，更新Bash本身就会破坏[过去工作完全正常][34]的脚本。让我们回顾一下[使用无正式文件的Bash功能有多危险][35]。

> 危险正在接近你 --
小心，小心，小心，小心。
许多勇敢的心都在沉睡。
所以一定要小心 --
> 小心。

> ——A.J. Lamb and H.W. Petrie
[1]: http://tldp.org/LDP/abs/html/special-chars.html#WHITESPACEREF
[2]: http://tldp.org/LDP/abs/html/special-chars.html#CODEBLOCKREF
[3]: http://tldp.org/LDP/abs/html/special-chars.html#SEMICOLONREF
[4]: http://tldp.org/LDP/abs/html/comparison-ops.html#SCOMPARISON1
[5]: http://tldp.org/LDP/abs/html/internal.html#LETREF
[6]: http://tldp.org/LDP/abs/html/comparison-ops.html#STRTEST
[7]: http://tldp.org/LDP/abs/html/redircb.html#REDIR2
[8]: http://tldp.org/LDP/abs/html/internalvariables.html#ARGLIST
[9]: http://tldp.org/LDP/abs/html/quotingvar.html#WSQUO
[10]: http://tldp.org/LDP/abs/html/quotingvar.html#VARSPLITTING
[11]: http://tldp.org/LDP/abs/html/bashver2.html#BASH2REF
[12]: http://tldp.org/LDP/abs/html/why-shell.html#BASHDEF
[13]: http://tldp.org/LDP/abs/html/gotchas.html#BINSH
[14]: http://tldp.org/LDP/abs/html/exit-status.html#EXITSTATUSREF
[15]: http://tldp.org/LDP/abs/html/complexfunct.html#RETURNREF
[16]: http://tldp.org/LDP/abs/html/complexfunct.html#RETURNTEST
[17]: http://tldp.org/LDP/abs/html/exit-status.html#EXITSTATUSREF
[18]: http://tldp.org/LDP/abs/html/localvar.html#EXITVALANOMALY01
[19]: http://tldp.org/LDP/abs/html/internal.html#EXITVALANOMALY02
[20]: http://tldp.org/LDP/abs/html/testconstructs.html#ARXS
[21]: http://tldp.org/LDP/abs/html/here-docs.html#HEREDOCREF
[22]: http://tldp.org/LDP/abs/html/here-docs.html#INDENTEDLS
[23]: http://tldp.org/LDP/abs/html/assortedtips.html#RVT
[24]: http://tldp.org/LDP/abs/html/assortedtips.html#RVTCAUTION
[25]: http://tldp.org/LDP/abs/html/internal.html#FORKREF
[26]: http://tldp.org/LDP/abs/html/subshells.html#SUBSHELLSREF
[27]: http://tldp.org/LDP/abs/html/special-chars.html#PIPEREF
[28]: http://tldp.org/LDP/abs/html/internal.html#READREF
[29]: http://tldp.org/LDP/abs/html/internal.html#SETREF
[30]: http://tldp.org/LDP/abs/html/internal.html#SETPOS
[31]: http://tldp.org/LDP/abs/html/textproc.html#GREPREF
[32]: http://tldp.org/LDP/abs/html/internal.html#DOUBLESLASHREF
[33]: http://tldp.org/LDP/abs/html/textproc.html#TRREF
[34]: http://tldp.org/LDP/abs/html/string-manipulation.html#PARAGRAPHSPACE
[35]: http://tldp.org/LDP/abs/html/gotchas.html#UNDOCF
[36]: http://tldp.org/LDP/abs/html/fto.html#SUIDREF

#### 注意事项

[^suid]: 在Linux和绝大多数的UNIX机器上，给脚本设置[suid][36]权限是没用的。