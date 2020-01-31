---
title: JavaScript 教程 - part 2
date: 2020-01-30 22:28:48
categories: FrontEnd
tags:
    - JavaScript
    - Object Oriented
    - FrontEnd
top:
---

# 1. 面向对象编程

## 1.1 实例对象与New命令

+ 对象
    + 对世界各种复杂关系的抽象
    + 是一个容器，封装了属性和方法
+ 构造函数
    + java c++是通过类的概念来作为对象的模板，而对象就是类的实例
    + 而在js当中，对象体系是基于构造函数和原型链的
### 1.1.1 构造函数
构造函数是用来专门生成实例对象的函数，它就是对象的模板，描述了对象的基本结构。

+ 构造函数的特点：
    + 函数体内部使用**this关键字**，代表了所要生成的对象实例
    + 生成对象的时候，必须使用new命令


    var Vehicle = function () {
      this.price = 1000;
    };

### 1.1.2 New

+ new命令的作用就是执行构造函数返回一个实例对象
+ new指令执行的时候，构造函数内部的this就代表了新生成的实例对象
+ new指令的原理
    + 构建一个空对象，作为要返回的对象的实例
    + 将这个空对象的原型，指向构造函数的prototype属性
    + 将这个空对象赋值给函数内部的this关键字
    + 开始执行构造函数内部的代码


    var Vehicle = function () {
      this.price = 1000;
    };
    
    var v = new Vehicle();
    v.price // 1000


+ new.target
    + 使用new.target关键字
    + 如果当前函数式new命令调用的，指向当前函数，否则为undefined
+ `Object.create()`创建实例对象
    + 构造函数作为模板，可以生成实例对象。但是，有时拿不到构造函数，只能拿到一个现有的对象。我们希望以这个现有的对象作为模板，生成新的实例对象，这时就可以使用Object.create()方法。


    var person1 = {
      name: '张三',
      age: 38,
      greeting: function() {
        console.log('Hi! I\'m ' + this.name + '.');
      }
    };
    
    var person2 = Object.create(person1);
    
    person2.name // 张三
    person2.greeting() // Hi! I'm 张三.

## 1.2 this关键字

this是属性或者方法当前所在的对象。

对象的属性可以赋给另外一个对象，所以属性所在的当前对象是可变的，this的指向是可变的

    function f() {
      return '姓名：'+ this.name;
    }
    
    var A = {
      name: '张三',
      describe: f
    };
    
    var B = {
      name: '李四',
      describe: f
    };
    
    A.describe() // "姓名：张三"
    B.describe() // "姓名：李四"



    <input type="text" name="age" size=3 onChange="validate(this, 18, 99);">
    
    <script>
    function validate(obj, lowval, hival){
      if ((obj.value < lowval) || (obj.value > hival))
        console.log('Invalid Value!');
    }
    </script>
    
+ JS当中一切都是对象，运行环境也是对象，所以函数总归是在某个对象之中运行的，this就是函数运行时所在的对象
+ this为的是在函数体内可以获得函数的当前运行环境的相关信息

### 1.2.1 this实质

+ 首先需要看我们是如何在内存当中进行数据存储的
    + `var obj = { foo: 5}`;
        + 将一个对象赋值给变量obj
        + JS引擎会先在内存里面生成一个对象
        + 将这个对象的内存地址赋值给变量obj
        + ----> 变量obj是一个地址 reference。后面如果要读取obj.foo， 引擎先从obj拿到内存地址，然后再从该地址读出原始对象，返回其foo属性
    + 原始的对象以字典结构存储，每一个属性名都对应一个属性描述对象
        + 如果对象的属性是一个函数的话
        + JS引擎会将函数单独保存在内存当中，然后将函数的地址赋值给foo的value属性
        + 因为**函数是一个单独的值，所以它可以在不同的上下文去执行**
    + this的想法，或者说目的就是在函数体内部，指代函数当前的运行环境


    // 对象的实际保存形式
    {
      foo: {
        [[value]]: 5
        [[writable]]: true
        [[enumerable]]: true
        [[configurable]]: true
      }
    }
    
    var f = function () {
        console.log(this.x);
    }


### 1.2.2 使用场合

+ 全局环境
    + 指顶层对象window
+ 构造函数
    + 指实例对象 
    + 在构造函数当中使用this，定义了属性，那就意味着定义了实例对象有一个p属性



    // this指实例对象，这个时候为实例对象定义了p属性
    var Obj = function(p) {
        this.p = p;
    };
+ 对象方法
    + this指向是方法运行时所在的对象，该方法赋值给另一个对象，就会改变this的指向
    + 这是因为obj.foo就是一个值，这个值真正调用的时候，运行环境已经不是obj了，而是全局环境，所以this不再指向obj


    // 指向obj
    var obj ={
      foo: function () {
        console.log(this);
      }
    };
    
    obj.foo() // obj
    
    // 以下情况都会改变this的指向
    // 情况一
    (obj.foo = obj.foo)() // window
    // 情况二
    (false || obj.foo)() // window
    // 情况三
    (1, obj.foo)() // window

