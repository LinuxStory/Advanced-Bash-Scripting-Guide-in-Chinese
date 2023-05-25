# 24 函数
### 本章目录

- [24.1 复杂函数和函数复杂性](24_1_complex_functions_and_function_complexities.md)
- [24.2 局部变量](24_2_local_variables.md)
- [24.3 不使用局部变量的递归](24_3_recursion_without_local_variables.md)

和其它“真正”的编程语言一样，Bash也有函数，尽管它在实现方面有一些限制。一个函数就是一个子程序，实现一系列操作的[代码块](http://tldp.org/LDP/abs/html/special-chars.html#CODEBLOCKREF)，执行一个特定任务的“黑盒子”。有重复代码的地方，当一个过程只需要轻微修改任务就会重复执行的时候，那么你就需要考虑使用函数了。
```
function function_name {
command...
}
```
或者
```
function_name () { 
command...
}
```
第二种形式可能会更受C程序员的喜爱（并且它更具有[可移植性](http://tldp.org/LDP/abs/html/portabilityissues.html)）。
在C语言里面，函数的左大括号可以出现在第二行。
```
function_name ()
{
command...
}
```
![extra](http://tldp.org/LDP/abs/images/note.gif) 一个函数可能被“压缩”到一个单独行里。
```
￼fun () { echo "This is a function"; echo; } 
#                                 ^     ^
```
然而，在这种情况下，函数里的最后一个命令必须跟有一个分号。

```
fun () { echo "This is a function"; echo } # Error! 
#                                       ^
fun2 () { echo "Even a single-command function? Yes!"; } 
#                                                    ^
```
只需要引用函数名字就可以调用或者触发函数。一个函数调用相当于一个命令。

例子 24-1. 简单的函数
```
#!/bin/bash
# ex59.sh: 练习函数(简单的).

JUST_A_SECOND=1

funky ()
{ # 这是一个简单的函数
    echo "This is a funky function."
    echo "Now exiting funky function."
} # 函数必须在调用前声明.


fun ()
{   # 一个稍微复杂点的函数.
    i=0
    REPEATS=30

    echo
    echo "And now the fun really begins."
    echo

    sleep $JUST_A_SECOND    # Hey, 等一秒钟!
    while [ $i -lt $REPEATS ]
    do
        echo "----------FUNCTIONS---------->"
        echo "<------------ARE-------------"
        echo "<------------FUN------------>"
        echo
        let "i+=1"
    done
}

# 现在，调用这些函数.

funky
fun

exit $?
```

函数定义必须在第一次函数调用之前。没有声明函数的方法，比如像C语言中一样。
```
f1
# 将会产生一个错误消息，因为“f1”函数还没有定义。

declare -f f1      # 这样也不会有帮助。
f1                 # 仍然会产生一个错误消息。

# 然而...


f1 () {
    echo "Calling function \"f2\" from within function \"f1\"."
    f2 
}

f2 () {
    echo "Function \"f2\"."
}

f1  #  在此之前，事实上函数“f2”是没有被调用的，
    #+ 尽管在它定义之前被引用了。
    #  这是可以的。
    # 感谢, S.C.
```

![extra](http://tldp.org/LDP/abs/images/note.gif)  函数不能为空！
```
#!/bin/bash
# empty-functionn.sh

empty () 
{
}

exit 0  # 这里将不会退出!


# $ sh empty-function.sh
# empty-function.sh: line 6: syntax error near unexpected token `}'
# empty-function.sh: line 6: `}'

# $ echo $? 
# 2

# 请注意，只包含注释的函数也是空函数。

func () 
{
    # 注释 1.
    # 注释 2.
    # 这仍然是一个空函数。
    # 感谢, Mark Bova将这一点指出来。
}
# 结果会出现和上面一样的错误信息。

# 然而 ...

not_quite_empty ()
{
    illegal_command
} #  一个包含这个函数的脚本将不会出错
    #+ 只要这个函数没有被调用。
not_empty ()
{
    :
} # 包含一个 : (空命令符），这样是可以的。

# 感谢, Dominick Geyer 和 Thiemo Kellner.
```
甚至，把一个函数嵌套在另外一个函数里也是可行的，尽管这并没有什么用。

```
f1 () 
{
    f2 () # 嵌套函数
    {
        echo "Function \"f2\", inside \"f1\"."
    }
}

f2  #  将会产生一个错误消息。
    #  即使有一个前置的 "declare -f f2" 也不会有什么作用。

echo

f1  #  不会做任何事情，因为调用“f1”的时候，并不会自动调用“f2”。
    #  现在，调用“f2”是可以的，
    #+ 因为通过调用“f1”，它的定义现在已是可见的。

    # 感谢, S.C.
```
函数定义可能出现在不太可能出现的地方，甚至出现在本应该是命令出现的地方。

```
ls -l | foo() { echo "foo"; }  # 可行的，尽管没有什么作用。


if [ "$USER" = bozo ]
then
    bozo_greet ()   # 函数定义嵌套在if/then的结构体中。
    {
        echo "Hello, Bozo."
    }
fi

bozo_greet        # 只有Bozo用户工作，其它用户会得到一个错误消息。


# 在某些场景中，像下面这些东西可能会很有用。
NO_EXIT=1   # 将会激活下面的函数定义。

[[ $NO_EXIT -eq 1 ]] && exit() { true; }     # 函数定义出现在“与列表”中。
# 如果 $NO_EXIT 等于 1, 定义 "exit ()".
# 通过把exit函数别名为“true”，这样把内置的exit命令给禁用了。

exit  # 调用 "exit ()" 函数, 而不是内置的 "exit" 命令。


# 或者，类似地:
filename=file1

[ -f "$filename" ] &&
foo () { rm -f "$filename"; echo "File "$filename" deleted."; } ||
foo () { echo "File "$filename" not found."; touch bar; }

foo

# 感谢, S.C. 和 Christopher Head
```
函数名字可以呈现各种奇怪的形式。

```
_(){ for i in {1..10}; do echo -n "$FUNCNAME"; done; echo; }
# ^^^         函数名字和圆括号之间没有空格。
#             这并不会总是会正常工作。为什么呢？

# 现在，我们来调用函数。
_         # __________
#           ^^^^^^^^^^   10 个下划线（10 倍的函数名字）！
# 一个“假”的下划线也是一个可以接受的函数名字。

# 事实上，一个分号也是一个可以接受的函数名字。

:(){ echo ":"; }; :

# 这有什么作用呢？
# 这是一个狡诈的方式去混淆脚本中的代码。
```
也可以参见 [Example A-56](http://tldp.org/LDP/abs/html/contributed-scripts.html#GRONSFELD)

小提示：当一个函数的不同版本出现在一个脚本中，会发生什么事情呢？
```
#  正如Yan Chen 指出的那样,
#  当一个函数被多次定义的时候，
#  最后一个函数是被调用的那个。
#  然而这并不是特别有用。

func () 
{
    echo "First version of func ()."
}

func () 
{
    echo "Second version of func ()."
}

func   # 调用的是第二个 func () 函数版本。

exit $?

#  甚至，可能用函数去覆盖
#+ 或者占用系统命令。
#  当然，这并不是可取的。
```



