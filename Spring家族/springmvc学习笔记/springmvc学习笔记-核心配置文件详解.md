# springmvc学习笔记(6)-核心配置文件详解

## SpringMVC的构成要素

### 1. 指定SpringMVC的入口程序（在web.xml中）

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

以一个Servlet作为入口程序是绝大多数MVC框架都遵循的基本设计方案。这里的DispatcherServlet被我们称之为核心分发器，是SpringMVC最重要的类之一，之后我们会对其单独展开进行分析。

### 2. 编写SpringMVC的核心配置文件（在[servlet-name]-servlet.xml中）

```
<beans xmlns="http://www.springframework.org/schema/beans"  
       xmlns:mvc="http://www.springframework.org/schema/mvc"  
       xmlns:context="http://www.springframework.org/schema/context"  
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
       xsi:schemaLocation="  
            http://www.springframework.org/schema/beans  
            http://www.springframework.org/schema/beans/spring-beans-3.1.xsd  
            http://www.springframework.org/schema/context   
            http://www.springframework.org/schema/context/spring-context-3.1.xsd  
            http://www.springframework.org/schema/mvc  
            http://www.springframework.org/schema/mvc/spring-mvc-3.1.xsd"   
       default-autowire="byName">  
      
    <!-- Enables the Spring MVC @Controller programming model -->  
    <mvc:annotation-driven />  
      
    <context:component-scan base-package="com.demo2do" />  
      
      
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">  
        <property name="prefix" value="/" />  
        <property name="suffix" value=".jsp" />  
    </bean>  
      
</beans>  
```

SpringMVC自身由众多不同的组件共同构成，而每一个组件又有众多不同的实现模式。这里的SpringMVC核心配置文件是定义SpringMVC行为方式的一个窗口，用于指定每一个组件的实现模式。

### 3. 编写控制(Controller)层的代码

```
@Controller  
@RequestMapping  
public class UserController {  
  
    @RequestMapping("/login")  
    public ModelAndView login(String name, String password) {  
       // write your logic here   
           return new ModelAndView("success");  
    }  
  
}  
```

控制（Controller）层的代码编写在一个Java文件中。我们可以看到这个Java文件是一个普通的Java类并不依赖于任何接口。只是在响应类和响应方法上使用了Annotation的语法将它与Http请求对应起来。



从这个例子中，我们实际上已经归纳了构成基于SpringMVC应用程序的最基本要素。它们分别是： 

- 入口程序 —— DispatcherServlet
- 核心配置 —— [servlet-name]-servlet.xml
- 控制逻辑 —— UserController

## SpringMVC的核心配置文件

SpringMVC的核心配置文件是构成SpringMVC应用程序的必要元素之一。

这是我们在讲有关SpringMVC的构成要素时就曾经提到过的一个重要结论。当时我们所说的另外两大必要元素就是DispatcherServlet和Controller。因而，SpringMVC的核心配置文件在整个应用程序中所起到的作用也是举足轻重的。这也就是我们在这里需要补充对这个文件进行详细分析的原因。 

作为Spring Framework的一部分，我们可以认为SpringMVC是整个Spring Framework的一个组件。因而两者的配置体系和管理体系完全相同也属情理之中。实际上，SpringMVC所采取的策略，就是借用Spring Framework强大的容器（ApplicationContext）功能，而绝非自行实现。

DispatcherServlet负责对WebApplicationContext进行初始化，而初始化的依据，就是这个SpringMVC的核心配置文件。所以，SpringMVC的核心配置文件的内容解读将揭开整个SpringMVC初始化主线的全部秘密。 

## **核心配置文件概览** 

配置文件的概况： 

xml代码：

