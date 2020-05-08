# 4.1 变量替换

变量名是其所指向值的一个占位符（placeholder）。引用变量值的过程我们称之为变量替换（variable substitution）。

### $

接下来我们仔细区分一下**变量名**与**变量值**。如果变量名是 `variable1`， 那么 `$variable1` 就是对变量值的引用。[^1]

```
bash$ variable1=23


bash$ echo variable1
variable1

bash$ echo $variable1
23
```

变量仅仅在声明时、赋值时、被删除时（`unset`）、被导出时（`export`），算术运算中使用双括号结构((...))时或在代表信号时（signal，查看样例 32-5）才不需要有 $ 前缀。赋值可以是使用 =（比如 `var1=27`），可以是在 `read` 语句中，也可以是在循环的头部（`for var2 in 1 2 3`）。

在双引号`""`字符串中可以使用变量替换。我们称之为部分引用，有时候也称弱引用。而使用单引号`''`引用时，变量只会作为字符串显示，变量替换不会发生。我们称之为全引用，有时也称强引用。更多细节将在第五章讲解。

实际上, `$variable` 这种写法是 `${variable}` 的简化形式。在某些特殊情况下，使用 `$variable` 写法会造成语法错误，使用完整形式会更好（查看章节 10.2）。

样例 4-1. 变量赋值与替换

```bash
#!/bin/bash
# ex9.sh

# 变量赋值与替换

a=375
hello=$a
#   ^ ^

#----------------------------------------------------
# 初始化变量时，赋值号 = 的两侧绝不允许有空格出现。
# 如果有空格会发生什么？

#   "VARIABLE =value"
#            ^
#% 脚本将会尝试运行带参数 "=value" 的 "VARIABLE " 命令。

#   "VARIABLE= value"
#             ^
#% 脚本将会尝试运行 "value" 命令，
#+ 同时设置环境变量 "VARIABLE" 为 ""。
#----------------------------------------------------


echo hello    # hello
# 没有引用变量，"hello" 只是一个字符串...

echo $hello   # 375
#    ^          这是变量引用。

echo ${hello} # 375
#               与上面的类似，变量引用。

# 字符串内引用变量
echo "$hello"    # 375
echo "${hello}"  # 375

echo

hello="A B  C   D"
echo $hello   # A B C D
echo "$hello" # A B  C   D
# 正如我们所见，echo $hello 与 echo "$hello" 的结果不同。
# ====================================
# 字符串内引用变量将会保留变量的空白符。
# ====================================

echo

echo '$hello'  # $hello
#    ^      ^
#  单引号会禁用掉（转义）变量引用，这导致 "$" 将以普通字符形式被解析。

# 注意单双引号字符串引用效果的不同。

hello=    # 将其设置为空值
echo "\$hello (null value) = $hello"      # $hello (null value) =
# 注意 
# 将一个变量设置为空与删除(unset)它不同，尽管它们的表现形式相同。

# -----------------------------------------------

# 使用空白符分隔，可以在一行内对多个变量进行赋值。
# 但是这会降低程序的可读性，并且可能会导致部分程序不兼容的问题。

var1=21  var2=22  var3=$V3
echo
echo "var1=$var1   var2=$var2   var3=$var3"

# 在一些老版本的 shell 中这样写可能会有问题。

# -----------------------------------------------

echo; echo

numbers="one two three"
#           ^   ^
other_numbers="1 2 3"
#               ^ ^
# 如果变量中有空白符号，那么必须用引号进行引用。
# other_numbers=1 2 3                  # 出错
echo "numbers = $numbers"
echo "other_numbers = $other_numbers"  # other_numbers = 1 2 3
# 也可以转义空白符。
mixed_bag=2\ ---\ Whatever
#           ^    ^ 使用 \ 转义空格

echo "$mixed_bag"         # 2 --- Whatever

echo; echo

echo "uninitialized_variable = $uninitialized_variable"
# 未初始化的变量是空值(null表示不含有任何值)。
uninitialized_variable=   # 只声明而不初始化，等同于设为空值。
echo "uninitialized_variable = $uninitialized_variable" # 仍旧为空

uninitialized_variable=23       # 设置变量
unset uninitialized_variable    # 删除变量
echo "uninitialized_variable = $uninitialized_variable"
                                # uninitialized_variable =
                                # 变量值为空
echo

exit 0
```

> ![notice](http://tldp.org/LDP/abs/images/caution.gif) 一个未被赋值或未初始化的变量拥有空值（null value）。*注意：null值不等同于0*。
> 
```bash
if [ -z "$unassigned" ]
then
  echo "\$unassigned is NULL."
fi     # $unassigned is NULL.
```
> 在赋值前使用变量可能会导致错误。但在算术运算中使用未赋值变量是可行的。
>   
```bash
echo "$uninitialized"            # 空行
let "uninitialized += 5"         # 加5
echo "$uninitialized"            # 5
# 结论：
# 一个未初始化的变量不含值(null)，但在算术运算中会被作为0处理。
```
>  
> 也可参考样例 15-23。

[^1]: 实际上，变量名是被称作左值（lvalue），意思是出现在赋值表达式的左侧的值，比如 `VARIABLE=23`。变量值被称作右值（rvalue），意思是出现在赋值表达式右侧的值，比如 `VAR2=$VARIABLE`。<br />事实上，变量名只是一个引用，一枚指针，指向实际存储数据内存地址的指针。