### 1.2.3 Tips

+ 避免多层this
    + 因为this的指向是不确定的，所以不应该在函数当中包含多层的this
    + 如果是多层的话，可以在第二层增加一个指向外层this的变量来做


    var o = {
      f1: function () {
        console.log(this);
        var f2 = function () {
          console.log(this);
        }();
      }
    }
    
    o.f1()
    // Object
    // Window
    
    var o = {
      f1: function() {
        console.log(this);
        var that = this;
        var f2 = function() {
          console.log(that);
        }();
      }
    }
    
    o.f1()
    // Object
    // Object

+ 数组处理方法当中的this
    + 还是需要使用中间变量来固定住this 


### 1.2.4 绑定this的方法

this的动态切换，可以为JS提供灵活性，但也使得编程变得困难和模糊，因此我们需要将this固定下来，避免出现意想不到的情况。JS提供了call, apply, bind三个方法，来切换和固定this的指向


+ `Function.prototype.call()`
    + call方法可以指定函数内部的this的指向，(函数执行时所在的作用域)，然后在所指定的作用域当中，调用该函数
    + call可以接受多个参数 `func.call(thisValue, arg1, arg2)` 
    + 第一个参数是this所要指向的那个对象，后面的参数则是函数调用时所需的参数


    var obj = {};
    
    var f = function () {
      return this;
    };
    
    f() === window // true
    // call可以带个对象，然后这个时候就将this的指向固定下来了
    f.call(obj) === obj // true

+ `Function.prototype.apply()`
    + apply接收一个数组作为函数执行时的参数，其他和call函数是一致的
    + `func.apply(thisValue, [arg1, arg2, ...])`


    function f(x, y){
      console.log(x + y);
    }
    
    f.call(null, 1, 1) // 2
    f.apply(null, [1, 1]) // 2

+ `Function.prototype.bind()`
    + bind方法用于将函数体内的this绑定到某个对象，然后返回一个新函数
    + bind方法每次都会返回一个新函数


    // 上面代码中，bind方法将getTime方法内部的this绑定到d对象，这时就可以安全地将这个方法赋值给其他变量了。
    var d = new Date();
    d.getTime() // 1481869925657
    
    var print = d.getTime;
    print() // Uncaught TypeError: this is not a Date object.
    
    var print = d.getTime.bind(d);
    print() // 1481869925657

## 1.3 对象的继承
A对象通过继承B对象，就能直接拥有B对象的所有属性和方法，这对于代码的复用很有帮助。JavaScript通过原型对象(prototype)来实现继承关系。

注意ES6当中引入了class语法，可以做基于class的继承了

### 1.3.1 原型对象概述
+ 构造函数
    + JS通过构造函数生成新对象，因此构造函数可以视为对象的模板
    + 实例对象的属性和方法定义在构造函数的内部
    + 属性实际上是定义在了实例对象上的，但是同一个构造函数的多个实例之间，无法共享属性，从而造成了资源的浪费
    + JavaScript使用原型对象来解决这个问题  -- prototype 
        + 其作用就是定义所有实例对象共享的属性和方法 

    function Cat (name, color) {
      this.name = name;
      this.color = color;
    }
    
    var cat1 = new Cat('大毛', '白色');
    
    cat1.name // '大毛'
    cat1.color // '白色'

+ prototype属性的作用
    + 原型对象的所有属性和方法都能被实例对象共享
    + 如果属性和方法定义在原型上，那么所有实例对象都能共享
        + 节省内存
        + 体现实例对象之间的联系
    + 原型对象的属性不是实例对象自身的属性，只要修改原型对象，变动就立刻会体现在所有实例对象上
    + 当实例对象没有某个属性或者方法的时候，会与原型对象上去找，如果有，就用自己的


    function Animal(name) {
      this.name = name;
    }
    Animal.prototype.color = 'white';
    
    var cat1 = new Animal('大毛');
    var cat2 = new Animal('二毛');
    
    cat1.color // 'white'
    cat2.color // 'white'
    
    cat1.color = 'black';
    cat1.color // 'black'

+ 原型链
    + JS规定所有对象都有自己的原型对象(prototype)
    + 一层层回溯，所有对象的原型最终上溯到Object.prototype,即Object构造函数的prototype属性
    + 而Object.prototype的原型是null
    + 读取对象的某个属性时，先寻找对象本身的属性，如果找不到，就去原型找，如果还是找不到，就去原型的原型，一层层溯源。一直没有就返回undefined
    + 如果对象自身和它的原型**都定义了同名属性，那么优先读取对象自身的属性**， -- overriding 


    var MyArray = function () {};
    
    MyArray.prototype = new Array();
    MyArray.prototype.constructor = MyArray;
    
    // mine是MyArray的一个实例对象，由于MyArray.protutype指向一个数组实例，使得mine可以调用数组方法
    var mine = new MyArray();
    mine.push(1, 2, 3);
    mine.length // 3
    mine instanceof Array // true


