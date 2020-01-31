---
title: JavaScript 教程 - part 3
date: 2020-01-30 22:30:15
categories: FrontEnd
tags:
    - FrontEnd
    - JavaScript
top:
---

# 1. 事件

## 1.1 EventTarget 接口

事件本质是程序各个组成部分之间的一种通信方式。DOM的事件触发都定义在EventTarget接口当中，所有节点对象都部署了这个接口 

该接口主要提供了三个实例方法: 

+ addEventListener  绑定事件的监听函数
+ removeEventListener 移除事件的监听函数
+ dispatchEvent  触发事件


### 1.1.1 EventTarget.addEventListener() 

+ 用于在当前节点或对象上，定义一个特定的时间的监听函数，一旦这个事件发生，就会执行监听函数
+ 没有返回值
+  `target.addEventListener(type, listener, [useCapture])`
    + type 事件名称
    + listener  监听函数  事件发生，就调用这个监听函数
    + useCapture  表示监听函数是否在捕获阶段触发，默认为false (只在冒泡阶段被触发)
        + 捕获指的是事件从最外层开始发生，直到最具体的元素
        + 冒泡指从最内层开始发生
    + 第三个参数 -- 是个属性配置对象，即除了useCapture 你还可以配置很多其他属性的
        + capture  是否在捕获阶段触发监听函数
        + once  是否只触发一次，然后就自动移除
        + passive  表示监听函数不会调用事件的preventDefault方法，如果监听函数调用了，浏览器就会忽略这个要求，并在监控台上输出一行警告


    function hello() {
      console.log('Hello world');
    }
    
    var button = document.getElementById('btn');
    button.addEventListener('click', hello, false);

### 1.1.2 EventTarget.removeEventListener() 

用来移除addEventListener方法添加的事件监听函数，该方法没有返回值。

+ removeEnventListener方法移除的监听函数必须是addEventListener方法已经添加过得，而且必须在同一个元素节点上，否则无效


### 1.1.3 EventTarget.dispatchEvent() 

+ 在当前节点触发指定事件，从而触发监听函数的执行  返回一个布尔值
+ 只要有一个监听函数调用了`Event.preventDefault()`,返回值为false，否则为true


    para.addEventListener('click', hello, false);
    var event = new Event('click');
    para.dispatchEvent(event);


## 1.2 事件模型

这一部分想要解决的问题是JS作为使用事件驱动编程模式(event-driven)的编程语言，是怎么样给事件绑定监听函数的。

### 1.2.1 绑定监听函数 

#### 1.2.1.1 使用html on属性

HTML允许在元素属性当中，直接定义某些事件的监听代码

    <body onload="doSomething()">

+ 元素的监听属性，都是on加上事件名
+ 属性的值为将要执行的代码，单单函数名是不被允许的
+ 该种方式的监听代码，只会在**冒泡阶段被触发**


    // 先输出1 再输出2
    <div onClick="console.log(2)">
      <button onClick="console.log(1)">点击</button>
    </div>

#### 1.2.1.2 使用元素节点的事件属性

+ 也是只可以在冒泡阶段进行触发


    window.onload = doSomething;
    
    div.onclick = function (event) {
      console.log('触发事件');
    };

#### 1.2.1.3 使用addEventListener() 

用来为该节点定义事件的监听函数

### 1.2.2 事件的传播

事件发生后，会在子元素和父元素之间进行传播，分为以下几个阶段:

1. 从window对象传导到目标节点(上层传到底层)，成为捕获阶段(capture phase)
2. 在目标节点上触发，称为目标阶段
3. 从目标节点传导会window对象，称为冒泡阶段 - bubbling phase


这种三阶段的传播模型，使得同一个事件会在多个节点上触发。

    <div>
      <p>点击</p>
    </div>

    var phases = {
      1: 'capture',
      2: 'target',
      3: 'bubble'
    };
    
    var div = document.querySelector('div');
    var p = document.querySelector('p');
    
    div.addEventListener('click', callback, true);
    p.addEventListener('click', callback, true);
    div.addEventListener('click', callback, false);
    p.addEventListener('click', callback, false);
    
    function callback(event) {
      var tag = event.currentTarget.tagName;
      var phase = phases[event.eventPhase];
      console.log("Tag: '" + tag + "'. EventPhase: '" + phase + "'");
    }
    
    // 点击以后的结果
    // Tag: 'DIV'. EventPhase: 'capture'
    // Tag: 'P'. EventPhase: 'target'
    // Tag: 'P'. EventPhase: 'target'
    // Tag: 'DIV'. EventPhase: 'bubble'

### 1.2.3 事件的代理

