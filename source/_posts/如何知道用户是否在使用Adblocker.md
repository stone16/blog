---
title: 如何知道用户是否在使用Adblocker
date: 2020-04-07 11:58:04
categories: Web
tags:
    - Tricks
top:
---
广告监测和屏蔽是很好的功能，而Ad-blocker基本是市面上做屏蔽广告插件的翘楚了。根据GlobalWebIndex在2019年五月做的统计，现在有47% 的用户正在使用广告屏蔽的软件。

作为网站管理者，你可以做一个pop up 的信息框，希望用户能够停止对于广告的屏蔽，或者你可以直接不允许用户来访问页面，知道去除了广告屏蔽之后。

那么如何知道用户是否在使用AbBlocker呢，这里需要了解下adblocker的运行机制，广告屏蔽的运行依赖于过滤规则(filter rules)， adblock会将你正在访问的URL和过滤列表相比较，如果匹配，那么这个请求就会被屏蔽掉。

但是还有部分的广告是不会发起一个HTTP请求，相对应的，他们会使用`data:image/png`这种方式来加载广告，对于这种广告，AdBlock 在每个页面都会插入一个样式表，然后这个样式表包括选择器，来display:none 通过这种方式来隐藏页面上的广告

而这些屏蔽列表来自于一些过去的积累，譬如：
+ [Easylist](https://easylist.to/)
+ [accesptable list](https://help.getadblock.com/support/solutions/articles/6000092027-why-am-i-suddenly-seeing-taboola-outbrain-and-google-ads-)


除此以外，adblocker还会检测前端调用的js文件，如果还有ad关键词，也会直接屏蔽，所以我们可以直接使用命名，来做一个简单的判断。

    // 创建名为ads.js的文件
    isAdBlockActive = false;

    // 创建script.js
    var AdBlocker = (function () {
    
        function showModal() {
            $('#modal_ad_blocker').modal(); // show a message to the user when ads are blocked
        }
        
        setInterval(function () {
            // Get the first AdSense ad unit on the page
            var ad = document.querySelector("ins.adsbygoogle");
        
            // If the ads.js or the Google ads are not loaded, show modal and track the event
            if (typeof isAdBlockActive === 'undefined'
                || (ad && ad.innerHTML.replace(/\s/g, "").length === 0)) {
        
                showModal();
        
                if (typeof ga !== 'undefined') {
                    // Log an event in Universal Analytics
                    // but without affecting overall bounce rate
                    ga('send', 'event', 'Adblock', 'Yes', {'nonInteraction': 1});
        
                } else if (typeof _gaq !== 'undefined') {
                    // Log a non-interactive event in old Google Analytics
                    _gaq.push(['_trackEvent', 'Adblock', 'Yes', undefined, undefined, true]);
                }
            }
        }, 5000); // check every 5 seconds
    
    })();

通过检测我们在ads.js当中定义的isAdBlockActive 是否能够被检测到(undefined or have some value)。我们就可以看出是否有adBlocker正在运行，然后根据检测结果我们可以给用户提醒，或者直接阻止访问。

# Reference 
1. https://blog.rampatra.com/how-to-know-whether-a-user-is-using-an-adblocker 
2. https://help.getadblock.com/support/solutions/articles/6000087914-how-does-adblock-work- 