# 16.8 数学命令

## “与数字打交道”

### factor

将整数分解为质数。

```shell
bash$ factor 27417
27417: 3 13 19 37
```

**样例 16-46. 生成质数**

```shell
#!/bin/bash
# primes2.sh

#  生成质数是快速简便的方法，
#  无需诉诸花哨的算法。

CEILING=10000   # 1 至 10000
PRIME=0
E_NOTPRIME=

is_prime ()
{
  local factors
  factors=( $(factor $1) )  # 将`factor`的输出加载到数组中。

if [ -z "${factors[2]}" ]
#  "factors"数组的第三个元素：
#  ${factors[2]}是在参数中的第二个因数。
#  如果它是空白的，那么就不存在第二个因数，
#  并且参数就是素数。

then
  return $PRIME             # 0
else
  return $E_NOTPRIME        # null
fi
}

echo
for n in $(seq $CEILING)
do
  if is_prime $n
  then
    printf %5d $n
  fi   #    ^  每个数字五个位置就足够了。
done   #       对于更大的$CEILING，如果有必要，请调整上限。

echo

exit
```

### bc

Bash不能处理浮点计算，并且缺少某些重要数学函数的运算符。幸运的是，**bc**伸出了援手。

**bc**不仅是通用的，任意精度的计算实用程序，还提供了编程语言的许多功能。它的语法一定层面上类似于**C**语言。

