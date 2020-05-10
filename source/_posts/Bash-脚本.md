---
title: Bash 脚本
date: 2020-05-07 22:24:54
categories: Linux
tags:
    - bash
top:
---
Bash是大多数Linux发行版的默认Shell（命令行环境），值得去研究一波~

最近也有很多人转而使用zsh，看到一个不错的[post](https://apple.stackexchange.com/questions/361870/what-are-the-practical-differences-between-bash-and-zsh),讲了二者的主要区别

# 1. 基本语法

+ echo 在屏幕输出一行文本，可以将该命令的参数原样输出
    + `-n` 取消末尾的回车符
    + `-e` 解释引号当中的特殊字符，进行转义 
+ 命令格式
    + `command [arg1 ... argN]`
    + `ls -l` 等于 `ls --list`
        + 其实主要是写script的时候为了让语句自己能够解释自己，会选用长形式，其余时候一般都选用短形式的语句
+ 分号
    + 命令的结束符，使得一行可以放置多个命令
    + 上个命令执行完之后，才会执行下一个命令
    + 后一个指令总会接着第一个来执行，不管第一个成功或者失败
+ 命令组合符
    + `command1 && command2` 
        + 如果command1成功，才会继续执行command2
    + `command1 || command2`
        + 如果command1成功，就不执行command2了

+ type命令  -- 用于判断命令的来源，是内置的命令或者外部程序
    + `-a` 去查看一个命令的所有定义
    + `-t` 可以返回一个命令的类型
        + alias
        + keyword
        + function
        + builtin
        + file

+ 快捷键
    + `Ctrl + L`：清除屏幕并将当前行移到页面顶部。
    + `Ctrl + C`：中止当前正在执行的命令。
    + `Shift + PageUp`：向上滚动。
    + `Shift + PageDown`：向下滚动。
    + `Ctrl + U`：从光标位置删除到行首。
    + `Ctrl + K`：从光标位置删除到行尾。
    + `Ctrl + D`：关闭 Shell 会话。

# 2. 模式扩展
Shell接到用户输入命令，通过空格进行对输入的分割，拆成词元，然后扩展词元里面的特殊字符，来调用相应的命令。

+ 波浪线扩展
    + 自动扩展为当前用户的主目录

+ `？`扩展
    + ？代表文件路径里面的任意单个字符，不包括空字符
    + `file???`就表示file后面跟着三个字符的文件名

+ `*`扩展
    + 代表文件路径里面的任意数量的字符，包括零个字符
    + 注意不会匹配隐藏文件

+ `[]`扩展
    + 匹配内部包含的任意一个
    + `[abcde]`就会匹配abcde里面的任意一个
    + `[!abc]` or `[^abc]` 表示匹配除了abc以外的其他字符

+ `[start-end]`扩展
    + 表示匹配一个连续的范围
    + `[a-c]`等同于[abc]

+ `{...}`扩展
    + 指分别扩展为大括号当中定义的所有值
    + 大括号颞部逗号前后不能有空格
    + `echo d{a,e,i,u,o}g`
        + output:  dag deg dig dug dog

+ `{start..end}`扩展
    + `echo {1..4}`
        + output: 1 2 3 4

+ 字符类
    + `[[:class:]]` 表示一个字符类，扩展成某一类特定字符之中的一个
    + `[[:alnum:]]`：匹配任意英文字母与数字
    + `[[:alpha:]]`：匹配任意英文字母
    + `[[:blank:]]`：空格和 Tab 键。
    + `[[:cntrl:]]`：ASCII 码 0-31 的不可打印字符。
    + `[[:digit:]]`：匹配任意数字 0-9。
    + `[[:graph:]]`：A-Z、a-z、0-9 和标点符号。
    + `[[:lower:]]`：匹配任意小写字母 a-z。
    + `[[:print:]]`：ASCII 码 32-127 的可打印字符。
    + `[[:punct:]]`：标点符号（除了 A-Z、a-z、0-9 的可打印字符）。
    + `[[:space:]]`：空格、Tab、LF（10）、VT（11）、FF（12）、CR（13）。
    + `[[:upper:]]`：匹配任意大写字母 A-Z。
    + `[[:xdigit:]]`：16进制字符（A-F、a-f、0-9）

+ 量词语法
    + `?(pattern-list)`：匹配零个或一个模式。
    + `*(pattern-list)`：匹配零个或多个模式。
    + `+(pattern-list)`：匹配一个或多个模式。
    + `@(pattern-list)`：只匹配一个模式。
    + `!(pattern-list)`：匹配零个或一个以上的模式，但不匹配单独一个的模式

+ shopt 命令 -- 用来调整bach的行为
    + -s 打开某个参数
    + -u 关闭某个参数
    + 直接加 optionName  可以来查询某个参数是关闭的还是打开的
    + 参数
        + dotglob  让扩展结果包括隐藏文件
        + nullglob 让通配符不匹配任何文件名，返回空字符
        + failglob 使得通配符不匹配任何文件名时，Bash 会直接报错，而不是让各个命令去处理
        + extglob 支持ksh的一些扩展语法
        + nocaseglob 让通配符扩展不区分大小写
        + globstar  是的`**`可以匹配零个或多个子目录
+ tips
    + 通配符是先解释，再执行
    + 文件名扩展不匹配的时候，会原样输出
    + 只适用于单层路径 

# Reference 
1. https://wangdoc.com/bash/grammar.html 