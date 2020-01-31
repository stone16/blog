---
title: Linux常用指令(2)
date: 2020-01-30 20:44:54
categories: Linux
tags:
    - Linux
    - Cli
top:
---

继续上篇来介绍一些基本的指令

# 1. tr指令

+ tr是用来对从stdin进入的字符进行相关的翻译和转换的，并且将其呈现在stdout当中
+ tr会接收两个字符set，然后用第二个set里面的字符来替换第一个set里面的字符
+ tr预定义了一些sets供用户来直接使用
    + alnum: 字母数字组成的字符集
    + alpha：按字母顺序的字符集
    + blank: 空白字符集
    + cntrl: 控制字符集
    + digit: 数字字符集
    + graph: 图像字符集
    + lower: 小写字母字符集
    + print: 可以写出的字符集
    + punct: 标点符号字符集
    + space: 空格字符集
    + upper: 大写字母字符集
    + xdigit: 十六进制字符集
+ tr -d [blabla] 删除[]里面定义的字符

# 2. colrm 指令

+ 用来删除行的指令

    
    // 删除从第四行开始到末尾的所有
    $ cat grocery.list | colrm 4 

# 3. expand 和 unexpand 指令

+ expand指令能够将tabs变成空格
+ unexpand 指令能够将空格变成tab

# 4. comm, cmp以及diff指令

+ diff指令比较两个文件，并且告知二者之间的不同
    + -w 忽视空格上的不同
    + -i 忽视大小写的不同
+ comm 也是比较两个文件，但是行为不太一样。会生成三行输出
    + 只在第一个文件当中出现的行
    + 只在第二个文件当中出现的行
    + 在两个文件当中都出现的行
+ cmp 比较两个文件
    + 显示出两个文件不同的byte和行号 

# 5. bc指令

+ 使用bc来做基础运算 - basic calculator 


    $ echo 2+3 | bc 

# 6. sed 指令

https://coolshell.cn/articles/9104.html

+ sed指令可以用来对文件或者数据流进行转化
    + 一次读取一行的数据
    + 对这一行数据实行特定的操作
    + 一般来说输出到stdout
+ 功能
    + 从buffer里面删除文字
    + 在buffer附上文本或者插入文本
    + 写入文件
    + 用regex定义的规则来转化文本

+ -e: 指定表达式或者编辑的脚本   用来做替换
    + s 表示这是个替换命令
    + 用 / 作为分隔符 
    + 前面是待替换的文本，后面是替换的文本
    + g 便是让这个替换在现在的整个buffer里都生效

    
    $ echo "IBM 174.99" |sed –e 's/IBM/International Business Machines/g' 
    International Business Machines 174.99
    
    $ echo "Oracle DB"|sed -e 's/Oracle/IBM/g' -e 's/DB/DB2/g'
    IBM DB2
    
    $ echo "C:\Program Files\PuTTY\putty.exe"| sed -e 's/\\/\//g' -e 's/ /_/g' -e 's/://g'
    C/Program_Files/PuTTY/putty.exe


+ 用来做漏斗
    + d 表示要删除的东西
    + 然后中间跟着的是正则表达式 表示的一个string pattern




    cat << EOF > dummy_sed.txt
    # top of file
      # the next line here
    # Last Name, Phone
    Smith, 555-1212
    Jones, 555-5555 # last number
    EOF
     
    $ sed '/^[[:space:]]*#/d' dummy_sed.txt
    Smith, 555-1212
    Jones, 555-5555 # last number
     
    $ grep -v  ^[[:space:]]*# dummy_sed.txt
    Smith, 555-1212
    Jones, 555-5555 # last number

# 7. awk指令

## 7.1 指令范式

    // $1 表示第1列，$4表示第4列  注意$0表示整行
    $ awk '{print $1, $4}'  netstat.txt 

https://coolshell.cn/tag/awk 

+ 用来做转换，漏斗，格式化等
+ 从stdin拿数据，然后展示在stdout当中

## 7.2 格式化输出
格式化输出的样式基本上和c语言的一样

    $ awk '{printf "%-8s %-8s %-8s %-18s %-22s %-15s\n",$1,$2,$3,$4,$5,$6}' netstat.txt

## 7.3 过滤记录的功能

    $ awk '$3==0 && $6=="LISTEN" ' netstat.txt
    
    // 内建变量NR来展示表头
    $ awk '$3==0 && $6=="LISTEN" || NR==1 ' netstat.txt
    
    $ awk '$3==0 && $6=="LISTEN" || NR==1 {printf "%-20s %-20s %s\n",$4,$5,$6}' netstat.txt
    

## 7.4 awk的内建变量

+ $0 当前记录 
+ $1 - $n 当前记录的第n个字段
+ FS  输入字段分隔符，默认是空格或者Tab
+ NF  当前记录当中的字段个数
+ NR  已经读出的记录数，行号
+ FNR 当前记录数，与NR不同的是，这个值会是各个文件自己的行号
+ RS  输入的记录分隔符，默认为换行符
+ OFS 输出字段分隔符，默认空格
+ ORS 输出记录分隔符，默认换行符
+ FILENAME 当前输入文件的名字
