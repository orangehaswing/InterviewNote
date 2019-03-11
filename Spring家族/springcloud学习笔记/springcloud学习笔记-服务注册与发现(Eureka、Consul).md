# springcloud学习笔记-服务注册与发现(Eureka、Consul)

## Spring Cloud简介

Spring Cloud是一个基于Spring Boot实现的云应用开发工具，它为基于JVM的云应用开发中涉及的配置管理、服务发现、断路器、智能路由、微代理、控制总线、全局锁、决策竞选、分布式会话和集群状态管理等操作提供了一种简单的开发方式。

Spring Cloud包含了多个子项目（针对分布式系统中涉及的多个不同开源产品），比如：Spring Cloud Config、Spring Cloud Netflix、Spring Cloud0 CloudFoundry、Spring Cloud AWS、Spring Cloud Security、Spring Cloud Commons、Spring Cloud Zookeeper、Spring Cloud CLI等项目。

## 微服务架构

简单的说，微服务架构就是将一个完整的应用从数据存储开始垂直拆分成多个不同的服务，每个服务都能独立部署、独立维护、独立扩展，服务与服务间通过诸如RESTful API的方式互相调用。

[Reference](https://mp.weixin.qq.com/s/fzk-kENu0I22P3F2Vu7KBA?)

## 服务治理

在简单介绍了Spring Cloud和微服务架构之后，下面回归本文的主旨内容，如何使用Spring Cloud来实现服务治理。

由于Spring Cloud为服务治理做了一层抽象接口，所以在Spring Cloud应用中可以支持多种不同的服务治理框架，比如：Netflix Eureka、Consul、Zookeeper。在Spring Cloud服务治理抽象层的作用下，我们可以无缝地切换服务治理实现，并且不影响任何其他的服务注册、服务发现、服务调用等逻辑。

所以，下面我们通过介绍两种服务治理的实现来体会Spring Cloud这一层抽象所带来的好处。

### Spring Cloud Eureka

首先，我们来尝试使用Spring Cloud Eureka来实现服务治理。

Spring Cloud Eureka是Spring Cloud Netflix项目下的服务治理模块。而Spring Cloud Netflix项目是Spring Cloud的子项目之一，主要内容是对Netflix公司一系列开源产品的包装，它为Spring Boot应用提供了自配置的Netflix OSS整合。通过一些简单的注解，开发者就可以快速的在应用中配置一下常用模块并构建庞大的分布式系统。它主要提供的模块包括：服务发现（Eureka），断路器（Hystrix），智能路由（Zuul），客户端负载均衡（Ribbon）等。

下面，就来具体看看如何使用Spring Cloud Eureka实现服务治理。

#### 创建“服务注册中心”

创建一个基础的Spring Boot工程，命名为`eureka-server`，并在`pom.xml`中引入需要的依赖内容：

```
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>1.5.4.RELEASE</version>
    <relativePath/>
</parent>

<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-eureka-server</artifactId>
    </dependency>
</dependencies>

<dependencyManagement>
    <dependencies>
        <dependency>
           <groupId>org.springframework.cloud</groupId>
           <artifactId>spring-cloud-dependencies</artifactId>
           <version>Dalston.SR1</version>
           <type>pom</type>
           <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>

```

通过`@EnableEurekaServer`注解启动一个服务注册中心提供给其他应用进行对话。这一步非常的简单，只需要在一个普通的Spring Boot应用中添加这个注解就能开启此功能，比如下面的例子：

```
@EnableEurekaServer
@SpringBootApplication
public class Application {

    public static void main(String[] args) {
        new SpringApplicationBuilder(Application.class)
                    .web(true).run(args);
    }
}

```

在默认设置下，该服务注册中心也会将自己作为客户端来尝试注册它自己，所以我们需要禁用它的客户端注册行为，只需要在`application.properties`配置文件中增加如下信息：

```
spring.application.name=eureka-server
server.port=1001

eureka.instance.hostname=localhost
eureka.client.register-with-eureka=false
eureka.client.fetch-registry=false

```

为了与后续要进行注册的服务区分，这里将服务注册中心的端口通过server.port属性设置为1001。启动工程后，访问：[http://localhost:1001/](http://localhost:1001/)，可以看到下面的页面，其中还没有发现任何服务。

[![img](http://blog.didispace.com/content/images/posts/spring-cloud-starter-dalston-1-1.png)](http://blog.didispace.com/content/images/posts/spring-cloud-starter-dalston-1-1.png)

#### 创建“服务提供方”

下面我们创建提供服务的客户端，并向服务注册中心注册自己。本文我们主要介绍服务的注册与发现，所以我们不妨在服务提供方中尝试着提供一个接口来获取当前所有的服务信息。

首先，创建一个基本的Spring Boot应用。命名为`eureka-client`，在`pom.xml`中，加入如下配置：

```
<parent> 
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>1.5.4.RELEASE</version>
    <relativePath/> <!-- lookup parent from repository -->
</parent>

<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-eureka</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>

<dependencyManagement>
    <dependencies>
        <dependency>
           <groupId>org.springframework.cloud</groupId>
           <artifactId>spring-cloud-dependencies</artifactId>
           <version>Dalston.SR1</version>
           <type>pom</type>
           <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>

```

其次，实现/dc请求处理接口，通过DiscoveryClient对象，在日志中打印出服务实例的相关内容。

```
@RestController
public class DcController {

    @Autowired
    DiscoveryClient discoveryClient;

    @GetMapping("/dc")
    public String dc() {
        String services = "Services: " + discoveryClient.getServices();
        System.out.println(services);
        return services;
    }

}

```

最后在应用主类中通过加上`@EnableDiscoveryClient`注解，该注解能激活Eureka中的DiscoveryClient实现，这样才能实现Controller中对服务信息的输出。

```
@EnableDiscoveryClient
@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        new SpringApplicationBuilder(
            ComputeServiceApplication.class)
            .web(true).run(args);
    }
}

```

我们在完成了服务内容的实现之后，再继续对`application.properties`做一些配置工作，具体如下：

```
spring.application.name=eureka-client
server.port=2001
eureka.client.serviceUrl.defaultZone=http://localhost:1001/eureka/

```

通过`spring.application.name`属性，我们可以指定微服务的名称后续在调用的时候只需要使用该名称就可以进行服务的访问。`eureka.client.serviceUrl.defaultZone`属性对应服务注册中心的配置内容，指定服务注册中心的位置。为了在本机上测试区分服务提供方和服务注册中心，使用`server.port`属性设置不同的端口。

启动该工程后，再次访问：http://localhost:1001/。可以如下图内容，我们定义的服务被成功注册了

[![img](http://blog.didispace.com/content/images/posts/spring-cloud-starter-dalston-1-2.png)](http://blog.didispace.com/content/images/posts/spring-cloud-starter-dalston-1-2.png)

当然，我们也可以通过直接访问`eureka-client`服务提供的`/dc`接口来获取当前的服务清单，只需要访问：http://localhost:2001/dc，我们可以得到如下输出返回：

```
Services: [eureka-client]

```

其中，方括号中的`eureka-client`就是通过Spring Cloud定义的`DiscoveryClient`接口在eureka的实现中获取到的所有服务清单。由于Spring Cloud在服务发现这一层做了非常好的抽象，所以，对于上面的程序，我们可以无缝的从eureka的服务治理体系切换到consul的服务治理体系中区。

### Spring Cloud Consul

Spring Cloud Consul项目是针对Consul的服务治理实现。Consul是一个分布式高可用的系统，它包含多个组件，但是作为一个整体，在微服务架构中为我们的基础设施提供服务发现和服务配置的工具。它包含了下面几个特性：

- 服务发现
- 健康检查
- Key/Value存储
- 多数据中心

由于Spring Cloud Consul项目的实现，我们可以轻松的将基于Spring Boot的微服务应用注册到Consul上，并通过此实现微服务架构中的服务治理。

以之前实现的基于Eureka的示例（eureka-client）为基础，我们如何将之前实现的服务提供者注册到Consul上呢？方法非常简单，我们只需要在`pom.xml`中将eureka的依赖修改为如下依赖：

```
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-consul-discovery</artifactId>
</dependency>

```

接下来再修改一下`application.properites`，将consul需要的配置信息加入即可，比如：（下面配置是默认值）

```
spring.cloud.consul.host=localhost
spring.cloud.consul.port=8500

```

到此为止，我们将eureka-client转换为基于consul服务治理的服务提供者就完成了。前文我们已经有提到过服务发现的接口`DiscoveryClient`是Spring Cloud对服务治理做的一层抽象，所以可以屏蔽Eureka和Consul服务治理的实现细节，我们的程序不需要做任何改变，只需要引入不同的服务治理依赖，并配置相关的配置属性就能轻松的将微服务纳入Spring Cloud的各个服务治理框架中。

下面可以尝试让consul的服务提供者运行起来。这里可能读者会问，不需要创建类似eureka-server的服务端吗？由于Consul自身提供了服务端，所以我们不需要像之前实现Eureka的时候创建服务注册中心，直接通过下载consul的服务端程序就可以使用。

我们可以用下面的命令启动consul的开发模式：

```
$consul agent -dev
==> Starting Consul agent...
==> Starting Consul agent RPC...
==> Consul agent running!
           Version: 'v0.7.2'
         Node name: 'Lenovo-zhaiyc'
        Datacenter: 'dc1'
            Server: true (bootstrap: false)
       Client Addr: 127.0.0.1 (HTTP: 8500, HTTPS: -1, DNS: 8600, RPC: 8400)
      Cluster Addr: 127.0.0.1 (LAN: 8301, WAN: 8302)
    Gossip encrypt: false, RPC-TLS: false, TLS-Incoming: false
             Atlas: <disabled>

==> Log data will now stream in as it occurs:

    2017/06/22 07:50:54 [INFO] raft: Initial configuration (index=1): [{Suffrage:Voter ID:127.0.0.1:8300 Address:127.0.0.1:8300}]
    2017/06/22 07:50:54 [INFO] raft: Node at 127.0.0.1:8300 [Follower] entering Follower state (Leader: "")
    2017/06/22 07:50:54 [INFO] serf: EventMemberJoin: Lenovo-zhaiyc 127.0.0.1
    2017/06/22 07:50:54 [INFO] consul: Adding LAN server Lenovo-zhaiyc (Addr: tcp/127.0.0.1:8300) (DC: dc1)
    2017/06/22 07:50:54 [INFO] serf: EventMemberJoin: Lenovo-zhaiyc.dc1 127.0.0.1
    2017/06/22 07:50:54 [INFO] consul: Adding WAN server Lenovo-zhaiyc.dc1 (Addr: tcp/127.0.0.1:8300) (DC: dc1)
    2017/06/22 07:51:01 [ERR] agent: failed to sync remote state: No cluster leader
    2017/06/22 07:51:02 [WARN] raft: Heartbeat timeout from "" reached, starting election
    2017/06/22 07:51:02 [INFO] raft: Node at 127.0.0.1:8300 [Candidate] entering Candidate state in term 2
    2017/06/22 07:51:02 [DEBUG] raft: Votes needed: 1
    2017/06/22 07:51:02 [DEBUG] raft: Vote granted from 127.0.0.1:8300 in term 2. Tally: 1
    2017/06/22 07:51:02 [INFO] raft: Election won. Tally: 1
    2017/06/22 07:51:02 [INFO] raft: Node at 127.0.0.1:8300 [Leader] entering Leader state
    2017/06/22 07:51:02 [INFO] consul: cluster leadership acquired
    2017/06/22 07:51:02 [INFO] consul: New leader elected: Lenovo-zhaiyc
    2017/06/22 07:51:02 [DEBUG] consul: reset tombstone GC to index 3
    2017/06/22 07:51:02 [INFO] consul: member 'Lenovo-zhaiyc' joined, marking health alive
    2017/06/22 07:51:02 [INFO] agent: Synced service 'consul'
    2017/06/22 07:51:02 [DEBUG] agent: Node info in sync
```

consul服务端启动完成之后，我们再将之前改造后的consul服务提供者启动起来。consul与eureka一样，都提供了简单的ui界面来查看服务的注册情况：

[![img](http://blog.didispace.com/content/images/posts/spring-cloud-starter-dalston-1-3.png)](http://blog.didispace.com/content/images/posts/spring-cloud-starter-dalston-1-3.png)