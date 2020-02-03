---
title: JavaScript教程 Part 1
date: 2020-01-30 21:12:39
categories: FrontEnd
tags:
    - JavaScript
    - FrontEnd
top:
---
发现前面看的JS还是不太成系统，遇到的这方面的问题越来越多，需要好好的深入研究下这块内容了。

# 1. Basic

## 1.1 概览

+ 嵌入式语言
+ 本身的核心语法只有数学逻辑运算相关的
+ 靠宿主环境提供各种输入输出相关的API
    + 浏览器
    + 服务器环境 - nodejs
+ JS的核心语法部分
    + 基本语法构造
        + 操作符
        + 控制结构
        + 语句
    + 标准库
        + Array
        + Date 
        + Math
        + ...
+ 除此以外，便是其宿主环境提供的各种不同的API了
+ 浏览器
    + 浏览器控制类 
    + DOM类  html
    + Web类  ajax call

本文会从基本语法，标准库，浏览器API以及DOM四个大方面来解释整个js的运行。大部分内容出自阮一峰的电子书，其中加了一下自己认为不错的例子。

+ JS性能
    + JS基本上都是编译运行，运行效率很高
    + 采取事件驱动 event-driven 
    + 非阻塞式 non-blocking 

## 1.2 基本语法

+ 变量提升
    + JS引擎会先解析代码，获取所有被声明的变量，然后再一行一行的运行。
    + 即所有的变量的声明语句都会被提升到代码的头部。
+ 使用label
    + 定位符，用于跳转到程序的任意位置
    + 比较常用的场景：可以跳出双重循环 - 直接


    outOfTwoLoops:
        for (let i = 0; i < 10; i ++) {
            for (let j = 0; j < 10; j ++) {
                if (i*j < 99) break outOfTwoLoops;
            }
        }

## 1.3 数据类型概述

+ 基本数据类型
    + number
    + string
    + boolean
    + undefined
        + nothing defined (value)
        + when switch to number, come to be NaN 
        + 表示未定义
    + null 
        + empty object 
        + when switch to number, come to be 0
        + 表示为空
    + object
        + 狭义的对象
        + array
        + function


JS里面将函数function也作为一种对象来进行处理，好处是可以做函数式的编程了，即可以将整个函数赋给一个变量，这就可以为编程带来非常大的灵活性了。

+ typeof
    + `function f() {}`
    + `typeof f // function`
    + `typeof d // undefined`
    + `typeof [] // object`
    + `typeof {} // object`

+ 数值
    + 所有数字都是以64位浮点数形式储存的，即使整数也是如此
    + 浮点数本身就是不精确的，涉及小数的运算需要非常小心
    + 相关方法


    // 字符串转化为整数
    parseInt('123')  // 123
    parseInt('123CannotProcessString')  // 123
    parseInt('12.343')  // 12
    parseInt('1.99')  // 1
    parseInt('hahah')  // NaN 
    
    // parseInt第二个参数表示被解析的值得进制，返回的值是10进制的
    parseInt('1000', 2)  // 8
    
    // parseFloat same pattern with parseInt
    parseFloat('') // NaN
    parseFloat('123.45') // 123.45
    
    // 判断一个值是否为NaN, 只对数值有效，对于其他类型，首先转化为数值的时候就变成NaN了
    isNaN(123)  // false
    

 
+ NaN
    + not a number
    + 主要出现在将字符串解析成数组出错的场合
    + `typeof NaN    // 'number'`

