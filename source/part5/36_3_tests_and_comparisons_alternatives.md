# 36.3 测试和比较的其他方法

对于判断（test命令）来说，[[ ]]比[ ]更加合适。同样的，算数运算符（译注：-eq之类的）比(( ))更有优势。

```
a=8

# 下面所有这些比较的结果都应该是相等的
test "$a" -lt 16 && echo "yes, $a < 16"         # "and list"
/bin/test "$a" -lt 16 && echo "yes, $a < 16" 
[ "$a" -lt 16 ] && echo "yes, $a < 16" 
[[ $a -lt 16 ]] && echo "yes, $a < 16"          # 为表达式添加
(( a < 16 )) && echo "yes, $a < 16"             # [[ ]]和(( ))并不是必须的

city="New York"
# 下面这些结果也是相等的
test "$city" \< Paris && echo "Yes, Paris is greater than $city"
                                  # ASCII字符比较
/bin/test "$city" \< Paris && echo "Yes, Paris is greater than $city" 
[ "$city" \< Paris ] && echo "Yes, Paris is greater than $city" 
[[ $city < Paris ]] && echo "Yes, Paris is greater than $city"
                                  # 并不需要为$city变量加引号。

# 向S.C.致谢
```
