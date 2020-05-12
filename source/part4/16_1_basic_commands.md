# 16.1 基本命令

新手的第一条命令。

### ls

列出文件列表。这条命令看似简单，实则强大。例如，使用递归选项 `-R` 可以列出树状的目录结构。其他常用的选项有 `-S` 按文件大小排序、`-t` 按文件修改时间排序、`-v` 按文件名中的（数字化）版本号排序[1][1]、`-b` 显示转义字符、`-i` 显示 inode 信息（见[示例 16-4][2]）。

```bash
bash$ ls -l
 -rw-rw-r-- 1 bozo bozo 0 Sep 14 18:44 chapter10.txt
 -rw-rw-r-- 1 bozo bozo 0 Sep 14 18:44 chapter11.txt
 -rw-rw-r-- 1 bozo bozo 0 Sep 14 18:44 chapter12.txt
 -rw-rw-r-- 1 bozo bozo 0 Sep 14 18:44 chapter1.txt
 -rw-rw-r-- 1 bozo bozo 0 Sep 14 18:44 chapter2.txt
 -rw-rw-r-- 1 bozo bozo 0 Sep 14 18:44 chapter3.txt
 -rw-rw-r-- 1 bozo bozo 0 Sep 14 18:49 Chapter_headings.txt
 -rw-rw-r-- 1 bozo bozo 0 Sep 14 18:49 Preface.txt


bash$ ls -lv
 total 0
 -rw-rw-r-- 1 bozo bozo 0 Sep 14 18:49 Chapter_headings.txt
 -rw-rw-r-- 1 bozo bozo 0 Sep 14 18:49 Preface.txt
 -rw-rw-r-- 1 bozo bozo 0 Sep 14 18:44 chapter1.txt
 -rw-rw-r-- 1 bozo bozo 0 Sep 14 18:44 chapter2.txt
 -rw-rw-r-- 1 bozo bozo 0 Sep 14 18:44 chapter3.txt
 -rw-rw-r-- 1 bozo bozo 0 Sep 14 18:44 chapter10.txt
 -rw-rw-r-- 1 bozo bozo 0 Sep 14 18:44 chapter11.txt
 -rw-rw-r-- 1 bozo bozo 0 Sep 14 18:44 chapter12.txt
```