+ 字符串
    + 默认只能写在一行当中，多行会报错
    + 如果想分在多行，需要在每一行的尾部使用反斜杠 `\`
    + 转义 
        + \0 nukk
        + \b 后退
        + \f 换页
        + \n 换行
        + \r 回车
        + \t 制表
        + \v 垂直制表
        + \' 单引号
        + \" 双引号
        + \\ 反斜杠
    + 字符串与数组
        + 字符串可以视为字符数组，因此可以使用数组的方括号运算符，用来返回某个位置的字符
        + 但是字符串里面的单个字符无法改变和增删的，无法通过数组的形式来做这个
        + `str.length`返回字符串的长度


    var s = 'hello';
    s[0] // "h"
    s[1] // "e"
    s[4] // "o"
    
    // 直接对字符串使用方括号运算符
    'hello'[1] // "e"

+ 字符集 
    + JS使用Unicode字符集，JS引擎内部所有字符都是使用Unicode来进行表示的。 
    + 因此可以直接将字符写成 `\uxxxx`的形式，后四位是该字符的Unicode码点
+ Base64转码
    + 做转码是因为文本里面有时会有一些不可打印的符号，不是为了加密，就是为了简化程序的处理过程
    + Base64可以将其转化成可以打印的字符
    + 有时也需要以文本的格式传递二进制数据，也可以使用Base64编码
    + btoa() 任意值转化为base64编码  binary to ASCII
    + atob() Base64编码转为原来的值  ASCII to binary 
    + 对于ASCII无法表示的字符，我们需要加一个encodeURIComponent的指令，来编码Uniform Resource Identifier（URI）


+ 对象
    + 对象是键值对的集合，是一种无序的复合数据集合
    + 对象的所有键名都是字符串，所以可以选择不加引号
    + 对象的每一个键名称为一个属性 property 
        + 其键值可以是任何数据类型，如果一个属性的值为函数，那么就可以将这个属性称为方法，可以像函数那样来做调用 
    + 属性是可以**动态创建**的，不必在对象声明的时候就指定
    + 查看对象本身的属性 `Object.keys`
    + 删除属性  `delete obj.p`     
    + 属性是否存在 - 检查键名  `'p' in obj`


    var obj = {
        foo: 'hello',
        bar: 'world'
    };

+ 表达式 or  语句
    + `{foo: 123}`
    + 这样子可以理解为一个表达式，也可以理解为一个语句，foo是个标签，指向123.
    + JS引擎的做法是一律解释为代码块
    + 因此如果想要表示成一个表达式，那么我们就应该在想要表达的东西的外部加上一对小括号以撇清关系
    + 语句 用分号进行分隔 做某种行为
    + 表达式 由运算符构成，并运算产生结果的语法结构，用逗号进行分割 是会生成一个值得
    + 当JS期待得到一个语句的时候，你可以使用一个表达式语句来做；但是当JS期待一个表达式的时候，你是无法带入一个语句的，比如if 语句就无法作为一个方法的参数

+ 函数
    + 可以反复调用的代码块
    + 可以接受输入的参数，
    + 三种声明函数的方式：
        + function 
        + 函数表达式
            + 函数表达式的等号右侧是可以有函数名的，但是这个函数名只在这个函数体内有效，其他是无效的 
        + function构造函数
    + 函数重复声明，后声明的会覆盖前面声明的
    + 调用函数，使用圆括号
    + 在JS中，凡是可以使用值的地方，就可以使用函数
    + 因为将函数名视为变量名，所有采用function命令声明函数时，整个函数会像变量声明一样，提升到代码的头部。
    + 函数的属性与方法
        + name
            + 根据定义的方法的不同，返回不同值
                + 函数定义  返回函数名
                + 函数表达式定义  返回变量名
        + length
            + 返回函数定义之中的参数个数
            + length属性提供了一种机制，来判断定义的时候和调用参数的差异，以便实现面向对象编程的方法重载
        + toString()
            + 返回一个字符串，内容是函数的源码 
    + 函数作用域
        + global variable
        + local variable
        + 函数执行时所在的作用域，是定义时的作用域，而不是调用的时候所在的作用域
    + 传递方式
        + 对于原始类型来说，是值传递的 (pass by value)，也就是说，在函数体内修改参数值，不会影响到函数的外部。
        + 如果函数参数是复合类型的值(数组、对象、函数),那么传递方式就是传址传递 (pass by reference)。也就是说，传入函数的原始值的地址，因此在函数内部修改参数，将会影响到原始值。
    + arguments对象
        + 因为JS当中允许函数有不同数目的参数，因此需要有一种机制能够在函数体内部读取所有的参数，因此定义了arguments对象来做这件事
        + arguments对象包含了函数运行时的所有参数
        + 而且注意arguments对象与函数参数并不具有联动关系。
        + arguments.length 返回函数实际传入的参数的数量
    + 闭包
        + JS的链式作用域，子对象会逐级向上寻找所有父对象的变量，因此父对象的所有变量对子对象而言都是可见的，反之不成立
        + 闭包就是函数f2，为的是能读取函数内部的变量，并将其传递出去。
        + 是将函数内部和函数外部连接起来的一座桥梁


    function f1() {
      var n = 999;
      function f2() {
        console.log(n);
      }
      return f2;
    }
    
    var result = f1();
    result(); // 999





    // function
    function sayHello(var name) {
        console.log("Hello" + name);
    }
    
    // function expression 
    var sayHello = function(var name) {
        console.log("Hello" + name);
    };

    // function constructor 
    var add = new Function(
        'x',
        'y',
        'return x + y'
    );
    
    // arguments 
    var f = function(a,b) {
        arguments[0] = 2;
        arguments[1] = 10;
        return a + b;
    }

    // f(1, 2) ----> 12 

+ 立即调用的函数表达式(IIFE) - immediately invoked function expression
    + 在JS当中，圆括号本身就是运算符，跟在函数名之后，表示调用了这个函数
    + 有时，我们需要在定义函数之后，立即调用该函数。这时，你不能在函数的定义之后加上圆括号，这会产生语法错误。
    + `function(){ /* code */ }(); // SyntaxError: Unexpected token `
    + 产生这个错误的原因是因为function这个关键字既可以作为语句也可以作为表达式，JS引擎为了避免歧义，就规定只要function关键字出现在行首，就一律解释为语句。
    + `(function(){}());` 通过这种方式来进行直接的运行
+ eval 指令
    + 用于在当前作用域当中，注入代码
    + 但是因为安全风险和不利于JS引擎去优化执行速度，所以一般不推荐使用
    + 最常见的场合是解析JSON数据的字符串，但一般用JSON.parse会更加合适
+ 数组
    + 任何类型的数据都可以放入数组
    + 数组本身是一种特殊的对象
    + length 属性  = 键名中的最大整数加上1
    + 数组的数字键不需要连续，length属性的值总是比最大的整数键大1
    + 即数组是一个动态的数据结构，可以随时增减数组的成员
    + 清空数组可以通过将数组的长度设为0来实现的


    var arr = ['a', 'b'];
    arr.length // 2
    
    arr[2] = 'c';
    arr.length // 3
    
    arr[9] = 'd';
    arr.length // 10
    
    arr[1000] = 'e';
    arr.length // 1001