+ constructor属性
    + prototype对象的constructor属性，默认指向prototype对象所在的构造函数
    + constructor属性的作用是可以得知某个是力度向，到底是哪一个构造函数产生的
+ instanceof运算符
    + 返回一个布尔值，表示对象是否为某个构造函数的实例 

+ 构造函数的继承
    + 让一个构造函数继承另一个构造函数
        + 在子类的构造函数当中，调用父类的构造函数
        + 让子类的原型指向父类的原型，使得子类继承父类的原型

    
    // Sub是构造函数，this是子类的实例
    // 子类上调用父类的构造函数super，使得子类实例具有父类实例的属性
    function Sub(value) {
      Super.call(this);
      this.prop = value;
    }
    
    // 让子类的原型指向父类的原型，使得子类继承父类的原型
    // 使用Object.create是为了不对父类原型造成修改
    Sub.prototype = Object.create(Super.prototype);
    Sub.prototype.constructor = Sub;
    Sub.prototype.method = '...';


一个继承的实例:

    function Shape() {
      this.x = 0;
      this.y = 0;
    }
    
    Shape.prototype.move = function (x, y) {
      this.x += x;
      this.y += y;
      console.info('Shape moved.');
    };
    
    // 第一步，子类继承父类的实例
    function Rectangle() {
      Shape.call(this); // 调用父类构造函数
    }
    // 另一种写法
    function Rectangle() {
      this.base = Shape;
      this.base();
    }
    
    // 第二步，子类继承父类的原型
    Rectangle.prototype = Object.create(Shape.prototype);
    Rectangle.prototype.constructor = Rectangle;

## 1.4 Object对象的相关方法

+ Object.getPrototypeOf()
    +  返回参数对象的原型


    // 空对象的原型是 Object.prototype
    Object.getPrototypeOf({}) === Object.prototype // true
    
    // Object.prototype 的原型是 null
    Object.getPrototypeOf(Object.prototype) === null // true
    
    // 函数的原型是 Function.prototype
    function f() {}
    Object.getPrototypeOf(f) === Function.prototype // true

+ Object.setPrototypeOf() 
    +  为参数对象设置原型，返回该参数的对象
    +  第一个参数为现有对象，第二个为原型对象


    var a = {};
    var b = {x: 1};
    Object.setPrototypeOf(a, b);
    
    Object.getPrototypeOf(a) === b // true
    a.x // 1

+ Object.create()
    + 接受一个对象作为参数，然后以它为原型，返回一个实例对象。该实例完全继承原型对象的属性
+ Object.prototype.__proto__
    + 实例对象的该属性，返回该对象的原型 

## 1.5 严格模式

该种模式采用更加严格的JavaScript语法
+ 明确机制不合理，不严谨的语法
+ 增加更多的报错场合，消除代码运行的不安全之处，保证代码的运行安全
+ 提高编译器效率，增加运行速度


开启的话，使用`use strict`

+ use strict放在脚本的第一行，整个脚本都将以严格模式来运行，如果不在第一行就无效，会以正常模式运行
+ 显式报错
    + 只读属性不可写
    + 只设置了getter的不可写
+ 安全措施
    + 全局变量显式声明
    + 禁止this关键字指向全局对象
    

# 2. 异步操作

## 2.1 General

### 2.1.1 单线程模型

JS只在一个线程上运行的  单个脚本只能在一个线程上运行，其他线程都是在后台配合的

好吃执行简单，坏处就是慢，只要一个任务耗时很长，那么后面的任务就都需要排队等着，会拖延整个程序的执行。

JS本身不慢，慢的是读写外部数据，等待Ajax请求返回结果。这个时候如果对方服务器一直没有反应，或者网络不通畅，就会导致脚本的长时间停滞。

因为很多时候慢在IO操作，而CPU实际上是处于闲置状态的，因此CPU这个时候完全可以不管IO操作，挂起处于等待中的任务，先运行排在后面的任务，等到IO操作返回结果，再回头将挂起的任务继续执行下去


### 2.1.2 同步任务与异步任务

+ 同步任务
    + 没有被引擎挂起，在主线程上排队执行的任务
    + 只有前一个任务执行完毕，后一个任务才能开始执行
+ 异步任务
    + 被引擎放在一边，不进入主线程，而进入任务队列的任务
    + 只有引擎认为某个异步任务可以执行了，该任务才会进入主线程执行
    + 排在异步任务后面的代码。不用等待异步任务结束会马上运行，也就是说，异步任务不具有堵塞效应


### 2.1.3 任务队列和事件循环

+ 任务队列
    + 存储需要当前程序处理的异步任务
    + 主线程会执行所有的同步任务，等到同步任务执行完以后，就会去看任务队列里面的异步任务
    + 如果满足条件，异步任务就重新进入主线程进行执行，此时变成了同步任务了
    + 异步任务的写法即回调函数
