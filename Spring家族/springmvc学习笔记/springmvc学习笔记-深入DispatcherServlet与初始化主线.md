# springmvc学习笔记(5)-深入DispatcherServlet与初始化主线

## DispatcherServlet的体系结构

通过不同的角度来观察DispatcherServlet会得到不同的结论。

我们在这里选取了三个不同的角度：

- 运行特性
- 继承结构
- 数据结构

### 运行主线

从DispatcherServlet所实现的接口来看，DispatcherServlet的核心本质：是一个Servlet。这个结论似乎很幼稚，不过这个幼稚的结论却蕴含了一个对整个框架都至关重要的内在原则：Servlet可以根据其特性进行运行主线的划分。

根据Servlet规范的定义，Servlet中的两大核心方法init方法和service方法，它们的运行时间和触发条件都截然不同：

**init方法**

在整个系统启动时运行，且只运行一次。因此，在init方法中我们往往会对整个应用程序进行初始化操作。这些初始化操作可能包括对容器（WebApplicationContext）的初始化、组件和外部资源的初始化等等。

**service方法**

在整个系统运行的过程中处于侦听模式，侦听并处理所有的Web请求。因此，在service及其相关方法中，我们看到的则是对Http请求的处理流程。因而在这里，Servlet的这一特性就被SpringMVC用于对不同的逻辑职责加以划分，从而形成两条互不相关的逻辑运行主线：

- 初始化主线 —— 负责对SpringMVC的运行要素进行初始化
- Http请求处理主线 —— 负责对SpringMVC中的组件进行逻辑调度完成对Http请求的处理

**注**：SpringMVC运行主线的划分依据是Servlet对象中不同方法的生命周期。事实上，几乎所有的MVC都是以此为依据来进行运行主线的划分。这进一步可以证明所有的MVC框架的核心基础还是**Servlet规范**，而设计理念的差异也导致了不同的框架走向了完全不同的发展道路。

### 继承结构

DispatcherServlet的继承结构：

