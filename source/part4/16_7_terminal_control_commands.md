# 16.7 终端控制命令

## 影响控制台或终端的命令

### tput

初始化终端并/或从terminfo数据中获取信息。可以通过各种选项进行各种终端操作：**tput clear**等效于[clear](https://tldp.org/LDP/abs/html/terminalccmds.html#CLEARREF)命令；**tput reset**等效于[reset](https://tldp.org/LDP/abs/html/terminalccmds.html#RESETREF)命令。

```shell
bash$ tput longname
xterm terminal emulator (X Window System)
```

执行**tput cup X Y**会将光标移动到当前终端的(X, Y)坐标。**clear**命令通常在此之前执行清空终端屏幕。

一些有趣的*tput*选项：

- `bold`，粗体

- `suml`，给终端中的文本添加下划线

- `smso`，反向呈现文本

- `sgr0`，重置终端参数 (恢复正常)，不清除屏幕

使用*tput*的样例脚本：

1. [样例 36-15](https://tldp.org/LDP/abs/html/colorizing.html#COLORECHO)

2. [样例 36-13](https://tldp.org/LDP/abs/html/colorizing.html#EX30A)

3. [样例 A-44](https://tldp.org/LDP/abs/html/contributed-scripts.html#HOMEWORK)

4. [样例 A-42](https://tldp.org/LDP/abs/html/contributed-scripts.html#NIM)

5. [样例 27-2](https://tldp.org/LDP/abs/html/arrays.html#POEM)

请注意，[stty](https://tldp.org/LDP/abs/html/system.html#STTYREF)提供了更强大的命令集来控制终端。

### infocmp

该命令用于打印出有关当前终端的大量信息。它引用了*terminfo*数据库。

```shell
{% raw %}bash$ infocmp
#       Reconstructed via infocmp from file:
 /usr/share/terminfo/r/rxvt
 rxvt|rxvt terminal emulator (X Window System), 
         am, bce, eo, km, mir, msgr, xenl, xon, 
         colors#8, cols#80, it#8, lines#24, pairs#64, 
         acsc=``aaffggjjkkllmmnnooppqqrrssttuuvvwwxxyyzz{{||}}~~, 
         bel=^G, blink=\E[5m, bold=\E[1m,
         civis=\E[?25l, 
         clear=\E[H\E[2J, cnorm=\E[?25h, cr=^M, 
         ...

{% endraw %}
```

### reset

重置终端参数和清空屏幕。与**clear**命令一样，光标和提示将再次出现在终端的左上角。

### clear

**clear**命令只会简单地清空控制台或*xterm*屏幕。提示和光标将重新出现在屏幕或*xterm*窗口的左上角。可以在命令行或脚本中使用此命令。请参阅[样例 11-26](https://tldp.org/LDP/abs/html/testbranch.html#EX30)。

### resize

查看及设置`$TERM`和`$TERMCAP`变量的必要命令，用于进行复制当前终端的大小 (尺寸)。

```shell
bash$ resize
set noglob;
 setenv COLUMNS '80';
 setenv LINES '24';
 unset noglob;
```

### script

此实用程序会记录（保存在文件中）用户在控制台或xterm窗口的命令行中所有键盘操作。这实际上创建了一个会话记录。
