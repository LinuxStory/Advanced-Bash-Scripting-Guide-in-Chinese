# 32 调试
<blockquote class="blockquote-center">
调试代码要比写代码困难两倍。因此，你写代码时越多的使用奇技淫巧（自做聪明），顾名思义，你越难以调试它。    --Brian Kernighan
</blockquote>

Bash shell中不包含内置的debug工具，甚至没有调试专用的命令和结构。当调试非功能脚本，产生语法错误或者有错别字时，往往是无用的错误提示消息。

例子 32-1. 一个错误脚本

```
#!/bin/bash
# ex74.sh

# 这是一个错误脚本，但是它错在哪？

a=37

if [$a -gt 27 ]
then
  echo $a
fi  

exit $?   # 0! 为什么?
```
脚本的输出:

```
./ex74.sh: [37: command not found
```
上边的脚本究竟哪错了(提示: 注意if的后边)

例子 32-2. 缺少关键字

```
#!/bin/bash
# missing-keyword.sh
# 这个脚本会提示什么错误信息？

for a in 1 2 3
do
  echo "$a"
# done     #所需关键字'done'在第8行被注释掉.

exit 0     # 将不会在这退出!

#在命令行执行完此脚本后
输入：echo $?    
输出：2
```

脚本的输出:

```
missing-keyword.sh: line 10: syntax error: unexpected end of file
```
注意, 其实不必参考错误信息中指出的错误行号. 这行只不过是Bash解释器最终认定错误的地方.
出错信息在报告产生语法错误的行号时, 可能会忽略脚本的注释行.
如果脚本可以执行, 但并不如你所期望的那样工作, 怎么办? 通常情况下, 这都是由常见的逻辑错误所
产生的.
