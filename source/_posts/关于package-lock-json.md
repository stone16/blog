---
title: 关于package-lock.json
date: 2020-04-17 18:10:23
categories:  FrontEnd
tags:
    - npm
top:
---
当我们将node package manager (npm) 升级到5.0以上的版本的时候，你会发现npm运行的时候会自动创建一个新文件 -- package-lock.json。

里面包含的是我们的依赖关系，各种依赖的包和版本号。package-lock.json会在npm修改了node_modules 树或者修改了package.json之后自动生成。它精确的描述了整个生成的树，使得接下来任何一次的装配都可以生成完全一致的依赖树.这个生成的文件是需要commit 到remote branch上的，其目的在于：

+ 描述单个依赖树，使得其他人在做deploy的时候使用的是完全一致的依赖
+ 使得使用人员有能力直接跳转回原先的依赖状态，而不需要将代码也回退到之前的版本
+ 加强了依赖改变的阅读性，我们可以相对直观的看到每次的commit都有什么依赖被改变了
+ 也可以通过是的npm跳过对于原先安装过的包的重复的元数据分析来优化整个安装的进程

针对其特征，我们是应该将package-lock.json也提交上去的，这会给开发，同步带来不小的帮助。

# Reference 
1. https://medium.com/coinmonks/everything-you-wanted-to-know-about-package-lock-json-b81911aa8ab8