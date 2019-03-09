# Kafka Streams

Kafka Streams从一个或多个输入topic进行连续的计算并输出到0或多个外部topic中。

可以通过TopologyBuilder类定义一个计算逻辑`处理器`DAG拓扑。或者也可以通过提供的高级别KStream DSL来定义转换的KStreamBuilder。（PS：计算逻辑其实就是自己的代码逻辑）

KafkaStreams类管理Kafka Streams实例的生命周期。一个stream实例可以在配置文件中为`处理器`指定一个或多个Thread。

KafkaStreams实例可以作为单个streams处理客户端（也可能是分布式的），与其他的相同应用ID的实例进行协调（无论是否在同一个进程中，在同一台机器的其他进程中，或远程机器上）。这些实例将根据输入topic分区的基础上来划分工作，以便所有的分区都被消费掉。如果实例添加或失败，所有实例将重新平衡它们之间的分区分配，以保证负载平衡。

在内部，KafkaStreams实例包含一个正常的`KafkaProducer`和`KafkaConsumer`实例，用于读取和写入，

一个简单的例子：

```
    Map<String, Object> props = new HashMap<>();
    props.put(StreamsConfig.APPLICATION_ID_CONFIG, "my-stream-processing-application");
    props.put(StreamsConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
    props.put(StreamsConfig.KEY_SERDE_CLASS_CONFIG, Serdes.String().getClass().getName());
    props.put(StreamsConfig.VALUE_SERDE_CLASS_CONFIG, Serdes.String().getClass().getName());
    StreamsConfig config = new StreamsConfig(props);

    KStreamBuilder builder = new KStreamBuilder();
    builder.stream("my-input-topic").mapValues(value -> value.length().toString()).to("my-output-topic");

    KafkaStreams streams = new KafkaStreams(builder, config);
    streams.start();
```

# Kafka Streams API

在0.10.0增加了一个新的客户端库，Kafka Stream，Kafka Stream具有Alpha的优点，你可以使用maven引入到你的项目：

```
    <dependency>
        <groupId>org.apache.kafka</groupId>
        <artifactId>kafka-streams</artifactId>
        <version>0.10.0.0</version>
    </dependency>
```