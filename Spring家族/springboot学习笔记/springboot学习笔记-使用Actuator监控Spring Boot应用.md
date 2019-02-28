# springboot学习笔记-使用Actuator监控Spring Boot应用

Spring Boot是一个用于快速开发Java Web的框架，不需要太多的配置即可使用Spring的大量功能。Spring Boot遵循着“约定大于配置”的原则，许多功能使用默认的配置即可。但过于简单的配置过程会让我们在了解各种依赖，配置之间的关系过程上带来一些困难。在Spring Boot中，我们可以使用Actuator来监控应用，Actuator提供了一系列的RESTful API让我们可以更为细致的了解各种信息。

# 引入Actuator

```
<dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

# 配置Actuator

在application.yml中配置：

```
server:
  port: 8080
management:
  security:
    enabled: false #关掉安全认证
  port: 8088 #管理端口调整成8088
  context-path: /monitor #actuator的访问路径
endpoints:
  shutdown:
    enabled: true

info:
   app:
      name: spring-boot-actuator
      version: 1.0.0
```

配置中关闭了安全认证的功能，如果需要开启这个功能的话还需引入`spring-boot-starter-security`依赖。除了使用Spring Security来开启监控路径安全认证外，还可以使用Shiro对监控路径进行权限控制。

监控的端口和应用一致，配置`context-path`为`/monitor`，这样可以避免和自己应用的路径映射地址重复。

`endpoints.shutdown.enabled: true`提供了使用post请求来关闭Spring Boot应用的功能。

# Actuator接口列表

Actuator提供了13个接口，可以分为三大类：配置接口、度量接口和其它接口，具体如下表所示：

| HTTP 方法 | 路径              | 描述                                       |
| ------- | --------------- | ---------------------------------------- |
| GET     | /autoconfig     | 提供了一份自动配置报告，记录哪些自动配置条件通过了，哪些没通过          |
| GET     | /configprops    | 描述配置属性(包含默认值)如何注入Bean                    |
| GET     | /beans          | 描述应用程序上下文里全部的Bean，以及它们的关系                |
| GET     | /dump           | 获取线程活动的快照                                |
| GET     | /env            | 获取全部环境属性                                 |
| GET     | /env/{name}     | 根据名称获取特定的环境属性值                           |
| GET     | /health         | 报告应用程序的健康指标，这些值由HealthIndicator的实现类提供    |
| GET     | /info           | 获取应用程序的定制信息，这些信息由info打头的属性提供             |
| GET     | /mappings       | 描述全部的URI路径，以及它们和控制器(包含Actuator端点)的映射关系   |
| GET     | /metrics        | 报告各种应用程序度量信息，比如内存用量和HTTP请求计数             |
| GET     | /metrics/{name} | 报告指定名称的应用程序度量值                           |
| POST    | /shutdown       | 关闭应用程序，要求endpoints.shutdown.enabled设置为true |
| GET     | /trace          | 提供基本的HTTP请求跟踪信息(时间戳、HTTP头等)              |

启动示例项目，访问：`http://localhost:8088/monitor/autoconfig` 等接口就可以看到详细信息

# 定制Actuator

## 修改接口ID

每个Actuator接口都有一个ID用来决定接口的路径，比方说，`/beans`接口的默认ID就是beans。比如要修改`/beans`为 `/instances`，则设置如下：

```
endpoints:
  beans:
    id: instances

```

## 启用和禁用接口

虽然Actuator的接口都很有用，但你不一定需要全部这些接口。默认情况下，所有接口（除了`/shutdown`）都启用。比如要禁用 `/metrics` 接口，则可以设置如下：

```
endpoints:
  metrics:
    enabled: false
```

如果你只想打开一两个接口，那就先禁用全部接口，然后启用那几个你要的，这样更方便。

```
endpoints:
  enabled: false
  metrics:
    enabled: true
```
