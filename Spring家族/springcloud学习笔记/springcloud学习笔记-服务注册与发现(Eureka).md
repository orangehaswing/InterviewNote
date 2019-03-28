# springcloud学习笔记-服务注册与发现(Eureka、Consul)

# 注册中心

在 pom.xml 中引入以下依赖

```
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.0.1.RELEASE</version>
    <relativePath/> <!-- lookup parent from repository -->
</parent>

<properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
    <java.version>1.8</java.version>
    <spring-cloud.version>Finchley.RC1</spring-cloud.version>
</properties>

<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
    </dependency>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>

<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>${spring-cloud.version}</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>

<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugins>
</build>

```

通过`@EnableEurekaServer`注解启动一个服务注册中心提供给其他应用进行对话。这一步非常的简单，只需要在一个普通的 Spring Boot 应用中添加这个注解就能开启此功能，比如

```
@EnableEurekaServer
@SpringBootApplication
public class EurekaServerApplication {

    public static void main(String[] args) {
        SpringApplication.run(EurekaServerApplication.class, args);
    }
}

```

在默认设置下，该服务注册中心也会将自己作为客户端来尝试注册它自己，所以我们需要禁用它的客户端注册行为，只需要在 application.yml 配置文件中增加如下信息：

```
spring:
  application:
    name: eureka-server
server:
  port: 7000
eureka:
  instance:
    hostname: localhost
  client:
    register-with-eureka: false
    fetch-registry: false
    service-url:
      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/

```

