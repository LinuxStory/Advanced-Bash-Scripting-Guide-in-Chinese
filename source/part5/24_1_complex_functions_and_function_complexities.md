# 24.1 复杂函数和函数复杂性
函数可以处理传递给它的参数，并且能返回它的[退出状态码](http://tldp.org/LDP/abs/html/exit-status.html#EXITSTATUSREF)给脚本，以便后续处理。
```
function_name $arg1 $arg2
```

函数通过位置来引用传递过来的参数（就好像它们是[位置参数](http://tldp.org/LDP/abs/html/internalvariables.html#POSPARAMREF)），例如，$1, $2，等等。

例子 24-2. 带参数的函数
```
#!/bin/bash
# 函数和参数

DEFAULT=default                 # 默认参数值。D

func2 () {
    if [ -z "$1" ]              # 第一个参数长度是否为零？
    then
        echo "-Parameter #1 is zero length.-"  # 或者没有参数传递进来。
    else
        echo "-Parameter #1 is \"$1\".-"
    fi

    variable=${1-$DEFAULT}
    echo "variable = $variable"     #  这里的参数替换
                                    #+ 表示什么？
                                    #  ---------------------------
                                    #  为了区分没有参数的情况
                                    #+ 和只有一个null参数的情况。

    if [ "$2" ]
    then
        echo "-Parameter #2 is \"$2\".-"
    fi

    return 0
}

echo

echo "Nothing passed."
func2                          # 不带参数调用
echo


echo "Zero-length parameter passed."
func2 ""                        # 使用0长度的参数进行调用
echo

echo "Null parameter passed."
func2 "$uninitialized_param"    # 使用未初始化的参数进行调用
echo


echo "One parameter passed."
func2 first           # 带一个参数的调用
echo

echo "Two parameters passed."
func2 first second    # 带两个参数的调用
echo

echo "\"\" \"second\" passed."
func2 "" second       # 第一个调用参数为0长度参数，
echo                  # 第二个是ASCII码的字符串参数。

exit 0
```
![notice](http://tldp.org/LDP/abs/images/caution.gif) 也可以使用[shift](http://tldp.org/LDP/abs/html/othertypesv.html#SHIFTREF)命令来处理传递给函数的参数（请参考[例子 33-18](http://tldp.org/LDP/abs/html/assortedtips.html#MULTIPLICATION).
但是，传递给脚本的命令行参数怎么办？在函数内部，可以看见这些命令行参数么？好，现在让我们弄清楚这个困惑。

例子 34-3. 函数以及传递给脚本的命令行参数。
```
#!/bin/bash
# func-cmdlinearg.sh
#  带一个命令行参数来执行这个脚本，
#+ 类似于 $0 arg1.


func ()
{
    echo "$1"   # 显示传递给这个函数的第一个参数。
}               # 命令行参数可以么？

echo "First call to function: no arg passed."
echo "See if command-line arg is seen."
func
# 不! 没有见到命令行参数.

echo "============================================================"
echo
echo "Second call to function: command-line arg passed explicitly."

func $1
# 现在，见到命令行参数了！

exit 0
```
和其它的编程语言相比，shell脚本一般只会传值给函数。如果把变量名（事实上就是指针）作为参数传递给函数的话，那将被解释为字面含义，也就是被看做字符串。 函数只会以字面含义来解释函数参数。

[变量的间接引用](http://tldp.org/LDP/abs/html/ivr.html#IVRREF)（请参考[例子 37-2](http://tldp.org/LDP/abs/html/bashver2.html#EX78)）提供了一种笨拙的机制，来将变量指针传递给函数。

例子 24-4. 将一个间接引用传递给函数
```
#!/bin/bash
# ind-func.sh: 将一个间接引用传递给函数。

echo_var ()
{
    echo "$1"
}

message=Hello
Hello=Goodbye

echo_var "$message"         # Hello
# 现在，让我们传递一个间接引用给函数。
echo_var "${!message}"     # Goodbye
echo "-------------"

# 如果我们改变“hello”的值会发生什么？
Hello="Hello, again!"
echo_var "$message"        # Hello
echo_var "${!message}"     # Hello, again!

exit 0
```
接下来的一个逻辑问题就是，将参数传递给函数之后，参数能否被解除引用。

例子 24-5. 对一个传递给函数的参数进行解除引用的操作
```
#!/bin/bash
# dereference.sh
# 对一个传递给函数的参数进行解除引用的操作。
# 此脚本由Bruce W. Clare编写.

dereference ()
{
    y=\$"$1"   # 变量名（而不是值）.
    echo $y    # $Junk

    x=`eval "expr \"$y\" "`
    echo $1=$x
    eval "$1=\"Some Different Text \""  # 赋新值.
}

Junk="Some Text"
echo $Junk "before"         # Some Text before

dereference Junk
echo $Junk "after"          # Some Different Text after

exit 0
```

例子 24-6. 再来一次，对一个传递给函数的参数进行解除引用的操作
```
#!/bin/bash
# ref-params.sh: 解除传递给函数的参数引用。
#                (复杂的例子C)

ITERATIONS=3  # 取得输入的次数。
icount=1

my_read () {
    #  用my_read varname这种形式来调用,
    #+ 将之前用括号括起的值作为默认值输出,
    #+ 然后要求输入一个新值.

    local local_var

    echo -n "Enter a value "
    eval 'echo -n "[$'$1'] "'  #  之前的值.
# eval echo -n "[\$$1] "     #  更容易理解,
                             #+ 但会丢失用户在尾部输入的空格。
    read local_var
    [ -n "$local_var" ] && eval $1=\$local_var

    # "与列表": 如果 "local_var" 的测试结果为true，则把变量"$1"的值赋给它。
}

echo

while [ "$icount" -le "$ITERATIONS" ]
do
    my_read var
    echo "Entry #$icount = $var"
    let "icount += 1"
    echo
done

# 感谢Stephane Chazelas 提供这个例子。

exit 0
```

## 退出与返回码

### 退出状态码

函数返回一个值，被称为退出状态码。这和一条命令返回的[退出状态码](http://tldp.org/LDP/abs/html/exit-status.html#EXITSTATUSREF)类似。退出状态码可以由**return** 命令明确指定，也可以由函数中最后一条命令的退出状态码来指定（如果成功，则返回0，否则返回非0值）。可以在脚本中使用[$?](http://tldp.org/LDP/abs/html/internalvariables.html#XSTATVARREF)来引用[退出状态码](http://tldp.org/LDP/abs/html/exit-status.html#EXITSTATUSREF)。 因为有了这种机制，所以脚本函数也可以像C函数一样有“返回值”。

### return

终止一个函数。一个return命令[1](http://tldp.org/LDP/abs/html/complexfunct.html#FTN.AEN18474) 可选的允许带一个整形参数，这个整形参数将作为函数的“退出状态码”返回给调用这个函数的脚本，并且这个退出状态码也被赋值给变量[$?](http://tldp.org/LDP/abs/html/internalvariables.html#XSTATVARREF).

例子 24-7. 取两个数中的最大值
```
#!/bin/bash
# max.sh: 取两个Maximum of two integers.
E_PARAM_ERR=250    # 如果传给函数的参数少于两个时，就返回这个值。
EQUAL=251          # 如果两个参数相等时，就返回这个值。
#  任意超出范围的
#+ 参数值都可能传递到函数中。

max2 ()             # 返回两个数中的最大值。
{                   # 注意：参与比较的数必须小于250.
    if [ -z "$2" ]
    then
        return $E_PARAM_ERR
    fi

    if [ "$1" -eq "$2" ]
    then
        return $EQUAL
    else
        if [ "$1" -gt "$2" ]
        then
            return $1
        else
            return $2
        fi
    fi
}

max2 33 34
return_val=$?

if [ "$return_val" -eq $E_PARAM_ERR ]
then
    echo "Need to pass two parameters to the function."
elif [ "$return_val" -eq $EQUAL ]
then
    echo "The two numbers are equal."
else
    echo "The larger of the two numbers is $return_val."
fi

exit 0
#  练习 (easy):
￼#  ---------------
#  把这个脚本转化为交互脚本，
#+ 也就是，修改这个脚本，让其要求调用者输入2个数。
```

![info](http://tldp.org/LDP/abs/images/tip.gif) 为了让函数可以返回字符串或者是数组，可以使用一个在函数外可见的专用全局变量。

```
count_lines_in_etc_passwd()
{
    [[ -r /etc/passwd ]] && REPLY=$(echo $(wc -l < /etc/passwd))
    #  如果 /etc/passwd 可读，让 REPLY 等于 文件的行数.
    #  这样就可以同时返回参数值与状态信息。
    #  'echo' 看上去没什么用，可是...
    #+ 它的作用是删除输出中的多余空白符。
}

if count_lines_in_etc_passwd
then
    echo "There are $REPLY lines in /etc/passwd."
else
    echo "Cannot count lines in /etc/passwd."
fi

# 感谢, S.C.
```

例子 24-8. 将阿拉伯数字转化为罗马数字
```
#!/bin/bash
# 将阿拉伯数字转化为罗马数字。
# 范围：0 - 200
# 比较粗糙，但可以正常工作。
# 扩展范围, 并且完善这个脚本, 作为练习.
# 用法: roman number-to-convert

LIMIT=200
E_ARG_ERR=65
E_OUT_OF_RANGE=66

if [ -z "$1" ]
then
    echo "Usage: `basename $0` number-to-convert"
exit $E_ARG_ERR
fi

num=$1
if [ "$num" -gt $LIMIT ]
then
    echo "Out of range!"
    exit $E_OUT_OF_RANGE
fi

to_roman ()             # 在第一次调用函数前必须先定义它。
{
    number=$1
    factor=$2
    rchar=$3
    let "remainder = number - factor"
    while [ "$remainder" -ge 0 ]
    do
        echo -n $rchar
        let "number -= factor"
        let "remainder = number - factor"
    done

    return $number
        # 练习:
        # ---------
        # 1) 解释这个函数如何工作
        #    提示: 依靠不断的除，来分割数字。
        # 2) 扩展函数的范围：
        #    提示: 使用echo和substitution命令.
}

to_roman $num 100 C
num=$?
to_roman $num 90 LXXXX
num=$?
to_roman $num 50 L
num=$?
to_roman $num 40 XL
num=$?
to_roman $num 10 X
num=$?
to_roman $num 9 IX
num=$?
to_roman $num 5 V
num=$?
to_roman $num 4 IV
num=$?
to_roman $num 1 I
# 成功调用了转换函数。
# 这真的是必须的么？ 这个可以简化么？

echo

exit 
```
也可以参见[例子 11-29](http://tldp.org/LDP/abs/html/testbranch.html#ISALPHA)

![notice](http://tldp.org/LDP/abs/images/caution.gif) 函数所能返回最大的正整数是255. return命令和[退出状态码](http://tldp.org/LDP/abs/html/exit-status.html#EXITSTATUSREF)的概念紧密联系在一起，并且退出状态码的值受此限制。 幸运的是，如果想让函数返回大整数的话，有好多种不同的[变通方法](http://tldp.org/LDP/abs/html/assortedtips.html#RVT) 能够应对这个情况。

例子24-9. 测试函数最大的返回值
```
#!/bin/bash
# return-test.sh
# 函数所能返回的最大正整数为255.

return_test ()         # 传给函数什么值，就返回什么值。
{
    return $1
}

return_test 27          # o.k.
echo $?                 # 返回27.

return_test 255         # Still o.k.
echo $?                 # 返回 255.

return_test 257         # 错误!
echo $?                 # 返回 1 (对应各种错误的返回码).

# =========================================================
return_test -151896    # 能返回一个大负数么?
echo $?                # 能否返回 -151896?
                        # 不行! 返回的是 168.

#  Bash 2.05b 之前的版本
#+ 允许返回大负数。
#  这可能是个有用的特性。
#  Bash之后的新版本修正了这个漏洞。
#  这可能会影响以前所编写的脚本。
#  一定要小心!
# =========================================================

exit 0
```

如果你想获得大整数“返回值”的话，简单的方法就是将“要返回的值”保存到一个全局变量中。
```
Return_Val=   # 用于保存函数特大返回值的全局变量。

alt_return_test ()
{
    fvar=$1
    Return_Val=$fvar
    return   # 返回 0 (成功).
}

alt_return_test 1
echo $?             #0
echo "return value = $Return_Val"            #1

alt_return_test 256
echo "return value = $Return_Val"           # 256

alt_return_test 257
echo "return value = $Return_Val"           # 257

alt_return_test 25701
echo "return value = $Return_Val"           #25701
```

一种更优雅的做法是在函数中使用echo命令将"返回值输出到stdout“，然后用[命令替换](http://tldp.org/LDP/abs/html/commandsub.html#COMMANDSUBREF)来捕捉此值。请参考[36.7小节](http://tldp.org/LDP/abs/html/assortedtips.html) 中[关于这种用法的讨论](http://tldp.org/LDP/abs/html/assortedtips.html#RVT)。

例子 24-10. 比较两个大整数

```
#!/bin/bash
# max2.sh: 取两个大整数中的最大值。

#  这是前一个例子 "max.sh" 的修改版,
#+ 这个版本可以比较两个大整数。

EQUAL=0             # 如果两个值相等，那就返回这个值。
E_PARAM_ERR=-99999  # 没有足够多的参数，那就返回这个值。
#           ^^^^^^    任意超出范围的参数都可以传递进来。

max2 ()             # "返回" 两个整数中最大的那个。
{
    if [ -z "$2" ]
    then
        echo $E_PARAM_ERR
        return
    fi

    if [ "$1" -eq "$2" ]
    then
        echo $EQUAL
        return
    else
        if [ "$1" -gt "$2" ]
        then
            retval=$1
        else
            retval=$2
        fi
    fi

    echo $retval            # 输出 (到 stdout), 而没有用返回值。
                            # 为什么?
}


return_val=$(max2 33001 33997)
#            ^^^^             函数名
#                 ^^^^^ ^^^^^ 传递进来的参数
#  这其实是命令替换的一种形式：
#+ 可以把函数看作一个命令，
#+ 然后把函数的stdout赋值给变量“return_val".


# ========================= OUTPUT ========================
if [ "$return_val" -eq "$E_PARAM_ERR" ]
then
    echo "Error in parameters passed to comparison function!"
elif [ "$return_val" -eq "$EQUAL" ]
then
    echo "The two numbers are equal."
else
    echo "The larger of the two numbers is $return_val."
fi
# =========================================================

exit 0

#  练习:
#  ---------
#  1) 找到一种更优雅的方法，
#+    去测试传递给函数的参数。
#  2) 简化”输出“段的if/then结构。
#  3) 重写这个脚本，使其能够从命令行参数中获得输入。
```

这是另一个能够捕捉函数”返回值“的例子。要想搞明白这个例子，需要一些[awk](http://tldp.org/LDP/abs/html/awk.html#AWKREF)的知识。

```
month_length ()  # 把月份作为参数。
{                # 返回该月包含的天数。
    monthD="31 28 31 30 31 30 31 31 30 31 30 31"  # 作为局部变量声明？
    echo "$monthD" | awk '{ print $'"${1}"' }'    # 小技巧.
#                             ^^^^^^^^^
# 传递给函数的参数  ($1 -- 月份), 然后传给 awk.
# Awk 把参数解释为"print $1 . . . print $12" (这依赖于月份号)
# 这是一个模板，用于将参数传递给内嵌awk的脚本：
#                                 $'"${script_parameter}"'
#    这里是一个简单的awk结构：
#    echo $monthD | awk -v month=$1 '{print $(month)}'
#    使用awk的-v选项，可以把一个变量值赋给
#+   awk程序块的执行体。
#    感谢 Rich.
#  需要做一些错误检查，来保证月份好正确，在范围（1-12）之间， 
#+ 别忘了检查闰年的二月。
}
# ----------------------------------------------
# 用例：
month=4        # 以四月为例。
days_in=$(month_length $month)
echo $days_in  # 30
# ----------------------------------------------
```
也请参考[例子 A-7](http://tldp.org/LDP/abs/html/contributed-scripts.html#DAYSBETWEEN) 和[例子A-37](http://tldp.org/LDP/abs/html/contributed-scripts.html#STDDEV).

练习:使用目前我们已经学到的知识，来扩展之前的例子 [将阿拉伯数字转化为罗马数字]()，让它能够接受任意大的输入。

重定向
重定向函数的stdin
函数本质上其实就是一个[代码块](http://tldp.org/LDP/abs/html/special-chars.html#CODEBLOCKREF),这就意味着它的stdin可以被重定向（比如[例子3-1](http://tldp.org/LDP/abs/html/special-chars.html#EX8))。

例子 24-11. 从username中取得用户的真名
```
#!/bin/bash
# realname.sh
#
# 依靠username，从/etc/passwd 中获得“真名”.


ARGCOUNT=1       # 需要一个参数.
E_WRONGARGS=85

file=/etc/passwd
pattern=$1

if [ $# -ne "$ARGCOUNT" ]
then
    echo "Usage: `basename $0` USERNAME"
    exit $E_WRONGARGS
fi

file_excerpt ()    # 按照要求的模式来扫描文件,
{                  #+ 然后打印文件的相关部分。
    while read line  # "while" 并不一定非得有 [ 条件 ] 不可。
    do
        echo "$line" | grep $1 | awk -F":" '{ print $5 }'
        # awk用":" 作为界定符。
    done
    } <$file  # 重定向到函数的stdin。

file_excerpt $pattern
# 是的，整个脚本其实可以被缩减为
#       grep PATTERN /etc/passwd | awk -F":" '{ print $5 }'
# or
#       awk -F: '/PATTERN/ {print $5}'
# or
#       awk -F: '($1 == "username") { print $5 }' # 从username中获取真名
# 但是，这些起不到示例的作用。

exit 0
```
还有一个办法，或许能够更好的理解重定向函数的stdin。 它在函数内添加了一对大括号，并且将重定向stdin的行为放在这对添加的大括号上。
```
# 用下面的方法来代替它:
Function ()
{
    ...
} < file

# 试试这个:
Function ()
{
    { 
        ...
    } < file 
}

# 同样的,

Function ()  # 没问题.
{
    {
        echo $*
    } | tr a b 
}

Function ()  # 不行.
{
    echo $*
} | tr a b   # 这儿的内嵌代码块是被强制的。
# 感谢, S.C.
```
![extra](http://tldp.org/LDP/abs/images/note.gif) Emmanuel Rouat的 [sample bash](http://tldp.org/LDP/abs/html/sample-bashrc.html) 文件包含了一些很有指导性意义的函数例子。




















