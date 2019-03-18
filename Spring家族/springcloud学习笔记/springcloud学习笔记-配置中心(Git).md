# springcloud学习笔记-配置中心(Git)

Spring Cloud Config 是 Spring Cloud 团队创建的一个全新项目，用来为分布式系统中的基础设施和微服务应用提供集中化的外部配置支持，它分为服务端与客户端两个部分。其中服务端也称为分布式配置中心，它是一个独立的微服务应用，用来连接配置仓库并为客户端提供获取配置信息、加密 / 解密信息等访问接口；而客户端则是微服务架构中的各个微服务应用或基础设施，它们通过指定的配置中心来管理应用资源与业务相关的配置内容，并在启动的时候从配置中心获取和加载配置信息。Spring Cloud Config 实现了对服务端和客户端中环境变量和属性配置的抽象映射，所以它除了适用于 Spring 构建的应用程序之外，也可以在任何其他语言运行的应用程序中使用。由于 Spring Cloud Config 实现的配置中心默认采用 Git 来存储配置信息，所以使用 Spring Cloud Config 构建的配置服务器，天然就支持对微服务应用配置信息的版本管理，并且可以通过 Git 客户端工具来方便的管理和访问配置内容。当然它也提供了对其他存储方式的支持，比如：SVN 仓库、本地化文件系统。

在本文中，我们将学习如何构建一个基于 Git 存储的分布式配置中心，并对客户端进行改造，并让其能够从配置中心获取配置信息并绑定到代码中的整个过程。最后，我们还将了解如何能让客户端获取到修改后的最新配置。

# 准备工作

准备一个 Git 仓库，在 Github 上面创建了一个文件夹 config-repo 用来存放配置文件，为了模拟生产环境，我们创建以下三个配置文件：

```
// 开发环境
config-client-dev.yml
// 测试环境
config-client-test.yml
// 生产环境
config-client-prod.yml

```

每个配置文件中都写一个属性 neo.hello, 属性值分别是 dev/test/prod。下面我们开始配置 Server 端。

# Server 端

创建一个基础的 Spring Boot 工程，命名为：config-server-git

## 添加依赖

只需要在 pom.xml 中 加入 spring-cloud-config-server 即可

```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-config-server</artifactId>
</dependency>

```

## 配置文件

在 application.yml 中添加配置服务的基本信息以及 Git 仓库的相关信息

```
spring:
  application:
    name: config-server
  cloud:
    config:
      server:
        git:
          uri: https://github.com/zhaoyibo/spring-cloud-study # 配置git仓库的地址
          search-paths: config-repo # git仓库地址下的相对地址，可以配置多个，用,分割。
server:
  port: 12000

```

Spring Cloud Config 也提供本地存储配置的方式。我们只需要设置属性`spring.profiles.active=native`，Config Server 会默认从应用的`src/main/resource`目录下检索配置文件。也可以通过`spring.cloud.config.server.native.searchLocations=file:E:/properties/`属性来指定配置文件的位置。虽然 Spring Cloud Config 提供了这样的功能，但是为了支持更好的管理内容和版本控制的功能，还是推荐使用 Git 的方式。

> 如果我们的 Git 仓库需要权限访问，那么可以通过配置下面的两个属性来实现；
> spring.cloud.config.server.git.username：访问 Git 仓库的用户名
> spring.cloud.config.server.git.password：访问 Git 仓库的用户密码

## 启动类

启动类添加`@EnableConfigServer`，激活对配置中心的支持

```
@SpringBootApplication
@EnableConfigServer
public class ConfigServerApplication {

    public static void main(String[] args) {
        SpringApplication.run(ConfigServerApplication.class, args);
    }
}

```

到此 Server 端相关配置已经完成。

## 测试