因为事件会在冒泡阶段向上传播到父节点，因此可以将子节点的监听函数定义在父节点上，由父节点的监听函数统一处理多个子元素的事件，这种方式称为事件的代理


    var ul = document.querySelector('ul');
    
    ul.addEventListener('click', function (event) {
      if (event.target.tagName.toLowerCase() === 'li') {
        // some code
      }
    });

上面代码中，click事件的监听函数定义在`<ul>`节点，但是实际上，它处理的是子节点`<li>`的click事件。这样做的好处是，只要定义一个监听函数，就能处理多个子节点的事件，而不用在每个`<li>`节点上定义监听函数。而且以后再添加子节点，监听函数依然有效。


另外，我们可以使用`stopPorpagation`方法来使事件传播到某个节点就停下来，不再传播


    // stopPropagation不会停止在同一个element上的其他事件的执行
    p.addEventListener('click', function (event) {
      event.stopPropagation();
      console.log(1);
    });
    
    p.addEventListener('click', function(event) {
      // 会触发
      console.log(2);
    });
    
+ 如果我们想不触发同一个element上的其他监听函数，可以使用`stopImmediatePropogation()`方法


## 1.3 事件对象

事件发生以后，会产生一个事件对象，作为参数传给监听函数。浏览器原生提供一个Event对象，所有事件都是这个对象的实例，即继承了Event.prototype

+ Event构造函数: `event = new Event(type, option);`
    + type 一个字符串，表示事件的名称
    + options 一个对象，表示事件对象的配置
        + bubbles  boolean  
            + default to false
            + 表示对象是否冒泡
        + cancelable boolean 
            + default to false
            + 表示事情能否被取消，即能否用event.preventDefault()取消这个时间


### 1.3.1 实例属性

+ Event.bubbles 
    + boolean
    + 表示当前事件是否会冒泡
    + 只读属性，用于了解Event实例是否可以冒泡
+ Event.eventPhase 
    +  返回当前事件所处的阶段，该属性只读
        + 0 还没有发生
        + 1 捕获阶段  处于从祖先节点向目标节点的传播过程当中
        + 2 事件到达目标节点
        + 3 事件处于冒泡阶段
+ Event.currentTarget
    + 正在通过的节点
+ Event.target 
    + 事件的原始触发节点
+ Event.timeStamp
+ Event.isTrusted
    + 表示该事件是否由一个真实的用户行为产生
+ Event.detail
    + 只有浏览器的UI事件才具有
    + 返回一个数值表示事件的某种信息
    + 具体含义与事件类型相关
        + 点击 次数
        + 鼠标滚轮 距离

### 1.3.2 实例方法

+ event.preventDefault()
    + 取消浏览器对当前事件的默认行为
    + 生效的前提是事件对象的cancelable属性为true


    
    // HTML 代码为
    // <input type="text" id="my-input" />
    // 实现只接受小写的功能
    var input = document.getElementById('my-input');
    input.addEventListener('keypress', checkName, false);
    
    function checkName(e) {
      if (e.charCode < 97 || e.charCode > 122) {
        e.preventDefault();
      }
    }

+ event.stopPropagation()
    + 阻止事件在DOM中继续传播，防止再触发定义在别的节点上的监听函数，但是不包括在当前节点上其他的事件监听函数
+ event.stopImmediatePropogation()
    + 阻止同一个事件的其他监听函数被调用
    + 更彻底的阻止事件的传播
+ event.composedPath() 
    + 返回一个数组
    + 成员是事件的最底层节点和依次冒泡经过的所有上层节点


    // HTML 代码如下
    // <div>
    //   <p>Hello</p>
    // </div>
    var div = document.querySelector('div');
    var p = document.querySelector('p');
    
    div.addEventListener('click', function (e) {
      console.log(e.composedPath());
    }, false);
    // [p, div, body, html, document, Window]

## 1.4 各类事件的描述
### 1.4.1 [鼠标事件](https://wangdoc.com/javascript/events/mouse.html)
+ 继承MouseEvent接口
    + click
        + mouseDown + mouseUp 
    + dblclick 在用一个元素上双击鼠标时触发
        + mouseDown + mouseUp + click  
    + mousedown 按下鼠标键时触发
    + mouseup 释放按下的鼠标键的时候触发
    + mousemove 
        + 当鼠标在一个结点内部移动的时候触发
        + 当鼠标持续移动时，就会持续触发
        + 应当对该事件的监听函数做一些限定，比如一定时间内只能运行一次
    + mouseenter 
        + 进入一个节点时触发
    + mouseover
        + 鼠标进入一个节点时触发，进入子节点会再一次触发这个事件
    + mouseout
        + 鼠标离开一个节点的时候触发，离开父节点也会触发
    + mouseleave
        + 鼠标离开一个节点时触发，离开父节点不会触发这个事件 
    + contextmenu
        + 按下鼠标右键时触发的
    + wheel
        + 滚动鼠标的滚轮时触发的，该事件继承的是WheelEvent接口 
