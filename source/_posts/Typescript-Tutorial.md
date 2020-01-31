---
title: Typescript Tutorial
date: 2020-01-30 22:31:34
categories: FrontEnd
tags:
    - TypeScript
    - FrontEnd
top:
---
# 1. Overview 

+ A typed superset of JavaScript that compiles to plain JavaScript
+ Pure object oriented with classes, interfaces 
+ for application scale development 
+ benefits
    + provides the error checking feature 
    + will compile the code and generate compilation errors 
    + strong static typing 
    + supports type definitions for existing JS libraries
        + TS definition file (with `.d.ts` extension) provides definition for external JS libraries
+ components
    + language 
        + comprises of 
            + syntax
            + keywords
            + type annotations
    + the typeScript Compiler 
        + The TypeScript Comiler converts the instructions written in TypeScript to its JavaScript equivalent 
    + the typeScript Language Service
        + expose an additional layer around the core compiler pipeline that are editor-like applications
        + language service supports 
            + statement completions
            + signature help
            + code formatting and outlining
            + colorization 
+ declaration files 
    + declaration file 
        + interface to the components in the compiled javaScript
        + analogous to the header files 

# 2. Basic Syntax 

+ composed of 
    + module 
    + fucntions 
    + variables
    + statements and expressions
    + comments
+ tsc will translate ts file to js 
+ ts and oop
    + object 
        + state 
        + behavior 
        + identity 
    + class
    + method


    class Greeting { 
       greet():void { 
          console.log("Hello World!!!") 
       } 
    } 
    var obj = new Greeting(); 
    obj.greet();

## 2.1 Types

Represents the different types of values supported by the language, The type system checks the validity of the supplied values, before they are stored or manipulated by the program. 

The type system further allows for **richer code hinting** and **automated documentation** too. 

+ any type
    + super type of all types in TS 
    + denotes a dynamic type 
    + use the any type is equivalent to opting out of type checking for a variable 
+ built-in types
    + Number 
        + double precision 64 bit floating point values 
    + String 
        + a sequence of Unicode characters 
    + Boolean 
        + logical values 
    + Void 
        + used on function return types to represent non-returning functions 
    + Null 
        + represents an intentional absense of an object value 
    + Undefined 
        + denotes value given to all uninitialized variables 

## 2.2 Variables 

+ Variable - name space in the memory that stores values.
+ acts as a container for values in a program 
+ use the `var` keyword to declare variables 

    
    var [identifier] : [type-annotation] = value;
    
### 2.2.1 Type assertion 

+ to change a variable from one type to another 


    var str = '1';
    var str:number = <number> str;

+ if you doesn't add type for variables, ts will determine the type of the variable on the basis of the value assigned to it

### 2.2.2 Variable Scope

+ global scope
    + declare outside the programming constructs
    + can be accessed from anywhere within code
+ class scope
    + called fields 
    + are declared within the class but outside the methods 
    + can be accessed using the object of the class 


    var global_num = 12          //global variable 
    class Numbers { 
       num_val = 13;             //class variable 
       static sval = 10;         //static field 
       
       storeNum():void { 
          var local_num = 14;    //local variable 
       } 
    } 
    console.log("Global num: "+global_num)  
    console.log(Numbers.sval)   //static variable  
    var obj = new Numbers(); 
    console.log("Global num: "+obj.num_val) 

## 2.3 Operators 

+ Defines some function that will be performed on the data 
    + arithmetic operators 
    + relational operators 
    + logical operators 
    + bitwise operators 
    + assignment operators 
    + type operators
        + typeof 
            + return the type of variable
        + instance of 
            + return the class of an object  

## 2.4 Functions 

+ building blocks of readable, maintainable and reusable code 
+ organize the program into logical blocks of code 
+ function declaration tells the compiler about a function's name, return type, and parameters 

### 2.4.1 Optional Parameters 

+ Add a question mark to its name indicating as optional 
+ should be set as the last argument in a function 


    function disp_details(id:number,name:string,mail_id?:string) { 
       console.log("ID:", id); 
       console.log("Name",name); 
       
       if(mail_id!=undefined)  
       console.log("Email Id",mail_id); 
    }
    disp_details(123,"John");
    disp_details(111,"mary","mary@xyz.com");


### 2.4.2 Rest Parameters 

+ not restrict the number of values that you can pass to a function 
+ values passed in mush be the same type 
+ parameter is prefixed with three periods , non rest parameters should come before the rest parameter 
+ a function can have at most one rest parameter 


    function addNumbers(...nums:number[]) {
        var i;
        var sum:number = 0;
        
        for (i = 0; i < nums.length; i++) {
            sum += nums[i];
        }
        console.log("sum of numbers", sum);
    }

### 2.4.3 Default Parameters 

+   you could assign paramters with default values  


    function calculate_discount(price:number,rate:number = 0.50) { 
       var discount = price * rate; 
       console.log("Discount Amount: ",discount); 
    } 
    calculate_discount(1000) 
    calculate_discount(1000,0.30)


### 2.4.4 Function Overloads

+ have different data type 
+ have different number of parameters 


    function disp(string):void; 
    function disp(number):void;
    
    function disp(n1:number):void; 
    function disp(x:number,y:number):void;
    
    function disp(n1:number,s1:string):void; 
    function disp(s:string,n:number):void;

## 2.5 Numbers

+ Number class acts as a wrapper and enables manipulation of numeric literals as they are objects 
+ `var var_name = new Number(value)`
+ property 
    + MAX_VALUE
    + MIN_VALUE
    + NaN
    + NEGATIVE_INFINITY
    + POSITIVE_INFINITY
    + prototype 
        + a static property of the Number object 
        + use the prototype property to assign new properties and methods to the Number object in the current document 
    + constructor 
        + returns the function that created this object's instance
