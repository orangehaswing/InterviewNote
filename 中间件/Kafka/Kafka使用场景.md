# Kafka使用场景

下面是一些关于`Apache kafka` 流行的使用场景。这些领域的概述，可查看[博客文章](https://engineering.linkedin.com/distributed-systems/log-what-every-software-engineer-should-know-about-real-time-datas-unifying)。

## 消息

kafka更好的替换传统的消息系统，消息系统被用于各种场景（解耦数据生产者，缓存未处理的消息，等），与大多数消息系统比较，kafka有更好的吞吐量，内置分区，副本和故障转移，这有利于处理大规模的消息。

根据我们的经验，消息往往用于较低的吞吐量，但需要低的`端到端`延迟，并需要提供强大的耐用性的保证。

在这一领域的kafka比得上传统的消息系统，如的`ActiveMQ`或`RabbitMQ`的。

## 网站活动追踪

kafka原本的使用场景：用户的活动追踪，网站的活动（网页游览，搜索或其他用户的操作信息）发布到不同的话题中心，这些消息可实时处理，实时监测，也可加载到Hadoop或离线处理数据仓库。

每个用户页面视图都会产生非常高的量。

## 指标

kafka也常常用于监测数据。分布式应用程序生成的统计数据集中聚合。

## 日志聚合

使用kafka代替一个日志聚合的解决方案。

## 流处理

kafka消息处理包含多个阶段。其中原始输入数据是从kafka主题消费的，然后汇总，丰富，或者以其他的方式处理转化为新主题，例如，一个推荐新闻文章，文章内容可能从“articles”主题获取；然后进一步处理内容，得到一个处理后的新内容，最后推荐给用户。这种处理是基于单个主题的实时数据流。从`0.10.0.0`开始，轻量，但功能强大的流处理，就进行这样的数据处理了。

除了Kafka Streams，还有Apache Storm和Apache Samza可选择。

## 事件采集

事件采集是一种应用程序的设计风格，其中状态的变化根据时间的顺序记录下来，kafka支持这种非常大的存储日志数据的场景。

## 提交日志

kafka可以作为一种分布式的外部提交日志，日志帮助节点之间复制数据，并作为失败的节点来恢复数据重新同步，kafka的日志压缩功能很好的支持这种用法，这种用法类似于`Apacha BookKeeper`项目。

# kafka接口API

Apache Kafka引入一个新的java客户端（在org.apache.kafka.clients 包中），替代老的Scala客户端，但是为了兼容，将会共存一段时间。为了减少依赖，这些客户端都有一个独立的jar，而旧的Scala客户端继续与服务端保留在同个包下。

## Kafka有4个核心API：

- [Producer API](http://orchome.com/190) 允许应用程序发送数据流到kafka集群中的topic。
- [Consumer API](http://orchome.com/200) 允许应用程序从kafka集群的topic中读取数据流。
- [Streams API](http://orchome.com/304) 允许从输入topic转换数据流到输出topic。
- [Connect API](http://orchome.com/455) 通过实现连接器（connector），不断地从一些源系统或应用程序中拉取数据到kafka，或从kafka提交数据到宿系统（sink system）或应用程序。

kafka公开了其所有的功能协议，与语言无关。只有java客户端作为kafka项目的一部分进行维护，其他的作为开源的项目提供，这里提供了非java客户端的列表。
[https://cwiki.apache.org/confluence/display/KAFKA/Clients](https://cwiki.apache.org/confluence/display/KAFKA/Clients)