## 1.4 运算符

+ 算数运算符
    + 对象的相加
        + 对象转成原始类型的值的规则为：
        + 首先自动调用对象的valueOf方法，一般会返回对象本身
        + 再自动调用对象的toString方法，将其转为字符串


    var obj = {p: 1}
    obj + 2 // [object Object]2  ----> obj.valueOf().toString()
    
    //可以自定义valueOf方法的
    
    var obj = {
        valueOf: function() {
            return 1;
        }
    };
    
    obj + 2 // 3
    
+ 比较运算符
+ 布尔运算符
    + !
    + && 
    + || 
    + ?:
+ 二进制位运算符
+ 其他运算符
    + void
        + 作用是执行一个表达式，但是不返回任何值(undefined)
        + 主要目的是为了在超级链接当中可以插入代码，但是又不产生跳转
    + 逗号运算符
        + 用于对两个表达式求值，并且返回后一个表达式的值 

    
    // 下述代码因为onclick return false所以不会产生跳转的
    <script>
    function f() {
      console.log('Hello World');
    }
    </script>
    <a href="http://example.com" onclick="f(); return false;">点击</a>
    
    // 可以直接用void function来取代 return false的操作  显得清晰简洁很多的
    <a href="javascript: void(f())">文字</a>

# 2. 常用语法
## 2.1 数据类型的转换
变量没有类型限制，e.g: `var x = y ? 1 : 'a'`

在上述的例子当中，我们在编译的时候是不知道到底x是个数值还是个字符的，必须等到运行的时候才可以有数据。而JS当进行运算的时候，如果发现实际的类型和期待的不相符，会直接将其转化为期待的类型的。

+ 强制转换
    + `Number()`
        + 可以将任意类型的值转化为数值
        + Number函数将字符串转为数值，要比`parseInt`严格非常多，只要有一个字符无法转成数值，整个字符串就只能被转化为NaN了
        + Number进行转化的规则为：
            + 调用对象自身的valueOf；如果为原始类型，直接对该值使用Number函数
            + 如果valueOf返回的还是对象，那么改成调用对象的toString方法，如果toString方法返回原始类型的值，就对该值使用Number函数，不再进行后续操作
            + 如果toString方法返回的是对象，就报错
    + `String()`
        + 原始类型值
            +  undefined -> "undefined"
            +  null -> "null"
        + 对象
            + 对象返回类型字符串   `String({a: 1}) // "[object Object]"`
            + 数组返回其字符串形式 `String([1, 2, 3]) // "1,2,3"`
    + `Boolean()`
        + 只有以下五个值的转换结果为false,其余全部为true
            + undefined 
            + null
            + 0
            + NaN
            + ''

## 2.2 错误处理机制

### 2.2.1 Error实例对象

JS解析或运行时发生错误，引擎就会抛出一个错误对象。JS原生提供Error构造函数，所有抛出的错误都是这个构造函数的实例。

    var error = new Error('Some error occur');
    error.message // "Some error occur"

Error构造函数接受一个参数，表示错误提示，可以从实例的message属性读到这个参数。抛出Error实例对象以后，整个程序就中断在发生错误的地方，不再往下执行。

JS当中只有Error的message属性是必带的，但是一般来说，还会携带有`name`以及`stack`的信息，分别表示错误的名称和错误的堆栈

### 2.2.2 原生错误类型

下述错误类型都是构造函数，可以直接使用它来手动生成错误对象。这些构造函数都接收一个参数，代表错误提示信息

+ SyntaxError 
    + 解析代码时发生的语法错误
+ ReferenceError
    + 引用一个不存在的变量
    + 或者将一个值分配给无法分配的对象
+ RangeError
    + 值超出有效范围时发生的错误
        + 数组长度为负
        + 对象方法参数超出范围
        + 函数堆栈超过最大值
+ TypeError 
    + 变量或者参数不是预期类型时发生的错误 
+ URIError
    + 是URI相关的函数的参数不正确时抛出的错误
        + encodeURI()
        + decodeURI()
        + encodeURIComponent()
        + decodeURIComponent()
        + escape()
        + unescape() 
+ EvalError
    + eval函数没有被正确执行 

### 2.2.3 自定义错误

    function UserError(msg) {
        this.message = msg;
        this.name = "UserError";
    }
    
    // 继承了Error对象，声明其构造器
    UserError.prototype = new Error();
    UserError.prototype.constructor = UserError;
    
而后我们也可以使用`throw` 抛出任何我们想抛出的值或者对象，程序运行到throw这里会终止。

### 2.2.4 try - catch - finally 

这里注意三者之间的执行顺序 

先try，捕获异常，而后finally代码块是不管是否出现错误，都必须最后运行的语句

    function cleansUp() {
      try {
        throw new Error('出错了……');
        console.log('此行不会执行');
      } finally {
        console.log('完成清理工作');
      }
    }
    
    cleansUp()
    // 完成清理工作
    // Uncaught Error: 出错了……
    //    at cleansUp (<anonymous>:3:11)
    //    at <anonymous>:10:1

上述例子当中，中断执行了，会先执行finally代码块，再向用户提示报错信息。但是实际上try是已经执行完了的，finally代码块的运行不能改变try里面的输出或者function 


    var count = 0;
    function countUp() {
      try {
        return count;
      } finally {
        count++;
      }
    }
    
    countUp()
    // 0
    count
    // 1

