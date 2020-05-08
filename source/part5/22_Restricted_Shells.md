# 第二十二章. 限制模式的Shell
#### 限制模式下被禁用的命令
* 在限制模式下运行一个脚本或部分脚本将禁用一些命令，尽管这些命令在正常模式下是可用的。这是个安全措施，可以限制脚本用户的权限，减少运行脚本可能带来的损害。

被禁用的命令和功能：
* 使用 _cd_ 来改变工作目录。
* 修改 _$PATH_, _$SHELL_, *$BASH_ENV* 或 _$ENV_ 等[环境变量](http://tldp.org/LDP/abs/html/othertypesv.html#ENVREF)
* 读取或修改 _$SHELLOPTS_，shell环境选项。
* 输出重定向。
* 调用包含 / 的命令。
* 调用 [exec](http://tldp.org/LDP/abs/html/internal.html#EXECREF) 来替代shell进程。
* 其他各种会造成混乱或颠覆脚本用途的命令。
* 在脚本中跳出限制模式。

#### 例 22-1. 在限制模式运行脚本
```
#!/bin/bash

#  在脚本开头用"#!/bin/bash -r"
#+ 可以让整个脚本在限制模式运行。

echo

echo "改变目录。"
cd /usr/local
echo "现在是在 `pwd`"
echo "回到家目录。"
cd
echo "现在是在 `pwd`"
echo

# 到此为止一切都是正常的，非限制模式。

set -r
# set --restricted 效果相同。
echo "==> 现在是限制模式 <=="

echo
echo

echo "在限制模式试图改变目录。"
cd ..
echo "依旧在 `pwd`"

echo
echo

echo "\$SHELL = $SHELL"
echo "试图在限制模式改变Shell 。"
SHELL="/bin/ash"
echo
echo "\$SHELL= $SHELL"

echo
echo

echo "试图在限制模式重定向输出内容。"
ls -l /usr/bin > bin.files
ls -l bin.files    # 尝试列出试图创建的文件。

echo

exit 0

```
