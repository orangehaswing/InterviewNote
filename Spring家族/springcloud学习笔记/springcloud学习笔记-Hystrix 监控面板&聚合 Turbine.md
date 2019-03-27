# springcloud学习笔记-Hystrix 监控面板

断路器是根据一段时间窗内的请求情况来判断并操作断路器的打开和关闭状态的。而这些请求情况的指标信息都是 HystrixCommand 和 HystrixObservableCommand 实例在执行过程中记录的重要度量信息，它们除了 Hystrix 断路器实现中使用之外，对于系统运维也有非常大的帮助。这些指标信息会以 “滚动时间窗” 与 “桶” 结合的方式进行汇总，并在内存中驻留一段时间，以供内部或外部进行查询使用。

下面我们基于之前的示例来结合 Hystrix Dashboard 实现 Hystrix 指标数据的可视化面板，这里我们将用到下之前实现的几个应用，包括：

- eureka-server：服务注册中心
- eureka-producer：服务提供者
- eureka-consumer-feign-hystrix：使用 Feign 和 Hystrix 实现的服务消费者

# 创建 Hystrix Dashboard

创建一个标准的 Spring Boot 工程，命名为：hystrix-dashboard

## POM 配置

在 pom.xml 引入相关的依赖

```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-hystrix-dashboard</artifactId>
</dependency>

```

## 启动类

在 Spring Boot 的启动类上面引入注解`@EnableHystrixDashboard`，启用 Hystrix Dashboard 功能。

```
@EnableHystrixDashboard
@SpringBootApplication
public class HystrixDashboardApplication {

    public static void main(String[] args) {
        SpringApplication.run(HystrixDashboardApplication.class, args);
    }
}

```

## 配置文件

修改配置文件 application.yml

```
spring:
  application:
    name: hystrix-dashboard
server:
  port: 11000

```

