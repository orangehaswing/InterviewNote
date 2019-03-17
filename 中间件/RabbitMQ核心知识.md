# RabbitMQ

消息队列可以实现流量削峰、降低系统耦合度、最终一致性、广播、提高系统性能等。

**RabbitMQ**是一个实现了AMQP协议（Advanced Message Queue Protocol）的消息队列。

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



## 生产者与消费者

生产者： 创建消息，然后发送到代理服务器(RabbitMQ)的程序

消费者：连接到代理服务器，并订阅到队列上接收消息

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

## 消息投递策略

默认情况下RabbitMQ的队列和交换机在RabbitMQ服务器重启之后会消失，原因在于队列和交换机的durable属性，该属性默认情况下为false.

能从AMQP服务器崩溃中恢复的消息称为持久化消息，如果想要从崩溃中恢复那么消息必须

- 投递模式设置2，来标记消息为持久化
- 发送到持久化的交换机
- 发送到持久化的队列

缺点：消息写入磁盘性能差很多。除非特别关键的消息会使用

## 关键API

以上都是概念性的内容，实际我们还是要通过编程来实现我们的目的，RabbitMQ的客户端api提供了很多功能，通过看代码，来了解它的强大之处。

基本步骤之前的[RabbitMQ快速入门](http://link.zhihu.com/?target=https%3A//www.jianshu.com/p/7f47bd851c9a)已经提过了，Channel类是关键的部分：包含了很多我们想要的功能

![img](https://pic1.zhimg.com/v2-f731a1bddbdc867b89f9e2c6901768d4_b.jpg)

## 消息确认

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

## 监听不可达消息

我们的消息生产者通过指定交换机和路由键来把消息送到队列中，但有时候指定的路由键不存在，或者交换机不存在，那么消息就会return，我们可以通过添加return listener来实现：

```
channel.addReturnListener(new ReturnListener() {
			@Override
			public void handleReturn(int replyCode, String replyText, String exchange,
					String routingKey, BasicProperties properties, byte[] body) throws IOException {
				
				System.err.println("---------handle  return----------");
				System.err.println("replyCode: " + replyCode);
				System.err.println("replyText: " + replyText);
				System.err.println("exchange: " + exchange);
				System.err.println("routingKey: " + routingKey);
				System.err.println("properties: " + properties);
				System.err.println("body: " + new String(body));
			}
		});
		

		channel.basicPublish(exchange, routingKeyError, true, null, msg.getBytes());

```

在basicPublish中的Mandatory要设置为true才会生效，否则broker会删除该消息

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

## 死信队列(DLX)

当消息在队列中变成死信，没有消费者进行消费的时候，消息可能会被重新发布到另外一个队列中，这个队列就是死信队列。

以下情况会导致消息进入死信队列：

- basic.reject/basic.nack 并且 requeue为false(不重回队列)的时候，消息就是死信
- 消息TTL过期
- 队列达到最大的长度

死信队列也是正常的Exchange，和一般的Exchange没什么区别，不过要做一点操作。

设置死信队列包括：

- 设置Exchange(dlx.exchange名称随意),设置Queue(dlx.queue),设置RoutingKey(#)
- 创建正常的交换机，队列，绑定，只不过加上一个参数 arguments.put(“x-dead-letter-exchange”,“dlx.exchange”)

```
      // 这就是一个普通的交换机 和 队列 以及路由
		String exchangeName = "test_dlx_exchange";
		String routingKey = "dlx.#";
		String queueName = "test_dlx_queue";
		
		channel.exchangeDeclare(exchangeName, "topic", true, false, null);
		
		Map<String, Object> agruments = new HashMap<String, Object>();
		agruments.put("x-dead-letter-exchange", "dlx.exchange");
		//这个agruments属性，要设置到声明队列上
		channel.queueDeclare(queueName, true, false, false, agruments);
		channel.queueBind(queueName, exchangeName, routingKey);
		
		//要进行死信队列的声明:
		channel.exchangeDeclare("dlx.exchange", "topic", true, false, null);
		channel.queueDeclare("dlx.queue", true, false, false, null);
		channel.queueBind("dlx.queue", "dlx.exchange", "#");
```

## RabbitMQ 集群

RabbitMQ 最优秀的功能之一就是内建集群，这个功能设计的目的是允许消费者和生产者在节点崩溃的情况下继续运行，以及通过添加更多的节点来线性扩展消息通信吞吐量。RabbitMQ 内部利用 Erlang 提供的分布式通信框架 OTP 来满足上述需求，使客户端在失去一个 RabbitMQ 节点连接的情况下，还是能够重新连接到集群中的任何其他节点继续生产、消费消息。

### 概念

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

