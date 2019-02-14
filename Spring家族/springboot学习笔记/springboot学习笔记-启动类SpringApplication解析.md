# springboot学习笔记-启动类SpringApplication解析

## SpringBootApplication

@SpringBootApplication注解主要组合了

- @Configuration
- @EnableAutoConfiguration:让Spring Boot根据类路径中的jar包依赖为当前项目进行自动配置
- @ComponentScan

SpringBoot提供了内嵌Servlet容器并且提供了非常简单的应用启动方式：SpringApplication.run()

在任意的main方法中使用SpringApplication.run()即可完成应用的引导和启动.
那么接下来为大家介绍一下它是如何启动的、它都做了些什么.

默认情况下引导和启动应用的步骤如下:

- 创建合适的ApplicationContext(这取决于你的classpath–WebApplicationType.deduceFromClassPath从classpath推断应用类型,这将在下面代码中介绍到)
- 注册CommandLinePropertySource用来暴露命令行参数(command line arguments)和Spring properties
- 刷新ApplicationContext,loading所有单例bean
- 触发所有的CommandLineRunner beans

大多数情况下我们在main方法中是这样引导和启动应用的:

```
public static void main(String[] args) throws Exception {
     SpringApplication.run(MyApplication.class, args);
}

```

- 创建SpringApplication实例,并调用run方法.
- 直接使用SpringApplication中的main方法启动

通过SpringApplication的静态run方法,那我们接着来看下这个静态方法做了什么

```
public static ConfigurableApplicationContext run(Class<?>[] primarySources,String[] args) {
    return new SpringApplication(primarySources).run(args);
}

```

它创建了一个SpringApplication实例并调用了实例中的公有run方法,并将main方法的参数传递过去(这些参数将会用来运行CommandLineRunner beans)
我们进去实例方法中看一下做了什么

```
public SpringApplication(Class<?>... primarySources) {
    this(null, primarySources);
}

```

这是一个重载函数,它最终会进入

```
public SpringApplication(ResourceLoader resourceLoader, Class<?>... primarySources) {
    this.resourceLoader = resourceLoader;
    Assert.notNull(primarySources, "PrimarySources must not be null");
    this.primarySources = new LinkedHashSet<>(Arrays.asList(primarySources));
    this.webApplicationType = WebApplicationType.deduceFromClasspath();
    setInitializers((Collection) getSpringFactoriesInstances(ApplicationContextInitializer.class));
    setListeners((Collection) getSpringFactoriesInstances(ApplicationListener.class));
    this.mainApplicationClass = deduceMainApplicationClass();
}

```

在第5行我们可以看到上面提到过的应用类型推断函数WebApplicationType.deduceFromClasspath(),具体是如何进行推断的如下

```
static WebApplicationType deduceFromClasspath() {
    if (ClassUtils.isPresent(WEBFLUX_INDICATOR_CLASS, null)
            && !ClassUtils.isPresent(WEBMVC_INDICATOR_CLASS, null)
            && !ClassUtils.isPresent(JERSEY_INDICATOR_CLASS, null)) {
        return WebApplicationType.REACTIVE;
    }
    for (String className : SERVLET_INDICATOR_CLASSES) {
        if (!ClassUtils.isPresent(className, null)) {
            return WebApplicationType.NONE;
        }
    }
    return WebApplicationType.SERVLET;
}

```

例如其中的 WEBFLUX_INDICATOR_CLASS ,它的值是`org.springframework.web.reactive.DispatcherHandler`,也就是说系统去尝试加载此类以及其依赖类,如果加载不成功那么它就不是此应用类型
我们在回到SpringApplication的重载函数观察第6、7行, getSpringFactoriesInstances 是根据指定的类类型从META/INF/spring.factories加载bean,也就是说第6、7行加载了初始化类以及监听器,为了更加直观,下面给出spring.factories文件