+ 事件循环
    + 主线程来看异步任务是否有结果的方式
    + 就是不断检查，去看挂起来的任务们有没有结果了

# 2.1.4 异步操作的模式

+ 回调函数


    function f1(callback) {
      // ...
      callback();
    }
    
    function f2() {
      // ...
    }
    
    f1(f2);


+ 事件监听
    + 异步任务的执行不取决于代码的顺序，而取决于某个事件是否发生


    f1.on('done', f2);
    function f1() {
      setTimeout(function () {
        // ...
        f1.trigger('done');
      }, 1000);
    }


+ 发布订阅
    + 某个任务执行完成，向信号中心发布一个信号
    + 其他任务向信号中心订阅这个信号，来知道什么时候自己可以开始执行



    jQuery.subscribe('done', f2);
    
    function f1() {
      setTimeout(function () {
        // ...
        jQuery.publish('done');
      }, 1000);
    }

### 2.1.5 异步操作的流程控制

+ 串行执行


    var items = [ 1, 2, 3, 4, 5, 6 ];
    var results = [];
    
    function async(arg, callback) {
      console.log('参数为 ' + arg +' , 1秒后返回结果');
      setTimeout(function () { callback(arg * 2); }, 1000);
    }
    
    function final(value) {
      console.log('完成: ', value);
    }
    
    function series(item) {
      if(item) {
        async( item, function(result) {
          results.push(result);
          return series(items.shift());
        });
      } else {
        return final(results[results.length - 1]);
      }
    }
    
    series(items.shift());


+ 并行执行
    + 并行会快，但是如果并行任务太多，很容易耗尽系统的资源，拖慢运行速度 


    var items = [ 1, 2, 3, 4, 5, 6 ];
    var results = [];
    
    function async(arg, callback) {
      console.log('参数为 ' + arg +' , 1秒后返回结果');
      setTimeout(function () { callback(arg * 2); }, 1000);
    }
    
    function final(value) {
      console.log('完成: ', value);
    }
    
    items.forEach(function(item) {
      async(item, function(result){
        results.push(result);
        if(results.length === items.length) {
          final(results[results.length - 1]);
        }
      })
    });


+ 并行与串行相结合!!!!!
    + 设置一个门槛，每次最多只能并行执行n个异步任务


    var items = [ 1, 2, 3, 4, 5, 6 ];
    var results = [];
    var running = 0;
    var limit = 2;
    
    function async(arg, callback) {
      console.log('参数为 ' + arg +' , 1秒后返回结果');
      setTimeout(function () { callback(arg * 2); }, 1000);
    }
    
    function final(value) {
      console.log('完成: ', value);
    }
    
    function launcher() {
      while(running < limit && items.length > 0) {
        var item = items.shift();
        async(item, function(result) {
          results.push(result);
          running--;
          if(items.length > 0) {
            launcher();
          } else if(running == 0) {
            final(results);
          }
        });
        running++;
      }
    }
    
    launcher();

 ## 2.2 定时器

JS提供定时执行代码的功能，由setTimeout()以及setInterval()两个函数来完成，它们可以向任务队列添加定时任务。

+ setTimeout()
    + 指定某个函数或者某段代码，在多少毫秒之后执行
    + 返回一个整数，表示定时器的编号
    + `var timerId = setTimeout(func|code, delay)`


    // 1,1是回调函数的参数
    setTimeout(function (a,b) {
      console.log(a + b);
    }, 1000, 1, 1);

+ setInterval()
    + setInterval是设置某个任务每隔一段时间就执行一次，为无限次的执行的
    + 指定的是开始执行之间的间隔，没有考虑每次任务执行本身所消耗的时间
    + 为了确定两次执行之间的固定间隔，可以使用setTimeout来指定下一次执行的具体时间


    var i = 1;
    var timer = setTimeout(function f() {
      // ...
      timer = setTimeout(f, 2000);
    }, 2000);


+ clearTimeout() clearInterval()
    + 传入对应的计数器编号，就可以取消对应的定时器了


### 2.2.1 Debounce 函数

想要解决的问题：不希望回调函数被频繁调用，即当用户填入网页输入框的内容以后，用户会点击button希望能传回数据，但是很有可能是在多次重复相同的信息的。

因此我们应该加一个门槛，表示两次Ajax通信的最小间隔时间。如果在间隔时间内发生了新的keydown时间，就不触发Ajax通信，并且开始重新计时。如果过了指定时间，没有发生新的keydown事件，再将数据发送出去。


    // 在2.5秒内，当用户再次敲击的时候，会取消上次定时器，然后再新建一个定时器
    $('textarea').on('keydown', debounce(ajaxAction, 2500));
    
    function debounce(fn, delay){
      var timer = null; // 声明计时器
      return function() {
        var context = this;
        var args = arguments;
        clearTimeout(timer);
        timer = setTimeout(function () {
          fn.apply(context, args);
        }, delay);
      };
    }

