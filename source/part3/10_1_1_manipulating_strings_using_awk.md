# 10.1.1 使用 `awk` 处理字符串

在 Bash 脚本中可以调用字符串处理工具 `awk` 来替换内置的字符串处理操作。

样例 10-6. 使用另一种方式来截取和定位子字符串

```bash
#!/bin/bash
# substring-extraction.sh

String=23skidoo1
#      012345678    Bash
#      123456789    awk
# 注意不同字符串索引系统：
# Bash 中第一个字符的位置为0。
# Awk 中第一个字符的位置为1。

echo ${String:2:4} # 从第3位开始（0-1-2），4个字符的长度
                                         # skid

# Awk 中与 ${string:pos:length} 等价的是 substr(string,pos,length)。
echo | awk '
{ print substr("'"${String}"'",3,4)      # skid
}
'
#  将空的 "echo" 通过管道传递给 awk 作为一个模拟输入，
#+ 这样就不需要提供一个文件名来操作 awk 了。

echo "----"

# 同样的：

echo | awk '
{ print index("'"${String}"'", "skid")      # 3
}                                           # （skid 从第3位开始）
'   # 这里使用 awk 等价于 "expr index"。

exit 0
```