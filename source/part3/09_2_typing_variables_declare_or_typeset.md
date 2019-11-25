# 9.2 变量类型标注：`declare` 与 `typeset`

[内建命令]() `declare` 和 `typeset` 是完全相同的命令，其被用于修改变量的属性。相比起一部分编程语言来说，这种修改属于非常弱的类型标注方式[^1]。`declare` 命令只有在 Bash version 2 及更高版本才能使用，而 `typeset` 命令可以在 ksh 脚本中运行。

## `declare`/`typeset` 命令选项

### -r 只读（readonly）

（`declare -r var1` 与 `readonly var1` 的作用相同）

该选项约等于 C 语言中的类型限定符 `const`。任何尝试修改只读变量的行为都会导致脚本出错。

```bash
declare -r var1=1
echo "var1 = $var1"   # var1 = 1

(( var1++ ))          # x.sh: line 4: var1: readonly variable
```

### -i 整型（integer）

```bash
declare -i number
# 脚本会将之后所有出现的 "number" 变量的类型都视作整型。

number=3
echo "Number = $number"     # Number = 3

number=three
echo "Number = $number"     # Number = 0
# 脚本试图将字符串 "three" 视为整型。
```

被视为整型的变量无需命令 [`expr`]() 或是 [`let`]() 即可进行数学运算。

```bash
n=6/3
echo "n = $n"       # n = 6/3

declare -i n
n=6/3
echo "n = $n"       # n = 2
```

### -a 数组（array）

```bash
declare -a indices
```

变量 `indices` 会被视作 [数组]()。

### -f 函数（function）

```bash
declare -f
```

如果没有在 `declare -f` 后带上任何参数，该语句将会列出在脚本中已经定义的所有函数。

```bash
declare -f function_name
```

而 `declare -f function_name` 则仅仅列出名为 `function_name` 的函数。

### -x 导出（export）

```bash
declare -x var3
```

该语句声明了变量 `var3` 可以导出到该变量所属脚本之外的 shell 环境中。

### -x var=$value

```bash
declare -x var3=373
```

`declare` 命令允许在设置变量属性的同时给变量赋值。

#### 样例9-10. 使用 `declare` 命令标注变量类型

```bash
#!/bin/bash

func1 ()
{
  echo This is a function.
}

declare -f        # 显示上面的所有函数。

echo

declare -i var1   # var1 是一个整型变量。
var1=2367
echo "var1 declared as $var1"
var1=var1+1       # 整型变量的运算可以省略 let 命令。
echo "var1 incremented by 1 is $var1."
# 尝试修改整型变量。
echo "Attempting to change var1 to floating point value, 2367.1."
var1=2367.1       # 报错，并且 var1 的值并没有被修改。
echo "var1 is still $var1"

echo

declare -r var2=13.36         # 'declare' 允许在设置变量属性时，
                              #+ 同时给变量赋值。
echo "var2 declared as $var2" # 尝试修改只读变量。
var2=13.37                    # 报错，然后脚本异常结束。

echo "var2 is still $var2"    # 这行语句将不会被执行。

exit 0                        # 脚本也不会从这里结束。
```

{% hint style="warning" %}

使用内建命令 `declare` 还可以限制变量的 [作用域]()。

```bash
foo ()
{
FOO="bar"
}

bar ()
{
foo
echo $FOO
}

bar   # 输出 bar。
```

但是...

```bash
foo(){
declare FOO="bar"
}

bar ()
{
foo
echo $FOO
}

bar  # 什么都不会输出。


# 感谢 Michael Iatrou 指出这点。
```

{% endhint %}

## 注记

{% hint style="info" %}
在本书中，变量类型标注（typing）是指指定变量类型并限制其属性。例如一个变量被 `declared` 或是 `typed` 命令声明为整型，则该变量不再适用于各种 [字符串操作]()。

```bash
declare -i intvar

intvar=23
echo "$intvar"   # 23
intvar=stringval
echo "$intvar"   # 0
```
{% endhint %}

[^1]: Footnotes placeholder