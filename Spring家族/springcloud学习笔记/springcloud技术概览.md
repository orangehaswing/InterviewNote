# springcloud

## 微服务架构优势

- **复杂度可控：**每一个微服务专注于单一功能，并通过定义良好的接口清晰表述服务边界。
- **独立部署：**
- **技术选型灵活：**技术选型是去中心化的。
- **容错（fault isolation）：** 故障会被隔离在单个服务中。若设计良好，其他服务可通过重试、平稳退化等机制实现应用层面的容错。
- **扩展：** 每个服务可以根据自己的需要部署到合适的硬件服务器上。当应用的不同组件在扩展需求上存在差异时，微服务架构便体现出其灵活性，因为每个服务可以根据实际需求独立进行扩展。

## CAP 定理

C——数据一致性，A——服务可用性，P——服务对网络分区故障的容错性。这三个特性在任何分布式系统中不能同时满足，最多同时满足两个。

# spring Cloud 技术概览

下图展示了 Spring Cloud 的完整技术组成：
[![img](https://ws1.sinaimg.cn/large/006tNc79ly1fqc4dcx75gj31ab0o775w.jpg)](https://ws1.sinaimg.cn/large/006tNc79ly1fqc4dcx75gj31ab0o775w.jpg)

- 服务治理：包括用于服务注册和发现的 Eureka，调用断路器 Hystrix，调用端负载均衡 Ribbon，Rest 客户端 Feign，智能服务路由 Zuul，用于监控数据收集和展示的 Spectator、Servo、Atlas，用于配置读取的 Archaius 和提供 Controller 层 Reactive 封装的 RxJava。
- 分布式链路监控：Spring Cloud Sleuth 提供了全自动、可配置的数据埋点，以收集微服务调用链路上的性能数据，并发送给 Zipkin 进行存储、统计和展示。
- 消息组件：Spring Cloud Stream 对于分布式消息的各种需求进行了抽象，包括发布订阅、分组消费、消息分片等功能，实现了微服务之间的异步通信。Spring Cloud Stream 也集成了第三方的 RabbitMQ 和 Apache Kafka 作为消息队列的实现。而 Spring Cloud Bus 基于 Spring Cloud Stream，主要提供了服务间的事件通信（比如刷新配置）。
- 配置中心：基于 Spring Cloud Netflix 和 Spring Cloud Bus，Spring 又提供了 Spring Cloud Config，实现了配置集中管理、动态刷新的配置中心概念。配置通过 Git 或者简单文件来存储，支持加解密。
- 安全控制：Spring Cloud Security 基于 OAuth2 这个开放网络的安全标准，提供了微服务环境下的单点登录、资源授权、令牌管理等功能。
- 命令行工具：Spring Cloud Cli 提供了以命令行和脚本的方式来管理微服务及 Spring Cloud 组件的方式。
- 集群工具：Spring Cloud Cluster 提供了集群选主、分布式锁（暂未实现）、一次性令牌（暂未实现）等分布式集群需要的技术组件。

## Eureka-服务中心 

Eureka 就是一个服务中心，将所有的可以提供的服务都注册到它这里来管理，其它各调用者需要的时候去注册中心获取，然后再进行调用。自动具有了注册中心、负载均衡、故障转移的功能。

- **服务注册**：每个服务单元向注册中心登记自己提供的服务，将主机与端口号、版本号、通信协议等告知注册中心，注册中心按照服务名分类组织服务清单，服务注册中心还需要以心跳的方式去监控清单中的服务是否可用，若不可用需要从服务清单中剔除，达到排除故障服务的效果。
- **服务发现**：由于在服务治理框架下运行，服务间的调用不再通过指定具体的实例地址来实现，而是通过向服务名发起请求调用实现。


- **Eureka Server**：服务的注册中心，负责维护注册的服务列表, 同其他服务注册中心一样，支持高可用配置。
- **Service Provider**：服务提供方，作为一个 Eureka Client，向 Eureka Server 做服务注册、续约和下线等操作，注册的主要数据包括服务名、机器 ip、端口号、域名等等。
- **Service Consumer**：服务消费方，作为一个 Eureka Client，向 Eureka Server 获取 Service Provider 的注册信息，并通过远程调用与 Service Provider 进行通信。

## Ribbon-负载均衡

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

## Hystrix-断路器

 服务雪崩效应是一种因 “服务提供者” 的不可用导致 “服务消费者” 的不可用, 并将不可用逐渐放大的过程。

在某个服务连续调用 超过一定阈值不响应的情况下，立即通知调用端调用失败，避免调用端持续等待而影响了整体服务。Hystrix 间隔时间会再次检查此服务，如果服务恢复将继续提供服务。

阈值：快照时间窗、请求总数下限、错误百分比下限。

资源隔离主要通过线程池来实现资源隔离。

服务雪崩：

1. 服务提供者不可用
   1. 硬件故障
   2. 程序 Bug
   3. 缓存击穿：缓存应用重启，所有缓存被清空时，以及短时间内大量缓存失效
   4. 用户大量请求：秒杀和大促销
2. 重试加大流量
   1. 用户重试：不断刷新页面甚至提交表单。
   2. 代码逻辑重试：大量服务异常后的重试逻辑
3. 服务调用者不可用：同步等待造成的资源耗尽，使用 **同步调用** 时，会产生大量的等待线程占用系统资源。一旦线程资源被耗尽，服务调用者提供的服务也将处于不可用状态。

应对策略

1. 流量控制：网关限流；用户交互限流；关闭重试
2. 改进缓存模式：缓存预加载；同步改为异步刷新
3. 服务自动扩容：AWS 的 auto scaling
4. 服务调用者降级服务：资源隔离；对依赖服务进行分类；不可用服务的调用快速失败

[ref](https://windmt.com/2018/04/15/spring-cloud-4-hystrix/)

## Hystrix Dashboard 和 Turbine监控

Hystrix-dashboard 是实时监控的工具， Turbine汇总系统内多个服务的数据并显示

## Spring Cloud Config-配置中心

Server 端将所有的配置文件服务化，需要配置文件的服务实例去 Config Server 获取对应的数据。将所有的配置文件统一整理，避免了配置文件碎片化。

## Spring Cloud Bus-通知

在真正的实践生产中，可能会有 N 多的服务需要更新配置，如果每次依靠手动 Refresh 将是一个巨大的工作量。有了 Spring Cloud Bus 之后，当我们改变配置文件提交到版本库中时，会自动的触发对应实例的 Refresh，具体的工作流程如下：

[![img](https://ws1.sinaimg.cn/large/006tNc79ly1fqc5hzoj1qj30hw0bqq38.jpg)](https://ws1.sinaimg.cn/large/006tNc79ly1fqc5hzoj1qj30hw0bqq38.jpg)

## Spring Cloud Zuul-过滤器

对于客户端而言很难发现动态改变的服务实例的访问地址信息。引入 API Gateway 作为轻量级网关，同时 API Gateway 中也会实现相关的认证逻辑从而简化内部服务之间相互调用的复杂度。

服务转发，接收并转发所有内外部的客户端调用。使用 Zuul 可以作为资源的统一访问入口，同时也可以在网关做一些权限校验等类似的功能。

## Spring Cloud Sleuth-链路跟踪

在实际的使用中我们需要监控服务和服务之间通讯的各项指标，这些数据将是我们改进系统架构的主要依据。为服务之间调用提供链路追踪。通过 Sleuth 可以很清楚的了解到一个服务请求经过了哪些服务，每个服务处理花费了多长时间。

Zipkin 是 Twitter 的一个开源项目，允许开发者收集 Twitter 各个服务上的监控数据，并提供查询接口。

## 架构

- 其中 Eureka 负责服务的注册与发现，很好将各服务连接起来
- Hystrix 负责监控服务之间的调用情况，连续多次失败进行熔断保护
- Hystrix dashboard，Turbine 负责监控 Hystrix 的熔断情况，并给予图形化的展示
- Spring Cloud Config 提供了统一的配置中心服务
- 当配置文件发生变化的时候，Spring Cloud Bus 负责通知各服务去获取最新的配置信息
- 所有对外的请求和服务，我们都通过 Zuul 来进行转发，起到 API 网关的作用
- 最后我们使用 Sleuth+Zipkin 将所有的请求数据记录下来，方便我们进行后续分析分布式链路跟踪需要 Sleuth+Zipkin 结合来实现

[ref](https://windmt.com/2018/04/14/spring-cloud-1-services-governance/)





