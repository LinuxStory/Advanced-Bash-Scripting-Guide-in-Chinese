# 9.3 `$RANDOM`：生成随机数

> 任何试图通过确定性方法生成随机数的行为都是在犯罪。
> 
> —— 约翰·冯·诺伊曼

`$RANDOM` 是 Bash 中用来生成 0 至 32767 之间随机整数[^1]的一个内置 [函数]()（而非常量）。其**不应**被用于生成密钥。

#### 样例 9-11. 生成随机数

```bash
#!/bin/bash

# $RANDOM 每一次调用都会返回一个随机的不同的整数。
# 随机数的标称范围为 0 - 32767（16位有符号整型）。

MAXCOUNT=10
count=1

echo
echo "$MAXCOUNT random numbers:"
echo "-----------------"
while [ "$count" -le $MAXCOUNT ]      # 生成 10 ($MAXCOUNT) 个随机整数。
do
  number=$RANDOM
  echo $number
  let "count += 1"  # 增加计数。
done
echo "-----------------"

# 如果你需要一个小于指定上界的随机数，可以使用 'modulo' 操作符。
# 该操作符可以返回除法后的余数。

RANGE=500

echo

number=$RANDOM
let "number %= $RANGE"
#           ^^
echo "Random number less than $RANGE --- $number"

echo



#  如果你需要生成的随机数大于一个指定的下界，
#+ 可以增加一步判断，判别并丢弃所有小于下界的数。

FLOOR=200

number=0   # 初始化
while [ "$number" -le $FLOOR ]
do
  number=$RANDOM
done
echo "Random number greater than $FLOOR --- $number"
echo

   # 现在来看一种可以代替上面循环的更简单的方式，也就是
   #       let "number = $RANDOM + $FLOOR"
   # 该方式可以不使用 while 循环，效率更高。
   # 但是，该方法可能会产生一些问题，是什么呢？



# 通过结合上面的两种方法，可以获得一个特定范围内的随机数。
number=0   # 初始化
while [ "$number" -le $FLOOR ]
do
  number=$RANDOM
  let "number %= $RANGE"  # 将 $number 缩小至 $RANGE 的范围内。
done
echo "Random number between $FLOOR and $RANGE --- $number"
echo



# 生成二元选择值，即真(true)或假(false)。
BINARY=2
T=1
number=$RANDOM

let "number %= $BINARY"
#  如果使用    let "number >>= 14"    可以获得更优的随机分布
#+ （除了最低位，其余二进制位都右移）。
if [ "$number" -eq $T ]
then
  echo "TRUE"
else
  echo "FALSE"
fi

echo


# 扔一个骰子。
SPOTS=6   # 模 6 的余数范围为 0 - 5。
          # 然后加 1 就可以得到期望的范围 1 - 6。
          # 感谢 Paulo Marcel Coelho Aragao 简化了代码。
die1=0
die2=0
# 如果设置 SPOTS=7 就可以不用加 1 得到值。这是不是一种更好的方法，为什么？

# 为了保证公平，独立的投每一个骰子。

    let "die1 = $RANDOM % $SPOTS + 1" # 投第一个骰子。
    let "die2 = $RANDOM % $SPOTS + 1" # 投第二个骰子。
    #  哪一种运算符有更高的优先级，
    #+ 取余(%)还是加法(+)？


let "throw = $die1 + $die2"
echo "Throw of the dice = $throw"
echo


exit 0
```

#### 样例 9-12. 从牌组中随机选牌

```bash
#!/bin/bash
# pick-card.sh

# 该样例演示了如何从数组中随机选择元素。


# 随机选择任意一张牌。

Suites="Clubs
Diamonds
Hearts
Spades"

Denominations="2
3
4
5
6
7
8
9
10
Jack
Queen
King
Ace"

# 注意一个变量占了多行。


suite=($Suites)                # 读入数组变量。
denomination=($Denominations)

num_suites=${#suite[*]}        # 数组中的元素数量。
num_denominations=${#denomination[*]}

echo -n "${denomination[$((RANDOM%num_denominations))]} of "
echo ${suite[$((RANDOM%num_suites))]}


# $bozo sh pick-cards.sh
# Jack of Clubs


# 感谢 jipe 指出可以用 $RANDOM 随机选牌。
exit 0
```

