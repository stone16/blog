---
title: Linux 常用指令(1)
date: 2020-01-30 20:43:36
categories: Linux
tags:
    - Linux
    - Cli
top:
---

Unix的一大哲学就是创建只做一件事情的程序块，并且将这件事情做好。在这种哲学下，仔细思考需要的接口，并且用pipeline联结起来并最终产生了有用的结果就变成一件很重要的事情了。想象一下数据通过管道在不同的指令之间流动的过程，很难不用优美来形容，哈哈。本文总结一些基本的文本处理的命令，注意这些命令一般来说也是可以串联起来实现一些相对复杂且完成度比较高的功能的。

下面将逐个介绍各个指令：

# 1. Cat

+ 用来创建，连接，展示文件的


    // 创建新文件，在本行命令以后就可以输入想存储的内容了
    $ cat > grocery.list

    // 将新输入的内容附在原文件尾
    $ cat >> grocery.list 
    
    // 查看文件当中的内容
    $ cat grocery.list
    
    // 查看文件当中的内容，带行数的
    $ cat -n grocery.list 
    
# 2. nl

nl可以从stdin或者文件里读取行。输出会写到stdout或者重指向到一个文件，或者通过管道将内容传输到其他指令处。

+ nl
    + -b 指定要计数的行
        + a : 所有行都算
        + t : 不计空行，或者只有空格的行
        + n : 全都不计
        + p : 根据某种特征
    + -s 指定行号和具体内容的分隔符

    // nl 类似于 cat -n
    $ nl grocery.list 
    
    // 根据 p定义的特征：这里是只记录起始字母为a或b的
    $ nl -b p^[ba] grocery.list
    
    // 让行号和具体内容之间的分隔符变成等号
    $ nl -s= grocery.list 
    
# 3. wc 

wc是wordcount的简称，顾名思义，用来统计行数，词语数量，或者是字母的数量

+ wc
    + -l 行数
    + -w 词语数
    + -c 字母数

# 4. grep

会搜索特定的文件，或者从stdin里，去寻找符合定义的某种特征的表达

+ grep
    + -c 统计出现了的行数
    + -h 搜索多个文件的时候不再显示出文件名字
    + -i 忽略大小写的不同
    + -l 只打印满足指定特征的文件名
    + -n 打印所在的行位置
    + -v 输出所有不满足特征的行
    + -w 

# 5. streams, pipes, redirects, tee 

在Unix当中，一个terminal常规是包含三个流的，一个为了输入，两个为了输出。输入流，指的是stdin,一般来说是指向keyboard的；标准输出流一般指的是stdout， 会将结果输出到terminal。每个流都有自己的文件描述符，每一个都可以做管道化，分开来做重定向到其他命令当中去。 

+ stdin 0  < 
+ stdout 1 > 
+ stderr 2 

"|" 指一个管道，会将前一个指令的输出作为下一个指令的输入

    $ cat grocery.list | nl 

+ << 
    + 可以进行多行的输入

# 6. Using head and tail 

这两个指令是用来看一个文件的头部或者尾部的部分

+ -n
    + 用上述指令加想要显示的行数
+ -c
    + 显示的字符的数量 
    
    //显示grocery文件的前10行
    $ head -n10 grocery.list

    //显示grocery文件最后12的个字符
    $ tail -c12 grocery.list