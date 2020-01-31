---
title: React Advanced(4)
date: 2020-01-30 23:15:33
categories: FrontEnd
tags:
    - FrontEnd
    - React
top:
---

# 1. The component lifecycle 

Each component has several lifecycle methods that you can override to run code at perticular times in the process. 

## 1.1 Mounting 

Methods are called **in this order** when an instance of a component is being created and inserted into the DOM. 

1. constructor()
2. static getDerivedStateFromProps(): exists for use cases where the state depends on changes in props over time 
3. render()
4. componentDidMount()

## 1.2 Updating 

1. static getDerivedStateFromProps()
2. shouldComponentUpdate(): judge if a component's output is not affected by the current change in state or props
3. render()
4. getSnapshotBeforeUpdate()
5. componentDidUpdate() 

## 1.3 Unmounting

1. ComponentWillUnmount()

## 1.4 Error Handling 

1. static getDerivedStateFromError()
2. componentDidCatch() 


# 2. JSX in Depth 

## 2.1 Specifying the react element type

Capitalized types indicate that the JSX tag is **referring to a React component**. These tages get compiled into a direct reference to the named variable, so if you use the JSX `<Foo/>` expression, Foo must be in scope. 

E.G Here: 

    import React from 'react';
    import CustomButton from './CustomButton';
    
    function WarningButton() {
      // return React.createElement(CustomButton, {color: 'red'}, null);
      return <CustomButton color="red" />;
    }
    
Need to import those things before truly use it in function scope. 

## 2.2 User defined components must be capitalized 

When an element type starts with a lowercase letter, it refers to a build-in component like `<div>` or `<span>` passed to React.createElement.  

***Types that start with a capital letter like `<Foo/> ` compile to React.createElement(Foo) and correspond to a component defined or imported in your js file. 

## 2.3 Spread Attributes 

If you already have props as an object, and you want to pass it in JSX, you can use `...` as a spread operator to pass the whole props object. 

Equivalent expressions: 

    function App1() {
      return <Greeting firstName="Ben" lastName="Hector" />;
    }
    
    function App2() {
      const props = {firstName: 'Ben', lastName: 'Hector'};
      return <Greeting {...props} />;
    }
    
## 2.4 Children in JSX

In JSX expressions that contain both an opening tag and a closing tag, the content between those tags is passed as a special prop: `props.children`. Several different ways to pass children: 

1. String literals 

`<div>Hello World!</div>`

2. JSX children

3. JS Expressions as Children

Wrap it within `{}`
