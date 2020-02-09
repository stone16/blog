---
title: Ruby Totorial
date: 2020-02-08 21:22:07
categories: BackEnd
tags:
    - Ruby
top:
---

Met circumstances where need Ruby knowledge to resolve problems, thus need to do a quick touch on Ruby, at least know how to read ruby code. 

Leran by doing, or we say, learn by satisfying current needs. 

# 1. Basics 

+ features 
    + object-oriented 
    + server side scripting language 
    + can be used to write common gateway interface(CGI) scripts

## 1.1 Syntax of Ruby

+ whitespace 
    + ignored in ruby code, except when they appear in strings. 
+ end of line
    + you could use 
        + semicolons 
        + newline characters as the ending of a statement 
+ Ruby Identifiers
    +  case sensitive 
+ comments
    + `#`
    + `=begin` at beginning, `=end` at the end
    

    // Declares code to be called before the program run 
    BEGIN {
        code
    }
    
    // Declares code to be called at the end of the program 
    END {
        code
    }
+ classes and objects 
    + features
        + data encapsulation 
        + data abstraction 
        + polymorphism
        + inheritance 


    // Class example
    Class Vehicle {
    
       Number no_of_wheels
       Number horsepower
       Characters type_of_tank
       Number Capacity
       Function speeding {
       }
       
       Function driving {
       }
       
       Function halting {
       }
    }


## 1.2 Class and Objects
  
+ Define a class in Ruby 


    // A class always starts with keyword class, followed by the name of the class. 
    // Terminate a class by using the keyword end. 
    class Customer
    end

+ Variables in a Ruby Class 
    + local variables 
        + defined in a method
        + begin with a lowercase letter or _. 
    + instance variables 
        + available across methods for any particular instance or object 
        + instance variables change from object to object 
        + `@`
    + class variables 
        + available across different objects 
        + belongs to the class and is a characteristic of a class 
        + `@@`
    + global variables 
        + Class variables are not available across classes, while global variables are.  
        + `$`


    // Determine the number of objects that are being ccreated 
    class Customer
       @@no_of_customers = 0
    end

+ creating objects with `new` method
    +  `object1 = Customer.new`
    +  object1 is object name
    +  Customer is class
    +  To instantiate a new object, you need to use class name followed by dot and new(keyword)
+ custom method to create ruby objects (similar to constructor concept in Java)
    + pass parameters to method new 
    + when you plan to declare new method with parameters, you need to declare the method **initialize** at the time of the class creation 


    class Customer
       @@no_of_customers = 0
       def initialize(id, name, addr)
          @cust_id = id
          @cust_name = name
          @cust_addr = addr
       end
    end
    
    // To create objects 
    cust1 = Customer.new("1", "John", "Wisdom Apartments, Ludhiya")

+ member functions in class 
    + each method in a class starts with the keyword `def` followed by the method name 


    class Sample
        def function
            statement 1
            statement 2
        end
    end
    
    
    
    // A full example 
    #!/usr/bin/ruby
    class Sample
       def hello
          puts "Hello Ruby!"
       end
    end
    
    # Now using above class to create objects
    object = Sample. new
    object.hello

## 1.3 Variables 

+ Global Variables 
    + begin with $ 
    + uninitialized global variables have the value `nil` 


    #!/usr/bin/ruby
    
    $global_variable = 10
    class Class1
       def print_global
          # In ruby, you can use HashTag to access any variables value 
          puts "Global variable in Class1 is #$global_variable"
       end
    end
    class Class2
       def print_global
          puts "Global variable in Class2 is #$global_variable"
       end
    end
    
    class1obj = Class1.new
    class1obj.print_global
    class2obj = Class2.new
    class2obj.print_global


+ Instance Variables 
    +  begin with `@`


    #!/usr/bin/ruby
    
    class Customer
       def initialize(id, name, addr)
          @cust_id = id
          @cust_name = name
          @cust_addr = addr
       end
       def display_details()
          puts "Customer id #@cust_id"
          puts "Customer name #@cust_name"
          puts "Customer address #@cust_addr"
       end
    end
    
    # Create Objects
    cust1 = Customer.new("1", "John", "Wisdom Apartments, Ludhiya")
    cust2 = Customer.new("2", "Poul", "New Empire road, Khandala")
    
    # Call Methods
    cust1.display_details()
    cust2.display_details()


+ Class Variables 
    + begin with @@
    + must be initialized before they can be used in method definitions 


    #!/usr/bin/ruby
    
    class Customer
       @@no_of_customers = 0
       def initialize(id, name, addr)
          @cust_id = id
          @cust_name = name
          @cust_addr = addr
       end
       def display_details()
          puts "Customer id #@cust_id"
          puts "Customer name #@cust_name"
          puts "Customer address #@cust_addr"
       end
       def total_no_of_customers()
          @@no_of_customers += 1
          puts "Total number of customers: #@@no_of_customers"
       end
    end
    
    # Create Objects
    cust1 = Customer.new("1", "John", "Wisdom Apartments, Ludhiya")
    cust2 = Customer.new("2", "Poul", "New Empire road, Khandala")
    
    # Call Methods
    cust1.total_no_of_customers()
    cust2.total_no_of_customers()
    
+ Local variables 
    + begin with a lowercase letter or `_`
    + scope
        + class
        + module
        + def
        + do to the corresponding end
        + block's opening brace to its close brace 
+ Constants
    + Begin with an **uppercase** letter  
    + defined within a class or module 
