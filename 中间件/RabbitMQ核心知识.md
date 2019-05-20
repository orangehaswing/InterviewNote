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

![img](https://github.com/orangehaswing/InterviewNote/blob/master/%E4%B8%AD%E9%97%B4%E4%BB%B6/resources/v2-0b0ff0ad543efd323f5cdd1cd7d95774_b.jpg?raw=true)

## 运作原理

RabbitMQ消息传递（单个队列）：

![img](https://github.com/orangehaswing/InterviewNote/blob/master/%E4%B8%AD%E9%97%B4%E4%BB%B6/resources/v2-1acb424be583a66df1e129fb398af8c6_b.jpg?raw=true)

RabbitMQ消息传递（多个队列）：

![img](https://github.com/orangehaswing/InterviewNote/blob/master/%E4%B8%AD%E9%97%B4%E4%BB%B6/resources/v2-a9afe2ede619405e23264a0476e29363_b.gif?raw=true)

多个Queue的场景中，**消息会被Exchange按一定的路由规则分发到指定的Queue中去**：

- 生产者指定Message的routing key，并指定Message发送到哪个Exchange
- Queue会通过binding key绑定到指定的Exchange
- Exchange根据对比Message的routing key和Queue的binding key，然后按一定的分发路由规则，决定Message发送到哪个Queue

![img](https://github.com/orangehaswing/InterviewNote/blob/master/%E4%B8%AD%E9%97%B4%E4%BB%B6/resources/v2-ae6c7ecdad7e67ee1c9a9e4ee89426fb_b.gif?raw=true)

**每一类Exchange都有自己的分发路由规则：**

**Fanout Exchange**：忽略key对比，发送Message到Exchange下游绑定的所有Queue

![img](https://github.com/orangehaswing/InterviewNote/blob/master/%E4%B8%AD%E9%97%B4%E4%BB%B6/resources/v2-06d6cbd07d275d348b0249765f26bd6f_b.gif?raw=true)

**Direct Exchange**：比较Message的routing key和Queue的binding key，完全匹配时，Message才会发送到该Queue

![img](https://github.com/orangehaswing/InterviewNote/blob/master/%E4%B8%AD%E9%97%B4%E4%BB%B6/resources/v2-560566719b80909614fb4d97da634786_b.gif?raw=true)

**Topic Exchange**：比较Message的routing key和Queue的binding key，按规则匹配成功时，Message才会发送到该Queue

- **routing key命名规则**：用"."分割的字母或数字

- **匹配规则**：

- - *：匹配单个字母或数字
  - \#：匹配0~多个字母或数字
  - 比如：*.stock.#与usd.stock、eur.stock.db匹配；但与stock.nasdaq不匹配

![img](https://github.com/orangehaswing/InterviewNote/blob/master/%E4%B8%AD%E9%97%B4%E4%BB%B6/resources/v2-558ca492c195870ed3fff19799b27b6a_b.gif?raw=true)

**默认Exchange**：比较Message的routing key和Queue的名字，完全匹配时，Message才会发送到该Queue

![img](https://github.com/orangehaswing/InterviewNote/blob/master/%E4%B8%AD%E9%97%B4%E4%BB%B6/resources/v2-ccc82d0bae2677f95a13e00871a44780_b.gif?raw=true)

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

![img](https://github.com/orangehaswing/InterviewNote/blob/master/%E4%B8%AD%E9%97%B4%E4%BB%B6/resources/v2-de6d821388d33124d2e61b4ac21ef19f_b.gif?raw=true)

# 客户端开发导向

## 消息消费

- 推送模式：可以注册一个消费者后，RabbitMQ会在消息可用时，自动将消息进行推送给消费者。
- 拉取模式：属于一种轮询模型，发送一次get请求，获得一个消息。如果此时RabbitMQ中没有消息，会获得一个表示空的回复。

## 消息流程

AMQP协议规定，AMQP消息必须有三部分，交换机，队列和绑定。生产者把消息发送到交换机，交换机与队列的绑定关系决定了消息如何路由到特定的队列，最终被消费者接收。

![img](https://github.com/orangehaswing/InterviewNote/blob/master/%E4%B8%AD%E9%97%B4%E4%BB%B6/resources/v2-7e4c2b538b4f2bcdd626938a5ad46adc_b.jpg?raw=true)

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

![img](https://github.com/orangehaswing/InterviewNote/blob/master/%E4%B8%AD%E9%97%B4%E4%BB%B6/resources/v2-c1976e9a3eb21e18fe4a504c671bb268_b.jpg?raw=true)

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

![img](https://github.com/orangehaswing/InterviewNote/blob/master/%E4%B8%AD%E9%97%B4%E4%BB%B6/resources/v2-f3bf1f8652dd507a5430f175796f93ae_b.jpg?raw=true)

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

# 进阶

## mandatory和immediate

mandatory
当mandatory标志位设置为true时，如果exchange根据自身类型和消息routeKey无法找到一个**符合条件的queue**，那么会调用basic.return方法将消息返回给生产者（Basic.Return + Content-Header + Content-Body）；当mandatory设置为false时，出现上述情形broker会直接将消息扔掉。

immediate
当immediate标志位设置为true时，如果exchange在将消息路由到queue(s)时发现对于的**queue上没有消费者**，那么这条消息不会放入队列中。当与消息routeKey关联的所有queue（一个或者多个）都没有消费者时，该消息会通过basic.return方法返还给生产者。

## 备份交换器

生产者在发送消息的时候如果不设置mandatory参数，那么消息在未被路由的情况下将会丢失；如果设置了mandatory参数，那么需要添加ReturnListener的编程逻辑，生产者的代码将变得复杂。如果既不想复杂化生产者的编程逻辑，又不想消息丢失，那么可以使用备份交换器，这样可以将未被路由的消息存储在RabbitMQ中，再在需要的时候去处理这些消息。

![img](http://download.broadview.com.cn/Original/1712a3a7a532e1cde9c5)

备份交换器和普通的交换器没有区别，为了方便使用，设置成fanout类型。

特殊情况

- 设置的备份交换器不存在，客户端和RabbitMQ服务端都不会有异常出现，消息丢失；
- 备份交换器没有绑定任何队列，客户端和RabbitMQ服务端都不会有异常出现，消息丢失；
- 备份交换器没有任何匹配的队列，客户端和RabbitMQ服务端都不会有异常出现，消息丢失；
- 备份交换器和mandatory参数一起使用，那么mandatory参数无效；

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

### 延迟队列

当消息被发送以后，并不想让消费者立即拿到消息，而是等待指定时间后，消费者才拿到这个消息进行消费。

使用场景：

- 场景一：在订单系统中，一个用户下单之后通常有30分钟的时间进行支付，如果30分钟之内没有支付成功，那么这个订单将进行一场处理。这是就可以使用延迟队列将订单信息发送到延迟队列。
- 场景二：用户希望通过手机远程遥控家里的智能设备在指定的时间进行工作。这时候就可以将用户指令发送到延迟队列，当指令设定的时间到了再将指令推送到只能设备。

实现原理：

TTL和DLX模拟出延迟队列的功能

消息在队列的生存时间一旦超过设置的TTL值，就成为dead letter。然后利用DLX，当消息在一个队列中变成死信后，它能被重新publish到另一个Exchange。这时候消息就可以重新被消费。

### 优先级队列

具有更高优先级的队列具有较高的优先权，优先级高的消息具备优先被消费的特权。

### 镜像队列

将queue镜像到cluster中其他的节点之上。如果集群中的一个节点失效了，queue能自动地切换到镜像中的另一个节点以保证服务的可用性。

在通常的用法中，针对每一个镜像队列都包含一个master和多个slave，分别对应于不同的节点。

除了publish外所有动作都只会向master发送，然后由master将命令执行的结果广播给slave们，故看似从镜像队列中的消费操作实际上是在master上执行的。

## 生产者消息确认机制

当消息的发布者在将消息发送出去之后，消息到底有没有正确到达broker代理服务器呢？如果不进行特殊配置的话，默认情况下发布操作是不会返回任何信息给生产者的。

两种消息确认机制

- 通过事务实现
- 通过发送方确认(publisher confirm)机制实现

### 事务机制

- txSelect用于将当前channel设置成transaction模式
- txCommit用于提交事务
- txRollback用于回滚事务

协议转换流程

1. client发送Tx.Select，broker接收信号
2. broker发送Tx.Select-Ok，client接收信号
3. Basic.Publish(消息发送)
4. client发送Tx.Commit，broker接收信号
5. broker发送Tx.Commit-Ok，client接收信号

 txSelect用于将当前channel设置成transaction模式，txCommit用于提交事务，txRollback用于回滚事务

缺点：采用事务机制实现会降低RabbitMQ的消息吞吐量

### Confirm模式

一旦信道进入confirm模式，所有在该信道上面发布的消息都会被指派一个唯一的ID(从1开始)，一旦消息被投递到所有匹配的队列之后，broker就会发送一个确认给生产者（包含消息的唯一ID）,这就使得生产者知道消息已经正确到达目的队列了。

如果消息和队列是可持久化的，那么确认消息会将消息写入磁盘之后发出，broker回传给生产者的确认消息中deliver-tag域包含了确认消息的序列号，此外broker也可以设置basic.ack的multiple域，表示到这个序列号之前的所有消息都已经得到了处理。

优点：confirm模式是异步的，生产者可以在等信道返回确认的同时发送下一条消息。当消息最终得到确认，生产者应用便可以通过回调处理该确认消息。

## 持久化

分为三个部分：

- 交换机持久化：元数据存在丢失情况
- 队列持久化：元数据丢失情况，同时数据也会丢失
- 消息持久化：队列持久化不能保证内部存储的消息不会丢失，将消息投递模式设置为2可实现消息持久化

能从AMQP服务器崩溃中恢复的消息称为持久化消息，如果想要从崩溃中恢复那么消息必须

- 投递模式设置2，来标记消息为持久化
- 发送到持久化的交换机
- 发送到持久化的队列

缺点：消息写入磁盘性能差很多。除非特别关键的消息会使用

### 分析数据丢失情况

- 消费者方

  autoAck=true，当consumer接收到相关消息之后，没来得及处理就宕机，这样就数据丢失。

  只需将autoAck设置为false，然后在正确处理完消息之后进行手动ack（channel.basicAck）.

- RabbitMQ

  消息在正确存入RabbitMQ之后，还需要有一段时间才能存入磁盘之中，在这段时间内RabbitMQ broker发生宕机，重启等异常, 消息还没来得及落盘，那么这些消息将会丢失。

  解决方法是引入镜像队列，相当于配置了副本。如果直接点挂掉，可以自动切换到从节点，保证高可用性。

- 发送端

  在发送端引入事务机制或者发送方确认机制，保证RabbitMQ持久化完成后，进行事务提交或确认。

## 消费端要点

- 消息分发：保存消费者的列表，每发送一条消息都会为对应的消费者记数。如果达到所设定的上限，不会向这个消费者再发送任何消息。
- 消息顺序性：在业务方使用RabbitMQ进一步处理，比如在消息体内添加全局有序标识。

## 消息传输保障

RabbitMQ支持最多一次、最少一次投递消息实现。

重复消费：消费者在消费完一条消息之后，向RabbitMQ发送确认命令。由于网络断开，没有收到确认消息。RabbitMQ不会将此条消息标记删除。重新连接后，消费者还会收到一条消息。

去重：在业务客户端实现，引入全局ID，利用Redis的幂等性。

# 高阶

## 存储机制

### 队列结构

![img](https://img-blog.csdn.net/20170502193551612?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxMzI1NjgxNg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

通常队列由两部分组成：

- 一部分是AMQQueue，负责AMQP协议相关的消息处理，即接收生产者发布的消息、向消费者投递消息、处理消息confirm、acknowledge等等；

- 另一部分是BackingQueue，它提供了相关的接口供AMQQueue调用，完成消息的存储以及可能的持久化工作等。

  BackingQueue由5个子队列组成：Q1, Q2, Delta, Q3和Q4

消息一旦进入队列，不是固定不变的，它会随着系统的负载在队列中不断流动，消息的不断发生变化。

BackingQueue中消息的生命周期分为4个状态：

- Alpha：消息的内容和消息索引都在RAM中。Q1和Q4的状态。
- Beta：消息的内容保存在DISK上，消息索引保存在RAM中。Q2和Q3的状态。
- Gamma：消息内容保存在DISK上，消息索引在DISK和RAM都有。Q2和Q3的状态。
- Delta：消息内容和索引都在DISK上。

### 惰性队列

惰性队列会尽可能的将消息存入磁盘中，而在消费者消费到相应的消息时才会被加载到内存中，它的一个重要的设计目标是能够支持更长的队列，即支持更多的消息存储。

惰性队列会将接收到的消息直接存入文件系统中，而不管是持久化的或者是非持久化的，这样可以减少了内存的消耗，但是会增加I/O的使用，如果消息是持久化的，那么这样的I/O操作不可避免。

### 镜像队列

![这里写图片描述](https://img-blog.csdn.net/20170502193643535?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxMzI1NjgxNg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

通过组播GM的方式同步到各slave节点。GM负责消息的广播，mirror_queue_slave负责回调处理。

**节点的失效**

- slave失效，系统处理做些记录外不做其他处理：master依旧是master，客户端不需要采取任何行动，或者被通知slave失效。
- master失效，那么slave中的一个必须被选中为master。被选中作为新的master的slave通常是最老的那个，因为最老的slave与前任master之间的同步状态应该是最好的。

**消息的同步**

默认情况下，镜像队列中的消息不会主动同步到新节点

当ha-sync-mode=automatic时，新加入节点时会默认同步已知的镜像队列。

**宕机情况**

当slave宕机，除了与slave相连的客户端连接全部断开之外，没有其他影响。

当master宕掉时，会有以下连锁反应：

- 与master相连的客户端连接全部断开；
- 选举最老的slave节点为master。若此时所有slave处于未同步状态，则未同步部分消息丢失；
- 新的master节点requeue所有unack消息，因为这个新节点无法区分这些unack消息是否已经到达客户端，亦或是ack消息丢失在老的master的链路上，亦或者是丢在master组播ack消息到所有slave的链路上。所以处于消息可靠性的考虑，requeue所有unack的消息。此时客户端可能有重复消息；
- 如果客户端连着slave，并且Basic.Consume消费时指定了x-cancel-on-ha-failover参数，那么客户端会受到一个Consumer Cancellation Notification通知，Java SDK中会回调Consumer接口的handleCancel方法，故需覆盖此方法。如果未指定x-cancal-on-ha-failover参数，那么消费者就无法感知master宕机，会一直等待下去。

## 负载均衡

- **轮询法**
- **随机法**
- **源地址哈希法**
- **加权轮询法**
- **加权随机法**
- **最小连接数法**

## 流量控制

流量机制是用来避免消息的发送速率过快导致服务器难以支撑的情形。

使用基于信用证算法机制：通过监控各个进程的邮箱，当某个进程负载过高而来不及处理消息时，这个进程的邮箱就会开始堆积消息。当堆积到一定量时，就会阻塞而不接收上游新消息。从而上游进程邮箱也开始堆积，直到阻塞，暂停接收新的数据。

## 网络分区



## 特点

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

