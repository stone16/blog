---
title: 迪米特法则 (LOD) — 高内聚，低耦合
date: 2020-03-22 20:30:09
categories: SystemDesign
tags: 
    - 高内聚
    - 低耦合
    - 迪米特
top:
---
高内聚 低耦合是比较通用的设计思想，可以用来指导不同的粒度的代码的设计和开发的工作，比如系统，模块，类，甚至是函数。也可以去使用到不同的开发场景当中，比如微服务，框架，组件，类库等等。

在这个原则当中，高内聚指的是类本身的设计，低耦合指的是类和类之间的依赖关系的设计。

迪米特法则，可以称之为The least knowledge principle. 
Each unit should have only limited knowledge about other units: only units “closely” related to the current unit. Or: Each unit should only talk to its friends; Don’t talk to strangers.

# 1. 什么是高内聚？

指的是相近的功能应该放到同一个类当中，不相近的功能不要放在同一类。代码集中相对来说就会更加容易维护了。

# 2. 什么是低耦合？

类和类之间的依赖关系简单清晰，即尽管两个类之间有依赖关系。一个类的代码的改动不会或者很少导致依赖类的代码的改动。

# 3. 内聚和耦合的关系

![内聚耦合关系.png](https://i.loli.net/2020/03/23/yZVTqaQSgvbtE4l.png)

如图所示，左侧就是很好的高内聚低耦合的范例，我们将类最小化，即每个类只做一件事情，这样子其他依赖就会少很多。在修改或增加功能的时候，就不会对其他的类造成很大的影响。

# 4. 实战


    public class NetworkTransporter {
        // 存在问题，NetworkTransporter作为一个底层类，不应该依赖于HtmlRequest类；与之相反的，因为其实他需要的是string address，以及byte的数组，那我们应该直接提供这些primitive type的数据
        public Byte[] send(HtmlRequest htmlRequest) {
          //...
        }
    }
    
    public class HtmlDownloader {
      private NetworkTransporter transporter;//通过构造函数或IOC注入
      
      public Html downloadHtml(String url) {
      // 根据上面NetworkTransporter我们希望做的改动，这里传入的不应该是HtmlRequest类的实例了
        Byte[] rawHtml = transporter.send(new HtmlRequest(url));
        return new Html(rawHtml);
      }
    }
    
    public class Document {
      private Html html;
      private String url;
      
      public Document(String url) {
        this.url = url;
        // downloader.downloadHtml逻辑复杂，不应该放在构造函数当中，也会很不好测试
        // 构造函数中使用new来做实例，违反了基于接口而非实现编程的原则
        HtmlDownloader downloader = new HtmlDownloader();
        this.html = downloader.downloadHtml(url);
      }
      //...
    }

修改以后的代码： 


    public class NetworkTransporter {
        // 省略属性和其他方法...
        public Byte[] send(String address, Byte[] data) {
          //...
        }
    }
    
    
    public class HtmlDownloader {
      private NetworkTransporter transporter;//通过构造函数或IOC注入
      
      // HtmlDownloader这里也要有相应的修改
      public Html downloadHtml(String url) {
        HtmlRequest htmlRequest = new HtmlRequest(url);
        Byte[] rawHtml = transporter.send(
          htmlRequest.getAddress(), htmlRequest.getContent().getBytes());
        return new Html(rawHtml);
      }
    }
    
    
    public class Document {
      private Html html;
      private String url;
      
      public Document(String url, Html html) {
        this.html = html;
        this.url = url;
      }
      //...
    }
    
    // 通过一个工厂方法来创建Document
    public class DocumentFactory {
      private HtmlDownloader downloader;
      
      public DocumentFactory(HtmlDownloader downloader) {
        this.downloader = downloader;
      }
      
      public Document createDocument(String url) {
        Html html = downloader.downloadHtml(url);
        return new Document(url, html);
      }
    }