---
title: Java 理解多态
date: 2020-02-29 08:34:38
categories: BackEnd
tags:
    - Java 
    - Polymorphism
top:
---
# 1. 什么是多态？ 

多态目的是为了分离做什么和怎么做，从而实现接口和实现的分离。通过多态，可以改善代码的组织结构和可读性，最重要的是能够创建可扩展的程序。

有继承关系的类，子类重写父类的方法，然后父类的引用指向子类。通过这种方式，就可以对于父类的声明指向子类的实际对象，

# 2. 为什么需要多态？

多态的好处很多，令代码可扩展，解耦接口与实现，让代码对于改动封闭，对于扩展开放, etc. 如果直接从代码的角度来看，我们可以比较直观的看到他的优势


    public class Animal {
        public void move() {
            System.out.println("Animal move");
        }
    }
    
    public class Cat extends Animal {
        @Override
        public void move() {
            System.out.println("cat climb");
        }
    }
    
    public calss Dog entends Animal {
        @Override
        public void move() {
            System.out.println("dog run");
        }
    }


    public static void main(String [] args) {
        Animal animal = new Cat();
        animal.move();
        // output: cat climb
    }

通过这种方式可以实现接口与实现的解耦。

# 3. 多态和继承的关系

继承指在子类当中使用父类的数据和方法

多态指在子类当中改变父类的行为。

# 4. 构造器内部的多态方法的行为

如果在一个构造器的内部调用正在构造的对象的某个动态绑定的方法，会出现一些不可知的错误。

因为构造器内部的动态绑定意味着要用到方法被覆盖以后的定义，而这意味着被覆盖的方法在对象被完全构造之前就会被调用了，

    class Graph {
        void draw() {
            print("Graph draw()");
        }
        
        Graph() {
            print("Graph() before draw()");
            draw();
            print("Graph() after draw()");
        }
    }
    
    class RoundGraph extends Graph {
        private int radius = 1;
        
        RoundGraph(int r) {
            radius = r;
            print("RoundGraph.RoundGraph(), radius = " + radius);
        }
        
        void draw() {
            print("RoundGraph.draw(), radius = " + radius);
        }
    }
    
    public class PolyConstructors {
        public static void main(String[] args) {
            new RoundGraph(5);
        }
    }
    
    // All output 
    Graph() before draw()
    RoundGraph.draw(), radius = 0
    Graph() after draw()
    ROundGraph,RoundGraph(), radius = 5


有这样的输出的原因是当我们实例化R欧尼的Graph的时候，会调用基类的构造器，在Graph类的构造器当中调用了draw()方法，这个时候动态绑定，是要去调用RoundGraph类的draw方法的，但是这个时候还在构建Graph 的实例，RoundGraph还没有构建好，所以就出现了返回的Radius刚开始值为0的问题了

# Reference 

1. Thinking in Java Ch.8 