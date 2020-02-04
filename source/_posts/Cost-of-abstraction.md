---
title: Cost of abstraction
date: 2020-02-04 09:08:47
categories: BackEnd
tags:
    - Java
top:
---
Duplicate is definitely something we are trying to get rid of, however, what's the cost of abstraction? It seems some problem we could think for a while. 

We should think of the tradeoff between code duplication and increased level of abstraction. 

# 1. Definition - cost of abstraction 

+ An abstraction is adding to the cognitive load of whoever works with the code 
+ The main cost of abstraction: separating the implementation from the specification. Or we say separate the letter of function from the spirit of the function
+ The former being what the function does, the latter being what everybody believes it should do 
+ Should involve everybody to consider what the code does, or we say what the function does 

# 2. Thought 

+ The decision about creating an abstraction should not be taken lightly.
+ There's a large social cost to every abstraction, may lead the project to be unmaintainable.
+ lambda is also a way to reduce the abstraction layers in some way 
+ Abstraction is a good way to get rid of verbose code, but itself may bring uncertainty to some extent. Only implement ***necessary*** abstraction. 