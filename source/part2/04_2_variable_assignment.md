# 4.2 变量赋值

### =

赋值操作符（在其前后没有空白符）。

> ![noitce](http://tldp.org/LDP/abs/images/caution.gif) 不要混淆 = 与 -eq，后者用来进行比较而非赋值。
> 
> 同时也要注意 = 根据使用场景既可作赋值操作符，也可作比较操作符。

样例 4-2. 变量赋值

```bash
#!/bin/bash
# 非引用形式变量

echo

# 什么时候变量是非引用形式，即变量名前没有 '$' 符号的呢？
# 当变量在被赋值而不是被引用时。

# 赋值
a=879
echo "The value of \"a\" is $a."

# 使用 'let' 进行赋值
let a=16+5
echo "The value of \"a\" is now $a."

echo

# 在 'for' 循环中赋值（隐式赋值）
echo -n "Values of \"a\" in the loop are: "
for a in 7 8 9 11
do
  echo -n "$a "
done

echo
echo

# 在 'read' 表达式中（另一种赋值形式）
echo -n "Enter \"a\" "
read a
echo "The value of \"a\" is now $a."

echo

exit 0
```

样例 4-3. 奇妙的变量赋值

```bash
#!/bin/bash

a=23              # 简单形式
echo $a
b=$a
echo $b

# 来我们玩点炫的（命令替换）。

a=`echo Hello!`   # 将 'echo' 命令的结果赋值给 'a'
echo $a
#  注意在命令替换结构中包含感叹号(!)在命令行中使用将会失效，
#+ 因为它将会触发 Bash 的历史(history)机制。
#  在shell脚本内，Bash 的历史机制默认关闭。

a=`ls -l`         # 将 'ls -l' 命令的结果赋值给 'a'
echo $a           # 不带引号引用，将会移除所有的制表符与分行符
echo
echo "$a"         # 引号引用变量将会保留空白符
                  # 查看 "引用" 章节。
                  
exit 0
```

使用 `$(...)` 形式进行赋值（与反引号不同的新形式），与命令替换形式相似。

```bash
# 摘自 /etc/rc.d/rc.local
R=$(cat /etc/redhat-release)
arch=$(uname -m)
```