```
# Application Context Initializers
org.springframework.context.ApplicationContextInitializer=
org.springframework.boot.context.ConfigurationWarningsApplicationContextInitializer,
org.springframework.boot.context.ContextIdApplicationContextInitializer,
org.springframework.boot.context.config.DelegatingApplicationContextInitializer,
org.springframework.boot.web.context.ServerPortInfoApplicationContextInitializer

# Application Listeners
org.springframework.context.ApplicationListener=
org.springframework.boot.ClearCachesApplicationListener,
org.springframework.boot.builder.ParentContextCloserApplicationListener,
org.springframework.boot.context.FileEncodingApplicationListener,
org.springframework.boot.context.config.AnsiOutputApplicationListener,
org.springframework.boot.context.config.ConfigFileApplicationListener,
org.springframework.boot.context.config.DelegatingApplicationListener,
org.springframework.boot.context.logging.ClasspathLoggingApplicationListener,
org.springframework.boot.context.logging.LoggingApplicationListener,
org.springframework.boot.liquibase.LiquibaseServiceLocatorApplicationListener

```

我们在回到SpringApplication的重载函数观察第8行,它从stack trace中推断main方法所在的类并加载返回,至此SpringApplication构造器已经完成它的工作,在前面提到的SpringApplication静态run方法中已经完成了构造,紧接着就是调用了公用run方法

```
public ConfigurableApplicationContext run(String... args) {
    StopWatch stopWatch = new StopWatch();//用来计时的
    stopWatch.start();
    ConfigurableApplicationContext context = null;
    Collection<SpringBootExceptionReporter> exceptionReporters = new ArrayList<>();
    configureHeadlessProperty();
    SpringApplicationRunListeners listeners = getRunListeners(args);
    listeners.starting();
    try {
        ApplicationArguments applicationArguments = new DefaultApplicationArguments(args);
        ConfigurableEnvironment environment = prepareEnvironment(listeners,applicationArguments);
        configureIgnoreBeanInfo(environment);
        Banner printedBanner = printBanner(environment);//  打印Banner用的对象
        context = createApplicationContext();
        // 从spring.facroties加载了 Error Reporters
        exceptionReporters = getSpringFactoriesInstances(SpringBootExceptionReporter.class,
        					new Class[] { ConfigurableApplicationContext.class }, context);
        prepareContext(context, environment, listeners, applicationArguments,printedBanner);
        refreshContext(context);
        afterRefresh(context, applicationArguments);
        stopWatch.stop();// 表停了
        if (this.logStartupInfo) {// 记日志
            new StartupInfoLogger(this.mainApplicationClass)
                    .logStarted(getApplicationLog(), stopWatch);
        }
        listeners.started(context);
        callRunners(context, applicationArguments);
    }
    catch (Throwable ex) {
        handleRunFailure(context, ex, exceptionReporters, listeners);
        throw new IllegalStateException(ex);
    }

    try {
        listeners.running(context);
    }
    catch (Throwable ex) {
        handleRunFailure(context, ex, exceptionReporters, null);
        throw new IllegalStateException(ex);
    }
    return context;
}
```

首先在第4行创建了用来返回的可配置应用上下文对象 ConfigurableApplicationContext
在第7行处从spring.factories加载了所有runListeners

```
# Run Listeners
org.springframework.boot.SpringApplicationRunListener=
org.springframework.boot.context.event.EventPublishingRunListener

```

内部有类 SimpleApplicationEventMulticaster 用来发布事件用的,继承了SpringRunApplicationListener。用来多路广播事件,事件如下:

- starting 首次调用run方法启动时立即调用
- enviromentPrepared 环境已经准备完毕时调用,ApplicationContext被创建之前
- contextPrepared ApplicationContext被创建且准备完毕调用,在资源加载之前
- contextLoaded 在ApplicationContext加载之后,刷新之前调用
- started 上下文被刷新应用启动完毕调用,但在CommandLineRunners和ApplicationRunners被调用之前
- running 在run方法结束前立即调用,此时应用上下文已被刷新,且CommandLineRunners和ApplicationRunners已被调用
- failed 顾名思义,当启动应用出错时调用