首先我们先要测试 Server 端是否可以读取到 github 上面的配置信息，直接访问 [http://localhost:12000/config-client/dev](http://localhost:12000/config-client/dev) 返回信息如下：

```
{
  "name": "config-client",
  "profiles": ["dev"],
  "label": null,
  "version": "4e3ca4b9e2bb96c9a0ba012f6c6e0b6cadc48f3e",
  "state": null,
  "propertySources": [
    {
      "name": "https://github.com/zhaoyibo/spring-cloud-study/config-repo/config-client-dev.yml",
      "source": {
        "info.profile": "dev"
      }
    }
  ]
}

```

上述的返回的信息包含了配置文件的位置、版本、配置文件的名称以及配置文件中的具体内容，说明 Server 端已经成功获取了 Git 仓库的配置信息。

如果直接查看配置文件中的配置信息可访问 [http://localhost:12000/config-client-dev.yml](http://localhost:12000/config-client-dev.yml) 返回：

```
info:
  profile: dev

```

修改配置文件`config-client-dev.yml`中配置信息为：`dev update`, 再次在浏览器访问 [http://localhost:12000/config-client-dev.yml](http://localhost:12000/config-client-dev.yml) 返回：`dev update`，说明 Server 端会自动读取最新提交的内容。

> 仓库中的配置文件会被转换成 Web 接口，访问可以参照以下的规则：
>
> - /{application}/{profile}[/{label}]
> - /{application}-{profile}.yml
> - /{label}/{application}-{profile}.yml
> - /{application}-{profile}.properties
> - /{label}/{application}-{profile}.properties
>
> 上面的 URL 会映射`{application}-{profile}.yml`对应的配置文件，其中`{label}`对应 Git 上不同的分支，默认为 master。以 config-client-dev.yml 为例子，它的 application 是 config-client，profile 是 dev。

# Client 端

在完成了上述验证之后，确定配置服务中心已经正常运作，下面我们尝试如何在微服务应用中获取上述的配置信息。
再创建一个基础的 Spring Boot 应用，命名为 config-client。

## 添加依赖

在 pom.xml 中添加下述依赖：

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-webflux</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-config</artifactId>
</dependency>

```

> 引入 spring-boot-starter-webflux 是为了方便 Web 测试。Spring WebFlux 是随 Spring 5 推出的响应式 Web 框架，这里不展开说明，如果不了解的话直接用 MVC 就好了。

## 配置文件

需要配置两个配置文件，application.yml 和 bootstrap.yml，配置分别如下：
application.yml

```
spring:
  application:
    name: config-git
server:
  port: 13000

```

bootstrap.yml

```
spring:
  cloud:
    config:
      uri: http://localhost:12000 # 配置中心的具体地址，即 config-server
      name: config-client # 对应 {application} 部分
      profile: dev # 对应 {profile} 部分
      label: master # 对应 {label} 部分，即 Git 的分支。如果配置中心使用的是本地存储，则该参数无用

```

**特别注意**：上面这些与 Spring Cloud Config 相关的属性必须配置在 bootstrap.yml 中，config 部分内容才能被正确加载。因为 config 的相关配置会先于 application.yml，而 bootstrap.yml 的加载也是先于 application.yml。

## 启动类

启动类不用修改，只用`@SpringBootApplication`就行了

```
@SpringBootApplication
public class SpringCloudConfigClientApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringCloudConfigClientApplication.class, args);
    }
}

```

在 Controller 中使用`@Value`注解来获取 Server 端参数的值

```
@RestController
public class HelloController {

    @Value("${info.profile:error}")
    private String profile;

    @GetMapping("/info")
    public Mono<String> hello() {
        return Mono.justOrEmpty(profile);
    }
}

```

## 测试

启动项目后访问 [http://localhost:13000/info](http://localhost:13000/info) 返回`dev`说明已经正确的从 Server 端获取到了参数。到此一个完整的服务端提供配置服务，客户端获取配置参数的例子就完成了。

我们再做一个小实验，手动修改 config-client-dev.yml 中配置信息为：`dev update` 提交到 Github, 再次在浏览器访问 [http://localhost:13000/info](http://localhost:13000/info) 返回：`dev`，说明获取的信息还是旧的参数，这是为什么呢？

因为 Spring Cloud Config 分服务端和客户端，服务端负责将 Git 中存储的配置文件发布成 REST 接口，客户端可以从服务端 REST 接口获取配置。但客户端并不能主动感知到配置的变化，从而主动去获取新的配置。客户端如何去主动获取新的配置信息呢，Spring Cloud 已经给我们提供了解决方案，每个客户端通过 POST 方法触发各自的 `/actuator/refresh`。

## Refresh

仅修改客户端即 config-client 项目，就可以实现 refresh 的功能。

### 添加依赖

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>

```