### 2.2.2 运行机制

setTimeout和setInterval都是将指定的代码移出本轮事件循环，等到下一轮事件循环，再检查时间是否到了指定的时间。如果到了，就执行对应的代码，如果不到，就继续等待。

这意味着，setTimeout 和 setInterval指定的回调函数，必须等到**本轮事件循环**的所有同步任务都执行完. 由于前面的任务到底需要多少时间执行完，是不确定的，所以我们是无法保证二者指定的任务一定会按照预定时间执行



    setInterval(function () {
      console.log(2);
    }, 1000);
    
    sleep(3000);
    
    function sleep(ms) {
      var start = Date.now();
      while ((Date.now() - start) < ms) {
      }
    }

上面例子会先sleep 3秒然后才会输出2，注意是不累计的，即只输出一个2的

### 2.2.3 setTimeout(f,0) 及其应用

setTimeout(f,0)也不会直接执行的，因为它必须要等待当前脚本的同步任务全部处理完以后，才会执行setTimeout指定的回调函数f，即setTimeout(f,0)会在下一轮事件循环开始的时候执行

    setTimeout(function () {
      console.log(1);
    }, 0);
    console.log(2);
    // 2
    // 1

**setTimeout的应用**
+ 调整事件的发生顺序
+ 调整用户自定义的回调函数和浏览器默认动作之间的执行顺序
+ setTimeout(f,0)意味着将任务放到浏览器最早可得的空闲时段执行，所以那些计算量大，耗时长的任务通常可以分成几个小部分分别方法这个方法当中去


    // HTML 代码如下
    // <input type="button" id="myButton" value="click">
    
    var input = document.getElementById('myButton');
    
    input.onclick = function A() {
      setTimeout(function B() {
        input.value +=' input';
      }, 0)
    };
    
    document.body.onclick = function C() {
      input.value += ' body'
    };
    
    
    // HTML 代码如下
    // <input type="text" id="input-box">
    
    document.getElementById('input-box').onkeypress = function (event) {
      this.value = this.value.toUpperCase();
    }
    
    
    document.getElementById('input-box').onkeypress = function() {
      var self = this;
      setTimeout(function() {
        self.value = self.value.toUpperCase();
      }, 0);
    }
    
    
    
    var div = document.getElementsByTagName('div')[0];
    
    // 写法一
    for (var i = 0xA00000; i < 0xFFFFFF; i++) {
      div.style.backgroundColor = '#' + i.toString(16);
    }
    
    // 写法二
    var timer;
    var i=0x100000;
    
    function func() {
      timer = setTimeout(func, 0);
      div.style.backgroundColor = '#' + i.toString(16);
      if (i++ == 0xFFFFFF) clearTimeout(timer);
    }
    
    timer = setTimeout(func, 0);


## 2.3 Promise对象

JS的异步操作解决方案，为异步操作提供统一的接口，起到代理的作用，充当异步操作与回调函数之间的中介，使得异步操作具备同步操作的接口。Promise可以让异步操作写法和写同步操作相似，不必一层层地嵌套回调函数

    function f1(resolve, reject) {
        // 异步代码
    }
    
    var p1 = new Promise(f1);

上述例子当中接受一个回调函数f1作为参数，f1里面是异步操作的代码，然后返回p1为一个Promise实例。

所有异步任务都返回一个Promise实例，然后通过自带的then方法，来指定下一步的回调函数。通过then的写法，来减少函数和函数之间的嵌套的写法。

### 2.3.1 Promise对象的状态

+ 异步操作未完成 - pending
+ resolved
    + 异步操作成功 - fulfilled
    + 异步操作失败 - rejected

一旦状态发生变化，就不会再有新的状态变化了，即Promise的实例的状态变化只可能发生一次

Promise原生构造函数接受一个函数作为参数，有两个参数，分别为resolve还有reject。

### 2.3.2 Promise.prototype.then()

+ then方法用来添加回调函数
    + 第一个回调函数是异步操作成功用的
    + 第二个是异步操作失败使用
+ Promise的报错具有传递性


    p1
      .then(step1)
      .then(step2)
      .then(step3)
      .then(
        console.log,
        console.error
      );

在上述的例子里面，console.log 只显示step3的内容，但是console.error会显示出step1，2，3出现的任意错误。Promise对象的报错具有传递性 

# 3. DOM

## 3.1 Intro

DOM是JavaScript操作网页的接口，全称为文档对象模型(Document Object Model).它的作用是将网页转为一个JavaScript对象，从而可以用脚本进行各种操作。

浏览器根据DOM模型，将结构化文档解析成一系列的节点，再由这些节点组成一个树状结构 - DOM tree。所有的节点和最终的树状架构，都有规范的对外接口。

+ 节点  
    + DOM的最小组成单位
+ 节点类型
    + Document 文档树的顶层节点
    + DocumentType doctype标签，比如    `<!DOCTYPE html>`
    + Element 网页的各种HTML标签
    + Attribute  网页元素的属性
    + Text  标签之间或标签包含的文本
    + comment 注释
    + DocumentFragment  文档片段
