# springcloud学习笔记-服务容错保护 Hystrix

分布式系统中经常会出现某个基础服务不可用造成整个系统不可用的情况，这种现象被称为服务雪崩效应。为了应对服务雪崩，一种常见的做法是手动服务降级。而 Hystrix 的出现，给我们提供了另一种选择。

[![img](https://ws4.sinaimg.cn/large/006tNc79ly1fqdrcb4m2rj30hs069mz8.jpg)](https://ws4.sinaimg.cn/large/006tNc79ly1fqdrcb4m2rj30hs069mz8.jpg)

> Hystrix [hɪst’rɪks] 的中文含义是 “豪猪”，豪猪周身长满了刺，能保护自己不受天敌的伤害，代表了一种防御机制，这与 Hystrix 本身的功能不谋而合，因此 Netflix 团队将该框架命名为 Hystrix，并使用了对应的卡通形象做作为 logo。

# 服务雪崩效应

## 定义

服务雪崩效应是一种因 **服务提供者** 的不可用导致 **服务调用者** 的不可用，并将不可用 **逐渐放大** 的过程。如果所示:


[![img](https://ws4.sinaimg.cn/large/006tNc79ly1fqdrhznd6yj30ak0cyaep.jpg)](https://ws4.sinaimg.cn/large/006tNc79ly1fqdrhznd6yj30ak0cyaep.jpg)


上图中，A 为服务提供者，B 为 A 的服务调用者，C 和 D 是 B 的服务调用者。当 A 的不可用，引起 B 的不可用，并将不可用逐渐放大 C 和 D 时，服务雪崩就形成了。

## 形成的原因

我把服务雪崩的参与者简化为 **服务提供者** 和 **服务调用者**，并将服务雪崩产生的过程分为以下三个阶段来分析形成的原因:

1. 服务提供者不可用
2. 重试加大流量
3. 服务调用者不可用

[![img](https://ws4.sinaimg.cn/large/006tNc79ly1fqdriwoonkj30ak0e3gpl.jpg)](https://ws4.sinaimg.cn/large/006tNc79ly1fqdriwoonkj30ak0e3gpl.jpg)

服务雪崩的每个阶段都可能由不同的原因造成，比如造成 **服务不可用** 的原因有:

- 硬件故障
- 程序 Bug
- 缓存击穿
- 用户大量请求

硬件故障可能为硬件损坏造成的服务器主机宕机，网络硬件故障造成的服务提供者的不可访问。
缓存击穿一般发生在缓存应用重启，所有缓存被清空时，以及短时间内大量缓存失效时。大量的缓存不命中，使请求直击后端，造成服务提供者超负荷运行，引起服务不可用。
在秒杀和大促开始前，如果准备不充分，用户发起大量请求也会造成服务提供者的不可用。

而形成 **重试加大流量** 的原因有:

- 用户重试
- 代码逻辑重试

在服务提供者不可用后，用户由于忍受不了界面上长时间的等待，而不断刷新页面甚至提交表单。
服务调用端的会存在大量服务异常后的重试逻辑。
这些重试都会进一步加大请求流量。

最后, **服务调用者不可用** 产生的主要原因是:

- 同步等待造成的资源耗尽

当服务调用者使用 **同步调用** 时，会产生大量的等待线程占用系统资源。一旦线程资源被耗尽，服务调用者提供的服务也将处于不可用状态，于是服务雪崩效应产生了。

## 应对策略

针对造成服务雪崩的不同原因，可以使用不同的应对策略:

1. 流量控制
2. 改进缓存模式
3. 服务自动扩容
4. 服务调用者降级服务

**流量控制** 的具体措施包括:

- 网关限流
- 用户交互限流
- 关闭重试

因为 Nginx 的高性能，目前一线互联网公司大量采用 Nginx+Lua 的网关进行流量控制，由此而来的 OpenResty 也越来越热门。

用户交互限流的具体措施有: 1. 采用加载动画，提高用户的忍耐等待时间。2. 提交按钮添加强制等待时间机制。

**改进缓存模式** 的措施包括:

- 缓存预加载
- 同步改为异步刷新

**服务自动扩容** 的措施主要有:

- AWS 的 auto scaling

**服务调用者降级服务** 的措施包括:

- 资源隔离
- 对依赖服务进行分类
- 不可用服务的调用快速失败

资源隔离主要是对调用服务的线程池进行隔离。

我们根据具体业务，将依赖服务分为: 强依赖和若依赖。强依赖服务不可用会导致当前业务中止，而弱依赖服务的不可用不会导致当前业务的中止。

不可用服务的调用快速失败一般通过 **超时机制**, **熔断器** 和熔断后的 **降级方法** 来实现。

# 使用 Hystrix 预防服务雪崩

因为一个接口的异常，有可能导制线程阻塞，影响到其它接口的服务，甚至整个系统的服务给拖跨。Hystrix提供了几种解决方案，分别为：

- 线程池隔离
- 信号量隔离
- 熔断
- 降级回退

## 服务降级（Fallback）

对于查询操作，我们可以实现一个 fallback 方法，当请求后端服务出现异常的时候，可以使用 fallback 方法返回的值。fallback 方法的返回值一般是设置的默认值或者来自缓存。

## 资源隔离

货船为了进行防止漏水和火灾的扩散，会将货仓分隔为多个，如下图所示:

[![img](https://ws4.sinaimg.cn/large/006tNc79ly1fqdrrru8tcj30ja08ytgf.jpg)](https://ws4.sinaimg.cn/large/006tNc79ly1fqdrrru8tcj30ja08ytgf.jpg)

这种资源隔离减少风险的方式被称为: Bulkheads(舱壁隔离模式)。
Hystrix 将同样的模式运用到了服务调用者上。

在 Hystrix 中，主要通过线程池来实现资源隔离。通常在使用的时候我们会根据调用的远程服务划分出多个线程池。例如调用产品服务的 Command 放入 A 线程池，调用账户服务的 Command 放入 B 线程池。这样做的主要优点是运行环境被隔离开了。这样就算调用服务的代码存在 bug 或者由于其他原因导致自己所在线程池被耗尽时，不会对系统的其他服务造成影响。
通过对依赖服务的线程池隔离实现，可以带来如下优势：

- 应用自身得到完全的保护，不会受不可控的依赖服务影响。即便给依赖服务分配的线程池被填满，也不会影响应用自身的额其余部分。
- 可以有效的降低接入新服务的风险。如果新服务接入后运行不稳定或存在问题，完全不会影响到应用其他的请求。
- 当依赖的服务从失效恢复正常后，它的线程池会被清理并且能够马上恢复健康的服务，相比之下容器级别的清理恢复速度要慢得多。
- 当依赖的服务出现配置错误的时候，线程池会快速的反应出此问题（通过失败次数、延迟、超时、拒绝等指标的增加情况）。同时，我们可以在不影响应用功能的情况下通过实时的动态属性刷新（后续会通过 Spring Cloud Config 与 Spring Cloud Bus 的联合使用来介绍）来处理它。
- 当依赖的服务因实现机制调整等原因造成其性能出现很大变化的时候，此时线程池的监控指标信息会反映出这样的变化。同时，我们也可以通过实时动态刷新自身应用对依赖服务的阈值进行调整以适应依赖方的改变。
- 除了上面通过线程池隔离服务发挥的优点之外，每个专有线程池都提供了内置的并发实现，可以利用它为同步的依赖服务构建异步的访问。

总之，通过对依赖服务实现线程池隔离，让我们的应用更加健壮，不会因为个别依赖服务出现问题而引起非相关服务的异常。同时，也使得我们的应用变得更加灵活，可以在不停止服务的情况下，配合动态配置刷新实现性能配置上的调整。

虽然线程池隔离的方案带了如此多的好处，但是很多使用者可能会担心为每一个依赖服务都分配一个线程池是否会过多地增加系统的负载和开销。对于这一点，使用者不用过于担心，因为这些顾虑也是大部分工程师们会考虑到的，Netflix 在设计 Hystrix 的时候，认为线程池上的开销相对于隔离所带来的好处是无法比拟的。同时，Netflix 也针对线程池的开销做了相关的测试，以证明和打消 Hystrix 实现对性能影响的顾虑。

下图是 Netflix Hystrix 官方提供的一个 Hystrix 命令的性能监控，该命令以每秒 60 个请求的速度（QPS）向一个单服务实例进行访问，该服务实例每秒运行的线程数峰值为 350 个。

[![img](https://ws1.sinaimg.cn/large/006tNc79ly1fqdppn9wwgj30pr0fymyd.jpg)](https://ws1.sinaimg.cn/large/006tNc79ly1fqdppn9wwgj30pr0fymyd.jpg)

从图中的统计我们可以看到，使用线程池隔离与不使用线程池隔离的耗时差异如下表所示：

| 比较情况   | 未使用线程池隔离 | 使用了线程池隔离 | 耗时差距 |
| ------ | -------- | -------- | ---- |
| 中位数    | 2ms      | 2ms      | 2ms  |
| 90 百分位 | 5ms      | 8ms      | 3ms  |
| 99 百分位 | 28ms     | 37ms     | 9ms  |

在 99% 的情况下，使用线程池隔离的延迟有 9ms，对于大多数需求来说这样的消耗是微乎其微的，更何况为系统在稳定性和灵活性上所带来的巨大提升。虽然对于大部分的请求我们可以忽略线程池的额外开销，而对于小部分延迟本身就非常小的请求（可能只需要 1ms），那么 9ms 的延迟开销还是非常昂贵的。实际上 Hystrix 也为此设计了另外的一个解决方案：信号量（Semaphores）。

Hystrix 中除了使用线程池之外，还可以使用信号量来控制单个依赖服务的并发度，信号量的开销要远比线程池的开销小得多，但是它不能设置超时和实现异步访问。所以，只有在依赖服务是足够可靠的情况下才使用信号量。在 HystrixCommand 和 HystrixObservableCommand 中 2 处支持信号量的使用：

- 命令执行：如果隔离策略参数 execution.isolation.strategy 设置为 SEMAPHORE，Hystrix 会使用信号量替代线程池来控制依赖服务的并发控制。
- 降级逻辑：当 Hystrix 尝试降级逻辑时候，它会在调用线程中使用信号量。

信号量的默认值为 10，我们也可以通过动态刷新配置的方式来控制并发线程的数量。对于信号量大小的估算方法与线程池并发度的估算类似。仅访问内存数据的请求一般耗时在 1ms 以内，性能可以达到 5000rps，这样级别的请求我们可以将信号量设置为 1 或者 2，我们可以按此标准并根据实际请求耗时来设置信号量。

## 断路器模式

断路器模式源于 Martin Fowler 的 [Circuit Breaker](https://martinfowler.com/bliki/CircuitBreaker.html) 一文。“断路器” 本身是一种开关装置，用于在电路上保护线路过载，当线路中有电器发生短路时，“断路器” 能够及时的切断故障电路，防止发生过载、发热、甚至起火等严重后果。

在分布式架构中，断路器模式的作用也是类似的，当某个服务单元发生故障（类似用电器发生短路）之后，通过断路器的故障监控（类似熔断保险丝），直接切断原来的主逻辑调用。但是，在 Hystrix 中的断路器除了切断主逻辑的功能之外，还有更复杂的逻辑，下面我们来看看它更为深层次的处理逻辑。

断路器开关相互转换的逻辑如下图：

[![img](https://ws4.sinaimg.cn/large/006tNc79ly1fqdp243wfbj30dv06zwgz.jpg)](https://ws4.sinaimg.cn/large/006tNc79ly1fqdp243wfbj30dv06zwgz.jpg)

当 Hystrix Command 请求后端服务失败数量超过一定阈值，断路器会切换到开路状态 (Open)。这时所有请求会直接失败而不会发送到后端服务。

> 这个阈值涉及到三个重要参数：快照时间窗、请求总数下限、错误百分比下限。这个参数的作用分别是：
> 快照时间窗：断路器确定是否打开需要统计一些请求和错误数据，而统计的时间范围就是快照时间窗，默认为最近的 10 秒。
> 请求总数下限：在快照时间窗内，必须满足请求总数下限才有资格进行熔断。默认为 20，意味着在 10 秒内，如果该 Hystrix Command 的调用此时不足 20 次，即时所有的请求都超时或其他原因失败，断路器都不会打开。
> 错误百分比下限：当请求总数在快照时间窗内超过了下限，比如发生了 30 次调用，如果在这 30 次调用中，有 16 次发生了超时异常，也就是超过 50% 的错误百分比，在默认设定 50% 下限情况下，这时候就会将断路器打开。

断路器保持在开路状态一段时间后 (默认 5 秒)，自动切换到半开路状态 (HALF-OPEN)。这时会判断下一次请求的返回情况，如果请求成功，断路器切回闭路状态 (CLOSED)，否则重新切换到开路状态 (OPEN)。

# 使用 Feign Hystrix

因为熔断只是作用在服务调用这一端，因此我们根据上一篇的示例代码只需要改动 eureka-consumer-feign 项目相关代码就可以。

## POM 配置

因为 Feign 中已经依赖了 Hystrix 所以在 maven 配置上不用做任何改动。

## 配置文件

在原来的 application.yml 配置的基础上修改

```
spring:
  application:
    name: eureka-consumer-feign-hystrix
eureka:
  client:
    service-url:
      defaultZone: http://localhost:7000/eureka/
server:
  port: 9003
feign:
  hystrix:
    enabled: true

```

## 创建回调类

创建 HelloRemoteHystrix 类实现 HelloRemote 中实现回调的方法

```
@Component
public class HelloRemoteHystrix implements HelloRemote {

    @Override
    public String hello(@RequestParam(value = "name") String name) {
        return "Hello World!";
    }

}

```

## 添加 fallback 属性

在`HelloRemote`类添加指定 fallback 类，在服务熔断的时候返回 fallback 类中的内容。

```
@FeignClient(name = "eureka-producer", fallback = HelloRemoteHystrix.class)
public interface HelloRemote {

    @GetMapping("/hello/")
    String hello(@RequestParam(value = "name") String name);

}

```

别的就不用动了，很简单吧！

## 测试

依次启动 eureka-server、eureka-producer 和刚刚的 eureka-consumer-hystrix 这三个项目。

访问：[http://localhost:9003/hello/windmt](http://localhost:9003/hello/windmt)
返回：`[0]Hello, windmt! Sun Apr 15 23:14:25 CST 2018`

说明加入 Hystrix 后，不影响正常的访问。接下来我们手动停止 eureka-producer 项目再次测试：

访问：[http://localhost:9003/hello/windmt](http://localhost:9003/hello/windmt)
返回：`Hello World!`

这时候我们再次启动 eureka-producer 项目进行测试：

访问：[http://localhost:9003/hello/windmt](http://localhost:9003/hello/windmt)
返回：`[0]Hello, windmt! Sun Apr 15 23:14:52 CST 2018`

根据返回结果说明熔断成功。

# 原理分析

## 处理流程

![img](https://upload-images.jianshu.io/upload_images/7378149-8821119882b0fcec.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp)



1. 构造HystrixCommand或HystrixObservableCommand对象

2. 执行Command 命令

   execute(): 同步阻塞直至从依赖服务返回结果或抛出异常

   queue(): 异步模式，返回Future，Future封装返回的内容

   observe() : 直接订阅Observable ，此对象包含了从依赖服务返回的结果

   toObservable() : 返回Observable 对象，当你订阅他时，它会执行Hystrix命令并返回结果

3. 判断结束是否有缓存
   如果请求缓存功能开启，并且请求在缓存命中，那么返回一个Observable，此对象包含请求的结束

4. 判断短路器是否开启

5. 判断线程池/队列/信号资源是否满了

6. 执行HystrixObservableCommand.construct()或HystrixCommand.run()

7. Calculate Circuit Health

   Hystrix向断路器报告成功、失败、拒绝和超时。断路器维护一组计数器来统计执行数据。

8. 获取 Fallback逻辑

   在执行时construct() or run() ，跑出异常 (发生在步骤6.)

   断路器打开时，命令被断路 (发生在步骤4.)

   当执行命令时，依赖的线程池、队列或信号量满(发生在步骤5.)

9. 执行命令超时

10. 返回执行结束或者Observable























