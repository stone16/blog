---
title: IOC容器和Dependency Injection模式
date: 2020-02-02 13:59:46
categories: BackEnd
tags:
    - IOC
    - Dependency Injection
    - BackEnd
    - Java
top:
---
Martin Fowler的文章，在文中深入探索控制反转的的工作原理，给它一个更能描述其特点的名字——”依赖注入”（Dependency Injection），并将其与”服务定位器”（Service Locator）模式作一个比较。探讨了异同。最最重要的，也是每个程序员都应该注意的是：应该将服务的配置和应用程序内部对服务的使用分离开。这也是控制反转以及服务定位器一直在做的。


# 1. 问题
J2EE开发者常遇到的一个问题就是***如何组装不同的程序元素***：如果web控制器体系结构和数据库接口是由不同的团队所开发的，彼此几乎一无所知，你应该如何让它们配合工作？很多框架尝试过解决这个问题，有几个框架索性朝这个方向发展，提供了更通用的”组装各层组件”的方案。这样的框架通常被称为”轻量级容器”，PicoContainer和Spring都在此列中。

## 2.实例

有一个提供一份电影清单的组件，清单上列出有一位特定导演执导的影片

    class MovieLister...

    public Movie[] moviesDirectedBy(String arg)
    {
        List allMovies = finder.findAll();
        for (Iterator it = allMovies.iterator(); it.hasNext();)
        {
            Movie movie = (Movie) it.next();
            if (!movie.getDirector().equals(arg))
            {
                it.remove();
            }

        }
        return (Movie[]) allMovies.toArray(new Movie[allMovies.size()]);
    }


这个功能的实现极其简单：moviesDirectedBy方法首先请求finder（影片搜寻者）对象（我们稍后会谈到这个对象）返回后者所知道的所有影片，然后遍历finder对象返回的清单，并返回其中由特定的某个导演执导的影片。非常简单，不过不必担心，这只是整个例子的脚手架罢了。***我们真正想要考察的是finder对象，或者说，如何将MovieLister对象与特定的finder对象连接起来***。为什么我们对这个问题特别感兴趣？因为***我希望上面这个漂亮的moviesDirectedBy方法完全不依赖于影片的实际存储方式。***所以，这个方法只能引用一个finder对象，而finder对象则必须知道如何对findAll 方法作出回应。为了帮助读者更清楚地理解，我给finder定义了一个接口：

    public interface MovieFinder
    {
        List findAll();
    }

现在，两个对象之间没有什么耦合关系。但是，当我要实际寻找影片时，就必须涉及到MovieFinder的某个具体子类。在这里，*我把涉及具体子类的代码放在MovieLister类的构造函数中*。

    class MovieLister...
    private MovieFinder finder;
    public MovieLister()
    {
        finder = new ColonDelimitedMovieFinder("movies1.txt");
    }

这个实现类的名字就说明：我将要从一个逗号分隔的文本文件中获得影片列表。你不必操心具体的实现细节，只要设想这样一个实现类就可以了。如果这个类只由我自己使用，一切都没问题。但是，如果我的朋友叹服于这个精彩的功能，也想使用我的程序，那又会怎么样呢？如果他们也把影片清单保存在一个逗号分隔的文本文件中，并且也把这个文件命名为” movie1.txt “，那么一切还是没问题。如果他们只是给这个文件改改名，我也可以从一个配置文件获得文件名，这也很容易。**但是，如果他们用完全不同的方式——例如SQL 数据库、XML 文件、web service，或者另一种格式的文本文件——来存储影片清单呢？**在这种情况下，我们需要用另一个类来获取数据。由于已经定义了MovieFinder接口，我可以不用修改moviesDirectedBy方法。但是，我***仍然需要通过某种途径获得合适的MovieFinder实现类的实例***。