+ mouseEvent
    + 浏览器提供一个MouseEvent构造函数，用于新建一个MouseEvent实例
    + `var event = new MouseEvent(type, options);`
    + 第一个参数为字符串，表示事件名称
    + 第二个参数是一个事件配置对象，可以配一些不同的属性
        + screenX
        + screenY
        + clientX
            + 相对于程序窗口的水平位置
        + clientY
            + 相对于程序窗口的垂直位置
        + ctrlKey
        + shiftKey
        + altKey
        + metaKey 
        + button 
            + 0 按下主键
            + 1 按下辅助键
            + 2 按下次要键
        + buttons

### 1.4.2 键盘事件

+ 键盘事件的种类
    + keydown
        + 按下键盘的时候触发
        
    + keypress
        + 按下有值的键再触发，即按下Ctrl, Alt, Shift, Meta这样的无值的键不会触发 
        + 对于有值的键，还是会先触发keydown事件，再触发这个事件
    + keyup
        + 松开键盘的时候触发该事件
        + 如果一直不松开按键，那么就会连续触发键盘事件
            + 触发顺序：
                + keydown
                + keypress
                + keydown
                + keypress
                + ....
                + 放开 -> keyup
+ KeyboardEvent 接口概述
    + 描述用户与键盘的互动
    + 继承了Event接口，并且定义了自己的实例属性和实例方法
    + 浏览器原生提供KeyboardEvent构造函数，用来新建键盘事件的实例
    +  `new KeyboardEvent(type, options)`
    +  第一个参数为字符串，表示事件类型
    +  第二个参数为一个事件配置对象，参数可选
        + key
        + code
            + 0-9   digital0-digital9
            + A-Z   KeyA - KeyZ
            + F1 - F12
            + 方向键
                + ArrowDown
                + ArrowUp
                + ArrowLeft
                + ArrowRight
            + Alt
                + AltLeft/ AltRight
                + similar to shift, ctrl
        + location
            + 返回键盘的区域 
        + ctrlKey
        + shiftKey
        + altKey
        + metaKey
        + repeat 
            + 看该案件是否被按着不放，以便判断是否重复这个键
+ KeyboardEvent实例方法
    + getModifierState() 
        + 返回一个布尔值，表示是否按下或激活指定的功能键
            + alt
            + capslock
            + control
            + meta
            + numlock
            + shift

### 1.4.3 进度事件

用来描述资源加载的进度，主要由:
+ AJAX请求
+ `<img>`
+ `<audio>`
+ `<video>`
+ `<style>`
+ `<link>`

该类外部资源的加载触发，继承了ProgressEvent接口。主要包含以下几种事件：

+ abort  外部资源中止加载时被触发
+ error  由于错误导致外部资源无法加载时触发
+ load   外部资源加载成功时触发
+ loadstart  外部资源开始加载时触发
+ loadend  外部资源停止加载时触发
+ progress  外部资源加载过程中不断触发
+ timeout  加载超时时触发



+ ProgressEvent接口
    +  用来描述外部资源加载的进度
    +  `new ProgressEvent(type, options)`
        + types 事件类型
        + options - 配置对象
            + lengthComputable 布尔值，表示加载的总量是否可以计算，默认为false
            + loaded 表示已经加载的量
            + total 表示需要加载的量
### 1.4.4 表单事件

+ 表单事件的种类
    + input
        + 当`<input>, <select>, <textarea>`的值发生变化时触发
        + input事件会连续触发，每按下一个键，都会触发一次input事件的
    + select
        + 是在`<input> <select> <textarea>`里面的值发生变化的时候触发 
    + change
        + 在元素失去焦点时发生
        + 即当有连续变化的时候，input事件会被触发很多次，而change事件只在失去焦点的时候被触发一次。
        + 换个角度看，input事件是一定伴随着change事件的，具体分为以下几种情况
            + 激活单选框或复选框时触发
            + 用户提交时触发
            + 当文本框或textarea元素的值发生改变，并且失去焦点的时候触发
    + invalid
        + 用户提交表单，当表单元素的值不满足校验条件，就会触发invalid事件 
    + reset
        + 发生在表单对象上
        + 表示表单重置时锁触发的事件
    + submit
        + 当表单数据向服务器提交时触发
+ inputEvent接口
    + `new InputEvent(type, options)`
    + type 字符串，表示事件名称
    + options 配置对象
        + inputType
        + data  表示插入的字符串
        + dataTransfer 

### 1.4.5 触摸事件

