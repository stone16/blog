---
title: TypeScript笔记
date: 2020-04-19 10:44:45
categories: FrontEnd
tags:
    - TypeScript
top:
---

这会是一篇很长的博文，大部分内容都直接来自Reference当中的TypeScript教程和ES6教程，只是为了总结一些自己认为重要的知识点，以及一些认为需要深入理解的地方及其延伸的链接，希望有帮助。

# 1. Intro
## 1.1 定义
+ JavaScript超集，提供了类型系统和对ES6的支持，由Microsoft开发

## 1.2 优势
+ 增加代码的可读性和可维护性
    + 类型系统是很好的文档，看类型的定义我们就能够知道如何使用了
    + 可以在编译阶段发现大部分错误，比运行时出错好很多
    + 增强编辑器的功能，包括代码补全，接口提示，跳转到定义，重构等
+ 兼容性好
    + js文件实际上是可以直接重命名为ts文件的
    + 及时不显式定义类型，也能够自动做出类型推论  

## 1.3 相对劣势
+ 学习成本 
    + 接口 
    + 泛型
    + 类
+ 短期增加开发成本，要多写一些类型的定义，但是对于需要长期维护的项目，TypeScript能够减少其维护成本
+ 集成到构建流程当中需要一些工作量的 

# 2. 基础知识

## 2.1 原始数据类型

+ 布尔值 - boolean
```
    let isSuccessful: boolean = true;
    
    let createdByNewBoolean: boolean = new Boolean(1);
    // Type 'Boolean' is not assignable to type 'boolean'.
    // 'boolean' is a primitive, but 'Boolean' is a wrapper object. Prefer using 'boolean' when possible.
    
    let createdByNewBoolean: Boolean = new Boolean(1);
    // when you new Boolean, it will create a Boolean - a wrapper object
    
    let createdByBoolean: boolean = Boolean(1); 
    // create a boolean 
```
    
   
+ 数值 - number
```
    let decLiteral: number = 6;
```

+ 字符串 - string

```
    let myName: string = 'Tom';
    let myAge: number = 25;

    // 模板字符串  `用来定义模板字符串，${}用来在模板字符串中嵌入表达式
    let sentence: string = `Hello, my name is ${myName}.
    I'll be ${myAge + 1} years old next month.`;

```

+ null & undefined
    + null 和 undefined是所有类型的子类型
    + void类型的变量不能赋值给number类型的变量
+ symbol

## 2.2 任意值 - Any
任意值用来表示允许赋值为任意类型。在Typescript当中，普通类型在复制过程中改变类型是不被允许的，但是如果是any类型，则允许被赋值为任意类型。
```
    let myFavoriteNumber: any = 'seven';
    myFavoriteNumber = 7;
```

+ 任意值上访问任何属性都是允许的，也允许调用任何方法
+ 返回的类型都是任意值
+ 对于未声明类型的变量
    + 未指定类型，那么会被识别为任意值类型


## 2.3 类型推论 
如果没有明确的指定类型，那么TypeScript会按照类型推论 - Type Inference的规则推断出一个类型。

    let myFavoriteNumber = 'seven';
    myFavoriteNumber = 7;

    // index.ts(2,1): error TS2322: Type 'number' is not assignable to type 'string'.  编译的时候出错，因为TS自动做了类型推论，并且认定其为一个string类型


但是如果定义的时候没有赋值，那么不管接下来是否会赋值，都会被推断成any类型而完全不被类型检查

    let myFavoriteNumber;
    myFavoriteNumber = 'seven';
    myFavoriteNumber = 7;

## 2.4 联合类型 - Union Types

    let myFavoriteNumber: string | number;
    myFavoriteNumber = 'seven';
    myFavoriteNumber = 7;

联合类型当中使用 `|`来分割每个类型

## 2.5 接口

+ 在TypeScript当中，使用接口定义对象的类型，除了可以对类的一部分行为进行抽象以外，也chang'yo个与对 对象的形状进行描述。

    interface Person {
        name: string;
        age: number;
    }

    let tom: Person = {
        name: 'Tom',
        age: 25
    };

在做赋值的时候，定义的变量需要和接口有一样的属性。

+ 对于我们想要可选择的匹配的属性，我们可以用可选属性的方式：

    interface Person {
        name: string;
        age?: number;
    }

    let tom: Person = {
        name: 'Tom'
    };

+ 也可以配置，是的接口能够接任意的属性： -- `[propName: string] : any`
    + 需要注意的是一旦定义了任意属性，那么确定属性和可选属性的类型都必须是它的类型的子集 

    interface Person {
        name: string;
        age?: number;
        [propName: string]: any;
    }

    let tom: Person = {
        name: 'Tom',
        gender: 'male'
    };

+ 只读属性
    + 有use case我们希望对象当中的一些字段只能在创建的时候被赋值，那么就可以通过使用readonly定义只读属性


## 2.6 数组类型

+ 使用类型+方括号来定义


    let fibonacci: number[] = [1, 1, 2, 3, 5];

+ 使用数组泛型来表示数组 -- `Array<elemType>`


    let fibonacci: Array<number> = [1, 1, 2, 3, 5];

+ 使用接口表示数组


    interface NumberArray {
        [index: number]: number;
    }
    let fibonacci: NumberArray = [1, 1, 2, 3, 5];


## 2.7 函数类型

### 2.7.1 函数声明

    // 函数声明（Function Declaration）
    function sum(x, y) {
        return x + y;
    }

    // 函数表达式（Function Expression）
    let mySum = function (x, y) {
        return x + y;
    };
    
    // TypeScript下的函数声明
    function sum(x: number, y:number):number {
        return x + y;
    }

### 2.7.2 函数表达式

    let mySum: (x: number, y: number) => number = function (x: number, y: number): number {
        return x + y;
    };
    
注意在上述代码当中，跟在mySum后面的是对于输入参数和输出参数的规定，中间用箭头来进行连接，这是TypeScript的规范。

另外我们也可以通过使用接口来定义函数的形状：

    interface SearchFunc {
        (source: string, subString: string): boolean;
    }

    let mySearch: SearchFunc;
    mySearch = function(source: string, subString: string) {
        return source.search(subString) !== -1;
    }


### 2.7.3 可选参数
+ 使用问号跟在参数名字之后表示是可选的，注意可选参数需要在参数列表的末尾，其之后不能有必需参数了


    function buildName(firstName: string, lastName?: string) {
        if (lastName) {
            return firstName + ' ' + lastName;
        } else {
            return firstName;
        }
    }

### 2.7.4 参数默认与剩余参数

    // 默认参数
    function buildName(firstName: string, lastName: string = 'Cat') {
        return firstName + ' ' + lastName;
    }
    let tomcat = buildName('Tom', 'Cat');
    let tom = buildName('Tom');

    // 使用 ...来获取函数当中的剩余参数
    function push(array, ...items) {
        items.forEach(function(item) {
            array.push(item);
        });
    }

    let a: any[] = [];
    push(a, 1, 2, 3);
# Reference
1. https://ts.xcatliu.com/
2. https://es6.ruanyifeng.com/