## 3.2 Node简述
###  3.2.1 Node接口

+ nodeType 
    + 返回一个整数值，表示节点的类型
        + document 9 
        + element  1
        + attr     2
        + text     3
        + documentFragment 11 
        + documentType     10
        + Comment          8
+ nodeName
    + 返回节点的名称
+ nodeValue
    + 返回一个字符串，表示当前节点本身的文本值，该属性可读写


    var div = document.getElementById('d1');
    div.nodeValue // null
    div.firstChild.nodeValue // "hello world"

+ Node.prototype.textContent
    + textContent属性返回当前节点和它的所有后代节点的文本内容
    + 该属性是可读写的，设置该属性的值，会用一个新的文本节点代替原有的子节点
    + 注意转译的时候HTML标签会被忽略掉的


    document.getElementById('foo').textContent = '<p>GoodBye!</p>';

+ Node.prototype.baseURI
    + 返回一个字符串，表示当前网页的绝对路径
    + 浏览器会根据这个属性计算网页上的相对路径的URL  
    + 该属性为只读 


    document.baseURI
    // "http://www.example.com/index.html"

+ Node.prototype.ownerDocument 
    + 返回当前节点所在的顶层文档对象，即document对象

+ Node.prototype.nextSibling 
    + 返回紧跟在当前节点后面的第一个同级节点 
    + 如果当前节点后面没有同级节点，就返回null 
    + 该属性可以用来遍历所有子节点


    var d1 = document.getElementById('d1');
    var d2 = document.getElementById('d2');
    
    d1.nextSibling === d2 // true

    // 遍历所有同级节点
    var el = document.getElementById('div1').firstChild;
    
    while (el !== null) {
      console.log(el.nodeName);
      el = el.nextSibling;
    }

+ Node.prototype.previousSibling 
    + 返回倩倩节点前面的，距离最近的，同级的节点
    + 如果没有同级节点，就返回null 
+ Node.prototype.parentNode
    + 返回当前节点的父节点
    + 对于一个节点来说，其父节点可能为：
        + element 
        + document 
        + documentFragment 
+ Node.prototype.parentElement 
+ Node.prototype.firstChild
+ Node.prototype.lastChild
+ Node.prototype.childNodes
    + 返回Nodelist集合，包括当前节点的所有子节点
+ Node.prototype.isConnected
    + 返回一个布尔值，表示当前节点是否在文档当中


### 3.2.2 node方法

+ appendChild()
    + 接受一个节点对象作为参数，将其作为最后一个子节点，插入当前节点
    + 如果参数节点是DOM已经存在的节点，appendChild方法会将其从原来的位置移动到新位置
+ hasChildNodes()
    + 返回一个布尔值，表示当前节点是否有子节点
+ cloneNode()
    + 克隆一个节点，接受一个布尔值作为参数， 表示是否同时克隆子节点
    + 返回值为一个克隆出来的新节点
+ insertBefore()
    + 用来将某个节点插入父节点内部的指定位置
    +  `var insertedNode = parentNode.insertBefore(newNode, referenceNode);`
+ removeChild()
    + 接受一个子节点作为参数，用于从当前节点移除该子节点
+ replaceChild()
    + 用于将一个新的节点替换当前节点的某一个子节点
+ contains()
    + 返回布尔值，看参数节点是否满足以下条件
        + 参数节点为当前节点
        + 为当前节点的子节点
        + 为当前节点的后代节点
+ getRootNode()
    + 返回当前节点所在文档的根节点document,与ownerDocument属性的作用相同 

## 3.3 Document节点

document节点对象代表整个文档，每个网页都有自己的document对象，`window.document`属性就指向这个对象

+ 正常网页可以直接使用docuemnt 还有window.document
+ iframe框架里面的网页，使用iframe节点的contentDocument属性
+ Ajax操作返回的文档，使用XMLHttpRequest对象的responseXML属性


### 3.3.1 快捷方式属性

+ document.defaultView 
    + 返回document对象所属的window对象
+ document.doctype
    

    var doctype = document.doctype;
    doctype // "<!DOCTYPE html>"
    doctype.name // "html"
+ document.documentElement 
    + 返回当前文档的根元素节点
    + 通常为document节点的第二个子节点，紧跟在document.doctype节点后面
+ document.body
    + 指向body节点
+ document.head
    + 指向head节点
+ document.scrollingElement 
    + 返回文档的滚动元素
+ document.activeElement 
    + 返回获得当前焦点的DOM元素
    + 通常这个属性返回的是`<input> <textarea> <select>`这类表单元素
+ document.fullscreenElement 
    + 返回当前以全屏状态展示的DOM元素 

### 3.3.2 节点集合属性

返回一个HTMLCollection实例，表示文档内部特定元素的集合。这些集合是动态的，原节点有任何变化，立刻会反映在集合当中 