+ 触摸操作
    + touch  一个触摸点
        + 位置
        + 大小
        + 形状
        + 压力
        + 目标元素
    + touchList  多个触摸点的集合
        + 成员为Touch的实例对象
        + 属性
            + length  表示触摸点的数量
            + item() 返回指定的成员
    + touchEvent  触摸引发的事件实例
+ Touch 接口概述
    + `var touch = new Touch(touchOptions)` 
    + touchOptions
        + identifier 
            + 触摸点的唯一ID
        + target
        + clientX
        + clientY
        + screenX
        + screenY
        + pageX
        + pageY
        + radiusX 
        + radiusY
        + rotationAngle
        + force 
            + 0 -1 范围
            + 表示触摸压力
+ 触摸事件的种类
    + touchstart
    + touchend
    + touchmove
    + touchcancel 
### 1.4.6 拖拉事件

+ 拖拉定义
    + 用户在某个对象上按下鼠标键不放，拖动它到另一个位置，然后释放鼠标键，将该对象放在那里
+ 拖拉的对象
    + 元素节点
    + 图片
    + 链接
    + 选中的文字

### 1.4.7 其他常见事件

+ 资源事件
    + beforeunload 
        + 在窗口，文档，各种资源将要卸载前触发
        + 用于防止用户不小心卸载资源
    + unload
        + 在窗口关闭或者document对象将要卸载时触发
        + 触发顺序排在beforeunload, pagehide事件后面
    + load
        + 在页面或者某个资源加载成功时触发
        + 页面或者资源从浏览器缓存加载，并不会触发load事件
+ session历史事件
    + pageshow
        + 页面加载时触发，如果要指定页面每次加载时都运行的打字吗，可以放在这个事件的监听函数当中 
    + pagehide
    + popstate
        + 在浏览器的history对象的当前记录发生显式切换时触发
    + hashchange
        + URL的hash部分发生变化的时候触发 
+ 网页状态事件 
    + DOMContentLoaded 事件
        + 网页下载并解析完成以后在document对象上触发该事件  
+ 窗口事件
    + scroll 
        + 用户拖动滚动条
    + resize
    + fullscreenchange
    + fullscreenerror 
+ 焦点事件
    + focus 获得焦点以后触发
    + blur  失去焦点以后触发
    + focusin  将要获得焦点时触发
    + focusout 将要失去焦点时触发


# 2. 浏览器模型

主要来介绍浏览器提供的各种JS接口

## 2.1 浏览器环境概述

+ 代码嵌入网页方法
    + <script> 直接嵌入代码
        + type属性  用来指定脚本类型
            + text/javascript
            + application/javascript
    + <script> 标签加载外部脚本
        + 为了防止攻击者篡改外部脚本，script标签允许设置一个`integrity`属性，写入外部脚本的Hash签名，用来验证脚本的一致性 
    + 事件属性
        + 网页元素的事件  比如onclick  onmouseover 可以写入JS代码 
    + URL协议
        + 在URL的位置写入代码，使用这个URL的时候就会执行JS代码了 


    <script src="/assets/application.js"
      integrity="sha256-TvVUHzSfftWg1rcfL6TIJ0XKEGrgLyEq6lEpcmrG9qs=">
    </script>


    <a href="javascript:console.log('Hello')">点击</a>

### 2.1.1 script工作原理
+ 网页加载流程
    + 浏览器一边下载HTML网页，一边开始解析 
    + 当发现`<script>`元素的时候，就暂停解析，将网页渲染的控制权交给JavaScript引擎
    + 如果`<script>`元素引用了外部脚本，就下载该脚本再执行，否则就直接执行代码
    + JS引擎执行完毕，控制权交还给渲染引擎，恢复往下解析HTML网页
+ 为什么加载脚本的时候会停止页面渲染？
    + 因为JS代码可以修改DOM，所以需要将DOM控制权给它，否则会出现线程竞争的问题
    + 问题点在于如果外部脚本加载时间非常长，那么浏览器会一直等待脚本下载完成，造成网页长时间失去响应，浏览器就会呈现出假死状态，称为阻塞效应。
    + 因为相对较好的做法是将Script标签放在页面底部，这样即使遇到脚本失去响应，网页主体的渲染已经完成，用户就可以看到内容的。
+ 脚本是有执行顺序的
    + 执行顺序由在页面中出现的顺序决定，为了保证脚本之间的依赖关系不受到破坏
    + 加载脚本都会产生阻塞效应，需要等他们都加载完成浏览器才能继续页面渲染
