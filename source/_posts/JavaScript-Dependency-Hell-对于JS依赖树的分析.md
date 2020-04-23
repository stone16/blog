---
title: JavaScript Dependency Hell - 对于JS依赖树的分析
date: 2020-04-23 16:32:31
categories: FrontEnd
tags:
    - JaveScript
    - JS Dependency
top:
---
做前端开发的同学肯定都对npm很熟悉，node package manager，一个非常受欢迎的包管理器。npm通过`package.json`来对项目当中的包进行管理，在这个json文件当中的包相当于就被npm注册了。

# 1. 什么是package.json?

+ 其定义了你的项目所依赖的所有包
+ 并指定你的项目所用的包的版本号
+ 让你的build是可以复制的，方便其他开发者来使用
# 2. Package.json 中的依赖种类

+ dependencies 
    + 这里定义了你的代码所需要的关键依赖
+ devDependencies 
    + 这里定义了你的开发所使用的的依赖，比如给代码样式的perttier 库
+ peerDependencies 
    + 这里是告诉其他开发者，当使用了你的这个包以后，他们需要定义在这里的包的特定版本
+ optionalDependencies 
    + 可选的依赖，不安装他们不会毁坏安装的过程
+ bundledDependencies 
    + 这里包含的是一个列表的包，他们会被打包到一起来引入到你的项目当中。这个在你的依赖包不在npm当中的情况下是很有用的。    
# 3. 使用package-lock.json的目的
package-lock的使用目的，我们在前面的博文当中有详细的描述过 -- [相关博文](https://llchen60.com/%E5%85%B3%E4%BA%8Epackage-lock-json/)。总的来说，有packge-lock.json 能够给我们更大的自由度，将commit的回退和依赖的回退分隔开，即我可以使用过去的依赖树运行当前的代码，这在没有lock json的时候是很难实现的。

# 4. 依赖树的例子与简化

    // 以gatsby为例, know the size of your node_modules overall 
    du -sh node_modules
    
    // list the size decending 
    $ du -sh ./node_modules/* | sort -nr | grep '\dM.*'
     17M    ./node_modules/rxjs
    8.4M    ./node_modules/@types
    7.4M    ./node_modules/core-js
    6.8M    ./node_modules/@babel
    5.4M    ./node_modules/gatsby
    5.2M    ./node_modules/eslint
    4.8M    ./node_modules/lodash
    3.6M    ./node_modules/graphql-compose
    3.6M    ./node_modules/@typescript-eslint
    3.5M    ./node_modules/webpack
    3.4M    ./node_modules/moment
    3.3M    ./node_modules/webpack-dev-server
    3.2M    ./node_modules/caniuse-lite
    3.1M    ./node_modules/graphql
...

    // !!! moved unused modules and dependencies
    npm dedup 

解耦操作的运行机理，就是寻找不同依赖之间的公有的包，然后复用这些共有的包。

对于可视化，用一些现成的工具可以被用来观察整个包的依赖状态，譬如：

+ https://npm.anvaka.com/#/
+ http://npm.broofa.com/

# Reference 
1. https://blog.appsignal.com/2020/04/09/ride-down-the-javascript-dependency-hell.html 