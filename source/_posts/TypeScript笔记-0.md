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



# Reference
1. https://ts.xcatliu.com/
2. https://es6.ruanyifeng.com/