+ 此外，对于来自**同一个域名的资源**，比如脚本文件、样式表文件、图片文件等，浏览器一般有限制，同时最多下载6～20个资源，即最多同时打开的 TCP 连接有限制，这是为了防止对服务器造成太大压力。如果是来自不同域名的资源，就没有这个限制。所以，通常把静态文件放在不同的域名之下，以加快下载速度。
### 2.1.2 defer属性
+ 为了解决脚本文件下载阻塞网页渲染的问题
+ 在`<script>`当中加入defer属性，作用是延迟脚本的执行，等到DOM加载生成后，再执行脚本 
+ 运行流程
    + 浏览器开始解析HTML网页
    + 发现带有defer属性的script标签
    + 继续往下解析HTML网页，并且并行下载script元素加载外部脚本
    + 完成解析HTML网页，回头再执行已经下载了的脚本

### 2.1.3 async属性

async属性的作用在于使用另一个进程下载脚本，下载时不会阻塞渲染

+ 浏览器开始解析HTML网页
+ 解析过程中，发现带有async属性的script标签
+ 继续向下解析，同时并行下载script标签当中的外部脚本
+ 脚本下载完成，浏览器暂停解析HTML网页，开始执行下载的脚本
+ 脚本执行完毕，浏览器恢复解析HTML网页

aync属性可以保证脚本下载的同时，浏览器继续渲染。  注意: 一旦采用，脚本执行顺序就无法确定了

### 2.1.4 脚本动态加载

script元素可以动态生成，再插入页面，从而实现脚本的动态加载。

    ['a.js', 'b.js'].forEach(function(src) {
      var script = document.createElement('script');
      script.src = src;
      document.head.appendChild(script);
    });

动态生成的script标签不会阻塞页面渲染，也就不会造成浏览器假死。但是问题在于，这种方法无法保证脚本的执行顺序，哪个脚本文件先下载完成，就先执行哪个

+ 另外我们可以指定我们需要加载的协议，可以使用HTTP或者HTTPS。默认是HTTP的，如果想使用HTTPS，需要做指定 -- `<script src="https://example.js"></script>
`
### 2.1.5 浏览器组成

+ 渲染引擎
    + 将网页代码渲染为用户视觉可以感知的平面文档
    + 渲染的步骤
        + 解析代码
            + 将HTML代码解析为DOM，CSS解析为CSSOM (CSS Object Model)
        + 对象合成
            + 将DOM和CSSOM合成一棵渲染树 (render tree)
        + 布局
            + 计算出渲染树的布局
        + 绘制
            + 将渲染树绘制到屏幕上
+ 重流 vs 重绘
    + 重流必然导致重绘
    + 重绘不一定导致重流
    + 尽量减少重流
    + 优化技巧
        + DOM尽量写在一起，不要读取一个节点就写入一个
        + 缓存DOM信息
        + 动画使用absolute定位或者fixed定位，可以减少对其他元素的影响
        + 只有必要时才显示隐藏元素
+ JavaScript解释器
    + 读取网页中的JS代码，处理后运行
    + js不需要编译，由解释器实时运行  这样的好处是运行和修改都比较方便，刷新页面就可以重新解释；缺点是每次运行都要调用解释器，系统开销较大，运行速度慢于编译型语言
    + 为加快速度，浏览器一般会进行预编译，生成类似字节码的中间代码
    + 即时编译  Just in time 编译器直接把源码编译成机器码来运行 
        + 字节码只在运行时编译，用到哪一行就编译哪一行，然后把编译结果缓存 

## 2.2 Window对象

### 2.2.1 属性

window指当前的浏览器窗口，是当前页面的顶层对象，所有的其他对象都是其下属。如果一个变量未声明，那么默认就是顶层对象的属性。

    a = 1;
    window.a // 1
    
+ window.name  
    + 表示当前窗口的名字
+ window.closed 
    + 检查当前窗口是否关闭
+ window.opener 
    + 表示打开当前窗口的父窗口 
+ window.window   window.window
    + 指向窗口本身，这两个属性都只读
+ window.frames  window,length 
    + frames返回一个类似数组的对象，成员为页面内所有框架窗口
+ window.screenX window.screenY
+ window.innerHeight  window.innerWidth 
    + 可见部分的高度宽度
+ window.outerHeight  window.outerWidth 
+ window.scrollX  window.scrollY
+ 组件属性
    + window.locationbar  地址栏对象
    + window.menubar  菜单栏对象
    + window.scrollbars 窗口的滚动条对象
    + window.toolbar 工具栏对象
    + window.statusbar 状态栏对象
    + window.personalbar 个人安装的工具栏对象
+ 全局对象属性
    + document 
    + location 
    + navigator 
    + history
    + localStorage 
    + sessionStorage 
    + console 
    + screen 
