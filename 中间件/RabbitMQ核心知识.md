# RabbitMQ

消息队列可以实现流量削峰、降低系统耦合度、最终一致性、广播、提高系统性能等。

**RabbitMQ**是一个实现了AMQP协议（Advanced Message Queue Protocol）的消息队列。

AMQP协议：

- Functional Layer

  ​ 功能层，位于协议上层主要定义了一组命令（基于功能的逻辑分类），用于应用程序调用实现自身所需的业务逻辑。例如：应用程序可以通过功能层定义队列名称，生产消息到指定队列，消费指定队列消息等基于（Message queues 模型）

- Transport Layer

  ​ 传输层，基于二进制数据流传输，用于将应用程序调用的指令传回服务器，并返回结果，同时可以处理信道复用，帧处理，内容编码，心跳传输，数据传输和异常处理。

## 概念

- **producer**： producer 是一个发送消息的应用
- **exchange**：producer 并不会直接将消息发送到 queue 上，而是将消息发送给 exchange，由 exchange 按照一定规则转发给指定queue
- **queue**： queue 用来存储 producer 发送的消息
- **consumer**： consumer是接收并处理消息的应用
- **Broker:** 消息队列服务器的实体，负责接收生产者消息，然后将消息发送至消息接受者或其他Broker
- **Binding：** 绑定，把exchange和queue按照规定绑定起来。
- **Virtual host：** 虚拟主机，Broker的虚拟划分，将消费者，生产者和它们的依赖的AMQP结构隔离。

