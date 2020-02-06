---
title: Java 继承与组合
date: 2020-02-05 19:03:28
categories: BackEnd
tags:
    - Java
    - Composition
top:
---

# 1. 什么是组合？ 

组合将类定义为各个部分的集合


    public class MerchantListHelper {
    
        // TODO 
        void addToDDB (ListEntry entry) {
    
        }
    
        void removeFromDDB(DDBRecord record) {
    
        }
    
        List<ListEntry> getFromDDB() {
    
        }
    
    }
    
    public class MerchantBlacklistBizLogic {
        private MerchantListHelper merchantListHelper;
    
        merchantListHelper.addToDDB(blabla..);
    
        merchantListHelper.removeFromDDB(blabla..)
    }

# 2. 什么是继承？ 

将父类和子类通过集成关系紧密联系在一起 （紧耦合）。但与之相对的是继承会允许再利用类的方法以及其他的属性，会很便捷。

使用super()方法来直接访问父类的方法，构造器，属性等。

    public Abstract class MerchantListHelper {
    
        // TODO 
        abstract void addToDDB (ListEntry entry);
    
        abstract void removeFromDDB(DDBRecord record);
    
        abstract List<ListEntry> getFromDDB();
    
    }
    
    public class MerchantBlacklistBizLogic {
        private MerchantListHelper merchantListHelper;
    
        @Override
        void addToDDB(ListEntry entry) {
            // TODO
        }
    
        @Override
        void removeFromDDB(DDBRecord record) {
            // TODO
        }
    
        ...
    }
# 3. Use cases

组合和继承都可以将子对象放到新的类当中，组合一般是当你想要在这个新类当中使用一个已经存在的类的特征的时候。这意味着，通过这种方式你可以嵌入一个对象，并将其放在新的类当中。 继承表示的是一种is-a的关系，组合表达的含有某种功能。

如何判断是否需要继承或者组合 -> 看是否需要从你的新类到基本类进行向上转型。




# Reference
1. http://thedeanbear.com/2012/09/24/composition_vs_inheritance/
2. Java Challengers #7: Debugging Java inheritance 
3. javaworld.com/article/3409071/java-challenger-7-debugging-java-inheritance.html