启动应用，然后再浏览器中输入 [http://localhost:11000/hystrix](http://localhost:11000/hystrix) 可以看到如下界面
[![img](https://ws2.sinaimg.cn/large/006tNc79ly1fqea896gthj31kw0v9n3a.jpg)](https://ws2.sinaimg.cn/large/006tNc79ly1fqea896gthj31kw0v9n3a.jpg)
通过 Hystrix Dashboard 主页面的文字介绍，我们可以知道，Hystrix Dashboard 共支持三种不同的监控方式：

- 默认的集群监控：通过 URL：[http://turbine-hostname:port/turbine.stream](http://turbine-hostname:port/turbine.stream) 开启，实现对默认集群的监控。
- 指定的集群监控：通过 URL：[http://turbine-hostname:port/turbine.stream?cluster=[clusterName\]](http://turbine-hostname:port/turbine.stream?cluster=[clusterName]) 开启，实现对 clusterName 集群的监控。
- 单体应用的监控： ~~通过 URL：http://hystrix-app:port/hystrix.stream 开启~~ ，实现对具体某个服务实例的监控。**（现在这里的 URL 应该为 http://hystrix-app:port/actuator/hystrix.stream，Actuator 2.x 以后  endpoints 全部在/actuator下，可以通过management.endpoints.web.base-path修改）**

前两者都对集群的监控，需要整合 Turbine 才能实现。这一部分我们先实现对单体应用的监控，这里的单体应用就用我们之前使用 Feign 和 Hystrix 实现的服务消费者——eureka-consumer-feign-hystrix。

页面上的另外两个参数：

- Delay：控制服务器上轮询监控信息的延迟时间，默认为 2000 毫秒，可以通过配置该属性来降低客户端的网络和 CPU 消耗。
- Title：该参数可以展示合适的标题。

# 为服务实例添加 endpoint

既然 Hystrix Dashboard 监控单实例节点需要通过访问实例的`/actuator/hystrix.stream`接口来实现，自然我们需要为服务实例添加这个 endpoint。

## POM 配置

在服务实例`pom.xml`中的`dependencies`节点中新增`spring-boot-starter-actuator`监控模块以开启监控相关的端点，并确保已经引入断路器的依赖`spring-cloud-starter-netflix-hystrix`

```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>

```

## 启动类

为启动类添加`@EnableCircuitBreaker`或`@EnableHystrix`注解，开启断路器功能。

```
@EnableHystrix
@EnableFeignClients
@SpringBootApplication
public class EurekaConsumerHystrixApplication {

    public static void main(String[] args) {
        SpringApplication.run(EurekaConsumerHystrixApplication.class, args);
    }
}

```

## 配置文件

在配置文件 application.yml 中添加

```
management:
  endpoints:
    web:
      exposure:
        include: hystrix.stream

```

`management.endpoints.web.exposure.include`这个是用来暴露 endpoints 的。由于 endpoints 中会包含很多敏感信息，除了 health 和 info 两个支持 web 访问外，其他的默认不支持 web 访问。

## 测试

在 Hystrix-Dashboard 的主界面上输入 eureka-consumer-feign-hystrix 对应的地址 [http://localhost:9004/actuator/hystrix.stream](http://localhost:9004/actuator/hystrix.stream) 然后点击 Monitor Stream 按钮，进入页面。

如果没有请求会一直显示 “Loading…”，这时访问 [http://localhost:9004/actuator/hystrix.stream](http://localhost:9004/actuator/hystrix.stream) 也是不断的显示 “ping”。

这时候访问一下 [http://localhost:9004/hello/windmt](http://localhost:9004/hello/windmt)，可以看到 Hystrix Dashboard 中出现了类似下面的效果
[![img](https://ws2.sinaimg.cn/large/006tKfTcly1fr2mou2nhfj30sg0brwgm.jpg)](https://ws2.sinaimg.cn/large/006tKfTcly1fr2mou2nhfj30sg0brwgm.jpg)

如果在这个页面看到报错：`Unable to connect to Command Metric Stream.`，可以参考这个 [Issue](https://github.com/Netflix/Hystrix/issues/1566) 解决。

# 界面解读

[![img](https://ws1.sinaimg.cn/large/006tNc79ly1fqefia3o64j30h90ae754.jpg)](https://ws1.sinaimg.cn/large/006tNc79ly1fqefia3o64j30h90ae754.jpg)
以上图来说明其中各元素的具体含义：

- 实心圆：它有颜色和大小之分，分别代表实例的监控程度和流量大小。如上图所示，它的健康度从绿色、黄色、橙色、红色递减。通过该实心圆的展示，我们就可以在大量的实例中快速的发现故障实例和高压力实例。
- 曲线：用来记录 2 分钟内流量的相对变化，我们可以通过它来观察到流量的上升和下降趋势。
- 其他一些数量指标如下图所示
  [![img](https://ws2.sinaimg.cn/large/006tNc79ly1fqeflypfdaj30o80cldhq.jpg)](https://ws2.sinaimg.cn/large/006tNc79ly1fqeflypfdaj30o80cldhq.jpg)

到此单个应用的熔断监控已经完成。

# Hystrix 监控数据聚合 Turbine

通过 Hystrix Dashboard，我们可以查看服务调用次数、服务调用延迟等。在生产环境我们的服务是肯定需要做高可用的，那么对于多实例的情况，我们就需要将这些度量指标数据进行聚合。

# 准备工作

在开始使用 Turbine 之前，我们先回顾一下上一篇中实现的架构，如下图所示：

[![img](https://ws3.sinaimg.cn/large/006tNc79ly1fqekbdszlgj30vm0eq75c.jpg)](https://ws3.sinaimg.cn/large/006tNc79ly1fqekbdszlgj30vm0eq75c.jpg)

其中，我们构建的内容包括：

- eureka-server：服务注册中心
- eureka-producer：服务提供者
- eureka-consumer-hystrix：使用 Feign 和 Hystrix 实现的服务消费者
- hystrix-dashboard：用于展示`eureka-consumer-hystrix`服务的 Hystrix 数据

# 创建 Turbine

在上述架构基础上，引入 Turbine 来对服务的 Hystrix 数据进行聚合展示两种聚合方式。

## 通过 HTTP 收集聚合

创建一个标准的 Spring Boot 工程，命名为：turbine。

### POM 配置

在 pom.xml 中添加以下依赖

```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-turbine</artifactId>
</dependency>

```

### 启动类

在启动类上使用`@EnableTurbine`注解开启 Turbine

```
@EnableTurbine
@SpringBootApplication
public class TurbineApplication {

    public static void main(String[] args) {
        SpringApplication.run(TurbineApplication.class, args);
    }
}

```

### 配置文件

在 application.yml 加入 Eureka 和 Turbine 的相关配置

```
spring:
  application:
    name: turbine
server:
  port: 8080
management:
  port: 8081
eureka:
  client:
    service-url:
      defaultZone: http://localhost:7000/eureka/
turbine:
  app-config: eureka-consumer-hystrix
  cluster-name-expression: new String("default")
  combine-host-port: true

```

**参数说明**

- `turbine.app-config`参数指定了需要收集监控信息的服务名；
- `turbine.cluster-name-expression` 参数指定了集群名称为 `default`，当我们服务数量非常多的时候，可以启动多个 Turbine 服务来构建不同的聚合集群，而该参数可以用来区分这些不同的聚合集群，同时该参数值可以在 Hystrix 仪表盘中用来定位不同的聚合集群，只需要在 Hystrix Stream 的 URL 中通过 cluster 参数来指定；
- `turbine.combine-host-port`参数设置为`true`，可以让同一主机上的服务通过主机名与端口号的组合来进行区分，默认情况下会以 host 来区分不同的服务，这会使得在本地调试的时候，本机上的不同服务聚合成一个服务来统计。

注意：`new String("default")`这个一定要用 String 来包一下，否则启动的时候会抛出异常：

```
org.springframework.expression.spel.SpelEvaluationException: EL1008E: Property or field 'default' cannot be found on object of type 'com.netflix.appinfo.InstanceInfo' - maybe not public or not valid?

```

### 测试

在完成了上面的内容构建之后，我们来体验一下 Turbine 对集群的监控能力。分别启动

- eureka-server
- eureka-producer
- eureka-consumer-hystrix
- turbine
- hystrix-dashboard

访问 [Hystrix Dashboard](http://localhost:11000/hystrix) 并开启对 [http://localhost:8080/turbine.stream](http://localhost:8080/turbine.stream) 的监控，这时候，我们将看到针对服务 `eureka-consumer-hystrix` 的聚合监控数据。

此时的架构如下图所示：
[![img](https://ws4.sinaimg.cn/large/006tNc79ly1fqekjxtja6j31140erabk.jpg)](https://ws4.sinaimg.cn/large/006tNc79ly1fqekjxtja6j31140erabk.jpg)

## 通过消息代理收集聚合

Spring Cloud 在封装 Turbine 的时候，还实现了基于消息代理的收集实现。所以，我们可以将所有需要收集的监控信息都输出到消息代理中，然后 Turbine 服务再从消息代理中异步的获取这些监控信息，最后将这些监控信息聚合并输出到 Hystrix Dashboard 中。通过引入消息代理，我们的 Turbine 和 Hystrix Dashoard 实现的监控架构可以改成如下图所示的结构：

[![img](https://ws4.sinaimg.cn/large/006tNc79ly1fqeklbqeg5j31310ewdhe.jpg)](https://ws4.sinaimg.cn/large/006tNc79ly1fqeklbqeg5j31310ewdhe.jpg)

从图中我们可以看到，这里多了一个重要元素：RabbitMQ。我们可以来构建一个新的应用来实现基于消息代理的 Turbine 聚合服务。

### Turbine Stream

需要注意一点的是，Turbine Stream 默认的端口已经从 8989 改为 8080 了。

创建一个标准的 Spring Boot 工程，命名为：turbine-stream-rabbitmq

#### POM

```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-turbine-stream</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-stream-rabbit</artifactId>
</dependency>

```

#### 配置文件

```
spring:
  application:
    name: turbine-stream-rabbitmq
eureka:
  client:
    service-url:
      defaultZone: http://localhost:7000/eureka/

```

#### 启动类

```
@SpringBootApplication
@EnableTurbineStream
public class TurbineStreamRabbitmqApplication {

    public static void main(String[] args) {
        SpringApplication.run(TurbineStreamRabbitmqApplication.class, args);
    }

    @Bean
    public ConfigurableCompositeMessageConverter integrationArgumentResolverMessageConverter(CompositeMessageConverterFactory factory) {
        return new ConfigurableCompositeMessageConverter(factory.getMessageConverterForAllRegistered().getConverters());
    }

}

```

### 改造服务调用者

以之前的 eureka-consumer-hystrix 项目为基础，在 pom.xml 里加入以下依赖

```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-netflix-hystrix-stream</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-stream-rabbit</artifactId>
</dependency>

```

再在启动类上加上`@EnableHystrix`注解

```
@EnableHystrix
@EnableFeignClients
@SpringBootApplication
public class EurekaConsumerHystrixApplication {

    public static void main(String[] args) {
        SpringApplication.run(EurekaConsumerHystrixApplication.class, args);
    }

}

```

### 测试

分别启动 eureka-consumer-hystrix、turbine-stream-rabbitmq 这两个项目，然后在 RabbitMQ 的管理后台可以看到，自动创建了一个 Exchange 和 Queue

[![img](https://ws2.sinaimg.cn/large/006tNc79ly1fqes6fk76xj31kw0v97bj.jpg)](https://ws2.sinaimg.cn/large/006tNc79ly1fqes6fk76xj31kw0v97bj.jpg)
![img](https://ws3.sinaimg.cn/large/006tNc79ly1fqes8rrtu3j31kw0v9tfh.jpg)