![img](https://pic1.zhimg.com/v2-0b0ff0ad543efd323f5cdd1cd7d95774_b.jpg)

## 运作原理

RabbitMQ消息传递（单个队列）：

![img](https://pic3.zhimg.com/v2-1acb424be583a66df1e129fb398af8c6_b.jpg)

RabbitMQ消息传递（多个队列）：

![img](https://pic4.zhimg.com/v2-a9afe2ede619405e23264a0476e29363_b.gif)

多个Queue的场景中，**消息会被Exchange按一定的路由规则分发到指定的Queue中去**：

- 生产者指定Message的routing key，并指定Message发送到哪个Exchange
- Queue会通过binding key绑定到指定的Exchange
- Exchange根据对比Message的routing key和Queue的binding key，然后按一定的分发路由规则，决定Message发送到哪个Queue

![img](https://pic4.zhimg.com/v2-ae6c7ecdad7e67ee1c9a9e4ee89426fb_b.gif)

**每一类Exchange都有自己的分发路由规则：**

**Fanout Exchange**：忽略key对比，发送Message到Exchange下游绑定的所有Queue

![img](https://pic4.zhimg.com/v2-06d6cbd07d275d348b0249765f26bd6f_b.gif)

**Direct Exchange**：比较Message的routing key和Queue的binding key，完全匹配时，Message才会发送到该Queue

![img](https://pic3.zhimg.com/v2-560566719b80909614fb4d97da634786_b.gif)

**Topic Exchange**：比较Message的routing key和Queue的binding key，按规则匹配成功时，Message才会发送到该Queue

- **routing key命名规则**：用"."分割的字母或数字

- **匹配规则**：

- - *：匹配单个字母或数字
  - \#：匹配0~多个字母或数字
  - 比如：*.stock.#与usd.stock、eur.stock.db匹配；但与stock.nasdaq不匹配

![img](https://pic3.zhimg.com/v2-558ca492c195870ed3fff19799b27b6a_b.gif)

**默认Exchange**：比较Message的routing key和Queue的名字，完全匹配时，Message才会发送到该Queue

![img](https://pic1.zhimg.com/v2-ccc82d0bae2677f95a13e00871a44780_b.gif)

**Headers Exchange**

headers 也是根据规则匹配, 相较于 direct 和 topic 固定地使用 routing_key , headers 则是一个自定义匹配规则的类型.
在队列与交换器绑定时, 会设定一组键值对规则, 消息中也包括一组键值对( headers 属性), 当这些键值对有一对, 或全部匹配时, 消息被投送到对应队列。

## 消息投递

- 客户端连接到消息队列服务器，打开一个Channel；
- 客户端声明一个Exchange，并设置相关属性
- 客户端声明一个Queue，并设置相关属性
- 客户端使用Routing Key，在Exchange和Queue之间建立好绑定关系
- 客户端投递消息到Exchange
- Exchange接收消息后，根据消息的Key和已经设置的Binding，进行消息路由，将消息投递到一个或多个queue

## 控制台

RabbitMQ控制台可以：

- 创建Queue
- 创建Exchange
- 通过binding key绑定Queue到Exchage
- 发送消息到Exchange
- 从Queue接收消息
- 其他运维监控功能

![img](https://pic4.zhimg.com/v2-de6d821388d33124d2e61b4ac21ef19f_b.gif)

# 客户端开发导向

## 消息消费

- 推送模式：可以注册一个消费者后，RabbitMQ会在消息可用时，自动将消息进行推送给消费者。
- 拉取模式：属于一种轮询模型，发送一次get请求，获得一个消息。如果此时RabbitMQ中没有消息，会获得一个表示空的回复。

## 消息流程

AMQP协议规定，AMQP消息必须有三部分，交换机，队列和绑定。生产者把消息发送到交换机，交换机与队列的绑定关系决定了消息如何路由到特定的队列，最终被消费者接收。

![img](https://pic1.zhimg.com/v2-7e4c2b538b4f2bcdd626938a5ad46adc_b.jpg)

Note： 消息是不能直接到达队列(Queue)的

## 交换机

消息实际上投递到的是交换机，具体路由到那个队列由交换机根据路由键(routing key)完成。

- 当你发消息到代理服务器时，即便路由键是空的，RabbitMQ也会将其和使用的路由键进行匹配。如果路由的消息不匹配任何绑定模式，消息将会进入黑洞。

交换机在队列与消息中间起到了中间层的作用，有了交换机我们可以实现更灵活的功能，RabbitMQ中有三种常用的交换机类型：

- direct: 如果路由键匹配，消息就投递到对应的队列

- fanout：投递消息给所有绑定在当前交换机上面的队列

- topic：允许实现有趣的消息通信场景，使得5不同源头的消息能够达到同一个队列。topic队列名称有两个特殊的关键字。

  - `*` 可以替换一个单词
  - `#` 可以替换所有的单词

可以理解，direct为1v1, fanout为1v所有，topic比较灵活，可以1v任意。

![img](https://pic1.zhimg.com/v2-c1976e9a3eb21e18fe4a504c671bb268_b.jpg)

## 虚拟主机

每一个虚拟主机(vhost)相当于mini版的RabbitMQ服务器，拥有自己的队列，交换机和绑定，权限… 这使得一个RabbitMQ服务众多的应用程序，而不会互相冲突。

rabbitMQ默认的虚拟主机为: “/” ，一般我们在创建Rabbit的用户时会再给用户分配一个虚拟主机。

操作虚拟主机，除了命令行之外还有一个web管理页面

```
#创建虚拟主机
rabbitmqctl add vhost [vhost_name]
#删除虚拟主机
rabbitmqctl delete vhost [vhost_name]
#列出虚拟主机
rabbitmqctl list_vhosts
```

![img](https://pic3.zhimg.com/v2-f3bf1f8652dd507a5430f175796f93ae_b.jpg)

## 消息确认

消息确认模式有：

- AcknowledgeMode.NONE：自动确认
- AcknowledgeMode.AUTO：根据情况确认
- AcknowledgeMode.MANUAL：手动确认

生成端可以添加监听事件：

```
channel.addConfirmListener(new ConfirmListener() {
			@Override
			public void handleNack(long deliveryTag, boolean multiple) throws IOException {
				System.err.println("-------no ack!-----------");
			}
			
			@Override
			public void handleAck(long deliveryTag, boolean multiple) throws IOException {
				System.err.println("-------ack!-----------");
			}
		});
```

消费端可以确认消息状态：

```
public class MyConsumer extends DefaultConsumer {

	private Channel channel ;
	
	public MyConsumer(Channel channel) {
		super(channel);
		this.channel = channel;
	}

	@Override
	public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
		System.err.println("-----------consume message----------");
		System.err.println("body: " + new String(body));
		try {
			Thread.sleep(2000);
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
		if((Integer)properties.getHeaders().get("num") == 0) {
			channel.basicNack(envelope.getDeliveryTag(), false, true);
		} else {
			channel.basicAck(envelope.getDeliveryTag(), false);
		}
	}}
```

channel.basicAck与basicNack最后一个参数指定消息是否重回队列。

basicAck 方法需要传递两个参数

- **deliveryTag（唯一标识 ID）**：当一个消费者向 RabbitMQ 注册后，会建立起一个 Channel ，RabbitMQ 会用 basic.deliver 方法向消费者推送消息，这个方法携带了一个 delivery tag， **它代表了 RabbitMQ 向该 Channel 投递的这条消息的唯一标识 ID**，是一个单调递增的正整数，delivery tag 的范围仅限于 Channel
- **multiple**：为了减少网络流量，手动确认可以被批处理，**当该参数为 true 时，则可以一次性确认 delivery_tag 小于等于传入值的所有消息**

# 进阶开发

## mandatory和immediate

mandatory
当mandatory标志位设置为true时，如果exchange根据自身类型和消息routeKey无法找到一个符合条件的queue，那么会调用basic.return方法将消息返回给生产者（Basic.Return + Content-Header + Content-Body）；当mandatory设置为false时，出现上述情形broker会直接将消息扔掉。

immediate
当immediate标志位设置为true时，如果exchange在将消息路由到queue(s)时发现对于的queue上么有消费者，那么这条消息不会放入队列中。当与消息routeKey关联的所有queue（一个或者多个）都没有消费者时，该消息会通过basic.return方法返还给生产者。

## 备份交换器

生产者在发送消息的时候如果不设置mandatory参数，那么消息在未被路由的情况下将会丢失；如果设置了mandatory参数，那么需要添加ReturnListener的编程逻辑，生产者的代码将变得复杂。如果既不想复杂化生产者的编程逻辑，又不想消息丢失，那么可以使用备份交换器，这样可以将未被路由的消息存储在RabbitMQ中，再在需要的时候去处理这些消息。

![img](http://download.broadview.com.cn/Original/1712a3a7a532e1cde9c5)

## TTL（Time-To-Live 过期时间）

- 通过队列属性设置，队列中所有消息都有相同的过期时间。
- 对消息进行单独设置，每条消息TTL可以不同。

如果上述两种方法同时使用，则消息的过期时间以两者之间TTL较小的那个数值为准。消息在队列的生存时间一旦超过设置的TTL值，就称为dead message， 消费者将无法再收到该消息。

## 队列

### 死信队列(DLX)

当消息在队列中变成死信，没有消费者进行消费的时候，消息可能会被重新发布到另外一个队列中，这个队列就是死信队列。

以下情况会导致消息进入死信队列：

- basic.reject/basic.nack 并且 requeue为false(不重回队列)的时候，消息就是死信
- 消息TTL过期
- 队列达到最大的长度

死信队列也是正常的Exchange，和一般的Exchange没什么区别，不过要做一点操作。

设置死信队列包括：

- 设置Exchange(dlx.exchange名称随意),设置Queue(dlx.queue),设置RoutingKey(#)
- 创建正常的交换机，队列，绑定，只不过加上一个参数 arguments.put(“x-dead-letter-exchange”,“dlx.exchange”)

### 优先级队列

具有更高优先级的队列具有较高的优先权，优先级高的消息具备优先被消费的特权。

### 延迟队列

延迟队列存储的对象肯定是对应的延迟消息，所谓”延迟消息”是指当消息被发送以后，并不想让消费者立即拿到消息，而是等待指定时间后，消费者才拿到这个消息进行消费。

### 镜像队列

将queue镜像到cluster中其他的节点之上。在该实现下，如果集群中的一个节点失效了，queue能自动地切换到镜像中的另一个节点以保证服务的可用性。

## 消息确认机制

### 事务机制

- txSelect用于将当前channel设置成transaction模式
- txCommit用于提交事务
- txRollback用于回滚事务

### Confirm模式

一旦信道进入confirm模式，所有在该信道上面发布的消息都会被指派一个唯一的ID(从1开始)，一旦消息被投递到所有匹配的队列之后，broker就会发送一个确认给生产者（包含消息的唯一ID）,这就使得生产者知道消息已经正确到达目的队列了。

如果消息和队列是可持久化的，那么确认消息会将消息写入磁盘之后发出，broker回传给生产者的确认消息中deliver-tag域包含了确认消息的序列号，此外broker也可以设置basic.ack的multiple域，表示到这个序列号之前的所有消息都已经得到了处理。

## 持久化

默认情况下RabbitMQ的队列和交换机在RabbitMQ服务器重启之后会消失，原因在于队列和交换机的durable属性，该属性默认情况下为false.

能从AMQP服务器崩溃中恢复的消息称为持久化消息，如果想要从崩溃中恢复那么消息必须

- 投递模式设置2，来标记消息为持久化
- 发送到持久化的交换机
- 发送到持久化的队列

缺点：消息写入磁盘性能差很多。除非特别关键的消息会使用

## 消费端限流

假设MQ服务器上面囤积了成千上万条的消息的时候，这个时候突然连接消费端，那么巨量的消息全部推过来，但是客户端无法一次性处理这么多的数据。

在高并发的时候，瞬间产生的流量很大，消息很大，而MQ有个重要的作用就是限流，限流则是消费端做的。

RabbitMQ提供了一种Qos(服务质量保证)功能，即在非自动确认消息的前提下，在一定数量的消息未被消费前，不进行消费新的消息。

```
// prefetchSize消息的限制大小，一般设置为0，在生产端限制
// prefetchCount 我们一次最多消费多少条消息，一般设置为1
// global，一般设置为false，在消费端进行限制channel.basicQos(int prefetchSize, int prefetchCount, boolean global) 
// 使用channel.basicQos(0, 1, false);channel.basicConsume(queueName, false, new MyConsumer(channel));  
```

Note： autoAck设置为false, 一定要手工签收消息

# RabbitMQ 集群与网络分区

RabbitMQ 最优秀的功能之一就是内建集群，这个功能设计的目的是允许消费者和生产者在节点崩溃的情况下继续运行，以及通过添加更多的节点来线性扩展消息通信吞吐量。RabbitMQ 内部利用 Erlang 提供的分布式通信框架 OTP 来满足上述需求，使客户端在失去一个 RabbitMQ 节点连接的情况下，还是能够重新连接到集群中的任何其他节点继续生产、消费消息。

## 概念

RabbitMQ 会始终记录以下四种类型的内部元数据：

1. 队列元数据
   包括队列名称和它们的属性，比如是否可持久化，是否自动删除
2. 交换器元数据
   交换器名称、类型、属性
3. 绑定元数据
   内部是一张表格记录如何将消息路由到队列
4. vhost 元数据
   为 vhost 内部的队列、交换器、绑定提供命名空间和安全属性

在单一节点中，RabbitMQ 会将所有这些信息存储在内存中，同时将标记为可持久化的队列、交换器、绑定存储到硬盘上。存到硬盘上可以确保队列和交换器在节点重启后能够重建。而在集群模式下同样也提供两种选择：存到硬盘上（独立节点的默认设置），存在内存中。

如果在集群中创建队列，集群只会在单个节点而不是所有节点上创建完整的队列信息（元数据、状态、内容）。结果是只有队列的所有者节点知道有关队列的所有信息，因此当集群节点崩溃时，该节点的队列和绑定就消失了，并且任何匹配该队列的绑定的新消息也丢失了。还好RabbitMQ 2.6.0之后提供了镜像队列以避免集群节点故障导致的队列内容不可用。

RabbitMQ 集群中可以共享 user、vhost、exchange等，所有的数据和状态都是必须在所有节点上复制的，例外就是上面所说的消息队列。RabbitMQ 节点可以动态的加入到集群中。

当在集群中声明队列、交换器、绑定的时候，这些操作会直到所有集群节点都成功提交元数据变更后才返回。集群中有内存节点和磁盘节点两种类型，内存节点虽然不写入磁盘，但是它的执行比磁盘节点要好。内存节点可以提供出色的性能，磁盘节点能保障配置信息在节点重启后仍然可用，那集群中如何平衡这两者呢？

RabbitMQ 只要求集群中至少有一个磁盘节点，所有其他节点可以是内存节点，当节点加入火离开集群时，它们必须要将该变更通知到至少一个磁盘节点。如果只有一个磁盘节点，刚好又是该节点崩溃了，那么集群可以继续路由消息，但不能创建队列、创建交换器、创建绑定、添加用户、更改权限、添加或删除集群节点。换句话说集群中的唯一磁盘节点崩溃的话，集群仍然可以运行，但知道该节点恢复，否则无法更改任何东西。

## 负载均衡

- **轮询法**
- **随机法**
- **源地址哈希法**
- **加权轮询法**
- **加权随机法**
- **最小连接数法**

## 特点

具体特点包括：

1. 可靠性（Reliability）

   RabbitMQ 使用一些机制来保证可靠性，如持久化、传输确认、发布确认。

2. 灵活的路由（Flexible Routing）

   在消息进入队列之前，通过 Exchange 来路由消息的。对于典型的路由功能，RabbitMQ 已经提供了一些内置的 Exchange 来实现。针对更复杂的路由功能，可以将多个 Exchange 绑定在一起，也通过插件机制实现自己的 Exchange 。

3. 消息集群（Clustering）

   多个 RabbitMQ 服务器可以组成一个集群，形成一个逻辑 Broker 。

4. 高可用（Highly Available Queues）

   队列可以在集群中的机器上进行镜像，使得在部分节点出问题的情况下队列仍然可用。

5. 多种协议（Multi-protocol）

   RabbitMQ 支持多种消息队列协议，比如 STOMP、MQTT 等等。

6. 多语言客户端（Many Clients）

   RabbitMQ 几乎支持所有常用语言，比如 Java、.NET、Ruby 等等。

7. 管理界面（Management UI）

   RabbitMQ 提供了一个易用的用户界面，使得用户可以监控和管理消息 Broker 的许多方面。

8. 跟踪机制（Tracing）

   如果消息异常，RabbitMQ 提供了消息跟踪机制，使用者可以找出发生了什么。

9. 插件机制（Plugin System）

   RabbitMQ 提供了许多插件，来从多方面进行扩展，也可以编写自己的插件。

