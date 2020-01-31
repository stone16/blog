---
title: Linux 常用指令(3) - Inspired by THE ART OF COMMAND LINE
date: 2020-01-30 20:45:49
categories: Linux
tags:
    - Linux
    - Cli
top:
---

# 1. 基础

+  `apropos`查找文档
+  `type [cmd]` 看这个命令是可执行文件、shell还是别名
+  任务管理工具们
    + &
    + ctrl + z
    + ctrl + c
    + jobs
    + fg
    + bg
    + kill
+ 文件管理工具
    + ls    ls -l
    + less 
    + head
    + tail   tail -f
    + chown 
    + chmod 
    + du -hs*
+ 文件系统管理
    + df
    + mount 
    + fdisk
    + mkfs
    + lsblk
+ 网络知识
    + ip
    + ipconfig
    + dig
+ grep
    + -i
    + -o
    + -v
    + -A
    + -B
    + -C

# 2. 日常使用

+ 补全
    + tab 自动补全
    + ctrl-r 搜索命令行历史记录 
        + enter 执行
        + 鼠标右键 edit
+ 删除 & 移动
    + ctrl-w 删除键入的最后一个单词
    + ctrl-u 删除行内光标所在位置之前的内容
    + alt-b alt-f以单词为单位移动光标
    + ctrl-a 到行的开始
    + ctrl-e 到行的结尾
+ xargs 
    + 接收pipeline的输入，然后用这个输入继续做操作
+ pgrep, pkill 
    + 通过名字或者属性来找到进程
+ netstat -ltnp 
    + 看现在处在listening 状态下的进程们 


# Reference

1. https://www.quora.com/Linux/What-are-some-time-saving-tips-that-every-Linux-user-should-know#
2. https://coolshell.cn/articles/7829.html
3. https://github.com/jlevy/the-art-of-command-line/blob/master/README-zh.md