+ document.links 
+ document.forms
+ document.images 
+ document.embeds  / document.plugins 
    + 返回所有<embed>节点
+ document.scripts
+ document.styleSheets


### 3.3.3 文档静态信息属性

+ document.documentURI / document.URL
    + 返回一个字符串，表示当前文档的网址
    + 不同之处在于他们继承自不同的接口
    + documentURI 继承自Document接口，可用于所有文档
    + URL 继承自HTMLDocument接口，只可用于HTML文档
+ document.domain
    + 返回当前文档的域名 
    + 比如，网页的网址是http://www.example.com:80/hello.html，那么document.domain属性就等于www.example.com
+ document.location 
    + 浏览器原生对象，提供URL相关的信息和操作方法
    + 通过window.location 和document.location属性，可以拿到这个对象
+ document.lastModified 
    + 返回当前文档的问候修改时间
+ document.title 
+ document.characterSet 
    + UTF-8
    + ISO-8859-1
    + etc.
+ document.referrer 
    + 返回字符串，表示当前文档的访问者来的地方
+ document.dir 
    + 返回一个字符串
        + rtl
        + ltr
+ document.compatMode
    + 返回浏览器处理文档的模式 
    + BackCompat 向后兼容模式
    + CSS1Compat 严格模式


### 3.3.4 文档状态属性

+ hidden
    + 返回一个布尔值
+ visibilityState  返回当前文档的可见状态
    + visible
    + hidden
    + prerender 正在渲染的状态，对于用户来说，这个页面不可见
    + unloaded 从内存中卸载了
+ readyState  返回当前文档的状态
    +  loading 加载HTML代码阶段
    +  interactive  加载外部资源阶段
    +  complete  加载完成
+ cookie
+ designMode 
    + on/ off
    + 当开启以后，用户就可以编辑整个文档的内容了
+ implementation 
    + 返回一个DOMImplementation对象 
    + DOMImplementation.createDocument()
    + DOMImplementation.createHTMLDocument()
    + DOMImplementation.createDocumentType() 

### 3.3.5 Document相关方法

+ document.open()  document.close() 
    + open用来清除当前文档的所有内容，使得文档处于可写状态
    + document.close用来关闭打开的文档
+ document.write()  document.writeln()
    + write用于向当前文档写入内容
    + 当页面已经解析完成以后，再调用write会先调用open方法，这样所有文档的内容就已经被擦除了
    + 在页面渲染过程中的调用write方法，并不会自动调用open方法
    + writeln是在末尾会添加换行符



    document.addEventListener('DOMContentLoaded', function (event) {
      document.write('<p>Hello World!</p>');
    });
    
    // 等同于
    document.addEventListener('DOMContentLoaded', function (event) {
      document.open();
      document.write('<p>Hello World!</p>');
      document.close();
    });

+ document.querySelector()  document.querySelectorAll()
    + 接受一个CSS选择器作为参数，返回匹配该选择器的元素节点。如果多个节点满足匹配条件，则返回第一个匹配的节点。如果没有发现匹配的节点，则返回null
    + querySelectorAll方法与querySelector类似，区别在于返回的是NodeList对象，包含所有匹配给定选择器的节点


    var el1 = document.querySelector('.myclass');
    var el2 = document.querySelector('#myParent > [ng-click]');
    
    // 选中 data-foo-bar 属性等于 someval 的元素
    document.querySelectorAll('[data-foo-bar="someval"]');
    
    // 选中 myForm 表单中所有不通过验证的元素
    document.querySelectorAll('#myForm :invalid');
    
    // 选中div元素，那些 class 含 ignore 的除外
    document.querySelectorAll('DIV:not(.ignore)');
    
    // 同时选中 div，a，script 三类元素
    document.querySelectorAll('DIV, A, SCRIPT');

+ document.getElementsByTagName()
    + 搜索HTML标签名，返回符合条件的元素
    + 返回的为一个HTMLCollection对象
    + 可以实时反映HTML文档的变化
+ document.getElementsByClassName()
    + 返回一个类似数组的对象，包括了所有class名字符合指定条件的元素，元素的变化实时反映在返回结果当中
+ document.getElementsByName()
    + 用于选择拥有name属性的HTML元素，返回一个类似数组的对象
+ document.getElementById()
    + 返回匹配指定id属性的元素节点
    + 如果没有发现匹配的节点，则返回null
+ document.elementFromPoint()  document.elementsFromPoint()
    + 返回位于页面指定位置最上层的元素节点 
    + `var element = document.elementFromPoint(50, 50);`
    + 该代码就会选中在这个坐标上的最上层的HTML元素
    + 两个参数是相对于左上角的横坐标和纵坐标，单位是像素
+ document.createElement()
    + 用来生成元素节点，并返回该节点
    + 方法的参数为元素的标签名，即元素节点的tagName属性
