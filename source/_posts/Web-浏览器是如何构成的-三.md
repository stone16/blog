---
title: Web 浏览器是如何构成的(三)
date: 2020-02-01 14:13:39
categories: Web
tags:
    - Web
    - Browser
top:
---

在前面的两篇文章当中，我们覆盖了木偶进程的架构以及点击输入框以后发生的事情。在本文中，我们会深入去看看渲染进程当中发生了一些什么。

渲染进程影响网络表现的很多方面。本文还是起到给一个大致脉络的作用，如果想更深入了解，可以看 [Performance Section](https://developers.google.com/web/fundamentals/performance/why-performance-matters/).

# 1. 渲染进程处理web内容

渲染进程负责发生在tab当中的所有事情，主线程会处理你发送给用户的大部分代码。一些时候一部分你的js代码会被worker线程来处理(如果你使用web worker或者service worker的话)。Compositor 和 raster线程也是在渲染进程当中运行，使得整个渲染过程更加高效顺滑。

渲染进程的核心工作就是将HTML，CSS以及JavaScript转换成用户可以进行交互的页面。

# 2. Parsing

## 2.1 DOM的构建

当渲染进程收到一个commit信息，开始接受HTML的数据的时候，主线程就开始parseHTML，并且将其转化成Document Object Model了(DOM). 

DOM是浏览器内部的对一个页面，数据结构以及JS构成的API的表示。parse的规则则是由[HTML Standard](https://html.spec.whatwg.org/)来进行定义的。Parser有很强的鲁棒性，因为他基本上可以处理绝大部分html本身写得时候的问题，比如忘记`</b>`。都可以很优雅的进行解决。[Introduction to error handling](https://html.spec.whatwg.org/multipage/parsing.html#an-introduction-to-error-handling-and-strange-cases-in-the-parser)

## 2.2 子资源的加载

一个网站经常会使用外部资源比如图像，CSS还有JavaScript. 这些文件需要从网络或者Cache里面加载出来。主线程可以在构建DOM树的时候依次去请求他们，但是为了加快整个过程，preload scanner 会同步运行这种加载任务。

如果遇到像`<img> <link>`这种tag，preload scanner就会看一下HTML parser转化来的tiken，然后向浏览器进程里的网络线程提交请求。  

当HTML parser发现一个`<script>`tag的时候，它会停止对于HTML文件的转译，然后必须要先去加载转译执行javascript代码。这样做的原因是JavaScript是可以去改变整个DOM树的结构的。需要先等其执行完以后再继续向下进行。

## 3. 自定义加载资源的方式

+ 如果你的JS文件不包含document.write()这类指令，那么你就可以在script tag当中加上`async` or `defer` 属性。通过这种提示，浏览器在进行转译的时候就会异步执行js代码，不会阻止parsing的继续进行了。
+ `<link rel="preload">`会告知浏览器这个资源对于当前的页面导览是必需的，浏览器需要尽快将其下载下来。


## 4. 样式的处理

只有DOM是远远不够描述页面会是什么样子的。因为我们可以使用CSS来处理页面的各部分的样式。主线程会转译CSS文件，并且决定计算过得各个DOM节点的样式。这种判断取决于CSS选择器给出的各部分需要的样式。

而且即便你没有提供任何的CSS，每个DOM节点都会有一个计算过的本身的样式。[Chrome默认样式文件](https://cs.chromium.org/chromium/src/third_party/blink/renderer/core/html/resources/html.css)

# 5. 布局

当前的渲染进程已经知道了整个文件的结构，以及每个节点的样式了。但是很可能这还不足以去渲染一个页面。因为除了各部分的分别的大小以外，我们还需要知道他们的相对位置，是如何布局，最终构成整个页面的。

主线程会遍览DOM树和计算过的样式，然后创建布局样式树，包含了x,y坐标信息和包围的box的大小。注意布局树和DOM树非常相似，但是对于`display:none`具有这种属性的节点，就完全不会在布局树里出现了。然而，`visibility:hidden`依然会出现在布局树当中。

# 6. 上色

主线程会遍历样式树，去创建上色记录。这个记录里面包含了整个上色所需的过程，比如先背景，再文字，blabla 

# 7. 更新渲染的管道消耗很大

在前面所叙述的过程当中，你会发现上一步的结果会成为下一步的输入。换句话说，假设你现在在html里面加了一个节点，那么整个过程基本上就要完全重来一遍了。强烈建议访问[link](https://developers.google.com/web/updates/2018/09/inside-browser-part3#updating_rendering_pipeline_is_costly)，里面有整个过程的动图，很有助于理解。

# 8. 拼合 (Compositing)

现在浏览器获得了整个文件的结构，每个成分的样式，整个页面的布局，上色的顺序。接下来就是拼接额过程了，这个将信息转化成像素的过程叫做rasterizing 

最最开始的时候采取的拼接方案就是随着下滑一点点来渲染的，但是现在方案变得越来越复杂了。

首先会对整个页面做分层处理，然后分别对其进行渲染，像素化。[Layer 详解](https://blog.logrocket.com/eliminate-content-repaints-with-the-new-layers-panel-in-chrome-e2c306d4d752/?gi=cd6271834cea)

为了知道每个成分都在哪一层，主线程会遍历布局树，去创建一个层级树。

一旦层级树被创建出来，主线程会将这些信息交给拼接线程。拼接线程接下来就会对每一层做像素化的处理。每一层可能都和整个页面一样大，相互之间是有重叠的，接下来拼接线程就会将他们分开，然后交给不同的像素化线程。像素化线程是在GPU下执行的，拼接线程会对像素胡线程进行优化，让相近的部分先一起像素化完。

一旦像素化完，拼接线程就会聚集这一部分信息，去创建拼接片。拼接片中包含了内存地址，还有在整个页面当中应处的位置。

拼接片然后会通过IPC被提交到浏览器进程当中。用拼合线程的好处是主线程不用参与，这样的话在主线程继续计算样式，执行javascript的时候拼接线程已经可以开始工作了。



# Reference 
1. https://developers.google.com/web/updates/2018/09/inside-browser-part1
2. https://developers.google.com/web/updates/2018/09/inside-browser-part2
3. https://developers.google.com/web/updates/2018/09/inside-browser-part3
4. https://developers.google.com/web/updates/2018/09/inside-browser-part4 