由于它是一个表现良好的UNIX实用程序，也可以在[管道](https://tldp.org/LDP/abs/html/special-chars.html#PIPEREF)中使用，因此**bc**在脚本中可以派上用场。

这是一个使用**bc**计算脚本变量的简单模板。它使用了[命令替换](https://tldp.org/LDP/abs/html/commandsub.html#COMMANDSUBREF)。

```shell
          variable=$(echo "OPTIONS; OPERATIONS" | bc)
```

**样例 16-47. 每月支付抵押贷款**

```shell
#!/bin/bash
# monthlypmt.sh: 计算抵押贷款的每月还款额

#  这是在"mcalc"(抵押计算器)包中修改后的代码，
#  作者Jeff Schmidt
#  和
#  Mendel Cooper (当然有本书作者啦)。
#  http://www.ibiblio.org/pub/Linux/apps/financial/mcalc-1.6.tar.gz

echo
echo "Given the principal, interest rate, and term of a mortgage,"
echo "calculate the monthly payment."

bottom=1.0

echo
echo -n "Enter principal (no commas) "
read principal
echo -n "Enter interest rate (percent) "  # 如果是12%，请输入"12"，而不是".12"。
read interest_r
echo -n "Enter term (months) "
read term


 interest_r=$(echo "scale=9; $interest_r/100.0" | bc) # 转换为小数。
                 #           ^^^^^^^^^^^^^^^^^  除以100。 
                 # "scale"决定了小数点后的位数。

 interest_rate=$(echo "scale=9; $interest_r/12 + 1.0" | bc)


 top=$(echo "scale=9; $principal*$interest_rate^$term" | bc)
          #           ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
          #           计算利息的标准公式。

 echo; echo "Please be patient. This may take a while."

 let "months = $term - 1"
# ==================================================================== 
 for ((x=$months; x > 0; x--))
 do
   bot=$(echo "scale=9; $interest_rate^$x" | bc)
   bottom=$(echo "scale=9; $bottom+$bot" | bc)
#  bottom = $(($bottom + $bot"))
 done
# ==================================================================== 

# -------------------------------------------------------------------- 
#  Rick Boivie指出了上述循环更有效的实现方案，
#  它将计算时间减少了2/3。

# for ((x=1; x <= $months; x++))
# do
#   bottom=$(echo "scale=9; $bottom * $interest_rate + 1" | bc)
# done


#  然后，他提出了一种甚至更有效的替代方案，
#  该方案可将运行时间缩短约95%!

# bottom=`{
#     echo "scale=9; bottom=$bottom; interest_rate=$interest_rate"
#     for ((x=1; x <= $months; x++))
#     do
#          echo 'bottom = bottom * interest_rate + 1'
#     done
#     echo 'bottom'
#     } | bc`       # 使用命令替换内联'for循环'。
# --------------------------------------------------------------------------
#  另一方面，Frank Wang建议：
#  bottom=$(echo "scale=9; ($interest_rate^$term-1)/($interest_rate-1)" | bc)

#  因为 . . .
#  循环背后的算法实际上是几何比例级数的和。
#  求和公式为 e0(1-q^n)/(1-q)，
#  其中e0是第一个元素
#  q=e(n+1)/e(n)
#  n是元素的数量。
# --------------------------------------------------------------------------


 # let "payment = $top/$bottom"
 payment=$(echo "scale=2; $top/$bottom" | bc)
 # 美元和美分使用两位小数。

 echo
 echo "monthly payment = \$$payment"  # 在数量前输出一个美元($)符号
 echo


 exit 0


 # 练习：
 #   1) 过滤输入使本金数量以逗号分割。
 #   2) 过滤输入，允许将利息以百分比或十进制输入。
 #   3) 如果你还意犹未尽，
 #      请扩展此脚本以打印完整的摊销表。
```

**样例 16-48. 进制转换**

```shell
#!/bin/bash
###########################################################################
# Shell脚本   :    base.sh - 以不同的进制打印数字 (Bourne Shell)
# 作者        :    Heiner Steven (heiner.steven@odn.de)
# 时间        :    07-03-95
# 种类        :    桌面程序
# $Id: base.sh,v 1.2 2000/02/06 19:55:35 heiner Exp $
# ==> 上面一行是RCS ID信息。
###########################################################################
# 描述
#
# 更改项
# 21-03-95 stv    修复了以0xb作为输入导致的错误 (0.2)
###########################################################################

# ==> 在脚本作者的许可下在本书中使用。
# ==> 来自本书作者的注释。

NOARGS=85
PN=`basename "$0"`                   # 程序名
VER=`echo '$Revision: 1.2 $' | cut -d' ' -f2`  # ==> VER=1.2

Usage () {
    echo "$PN - print number to different bases, $VER (stv '95)
usage: $PN [number ...]

If no number is given, the numbers are read from standard input.
A number may be
    binary (base 2)        starting with 0b (i.e. 0b1100)
    octal (base 8)        starting with 0  (i.e. 014)
    hexadecimal (base 16)    starting with 0x (i.e. 0xc)
    decimal            otherwise (i.e. 12)" >&2
    exit $NOARGS 
}   # ==> 输出用法信息。

Msg () {
    for i   # ==> in [list] 不见了。为什么？
    do echo "$PN: $i" >&2
    done
}

Fatal () { Msg "$@"; exit 66; }

PrintBases () {
    # 确定数字的进制。
    for i      # ==> in [list] 不见了...
    do         # ==> 所以对命令行参数进行处理。
    case "$i" in
        0b*)        ibase=2;;    # 二进制
        0x*|[a-f]*|[A-F]*)    ibase=16;;    # 十六进制
        0*)            ibase=8;;    # 八进制
        [1-9]*)        ibase=10;;    # 十进制
        *)
        Msg "illegal number $i - ignored"
        continue;;
    esac

    # 删除前缀，将十六进制数字转换为大写 (bc需要进行这个处理过程)。
    number=`echo "$i" | sed -e 's:^0[bBxX]::' | tr '[a-f]' '[A-F]'`
    # ==> 使用":"作为sed命令的分割符，而不是"/"。

    # 将数字转换为十进制
    dec=`echo "ibase=$ibase; $number" | bc`  # ==> 'bc'是计算器实用程序。
    case "$dec" in
        [0-9]*)    ;;             # 正确
        *)        continue;;         # 错误：忽略
    esac

    # 在一行中打印所有进制转换。
    # ==> 以下操作将命令列表传递给'bc'程序。
    echo `bc <<!
        obase=16; "hex="; $dec
        obase=10; "dec="; $dec
        obase=8;  "oct="; $dec
        obase=2;  "bin="; $dec
!
    ` | sed -e 's: :    :g'

    done
}

