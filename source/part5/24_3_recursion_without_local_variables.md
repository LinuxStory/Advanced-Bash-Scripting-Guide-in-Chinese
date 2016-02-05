# 24.3 不使用局部变量的递归
即使不适用局部变量，函数也可以递归的调用自身。

例子24-16. 斐波那契序列
```
#!/bin/bash
# fibo.sh : 斐波那契序列 (递归)
# 作者: M. Cooper
# License: GPL3

# ----------算法--------------
# Fibo(0) = 0
# Fibo(1) = 1
# else
#   Fibo(j) = Fibo(j-1) + Fibo(j-2)
# ---------------------------------

MAXTERM=15       # 要产生的计算次数。
MINIDX=2         # 如果下标小于2，那么 Fibo(idx) = idx.

Fibonacci ()
{
    idx=$1   # 不需要是局部变量，为什么？
    if [ "$idx" -lt "$MINIDX" ]
    then
        echo "$idx"  # 前两个下标是0和1 ... 从上面的算法可以看出来。
    else
        (( --idx ))  # j-1
        term1=$( Fibonacci $idx )   #  Fibo(j-1)
        (( --idx ))  # j-2
        term2=$( Fibonacci $idx )   #  Fibo(j-2)
        echo $(( term1 + term2 ))
    fi
    #  一个丑陋的实现
    #  C语言里，一个更加优雅的斐波那契递归实现
    #+ 是一个简单的只需要7-10代码的算法翻译。
}

for i in $(seq 0 $MAXTERM)
do  # 计算 $MAXTERM+1 次.
    FIBO=$(Fibonacci $i)
    echo -n "$FIBO "
done
# 0 1 1 2 3 5 8 13 21 34 55 89 144 233 377 610
# 要花费一段时间，不是么？ 一个递归脚本是有些慢的。

echo

exit 0
```

例子 24-17. 汉诺塔
```
#! /bin/bash
#
# 汉诺塔
# Bash script
# Copyright (C) 2000 Amit Singh. All Rights Reserved.
# http://hanoi.kernelthread.com
#
# 在 Bash version 2.05b.0(13)-release下通过测试.
# 同样在Bash3.x版本下工作正常。
#
#  在 "Advanced Bash Scripting Guide" 一书中使用
#+ 经过了脚本作者的许可。
#  ABS的作者对脚本进行了轻微的修改和注释。
#=================================================================#
#  汉诺塔是由Edouard Lucas,提出的数学谜题， 
#+  他是19世纪的法国数学家。
#
# 有三个直立的柱子竖在地面上。
# 第一个柱子上有一组盘子套在上面。
# 这些盘子是平的，中间有孔，
#+ 可以套在柱子上面。
# 这些盘子的直径不同，它们从下而上， 
#+ 按照尺寸递减的顺序摆放。
# 也就是说，最小的在最上边，最大的在最下面。
#
# 现在的任务是要把这组盘子  
#+ 从一个柱子上全部搬到另一个柱子上 
# 你每次只能将一个盘子从一个柱子移到另一个柱子上。
# 你也可以把盘子从其他的柱子上移回到原来的柱子上。
# 你只能把小的盘子放到大的盘子上。
#+ 反过来就不行。
# 切记，绝对不能把大盘子放到小盘子的上面。
# 如果盘子的数量比较少，那么移不了几次就能完成。
#+ 但是随着盘子数量的增加，
#+ 移动次数几乎成倍的增长，
#+ 而且移动的“策略”也会变得越来越复杂。
#
# 想了解更多信息的话，请访问http://hanoi.kernelthread.com
#+ 或者 pp. 186-92 of _The Armchair Universe_ by A.K. Dewdney.
#
#
#           ...             ...         ...
#           | |             | |         | |
#          _|_|_            | |         | |
#         |_____|           | |         | |
#        |_______|          | |         | |
#       |_________|         | |         | |
#      |___________|        | |         | |
#     |             |       | |         | |
# .--------------------------------------------------------------. 
# |**************************************************************| 
#            #1              #2          #3
# #=================================================================#

E_NOPARAM=66  # 没有参数传给脚本。
E_BADPARAM=67 # 传给脚本的盘子个数不符合要求。
Moves=        # 保存移动次数的全局变量。
              # 这里修改了原来的脚本。

dohanoi() {   # 递归函数
    case $1 in
    0)
        ;;
    *)
        dohanoi "$(($1-1))" $2 $4 $3
        echo move $2 "-->" $3
        ((Moves++))          # 这里修改了原来的脚本。
        dohanoi "$(($1-1))" $4 $3 $2
        ;;
    esac 
}

case $# in
    1) case $(($1>0)) in     # 至少要有一个盘子
        1)  # Nested case statement.
            dohanoi $1 1 3 2
            echo "Total moves = $Moves"     # 2^n - 1, where n = # of disks.
            exit 0;
            ;; 
        *)    
            echo "$0: illegal value for number of disks";
            exit $E_BADPARAM;
            ;;
        esac ;;
    *)
        echo "usage: $0 N"
        echo "       Where \"N\" is the number of disks."
        exit $E_NOPARAM;
        ;;
    esac

# 练习:
# ---------
# 1) 这个位置以下的代码会不会执行？ 
#    为什么不(容易)
# 2) 解释一下这个 "dohanoi" 函数的运行原理.
#    (比较难 可以参考上面的Dewdney 的引用)
```