+ document.createTextNode()
    + 用来生成文本节点，并返回该节点
    + 参数是文本节点的内容


    var newDiv = document.createElement('div');
    var newContent = document.createTextNode('Hello');
    newDiv.appendChild(newContent);

+ document.createAttribute() 
    + 生成一个新的属性节点


    var node = document.getElementById('div1');
    
    var a = document.createAttribute('my_attrib');
    a.value = 'newVal';
    
    node.setAttributeNode(a);
    // 或者
    node.setAttribute('my_attrib', 'newVal');

+ document.createComment() 
+ document.createDocumentFragment()
    + 生成一个空的文档片段对象 
    + 是存于内存的DOM片段，不属于当前文档，常常用来生成一段较为复杂的DOM结构，然后再插入到文档当中
    + 因为DocumentFragment不属于当前文档，对其任何改动都不会引发网页的重新渲染，比直接修改当前文档的DOM有更好的性能表现


+ document.createEvent() 
    + 生成一个事件对象
    + 其参数为事件类型，比如
        + MouseEvents
        + MutationEvents
        + HTMLEvents 


    var event = document.createEvent('Event');
    event.initEvent('build', true, true);
    document.addEventListener('build', function (e) {
      console.log(e.type); // "build"
    }, false);
    document.dispatchEvent(event);

+ document.addEventListener() 
    + 添加事件监听函数 
+ document.removeEventListner()
    + 移除事件监听函数 
+ document.dispatchEvent()
    + 触发事件
+ document.hasFocus()
    + 返回一个布尔值，表示当前文档是否有元素被激活或者获得了焦点
+ document.adoptNode()
    + document.adoptNode方法将某个节点及其子节点，从原来所在的文档或DocumentFragment里面移除，归属当前document对象，返回插入后的新节点。插入的节点对象的ownerDocument属性，会变成当前的document对象，而parentNode属性是null


    var node = document.adoptNode(externalNode);
    document.appendChild(node);


## 3.4 Element 节点
Element结点对应网页的HTML元素，每个HTML原色，在DOM树上都会转化成一个ELement节点对象

### 3.4.1 实例属性

+ Element.id
+ Element.tagName 
    + 返回指定元素的大写标签名
+ Element.dir
    + 用于读写当前元素的文字方向
+ Element.accessKey
    +  用于读写分配给当前元素的快捷键


    // HTML 代码为 <p id="foo">
    var p = document.querySelector('p');
    p.id // "foo"
    
    // HTML代码为
    // <span id="myspan">Hello</span>
    var span = document.getElementById('myspan');
    span.id // "myspan"
    span.tagName // "SPAN"
    
    // HTML 代码如下
    // <button accesskey="h" id="btn">点击</button>
    var btn = document.getElementById('btn');
    btn.accessKey // "h"


+ Element.draggable
    + 返回一个布尔值，表示当前元素是否可拖动，该属性可读写
+ Element.lang
    + 返回当前元素的语言设置 
+ Element.tabIndex
    + 返回一个整数值，表示当前元素在tab键遍历的时候的顺序
    + 如果为负值，在tab不会遍历到这个原色
    + 对于正整数，按照顺序从小到大进行遍历
+ Element.title
    + 用于读写当前元素HTML属性的title 
    + 用于指定鼠标悬浮时弹出的文字提示框

+ Element.hidden
    + 控制当前元素是否可见
    + CSS的设置高于hidden  即如果css层规定了`display:none`或者`display:hidden` 那么element.hidden是无法改变其可见性的

+ Element.contentEditable  Element.isContentEditable 
+ Element.attributes 
    + 返回一个类似数组的对象，成员是当前元素节点的所有属性节点
+ Element.className Element.classList
    + 用来读写当前原色节点的class属性
    + 其值为一个字符串，每个class之间用空格来分割
    + st
    + 返回一个类似数组的对象


    // HTML 代码 <div class="one two three" id="myDiv"></div>
    var div = document.getElementById('myDiv');
    
    div.className
    // "one two three"
    
    div.classList
    // {
    //   0: "one"
    //   1: "two"
    //   2: "three"
    //   length: 3
    // }

+ Element.innerHTML 
    + 返回一个字符串，等同于该元素包含的所有HTML代码
    + 可以读写
    + 用于设定某个节点的内容
    + 能够改变所有元素结点的内容，包括HTML body这类元素


    // HTML代码如下 <p id="para"> 5 > 3 </p>
    document.getElementById('para').innerHTML
    // 5 &gt; 3

+ Element.outerHTML 
    + 返回一个字符串，表示当前元素结点的所有HTML代码，包括该元素本身和所有子元素
+ Element.clientHeight Element.clientWidth
    + 返回一个整数值表示元素节点的CSS高度和宽度


### 3.4.2 实例方法

+ getAttribute()
+ getAttributeNames() 
+ setAttribute()
+ hasAttribute()
+ hasAttributes()
+ removeAttribute() 

### 3.4.3 事件相关方法

+ Element.addEventListener()
+ Element.removeEventListener()
+ Element.dispatchEvent()