#### Example 9-13. 模拟布朗运动

```bash
#!/bin/bash
# brownian.sh
# 作者：Mendel Cooper
# 发布日期：10/26/07
# 开源协议：GPL3

#  ----------------------------------------------------------------
#  该脚本模拟了布朗运动。
#+ 布朗运动是指微小粒子受到流体粒子随机碰撞，
#+ 而在流体中做的无规则随机运动。
#+ 也就是俗称的“醉汉走路”。

#  布朗运动也可以被视作是一个简化的高尔顿板。
#+ 高尔顿板是一个有着交错排列的钉子的倾斜板子，
#+ 每次可以从中向下滚动一堆石子。
#+ 在板子底端是一排槽位，
#+ 石子最后会落在槽位中。
#  把它想象成一个简单的弹珠游戏就可以了。
#  当运行这个脚本之后，
#+ 你就会发现大部分的石子都聚集在中间的槽位里。
#+ 这与预期的二项分布相符。
#  作为模拟高尔顿板的程序，
#+ 脚本忽略了许多参数，
#+ 例如板子的倾斜角度、石子滚动的摩擦系数、
#+ 冲击角度以及钉子的弹性系数等等。
#  忽略的这些参数能够在多大程度上影响模拟的精度？
#  -------------------------------------------------------------

PASSES=500            #  粒子作用数 / 石子数。
ROWS=10               #  碰撞数 / 每一排钉子的数量。
RANGE=3               #  $RANDOM 的输出范围为 0 - 2。
POS=0                 #  滚落左侧或是右侧。
RANDOM=$$             #  将脚本的进程 ID 作为
                      #+ 生成随机数的种子。

declare -a Slots      # 用于储存落入每一个槽位的石子数量。
NUMSLOTS=21           # 底部槽位的数量。


Initialize_Slots () { # 初始化数组。
for i in $( seq $NUMSLOTS )
do
  Slots[$i]=0
done

echo                  # 在正式模拟开始之前先输出空行。
  }


Show_Slots () {
echo; echo
echo -n " "
for i in $( seq $NUMSLOTS )   # 更精致地输出数组中的所有元素。
do
  printf "%3d" ${Slots[$i]}   # 每个结果都占三个字符的宽度。
done

echo # 槽位：
echo " |__|__|__|__|__|__|__|__|__|__|__|__|__|__|__|__|__|__|__|__|__|"
echo "                                ||"
echo #  需要注意的是，如果任意一个槽位中石子的数量超过 99，
     #+ 将会打乱整个程序的显示效果。
     #  如果只运行 500 次通常可以避免这个问题。
  }


Move () {              # 将一个单位左移、右移或保持原地不动。
  Move=$RANDOM         # $RANDOM 到底有多随机？让我们看看...
  let "Move %= RANGE"  # 标准化至范围 0 - 2。
  case "$Move" in
    0 ) ;;                   # 什么也不做，也就是原地不动。
    1 ) ((POS--));;          # 左移。
    2 ) ((POS++));;          # 右移。
    * ) echo -n "Error ";;   # 出现异常！（应该永远不会发生）
  esac
  }


Play () {                    # 模拟单次运行（内部循环）。
i=0
while [ "$i" -lt "$ROWS" ]   # 每一排钉子经过且仅经过一次石子。
do
  Move
  ((i++));
done

SHIFT=11                     # 为什么是 11 而不是 10？
let "POS += $SHIFT"          # 将原点移到中间。
(( Slots[$POS]++ ))          # 调试：echo $POS

# echo -n "$POS "

  }


Run () {                     # 外部循环。
p=0
while [ "$p" -lt "$PASSES ]
do
  Play
  (( p++ ))
  POS=0                      # 重置为 0。为什么要这么做？
done
  }


# --------------
# main ()
Initialize_Slots
Run
Show_Slots
# --------------

exit $?

#  练习：
#  ---------
#  1) 将结果显示为一张直方图，
#+    或者是一张散点图。
#  2) 修改脚本，使用 /dev/urandom 提到 $RANDOM。
#     这会使脚本更加的随机化么？
#  3) 当每一个石子落下的时候，
#+    尝试添加一些动画效果。
```

