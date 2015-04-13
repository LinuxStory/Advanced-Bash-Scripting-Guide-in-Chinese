##第五章 引用##

###本章目录###

* 5.1 引用变量
* 5.2 反义字符
* * * *
<p>
引用相当于将一个字符串用引号括起来。这样做的效果相当于保护了SHELL或者SHELL脚本中重新解释过或者有扩展功能的[特殊字符](http://tldp.org/LDP/abs/html/special-chars.html)（如果一个字符有其特殊解释而不仅仅是字面意义的话，那么这个字符就能称为“特殊字符”。比如星号“*”就能表示[正则表达式](http://tldp.org/LDP/abs/html/regexp.html#REGEXREF)中的一个[通配符](http://tldp.org/LDP/abs/html/globbingref.html)）。
<p>

	bash$ ls -l [Vv]*
	-rw-rw-r--    1 bozo  bozo       324 Apr  2 15:05 VIEWDATA.BAT
	-rw-rw-r--    1 bozo  bozo       507 May  4 14:25 vartrace.sh
	-rw-rw-r--    1 bozo  bozo       539 Apr 14 17:11 viewdata.sh

	bash$ ls -l '[Vv]*'
	ls: [Vv]*: No such file or directory
	#可以看到提示不存在该文件。这里的【Vv】*被当成了文件名
	#在日常的沟通和写作中，当我们引用一个短语的时候，我们会将它单独隔开并赋予它特殊的意义，
	#而在bash脚本中，当我们引用一个字符串，就会保留它的字面意义。

一些程序和公用的代码在被引用的字符串中重复解释或者有扩展功能的特殊字符串。所以引用的一个重用用途是保护SHELL中的命令行参数，但是仍然允许调用的程序扩展它。
	
	bash$ grep '[Ff]irst' *.txt
	file1.txt:This is the first line of file1.txt.
	file2.txt:This is the First line of file2.txt.
	#这里的功能是在txt文件中找到包含first或者First的文件

注意,如果使用了没加引号的 **grep [Ff]irst \*.txt**，除非当前目录下有文件名为First或者first的文件，否则就会失效，这也是引用的另一个原因。（感谢Harald Koenig指出了这一点）

当然引用同时能满足[echo](http://tldp.org/LDP/abs/html/internal.html#ECHOREF)爱好者的需求，它能把结果一行一行输出。

	bash$ echo $(ls -l)
	total 8 -rw-rw-r-- 1 bo bo 13 Aug 21 12:57 t.sh -rw-rw-r-- 1 bo bo 78 Aug 21 12:57 u.sh
	bash$ echo "$(ls -l)"
	total 8
	-rw-rw-r--  1 bo bo  13 Aug 21 12:57 t.sh
	-rw-rw-r--  1 bo bo  78 Aug 21 12:57 u.sh