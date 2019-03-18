# springcloud学习笔记-消息总线

# Spring Cloud Bus

Spring Cloud Bus 通过轻量消息代理连接各个分布的节点。这会用在广播状态的变化（例如配置变化）或者其他的消息指令。Spring Bus 的一个核心思想是通过分布式的启动器对 Spring Boot 应用进行扩展，也可以用来建立一个多个应用之间的通信频道。目前唯一实现的方式是用 Amqp 消息代理作为通道，同样特性的设置（有些取决于通道的设置）在更多通道的文档中。

Spring Cloud Bus 被国内很多都翻译为消息总线，也挺形象的。大家可以将它理解为管理和传播所有分布式项目中的消息既可，其实本质是利用了 MQ 的广播机制在分布式的系统中传播消息，目前常用的有 Kafka 和 RabbitMQ。利用 Bus 的机制可以做很多的事情，其中配置中心客户端刷新就是典型的应用场景之一，我们用一张图来描述 Bus 在配置中心使用的机制。

[![img](https://ws4.sinaimg.cn/large/006tKfTcly1fqiw37qgv3j31220a2dg4.jpg)](https://ws4.sinaimg.cn/large/006tKfTcly1fqiw37qgv3j31220a2dg4.jpg)

# 实战

我们选择上一篇文章 [Spring Cloud（八）：配置中心（服务化与高可用）](https://windmt.com/2018/04/19/spring-cloud-8-config-with-eureka/) 版本的示例代码来改造, MQ 我们使用 RabbitMQ 来做示例。

因为用的是 Spring Boot 2.0.1 + Spring Cloud Finchley.RC1，更新较多，坑也比较多，这里就把代码再贴一遍了。

示例代码：[GitHub](https://github.com/zhaoyibo/spring-cloud-study/tree/master/config-eureka-bus)

## 服务端

### POM 配置

在 pom.xml 里添加，这 4 个是必须的

```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-config-server</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-bus</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-stream-binder-rabbit</artifactId>
</dependency>

```

### 配置文件

application.yml 内容如下

```
spring:
  application:
    name: config-server
  cloud:
    config:
      server:
        git:
          uri: https://github.com/zhaoyibo/spring-cloud-study
          search-paths: config-repo
    bus:
      enabled: true
      trace:
        enabled: true
server:
  port: 12000
eureka:
  client:
    service-url:
      defaultZone: http://localhost:7000/eureka/
management:
  endpoints:
    web:
      exposure:
        include: bus-refresh

```

### 启动类

加`@EnableConfigServer`注解

```
@SpringBootApplication
@EnableConfigServer
public class ConfigServerApplication {

    public static void main(String[] args) {
        SpringApplication.run(ConfigServerApplication.class, args);
    }
}

```

## 客户端

### POM 配置

在 pom.xml 里添加以下依赖，前 5 个是必须的，最后一个 webflux 你可以用 web 来代替

```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-config</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-bus</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-stream-binder-rabbit</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-webflux</artifactId>
</dependency>

```

这里容易漏掉`spring-boot-starter-actuator`，如果缺了这个，当对服务端执行`/actuator/bus-refresh`的时候，客户端接收不到信息，Console 里只会显示以下信息（我就是在这儿被困了好久……）

```
2018-04-19 18:50:05.711  INFO 39762 --- [vOu-GI8c8mJtQ-1] o.s.a.r.c.CachingConnectionFactory       : Attempting to connect to: [localhost:5672]
2018-04-19 18:50:05.739  INFO 39762 --- [vOu-GI8c8mJtQ-1] o.s.a.r.c.CachingConnectionFactory       : Created new connection: rabbitConnectionFactory.publisher#2bc15b3b:0/SimpleConnection@14eb0d5e [delegate=amqp://guest@127.0.0.1:5672/, localPort= 60107]
2018-04-19 18:50:05.749  INFO 39762 --- [vOu-GI8c8mJtQ-1] o.s.amqp.rabbit.core.RabbitAdmin         : Auto-declaring a non-durable, auto-delete, or exclusive Queue (springCloudBus.anonymous.bOoVqQuSQvOu-GI8c8mJtQ) durable:false, auto-delete:true, exclusive:true. It will be redeclared if the broker stops and is restarted while the connection factory is alive, but all messages will be lost.

```

### 配置文件

还是分为两个，分别如下
application.yml

```
spring:
  application:
    name: config-client
  cloud:
    bus:
      trace:
        enabled: true
      enabled: true
server:
  port: 13000

```

bootstrap.yml

```
spring:
  cloud:
    config:
      name: config-client
      profile: dev
      label: master
      discovery:
        enabled: true
        service-id: config-server
eureka:
  client:
    service-url:
      defaultZone: http://localhost:7000/eureka/

```

### Controller

```
@RestController
@RefreshScope
public class HelloController {

    @Value("${info.profile:error}")
    private String profile;

    @GetMapping("/info")
    public Mono<String> hello() {
        return Mono.justOrEmpty(profile);
    }

}

```

`@RefreshScope`必须加，否则客户端会受到服务端的更新消息，但是更新不了，因为不知道更新哪里的。

至于启动主类，用默认生成的不用改，就不贴了。

### 测试

分别启动 eureka、config-server 和两个 config-client

```
# 打包
./mvnw clean package -Dmaven.test.skip=true

# 运行两个 client
java -jar target/spring-cloud-config-client-0.0.1-SNAPSHOT.jar --server.port=13000
java -jar target/spring-cloud-config-client-0.0.1-SNAPSHOT.jar --server.port=13001

```

启动后，RabbitMQ 中会自动创建一个 topic 类型的 Exchange 和两个以`springCloudBus.anonymous.`开头的匿名 Queue
[![img](https://ws4.sinaimg.cn/large/006tNc79ly1fqi6dawle6j31kw0v9qal.jpg)](https://ws4.sinaimg.cn/large/006tNc79ly1fqi6dawle6j31kw0v9qal.jpg)

我们访问 [http://localhost:13000/info](http://localhost:13000/info) 和 [http://localhost:13001/info](http://localhost:13001/info) 返回内容的都是`dev`。
将 Git 中的配置信息由`dev`改为`dev bus`，并执行

```
curl -X POST http://localhost:12000/actuator/bus-refresh/

```

再次访问 [http://localhost:13000/info](http://localhost:13000/info) 和 [http://localhost:13001/info](http://localhost:13001/info) 这时返回内容就是修改之后的`dev bus`了，说明成功了。

## 思考

这里我们再想一个问题：如果对客户端使用 `/actuator/bus-refresh`会发生什么呢？是只刷新了当前客户端还是刷新全部客户端，还是一个都没刷新呢？

不如来试一下吧：

首先我们需要把客户端上的`bus-refresh`端点给放出来

```
management:
  endpoints:
    web:
      exposure:
        include: bus-refresh

```

重启客户端，这时候访问 [http://localhost:13000/info](http://localhost:13000/info) 和 [http://localhost:13001/info](http://localhost:13001/info) 返回的都是`dev bus`。

去 Git 里把配置信息由`dev bus`改为`dev`，然后执行

```
curl -X POST http://localhost:13000/actuator/bus-refresh/

```

再次访问 [http://localhost:13000/info](http://localhost:13000/info) 和 [http://localhost:13001/info](http://localhost:13001/info) 发现这时返回的都是`dev`了，说明只要开启 Spring Cloud Bus 后，不管是对 config-server 还是 config-client 执行`/actuator/bus-refresh`都是可以更新配置的。

# 其它

**因为版本问题，以下两部分还未在该版本进行验证。待有时间了试一下。**

## 局部刷新

某些场景下（例如灰度发布），我们可能只想刷新部分微服务的配置，此时可通过`/actuator/bus-refresh/{destination}`端点的 destination 参数来定位要刷新的应用程序。

例如：`/actuator/bus-refresh/customers:8000`，这样消息总线上的微服务实例就会根据 destination 参数的值来判断是否需要要刷新。其中，`customers:8000`指的是各个微服务的 ApplicationContext ID。

destination 参数也可以用来定位特定的微服务。例如：`/actuator/bus-refresh/customers:**`，这样就可以触发 customers 微服务所有实例的配置刷新。

## 跟踪总线事件

一些场景下，我们可能希望知道 Spring Cloud Bus 事件传播的细节。此时，我们可以跟踪总线事件（RemoteApplicationEvent 的子类都是总线事件）。

跟踪总线事件非常简单，只需设置`spring.cloud.bus.trace.enabled=true`，这样在`/actuator/bus-refresh`端点被请求后，访问 ~~`/trace`~~ （现在好像是`/actuator/httptrace`）端点就可获得 ~~类似如下的结果~~ ：

```
{
  "timestamp": 1495851419032,
  "info": {
    "signal": "spring.cloud.bus.ack",
    "type": "RefreshRemoteApplicationEvent",
    "id": "c4d374b7-58ea-4928-a312-31984def293b",
    "origin": "stores:8002",
    "destination": "*:**"
  }
  },
  {
  "timestamp": 1495851419033,
  "info": {
    "signal": "spring.cloud.bus.sent",
    "type": "RefreshRemoteApplicationEvent",
    "id": "c4d374b7-58ea-4928-a312-31984def293b",
    "origin": "spring-cloud-config-client:8001",
    "destination": "*:**"
  }
  },
  {
  "timestamp": 1495851422175,
  "info": {
    "signal": "spring.cloud.bus.ack",
    "type": "RefreshRemoteApplicationEvent",
    "id": "c4d374b7-58ea-4928-a312-31984def293b",
    "origin": "customers:8001",
    "destination": "*:**"
  }
}

```

这个日志显示了`customers:8001`发出了 RefreshRemoteApplicationEvent 事件，广播给所有的服务，被`customers:9000`和`stores:8081`接受到了。想要对接受到的消息自定义自己的处理方式的话，可以添加`@EventListener`注解的 AckRemoteApplicationEvent 和 SentApplicationEvent 类型到你自己的应用中。或者到 TraceRepository 类中，直接处理数据。

这样，我们就可清晰地知道事件的传播细节。