+ 常用方法
    + window.open(url, windowName, [windowFeatures])
        + 打开一个新的浏览器窗口
    + window.close()
    + window.stop()
        + 相当于点击浏览器的停止按钮，会停止加载图像等正在加载的对象
    + window.focus()  获得焦点
    + window.blur()  失去焦点
    + window.getSelection()   表示用户现在选中的文本

    var popup = window.open(
      'somepage.html',
      'DefinitionsWindows',
      'height=200,width=200,location=no,status=yes,resizable=yes,scrollbars=yes'
    );

+ 事件
    + load onload
    + error  onerror 

## 2.3 Navigator对象
https://wangdoc.com/javascript/bom/navigator.html

## 2.4 Cookie对象

### 2.4.1 概述

Cookie是服务器保存在浏览器的一小段文本信息，浏览器每次向服务器发出请求的时候，就会自动附上这段信息

+ 用途
    + session 对话管理，保存登录，购物车等等需要记录的信息
    + 个性化信息  用户偏好等
    + 追踪用户  记录和分析用户的行为
+ 元数据
    + cookie名字
    + cookie值
    + 到期时间
    + 所属域名
    + 生效路径
## 2.5 Location对象

### 2.5.1 Location对象

+ `window.location` or `document.location`去拿到这个对象
+ 属性
    + Location.href  整个URL
    + Location.protocol  当前URL的协议，包括冒号
    + Location.host 主机  
    + Location.hostname  主机  不包括端口号
    + Location.port  端口号
    + Location.pathname  URL的路径部分，从根目录的`/`开始
    + Location.search  查询字符串部分，从问号?开始
    + Location.hash 片段字符串部分  从#开始
    + Location.username 
    + Location.password 
    + Location.origin  URL的协议，主机名和端口



    // 当前网址为
    // http://user:passwd@www.example.com:4097/path/a.html?x=111#part1
    document.location.href
    // "http://user:passwd@www.example.com:4097/path/a.html?x=111#part1"
    document.location.protocol
    // "http:"
    document.location.host
    // "www.example.com:4097"
    document.location.hostname
    // "www.example.com"
    document.location.port
    // "4097"
    document.location.pathname
    // "/path/a.html"
    document.location.search
    // "?x=111"
    document.location.hash
    // "#part1"
    document.location.username
    // "user"
    document.location.password
    // "passwd"
    document.location.origin
    // "http://user:passwd@www.example.com:4097"

Location.href属性是浏览器唯一允许跨域写入的属性，即非同源的窗口可以改写另一个窗口的location.href属性，导致后者的网址跳转。location的其他属性都不允许跨域写入的。

### 2.5.2 方法

+ location.assgin() 
    + 接受一个URL字符串作为参数，使得浏览器立刻跳转到新的URL。如果参数不是有效的URL，则会报错
+ location.replace()
    + 接受一个URL字符串作为参数，使得浏览器立刻跳转到新的URL
    + 与assign的不同在于会在浏览历史里删除掉当前网址
    + 一个应用在于当脚本发现当前是移动设备时，就立刻跳转到移动版网页当中
+ location.reload()
    + 一个布尔值的参数
    + true - 向服务器重新请求这个网页，重新加载后，网页将滚动到头部
    + false - 从本地缓存重新加载该页面，并且重新加载后，网页的位置是重新加载前的位置
+ location.toString()
    + 返回整个URL字符串 


### 2.5.3 URL的编码和解码 

除去URL元字符和语义字符以外的其他字符是需要转义的，比如中文都需要进行转义，转义的时候在每个字节前面都会加上百分号，这就构成了URL的编码/解码语法。

+ encodeURI()
    + 转码整个URL 
    + 参数为一个字符串，代表整个URL 
    + 会将元字符和语义字符之外的字符都进行转义
+ encodeURIComponent()
    + 用于转码URL的组成部分，会转码除了语义字符之外的所有字符，即元字符也会被转码
+ decodeURI()
    + 解码整个URL 
+ decodeURIComponent()
    + 解码片段


    encodeURI('http://www.example.com/q=春节')
    // "http://www.example.com/q=%E6%98%A5%E8%8A%82"
    
    encodeURIComponent('春节')
    // "%E6%98%A5%E8%8A%82"
    encodeURIComponent('http://www.example.com/q=春节')
    // "http%3A%2F%2Fwww.example.com%2Fq%3D%E6%98%A5%E8%8A%82"
    
    decodeURI('http://www.example.com/q=%E6%98%A5%E8%8A%82')
    // "http://www.example.com/q=春节"
    
    decodeURIComponent('%E6%98%A5%E8%8A%82')
    // "春节"

### 2.5.4 接口

