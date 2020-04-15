# 7.1 测试结构

- `if/then` 结构是用来检测一系列命令的 [退出状态](http://tldp.org/LDP/abs/html/exit-status.html#EXITSTATUSREF) 是否为0（按 UNIX 惯例,退出码 0 表示命令执行成功），如果为0，则执行接下来的一个或多个命令。
- 测试结构会使用一个特殊的命令 `[`（参看特殊字符章节 [左方括号](http://tldp.org/LDP/abs/html/special-chars.html#LEFTBRACKET)）。等同于 `test` 命令，它是一个[内建命令](http://tldp.org/LDP/abs/html/internal.html#BUILTINREF)，写法更加简洁高效。该命令将其参数视为比较表达式或文件测试，以比较结果作为其退出状态码返回（0 为真，1 为假）。
- Bash 在 2.02 版本中引入了扩展测试命令 [`[[...]]`](http://tldp.org/LDP/abs/html/testconstructs.html#DBLBRACKETS)，它提供了一种与其他语言语法更为相似的方式进行比较操作。注意， `[[` 是一个 [关键字](http://tldp.org/LDP/abs/html/internal.html#KEYWORDREF) 而非一个命令。

    Bash 将 `[[ $a -lt $b ]]` 视为一整条语句，执行并返回退出状态。

- 结构 [`(( ... ))`](http://tldp.org/LDP/abs/html/dblparens.html) 和 [`let ...`](http://tldp.org/LDP/abs/html/internal.html#LETREF) 根据其执行的算术表达式的结果决定退出状态码。这样的 [算术扩展](http://tldp.org/LDP/abs/html/arithexp.html#ARITHEXPREF) 结构可以用来进行 [数值比较](http://tldp.org/LDP/abs/html/comparison-ops.html#ICOMPARISON1)。

```bash
(( 0 && 1 ))                 # 逻辑与
echo $?     # 1     ***
# 然后 ...
let "num = (( 0 && 1 ))"
echo $num   # 0
# 然而 ...
let "num = (( 0 && 1 ))"
echo $?     # 1     ***


(( 200 || 11 ))              # 逻辑或
echo $?     # 0     ***
# ...
let "num = (( 200 || 11 ))"
echo $num   # 1
let "num = (( 200 || 11 ))"
echo $?     # 0     ***


(( 200 | 11 ))               # 按位或
echo $?                      # 0     ***
# ...
let "num = (( 200 | 11 ))"
echo $num                    # 203
let "num = (( 200 | 11 ))"
echo $?                      # 0     ***

# "let" 结构的退出状态与双括号算术扩展的退出状态是相同的。
```
    
![caution](http://tldp.org/LDP/abs/images/caution.gif) 注意，双括号算术扩展表达式的退出状态码不是一个错误的值。算术表达式为0，返回1；算术表达式不为0，返回0。

```bash
var=-2 && (( var+=2 ))
echo $?                   # 1

var=-2 && (( var+=2 )) && echo $var
                          # 并不会输出 $var, 因为((var+=2))的状态码为1
```

- `if` 不仅可以用来测试括号内的条件表达式，还可以用来测试其他任何命令。

```bash
if cmp a b &> /dev/null  # 消去输出结果
then echo "Files a and b are identical."
else echo "Files a and b differ."
fi

# 下面介绍一个非常实用的 “if-grep" 结构：
# -----------------------------------
if grep -q Bash file
  then echo "File contains at least one occurrence of Bash."
fi
    
word=Linux
letter_sequence=inu
if echo "$word" | grep -q "$letter_sequence"
# 使用 -q 选项消去 grep 的输出结果
then
  echo "$letter_sequence found in $word"
else
  echo "$letter_sequence not found in $word"
fi


if COMMAND_WHOSE_EXIT_STATUS_IS_0_UNLESS_ERROR_OCCURRED
  then echo "Command succeed."
  else echo "Command failed."
fi
```
- 感谢 Stéphane Chazelas 提供了后两个例子。

样例 7-1. 什么才是真？

```bash
#!/bin/bash

# 提示：
# 如果你不确定某个表达式的布尔值，可以用 if 结构进行测试。

echo

echo "Testing \"0\""
if [ 0 ]
then
  echo "0 is true."
else
  echo "0 is false."
fi            # 0 为真。

echo

echo "Testing \"1\""
if [ 1 ]
then
  echo "1 is true."
else
  echo "1 is false."
fi            # 1 为真。

echo

echo "Testing \"-1\""
if [ -1 ]
then
  echo "-1 is true."
else
  echo "-1 is false."
fi            # -1 为真。

echo

echo "Testing \"NULL\""
if [ ]        # NULL, 空
then
  echo "NULL is true."
else
  echo "NULL is false."
fi            # NULL 为假。

echo

echo "Testing \"xyz\""
if [ xyz ]    # 字符串
then
  echo "Random string is true."
else
  echo "Random string is false."
fi            # 随机字符串为真。

echo

echo "Testing \"$xyz\""
if [ $xyz ]   # 原意是测试 $xyz 是否为空，但是
              # 现在 $xyz 只是一个没有初始化的变量。
then
  echo "Uninitialized variable is true."
else
  echo "Uninitialized variable is flase."
fi            # 未初始化变量含有null空值，为假。

echo

echo "Testing \"-n \$xyz\""
if [ -n "$xyz" ]            # 更加准确的写法。
then
  echo "Uninitialized variable is true."
else
  echo "Uninitialized variable is false."
fi            # 未初始化变量为假。

echo


xyz=          # 初始化为空。

echo "Testing \"-n \$xyz\""
if [ -n "$xyz" ]
then
  echo "Null variable is true."
else
  echo "Null variable is false."
fi            # 空变量为假。

echo

# 什么时候 "false" 为真？

echo "Testing \"false\""
if [ "false" ]              #  看起来 "false" 只是一个字符串
then
  echo "\"false\" is true." #+ 测试结果为真。
else
  echo "\"false\" is false."
fi            # "false" 为真。

echo

echo "Testing \"\$false\""  # 未初始化的变量。
if [ "$false" ]
then
  echo "\"\$false\" is true."
else
  echo "\"\$false\" is false."
fi            # "$false" 为假。
              # 得到了我们想要的结果。

# 如果测试空变量 "$true" 会有什么样的结果？

echo

exit 0
```

练习：理解 [样例 7-1](http://tldp.org/LDP/abs/html/testconstructs.html#EX10)

```bash
if [ condition-true ]
then
   command 1
   command 2
   ...
else  # 如果测试条件为假，则执行 else 后面的代码段
   command 3
   command 4
   ...
fi
```

![note](http://tldp.org/LDP/abs/images/note.gif) 如果把 `if` 和 `then` 写在同一行时，则必须在 `if` 语句后加上一个分号来结束语句。因为 `if` 和 `then` 都是 [关键字](http://tldp.org/LDP/abs/html/internal.html#KEYWORDREF)。以关键字（或者命令）开头的语句，必须先结束该语句(分号;)，才能执行下一条语句。

```bash
if [ -x "$filename" ]; then
```

### Else if 与 elif

elif

`elif` 是 `else if` 的缩写。可以把多个 `if/then` 语句连到外边去，更加简洁明了。

```bash
if [ condition1 ]
then
   command1
   command2
   command3
elif [condition2 ]
# 等价于 else if
then
   command4
   command5
else
   default-command
fi
```

`if test condition-true` 完全等价于 `if [ condition-true ]`。当语句开始执行时，左括号 `[` 是作为调用 `test` 命令的标记[^1]，而右括号则不严格要求，但在新版本的 Bash 里，右括号必须补上。

![note](http://tldp.org/LDP/abs/images/note.gif) `test` 命令是 Bash 的 [内建命令](http://tldp.org/LDP/abs/html/internal.html#BUILTINREF)，可以用来检测文件类型和比较字符串。在 Bash 脚本中，`test` 不调用 `sh-utils` 包下的文件 `/usr/bin/test`。同样，`[` 也不会调用链接到 `/usr/bin/test` 的 `/usr/bin/[` 文件。

```
bash$ type test
test is a shell builtin
bash$ type '['
[ is a shell builtin
bash$ type '[['
[[ is a shell keyword
bash$ type ']]'
]] is a shell keyword
bash$ type ']'
bash: type: ]: not found
```

如果你想在 Bash 脚本中使用 `/usr/bin/test`，那你必须把路径写全。

样例 7-2. `test`，`/usr/bin/test`，`[]` 和 `/usr/bin/[` 的等价性

```bash
#!/bin/bash

echo

if test -z "$1"
then
  echo "No command-line arguments."
else
  echo "First command-line argument is $1."
fi

echo

if /usr/bin/test -z "$1"      # 等价于内建命令 "test"
#  ^^^^^^^^^^^^^              # 指定全路径
then
  echo "No command-line arguments."
else
  echo "First command-line argument is $1."
fi

echo

if [ -z "$1" ]                # 功能和上面的代码相同。
#   if [ -z "$1"                理论上可行，但是 Bash 会提示缺失右括号
then
  echo "No command-line arguments."
else
  echo "First command-line argument is $1."
fi

echo


if /usr/bin/[ -z "$1" ]       # 功能和上面的代码相同。
# if /usr/bin/[ -z "$1"       # 理论上可行，但是会报错
#                             # 已经在 Bash 3.x 版本被修复了
then
  echo "No command-line arguments."
else
  echo "First command-line argument is $1."
fi

echo

exit 0
```

在 Bash 里，``[[ ]]`` 是比 `[ ]` 更加通用的写法。其作为扩展`test` 命令从 ksh88 中被继承了过来。


在 ``[[`` 和 ``]]`` 中不会进行文件名扩展或字符串分割，但是可以进行参数扩展和命令替换。

```bash
file=/etc/passwd

if [[ -e $file ]]
then
  echo "Password file exists."
fi
```

使用 ``[[...]]`` 代替 ``[...]``可以避免很多逻辑错误。比如可以在 ``[[]]`` 中使用 ``&&``，``||``，`<` 和 `>` 运算符，而在 `[]` 中使用会报错。

在 ``[[]]`` 中会自动执行八进制和十六进制的进制转换操作。

```bash
# [[ 八进制和十六进制进制转换 ]]
# 感谢 Moritz Gronbach 提出。


decimal=15
octal=017   # = 15 (十进制)
hex=0x0f    # = 15 (十进制)

if [ "$decimal" -eq "$octal" ]
then
  echo "$decimal equals $octal"
else
  echo "$decimal is not equal to $octal"       # 15 不等于 017
fi      # 在单括号 [ ] 之间不会进行进制转换。


if [[ "$decimal" -eq "$octal" ]]
then
  echo "$decimal equals $octal"                # 15 等于 017
else
  echo "$decimal is not equal to $octal"
fi      # 在双括号 [[ ]] 之间会进行进制转换。

if [[ "$decimal" -eq "$hex" ]]
then
  echo "$decimal equals $hex"                  # 15 等于 0x0f
else
  echo "$decimal is not equal to $hex"
fi      # 十六进制也可以进行转换。
```

![note](http://tldp.org/LDP/abs/images/note.gif) 语法上并不严格要求在 `if` 之后一定要写 `test` 命令或者测试结构（`[]` 或 `[[]]`）。

```bash
dir=/home/bozo

if cd "$dir" 2>/dev/null; then   # "2>/dev/null" 重定向消去错误输出。
  echo "Now in $dir."
else
  echo "Can't change to $dir."
fi
```

`if COMMAND` 的退出状态就是`COMMAND` 的退出状态。

同样的，测试括号也不一定需要与 `if` 一起使用。其可以同 [列表结构](http://tldp.org/LDP/abs/html/list-cons.html#LISTCONSREF) 结合而不需要 `if`。

```bash
var1=20
var2=22
[ "$var1" -ne "$var2" ] && echo "$var1 is not equal to $var2"

home=/home/bozo
[ -d "$home" ] || echo "$home directory does not exist."
```

[`(( ))` 结构](http://tldp.org/LDP/abs/html/dblparens.html) 扩展和执行算术表达式。如果执行结果为0，其返回的 [退出状态码](http://tldp.org/LDP/abs/html/exit-status.html#EXITSTATUSREF) 为1（假）。非0表达式返回的退出状态为0（真）。这与上述所使用的 `test` 和 `[ ]` 结构形成鲜明的对比。

样例 7-3. 使用 `(( ))` 进行算术测试

```bash
#!/bin/bash
# arith-tests.sh
# 算术测试。

# (( ... )) 结构执行并测试算术表达式。
# 与 [ ... ] 结构的退出状态正好相反。

(( 0 ))
echo "Exit status of \"(( 0 ))\" is $?."         # 1

(( 1 ))
echo "Exit status of \"(( 1 ))\" is $?."         # 0

(( 5 > 4 ))                                      # 真
echo "Exit status of \"(( 5 > 4 ))\" is $?."     # 0

(( 5 > 9 ))                                      # 假
echo "Exit status of \"(( 5 > 9 ))\" is $?."     # 1

(( 5 == 5 ))                                     # 真
echo "Exit status of \"(( 5 == 5 ))\" is $?."    # 0
# (( 5 = 5 )) 会报错。

(( 5 - 5 ))                                      # 0
echo "Exit status of \"(( 5 - 5 ))\" is $?."     # 1

(( 5 / 4 ))                                      # 合法
echo "Exit status of \"(( 5 / 4 ))\" is $?."     # 0 

(( 1 / 2 ))                                      # 结果小于1
echo "Exit status of \"(( 1 / 2 ))\" is $?."     # 舍入至0。
                                                 # 1

(( 1 / 0 )) 2>/dev/null                          # 除0，非法
#           ^^^^^^^^^^^
echo "Exit status of \"(( 1 / 0 ))\" is $?."     # 1

# "2>/dev/null" 的作用是什么？
# 如果将其移除会发生什么？
# 尝试移除这条语句并重新执行脚本。

# ======================================= #

# (( ... )) 在 if-then 中也非常有用

var1=5
var2=4

if (( var1 > var2 ))
then #^      ^      注意不是 $var1 和 $var2，为什么？
  echo "$var1 is greater then $var2"
fi     # 5 大于 4

exit 0
```

[^1]: 标记是一个具有特殊意义（[元语义](http://tldp.org/LDP/abs/html/x17129.html#METAMEANINGREF)）的符号或者短字符串。在 Bash 里像 `[` 和 [`.（点命令）`](http://tldp.org/LDP/abs/html/special-chars.html#DOTREF) 这样的标记可以扩展成关键字和命令。

