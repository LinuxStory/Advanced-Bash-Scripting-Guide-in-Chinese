# 第十五章 内建命令

内建命令 `builtin` 是包含在 Bash 工具集中的命令，又写作 `built in`。使用内建命令通常出于性能原因或是需要直接存取 shell 内部变量的考量。内建命令执行速度比外部命令的执行速度快，因为外部命令通常需要派生[^1]出一个单独的进程执行。

命令或 shell 本身启动或生成一个子进程用于执行任务的操作被称为派生 `forking`。新生成的进程被称作子进程，而派生出子进程的进程被称作父进程。当子进程在执行任务时，父进程也仍在运行。

需要注意的是，父进程可以获取子进程的进程ID，并能传递参数给子进程，但反之则不行。[该机制会产生一些难以捉摸的问题。]()

#### 样例 15-1. 脚本生成多个自身实例

```bash
#!/bin/bash
# spawn.sh


PIDS=$(pidof sh $0)  # 脚本的多个实例的进程ID。
P_array=( $PIDS )    # 将进程ID放置到数组中（为什么？）。
echo $PIDS           # 显示父进程和子进程的进程ID。
let "instances = $(#P_array[*]} - 1"  # 元素数量减1。
                                      # 为什么要减1？
echo "$instances instance(s) of this script running."
echo "[Hit Ctl-C to exit.]"; echo


sleep 1              # 闲置等待。
sh $0                # 再运行一次。

exit 0               # 这不是必须的；脚本永远不会执行到这里。
                     # 为什么不是必须的？

#  在键入 Ctl-C 退出脚本后，
#+ 是否所有生成的脚本实例都会终止？
#  如果是，为什么？

# 注意：
# ----
# 注意不要长时间运行这个脚本。
# 它最终会占用大量的系统资源。

#  你认为让脚本生成大量的自身实例
#+ 是否是一个可取的编写脚本的技巧？
#  为什么？
```

通常来说，Bash 的内建命令不会在脚本中通过派生子进程来执行。而在脚本中调用外部系统命令或筛选器通常需要派生子进程。

一些内建命令可能与系统命令重名，但是这些都是在 Bash 内部重新实现后的命令。例如 Bash 中的 `echo` 命令与系统命令 `/bin/echo` 的功能基本一致，但是它们本质上并不一样。

```bash
#!/bin/bash

echo "This line uses the \"echo\" builtin."
/bin/echo "This line uses the /bin/echo system command."
```

关键词 `keyword` 是保留使用的词汇、标记或运算符。关键词在 shell 中具有特殊意义，是 shell 语法的组成部分。例如 `for`，`while`，`do` 和 `!` 都是关键词。关键词与 [内建命令]() 相似，它们都是硬编码到 Bash 中的。但是关键词本身不是命令，而是构成命令的子单位[^2]。

## 输入与输出 I/O

### echo

打印一个表达式或变量到标准输出 `stdout`（参考 [样例 4-1]()）。

```bash
echo Hello
echo $a
```

`echo` 命令需要 `-e` 选项来打印转义字符。参考 [样例 5-2]()。

通常情况下，每一个 `echo` 命令都会在最后打印一个终端换行符，但使用 `-n` 选项可以禁止这个行为。

{% hint style="info" %}

`echo` 可通过管道被用于给一系列命令提供值。

```bash
if echo "$VAR" | grep -q txt   # if [[ $VAR = *txt* ]]
then
  echo "$VAR contains the substring sequence \"txt\""
fi
```

{% endhint %}

{% hint style="info" %}

`echo` 与 [命令替换]() 结合可以用于给变量赋值。

``a=`echo "HELLO" | tr A-Z a-z` ``

参考 [样例 16-22]()，[样例 16-3]()，[样例 16-47]() 和 [样例 16-48]()。

{% endhint %}

需要注意的是 ``echo `command` `` 会删除 `command` 中生成的所有换行符。

内部字段分隔符 [`$IFS`]()



[^1]: Footnote Placeholder
[^2]: Footnote Placeholder