Jipe 提供了一些生成指定范围内随机数的方法。

```bash
#  生成范围为 6 到 30 的随机数。
   rnumber=$((RANDOM%25+6))

#  生成范围为 6 到 30 的随机数，
#+ 并且该随机数能被 3 整除。
   rnumber=$(((RANDOM%30/3+1)*3))

#  需要注意这种方法并不是在所有情况下都能起效。
#  会在 $RANDOM%30 为 0 时失效。

#  Frank Wang 建议可以换用下面的方法：
   rnumber=$(( RANDOM%27/3*3+6 ))
```

Bill Gradwohl 提出了一种改良后的仅适用于正数的公式。

```bash
rnumber=$(((RANDOM%(max-min+divisibleBy))/divisibleBy*divisibleBy+min))
```

Bill 在这还给出了一个生成指定范围内随机数的通用函数。

#### 样例 9-14. 指定范围随机数

```bash
#!/bin/bash
# random-between.sh
# 生成指定范围内的随机数。
# 本书作者在 Bill Gradwhol 所提供的脚本的基础上作了些细微修改。
# Anthony Le Clezio 修正了 187 行和 189 行。
# 本书被授权使用该脚本。


randomBetween() {
   #  生成一个范围在 $min 和 $max 之间，
   #+ 并且能被 $divisibleBy 整除的
   #+ 随机正数或负数。
   #  返回的随机数遵循合理的随机分布。
   
   #  Bill Gradwohl - Oct 1, 2003
   
   syntax() {
   # 嵌套函数。
      echo
      echo    "Syntax: randomBetween [min] [max] [multiple]"
      echo
      echo -n "Expects up to 3 passed parameters, "
      echo    "but all are completely optional."
      echo    "min is the minimum value"
      echo    "max is the maximum value"
      echo -n "multiple specifies that the answer must be "
      echo     "a multiple of this value."
      echo    "    i.e. answer must be evenly divisible by this number."
      echo
      echo    "If any value is missing, defaults area supplied as: 0 32767 1"
      echo -n "Successful completion returns 0, "
      echo      "unsuccessful completion returns"
      echo    "function syntax and 1."
      echo -n "The answer is returned in the global variable "
      echo    "randomBetweenAnswer"
      echo -n "Negative values for any passed parameter are "
      echo    "handled correctly."
   }
   
   local min=${1:-0}
   local max=${2:-32767}
   local divisibleBy=${3:-1}
   # 考虑到没有给函数传参的情况，给变量设置默认值。
   
   local x
   local spread
   
   # 确保 divisibleBy 的值为正数。
   [ ${divisibleBy} -lt 0 ] && divisibleBy=$((0-divisibleBy))
   
   # 合规校验。
   if [ $# -gt 3 -o ${divisibleBy} -eq 0 -o  ${min} -eq ${max} ]; then
      syntax
      return 1
   fi
   
   # 检查 min 和 max 的值是否颠倒。
   if [ ${min} -gt ${max} ]; then
      # 交换它们。
      x=${min}
      min=${max}
      max=${x}
   fi
   
   #  如果 min 值本身不能被 $divisibleBy 整除，
   #+ 则将其修正到范围内。
   if [ $((min/divisibleBy*divisibleBy)) -ne ${min} ]; then
      if [ ${min} -lt 0 ]; then
         min=$((min/divisibleBy*divisibleBy))
      else
         min=$((((min/divisibleBy)+1)*divisibleBy))
      fi
   fi
   
   #  如果 max 值本身不能被 $divisibleBy 整除，
   #+ 则将其修正到范围内。
   if [ $((max/divisibleBy*divisibleBy)) -ne ${max} ]; then
      if [ ${max} -lt 0 ]; then
         max=$((((max/divisibleBy)-1)*divisibleBy))
      else
         max=$((max/divisibleBy*divisibleBy))
      fi
   fi

   #  ---------------------------------------------------------------------
   #  接下来开始真正的内容。
   
   #  需要注意的是，为了得到端点间合理的随机分布，
   #+ 随机数的取值范围应是 0 至 abs(max-min)+divisibleBy，
   #+ 而不是简单的 abs(max-min)+1。
   
   #  少量的增长将会带来端点间
   #+ 合理的随机分布。
   
   #  将公式修改为使用 abs(max-min)+1 仍然可以得到正确的答案，
   #+ 但是获得的这些随机数的随机性是有缺陷的，
   #+ 因为这种情况下返回的端点值 ($min 和 $max) 的次数远少于
   #+ 使用正确公式时所返回的次数。
   #  ---------------------------------------------------------------------

   spread=$((max-min))
   #  Omair Eshkenazi 指出在这里没有必要进行校验，
   #+ 因为 max 和 min 的值已经被交换了。
   [ ${spread} -lt 0 ] && spread=$((0-spread))
   let spread+=divisibleBy
   randomBetweenAnswer=$(((RANDOM%spread)/divisibleBy*divisibleBy+min))
   
   return 0
   
   #  但是 Paulo Marcel Coelho Aragao 指出
   #+ 当 $max 和 $min 不能被 $divisibleBy 整除时，
   #+ 该公式就会失效。
   #
   #  他建议替换为下面的公式：
   #    rnumber = $(((RANDOM%(max-min+1)+min)/divisibleBy*divisibleBy))
   
}

# 接下来测试函数。
min=-14
max=20
divisibleBy=3


#  循环执行足够多次数的函数，生成包含这些随机数的数组，
#+ 然后校验数组中是否包含了端点范围内的每一个数字。

declare -a answer
minimum=${min}
maximum=${max}
   if [ $((minimum/divisibleBy*divisibleBy)) -ne ${minimum} ]; then
      if [ ${minimum} -lt 0 ]; then
         minimum=$((minimum/divisibleBy*divisibleBy))
      else
         minimum=$((((minimum/divisibleBy)+1)*divisibleBy))
      fi
   fi
   
   
   #  如果 max 值本身不能被 $divisibleBy 整除，
   #+ 则将其修正到范围内。
   
   if [ $((maximum/divisibleBy*divisibleBy)) -ne ${maximum} ]; then
      if [ ${maximum} -lt 0 ]; then
         maximum=$((((maximum/divisibleBy)-1)*divisibleBy))
      else
         maximum=$((maximum/divisibleBy*divisibleBy))
      fi
   fi


#  需要保证数组的下标只能为正数，
#+ 因此这里需要通过位移来保证
#+ 结果为正。

disp=$((0-minimum))
for ((i=${minimum}; i<=${maximum}; i+=divisibleBy)); do
   answer[i+disp]=0
done


# 现在开始循环执行函数以获得大量的随机数。
loopIt=1000   #  脚本的作者建议使用 100000，
              #+ 但是这会花费大量的时间。
              
for ((i=0; i<${loopIt}; i++)); do

   #  注意，我们在这里颠倒了 min 和 max 的值，
   #+ 为的是校验函数在这种情况下是否能正常执行。
   
   randomBetween ${max} ${min} ${divisibleBy}
   
   # 如果获得了非预期的答案，则报错。
   [ ${randomBetweenAnswer} -lt ${min} -o ${randomBetweenAnswer} -gt ${max} ] \
   && echo MIN or MAX error - ${randomBetweenAnswer}!
   [ $((randomBetweenAnswer%${divisibleBy})) -ne 0 ] \
   && echo DIVISIBLE BY error - ${randomBetweenAnswer}!
   
   # 保存统计结果。
   answer[randomBetweenAnswer+disp]=$((answer[randomBetweenAnswer+disp]+1))
done



# 校验最终结果。

for ((i=${minimum}; i<=${maximum}; i+=divisibleBy)); do
   [ ${answer[i+disp]} -eq 0 ] \
   && echo "We never got an answer of $i." \
   || echo "${i} occurred ${answer[i+disp]} times."
done

exit 0
```