```
<?xml version="1.0" encoding="UTF-8"?>  
<beans xmlns="http://www.springframework.org/schema/beans"  
       xmlns:mvc="http://www.springframework.org/schema/mvc"  
       xmlns:context="http://www.springframework.org/schema/context"  
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
       xsi:schemaLocation="http://www.springframework.org/schema/beans  
            http://www.springframework.org/schema/beans/spring-beans-3.1.xsd  
            http://www.springframework.org/schema/context   
            http://www.springframework.org/schema/context/spring-context-3.1.xsd  
            http://www.springframework.org/schema/mvc  
            http://www.springframework.org/schema/mvc/spring-mvc-3.1.xsd">  
  
    <!-- Enables the Spring MVC @Controller programming model -->  
    <mvc:annotation-driven />  
      
    <context:component-scan base-package="com.demo2do.sample.web.controller" />  
      
    <!-- Handles HTTP GET requests for /static/** by efficiently serving up static resources in the ${webappRoot}/static/ directory -->  
    <mvc:resources mapping="/static/**" location="/static/" />  
  
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">    
        <property name="prefix" value="/" />    
        <property name="suffix" value=".jsp" />    
    </bean>    
  
</beans>  
```

### 头部声明

```
<?xml version="1.0" encoding="UTF-8"?>  
<beans xmlns="http://www.springframework.org/schema/beans"  
       xmlns:mvc="http://www.springframework.org/schema/mvc"  
       xmlns:context="http://www.springframework.org/schema/context"  
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
       xsi:schemaLocation="http://www.springframework.org/schema/beans  
            http://www.springframework.org/schema/beans/spring-beans-3.1.xsd  
            http://www.springframework.org/schema/context   
            http://www.springframework.org/schema/context/spring-context-3.1.xsd  
            http://www.springframework.org/schema/mvc  
            http://www.springframework.org/schema/mvc/spring-mvc-3.1.xsd">  
      ......  
</beans>  
```

这个部分是整个SpringMVC核心配置文件的关键所在。这一段声明，被称之为Schema-based XML的声明部分。

### 组件定义

除了头部声明部分的其他配置部分，就是真正的组件定义部分。在这个部分中，我们可以看到两种不同类型的配置定义模式： 

**1. 基于Schema-based XML的配置定义模式** 

```
<mvc:annotation-driven />  
```

**2. 基于Traditional XML的配置定义模式** 

```
<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">    
       <property name="prefix" value="/" />    
       <property name="suffix" value=".jsp" />    
/bean>  
```

两种不同的组件定义模式，其目的是统一的：对SpringMVC中的组件进行声明，指定组件的行为方式。

虽然两种不同的组件定义模式的外在表现看上去有所不同，但是SpringMVC在对其进行解析时最终都会将其转化为组件的定义而加载到WebApplicationContext之中进行管理。所以我们需要理解的是蕴藏在配置背后的目的而非配置本身的形式。 

## 初始化日志的再分析

这次的分析，我们试图弄清楚以下问题： 

- Where —— 组件的声明在哪里？
- How —— 组件是如何被注册的？
- What —— 究竟哪些组件被注册了？

对于这三个问题的研究，我们需要结合日志和Schema based XML的运行机理来共同进行分析。

引用

