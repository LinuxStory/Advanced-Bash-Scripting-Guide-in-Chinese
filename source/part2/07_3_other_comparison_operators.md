# 7.3 其他比较操作

二元比较操作比较变量或者数量。注意整数和字符串比较使用的是两套运算符。

## 整数比较

### -eq

等于

`if [ "$a" -eq "$b" ]`

### -ne

不等于

`if [ "$a" -ne "$b" ]`

### -gt

大于

`if [ "$a" -gt "$b" ]`

### -ge

大于等于

`if [ "$a" -ge "$b" ]`

### -lt

小于

`if [ "$a" -lt "$b" ]`

### -le

小于等于

`if [ "$a" -le "$b" ]`

### <

小于（使用 [双圆括号](http://tldp.org/LDP/abs/html/dblparens.html)）

`(("$a" < "$b"))`

### <=

小于等于（使用双圆括号）

`(("$a" <= "$b"))`

### >

大于（使用双圆括号）

`(("$a" > "$b"))`

### >=

大于等于（使用双圆括号）

`(("$a" >= "$b"))`

## 字符串比较

### =

等于

`if [ "$a" = "$b" ]`

![caution](http://tldp.org/LDP/abs/images/caution.gif) 注意在`=`前后要加上[空格](http://tldp.org/LDP/abs/images/caution.gif)

`if [ "$a"="$b" ]` 和上面不等价。

### ==

等于

`if [ "$a" == "$b" ]`

和 `=` 同义

![note](http://tldp.org/LDP/abs/images/note.gif) `==` 运算符在 [双方括号](http://tldp.org/LDP/abs/html/testconstructs.html#DBLBRACKETS) 和单方括号里表现不同。

```bash
[[ $a == z* ]]   # $a 以 "z" 开头时为真（模式匹配）
[[ $a == "z*" ]] # $a 等于 z* 时为真（字符匹配）

[ $a == z* ]     # 发生文件匹配和字符分割。
[ "$a" == "z*" ] # $a 等于 z* 时为真（字符匹配）

# 感谢 Stéphane Chazelas
```

### !=

不等于

`if [ "$a" != "$b" ]`

在 [`[[ ... ]]`](http://tldp.org/LDP/abs/html/testconstructs.html#DBLBRACKETS) 结构中会进行模式匹配。

### <

小于，按照 [ASCII码](http://tldp.org/LDP/abs/html/special-chars.html#ASCIIDEF) 排序。

`if [[ "$a" < "$b" ]]`

`if [ "$a" \< "$b" ]`

注意在 `[]` 结构里 `<` 需要被 [转义](http://tldp.org/LDP/abs/html/escapingsection.html#ESCP)。

### >

大于，按照 ASCII 码排序。

`if [[ "$a" > "$b" ]]`

`if [ "$a" \> "$b" ]`

注意在 `[]` 结构里 `>` 需要被转义。

[样例 27-11](http://tldp.org/LDP/abs/html/arrays.html#BUBBLE) 包含了比较运算符。

### -z

字符串为空，即字符串长度为0。

```bash
String=''   # 长度为0的字符串变量。

if [ -z "$String" ]
then
  echo "\$String is null."
else
  echo "\$String is NOT null."
fi     # $String is null.
```

### -n

字符串非空（`null`）。

![caution](http://tldp.org/LDP/abs/images/caution.gif) 使用 `-n` 时字符串必须是在括号中且被引用的。使用 `! -z` 判断未引用的字符串或者直接判断（[样例 7-6](http://tldp.org/LDP/abs/html/comparison-ops.html#STRTEST)）通常可行，但是非常危险。判断字符串时一定要引用[^1]。

样例 7-5. 算术比较和字符串比较

```bash
#!/bin/bash

a=4
b=5

# 这里的 "a" 和 "b" 可以是整数也可以是字符串。
# 因为 Bash 的变量是弱类型的，因此字符串和整数比较有很多相同之处。

# 在 Bash 中可以用处理整数的方式来处理全是数字的字符串。
# 但是谨慎使用。

echo

if [ "$a" -ne "$b" ]
then
  echo "$a is not equal to $b"
  echo "(arithmetic comparison)"
fi

echo

if [ "$a" != "$b" ]
then
  echo "$a is not equal to $b."
  echo "(string comparison)"
  #     "4"  != "5"
  # ASCII 52 != ASCIII 53
fi

# 在这个例子里 "-ne" 和 "!=" 都可以。

echo

exit 0
```

样例 7-6. 测试字符串是否为空（`null`）

```bash
#!/bin/bash
# str-test.sh: 测试是否为空字符串或是未引用的字符串。

# 使用 if [ ... ] 结构

# 如果字符串未被初始化，则其值是未定义的。
# 这种状态就是空 "null"（并不是 0）。

if [ -n $string1 ]    # 并未声明或是初始化 string1。
then
  echo "String \"string1\" is not null."
else
  echo "String \"string1\" is null."
fi
# 尽管没有初始化 string1，但是结果显示其非空。

echo

# 再试一次。

if [ -n "$string1" ]   # 这次引用了 $string1。
then
  echo "String \"string1\" is not null."
else
  echo "String \"string1\" is null."
fi                    # 在测试括号内引用字符串得到了正确的结果。

echo

if [ $string1 ]       # 这次只有一个 $string1。
then
  echo "String \"string1\" is not null."
else
  echo "String \"string1\" is null."
fi                    # 结果正确。
# 独立的 [ ... ] 测试运算符可以用来检测字符串是否为空。
# 但是最好将字符串进行引用（if [ "$string1" ]）。
#
# Stephane Chazelas 指出：
#    if [ $string1 ]    只有一个参数 "]"
#    if [ "$string1" ]  则有两个参数，空的 "$string1" 和 "]"


echo


string1=initialized

if [ $string1 ]       # $string1 这次仍然没有被引用。
then
  echo "String \"string1\" is not null."
else
  echo "String \"string1\" is null."
fi                    # 这次的结果仍然是正确的。
# 最好将字符串引用（"$string1"）


string1="a = b"

if [ $string1 ]       # $string1 这次仍然没有被引用。
then
  echo "String \"string1\" is not null."
else
  echo "String \"string1\" is null."
fi                    # 这次没有引用就错了。

exit 0   # 同时感谢 Florian Wisser 的提示。
```

样例 7-7. `zmore`

```bash
#!/bin/bash
# zmore

# 使用筛选器 'more' 查看 gzipped 文件。

E_NOARGS=85
E_NOTFOUND=86
E_NOTGZIP=87

if [ $# -eq 0 ] # 作用和 if [ -z "$1" ] 相同。
# $1 可以为空： zmore "" arg2 arg3
then
  echo "Usage: `basename $0` filename" >&2
  # 将错误信息通过标准错误 stderr 进行输出。
  exit $E_NOARGS
  # 脚本的退出状态为 85.
fi

filename=$1

if [ ! -f "$filename" ]   # 引用字符串以防字符串中带有空格。
then
  echo "File $filename not found!" >&2   # 通过标准错误 stderr 进行输出。
  exit $E_NOTFOUND
fi

if [ ${filename##*.} != "gz" ]
# 在括号内使用变量代换。
then
  echo "File $1 is not a gzipped file!"
  exit $E_NOTGZIP
fi

zcat $1 | more

# 使用筛选器 'more'
# 也可以用 'less' 替代

exit $?   # 脚本的退出状态由管道 pipe 的退出状态决定。
#  实际上 "exit $?" 不一定要写出来，
#+ 因为无论如何脚本都会返回最后执行命令的退出状态。
```

## 复合比较

### -a

逻辑与

`exp1 -a exp2` 返回真当且仅当 `exp1` 和 `exp2` 均为真。

### -o

逻辑或

如果 `exp1` 或 `exp2` 为真，则 `exp1 -o exp2` 返回真。

以上两个操作和 [双方括号](http://tldp.org/LDP/abs/html/testconstructs.html#DBLBRACKETS) 结构中的 Bash 比较运算符号 `&&` 和 `||` 类似。

```bash
[[ condition1 && condition2 ]]
```

测试操作 `-o` 和 `-a` 可以在 [`test`](http://tldp.org/LDP/abs/html/testconstructs.html#TTESTREF) 命令或在测试括号中进行。

```bash
if [ "$expr1" -a "$expr2" ]
then
  echo "Both expr1 and expr2 are true."
else
  echo "Either expr1 or expr2 is false."
fi
```

![caution](http://tldp.org/LDP/abs/images/caution.gif) rihad 指出：

```bash
[ 1 -eq 1 ] && [ -n "`echo true 1>&2`" ]   # 真
[ 1 -eq 2 ] && [ -n "`echo true 1>&2`" ]   # 没有输出
# ^^^^^^^ 条件为假。到这里为止，一切都按预期执行。

# 但是
[ 1 -eq 2 -a -n "`echo true 1>&2`" ]       # 真
# ^^^^^^^ 条件为假。但是为什么结果为真？

# 是因为括号内的两个条件子句都执行了么？
[[ 1 -eq 2 && -n "`echo true 1>&2`" ]]     # 没有输出
# 并不是。

#  所以显然 && 和 || 具备“短路”机制，
#+ 例如对于 &&，若第一个表达式为假，则不执行第二个表达式直接返回假，
#+ 而 -a 和 -o 则不是。
```

复合比较操作的例子可以参考 [样例 8-3](http://tldp.org/LDP/abs/html/ops.html#ANDOR)，[样例 27-17](http://tldp.org/LDP/abs/html/arrays.html#TWODIM) 和 [样例 A-29](http://tldp.org/LDP/abs/html/contributed-scripts.html#WHX)。

[^1]: S.C. 指出在复合测试中，仅仅引用字符串可能还不够。比如表达式 `[ -n "$string" -o "$a" = "$b" ]` 在某些 Bash 版本下，如果 `$string` 为空可能会出错。更加安全的方式是，对于可能为空的字符串，添加一个额外的字符，例如 `[ "x$string" != x -o "x$a" = "x$b" ]`（其中的 x 互相抵消）。

