---
title: EnumMap vs HashMap
date: 2020-02-04 09:10:34
categories: BackEnd
tags:
    - Java
top:
---
# 1. EnumMap
+ Can only be used with **enum type** keys 
+ It's specialized implementation of Map Interface for use with enum type keys
+ Internally using **arrays** 
+ stored in **natural order**
+ not possible for collision 
+ much efficient compared with HashMap 

# 2. HashMap

+ Extends AbstrctMap and implement Map interface 
+ Internally using hashTable
+ Possible for collision 


# Reference

1. https://www.geeksforgeeks.org/enummap-class-java-example/ 
2. https://walkingtechie.blogspot.com/2017/03/difference-between-enummap-and-hashmap.html