# 25. 别名

Bash `别名` 本质上不外乎是键盘上的快捷键，缩写呢是避免输入很长的命令串的一种手段.举个例子, 在 [~/.bashrc](http://tldp.org/LDP/abs/html/sample-bashrc.html) 文件中包含别名 `lm="ls -l | more`, 而后每个命令行输入的 lm [[1]](http://tldp.org/LDP/abs/html/aliases.html#FTN.AEN18669) 将会自动被替换成 `ls -l | more`. 这可以节省大量的命令行输入和避免记住复杂的命令和选项. 设定别名 `rm="rm -i"` (交互的删除模式) 防止无意的删除重要文件，也许可以少些悲痛.

脚本中别名作用十分有限. 如果别名可以有一些 C 预处理器的功能会更好, 例如宏扩展, 但不幸的是 bash 别名中没有扩展参数. [[2]](http://tldp.org/LDP/abs/html/aliases.html#FTN.AEN18676) 另外, 脚本在 "复合结构" 中并不能扩展自身的别名，例如 [if/then](http://tldp.org/LDP/abs/html/tests.html#IFTHEN), 循环和函数. 另一个限制是，别名不能递归扩展. 基本上是我们无论怎么喜欢用别名都不如函数 [function](http://tldp.org/LDP/abs/html/functions.html#FUNCTIONREF) 来的更有效.

样例 25-1. 脚本中的别名
```
#!/bin/bash
# alias.sh

shopt -s expand_aliases
# 必须设置此选项, 否则脚本不能别名扩展.


# 首先来点好玩的东西.
alias Jesse_James='echo "\"Alias Jesse James\" was a 1959 comedy starring Bob Hope."'
Jesse_James

echo; echo; echo;

alias ll="ls -l"
# 可以任意使用单引号 (') 或双引号 (") 把别名括起来.

echo "Trying aliased \"ll\":"
ll /usr/X11R6/bin/mk*   #* 别名可以运行.

echo

directory=/usr/X11R6/bin/
prefix=mk*  # See if wild card causes problems.
echo "Variables \"directory\" + \"prefix\" = $directory$prefix"
echo

alias lll="ls -l $directory$prefix"

echo "Trying aliased \"lll\":"
lll         # 所有 /usr/X11R6/bin 文件清单以 mk 开始.
# 别名可以处理连续的变量 -- 包含 wild card -- o.k.




TRUE=1

echo

if [ TRUE ]
then
  alias rr="ls -l"
  echo "Trying aliased \"rr\" within if/then statement:"
  rr /usr/X11R6/bin/mk*   #* 结果报错!
  # 别名在复合的表达式中并没有生效.
  echo "However, previously expanded alias still recognized:"
  ll /usr/X11R6/bin/mk*
fi  

echo

count=0
while [ $count -lt 3 ]
do
  alias rrr="ls -l"
  echo "Trying aliased \"rrr\" within \"while\" loop:"
  rrr /usr/X11R6/bin/mk*   #* 这里的别名也没生效.
                           #  alias.sh: 行 57: rrr: 命令未找到
  let count+=1
done 

echo; echo

alias xyz='cat $0'   # 列出了自身.
                     # 注意强引.
xyz
#  这看起来能工作,
#+ 尽管 bash 文档不介意这么做.
#
#  然而, Steve Jacobson 指出,
#+ "$0" 参数的扩展在上面的别名申明后立刻生效.

exit 0
```
取消别名的命令删除之前设置的别名.

样例 25-2. unalias: 设置和取消一个别名
```
#!/bin/bash
# unalias.sh

shopt -s expand_aliases  # 开启别名扩展.

alias llm='ls -al | more'
llm

echo

unalias llm              # 取消别名.
llm
# 'llm' 不再被识别后的报错信息.

exit 0
bash$ ./unalias.sh
total 6
drwxrwxr-x    2 bozo     bozo         3072 Feb  6 14:04 .
drwxr-xr-x   40 bozo     bozo         2048 Feb  6 14:04 ..
-rwxr-xr-x    1 bozo     bozo          199 Feb  6 14:04 unalias.sh

./unalias.sh: llm: 命令未找到
```

#### 注意
[[1]](http://tldp.org/LDP/abs/html/aliases.html#AEN18669)	... 作为命令行的第一个词. 显然别名只在命令的开始有意义.
[[2]](http://tldp.org/LDP/abs/html/aliases.html#AEN18676)	然而, 别名可用来扩展位置参数.