![dispatecherservlet](http://static.zybuluo.com/vonzhou/v3ihj7s2lqr0r40mt1jnlmc1/1.JPG)

在这个继承结构中，我们可以看到DispatcherServlet在其继承树中包含了2个Spring的支持类：

- HttpServletBean
- FrameworkServlet。

我们分别来讨论一下这两个Spring的支持类在这里所起到的作用。

(a) HttpServletBean是Spring对于Servlet最低层次的抽象。

在这一层抽象中，Spring会将这个Servlet视作是一个Spring的bean，并将init-param中的值作为bean的属性注入进来：

```
public final void init() throws ServletException { 
    if (logger.isDebugEnabled()) { 
        logger.debug("Initializing servlet '" + getServletName() + "'"); 
    } 
 
    // Set bean properties from init parameters. 
    try { 
        PropertyValues pvs = new ServletConfigPropertyValues(getServletConfig(), this.requiredProperties); 
        BeanWrapper bw = PropertyAccessorFactory.forBeanPropertyAccess(this); 
        ResourceLoader resourceLoader = new ServletContextResourceLoader(getServletContext()); 
        bw.registerCustomEditor(Resource.class, new ResourceEditor(resourceLoader, this.environment)); 
        initBeanWrapper(bw); 
        bw.setPropertyValues(pvs, true); 
    } 
    catch (BeansException ex) { 
        logger.error("Failed to set bean properties on servlet '" + getServletName() + "'", ex); 
        throw ex; 
    } 
 
    // Let subclasses do whatever initialization they like. 
    initServletBean(); 
 
    if (logger.isDebugEnabled()) { 
        logger.debug("Servlet '" + getServletName() + "' configured successfully"); 
    } 
} 
```

从源码中，我们可以看到HttpServletBean利用了Servlet的init方法的执行特性，将一个普通的Servlet与Spring的容器联系在了一起。在这其中起到核心作用的代码是：BeanWrapper bw = PropertyAccessorFactory.forBeanPropertyAccess(this);将当前的这个Servlet类转化为一个BeanWrapper，从而能够以Spring的方式来对init-param的值进行注入。BeanWrapper的相关知识属于Spring Framework的内容。

FrameworkServlet则是在HttpServletBean的基础之上的进一步抽象。通过FrameworkServlet真正初始化了一个Spring的容器（WebApplicationContext），并引入到Servlet对象之中：

```
protected final void initServletBean() throws ServletException {  
    getServletContext().log("Initializing Spring FrameworkServlet '" + getServletName() + "'");  
    if (this.logger.isInfoEnabled()) {  
        this.logger.info("FrameworkServlet '" + getServletName() + "': initialization started");  
    }  
    long startTime = System.currentTimeMillis();  
  
    try {  
        this.webApplicationContext = initWebApplicationContext();  
        initFrameworkServlet();  
    } catch (ServletException ex) {  
        this.logger.error("Context initialization failed", ex);  
        throw ex;  
    } catch (RuntimeException ex) {  
        this.logger.error("Context initialization failed", ex);  
        throw ex;  
    }  
  
    if (this.logger.isInfoEnabled()) {  
        long elapsedTime = System.currentTimeMillis() - startTime;  
        this.logger.info("FrameworkServlet '" + getServletName() + "': initialization completed in " +  
                elapsedTime + " ms");  
    }  
}  
```

上面的这段代码就是FrameworkServlet初始化的核心代码。从中我们可以看到这个FrameworkServlet将调用其内部的方法initWebApplicationContext()对Spring的容器（WebApplicationContext）进行初始化。同时，FrameworkServlet还暴露了与之通讯的结构可供子类调用：

```
public abstract class FrameworkServlet extends HttpServletBean { 
 
    /** WebApplicationContext for this servlet */ 
    private WebApplicationContext webApplicationContext; 
 
        // 这里省略了其他所有的代码 
 
    /**
     * Return this servlet's WebApplicationContext.
     */ 
    public final WebApplicationContext getWebApplicationContext() { 
        return this.webApplicationContext; 
    } 
} 
```

我们在这里暂且不对Spring容器（WebApplicationContext）的初始化过程详加探查，稍后我们会讨论一些容器WebApplicationContext初始化过程中的配置选项。不过读者可以在这里体会到：FrameworkServlet在其内部初始化了一个Spring的容器（WebApplicationContext）并暴露了相关的操作接口，因而继承自FrameworkServlet的DispatcherServlet，也就直接拥有了与WebApplicationContext进行通信的能力。

通过对DispatcherServlet继承结构的研究，我们可以明确：

downpour 写道**结论** DispatcherServlet的继承体系架起了DispatcherServlet与Spring容器进行沟通的桥梁。

### 数据结构

DispatcherServlet的数据结构：

![数据结构](http://dl.iteye.com/upload/attachment/0062/6933/6ea52d3d-fe87-3579-976e-d6edd1f0deb3.png)

我们可以把在上面这张图中所构成DispatcherServlet的数据结构主要分为两类（我们在这里用一根分割线将其分割开来）：

- 配置参数 —— 控制SpringMVC组件的初始化行为方式
- 核心组件 —— SpringMVC的核心逻辑处理组件

可以看到，这两类数据结构都与SpringMVC中的核心要素组件有关。因此，我们可以得出这样一个结论：

downpour 写道**结论** 组件是整个DispatcherServlet的灵魂所在：它不仅是初始化主线中的初始化对象，同样也是Http请求处理主线中的逻辑调度载体。

**注**：我们可以看到被我们划为配置参数的那些变量都是boolean类型的，它们将在DispatcherServlet的初始化主线中起到一定的作用，我们在之后会使用源码进行说明。而这些boolean值可以通过web.xml中的init-param值进行设定覆盖（这是由HttpServletBean的特性带来的）。

## SpringMVC的运行体系

DispatcherServlet继承结构和数据结构，实际上表述的是DispatcherServlet与另外两大要素之间的关系：

- 继承结构 —— DispatcherServlet与Spring容器（WebApplicationContext）之间的关系
- 数据结构 —— DispatcherServlet与组件之间的关系

所以，其实我们可以这么说，SpringMVC的整个运行体系，是由这三者共同构成的：

- DispatcherServlet
- 组件
- 容器

在这个运行体系中，DispatcherServlet是逻辑处理的调度中心，组件则是被调度的操作对象。而容器在这里所起到的作用，是协助DispatcherServlet更好地对组件进行管理。

三者之间的关系简单的描述：

![三者关系](http://dl.iteye.com/upload/attachment/0062/6980/17a6f37b-340c-31a7-9352-944f87bb6a01.png)



**注**：在这幅图中，我们除了看到在图的左半边DispatcherServlet、组件和容器这三者之间的调用关系以外，还可以看到SpringMVC的运行体系与其它运行体系之间存在着关系。

既然是三个元素之间的关系表述，我们必须以两两关系的形式进行归纳：

- DispatcherServlet - 容器 —— DispatcherServlet对容器进行初始化
- 容器 - 组件 —— 容器对组件进行全局管理
- DispatcherServlet - 组件 —— DispatcherServlet对组件进行逻辑调用

值得注意的是，在上面这幅图中，三大元素之间的两两关系其实表现得并不明显，尤其是“容器 - 组件”和“DispatcherServlet - 组件”之间的关系。这主要是由于Spring官方reference所给出的这幅图是一个静态的关系表述，如果从动态的观点来对整个过程加以审视，我们就不得不将SpringMVC的运行体系与之前所提到的运行主线联系在一起，看看这些元素在不同的逻辑主线中所起到的作用。

以下是DispatcherServlet的两条运行主线：

### DispatcherServlet的初始化主线

对于DispatcherServlet的初始化主线，我们首先应该明确几个基本观点：

- 初始化主线的驱动要素 —— servlet中的init方法
- 初始化主线的执行次序 —— HttpServletBean -> FrameworkServlet -> DispatcherServlet
- 初始化主线的操作对象 —— Spring容器（WebApplicationContext）和组件

这三个基本观点，可以说是我们对之前所有讨论的一个小结。明确了这些内容，我们就可以更加深入地看看DispatcherServlet初始化主线的过程：

![初始化主线](http://dl.iteye.com/upload/attachment/0062/9586/61b32fbb-1c8f-35ae-91cd-05dfd027b123.png)

在这幅图中，我们站在一个动态的角度将DispatcherServlet、容器（WebApplicationContext）和组件这三者之间的关系表述出来，同时给出了这三者之间的运行顺序和逻辑过程。读者或许对其中的绝大多数细节还很陌生，甚至有一种无从下手的感觉。这没有关系，大家可以首先抓住图中的执行线，回忆一下之前有关DispatcherServlet的继承结构和数据结构的内容。接下来，我们就图中的内容逐一进行解释。

### WebApplicationContext的初始化

之前我们讨论了DispatcherServlet对于WebApplicationContext的初始化是在FrameworkServlet中完成的，不过我们并没有细究其中的细节。在默认情况下，这个初始化过程是由web.xml中的入口程序配置所驱动的：

```
<!-- Processes application requests --> 
<servlet> 
    <servlet-name>dispatcher</servlet-name> 
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class> 
    <load-on-startup>1</load-on-startup> 
</servlet> 
         
<servlet-mapping> 
    <servlet-name>dispatcher</servlet-name> 
    <url-pattern>/</url-pattern> 
</servlet-mapping> 
```

事实上，这段入口程序的配置中隐藏了SpringMVC的两大要素（核心分发器Dispatcher和核心配置文件[servlet-name]-servlet.xml）之间的关系表述：

在默认情况下，web.xml配置节点中`<servlet-name>`的值就是建立起核心分发器DispatcherServlet与核心配置文件之间联系的桥梁。DispatcherServlet在初始化时会加载位置在/WEB-INF/[servlet-name]-servlet.xml的配置文件作为SpringMVC的核心配置。

SpringMVC在这里采用了一个“命名约定”的方法进行关系映射，这种方法很廉价也很管用。以上面的配置为例，我们就必须在/WEB-INF/目录下，放一个名为dispatcher-servlet.xml的Spring配置文件作为SpringMVC的核心配置用以指定SpringMVC的基本组件声明定义。

这看上去似乎有一点别扭，因为在实际项目中，我们通常喜欢把配置文件放在classpath下，并使用不同的package进行区分。例如，在基于Maven的项目结构中，所有的配置文件应置于src/main/resources目录下，这样才比较符合配置文件统一化管理的最佳实践。

于是，Spring提供了一个初始化的配置选项，通过指定contextConfigLocation选项来自定义SpringMVC核心配置文件的位置：

```
<!-- Processes application requests --> 
<servlet> 
    <servlet-name>dispatcher</servlet-name> 
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class> 
    <init-param> 
        <param-name>contextConfigLocation</param-name> 
        <param-value>classpath:web/applicationContext-dispatcherServlet.xml</param-value> 
    </init-param> 
    <load-on-startup>1</load-on-startup> 
</servlet> 
         
<servlet-mapping> 
    <servlet-name>dispatcher</servlet-name> 
    <url-pattern>/</url-pattern> 
</servlet-mapping> 
```

这样一来，DispatcherServlet在初始化时，就会自动加载在classpath下，web这个package下名为applicationContext-dispatcherServlet.xml的文件作为其核心配置并用以初始化容器（WebApplicationContext）。

结论: SpringMVC核心配置文件中所有的bean定义，就是SpringMVC的组件定义，也是DispatcherServlet在初始化容器（WebApplicationContext）时，所要进行初始化的组件。

SpringMVC的组件是一个个的接口定义，当我们在SpringMVC的核心配置文件中定义一个组件时，使用的却是组件的实现类：

```
<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver"> 
    <property name="prefix" value="/" /> 
    <property name="suffix" value=".jsp" /> 
</bean> 
```

这也就是Spring管理组件的模式，用具体的实现类来指定接口的行为方式。不同的实现类，代表着不同的组件行为模式，它们在Spring容器中是可以共存的：

```
<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver"> 
    <property name="prefix" value="/" /> 
    <property name="suffix" value=".jsp" /> 
</bean> 
 
<bean class="org.springframework.web.servlet.view.freemarker.FreeMarkerViewResolver"> 
    <property name="prefix" value="/" /> 
    <property name="suffix" value=".ftl" /> 
</bean> 
```

所以，Spring的容器就像是一个聚宝盆，它只负责承载对象，管理好对象的生命周期，而并不关心一个组件接口到底有多少种实现类或者行为模式。这也就是我们在上面那幅图中，画了多个HandlerMappings、HandlerAdapters和ViewResolvers的原因：一个组件的多种行为模式可以在容器中共存，容器将负责对这些实现类进行管理。而具体如何使用这些对象，则由应用程序自身来决定。

如此一来，我们可以大致概括一下WebApplicationContext初始化的两个逻辑层次：

- DispatcherServlet负责对容器（WebApplicationContext）进行初始化。
- 容器（WebApplicationContext）将读取SpringMVC的核心配置文件进行组件的实例化。

**注**：启动日志是我们研究SpringMVC的主要途径之一，之后我们还将反复使用这种方法对SpringMVC的运行过程进行研究。读者应该仔细品味每一条日志的作用，从而能够与之后的分析讲解呼应起来。

### 独立的WebApplicationContext体系

独立的WebApplicationContext体系，是SpringMVC初始化主线中的一个非常重要的概念。DispatcherServlet、容器和组件三者之间的关系，我们在引用的那副官方reference的示意图中，实际上已经包含了这一层意思：

在`DispatcherServlet`初始化的过程中所构建的`WebApplicationContext`独立于Spring自身的所构建的其他`WebApplicationContext`体系而存在。

```
<context-param> 
    <param-name>contextConfigLocation</param-name> 
    <param-value>classpath:context/applicationContext-*.xml</param-value> 
</context-param> 
     
<listener> 
    <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class> 
</listener> 
```

在上面的代码中，我们定义了一个Listener，它会在整个Web应用程序启动的时候运行一次，并初始化传统意义上的Spring的容器。这也是一般情况下，当并不使用SpringMVC作为我们的表示层解决方案，却希望在我们的Web应用程序中使用Spring相关功能时所采取的一种配置方式。

如果我们要在这里引入SpringMVC，整个配置看上去就像这样：

```
<context-param> 
    <param-name>contextConfigLocation</param-name> 
    <param-value>classpath:context/applicationContext-*.xml</param-value> 
</context-param> 
     
<listener> 
    <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class> 
</listener> 
     
<!-- Processes application requests --> 
<servlet> 
    <servlet-name>dispatcher</servlet-name> 
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class> 
    <init-param> 
        <param-name>contextConfigLocation</param-name> 
        <param-value>classpath:web/applicationContext-dispatcherServlet.xml</param-value> 
    </init-param> 
    <load-on-startup>1</load-on-startup> 
</servlet> 
         
<servlet-mapping> 
    <servlet-name>dispatcher</servlet-name> 
    <url-pattern>/</url-pattern> 
</servlet-mapping> 
```

在这种情况下，DispatcherServlet和ContextLoaderListener会分别构建不同作用范围的容器（WebApplicationContext）。我们可以引入两个不同的概念来对其进行表述：

- ContextLoaderListener所初始化的容器，们称之为Root WebApplicationContext；
- DispatcherServlet所初始化的容器，是SpringMVC WebApplicationContext。

### 组件默认行为的指定

DispatcherServlet的初始化主线的执行体系是顺着其继承结构依次进行的，我们在之前曾经讨论过它的执行次序。所以，只有在FrameworkServlet完成了对于WebApplicationContext和组件的初始化之后，执行权才被正式转移到DispatcherServlet中。我们可以来看看DispatcherServlet此时究竟干了哪些事：

```
/**
* This implementation calls {@link #initStrategies}.
*/ 
@Override 
protected void onRefresh(ApplicationContext context) { 
    initStrategies(context); 
} 
 
/**
* Initialize the strategy objects that this servlet uses.
* <p>May be overridden in subclasses in order to initialize further strategy objects.
*/ 
protected void initStrategies(ApplicationContext context) { 
    initMultipartResolver(context); 
    initLocaleResolver(context); 
    initThemeResolver(context); 
    initHandlerMappings(context); 
    initHandlerAdapters(context); 
    initHandlerExceptionResolvers(context); 
    initRequestToViewNameTranslator(context); 
    initViewResolvers(context); 
    initFlashMapManager(context); 
} 
```

onRefresh是FrameworkServlet中预留的扩展方法，在DispatcherServlet中做了一个基本实现：initStrategies。我们粗略一看，很容易就能明白DispatcherServlet到底在这里干些什么了：初始化组件。

读者或许会问，组件不是已经在WebApplicationContext初始化的时候已经被初始化过了嘛？这里所谓的组件初始化，指的又是什么呢？让我们来看看其中的一个方法的源码：

```
/**
* Initialize the MultipartResolver used by this class.
* <p>If no bean is defined with the given name in the BeanFactory for this namespace,
* no multipart handling is provided.
*/ 
private void initMultipartResolver(ApplicationContext context) { 
    try { 
        this.multipartResolver = context.getBean(MULTIPART_RESOLVER_BEAN_NAME, MultipartResolver.class); 
        if (logger.isDebugEnabled()) { 
            logger.debug("Using MultipartResolver [" + this.multipartResolver + "]"); 
        } 
    } catch (NoSuchBeanDefinitionException ex) { 
        // Default is no multipart resolver. 
        this.multipartResolver = null; 
        if (logger.isDebugEnabled()) { 
            logger.debug("Unable to locate MultipartResolver with name '" + MULTIPART_RESOLVER_BEAN_NAME + 
                    "': no multipart request handling provided"); 
        } 
    } 
} 
```

这里的初始化，指的是DispatcherServlet从容器（WebApplicationContext）中读取组件的实现类，并缓存于DispatcherServlet内部的过程。还记得我们之前给出的DispatcherServlet的数据结构吗？这些位于DispatcherServlet内部的组件实际上只是一些来源于容器缓存实例，不过它们同样也是DispatcherServlet进行后续操作的基础。

**注**：Servlet实例内部的属性的访问有线程安全问题。而在这里，我们可以看到所有的组件都以Servlet内部属性的形式被调用，充分证实了这些组件本身也都是无状态的单例对象，所以我们在这里不必考虑线程安全的问题。

如果对上面的代码加以详细分析，我们会发现initMultipartResolver的过程是查找特定MultipartResolver实现类的过程。因为在容器中查找组件的时候，采取的是根据特定名称（MULTIPART_RESOLVER_BEAN_NAME）进行查找的策略。由此，我们可以看到DispatcherServlet进行组件初始化的特点：

结论: DispatcherServlet中对于组件的初始化过程实际上是应用程序在WebApplicationContext中选择和查找组件实现类的过程，也是指定组件在SpringMVC中的默认行为方式的过程。

除了根据特定名称进行查找的策略以外，我们还对DispatcherServlet中指定SpringMVC默认行为方式的其他的策略进行的总结：

- 名称查找 —— 根据bean的名字在容器中查找相应的实现类
- 自动搜索 —— 自动搜索容器中所有某个特定组件（接口）的所有实现类
- 默认配置 —— 根据一个默认的配置文件指定进行实现类加载

这三条策略恰巧在initHandlerMappings的过程中都有体现，读者可以从其源码中找到相应的线索：

```
private void initHandlerAdapters(ApplicationContext context) { 
    this.handlerAdapters = null; 
 
    if (this.detectAllHandlerAdapters) { 
        // Find all HandlerAdapters in the ApplicationContext, including ancestor contexts. 
        Map<String, HandlerAdapter> matchingBeans = BeanFactoryUtils.beansOfTypeIncludingAncestors(context, HandlerAdapter.class,true, false); 
        if (!matchingBeans.isEmpty()) { 
            this.handlerAdapters = new ArrayList<HandlerAdapter>(matchingBeans.values()); 
            // We keep HandlerAdapters in sorted order. 
            OrderComparator.sort(this.handlerAdapters); 
        } 
    } 
    else { 
        try { 
            HandlerAdapter ha = context.getBean(HANDLER_ADAPTER_BEAN_NAME, HandlerAdapter.class); 
            this.handlerAdapters = Collections.singletonList(ha); 
        } 
        catch (NoSuchBeanDefinitionException ex) { 
            // Ignore, we'll add a default HandlerAdapter later. 
        } 
    } 
 
    // Ensure we have at least some HandlerAdapters, by registering 
    // default HandlerAdapters if no other adapters are found. 
    if (this.handlerAdapters == null) { 
        this.handlerAdapters = getDefaultStrategies(context, HandlerAdapter.class); 
        if (logger.isDebugEnabled()) { 
            logger.debug("No HandlerAdapters found in servlet '" + getServletName() + "': using default"); 
        } 
    } 
} 
```

这里有必要对“默认策略”做一个简要的说明。SpringMVC为一些核心组件设置了默认行为方式的说明，这个说明以一个properties文件的形式位于SpringMVC分发包（例如spring-webmvc-3.1.0.RELEASE.jar）的内部：

![默认策略](http://dl.iteye.com/upload/attachment/0062/9596/52159265-14d6-38ed-a4a3-6766a9876c53.png)

结合刚才initHandlerMappings的源码，我们可以发现如果没有开启detectAllHandlerAdapters选项或者根据HANDLER_ADAPTER_BEAN_NAME的名称没有找到相应的组件实现类，就会使用DispatcherServlet.properties文件中对于HandlerMapping接口的实现来进行组件默认行为的初始化。

由此可见，DispatcherServlet.properties中所指定的所有接口的实现方式在Spring的容器WebApplicationContext中总有相应的定义。这一点，我们在组件的讨论中还会详谈。

这个部分我们的侧重点是图中DispatcherServlet与容器之间的关系。读者需要理解的是图中为什么会有两份组件定义，它们之间的区别在哪里，以及DispatcherServlet在容器中查找组件的三种策略。

## 小结

在本文中，我们对SpringMVC的核心类：DispatcherServlet进行了一番梳理。也对整个SpringMVC的两条主线之一的初始化主线做了详细的分析。

对于DispatcherServlet而言，重要的其实并不是这个类中的代码和逻辑，而是应该掌握这个类在整个框架中的作用以及与SpringMVC中其他要素的关系。

对于初始化主线而言，核心其实仅仅在于那张笔者为大家精心打造的图。读者只要掌握了这张图，相信对整个SpringMVC的初始化过程会有一个全新的认识。

## DispatcherServlet源码

### DispatcherServlet 初始化

DispatcherServlet 的父类 HttpServletBean 覆盖了 HttpServlet 的 init 方法，实现该 servlet 的初始化。

```
/**
     * Map config parameters onto bean properties of this servlet, and
     * invoke subclass initialization.
     * @throws ServletException if bean properties are invalid (or required
     * properties are missing), or if subclass initialization fails.
     */
    @Override
    public final void init() throws ServletException {
        if (logger.isDebugEnabled()) {
            logger.debug("Initializing servlet '" + getServletName() + "'");
        }

        // Set bean properties from init parameters.
        try {
            PropertyValues pvs = new ServletConfigPropertyValues(getServletConfig(), this.requiredProperties);
            BeanWrapper bw = PropertyAccessorFactory.forBeanPropertyAccess(this);
            ResourceLoader resourceLoader = new ServletContextResourceLoader(getServletContext());
            bw.registerCustomEditor(Resource.class, new ResourceEditor(resourceLoader, getEnvironment()));
            initBeanWrapper(bw);
            bw.setPropertyValues(pvs, true);
        }
        catch (BeansException ex) {
            logger.error("Failed to set bean properties on servlet '" + getServletName() + "'", ex);
            throw ex;
        }

        // Let subclasses do whatever initialization they like.
        initServletBean();

        if (logger.isDebugEnabled()) {
            logger.debug("Servlet '" + getServletName() + "' configured successfully");
        }
    }
```

正如注释所说 initServletBean() 留由子类实现，体现了模板方法模式。

当上述bean属性设置完成后，进入这里 `FrameworkServlet.init() `创建 Servlet 的上下文 `WebApplicationContext，initWebApplicationContext `首先会获得该 Web 应用的` root WebApplicationContext` （通常是由 `org.springframework.web.context.ContextLoaderListener` 加载的），然后根据这个根上下文得到我们这个 Servlet 的 `WebApplicationContext。initFrameworkServlet` 方法是空的，而且子类 DispatcherServlet 也没有覆盖。

```
 /**
     * Overridden method of {@link HttpServletBean}, invoked after any bean properties
     * have been set. Creates this servlet's WebApplicationContext.
     */
    @Override
    protected final void initServletBean() throws ServletException {
        getServletContext().log("Initializing Spring FrameworkServlet '" + getServletName() + "'");
        if (this.logger.isInfoEnabled()) {
            this.logger.info("FrameworkServlet '" + getServletName() + "': initialization started");
        }
        long startTime = System.currentTimeMillis();

        try {
            this.webApplicationContext = initWebApplicationContext();
            initFrameworkServlet();
        }
        catch (ServletException ex) {
            this.logger.error("Context initialization failed", ex);
            throw ex;
        }
        catch (RuntimeException ex) {
            this.logger.error("Context initialization failed", ex);
            throw ex;
        }

        if (this.logger.isInfoEnabled()) {
            long elapsedTime = System.currentTimeMillis() - startTime;
            this.logger.info("FrameworkServlet '" + getServletName() + "': initialization completed in " +
                    elapsedTime + " ms");
        }
    }

    /**
     * Initialize and publish the WebApplicationContext for this servlet.
     * <p>Delegates to {@link #createWebApplicationContext} for actual creation
     * of the context. Can be overridden in subclasses.
     * @return the WebApplicationContext instance
     * @see #FrameworkServlet(WebApplicationContext)
     * @see #setContextClass
     * @see #setContextConfigLocation
     */
    protected WebApplicationContext initWebApplicationContext() {
        WebApplicationContext rootContext =
                WebApplicationContextUtils.getWebApplicationContext(getServletContext());
        WebApplicationContext wac = null;

        if (this.webApplicationContext != null) {
            // A context instance was injected at construction time -> use it
            wac = this.webApplicationContext;
            if (wac instanceof ConfigurableWebApplicationContext) {
                ConfigurableWebApplicationContext cwac = (ConfigurableWebApplicationContext) wac;
                if (!cwac.isActive()) {
                    // The context has not yet been refreshed -> provide services such as
                    // setting the parent context, setting the application context id, etc
                    if (cwac.getParent() == null) {
                        // The context instance was injected without an explicit parent -> set
                        // the root application context (if any; may be null) as the parent
                        cwac.setParent(rootContext);
                    }
                    configureAndRefreshWebApplicationContext(cwac);
                }
            }
        }
        if (wac == null) {
            // No context instance was injected at construction time -> see if one
            // has been registered in the servlet context. If one exists, it is assumed
            // that the parent context (if any) has already been set and that the
            // user has performed any initialization such as setting the context id
            wac = findWebApplicationContext();
        }
        if (wac == null) {
            // No context instance is defined for this servlet -> create a local one
            wac = createWebApplicationContext(rootContext);
        }

        if (!this.refreshEventReceived) {
            // Either the context is not a ConfigurableApplicationContext with refresh
            // support or the context injected at construction time had already been
            // refreshed -> trigger initial onRefresh manually here.
            onRefresh(wac);
        }

        if (this.publishContext) {
            // Publish the context as a servlet context attribute.
            String attrName = getServletContextAttributeName();
            getServletContext().setAttribute(attrName, wac);
            if (this.logger.isDebugEnabled()) {
                this.logger.debug("Published WebApplicationContext of servlet '" + getServletName() +
                        "' as ServletContext attribute with name [" + attrName + "]");
            }
        }

        return wac;
    }
```

### DispatcherServlet 处理请求流程

FrameworkServlet 中覆盖了 HttpServlet 的 doGet(),doPost()等方法，而 doGet(),doPost()等又直接调用方法 processRequest 来处理请求，代码如下:

```
/**
     * Delegate GET requests to processRequest/doService.
     * <p>Will also be invoked by HttpServlet's default implementation of {@code doHead},
     * with a {@code NoBodyResponse} that just captures the content length.
     * @see #doService
     * @see #doHead
     */
    @Override
    protected final void doGet(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {

        processRequest(request, response);
    }

    /**
     * Delegate POST requests to {@link #processRequest}.
     * @see #doService
     */
    @Override
    protected final void doPost(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {

        processRequest(request, response);
    }
```

然后我们进入 processRequest 方法，实际的请求处理是调用其抽象方法 doService:

```
/**
     * Process this request, publishing an event regardless of the outcome.
     * <p>The actual event handling is performed by the abstract
     * {@link #doService} template method.
     */
    protected final void processRequest(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {

        long startTime = System.currentTimeMillis();
        Throwable failureCause = null;

        LocaleContext previousLocaleContext = LocaleContextHolder.getLocaleContext();
        LocaleContext localeContext = buildLocaleContext(request);

        RequestAttributes previousAttributes = RequestContextHolder.getRequestAttributes();
        ServletRequestAttributes requestAttributes = buildRequestAttributes(request, response, previousAttributes);

        WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);
        asyncManager.registerCallableInterceptor(FrameworkServlet.class.getName(), new RequestBindingInterceptor());

        initContextHolders(request, localeContext, requestAttributes);

        try {
            doService(request, response);
        }
        catch (ServletException ex) {
            failureCause = ex;
            throw ex;
        }
        catch (IOException ex) {
            failureCause = ex;
            throw ex;
        }
        catch (Throwable ex) {
            failureCause = ex;
            throw new NestedServletException("Request processing failed", ex);
        }

        finally {
            resetContextHolders(request, previousLocaleContext, previousAttributes);
            if (requestAttributes != null) {
                requestAttributes.requestCompleted();
            }

            if (logger.isDebugEnabled()) {
                if (failureCause != null) {
                    this.logger.debug("Could not complete request", failureCause);
                }
                else {
                    if (asyncManager.isConcurrentHandlingStarted()) {
                        logger.debug("Leaving response open for concurrent processing");
                    }
                    else {
                        this.logger.debug("Successfully completed request");
                    }
                }
            }

            publishRequestHandledEvent(request, response, startTime, failureCause);
        }
    }

    /**
     * Subclasses must implement this method to do the work of request handling,
     * receiving a centralized callback for GET, POST, PUT and DELETE.
     * <p>The contract is essentially the same as that for the commonly overridden
     * {@code doGet} or {@code doPost} methods of HttpServlet.
     * <p>This class intercepts calls to ensure that exception handling and
     * event publication takes place.
     * @param request current HTTP request
     * @param response current HTTP response
     * @throws Exception in case of any kind of processing failure
     * @see javax.servlet.http.HttpServlet#doGet
     * @see javax.servlet.http.HttpServlet#doPost
     */
    protected abstract void doService(HttpServletRequest request, HttpServletResponse response)
            throws Exception;
```

然后在 DispatcherServlet 中具体实现请求的处理分发，先是把一些资源放到请求属性中，然后调用 doDispatch 实现请求分发到控制器的 handler。doDispatch 中首先会判断是否是文件传输流的请求（利用MultipartResolver），如果是的话就会转为 MultipartHttpServletRequest。接下来 getHandler(processedRequest) 根据请求获得对应的handler，最后调用 handle() 处理请求，会反射到在控制器中实现的方法。

以下是doService源码:

```
@Override
protected void doService(HttpServletRequest request, HttpServletResponse response) throws Exception {
    //log ...

    //建立一个attributesSnapshot用于备份
    Map<String, Object> attributesSnapshot = null;
    // 当request是一个include request时
    if (WebUtils.isIncludeRequest(request)) {
        attributesSnapshot = new HashMap<String, Object>();
        Enumeration<?> attrNames = request.getAttributeNames();
        while (attrNames.hasMoreElements()) {
            String attrName = (String) attrNames.nextElement();
            if (this.cleanupAfterInclude || attrName.startsWith("org.springframework.web.servlet")) {
                attributesSnapshot.put(attrName, request.getAttribute(attrName));
            }
        }
    }

    //对request设置一些属性
    request.setAttribute(WEB_APPLICATION_CONTEXT_ATTRIBUTE, getWebApplicationContext());
    request.setAttribute(LOCALE_RESOLVER_ATTRIBUTE, this.localeResolver);
    request.setAttribute(THEME_RESOLVER_ATTRIBUTE, this.themeResolver);
    request.setAttribute(THEME_SOURCE_ATTRIBUTE, getThemeSource());

    FlashMap inputFlashMap = this.flashMapManager.retrieveAndUpdate(request, response);
    if (inputFlashMap != null) {
        request.setAttribute(INPUT_FLASH_MAP_ATTRIBUTE, Collections.unmodifiableMap(inputFlashMap));
    }
    request.setAttribute(OUTPUT_FLASH_MAP_ATTRIBUTE, new FlashMap());
    request.setAttribute(FLASH_MAP_MANAGER_ATTRIBUTE, this.flashMapManager);

    try {
        //执行处理请求的主方法
        doDispatch(request, response);
    }
    finally {
        if (!WebAsyncUtils.getAsyncManager(request).isConcurrentHandlingStarted()) {
            // Restore the original attribute snapshot, in case of an include.
            if (attributesSnapshot != null) {
                restoreAttributesAfterInclude(request, attributesSnapshot);
            }
        }
    }
}
```

**总的来说做的事：** 

1. 判断是不是include请求? 对request的Attribute做个快照备份 : do nothing; 
2. 对request设置了一些属性。 这里request.setAttribute前四个属性与handler和view有关。 
3. doDispatch(request,response); 
4. 如果做了快照，进行还原。

以下是doDispatch源码:

```
 protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {
        HttpServletRequest processedRequest = request;//processedRequest 是在本方法中实际处理的request对象
        HandlerExecutionChain mappedHandler = null;//非常重要的Interceptor
        boolean multipartRequestParsed = false;

        WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);

        try {
            ModelAndView mv = null;
            Exception dispatchException = null;

            try {
                processedRequest = checkMultipart(request);//1、检查是不是上传文件请求
                multipartRequestParsed = (processedRequest != request);//是的话留下标记


                // Determine handler for the current request.
                mappedHandler = getHandler(processedRequest);//2、根据request找到handler
                if (mappedHandler == null || mappedHandler.getHandler() == null) {
                    noHandlerFound(processedRequest, response);
                    return;
                }

                // Determine handler adapter for the current request.
                HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());//3、根据handler找到handlerAdapter

                // Process last-modified header, if supported by the handler.
                String method = request.getMethod();  //4、处理是否被修改字段，下文详细解释
                boolean isGet = "GET".equals(method);
                if (isGet || "HEAD".equals(method)) {
                    long lastModified = ha.getLastModified(request, mappedHandler.getHandler());
                    if (logger.isDebugEnabled()) {
                        logger.debug("Last-Modified value for [" + getRequestUri(request) + "] is: " + lastModified);
                    }
                    if (new ServletWebRequest(request, response).checkNotModified(lastModified) && isGet) {
                        return;
                    }
                }

                if (!mappedHandler.applyPreHandle(processedRequest, response)) {//5、interceptor 对request 进行preHandle
                    return;
                }

                // Actually invoke the handler.
                mv = ha.handle(processedRequest, response, mappedHandler.getHandler());//6、 实际处理请求发生的地方

                //如果需要异步处理，直接返回
                if (asyncManager.isConcurrentHandlingStarted()) {
                    return;
                }
                applyDefaultViewName(processedRequest, mv);//7、处理view，若view为空，根据request设置默认view
                mappedHandler.applyPostHandle(processedRequest, response, mv);//8、interceptor 对request 进行postHandle
            }
            catch (Exception ex) {
                dispatchException = ex;
            }

            processDispatchResult(processedRequest, response, mappedHandler, mv, dispatchException);//如果捕捉到Exception，调用该方法进行处理，注意下方还有外层catch捕捉该方法throw的exception
        }
        catch (Exception ex) {
            triggerAfterCompletion(processedRequest, response, mappedHandler, ex);
        }
        catch (Error err) {
            triggerAfterCompletionWithError(processedRequest, response, mappedHandler, err);
        }
        finally {
            //判断是否执行异步请求
            if (asyncManager.isConcurrentHandlingStarted()) {
                // Instead of postHandle and afterCompletion
                if (mappedHandler != null) {
                    mappedHandler.applyAfterConcurrentHandlingStarted(processedRequest, response);
                }
            }
            else {
                // Clean up any resources used by a multipart request.
                if (multipartRequestParsed) {
                    cleanupMultipart(processedRequest);//8、如果是上传请求，最后清空生成的临时资源
                }
            }
        }
    }
```

- Last-Modeified：浏览器第一次跟服务器请求资源(GET,Head请求)时，服务器在返回的请求头里面会包含一个Last-Modified的属性，代表资源什么时候修改的，浏览器之后发送请求会发送之前接受到的Last-Modified。服务器接收到Last-Modified后会和自己实际资源的修改时间对比。过期了就返回新的资源和新的Last-Modified。否则直接返回304状态码，浏览器直接使用之前缓存的结果。
- Handler：处理器，对应Controller层，具体表象形式很多，是Object类型。 只要可以实际处理请求就可以试Handler 
- HandlerMapping：用来查找Handler。根据不同的请求获得不同的Handler。 

**因此处理流程可以理解为：** 

用HandlerMapping找到负责处理请求的Handler，负责使用Handler的HandlerAdapter。 HandlerAdapter使用Handler处理请求。 生成ModelAndView，返回给用户。

**异常处理结构：** 

doDispatch有两层异常捕获，内层捕获处理请求的异常,由processDispatchResult处理。 外层处理processDispatchResult方法抛出的异常。渲染页面时抛出的异常。



























