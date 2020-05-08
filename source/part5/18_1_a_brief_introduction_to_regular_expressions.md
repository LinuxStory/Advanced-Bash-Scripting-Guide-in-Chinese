# 18.1 正则表达式简介
正则表达式是一系列的字符串。这些包含超过其字面含义的字符串被称之为元字符。例如，一个符号前面的引用符代表一个人的言语能力，或者按照上面的说法，代表着meta-meaning[[1]](http://tldp.org/LDP/abs/html/x17129.html#FTN.AEN17134)。正则表达式是一组字符串和（或者）一组匹配（特定的）模式的元字符。

一个正则表达式包含下面的一个或多个选项：

- 一组字符串。这是仅仅表示字面意思的字符串。最简单形式的正则表达式仅仅包含一组字符串。
- 一个锚字符。锚节点指定了正则表达式在一行文本中的匹配位置。例如，^和$就是锚字符。
- 修饰符。修饰符扩展或者限定（修改）了正则表达式在文本中的匹配范围。修饰符包括星号、方括号和反斜线。

正则表达式的主要用在文本搜索和字符串操作。一个正则表达式匹配单个字符或者一组字符 -- 一系列的字符或者字符串的一部分。

- 星号 * 匹配前面的子表达式任意次，包括0次

	"1133*"匹配"11"加一个或多个"3"：113，1133，1133333，及以后
- 点号 . 匹配任意字符，除了新的一行[[2]](http://tldp.org/LDP/abs/html/x17129.html#AEN17189)

	"13."匹配"13"加至少一个字符（包括空格）：1133，11333，但不是13（缺少额外的字符）  
	参见例子[16-18](http://tldp.org/LDP/abs/html/textproc.html#CWSOLVER)，展示单字符匹配
- 脱字符 ^ 匹配行的起始位置，但有时候会根据上下文环境匹配其相反的意义（译者注：例如[^a]匹配任意一个非a的字符）
- 美元符 $ 匹配行的结束位置

	"XXX$"匹配行尾处的"XXX"  
	"^$"匹配空行
- 方括号 [...] 匹配所包含的任意一个字符

	"[xyz]"匹配x、y或z中的任意一个字符  
	"[c-n]"匹配c到n之间的任意一个字符  
	"[B-Pk-y]"匹配B到P和k到y之间任意一个字符  
	"[a-z0-9]"匹配任意一个小写字符和任意一个数字  
	"[^b-d]"匹配任意一个不在b到d之间的字符。这是一个很好的例子，展示了"^"的匹配了正则表达式的反义（类似在其他环境下的"!"符号所起的作用）  
	组合一连串的用方括号括起来的字符能匹配非常多的词组模式。"[Yy][Ee][Ss]"匹配yes、Yes、YES、yEs等等。"[0-9][0-9][0-9]-[0-9][0-9]-[0-9][0-9][0-9][0-9]"匹配任何一个社会保险号
- 反斜线 \ 转义一个特殊字符，意味着这个字符被解释为字面意义（因此不再包含特殊意思）

	"\$"表示回它的字面意义"$"，而不是它原本在正则表达式中代表行尾的意义。同样，"\\"表示字面意义"\"
- 转义后的尖括号 \<..\> 代表词组的边界

	尖括号必须进行转义，否则它们就代表其字面意义  
	"\<the\>"匹配词组"the"，而不是词组"them," "there," "other,"等等

```shell
bash$ cat textfile
This is line 1, of which there is only one instance.
 This is the only instance of line 2.
 This is line 3, another line.
 This is line 4.


bash$ grep 'the' textfile
This is line 1, of which there is only one instance.
 This is the only instance of line 2.
 This is line 3, another line.


bash$ grep '\<the\>' textfile
This is the only instance of line 2.
```

唯一判断一个特定的正则表达式是否有效的方法就是测试它。
```shell
测试文件: tstfile                          # No match.
                                            # No match.
运行 grep "1133*"  tstfile                  # Match.
                                            # No match.
                                            # No match.
This line contains the number 113.          # Match.
This line contains the number 13.           # No match.
This line contains the number 133.          # No match.
This line contains the number 1133.         # Match.
This line contains the number 113312.       # Match.
This line contains the number 1112.         # No match.
This line contains the number 113312312.    # Match.
This line contains no numbers at all.       # No match.
```
```shell
bash$ grep "1133*" tstfile
Run   grep "1133*"  on this file.           # Match.
 This line contains the number 113.          # Match.
 This line contains the number 1133.         # Match.
 This line contains the number 113312.       # Match.
 This line contains the number 113312312.    # Match.
```   


注解

[[1]](http://tldp.org/LDP/abs/html/x17129.html#FTN.AEN17134) 元意义指的是一个词组或者表达式在更高层次的抽象上的意义。例如，正则表达式的字面意思就是所有人接受其用法的普通表达式。元意义则完全不同，正如在本章最终讨论的那样。
[[2]](http://tldp.org/LDP/abs/html/x17129.html#AEN17189)
Since sed, awk, and grep process single lines, there will usually not be a newline to match. In those cases where there is a newline in a multiple line expression, the dot will match the newline.

```shell
#!/bin/bash

sed -e 'N;s/.*/[&]/' << EOF   # Here Document
line1
line2
EOF
# OUTPUT:
# [line1
# line2]



echo

awk '{ $0=$1 "\n" $2; if (/line.1/) {print}}' << EOF
line 1
line 2
EOF
# OUTPUT:
# line
# 1


# Thanks, S.C.

exit 0
```