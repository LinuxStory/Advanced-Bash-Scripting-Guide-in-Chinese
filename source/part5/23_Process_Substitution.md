# 第二十三章. 进程替换

用[管道](http://tldp.org/LDP/abs/html/special-chars.html#PIPEREF) 将一个命令的 ```标准输出``` 输送到另一个命令的 ```标准输入``` 是个强大的技术。但是如果你需要用管道输送_多个_命令的 ```标准输出``` 怎么办？这时候 _进程替换_ 就派上用场了。

 _进程替换_ 把一个（或多个）[进程](http://tldp.org/LDP/abs/html/special-chars.html#PROCESSREF) 的输出送到另一个进程的 ```标准输入```。

**样板**
命令列表要用括号括起来
 ```
>(command_list)
<(command_list)
 ```
进程替换使用 ```/dev/fd/<n>``` 文件发送括号内进程的结果到另一个进程。[1]

<img src="http://tldp.org/LDP/abs/images/caution.gif">"<"或">"与括号之间没有空格，加上空格或报错。

```
bash$ echo >(true)
/dev/fd/63

bash$ echo <(true)
/dev/fd/63

bash$ echo >(true) <(true)
/dev/fd/63 /dev/fd/62

bash$ wc <(cat /usr/share/dict/linux.words)
 483523  483523 4992010 /dev/fd/63

bash$ grep script /usr/share/dict/linux.words | wc
    262     262    3601

bash$ wc <(grep script /usr/share/dict/linux.words)
    262     262    3601 /dev/fd/63
```

<img src="http://tldp.org/LDP/abs/images/note.gif">Bash用两个文件描述符创建管道，```--fIn 和 fOut--``` 。[true](http://tldp.org/LDP/abs/html/internal.html#TRUEREF) 的```标准输入```连接 fOut(dup2(fOut, 0))，然后Bash 传递一个 ```/dev/fd/fIn``` 参数给 **echo** 。在不使用 ```/dev/fd/<n>``` 的系统里，Bash可以用临时文件（感谢 S.C. 指出这点）。

进程替换可以比较两个不同命令的输出，或者同一个命令使用不同选项的输出。
```
bash$ comm <(ls -l) <(ls -al)
total 12
-rw-rw-r--    1 bozo bozo       78 Mar 10 12:58 File0
-rw-rw-r--    1 bozo bozo       42 Mar 10 12:58 File2
-rw-rw-r--    1 bozo bozo      103 Mar 10 12:58 t2.sh
        total 20
        drwxrwxrwx    2 bozo bozo     4096 Mar 10 18:10 .
        drwx------   72 bozo bozo     4096 Mar 10 17:58 ..
        -rw-rw-r--    1 bozo bozo       78 Mar 10 12:58 File0
        -rw-rw-r--    1 bozo bozo       42 Mar 10 12:58 File2
        -rw-rw-r--    1 bozo bozo      103 Mar 10 12:58 t2.sh
```
进程替换可以比较两个目录的内容——来检查哪些文件在这个目录而不在那个目录。
```
diff <(ls $first_directory) <(ls $second_directory)
```
进程替换的一些其他用法：
```
read -a list < <( od -Ad -w24 -t u2 /dev/urandom )
#  从 /dev/urandom 读取一个随机数列表
#+ 用 "od" 处理
#+ 输送到 "read" 的标准输入. . .
#  来自 "insertion-sort.bash" 示例脚本。
#  致谢：JuanJo Ciarlante。
```

```
PORT=6881   # bittorrent（BT端口）

#  扫描端口，确保没有恶意行为
netcat -l $PORT | tee>(md5sum ->mydata-orig.md5) |
gzip | tee>(md5sum - | sed 's/-$/mydata.lz2/'>mydata-gz.md5)>mydata.gz

#  检查解压缩结果：
  gzip -d<mydata.gz | md5sum -c mydata-orig.md5)
#  对原件的MD5校验用来检查标准输入，并且探测压缩当中出现的问题。

#  Bill Davidsen 贡献了这个例子
#+ （ABS指南作者做了轻微修改）。
```

```
cat <(ls -l)
# 等价于	ls -l | cat

sort -k 9 <(ls -l /bin) <(ls -l /usr/bin) <(ls -l /usr/X11R6/bin)
#  列出 3 个主要 'bin' 目录的文件，按照文件名排序。
#  注意，有三个（数一下）单独的命令输送给了 'sort'。

diff <(command1) <(command2)    # 比较命令输出结果的不同之处。

tar cf >(bzip2 -c > file.tar.bz2) $directory_name

#  调用 "tar cf /dev/fd/?? $directory_name"，然后 "bzip2 -c > file.tar.bz2"。
#
#  因为 /dev/fd/<n> 系统特性
#  不需要在两个命令之间使用管道符
#
#  这个可以模拟
#
bzip2 -c < pipe > file.tar.bz2&
tar cf pipe $directory_name
rm pipe
#	或者
exec 3>&1
tar cf /dev/fd/4 $directory_name 4>&1 >&3 3>&- | bzip2 -c > file.tar.bz2 3>&-
exec 3>&-

# 致谢：Stéphane Chazelas
```

在子shell中 [echo 命令用管道输送给 while-read 循环](http://tldp.org/LDP/abs/html/gotchas.html#BADREAD0)时会出现问题，下面是避免的方法：

**例23-1 不用 fork 的代码块重定向。**
```
#!/bin/bash

#  wr-ps.bash: 使用进程替换的 while-read 循环。

#  示例由 Tomas Pospisek 贡献。
# （ABS指南作者做了大量改动。）

echo

echo "random input" | while read i
do
  global=3D": Not available outside the loop."
  # ... 因为在子 shell 中运行。
done

echo "\$global (从子进程之外) = $global"
# $global (从子进程之外) =

echo; echo "--"; echo

while read i
do
  echo $i
  global=3D": Available outside the loop."
  # ... 因为没有在子 shell 中运行。
done < <( echo "random input" )
#    ^ ^

echo "\$global (使用进程替换) = $global"
#  随机输入
#  $global (使用进程替换)= 3D: Available outside the loop.


echo; echo "##########"; echo



# 同样道理 . . .

declare -a inloop
index=0
cat $0 | while read line
do
  inloop[$index]="$line"
  ((index++))
  # 在子 shell 中运行，所以 ...
done
echo "OUTPUT = "
echo ${inloop[*]}           # ... 什么也没有显示。


echo; echo "--"; echo


declare -a outloop
index=0
while read line
do
  outloop[$index]="$line"
  ((index++))
  # 没有在子 shell 中运行，所以 ...
done < <( cat $0 )
echo "OUTPUT = "
echo ${outloop[*]}          # ... 整个脚本的结果显示出来。

exit $?
```
下面是个类似的例子。

**例 23-2. 重定向进程替换的输出到一个循环内**
```
#!/bin/bash
# psub.bash
#  受 Diego Molina 启发（感谢！）。

declare -a array0
while read
do
  array0[${#array0[@]}]="$REPLY"
done < <( sed -e 's/bash/CRASH-BANG!/' $0 | grep bin | awk '{print $1}' )
#  由进程替换来设置'read'默认变量（$REPLY）。
#+ 然后将变量复制到一个数组。

echo "${array0[@]}"

exit $?

# ====================================== #
# 运行结果：
bash psub.bash

#!/bin/CRASH-BANG! done #!/bin/CRASH-BANG!
```
一个读者发来一个有趣的进程替换例子，如下：
```
# SuSE 发行版中提取的脚本片段：

# --------------------------------------------------------------#
while read  des what mask iface; do
# 一些命令 ...
done < <(route -n)  
#    ^ ^  第一个 < 是重定向，第二个是进程替换。

#  为了测试，我们让它来做点儿事情。
while read  des what mask iface; do
  echo $des $what $mask $iface
done < <(route -n)  

# 输出内容:
# Kernel IP routing table
# Destination Gateway Genmask Flags Metric Ref Use Iface
# 127.0.0.0 0.0.0.0 255.0.0.0 U 0 0 0 lo
# --------------------------------------------------------------#

#  正如 Stéphane Chazelas 指出的,
#+ 一个更容易理解的等价代码如下：
route -n |
  while read des what mask iface; do   # 通过管道输出设置的变量
    echo $des $what $mask $iface
  done  #  这段代码的结果更上面的相同。
        #  但是，Ulrich Gayer 指出 . . .
        #+ 这段简化版等价代码在 while 循环里用了子 shell，
        #+ 因此当管道终止时变量都消失了。

# --------------------------------------------------------------#

#  然而，Filip Moritz 说上面的两个例子有一个微妙的区别，
#+ 见下面的代码

(
route -n | while read x; do ((y++)); done
echo $y # $y is still unset

while read x; do ((y++)); done < <(route -n)
echo $y # $y has the number of lines of output of route -n
)

#  更通俗地说（译者注：原文本行少了注释符）
(
: | x=x
# 似乎启动了子 shell ，就像
: | ( x=x )
# 而
x=x < <(:)
# 并没有。
)
#  这个方法在解析 csv 和类似格式时很有用。
#  也就是在效果上，原始 SuSE 系统的代码片段就是做这个用的。
```

注解 [1]
这个与命名管道（使用临时文件）的效果相同，而且事实上，进程替换也曾经用过命名管道。
