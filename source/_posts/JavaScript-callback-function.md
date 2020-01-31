---
title: JavaScript - callback function
date: 2020-01-30 20:57:57
categories: FrontEnd
tags:
    - FrontEnd
    - JavaScript
    - Callback
top:
---
# 1. What is

+ A function that is to be executed after another function has finished executing -- hence the name call back 
+ In JavaScript, functions are **objects**. Because of this, **functions can take functions as arguments**, and can be returned by other functions. Functions that do this are called **higher-order functions**. Any function that is passed as an argument is called a callback function.


# 2. Why need

JS is an event driven language, instead of waiting for a response before moving on, JS will keep executing while listening for other events. 

    function first(){
      // Simulate a code delay
      setTimeout( function(){
        console.log(1);
      }, 500 );
    }
    
    function second(){
      console.log(2);
    }
    
    first();
    second();
    
    // 2, 1

The above example shows JS didn't wait for a response from first() before moving on to execute second() 

# 3. How to 

    function doHomework(subject, callback) {
      alert(`Starting my ${subject} homework.`);
      callback();
    }
    
    doHomework('math', function() {
      alert('Finished my homework');
    });
    
# Reference 
https://developer.mozilla.org/en-US/docs/Glossary/Callback_function