> ![note](http://tldp.org/LDP/abs/images/note.gif)
> 当试图列出一个不存在的文件时，*ls* 命令会返回非零的[退出状态][3]。

```bash
bash$ ls abc
ls: abc: No such file or directory


bash$ echo $?
2
```

**样例 16-1. 使用 *ls* 来创建用于烧录 CDR 光盘的内容表**

```bash
#!/bin/bash
# ex40.sh (burn-cd.sh)
# 自动烧录 CDR 的脚本


SPEED=10         # 如果你的硬件支持，这里的速度可以改得更高
IMAGEFILE=cdimage.iso
CONTENTSFILE=contents
# DEVICE=/dev/cdrom     用于老版本的 cdrecord
DEVICE="1,0,0"
DEFAULTDIR=/opt  # 待烧录的数据源目录
                 # 务必确保它存在。练习：为此添加一个测试

# 使用了 Joerg Schilling 的 "cdrecord" 包:
# http://www.fokus.fhg.de/usr/schilling/cdrecord.html

# 如果想以普通用户身份调用本脚本，则需要为 cdrecord 提权，使其以 root 身份运行：
#+ chmod u+s /usr/bin/cdrecord
# 显然，这产生了一个安全漏洞，尽管不是什么大问题。

if [ -z "$1" ]
then
  IMAGE_DIRECTORY=$DEFAULTDIR
  # 默认目录。
else
    IMAGE_DIRECTORY=$1
fi

# 创建一个“内容表”文件。
ls -lRF $IMAGE_DIRECTORY > $IMAGE_DIRECTORY/$CONTENTSFILE
# "l" 选项表示输出（相对）更丰富的文件信息。
# "R" 选项表示进行递归列出。
# "F" 选项表示标记文件格式（如：目录文件名尾添加一个 / 符号）。
echo "Creating table of contents."

# 创建一个可烧录的映像文件。
mkisofs -r -o $IMAGEFILE $IMAGE_DIRECTORY
echo "Creating ISO9660 file system image ($IMAGEFILE)."

# 烧录。
echo "Burning the disk."
echo "Please be patient, this will take a while."
wodim -v -isosize dev=$DEVICE $IMAGEFILE
# 现代 Linux 发行版中实用程序 "wodim" 会假定 "cdrecord" 有效而不做判断。
exitcode=$?
echo "Exit code = $exitcode"

exit $exitcode
```

### cat, tac

cat 是 concatenate "拼接" 一词的缩写，会把文件内容输出到 `stdout`。通常用语配合重定向功能（`>` `>>`）来拼接多个文件。

```bash
cat filename                          # 输出文件内容到 stdout

cat file.1 file.2 file.3 > file.123   # 拼接三个文件
```

选项 `-n` 会使 cat 为每个文件的每行前插入连续号码。选项 `-b` 会使 `-n` 过滤掉空行。选项 `-v` 会用 `^` 记号显示不可打印字符。选项 `-s` 会合并连续的空行为一行。

参见[样例 16-28][4]和[样例 16-24][5]。

> ![note](http://tldp.org/LDP/abs/images/note.gif) 在[管道][6]中，[重定向][7] `stdin` 到文件会比用 **cat** 命令更高效。

```bash
cat filename | tr a-z A-Z

tr a-z A-Z < filename   # 效果相同，但是少启动了一个程序，也省略了管道的开销。
```

**tac** 是反过来的 *cat*，从后向前地输出文件内容。

### rev

给定一个文件，逆序输出它的每一行到 `stdout`。这与 **tac** 不同，因为 **rev** 保留了每行内部的顺序。

```bash
bash$ cat file1.txt
This is line 1.
 This is line 2.


bash$ tac file1.txt
This is line 2.
 This is line 1.


bash$ rev file1.txt
.1 enil si sihT
 .2 enil si sihT
```

### cp

文件复制文件。`cp file1 file2` 将 `file1` 复制为 `file2`，如果 `file2` 已存在则覆盖它（见[样例 16-6][8]）。

> ![note](http://tldp.org/LDP/abs/images/note.gif)
> 特别有用的选项有：`-a` 归档（复制完整的目录树）、`-u` 更新（不再覆盖同名的新文件）、`-r` `-R` 递归。

```bash
cp -u source_dir/* dest_dir
# 根据 source_dir "同步" dest_dir，这会从 source_dir 中复制 dest_dir 中不存在的文件，以及已存在的但是“比 source_dir 中的文件更老的”文件。
```

### mv

文件移动命令。等价于 **cp** 和 **rm** 的组合。常用于移动文件和重命名文件。[样例 10-11][9]和[样例 A-2][10] 有一些在脚本中使用 **mv** 的例子。

> ![note](http://tldp.org/LDP/abs/images/note.gif)
> 在非交互脚本中，**mv** 可以使用“强制”选项 `-f` 以省略用户的确认。
> 当移动一个目录到另一个已存在的目录中，那么它会成为目标目录的子目录。

```bash
bash$ mv source_directory target_directory

bash$ ls -lF target_directory
total 1
 drwxrwxr-x    2 bozo  bozo      1024 May 28 19:20 source_directory/
```

### rm

文件删除命令。选项 `-f` 可以强制删除只读文件，也常用于在脚本中省略用户的确认。

> ![note](http://tldp.org/LDP/abs/images/note.gif)
> *rm* 在处理 `-` 打头的文件时会失败，因为它将其视为选项。一个变通办法是在输入文件名参数前提供选项结束标志 `--`，另一个办法是提供相对路径 `./` 而非直接输入文件名。

```bash
bash$ rm -badname
rm: invalid option -- b
 Try `rm --help' for more information.

bash$ rm -- -badname

bash$ rm ./-badname
```

### rmdir

目录删除命令。目录需要是空的，注意也不能有“隐藏”文件（[点文件][11]）。

### mkdir

目录创建命令。如 `mkdir -p project/programs/December`，这里的选项 `-p` 会自动按需创建不存在的上级目录。

### chmod

改变文件或目录的属性（见[样例 15-14][12]）。

```bash
chmod +x filename
# 使 "filename" 对任意用户是可执行的。

chmod u+s filename
# 为 "filename" 设置 "suid" 标志位。
# 普通用户执行 "filename" 时，将拥有等同于该文件所有者的特权（privilege）。这对 shell 脚本无效。
```

```bash
chmod 644 filename
# 使 "filename" 对文件所有者可读可写，对其他用户只读。
# 注意，这里的输入是八进制的。

chmod 444 filename
# 使 "filename" 对所有者、所属组、其他用户都只读。
# 文件所有者必须“强制写入”才能修改，其他用户不允许修改。root 不受此限制。
# 删除文件遵循同样的限制。
```

```bash
chmod 1777 directory-name
# 任意用户可读可写可执行，并设置了 "sticky bit" 标志位。
# 该标志位表示只有目录、目录中文件的所有者和 root 能够删除其中的文件。

chmod 111 directory-name
# 任意用户不可读不可写可执行。
# 目录的可执行表示你能访问目录中的文件，不可读表示你不能列出目录中的文件或用 "find" 命令来搜索其中的文件。
# 补充：此时你可以通过 /path/file 的路径直接访问目录中的文件，只要你有该文件的相应权限，尽管无法用 ls /path/ 等方法知道该目录中有它。
# root 不受此限制。

chmod 000 directory-name
# 任意用户不可读不可写不可执行。
# 但是，如果目录是空的，那么可以移动（mv）或删除（rmdir）。
# 你甚至可以链接目录中的文件，虽然这个链接不可读不可写不可执行。
# root 不受此限制。
```

### chattr

修改文件属性。这听起来与 **chmod** 类似，但针对的是不同的选项和语法，且只在 ext2/ext3 文件系统上有效。

一个具体的有趣选项是 `i`，`chattr +i filename` 可以将文件标记为不可变的，即文件不可写、不可被链接、不可删除。甚至是 root 也不允许。这个属性只能被 root 设置或取消。类似地，`a` 选项表示文件是“只可追加的”。

```bash
root# chattr +i file1.txt

root# rm file1.txt
rm: remove write-protected regular file `file1.txt'? y
rm: cannot remove `file1.txt': Operation not permitted
```

如果文件属性有 `s`（secure, 安全）选项，那么文件删除时会擦写清零对应的块，以避免数据恢复。

如果文件属性有 `u`（undelete, 可恢复）选项，那么文件删除后，其文件内容仍然可访问。

如果文件属性有 `c`（compress, 压缩）选项，那么写入硬盘、从硬盘中读出时会自动压缩、解压缩。

> ![note](http://tldp.org/LDP/abs/images/note.gif)
> **chattr** 设置的文件属性不会在 **ls -l** 中显示。

### ln

创建一个已存在的文件的链接。“链接”是文件的引用，或称为别名。**ln** 命令可以多次链接一个文件（见[样例 4-6][14]）。

**ln** 创建的文件只占用少量字节，是一个引用，一个指向源文件的指针。

**ln** 通常搭配选项 `-s` 使用，s 可以解释为符号的（symbolic）、软的（soft）。软链接的优势是可以创建一个目录链接或是跨文件系统的链接。

> ![warn](http://tldp.org/LDP/abs/images/caution.gif)
> 如果文件不存在，会报错。

> **该采用何种链接？**
> 就像 John Macdonald 说的：
> 两种（链接类型）都提供了一定程度的双向引用——编辑原文件、硬链接文件、软链接文件三者任意一个，其修改都会同时生效。当你在高层工作时会有些区别：硬链接的新名称完全独立于旧名称，如果删除了旧的，新的仍然指向数据，不受影响；软链接则不然，软链接在其指向的旧名称被删除后会失效，尽管软链接文件仍然存在。软链接的优点是可以对另一个文件系统做引用（因为它只是引用文件名，而非其数据），而且软链接还能引用目录，硬链接不可以。

链接使得脚本（以及其他可执行文件）可以通过多种名称被调用，且其行为是一致的。

**样例 16-2. 你好、再见**

```bash
#!/bin/bash
# hello.sh: 根据调用情况，说 "hello" 或是 "goodbye"。

# 在当前目录 ($PWD) 下建立一个指向本脚本的链接：
#    ln -s hello.sh goodbye
# 现在，试着分别用两种方式调用本脚本：
# ./hello.sh
# ./goodbye


HELLO_CALL=65
GOODBYE_CALL=66

if [ $0 = "./goodbye" ]
then
  echo "Good-bye!"
  # 这里可以放一些告别相关的其他命令
  exit $GOODBYE_CALL
fi

echo "Hello!"
# 这里可以放一些见面相关的其他命令
exit $HELLO_CALL
```

### man, info

访问关于系统命令与安装的实用程序的手册和信息页。如果可用，一般 `info` 比 `man` 包含更详细的描述。

有多种“自动”编写手册页（*man pages*）的尝试。[样例 A-39][15] 在这方面做出了尝试。

### 注释

[1][1] 选项 `-v` 还可以根据文件名的大小写前缀来排序。

[2][11] 点文件是以 `.` 打头的文件，如 `~/.Xdefaults`。这样的文件名不显示在默认的 `ls` 结果中（可以使用 `ls -a` 使其显示），也不会被 `rm -rf *` 所删除。点文件通常作为初始化或配置文件出现在用户家目录中。

[3][13] 这个特性可能尚未被你用的 ext2/ext3 文件系统版本所实现。请检查你所用的 Linux 发行版的文档。

[1]: http://tldp.org/LDP/abs/html/basic.html#FTN.AEN10025
[2]: http://tldp.org/LDP/abs/html/moreadv.html#IDELETE
[3]: http://tldp.org/LDP/abs/html/exit-status.html#EXITSTATUSREF
[4]: http://tldp.org/LDP/abs/html/textproc.html#LNUM
[5]: http://tldp.org/LDP/abs/html/textproc.html#ROT13
[6]: http://tldp.org/LDP/abs/html/special-chars.html#PIPEREF
[7]: http://tldp.org/LDP/abs/html/io-redirection.html#IOREDIRREF
[8]: http://tldp.org/LDP/abs/html/moreadv.html#EX42
[9]: http://tldp.org/LDP/abs/html/parameter-substitution.html#RFE
[10]: http://tldp.org/LDP/abs/html/contributed-scripts.html#RN
[11]: http://tldp.org/LDP/abs/html/basic.html#FTN.AEN10228
[12]: http://tldp.org/LDP/abs/html/internal.html#EX44
[13]: http://tldp.org/LDP/abs/html/basic.html#FTN.AEN10301
[14]: http://tldp.org/LDP/abs/html/othertypesv.html#EX18
[15]: http://tldp.org/LDP/abs/html/contributed-scripts.html#MANED
