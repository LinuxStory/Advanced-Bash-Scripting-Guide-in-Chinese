# 8.2. 数字常量

通常情况下，shell脚本会把数字以十进制整数看待(base 10)，除非数字加了特殊的前缀或标记。
带前缀0的数字是八进制数(base 8)；带前缀0x的数字是十六进制数(base 16)。
内嵌 # 的数字会以 BASE#NUMBER 的方式进行求值（不能超出当前shell支持整数的范围）。

**样例 8-4. 数字常量的表示**

```
#!/bin/bash
# numbers.sh: 不同进制数的表示

# 十进制数: 默认
let "dec = 32"
echo "decimal number = $dec"             # 32
# 一切正常。


# 八进制数: 带前导'0'的数
let "oct = 032"
echo "octal number = $oct"               # 26
# 结果以 十进制 打印输出了。
# ------ ------ -----------


# 十六进制数: 带前导'0x'或'0X'的数
let "hex = 0x32"
echo "hexadecimal number = $hex"         # 50

echo $((0x9abc))                         # 39612
#     ^^      ^^   双圆括号进行表达式求值
# 结果以十进制打印输出。



# 其他进制数: BASE#NUMBER
# BASE 范围:  2 - 64
# NUMBER 必须以 BASE 规定的正确形式书写，如下:

let "bin = 2#111100111001101"
echo "binary number = $bin"              # 31181

let "b32 = 32#77"
echo "base-32 number = $b32"             # 231

let "b64 = 64#@_"
echo "base-64 number = $b64"             # 4031

# 这种表示法只对进制范围(2 - 64)内的 ASCII 字符有效。
# 10 数字 + 26 小写字母 + 26 大写字母 + @ + _


echo

echo $((36#zz)) $((2#10101010)) $((16#AF16)) $((53#1aA))
                                         # 1295 170 44822 3375

#  重要提醒:
#  ---------
#  使用超出进制范围以外的符号会报错。

let "bad_oct = 081"

# (可能的) 报错信息:
#  bad_oct = 081: value too great for base (error token is "081")
#              Octal numbers use only digits in the range 0 - 7.

exit $?        # 退出码 = 1 (错误)

# 感谢 Rich Bartell 和 Stephane Chazelas 的说明。

```

