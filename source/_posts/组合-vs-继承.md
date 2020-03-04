---
title: 组合 vs 继承
date: 2020-03-03 20:21:56
categories: BackEnd
tags:
    - Java
    - Composition
    - Inheritance
top:
---
组合还有继承都是面向对象的很重要的特性，但是有一条非常重要的设计原则说 -- 组合优于继承，想在这篇博文当中分析一下为什么认为组合优于继承，以及什么情况下我们仍然应该使用继承。

继承可以表示类之间的is-a的关系，可以一定程度上解决代码复用性的问题，但是继承层次过深，也会影响到代码的可维护性。

# 1. 继承的劣势

譬如 我们现在要实现一个关于哺乳动物的类，我们首先需要将哺乳动物定义为一个抽象的类

    public class Mammal {
        public void breathWithLung() {
            
        }
    }

此时我们就可以实例化Monkey，Whale等一系列哺乳动物了。但是对于哺乳动物来说，他的属，科，目门类很多，海陆空都有，假设我们按照他们的行为来进行分类的话。可以分成会飞的，会游的，还有会跑的。所以就可以写如下的代码：
    
    public class flyableMammal extends Mammal {
        public void fly() {
            
        }
    }
    
    public class underwaterMammal extends Mammal {
        public void swim() {
            
        }
    }
    
    public class onLandMammal extends Mammal {
        public void run() {
            
        }
    }

这个时候我们已经有两层的继承了，而后我们可以实例化一些哺乳动物，譬如鲸鱼，海豚，狮子等等来创建真的对象。然后问题来了，我们现在想探究会飞的动物当中，夜行的类目，那就意味着我们需要再创建一个新的层级来进行研究了。

长此以往，整个层级就会变得很深。而深度的层级意味着每当我们想真真切切去研究到底这个类做了什么的时候，我们需要去他的父类，去他的父类的父类，追本溯源，一个一个看其中定义的属性和方法，才能完全理解他做了什么。

这样子来做，首先造成了代码的可读性变得非常差，而对于类本身而言，破坏了其封装特性，将父类的实现细节暴露给了子类。在这种情况下，一旦父类代码修改，就会影响所有子类的逻辑。

# 2. 组合的优势

对于上述的场景，我们完全可以用组合的方式来实现。通过设立多个功能接口，来表示当前类的属性，譬如flyable, runnable, swimmable, etc. 通过这种方式，我们让concrete class直接implements对应的接口，并override写出自己的实现。这样子就能够解决这个问题了。

除了使用接口之外，我们也可以使用委托的方式，即仍然定于对应的接口，但是还定义了实现了接口方法的实现类，在实际使用的时候，调用实现类的方法直接使用，例子如下


    public interface Flyable {
      void fly()；
    }
    public class FlyAbility implements Flyable {
      @Override
      public void fly() { //... }
    }
    //省略Tweetable/TweetAbility/EggLayable/EggLayAbility
    
    public class Ostrich implements Tweetable, EggLayable {//鸵鸟
      private TweetAbility tweetAbility = new TweetAbility(); //组合
      private EggLayAbility eggLayAbility = new EggLayAbility(); //组合
      //... 省略其他属性和方法...
      @Override
      public void tweet() {
        tweetAbility.tweet(); // 委托
      }
      @Override
      public void layEgg() {
        eggLayAbility.layEgg(); // 委托
      }
    }

# 3. 如何判断该使用继承还是组合？

组合也并不完美，组合需要对类做更细度的拆分，要定义更多的类和接口，因此在实际开发的过程当中，我们还是要根据具体的情况，来具体选择该使用继承还是组合。

如果类之间的继承结构稳定 -- 不会轻易改变，继承关系比较浅 -- 譬如两层到三层的继承关系，那么我们可以直接使用继承。反之，对于系统不够稳定，继承层次会很深，且关系复杂的，我们就应该尽量使用组合来替代继承了。

或者当我们想要使用多态的特性的时候，我们就需要使用继承了。