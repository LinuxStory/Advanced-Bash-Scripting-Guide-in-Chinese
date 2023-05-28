# 11.4 测试与分支

`case` 和 `select` 结构并不属于循环结构，因为它们并没有反复执行代码块。但是和循环结构相似的是，它们会根据代码块顶部或尾部的条件控制程序流。

下面介绍两种在代码块中控制程序流的方法：

### `case (in)` / `esac`

在 shell 脚本中，`case` 模拟了 C/C++ 语言中的 `switch`，可以根据条件跳转到其中一个分支。其相当于简写版的 `if/then/else` 语句。很适合用来创建菜单选项哟！

```bash
case "$variable" in
  "$condition1" )
    command...
  ;;
  "$condition2" )
    command...
  ;;
esac
```

> ![note](http://tldp.org/LDP/abs/images/note.gif)
> 
> - 对变量进行引用不是必须的，因为在这里不会进行字符分割。
> 
> - 条件测试语句必须以右括号 ) 结束。[^1]
> 
> - 每一段代码块都必须以双分号 ;; 结束。
> 
> - 如果测试条件为真，其对应的代码块将被执行，而后整个 `case` 代码段结束执行。
> 
> - `case` 代码段必须以 `esac` 结束（倒着拼写case）。

样例 11-25. 如何使用 `case`

```bash
#!/bin/bash
# 测试字符的种类。

echo; echo "Hit a key, then hit return."
read Keypress

case "$Keypress" in
  [[:lower:]]   ) echo "Lowercase letter";;
  [[:upper:]]   ) echo "Uppercase letter";;
  [0-9]         ) echo "Digit";;
  *             ) echo "Punctuation, whitespace, or other";;
esac      #  字符范围可以用[方括号]表示，也可以用 POSIX 形式的[[双方括号]]表示。

# 在这个例子的第一个版本中，用来测试是小写还是大写字符使用的是 [a-z] 和 [A-Z]。
# 这在一些特定的语言环境和 Linux 发行版中不起效。
# POSIX 形式具有更好的兼容性。
# 感谢 Frank Wang 指出这一点。

# 练习：
# -----
# 这个脚本接受一个单字符然后结束。
# 修改脚本，使得其可以循环接受输入，并且检测键入的每一个字符，直到键入 "X" 为止。
# 提示：将所有东西包在 "while" 中。

exit 0
```

样例 11-26. 使用 `case` 创建菜单

```bash
#!/bin/bash

# 简易的通讯录数据库

clear # 清屏。

echo "          Contact List"
echo "          ------- ----"
echo "Choose one of the following persons:" 
echo
echo "[E]vans, Roland"
echo "[J]ones, Mildred"
echo "[S]mith, Julie"
echo "[Z]ane, Morris"
echo

read person

case "$person" in
# 注意变量是被引用的。

  "E" | "e" )
  # 同时接受大小写的输入。
  echo
  echo "Roland Evans"
  echo "4321 Flash Dr."
  echo "Hardscrabble, CO 80753"
  echo "(303) 734-9874"
  echo "(303) 734-9892 fax"
  echo "revans@zzy.net"
  echo "Business partner & old friend"
  ;;
  # 注意用双分号结束这一个选项。

  "J" | "j" )
  echo
  echo "Mildred Jones"
  echo "249 E. 7th St., Apt. 19"
  echo "New York, NY 10009"
  echo "(212) 533-2814"
  echo "(212) 533-9972 fax"
  echo "milliej@loisaida.com"
  echo "Ex-girlfriend"
  echo "Birthday: Feb. 11"
  ;;
  
  # Smith 和 Zane 的信息稍后添加。

  *         )
  # 缺省设置。
  # 空输入（直接键入回车）也是执行这一部分。
  echo
  echo "Not yet in database."
  ;;
  
esac

echo

# 练习：
# -----
# 修改脚本，使得其可以循环接受多次输入而不是只显示一个地址后终止脚本。

exit 0
```

