# 16.9 杂项命令

## 找不到特定类别进行归类的命令

### jot, seq

这些实用程序会生成一系列整数，且用户可选择增加量。

每个整数之间的默认分隔符是换行符，但这可以使用`-s`选项进行更改。

```shell
bash$ seq 5
1
 2
 3
 4
 5



bash$ seq -s : 5
1:2:3:4:5
```

**jot**和**seq**在[for循环](https://tldp.org/LDP/abs/html/loops1.html#FORLOOPREF1)中都能派上用场。

**样例 16-54. 使用*seq*生成循环参数**

```shell
#!/bin/bash
# 使用"seq"命令

echo

for a in `seq 80`  # 或者   for a in $( seq 80 )
# 与for a in 1 2 3 4 5 ... 80 等效(可以节省许多字数)。
# 也可以使用'jot'命令(如果系统上存在该命令)。
do
  echo -n "$a "
done      # 1 2 3 4 5 ... 80
# 这是使用命令的输出来
# 生成"for"循环[list]的样例。

echo; echo


COUNT=80  # 是的，'seq'命令也接受可替换的参数。

for a in `seq $COUNT`  # 或者是  for a in $( seq $COUNT )
do
  echo -n "$a "
done      # 1 2 3 4 5 ... 80

echo; echo

BEGIN=75
END=80

for a in `seq $BEGIN $END`
#  给"seq"命令两个参数，从第一个参数开始计数，
#  并一直持续到第二个参数。
do
  echo -n "$a "
done      # 75 76 77 78 79 80

echo; echo

BEGIN=45
INTERVAL=5
END=80

for a in `seq $BEGIN $INTERVAL $END`
#  给"seq"三个参数
#  从第一个参数开始计数，
#  以第二个参数为步长，
#  然后持续到第三个参数停止。
do
  echo -n "$a "
done      # 45 50 55 60 65 70 75 80

echo; echo

exit 0
```

一个更为简单的例子：

```shell
#  创建一组共10个文件，
#  名称为file.1, file.2 . . . file.10。
COUNT=10
PREFIX=file

for filename in `seq $COUNT`
do
  touch $PREFIX.$filename
  #  或者，可以进行其他操作，
  #  比如执行rm、grep等等。
done
```

**样例 16-55. 数字母**

```shell
#!/bin/bash
# letter-count.sh: 计算文本文件中出现的字母次数。
# 本脚本由Stefano Palmeri编写。
# 经许可在本书中使用。
# 由本书作者稍作修改。

MINARGS=2          # 本脚本需要至少两个参数。
E_BADARGS=65
FILE=$1

let LETTERS=$#-1   # 指定了多少字母（作为命令行参数）。
                   # (从命令行参数的数量中减去1。)


show_help(){
       echo
           echo Usage: `basename $0` file letters  
           echo Note: `basename $0` arguments are case sensitive.
           echo Example: `basename $0` foobar.txt G n U L i N U x.
       echo
}

# 检查参数数量。
if [ $# -lt $MINARGS ]; then
   echo
   echo "Not enough arguments."
   echo
   show_help
   exit $E_BADARGS
fi  


# 检查文件是否存在。
if [ ! -f $FILE ]; then
    echo "File \"$FILE\" does not exist."
    exit $E_BADARGS
fi



# 计算字母出现次数。
for n in `seq $LETTERS`; do
      shift
      if [[ `echo -n "$1" | wc -c` -eq 1 ]]; then             #  检查参数。
             echo "$1" -\> `cat $FILE | tr -cd  "$1" | wc -c` #  数数。
      else
             echo "$1 is not a  single char."
      fi  
done

exit $?

#  从功能上讲，该脚本与letter-count2.sh完全相同，
#  但是执行速度更快。
#  这是为什么？
```

> ![note](https://tldp.org/LDP/abs/images/note.gif)**jot**是一个经典的UNIX实用程序，比*seq*的功能更强，然而通常不包含在标准Linux发行版中。但是，源*rpm*包可以从[MIT库]()中下载。

与*seq*不同，**jot**可以使用`-r`选项来生成随机数序列。

```shell
bash$ jot -r 3 999
1069
 1272
 1428
```

### getopt

**getopt**命令用于解析带有[破折号](https://tldp.org/LDP/abs/html/special-chars.html#DASHREF)的命令行选项。此外部命令对应[getopts](https://tldp.org/LDP/abs/html/internal.html#GETOPTSX) Bash内置程序。**getopt**命令可以通过`-l`标志处理长选项，并且不要求参数顺序。

**样例 16-56. 使用*getopt*来解析命令行参数**

```shell
#!/bin/bash
# 使用 getopt

# 尝试使用以下的形式来调用该脚本：
#   sh ex33a.sh -a
#   sh ex33a.sh -abc
#   sh ex33a.sh -a -b -c
#   sh ex33a.sh -d
#   sh ex33a.sh -dXYZ
#   sh ex33a.sh -d XYZ
#   sh ex33a.sh -abcd
#   sh ex33a.sh -abcdZ
#   sh ex33a.sh -z
#   sh ex33a.sh a
# 解释上述每个命令的结果。

E_OPTERR=65

if [ "$#" -eq 0 ]
then   # 脚本至少需要一个参数。
  echo "Usage $0 -[options a,b,c]"
  exit $E_OPTERR
fi  

set -- `getopt "abcd:" "$@"`
# 将位置参数设置为命令行参数。
# 如果使用"$*"来替代"$@"会发生什么？

while [ ! -z "$1" ]
do
  case "$1" in
    -a) echo "Option \"a\"";;
    -b) echo "Option \"b\"";;
    -c) echo "Option \"c\"";;
    -d) echo "Option \"d\" $2";;
     *) break;;
  esac

  shift
done

#  通常最好在脚本中使用内置的'getopts'命令。
#  请参阅"ex33.sh"。

exit 0
```

> ![note](https://tldp.org/LDP/abs/images/note.gif)正如*Peggy Russell*指出：
> 
> 通常需要包含一个[eval程序](https://tldp.org/LDP/abs/html/internal.html#EVALREF)来正确处理[空格](https://tldp.org/LDP/abs/html/special-chars.html#WHITESPACEREF)和*引号*。
> 
> ```shell
> args=$(getopt -o a:bc:d -- "$@")
> eval set -- "$args"
> ```

请参阅[样例 10-5](https://tldp.org/LDP/abs/html/string-manipulation.html#GETOPTSIMPLE)**getopt程序**简化模拟。

### run-parts

**run-parts**命令 [[1]](https://tldp.org/LDP/abs/html/extmisc.html#FTN. AEN14105) 按文件名的ASCII码顺序执行目标目录中的所有脚本。当然，脚本需要具有执行权限。

 [cron命令](https://tldp.org/LDP/abs/html/system.html#CRONREF) [守护进程](https://tldp.org/LDP/abs/html/communications.html#DAEMONREF)会调用**run-parts**来执行位于`/etc/cron.*`中的脚本。

### yes

默认**yes**命令将字符y的连续不断地馈送到`标准输出(stdout)`。键入**control-C**终止运行。可以指定不同的输出字符串，例如在<code>**yes different string**</code>中，将不断从`标准输出(stdout)`输出`different string`。

你可能会问这样做有何目的。从命令行或脚本中，**yes**的输出可以重定向或通过管道传输到期望用户输入的程序中。实际上，这成为了*expect*脚本用于man文档中的一种不好的写法。

<code>**yes | fsck /dev/hda1**</code>将在非交互模式下运行**fsck**（小心！）。

<code>**yes | rm -r dirname**</code>与<code>**rm -rf dirname**</code>等效（小心！）。

> ![note](https://tldp.org/LDP/abs/images/warning.gif)请注意，将*yes*通过管道传输到具有潜在危险的系统命令（例如[fsck](https://tldp.org/LDP/abs/html/system.html#FSCKREF)和[fdisk](https://tldp.org/LDP/abs/html/system.html#FDISKREF)）时，可能会产生非预期的后果。

> ![note](https://tldp.org/LDP/abs/images/note.gif)*yes*命令会解析变量，更加确切地说，它输出被解析的变量。例如：
> 
> ```shell
> bash$ yes $BASH_VERSION
> 3.1.17(1)-release
>  3.1.17(1)-release
>  3.1.17(1)-release
>  3.1.17(1)-release
>  3.1.17(1)-release
>  . . .
> ```

这个特定的 “功能” 可用于动态创建一个*非常大*的ASCII文件:

```shell
bash$ yes $PATH > huge_file.txt
Ctl-C          
```

请快速地键入<code>**Ctl-C**</code>，计算机可不会和你讨价还价. . . .

我们可以在非常简单的脚本[函数](https://tldp.org/LDP/abs/html/functions.html#FUNCTIONREF)中模拟yes命令。

```shell
yes ()
{ # "yes"的简单模拟 ...
  local DEFAULT_TEXT="y"
  while [ true ]   # 死循环。
  do
    if [ -z "$1" ]
    then
      echo "$DEFAULT_TEXT"
    else           # 如果有参数 ...
      echo "$1"    # ... 应用并且打印它。
    fi
  done             #  唯一缺失的部分是
}                  #  --help 和 --version 选项。
```

### banner

使用[ASCII](https://tldp.org/LDP/abs/html/special-chars.html#ASCIIDEF)字符 (默认为 “#”) 将参数展现为垂直横幅并打印在`标准输出(stdout)`上。也可以重定向到打印机进行硬拷贝。

请注意*banner*命令在许多Linux发行版上已经删除，大概是因为它不再被认为有用。

### printenv

显示指定用户设置的所有[环境变量](https://tldp.org/LDP/abs/html/othertypesv.html#ENVREF)。

```shell
bash$ printenv | grep HOME
HOME=/home/bozo
```

### lp

**lp**和**lpr**命令用于将文件发送到打印队列，以进行硬拷贝。[[2]](https://tldp.org/LDP/abs/html/extmisc.html#FTN.AEN14214) 这些命令名可以追溯到另一个时代的行式打印机(line printer)。[[3]](https://tldp.org/LDP/abs/html/extmisc.html#FTN.AEN14218)

<code>bash$ **lp file1.txt**</code> or <code>bash$ **lp <file1.txt**</code>

将由**pr**生成的格式化文本通过管道输出到**lp**很有用。

<code>bash$ **pr -options file1.txt | lp**</code>

格式化软件包 (例如[groff](https://tldp.org/LDP/abs/html/textproc.html#GROFFREF)和*Ghostscript*) 可能会将其输出直接发送到**lp**进行处理。

<code>bash$ **groff -Tascii file.tr | lp**</code>

<code>bash$ **gs -options | lp file.ps**</code>

相关命令是用于查看打印队列的**lpq**，和用于从打印队列中删除作业的**lprm**。

### tee

[这是UNIX借鉴水暖行业的一个想法。]

这是一个重定向运算符，但有一点区别。像水管工的*tee*一样，它允许将一个或多个命令的输出 “虹吸” 到一个*文件*中，但不会影响结果。这对于将正在进行的过程打印到文件或纸张上很有用，也许可以通过它进行调试。

```shell
                      (重定向)
                     |----> 到文件
                     |
  ===================|===========================
  命令 ---> 命令 ---> |tee ---> 命令 ---> ---> 管道的输出
  ===============================================
```

```shell
cat listfile* | sort | tee check.file | uniq > result.file
#                      ^^^^^^^^^^^^^^   ^^^^    

#  文件"check.file"包含已排序未去重的“listfiles”，
#  然后由"uniq"删除重复的行。
```

### mkfifo

这个模糊的命令创建了一个*命名管道*，一个临时的*先进先出(first-in-first-out，即FIFO)缓冲区*，用于在进程之间传输数据。[[4]](https://tldp.org/LDP/abs/html/extmisc.html#FTN.AEN14280) 通常，一个进程写入FIFO，另一个进程从中读取。请参阅[样例 A-14](https://tldp.org/LDP/abs/html/contributed-scripts.html#FIFO)。

```shell
#!/bin/bash
# 该简短的脚本由Omair Eshkenazi撰写。
# 经许可在本书中使用（感谢！）。

mkfifo pipe1   # 没错，管道可以取名。
mkfifo pipe2   # 因此，被叫做“命名管道”。

(cut -d' ' -f1 | tr "a-z" "A-Z") >pipe2 <pipe1 &
ls -l | tr -s ' ' | cut -d' ' -f3,9- | tee pipe1 |
cut -d' ' -f2 | paste - pipe2

rm -f pipe1
rm -f pipe2

# 不需要在脚本执行结束的时候杀死后台进程（为什么？）。

exit $?

现在，请调用这个脚本并解释一下它的输出：
sh mkfifo-example.sh

4830.tar.gz          BOZO
pipe1   BOZO
pipe2   BOZO
mkfifo-example.sh    BOZO
Mixed.msg BOZO
```

### pathchk

此命令用于检查文件名的有效性。如果文件名超过允许的最大长度 (255个字符)，或者其路径中的一个或多个目录不可搜索，则会显示一条错误消息。

不幸的是，**pathchk**不会返回可识别的错误代码，因此在脚本中几乎没有用。实际情况下，可以考虑使用[文件测试运算符](https://tldp.org/LDP/abs/html/fto.html#RTIF)。

### dd

尽管这个有点晦涩和令人恐惧的数据复制器命令起源于UNIX小型机和IBM大型机之间交换磁带上的数据，但它仍然有其用途。**dd**命令作用只是复制一个文件 (或`标准输入(stdin) / 标准输出(stdout)`)，但具有转换功能，包括ASCII/EBCDIC，[[5]](https://tldp.org/LDP/abs/html/extmisc.html#FTN.AEN14318)大写/小写，在输入和输出之间交换字节对，以及跳过和/或截断输入文件的头或尾。

```shell
# 将文件转换为全部大写:

dd if=$filename conv=ucase > $filename.uppercase
#                    lcase   # 如果转换成小写
```

**dd**的一些基本参数：

- if=INFILE
  
  INFILE是*源*文件。

- of=OUTFILE
  
  OUTFILE是*目标*文件，是写入数据的文件。

- bs=BLOCKSIZE
  
  这是读取和写入的每个数据块的大小，通常为2的幂。

- skip=BLOCKS
  
  在复制之前，要在INFILE中跳过多少个数据块。当INFILE开头中有 “垃圾” 或乱码数据时，或者当希望仅复制INFILE的一部分时，这是一个很有用的选项。

- seek=BLOCKS
  
  在复制之前，要在OUTFILE中跳过多少个数据块，在OUTFILE开头留下空白数据。

- count=BLOCKS
  
  仅复制这么多数据块，而不是整个INFILE。

- conv=CONVERSION
  
  在复制操作之前应用到INFILE数据的转换方法。

<code>**dd --help**</code>会列出该强大命令的所有选项。

**样例 16-57. 一个复制自己的脚本**

```shell
#!/bin/bash
# self-copy.sh

# 这个脚本会复制自己。

file_subscript=copy

dd if=$0 of=$0.$file_subscript 2>/dev/null
# 禁止dd命令输出消息:            ^^^^^^^^^^^

exit $?


#  当一个程序的唯一输出是自己的源代码时，这个程序被称为Quine。
#  这个脚本能被成为一个Quine。
```

**样例 16-58. 练习*dd*命令**

```shell
#!/bin/bash
# exercising-dd.sh

# 该脚本由Stephane Chazelas所写。
# 由本书作者略微修改。

infile=$0           # 这个脚本。
outfile=log.txt     # 留下的输出文件。
n=8
p=11

dd if=$infile of=$outfile bs=1 skip=$((n-1)) count=$((p-n+1)) 2> /dev/null
# 从此脚本 (“bash”) 中，提取字符数n到p (8到11)的字符。

# ----------------------------------------------------------------

echo -n "hello vertical world" | dd cbs=1 conv=unblock 2> /dev/null
# 垂直向下输出"hello vertical world"。
# 为什么？每个dd命令在写入时会换行。

exit $?
```

为了演示**dd**的丰富功能，让我们用它来捕获键盘输入。

**样例 16-59. 捕获键盘输入**

```shell
#!/bin/bash
# dd-keypress.sh: 不需要键入ENTER就可以捕捉键盘输入。


keypresses=4                      # 要捕获的按键次数。


old_tty_setting=$(stty -g)        # 保存旧终端设置。

echo "Press $keypresses keys."
stty -icanon -echo                # 禁用规范模式。
                                  # 禁用本地的echo。
keys=$(dd bs=1 count=$keypresses 2> /dev/null)
# 当选项'if'(输入文件)没有指定时，'dd'命令使用标准输入(stdin)。

stty "$old_tty_setting"           # 回复旧终端设置。

echo "You pressed the \"$keys\" keys."

# 感谢Stephane Chazelas。
exit 0
```

**dd**命令可以对数据流进行随机访问。

```shell
echo -n . | dd bs=1 seek=4 of=file conv=notrunc
#  "conv=notrunc"选项意味着输出文件不会被截断。

# 感谢S.C.
```

**dd**命令可以将原始数据和磁盘映像复制到设备，例如软盘和磁带驱动器 ([样例 A-5](https://tldp.org/LDP/abs/html/contributed-scripts.html#COPYCD))。一个常见的用途是创建引导软盘。

<code>**dd if=kernel-image of=/dev/fd0H1440**</code>

同样，**dd**可以将软盘的全部内容 (甚至是用 “外来” 操作系统格式化的软盘) 作为镜像文件复制到硬盘驱动器。

<code>**dd if=/dev/fd0 of=/home/bozo/projects/floppy.img**</code>

同样，**dd**可以创建可引导的闪存驱动器和sd卡。

<code>**dd if=image.iso of=/dev/sdb**</code>

**样例 16-30. 在SD卡上刻录可启动的*树莓派*系统**

```shell
#!/bin/bash
# rp.sdcard.sh
# 准备一个刻录有能启动的树莓派镜像的SD卡。

# $1 = 镜像名
# $2 = sd卡 (设备文件)
# 其他的默认为默认项，见下。

DEFAULTbs=4M                                 # 块大小, 默认4mb。
DEFAULTif="2013-07-26-wheezy-raspbian.img"   # 重用发行版。
DEFAULTsdcard="/dev/mmcblk0"                 # 可能会不同，请检查！
ROOTUSER_NAME=root                           # 必须使用root权限运行！
E_NOTROOT=81
E_NOIMAGE=82

username=$(id -nu)                           # 谁在运行这个脚本？
if [ "$username" != "$ROOTUSER_NAME" ]
then
  echo "This script must run as root or with root privileges."
  exit $E_NOTROOT
fi

if [ -n "$1" ]
then
  imagefile="$1"
else
  imagefile="$DEFAULTif"
fi

if [ -n "$2" ]
then
  sdcard="$2"
else
  sdcard="$DEFAULTsdcard"
fi

if [ ! -e $imagefile ]
then
  echo "Image file \"$imagefile\" not found!"
  exit $E_NOIMAGE
fi

echo "Last chance to change your mind!"; echo
read -s -n1 -p "Hit a key to write $imagefile to $sdcard [Ctl-c to exit]."
echo; echo

echo "Writing $imagefile to $sdcard ..."
dd bs=$DEFAULTbs if=$imagefile of=$sdcard

exit $?

# 练习：
# ---------
# 1) 提供额外的错误检查。
# 2) 让脚本自动检测sd卡的设备文件 (困难!)。
# 3) 让脚本自动检测$PWD下的镜像文件(*img)。
```

**dd**的其他应用包括初始化临时交换文件（[样例 31-2](https://tldp.org/LDP/abs/html/zeros.html#EX73)）和Ramdisk（[样例31-3](https://tldp.org/LDP/abs/html/zeros.html#RAMDISK)）。它甚至可以对整个硬盘驱动器分区进行低级复制，尽管通常不建议这样做。

人们 (可能与他们的空闲时间没有更好的关系) 一直在思考**dd**命令的有趣应用。

**样例 16-61. 安全地删除一个文件**

```shell
#!/bin/bash
# blot-out.sh: 删除文件的 “所有” 痕迹。

#  这个脚本使用随机字节交替覆写一个目标文件，
#  然后在最终删除它之前归零。
#  在这之后，即使通过常规方法检查原始磁盘扇区
#  也不会显示原始文件数据。

PASSES=7         #  传递的文件碎片数量。
                 #  增加这个值会减缓脚本的执行速度，
                 #  尤其是在比较大的目标文件上。
BLOCKSIZE=1      #  拿到/dev/urandom的I/O需要指定单位块大小，
                 #  否则你将会得到一个奇怪的结果。
E_BADARGS=70     #  各种错误退出码。
E_NOT_FOUND=71
E_CHANGED_MIND=72

if [ -z "$1" ]   # 不指定文件名。
then
  echo "Usage: `basename $0` filename"
  exit $E_BADARGS
fi

file=$1

if [ ! -e "$file" ]
then
  echo "File \"$file\" not found."
  exit $E_NOT_FOUND
fi  

echo; echo -n "Are you absolutely sure you want to blot out \"$file\" (y/n)? "
read answer
case "$answer" in
[nN]) echo "Changed your mind, huh?"
      exit $E_CHANGED_MIND
      ;;
*)    echo "Blotting out file \"$file\".";;
esac


flength=$(ls -l "$file" | awk '{print $5}')  # 第五个字段是文件长度。
pass_count=1

chmod u+w "$file"   # 允许覆写/删除这个文件。

echo

while [ "$pass_count" -le "$PASSES" ]
do
  echo "Pass #$pass_count"
  sync         # 刷新缓存。
  dd if=/dev/urandom of=$file bs=$BLOCKSIZE count=$flength
               # 用随机字节进行填充。
  sync         # 再次刷新缓存。
  dd if=/dev/zero of=$file bs=$BLOCKSIZE count=$flength
               # 用0进行填充。
  sync         # 又一次刷新缓存。
  let "pass_count += 1"
  echo
done  


rm -f $file    # 最后，删除乱七八糟的文件。
sync           # 最后一次刷新缓存。

echo "File \"$file\" blotted out and deleted."; echo


exit 0

#  这是一种相当安全的、效率低下
#  且缓慢地彻底“粉碎”文件的方法。
#  GNU "fileutils"软件包的一部分"shred"命令执行相同的操作，
#  但是效率更高。

#  将不能通过普通方法“取消删除”或检索该文件。
#  但是 . . .
#  这种简单的方法 *不可能* 经得起复杂的数据恢复算法。

#  此脚本在日志文件系统中可能无法正常运行。
#  练习（困难）：修复这个问题。

#  Tom Vier的"wipe"文件删除软件包
#  比这个简单的脚本在文件粉碎方面做得更彻底。
#     http://www.ibiblio.org/pub/Linux/utils/file/wipe-2.0.0.tar.bz2

#  想深入分析文件删除和安全这一主题，
#  请参阅Peter Gutmann的论文，
#      "Secure Deletion of Data From Magnetic and Solid-State Memory".
#       http://www.cs.auckland.ac.nz/~pgut001/pubs/secure_del.html
```

另请参阅[参考文献](https://tldp.org/LDP/abs/html/biblio.html#BIBLIOREF)中的[dd线程](https://tldp.org/LDP/abs/html/biblio.html#DDLINK)条目。

### od

**od**，即*八进制转储过滤器*，将输入 (或文件) 转换为八进制 (base-8) 或其他基数。作为一个二进制数据过滤器，这对于查看或处理二进制数据文件或其他不可读的系统[设备文件](https://tldp.org/LDP/abs/html/devref1.html#DEVFILEREF) (例如`/dev/urandom`) 很有用。

```shell
head -c4 /dev/urandom | od -N4 -tu4 | sed -ne '1s/.* //p'
# 样例输出: 1324725719, 3918166450, 2989231420, 等等。

# 来自Stéphane Chazelas所写的rnd.sh样例脚本。
```

另请参阅[样例 9-16](https://tldp.org/LDP/abs/html/randomvar.html#SEEDINGRANDOM)和[样例 A-36](https://tldp.org/LDP/abs/html/contributed-scripts.html#INSERTIONSORT)。

### hexdump

执行二进制文件的十六进制、八进制、十进制或ASCII转储。此命令与上面的**od**命令大致等效，但作用不大。可结合[dd](https://tldp.org/LDP/abs/html/extmisc.html#DDREF)和[less](https://tldp.org/LDP/abs/html/filearchiv.html#LESSREF)查看二进制文件的内容。

```shell
dd if=/bin/ls | hexdump -C | less
# -C选项很好将输出格式化为表格形式。
```

### objdump

以十六进制或反汇编清单 (带有`-d`选项)的形式，显示有关目标文件或二进制可执行文件的信息。

```shell
bash$ objdump -d /bin/ls
/bin/ls:     file format elf32-i386

 Disassembly of section .init:

 080490bc <.init>:
  80490bc:       55                      push   %ebp
  80490bd:       89 e5                   mov    %esp,%ebp
  . . .
```

### mcookie

此命令生成 “魔术cookie”，即128位 (32字符) 伪随机十六进制数，通常由X服务器用作授权 “签名”。这也可以在脚本中用于生成 “快捷实用”的随机数。

```shell
random000=$(mcookie)
```

当然，出于相同目的脚本也可以使用[md5sum](https://tldp.org/LDP/abs/html/filearchiv.html#MD5SUMREF)命令。

```shell
# 使用脚本本身生成md5校验和。
random001=`md5sum $0 | awk '{print $1}'`
# 使用'awk'删除文件名。
```

**mcookie** 命令提供了另一种生成 “唯一” 文件名的方法。

**样例 16-52. 文件名生成器**

```shell
#!/bin/bash
# tempfile-name.sh:  临时文件文件名生成器

BASE_STR=`mcookie`   # 32位魔术cookie。
POS=11               # 魔术cookie字符串中的任意位置。
LEN=5                # 获得$LEN连续字符。

prefix=temp          #  毕竟，这是一个"temp"文件
                     #  若要加强 “唯一性”，
                     #  请使用与后缀相同的方法生成文件名前缀，如下所示。

suffix=${BASE_STR:POS:LEN}
                     #  从位置11
                     #  提取一个5个字符的字符串。

temp_filename=$prefix.$suffix
                     #  重构文件名。

echo "Temp filename = "$temp_filename""

# sh tempfile-name.sh
# Temp filename = temp.e19ea

#  请将此生成“唯一”文件名的方法
#  与ex51.sh中的'date'方法进行比较。
exit 0
```

### units

此实用程序用于在不同的*度量单位*之间进行转换。通常通过交互模式进行调用，**unit**也可能会在脚本中使用。

```shell
#!/bin/bash
# unit-conversion.sh
# 必须已安装'unit'实用程序

convert_units ()  # 获取unit用于转换的参数
{
  cf=$(units "$1" "$2" | sed --silent -e '1p' | awk '{print $2}')
  # 剥离除转换率以外的所有内容。
  echo "$cf"
}  

Unit1=miles
Unit2=meters
cfactor=`convert_units $Unit1 $Unit2`
quantity=3.73

result=$(echo $quantity*$cfactor | bc)

echo "There are $result $Unit2 in $quantity $Unit1."

#  如果你将不兼容的单位，
#  如“英亩”和“英里”传递给函数，会发生什么？

exit 0

# 练习: 编辑此脚本以接受命令行参数，
#      当然，并进行适当的错误检查。
```

### m4

恭喜你找到这个宝藏命令，**m4**是一个强大的宏[[6]](https://tldp.org/LDP/abs/html/extmisc.html#FTN.AEN14523)处理过滤器，实际上是一个完整的语言。尽管最初作为*RatFor*的预处理器，但其实**m4**作为独立的实用程序也很有用。实际上，**m4**除了大量的宏扩展功能外，还结合了[eval](https://tldp.org/LDP/abs/html/internal.html#EVALREF)，[tr](https://tldp.org/LDP/abs/html/textproc.html#TRREF)和[awk](https://tldp.org/LDP/abs/html/awk.html#AWKREF)的一些功能。

2002年4月期的[*《Linux Journal》*](http://www.linuxjournal.com/)有一篇介绍**m4**及其用途的非常好的文章。

**样例 16-64. *m4*牛刀小试**

```shell
#!/bin/bash
# m4.sh: 使用m4宏处理器

# 字符串
string=abcdA01
echo "len($string)" | m4                            #   7
echo "substr($string,4)" | m4                       # A01
echo "regexp($string,[0-1][0-1],\&Z)" | m4      # 01Z

# 数学运算
var=99
echo "incr($var)" | m4                              #  100
echo "eval($var / 3)" | m4                          #   33

exit
```

### xmessage

这种基于X的[echo](https://tldp.org/LDP/abs/html/internal.html#ECHOREF)变体在会桌面上弹出一个消息/查询窗口。

```shell
xmessage Left click to continue -button okay
```

### zenity

[zenity](http://freshmeat.net/projects/zenity)实用程序用于显示<em>GTK+</em>对话[窗口小部件](https://tldp.org/LDP/abs/html/assortedtips.html#WIDGETREF)，[非常适合用于编写脚本](https://tldp.org/LDP/abs/html/assortedtips.html#ZENITYREF2)。

### doexec

**doexec**命令允许将任意参数列表传递给*二进制可执行文件*。特别是当传递argv[0] (在脚本中对应于[$0](https://tldp.org/LDP/abs/html/othertypesv.html#POSPARAMREF1))时，可以通过各种名称调用可执行文件，然后根据调用它的名称执行不同的操作集。这相当于将选项参数传递给可执行文件的这种方式。

例如，`/usr/local/bin`目录可能包含一个名为 “aaa” 的二进制文件。调用**doexec /usr/local/bin/aaa list**将*列出*当前工作目录中所有以 “a” 开头的文件，而调用 (相同的可执行文件) **doexec /usr/local/bin/aaa delete**将*删除*这些文件。

> ![note](https://tldp.org/LDP/abs/images/note.gif)可执行文件的各种行为必须在可执行文件自身代码中定义，类似于shell脚本中的以下内容:
> 
> ```shell
> case `basename $0` in
> "name1" ) do_something;;
> "name2" ) do_something_else;;
> "name3" ) do_yet_another_thing;;
> *       ) bail_out;;
> esac
> ```

### dialog

[dialog](https://tldp.org/LDP/abs/html/assortedtips.html#DIALOGREF)工具系列提供了一种从脚本中调用交互式 “对话框” 的方法。**dialog**的更精细的变体 -- **gdialog**、**Xdialog**和**kdialog** -- 实际上调用了X-Windows[窗口小部件](https://tldp.org/LDP/abs/html/assortedtips.html#WIDGETREF)。

### sox

**sox**或 “声音交换” 命令用于播放和转换音频文件。实际上，`/usr/bin/play`可执行文件 (现已弃用) 不过是*sox*的shell包装器。

例如，**sox soundfile.wav soundfile.au**将WAV音频文件更改为 (Sun音频格式) AU音频文件。

Shell脚本非常适合对音频文件进行**sox**批处理。有关样例，请参见[Linux Radio Timeshift HOWTO](http://osl.iu.edu/~tveldhui/radio/)和[MP3do Project](http://savannah.nongnu.org/projects/audiodo)。

## 注记

[[1]](https://tldp.org/LDP/abs/html/extmisc.html#AEN14105)这实际上改编自Debian Linux发行版的脚本。

[[2]](https://tldp.org/LDP/abs/html/extmisc.html#AEN14214)*打印队列*是指一组“等待”打印的作业组。

[[3]](https://tldp.org/LDP/abs/html/extmisc.html#AEN14218)大型机械*行式打印机*一次将一行字体打印到互相连接的*greenbar*纸上，并伴随着[大量的噪音]([The IBM 1403 Printer](http://www.columbia.edu/cu/computinghistory/1403.html))。所印刷的硬拷贝被称为*printout*。

[[4]](https://tldp.org/LDP/abs/html/extmisc.html#AEN14280)有关此主题的出色概述，请参阅1997年9月期[*《Linux Journal》*]([http://www.linuxjournal.com/](http://www.linuxjournal.com/))中的Andy Vaught所著的文章[《命名管道简介》](http://www2.linuxjournal.com/lj-issues/issue41/2156.html)”。

[[5]](https://tldp.org/LDP/abs/html/extmisc.html#AEN14318)EBCDIC (发音为 “ebb-sid-ick”) 是扩展二进制编码的十进制交换码 (一种过时的IBM数据格式) 的首字母缩写。**dd**的`conv=ebcdic`选项的一个奇怪应用场景是作为一个“快捷简单”，但不是很安全的文本文件编码器。

```shell
cat $file | dd conv=swab,ebcdic > $file_encrypted
# 编码（这文件看起来有点像乱码）。    
# 选项也可以切换为byte(swab)，使其看起来更费解一些。

cat $file_encrypted | dd conv=swab,ascii > $file_plaintext
# 解码。
```

[[6]](https://tldp.org/LDP/abs/html/extmisc.html#AEN14523)*宏*是一个符号常数，它扩展为一个命令字符串或一组对参数的操作集。简单地说，它是一个快捷方式或缩写。
