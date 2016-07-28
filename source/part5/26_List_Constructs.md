# 第二十六章. 列表结构

_and 列表_ 和 _or 列表_ 结构提供了连续执行若干命令的方法，可以有效地替换复杂的嵌套 [if/then](http://tldp.org/LDP/abs/html/testconstructs.html#TESTCONSTRUCTS1) ，甚至 [case](http://tldp.org/LDP/abs/html/testbranch.html#CASEESAC1) 语句。

### 链接多个命令

**and 列表**

``` command-1 && command-2 && command-3 && ... command-n ```

只要前一个命令返回 _true_（即 0），每一个命令就依次执行。当第一个 _false_（即 非0）返回时，命令链条即终止（第一个返回 _false_ 的命令是最后一个执行的）。

在[YongYe](https://github.com/yongye)早期版本的[俄罗斯方块游戏](http://bash.deta.in/Tetris_Game.sh)脚本里，一个有趣的双条件 _and 列表_ 用法：
```
equation()

{  # core algorithm used for doubling and halving the coordinates
   [[ ${cdx} ]] && ((y=cy+(ccy-cdy)${2}2))
   eval ${1}+=\"${x} ${y} \"
}
```

**例 26-1. 使用 _and 列表_ 来测试命令行参数**
```
#!/bin/bash
# and list

if [ ! -z "$1" ] && echo "Argument #1 = $1" && [ ! -z "$2" ] && \
#                ^^                         ^^               ^^
echo "Argument #2 = $2"
then
  echo "At least 2 arguments passed to script."
  # 链条内的所有命令都返回 true。
else
  echo "Fewer than 2 arguments passed to script."
  # 链条内至少有一个命令返回 false。
fi  
# 注意： "if [ ! -z $1 ]" 是好用的，但是宣传与之等同的
#   "if [ -n $1 ]" 并不好用。
#  不过，用引号就能解决问题，
#   "if [ -n "$1" ]" 好用（译者注：原文本行内第一个引号位置错了）。
#           ^  ^    小心!
# 被测试的变量放在引号内总是最好的选择。


# 下面的代码功能一样，用的是“纯粹”的 if/then 语句。
if [ ! -z "$1" ]
then
  echo "Argument #1 = $1"
fi
if [ ! -z "$2" ]
then
  echo "Argument #2 = $2"
  echo "At least 2 arguments passed to script."
else
  echo "Fewer than 2 arguments passed to script."
fi
# 比起用“and 列表”要更长、更笨重。


exit $?
```

**例 26-2. 使用 _and 列表_ 来测试命令行参数2**

```
#!/bin/bash

ARGS=1        # 预期的参数数量。
E_BADARGS=85  # 参数数量错误时返回的值。

test $# -ne $ARGS && \
#    ^^^^^^^^^^^^ 条件 #1
echo "Usage: `basename $0` $ARGS argument(s)" && exit $E_BADARGS
#                                             ^^
#  如果条件 #1 结果为 true (传递给脚本的参数数量错误),
#+ 那么执行本行剩余的命令，脚本终止。

# 下面的代码行只有在上面的测试失败时才执行。
echo "Correct number of arguments passed to this script."

exit 0

#  如果要检查退出值，脚本终止后运行 "echo $?"。
```

当然，_and 列表_ 也可以给变量设置默认值。

```
arg1=$@ && [ -z "$arg1" ] && arg1=DEFAULT

              # 如果有命令行参数，则把参数值赋给 $arg1 。
              # 但是... 如果没有参数，则使用DEFAULT给 $arg1 赋值。
```

**or 列表**
```
command-1 || command-2 || command-3 || ... command-n
```
只要前一个命令返回_false_，每一个命令就依次执行。当第一个_true_ 返回时，命令链条即终止（第一个返回_true_ 的命令是最后一个执行的）。很明显它与“and 列表”相反。

例 26-3. _or 列表_ 与 _and 列表_ 结合使用
```
#!/bin/bash

#  delete.sh, 不那么巧妙的文件删除工具。
#  用法： delete 文件名

E_BADARGS=85

if [ -z "$1" ]
then
  echo "Usage: `basename $0` filename"
  exit $E_BADARGS  # No arg? Bail out.
else  
  file=$1          # Set filename.
fi  


[ ! -f "$file" ] && echo "File \"$file\" not found. \
Cowardly refusing to delete a nonexistent file."
# AND 列表，如果文件不存在则显示出错信息。
# 注意，echo 消息内容分成了两行，中间通过转义符（\）连接。

[ ! -f "$file" ] || (rm -f $file; echo "File \"$file\" deleted.")
# OR 列表，删除存在的文件。

# 注意上面的逻辑颠倒。 Note logic inversion above.
# “AND 列表” 在得到 true 时执行, “OR 列表”在得到 false 时执行。

exit $?
```

<img src="http://tldp.org/LDP/abs/images/caution.gif"> 如果 _or 列表_ 第一个命令返回 true，它**会**执行。
```
# ==> 下面的代码段来自 /etc/rc.d/init.d/single
#+==> 作者 Miquel van Smoorenburg
#+==> 说明了 "and" 和 "or" 列表。
# ==> 带箭头的注释是本文作者添加的。

[ -x /usr/bin/clear ] && /usr/bin/clear
  # ==> 如果 /usr/bin/clear 存在, 则调用它。
  # ==> 调用命令之前检查它是否存在，
  #+==> 可以避免出错消息和其他怪异的结果。

  # ==> . . .

#  If they want to run something in single user mode, might as well run it...
for i in /etc/rc1.d/S[0-9][0-9]* ; do
        # 检查脚本是否存在。
        [ -x "$i" ] || continue
  # ==> 如果对应的文件在 $PWD 里*没有*找到，
  #+==> 则跳回到循环顶端“继续运行”。

        # 丢弃备份文件和 rpm 生成的文件。
        case "$1" in
                *.rpmsave|*.rpmorig|*.rpmnew|*~|*.orig)
                        continue;;
        esac
        [ "$i" = "/etc/rc1.d/S00single" ] && continue
  # ==> 设置脚本名，但先不要执行
        $i start
done

  # ==> . . .
```
<img src="http://tldp.org/LDP/abs/images/important.gif">
_and 列表_ 或 _or 列表_ 的[退出状态](http://tldp.org/LDP/abs/html/exit-status.html#EXITSTATUSREF)就是最后一个执行的命令的退出状态。

聪明地结合 _and 列表_ 和 _or 列表_ 是可能的，但是程序逻辑会很容易地变得令人费解，需要密切注意[操作符优先规则](http://tldp.org/LDP/abs/html/opprecedence.html#OPPRECEDENCE1)，而且，会带来大量的调试工作。
```
false && true || echo false         # false

# 下面的代码结果相同
( false && true ) || echo false     # false
# 但这个就不同了
false && ( true || echo false )     # (什么都不显示)

#  注意语句是从左到右组合和解释的。

#  通常情况下最好避免这种复杂性。

#  感谢, S.C.
```
[例 A-7](http://tldp.org/LDP/abs/html/contributed-scripts.html#DAYSBETWEEN) 和 [例 7-4](http://tldp.org/LDP/abs/html/fto.html#BROKENLINK) 解释了用 _and 列表_ / _or 列表_ 来测试变量。