那么 `$RANDOM` 到底有多随机？最好的测试方法就是写一个脚本跟踪由 `$RANDOM` 生成的随机数的分布。接下来让我们多投几次由 `$RANDOM` 做的骰子...

#### 样例 9-15. 用 `RANDOM` 投骰子

```bash
#!/bin/bash
# RANDOM 有多随机？

RANDOM=$$       # 用脚本的进程 ID 重置随机数生成器种子。

PIPS=6          # 骰子有 6 个点。
MAXTHORWS=600   # 如果你没有更好消磨时间的办法，就增加这个值。
                # 投骰子的次数。

ones=0          #  必须初始化计数器的值为 0，
twos=0          #+ 因为未初始化的变量的值为 null 而非 0。
threes=0
fours=0
fives=0
sixes=0

print_result ()
{
echo
echo "ones =   $ones"
echo "twos =   $twos"
echo "threes = $threes"
echo "fours =  $fours"
echo "fives =  $fives"
echo "sixes =  $sixes"
echo
}

update_count()
{
case "$1" in
  0) ((ones++));;   # 因为骰子没有 0 点，所以这个其实对应的是 1 点。
  1) ((twos++));;   # 这个对应 2 点。
  2) ((threes++));; # 以此类推。
  3) ((fours++));;
  4) ((fives++));;
  5) ((sixes++));;
esac
}

echo


while [ "$throw" -lt "$MAXTHROWS" ]
do
  let "die1 = RANDOM % $PIPS"
  update_count $die1
  let "throw += 1"
done

print_result

exit $?

#  假设 RANDOM 是真随机，那么计数结果应该均匀分布。
#  当 $MAXTHROWS 的值为 600 时，每一个计数器的值都应该在 100 左右，
#+ 上下浮动大约 20。
#
#  记住 RANDOM 是一个 ***伪随机*** 生成器，
#+ 并且也不是其中最优秀的那一个。

#  随机化是一个很深奥且复杂的话题。
#  足够长的“随机”序列可能会出现一些
#+ 混乱或其他非随机化的表现。

# 练习（简单）：
# ---------------
# 重写脚本，修改为投掷硬币 1000 次。
# 显示为正面 "HEADS" 和背面 "TAILS"。
```

