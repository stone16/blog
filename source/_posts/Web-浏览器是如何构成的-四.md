---
title: Web 浏览器是如何构成的(四)
date: 2020-02-01 14:14:28
categories: Web
tags:
    - Web
    - Browser
top:
---
上一篇文章我们一起研究了渲染过程，并且初步接触了排版器。本篇文章当中，我们会一起研究下排版器(Compositor)是如何实现当用户输入内容时顺滑的进行交互的。

# 1. 从浏览器的角度看输入事件

当你听到输入事件的时候，你可能只想到在textbox里面输入或者鼠标的点击，但是从浏览器的角度来说，输入代表的是用户任何行为。滚动鼠标滑轮是个输入事件，碰触或者将鼠标悬在某个事件上面也是一个输入事件。

当用户的某种行为发生的时候，浏览器进程是第一个收到这行为的。但是浏览器进程只是获取这个动作，而真正的处理还是交由渲染进程来做的。因此浏览器进程会将事件类型以及其坐标发送给渲染进程。渲染进程会通过找到事件目标，运行其相应的监听者来对事件进行处理。

# 2. 理解non-fast scrollable 区域

运行JavaScript是主线程的任务，当对一个页面进行拼装的时候，拼装线程标注页面的有事件监听器的部分，标注为Non-fast scrollable region. 通过这种标注来确保党事件在该区域发生的时候，拼装线程可以将这个时间发送给主线程。如果输入事件来自不同的地方，那么拼装线程就会创建一个新的frame来装配这块区域，而不会等待主线程了。

# 3. Tips 关于写event handlers

一般写event handling的pattern是事件代理。当事件发生的时候，你可以在最顶层的组件上加一个event handler 然后根据具体的事件目标来分配任务。代码可能会像下面这样子: 

    document.body.addEventListener('touchstart', event => {
        if (event.target === area) {
            event.preventDefault();
        }
    });

从开发者的角度当时是个好事情，你只需要写一个event handler就可以了。但是从浏览器的角度意味着整个页面都被标注成non-fast scrollable region了，这意味着即使整个页面不在意页面某个部分的输入，拼装线程仍然需要和主线程沟通，并且每次有输入的时候就要等在那里。因此，拼装线程的轮转能力就失效了。

为了解决这个问题，你可以加上`passive:true`选项在你的事件监听器当中。这提示了浏览器你仍然想听主线程的事件，但是compositor, 即拼装线程可以自己做自己的事情，然后生成新的frame. 一篇很棒的讲`passive`的[文章](https://medium.com/@devlucky/about-passive-event-listeners-224ff620e68c). 

# 4. 检查一个事件是否是cancelable 

假定你现在在页面中有个box，你想限制其滚动方向，只能横向的滚动。

这个时候使用`passive:true`只是保证页面的滚动可以足够顺滑，但是你还需要使用`preventDefault`来限制滚动的方向。

    document.body.addEventListener('pointermove', event => {
        if (event.cancelable) {
            event.preventDefault(); // block the native scroll
            /*
            *  do what you want the application to do here
            */
        }
    }, {passive: true});

# 5. 减少派送到主线程的事件

对于输入来说，一个触屏设备可以每秒钟发布60-120次碰触事件，而鼠标的移动可以每秒触发100次。如果像触碰的移动这种事件每秒发给主线程120次，那么他会再触发大量的点击测试和js的执行，这样子会导致整个页面的更新非常慢。

Chrome合并了连续时间，比如滚动，鼠标的滑动，光标移动等，然后延迟分发知道下一次请求渲染的时候。

如果想知道轨迹的这种信息，可以使用`getCoalescedEvents`来获取


# Reference 
1. https://developers.google.com/web/updates/2018/09/inside-browser-part1
2. https://developers.google.com/web/updates/2018/09/inside-browser-part2
3. https://developers.google.com/web/updates/2018/09/inside-browser-part3
4. https://developers.google.com/web/updates/2018/09/inside-browser-part4 