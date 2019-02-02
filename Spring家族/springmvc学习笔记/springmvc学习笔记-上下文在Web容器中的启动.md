# springmvc学习笔记-上下文在Web容器中的启动

# IOC容器启动的基本过程

IOC容器的启动过程就是建立上下文的过程，该上下文是与ServletContext相伴而生，同时也是IOC容器在Web应用环境中的具体表现。由ContextLoaderListener启动的上下文为根上下文，在根上下文的基础上，还有一个与Web MVC相关的上下文用来保存控制器 (DispatcherServlet) 需要的MVC对象。

上下文体系的建立是由ContextLoader来完成的，具体过程如下：

 ![图片1](E:\笔记\Spring家族\resource\图片1.png)

在web.xml中，已经配置了ContextLoaderListener。具体的载入IOC容器的过程是由ContextLoaderListener交由ContextLoader来完成的。它们之间的关系是：

 ![图片2](E:\笔记\Spring家族\resource\图片2.png)



在ContextLoader中，完成了两个IOC容器建立的基本过程，一个是在Web容器中建立起双亲IOC容器，另一个是生成相应的WebApplicationContext并将其初始化。

# Web容器中的上下文设计

为了方便在web环境中使用IOC容器，Spring为Web应用提供了上下文的扩展接口WebApplicationContext满足启动过程的需要，这个接口的层次关系如下图：

 ![图片3](E:\笔记\Spring家族\resource\图片3.png)

这个接口定义了一个getServletContext方法。通过这个方法，相当于提供了一个Web容器级别的全局环境。

 ![图片4](E:\笔记\Spring家族\resource\图片4.png)

在启动过程中，Spring会使用一个默认的WebApplicationContext实现作为IOC容器。这个默认使用的IOC容器就是XmlWebApplicationContext。该实现在基本的WebApplicationContext功能基础上，增加了对Web环境和XML配置定义的处理。 ![图片5](E:\笔记\Spring家族\resource\图片5.png)

 ![图片6](E:\笔记\Spring家族\resource\图片6.png)

 ![图片7](E:\笔记\Spring家族\resource\图片7.png)

# ContextLoader的设计与实现

对于使用Spring的Web应用，可以指定在Web应用程序启动时载入IOC容器（root WebApplicationContext）。这个功能是由ContextLoaderListener类完成的。在Web容器中配置监听器。ContextLoaderListener通过使用ContextLoder来完成实际的WebApplicationContext，也就是IOC容器的初始化工作。

监听器是启动根IOC容器并把它载入到Web容器的主要功能模块。这里对启动器ContextLoaderListener的实现进行分析。

 在ContextLoaderListener中，实现的是ServletContextListener接口。它是ServletContext的监听者，如果ServletContext发生变化，会触发出相应的事件，而监听器一直对这些事件进行监听。

在服务器启动时，ServletContextListener的contextInitialized()方法被调用；服务器将要关闭时，contextDestroyed()方法被调用。 ![图片8](E:\笔记\Spring家族\resource\图片8.png)

具体的初始化工作交给ContextLoader来完成。 ![图片9](E:\笔记\Spring家族\resource\图片9.png)

 ![图片10](E:\笔记\Spring家族\resource\图片10.png)

具体的根上下文创建如下：

 ![图片11](E:\笔记\Spring家族\resource\图片11.png)

![图片12](E:\笔记\Spring家族\resource\图片12.png)

使用什么样的类作为上下文是在determineContextClass方法中确定的。 ![图片13](E:\笔记\Spring家族\resource\图片13.png)

这就是IOC容器在Web容器中的启动过程。在初始化这个上下文以后，该上下文会被存储到ServletContext中，这样就建立了一个全局的关于整个应用的上下文。同时，在启动Spring MVC时，我们还会看到这个上下文被以后的DispatcherServlet在进行自己持有的上下文初始化时，设置为DispatcherServlet自带的上下文的双亲上下文。