while [ $# -gt 0 ]
# ==>  这里的“while循环”真的有必要吗，
# ==>+ 因为所有情况下要么退出循环，
# ==>+ 要么终止脚本。
# ==> (以上注释由Paulo Marcel Coelho Aragao所写。)
do
    case "$1" in
    --)     shift; break;;
    -h)     Usage;;                 # ==> 帮助信息。
    -*)     Usage;;
     *)     break;;                 # 第一个数字。
    esac   # ==> 添加非法输入检查可能是合理的。
    shift
done

if [ $# -gt 0 ]
then
    PrintBases "$@"
else                    # 从标准输入(stdin)读取。
    while read line
    do
    PrintBases $line
    done
fi


exit
```

调用**bc**的另一种方法涉及使用嵌入[命令替换](https://tldp.org/LDP/abs/html/commandsub.html#COMMANDSUBREF)块中的[here文档](https://tldp.org/LDP/abs/html/here-docs.html#HEREDOCREF)。当脚本需要将选项和命令列表传递给**bc**时，这尤其合适。

```shell
variable=`bc << LIMIT_STRING
options
statements
operations
LIMIT_STRING
`

...or...


variable=$(bc << LIMIT_STRING
options
statements
operations
LIMIT_STRING
)
```

**样例 16-49. 使用*here文档*调用*bc*程序**

```shell
#!/bin/bash
# 使用命令替换并结合'here文档'
# 来调用'bc'命令。


var1=`bc << EOF
18.33 * 19.78
EOF
`
echo $var1       # 362.56


#  $( ... ) 表示法同样奏效。
v1=23.53
v2=17.881
v3=83.501
v4=171.63

var2=$(bc << EOF
scale = 4
a = ( $v1 + $v2 )
b = ( $v3 * $v4 )
a * b + 15.35
EOF
)
echo $var2       # 593487.8452


var3=$(bc -l << EOF
scale = 9
s ( 1.7 )
EOF
)
# 返回1.7弧度的正弦。
# "-l"选项会调用'bc'的math库。
echo $var3       # .991664810


# 现在，试试看在函数中使用...
hypotenuse ()    # 计算直角三角形的斜边。
{                # c = sqrt( a^2 + b^2 )
hyp=$(bc -l << EOF
scale = 9
sqrt ( $1 * $1 + $2 * $2 )
EOF
)
# 无法直接从Bash函数返回浮点值。
# 但是，可是echo和捕获。
echo "$hyp"
}

hyp=$(hypotenuse 3.68 7.31)
echo "hypotenuse = $hyp"    # 8.184039344


exit 0
```

**样例 16-50. 计算PI值**

```shell
#!/bin/bash
# cannon.sh: 通过发射炮弹来近似PI。

# 作者: Mendel Cooper
# 许可证: Public Domain
# 版本 2.2, reldate 13oct08.

# 这是“蒙特卡洛”模拟的一个非常简单的实例: 
# 使用伪随机数模拟随机情况，
# 构建真实事件的数学模型，

#  考虑一个完美的正方形广场，边长为10000个单位。
#  这片土地的中心有一个完美的圆形湖泊，
#  直径为10000个单位。
#  该地块实际上主要是水，但四个角落的土地除外。
#  (把它想象成一个有内切圆的正方形。)
#
#  我们将在广场上用老式加农炮发射铁炮弹。
#  所有的炮弹都会落在广场上的某个地方，
#  要么在湖泊中，要么在干燥的角落上。
#  由于湖泊占据了大部分区域，
#  因此大多数炮弹都会噗通！地掉入水中。
#  仅有少数炮弹会砰！地炸在广场的四个实质地面上。
#
#  如果我们在广场上发射足够多的随机、没有目标的炮弹，
#  那么“噗通”的炮弹数与总发射数量的比率将接近于 PI/4。
#
#  简化的解释是，加农炮实际上只在正方形广场的右上象限发射，
#  即笛卡尔坐标平面的象限I上发射。
#
#
#  从理论上讲，发射的数量越多，拟合效果越好。
#  然而，与内置浮点数的编译语言相反，
#  shell脚本需要一些折中。
#  这降低了模拟的准确性。


DIMENSION=10000  # 图像两边的长度。
                 # 同时为生成的随机整数设置上限。

MAXSHOTS=1000    # 发射多少炮弹
                 # 设置10000或更多会更好，但是这会提高脚本执行时间。
PMULTIPLIER=4.0  # 缩放因子。

declare -r M_PI=3.141592654
                 # PI的实际9位值，用于比较。

get_random ()
{
SEED=$(head -n 1 /dev/urandom | od -N 1 | awk '{ print $2 }')
RANDOM=$SEED                                  #  来自样例脚本
                                              #  "seeding-random.sh"
let "rnum = $RANDOM % $DIMENSION"             #  范围少于10000.
echo $rnum
}

distance=        # 声明全局变量。
hypotenuse ()    # 计算直角三角形的斜边。
{                # 来自"alt-bc.sh"样例。
distance=$(bc -l << EOF
scale = 0
sqrt ( $1 * $1 + $2 * $2 )
EOF
)
#  将"scale"设置为零，将结果四舍五入为整数值，
#  这是此脚本中的必要折中方案。
#  它降低了该模拟的准确性。
}


# ==========================================================
# main() {
# “Main” 代码块，模仿c语言main()函数。

# 初始化变量
shots=0
splashes=0
thuds=0
Pi=0
error=0

while [ "$shots" -lt  "$MAXSHOTS" ]           # 主循环。
do

  xCoord=$(get_random)                        # 拿到随机X和Y坐标。
  yCoord=$(get_random)
  hypotenuse $xCoord $yCoord                  #  直角三角形的斜边 = 距离。

  ((shots++))

  printf "#%4d   " $shots
  printf "Xc = %4d  " $xCoord
  printf "Yc = %4d  " $yCoord
  printf "Distance = %5d  " $distance         #   距湖中心的距离
                                              #   -- “原点” --
                                              #   坐标 (0,0)。

  if [ "$distance" -le "$DIMENSION" ]
  then
    echo -n "SPLASH!  "
    ((splashes++))
  else
    echo -n "THUD!    "
    ((thuds++))
  fi

  Pi=$(echo "scale=9; $PMULTIPLIER*$splashes/$shots" | bc)
  # 将比率乘以4.0。
  echo -n "PI ~ $Pi"
  echo

done

echo
echo "After $shots shots, PI looks like approximately   $Pi"
#  往往值会偏高，
#  可能是由于舍入误差和$RANDOM的不完美随机性。
#  但通常仍在正负5%内 . . .
#  相当合理的粗略近似方法。
error=$(echo "scale=9; $Pi - $M_PI" | bc)
pct_error=$(echo "scale=2; 100.0 * $error / $M_PI" | bc)
echo -n "Deviation from mathematical value of PI =        $error"
echo " ($pct_error% error)"
echo

# "main"代码块结束。
# }
# ==========================================================

exit 0

#  人们可能会怀疑shell脚本是否适合
#  运行模拟复杂且计算密集的应用程序。
#
#  至少有两种理由：
#  1) 从概念上来讲: 它是可以做到的。
#  2) 在用编译的高级语言重写算法之前，可以对算法进行原型设计和测试。
```

另请参阅[样例 A-37](https://tldp.org/LDP/abs/html/contributed-scripts.html#STDDEV)。

### dc

**dc**(桌面计算器) 实用程序是[面向堆栈](https://tldp.org/LDP/abs/html/internalvariables.html#STACKDEFREF)的，使用RPN (*逆波兰表达式*)。与**bc**一样，具有编程语言的强大功能。

与**bc**的执行过程类似，将命令字符串[echo](https://tldp.org/LDP/abs/html/internal.html#ECHOREF)到**dc**。

```shell
echo "[Printing a string ... ]P" | dc
# 打印在前面括号之间的字符串。

# 现在是一些简单的算术。
echo "7 8 * p" | dc     # 56
#  首先将7，然后是8压入栈，
#  乘上("*"运算符), 再是打印结果("p"运算符)。
```

大多数人都避免使用**dc**，因为它具有非直观的输入和相当神秘的运算符。然而，它有其用途。

**样例 16-51. 将十进制数转换为十六进制**

```shell
#!/bin/bash
# hexconvert.sh: 将十进制数转换为十六进制

E_NOARGS=85 # 找不到命令行参数。
BASE=16     # 十六进制数。

if [ -z "$1" ]
then        # 需要一个命令行参数。
  echo "Usage: $0 number"
  exit $E_NOARGS
fi          # 练习: 添加参数有效性检查。


hexcvt ()
{
if [ -z "$1" ]
then
  echo 0
  return    # 如果没有参数传递给这个函数"Return" 0
fi

echo ""$1" "$BASE" o p" | dc
#                  o    设置输出的基数 (数值进制)。
#                    p  打印堆栈上的顶端。
# 对于其他选项: 'man dc' ...
return
}

hexcvt "$1"

exit
```

研究**dc**的[info](https://tldp.org/LDP/abs/html/basic.html#INFOREF)手册以理解其复杂性是一条苦痛之路。似乎有一小群*dc巫师*精英，他们乐于炫耀自己对这种强大但神秘的实用工具的技能。

```shell
bash$ echo "16i[q]sa[ln0=aln100%Pln100/snlbx]sbA0D68736142snlbxq" | dc
Bash
```

```shell
dc <<< 10k5v1+2/p # 1.6180339887
#  ^^^            使用Here字符串将操作馈送到dc。
#      ^^^        压入10并将其设置为精度 (10k)。
#         ^^      压入5并计算它的平方根
#                 (5v, v = 平方根)。
#           ^^    压入1并将其添加到运行总数中 (1+)。
#             ^^  压入2并将运行总数除以 (2/)。               
#               ^ 弹出并打印结果 (p)
#  结果是1.6180339887 ...
#  ... 这恰好是毕达哥拉斯的黄金比例，精度为10个小数点。
```

（译者注：数学表示为$$\frac{\sqrt{5}+1}{2}=1.6180339887$$）

**样例 16-52. 因式分解**

```shell
#!/bin/bash
# factr.sh: 将一个数字因式分解

MIN=2       # 不会对小于这个的数字起作用。
E_NOARGS=85
E_TOOSMALL=86

if [ -z $1 ]
then
  echo "Usage: $0 number"
  exit $E_NOARGS
fi

if [ "$1" -lt "$MIN" ]
then
  echo "Number to factor must be $MIN or greater."
  exit $E_TOOSMALL
fi  

# 练习：添加类型检查(拒绝非数值型参数)。

echo "Factors of $1:"
# -------------------------------------------------------
echo  "$1[p]s2[lip/dli%0=1dvsr]s12sid2%0=13sidvsr[dli%0=\
1lrli2+dsi!>.]ds.xd1<2" | dc
# -------------------------------------------------------
#  以上的代码由Michel Charpentier <charpov@cs.unh.edu>所写
#  (作为一个喜欢不换行的程序员，这里为了展示的需要分为两行)
#  本书获得了该脚本的使用许可(感谢！)

 exit

 # $ sh factr.sh 270138
 # 2
 # 3
 # 11
 # 4093
```

### awk

在脚本中进行浮点数学运算的另一种方法是在[shell包装器](https://tldp.org/LDP/abs/html/wrapper.html#SHWRAPPER)中使用[awk](https://tldp.org/LDP/abs/html/awk.html#AWKREF)的内置数学函数。

**样例 16-53. 计算三角形的斜边**

```shell
#!/bin/bash
# hypotenuse.sh: 返回直角三角形的 “斜边”长度。
#                ("直角边"的平方和的平方根)

ARGS=2                # 脚本需要传递三角形的两条直角边参数
E_BADARGS=85          # 错误的参数数量

if [ $# -ne "$ARGS" ] # 测试脚本的参数数量。
then
  echo "Usage: `basename $0` side_1 side_2"
  exit $E_BADARGS
fi


AWKSCRIPT=' { printf( "%3.7f\n", sqrt($1*$1 + $2*$2) ) } '
#             传递给awk的命令/参数


# 现在，通过管道传输给awk。
    echo -n "Hypotenuse of $1 and $2 = "
    echo $1 $2 | awk "$AWKSCRIPT"
#   ^^^^^^^^^^^^
# echo和管道的组合是将shell参数传递给awk的简易方式。

exit

# 练习：使用'bc'重写这个脚本，而不使用'awk'。
#           哪种方法更加直观？
```