+ `var url = new URL('http://www.example.com/index.html');`
+ 接受两个参数，第一个参数表示相对路径，第二个参数表示绝对路径


    var url1 = new URL('index.html', 'http://example.com');
    url1.href
    // "http://example.com/index.html"
    
    var url2 = new URL('page2.html', 'http://example.com/page1.html');
    url2.href
    // "http://example.com/page2.html"
    
    var url3 = new URL('..', 'http://example.com/a/b.html')
    url3.href
    // "http://example.com/"

+ 方法
    +  url.createObjectURL() 
        + 用来为上传下载文件，流媒体文件生成一个URL字符串。这个字符串代表了FIle对象或Blob对象的URL
    + url.revokeObjectURL() 

### 2.5.5 URLSearchParams对象

+ 用来构造，解析，处理URL的查询字符串 


    var params = new URLSearchParams('?foo=1&bar=2');
    // 等同于
    var params = new URLSearchParams(document.location.search);
    
    // 方法二：传入数组
    var params = new URLSearchParams([['foo', 1], ['bar', 2]]);
    
    // 方法三：传入对象
    var params = new URLSearchParams({'foo' : 1 , 'bar' : 2});


## 2.6 Blob对象

ArrayBuffer对象表示一段二进制数据，用来模拟内存里的数据

Blob对象表示一个二进制文件的数据内容，- Binary Large Object.Blob用于操作二进制文件。

+ Blob构造函数
    + `new Blob(array [, options])`
    + 第一个参数 数组 成员是字符串或二进制对象，表示新生成的Blob实例对象的内容
    + 第二个参数  配置对象
        + type 表示数据的MIME类型，默认是空字符串


    var htmlFragment = ['<a id="a"><b id="b">hey!</b></a>'];
    var myBlob = new Blob(htmlFragment, {type : 'text/html'});
    
    var obj = { hello: 'world' };
    var blob = new Blob([ JSON.stringify(obj) ], {type : 'application/json'});

### 2.6.1 实例对象和实例方法

+ size
    + 数据大小
+ type
    + 数据类型
        + text/html 
+ slice(start, end, contentType)
    + 用来拷贝原来的数据，返回的也是一个Blob实例 
    + 起始的字节位置，结束的字节位置，新实例的数据类型

### 2.6.2 获取文件信息

+ 文件选择器 `<input type="file">`用来让用户选取文件
+ 出于安全考虑，浏览器不允许脚本自行设置其value属性，必须手动选取
+ 文件选择器返回一个FileList对象，每个成员都是一个File实例对象。其为一个特殊的Blob实例，增加了name和lastModifiedDate属性。

### 2.6.3 下载文件

AJAX 请求时，如果指定responseType属性为blob，下载下来的就是一个 Blob 对象。

    function getBlob(url, callback) {
      var xhr = new XMLHttpRequest();
      xhr.open('GET', url);
      xhr.responseType = 'blob';
      xhr.onload = function () {
        callback(xhr.response);
      }
      xhr.send(null);
    }

### 2.6.4 生成URL

+ 可以通过使用`URL.createObjectURL()`方法，针对Blob对象生成一个临时URL，以便于某些API使用
+ URL以`Blob://`开头，表明对应一个Blob对象，协议头后面是一个识别符，用来唯一对应内存的Blob对象


    var droptarget = document.getElementById('droptarget');
    
    droptarget.ondrop = function (e) {
      var files = e.dataTransfer.files;
      for (var i = 0; i < files.length; i++) {
        var type = files[i].type;
        if (type.substring(0,6) !== 'image/')
          continue;
        var img = document.createElement('img');
        img.src = URL.createObjectURL(files[i]);
        img.onload = function () {
          this.width = 100;
          document.body.appendChild(this);
          URL.revokeObjectURL(this.src);
        }
      }
    }

该段代码通过为拖放的图片文件生成一个URL，产生他们的缩略图，从而使得用户可以预览选择的文件。

### 2.6.5 读取文件

获取Blob对象以后，通过FileReader对象，读取Blob对象的内容。Blob对象作为参数传入FilreReader提供的处理方法当中，然后以指定的格式返回

+ FileReader.readAsText()   返回文本，需要指定文本编码
+ FileReader.readAsArrayBuffer()   返回ArrayBuffer对象
+ FileReader.readAsDataURL()  返回Data URL
+ FileReader.readAsBinaryString()  返回原始的二进制字符串

## 2.7 表单 FormData对象

### 2.7.1 General
表单对象用来收集用户提交的数据，发送到服务器。

    <form action="/handling-page" method="post">
      <div>
        <label for="name">用户名：</label>
        <input type="text" id="name" name="user_name" />
      </div>
      <div>
        <label for="passwd">密码：</label>
        <input type="password" id="passwd" name="user_passwd" />
      </div>
      <div>
        <input type="submit" id="submit" name="submit_button" value="提交" />
      </div>
    </form>

