# 28 间接引用

我们已经看到[引用变量](https://www.tldp.org/LDP/abs/html/varsubn.html)，`$var`，获取其*值*。但是*值的值*又如何呢？可以是 `$$var` 吗？

真正的表达式是 `\$$var`，一般前面有 eval（有时候是 echo）。这被称作*间接引用*。

**例 28-1. 间接变量引用**

```bash
#!/bin/bash
# ind-ref.sh: 间接变量引用。
# 获取一个变量内容的内容。

# 首先让我们随便玩玩。

var=23

echo "\$var   = $var"           # $var   = 23
# 到现在为止，一切都在预料中。但是……

echo "\$\$var  = $$var"         # $$var  = 4570var
#  没用
#  \$\$ 展开为脚本的 PID
#  -- 引用了 $$ 变量的条目 --
#+ 并且“var”作为文本打印了出来。
#  (感谢你，Jakob Bohm，指出了这一点。)

echo "\\\$\$var = \$$var"       # \$$var = $23
#  符合预期。第一个 $ 被转义，传给了 var 的值（$var = 23）。
#  有意思了，但还不能用。

# 现在，让我们开始做正确的事情。

# ============================================== #


a=letter_of_alphabet   # 变量“a”保存了另一个变量的名字。
letter_of_alphabet=z

echo

# 直接引用。
echo "a = $a"          # a = letter_of_alphabet

# 间接引用。
  eval a=\$$a
# ^^^        强制运算（evaluation），并且……
#        ^   转义第一个 $ ……
# ------------------------------------------------------------------------
# “eval”强制更新了 $a，将其设置为更新后的 \$$a 的值。
# 所以，我们看到为什么“eval”如此频繁地出现在间接引用的表达式中了。
# ------------------------------------------------------------------------
  echo "Now a = $a"    # 现在 a = z

echo


# 现在，让我们尝试改变第二个顺序的引用。

t=table_cell_3
table_cell_3=24
echo "\"table_cell_3\" = $table_cell_3"            # "table_cell_3" = 24
echo -n "dereferenced \"t\" = "; eval echo \$$t    # dereferenced "t" = 24
# 在这个简单的情况中，下面的语句也可以（为什么？）。
#         eval t=\$$t; echo "\"t\" = $t"

echo

t=table_cell_3
NEW_VAL=387
table_cell_3=$NEW_VAL
echo "Changing value of \"table_cell_3\" to $NEW_VAL."
echo "\"table_cell_3\" now $table_cell_3"
echo -n "dereferenced \"t\" now "; eval echo \$$t
# “eval”获取两个参数“echo”和“\$$t”（设为等于 $table_cell_3）

echo

# (Thanks, Stephane Chazelas, for clearing up the above behavior.)


#   更直接的方法是 ${!t} 表示法，在 “Bash，版本 2” 一节中讨论。
#   另请参考 ex78.sh。

exit 0
```

> Bash 中的间接引用是一个多步骤过程。首先，获取变量名称：`varname`。然后，引用它：`$varname`。然后，引用这个引用：`$$varname`。然后，*转义*第一个 $：`\$$varname`。最后，强制重新计算表达式并赋值：**eval newvar=\\$$varname**。

间接引用变量有什么实际用途？它给了 Bash 一些 C 语言中指针的一点功能，例如，表格查询。此外，它也有其他的一些非常有趣的应用……

Nils Radtke 展示了如何构建“动态”变量名称并计算他们的内容。这在 source 配置文件的时候会有用处。

```bash
#!/bin/bash


# ---------------------------------------------
# 这部分可以从独立的文件中“source”。
isdnMyProviderRemoteNet=172.16.0.100
isdnYourProviderRemoteNet=10.0.0.10
isdnOnlineService="MyProvider"
# ---------------------------------------------
      

remoteNet=$(eval "echo \$$(echo isdn${isdnOnlineService}RemoteNet)")
remoteNet=$(eval "echo \$$(echo isdnMyProviderRemoteNet)")
remoteNet=$(eval "echo \$isdnMyProviderRemoteNet")
remoteNet=$(eval "echo $isdnMyProviderRemoteNet")

echo "$remoteNet"    # 172.16.0.100

# ================================================================

#  更好了。

#  考虑下面这个片段，给了变量 getSparc，但没有变量 getIa64：

chkMirrorArchs () { 
  arch="$1";
  if [ "$(eval "echo \${$(echo get$(echo -ne $arch |
       sed 's/^\(.\).*/\1/g' | tr 'a-z' 'A-Z'; echo $arch |
       sed 's/^.\(.*\)/\1/g')):-false}")" = true ]
  then
     return 0;
  else
     return 1;
  fi;
}

getSparc="true"
unset getIa64
chkMirrorArchs sparc
echo $?        # 0
               # True

chkMirrorArchs Ia64
echo $?        # 1
               # False

# 注：
# -----
# 即使是要被替换的变量名称部分也是独立构建的。
# 给 chkMirrorArchs 调用的参数都是小写。
# 变量名称由两部分组成：“get”和“Sparc”……
```

**例 28-2. 传递间接引用给 *awk***

```bash
#!/bin/bash

#  “列求和”脚本的另一个版本，它将目标文件中指定列的数字累加。
#  这个版本使用了间接引用。
ARGS=2
E_WRONGARGS=85

if [ $# -ne "$ARGS" ] # 检查命令行参数的数量是否正确
then
   echo "Usage: `basename $0` filename column-number"
   exit $E_WRONGARGS
fi

filename=$1         # 操作的文件名
column_number=$2    # 对哪一列求和

#===== 到此为止与原脚本一样 =====#


# 多行 awk 脚本，由 awk 调用
#   awk "
#   ...
#   ...
#   ...
#   "


# awk 脚本开始。
# -------------------------------------------------
awk "

{ total += \$${column_number} # 间接引用
}
END {
     print total
     }

     " "$filename"
# 注意 awk 不需要在 \$$ 前置 eval。
# -------------------------------------------------
# awk 脚本结束。

#  间接变量引用避免了嵌入 awk 脚本内引用 shell 变量的麻烦。
#  感谢 Stephane Chazelas。


exit $?
```

![Caution](https://www.tldp.org/LDP/abs/images/caution.gif) 这个间接引用的方法有些小技巧。如果二级变量改变了值，那么必须正确地取消对一级变量的引用。所幸的是，版本 2 的 Bash 引入了 `${!variable}` 表示法（参见[例 37-2](https://www.tldp.org/LDP/abs/html/bashver2.html#EX78) 和[例 A-22](https://www.tldp.org/LDP/abs/html/contributed-scripts.html#HASHEX2))），让间接引用更直观。

> Bash 不支持指针运算，这严重限制了间接引用的用处。实际上，在脚本语言中间接引用充其量只是事后的想法。