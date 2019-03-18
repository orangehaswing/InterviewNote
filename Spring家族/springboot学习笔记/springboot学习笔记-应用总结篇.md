# springboot学习笔记-应用总结篇

## 共享 Session

分布式系统中，Session 共享有很多的解决方案，其中托管到缓存中应该是最常用的方案之一，

### 说明

Spring Session 提供了一套创建和管理 Servlet HttpSession 的方案。Spring Session 提供了集群 Session（Clustered Sessions）功能，默认采用外置的 Redis 来存储 Session 数据，以此来解决 Session 共享的问题。

## 模板引擎

- Thymeleaf 可以完全替代 JSP ，在有网络和无网络的环境下皆可运行，即它可以让美工在浏览器查看页面的静态效果，也可以让程序员在服务器查看带数据的动态页面效果。这是由于它支持 html 原型，然后在 html 标签里增加额外的属性来达到模板+数据的展示方式。
- Thymeleaf 开箱即用的特性。它提供标准和 Spring 标准两种语言。
- Thymeleaf 提供 Spring 标准语言和一个与 SpringMVC 完美集成的可选模块，可以快速的实现表单绑定、属性编辑器、国际化等功能。

### 设置不校验html标签

默认配置下，thymeleaf对.html的内容要求很严格，比如，如果少封闭符号/，就会报错而转到错误页。

通过设置thymeleaf模板可以解决这个问题，下面是具体的配置:

```
spring.thymeleaf.cache=false
spring.thymeleaf.mode=LEGACYHTML5
```

pom.xml

```
<dependency>
	<groupId>net.sourceforge.nekohtml</groupId>
	<artifactId>nekohtml</artifactId>
	<version>1.9.22</version>
</dependency>
```

## Mysql导入数据

### 使用Jpa

### 使用Spring JDBC



## 管理监控springboot

Spring Boot Actuator提供了对单个Spring Boot的监控，信息包含：应用状态、内存、线程、堆栈等等，比较全面的监控了Spring Boot应用的整个生命周期。

Spring Boot Admin 是一个管理和监控Spring Boot 应用程序的开源软件。每个应用都认为是一个客户端，通过HTTP或者使用 Eureka注册到admin server中进行展示，Spring Boot Admin UI部分使用AngularJs将数据展示在前端。Spring Boot Admin 是一个针对spring-boot的actuator接口进行UI美化封装的监控工具。

## Spring Boot 特性

- 使用 Spring 项目引导页面可以在几秒构建一个项目
- 方便对外输出各种形式的服务，如 REST API、WebSocket、Web、Streaming、Tasks
- 非常简洁的安全策略集成
- 支持关系数据库和非关系数据库
- 支持运行期内嵌容器，如 Tomcat、Jetty
- 强大的开发包，支持热启动
- 自动管理依赖
- 自带应用监控
- 支持各种 IED，如 IntelliJ IDEA 、NetBeans


## @SpringBootApplication 注解

@Configuration : 表示Application作为sprig配置文件存在 

@EnableAutoConfiguration: 启动spring boot内置的自动配置 

@ComponentScan : 扫描bean，路径为Application类所在package以及package下的子路径，这里为 com.lkl.springboot，在spring boot中bean都放置在该路径已经子路径下。

## 事件监听

1. 监听类实现ApplicationListener接口 
2. 将监听类添加到`SpringApplication`实例

## properties与yml

- *.properties属性文件；属于最常见的一种； 

- *.yml是yaml格式的文件，yaml是一种非常简洁的标记语言。以数据做为中心，能够被电脑识别的数据序列化格式，可读性高并且容易被人类阅读，容易和脚本语言]交互，用来表达资料序列的编程语言。

  ​

## 禁用特定的自动配置

```
@EnableAutoConfiguration(exclude = DataSourceAutoConfiguration.class)
```

## 注册自定义自动配置

```
org.springframework.boot.autoconfigure.EnableAutoConfiguration=com.baeldung.autoconfigure.CustomAutoConfiguration
```

## 在bean已经存在情况下退出

```
@Bean
@ConditionalOnMissingBean
public CustomService service() { ... }
```

## 部署为JAR和WAR

**War文件**包含全部Web应用程序。

使用插件

```
<plugin>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-maven-plugin</artifactId>
</plugin>
```

pom

```
<packaging>jar</packaging>
```

## 支持轻松绑定

环境属性的键不需要与属性名称完全匹配。这样的环境属性可以用驼峰camelCase，kebab-case，snake_case或大写字母写成，单词用下划线分隔。

## 集成测试

@SpringBootTest；使用JUnit 4，我们必须使用@RunWith（SpringRunner.class）修饰测试类。

## 修改端口

1. serve.port = 8004；在SpringBoot中有一个类：ServerProperties。@ConfigurationProperties注解，读取SpringBoot的默认配置文件application.properties的值注入到bean里。这里定义了一个server的前缀和一个port字段
2. 实现EmbeddedServletContainerCustomizer接口，修改。
3. 命令行参数
4. 使用虚拟机参数


## 监视器









