---
title: 前端请求的timeout设置
date: 2020-09-17 21:09:01
categories: FrontEnd
tags:
top:
---

当我们发出一个网络请求，但是没有做超时设置，一个隐含的假设是我们认为这个请求一定会成功。然而，我们无法做出请求一定会成功的保证的。

+ 当你发出的同步请求从没有返回的时候，线程会一直被占用的
+ 异步请求未返回的线程也无法继续复用，因为sockets会有泄露，socket池的容量是有限的，未返回结果的线程会一直开着连接，最终可能会导致连接的短缺

因此best practice应当是对于ajax请求，做好timeout的设置，因为XMLHttpRequest的默认timeout是0，即没有超时设置。

Client端的timeout设置和server端一样重要，浏览器可以开的socket的数量也是有限的，我们应该通过设置timeout来充分利用socket pool。Fetch API是当前比较流行的XMLHttpRequest API的替代品，然而现在还没有一个直接的设置timeout的方法，最近刚刚推出了abort API，可以用来支持timeout。

用法比如： 

    const controller = new AbortController();

    const signal = controller.signal;

    const fetchPromise = fetch(url, {signal});  

    // No timeout by default!
    setTimeout(() => controller.abort(), 10000); 

而对于Jquery的ajax call，我们可以使用： 

    $.ajax({
        url: "test.html",
        error: function(){
            // will fire when timeout is reached
        },
        success: function(){
            //do something
        },
        timeout: 3000 // sets timeout to 3 seconds
    });

# Reference
https://robertovitillo.com/default-timeouts/ 