你可以用 `case` 来检测命令行参数。

```bash
#!/bin/bash

case "$1" in
  "") echo "Usage: ${0##*/} <filename>"; exit $E_PARAM;;
                      # 没有命令行参数，或者第一个参数为空。
                      # 注意 ${0##*/} 是参数替换 ${var##pattern} 的一种形式。
                      # 最后的结果是 $0.
  
  -*) FILENAME=./$1;; #  如果传入的参数以短横线开头，那么将其替换为 ./$1
                      #+ 以避免后续的命令将其解释为一个选项。
  
  * ) FILENAME=$1;;   # 否则赋值为 $1。
esac                  
```

下面是一个更加直观的处理命令行参数的例子：

```bash
#!/bin/bash

while [ $# -gt 0 ]; do    # 遍历完所有参数
  case "$1" in
    -d|--debug)
              # 检测是否是 "-d" 或者 "--debug"。
              DEBUG=1
              ;;
    -c|--conf)
              CONFFILE="$2"
              shift
              if [ ! -f $CONFFILE ]; then
                echo "Error: Supplied file doesn't exist!"
                exit $E_CONFFILE     # 找不到文件。
              fi
              ;;
  esac
  shift       # 检测下一个参数
done

# 摘自 Stefano Falsetto 的 "Log2Rot" 脚本中 "rottlog" 包的一部分。
# 已授权使用。
```

样例 11-27. 使用命令替换生成 `case` 变量

```bash
#!/bin/bash
# case-cmd.sh: 使用命令替换生成 "case" 变量。

case $( arch ) in   # $( arch ) 返回设备架构。
                    # 等价于 'uname -m"。
  i386 ) echo "80386-based machine";;
  i486 ) echo "80486-based machine";;
  i586 ) echo "Pentium-based machine";;
  i686 ) echo "Pentium2+-based machine";;
  *    ) echo "Other type of machine";;
esac

exit 0  
```

`case` 还可以用来做字符串模式匹配。

样例 11-28. 简单的字符串匹配

```bash
#!/bin/bash
# match-string.sh: 使用 'case' 结构进行简单的字符串匹配。

match_string ()
{ # 字符串精确匹配。
  MATCH=0
  E_NOMATCH=90
  PARAMS=2     # 需要2个参数。
  E_BAD_PARAMS=91
  
  [ $# -eq $PARAMS ] || return $E_BAD_PARAMS
  
  case "$1" in
    "$2") return $MATCH;;
    *   ) return $E_NOMATCH;;
  esac
  
}


a=one
b=two
c=three
d=two

match_string $a     # 参数个数不够
echo $?             # 91

match_string $a $b  # 匹配不到
echo $?             # 90

match_string $b $d  # 匹配成功
echo $?             # 0


exit 0
```

样例 11-29. 检查输入

