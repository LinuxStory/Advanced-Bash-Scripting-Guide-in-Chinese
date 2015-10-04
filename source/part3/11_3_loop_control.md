# 11.3 循环控制

> Tournez cent tours, tournez mille tours,
>
> Tournez souvent et tournez toujours . . .
>
> ——保尔·魏尔伦，《木马》

本节介绍两个会影响循环行为的命令。

### break, continue

`break` 和 `continue` 命令[^1]的作用和在其他编程语言中的作用一样。`break` 用来中止（跳出）循环，而 `continue` 则是略过未执行的循环部分，直接进行下一次循环。

样例 11-21. 循环中 `break` 与 `continue` 的作用

```bash
#!/bin/bash

LIMIT=19  # 循环上界

echo
echo "Printing Numbers 1 through 20 (but not 3 and 11)."

a=0

while [ $a -le "$LIMIT" ]
do
 a=$(($a+1))
 
 if [ "$a" -eq 3 ] || [ "$a" -eq 11 ]  # 除了 3 和 11。
 then
   continue      # 略过本次循环的剩余部分。
 fi
 
 echo -n "$a "   # 当 a 等于 3 和 11 时，将不会执行这条语句。
done

# 思考：
# 为什么循环不会输出到20？

echo; echo

echo Printing Numbers 1 through 20, but something happens after 2.

##################################################################

# 用 'break' 代替了 'continue'。

a=0

while [ "$a" -le "$LIMIT" ]
do
 a=$(($a+1))
 
 if [ "$a" -gt 2 ]
 then
   break  # 中止循环。
 fi
 
 echo -n "$a"
done

echo; echo; echo

exit 0
```

`break` 命令接受一个参数。普通的 `break` 命令仅仅跳出其所在的那层循环，而 `break N` 命令则可以跳出其上 N 层的循环。

样例 11-22. 跳出多层循环

```bash
#!/bin/bash
# break-levels.sh: 跳出循环.

# "break N" 跳出 N 层循环。

for outerloop in 1 2 3 4 5
do
  echo -n "Group $outerloop:   "

  # ------------------------------------------
  for innerloop in 1 2 3 4 5
  do
    echo -n "$innerloop "
    
    if [ "$innerloop" -eq 3 ]
    then
      break  # 尝试一下 break 2 看看会发生什么。
             # （它同时中止了内层和外层循环。）
    fi
  done
  # ------------------------------------------

  echo
done

echo

exit 0
```

与 `break` 类似，`continue` 也接受一个参数。普通的 `continue` 命令仅仅影响其所在的那层循环，而 `continue N` 命令则可以影响其上 N 层的循环。

样例 11-23. `continue` 影响外层循环

```bash
#!/bin/bash
# "continue N" 命令可以影响其上 N 层循环。

for outer in I II III IV V           # 外层循环
do
  echo; echo -n "Group $outer: "
  
  # --------------------------------------------------------------------
  for inner in 1 2 3 4 5 6 7 8 9 10  # 内层循环
  do
  
    if [[ "$inner" -eq 7 && "$outer" = "III" ]]
    then
      continue 2  # 影响两层循环，包括“外层循环”。
                  # 将其替换为普通的 "continue"，那么只会影响内层循环。
    fi
    
    echo -n "$inner "  # 7 8 9 10 将不会出现在 "Group III."中。
  done
  # --------------------------------------------------------------------

done

echo; echo

# 思考：
# 想一个 "continue N" 在脚本中的实际应用情况。

exit 0
```

样例 11-24. 真实环境中的 `continue N`

```bash
# Albert Reiner 举出了一个如何使用 "continue N" 的例子：
# ---------------------------------------------------

#  如果我有许多任务需要运行，并且运行所需要的数据都以文件的形
#+ 式存在文件夹中。现在有多台设备可以访问这个文件夹，我想将任
#+ 务分配给这些不同的设备来完成。
#  那么我通常会在每台设备上执行下面的代码：

while true:
do
  for n in .iso.*
  do
    [ "$n" = ".iso.opts" ] && continue
    beta=${n#.iso.}
    [ -r .Iso.$beta ] && continue
    [ -r .lock.$beta ] && sleep 10 && continue
    lockfile -r0 .lock.$beta || continue
    echo -n "$beta: " `date`
    run-isotherm $beta
    date
    ls -alF .Iso.$beta
    [ -r .Iso.$beta ] && rm -rf .lock.$beta
    continue 2
  done
  break
done

exit 0

# 这个脚本中出现的 sleep N 只针对这个脚本，通常的形式是：

while true
do
  for job in {pattern}
  do
    {job already done or running} && continue
    {mark job as running, do job, mark job as done}
    continue 2
  done
  break        # 或者使用类似 `sleep 600` 这样的语句来防止脚本结束。
done

#  这样做可以保证脚本只会在没有任务时（包括在运行过程中添加的任务）
#+ 才会停止。合理使用文件锁保证多台设备可以无重复的并行执行任务（这
#+ 在我的设备上通常会消耗好几个小时，所以我想避免重复计算）。并且，
#+ 因为每次总是从头开始搜索文件，因此可以通过文件名决定执行的先后
#+ 顺序。当然，你可以不使用 'continue 2' 来完成这些，但是你必须
#+ 添加代码去检测某项任务是否完成（以此判断是否可以执行下一项任务或
#+ 终止、休眠一段时间再执行下一项任务）。
```

> ![caution](http://tldp.org/LDP/abs/images/caution.gif) `continue N` 结构不易理解并且可能在一些情况下有歧义，因此不建议使用。

[^1]: 这两个命令是 [内建命令](http://tldp.org/LDP/abs/html/internal.html#BUILTINREF)，而另外的循环命令，如 [`while`](http://tldp.org/LDP/abs/html/loops1.html#WHILELOOPREF) 和 [`case`](http://tldp.org/LDP/abs/html/testbranch.html#CASEESAC1) 则是 [关键词](http://tldp.org/LDP/abs/html/internal.html#KEYWORDREF)。