键值对会提高到服务器当中，提交的数据格式跟form元素的method属性有关，该属性指定了提交数据的HTTP方法。如果是GET方法，所有键值对会以URL的查询字符串形式，提交到服务器当中。

如果是post请求，所有键值对会连接成一行，作为HTTP请求的数据体发送到服务器

    GET /handling-page?user_name=张三&user_passwd=123&submit_button=提交
    Host: example.com
    
    POST /handling-page HTTP/1.1
    Host: example.com
    Content-Type: application/x-www-form-urlencoded
    Content-Length: 74
    
    user_name=张三&user_passwd=123&submit_button=提交

注意如果提交的时候，如果键值不是URL的合法字符，浏览器就会自动对其进行编码。

点击submit控件就可以提交表单了。

### 2.7.2 FormData对象

通过脚本完成对于表单键值对的构建，然后通过`XMLHttpRequest.send()`方式来进行发送，浏览器原生提供了FormData对象来完成这项工作。


+ FormData() 构造函数的参数是一个表单元素，这个参数是可选的，如果省略参数，就表示一个空的表单，否则就会处理表单元素里面的键值对。


    <form id="myForm" name="myForm">
      <div>
        <label for="username">用户名：</label>
        <input type="text" id="username" name="username">
      </div>
      <div>
        <label for="useracc">账号：</label>
        <input type="text" id="useracc" name="useracc">
      </div>
      <div>
        <label for="userfile">上传文件：</label>
        <input type="file" id="userfile" name="userfile">
      </div>
    <input type="submit" value="Submit!">
    </form>


    var myForm = document.getElementById('myForm');
    var formData = new FormData(myForm);
    
    // 获取某个控件的值
    formData.get('username') // ""
    
    // 设置某个控件的值
    formData.set('username', '张三');
    
    formData.get('username') // "张三"


+ FormData.get(key)
+ FormData.getAll(key)
+ FormData.set(key, value)
+ FormData.delete(key)
+ FormData.append(key, value)
+ FormData.has(key)
+ FormData.keys()  
+ FormData.values()
+ FormData.entries() 

### 2.7.3 表单的内置验证

+ 自动校验
    + 可以在表单提交的时候指定一些条件，来自动验证各个表单控件的值是否符合条件


    <!-- 必填 -->
    <input required>
    
    <!-- 必须符合正则表达式 -->
    <input pattern="banana|cherry">
    
    <!-- 字符串长度必须为6个字符 -->
    <input minlength="6" maxlength="6">
    
    <!-- 数值必须在1到10之间 -->
    <input type="number" min="1" max="10">
    
    <!-- 必须填入 Email 地址 -->
    <input type="email">
    
    <!-- 必须填入 URL -->
    <input type="URL">

    // 通过CSS伪类来控制整个
    input:invalid {
      border-color: red;
    }
    input,
    input:valid {
      border-color: #ccc;
    }

+ checkValidity() 
    + 用于手动触发表单的校验
    + 表单元素和表单控件都有checkValidity()方法，用于手动触发校验


    function submitForm(action) {
      var form = document.getElementById('form');
      form.action = action;
      if (form.checkValidity()) {
        form.submit();
      }
    }


+ validationMessage 属性
    + 返回一个字符串，表示控件不满足校验条件时，浏览器显示的提示文本


    var myInput = document.getElementById('myinput');
    if (!myInput.checkValidity()) {
      document.getElementById('prompt').innerHTML = myInput.validationMessage;
    }

+ validity属性
    + 返回一个validityState对象，包含当前校验状态的信息
    + 属性
        + validityState.badInput
        + validityState.customError 
        + validityState.patternMismatch
        + validityState.rangeOverflow 
        + validityState.rangeUnderflow 
        + validityState.stepMismatch 
        + validityState.tooLong
        + validityState.tooShort 
        + validityState.typeMismatch 
        + validityState.valid
        + valitityState.valueMising 

+ 表单novalide属性
    + 可以关闭浏览器的自动校验

### 2.7.4 enctype属性
+ GET - URL查询字符串
    + enctype属性无效 
+ POST - `application/x-www-form-yrlencoded`
    + 当省略enctype属性的时候，数据会以默认的`application/x-www-form-urlencoded`格式进行发送 
+ POST - `text/plain`


    <form
      action="register.php"
      method="post"
      enctype="text/plain"
      onsubmit="AJAXSubmit(this); return false;"
    >
    </form>

+ POST - `multipart/form-data`
    + 数据以混合的方式来发送出去

# Reference 

1. https://wangdoc.com/javascript/events/index.html 
2. https://segmentfault.com/a/1190000008227026