```bash
#!/bin/bash
# isaplpha.sh: 使用 "case" 结构检查输入。

SUCCESS=0
FAILURE=1   #  以前是FAILURE=-1,
            #+ 但现在 Bash 不允许返回负值。

isalpha ()  # 测试字符串的第一个字符是否是字母。
{
if [ -z "$1" ]                # 检测是否传入参数。
then
  return $FAILURE
fi

case "$1" in
  [a-zA-Z]*) return $SUCCESS;;  # 是否以字母形式开始？
  *        ) return $FAILURE;;
esac
}             # 可以与 C 语言中的函数 "isalpha ()" 作比较。


isalpha2 ()   # 测试整个字符串是否都是字母。
{
  [ $# -eq 1 ] || return $FAILURE
  
  case $1 in
  *[!a-zA-Z]*|"") return $FAILURE;;
               *) return $SUCCESS;;
  esac
}

isdigit ()    # 测试整个字符串是否都是数字。
{             # 换句话说，也就是测试是否是一个整型变量。
  [ $# -eq 1 ] || return $FAILURE
  
  case $1 in
    *[!0-9]*|"") return $FAILURE;;
              *) return $SUCCESS;;
  esac
}



check_var ()  # 包装后的 isalpha ()。
{
if isalpha "$@"
then
  echo "\"$*\" begins with an alpha character."
  if isalpha2 "$@"
  then        # 其实没必要检查第一个字符是不是字母。
    echo "\"$*\" contains only alpha characters."
  else
    echo "\"$*\" contains at least one non-alpha character."
  fi
else
  echo "\"$*\" begins with a non-alpha character."
              # 如果没有传入参数同样同样返回“存在非字母”。
fi
  
echo
  
}

digit_check ()  # 包装后的 isdigit ()。
{
if isdigit "$@"
then
  echo "\"$*\" contains only digits [0 - 9]."
else
  echo "\"$*\" has at least one non-digit character."
fi
  
echo
  
}


a=23skidoo
b=H3llo
c=-What?
d=What?
e=$(echo $b)   # 命令替换。
f=AbcDef
g=27234
h=27a34
i=27.34

check_var $a
check_var $b
check_var $c
check_var $d
check_var $e
check_var $f
check_var     # 如果不传入参数会发送什么？
#
digit_check $g
digit_check $h
digit_check $i


exit 0        # S.C. 改进了本脚本。

# 练习：
# -----
# 写一个函数 'isfloat ()' 来检测输入值是否是浮点数。
# 提示：可以参考函数 'isdigit ()'，在其中加入检测合法的小数点即可。
```

### `select`

`select` 结构是学习自 Korn Shell。其同样可以用来构建菜单。

```bash
select variable [in list]
do
 command...
 break
done
```

而效果则是终端会提示用户输入列表中的一个选项。注意，`select` 默认使用提示字串3（Prompt String 3，`$PS3`, 即#?），但同样可以被修改。

样例 11-30. 使用 `select` 创建菜单

```bash
#!/bin/bash

PS3='Choose your favorite vegetable: ' # 设置提示字串。
                                       # 否则默认为 #?。

echo

select vegetable in "beans" "carrots" "potatoes" "onions" "rutabagas"
do
  echo
  echo "Your favorite veggie is $vegetable."
  echo "Yuck!"
  echo
  break  # 如果没有 'break' 会发生什么？
done

exit

# 练习:
# -----
# 修改脚本，使得其可以接受其他输入而不是 "select" 语句中所指定的。
# 例如，如果用户输入 "peas,"，那么脚本会通知用户 "Sorry. That is not on the menu."
```

如果 *in list* 被省略，那么 `select` 将会使用传入脚本的命令行参数（`$@`）或者传入函数的参数作为 *list*。

可以与 `for variable [in list]` 中 *in list* 被省略的情况做比较。

样例 11-31. 在函数中使用 `select` 创建菜单

```bash
#!/bin/bash

PS3='Choose your favorite vegetable: '

echo

choice_of()
{
select vegetable
# [in list] 被省略，因此 'select' 将会使用传入函数的参数作为 list。
do
  echo
  echo "Your favorite veggie is $vegetable."
  echo "Yuck!"
  echo
  break
done
}

choice_of beans rice carrorts radishes rutabaga spinach
#         $1    $2   $3      $4       $5       $6
#         传入了函数 choice_of()

exit 0
```

还可以参照 [样例37-3](http://tldp.org/LDP/abs/html/bashver2.html#RESISTOR)。

[^1]: 在写匹配行的时候，可以在左边加上左括号 (，使整个结构看起来更加优雅。<pre>case $( arch ) in   # $( arch ) 返回设备架构。<br>  ( i386 ) echo "80386-based machine";;<br># ^      ^<br>  ( i486 ) echo "80486-based machine";;<br>  ( i586 ) echo "Pentium-based machine";;<br>  ( i686 ) echo "Pentium2+-based machine";;<br>  (    * ) echo "Other type of machine";;<br>esac</pre>
