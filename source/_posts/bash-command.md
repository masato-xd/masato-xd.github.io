---
title: Bash Shell快捷键
date: 2017-06-27 22:19:43
tags: shell
categories: linux
---

简单记录一下shell下常用的快捷键
<!-- more -->
### 移动光标
* `crtl+b`：前移一个字符
* `ctrl+f`：后移一个字符
* `alt+b`：迁移一个单词
* `alt+f`：后移一个单词
* `ctrl+a`：移到行首
* `crtl+e`：移到行尾
* `ctrl+x`：行首到当前光标替换
* <font color=red>`ESC+f`：移动到下一个单词
* <font color=red>`ESC+b`：移动到上一个单词

### 编辑命令
* `alt+d`：删除当前光标到临近右边单词开始(delete)
* `ctrl+w`： 删除当前光标到临近左边单词结束(word)
* `ctrl+h`： 删除光标前一个字符（相当于backspace）
* `ctrl+d`： 删除光标后一个字符（相当于delete）
* <font color=red>`ctrl+u`： 删除光标左边所有
* <font color=red>`ctrl+k`： 删除光标右边所有
* `ctrl+l`： 清屏
* `ctrl+shift+c`：复制（相当于鼠标左键拖拽）
* `ctrl+shift+v`： 粘贴（相当于鼠标中键）

### 其他命令
* `ctrl+r`：查找历史命令，输入关键字。


### Vim
* `dw`：从当前光标开始删除到下一个单词头
* `de`： 从当前光标开始删除到单词尾
* `dW`：从当前光标开始删除到行尾