![figure1.gif](https://i.loli.net/2020/02/03/B2Nf9MX6VIL4oKP.gif)

MovieLister类既依赖于MovieFinder接口，也依赖于具体的实现类。我们当然希望MovieLister类只依赖于接口，但我们要如何获得一个MovieFinder子类的实例呢？

在Patterns of Enterprise Application Architecture一书中，我们把这种情况称为**插件（plugin）**：MovieFinder的实现类**不是在编译期连入程序之中的**，因为我并不知道我的朋友会使用哪个实现类。我们希望MovieLister类能够与MovieFinder的任何实现类配合工作，并且允许**在运行期插入具体的实现类**，插入动作完全脱离我（原作者）的控制。这里的问题就是：**如何设计这个连接过程，使MovieLister类在不知道实现类细节的前提下与其实例协同工作。**

将这个例子推而广之，在一个真实的系统中，我们可能有数十个服务和组件。在任何时候，我们总可以对使用组件的情形加以抽象，**通过接口与具体的组件交流**（如果组件并没有设计一个接口，也可以通过适配器与之交流）。但是，如果我们希望以不同的方式部署这个系统，就需要用插件机制来处理服务之间的交互过程，这样我们才可能在不同的部署方案中使用不同的实现。所以，现在的核心问题就是：**如何将这些插件组合成一个应用程序？这正是新生的轻量级容器所面临的主要问题，而它们解决这个问题的手段无一例外地是控制反转（Inversion of Control）模式**。

# 3. 控制反转

几位轻量级容器的作者曾骄傲地对我说：这些容器非常有用，因为它们实现了控制反转。这样的说辞让我深感迷惑：控制反转是框架所共有的特征，如果仅仅因为使用了控制反转就认为这些轻量级容器与众不同，就好象在说我的轿车是与众不同的，因为它有四个轮子。

问题的关键在于：它们反转了哪方面的控制？我第一次接触到的控制反转针对的是用户界面的主控权。早期的用户界面是完全由应用程序来控制的，你预先设计一系列命令，例如输入姓名、输入地址等，应用程序逐条输出提示信息，并取回用户的响应。而在图形用户界面环境下，UI框架将负责执行一个主循环，你的应用程序只需为屏幕的各个区域提供事件处理函数即可。在这里，程序的主控权发生了反转：从应用程序移到了框架。对于这些新生的容器，它们反转的是如何定位插件的具体实现。在前面那个简单的例子中，MovieLister类负责定位MovieFinder的具体实现——它直接实例化后者的一个子类。这样一来，MovieFinder也就不成其为一个插件了，因为它并不是在运行期插入应用程序中的。而这些轻量级容器则使用了更为灵活的办法，**只要插件遵循一定的规则，一个独立的组装模块就能够将插件的具体实现注射到应用程序中。**因此，我想我们需要给这个模式起一个更能说明其特点的名字——”控制反转”这个名字太泛了，常常让人有些迷惑。与多位IoC 爱好者讨论之后，我们决定将这个模式叫做”依赖注入”（Dependency Injection）。

下面，我将开始介绍Dependency Injection模式的几种不同形式。不过，在此之前，我要首先指出：要消除应用程序对插件实现的依赖，依赖注入并不是唯一的选择，你也可以用ServiceLocator模式获得同样的效果。介绍完Dependency Injection模式之后，我也会谈到ServiceLocator 模式。

## 3.1 依赖注入的几种形式

Dependency Injection模式的基本思想是：用一个单独的对象（装配器）来获得MovieFinder的一个合适的实现，并将其实例赋给MovieLister类的一个字段。这样一来，我们就得到了图2所示的依赖图：

![figure2.gif](https://i.loli.net/2020/02/03/wySa8WINcA2Xp49.gif)

### 3.1.1 构造函数注入
这里使用PicoContainer，一个轻量级容器来完成依赖注入。

PicoContainer通过构造函数来判断如何将MovieFinder实例注入MovieLister 类。因此，MovieLister类必须声明一个构造函数，并在其中包含所有需要注入的元素：

    class MovieLister...
        public MovieLister(MovieFinder finder)
        {
            this.finder = finder;
        }
MovieFinder实例本身也将由PicoContainer来管理，因此文本文件的名字也可以由容器注入：

    class ColonMovieFinder...
        public ColonMovieFinder(String filename)
        {
            this.filename = filename;
        }
        
随后，需要告诉PicoContainer：各个接口分别与哪个实现类关联、将哪个字符串注入MovieFinder组件。

    private MutablePicoContainer configureContainer()
    {
        MutablePicoContainer pico = new DefaultPicoContainer();
        Parameter[] finderParams = {newConstantParameter("movies1.txt")};
        pico.registerComponentImplementation(MovieFinder.class,ColonMovieFinder.class, finderParams);
        pico.registerComponentImplementation(MovieLister.class);
        return pico;
    }
    
这段配置代码通常位于另一个类。对于我们这个例子，使用我的MovieLister 类的朋友需要在自己的设置类中编写合适的配置代码。当然，还可以将这些配置信息放在一个单独的配置文件中，这也是一种常见的做法。你可以编写一个类来读取配置文件，然后对容器进行合适的设置。尽管PicoContainer本身并不包含这项功能，但另一个与它关系紧密的项目NanoContainer提供了一些包装，允许开发者使用XML配置文件保存配置信息。NanoContainer能够解析XML文件，并对底下的PicoContainer进行配置。这个项目的哲学观念就是：将配置文件的格式与底下的配置机制分离开。

    public void testWithPico()
    {
        MutablePicoContainer pico = configureContainer();
        MovieLister lister = (MovieLister)pico.getComponentInstance(MovieLister.class);
        Movie[] movies = lister.moviesDirectedBy("Sergio Leone");
        assertEquals("Once Upon a Time in the West",movies[0].getTitle());
    }
    

### 3.1.2 设值方法注入

Spring 框架是一个用途广泛的企业级Java 开发框架，其中包括了针对事务、持久化框架、web应用开发和JDBC等常用功能的抽象。和PicoContainer一样，它也同时支持构造函数注入和设值方法注入，但该项目的开发者更推荐使用设值方法注入——恰好适合这个例子。为了让MovieLister类接受注入，我需要为它定义一个设值方法，该方法接受类型为MovieFinder的参数：

    class MovieLister...
    private MovieFinder finder;
    public void setFinder(MovieFinder finder)
    {
        this.finder = finder;
    }

类似地，在MovieFinder的实现类中，我也定义了一个设值方法，接受类型为String 的参数：

    class ColonMovieFinder...
        public void setFilename(String filename)
        {
            this.filename = filename;
        }

第三步是设定配置文件。Spring 支持多种配置方式，你可以通过XML 文件进行配置，也可以直接在代码中配置。不过，XML 文件是比较理想的配置方式。

    <beans>
        <bean id="MovieLister" class="spring.MovieLister">
            <property name="finder">
                <ref local="MovieFinder"/>
            </property>
        </bean>
        <bean id="MovieFinder" class="spring.ColonMovieFinder">
            <property name="filename">
                <value>movies1.txt</value>
            </property>
        </bean>
    </beans>

测试代码：

    public void testWithSpring() throws Exception
    {
        ApplicationContext ctx = newFileSystemXmlApplicationContext("spring.xml");
        MovieLister lister = (MovieLister) ctx.getBean("MovieLister");
        Movie[] movies = lister.moviesDirectedBy("Sergio Leone");
        assertEquals("Once Upon a Time in the West",movies[0].getTitle());
    }
    
    
### 3.1.3 接口注入

除了前面两种注入技术，还可以在接口中定义需要注入的信息，并通过接口完成注入。Avalon框架就使用了类似的技术。在这里，我首先用简单的范例代码说明它的用法，后面还会有更深入的讨论。首先，我需要定义一个接口，**组件的注入将通过这个接口进行**。在本例中，这个接口的用途是将一个MovieFinder实例注入继承了该接口的对象。

    public interface InjectFinder
    {
        void injectFinder(MovieFinder finder);
    }

这个接口应该由提供MovieFinder接口的人一并提供。任何想要使用MovieFinder实例的类（例如MovieLister类）都必须实现这个接口。

    class MovieLister implements InjectFinder...
        public void injectFinder(MovieFinder finder)
        {
            this.finder = finder;
        }
        
然后，我使用类似的方法将文件名注入MovieFinder的实现类：

    public interface InjectFilename
    {
        void injectFilename (String filename);
    }
    
    class ColonMovieFinder implements MovieFinder, InjectFilename...
        public void injectFilename(String filename)
        {
            this.filename = filename;
        }
        
现在，还需要用一些配置代码将所有的组件实现装配起来。简单起见，我直接在代码中完成配置，并将配置好的MovieLister 对象保存在名为lister的字段中：

    class IfaceTester...
        private MovieLister lister;
        private void configureLister()
        {
            ColonMovieFinder finder = new ColonMovieFinder();
            finder.injectFilename("movies1.txt");
            lister = new MovieLister();
            lister.injectFinder(finder);
        }
测试代码：

    class IfaceTester...
    public void testIface()
    {
        configureLister();
        Movie[] movies = lister.moviesDirectedBy("Sergio Leone");
        assertEquals("Once Upon a Time in the West",movies[0].getTitle());
    }
    
## 3.2 构造函数注入 vs. 设值方法注入

在组合服务时，你总得遵循一定的约定，才可能将所有东西拼装起来。**依赖注入的优点主要在于：它只需要非常简单的约定——至少对于构造函数注入和设值方法注入来说是这样**。相比于这两者，接口注入的侵略性要强得多，比起Service Locator模式的优势也不那么明显。所以，如果你想要提供一个组件给多个使用者，构造函数注入和设值方法注入看起来很有吸引力。你不必在组件中加入什么希奇古怪的东西，注入器可以相当轻松地把所有东西配置起来。

设值函数注入和构造函数注入之间的选择相当有趣，因为它折射出面向对象编程的一些更普遍的问题：应该在哪里填充对象的字段，构造函数还是设值方法？

一直以来，我首选的做法是尽量在构造阶段就创建完整、合法的对象——也就是说，在构造函数中填充对象字段。这样做的好处可以追溯到Kent Beck在Smalltalk Best Practice Patterns一书中介绍的两个模式：Constructor Method和Constructor Parameter Method。带有参数的构造函数可以明确地告诉你如何创建一个合法的对象。如果创建合法对象的方式不止一种，你还可以提供多个构造函数，以说明不同的组合方式。

构造函数初始化的另一个好处是：你可以隐藏任何不可变的字段——只要不为它提供设值方法就行了。我认为这很重要：如果某个字段是不应该被改变的，没有针对该字段的设值方法就很清楚地说明了这一点。如果你通过设值方法完成初始化，暴露出来的设值方法很可能成为你心头永远的痛。（实际上，在这种时候我更愿意回避通常的设值方法约定，而是使用诸如initFoo之类的方法名，以表明该方法只应该在对象创建之初调用。）

不过，世事总有例外。如果参数太多，构造函数会显得凌乱不堪，特别是对于不支持关键字参数的语言更是如此。的确，如果构造函数参数列表太长，通常标志着对象太过繁忙，理应将其拆分成几个对象，但有些时候也确实需要那么多的参数。如果有不止一种的方式可以构造一个合法的对象，也很难通过构造函数描述这一信息，因为构造函数之间只能通过参数的个数和类型加以区分。这就是Factory Method模式适用的场合了，工厂方法**可以借助多个私有构造函数和设值方法的组合来完成自己的任务**。经典Factory Method模式的问题在于：它们往往以静态方法的形式出现，你无法在接口中声明它们。你可以创建一个工厂类，但那又变成另一个服务实体了。工厂服务是一种不错的技巧，但你仍然需要以某种方式实例化这个工厂对象，问题仍然没有解决。

如果要传入的参数是像字符串这样的简单类型，构造函数注入也会带来一些麻烦。使用设值方法注入时，你可以在每个设值方法的名字中说明参数的用途；而使用构造函数注入时，你只能靠参数的位置来决定每个参数的作用，而记住参数的正确位置显然要困难得多。

如果对象有多个构造函数，对象之间又存在继承关系，事情就会变得特别讨厌。为了让所有东西都正确地初始化，你必须将对子类构造函数的调用转发给超类的构造函数，然后处理自己的参数。这可能造成构造函数规模的进一步膨胀。

尽管有这些缺陷，但我仍然建议你首先考虑构造函数注入。不过，一旦前面提到的问题真的成了问题，你就应该准备转为使用设值方法注入。

## 3.3 代码配置 vs 配置文件
另一个问题相对独立，但也经常与其他问题牵涉在一起：如何配置服务的组装，通过配置文件还是直接编码组装？对于大多数需要在多处部署的应用程序来说，一个单独的配置文件会更合适。配置文件几乎都是XML 文件，XML 也的确很适合这一用途。不过，有些时候直接在程序代码中实现装配会更简单。譬如一个简单的应用程序，也没有很多部署上的变化，这时用几句代码来配置就比XML 文件要清晰得多。

与之相对的，有时应用程序的组装非常复杂，涉及大量的条件步骤。一旦编程语言中的配置逻辑开始变得复杂，你就应该用一种合适的语言来描述配置信息，使程序逻辑变得更清晰。然后，**你可以编写一个构造器（builder）类来完成装配工作**。如果使用构造器的情景不止一种，你可以提供多个构造器类，然后通过一个简单的配置文件在它们之间选择。

我常常发现，人们太急于定义配置文件。编程语言通常会提供简捷而强大的配置管理机制，现代编程语言也可以将程序编译成小的模块，并将其插入大型系统中。如果编译过程会很费力，脚本语言也可以在这方面提供帮助。通常认为，配置文件不应该用编程语言来编写，因为它们需要能够被不懂编程的系统管理人员编辑。但是，这种情况出现的几率有多大呢？我们真的希望不懂编程的系统管理人员来改变一个复杂的服务器端应用程序的事务隔离等级吗？只有在非常简单的时候，非编程语言的配置文件才有最好的效果。如果配置信息开始变得复杂，就应该考虑选择一种合适的编程语言来编写配置文件。

在Java 世界里，我们听到了来自配置文件的不和谐音——每个组件都有它自己的配置文件，而且格式还各不相同。如果你要使用一打这样的组件，你就得维护一打的配置文件，那会很快让你烦死。

在这里，我的建议是：始终提供一种标准的配置方式，使程序员能够通过同一个编程接口轻松地完成配置工作。至于其他的配置文件，仅仅把它们当作一种可选的功能。借助这个编程接口，开发者可以轻松地管理配置文件。如果你编写了一个组件，则可以由组件的使用者来选择如何管理配置信息：使用你的编程接口、直接操作配置文件格式，或者定义他们自己的配置文件格式，并将其与你的编程接口相结合。

## 3.4 分离配置和使用

所有这一切的关键在于：***服务的配置应该与使用分开***。实际上，这是一个基本的设计原则——分离接口与实现。在面向对象程序里，我们在一个地方用条件逻辑来决定具体实例化哪一个类，以后的条件分支都由多态来实现，而不是继续重复前面的条件逻辑，这就是分离接口与实现的原则。

如果对于一段代码而言，接口与实现的分离还只是有用的话，那么当你需要使用外部元素（例如组件和服务）时，它就是生死攸关的大事。这里的第一个问题是：你是否希望将选择具体实现类的决策推迟到部署阶段。如果是，那么你需要使用插入技术。使用了插入技术之后，插件的装配原则上是与应用程序的其余部分分开的，这样你就可以轻松地针对不同的部署替换不同的配置。这种配置机制可以通过服务定位器来实现（Service Locator模式），也可以借助依赖注入直接完成（Dependency Injection 模式）。

# 4. 结论与思考

在时下流行的轻量级容器都使用了一个共同的模式来组装应用程序所需的服务，我把这个模式称为Dependency Injection，它可以有效地替代Service Locator模式。在开发应用程序时，两者不相上下，但我认为Service Locator模式略有优势，因为它的行为方式更为直观。但是，如果你开发的组件要交给多个应用程序去使用，那么Dependency Injection模式会是更好的选择。

如果你决定使用Dependency Injection模式，这里还有几种不同的风格可供选择。我建议你首先考虑构造函数注入；如果遇到了某些特定的问题，再改用设值方法注入。如果你要选择一个容器，在其之上进行开发，我建议你选择同时支持这两种注入方式的容器。

Service Locator 模式和Dependency Injection 模式之间的选择并是最重要的，更重要的是：应该将服务的配置和应用程序内部对服务的使用分离开。


[1.[Inversion of Control Containers and the Dependency Injection pattern]](https://martinfowler.com/articles/injection.html)