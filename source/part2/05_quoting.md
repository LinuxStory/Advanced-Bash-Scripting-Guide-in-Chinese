# 第五章 引用

### 本章目录

- [5.1 引用变量](05_1_quoting_variables.md)
- [5.2 转义](05_2_escaping.md)

---

引用就是将一个字符串用引号括起来。这样做是为了保护Shell/Shell脚本中被重新解释过或带扩展功能的[特殊字符](http://tldp.org/LDP/abs/html/special-chars.html)（如果一个字符带有其特殊意义而不仅仅是字面量的话，这个字符就能称为“特殊字符”。比如星号“*”就能表示[正则表达式](http://tldp.org/LDP/abs/html/regexp.html#REGEXREF)中的一个[通配符](http://tldp.org/LDP/abs/html/globbingref.html)）。

```
bash$ ls -l [Vv]*
-rw-rw-r--    1 bozo  bozo       324 Apr  2 15:05 VIEWDATA.BAT
-rw-rw-r--    1 bozo  bozo       507 May  4 14:25 vartrace.sh
-rw-rw-r--    1 bozo  bozo       539 Apr 14 17:11 viewdata.sh

bash$ ls -l '[Vv]*'
ls: [Vv]*: No such file or directory
```

> 可以看到，提示不存在该文件。这里的`'[Vv]*`被当成了文件名。
> 在日常沟通和写作中，当我们引用一个短语的时候，我们会将它单独隔开并赋予它特殊的意义，而在bash脚本中，当我们*引用*一个字符串，意味着保留它的*字面量*。

很多程序和公用代码会展开被引用字符串中的特殊字符。引用的一个重用用途是保护Shell中的命令行参数，但仍然允许调用的程序扩展它。

```	
bash$ grep '[Ff]irst' *.txt
file1.txt:This is the first line of file1.txt.
file2.txt:This is the First line of file2.txt.
```
> 在所有.txt文件中找出包含first或者First字符串的行

注意，不加引号的 `grep [Ff]irst *.txt` 在Bash下也同样有效。[^1]

引用也可以控制[echo](http://tldp.org/LDP/abs/html/internal.html#ECHOREF)命令的断行符。

```
bash$ echo $(ls -l)
total 8 -rw-rw-r-- 1 bo bo 13 Aug 21 12:57 t.sh -rw-rw-r-- 1 bo bo 78 Aug 21 12:57 u.sh


bash$ echo "$(ls -l)"
total 8
 -rw-rw-r--  1 bo bo  13 Aug 21 12:57 t.sh
 -rw-rw-r--  1 bo bo  78 Aug 21 12:57 u.sh
```

[^1]: 前提是当前目录下有文件名为First或first的文件。这也是使用引用的另一个原因。（感谢 Harald Koenig 指出了这一点）

