---
title: printf的使用
date: 2017-06-26 22:12:20
tags: shell
categories: shell
---
## Linux Shell 中printf的使用
printf 命令的语法：
>printf  format-string  [arguments...]

**参数说明：**
-	format-string: 为格式控制字符串
-	arguments: 为参数列表。
<!-- more -->
```powershell
for((a=0; a<=10; a++))
do
    for((b=0; b<=10; b++))
    do
       printf -v file "%s %02d:%02d:00" 0618 $a $b
#       grep "$file" 1.txt
    done
done
```
结果为:
```powershell
0618 09:02:00
0618 09:03:00
0618 09:04:00
0618 09:05:00
...
```
>printf -v file 是指将输出结果使用file变量存储
>通过$file 来调用打印
>%02d 显示两位,不足两位整数用0补齐

[printf文档](http://www.linuxdaxue.com/explain-of-shell-printf-command.html)