从上一个样例中我们可以发现，在每次调用 `RANDOM` 生成器时，最好利用重置生成器种子。在 `RANDOM` 生成器中使用相同的种子会生成相同序列的随机数。[^2]（与 C 语言中的 `random()` 函数的行为一致）

#### 样例 9-16. 重置 `RANDOM` 种子

```bash
#!/bin/bash
# seeding-random.sh: 设置 RANDOM 变量的种子。
# 版本号 1.1, 发布日期 09 Feb 2013

MAXCOUNT=25       # 生成随机数的个数。
SEED=

random_numbers ()
{
local count=0
local number

while [ "$count" -lt "$MAXCOUNT" ]
do
  number=$RANDOM
  echo -n "$number "
  let "count++"
done
}

echo; echo

SEED=1
RANDOM=$SEED      # 设置变量 RANDOM 会为随机数生成器设置种子。
echo "Random seed = $SEED"
random_numbers

RANDOM=$SEED      # 同样的种子 ...
echo; echo "Again, with same random seed ..."
echo "Random seed = $SEED"
random_numbers    # ... 生成了同样的数字序列。
                  #
                  # 在什么情况下重复一个随机化序列会有用？
                  
echo; echo

SEED=2
RANDOM=$SEED      # 用不同的种子再试一次 ...
echo "Random seed = $SEED"
random_numbers    # ... 生成了不同的数字序列。

echo; echo

# RANDOM=$$  利用脚本的进程 ID 设置 RANDOM 的种子。
# 同样也可以利用 'time' 或是 'date' 命令设置 RANDOM 的种子。

# 更花哨一点的 ...
SEED=$(head -1 /dev/urandom | od -N 1 | awk '{ print $2 }'| sed s/^0*//)
#  从 /dev/urandom （系统的伪随机设备文件）中
#+ 获取伪随机输出，
#+ 然后通过 "od" 转换为可打印八进制字符行，
#+ 然后 "awk" 命令会检索出一个数字作为种子，
#+ 最后用 "sed" 命令删除数字前面所有的前置 0。
RANDOM=$SEED
echo "Random seed = $SEED"
random_numbers

echo; echo

exit 0
```

