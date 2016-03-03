# 23. 进程替换
用[管道](http://tldp.org/LDP/abs/html/special-chars.html#PIPEREF) 将一个命令的 _标准输出_ 输送到另一个命令的 _标准输入_ 是个强大的技术。但是如果你需要用管道输送多个命令的 _标准输出_ 怎么办？这时候 _进程替换_ 就派上用场了。

 _进程替换_ 把一个（或多个）[进程](http://tldp.org/LDP/abs/html/special-chars.html#PROCESSREF) 的输出送到另一个进程的
