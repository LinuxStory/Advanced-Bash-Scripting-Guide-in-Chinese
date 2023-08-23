# 17.1 分析一个系统脚本

让我们利用对系统管理命令的了解，一起分析一个系统脚本吧。最简单易懂的脚本之一是“killall”，[[1]](https://tldp.org/LDP/abs/html/sysscripts.html#FTN.AEN17079)用于在系统关闭时暂停正在运行的进程。

**样例 17-12. *killall*脚本，来自`/etc/rc.d/init.d`**

```shell
#!/bin/sh

# --> 本文档作者添加的注释为“#-->”。

# --> 这是'rc'脚本软件包的一部分。
# --> 该脚本由Miquel van Smoorenburg所撰，<miquels@drinkel.nl.mugnet.org>.

# --> 该脚本似乎仅用于特定的Red Hat / FC系统
# --> (可能不会出现在其他发行版中)。

#  关闭所有非必需但仍在运行的服务。
#  (其实并不应该有，所以这只是一个健全性检查)

for i in /var/lock/subsys/*; do
        # --> 标准的for/in循环，但“do”写在同一行，
        # --> 有必要加";"。
        # 检查是否脚本还在那里。
        [ ! -f $i ] && continue
        # --> 这是一个“与列表”的巧妙运用，等价于：
        # --> if [ ! -f "$i" ]; then continue

        # 获取子系统名称。
        subsys=${i#/var/lock/subsys/}
        # --> 匹配变量名，在本例中是文件名。
        # --> 完全等价于 subsys=`basename $i`。

        # -->  从锁文件中获取
        # -->  (如果存在锁文件，
        # -->   那就证明了进程还在运行)。
        # -->  请参见上面的“锁文件”条目。


        # 关闭子系统。
        if [ -f /etc/rc.d/init.d/$subsys.init ]; then
           /etc/rc.d/init.d/$subsys.init stop
        else
           /etc/rc.d/init.d/$subsys stop
        # -->  挂起正在运行的工作和守护进程。
        # -->  注意"stop"是一个位置参数，
        # -->  并不是一个shell内置程序。
        fi
done
```

仔细看看还是能读懂的。除了一点变量匹配的小技巧，没有新的东西。

<strong>练习 1.</strong>请分析位于`/etc/rc.d/init.d`目录下的**halt**脚本。它比**killall**稍长，但概念相似。请复制将这个脚本拷贝到主目录中的某个地方进行试验（*不要*以*root*用户身份运行它）。可以使用`-vn`标志（**sh -vn scriptname**）模拟运行脚本。在此过程中，你可以添加更多的注释。把命令改成[echo](https://tldp.org/LDP/abs/html/internal.html#ECHOREF)。

<strong>练习 2.</strong>请观察`/etc/rc.d/init.d`下一些更复杂的脚本，试着至少理解它们的一部分。按照上面的过程来分析它们。为了获得更多的信息，你还可以检查`/usr/share/doc/initscripts-?.??`中的`sysvinitfile`文件，它们是“初始脚本”文档中的一部分。

## 注记

[[1]](https://tldp.org/LDP/abs/html/sysscripts.html#AEN17079)请不要将*killall*系统脚本与位于`/usr/bin`中的[killall](https://tldp.org/LDP/abs/html/x9644.html#KILLALLREF)命令相混淆。
