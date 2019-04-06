# Spring家族

## **Spring**

### **体系结构**

Spring 有可能成为所有企业应用程序的一站式服务点，然而，Spring 是模块化的，允许你挑选和选择适用于你的模块，不必要把剩余部分也引入。下面的部分对在 Spring 框架中所有可用的模块给出了详细的介绍。

Spring 框架提供约 20 个模块，可以根据应用程序的要求来使用。

![img](https://7n.w3cschool.cn/attachments/image/wk/wkspring/arch1.png)



#### **核心容器**

核心容器由**spring-core，spring-beans，spring-context，spring-context-support和spring-expression**（SpEL，Spring表达式语言，Spring Expression Language）等模块组成，它们的细节如下：

- **spring-core**模块提供了框架的基本组成部分，包括 IoC 和依赖注入功能。
- **spring-beans** 模块提供 BeanFactory，工厂模式的微妙实现，它移除了编码式单例的需要，并且可以把配置和依赖从实际编码逻辑中解耦。
- **context**模块建立在由**core**和 **beans** 模块的基础上建立起来的，它以一种类似于JNDI注册的方式访问对象。Context模块继承自Bean模块，并且添加了国际化（比如，使用资源束）、事件传播、资源加载和透明地创建上下文（比如，通过Servelet容器）等功能。Context模块也支持Java EE的功能，比如EJB、JMX和远程调用等。**ApplicationContext**接口是Context模块的焦点。**spring-context-support**提供了对第三方库集成到Spring上下文的支持，比如缓存（EhCache, Guava, JCache）、邮件（JavaMail）、调度（CommonJ, Quartz）、模板引擎（FreeMarker, JasperReports, Velocity）等。
- **spring-expression**模块提供了强大的表达式语言，用于在运行时查询和操作对象图。它是JSP2.1规范中定义的统一表达式语言的扩展，支持set和get属性值、属性赋值、方法调用、访问数组集合及索引的内容、逻辑算术运算、命名变量、通过名字从Spring IoC容器检索对象，还支持列表的投影、选择以及聚合等。

#### **数据访问/集成**

数据访问/集成层包括 JDBC，ORM，OXM，JMS 和事务处理模块，它们的细节如下：

（注：JDBC=Java Data Base Connectivity，ORM=Object Relational Mapping，OXM=Object XML Mapping，JMS=Java Message Service）

- **JDBC** 模块提供了JDBC抽象层，它消除了冗长的JDBC编码和对数据库供应商特定错误代码的解析。
- **ORM** 模块提供了对流行的对象关系映射API的集成，包括JPA、JDO和Hibernate等。通过此模块可以让这些ORM框架和spring的其它功能整合，比如前面提及的事务管理。
- **OXM** 模块提供了对OXM实现的支持，比如JAXB、Castor、XML Beans、JiBX、XStream等。
- **JMS** 模块包含生产（produce）和消费（consume）消息的功能。从Spring 4.1开始，集成了spring-messaging模块。。
- **事务**模块为实现特殊接口类及所有的 POJO 支持编程式和声明式事务管理。（注：编程式事务需要自己写beginTransaction()、commit()、rollback()等事务管理方法，声明式事务是通过注解或配置由spring自动处理，编程式事务粒度更细）

#### **Web**

Web 层由 Web，Web-MVC，Web-Socket 和 Web-Portlet 组成，它们的细节如下：

- **Web** 模块提供面向web的基本功能和面向web的应用上下文，比如多部分（multipart）文件上传功能、使用Servlet监听器初始化IoC容器等。它还包括HTTP客户端以及Spring远程调用中与web相关的部分。
- **Web-MVC** 模块为web应用提供了模型视图控制（MVC）和REST Web服务的实现。Spring的MVC框架可以使领域模型代码和web表单完全地分离，且可以与Spring框架的其它所有功能进行集成。
- **Web-Socket** 模块为 WebSocket-based 提供了支持，而且在 web 应用程序中提供了客户端和服务器端之间通信的两种方式。
- **Web-Portlet** 模块提供了用于Portlet环境的MVC实现，并反映了spring-webmvc模块的功能。

#### **其他**

还有其他一些重要的模块，像 AOP，Aspects，Instrumentation，Web 和测试模块，它们的细节如下：

- **AOP** 模块提供了面向方面的编程实现，允许你定义方法拦截器和切入点对代码进行干净地解耦，从而使实现功能的代码彻底的解耦出来。使用源码级的元数据，可以用类似于.Net属性的方式合并行为信息到代码中。
- **Aspects** 模块提供了与 **AspectJ** 的集成，这是一个功能强大且成熟的面向切面编程（AOP）框架。
- **Instrumentation** 模块在一定的应用服务器中提供了类 instrumentation 的支持和类加载器的实现。
- **Messaging** 模块为 STOMP 提供了支持作为在应用程序中 WebSocket 子协议的使用。它也支持一个注解编程模型，它是为了选路和处理来自 WebSocket 客户端的 STOMP 信息。
- **测试**模块支持对具有 JUnit 或 TestNG 框架的 Spring 组件的测试。



### **IoC 容器**

#### **BeanFactory 容器**

这是一个最简单的容器，它主要的功能是为依赖注入 （DI） 提供支持，这个容器接口在 org.springframework.beans.factory.BeanFactor 中被定义。

在 Spring 中，有大量对 BeanFactory 接口的实现。其中，最常被使用的是 **XmlBeanFactory** 类。这个容器从一个 XML 文件中读取配置元数据，由这些元数据来生成一个被配置化的系统或者应用。

在资源宝贵的移动设备或者基于 applet 的应用当中， BeanFactory 会被优先选择。否则，一般使用的是 ApplicationContext，除非你有更好的理由选择 BeanFactory。

#### **ApplicationContext 容器**

Application Context 是 spring 中较高级的容器。和 BeanFactory 类似，它可以加载配置文件中定义的 bean，将所有的 bean 集中在一起，当有请求的时候分配 bean。 

最常被使用的 ApplicationContext 接口实现：

- **FileSystemXmlApplicationContext**：该容器从 XML 文件中加载已被定义的 bean。在这里，你需要提供给构造器 XML 文件的完整路径。
- **ClassPathXmlApplicationContext**：该容器从 XML 文件中加载已被定义的 bean。在这里，你不需要提供 XML 文件的完整路径，只需正确配置 CLASSPATH 环境变量即可，因为，容器会从 CLASSPATH 中搜索 bean 配置文件。
- **WebXmlApplicationContext**：该容器会在一个 web 应用程序的范围内加载在 XML 文件中已被定义的 bean。

我们已经在 Spring Hello World Example章节中看到过 ClassPathXmlApplicationContext 容器，并且，在基于 spring 的 web 应用程序这个独立的章节中，我们讨论了很多关于 XmlWebApplicationContext。



#### **Bean 定义**

被称作 bean 的对象是构成应用程序的支柱也是由 Spring IoC 容器管理的。bean 是一个被实例化，组装，并通过 Spring IoC 容器所管理的对象。这些 bean 是由用容器提供的配置元数据创建的。



#### **基于注解的配置**

从 Spring 2.5 开始就可以使用**注解**来配置依赖注入。而不是采用 XML 来描述一个 bean 连线，你可以使用相关类，方法或字段声明的注解，将 bean 配置移动到组件类本身。

在 XML 注入之前进行注解注入，因此后者的配置将通过两种方式的属性连线被前者重写。

注解连线在默认情况下在 Spring 容器中不打开。因此，在可以使用基于注解的连线之前，我们将需要在我们的 Spring 配置文件中启用它。所以如果你想在 Spring 应用程序中使用的任何注解，可以考虑到下面的配置文件。

```
<?xml version="1.0" encoding="UTF-8"?>

<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
    http://www.springframework.org/schema/context
    http://www.springframework.org/schema/context/spring-context-3.0.xsd">

   <context:annotation-config/>
   <!-- bean definitions go here -->

</beans>
```

一旦 被配置后，你就可以开始注解你的代码，表明 Spring 应该自动连接值到属性，方法和构造函数。让我们来看看几个重要的注解，并且了解它们是如何工作的：

|  序号  |                 注解 & 描述                  |
| :--: | :--------------------------------------: |
|  1   | [@Required](https://www.w3cschool.cn/wkspring/9sle1mmh.html)@Required 注解应用于 bean 属性的 setter 方法。 |
|  2   | [@Autowired](https://www.w3cschool.cn/wkspring/rw2h1mmj.html)@Autowired 注解可以应用到 bean 属性的 setter 方法，非 setter 方法，构造函数和属性。 |
|  3   | [@Qualifier](https://www.w3cschool.cn/wkspring/knqr1mm2.html)通过指定确切的将被连线的 bean，@Autowired 和 @Qualifier 注解可以用来删除混乱。 |
|  4   | [JSR-250 Annotations](https://www.w3cschool.cn/wkspring/lmsq1mm4.html)Spring 支持 JSR-250 的基础的注解，其中包括了 @Resource，@PostConstruct 和 @PreDestroy 注解。 |



### **常用注解**

**声明Bean的注解:**

@Component : 组件,没有明确的角色
@Service : 在业务逻辑层(service层)使用
@Repository : 在数据访问层(dao层)使用.
@Controller : 在展现层(MVC--SpringMVC)使用

**注入Bean的注解:**

@Aautowired : Spring提供的注解.
@Inject : JSR-330提供的注解
@Resource : JSR-250提供的注解

**配置文件的注解:**

@Configuration : 声明当前类是个配置类,相当于一个Spring配置的xml文件.
@ComponentScan (cn.test.demo): 自动扫描包名下所有使用 @Component @Service  @Repository @Controller 的类,并注册为Bean
@WiselyConfiguration : 组合注解可以替代 @Configuration和@ComponentScan
@Bean : 注解在方法上,声明当前方法的返回值为一个Bean.·         
@Bean(initMethod="aa",destroyMethod="bb")--> 指定 aa和bb方法在构造之后.Bean销毁之前执行.

**AOP切面编程注解:**

@Aspect : 声明这是一个切面 
@After @Before. @Around 定义切面,可以直接将拦截规则(切入点 PointCut)作为参数
@PointCut : 专门定义拦截规则然后在 @After @Before. @Around 中调用
@Transcational : 事务处理
@Cacheable : 数据缓存
@EnableAaspectJAutoProxy : 开启Spring 对这个切面(Aspect )的支持
@Target (ElementType.TYPE):元注解,用来指定注解修饰类的那个成员 -->指定拦截规则
@Retention(RetentionPolicy.RUNTIME) ·         
--->当定义的注解的@Retention为RUNTIME时，才能够通过运行时的反射机制来处理注解.-->指定拦截规则

**Spring 常用配置:**

@import :导入配置类
@Scope : 新建Bean的实例 @Scope("prototype") 声明Scope 为 Prototype
@Value : 属性注入·         
@Value ("我爱你")  --> 普通字符串注入
@Value ("#{systemProperties['os.name']}") -->注入操作系统属性
@Value ("#{ T (java.lang.Math).random()  * 100.0 }") --> 注入表达式结果
@Value ("#{demoService.another}") --> 注入其他Bean属性
@Value ( "classpath:com/wisely/highlight_spring4/ch2/el/test.txt" ) --> 注入文件资源
@Value ("http://www.baidu.com")-->注入网址资源
@Value ("${book.name}" ) --> 注入配置文件  注意: 使用的是$ 而不是 #
@PostConstruct : 在构造函数执行完之后执行
@PreDestroy  : 在 Bean 销毁之前执行
@ActiveProfiles : 用来声明活动的 profile
@profile: 为不同环境下使用不同的配置提供了支持·         
 @Profile("dev") .......对方法名为 dev-xxxx的方法提供实例化Bean
@ActiveProfiles("dev")测试类中激活相应环境Bean
@EnableAsync : 开启异步任务的支持(多线程)
@Asyns : 声明这是一个异步任务,可以在类级别和方法级别声明.
@EnableScheduling : 开启对计划任务的支持(定时器)
@Scheduled : 声明这是一个计划任务支持多种计划任务,包含 cron. fixDelay fixRate·         
@Scheduled (dixedDelay = 5000) 通过注解定时更新
@Conditional : 条件注解,根据满足某一特定条件创建一个特定的Bean
@ContextConfiguration : 加载配置文件·         
@ContextConfiguration(classes = {TestConfig.class})
@ContextConfiguration用来加载ApplicationContext 
classes属性用来加载配置类
@WebAppCofiguration : 指定加载 ApplicationContext是一个WebApplicationContext

Link：https://www.w3cschool.cn/wkspring/



## **SpringMVC**

### **Spring MVC 框架简介**

Spring的模型-视图-控制器（MVC）框架是围绕一个`DispatcherServlet`来设计的，这个Servlet会把请求分发给各个处理器，并支持可配置的处理器映射、视图渲染、本地化、时区与主题渲染等，甚至还能支持文件上传。处理器是你的应用中注解了`@Controller`和`@RequestMapping`的类和方法，Spring为处理器方法提供了极其多样灵活的配置。Spring 3.0以后提供了`@Controller`注解机制、`@PathVariable`注解以及一些其他的特性，你可以使用它们来进行RESTful web站点和应用的开发。

### **DispatcherServlet处理请求**

Spring MVC框架，与其他很多web的MVC框架一样：请求驱动；所有设计都围绕着一个中央Servlet来展开，它负责把所有请求分发到控制器；同时提供其他web应用开发所需要的功能。不过Spring的中央处理器，`DispatcherServlet`，能做的比这更多。它与Spring IoC容器做到了无缝集成，这意味着，Spring提供的任何特性，在Spring MVC中你都可以使用。

`DispatcherServlet`其实就是个`Servlet`（它继承自`HttpServlet`基类），同样也需要在你web应用的`web.xml`配置文件下声明。你需要在`web.xml`文件中把你希望`DispatcherServlet`处理的请求映射到对应的URL上去。这就是标准的Java EE Servlet配置；下面的代码就展示了对`DispatcherServlet`和路径映射的声明：

```
<web-app>
    <servlet>
        <servlet-name>example</servlet-name>
		<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>   
		<param-name>contextConfigLocation</param-name>   
		<param-value>/WEB-INF/spring/appServlet/servlet-context.xml</param-value>
		</init-param><load-on-startup>1</load-on-startup>
		<async-supported>true</async-supported>
    </servlet>
    
    <servlet-mapping>
        <servlet-name>example</servlet-name>
        <url-pattern>/example/*</url-pattern>
    </servlet-mapping>
</web-app>
```

在上面的例子中，所有路径以`/*`开头的请求都会被`DispatcherServlet`处理。

你还可以用编程的方式配置Servlet容器。下面是一段这种基于代码配置的例子，它与上面定义的`web.xml`配置文件是等效的。

```
public class MyWebApplicationInitializer implements WebApplicationInitializer {
    @Override
    public void onStartup(ServletContext container) {
        ServletRegistration.Dynamic registration = container.addServlet("dispatcher", new DispatcherServlet());
        registration.setLoadOnStartup(1);
        registration.addMapping("/example/*");
    }
}
```



### **DispatcherServlet的处理流程**

配置好`DispatcherServlet`以后，开始有请求会经过这个`DispatcherServlet`。此时，`DispatcherServlet`会依照以下的次序对请求进行处理：

- 首先，搜索应用的上下文对象`WebApplicationContext`并把它作为一个属性（attribute）绑定到该请求上，以便控制器和其他组件能够使用它。属性的键名默认为`DispatcherServlet.WEB_APPLICATION_CONTEXT_ATTRIBUTE`
- 将地区（locale）解析器绑定到请求上，以便其他组件在处理请求（渲染视图、准备数据等）时可以获取区域相关的信息。如果你的应用不需要解析区域相关的信息，忽略它即可
- 将主题（theme）解析器绑定到请求上，以便其他组件（比如视图等）能够了解要渲染哪个主题文件。同样，如果你不需要使用主题相关的特性，忽略它即可
- 如果你配置了multipart文件处理器，那么框架将查找该文件是不是multipart（分为多个部分连续上传）的。若是，则将该请求包装成一个`MultipartHttpServletRequest`对象，以便处理链中的其他组件对它做进一步的处理
- 为该请求查找一个合适的处理器。如果可以找到对应的处理器，则与该处理器关联的整条执行链（前处理器、后处理器、控制器等）都会被执行，以完成相应模型的准备或视图的渲染
- 如果处理器返回的是一个模型（model），那么框架将渲染相应的视图。若没有返回任何模型（可能是因为前后的处理器出于某些原因拦截了请求等，比如，安全问题），则框架不会渲染任何视图，此时认为对请求的处理可能已经由处理链完成了

如果在处理请求的过程中抛出了异常，那么上下文`WebApplicationContext`对象中所定义的异常处理器将会负责捕获这些异常。通过配置你自己的异常处理器，你可以定制自己处理异常的方式。

Spring的`DispatcherServlet`也允许处理器返回一个Servlet API规范中定义的 *最后修改时间戳（last-modification-date）* 值。决定请求最后修改时间的方式很直接：`DispatcherServlet`会先查找合适的处理器映射来找到请求对应的处理器，然后检测它是否实现了 *LastModified* 接口。若是，则调用接口的`long getLastModified(request)`方法，并将该返回值返回给客户端。

可以定制`DispatcherServlet`的配置，具体的做法，是在`web.xml`文件中，Servlet的声明元素上添加一些Servlet的初始化参数（通过`init-param`元素）。该元素可选的参数列表如下：

|          可选参数           |                    解释                    |
| :---------------------: | :--------------------------------------: |
|     `contextClass`      | 任意实现了`WebApplicationContext`接口的类。这个类会初始化该servlet所需要用到的上下文对象。默认情况下，框架会使用一个`XmlWebApplicationContext`对象。 |
| `contextConfigLocation` | 一个指定了上下文配置文件路径的字符串，该值会被传入给`contextClass`所指定的上下文实例对象。该字符串内可以包含多个字符串，字符串之间以逗号分隔，以此支持你进行多个上下文的配置。在多个上下文中重复定义的bean，以最后加载的bean定义为准 |
|       `namespace`       | `WebApplicationContext`的命名空间。默认是`[servlet-name]-servlet` |



###  **控制器的实现**

控制器作为应用程序逻辑的处理入口，它会负责去调用你已经实现的一些服务。通常，一个控制器会接收并解析用户的请求，然后把它转换成一个模型交给视图，由视图渲染出页面最终呈现给用户。Spring 2.5以后引入了基于注解的编程模型，你可以在你的控制器实现上添加`@RequestMapping`、`@RequestParam`、`@ModelAttribute`等注解。注解特性既支持基于Servlet的MVC，也可支持基于Portlet的MVC。通过此种方式实现的控制器既无需继承某个特定的基类，也无需实现某些特定的接口。而且，它通常也不会直接依赖于Servlet或Portlet的API来进行编程，不过你仍然可以很容易地获取Servlet或Portlet相关的变量、特性和设施等。

```
@Controller
public class HelloWorldController {

    @RequestMapping("/helloWorld")
    public String helloWorld(Model model) {
        model.addAttribute("message", "Hello World!");
        return "helloWorld";
    }
}
```



### **@Controller注解定义一个控制器**

`@Controller`注解表明了一个类是作为控制器的角色而存在的。Spring不要求你去继承任何控制器基类，也不要求你去实现Servlet的那套API。

`@Controller`注解可以认为是被标注类的原型（stereotype），表明了这个类所承担的角色。分派器（`DispatcherServlet`）会扫描所有注解了`@Controller`的类，检测其中通过`@RequestMapping`注解配置的方法。

你需要在配置中加入组件扫描的配置代码来开启框架对注解控制器的自动检测。请使用下面XML代码所示的spring-context schema：

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:p="http://www.springframework.org/schema/p"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd">

    <context:component-scan base-package="org.springframework.samples.petclinic.web"/>

    <!-- ... -->
</beans>
```



### **@RequestMapping注解映射请求路径**

可以使用`@RequestMapping`注解来将请求URL，如`/appointments`等，映射到整个类上或某个特定的处理器方法上。

下面这段代码展示了在Spring MVC中如何在控制器上使用`@RequestMapping`注解：

```
@Controller
@RequestMapping("/appointments")
public class AppointmentsController {

    private final AppointmentBook appointmentBook;

    @Autowired
    public AppointmentsController(AppointmentBook appointmentBook) {
        this.appointmentBook = appointmentBook;
    }

    @RequestMapping(method = RequestMethod.GET)
    public Map<String, Appointment> get() {
        return appointmentBook.getAppointmentsForToday();
    }

    @RequestMapping(path = "/{day}", method = RequestMethod.GET)
    public Map<String, Appointment> getForDay(@PathVariable @DateTimeFormat(iso=ISO.DATE) Date day, Model model) {
        return appointmentBook.getAppointmentsForDay(day);
    }

    @RequestMapping(path = "/new", method = RequestMethod.GET)
    public AppointmentForm getNewForm() {
        return new AppointmentForm();
    }

    @RequestMapping(method = RequestMethod.POST)
    public String add(@Valid AppointmentForm appointment, BindingResult result) {
        if (result.hasErrors()) {
            return "appointments/new";
        }
        appointmentBook.addAppointment(appointment);
        return "redirect:/appointments";
    }
}
```

第一次使用点是作用于类级别的，它指示了所有`/appointments`开头的路径都会被映射到控制器下。`get()`方法上的`@RequestMapping`注解对请求路径进行了进一步细化：它仅接受GET方法的请求。这样，一个请求路径为`/appointments`、HTTP方法为GET的请求，将会最终进入到这个方法被处理。`add()`方法也做了类似的细化，而`getNewForm()`方法则同时注解了能够接受的请求的HTTP方法和路径。

#### **@Controller和面向切面（AOP）代理**

有时，我们希望在运行时使用AOP代理来装饰控制器，比如当你直接在控制器上使用`@Transactional`注解时。这种情况下，我们推荐使用类级别（在控制器上使用）的代理方式。这一般是代理控制器的默认做法。如果控制器必须实现一些接口，而该接口又不支持Spring Context的回调（比如`InitializingBean`, `*Aware`等接口），那要配置类级别的代理就必须手动配置了。比如，原来的配置文件``需要显式配置为``。

Spring 3.1中新增了一组类用以增强`@RequestMapping`，分别是`RequestMappingHandlerMapping`和`RequestMappingHandlerAdapter`。

URI模板

URI模板是一个类似于URI的字符串，只不过其中包含了一个或多个的变量名。当你使用实际的值去填充这些变量名的时候，模板就退化成了一个URI。在URI模板的RFC提议中定义了一个URI是如何进行参数化的。比如说，一个这个URI模板`http://www.example.com/users/{userId}`就包含了一个变量名*userId*。将值*fred*赋给这个变量名后，它就变成了一个URI：`http://www.example.com/users/fred`。

在Spring MVC中你可以在方法参数上使用`@PathVariable`注解，将其与URI模板中的参数绑定起来：

```
@RequestMapping(path="/owners/{ownerId}", method=RequestMethod.GET)
public String findOwner(@PathVariable String ownerId, Model model) {
    Owner owner = ownerService.findOwner(ownerId);
    model.addAttribute("owner", owner);
    return "displayOwner";
}
```

URI模板"`/owners/{ownerId}`"指定了一个变量，名为`ownerId`。当控制器处理这个请求的时候，`ownerId`的值就会被URI模板中对应部分的值所填充。比如说，如果请求的URI是`/owners/fred`，此时变量`ownerId`的值就是`fred`。

为了处理`@PathVariables`注解，Spring MVC必须通过变量名来找到URI模板中相对应的变量。你可以在注解中直接声明：

```
@RequestMapping(path="/owners/{ownerId}}", method=RequestMethod.GET)
public String findOwner(@PathVariable("ownerId") String theOwner, Model model) {
    // 具体的方法代码…
}

```

或者，如果URI模板中的变量名与方法的参数名是相同的，则你可以不必再指定一次。只要你在编译的时候留下debug信息，Spring MVC就可以自动匹配URL模板中与方法参数名相同的变量名。

```
@RequestMapping(path="/owners/{ownerId}", method=RequestMethod.GET)
public String findOwner(@PathVariable String ownerId, Model model) {
    // 具体的方法代码…
}
```

`@PathVariable`可以被应用于所有 *简单类型* 的参数上，比如int、long、Date等类型。Spring会自动地帮你把参数转化成合适的类型，如果转换失败，就抛出一个`TypeMismatchException`。如果你需要处理其他数据类型的转换，也可以注册自己的类。



#### **带正则表达式的URI模板**

`@RequestMapping`注解支持你在URI模板变量中使用正则表达式。语法是`{varName:regex}`，其中第一部分定义了变量名，第二部分就是你所要应用的正则表达式。比如下面的代码样例：

```
@RequestMapping("/spring-web/{symbolicName:[a-z-]+}-{version:\\d\\.\\d\\.\\d}{extension:\\.[a-z]+}")
    public void handle(@PathVariable String version, @PathVariable String extension) {
        // 代码部分省略...
    }
}
```



#### **路径样式的匹配**

URI模板变量的数目和通配符数量的总和最少的那个路径模板更准确。举个例子，`/hotels/{hotel}/*`这个路径拥有一个URI变量和一个通配符，而`/hotels/{hotel}/**`这个路径则拥有一个URI变量和两个通配符，因此，我们认为前者是更准确的路径模板。

如果两个模板的URI模板数量和通配符数量总和一致，则路径更长的那个模板更准确。举个例子，`/foo/bar*`就被认为比`/foo/*`更准确，因为前者的路径更长。

如果两个模板的数量和长度均一致，则那个具有更少通配符的模板是更加准确的。比如，`/hotels/{hotel}`就比`/hotels/*`更精确。



#### **矩阵变量**

矩阵变量可以在任何路径段落中出现，每对矩阵变量之间使用一个分号“;”隔开。比如这样的URI：`"/cars;color=red;year=2012"`。多个值可以用逗号隔开`"color=red,green,blue"`，或者重复变量名多次`"color=red;color=green;color=blue"`。

下面是一个例子，展示了我们如何从矩阵变量中获取到变量“q”的值：

```
// GET /pets/42;q=11;r=22
@RequestMapping(path = "/pets/{petId}", method = RequestMethod.GET)
public void findPet(@PathVariable String petId, @MatrixVariable int q) {

    // petId == 42
    // q == 11

}
```

由于任意路径段落中都可以含有矩阵变量，在某些场景下，你需要用更精确的信息来指定一个矩阵变量的位置：

```
// GET /owners/42;q=11/pets/21;q=22
@RequestMapping(path = "/owners/{ownerId}/pets/{petId}", method = RequestMethod.GET)
public void findPet(
    @MatrixVariable(name="q", pathVar="ownerId") int q1,
    @MatrixVariable(name="q", pathVar="petId") int q2) {
    // q1 == 11
    // q2 == 22
}
```

可以通过一个Map来存储所有的矩阵变量：

```
// GET /owners/42;q=11;r=12/pets/21;q=22;s=23
@RequestMapping(path = "/owners/{ownerId}/pets/{petId}", method = RequestMethod.GET)
public void findPet(
    @MatrixVariable Map<String, String> matrixVars,
    @MatrixVariable(pathVar="petId") Map<String, String> petMatrixVars) {

    // matrixVars: ["q" : [11,22], "r" : 12, "s" : 23]
    // petMatrixVars: ["q" : 22, "s" : 23]

}
```



#### **可消费的媒体类型**

你可以指定一组可消费的媒体类型，缩小映射的范围。这样只有当请求头中 *Content-Type* 的值与指定可消费的媒体类型中有相同的时候，请求才会被匹配。比如下面这个例子：

```
@Controller
@RequestMapping(path = "/pets", method = RequestMethod.POST, consumes="application/json")
public void addPet(@RequestBody Pet pet, Model model) {
    // 方法实现省略
}
```



#### **可生产的媒体类型**

你可以指定一组可生产的媒体类型，缩小映射的范围。这样只有当请求头中 *Accept* 的值与指定可生产的媒体类型中有相同的时候，请求才会被匹配。而且，使用 *produces* 条件可以确保用于生成响应（response）的内容与指定的可生产的媒体类型是相同的。举个例子：

```
@Controller
@RequestMapping(path = "/pets/{petId}", method = RequestMethod.GET, produces = MediaType.APPLICATION_JSON_UTF8_VALUE)
@ResponseBody
public Pet getPet(@PathVariable String petId, Model model) {
    // 方法实现省略
}
```



#### **请求参数与请求头的值**

你可以筛选请求参数的条件来缩小请求匹配范围，比如`"myParam"`、`"!myParam"`及`"myParam=myValue"`等。前两个条件用于筛选存在/不存在某些请求参数的请求，第三个条件筛选具有特定参数值的请求。下面有个例子，展示了如何使用请求参数值的筛选条件：

```
@Controller
@RequestMapping("/owners/{ownerId}")
public class RelativePathUriTemplateController {

    @RequestMapping(path = "/pets/{petId}", method = RequestMethod.GET, params="myParam=myValue")
    public void findPet(@PathVariable String ownerId, @PathVariable String petId, Model model) {
        // 实际实现省略
    }
}
```

尽管，可以使用媒体类型的通配符（比如 *"content-type=text/\*"*）来匹配请求头 *Content-Type*和 *Accept*的值，但我们更推荐独立使用 *consumes*和 *produces*条件来筛选各自的请求。因为它们就是专门为区分这两种不同的场景而生的。



### **定义@RequestMapping注解的处理方法**

#### **支持的方法参数类型**

下面列出所有支持的方法参数类型：

- 请求或响应对象（Servlet API）。可以是任何具体的请求或响应类型的对象，比如，`ServletRequest`或`HttpServletRequest`对象等。
- `HttpSession`类型的会话对象（Servlet API）。使用该类型的参数将要求这样一个session的存在，因此这样的参数永不为`null`。



- `org.springframework.web.context.request.WebRequest`或`org.springframework.web.context.request.NativeWebRequest`。允许存取一般的请求参数和请求/会话范围的属性（attribute），同时无需绑定使用Servlet/Portlet的API
- 当前请求的地区信息`java.util.Locale`，由已配置的最相关的地区解析器解析得到。在MVC的环境下，就是应用中配置的`LocaleResolver`或`LocaleContextResolver`
- 与当前请求绑定的时区信息`java.util.TimeZone`（java 6以上的版本）/`java.time.ZoneId`（java 8），由`LocaleContextResolver`解析得到
- 用于存取请求正文的`java.io.InputStream`或`java.io.Reader`。该对象与通过Servlet API拿到的输入流/Reader是一样的
- 用于生成响应正文的`java.io.OutputStream`或`java.io.Writer`。该对象与通过Servlet API拿到的输出流/Writer是一样的
- `org.springframework.http.HttpMethod`。可以拿到HTTP请求方法
- 包装了当前被认证用户信息的`java.security.Principal`
- 带`@PathVariable`注解的方法参数，其存放了URI模板变量中的值。
- 带`@MatrixVariable`注解的方法参数，其存放了URI路径段中的键值对。
- 带`@RequestParam`注解的方法参数，其存放了Servlet请求中所指定的参数。参数的值会被转换成方法参数所声明的类型。
- 带`@RequestHeader`注解的方法参数，其存放了Servlet请求中所指定的HTTP请求头的值。参数的值会被转换成方法参数所声明的类型。
- 带`@RequestBody`注解的参数，提供了对HTTP请求体的存取。参数的值通过`HttpMessageConverter`被转换成方法参数所声明的类型。
- 带`@RequestPart`注解的参数，提供了对一个"multipart/form-data请求块（request part）内容的存取。
- `HttpEntity`类型的参数，其提供了对HTTP请求头和请求内容的存取。请求流是通过`HttpMessageConverter`被转换成entity对象的。
- `java.util.Map`/`org.springframework.io.Model`/`org.springframework.ui.ModelMap`类型的参数，用以增强默认暴露给视图层的模型(model)的功能。
- `org.springframework.web.servlet.mvc.support.RedirectAttributes`类型的参数，用以指定重定向下要使用到的属性集以及添加flash属性（暂存在服务端的属性，它们会在下次重定向请求的范围中有效）。
- 命令或表单对象，它们用于将请求参数直接绑定到bean字段（可能是通过setter方法）。你可以通过`@InitBinder`注解和/或`HanderAdapter`的配置来定制这个过程的类型转换。具体请参考`RequestMappingHandlerAdapter`类`webBindingInitializer`属性的文档。这样的命令对象，以及其上的验证结果，默认会被添加到模型model中，键名默认是该命令对象类的类名——比如，`some.package.OrderAddress`类型的命令对象就使用属性名`orderAddress`类获取。`ModelAttribute`注解可以应用在方法参数上，用以指定该模型所用的属性名
- `org.springframework.validation.Errors` / `org.springframework.validation.BindingResult`验证结果对象，用于存储前面的命令或表单对象的验证结果（紧接其前的第一个方法参数）。
- `org.springframework.web.bind.support.SessionStatus`对象，用以标记当前的表单处理已结束。这将触发一些清理操作：`@SessionAttributes`在类级别注解的属性将被移除
- `org.springframework.web.util.UriComponentsBuilder`构造器对象，用于构造当前请求URL相关的信息，比如主机名、端口号、资源类型（scheme）、上下文路径、servlet映射中的相对部分（literal part）等

在参数列表中，`Errors`或`BindingResult`参数必须紧跟在其所绑定的验证对象后面。这是因为，在参数列表中允许有多于一个的模型对象，Spring会为它们创建不同的`BindingResult`实例。因此，下面这样的代码是不能工作的：

BindingResult与@ModelAttribute错误的参数次序

```
@RequestMapping(method = RequestMethod.POST)
public String processSubmit(@ModelAttribute("pet") Pet pet, Model model, BindingResult result) { ... }

```

上例中，因为在模型对象`Pet`和验证结果对象`BindingResult`中间还插了一个`Model`参数，这是不行的。要达到预期的效果，必须调整一下参数的次序：

```
@RequestMapping(method = RequestMethod.POST)
public String processSubmit(@ModelAttribute("pet") Pet pet, BindingResult result, Model model) { ... }

```

对于一些带有`required`属性的注解（比如`@RequestParam`、`@RequestHeader`等），JDK 1.8的`java.util.Optional`可以作为被它们注解的方法参数。在这种情况下，使用`java.util.Optional`与`required=false`的作用是相同的。



#### **支持的方法返回类型**

以下是handler方法允许的所有返回类型：

- `ModelAndView`对象，其中model隐含填充了命令对象，以及注解了`@ModelAttribute`字段的存取器被调用所返回的值。

- `Model`对象，其中视图名称默认由`RequestToViewNameTranslator`决定，model隐含填充了命令对象以及注解了`@ModelAttribute`字段的存取器被调用所返回的值

- `Map`对象，用于暴露model，其中视图名称默认由`RequestToViewNameTranslator`决定，model隐含填充了命令对象以及注解了`@ModelAttribute`字段的存取器被调用所返回的值

- `View`对象。其中model隐含填充了命令对象，以及注解了`@ModelAttribute`字段的存取器被调用所返回的值。handler方法也可以增加一个`Model`类型的方法参数来增强model

- `String`对象，其值会被解析成一个逻辑视图名。其中，model将默认填充了命令对象以及注解了`@ModelAttribute`字段的存取器被调用所返回的值。handler方法也可以增加一个`Model`类型的方法参数来增强model

- `void`。如果处理器方法中已经对response响应数据进行了处理（比如在方法参数中定义一个`ServletResponse`或`HttpServletResponse`类型的参数并直接向其响应体中写东西），那么方法可以返回void。handler方法也可以增加一个`Model`类型的方法参数来增强model

- 如果处理器方法注解了`ResponseBody`，那么返回类型将被写到HTTP的响应体中，而返回值会被`HttpMessageConverters`转换成方法声明的参数类型。

- `HttpEntity`或`ResponseEntity`对象，用于提供对Servlet HTTP响应头和响应内容的存取。对象体会被`HttpMessageConverters`转换成响应流。

- `HttpHeaders`对象，返回一个不含响应体的response

- `Callable`对象。当应用希望异步地返回方法值时使用，这个过程由Spring MVC自身的线程来管理

- `DeferredResult`对象。当应用希望方法的返回值交由线程自身决定时使用

- `ListenableFuture`对象。当应用希望方法的返回值交由线程自身决定时使用

- `ResponseBodyEmitter`对象，可用它异步地向响应体中同时写多个对象，also supported as the body within a `ResponseEntity`

- `SseEmitter`对象，可用它异步地向响应体中写服务器端事件（Server-Sent Events）,also supported as the body within a `ResponseEntity`

- `StreamingResponseBody`对象，可用它异步地向响应对象的输出流中写东西。also supported as the body within a `ResponseEntity`

- 其他任何返回类型，都会被处理成model的一个属性并返回给视图，该属性的名称为方法级的`@ModelAttribute`所注解的字段名（或者以返回类型的类名作为默认的属性名）。model隐含填充了命令对象以及注解了`@ModelAttribute`字段的存取器被调用所返回的值

  ​



#### **使用@RequestParam将请求参数绑定至方法参数**

你可以使用`@RequestParam`注解将请求参数绑定到你控制器的方法参数上。

下面这段代码展示了它的用法：

```
@Controller
@RequestMapping("/pets")
@SessionAttributes("pet")
public class EditPetForm {
    // ...
    @RequestMapping(method = RequestMapping.GET)
    public String setupForm(@RequestParam("petId") int petId, ModelMap model) {
        Pet pet = this.clinic.loadPet(petId);
        model.addAttribute("pet", pet);
        return "petForm";
    }

    // ...
}
```

若参数使用了该注解，则该参数默认是必须提供的，但你也可以把该参数标注为非必须的：只需要将`@RequestParam`注解的`required`属性设置为`false`即可（比如，`@RequestParam(path="id", required=false)`）。

若所注解的方法参数类型不是`String`，则类型转换会自动地发生。

若`@RequestParam`注解的参数类型是`Map`或者`MultiValueMap`，则该Map中会自动填充所有的请求参数。



#### **使用@RequestBody注解映射请求体**

方法参数中的`@RequestBody`注解暗示了方法参数应该被绑定了HTTP请求体的值。举个例子：

```
@RequestMapping(path = "/something", method = RequestMethod.PUT)
public void handle(@RequestBody String body, Writer writer) throws IOException {
    writer.write(body);
}
```

请求体到方法参数的转换是由`HttpMessageConverter`完成的。`HttpMessageConverter`负责将HTTP请求信息转换成对象，以及将对象转换回一个HTTP响应体。

对于`@RequestBody`注解，`RequestMappingHandlerAdapter`提供了以下几种默认的`HttpMessageConverter`支持：

- `ByteArrayHttpMessageConverter`用以转换字节数组
- `StringHttpMessageConverter`用以转换字符串
- `FormHttpMessageConverter`用以将表格数据转换成`MultiValueMap`或从`MultiValueMap`中转换出表格数据
- `SourceHttpMessageConverter`用于`javax.xml.transform.Source`类的互相转换




#### **使用@ResponseBody注解映射响应体**

`@ResponseBody`注解与`@RequestBody`注解类似。`@ResponseBody`注解可被应用于方法上，标志该方法的返回值（更正，原文是return type，看起来应该是返回值）应该被直接写回到HTTP响应体中去（而不会被被放置到Model中或被解释为一个视图名）。举个例子：

#### **使用HTTP实体HttpEntity**

`HttpEntity`与`@RequestBody`和`@ResponseBody`很相似。除了能获得请求体和响应体中的内容之外，`HttpEntity`（以及专门负责处理响应的`ResponseEntity`子类）还可以存取请求头和响应头，像下面这样：

```
@RequestMapping("/something")
public ResponseEntity<String> handle(HttpEntity<byte[]> requestEntity) throws UnsupportedEncodingException {
    String requestHeader = requestEntity.getHeaders().getFirst("MyRequestHeader");
    byte[] requestBody = requestEntity.getBody();

    // do something with request header and body

    HttpHeaders responseHeaders = new HttpHeaders();
    responseHeaders.set("MyResponseHeader", "MyValue");
    return new ResponseEntity<String>("Hello World", responseHeaders, HttpStatus.CREATED);
}
```

上面这段示例代码先是获取了`MyRequestHeader`请求头的值，然后读取请求体的主体内容。读完以后往影响头中添加了一个自己的响应头`MyResponseHeader`，然后向响应流中写了字符串`Hello World`，最后把响应状态码设置为201（创建成功）。

与`@RequestBody`与`@ResponseBody`注解一样，Spring使用了`HttpMessageConverter`来对请求流和响应流进行转换。



#### **对方法使用@ModelAttribute注解**

注解在方法上的`@ModelAttribute`说明了方法的作用是用于添加一个或多个属性到model上。这样的方法能接受与`@RequestMapping`注解相同的参数类型，只不过不能直接被映射到具体的请求上。在同一个控制器中，注解了`@ModelAttribute`的方法实际上会在`@RequestMapping`方法之前被调用。以下是几个例子：

```
// Add one attribute
// The return value of the method is added to the model under the name "account"
// You can customize the name via @ModelAttribute("myAccount")

@ModelAttribute
public Account addAccount(@RequestParam String number) {
    return accountManager.findAccount(number);
}

// Add multiple attributes

@ModelAttribute
public void populateModel(@RequestParam String number, Model model) {
    model.addAttribute(accountManager.findAccount(number));
    // add more ...
}
```

`@ModelAttribute`方法通常被用来填充一些公共需要的属性或数据，比如一个下拉列表所预设的几种状态，或者宠物的几种类型，或者去取得一个HTML表单渲染所需要的命令对象，比如`Account`等。

留意`@ModelAttribute`方法的两种风格。在第一种写法中，方法通过返回值的方式默认地将添加一个属性；在第二种写法中，方法接收一个`Model`对象，然后可以向其中添加任意数量的属性。你可以在根据需要，在两种风格中选择合适的一种。



#### **在方法参数上使用@ModelAttribute注解**

注解在方法参数上的`@ModelAttribute`说明了该方法参数的值将由model中取得。如果model中找不到，那么该参数会先被实例化，然后被添加到model中。在model中存在以后，请求中所有名称匹配的参数都会填充到该参数中。这在Spring MVC中被称为数据绑定，一个非常有用的特性，节约了你每次都需要手动从表格数据中转换这些字段数据的时间。

```
@RequestMapping(path = "/owners/{ownerId}/pets/{petId}/edit", method = RequestMethod.POST)
public String processSubmit(@ModelAttribute Pet pet) { }
```

以上面的代码为例，这个Pet类型的实例可能来自哪里呢？有几种可能:

- 它可能因为`@SessionAttributes`注解的使用已经存在于model中
- 它可能因为在同个控制器中使用了`@ModelAttribute`方法已经存在于model中
- 它可能是由URI模板变量和类型转换中取得的
- 它可能是调用了自身的默认构造器被实例化出来的

`@ModelAttribute`方法常用于从数据库中取一个属性值，该值可能通过`@SessionAttributes`注解在请求中间传递。在一些情况下，使用URI模板变量和类型转换的方式来取得一个属性是更方便的方式。这里有个例子：

```
@RequestMapping(path = "/accounts/{account}", method = RequestMethod.PUT)
public String save(@ModelAttribute("account") Account account) {
}
```

上面这个例子中，model属性的名称（"account"）与URI模板变量的名称相匹配。如果你配置了一个可以将`String`类型的账户值转换成`Account`类型实例的转换器`Converter`，那么上面这段代码就可以工作的很好，而不需要再额外写一个`@ModelAttribute`方法。

下一步就是数据的绑定。`WebDataBinder`类能将请求参数——包括字符串的查询参数和表单字段等——通过名称匹配到model的属性上。成功匹配的字段在需要的时候会进行一次类型转换（从String类型到目标字段的类型），然后被填充到model对应的属性中。

进行了数据绑定后，则可能会出现一些错误，比如没有提供必须的字段、类型转换过程的错误等。若想检查这些错误，可以在注解了`@ModelAttribute`的参数紧跟着声明一个`BindingResult`参数：

```
@RequestMapping(path = "/owners/{ownerId}/pets/{petId}/edit", method = RequestMethod.POST)
public String processSubmit(@ModelAttribute("pet") Pet pet, BindingResult result) {
    if (result.hasErrors()) {
        return "petForm";
    }
}
```



#### **在请求之间使用@SessionAttributes注解，使用HTTP会话保存模型数据**

类型级别的`@SessionAttributes`注解声明了某个特定处理器所使用的会话属性。通常它会列出该类型希望存储到session或converstaion中的model属性名或model的类型名，一般是用于在请求之间保存一些表单数据的bean。

以下的代码段演示了该注解的用法，它指定了模型属性的名称

```
@Controller
@RequestMapping("/editPet.do")
@SessionAttributes("pet")
public class EditPetForm {
    // ...
}
```



#### **使用@RequestHeader注解映射请求头属性**

`@RequestHeader`注解能将一个方法参数与一个请求头属性进行绑定。

以下是一个请求头的例子：

```
    Host                    localhost:8080
    Accept                  text/html,application/xhtml+xml,application/xml;q=0.9
    Accept-Language         fr,en-gb;q=0.7,en;q=0.3
    Accept-Encoding         gzip,deflate
    Accept-Charset          ISO-8859-1,utf-8;q=0.7,*;q=0.7
    Keep-Alive              300

```

以下的代码片段展示了如何取得`Accept-Encoding`请求头和`Keep-Alive`请求头的值：

```
@RequestMapping("/displayHeaderInfo.do")
public void displayHeaderInfo(@RequestHeader("Accept-Encoding") String encoding,
        @RequestHeader("Keep-Alive") long keepAlive) {
    //...
}

```

若注解的目标方法参数不是`String`类型，则类型转换会自动进行。



#### **方法参数与类型转换**

从请求参数、路径变量、请求头属性或者cookie中抽取出来的`String`类型的值，可能需要被转换成其所绑定的目标方法参数或字段的类型（比如，通过`@ModelAttribute`将请求参数绑定到方法参数上）。如果目标类型不是`String`，Spring会自动进行类型转换。所有的简单类型诸如int、long、Date都有内置的支持。如果想进一步定制这个转换过程，你可以通过`WebDataBinder`，或者为`Formatters`配置一个`FormattingConversionService`来做到。



### **视图解析**

Spring原生支持JSP视图技术、Velocity模板技术和XSLT视图等。

有两个接口在Spring处理视图相关事宜时至关重要，分别是视图解析器接口`ViewResolver`和视图接口本身`View`。视图解析器`ViewResolver`负责处理视图名与实际视图之间的映射关系。视图接口`View`负责准备请求，并将请求的渲染交给某种具体的视图技术实现。

#### **使用ViewResolver接口解析视图**

Spring MVC中所有控制器的处理器方法都必须返回一个逻辑视图的名字，无论是显式返回（比如返回一个`String`、`View`或者`ModelAndView`）还是隐式返回（比如基于约定的返回）。

**表21.3 视图解析器**

|                  视图解析器                   |                    描述                    |
| :--------------------------------------: | :--------------------------------------: |
|      `AbstractCachingViewResolver`       | 一个抽象的视图解析器类，提供了缓存视图的功能。通常视图在能够被使用之前需要经过准备。继承这个基类的视图解析器即可以获得缓存视图的能力。 |
|            `XmlViewResolver`             | 视图解析器接口`ViewResolver`的一个实现，该类接受一个XML格式的配置文件。该XML文件必须与Spring XML的bean工厂有相同的DTD。默认的配置文件名是`/WEB-INF/views.xml`。 |
|       `ResourceBundleViewResolver`       | 视图解析器接口`ViewResolver`的一个实现，采用bundle根路径所指定的`ResourceBundle`中的bean定义作为配置。一般bundle都定义在classpath路径下的一个配置文件中。默认的配置文件名为`views.properties`。 |
|          `UrlBasedViewResolver`          | `ViewResolver`接口的一个简单实现。它直接使用URL来解析到逻辑视图名，除此之外不需要其他任何显式的映射声明。如果你的逻辑视图名与你真正的视图资源名是直接对应的，那么这种直接解析的方式就很方便，不需要你再指定额外的映射。 |
|      `InternalResourceViewResolver`      | `UrlBasedViewResolver`的一个好用的子类。它支持内部资源视图（具体来说，Servlet和JSP）、以及诸如`JstlView`和`TilesView`等类的子类。You can specify the view class for all views generated by this resolver by using `setViewClass(..)`。更多的细节，请见`UrlBasedViewResolver`类的java文档。 |
| `VelocityViewResolver` / `FreeMarkerViewResolver` | `UrlBasedViewResolver`下的实用子类，支持Velocity视图`VelocityView`（Velocity模板）和FreeMarker视图`FreeMarkerView`以及它们对应子类。 |
|     `ContentNegotiatingViewResolver`     | 视图解析器接口`ViewResolver`的一个实现，它会根据所请求的文件名或请求的`Accept`头来解析一个视图。更多细节请见[Spring MVC 内容协商视图解析器](https://www.w3cschool.cn/spring_mvc_documentation_linesh_translation/spring_mvc_documentation_linesh_translation-gal927rn.html)一小节。 |

可以举个例子，假设这里使用的是JSP视图技术，那么我们可以使用一个基于URL的视图解析器`UrlBasedViewResolver`。这个视图解析器会将URL解析成一个视图名，并将请求转交给请求分发器来进行视图渲染。

```
<bean id="viewResolver" class="org.springframework.web.servlet.view.UrlBasedViewResolver">
    <property name="viewClass" value="org.springframework.web.servlet.view.JstlView"/>
    <property name="prefix" value="/WEB-INF/jsp/"/>
    <property name="suffix" value=".jsp"/>
</bean>
```

若返回一个`test`逻辑视图名，那么该视图解析器会将请求转发到`RequestDispatcher`，后者会将请求交给`/WEB-INF/jsp/test.jsp`视图去渲染。

如果需要在应用中使用多种不同的视图技术，你可以使用`ResourceBundleViewResolver`：

```
<bean id="viewResolver"
        class="org.springframework.web.servlet.view.ResourceBundleViewResolver">
    <property name="basename" value="views"/>
    <property name="defaultParentView" value="parentView"/>
</bean>
```



### **视图链**

Spring支持同时使用多个视图解析器。因此，你可以配置一个解析器链，并做更多的事比如，在特定条件下覆写一个视图等。你可以通过把多个视图解析器设置到应用上下文(application context)中的方式来串联它们。如果需要指定它们的次序，那么设置`order`属性即可。请记住，order属性的值越大，该视图解析器在链中的位置就越靠后。

在下面的代码例子中，视图解析器链中包含了两个解析器：一个是`InternalResourceViewResolver`，它总是自动被放置在解析器链的最后；另一个是`XmlViewResolver`，它用来指定Excel视图。`InternalResourceViewResolver`不支持Excel视图。

```
<bean id="jspViewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver">
    <property name="viewClass" value="org.springframework.web.servlet.view.JstlView"/>
    <property name="prefix" value="/WEB-INF/jsp/"/>
    <property name="suffix" value=".jsp"/>
</bean>

<bean id="excelViewResolver" class="org.springframework.web.servlet.view.XmlViewResolver">
    <property name="order" value="1"/>
    <property name="location" value="/WEB-INF/views.xml"/>
</bean>

<!-- in views.xml -->
<beans>
    <bean name="report" class="org.springframework.example.ReportExcelView"/>
</beans>
```

如果一个视图解析器不能返回一个视图，那么Spring会继续检查上下文中其他的视图解析器。此时如果存在其他的解析器，Spring会继续调用它们，直到产生一个视图返回为止。如果最后所有视图解析器都不能返回一个视图，Spring就抛出一个`ServletException`。



### **Spring MVC 配置**

要启用MVC Java编程配置，你需要在其中一个注解了`@Configuration`的类上添加`@EnableWebMvc`注解：

```
@Configuration
@EnableWebMvc
public class WebConfig {
}
```

要启用XML命名空间，请在你的DispatcherServlet上下文中（如果没有定义任何DispatcherServlet上下文，那么就在根上下文中）添加一个`mvc:annotation-driven`元素：

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:mvc="http://www.springframework.org/schema/mvc"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/mvc
        http://www.springframework.org/schema/mvc/spring-mvc.xsd">

    <mvc:annotation-driven/>
</beans>
```

上面的简单的声明代码，就已经默认注册了一个`RequestMappingHandlerMapping`、一个`RequestMappingHandlerAdapter`，以及一个`ExceptionHandlerExceptionResolver`，以支持对使用了`@RequestMapping`、`@ExceptionHandler`及其他注解的控制器方法的请求处理。

同时，启用了以下的特性：

1. Spring 3风格的类型转换支持。这是使用一个配置的转换服务[ConversionService](http://docs.spring.io/spring-framework/docs/4.2.4.RELEASE/spring-framework-reference/html/validation.html#core-convert)实例，以及the JavaBeans PropertyEditors used for Data Binding.
2. 使用`@NumberFormat`对数字字段进行[格式化](http://docs.spring.io/spring-framework/docs/4.2.4.RELEASE/spring-framework-reference/html/validation.html#format)，类型转换由`ConversionService`实现
3. 使用`@DateTimeFormat`注解对`Date`、`Calendar`、`Long`及Joda Time类型的字段进行[格式化](http://docs.spring.io/spring-framework/docs/4.2.4.RELEASE/spring-framework-reference/html/validation.html#format)
4. 使用`@Valid`注解对`@Controller`输入进行[验证](http://docs.spring.io/spring-framework/docs/4.2.4.RELEASE/spring-framework-reference/html/mvc.html#mvc-config-validation)——前提是classpath路径下比如提供符合JSR-303规范的验证器
5. HTTP消息转换`HttpMessageConverter`的支持，对注解了`@RequestMapping`或`@ExceptionHandler`方法的`@RequestBody`方法参数或`@ResponseBody`返回值生效

下面给出了一份由`mvc:annotation-driven`注册可用的HTTP消息转换器的完整列表：

1. 转换字节数组的`ByteArrayHttpMessageConverter`
2. 转换字符串的`StringHttpMessageConverter`
3. `ResourceHttpMessageConverter`：`org.springframework.core.io.Resource`与所有媒体类型之间的互相转换
4. `SourceHttpMessageConverter`：从（到）`javax.xml.transform.Source`的转换
5. `FormHttpMessageConverter`：数据与`MultiValueMap`之间的互相转换
6. `Jaxb2RootElementHttpMessageConverter`：Java对象与XML之间的互相转换——该转换器在classpath路径下有JAXB2依赖并且没有Jackson 2 XML扩展时被注册
7. `MappingJackson2HttpMessageConverter`：从（到）JSON的转换——该转换器在classpath下有Jackson 2依赖时被注册
8. `MappingJackson2XmlHttpMessageConverter`：从（到）XML的转换——该转换器在classpath下有[Jackson 2 XML扩展](https://github.com/FasterXML/jackson-dataformat-xml)时被注册
9. `AtomFeedHttpMessageConverter`：Atom源的转换——该转换器在classpath路径下有Rome时被注册
10. `RssChannelHttpMessageConverter`：RSS源的转换——该转换器在classpath路径下有Rome时被注册

#### **转换与格式化**

数字的`Number`类型和日期`Date`类型的格式化是默认安装了的，包括`@NumberFormat`注解和`@DateTimeFormat`注解。

使用MVC命名空间时，``也会进行同样的默认配置。要注册定制的格式化器和转换器，只需要提供一个转换服务`ConversionService`：

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:mvc="http://www.springframework.org/schema/mvc"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/mvc
        http://www.springframework.org/schema/mvc/spring-mvc.xsd">

    <mvc:annotation-driven conversion-service="conversionService"/>

    <bean id="conversionService"
            class="org.springframework.format.support.FormattingConversionServiceFactoryBean">
        <property name="converters">
            <set>
                <bean class="org.example.MyConverter"/>
            </set>
        </property>
        <property name="formatters">
            <set>
                <bean class="org.example.MyFormatter"/>
                <bean class="org.example.MyAnnotationFormatterFactory"/>
            </set>
        </property>
        <property name="formatterRegistrars">
            <set>
                <bean class="org.example.MyFormatterRegistrar"/>
            </set>
        </property>
    </bean>

</beans>
```



#### **拦截器配置**

你可以配置处理器拦截器`HandlerInterceptors`或web请求拦截器`WebRequestInterceptors`等拦截器，并配置它们拦截所有进入容器的请求，或限定到符合特定模式的URL路径。在MVC XML命名空间下，则使用`<mvc:interceptors>`元素：

```
<mvc:interceptors>
    <bean class="org.springframework.web.servlet.i18n.LocaleChangeInterceptor"/>
    <mvc:interceptor>
        <mvc:mapping path="/**"/>
        <mvc:exclude-mapping path="/admin/**"/>
        <bean class="org.springframework.web.servlet.theme.ThemeChangeInterceptor"/>
    </mvc:interceptor>
    <mvc:interceptor>
        <mvc:mapping path="/secure/*"/>
        <bean class="org.example.SecurityInterceptor"/>
    </mvc:interceptor>
</mvc:interceptors>
```



#### **视图控制器**

以下是一个例子，展示了如何在MVC Java编程配置方式下将所有`"/"`请求直接转发给名字为`"home"`的视图：

在MVC XML命名空间下完成同样的配置，则使用`<mvc:view-controller>`元素：

```
<mvc:view-controller path="/" view-name="home"/>
```



#### **路径匹配配置**

在XML命名空间下实现同样的功能，可以使用`<mvc:path-matching>`元素：

```
    <mvc:annotation-driven>
        <mvc:path-matching
            suffix-pattern="true"
            trailing-slash="false"
            registered-suffixes-only="true"
            path-helper="pathHelper"
            path-matcher="pathMatcher"/>
    </mvc:annotation-driven>

    <bean id="pathHelper" class="org.example.app.MyPathHelper"/>
    <bean id="pathMatcher" class="org.example.app.MyPathMatcher"/>
```





## **SpringBoot**

入门：

IDEA Intellij使用Maven快速创建Spring Boot，Link: https://jingyan.baidu.com/article/574c521979f9be6c8d9dc1aa.html

pom.xml:

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>

	<groupId>com.springboot</groupId>
	<artifactId>demo</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<packaging>jar</packaging>

	<name>demo</name>
	<description>Demo project for Spring Boot</description>

	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.0.0.RELEASE</version>
		<relativePath/> <!-- lookup parent from repository -->
	</parent>

	<properties>
		<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
		<project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
		<java.version>1.8</java.version>
	</properties>

	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>
	</dependencies>

	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
		</plugins>
	</build>

</project>
```



main.java.com.springboot.demo.DemoApplication.java:

```
package com.springboot.demo;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@SpringBootApplication
public class DemoApplication {

	@RequestMapping("/")
	String home(){
		return "hello world!";
	}

	public static void main(String[] args) {
		SpringApplication.run(DemoApplication.class, args);
	}
}
```



### @SpringBootApplication注解

使用@SpringbootApplication注解可以解决根类或者配置类头上注解过多的问题，一个@SpringbootApplication相当于`@Configuration`, @EnableAutoConfiguration``和 ``@ComponentScan 并具有他们的默认属性值

@EnableAutoConfiguration注解

这个注释告诉SpringBoot“猜”你将如何想配置Spring,基于你已经添加jar依赖项。如果spring-boot-starter-web已经添加Tomcat和Spring MVC,这个注释自动将假设您正在开发一个web应用程序并添加相应的spring设置。

> 启动器和自动配置(Starters & Auto-Configuration)
> 自动配置旨在与“Starters”配合使用，但这两个概念不直接绑定。可以自由选择和选择起始者以外的`jar`依赖，**Spring Boot**仍将尽力自动配置应用程序。



### “main”方法

应用程序的最后一部分是主(main)方法。 这只是一个遵循Java约定的应用程序入口点的标准方法。`main`方法通过调用`run`来委托Spring Boot `SpringApplication`类。`SpringApplication`将引导应用程序，启动Spring，从而启动自动配置**Tomcat Web**服务器。需要传递`Example.class`作为`run`方法的参数来告诉`SpringApplication`，这是主要的Spring组件。`args`数组也被传递以暴露任何命令行参数。



spring boot启动，页面出现以下提示信息：

```
Whitelabel Error Page
This application has no explicit mapping for /error, so you are seeing this as a fallback.
```

solve：

1、出现这个提示的原因是Application启动类和要访问的URI地址不在同一个class或package中，将Application放在最外层，包含所有的子包即可，或在该Apllication类上使用@ComponentScan注解：

​	@ComponentScan( basePackageClasses = DatabaseConfig. class)

2、如果该类的注解是@Controller，同时，方法上不添加@ResponseBody出现该提示，加上注释则不提示。在这种情况下，将该类的@Controller替换成@RestController。

做测试单元时，遇到@SpringApplicationConfiguration(classes = MockServletContext.class)不能导包直接用：

这是低版本的注解，修改成@SpringBootTest(classes = MockServletContext.class)即可。



## **零碎笔记**

要启动Spring Boot，在含有main方法的class类上，必须包含@SpringBootApplication，

application.properties是Spring Boot的配置文件，建议使用application.yml配置文件形式；

@RestController，控制器相当于：

```
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Controller
@ResponseBody
```



配置文件内容注入到Class，在class中使用：

`@Value("${配置文件对应的变量}")`

`private String name;`



在配置文件中将多项变量放入一项内容中，在配置文件中使用：

`content: "name: ${name}, age: ${age}"`

class中使用：

`@Value("${content}")`

`private String content;`



在一个entity类中，将配置文件的内容一起注入到Class，使用:

`@ConfigurationProperties(prefix = "person")`

`public class person{`

`}`



在注入bean，使用@Autowired出现Could not autowire.No beans of...... 如：

`@Autowired`

`private Person person`

在该类上加入@Component即可解决。



如果映射两个URI，使用@RequestMapping(value = {"/hello","/hi"}, ...)集合的方式，可以使用@GetMapping和@PostMapping组合注解简化@RequestMapping;



Spring-Data-Jpa(Java persistence API),数据库管理;

创建接口 ：

public interface PersonRepository extends JpaRepository<Person, Integer>{

}

调用该接口，就可以直接实现MySql的相关命令操作：

@Autowired

private PersonRepository  personrepository

personrepository.findAll();

personrepository.save();

......



@Entity注解表示该类对应数据库的一张表



@Transactional作为数据库事务管理，该方法下的数据库操作多条数据时，在实现错误的情况下，数据库全部回滚；



@Valid表单验证,验证该person对象

```
public Person Add(@Valid Person person, bingdingResult bindingResult){
	if(bindingResult.hasErrors){
		sout...
	}
}
```



切面使用：

```
@Aspect
public class person{
	@Pointcut("execution(public * package )")
	public void log(){
      
	}

	@Before("log()")
	public void log(){
      sout....
	}
	
	@After("log()")
	public void doAfter(){
      sout....
	}
}

```





























































