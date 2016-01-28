# 20 I/O 重定向

目录
- [20.1 使用 exec](http://tldp.org/LDP/abs/html/x17974.html)
- [20.2 重定向代码块](http://tldp.org/LDP/abs/html/redircb.html)
- [20.3 应用程序](http://tldp.org/LDP/abs/html/redirapps.html)

有三个默认打开的文件[[1]](http://tldp.org/LDP/abs/html/io-redirection.html#FTN.AEN17884), `stdin`(标准输入，键盘),`stdout`(标准输出， 屏幕)和 `stderr`(标准错误，屏幕上输出的错误信息)。这些和任何其他打开的文件都可以被重定向。重定向仅仅意味着捕获输出文件，命令，脚本，甚至是一个脚本的代码块([样例 3-1](http://tldp.org/LDP/abs/html/special-chars.html#EX8))和([样例 3-2](http://tldp.org/LDP/abs/html/special-chars.html#EX8)) 作为另一个文件，命令，程序或脚本的输入。

每个打开的文件都有特定的文件描述符。[[2]](http://tldp.org/LDP/abs/html/io-redirection.html#FTN.AEN17894),而 `stdin`，`stdout`，`stderr` 的文件描述符分别为 0,1,2。当然了，还有附件的文件描述符 3 - 9。有时候为`stdin`，`stdout`，`stderr`临时性的复制链接分配这些附加的文件描述符会非常有用.[[3]](http://tldp.org/LDP/abs/html/io-redirection.html#FTN.AEN17906)。这简化了复杂重定向和重组后的恢复(见[样例 20-1](http://tldp.org/LDP/abs/html/x17974.html#REDIR1))
```
   COMMAND_OUTPUT >
      # 重定向标准输出到一个文件.
      # 如果文件不存在则创建，否则覆盖.

      ls -lR > dir-tree.list
      # 创建了一个包含目录树列表的文件.

   : > filename
      # ">" 清空了文件.
      # 如果文件不存在，则创建了个空文件 (效果类似 'touch').
      # ":" 是个虚拟占位符, 不会有输出.

   > filename    
      # ">" 清空了文件.
      # 如果文件不存在，则创建了个空文件 (效果类似 'touch').
      # (结果和上述的 ": >" 一样， 但在某些 shell 环境中不能正常运行.)

   COMMAND_OUTPUT >>
      # 重定向标准输出到一个文件.
      # 如果文件不存在则创建，否则新内容在文件末尾追加.


      # 单行重定向命令 (只作用于本身所在的那行):
      # --------------------------------------------------------------------

   1>filename
      # 以覆盖的方式将 标准错误 重定向到文件 "filename."
   1>>filename
      # 以追加的方式将 标准输出 重定向到文件 "filename."
   2>filename
      # 以覆盖的方式将 标准错误 重定向到文件 "filename."
   2>>filename
      # 以追加的方式将 标准错误 重定向到文件 "filename."
   &>filename
      # 以覆盖的方式将 标准错误 和 标准输出 同时重定向到文件 "filename."
      # 在 bash 4 中才有这个新功能.

   M>N
     # "M" 是个文件描述符, 如果不明确指定，默认为 1.
     # "N" 是个文件名.
     # 文件描述符 "M" 重定向到文件 "N."
   M>&N
     # "M" 是个文件描述符, 如果不设置默认为 1.
     # "N" 是另一个文件描述符.

      #==============================================================================

      # 重定向 标准输出，一次一行.
      LOGFILE=script.log

      echo "This statement is sent to the log file, \"$LOGFILE\"." 1>$LOGFILE
      echo "This statement is appended to \"$LOGFILE\"." 1>>$LOGFILE
      echo "This statement is also appended to \"$LOGFILE\"." 1>>$LOGFILE
      echo "This statement is echoed to stdout, and will not appear in \"$LOGFILE\"."
      # 这些重定向命令在每行结束后自动"重置".



      # 重定向 标准错误，一次一行.
      ERRORFILE=script.errors

      bad_command1 2>$ERRORFILE       #  Error message sent to $ERRORFILE.
      bad_command2 2>>$ERRORFILE      #  Error message appended to $ERRORFILE.
      bad_command3                    #  Error message echoed to stderr,
                                      #+ and does not appear in $ERRORFILE.
      # 这些重定向命令每行结束后会自动“重置”.
	#=======================================================================
```

```
   2>&1
      # 重定向 标准错误 到 标准输出.
      # 错误信息发送到标准输出相同的位置.
        >>filename 2>&1
            bad_command >>filename 2>&1
            # 同时将 标准输出 和 标准错误 追加到文件 "filename" 中 ...
        2>&1 | [command(s)]
            bad_command 2>&1 | awk '{print $5}'   # found
            # 通过管道传递 标准错误.
            # bash 4 中可以将 "2>&1 |" 缩写为 "|&".

   i>&j
      # 重定向文件描述符 i 到 j.
      # 文件描述符 i 指向的文件输出将会重定向到文件描述符 j 指向的文件

   >&j
      # 默认的标准输出 (stdout) 重定向到 j.
      # 所有的标准输出将会重定向到 j 指向的文件.
```

```
   0< FILENAME
    < FILENAME
      # 从文件接收输入.
      # 类似功能命令是 ">", 经常会组合使用.
      #
      # grep search-word <filename


   [j]<>filename
      #  打开并读写文件 "filename" ,
      #+ 并且分配文件描述符 "j".
      #  如果 "filename" 不存在则创建.
      #  如果文件描述符 "j" 未指定, 默认分配文件描述符 0, 标准输入.
      #
      #  这是一个写指定文件位置的应用程序. 
      echo 1234567890 > File    # 写字符串到 "File".
      exec 3<> File             # 打开并分配文件描述符 3 给 "File" .
      read -n 4 <&3             # 读取 4 字符.
      echo -n . >&3             # 写一个小数点.
      exec 3>&-                 # 关闭文件描述符 3.
      cat File                  # ==> 1234.67890
      #  随机访问.



   |
      # 管道.
      # 一般是命令和进程的链接工具.
      # 类似 ">", 但更一般.
      # 在连接命令，脚本，文件和程序方面非常有用.
      cat *.txt | sort | uniq > result-file
      # 所有 .txt 文件输出进行排序并且删除复制行,
      # 最终保存结果到 "result-file".
```

可以用单个命令行表示输入和输出的多个重定向或管道.
```
command < input-file > output-file
# 或者等价:
< input-file command > output-file   # 尽管这不标准.

command1 | command2 | command3 > output-file
```

更多详情见[样例 16-31](http://tldp.org/LDP/abs/html/filearchiv.html#DERPM) and [样例 A-14](http://tldp.org/LDP/abs/html/contributed-scripts.html#FIFO).

多个输出流可以重定向到一个文件.
```
ls -yz >> command.log 2>&1
#  捕获不合法选项 "yz" 的结果到文件 "command.log."
#  因为 标准错误输出 被重定向到了文件,
#+ 任何错误信息都会在这.

#  注意, 然而, 接下来的这个案例并 "不能" 同样的结果.
ls -yz 2>&1 >> command.log
#  输出一条错误信息，但是不会写入到文件.
#  恰恰的, 命令输出(这个例子里为空)写入到文件, 但错误信息只会在 标准输出 输出.

#  如果同时重定向 标准输出 和 标准错误输出,
#+ 命令的顺序不同会导致不同.
```

关闭文件描述符
```
n<&-
	关闭输入文件描述符 n.

0<&-, <&-
	关闭标准输入.

n>&-
	关闭输出文件描述符 n.

1>&-, >&-
	关闭标准输出.
```

子进程能继承文件描述符.这就是管道符能工作的原因.通过关闭文件描述符来防止继承 .
```
# 只重定向到 标准错误 到管道.

exec 3>&1                              # 保存当前 标准输出 "值".

ls -l 2>&1 >&3 3>&- | grep bad 3>&-    # 关闭 'grep' 文件描述符 3 (但不是 'ls').
#              ^^^^   ^^^^
exec 3>&-                              # 现在关闭它.

# 感谢, S.C.
```
更多关于 I/O 重定向详情见 [Appendix F](http://tldp.org/LDP/abs/html/ioredirintro.html).

#### 注意

[[1]](http://tldp.org/LDP/abs/html/io-redirection.html#AEN17884)	 在 UNIX 和 Linux 中, 数据流和周边外设([device files](http://tldp.org/LDP/abs/html/devref1.html#DEVFILEREF)) 都被看做文件.

[[2]](http://tldp.org/LDP/abs/html/io-redirection.html#AEN17894)	 `文件描述符` 仅仅是操作系统分配的一个可追踪的打开的文件号. 可以认为是一个简化的文件指针. 类似于 C 语言的 `文件句柄`.

[[3]](http://tldp.org/LDP/abs/html/io-redirection.html#AEN17906)	当 bash 创建一个子进程的时候使用 `文件描述符 5` 会有问题. 例如 [exec](http://tldp.org/LDP/abs/html/internal.html#EXECREF), 子进程继承了文件描述符 5 (详情见 Chet Ramey's 归档的 e-mail, [SUBJECT: RE: File descriptor 5 is held open](https://groups.google.com/forum/#!topic/gnu.bash.bug/E5Vdqv3tO1w)). 最好将这个文件描述符单独规避.
