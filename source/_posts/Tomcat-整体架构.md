---
title: Tomcat - 整体架构
date: 2020-02-01 14:15:34
categories: Web
tags:
    - Tomcat
top:
---

# 1. 整体架构

+ Tomcat的核心功能
    + 处理Socket连接，负责网络字节流与Request，Response对象的转化
    + 加载和管理Servlet，以及具体的处理Request请求
+ socket 
    + 连接器 Connector 
    + 支持大量不同的I/O模型
        + NIO
            + 非阻塞I/O
        + NIO.2
            + 异步I/O 
        + APR
            + 采用Apache可移植运行库数显 
    + 支持不同的应用层协议
        + HTTP/1.1
        + AJP
            + 用于和Web服务器集成 
        + HTTP/2
+ servlet管理
    + 容器 Container  
        + 一个容器可能对接多个连接器  -- 为了实现对于多种I/O和应用层协议的支持
        + 连接器和容器的组合叫做Service组件
        + Tomcat内可能有多个Service，通过这种配置，可以实现通过不同的端口号来访问同一个机器上部署的不同应用。

![fig1.jpg](https://i.loli.net/2020/02/02/DQSx67LbjXzVANW.jpg)

# 2. 连接器

连接器对Servlet容器屏蔽了协议以及I/O模型，无论是何种协议，在容器中获取的都是一个标准的ServletRequest对象。

细化连接器的功能：
+ 监听网络端口
+ 接收网络连接请求
+ 读取网络请求字节流
+ 根据应用层协议，解析字节流，生成统一的Tomcat Request对象
+ 将Tomcat Request对象转成标准的ServletRequest
+ 调用Servlet容器，得到ServletResponse
+ 将ServletResponse转成Tomcat Response对象
+ 将Tomcat Response转成网络字节流
+ 将相应字节流写回浏览器


为了实现一个高内聚低耦合的系统，连接器需要在几个方面做针对性的设置
+ 网络通信
+ 应用层协议解析
+ Tomcat Request/ Response 与 ServletRequest/ Response的转化


![fig2.jpg](https://i.loli.net/2020/02/02/9aqpn2lF5UZs184.jpg)

## 2.1 Endpoint (part of protocal handler)

ProtocalHandler是用来处理网络连接和应用层协议的

提供字节流给Processor

Endpoint是通信端点，即通信监听的接口，是具体的Socket接收和发送的处理器，是对传输层的抽象，因此Endpoint是用来实现TCP/IP协议的。

Endpoint是一个接口，对应的抽象实现类是AbstractEndpoint

+ AbstractEndpoint 
    + Acceptor 
        + 用于监听Socket连接请求 
    + SocketProcessor
        + 用于处理接收到的Socket请求，实现Runnable接口，在run方法里调用协议处理组件Processor进行处理 

## 2.2 Processor (part of protocal handler)

提供Tomcat Request对象给Adapter，在这里Processor接收来自Endpoint的Socket，读取字节流解析成Tomcat Request和Response对象，并通过Adapter将其提交到容器处理，Processor是对应用层协议的抽象。

一个定义了请求的处理方法的接口，其抽象实现类AbstractProcessor对协议共有的属性进行封装，没有对方法进行实现。这些具体的实现有AjpProcessor  Http11Processor等，具体的实现类实现了特定协议的解析方法和请求处理方式。

![fig3.jpg](https://i.loli.net/2020/02/02/QuqNCEvhrJInGSL.jpg)

Endpoint接收Socket连接，生成一个SocketProcessor任务，提交到线程池去处理，SocketProcessor的run方法会调用Processor组件去解析应用层协议，Processor通过解析生成request对象之后，会调用Adapter的Service方法。

## 2.3 Adapter 

由于协议不同，客户端发过来的请求信息也不尽相同，Tomcat定义了自己的Request类来存放这些请求信息。ProtocolHandler接口负责解析请求并生成TomcatRequest类。将传入的Tomcat Request对象转成ServletRequest，再调用容器的service方法。

Adapter负责提供ServletRequest对象给容器

# 3. 容器

Tomcat当中，容器是用来装载Servlet的。

## 3.1 容器的层次结构

Tomcat设计了4种容器，分别是Engine, Host, Context, Wrapper。通过这种分层的架构来增加Servlet容器的灵活性。

![fig4.jpg](https://i.loli.net/2020/02/02/nopbJmwXEx2KM5a.jpg)
+ Engine
    + 表示引擎
    + 用来管理多个虚拟站点
    + 一个Service最多只能有一个Engine
+ Host
    + 虚拟主机 - 站点
    + 给Tomcat配置多个虚拟主机地址
    + 一个虚拟主机啊下又可以部署多个Web应用程序
+ Context 
    + 表示一个Web应用程序 
    + 一个Context(Web应用程序)里可以有多个Servlet
+ Wrapper 
    + 表示一个Servlet 

Tomcat的server.xml配置文件 
![fig5.jpg](https://i.loli.net/2020/02/02/ED2SiGWfKzevJ7c.jpg)

Tomcat通过组合模式来管理这些容器，所有容器组件都会实现Container接口，因此组合模式可以使得用户对但容器对象和组合容器对象的使用具有高度一致性。

    public interface Container extends Lifecycle {
        public void setName(String name);
        public Container getParent();
        public void setParent(Container container);
        public void addChild(Container child);
        public void removeChild(Container child);
        public Container findChild(String name);
    }


## 3.2 请求是如何定位Servlet的 - Mapper

Tomcat使用Mapper组件来完成这个任务。

+ Mapper功能
    + 将用户请求的URL定位到一个Servlet
+ Mapper工作原理
    + Mapper组件保存web应用的配置信息 - 容器组件与访问路径的映射关系
        + host容器里配置的域名
        + Context容器里的Web应用路径
        + Wrapper容器里Servlet映射的路径
+ Mapper工作过程
    + 请求到来
    + Mapper组件解析请求URL里的域名和路径
    + 到自己保存的Map里面去查找
    + 定位到一个Servlet
    + 一个URL最终只会定位到一个Wrapper容器，即一个Servlet


+ E.g
    + 背景
        + 网购系统
            + 后台管理系统
            + 在线购物系统
        + 两个系统在同一个Tomcat上，为了隔离其访问域名，配置虚拟域名
    + Tomcat的功能
        + 创建一个Service组件和一个Engine容器组件
        + 在Engine容器下创建两个Host子容器
        + 每个Host下创建多个Context子容器

![fig6.jpg](https://i.loli.net/2020/02/02/yo2IVCkzvsY5gjw.jpg)

接着上面的例子，当用户访问一个URL，比如 `user.shopping.com:8080/order/buy`，Tomcat将这个URL定位到一个Servlet的过程如下：

+ 根据协议和端口号选定Service和Engine
    + HTTP或者AJP连接器都是由自己的默认端口号的
+ 根据域名选定Host
    + Mapper组件通过URL中的域名去查找响应的Host容器
+ 根据URL路径找到Context组件
    + 根据URL的路径来匹配相应的Web应用的路径
+ 根据URL路径找到Wrapper - Servlet
    + Context确定后，Mapper再根据web.xml中配置的Servlet映射路径来找到具体的Wrapper和Servlet

## 3.3 Pipeline - valve 
这整个层层递进的调用过程使用的是Pipeline-Value管道。

+ Pipeline-valve
    + 责任链模式
    + 在请求处理过程中有很多处理者依次对请求进行处理，每个处理着负责做自己的相应的处理，然后一个个传递给下一个处理者


![fig7.jpg](https://i.loli.net/2020/02/02/IMyaJTVZlbgmWEc.jpg)

    public interface Valve {
      public Valve getNext();
      public void setNext(Valve valve);
      public void invoke(Request request, Response response)
    }

    public interface Pipeline extends Container {
      public void addValve(Valve valve);
      public Valve getBasic();
      public void setBasic(Valve valve);
      public Valve getFirst();
    }


Pipelin中维护了Valve的链表，整个调用链的执行是被valve来完成的，valve完成自己的处理以后，就会调用getNext.invoke来触发下一个Valve调用

不同的容器的pipeline之间的触发，上一层的最后一个valve负责调用下一层的第一个valve，整个流程如下：

Wrapper容器的最后一个Valve会创建一个Filter链，并调用doFilter方法，最终会调用Servlet的service方法。

+ Valve和Filter
    + valve是tomcat的私有机制
    + Filter是在servlet级别的，是公有标准
    + valve工作在web容器级别，拦截所有应用的请求
    + Servlet Filter工作在应用级别，只能拦截某个Web应用的所有请求。

# 4. Tomcat一键启停

首先复习下Tomcat各个组件之间的关系。

![fig8.png](https://i.loli.net/2020/02/02/qaU4DIjHgezVodv.png)


+ 为了使得一个系统能够对外提供服务，我们需要创建、组装并启动这些组件
+ 在服务停止的时候，还需要释放资源，销毁这些组件
+ Tomcat需要动态地管理这些组件的生命周期


+ 大组件管理小组件
    + 需要先启动子组件，再启动父组件，子组件需要被注入到父组件当中去
+ 请求的处理过程是由外层组件来驱动的
    + 先创建内层组件，再创建外层组件，内层组件需要被注入到外层组件当中


## 4.1 LifeCycle接口

+ 不变的点
    + 每个组件都要经历创建、初始化、启动这几个过程
    + 创建Lifecycle接口
        + init
        + start 
        + stop
        + destroy
+ 变化的点
    + 每个具体组件的初始化方法 
    + addLifecycleListener
    + removeLifecycleListner


![fig9.png](https://i.loli.net/2020/02/02/hBfJx9GELY1NWgq.png)

+ 将公有逻辑抽象出来，放到抽象类当中，所以UML图就变成了图10的样子

![fig10.png](https://i.loli.net/2020/02/02/vMpFBLw6mrQASa5.png)

基类的具体实现如下: 

    @Override
    public final synchronized void init() throws LifecycleException {
        //1. 状态检查
        if (!state.equals(LifecycleState.NEW)) {
            invalidTransition(Lifecycle.BEFORE_INIT_EVENT);
        }
    
        try {
            //2. 触发 INITIALIZING 事件的监听器
            setStateInternal(LifecycleState.INITIALIZING, null, false);
            
            //3. 调用具体子类的初始化方法
            initInternal();
            
            //4. 触发 INITIALIZED 事件的监听器
            setStateInternal(LifecycleState.INITIALIZED, null, false);
        } catch (Throwable t) {
          ...
        }
    }


## 4.2 生命周期管理总体类图

![fig11.png](https://i.loli.net/2020/02/02/4r7yVeiPqAzhYFI.png)

# 5. Tomcat运行过程 

我们可以通过Tomcat的/bin目录下的脚本startup.sh来启动Tomcat，整个流程如下: 

![fig12.png](https://i.loli.net/2020/02/02/Par89z2MyQj7KYH.png)

+ 启动JVM，运行Tomcat启动类Bootstrap
+ Bootstrap类会初始化Tomcat的类加载器，并且创建Catalina
+ Catalina是一个启动类，通过解析server.xml，会创建相应的组件，并调用Server的start方法
+ server组件被用来管理Service组件，会负责调用Service的start方法
+ Service组件被用来管理连接器和顶层容器Engine，因此其会调用连接器和Engine的start方法

## 5.1 Catalina

主要任务是创建Server，通过解析server.xml文件，将server.xml里配置的各种组件一一创建出来，接着调用Server组件的init方法和start方法，以此启动整个Tomcat。

Tomcat还需要处理各种异常情况，比如强制关闭的处理，Catalina在JVM当中注册了一个关闭钩子。

    public void start() {
        //1. 如果持有的 Server 实例为空，就解析 server.xml 创建出来
        if (getServer() == null) {
            load();
        }
        //2. 如果创建失败，报错退出
        if (getServer() == null) {
            log.fatal(sm.getString("catalina.noServer"));
            return;
        }
    
        //3. 启动 Server
        try {
            getServer().start();
        } catch (LifecycleException e) {
            return;
        }
    
        // 创建并注册关闭钩子
        if (useShutdownHook) {
            if (shutdownHook == null) {
                shutdownHook = new CatalinaShutdownHook();
            }
            Runtime.getRuntime().addShutdownHook(shutdownHook);
        }
    
        // 用 await 方法监听停止请求
        if (await) {
            await();
            stop();
        }
    }

    // catalina的关闭钩子执行了Server的stop方法，会释放和清理所有的资源。
    protected class CatalinaShutdownHook extends Thread {
    
        @Override
        public void run() {
            try {
                if (getServer() != null) {
                    Catalina.this.stop();
                }
            } catch (Throwable ex) {
               ...
            }
        }
    }
    
## 5.2 Server组件

其具体实现类是StandardServer，继承了LifecycleBase，生命周期被统一管理，子组件是Service，因此还需要管理Service的生命周期，在启动时调用Service组件的启动方法，在停止时调用其停止方法

Server维护多个Service组件，以数组来保存

    @Override
    public void addService(Service service) {
    
        service.setServer(this);
    
        synchronized (servicesLock) {
            // 创建一个长度 +1 的新数组
            Service results[] = new Service[services.length + 1];
            
            // 将老的数据复制过去
            System.arraycopy(services, 0, results, 0, services.length);
            results[services.length] = service;
            services = results;
    
            // 启动 Service 组件
            if (getState().isAvailable()) {
                try {
                    service.start();
                } catch (LifecycleException e) {
                    // Ignore
                }
            }
    
            // 触发监听事件
            support.firePropertyChange("service", null, service);
        }
    }

节省内存空间的一种方式

Server组件会启动一个Socket来监听停止端口，在Catalina启动的时候，会调用Server的await方法，这个地方实际上是创建了一个Socket来监听一个端口，并在这个死循环里接收Socket的连接请求，如果有新的连接到来就建立连接，然后从Socket中读取数据；如果读到的数据是停止命令SHUTDOWN，就退出循环，进入stop流程

## 5.3 Service组件

具体实现类是StandardService

    public class StandardService extends LifecycleBase implements Service {
        // 名字
        private String name = null;
        
        //Server 实例
        private Server server = null;
    
        // 连接器数组
        protected Connector connectors[] = new Connector[0];
        private final Object connectorsLock = new Object();
    
        // 对应的 Engine 容器
        private Engine engine = null;
        
        // 映射器及其监听器
        protected final Mapper mapper = new Mapper();
        protected final MapperListener mapperListener = new MapperListener(this);


MapperListener 是为了支持热部署，当Web应用的部署发生变化时，Mapper中的映射信息也要跟着变化，MapperListener是一个监听器，它监听容器的变化，并将信息更新到Mapper当中。

    protected void startInternal() throws LifecycleException {
    
        //1. 触发启动监听器
        setState(LifecycleState.STARTING);
    
        //2. 先启动 Engine，Engine 会启动它子容器
        if (engine != null) {
            synchronized (engine) {
                engine.start();
            }
        }
        
        //3. 再启动 Mapper 监听器
        mapperListener.start();
    
        //4. 最后启动连接器，连接器会启动它子组件，比如 Endpoint
        synchronized (connectorsLock) {
            for (Connector connector: connectors) {
                if (connector.getState() != LifecycleState.FAILED) {
                    connector.start();
                }
            }
        }
    }


## 5.4 Engine

    public class StandardEngine extends ContainerBase implements Engine {
    }


Engine子容器是Host，其持有一个Host容器数组

    protected final HashMap<String, Container> children = new HashMap<>();

ContainerBase用HashMap保存了它的子容器，并且ContainerBase还实现了子容器的增删改查

    for (int i = 0; i < children.length; i++) {
       results.add(startStopExecutor.submit(new StartChild(children[i])));
    }

Engine通过将请求转发给某一个Host子容器来对请求进行处理，Engine的基础阀定义如下：

    final class StandardEngineValve extends ValveBase {
    
        public final void invoke(Request request, Response response)
          throws IOException, ServletException {
      
          // 拿到请求中的 Host 容器
          Host host = request.getHost();
          if (host == null) {
              return;
          }
      
          // 调用 Host 容器中的 Pipeline 中的第一个 Valve
          host.getPipeline().getFirst().invoke(request, response);
      }
      
    }