增加了`spring-boot-starter-actuator`包，`spring-boot-starter-actuator`是一套监控的功能，可以监控程序在运行时状态，其中就包括`/actuator/refresh`的功能。

### 开启更新机制

需要给加载变量的类上面加载`@RefreshScope`，在客户端执行`/actuator/refresh`的时候就会更新此类下面的变量值。

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

### 配置

Spring Boot 1.5.X 以上默认开通了安全认证，所以要在配置文件 application.yml 中添加以下配置以将`/actuator/refresh`这个 Endpoint 暴露出来

```
management:
  endpoints:
    web:
      exposure:
        include: refresh

```

### 测试

改造完之后，我们重启 config-client，我们以 POST 请求的方式来访问 [http://localhost:13000/actuator/refresh](http://localhost:13000/actuator/refresh) 就会更新配置文件至最新版本。

我们再来测试：

1. 访问 [http://localhost:13000/info](http://localhost:13000/info) 返回`dev`
2. 我将 Git 上对应配置文件里的值改为`dev update`
3. 执行 `curl -X POST http://localhost:13000/actuator/refresh`，返回`["config.client.version","info.profile"]%`
4. 再次访问 [http://localhost:13000/info](http://localhost:13000/info) 返回`dev update`

这就说明客户端已经得到了最新的值，Refresh 是有效的。

不过，每次手动刷新客户端也很麻烦，有没有什么办法只要提交代码就自动调用客户端来更新呢，Github 的 Webhook 是一个办法。

### Webhook

Webhook 是当某个事件发生时，通过发送 HTTP POST 请求的方式来通知信息接收方。Webhook 来监测你在 Github.com 上的各种事件，最常见的莫过于 push 事件。如果你设置了一个监测 push 事件的 Webhook，那么每当你的这个项目有了任何提交，这个 Webhook 都会被触发，这时 Github 就会发送一个 HTTP POST 请求到你配置好的地址。

如此一来，你就可以通过这种方式去自动完成一些重复性工作，比如，你可以用 Webhook 来自动触发一些持续集成（CI）工具的运作，比如 Travis CI；又或者是通过 Webhook 去部署你的线上服务器。下图就是 Github 上面的 Webhook 配置。

[![img](https://ws1.sinaimg.cn/large/006tNc79ly1fqhwt1r4fgj31kw0v9wl6.jpg)](https://ws1.sinaimg.cn/large/006tNc79ly1fqhwt1r4fgj31kw0v9wl6.jpg)

- `Payload URL` ：触发后回调的 URL
- `Content type` ：数据格式，两种一般使用 json
- `Secret` ：用作给 POST 的 body 加密的字符串。采用 HMAC 算法
- `events` ：触发的事件列表。

| events 事件类型 | 描述                |
| ----------- | ----------------- |
| push        | 仓库有 push 时触发。默认事件 |
| create      | 当有分支或标签被创建时触发     |
| delete      | 当有分支或标签被删除时触发     |

这样我们就可以利用 hook 的机制去触发客户端的更新，但是当客户端越来越多的时候，hook 机制也不够优雅了，另外每次增加客户端都需要改动 hook 也是不现实的。其实，Spring Cloud 给了我们更好解决方案——Spring Cloud Bus。后续我们将继续学习如何通过 Spring Cloud Bus 来实现以消息总线的方式进行通知配置信息的变化，完成集群上的自动化更新。

# 服务化与高可用

先来回顾一下，在前文中我们完成了什么：

- 构建了 config-server，连接到 Git 仓库
- 在 Git 上创建了一个 config-repo 目录，用来存储配置信息
- 构建了 config-client，来获取 Git 中的配置信息
- 在 config-client 中开启了 Refresh，动态刷新配置信息

在本文中，我们继续来看看 Spring Cloud Config 的一些其他能力。

# 高可用问题

## 传统作法

通常在生产环境，Config Server 与服务注册中心一样，我们也需要将其扩展为高可用的集群。在之前实现的 config-server 基础上来实现高可用非常简单，不需要我们为这些服务端做任何额外的配置，只需要遵守一个配置规则：将所有的 Config Server 都指向同一个 Git 仓库，这样所有的配置内容就通过统一的共享文件系统来维护，而客户端在指定 Config Server 位置时，只要配置 Config Server 外的均衡负载即可，就像如下图所示的结构：
[![img](https://ws4.sinaimg.cn/large/006tNc79ly1fqhz6kt3joj30ry0fa3z2.jpg)](https://ws4.sinaimg.cn/large/006tNc79ly1fqhz6kt3joj30ry0fa3z2.jpg)

## 注册为服务

虽然通过服务端负载均衡已经能够实现，但是作为架构内的配置管理，本身其实也是可以看作架构中的一个微服务。所以，另外一种方式更为简单的方法就是把 config-server 也注册为服务，这样所有客户端就能以服务的方式进行访问。通过这种方法，只需要启动多个指向同一 Git 仓库位置的 config-server 就能实现高可用了。

配置过程也非常简单，我们基于配置中心 Git 版本的内容来改造。

# 代码改造

## 服务端改造

### 添加依赖

在 pom.xml 里边添加以下依赖

```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>

```

### 配置文件

在 application.yml 里新增 Eureka 的配置

```
eureka:
  client:
    service-url:
      defaultZone: http://localhost:7000/eureka/

```

这样 Server 端的改造就完成了。先启动 Eureka 注册中心，在启动 Server 端，在浏览器中访问：[http://localhost:7000/](http://localhost:7000/) 就会看到 Server 端已经注册了到注册中心了。
[![img](https://ws3.sinaimg.cn/large/006tNc79ly1fqhzv4ouhqj31kw0bkwgn.jpg)](https://ws3.sinaimg.cn/large/006tNc79ly1fqhzv4ouhqj31kw0bkwgn.jpg)

## 客户端改造

### 添加依赖

```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>

```

### 配置文件

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

主要是去掉了`spring.cloud.config.uri`直接指向 Server 端地址的配置，增加了最后的三个配置：

- `spring.cloud.config.discovery.enabled`：开启 Config 服务发现支持
- `spring.cloud.config.discovery.serviceId`：指定 Server 端的 name, 也就是 Server 端`spring.application.name`的值
- `eureka.client.service-url.defaultZone`：指向配置中心的地址

这三个配置文件都需要放到`bootstrap.yml`的配置中。

启动 Client 端，在浏览器中访问：[http://localhost:7000/](http://localhost:7000/) 就会看到 Server 端和 Client 端都已经注册了到注册中心了。
[![img](https://ws4.sinaimg.cn/large/006tNc79ly1fqhzvwx9rdj31kw0dctbj.jpg)](https://ws4.sinaimg.cn/large/006tNc79ly1fqhzvwx9rdj31kw0dctbj.jpg)

# 高可用

为了模拟生产集群环境，我们启动两个 Server 端，端口分别为 12000 和 12001，提供高可用的 Server 端支持。

```
# 打包
./mvnw clean package -Dmaven.test.skip=true

# 启动两个 Server
java -jar target/spring-cloud-config-server-0.0.1-SNAPSHOT.jar --server.port=12000
java -jar target/spring-cloud-config-server-0.0.1-SNAPSHOT.jar --server.port=12001

```

[![img](https://ws4.sinaimg.cn/large/006tNc79ly1fqi05xzbhnj31kw0dvjuj.jpg)](https://ws4.sinaimg.cn/large/006tNc79ly1fqi05xzbhnj31kw0dvjuj.jpg)

如上图就可发现会有两个 Server 端同时提供配置中心的服务，防止某一台 down 掉之后影响整个系统的使用。

我们先单独测试服务端，分别访问：[http://localhost:12000/config-client/dev](http://localhost:12000/config-client/dev) 和 [http://localhost:12001/config-client/dev](http://localhost:12001/config-client/dev) 返回信息：

```
{
  "name": "config-client",
  "profiles": ["dev"],
  "label": null,
  "version": "90dd76966da0eed967a0cbce3320f0f7ff63eb6b",
  "state": null,
  "propertySources": [
    {
      "name": "https://github.com/zhaoyibo/spring-cloud-study/config-repo/config-client-dev.yml",
      "source": {
        "info.profile": "dev update"
      }
    }
  ]
}

```

说明两个 Server 端都正常读取到了配置信息。

再次访问 [http://localhost:13000/info](http://localhost:13000/info) 返回`dev update`。说明客户端已经读取到了 Server 端的内容，我们随机停掉一台 Server 端的服务，再次访问 [http://localhost:13000/info](http://localhost:13000/info) 依然返回`dev update`，说明达到了高可用的目的。









