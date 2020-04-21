---
title: Typescript学习笔记(二)
date: 2020-04-20 20:25:57
categories: FrontEnd
tags:
    - TypeScript
top:
---
+ 类型别名

    + 用来为一个类型起一个新名字


    type Name = string; 
+ 字符串字面量类型



    type EventNames = 'click' | 'scroll' | 'mousemove';
    function handleEvent(ele: Element, event: EventNames) {
        // do something
    }

    handleEvent(document.getElementById('hello'), 'scroll');  // 没问题
    handleEvent(document.getElementById('world'), 'dbclick'); // 报错，event 不能为 'dbclick'

    // index.ts(7,47): error TS2345: Argument of type '"dbclick"' is not assignable to parameter of type 'EventNames'.


+ 元组  - Tuple
    + Tuple可以用于合并不同类型的对象


    let tom: [string, number] = ['Tom', 25];

+ 枚举
    + 用于取值被限定在一定范围内的场景
    + 使用enum进行定义的
    + 枚举成员会被赋值为从0开始递增的数字，同时也会对枚举值到枚举名进行反向映射
 
 
    enum Days {Sun, Mon, Tue, Wed, Thu, Fri, Sat};

    console.log(Days["Sun"] === 0); // true
    console.log(Days["Mon"] === 1); // true
    console.log(Days["Tue"] === 2); // true
    console.log(Days["Sat"] === 6); // true

    console.log(Days[0] === "Sun"); // true
    console.log(Days[1] === "Mon"); // true
    console.log(Days[2] === "Tue"); // true
    console.log(Days[6] === "Sat"); // true

+ 类，类与接口
    + 使用class定义类，使用constructor定义构造函数，通过new生成新实例的时候，是会自动调用构造函数的


    class Animal {
        constructor(name) {
            this.name = name;
        }
        sayHi() {
            return `My name is ${this.name}`;
        }
    }

    let a = new Animal('Jack');
    console.log(a.sayHi()); // My name is Jack
    
+ 类通过extends继承
    + 然后通过super关键词来调用父类的构造函数和方法
   
    class Cat extends Animal {
        constructor(name) {
            super(name); // 调用父类的 constructor(name)
            console.log(this.name);
        }
        sayHi() {
            return 'Meow, ' + super.sayHi(); // 调用父类的 sayHi()
        }
    }

    let c = new Cat('Tom'); // Tom
    console.log(c.sayHi()); // Meow, My name is Tom

+ 泛型

    function createArray<T>(length: number, value: T): Array<T> {
        let result: T[] = [];
        for (let i = 0; i < length; i++) {
            result[i] = value;
        }
        return result;
    }

    createArray(3, 'x'); // ['x', 'x', 'x']

# Reference 
1. https://ts.xcatliu.com/advanced/type-aliases 