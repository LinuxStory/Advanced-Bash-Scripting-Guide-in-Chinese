# 4.1 变量替换

变量名是其所指向值的一个占位符（placeholder）。而引用这个值的过程我们称之为变量替换（variable substitution）。

### $

我们接下来仔细区分一下变量的名称与变量的值。如果变量名是 `variable1`， 那么 `$variable1` 就是对变量的值的引用。[^1]

```
bash$ variable1=23


bash$ echo variable1
variable1

bash$ echo $variable1
23
```

变量仅仅在声明时、赋值时、被删除时（`unset`）、被导出时（`export`），算术运算中使用双括号结构时或在代表信号时（signal，查看样例 32-5）才不需要有 $ 前缀。赋值可以是使用 =（比如 `var1=27`），可以是在 `read` 语句中，也可以是在循环的头部（`for var2 in 1 2 3`）。

变量即使在双引号`""`中被引用也不会影响变量替换。我们称之为部分引用，有时候也称弱引用。而使用单引号`''`引用时，变量将会作为字符串显示，变量替换也不会发生。我们称之为全引用，有时也称强引用。更多细节可以查看第五章。

实际上 `$variable` 是 `${variable}` 的简化形式。在一些使用 `$variable` 有语法错误的情况下，使用完整的形式可能会有效（查看章节 10.2）。

样例 4-1. 变量赋值与替换

```bash
#!/bin/bash
# ex9.sh

# 变量赋值与替换

a=375
hello=$a
#   ^ ^

#----------------------------------------------------
# 在初始化变量时，赋值号 = 的两侧不允许有空格出现。
# 如果有空格将会发生什么？

#   "VARIABLE =value"
#            ^
#% 脚本将会尝试运行带参数 "=value" 的 "VARIABLE " 命令。

#   "VARIABLE= value"
#             ^
#% 脚本将会尝试运行 "value" 命令，
#+ 同时设置环境变量 "VARIABLE" 为 ""。
#----------------------------------------------------


echo hello    # hello
# 没有引用变量，"hello" 只是一个字符串。

echo $hello   # 375
#    ^          这是一个变量引用。

echo ${hello} # 375
#               与上面的类似，是一个变量引用。

# 在引用时
echo "$hello"    # 375
echo "${hello}"  # 375

echo

hello="A B  C   D"
echo $hello   # A B C D
echo "$hello" # A B  C   D
# 正如我们所见，echo $hello 与 echo "$hello" 的结果不同。
# =========================
# 引用一个变量将会保留空白符。
# =========================

echo

echo '$hello'  # $hello
#    ^      ^
#  单引号将会禁用（转义）变量引用，这导致 "$" 将会以字符的形式被解析。

# 需要注意不同形式引用的效果。

hello=    # 将其设置为空值
echo "\$hello (null value) = $hello"      # $hello (null value) =
# 注意一个变量设置为空与删除它不同，尽管他们最后的结果是相同的。

# -----------------------------------------------

# 允许使用空白符分割，从而在一行内对多个变量进行赋值。
# 这将会降低可读性，并且可能会导致不兼容。

var1=21  var2=22  var3=$V3
echo
echo "var1=$var1   var2=$var2   var3=$var3"

# 在一些老版本的 shell 中可能会有问题。

# -----------------------------------------------

echo; echo

numbers="one two three"
#           ^   ^
other_numbers="1 2 3"
#               ^ ^
# 如果在变量中出现空格，那么必须进行引用。
# other_numbers=1 2 3                  # 出错
echo "numbers = $numbers"
echo "other_numbers = $other_numbers"  # other_numbers = 1 2 3
# 转义空格也可以达到相同的效果。
mixed_bag=2\ ---\ Whatever
#           ^    ^ 使用 \ 转义空格

echo "$mixed_bag"         # 2 --- Whatever

echo; echo

echo "uninitialized_variable = $uninitialized_variable"
# 未初始化的变量是空值。
uninitialized_variable=   # 只声明而不初始化，等同于设为空值。
echo "uninitialized_variable = $uninitialized_variable" # 仍旧为空

uninitialized_variable=23       # 设置变量
unset uninitialized_variable    # 删除变量
echo "uninitialized_variable = $uninitialized_variable"
                                # uninitialized_variable =
                                # 仍旧为空值
echo

exit 0
```

> ![notice](http://tldp.org/LDP/abs/images/caution.gif) 一个未初始化的变量拥有空值（null value），即没有被复值。空值不是0。
>
```bash
if [ -z "$unassigned" ]
then
  echo "\$unassigned is NULL."
fi     # $unassigned is NULL.
```
> 在赋值前使用变量可能会导致错误。但是，在算术运算中使用未赋值变量时可行的。
>
```bash
echo "$uninitialized"                                # 空行
let "uninitialized += 5"                             # 加上5
echo "$uninitialized"                                # 5
>
# 结论：
# 尽管一个未初始化的变量没有值，但是在算术运算中作为0处理。
```
> 也可参考样例 15-23。

[^1]: 实际上，变量名是被称作左值（lvalue），意思是出现在赋值表达式的左侧的值，比如 `VARIABLE=23`。变量的值被称作右值（rvalue），意思是出现在赋值表达式右侧的值，比如 `VAR2=$VARIABLE`。<br />事实上，变量名只是一个引用，它是指向数据存储的内存地址的指针。