- `server.port`：为了与后续要进行注册的服务区分，这里将服务注册中心的端口设置为 7000。
- `eureka.client.register-with-eureka`：表示是否将自己注册到 Eureka Server，默认为 true。
- `eureka.client.fetch-registry`：表示是否从 Eureka Server 获取注册信息，默认为 true。
- `eureka.client.service-url.defaultZone`：设置与 Eureka Server 交互的地址，查询服务和注册服务都需要依赖这个地址。默认是 [http://localhost:8761/eureka](http://localhost:8761/eureka) ；多个地址可使用英文逗号（,）分隔。

启动工程后，访问 [http://localhost:7000/](http://localhost:7000/)，可以看到下面的页面，其中还没有发现任何服务
[![img](https://ws1.sinaimg.cn/large/006tNc79ly1fqdhb1ql27j31kw0ri7d7.jpg)](https://ws1.sinaimg.cn/large/006tNc79ly1fqdhb1ql27j31kw0ri7d7.jpg)

# 集群

注册中心这么关键的服务，如果是单点话，遇到故障就是毁灭性的。在一个分布式系统中，服务注册中心是最重要的基础部分，理应随时处于可以提供服务的状态。为了维持其可用性，使用集群是很好的解决方案。Eureka 通过互相注册的方式来实现高可用的部署，所以我们只需要将 Eureke Server 配置其他可用的 service-url 就能实现高可用部署。

## 双节点注册中心

首先我们尝试一下双节点的注册中心的搭建。

1、我们将之前的 application.yml 复制一份并命名为 application-peer1.yml，作为 peer1 服务中心的配置，并将 service-url 指向 peer2

```
spring:
  application:
    name: eureka-server
server:
  port: 7001
eureka:
  instance:
    hostname: peer1
  client:
    register-with-eureka: true
    fetch-registry: true
    service-url:
      defaultZone: http://peer2:7002/eureka/

```

2、将之前的 application-peer1.yml 复制一份并命名为 application-peer2.yml，作为 peer2 服务中心的配置，并将 service-url 指向 peer1

```
spring:
  application:
    name: eureka-server
server:
  port: 7002
eureka:
  instance:
    hostname: peer2
  client:
    register-with-eureka: true
    fetch-registry: true
    service-url:
      defaultZone: http://peer1:7001/eureka/

```

3、配本地 host：
在 hosts 文件中加入如下配置

```
127.0.0.1 peer1 peer2

```

4、打包启动
依次执行下面命令

```
# 打包
mvn clean package  -Dmaven.test.skip=true

# 分别以 peer1 和 peer2 配置信息启动 Eureka
java -jar target/eureka-server-0.0.1-SNAPSHOT.jar  --spring.profiles.active=peer1
java -jar target/eureka-server-0.0.1-SNAPSHOT.jar  --spring.profiles.active=peer2

```

> 在刚启动 peer1 的时候，启动完成后会在控制台看到一些异常信息，大致就是拒绝连接、请求超时这一类的，这个不用管，启动 peer2 后就好了。

依次启动完成后，访问 [http://localhost:7001/](http://localhost:7001/)，效果如下
[![img](https://ws3.sinaimg.cn/large/006tKfTcly1fr2l67mm1bj30sg0iktbu.jpg)](https://ws3.sinaimg.cn/large/006tKfTcly1fr2l67mm1bj30sg0iktbu.jpg)

根据图可以看出 peer1 的注册中心 DS Replicas 已经有了 peer2 的相关配置信息，并且出现在 available-replicas 中。我们手动停止 peer2 来观察，发现 peer2 就会移动到 unavailable-replicas 一栏中，表示 peer2 不可用。

[![img](https://ws4.sinaimg.cn/large/006tKfTcly1fr2kyw47ndj30sg0i1q5q.jpg)](https://ws4.sinaimg.cn/large/006tKfTcly1fr2kyw47ndj30sg0i1q5q.jpg)

到此双节点的配置已经完成。

**注意事项**

- 在搭建 Eureka Server 双节点或集群的时候，要把`eureka.client.register-with-eureka`和`eureka.client.fetch-registry`均改为`true`（默认）。否则会出现实例列表为空，且 peer2 不在 available-replicas 而在 unavailable-replicas 的情况（这时其实只是启动了两个单点实例）。如果是像我这样图省事把之前的单节点配置和双节点的配置放在一个工程里，双节点的配置里要显示设置以上两个参数，直接删除是用不了默认配置的——Spring profile 会继承未在子配置里设置的父配置（application.yml）中的配置。
- 在注册的时候，配置文件中的`spring.application.name`必须一致，否则情况会是这样的
  [![img](https://ws3.sinaimg.cn/large/006tKfTcly1fr2l2jh5hmj30sg0izq65.jpg)](https://ws3.sinaimg.cn/large/006tKfTcly1fr2l2jh5hmj30sg0izq65.jpg)

## Eureka 集群使用

在生产中我们可能需要三台或者大于三台的注册中心来保证服务的稳定性，配置的原理其实都一样，将注册中心分别指向其它的注册中心。这里只介绍三台集群的配置情况，其实和双节点的注册中心类似，每台注册中心分别又指向其它两个节点即可。

application-peer1.yml

```
spring:
  application:
    name: eureka-server
server:
  port: 7001
eureka:
  instance:
    hostname: peer1
  client:
    service-url:
      defaultZone: http://peer2:7002/eureka/,http://peer3:7003/eureka/

```

application-peer2.yml

```
spring:
  application:
    name: eureka-server
server:
  port: 7002
eureka:
  instance:
    hostname: peer2
  client:
    service-url:
      defaultZone: http://peer1:7001/eureka/,http://peer3:7003/eureka/

```

application-peer3.yml

```
spring:
  application:
    name: eureka-server
server:
  port: 7003
eureka:
  instance:
    hostname: peer3
  client:
    service-url:
      defaultZone: http://peer1:7001/eureka/,http://peer2:7002/eureka/

```

修改 hosts 文件中的配置，添加 peer3

```
127.0.0.1 peer1 peer2 peer3

```

分别以 peer1、peer2、peer3 的配置参数启动 Eureka 注册中心

```
java -jar target/eureka-server-0.0.1-SNAPSHOT.jar  --spring.profiles.active=peer1
java -jar target/eureka-server-0.0.1-SNAPSHOT.jar  --spring.profiles.active=peer2
java -jar target/eureka-server-0.0.1-SNAPSHOT.jar  --spring.profiles.active=peer3

```

依次启动完成后，访问 [http://localhost:7001/](http://localhost:7001/)，效果如下
[![img](https://ws2.sinaimg.cn/large/006tKfTcly1fr2labbofmj30sg0kqtcf.jpg)](https://ws2.sinaimg.cn/large/006tKfTcly1fr2labbofmj30sg0kqtcf.jpg)

可以在 peer1 中看到了 peer2、peer3 的相关信息，至此 Eureka 集群也已经完成了。

# 源码分析

首先，服务注册中心、服务提供者、服务消费者这三个主要元素来说，后两者（也就是Eureka客户端）在整个运行机制中是大部分通信行为的主动发起者，而注册中心主要是处理请求的接收者。

所以，我们可以从Eureka的客户端作为入口看看它是如何完成这些主动通信行为的。

将一个普通的Spring Boot应用注册到Eureka Server中，或是从Eureka Server中获取服务列表时，主要就做了两件事：

- 在应用主类中配置了`@EnableDiscoveryClient`注解
- 在`application.properties`中用`eureka.client.serviceUrl.defaultZone`参数指定了服务注册中心的位置

顺着上面的线索，我们先查看`@EnableDiscoveryClient`的源码如下：

```
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@Import(EnableDiscoveryClientImportSelector.class)
public @interface EnableDiscoveryClient {

}
```

从该注解的注释我们可以知道：该注解用来开启`DiscoveryClient`的实例。通过梳理可以得到如下图的关系：

![img](http://blog.didispace.com/assets/eureka-code-1.png)

DiscoveryClient是Spring Cloud的接口，它定义了用来发现服务的常用抽象方法，而EurekaDiscoveryClient是对该接口的实现，它实现的是对Eureka发现服务的封装。

所以EurekaDiscoveryClient依赖了Eureka的EurekaClient接口，EurekaClient继承了LookupService接口，他们都是Netflix开源包中的内容，它主要定义了针对Eureka的发现服务的抽象方法，而真正实现发现服务的则是Netflix包中的DiscoveryClient类。

对Eureka Server的URL列表配置，在serviceUrl我们找到该属性相关的加载属性

### Region、Zone

客户端依次加载了两个内容，第一个是Region，第二个是Zone，从其加载逻上我们可以判断他们之间的关系：

- 通过`getRegion`函数，我们可以看到它从配置中读取了一个Region返回，所以一个微服务应用只可以属于一个Region，如果不特别配置，就默认为default。若我们要自己设置，可以通过`eureka.client.region`属性来定义。
- 通过`getAvailabilityZones`函数，我们可以知道当我们没有特别为Region配置Zone的时候，将默认采用defaultZone，这也是我们之前配置参数`eureka.client.serviceUrl.defaultZone`的由来。若要为应用指定Zone，我们可以通过`eureka.client.availability-zones`属性来进行设置。

### ServiceUrls

在获取了Region和Zone信息之后，才开始真正加载Eureka Server的具体地址。它根据传入的参数按一定算法确定加载位于哪一个Zone配置的serviceUrls。对于Zone和Region遵循这样的规则：优先访问同自己一个Zone中的实例，其次才访问其他Zone中的实例。

### 服务注册

DiscoveryClient 类实现“服务注册”行为，通过查看它的构造类，可以找到它调用了下面这个函数：

discoveryClient.register()

```
boolean register() throws Throwable {
    logger.info(PREFIX + appPathIdentifier + ": registering service...");
    EurekaHttpResponse<Void> httpResponse;
    try {
        httpResponse = eurekaTransport.registrationClient.register(instanceInfo);
    } catch (Exception e) {
        logger.warn("{} - registration failed {}", PREFIX + appPathIdentifier, e.getMessage(), e);
        throw e;
    }
    if (logger.isInfoEnabled()) {
        logger.info("{} - registration status: {}", PREFIX + appPathIdentifier, httpResponse.getStatusCode());
    }
    return httpResponse.getStatusCode() == 204;
}
```

注册操作也是通过REST请求的方式进行的。同时，这里我们也能看到发起注册请求的时候，传入了一个`InstanceInfo`对象，该对象就是注册时候客户端给服务端的服务的元数据。

### 服务获取与服务续约

`DiscoveryClient`的`initScheduledTasks`函数，其中还有两个定时任务，分别是“服务获取”和“服务续约”：

服务注册到Eureka Server后，自然需要一个心跳去续约，防止被剔除，所以他们肯定是成对出现的。对于服务续约相关的时间控制参数：

```
eureka.instance.lease-renewal-interval-in-seconds=30
eureka.instance.lease-expiration-duration-in-seconds=90
```

其中“服务续约”的实现较为简单，直接以REST请求的方式进行续约

而“服务获取”会根据是否第一次获取发起不同的REST请求和相应的处理

### 服务注册中心处理

所有的交互都是通过REST的请求来发起的。Eureka Server对于各类REST请求的定义都位于：com.netflix.eureka.resources包下。

- 对注册信息进行了校验
- 调用InstanceRegistry对象中的register(InstanceInfo info, int leaseDuration, boolean isReplication)函数来进行服务注册

















