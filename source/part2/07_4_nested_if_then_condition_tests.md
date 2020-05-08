# 7.4 嵌套 if/then 条件测试

可以嵌套 `if/then` 条件测试结构。嵌套的结果等价于使用 [`&&`](http://tldp.org/LDP/abs/html/ops.html#LOGOPS1) 复合比较运算符。

```bash
a=3

if [ "$a" -gt 0 ]
then
  if [ "$a" -lt 5 ]
  then
    echo "The value of \"a\" lies somewhere between 0 and 5."
  fi
fi

# 和下面的结果相同

if [ "$a" -gt 0 ] && [ "$a" -lt 5 ]
then
  echo "The value of \"a\" lies somewhere between 0 and 5."
fi
```

在 [样例 37-4](http://tldp.org/LDP/abs/html/bashver2.html#CARDS) 和 [样例 17-11](http://tldp.org/LDP/abs/html/system.html#BACKLIGHT) 中展示了嵌套 `if/then` 条件测试结构。


