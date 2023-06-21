# 16.5 文件与归档命令

## 归档命令

### tar

标准UNIX归档实用程序。[[1] ](https://tldp.org/LDP/abs/html/filearchiv.html#FTN.AEN11885)最初是一个*磁带归档*程序，现在已经发展成为一个通用软件包，可以处理所有类型的目标设备的各种归档方式，从磁带驱动器到常规文件，甚至是`标准输出(stdout)` (参阅[样例 3-4](https://tldp.org/LDP/abs/html/special-chars.html#EX58))。GNU *tar*程序已经成为能够接受各种压缩文件的过滤器，例如: **tar czvf archive_name.tar.gz ***，它递归地归档并[gzip](https://tldp.org/LDP/abs/html/filearchiv.html#GZIPREF)当前工作目录([$PWD](https://tldp.org/LDP/abs/html/internalvariables.html#PWDREF))下除[点文件](https://tldp.org/LDP/abs/html/basic.html#DOTFILESREF)之外的目录树中的所有文件。[[2]](https://tldp.org/LDP/abs/html/filearchiv.html#FTN.AEN11896)

一些有用的**tar**选项：

1. `-c`创建（一个新存档文件）

2. `-x`解析（现有存档文件）

3. `--delete`删除（现有存档文件）
   
   > ![note](https://tldp.org/LDP/abs/images/caution.gif)此选项在磁带设备上不起作用。

4. `-r`追加（文件到现有存档文件）

5. `-A`追加（*tar*文件到现有存档文件）

6. `-t`列出（现有存档文件的内容）

7. `-u`更新存档文件

8. `-d`将存档与指定的文件系统进行比较

9. `--after-date`仅处理指定日期*之后*带有日期戳的文件

10. `-z`[gzip](https://tldp.org/LDP/abs/html/filearchiv.html#GZIPREF)该存档文件
    
    (压缩还是解压缩，取决于与`-c`还是`-x`选项结合使用) 

11. `-j`[bzip2](https://tldp.org/LDP/abs/html/filearchiv.html#BZIPREF)该存档文件
    
    > ![note](https://tldp.org/LDP/abs/images/caution.gif)从损坏的*gzip压缩格式*的tar存档中恢复数据可能是一件很难的事情。当归档重要文件时，请进行多次备份。

### shar

*Shell归档*实用程序。能够使shell存档文件中的文本和/或二进制文件无需压缩即可连接，生成的存档文件本质上是一个shell脚本，带有 #!/bin/sh标头，包含所有必要的解压命令以及文件本身。目标文件中不可打印对的二进制字符将在输出的*shar*文件中转化为可打印的ASCII字符。幸好*Shar归档命令*仍在Usenet新闻组露面，否则**shar**会被**tar**/**gzip**命令取代。相反，**unshar**命令会解包*shar*存档文件。

**mailshar**命令是一个Bash脚本，它使用**shar**命令将多个文件连接成一个并发送电子邮件。这个脚本支持压缩和[uuencoding](https://tldp.org/LDP/abs/html/filearchiv.html#UUENCODEREF)。

### ar

用于存档文件的创建和操作的实用程序，主要用于二进制对象文件库。

### rpm

*红帽包管理器*，或是一个提供了源码或二进制存档文件包装器的rpm实用程序。除此之外，它还包括用于安装和检查软件包完整性的命令。

一个简单的 **rpm -i package_name.rpm**命令通常就能满足安装一个软件包的需求，除此之外还有更多可用的选项。

{% hint style="info" %}

<strong>`rpm -qf`</strong>标识文件来自哪个包。

```shell
bash$ rpm -qf /bin/ls
coreutils-5.2.1-31
```

{% endhint %}

{% hint style="info" %}

<strong>`rpm -qa`</strong>列举了给定系统上所有已安装的*rpm*包的完整列表。<strong>`rpm -qa package_name`</strong>仅列出与`package_name`相对应的软件包。

```shell
bash$ rpm -qa
redhat-logos-1.1.3-1
 glibc-2.2.4-13
 cracklib-2.7-12
 dosfstools-2.7-1
 gdbm-1.8.0-10
 ksymoops-2.4.1-1
 mktemp-1.5-11
 perl-5.6.0-17
 reiserfs-utils-3.x.0j-2
 ...


bash$ rpm -qa docbook-utils
docbook-utils-0.6.9-2


bash$ rpm -qa docbook | grep docbook
docbook-dtd31-sgml-1.0-10
 docbook-style-dsssl-1.64-3
 docbook-dtd30-sgml-1.0-10
 docbook-dtd40-sgml-1.0-11
 docbook-utils-pdf-0.6.9-2
 docbook-dtd41-sgml-1.0-10
 docbook-utils-0.6.9-2
```

{% endhint %}

### cpio

这种专门的归档复制命令 (复制输入和输出) 已被**tar**/**gzip**取代，现在很少见到。但是它仍然有其用途，例如移动目录树。指定适当的块大小 (用于复制) 后，它可能比**tar**快得多。

**样例 16-30. 使用*cpio*命令移动一个目录树**

```shell
#!/bin/bash

# 使用cpio命令复制一个目录树

# 使用'cpio'命令的优势:
#   复制速度的优势。它比管道中的'tar'命令快。
#   非常适合复制那些使用'cp'命令
#   可能性能很差的特殊文件 (命名管道等)

ARGS=2
E_BADARGS=65

if [ $# -ne "$ARGS" ]
then
  echo "Usage: `basename $0` source destination"
  exit $E_BADARGS
fi  

source="$1"
destination="$2"

###################################################################
find "$source" -depth | cpio -admvp "$destination"
#               ^^^^^         ^^^^^
#  请参阅'find'和'cpio'的info手册来了解这些选项的作用。
#  以上的工作仅与$PWD(当前目录)有关 . . .
#  指定完整路径名。
###################################################################


# 练习：
# --------
#  添加代码以检查'find | cpio'管道的退出状态($?)
#  以及如果出现任何问题，请输出适当的错误消息。

exit $?
```

### rpm2cpio

该命令会从[rpm包](https://tldp.org/LDP/abs/html/filearchiv.html#RPMREF)中解压cpio存档文件。

**样例 16-31. 解压一个*rpm*存档文件**

```shell
#!/bin/bash
# de-rpm.sh: 解压一个'rpm'存档文件

: ${1?"Usage: `basename $0` target-file"}
# 必须指定一个'rpm'存档文件名作为参数。


TEMPFILE=$$.cpio                         #  名称“特定的”临时文件。
                                         #  $$代表该脚本的进程ID。

rpm2cpio < $1 > $TEMPFILE                #  将rpm存档文件
                                         #  转化为cpio存档文件。
cpio --make-directories -F $TEMPFILE -i  #  解压cpio存档文件。
rm -f $TEMPFILE                          #  删除cpio存档文件。

exit 0

#  练习：
#  添加以下功能代码         1) 检查“目标文件”存在并且
#                         2) 是一个rpm存档文件。
#  提示：                     解析'file'命令的输出。
```

### pax

pax便携式存档交换工具包可促进定期文件备份，并旨在在各种版本的UNIX之间交叉兼容。它旨在取代[tar](https://tldp.org/LDP/abs/html/filearchiv.html#TARREF)和[cpio](https://tldp.org/LDP/abs/html/filearchiv.html#CPIOREF)命令。

```shell
pax -wf daily_backup.pax ~/linux-server/files 
#  创建目标目录中所有文件的tar存档。
#  请注意，pax命令的选项必须顺序正确 --
#  pax -fw     会起到完全不同的作用。

pax -f daily_backup.pax
#  列出存档中的文件。

pax -rf daily_backup.pax ~/bsd-server/files
#  将Linux机器上的备份文件恢复到BSD机器上。
```

请注意，*pax*可以处理许多标准的归档和压缩命令。

## 压缩命令

### gzip

标准的GNU/UNIX压缩实用程序，取代了劣等的和专有的**compress**命令。对应的解压缩命令是**gunzip**，等效于**gzip -d**。

> ![note](https://tldp.org/LDP/abs/images/note.gif)`-c`选项会将**gzip**的输出传递给`标准输出(stdout)`。当使用[管道](https://tldp.org/LDP/abs/html/special-chars.html#PIPEREF)传递给其他命令时这很有用。

**zcat**过滤器将*gzip*压缩文件解压缩到`标准输出(stdout)`，可能作为管道的输入或重定向。实际上，这是一个用于压缩文件 (包括使用较旧的[compress](https://tldp.org/LDP/abs/html/filearchiv.html#COMPRESSREF)实用工具处理的文件) 的**cat**命令。**zcat**命令等同于**gzip -dc**。

> ![note](https://tldp.org/LDP/abs/images/caution.gif)在某些商业UNIX系统上，**zcat**是**uncompress -c**的同义词，并且在*gzip压缩文件*上不起作用。

另请[样例 7-7](https://tldp.org/LDP/abs/html/comparison-ops.html#EX14)。

### bzip2

另一个压缩实用程序，通常比**gzip**效率更高 (但速度较慢)，尤其是在大文件上。其对应的解压缩命令是**bunzip2**。

类似于**zcat**命令，**bzcat**会解压一个*bzipped2*文件到`标准输出(stdout)`。

> ![note](https://tldp.org/LDP/abs/images/note.gif)较新版本的[tar](https://tldp.org/LDP/abs/html/filearchiv.html#TARREF)命令已经支持**bzip2**。

### compress, uncompress

这是在商业UNIX发行版中发现的较旧的专有压缩实用程序。效率更高的**gzip**已在很大程度上取代了它。Linux发行版通常包含**compress**以实现兼容性，尽管**gunzip**可以解压缩**compress**处理的存档文件。

{% hint style="info" %}

**znew**命令能够将*compress*压缩文件转换为*gzip*压缩文件。

{% endhint %}

### sq

这是另一个压缩实用程序，仅对有序的[ASCII](https://tldp.org/LDP/abs/html/special-chars.html#ASCIIDEF)单词列表起作用。它使用过滤器的标准调用语法，**sq < input-file > output-file**。速度快，但效率不及[gzip](https://tldp.org/LDP/abs/html/filearchiv.html#GZIPREF)。相应的解压缩过滤器是**unsq**，像**sq**一样调用即可。

{% hint style="info" %}

**sq**的输出可以通过管道传输到**gzip**以进行进一步压缩。

{% endhint %}

### zip, unzip

与DOS *pkzip.exe*兼容的跨平台文件归档和压缩实用程序。与 “tarballs” 相比，"zip"存档文件似乎是互联网上更常见的文件交换媒介。

### unarc, unarj, unrar

这些Linux实用程序能够解包使用DOS平台上*arc.exe*、*arj.exe*以及*rar.exe*程序压缩的存档文件。

### lzma, unlzma, lzcat

采用高效的Lempel-Ziv-Markov压缩算法。*lzma*的算法与*gzip*类似。[7-zip网站](http://www.7-zip.org/sdk.html)上有更多信息可供参阅。

### xz, unxz, xzcat

一种新的高效压缩工具，向后兼容*lzma*，并具有类似于*gzip*的调用语法。有关更多信息，请参见[维基百科条目](http://en.wikipedia.org/wiki/Xz)。

## 文件信息命令

### file

用于识别文件类型的实用程序。命令<code>**file file-name**</code>会返回`file-name`的文件规范，例如ascii文本或数据。它引用自`/usr/share/magic`、`/etc/magic`或`/usr/lib/magic`中找到的[幻数(magic number)](https://tldp.org/LDP/abs/html/sha-bang.html#MAGNUMREF)，取决于当前Linux/UNIX发行版的具体情况。

`-f`选项会时**file**命令以[批处理](https://tldp.org/LDP/abs/html/timedate.html#BATCHPROCREF)模式运行，并从指定文件中读取要分析的文件名列表。当在一个压缩文件上使用`-z`选项时，会强制尝试分析未压缩文件的类型。

```shell
bash$ file test.tar.gz
test.tar.gz: gzip compressed data, deflated,
 last modified: Sun Sep 16 13:34:51 2001, os: Unix

bash file -z test.tar.gz
test.tar.gz: GNU tar archive (gzip compressed data, deflated,
 last modified: Sun Sep 16 13:34:51 2001, os: Unix)
```

```shell
# 从给定目录下查找sh和Bash脚本：

DIRECTORY=/usr/local/bin
KEYWORD=Bourne
# Bourne和Bourne-Again shell脚本

file $DIRECTORY/* | fgrep $KEYWORD

# 输出:

# /usr/local/bin/burn-cd:          Bourne-Again shell script text executable
# /usr/local/bin/burnit:           Bourne-Again shell script text executable
# /usr/local/bin/cassette.sh:      Bourne shell script text executable
# /usr/local/bin/copy-cd:          Bourne-Again shell script text executable
# . . .
```

**样例 16-32. 去除C程序文件注释**

```shell
#!/bin/bash
# strip-comment.sh: 去除C程序文件注释(/* COMMENT */)

E_NOARGS=0
E_ARGERROR=66
E_WRONG_FILE_TYPE=67

if [ $# -eq "$E_NOARGS" ]
then
  echo "Usage: `basename $0` C-program-file" >&2 # 标准错误(stderr)输出错误信息。
  exit $E_ARGERROR
fi  

# 测试文件类型是否准确。
type=`file $1 | awk '{ print $2, $3, $4, $5 }'`
# "file $1" 输出文件类型 . . .
# awk命令移除第一个字段，文件名 . . .
# 结果赋值给变量"type"。
correct_type="ASCII C program text"

if [ "$type" != "$correct_type" ]
then
  echo
  echo "This script works on C program files only."
  echo
  exit $E_WRONG_FILE_TYPE
fi  


# 相当神秘的sed脚本：
#--------
sed '
/^\/\*/d
/.*\*\//d
' $1
#--------
# 如果你花几个小时学习sed基础知识，这很容易理解。


#  需要在sed脚本中再添加一行，
#  以处理代码行的同一行后面有注释的情况。
#  这是一项非同寻常的练习。

#  此外，上面的代码删除了带有"*/"的非注释的行 . . .
#  这并不是一个预期的结果。

exit 0


# ----------------------------------------------------------------
# 由于上面的'exit 0'，该行以下的代码将无法执行。

# Stephane Chazelas提出以下的替代方案：

usage() {
  echo "Usage: `basename $0` C-program-file" >&2
  exit 1
}

WEIRD=`echo -n -e '\377'`   # 或者 WEIRD=$'\377'
[[ $# -eq 1 ]] || usage
case `file "$1"` in
  *"C program text"*) sed -e "s%/\*%${WEIRD}%g;s%\*/%${WEIRD}%g" "$1" \
     | tr '\377\n' '\n\377' \
     | sed -ne 'p;n' \
     | tr -d '\n' | tr '\377' '\n';;
  *) usage;;
esac

#  这依然会被这样的代码欺骗:
#  printf("/*");
#  或者
#  /*  /* buggy embedded comment */
#
#  为了应对这种情况(在字符串里的注释， 
#  在字符中带有\", \\"的注释 ...)，
#  唯一的方法是写一个C解析器 (可能使用lex或yacc)？

exit 0
```

### which

**which command**给出了**command**的完整路径。这对于找出系统上是否安装了特定命令或实用程序很有帮助。

<code>**\$bash which rm**</code>

```shell
/usr/bin/rm
```

如果你想了解该命令有趣的使用方法，请参阅[样例 36-16](https://tldp.org/LDP/abs/html/colorizing.html#HORSERACE)。

### whereis

类似于以上提到的**which**命令，**whereis command**给出了**command**的完整路径，但同时还有它的[man手册](https://tldp.org/LDP/abs/html/basic.html#MANREF)。

<code>**\$bash whereis rm**</code>

```shell
rm: /bin/rm /usr/share/man/man1/rm.1.bz2
```

### whatis

**whatis command**会从`whatis`数据库中寻找**command**的相关信息。这对于识别系统命令和重要的配置文件很有用。可以将其视为简化的**man**命令。

<code>**\$bash whatis whatis**</code>

```shell
whatis               (1)  - search the whatis database for complete words
```

**样例 16-33. 浏览`/usr/X11R6/bin`**

```shell
#!/bin/bash

# /usr/X11R6/bin中那些神秘的二进制文件是什么?

DIRECTORY="/usr/X11R6/bin"
# 也可以尝试"/bin"、"/usr/bin"、"/usr/local/bin"等等。

for file in $DIRECTORY/*
do
  whatis `basename $file`   # 输出有关该二进制文件的信息。
done

exit 0

#  注意：为确保该命令有效，
#  你必须使用/usr/sbin/makewhatis创建一个"whatis"数据库。
#  您可能希望重定向此脚本的输出，就像这样:
#    ./what.sh >>whatis.db
#  或者在标准输出(stdout)一页上查看结果，
#    ./what.sh | less
```

另请参阅[样例 11-3](https://tldp.org/LDP/abs/html/loops1.html#FILEINFO)。

### vdir

显示详细的目录列表。效果类似于[ls -lb](https://tldp.org/LDP/abs/html/basic.html#LSREF)。

这是其中一个GNU *fileutil*。

```shell
bash$ vdir
total 10
 -rw-r--r--    1 bozo  bozo      4034 Jul 18 22:04 data1.xrolo
 -rw-r--r--    1 bozo  bozo      4602 May 25 13:58 data1.xrolo.bak
 -rw-r--r--    1 bozo  bozo       877 Dec 17  2000 employment.xrolo

bash ls -l
total 10
 -rw-r--r--    1 bozo  bozo      4034 Jul 18 22:04 data1.xrolo
 -rw-r--r--    1 bozo  bozo      4602 May 25 13:58 data1.xrolo.bak
 -rw-r--r--    1 bozo  bozo       877 Dec 17  2000 employment.xrolo
```

### locate, slocate

**locate**命令会前往专门的数据库中搜索文件。**slocate**命令是**locate**命令的安全版本 (也可能取别名为**slocate**)。

<code>**$bash locate hickson**</code>

```shell
/usr/lib/xephem/catalogs/hickson.edb
```

### getfacl, setfacl

这些命令*检索*或*设置*文件访问控制列表 -- *所有者*、*属组*和文件权限。

```shell
bash$ getfacl *
# file: test1.txt
 # owner: bozo
 # group: bozgrp
 user::rw-
 group::rw-
 other::r--

 # file: test2.txt
 # owner: bozo
 # group: bozgrp
 user::rw-
 group::rw-
 other::r--



bash$ setfacl -m u:bozo:rw yearly_budget.csv
bash$ getfacl yearly_budget.csv
# file: yearly_budget.csv
 # owner: accountant
 # group: budgetgrp
 user::rw-
 user:bozo:rw-
 user:accountant:rw-
 group::rw-
 mask::rw-
 other::r--
```

### readlink

查看符号链接指向的文件。

```shell
bash$ readlink /usr/bin/awk
../../bin/gawk
```

### strings

使用**strings**命令可以在二进制文件或数据文件中查找可打印字符串。它将列出在目标文件中找到的可打印字符序列。这便于快速检查核心转储(core dump)或查看未知图像文件 (<code>**strings image-file | more**</code>可能显示类似*JFIF*的东西，那么该文件标识为*jpeg*图形) 。在脚本中，也可以使用[grep](https://tldp.org/LDP/abs/html/textproc.html#GREPREF)或[sed](https://tldp.org/LDP/abs/html/sedawk.html#SEDREF)命令解析**strings**的输出。请参阅[样例 11-8](https://tldp.org/LDP/abs/html/loops1.html#BINGREP)和[样例 11-10](https://tldp.org/LDP/abs/html/loops1.html#FINDSTRING)。

**样例 16-34. 一个“改进的”*strings*命令**

```shell
#!/bin/bash
# wstrings.sh: "word-strings" (增强的 "strings" 命令)
#
#  此脚本通过对照标准单词列表文件
#  对"strings"的输出进行筛选。
#  这有效地消除了胡言乱语和噪音，
#  并且只输出可识别的单词。

# ===========================================================
#                 脚本参数的标准检查
ARGS=1
E_BADARGS=85
E_NOFILE=86

if [ $# -ne $ARGS ]
then
  echo "Usage: `basename $0` filename"
  exit $E_BADARGS
fi

if [ ! -f "$1" ]                      # 检查文件是否存在。
then
    echo "File \"$1\" does not exist."
    exit $E_NOFILE
fi
# ===========================================================


MINSTRLEN=3                           #  最小字符串长度。
WORDFILE=/usr/share/dict/linux.words  #  字典文件。
#  可以指定每行一个单词格式的单词列表文件。
#  例如，"yawl" 单词列表包，
#  http://bash.deta.in/yawl-0.3.2.tar.gz


wlist=`strings "$1" | tr A-Z a-z | tr '[:space:]' Z | \
       tr -cs '[:alpha:]' Z | tr -s '\173-\377' Z | tr Z ' '`

# 用'tr'的多次传递来翻译‘strings’命令的输出。
#  "tr A-Z a-z"  转换为小写字母。
#  "tr '[:space:]'" 将空格转换为'Z'。
#  "tr -cs '[:alpha:]' Z" 将非字母字符转化为Z，
#  并且将多个连续的'Z'压缩为一个。
#  "tr -s '\173-\377' Z"  将所有在'z'之前的字母转换为'Z'
#  并且将多个连续的'Z'压缩为一个，
#  从而消除那些之前转换中没能成功处理的奇怪字符。
#  最后，"tr Z ' '"将所有的'Z'转换为空白字符，
#  在以下的循环中空白字符将成为单词之间的分隔符。

#  ***********************************************************************
#  请注意将'tr'的输出馈送/管道传输回自身的方法，
#  在每次连续传输时具有不同的参数和/或选项。
#  ***********************************************************************


for word in $wlist                    #  重点：
                                      #  $wlist不能打引号。
                                      # "$wlist"无效
                                      #  这是为什么？
do
  strlen=${#word}                     #  字符串长度。
  if [ "$strlen" -lt "$MINSTRLEN" ]   #  跳过短字符串。
  then
    continue
  fi

  grep -Fw $word "$WORDFILE"          #   仅进行全字匹配。
#      ^^^                            #  "固定字符串" 和
                                      #  "全字"选项。 
done  

exit $?
```

## 比较命令

### diff, patch

**diff**: 灵活的文件比较实用程序。它按顺序逐行比较目标文件。在某些应用程序中，例如比较单词词典，在将它们输送到**diff**之前，建议先通过[sort](https://tldp.org/LDP/abs/html/textproc.html#SORTREF)和**uniq**进行过滤。<code>**diff file-1 file-2**</code>输出文件中不同的行，并带有插入符以显示每个特定的行属于哪个文件。

**diff**命令的`--side-by-side`选项在单独的列中逐行输出每个比较的文件，并标出不匹配的行。`-c`和`-u`选项同样使命令的输出更易于理解。

**diff**有各种优质的前端程序(frontend)，例如**sdiff**、**wdiff**、**xdiff**和**mgdiff**。

{% hint style="info" %}

如果比较的文件相同，则**diff**命令返回0的退出状态，如果它们不同则返回1的退出状态（比较*二进制*文件时返回2）。这允许在shell脚本中的测试结构中使用**diff**命令（见下文）。

{% endhint %}

**diff**命令的常见用途是生成要与**patch**命令一起使用的差异文件。`-e`选项输出适合**ed**或**ex**脚本的文件。

**patch**：灵活的版本控制实用程序。给定一个由**diff**命令生成的差异文件，**patch**可以将先前版本的软件包(package)升级到新版本。相较分发一整个新修订的软件包，分发一个相对较小的"diff"文件要方便得多。内核"patch"已成为分发Linux内核版本的首选方法。

```shell
patch -p1 <patch-file
# 接受'patch-file'中列出的所有更改，
# 并将其应用于引用其的文件。
# 这将使软件包升级到新版本。
```

给内核打补丁：

```shell
cd /usr/src
gzip -cd patchXX.gz | patch -p0
# 使用'patch'更新内核源码。
# 摘自Linux内核文档"README"，
# 来自匿名作者(Alan Cox?)。
```

> ![note](https://tldp.org/LDP/abs/images/note.gif)**diff**命令还可以递归地比较目录 (对于目录中的文件名)。

```shell
bash$ diff -r ~/notes1 ~/notes2
Only in /home/bozo/notes1: file02
 Only in /home/bozo/notes1: file03
 Only in /home/bozo/notes2: file04
```

{% hint style="info" %}

使用**zdiff**命令比较*gzip*文件。

{% endhint %}

{% hint style="info" %}

使用**diffstat**命令创建来自**diff**命令输出的直方图（点分布图）。

{% endhint %}

### diff3, merge

**diff**的扩展版本能够一次比较三个文件。此命令在成功执行后返回0的退出值，但不幸的是，并没有提供有关比较结果的信息。

```shell
bash$ diff3 file-1 file-2 file-3
====
 1:1c
   This is line 1 of "file-1".
 2:1c
   This is line 1 of "file-2".
 3:1c
   This is line 1 of "file-3"
```

**merge**（三向文件合并）命令是*diff3*命令的有趣附件。它的语法是**merge Mergefile file1 file2**。从`file1`到`file2`的更改将输出到`Mergefile`。可以将此命令视为*patch*命令的精简版本。

### sdiff

比较和/或编辑两个文件，以便将它们合并到输出文件中。由于其互动性，该命令在脚本中几乎没有用处。

### cmp

**cmp**命令是上述**diff**命令的一个更简单的版本。**diff**报告两个文件之间的差异，而**cmp**仅显示它们的差异点。

> ![note](https://tldp.org/LDP/abs/images/note.gif)类似于**diff**命令，当两个文件相同时**cmp**返回0的退出状态，当不同时返回1的退出状态。这允许在shell脚本中的测试结构中使用。

**样例 16-35. 使用*cmp*在脚本中比较两个文件**

```shell
#!/bin/bash
# file-comparison.sh

ARGS=2  # 该脚本期望两个参数。
E_BADARGS=85
E_UNREADABLE=86

if [ $# -ne "$ARGS" ]
then
  echo "Usage: `basename $0` file1 file2"
  exit $E_BADARGS
fi

if [[ ! -r "$1" || ! -r "$2" ]]
then
  echo "Both files to be compared must exist and be readable."
  exit $E_UNREADABLE
fi

cmp $1 $2 &> /dev/null
#   "cmp"的输出重定向至/dev/null
#   cmp -s $1 $2  有着相同效果 ("-s"是"cmp"命令的静默标志)
#   感谢Anders Gustavsson的指出。
#
#  即同样适用于'diff'命令，
#  diff $1 $2 &> /dev/null

if [ $? -eq 0 ]         # 测试"cmp"命令的退出状态。
then
  echo "File \"$1\" is identical to file \"$2\"."
else  
  echo "File \"$1\" differs from file \"$2\"."
fi

exit 0
```

{% hint style="info" %}

比较*gzip*文件时请使用**zcmp**。

{% endhint %}

### comm

多功能文件比较实用程序。使用前文件必须是有序的。

<strong>comm <em><code>-options first-file second-file</code></em></strong>

<code>**comm file-1 file-2**</code>输出三列：

- 第 1 列：`file-1`特有的行

- 第 2 列：`file-2`特有的行

- 第 3 列：共有的行

选项允许隐藏一列或多列的输出。

- `-1` 隐藏第`1`列

- `-2` 隐藏第`2`列

- `-3` 隐藏第`3`列

- `-12` 隐藏第`1`和第`2`列，以此类推

在比较“字典”和*单词列表*（有序的一行一个单词的文本文件）时该命令很实用。

## 实用工具

### basename

从文件名中剥离路径信息，仅打印文件名。<code>**basename \$0**</code>能够让脚本知道它的名字，也就是它被调用的名字。这可以用于提示脚本的"使用方法"，例如，如果调用脚本时缺少参数:

```shell
echo "Usage: `basename $0` arg1 arg2 ... argn"
```

### dirname

从文件名中剥离**basename**，仅打印路径信息。

> ![note](https://tldp.org/LDP/abs/images/note.gif)**basename**和**dirname**可以对任意字符串进行操作。该参数不需要引用现有文件，甚至不需要引用该文件的文件名。请参阅[样例 A-7](https://tldp.org/LDP/abs/html/contributed-scripts.html#DAYSBETWEEN)。

**样例 16-36. *basename*和*dirname***

```shell
#!/bin/bash

address=/home/bozo/daily-journal.txt

echo "Basename of /home/bozo/daily-journal.txt = `basename $address`"
echo "Dirname of /home/bozo/daily-journal.txt = `dirname $address`"
echo
echo "My own home is `basename ~/`."         # `basename ~` 同样有效。
echo "The home of my home is `dirname ~/`."  # `dirname ~`  同样有效。

exit 0
```

### split, csplit

这些是用于将文件拆分为较小的块的实用程序。它们通常用于拆分大文件，以便将它们备份在软盘上或者准备通过电子邮件发送或上传。

**csplit**命令根据*上下文*拆分文件，当模式匹配成功时则进行拆分。

**样例 16-37. 将自己分区域复制的脚本**

```shell
#!/bin/bash
# splitcopy.sh

#  这是一个将自己拆分为块，
#  然后将这些块重新组合为原脚本拷贝件的脚本。

CHUNKSIZE=4    #  第一个所拆分文件块的大小。
OUTPREFIX=xx   #  默认的"csplit"的前缀为"xx"

csplit "$0" "$CHUNKSIZE"

# 一些用于填充的注释行 . . .
# Line 15
# Line 16
# Line 17
# Line 18
# Line 19
# Line 20

cat "$OUTPREFIX"* > "$0.copy"  # 连接这些块。
rm "$OUTPREFIX"*               # 删除这些块。

exit $?
```

### 加密与解密命令

### sum, cksum, md5sum, sha1sum

这些是生成*校验和*的实用程序。*校验和*是根据文本内容进行数学计算后的数字[[3]](https://tldp.org/LDP/abs/html/filearchiv.html#FTN.AEN12840)，目的是检查其完整性。出于安全目的，脚本可能会引用校验和列表，例如确保关键系统的文件内容没有被更改或损坏。对于安全应用程序，请使用**md5sum** (消息摘要 5 校验和) 命令，或者更优的**sha1sum** (安全哈希算法)。[[4]](https://tldp.org/LDP/abs/html/filearchiv.html#FTN.AEN12849)

```shell
bash$ cksum /boot/vmlinuz
1670054224 804083 /boot/vmlinuz

bash$ echo -n "Top Secret" | cksum
3391003827 10



bash$ md5sum /boot/vmlinuz
0f43eccea8f09e0a0b2b5cf1dcf333ba  /boot/vmlinuz

bash$ echo -n "Top Secret" | md5sum
8babc97a6f62a4649716f4df8d61728f  -
```

> ![note](https://tldp.org/LDP/abs/images/note.gif)**cksum**命令会以byte的形式显示目标文件或`标准输出(stdout)`的大小。
> 
> 当**md5sum**和**sha1sum**命令从`标准输出(stdout)`中接受输入时会显示一个[短划线](https://tldp.org/LDP/abs/html/special-chars.html#DASHREF2)。

**样例 16-38. 检查文件完整性**

```shell
#!/bin/bash
# file-integrity.sh: 检查给定目录中的文件是否被篡改。


E_DIR_NOMATCH=80
E_BAD_DBFILE=81

dbfile=File_record.md5
# 用于存储记录的文件名 (数据库文件)。


set_up_database ()
{
  echo ""$directory"" > "$dbfile"
  # 在文件第一行写入目录名。
  md5sum "$directory"/* >> "$dbfile"
  # 追加md5校验和和文件名。
}

check_database ()
{
  local n=0
  local filename
  local checksum

  # ------------------------------------------- #
  #  此文件检查应该是不必要的，
  #  但总好过颔首道歉。

  if [ ! -r "$dbfile" ]
  then
    echo "Unable to read checksum database file!"
    exit $E_BAD_DBFILE
  fi
  # ------------------------------------------- #

  while read record[n]
  do

    directory_checked="${record[0]}"
    if [ "$directory_checked" != "$directory" ]
    then
      echo "Directories do not match up!"
      # 试图将文件用于不同的目录。
      exit $E_DIR_NOMATCH
    fi

    if [ "$n" -gt 0 ]   # 不是目录名。
    then
      filename[n]=$( echo ${record[$n]} | awk '{ print $2 }' )
      #  md5sum向后写入记录，
      #  首先是checksum，然后是filename。
      checksum[n]=$( md5sum "${filename[n]}" )


      if [ "${record[n]}" = "${checksum[n]}" ]
      then
        echo "${filename[n]} unchanged."

        elif [ "`basename ${filename[n]}`" != "$dbfile" ]
               #  跳过校验和数据库文件，
               #  因为它会随着脚本的每次调用而改变。
               #  ---
               #  不幸的是，这意味着在$PWD上运行此脚本时，
               #  不会检测到校验和数据库文件的篡改。
               #  练习：请修复这个问题。
        then
          echo "${filename[n]} : CHECKSUM ERROR!"
        # 自从上次检查前文件已经被更改。
        fi

      fi



    let "n+=1"
  done <"$dbfile"       # 从校验和数据库中读取。

}  

# =================================================== #
# main ()

if [ -z  "$1" ]
then
  directory="$PWD"      #  如果没有指定，
else                    #  就使用当前工作目录。
  directory="$1"
fi  

clear                   # 清屏。
echo " Running file integrity check on $directory"
echo

# ------------------------------------------------------------------ #
  if [ ! -r "$dbfile" ] # 需要创建一个数据库文件？
  then
    echo "Setting up database file, \""$directory"/"$dbfile"\"."; echo
    set_up_database
  fi  
# ------------------------------------------------------------------ #

check_database          # 做实际的工作。

echo 

#  你有可能希望将此脚本的输出重定向到一个文件，
#  特别是目录中有许多文件需要检查时。

exit 0

#  为了更彻底的文件完整性检查，
#  你可以考虑使用"Tripwire"包，
#  http://sourceforge.net/projects/tripwire/。
```

了解更多**md5sum**命令的有趣用法，另请参阅[样例 A-19](https://tldp.org/LDP/abs/html/contributed-scripts.html#DIRECTORYINFO)、[样例 36-16](https://tldp.org/LDP/abs/html/colorizing.html#HORSERACE)和[样例 10-2](https://tldp.org/LDP/abs/html/string-manipulation.html#RANDSTRING)。

> ![note](https://tldp.org/LDP/abs/images/note.gif)有报道称，128位**md5sum**可以被破解，因此更安全的160位**sha1sum**是校验和工具包的一个受欢迎的新成员。

```shell
bash$ md5sum testfile
e181e2c8720c60522c4c4c981108e367  testfile


bash$ sha1sum testfile
5d7425a9c08a66c3177f1e31286fa40986ffc996  testfile
```

安全顾问已经证明即使是**sha1sum**也可能被破解。幸运的是，较新的Linux发行版包括更长加密位的**sha224sum**、**sha256sum**、**sha384sum**和**sha512sum**命令。

### uuencode

该实用程序将二进制文件 (图像，声音文件，压缩文件等) 编码为[ASCII](https://tldp.org/LDP/abs/html/special-chars.html#ASCIIDEF)字符，使其适合在电子邮件正文或新闻组发布中传输。当在MIME (多媒体) 编码不可用的情况下尤其有用。

### uudecode

它反转了编码过程，解码*uuencode*编码文件成为原先的二进制文件。

**样例 16-39. 使用uudecode解码uuencode编码文件**

```shell
#!/bin/bash
# 在当前的工作目录下使用uudecode解码所有使用uuencode编码的文件。

lines=35        # 允许35行的头部空间（非常慷慨）。

for File in *   # 测试在$PWD下的所有文件。
do
  search1=`head -n $lines $File | grep begin | wc -w`
  search2=`tail -n $lines $File | grep end | wc -w`
  #  uuencode编码文件在接近开头的部分有一个"begin"字段。
  #  以及在接近末尾的部分有一个"end"字段。
  if [ "$search1" -gt 0 ]
  then
    if [ "$search2" -gt 0 ]
    then
      echo "uudecoding - $File -"
      uudecode $File
    fi  
  fi
done  

#  请注意，运行该脚本会误认为它自己也是uuencode编码文件，
#  因为它同时包含 “begin” 和 “end”。

#  练习：
#  --------
#  修改此脚本以检查每个文件中的新闻组标题，
#  如果找不到，请跳至下一步。

exit 0
```

{% hint style="info" %}

[fold -s](https://tldp.org/LDP/abs/html/textproc.html#FOLDREF)命令可能便于处理（可能在管道中）从Usenet新闻组下载的长uudecode文本消息。

{% endhint %}

### mimencode, mmencode

**mimencode**和**mmencode**命令用于处理多媒体编码的电子邮件附件。尽管*邮件用户代理服务器* (例如*pine*或*kmail*) 通常会自动处理此操作，但这些特定实用程序允许通过shell脚本从命令行或[批处理模式](https://tldp.org/LDP/abs/html/timedate.html#BATCHPROCREF)中手动操作此类附件。

### crypt

曾几何时，这是标准的UNIX文件加密实用程序。[[5]](https://tldp.org/LDP/abs/html/filearchiv.html#FTN.AEN12969)出于政治动机的政府法规禁止加密软件的出口，导致**crypt**从许多UNIX世界中消失，并且在大多数Linux发行版中仍然缺少它。幸运的是，程序员已经提出了许多不错的替代方案，其中包括作者自己的[作品]([ftp://metalab.unc.edu/pub/Linux/utils/file/cruft-0.2.tar.gz](ftp://metalab.unc.edu/pub/Linux/utils/file/cruft-0.2.tar.gz)) (请参见[样例 A-4](https://tldp.org/LDP/abs/html/contributed-scripts.html#ENCRYPTEDPW))。

### openssl

这是<em>安全套接字层(Secure Sockets Layer)</em>加密的开源实现。

```shell
# 加密一个文件：
openssl aes-128-ecb -salt -in file.txt -out file.encrypted \
-pass pass:my_password
#          ^^^^^^^^^^^   用户选择的密码。
#       aes-128-ecb      是选择的加密方法。

# 解密一个openssl加密的文件：
openssl aes-128-ecb -d -salt -in file.encrypted -out file.txt \
-pass pass:my_password
#          ^^^^^^^^^^^   用户选择的密码。
```

将*openssl*[管道传输](https://tldp.org/LDP/abs/html/special-chars.html#PIPEREF)到/从[tar](https://tldp.org/LDP/abs/html/filearchiv.html#TARREF)可以使加密整个目录树成为可能。

```shell
# 加密整个目录：

sourcedir="/home/bozo/testfiles"
encrfile="encr-dir.tar.gz"
password=my_secret_password

tar czvf - "$sourcedir" |
openssl des3 -salt -out "$encrfile" -pass pass:"$password"
#       ^^^^   使用des3加密方法
# 在当前工作目录中写入加密文件"encr-dir.tar.gz"。

# 解密生成的tarball：
openssl des3 -d -salt -in "$encrfile" -pass pass:"$password" |
tar -xzv
# 解密并解包到当前工作目录下。
```

当然，*openssl*有其他更多用法，例如获取网站的签名*证书*。请参阅[info](https://tldp.org/LDP/abs/html/basic.html#INFOREF)手册。

### shred

通过在删除文件之前用随机位模式多次覆盖文件来安全地擦除它。此命令与[样例 16-61](https://tldp.org/LDP/abs/html/extmisc.html#BLOTOUT)具有相同的效果，但以更彻底和优雅的方式进行。

这是其中一个*GNU* fileutils。

> ![note](https://tldp.org/LDP/abs/images/caution.gif)即使应用了**shred**，先进的取证技术仍可能恢复文件的内容。

## 杂项命令

### mktemp

使用一个“独有”的文件名创建一个*临时文件*[[6]](https://tldp.org/LDP/abs/html/filearchiv.html#FTN.AEN13030)。当不带任何参数通过命令行调用时，它将在`/tmp`目录下创建一个零长度的文件。

```shell
bash$ mktemp
/tmp/tmp.zzsvql3154
```

```shell
PREFIX=filename
tempfile=`mktemp $PREFIX.XXXXXX`
#                        ^^^^^^ 在文件名模板中，
#                               至少需要六位占位符。
#   如果没有指定文件名模板，则
#   默认为"tmp.XXXXXXXXXX"。

echo "tempfile name = $tempfile"
# tempfile name = filename.QA2ZpY
#                 或其他类似的...

#  以600文件权限在当前工作目录下创建一个以该名称命名的文件。
#  因此，"umask 177"是不必要的，
#  但它是很好的编程实践。
```

### make

用于构建和编译二进制包的实用程序。这也可以用于源文件中通过增量更改触发的任何一组操作。

*make*命令会检查`Makefile`文件，这是一个记录需要执行的文件依赖关系和操作的列表。

*make*实用程序实际上是一个在各个方面都类似于*Bash*的强大编程语言，但具有识别*依赖关系*的能力。若想更加深入地了解该有用的工具集，可参阅[GNU软件文档网站](http://www.gnu.org/manual/manual.html)。

### install

用于特殊目的的文件复制命令，类似于[cp](https://tldp.org/LDP/abs/html/basic.html#CPREF)命令，但能够设置复制文件的权限和属性。此命令似乎是为安装软件包而量身定制的，因此它经常出现在`Makefile`中（在`make install :`部分）。同样在安装脚本中，它也证明是有用的。

### dos2unix

该实用程序由Benjamin Lin和合作者共同编写，能够将DOS格式的文本文件 (由CR-LF终止的行) 转换为UNIX格式 (仅由LF终止的行)，[反之亦然](https://tldp.org/LDP/abs/html/gotchas.html#DOSNEWLINES)。

### ptx

<strong>ptx [targetfile]</strong>命令输出目标文件的置换索引 (permuted index 交叉引用列表)。如果需要，可以在管道中进一步过滤和格式化。

### more, less

用于将文本文件或流显示到`标准输出(stdout)`的寻呼机(pager)，一次一个屏幕。这些可能用于过滤来自`标准输出(stdout)`的输出 . . . 或者来自一个脚本。

*more*命令的一个有趣的应用是“尝试跑一下”一个命令序列，以防止潜在的不愉快后果。

```shell
ls /home/bozo | awk '{print "rm -rf " $1}' | more
#                                            ^^^^

# 测试以下 (灾难性) 命令行的效果:
#      ls /home/bozo | awk '{print "rm -rf " $1}' | sh
#      交给shell来执行 . . .                          ^^
```

*less*寻呼机(pager)具有一个有趣的属性，即对*man手册*的源进行格式化显示。请参阅[样例 A-39](https://tldp.org/LDP/abs/html/contributed-scripts.html#MANED)。

## 注记

[[1]](https://tldp.org/LDP/abs/html/filearchiv.html#AEN11885)这里所谓的*归档*，仅仅是指存储在单个位置的一组相关文件。

[[2]](https://tldp.org/LDP/abs/html/filearchiv.html#AEN11896)`tar czvf ArchiveName.tar.gz *`*会*包含在当前工作目录*下*子目录里的点文件(dotfile)。这是一个没有写进文档里的GNU **tar**“特性”。

[[3]](https://tldp.org/LDP/abs/html/filearchiv.html#AEN12840)校验和可以表示为*十六进制*数，也可以为其他进制数。

[[4]](https://tldp.org/LDP/abs/html/filearchiv.html#AEN12849)为了更好的安全性，请使用*sha256sum*、*sha512*和*sha1pass*命令。

[[5]](https://tldp.org/LDP/abs/html/filearchiv.html#AEN12969)这是对称分组密码，用于加密单个系统或本地网络上的文件，与*公钥*密码类相反，其中*pgp*是一个众所周知的例子。

[[6]](https://tldp.org/LDP/abs/html/filearchiv.html#AEN13030)当使用`-d`选项进行调用时会创建临时*目录*。