```
[main] INFO /sample - Initializing Spring FrameworkServlet 'dispatcher' 
19:49:48,670  INFO XmlWebApplicationContext:495 - Refreshing WebApplicationContext for namespace 'dispatcher-servlet': startup date [Thu Feb 16 19:49:48 CST 2012]; parent: Root WebApplicationContext 
19:49:48,674  INFO XmlBeanDefinitionReader:315 - Loading XML bean definitions from class path resource [web/applicationContext-dispatcher.xml] 

## Schema定位和加载 (开始) ## 
// 这个部分的日志反应出刚才我们所分析的Schema-based XML的工作原理。这是其中的第一步：读取META-INF/spring.schemas的内容，加载schema定义。然后找到相应的NamespaceHandler，执行其实现类。 

19:49:48,676 DEBUG DefaultDocumentLoader:72 - Using JAXP provider [com.sun.org.apache.xerces.internal.jaxp.DocumentBuilderFactoryImpl] 
19:49:48,678 DEBUG PluggableSchemaResolver:140 - Loading schema mappings from [META-INF/spring.schemas] 
19:49:48,690 DEBUG PluggableSchemaResolver:118 - Found XML schema [http://www.springframework.org/schema/beans/spring-beans-3.1.xsd] in classpath: org/springframework/beans/factory/xml/spring-beans-3.1.xsd 
19:49:48,710 DEBUG PluggableSchemaResolver:118 - Found XML schema [http://www.springframework.org/schema/mvc/spring-mvc-3.1.xsd] in classpath: org/springframework/web/servlet/config/spring-mvc-3.1.xsd 
19:49:48,715 DEBUG PluggableSchemaResolver:118 - Found XML schema [http://www.springframework.org/schema/tool/spring-tool-3.1.xsd] in classpath: org/springframework/beans/factory/xml/spring-tool-3.1.xsd 
19:49:48,722 DEBUG PluggableSchemaResolver:118 - Found XML schema [http://www.springframework.org/schema/context/spring-context-3.1.xsd] in classpath: org/springframework/context/config/spring-context-3.1.xsd 

## Schema定位和加载 (结束) ## 

## NamespaceHandler执行阶段 (开始) ## 
// 这个部分的日志，可以帮助我们回答本节一开始所提出的两个问题。绝大多数的组件，都是在BeanDefinitionParser的实现类中使用编程的方式注册的。 

19:49:48,731 DEBUG DefaultBeanDefinitionDocumentReader:108 - Loading bean definitions 
19:49:48,742 DEBUG DefaultNamespaceHandlerResolver:156 - Loaded NamespaceHandler mappings: {...} 

19:49:48,886 DEBUG PathMatchingResourcePatternResolver:550 - Looking for matching resources in directory tree [D:\Work\Demo2do\Sample\target\classes\com\demo2do\sample\web\controller] 

// (a),(b)段：这个部分的日志区域彻底回答了本节一开始所提出的最后一个问题：一共有18个组件被注册，就是红色标记的那18个bean。 

(a):
19:49:48,896 DEBUG XmlBeanDefinitionReader:216 - Loaded 18 bean definitions from location pattern [classpath:web/applicationContext-dispatcher.xml] 


19:49:48,897 DEBUG XmlWebApplicationContext:525 - Bean factory for WebApplicationContext for namespace 'dispatcher-servlet': org.springframework.beans.factory.support.DefaultListableBeanFactory@495c998a: defining beans [[

(b)
1. org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerMapping#0,
2. org.springframework.format.support.FormattingConversionServiceFactoryBean#0, 
3. org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter#0, 
4. org.springframework.web.servlet.handler.MappedInterceptor#0, 
5. org.springframework.web.servlet.mvc.method.annotation.ExceptionHandlerExceptionResolver#0,
6. org.springframework.web.servlet.mvc.annotation.ResponseStatusExceptionResolver#0, 
7. org.springframework.web.servlet.mvc.support.DefaultHandlerExceptionResolver#0, 
8. org.springframework.web.servlet.handler.BeanNameUrlHandlerMapping, 
9. org.springframework.web.servlet.mvc.HttpRequestHandlerAdapter, 
10. org.springframework.web.servlet.mvc.SimpleControllerHandlerAdapter, 
11. blogController, 
12. userController, 
13. org.springframework.context.annotation.internalConfigurationAnnotationProcessor, 
14. org.springframework.context.annotation.internalAutowiredAnnotationProcessor, 
15. org.springframework.context.annotation.internalRequiredAnnotationProcessor, 
16. org.springframework.context.annotation.internalCommonAnnotationProcessor, 
17. org.springframework.web.servlet.resource.ResourceHttpRequestHandler#0, 
18. org.springframework.web.servlet.handler.SimpleUrlHandlerMapping#0 


]; parent: org.springframework.beans.factory.support.DefaultListableBeanFactory@6602e323 
19:49:48,949 DEBUG XmlWebApplicationContext:794 - Unable to locate MessageSource with name 'messageSource': using default [org.springframework.context.support.DelegatingMessageSource@4b2922f6] 
19:49:48,949 DEBUG XmlWebApplicationContext:818 - Unable to locate ApplicationEventMulticaster with name 'applicationEventMulticaster': using default [org.springframework.context.event.SimpleApplicationEventMulticaster@79b66b06] 
19:49:48,949 DEBUG UiApplicationContextUtils:85 - Unable to locate ThemeSource with name 'themeSource': using default [org.springframework.ui.context.support.DelegatingThemeSource@372c9557] 
19:49:49,154 DEBUG RequestMappingHandlerMapping:98 - Looking for request mappings in application context: WebApplicationContext for namespace 'dispatcher-servlet': startup date [Thu Feb 16 19:49:48 CST 2012]; parent: Root WebApplicationContext 
19:49:49,175  INFO RequestMappingHandlerMapping:188 - Mapped "{[/blog],methods=[],params=[],headers=[],consumes=[],produces=[],custom=[]}" onto 
public org.springframework.web.servlet.ModelAndView com.demo2do.sample.web.controller.BlogController.index() 
19:49:49,177  INFO RequestMappingHandlerMapping:188 - Mapped "{[/register],methods=[],params=[],headers=[],consumes=[],produces=[],custom=[]}" onto 
public org.springframework.web.servlet.ModelAndView com.demo2do.sample.web.controller.UserController.register(com.demo2do.sample.entity.User) 
19:49:49,180  INFO RequestMappingHandlerMapping:188 - Mapped "{[/login],methods=[],params=[],headers=[],consumes=[],produces=[],custom=[]}" onto 
public org.springframework.web.servlet.ModelAndView com.demo2do.sample.web.controller.UserController.login(java.lang.String,java.lang.String) 
19:49:49,632 DEBUG BeanNameUrlHandlerMapping:71 - Looking for URL mappings in application context: WebApplicationContext for namespace 'dispatcher-servlet': startup date [Thu Feb 16 19:49:48 CST 2012]; parent: Root WebApplicationContext 
19:49:49,924  INFO SimpleUrlHandlerMapping:314 - Mapped URL path [/static/**] onto handler 'org.springframework.web.servlet.resource.ResourceHttpRequestHandler#0' 

## NamespaceHandler执行阶段 (结束) ## 

19:49:49,956 DEBUG DispatcherServlet:627 - Unable to locate RequestToViewNameTranslator with name 'viewNameTranslator': using default [org.springframework.web.servlet.view.DefaultRequestToViewNameTranslator@4d16318b] 
19:49:49,980 DEBUG DispatcherServlet:667 - No ViewResolvers found in servlet 'dispatcher': using default 
19:49:49,986 DEBUG DispatcherServlet:689 - Unable to locate FlashMapManager with name 'flashMapManager': using default [org.springframework.web.servlet.support.DefaultFlashMapManager@1816daa9] 
19:49:49,986 DEBUG DispatcherServlet:523 - Published WebApplicationContext of servlet 'dispatcher' as ServletContext attribute with name [org.springframework.web.servlet.FrameworkServlet.CONTEXT.dispatcher] 
19:49:49,986  INFO DispatcherServlet:463 - FrameworkServlet 'dispatcher': initialization completed in 1320 ms 
19:49:49,987 DEBUG DispatcherServlet:136 - Servlet 'dispatcher' configured successfully
```

## 小结

本文所涉及到的话题，主要围绕着SpringMVC的核心配置问题展开。读者可以将本文作为上一篇文章的续篇，将两者结合起来阅读。因为从宏观上说，本文的话题实际上也属于初始化主线的一个部分。





