上述例子证明了try先执行完，才轮到finally，但是是finally先进行输出的。

finally的经典应用场景一般设计文件描述符的关闭：

    openFile();
    
    try {
      writeFile(Data);
    } catch(e) {
      handleError(e);
    } finally {
      closeFile();
    }

    // try catch finally 执行顺序的反应
    function f() {
      try {
        console.log(0);
        throw 'bug';
      } catch(e) {
        console.log(1);
        return true; // 这句原本会延迟到 finally 代码块结束再执行
        console.log(2); // 不会运行
      } finally {
        console.log(3);
        return false; // 这句会覆盖掉前面那句 return
        console.log(4); // 不会运行
      }
    
      console.log(5); // 不会运行
    }
    
    var result = f();
    // 0
    // 1
    // 3
    
    result
    // false

## 2.3 编程风格

对于区块，大括号应该跟在这一行里面，不要另外起一行，因为JavaScript会自动添加句末的分号，导致一些难以察觉的错误。

+ 分号的使用
    + 不使用分号的情况
        + for/ while
        + if/ switch/ try
        + 函数的声明语句
    + 分号的自动添加
        + js会自动添加，但是还会看下一行能否连接起来成一条语句，或者表达式，如果可以的话，会直接来做连接的 
        + 一行的起首为自增或者自减运算符，那么其前面会自动添加分号
        + continue, break, return, throw这几个语句后面如果直接跟了换行符，都是会直接添加分号的