+ method 
    + toExponential()
    + toFixed()
        + formats a number with a specific number of digits to the right of the decimal 
    + toLocaleString()
    + toPrecision()
    + toString()
    + valueOf() 

## 2.6 Strings

+ String objects lets you work with a series of characters.
+ Wraps the string primitive data type with a number of helper methods 
+ `var var_name = new String(string)`
+ property 
    + constructor 
    + length
    + prototype 
        + allow you to add properties and methods to an object 
+ method
    + charAt()
    + charCodeAt()
    + concat()
    + indexOf()
    + lastIndexOf()
    + localeCompare()
    + match()
        + use to match a regular expression against a string 
    + replace()
    + search()
    + slice()
    + split()
    + substr()
    + substring()
    + toLocaleLowerCase()
    + toLocaleUpperCase()
    + toString()
    + toUpperCase()
    + valueOf()

## 2.7 Arrays 

+ array is a collection of values of the **same** data type 
+ features 
    + allocates sequential memory blocks 
    + arrays are static, once initialized cannot be resized 
    + each memory block represents an array element 
    + array elements are identified by a unique integer called as the subscript/ index of the element 
    + should be declared before they are used with `var`
    + array element values can be updated or modified but cannot be deleted 



    var arr_name:number[] = [2,4,6];
    var arr_name:number[] = new Array(4);
+ methods
    + concat() 
    + every()
        + return true if every element in this array satisfies the provided testing function 
    + filter()
    + forEach()
    + indexOf()
    + join()
    + lastIndexOf()
    + map()
    + pop()
    + push()
    + reduce()
        + apply a function simultaneously against two values of the array as to reduce it to a single value - from left to right
    + reduceRight()
        + from right to left
    + reverse()
    + shift()
        + removes the first element from an array and returns the element 
    + slice()
    + some()
        + Returns true if at least one element in this array satisfies the provided testing function 
    + sort()
    + splice()
        + add or remove elements from an array 
    + toString()
    + unshift()
## 2.8 Tuples

+ To store a collection of values of varied types 
+ represents a heterogeneous collection of values 


    // accessing 
    var tuple1 = [10, "hello"];
## 2.9 Union 

+ Give program the ability to combine one or two types 
+ Union can be used to express a value that can be one of the several types 
+ two or more types are combined with `|`

# 3. OOP related

## 3.1 Interfaces 

An interface defines the syntex that any entity must adhere to.

+ Interface define properties, methods and events
+ only contain the decalration of the members 
+ responsibility of the deriving class to define the members 


    interface IPerson { 
       firstName:string, 
       lastName:string, 
       sayHi: ()=>string 
    } 
    
    var customer:IPerson = { 
       firstName:"Tom",
       lastName:"Hanks", 
       sayHi: ():string =>{return "Hi there"} 
    } 
    
    console.log("Customer Object ") 
    console.log(customer.firstName) 
    console.log(customer.lastName) 
    console.log(customer.sayHi())  
    
    var employee:IPerson = { 
       firstName:"Jim",
       lastName:"Blakes", 
       sayHi: ():string =>{return "Hello!!!"} 
    } 
      
    console.log("Employee  Object ") 
    console.log(employee.firstName);
    console.log(employee.lastName);
    
+ An interface can be extended by other interfaces 
+ an interface can inherit from other interface 


    interface Person { 
       age:number 
    } 
    
    interface Musician extends Person { 
       instrument:string 
    } 
    
    var drummer = <Musician>{}; 
    drummer.age = 27 
    drummer.instrument = "Drums" 
    console.log("Age:  "+drummer.age) console.log("Instrument:  "+drummer.instrument)

## 3.2 Classes 

+ Object oriented JavaScript 
+ class can contain
    + fields
    + constructors 
    + functions 
+ class Inheritance 
    + create new class from an existing one 
    + the class that is extended to create newer classes is called the parent class/ super class 
    + child class inherit all properties and methods except private members and constructors from the parent class 
+ data hiding 
    + public 
    + private 
        + only accessible within the class that defines these members  
    + protected 
        + A protected data member is accessible by the members within the same class as that of the former and also by the members of the child classes.


## 3.3 Objects 

+ Object is an instance which contains set of key value pairs
+ the values can be scalar values or functions or even array of other objects 

## 3.4 namespace 

+ a way to logically group related code 
+ resolve the possibility of overwriting or miscontrucing the same variables 


    namespace SomeNameSpaceName { 
       export interface ISomeInterfaceName {      }  
       export class SomeClassName {      }  
    } 

+ marked classes or interfaces with keyword export to indicate they should be accessed outside the namespace 

## 3.5 Modules

Module is designed to organize code written in typeScript. Divided into Internal Modules and External Modules. 

+ Internal Modules
    + logically group classes, interfaces, functions into one unit and can be exported in another module 
    + now it names namespace in new version of ts
+ External Module 
    + exist to specify and load dependencies between multiple external js files  

### 3.5.1 Selecting a Module Loader

+ for browser 
    + RequireJS 
        + an implementation of AMD (asynchronous module definition) specification 
        + AMD can load js files all separately 

### 3.5.2 Defining external Module 

syntax for declaring an external module is using keyword export and import 

## 3.6 Ambient 

+ tell the TS compiler that the actual source code exists elsewhere 
+ kept in extension `d.ts`
+ When you are consuming a bunch of third party js libraries like jquery/angularjs/nodejs you can’t rewrite it in TypeScript.
