## 第六章 退出和退出状态 ##

>Bourne shell里存在不明确的地方，人们也会使用它们
>
> — — Chat Ramey


**exit**命令被用来结束脚本，就像C语言一样。它也会返回一个值来传给父进程，父进程会判断是否可用。

每个命令都会返回一个exit状态（有时也叫return状态或者退出码）。成功返回0，如果返回一个非0值，通常情况下都会被认为是一个错误代码。一个编写良好的UNIX命令、程序、和工具都会返回一个0作为退出码来表示成功，虽然偶尔也会有例外。

同样地，脚本中的函数和脚本本身都会返回退出状态。在脚本或者脚本函数中执行的最后的命令会决定退出状态。在脚本中，`exit nnn`命令将会把nnn退出状态传递给shell（nnn 必须是0-255之间的整数）。

> 当一个脚本以不带参数**exit**来结束时，脚本的退出状态就由脚本中最后执行命令来决定（**exit**命令之前）。
>
```bash
#!/bin/bash
COMMAND_1
...
COMMAND_LAST
#将以最后的命令来决定退出状态
exit
```
> **exit**与**exit $?** 和省略**exit** 效果等同。
>
```bash
#!/bin/bash 
COMMAND_1
...
COMMAND_LAST
#将以最后的命令来决定退出状态
exit $?
```
>
```bash
#!/bin/bash
COMMAND_1
...
COMMAND_LAST
#将以最后的命令来决定退出状态
```

$?读取最后执行命令的退出状态。在一个函数返回后，$?给出函数最后执行的那条命令的退出状态。这是Bash给函数一个返回值[^1]的方法。

一个[管道](Chapter3. pipe的介绍)执行后，一个$?给出最后执行的那条命令的退出状态。

脚本终止后，一个命令行的$?给出脚本的退出状态，即在脚本中最后的命令执行后给出退出状态。一般情况下，0为成功，1-255之间的整数为失败。

样例 6-1. exit/exit状态

```bash
#!/bin/bash
echo hello
echo $?    #返回值为0，因为执行成功。

lskdf      #不认识的命令。
echo $?    #返回非0值，因为失败了。

echo
exit 113   #将返回113给shell
           #To verify this, type"echo $?" after script terminates.
           #为了验证这些，在脚本结束的地方使用“echo $?”
```

$?对于测试脚本中的命令的结果特别有用（见Example 12-32和Example 12-17）

>注意： [!](Chapter3. !的介绍) 逻辑非操作,将会反转test命令的结果，并且这会影响exit状态。
>### Example 6-2. 否定一个条件使用！ ###
>     true           #true是shell内建命令，什么事都不做，就是shell返回0
>     echo "exit status of\"true\"=$?"    #0
>     ! true
>     echo "exit status of\"! true\"=$?"  #1
>     #注意："!"需要一个空格。
>     #!true 将导致一个"command not found"错误。
>     #
>     #如果一个命令以'!'开头，那么将使用Bash的历史机制，就是显示这个命令被使用的历史。
>     
>     true
>     !true
>     #这次就没有错误了。
>     #它不过时重复之前的命令（true）。
>     
>     #==================================================================#
>     #前面的_pipe_和!一起使用，将改变返回的退出状态。
>     ls | bogus_command      #bash: bogus_command: command not found
>     echo $?                 #127
>     
>     ! ls | bogus_command    #bash: bogus_command:command not found
>     echo $?                 #0
>     #注意："!"不改变管道的执行。
>     #只改变退出状态。
>     #==================================================================#
>     
>     #感谢Stéphane Chazelas 和 Kristopher Newsome.

### 注意事项： ###

特定的退出码都有预定的含义（见附录D），用户不应该在自己的脚本中指定它。

[^1]: 在函数没有[return](http://tldp.org/LDP/abs/html/complexfunct.html#RETURNREF)来结束这个函数的情况下。