+ Pseudo-variables 
    + self 
    + true
    + false
    + nil 
        + Value representing undefined 
    + `_FILE_`
        + the name of the current source file 
    + `_LINE_`
        + the current line number in the source file 
## 1.4 Arrays
Array are created by placing a comma-separated series of object references between the square brackets.

    #!/usr/bin/ruby
    
    ary = [  "fred", 10, 3.14, "This is a string", "last element", ]
    ary.each do |i|
       puts i
    end

## 1.5 Hashes
Hash is created by placing a list of key/value pairs between braces, with either a comma or the sequence => between the key and the value. A trailing comma is ignored.

    #!/usr/bin/ruby
    
    hsh = colors = { "red" => 0xf00, "green" => 0x0f0, "blue" => 0x00f }
    hsh.each do |key, value|
       print key, " is ", value, "\n"
    end

## 1.6 Ranges

A Range represents an interval which is a set of values with a start and an end. Ranges may be constructed using the s..e and s...e literals, or with Range.new.

    #!/usr/bin/ruby
    
    (10..15).each do |n| 
       print n, ' ' 
    end

## 1.7 Operators 

+ `<=>`
    + ruturn 0 if first operand equals second
    + 1 if first greater than second
    + -1 if first less than second 
+ `.eql?`
    + true if the receiver and argument have both the same type and equal values
+ `equal?`
    + true if the receiver and argument have the same object id 
+ `..`
    + 1..10 creates a range from 1 to 10 inclusive
+ `...`
    + 1...10 creates a range from 1 to 9  
+ defined? operators
    + takes the form of a method call to determine whether or not the passed expression is defined
    + returns a description string of the expression, or nil if the expression isn't defined 
+ dot operators
    +  
+ double colon `::` operators
    +  You call a module method by preceding its name with the module's name and a period, and you reference a constant using the module name and two colons. 
    +  `::` us a unary operator that allows constants, instance methods and class methods defined within a class or module to be accessed from anywhere outside the class or module 
    +  **Classes and methods are considered to be constants too **

## 1.8 Conditions 
    
    // if else condition check
    if condition
        code..
    elsif condition2
        code
    else 
        code
    end
    
    // case 
    #!/usr/bin/ruby

    $age =  5
    case $age
    when 0 .. 2
       puts "baby"
    when 3 .. 6
       puts "little child"
    when 7 .. 12
       puts "child"
    when 13 .. 18
       puts "youth"
    else
       puts "adult"
    end

## 1.9 Loops 

    while condition do 
        code
    end

Executes code while conditional is true 


    $i = 0
    $num = 5
    begin 
        puts("123")
        $i += 1
    end while $i < $num 

    // for loop
    for i in 0..5
        puts "Value of local variable is #{i}"
    end
    
    // subtitute way of for loop
    (expression).each do |variable| 
        code
    end
    
    // E.G 
    (0..5).each do |i|
        puts "Value of local variable is #{i}"
    end 
    
+ `next`
    + jump to the next iteration of the most internal loop 
+ `redo`
    + restarts this iteration of the most internal loop, without checking loop condition  
+ `retry`
+ `break`
    + terminate the most internal loop  

## 1.10 Methods

+ used to bundle one or more repeatable statements into a single unit 
+ method name should begin with a lowercase lettter 
+ method should be defined before calling them 
+ call the method by direcly type in the method name `method_name`
+ with parameters `method_name 25, 30`
+ Ruby will return the value of lat statement by default 
+ or use the return statement 
+ method defined in the class definition are marked as **public** by default 
+ a block is always invoked from a function with the same name as that of the block 

    def method_name (var1, var2)
        expr
    end 
    
+ variable number of parameters 
    + `def sample (*test)`   

## 1.11 Blocks

+ definition 
    + consists of chunks of code
    + assign a name to a block 
    + code in the block is always enclosed within braces `{}` or `()`
    + a block is always invoked from a function with the same name as that of the block
    + invoke a block by using the `yield` statement 
+ if the last argument of a method is preceded by &, then you can pass a block to this method and this block will be assigned to the last parameter. 


    #!/usr/bin/ruby
    
    def test(&block)
       block.call
    end
    test { puts "Hello World!"}

## 1.12 Modules and Mixins

+ Module
    + way of grouping together methods, classes, and constants 
    + provides namespace and prevent name clashes
        + a sandbox  
    + implement mixin facility 


    // syntax
    module Identifier
        statement1
        statement2
    end
    
+ call a module method by precedint its name with the module's name and a period 
+ reference a constant using the module name and two colons 



    #!/usr/bin/ruby

    # Module defined in trig.rb file
    
    module Trig
       PI = 3.141592654
       def Trig.sin(x)
       # ..
       end
       def Trig.cos(x)
       # ..
       end
    end

+ `require`
    + similar to import, include
    + if a third program wants to use any defined module, it can simply load the module files using the Ruby`require` statement 

+ mixin 
    + multiple inheratance 



    module A
       def a1
       end
       def a2
       end
    end
    module B
       def b1
       end
       def b2
       end
    end
    
    class Sample
    include A
    include B
       def s1
       end
    end
    
    samp = Sample.new
    samp.a1
    samp.a2
    samp.b1
    samp.b2
    samp.s1

In this way, samp could call method defined in Module A and Module B

## 1.13 Strings

+ holds and manipulates an arbitrary sequence of one or more bytes

## 1.14 Array 

+ ordered, integer indexed collections of any object
+ each element in an array is associated with and referred to by an index 
+ creating arrays
    + `Array.new` 