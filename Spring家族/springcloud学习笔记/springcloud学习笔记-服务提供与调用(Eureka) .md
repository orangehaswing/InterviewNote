# springcloud学习笔记-服务提供与调用(Eureka) 

案例中有三个角色：服务注册中心、服务提供者、服务消费者

流程如下：

1. 启动注册中心
2. 服务提供者生产服务并注册到服务中心中
3. 消费者从服务中心中获取服务并执行


[![img](https://ws2.sinaimg.cn/large/006tNc79ly1fqdhgdesulj30fa08hjrj.jpg)](https://ws2.sinaimg.cn/large/006tNc79ly1fqdhgdesulj30fa08hjrj.jpg)

# 服务提供者

我们假设服务提供者有一个 `hello()` 方法，可以根据传入的参数，提供输出 “hello xxx + 当前时间” 的服务。

## POM 包配置

创建一个基本的 Spring Boot 应用，命名为`eureka-producer`，在 pom.xml 中添加如下配置：

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>

```

## 配置文件

application.yml 配置如下

```
spring:
  application:
    name: eureka-producer
eureka:
  client:
    service-url:
      defaultZone: http://localhost:7000/eureka/
server:
  port: 8000

```

通过`spring.application.name`属性，我们可以指定微服务的名称后续在调用的时候只需要使用该名称就可以进行服务的访问。`eureka.client.serviceUrl.defaultZone`属性对应服务注册中心的配置内容，指定服务注册中心的位置。为了在本机上测试区分服务提供方和服务注册中心，使用`server.port`属性设置不同的端口。

## 启动类

保持默认生成的即可， Finchley.RC1 这个版本的 Spring Cloud 已经无需添加`@EnableDiscoveryClient`注解了。（那么如果我引入了相关的 jar 包又想禁用服务注册与发现怎么办？设置`eureka.client.enabled=false`）

```
@SpringBootApplication
public class EurekaProducerApplication {

    public static void main(String[] args) {
        SpringApplication.run(EurekaProducerApplication.class, args);
    }

}

```

## Controller

提供 hello 服务

```
@RestController
@RequestMapping("/hello")
public class HelloController {

    @GetMapping("/")
    public String hello(@RequestParam String name) {
        return "Hello, " + name + " " + new Date();
    }

}

```

启动工程后，就可以在注册中心 [Eureka](http://localhost:7000/) 的页面看到 EUERKA-PRODUCER 服务。
[![img](https://ws3.sinaimg.cn/large/006tKfTcly1fr2lk7o8mhj30sg044wfd.jpg)](https://ws3.sinaimg.cn/large/006tKfTcly1fr2lk7o8mhj30sg044wfd.jpg)

我们模拟一个请求试一下 Producer 能否正常工作
[http://localhost:8000/hello/?name=windmt](http://localhost:8000/hello/?name=windmt)

```
Hello, windmt Fri Apr 13 18:36:36 CST 2018

```

OK， 直接访问时没有问题的，到此服务提供者配置就完成了。

# 服务消费者

创建服务消费者根据使用 API 的不同，大致分为三种方式。虽然大家在实际使用中用的应该都是 Feign，但是这里还是把这三种都介绍一下吧，如果你只关心 Feign，可以直接跳到最后。

三种方式均使用同一配置文件，不再单独说明了

```
spring:
  application:
    name: eureka-consumer
eureka:
  client:
    service-url:
      defaultZone: http://localhost:7000/eureka/ # 指定 Eureka 注册中心的地址
server:
  port: 9000 # 分别为 9000、9001、9002

```

## 使用 LoadBalancerClient

从`LoadBalancerClient`接口的命名中，我们就知道这是一个负载均衡客户端的抽象定义，下面我们就看看如何使用 Spring Cloud 提供的负载均衡器客户端接口来实现服务的消费。

### POM 包配置

我们先来创建一个服务消费者工程，命名为：`eureka-consumer`。pom.xml 同 Producer 的，不再赘述。

### 启动类

初始化`RestTemplate`，用来发起 REST 请求。

```
@SpringBootApplication
public class EurekaConsumerApplication {

    @Bean
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }

    public static void main(String[] args) {
        SpringApplication.run(EurekaConsumerApplication.class, args);
    }
}

```

### Controller

创建一个接口用来消费 eureka-producer 提供的接口：

```
@RequestMapping("/hello")
@RestController
public class HelloController {

    @Autowired
    private LoadBalancerClient client;

    @Autowired
    private RestTemplate restTemplate;

    @GetMapping("/")
    public String hello(@RequestParam String name) {
        name += "!";
        ServiceInstance instance = client.choose("eureka-producer");
        String url = "http://" + instance.getHost() + ":" + instance.getPort() + "/hello/?name=" + name;
        return restTemplate.getForObject(url, String.class);
    }

}

```

可以看到这里，我们注入了`LoadBalancerClient`和`RestTemplate`，并在`hello`方法中，先通过`loadBalancerClient`的`choose`方法来负载均衡的选出一个`eureka-producer`的服务实例，这个服务实例的基本信息存储在`ServiceInstance`中，然后通过这些对象中的信息拼接出访问服务调用者的`/hello/`接口的详细地址，最后再利用`RestTemplate`对象实现对服务提供者接口的调用。

另外，为了在调用时能从返回结果上与服务提供者有个区分，在这里我简单处理了一下，`name+="!"`，即服务调用者的 response 中会比服务提供者的多一个感叹号（!）。

访问 [http://localhost:9000/hello/?name=windmt](http://localhost:9000/hello/?name=windmt) 以验证是否调用成功

```
Hello, windmt! Fri Apr 13 18:44:55 CST 2018

```

## Spring Cloud Ribbon

之前已经介绍过 Ribbon 了，它是一个基于 HTTP 和 TCP 的客户端负载均衡器。它可以通过在客户端中配置 ribbonServerList 来设置服务端列表去轮询访问以达到均衡负载的作用。

当 Ribbon 与 Eureka 联合使用时，ribbonServerList 会被 DiscoveryEnabledNIWSServerList 重写，扩展成从 Eureka 注册中心中获取服务实例列表。同时它也会用 NIWSDiscoveryPing 来取代 IPing，它将职责委托给 Eureka 来确定服务端是否已经启动。

### POM 包配置

将之前的 eureka-consumer 工程复制一份，并命名为 eureka-consumer-ribbon。

pom.xml 文件还用之前的就行。至于 spring-cloud-starter-ribbon，因为我使用的 Spring Cloud 版本是 Finchley.RC1，spring-cloud-starter-netflix-eureka-client 里边已经包含了 spring-cloud-starter-netflix-ribbon 了。

```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>

```

### 启动类

修改应用主类，为`RestTemplate`添加`@LoadBalanced`注解

```
@LoadBalanced
@Bean
public RestTemplate restTemplate() {
    return new RestTemplate();
}

```

### Controller

修改 controller，去掉`LoadBalancerClient`，并修改相应的方法，直接用 `RestTemplate`发起请求

```
@GetMapping("/")
public String hello(@RequestParam String name) {
    name += "!";
    String url = "http://eureka-producer/hello/?name=" + name;
    return restTemplate.getForObject(url, String.class);
}

```

可能你已经注意到了，这里直接用服务名`eureka-producer`取代了之前的具体的`host:port`。那么这样的请求为什么可以调用成功呢？因为 Spring Cloud Ribbon 有一个拦截器，它能够在这里进行实际调用的时候，自动的去选取服务实例，并将这里的服务名替换成实际要请求的 IP 地址和端口，从而完成服务接口的调用。

访问 [http://localhost:9001/hello/?name=windmt](http://localhost:9001/hello/?name=windmt) 以验证是否调用成功

```
Hello, windmt! Fri Apr 13 22:24:14 CST 2018

```

也可以通过启动多个 eureka-producer 服务来观察其负载均衡的效果。

## Spring Cloud Feign

在实际工作中，我们基本上都是使用 Feign 来完成调用的。我们通过一个例子来展现 Feign 如何方便的声明对 eureka-producer 服务的定义和调用。

### POM 包配置

创建一个基本的 Spring Boot 应用，命名为`eureka-producer-feign`，在 pom.xml 中添加如下配置：

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>

```

### 启动类

在启动类上加上`@EnableFeignClients`

```
@EnableFeignClients
@SpringBootApplication
public class EurekaConsumerFeignApplication {

    public static void main(String[] args) {
        SpringApplication.run(EurekaConsumerFeignApplication.class, args);
    }
}

```

### Feign 调用实现

创建一个 Feign 的客户端接口定义。使用`@FeignClient`注解来指定这个接口所要调用的服务名称，接口中定义的各个函数使用 Spring MVC 的注解就可以来绑定服务提供方的 REST 接口，比如下面就是绑定 eureka-producer 服务的`/hello/`接口的例子：

```
@FeignClient(name = "eureka-producer")
public interface HelloRemote {

    @GetMapping("/hello/")
    String hello(@RequestParam(value = "name") String name);

}

```

此类中的方法和远程服务中 Contoller 中的方法名和参数需保持一致。

> 这里有几个坑，后边有详细说明。

### Controller

修改 Controller，将 HelloRemote 注入到 controller 层，像普通方法一样去调用即可

```
@RequestMapping("/hello")
@RestController
public class HelloController {

    @Autowired
    HelloRemote helloRemote;

    @GetMapping("/{name}")
    public String index(@PathVariable("name") String name) {
        return helloRemote.hello(name + "!");
    }

}

```

通过 Spring Cloud Feign 来实现服务调用的方式非常简单，通过`@FeignClient`定义的接口来统一的声明我们需要依赖的微服务接口。而在具体使用的时候就跟调用本地方法一点的进行调用即可。由于 Feign 是基于 Ribbon 实现的，所以它自带了客户端负载均衡功能，也可以通过 Ribbon 的 IRule 进行策略扩展。另外，Feign 还整合的 Hystrix 来实现服务的容错保护，这个在后边会详细讲。（在 Finchley.RC1 版本中，Feign 的 Hystrix 默认是关闭的。

访问 [http://localhost:9002/hello/windmt](http://localhost:9002/hello/windmt) 以验证是否调用成功

```
Hello, windmt! Sat Apr 14 01:03:56 CST 2018
```

## 负载均衡

以上三种方式都能实现负载均衡，都是以轮询访问的方式实现的。这个以大家常用的 Feign 的方式做一个测试。

以上面 eureka-producer 为例子修改，将其中的 controller 改动如下：

```
@RestController
@RequestMapping("/hello")
public class HelloController {

    @Value("${config.producer.instance:0}")
    private int instance;

    @GetMapping("/")
    public String hello(@RequestParam String name) {
        return "[" + instance + "]" + "Hello, " + name + " " + new Date();
    }

}

```

打包启动

```
// 打包
mvn clean package  -Dmaven.test.skip=true

// 分别以 8000 和 8001 启动实例 1 和 实例 2
java -jar target/eureka-producer-0.0.1-SNAPSHOT.jar --config.producer.instance=1 --server.port=8000
java -jar target/eureka-producer-0.0.1-SNAPSHOT.jar --config.producer.instance=2 --server.port=8001

// 在端口 9002 上启动 eureka-consumer-feign
java -jar target/eureka-eureka-consumer-feign-0.0.1-SNAPSHOT.jar --server.port=9002

```

访问 [http://localhost:9002/hello/windmt](http://localhost:9002/hello/windmt) 进行测试。在不断的测试下去会发现两种结果交替出现

```
[1]Hello, , windmt Sun Apr 15 19:37:15 CST 2018
[2]Hello, , windmt Sun Apr 15 19:37:17 CST 2018
[1]Hello, , windmt Sun Apr 15 19:37:19 CST 2018
[2]Hello, , windmt Sun Apr 15 19:37:24 CST 2018
[1]Hello, , windmt Sun Apr 15 19:37:27 CST 2018

```

这说明两个服务中心自动提供了服务均衡负载的功能。如果我们将服务提供者的数量在提高为 N 个，测试结果一样，请求会自动轮询到每个服务端来处理。

# Server源码分析

## 服务注册

当Eureka客户端向Eureka Server注册时，它提供自身的元数据，比如IP地址、端口，运行状况指示符URL，主页

## 服务续约 

Eureka客户会每隔30秒发送一次心跳来续约。 Eureka Server在90秒没有收到Eureka客户的续约，它会将实例从其注册表中删除。 

## 服务下线

Eureka客户端在程序关闭时向Eureka服务器发送取消请求。 发送请求后，该客户端实例信息将从服务器的实例注册表中删除。

## 服务剔除

当Eureka客户端连续90秒没有向Eureka服务器发送服务续约，即心跳，Eureka服务器会将该服务实例从服务注册列表删除。

## 源码分析

@EnableDiscoveryClient注解启动DiscoveryClient实例。客户端依次加载了Region，Zone两个内容。从加载逻辑看，Region与Zone是一对多的关系。

当使用Ribbon来实现服务调用时，对于Zone的设置可以在负载均衡时实现区域亲和。默认策略会优先访问同一个客户端处于一个Zone中的服务端实例。

## 配置详解

Eureka客户端配置：

- 服务注册相关：注册中心的地址，服务获取的间隔时间，可用区域
- 服务实例相关，服务实例的名称，IP，端口号，健康检查路径


# Ribbon负载均衡

基于HTTP和TCP的客户端负载均衡工具，基于Netflix Ribbon实现。

- 软负载均衡：在一台机器上安装附加的某种软件，如nginx负载均衡
- 硬负载均衡：负载均衡器，F5负载均衡器

## 配置

- 服务提供者只需启动多个服务实例并注册到一个注册中心
- 服务消费者直接通过被@LoadBalanced注解修饰过的RestTemplate来实现接口调用

## 详解 RestTemplate

RestTemplate针对几种不同请求类型和参数类型的服务调用实现。

- GET：getForEntity函数，getForObject函数
- POST：postForEntity函数，postForObject函数，postForLocation函数。
- PUT
- DELETE

## @LoadBalanced注解

@LoadBalanced注解用来给RestTemplate标记，以使用负载均衡的客户端（LoadBalancerClient）来配置它。

客户端负载均衡器中应具备能力：

- **ServiceInstance ：**根据传入的服务名serviceId，从负载均衡器中挑选一个对应服务的实例
- **execute** ：使用从负载均衡器中挑选出的服务实例来执行请求内容。
- **reconstructURI ：** 为系统构建一个合适的host:port形式的URI

它是如何通过LoadBalancerInterceptor拦截器对RequestTemplate的请求进行拦截，并利用Spring Cloud的负载均衡器LoadBalancerClient将以服务名为host的URI转换成具体的服务实例地址的过程。

使用Ribbon实现负载均衡器的时候，实际使用的还是Ribbon中定义ILoadBalancer接口的实现，自动化配置会采用ZoneAwareLoadBalancer的实例来实现客户端负载均衡。

## 负载均衡器

六个组件：

- ServerList，负载均衡使用的服务器列表。这个列表会缓存在负载均衡器中，并定期更新。当 Ribbon 与 Eureka 结合使用时，ServerList 的实现类就是 DiscoveryEnabledNIWSServerList，它会保存 Eureka Server 中注册的服务实例表。
- ServerListFilter，服务器列表过滤器。这是一个接口，主要用于对 Service Consumer 获取到的服务器列表进行预过滤，过滤的结果也是 ServerList。Ribbon 提供了多种过滤器的实现。
- IPing，探测服务实例是否存活的策略。
- IRule，负载均衡策略，其实现类表述的策略包括：轮询、随机、根据响应时间加权等。
- ILoadBalancer，负载均衡器。这也是一个接口，Ribbon 为其提供了多个实现，比如 ZoneAwareLoadBalancer。而上层代码通过调用其 API 进行服务调用的负载均衡选择。一般 ILoadBalancer 的实现类中会引用一个 IRule。
- RestClient，服务调用器。顾名思义，这就是负载均衡后，Ribbon 向 Service Provider 发起 REST 请求的工具。

Ribbon 工作时会做四件事情：

1. 优先选择在同一个 Zone 且负载较少的 Eureka Server；
2. 定期从 Eureka 更新并过滤服务实例列表；
3. 根据用户指定的策略，在从 Server 取到的服务注册列表中选择一个实例的地址；
4. 通过 RestClient 进行服务调用


