# 第六章 退出与退出状态

> Bourne shell里存在不明确的地方，人们也会使用它们。
>
> —— Chat Ramey

跟C语言一样，`exit` 命令被用来结束脚本。同时，它也会返回一个父进程可用的值给父进程。

每个命令都会返回一个退出状态（exit status），有时也叫做返回状态（return status）或退出码（exit code）。成功返回0，如果返回一个非0值，通常情况下都会被认为是一个错误代码。一个运行状态良好的UNIX命令、程序和工具在正常退出后都会返回一个0退出码，虽然偶尔也会有些例外。

同样地，脚本中的函数和脚本本身都会返回退出状态。在脚本或者脚本函数中执行的最后的命令会决定退出状态。在脚本中，`exit nnn` 命令将会把nnn退出状态传递给shell（nnn 必须是 0-255 之间的整数）。

> ![note](http://tldp.org/LDP/abs/images/note.gif) 当一个脚本以不带参数的 `exit` 来结束时，脚本的退出状态就由脚本中最后执行命令来决定（`exit` 命令之前）。
>
```bash
#!/bin/bash
>
COMMAND_1
>
...
>
COMMAND_LAST
>
# 将以最后的命令来决定退出状态
>
exit
```

> `exit`，`exit $?` 以及省略 `exit` 效果等同。
>
```bash
#!/bin/bash 
>
COMMAND_1
>
...
>
COMMAND_LAST
>
#将以最后的命令来决定退出状态
>
exit $?
```

>
```bash
#!/bin/bash
>
COMMAND_1
>
...
>
COMMAND_LAST
>
#将以最后的命令来决定退出状态
```

`$?` 读取最后执行命令的退出状态。在一个函数返回后，`$?` 给出函数最后执行的那条命令的退出状态。这是Bash给函数一个返回值的方法。[^1]

在[管道](http://tldp.org/LDP/abs/html/special-chars.html#PIPEREF)执行后，`$?` 给出最后执行的那条命令的退出状态。

在脚本终止后，命令行下的 `$?` 给出脚本的退出状态，即在脚本中最后的命令执行后给出退出状态。一般情况下，0为成功，1-255之间的整数为失败。

样例 6-1. 退出与退出状态

```bash
#!/bin/bash

echo hello
echo $?    # 返回值为0，因为执行成功。

lskdf      # 不认识的命令。
echo $?    # 返回非0值，因为失败了。

echo

exit 113   # 将返回113给shell
           # 为了验证这些，在脚本结束的地方使用“echo $?”

#  按照惯例，'exit 0' 意味着执行成功，
#+ 非0意味着错误或者异常情况。
#  查看附录章节“特殊意义的退出码”
```

`$?` 对于测试脚本中的命令的结果特别有用（查看样例 16-35和样例 16-20）。

> ![note](http://tldp.org/LDP/abs/images/note.gif) 逻辑非操作符 [!](http://tldp.org/LDP/abs/html/special-chars.html#NOTREF) 将会反转测试或命令的结果，并且这将会影响退出状态。
> 样例 6-2. 否定一个条件使用!
>
```bash
true    # true 是 shell 内建命令。
echo "exit status of \"true\" = $?"     # 0
>
! true
echo "exit status of \"! true\" = $?"   # 1
# 注意在命令之间的 "!" 需要一个空格。
# !true 将导致一个"command not found"错误。
#
# 如果一个命令以'!'开头，那么将调用 Bash 的历史机制，显示这个命令被使用的历史。
>
true
!true
# 这次就没有错误了，但是同样也没有反转。
# 它不过是重复之前的命令（true）。
>
>
# ============================================================ #
# 在 _pipe_ 前使用 ! 将改变返回的退出状态。
ls | bogus_command      #bash: bogus_command: command not found
echo $?                 #127
>
! ls | bogus_command    #bash: bogus_command:command not found
echo $?                 #0
# 注意 ! 不改变管道的执行。
# 只改变退出状态。
#============================================================  #
>
# 感谢 Stéphane Chazelas 和 Kristopher Newsome。
```

> ![caution](http://tldp.org/LDP/abs/images/caution.gif) 某些特定的退出码具有一些特定的[保留含义](http://tldp.org/LDP/abs/html/exitcodes.html#EXITCODESREF)，用户不应该在自己的脚本中使用定义它们。

[^1]: 在函数没有[return](http://tldp.org/LDP/abs/html/complexfunct.html#RETURNREF)来结束这个函数的情况下。