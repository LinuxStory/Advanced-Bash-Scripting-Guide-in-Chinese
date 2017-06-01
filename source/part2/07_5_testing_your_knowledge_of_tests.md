# 7.5 牛刀小试

系统文件 `xinitrc` 可以用来启动软件 X Server。该文件包含了许多 `if/then` 测试结构。下面的代码摘录自较早版本的 `xinitrc`（大约在 Red Hat 7.1 版本）。

```bash
if [ -f $HOME/.Xclients ]; then
  exec $HOME/.Xclients
elif [ -f /etc/X11/xinit/Xclients ]; then
  exec /etc/X11/xinit/Xclients
else
    # 安全分支。尽管程序不会执行这个分支。
    # （我们在 Xclients 中也提供了相同的机制）增强程序可靠性。
    xclock -geometry 100x100-5+5 &
    xterm -geometry 80x50-50+150 &
    if [ -f /usr/bin/netscape -a -f /usr/share/doc/HTML/index.html ]; then
            netscape /usr/share/doc/HTML/index.html
    fi
fi
```

试着解释代码片段中的条件测试结构, 然后试着在 /etc/X11/xinit/xinitrc 查看最新版本，并且分析其中的 if/then 条件测试结构。为了更好的进行分析，你可能需要继续阅读后面章节中对 [`grep`](http://tldp.org/LDP/abs/html/textproc.html#GREPREF)，[`sed`](http://tldp.org/LDP/abs/html/sedawk.html#SEDREF) 和 [正则表达式](http://tldp.org/LDP/abs/html/regexp.html#REGEXREF) 的讨论。

