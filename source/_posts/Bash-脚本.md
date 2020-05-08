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

# Reference 
1. https://wangdoc.com/bash/grammar.html 