请注意此处的广播事件排序,在run方法启动时会以此顺序发布事件,(除了fail以外)。

接着分析run方法
刚看完SpringApplicationRunListeners,紧接着run方法第8行我们就发现立刻调用了starting()方法发布了 ApplicationStartingEvent ,这就对应了我们刚刚说到的starting调用时机,Called immediately when the run method has first started. Can be used for very early initialization.

```
@Override
public void starting() {
    this.initialMulticaster.multicastEvent(new ApplicationStartingEvent(this.application, this.args));
}
//  其他事件发布方法同理,不同的只是 事件对象 ,比如这里是ApplicationStartingEvent

```

之后在run方法第11行调用 prepareEnvironment() 创建了应用环境对象,我们来看一下

```
private ConfigurableEnvironment prepareEnvironment(
            SpringApplicationRunListeners listeners,
            ApplicationArguments applicationArguments) {
    // 创建和配置应用环境
    ConfigurableEnvironment environment = getOrCreateEnvironment();
    configureEnvironment(environment, applicationArguments.getSourceArgs());
    //  发布enviromentPrepared事件,对应事件顺序2,也就是说环境已经准备完毕,上下文被创建之前
    listeners.environmentPrepared(environment);
    //  将应用环境绑定到SpringApplication
    bindToSpringApplication(environment);
    if (!this.isCustomEnvironment) {
        environment = new EnvironmentConverter(getClassLoader())
                .convertEnvironmentIfNecessary(environment, deduceEnvironmentClass());
    }
    ConfigurationPropertySources.attach(environment);
    return environment;
}

```

run方法第14行调用了 createApplicationContext() 方法根据应用类型创建了对应的默认应用上下文对象,如 SERVLET 类型将创建`org.springframework.boot.web.servlet.context.AnnotationConfigServletWebServerApplicationContext`

```
protected ConfigurableApplicationContext createApplicationContext() {
        Class<?> contextClass = this.applicationContextClass;
    if (contextClass == null) {
        try {
            switch (this.webApplicationType) {
            case SERVLET:
                contextClass = Class.forName(DEFAULT_SERVLET_WEB_CONTEXT_CLASS);
                break;
            case REACTIVE:
                contextClass = Class.forName(DEFAULT_REACTIVE_WEB_CONTEXT_CLASS);
                break;
            default:
                contextClass = Class.forName(DEFAULT_CONTEXT_CLASS);
            }
        }
        catch (ClassNotFoundException ex) {
            throw new IllegalStateException(
                    "Unable create a default ApplicationContext, "
                            + "please specify an ApplicationContextClass",
                    ex);
        }
    }
    return (ConfigurableApplicationContext) BeanUtils.instantiateClass(contextClass);
}

```

run方法第16行调用了 prepareContext() 方法,如下:

```
private void prepareContext(ConfigurableApplicationContext context,
            ConfigurableEnvironment environment, SpringApplicationRunListeners listeners,
            ApplicationArguments applicationArguments, Banner printedBanner) {
    context.setEnvironment(environment);	// 填充应用环境到上下文
    postProcessApplicationContext(context);	// 应用上下文后处理
   	// 运行所有实现了ApplicationContextInitializer的bean
   	//(在构造函数中从spring.factories中加载过,也可添加自己的,通过addInitializers方法)
   	applyInitializers(context);
    listeners.contextPrepared(context);	// 发布contextPrepared事件,对应事件顺序3
    if (this.logStartupInfo) {
        logStartupInfo(context.getParent() == null);
        logStartupProfileInfo(context);
    }

    // Add boot specific singleton beans
    // 注册在run方法中创建了的 ApplicationArguments 的单例
    context.getBeanFactory().registerSingleton("springApplicationArguments",applicationArguments);
    if (printedBanner != null) {
        // 注册 Bannner 的单例
        context.getBeanFactory().registerSingleton("springBootBanner", printedBanner);
    }

    // Load the sources
    // getAllSource方法返回primarySources
    // (比如静态run方法的入参一)以及sources(应用环境Enviroment中的PropertySource)
    Set<Object> sources = getAllSources();
    Assert.notEmpty(sources, "Sources must not be empty");
    load(context, sources.toArray(new Object[0]));
    listeners.contextLoaded(context);// 发布contextLoaded事件,对应事件顺序4
}
```

