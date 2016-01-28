# 10.1 字符串处理

Bash 支持的字符串操作数量达到了一个惊人的数目。但可惜的是，这些操作工具缺乏一个统一的核心。他们中的一些是[参数代换](http://tldp.org/LDP/abs/html/parameter-substitution.html#PARAMSUBREF)的子集，另外一些则是 UNIX 下 [`expr`](http://tldp.org/LDP/abs/html/moreadv.html#EXPRREF) 函数的子集。这将会导致语法前后不一致或者功能上出现重叠，更不用说那些可能导致的混乱了。

### 字符串长度

#### `${#string}`

#### `expr length $string`

上面两个表达式等价于C语言中的 `strlen()` 函数。

#### `expr "$string" : '.*'`

```bash
stringZ=abcABC123ABCabc

echo ${#stringZ}                 # 15
echo `expr length $stringz`      # 15
echo `expr "$stringZ" : '.*'`    # 15
```

样例 10-1. 在文本的段落之间插入空行

```bash
#!/bin/bash
# paragraph-space.sh
# 版本 2.1，发布日期 2012年7月29日

# 在无空行的文本文件的段落之间插入空行。
# 像这样使用: $0 <FILENAME

MINLEN=60        # 可以试试修改这个值。它用来做判断。
#  假设一行的字符数小于 $MINLEN，并且以句点结束段落。
#+ 结尾部分有练习！

while read line  # 当文件有许多行的时候
do
  echo "$line"   # 输出行本身。
  
  len=${#line}
  if [[ "$len" -lt "$MINLEN" && "$line" =~ [*{\.}]$ ]]
# if [[ "$len" -lt "$MINLEN" && "$line" =~ \[*\.\] ]]
# 新版Bash将不能正常运行前一个版本的脚本。Ouch！
# 感谢 Halim Srama 指出这点，并且给出了修正版本。
    then echo    #  在该行以句点结束时，
  fi             #+ 增加一行空行。
done

exit

# 练习：
# -----
#  1) 该脚本通常会在文件的最后插入一个空行。
#+    尝试解决这个问题。
#  2) 在第17行仅仅考虑到了以句点作为句子终止的情况。
#+    修改以满足其他的终止符，例如 ?, ! 和 "。
```

### 起始部分字符串匹配长度

#### `expr match "$string" '$substring'`

其中，`$substring` 是一个[正则表达式](http://tldp.org/LDP/abs/html/regexp.html#REGEXREF)。

#### `expr "$string" : '$substring'`

其中，`$substring` 是一个正则表达式。

```bash

stringZ=abcABC123ABCabc
#       |------|
#       12345678

echo `expr match "$stringZ" 'abc[A-Z]*.2'`   # 8
echo `expr "$stringZ" : 'abc[A-Z]*.2'`       # 8
```

### 索引

#### `expr index $string $substring`

返回在 `$string` 中第一个出现的 `$substring` 字符所在的位置。

```bash
stringZ=abcABC123ABCabc
#       123456 ...
echo `expr index "$stringZ" C12`             # 6
                                             # C 的位置。
echo `expr index "$stringZ" 1c`              # 3
# 'c' (第三号位) 较 '1' 出现的更早。
```

几乎等价于C语言中的 `strchr()`。

### 截取字符串（字符串分片）

#### `${string:position}`

在 `$string` 中截取自 `$position` 起的字符串。

如果参数 `$string` 是 "*" 或者 "@"，那么将会截取自 `$position` 起的[位置参数](http://tldp.org/LDP/abs/html/internalvariables.html#POSPARAMREF)。[^1]

#### `${string:position:length}`

在 `$string` 中截取自 `$position` 起，长度为 `$length` 的字符串。

```bash

stringZ=abcABC123ABCabc
#       0123456789.....
#       索引位置从0开始。

echo ${stringZ:0}                            # abcABC123ABCabc
echo ${stringZ:1}                            # bcABC123ABCabc
echo ${stringZ:7}                            # 23ABCabc

echo ${stringz:7:3}                          # 23A
                                             # 三个字符的子字符串。
                                             
                                             

# 从右至左进行截取可行么？

echo ${stringZ:-4}                           # abcABC123ABCabc
# ${parameter:-default} 将会得到整个字符串。
# 但是……

echo ${stringZ:(-4)}                         # Cabc
echo ${stringZ: -4}                          # Cabc
# 现在可以了。
# 括号或者增加空格都可以"转义"位置参数。

# 感谢 Dan Jacobson 指出这些。
```

其中，参数 `position` 与 `length` 可以传入一个变量而不一定需要传入常量。

样例 10-2. 产生一个8个字符的随机字符串

```bash
#!/bin/bash
# rand-string.sh
# 产生一个8个字符的随机字符串。

if [ -n "$1" ]  #  如果在命令行中已经传入了参数，
then            #+ 那么就以它作为起始字符串。
  str0="$1"
else            #  否则，就将脚本的进程标识符PID作为起始字符串。
  str0="$$"
fi

POS=2  # 从字符串的第二位开始。
LEN=8  # 截取八个字符。

str1=$( echo "$str0" | md5sum | md5sum )
#                      ^^^^^^   ^^^^^^
# 将字符串通过管道计算两次 md5 来进行两次混淆。

randstring="${str1:$POS:$LEN}"
#                  ^^^^ ^^^^
# 允许传入参数

echo "$randstring"

exit $?

# bozo$ ./rand-string.sh my-password
# 1bdd88c4

# 不过不建议将其作为一种能够抵抗黑客的生成密码的方法。
```

如果参数 `$string` 是 "*" 或者 "@"，那么将会截取自 `$position` 起，最大个数为 `$length` 的位置参数。

```bash
echo ${*:2}          # 输出第二个及之后的所有位置参数。
echo ${@:2}          # 同上。

echo ${*:2:3}        # 从第二个位置参数起，输出三个位置参数。 
```

#### `expr substr $string $position $length`

在 `$string` 中截取自 `$position` 起，长度为 `$length` 的字符串。

```bash
stringZ=abcABC123ABCabc
#       123456789......
#       索引位置从1开始。

echo `expr substr $stringZ 1 2`              # ab
echo `expr substr $stringZ 4 3`              # ABC
```

#### `expr match "$string" '\($substring\)'`

在 `$string` 中截取自 `$position` 起的字符串，其中 `$substring` 是[正则表达式](http://tldp.org/LDP/abs/html/regexp.html#REGEXREF)。

#### `expr "$string" : '\($substring\)'`

在 `$string` 中截取自 `$position` 起的字符串，其中 `$substring` 是正则表达式。

```bash
stringZ=abcABC123ABCabc
#       =======

echo `expr match "$stringZ" '\(.[b-c]*[A-Z]..[0-9]\)'`   # abcABC1
echo `expr "$stringZ" : '\(.[b-c]*[A-Z]..[0-9]\)'`       # abcABC1
echo `expr "$stringZ" : '\(.......\)'`                   # abcABC1
# 上面所有的形式都给出了相同的结果。
```

#### `expr match "$string" '.*\($substring\)'`

从 `$string` 结尾部分截取 `$substring` 字符串，其中 `$substring` 是正则表达式。

#### `expr "$string" : '.*\($substring\)'`

从 `$string` 结尾部分截取 `$substring` 字符串，其中 `$substring` 是正则表达式。

```bash
stringZ=abcABC123ABCabc
#                ======

echo `expr match "$stringZ" '.*\([A-C][A-C][A-C][a-c]*\)'`    # ABCabc
echo `expr "$stringZ" : '.*\(......\)'`                       # ABCabc
```

### 删除子串

#### `${string#substring}`

删除从 `$string` 起始部分起，匹配到的最短的 `$substring`。

#### `${string##substring}`

删除从 `$string` 起始部分起，匹配到的最长的 `$substring`。

```bash
stringZ=abcABC123ABCabc
#       |----|          最长
#       |----------|    最短

echo ${stringZ#a*C}      # 123ABCabc
# 删除 'a' 与 'c' 之间最短的匹配。

echo ${stringZ##a*C}     # abc
# 删除 'a' 与 'c' 之间最长的匹配。



# 你可以使用变量代替 substring。

X='a*C'

echo ${stringZ#$X}      # 123ABCabc
echo ${stringZ##$X}     # abc
                        # 同上。
```

#### `${string%substring}`

删除从 `$string` 结尾部分起，匹配到的最短的 `$substring`。

例如：

```bash
# 将当前目录下所有后缀名为 "TXT" 的文件改为 "txt" 后缀。
# 例如 "file1.TXT" 改为 "file1.txt"。

SUFF=TXT
suff=txt

for i in $(ls *.$SUFF)
do
  mv -f $i $(i%.$SUFF).$suff
  #  除了从变量 $i 右侧匹配到的最短的字符串之外，
  #+ 其他一切都保持不变。
done ### 如果需要，循环可以压缩成一行的形式。

# 感谢 Rory Winston。
```

#### `${string%%substring}`

删除从 `$string` 结尾部分起，匹配到的最长的 `$substring`。

```bash

stringZ=abcABC123ABCabc
#                    ||     最短
#        |------------|     最长

echo ${stringZ%b*c}      # abcABC123ABCa
# 从结尾处删除 'b' 与 'c' 之间最短的匹配。

echo ${stringZ%%b*c}     # a
# 从结尾处删除 'b' 与 'c' 之间最长的匹配。
```

这个操作对生成文件名非常有帮助。

样例 10-3. 改变图像文件的格式及文件名

```bash
#!/bin/bash
#  cvt.sh:
#  将目录下所有的 MacPaint 文件转换为 "pbm" 格式。

#  使用由 Brian Henderson (bryanh@giraffe-data.com) 维护的
#+ "netpbm" 包下的 "macptobpm" 二进制工具。
#  Netpbm 是大多数 Linux 发行版的标准组成部分。

OPERATION=macptopbm
SUFFIX=pbm          # 新的文件名后缀。

if [ -n "$1" ]
then
  directory=$1      # 如果已经通过脚本参数传入了目录名的情况……
else
  directory=$PWD    # 否则就使用当前工作目录。
fi

#  假设目标目录下的所有 MacPaint 图像文件都拥有
#+ ".mac" 的文件后缀名。

for file in $directory/*    # 文件名匹配。
do
  filename=${file%.*c}      #  从文件名中删除 ".mac" 后缀
                            #+ ('.*c' 匹配 '.' 与 'c' 之间的
                            #  所有字符，包括其本身)。
  $OPERATION $file > "$filename.$SUFFIX"
                            # 将转换结果重定向到新的文件。
  rm -f $file               # 在转换后删除原文件。
  echo "$filename.$SUFFIX"  # 将记录输出到 stdout 中。
done

exit 0

# 练习：
# -----
# 这个脚本会将当前工作目录下的所有文件进行转换。
# 修改脚本，使得它仅转换 ".mac" 后缀的文件。



# *** 还可以使用另外一种方法。 *** #

#!/bin/bash
# 将图像批处理转换成不同的格式。
# 假设已经安装了 imagemagick。（在大部分 Linux 发行版中都有）

INFMT=png   # 可以是 tif, jpg, gif 等等。
OUTFMT=pdf  # 可以是 tif, jpg, gif, pdf 等等。

for pic in *"$INFMT"
do
  p2=$(ls "$pic" | sed -e s/\.$INFMT//)
  # echo $p2
  convert "$pic" $p2.$OUTFMT
done

exit $?
```

样例 10-4. 将流音频格式转换成 ogg 格式

```bash
#!/bin/bash
# ra2ogg.sh: 将流音频文件 (*.ra) 转换成 ogg 格式。

# 使用 "mplayer" 媒体播放器程序：
#      http://www.mplayerhq.hu/homepage
# 使用 "ogg" 库与 "oggenc"：
#      http://www.xiph.org/
#
# 脚本同时需要安装一些解码器，例如 sipr.so 等等一些。
# 这些解码器可以在 compat-libstdc++ 包中找到。


OFILEPREF=${1%%ra}      # 删除 "ra" 后缀。
OFILESUFF=wav           # wav 文件后缀。
OUTFILE="$OFILEPREF""$OFILESUFF"
E_NOARGS=85

if [ -z "$1" ]          # 必须指定一个文件进行转换。
then
  echo "Usage: `basename $0` [filename]"
  exit $E_NOAGRS
fi


######################################################
mplayer "$1" -ao pcm:file=$OUTFILE
oggenc "$OUTFILE"  # 由 oggenc 自动加上正确的文件后缀名。
######################################################

rm "$OUTFILE"      # 立即删除 *.wav 文件。
                   # 如果你仍需保留原文件，注释掉上面这一行即可。
                   
exit $?

#  注意：
#  -----
#  在网站上，点击一个 *.ram 的流媒体音频文件
#+ 通常只会下载到 *.ra 音频文件的 URL。
#  你可以使用 "wget" 或者类似的工具下载 *.ra 文件本身。


#  练习：
#  -----
#  这个脚本仅仅转换 *.ra 文件。
#  修改脚本增加适应性，使其可以转换 *.ram 或其他文件格式。
#
#  如果你非常有热情，你可以扩展这个脚本使其
#+ 可以自动下载并且转换流媒体音频文件。
#  给定一个 URL，自动下载流媒体音频文件 (使用 "wget")，
#+ 然后转换它。
```

下面是使用字符串截取结构对 [`getopt`](http://tldp.org/LDP/abs/html/extmisc.html#GETOPTY) 的一个简单模拟。

样例 10-5. 模拟 `getopt`

```bash
#!/bin/bash
# getopt-simple.sh
# 作者: Chris Morgan
# 允许在高级脚本编程指南中使用。


getopt_simple()
{
    echo "getopt_simple()"
    echo "Parameters are '$*'"
    until [ -z "$1" ]
    do
      echo "Processing parameter of: '$1'"
      if [ ${1:0:1} = '/' ]
      then
          tmp=${1:1}               # 删除开头的 '/'
          parameter=${tmp%%=*}     # 取出名称。
          value=${tmp##*=}         # 取出值。
          echo "Parameter: '$parameter', value: '$value'"
          eval $parameter=$value
      fi
      shift
    done
}

# 将所有参数传递给 getopt_simple()。
getopt_simple $*

echo "test is '$test'"
echo "test2 is '$test2'"

exit 0  # 可以查看该脚本的修改版 UseGetOpt.sh。

---

sh getopt_example.sh /test=value1 /test2=value2

Parameters are '/test=value1 /test2=value2'
Processing parameter of: '/test=value1'
Parameter: 'test', value: 'value1'
Processing parameter of: '/test2=value2'
Parameter: 'test2', value: 'value2'
test is 'value1'
test2 is 'value2'
```

### 子串替换

#### `${string/substring/replacement}`

替换匹配到的第一个 `$substring` 为 `$replacement`。[^2]

#### `${string//substring/replacement}`

替换匹配到的所有 `$substring` 为 `$replacement`。

```bash
stringZ=abcABC123ABCabc

echo ${stringZ/abc/xyz}       # xyzABC123ABCabc
                              # 将匹配到的第一个 'abc' 替换为 'xyz'。
                              
echo ${stringZ//abc/xyz}      # xyzABC123ABCxyz
                              # 将匹配到的所有 'abc' 替换为 'xyz'。
                              
echo  ---------------
echo "$stringZ"               # abcABC123ABCabc
echo  ---------------
                              # 字符串本身并不会被修改！
                              
# 匹配以及替换的字符串可以是参数么？
match=abc
repl=000
echo ${stringZ/$match/$repl}  # 000ABC123ABCabc
#              ^      ^         ^^^
echo ${stringZ//$match/$repl} # 000ABC123ABC000
# Yes!          ^      ^        ^^^         ^^^

echo

# 如果没有给定 $replacement 字符串会怎样？
echo ${stringZ/abc}           # ABC123ABCabc
echo ${stringZ//abc}          # ABC123ABC
# 仅仅是将其删除而已。
```

#### `${string/#substring/replacement}`

替换 `$string` 中最前端匹配到的 `$substring` 为 `$replacement`。

#### `${string/%substring/replacement}`

替换 `$string` 中最末端匹配到的 `$substring` 为 `$replacement`。

```bash
stringZ=abcABC123ABCabc

echo ${stringZ/#abc/XYZ}          # XYZABC123ABCabc
                                  # 将前端的 'abc' 替换为 'XYZ'
                                  
echo ${stringZ/%abc/XYZ}          # abcABC123ABCXYZ
                                  # 将末端的 'abc' 替换为 'XYZ'
```

[^1]: 这种情况同时适用于命令行参数和传入函数的参数。
[^2]: 注意根据使用时上下文的不同，`$substring` 和 `$replacement` 可以是文本字符串也可以是变量。可以参考第一个样例。