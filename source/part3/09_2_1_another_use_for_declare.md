# 9.2.1 `declare` 的另类用法

`declare` 命令可以帮助用户识别变量、[环境变量]() 或是其他信息，与 [数组]() 搭配效果更佳。

```bash
bash$ declare | grep HOME
HOME=/home/bozo


bash$ zzy=68
bash$ declare | grep zzy
zzy=68


bash$ Colors=([0]="purple" [1]="reddish-orange" [2]="light green")
bash$ echo ${Colors[@]}
purple reddish-orange light green
bash$ declare | grep Colors
Colors=([0]="purple" [1]="reddish-orange" [2]="light green")
```