{% hint style="info" %}

伪设备文件 `/dev/urandom` 提供了比 `$RANDOM` 变量更随机化的伪随机数。命令 `dd if=/dev/urandom of=targetfile bs=1 count=XXX` 将会创建一个包含均匀分布的伪随机数的文件。但是想要在脚本中将这些随机数赋值给变量需要做一些变通，比如使用命令 [`od`]() 进行过滤（参照上面的样例以及 [样例 16-14]() 和 [样例 A-36]()）或者使用管道导入命令 [md5sum]() 中（参照 [样例 36-16]()）。

当然也有其他在脚本中生成伪随机数的方法。比如 `Awk` 命令就提供了这样一种非常简易的方法。

#### 样例 9-17. 使用 [`awk`]() 命令生成伪随机数

```bash
#!/bin/bash
#  random2.sh: 返回大小在 0 - 1 内，
#+ 精度为小数点后 6 位的伪随机数。例如：0.822725
#  使用 awk rand() 函数。

AWKSCRIPT=' { srand(); print rand() } '
#           传递给 awk 的命令或参数
# 注意 srand() 重置了 awk 的随机数生成种子。


echo -n "Random number between 0 and 1 = "

echo | awk "$AWKSCRIPT"
# 如果省略 'echo' 将会发生什么？

exit 0


# 练习：
# ---------

# 1) 使用循环结构，输出 10 个不同的随机数。
#      （提示：你必须在每次循环中使用 srand() 函数重置种子以获得不同的随机数种子。
#+       如果你省略了这一步会发生什么？）

# 2) 利用整型乘数作为随机数的缩放因子，
#+   生成大小在 10 到 100 之间的随机数。

# 3) 内容与练习 #2 相同，只是这次生成随机整数。
```

同样，命令 [`date`]() 可以用于 [生成整型随机数序列]()。

{% endhint %}

## 注记

{% hint style="info" %}
真正的“随机性”，就其存在而言，只存在于一些类似放射性衰变这样还未被完全理解的自然现象中。计算机只能模拟这样的随机性，因此计算机生成的“随机数”序列被称作伪随机数。
{% endhint %}

{% hint style="info" %}
计算机用于生成伪随机数的种子可以被视作一个标识标签。例如，你可以将用种子 23 生成的随机数序列视作第23号序列。

伪随机数序列的一个属性是该序列在开始重复之前的周期长度。一个好的伪随机数生成器能够生成周期非常长的序列。
{% endhint %}

[^1]: Footnote Placeholder
[^2]: Footnote Placeholder