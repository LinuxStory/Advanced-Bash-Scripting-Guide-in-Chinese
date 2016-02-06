# 33 选项

选项用来更改shell和脚本的行为.

[set](http://tldp.org/LDP/abs/html/internal.html#SETREF)命令用来打开脚本中的选项. 你可以在脚本中任何你想让选项生效的地方插入**set -o option-name**, 或者使用更简单的形式, **set -option-abbrev**. 这两种形式是等价的.
```
#!/bin/bash

set -o verbose
# # 打印出所有执行前的命令.
```

```
#!/bin/bash

set -v
# 与上边的例子具有相同的效果.
```

![extra](http://tldp.org/LDP/abs/images/note.gif) 如果你想在脚本中禁用某个选项, 可以使用**set +o option-name**或**set +option-abbrev**.

```
#!/bin/bash
set -o verbose
# 激活命令回显.
command
...
command

set +o verbose
# 禁用命令回显.
command
# 没有命令回显了.

set -v
# 激活命令回显.
command
...
command

set +v
# 禁用命令回显.
command

exit 0
```

还有另一种可以在脚本中启用选项的方法, 那就是在脚本头部, #!的后边直接指定选项.
```
#!/bin/bash -x
#
# 下边是脚本的主要内容.
```

也可以从命令行中打开脚本的选项. 某些不能与**set**命令一起用的选项就可以使用这种方法来打开. - i就是其中之一, 这个选项用来强制脚本以交互的方式运行.

**bash - v script - name**

**bash - o verbose script - name**

下表列出了一些有用的选项. 它们都可以使用缩写的形式来指定(开头加一个破折号), 也可以使用完整名字来指定(开头加上双破折号, 或者使用-o选项来指定).

表格 33-1. Bash选项

|缩写 	| 名称  			 |						作用	|
| :------------ | :------------ | :------------ |
|-B  		|brace expansion | 开启[大括号展开]()(默认 setting = on)	|
|+B   		|brace expansion | 关闭大括号展开	|
|-C   		|noclobber 		 |防止重定向时覆盖文件(可能会被>\|覆盖)	|
|-D   		|(none) 		 |列出用双引号引用起来的, 以$为前缀的字符串, 但是不执行脚本中的命令	|
|-a   		|all export		 |export(导出)所有定义过的变量	|
|-b   		|notify		 	 |当后台运行的作业终止时, 给出通知(脚本中并不常见)	|
|-c ...		|(none) 		 |从...中读取命令	|
|checkjobs	|(none) 	 	 |通知有活跃shell[任务](http://tldp.org/LDP/abs/html/x9644.html#JOBSREF)的用户退出。[Bash 4](http://tldp.org/LDP/abs/html/bashver4.html#BASH4REF)版本中引入，仍然处于"实验"阶段. 用法:shopt -s checkjobs .(注意：可能会hang！	|
|-e   		|errexit		 |当脚本发生第一个错误时, 就退出脚本, 换种说法就是, 当一个命令返回非零值时, 就退出脚本(除了[until](http://tldp.org/LDP/abs/html/loops1.html#UNTILLOOPREF)或[while loops](http://tldp.org/LDP/abs/html/loops1.html#WHILELOOPREF), [if-tests](http://tldp.org/LDP/abs/html/testconstructs.html#TESTCONSTRUCTS1), [list constructs](http://tldp.org/LDP/abs/html/list-cons.html#LCONS1))	|
|-f   		|noglob		 	 |禁用文件名扩展(就是禁用globbing)	|
|globstar|[globbing star-match](http://tldp.org/LDP/abs/html/bashver4.html#GLOBSTARREF)|打开[globbling](http://tldp.org/LDP/abs/html/globbingref.html)操作符(Bash [4+](http://tldp.org/LDP/abs/html/bashver4.html#BASH4REF)). 使用方法：shopt -s globstar	|
|-i   		|interactive	 |让脚本以交互模式运行	|
|-n   		|noexec		 	 |从脚本中读取命令, 但是不执行它们(做语法检查)	|
|-o Option-Name |(none) 	 |调用Option-Name选项	|
|-o posix   |POSIX		 	 |修改Bash或被调用脚本的行为, 使其符合[POSIX](http://tldp.org/LDP/abs/html/sha-bang.html#POSIX2REF)标准.	|
|-o pipefail|pipe failure	 |创建一个管道去返回最后一条命令的[退出状态码](http://tldp.org/LDP/abs/html/exit-status.html#EXITSTATUSREF)，这个返回值是一个非0的返回值	|
|-p   		|privileged		 |以"suid"身份来运行脚本(小心!)	|
|-r  		|restricted	 	 |以受限模式来运行脚本(参考 [22](http://tldp.org/LDP/abs/html/restricted-sh.html)).	|
|-s  		|stdin		 	 |从stdin 中读取命令	|
|-t   		|(none)		 	 |执行完第一个命令之后, 就退出	|
|-u   		|nounset		 |如果尝试使用了未定义的变量, 就会输出一个错误消息, 然后强制退出	|
|-v   		|verbose	 	 |在执行每个命令之前, 把每个命令打印到stdout上	|
|-x   		|xtrace		 	 |与-v选项类似, 但是会打印完整命令	|
|-   		|(none)		 	 |选项结束标志. 后面的参数为[位置参数](http://tldp.org/LDP/abs/html/internalvariables.html#POSPARAMREF).	|
|--   		|(none)		 	 |unset(释放)位置参数. 如果指定了参数列表(-- arg1 arg2), 那么位置 参数将会依次设置到参数列表中.	|



