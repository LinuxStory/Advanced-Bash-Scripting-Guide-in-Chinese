# 5.1 引用变量

引用变量时，通常建议将变量包含在双引号中。因为这样可以防止除 `$`，`` ` ``（反引号）和`\`（转义符）之外的其他特殊字符被重新解释。[^1]在双引号中仍然可以使用`$`引用变量（`"$variable"`），也就是将变量名替换为变量值（详情查看样例 4-1）。

使用双引号可以防止字符串被分割。[^2]即使参数中拥有很多空白分隔符，被包在双引号中后依旧是算作单一字符。

```bash
List="one two three"

for a in $List     # 空白符将变量分成几个部分。
do
  echo "$a"
done
# one
# two
# three

echo "---"

for a in "$List"   # 在单一变量中保留所有空格。
do #     ^     ^
  echo "$a"
done
# one two three
```

下面是一个更加复杂的例子：

```bash
variable1="a variable containing five words"
COMMAND This is $variable1    # 带上7个参数执行COMMAND命令：
# "This" "is" "a" "variable" "containing" "five" "words"

COMMAND "This is $variable1"  # 带上1个参数执行COMMAND命令：
# "This is a variable containing five words"


variable2=""    # 空值。

COMMAND  $variable2 $variable2 $variable2
                # 不带参数执行COMMAND命令。
COMMAND "$variable2" "$variable2" "$variable2"
                # 带上3个参数执行COMMAND命令。
COMMAND "$variable2 $variable2 $variable2"
                # 带上1个参数执行COMMAND命令（2空格）。

# 感谢 Stéphane Chazelas。
```

> ![info](http://tldp.org/LDP/abs/images/tip.gif) 当字符分割或者保留空白符出现问题时，才需要在`echo`语句中用双引号包住参数。

样例 5-1. 输出一些奇怪的变量

```bash
#!/bin/bash
# weirdvars.sh: 输出一些奇怪的变量

echo

var="'(]\\{}\$\""
echo $var        # '(]\{}$"
echo "$var"      # '(]\{}$"     没有任何区别。

echo

IFS='\'
echo $var        # '(] {}$"     \ 被转换成了空格，为什么？
echo "$var"      # '(]\{}$"

# 上面的例子由 Stephane Chazelas 提供。

echo

var2="\\\\\""
echo $var2       #   "
echo "$var2"     # \\"
echo
# 但是...var2="\\\\"" 不是合法的语句，为什么？
var3='\\\\'
echo "$var3"     # \\\\
# 强引用是可以的。


# ************************************************************ #
# 就像第一个例子展示的那样，嵌套引用是允许的。

echo "$(echo '"')"           # "
#    ^           ^


# 在有些时候这种方法非常有用。

var1="Two bits"
echo "\$var1 = "$var1""      # $var1 = Two bits
#    ^                ^

# 或者，可以像 Chris Hiestand 指出的那样：

if [[ "$(du "$My_File1")" -gt "$(du "$My_File2")" ]]
#     ^     ^         ^ ^     ^     ^         ^ ^
then
  ...
fi
# ************************************************************ #
```

单引号（' '）与双引号类似，但是在单引号中不能引用变量，因为 `$` 不再具有特殊含义。在单引号中，除`'`之外的所有特殊字符都将会被直接按照字面意思解释。可以认为单引号（“全引用”）是双引号（“部分引用”）的一种更严格的形式。

> ![extra](http://tldp.org/LDP/abs/images/note.gif) 因为在单引号中转义符（\）都已经按照字面意思解释了，因此尝试在单引号中包含单引号将不会产生你所预期的结果。
>
```bash
echo "Why can't I write 's between single quotes"
>
echo
>
# 可以采取迂回的方式。
echo 'Why can'\''t I write '"'"'s between single quotes'
#    |-------|  |----------|   |-----------------------|
# 由三个单引号引用的字符串，再加上转义以及双引号包住的单引号组成。
>
# 感谢 Stéphane Chazelas 提供的例子。
```

[^1]: 在命令行里，如果双引号包含了 "!" 将会产生错误。这是因为shell将其解释为查看历史命令。而在脚本中，因为历史机制已经被关闭，所以不会产生这个问题。<br>我们更加需要注意的是在双引号中 `\` 的反常行为，尤其是在使用 `echo -e` 命令时。<br><pre>bash$ echo hello\\!<br>hello!<br>bash$ echo "hello\\!"<br>hello\\!<br><br><br>bash$ echo \\<br>><br>bash$ echo "\\"<br>><br>bash$ echo \a<br>a<br>bash$ echo "\a"<br>\a<br><br><br>bash$ echo x\ty<br>xty<br>bash$ echo "x\ty"<br>x\ty<br><br>bash$ echo -e x\ty<br>xty<br>bash$ echo -e "x\ty"<br>x       y</pre>在 `echo` 后的双引号中一般会转义 `\`。并且 `echo -e` 会将 `"\t"` 解释成制表符。<br>（感谢 Wayne Pollock 提出这些；感谢Geoff Lee 与 Daniel Barclay 对此做出的解释。）
[^2]: 字符分割（word splitting）在本文中的意思是指将一个字符串分割成独立的、离散的变量。
