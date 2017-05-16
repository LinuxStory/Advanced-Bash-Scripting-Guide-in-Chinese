# 4.3 Bash变量是弱类型的

不同于许多其他编程语言，Bash 并不区分变量的类型。本质上说，*Bash 变量是字符串*，但在某些情况下，Bash 允许对变量进行算术运算和比较。决定因素则是变量值是否只含有数字。

样例 4-4. 整数还是字符串？

```bash
#!/bin/bash
# int-or-string.sh

a=2334                   # 整数。
let "a += 1"
echo "a = $a "           # a = 2335
echo                     # 依旧是整数。


b=${a/23/BB}             # 将 "23" 替换为 "BB"。
                         # $b 变成了字符串。
echo "b = $b"            # b = BB35
declare -i b             # 将其声明为整数并没有什么卵用。
echo "b = $b"            # b = BB35

let "b += 1"             # BB35 + 1
echo "b = $b"            # b = 1
echo                     # Bash 认为字符串的"整数值"为0。

c=BB34
echo "c = $c"            # c = BB34
d=${c/BB/23}             # 将 "BB" 替换为 "23"。
                         # $d 变为了一个整数。
echo "d = $d"            # d = 2334
let "d += 1"             # 2334 + 1
echo "d = $d"            # d = 2335
echo


# 如果是空值会怎样呢？
e=''                     # ...也可以是 e="" 或 e=
echo "e = $e"            # e =
let "e += 1"             # 空值是否允许进行算术运算？
echo "e = $e"            # e = 1
echo                     # 空值变为了一个整数。

# 如果时未声明的变量呢？
echo "f = $f"            # f =
let "f += 1"             # 是否允许进行算术运算？
echo "f = $f"            # f = 1
echo                     # 未声明变量变为了一个整数。
#
# 然而……
let "f /= $undecl_var"   # 可以除以0么？
#   let: f /= : syntax error: operand expected (error token is " ")
# 语法错误！在这里 $undecl_var 并没有被设置为0！
#
# 但是，仍旧……
let "f /= 0"
#   let: f /= 0: division by 0 (error token is "0")
# 预期之中。


# 在执行算术运算时，Bash 通常将其空值的整数值设为0。
# 但是不要做这种事情！
# 因为这可能会导致一些意外的后果。


# 结论：上面的结果都表明 Bash 中的变量是弱类型的。

exit $?
```

弱类型变量有利有弊。它可以使编程更加灵活、更加容易（给与你足够的想象空间）。但它也同样的容易造成一些小错误，容易养成粗心大意的编程习惯。

为了减轻脚本持续跟踪变量类型的负担，Bash *不允许*变量声明。