run方法第17行调用了 refreshContext() 方法,如下:

```
private void refreshContext(ConfigurableApplicationContext context) {
    refresh(context);	// 其实调用了应用上下文对象的refresh方法,里面做了很多事情
    //如发布上下文更新事件
    if (this.registerShutdownHook) {
        try {
            context.registerShutdownHook();	// 注册应用关闭钩子,用于清理资源,调用AbstractApplicationContext#doClose()
        }
        catch (AccessControlException ex) {
            // Not allowed in some environments.
        }
    }
}
```

run方法第18行调用了 afterRefresh() 方法,如下:

```
//  没有默认实现,你可以继承此类并override此方法,比如下下面
//  你可以考虑override更多的方法,来实现自己的逻辑或者控制
protected void afterRefresh(ConfigurableApplicationContext context,
            ApplicationArguments args) {
}

public class MySpringApplication extends SpringApplication {

    private final Log logger = LogFactory.getLog(MySpringApplication.class);

    @Override
    protected void afterRefresh(ConfigurableApplicationContext context, ApplicationArguments args) {
        super.afterRefresh(context, args);
        logger.info("[Spring application refreshed]");
    }

    public MySpringApplication(Class<?>... primarySources) {
        super(primarySources);
    }

}

```

run方法第24行发布了 ApplicationStartedEvent 事件,listeners.started(context);,对应事件顺序5
run方法第25行调用了 callRunners() 方法,如下:

```
private void callRunners(ApplicationContext context, ApplicationArguments args) {
    //  调用所有的CommandLineRunners和ApplicationRunners
    List<Object> runners = new ArrayList<>();
    runners.addAll(context.getBeansOfType(ApplicationRunner.class).values());
    runners.addAll(context.getBeansOfType(CommandLineRunner.class).values());
    AnnotationAwareOrderComparator.sort(runners);
    for (Object runner : new LinkedHashSet<>(runners)) {
        if (runner instanceof ApplicationRunner) {
            callRunner((ApplicationRunner) runner, args);
        }
        if (runner instanceof CommandLineRunner) {
            callRunner((CommandLineRunner) runner, args);
        }
    }
}

```

run方法第33行发布了 ApplicationReadyEvent 事件,listeners.running(context);对应事件顺序6

最后没错的话返回上下文对象,应用启动完毕.

文中提到的ApplicationContextInitializer

```
@Order(Ordered.HIGHEST_PRECEDENCE)
public class ApplicationContextInit implements ApplicationContextInitializer {

    private final Logger logger = LogManager.getLogger(ApplicationContextInit.class);

    /**
     * spring.factories文件中定义了一系列PropertySource Loaders/Run Listeners/Error Reporters/
     * Application Context Initializers/Application Listeners/Environment Post Processors/
     * Failure Analyzers/FailureAnalysisReporters
     * 1.可以将此类加在spring.factories中的Application Context Initializers后启用(对所有应用生效)
     * 2.calling addInitializers() on SpringApplication before run it(对当前应用生效)
     * 3.通过配置context.initializer.classes(对当前应用生效)
     * 以上配置适用于listeners
     *
     * @param configurableApplicationContext
     */
    @Override
    public void initialize(ConfigurableApplicationContext configurableApplicationContext) {
        ConfigurableEnvironment environment = configurableApplicationContext.getEnvironment();
        Map<String, Object> systemEnvironment = environment.getSystemEnvironment();
        for (Map.Entry<String, Object> entry : systemEnvironment.entrySet()) {
            logger.info(entry.getKey() + ":" + entry.getValue());
        }
    }
}
```





