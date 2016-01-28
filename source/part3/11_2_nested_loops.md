# 11.2 嵌套循环

嵌套循环，顾名思义就是在循环里面还有循环。外层循环会不断的触发内层循环直到外层循环结束。当然，你仍然可以使用 `break` 可以终止外层或内层的循环。

样例 11-20. 嵌套循环

```bash
#!/bin/bash
# nested-loop.sh: 嵌套 "for" 循环。

outer=1             # 设置外层循环计数器。

# 外层循环。
for a in 1 2 3 4 5 
do
  echo "Pass $outer in outer loop."
  echo "---------------------"
  inner=1           # 重设内层循环计数器。
  
  # =====================================
  # 内层循环。
  for b in 1 2 3 4 5
  do
    echo "Pass $inner in inner loop."
    let "inner+=1"  # 增加内层循环计数器。
  done
  # 内层循环结束。
  # =====================================
  
  let "outer+=1"    # 增加外层循环计数器。
  echo              # 在每次外层循环输出中加入空行。
done
# 外层循环结束。

exit 0
```

查看 [样例 27-11](http://tldp.org/LDP/abs/html/arrays.html#BUBBLE) 详细了解嵌套 [while 循环](http://tldp.org/LDP/abs/html/loops1.html#WHILELOOPREF)。查看 [样例 27-13](http://tldp.org/LDP/abs/html/arrays.html#EX68) 详细了解嵌套 [until 循环](http://tldp.org/LDP/abs/html/loops1.html#UNTILLOOPREF)。