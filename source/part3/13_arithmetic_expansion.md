# 第十三章 算术扩展

算术扩展为脚本中的（整数）算术操作提供了强有力的工具。你可以使用反引号、双圆括号或者 `let` 将字符串转换为数学表达式。

### 差异比较

#### 使用 [反引号](http://tldp.org/LDP/abs/html/commandsub.html#BACKQUOTESREF) 的算术扩展（通常与 [`expr`](http://tldp.org/LDP/abs/html/moreadv.html#EXPRREF) 一起使用）

```bash
z=`expr $z + 3`         # 'expr' 命令执行了算术扩展。
```

#### 使用 [双圆括号](http://tldp.org/LDP/abs/html/dblparens.html) 或 [`let`](http://tldp.org/LDP/abs/html/internal.html#LETREF) 的算术扩展。

事实上，在算术扩展中，反引号已经被双圆括号 `((...))` 和 `$((...))` 以及 [`let`](http://tldp.org/LDP/abs/html/internal.html#LETREF) 所取代。

```bash
z=$(($z+3))
z=$((z+3))                     # 同样正确。
                               # 在双圆括号内，参数引用形式可用可不用。
                                            
# $((EXPRESSION)) 是算术扩展。  # 不要与命令替换混淆。



# 双圆括号不是只能用作赋值算术结果。

  n=0
  echo "n = $n"                # n = 0

  (( n += 1 ))                 # 自增。
# (( $n += 1 )) 是错误用法！
  echo "n = $n"                # n = 1


let z=z+3
let "z += 3"  # 引号允许在赋值表达式中使用空格。
              # 'let' 事实上执行的算术运算而非算术扩展。
```

以下是包含算术扩展的样例：

1. [样例 16-9](http://tldp.org/LDP/abs/html/moreadv.html#EX45)
2. [样例 11-15](http://tldp.org/LDP/abs/html/loops1.html#EX25)
3. [样例 27-1](http://tldp.org/LDP/abs/html/arrays.html#EX66)
4. [样例 27-11](http://tldp.org/LDP/abs/html/arrays.html#BUBBLE)
5. [样例 A-16](http://tldp.org/LDP/abs/html/contributed-scripts.html#TREE)