+ switch - case 
    + 可以将其重写成对象的格式，[medium上的例子](https://medium.com/chrisburgin/rewriting-javascript-replacing-the-switch-statement-cfff707cf045)
    + switch case 不利于格式统一，有点太冗长，而且很容易忘记break，用对象是个很好地选择实际上


    function doAction(action) {
      switch (action) {
        case 'hack':
          return 'hack';
        case 'slash':
          return 'slash';
        case 'run':
          return 'run';
        default:
          throw new Error('Invalid action.');
      }
    }
    
    function doAction(action) {
      var actions = {
        'hack': function () {
          return 'hack';
        },
        'slash': function () {
          return 'slash';
        },
        'run': function () {
          return 'run';
        }
      };
    
      if (typeof actions[action] !== 'function') {
        throw new Error('Invalid action.');
      }
    
      return actions[action]();
    }

## 2.4 console对象与控制台

+ console
    + JS的原生对象，可以输出各种信息到控制台，并提供了很多有用的辅助方法
    + 用于调试程序，提供网页代码运行时的错误信息
    + 提供一个命令行接口，用来和网页代码互动
    + console.log()
        + 占位符
            + %s
            + %d
            + %i 整数
            + %f
            + %o 对象的链接
            + %c css格式的字符串
                + 对应的参数必须是CSS代码，用来对输出内容进行CSS的渲染
    + console.table()
        + 输出复合类型的数据，以表格形式进行显示
    + console.count()
        + 记录被调用了多少次
    + console.dir()
        + 对一个对象进行检查，以便于阅读和打印的格式显示出来
    + console.dirxml()
        + 用于以目录树的形式，显示一个DOM节点
    + console.time()  console.timeEnd()
        + 用来看一个操作所花费的准确时间、
    + console.trace()
        + 显示当前执行的代码在堆栈中的调用路径 
# 3. 标准库

## 3.1 Object对象

    Object.prototype.print = function () {
      console.log(this);
    };
    
    var obj = new Object();
    obj.print() // Object

凡是定义在Object.prototype对象上面的属性和方法，将被所有实例对象共享。`Object.prototype`叫做原型对象

### 3.1.1 Object静态方法

Object静态方法指的是部署在Object对象自身的方法。

+ Object.keys
    + 方法参数为一个对象
    + 返回一个数组，该数组的成员都是该对象自身的(非继承的)所有属性名
+ Object.getOwnPropertyNames
    + 与Object.keys一样的用法
    + 不同之处在于这个也会返回不可枚举的属性名
+ 如何计算对象的属性个数？
    + Object.keys(obj).length 
+ 对象属性模型的相关方法
    + Object.getOwnPropertyDescriptor() 
        + 获取某个属性的描述对象
    + Object.defineProperty()
        + 通过描述对象，定义某个属性 
    + Object.defineProperties()
+ 控制对象状态的方法
    + Object.preventExtensions()
        + 防止对象扩展
        + 使得一个对象无法再添加新的属性
    + Object.isExtensible()
        + 判断对象是否扩展
    + Object.seal()
        + 禁止对象的配置
        + 无法添加也无法删除属性
    + Object.isSealed()
    + Object.freeze()
        + 冻结一个对象
        + 无法添加
        + 无法删除
        + 无法修改
    + Object.isFrozen()
        + 判断一个对象是否冻结
+ 原型链相关方法
    + Object.create()
        + 指定原型对象和属性，返回一个新的对象
    + Object.getPrototypeOf()
        + 获取对象的prototype对象 



    var obj = {
      p1: 123,
      p2: 456
    };
    
    Object.keys(obj) // ["p1", "p2"]
    
    var a = ['Hello', 'World'];

    Object.keys(a) // ["0", "1"]
    Object.getOwnPropertyNames(a) // ["0", "1", "length"]

### 3.1.2 Object的实例方法

指的是定义在Object.prototype对象的方法，称为实例方法。所有的Object对象都继承了这几个方法。

+ `Object.prototype.valueOf()`：返回当前对象对应的值。


    // Object.prototype.valueOf()  返回对象本身
    var obj = new Object();
    obj.valueOf() === obj; // true
    
+ `Object.prototype.toString()`：返回当前对象对应的字符串形式。
    + toString本身是返回一个对象的字符串形式，默认情况下返回类型字符串
    + 当对一个对象调用这个方法的时候，会返回字符串`[object Object]`
    + 可以自定义方法来返回更多的有用信息
    + 数组，字符串，函数，Date对象都有自己的自定义的tostring方法的
    + 我们可以用过`Object.prototype.toString.call(value)`来判断出一个值到底是什么类型的

    
    var o1 = new Object();
    o1.toString() // "[object Object]"
    
    var o2 = {a:1};
    o2.toString() // "[object Object]"
    
    var obj = new Object();
    obj.toString = function () {
        return "hello, I rewrite the toString() function";
    }
    
    Object.prototype.toString.call(2) // "[object Number]"
    Object.prototype.toString.call('') // "[object String]"
    Object.prototype.toString.call(true) // "[object Boolean]"
    Object.prototype.toString.call(undefined) // "[object Undefined]"
    Object.prototype.toString.call(null) // "[object Null]"
    Object.prototype.toString.call(Math) // "[object Math]"
    Object.prototype.toString.call({}) // "[object Object]"
    Object.prototype.toString.call([]) // "[object Array]"



+ `Object.prototype.toLocaleString()`：返回当前对象对应的本地字符串形式。
    + 为的就是留出一个接口，让各种不同的对象实现自己的版本的toLocaleString,用来返回针对某些特定地域的特定的值 
+ `Object.prototype.hasOwnProperty()`：判断某个属性是否为当前对象自身的属性，还是继承自原型对象的属性。
    + 接受一个字符串作为参数，返回一个布尔值，表示该实例对象自身是否有该属性 
+ `Object.prototype.isPrototypeOf()`：判断当前对象是否为另一个对象的原型。
+ `Object.prototype.propertyIsEnumerable()`：判断某个属性是否可枚举。 


### 3.1.3 Attributes Object

属性描述对象，控制其行为，比如该属性是否可写。可遍历等等。

属性描述对象提供了6个元属性
+ value 
+ writable 
+ enumerable 布尔值，表示该属性是否可遍历
+ configurable 可配置性 
    + 如果设置为false, 将组织某些操作改写该属性，比如无法删除，也不得改变该属性的属性描述对象
+ get
    + 取值函数
+ set
    + 存值函数


属性可以通过存取器来进行定义  

存值函数称为setter，取值函数称为getter 

    var obj = Object.defineProperty({}, 'p', {
      get: function () {
        return 'getter';
      },
      set: function (value) {
        console.log('setter: ' + value);
      }
    });
    
    obj.p // "getter"
    obj.p = 123 // "setter: 123"
        
# 3.2 Array对象

### 3.2.1 构造函数与静态方法

Array是JS的原生对象，也是一个构造函数，可以用其直接生成新的数组

    var arr = new Array(2);
    arr.length // 2

+ Array.isArray()
    + 判断一个obj是否为数组
    + 这样做的原因是为了弥补`typeof`的缺陷，typeof只能返回是否一个object的这种信息

### 3.2.2  实例方法

+ valueOf()
    + 返回数组本身 
+ toString() 
    + 返回数组的字符串形式
+ push()
    + 在数组的末端加一个或者多个元素，并返回添加新元素以后的数组长度 
+ pop()
    + pop方法用于删除数组的最后一个元素，并返回该元素。注意，该方法会改变原数组。
+ shift()
    + 用于删除数组的第一个元素，并且返回该元素
    + 可以用来遍历并且清空一个数组


    var list = [1,2,3,4];
    var item;
    
    while (item = list.shift()) {
        console.log(item);
    }

+ unshift()
    + 用于在数组的第一个位置添加元素，并且返回新元素添加以后的数组长度
+ join()
    + 以指定参数作为分隔符，将所有数组成员连接为一个字符串并返回
    + 如果数组成员是undefined/ null/ blank, 会被转成空字符串
    + join默认是用逗号来连接的


    var a = [1, 2, 3, 4];
    
    a.join(' ') // '1 2 3 4'
    a.join(' | ') // "1 | 2 | 3 | 4"
    a.join() // "1,2,3,4"

+ concat()
    + 用于多个数组的合并
    + 将新数组的成员，添加到原数组成员的后面，然后返回一个新数组，原数组不变
+ reverse()
+ slice()
    + 用于提取目标数组的一部分，返回一个新数组，原数组并不发生改变
    + 第一个参数为起始位置，第二个参数为终止位置，但是这个位置并不包含在内
    + 如果第二个参数省略，那就一直返回到原数组的最后一个成员
    + slice的一个重要应用时将类似数组的对象转为真正的数组


    Array.prototype.slice.call({ 0: 'a', 1: 'b', length: 2 })
    // ['a', 'b']
    
    Array.prototype.slice.call(document.querySelectorAll("div"));
    Array.prototype.slice.call(arguments);

+ splice()
    + 删除原数组的一部分成员，并可以在删除的位置添加新的数组成员，返回值是被删除的元素
    + `arr.splice(start, count, addElement1, addElement2, ...);`
+ sort()
+ map()
    + 将数组的所有成员依次传入参数函数，然后将每一次的执行结果组成一个新数组返回


    var numbers = [1, 2, 3];
    
    numbers.map(function (n) {
      return n + 1;
    });
    // [2, 3, 4]
    
    numbers
    // [1, 2, 3]
+ forEach()
    + 与map方法类似，对数组的所有成员依次执行参数函数。但是forEach不返回值，只用来操作数据 
    + forEach接收第二个参数，绑定参数函数的this变量
    + forEacg方法无法中断执行，总是会把所有成员遍历完，如果希望符合某种条件就中断遍历，那需要使用for循环


    var out = [];
    
    [1, 2, 3].forEach(function(elem) {
      this.push(elem * elem);
    }, out);
    
    out // [1, 4, 9]

+ filter()
    + 用于过滤数组成员，满足条件的成员组成一个新数组返回 
    + 参数是一个函数，返回结果为true的成员组成一个新数组返回


    // 当前成员，当前位置，整个数组
    [1, 2, 3, 4, 5].filter(function (elem, index, arr) {
      return index % 2 === 0;
    });
    // [1, 3, 5]
    
    var obj = { MAX: 3 };
    var myFilter = function (item) {
      if (item > this.MAX) return true;
    };
    
    var arr = [2, 8, 3, 4, 1, 3, 2, 9];
    arr.filter(myFilter, obj) // [8, 4, 9]
+ some()
    + 有满足条件的就返回true 
+ every()
    + 全都满足条件才返回true
+ reduce(), reduceRight()
    +  依次处理数组的每一个成员，最终累计为一个值
    +  reduce从左到右进行处理
    +  reduceRight从右向左进行处理
    +  四个参数
        + 累积变量
        + 当前变量
        + 当前位置
        + 原数组


    function findLongest(entries) {
      return entries.reduce(function (longest, entry) {
        return entry.length > longest.length ? entry : longest;
      }, '');
    }
    
    findLongest(['aaa', 'bb', 'c']) // "aaa"

+ indexOf()
    + 返回给定元素在数组中第一次出现的位置，如果没有出现就返回-1 
+ lastIndexOf()
    + 返回给定元素在数组中最后一次出现的位置，如果没有出现就返回-1 

## 3.3 包装对象

三种原始类型的值 - 数值，字符串，布尔值在一定条件下也会自动转为对象，称为原始类型的包装对象 - wrapper

    var v1 = new Number(123);
    var v2 = new String('abc');
    var v3 = new Boolean(true);
    
    typeof v1 // "object"
    typeof v2 // "object"
    typeof v3 // "object"
    
    v1 === 123 // false
    v2 === 'abc' // false
    v3 === true // false

+ valueOf()
    + 返回包装对象实例对应的原始类型的值
+ toString()
    + 返回对应的字符串形式
+ 原始类型与实例对象的自动转换
    + 一些时候我们将原始类型的值自动当做包装对象来调用了，这时JS引擎会自动将原始类型的值转为包装对象实例，并在使用后立刻销毁实例。
    + `'abc'.length // 3`
    + abc是字符串，但是js直接将其转化为了其包装对象，并在调用结束后，直接销毁这个临时对象，以此完成整个自动转换的过程

### 3.3.1 Boolean对象

    Boolean(undefined) // false
    Boolean(null) // false
    Boolean(0) // false
    Boolean('') // false
    Boolean(NaN) // false
    
    Boolean(1) // true
    Boolean('false') // true
    Boolean([]) // true
    Boolean({}) // true
    Boolean(function () {}) // true
    Boolean(/foo/) // true

### 3.3.2 Number对象

+ 静态属性
    + Number.POSITIVE_INFINITY
    + Number.NEGATIVE_INFINITY
    + Number.NaN
    + Number.MIN_VALUE
    + Number.MAX_SAFE_INTEGER
    + Number.MIN_SAFE_INTEGER
+ 实例方法
    + Number.prototype.toString()
    + Number.prototype.toFixed()
        + 将一个数转为指定位数的小数，然后返回这个小数对应的字符串
        + `(10).toFixed(2) // "10.00"`
    + Number.prototype.toExponential()
        + `(10).toExponential()  // "1e+1"`
        + 参数是小数点后有效数字的位数
    + Number.prototype.toPrecision()
        + `(12.34).toPrecision(2) // "12"`
        + 将数字转为指定位数的有效数字

### 3.3.3 String对象

用来生成字符串对象，会生成一个非常类似数组的对象

    new String('abc')
    // String {0: "a", 1: "b", 2: "c", length: 3}

+ 静态方法
    + String.fromCharCode()
        + 数值代表Unicode码点，返回值是这些码点组成的字符串
+ 实例方法
    + String.prototype.charAt()
    + String.prototype.charCodeAt()
        + 返回的是字符串指定位置的Unicode码点
    + String.prototype.concat()
        + 用来连接两个字符串，返回一个新的字符串，并不改变原来的字符串
        + `'a'.concat('b', 'c') // "abc"`
    + String.prototype.slice()
        + 用于从原字符串去除子字符串并返回，不改变原字符串。它的第一个参数是子字符串的开始位置，第二个参数是子字符串的结束位置（不含该位置）。 
    + String.prototype.substr()
        + 从原字符串取出子字符串并返回，不改变原字符串
        + 第一个参数为子字符串的开始位置，第二个参数为子字符串的长度
    + String.prototype.indexOf()，String.prototype.lastIndexOf()
    + String.prototype.trim()
        + 去除字符串两端的空格，返回一个新的字符串，不改变原字符串
        + 还包括 \t \v \n \r
    + String.prototype.toLowerCase()，String.prototype.toUpperCase()
    + String.prototype.match()
    + String.prototype.search()
        + 返回值为匹配的第一个位置，如果没有找到匹配则返回 -1
    + String.prototype.replace()
    + String.prototype.split()
        + 按照给定规则分割字符串，返回一个由分割出来的子字符串组成的数组


    'a|b|c'.split('|') // ["a", "b", "c"]
    
    'a||c'.split('|') // ['a', '', 'c']
    
    'a|b|c'.split('|', 0) // []
    'a|b|c'.split('|', 1) // ["a"]
    'a|b|c'.split('|', 2) // ["a", "b"]
    'a|b|c'.split('|', 3) // ["a", "b", "c"]
    'a|b|c'.split('|', 4) // ["a", "b", "c"]

## 3.4 Math对象

JS的原生对象，不是构造函数，无法生成对象，所有的属性和方法都需要在Math对象上进行调用

+ 静态属性
    + Math.E
    + Math.LN2
    + Math.LN10
    + Math.LOG2E
    + Math.PI
+ 静态方法
    + Math.abs()
    + Math.ceil()
        + 向上取整 
    + Math.floor()
        + 向下取整
    + Math.max()
    + Math.min()
    + Math.pow() 指数运算
    + Math.sqrt()
    + Math.log()
    + Math.exp() e的指数
    + Math.round() 四舍五入
    + Math.random() 随机数
    
## 3.5 Date对象
以UTC时间1970年1月1日00:00:00作为时间元点。

+ 直接调用Date() 返回当前时间
+ 构造函数
    + 如果带参数，就转化那个时间
    + 不带，就是当前时间
+ 静态方法
    + Date.now() 
        + 返回当前时间距离时间零点的毫秒数
    + Date.parse()
        + 解析日期字符串，返回该时间距离时间零点的毫秒数
        + 日期字符串需要符合RFC 2822和 ISO8061 两个标准
    + Date.UTC()
        + 接收年月日等变量作为参数，返回该时间距离时间零点的毫秒数 
+ to see doc https://wangdoc.com/javascript/stdlib/date.html
+ get see doc https://wangdoc.com/javascript/stdlib/date.html
+ set see doc https://wangdoc.com/javascript/stdlib/date.html 


## 3.6 RegExp对象
提供正则表达式的功能。表达文本模式的方法，可以使用字面量，以斜杠表示开始和结束，也可以使用RegExp构造函数

    // 引擎编译代码的时候，就新建正则表达式了
    var regex = /xyz/;
    
    // 运行的时候建立
    var regex = new RegExp('xyz');


### 3.6.1 实例属性

+ RegExp.prototype.ignoreCase：返回一个布尔值，表示是否设置了i修饰符。
+ RegExp.prototype.global：返回一个布尔值，表示是否设置了g修饰符。
+ RegExp.prototype.multiline：返回一个布尔值，表示是否设置了m修饰符。
+ RegExp.prototype.flags：返回一个字符串，包含了已经设置的所有修饰符，按字母排序。

### 3.6.2 实例方法

+ RegExp.prototype.test()
    + 返回一个布尔值，表示当前模式是否能匹配参数字符串
    + `/cat/.test('cats and dogs')  // true`
+ RegExp.prototype.exec()
    + 返回匹配结果，如果匹配，就返回一个数组，成员是匹配成功的子字符串，否则返回null 
+ String.prototype.match()
    + 所有匹配的子字符串
+ String.prototype.search()
    + 按照给定的正则表达式进行搜索，返回一个整数，表示匹配开始的位置
+ String.prototype.replace()
    + 按照给定的正则表达式进行替换，返回替换后的字符串
+ String.prototype.split()
    + 按照给定规则进行字符串分割，返回一个数组，包含分割后的各个成员。


### 3.6.3 各种符号表示

+ /g
    + 匹配所有符合正则表达式的值，如果没有，成功一次以后就停止了
+ 小括号 

    
    // 如果带括号，括号本身的也会返回的
    'aaa*a*'.split(/(a*)/)
    // [ '', 'aaa', '*', 'a', '*' ]
 
+ 字面量字符
    + 就是一对一的匹配
    + 比如 /dog/ 只匹配包含dog的
+ 元字符
    + 点字符  匹配一个
        + 匹配出了回车\r, 换行\n, 行分隔符 \u2028, 段分隔符\u2029之外的所有字符
    + 位置字符
        + ^
            + 表示字符串开始的位置 
        + $
            + 表示字符串结束的位置
    + 选择符 | 
        + 表示or的关系 
        + e.g `cat|dog`表示cat or dog都匹配
+ 转义符
    + 对于正则表达式当中本身就具有特殊含义的元字符的处理，当需要匹配自身的时候，在它们的前面加上反斜杠 
+ 字符类 
    + 表示有一系列字符可供选择，只要匹配其中一个就可以了 都放到方括号当中 `[]`
    + 脱字符 `^`
        + 若方括号内第一个字符是[^]，则是求反，即除了中括号之内的字符以外都可以进行匹配
        + 如果方括号内没有其他字符，就表示匹配一切字符了，包括换行符
        + 注意脱字符只有在字符类的第一个位置才有特殊含义，否则就是字面含义
    + 连字符 `-`
        + 对于连续序列的字符，用连字符来进行简写 
+ 预定义模式 - 常见模式的简写方式
    + `\d`
        + 匹配 0-9之间的任意数字，相当于`[0-9]` 
    + `\D`
        + 匹配所有 0-9之外的字符 相当于 `[^0-9]` 
    + `\w`
        + 匹配任意的字母、数字和下划线， 相当于`[A-Za-z0-9]` 
    + `\W`
        + 匹配除了字母数字和下划线以外的内容,  相当于`[^A-Za-z0-9]` 
    + `\s`
        + 匹配空格，包括换行符，制表符以及空格符，相当于`[\t\r\n\v\f]` 
    + `\S`
        + 匹配非空格的字符，相当于 `[^\t\r\n\v\f]` 
    + `\b`
        + 匹配词的边界 
    + `\B`
        + 匹配词的内部 



    // test必须出现在开始位置
    /^test/.test('test123') // true
    
    // test必须出现在结束位置
    /test$/.test('new test') // true
    
    // 从开始位置到结束位置只有test
    /^test$/.test('test') // true
    /^test$/.test('test test') // false
    
    /[abc]/.test('hello world') // false
    /[abc]/.test('apple') // true

    var s = 'Please yes\nmake my day!';
    s.match(/yes.*day/) // null
    s.match(/yes[^]*day/) // [ 'yes\nmake my day']
    
    // \s 的例子
    /\s\w*/.exec('hello world') // [" world"]
    
    // \b 的例子
    /\bworld/.test('hello world') // true
    /\bworld/.test('hello-world') // true
    /\bworld/.test('helloworld') // false
    
    // \B 的例子
    /\Bworld/.test('hello-world') // false
    /\Bworld/.test('helloworld') // true

+ 重复类 - `{}`
    + 模式的精准次数匹配，使用大括号表示
    + `{n}` 
        + 恰好重复n次 
    + `{n,}`
        + 至少重复n次 
    + `{n,m}`
        + 重复不少于n次不多于m次
+ 量词符 - 用来设定某个模式出现的次数
    + `? `
        + 出现0次或者1次
        + {0,1}
    + `*`
        + 出现0次或多次
        + {0,}
    + `+`
        + 出现1次或多次
        + {1,}
+ 贪婪模式
    + 通过在量词符后面加问号，从贪婪模式转化为非贪婪模式



    'abb'.match(/ab*b/) // ["abb"]
    'abb'.match(/ab*?b/) // ["ab"]
    
    'abb'.match(/ab?b/) // ["abb"]
    'abb'.match(/ab??b/) // ["ab"]

+ 修饰符
    + 表示模式的附加规则，放在正则模式的最尾部
    + 修饰符可以单个使用，也可以多个使用
    + g修饰符
        + 默认情况下，如果第一次匹配成功了，那么正则对象就会停止向下匹配了
        + g修饰符代表全局匹配，加上他之后，正则对象将匹配全部符合条件的结果，主要用于搜索替换
    + i修饰符
        + 表示忽略大小写 
    + m修饰符
        + m修饰符表示多行模式（multiline），会修改^和$的行为。默认情况下（即不加m修饰符时），^和$匹配字符串的开始处和结尾处，加上m修饰符以后，^和$还会匹配行首和行尾，即^和$会识别换行符（\n）。 
    



    var regex = /b/g;
    var str = 'abba';
    
    regex.test(str); // true
    regex.test(str); // true
    regex.test(str); // false
    
    /abc/.test('ABC') // false
    /abc/i.test('ABC') // true
    
    /world$/.test('hello world\n') // false
    /world$/m.test('hello world\n') // true

+ 组匹配
    + 括号表示分组匹配，括号中的模式用来匹配分组的内容 + 可以用\n表示第n个括号里面的内容
    + 非捕获组 `(?:x)`
        + 表示不返回该组匹配的内容 
    + 先行断言 `x(?=y)`
        + 表示只有x在y前面才匹配，y不会被计入返回结果
    + 先行否定断言 `x(?!y)`
        + x只有不在y前面才匹配，y不会被计入返回结果 


    var m = 'abcabc'.match(/(.)b(.)/);
    m // ['abc', 'a', 'c']
    
    /y(..)(.)\2\1/.test('yabccab') // true
    
    var m = 'abc'.match(/(?:.)b(.)/);
    m // ["abc", "c"]
    

## 3.7 JSON对象

### 3.7.1 JSON格式

1. 复合类型的值只能是数组或对象
2. 原始类型只有四种：字符串，数值，布尔值和null
3. 字符串必须使用双引号表示，不能使用单引号
4. 对象的键名必须放在双引号里面
5. 数组或对象最后一个成员的后面，不可以加逗号


### 3.7.2 JSON对象

+ JSON.stringify()
    + 将一个值转为JSON字符串
    + 会忽略对象的不可遍历的属性
+ JSON.parse()
    + 将JSON字符串转成对应的值 
    
# Reference
+ https://wangdoc.com/javascript/ 
+ 表达式和语句 https://www.cnblogs.com/ziyunfei/archive/2012/09/16/2687589.html 
+ statement and expressions https://2ality.com/2012/09/expressions-vs-statements.html