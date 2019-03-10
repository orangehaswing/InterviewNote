# Kafka核心知识

## KafKa

Kafka 最早是由 LinkedIn 公司开发一种分布式的基于发布/订阅的消息系统，之后成为 Apache 的顶级项目。主要特点如下：

1. 同时为发布和订阅提供高吞吐量

   Kafka 的设计目标是以时间复杂度为 O(1) 的方式提供消息持久化能力，即使对TB 级以上数据也能保证常数时间的访问性能。即使在非常廉价的商用机器上也能做到单机支持每秒 100K 条消息的传输。

2. 消息持久化

   将消息持久化到磁盘，因此可用于批量消费，例如 ETL 以及实时应用程序。通过将数据持久化到硬盘以及 replication 防止数据丢失。

3. 分布式

   支持 Server 间的消息分区及分布式消费，同时保证每个 partition 内的消息顺序传输。这样易于向外扩展，所有的producer、broker 和 consumer 都会有多个，均为分布式的。无需停机即可扩展机器。

4. 消费消息采用 pull 模式

   消息被处理的状态是在 consumer 端维护，而不是由 server 端维护，broker 无状态，consumer 自己保存 offset。

5. 支持 online 和 offline 的场景。

   同时支持离线数据处理和实时数据处理。

kafka是一种高吞吐量的分布式发布订阅消息系统，有如下特性：

- 通过O(1)的磁盘数据结构提供消息的持久化，这种结构对于即使数以TB的消息存储也能够保持长时间的稳定性能。
- 高吞吐量：即使是非常普通的硬件kafka也可以支持每秒数十万的消息。
- 支持通过kafka服务器和消费机集群来分区消息。
- 支持Hadoop并行数据加载。

Kafka的目的是提供一个发布订阅解决方案，它可以处理消费者规模的网站中的所有动作流数据。 这些数据通常是由于吞吐量的要求而通过处理日志和日志聚合来解决。 对于像Hadoop的一样的日志数据和离线分析系统，但又要求实时处理的限制，这是一个可行的解决方案。

## 概念

在Kafka有几个比较重要的概念：

- broker

  用于标识每一个Kafka服务，当然同一台服务器上可以开多个broker,只要他们的broker id不相同即可

- Topic

  消息主题，从逻辑上区分不同的消息类型

- Partition

  用于存放消息的队列，存放的消息都是有序的，同一主题可以分多个partition，如分多个partiton时，同样会以如partition1存放1,3,5消息,partition2存放2,4,6消息。

- Produce

  消息生产者，生产消息，可指定向哪个topic，topic哪个分区中生成消息。

- Consumer

  消息消费者，消费消息，同一消息只能被同一个consumer group中的consumer所消费。consumer是通过offset进行标识消息被消费的位置。当然consumer的个数取决于此topic所划分的partition，如同一group中的consumer个数大于partition的个数，多出的consumer将不会处理消息。

![img](https://user-gold-cdn.xitu.io/2018/1/24/1612607ba2bc04f8?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

![59aff8007b758.png](https://user-gold-cdn.xitu.io/2017/9/10/2ba8f4c7db02f9607fc23722b97f3440?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

![59aff8161052f.png](https://user-gold-cdn.xitu.io/2017/9/10/30b3b5622f7be1c9325b52245eda3ee6?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

## SpringBoot使用Kafka

Spring boot Kafka是由spring对kafka操作的一种封装，方便进行对kafka操作（Spring对Kafka的操作有spring-kaka和spring-integration-kafka,示例以spring-kafka操作kafka）。

### 创建spring boot项目

在pom文件内引入”spring-kafka”jar

```
<properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
    <java.version>1.8</java.version>

    <spring-kafka.version>1.2.2.RELEASE</spring-kafka.version>
</properties>
<!-- spring-kafka -->
<dependency>
    <groupId>org.springframework.kafka</groupId>
    <artifactId>spring-kafka</artifactId>
    <version>${spring-kafka.version}</version>
</dependency>
<dependency>
    <groupId>org.springframework.kafka</groupId>
    <artifactId>spring-kafka-test</artifactId>
    <version>${spring-kafka.version}</version>
    <scope>test</scope>
</dependency>
```

配置spring application.yml

```
spring:
  kafka:
    bootstrap-servers: 172.16.128.144:9092,172.16.128.145:9092,172.16.128.146:9092
    template:
      default-topic: my-test-topic
    consumer:
      group-id: mytesttopicgroup
    listener:
      concurrency: 5
```

### Produce

添加produce配置java文件及处理文件。

SenderConfig.java

```
@Configuration
public class SenderConfig {
    @Value("${spring.kafka.bootstrap-servers}")
    private String bootstrapServers;

    @Bean
    public Map<String, Object> producerConfigs() {
        Map<String, Object> props = new HashMap<>();
        props.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, bootstrapServers);
        props.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
        props.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
        return props;
    }

    @Bean
    public ProducerFactory<String, String> producerFactory() {
        return new DefaultKafkaProducerFactory<>(producerConfigs());
    }

    @Bean
    public KafkaTemplate<String, String> kafkaTemplate() {
        return new KafkaTemplate<>(producerFactory());
    }

    @Bean
    public Sender sender() {
        return new Sender();
    }
```

Sender.java

### Consumer

添加consumer配置java文件及处理文件。

ReceiverConfig.java

```
@Configuration
@EnableKafka
public class ReceiverConfig {
    @Value("${spring.kafka.bootstrap-servers}")
    private String bootstrapServers;

    @Value("${spring.kafka.consumer.group-id}")
    private String groupId;

    @Value("${spring.kafka.listener.concurrency}")
    private int concurrency;

    @Bean
    public Map<String, Object> consumerConfigs() {
        Map<String, Object> props = new HashMap<>();

        props.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, bootstrapServers);
        props.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
        props.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
        props.put(ConsumerConfig.GROUP_ID_CONFIG, groupId);

    

        return props;
    }

    @Bean
    public ConsumerFactory<String, String> consumerFactory() {
        return new DefaultKafkaConsumerFactory<>(consumerConfigs());
    }

    @Bean
    public ConcurrentKafkaListenerContainerFactory<String, String> kafkaListenerContainerFactory() {
        ConcurrentKafkaListenerContainerFactory<String, String> factory = new ConcurrentKafkaListenerContainerFactory<>();
        factory.setConsumerFactory(consumerFactory());
        factory.setConcurrency(concurrency); 
        return factory;
    }

    @Bean
    public Receiver receiver(){
        return new Receiver();
    }

}
```

Receiver.java

```
public class Receiver {

 @Autowired
 private MessageHandle messageHandle;
 private static final Logger LOGGER = LoggerFactory.getLogger(Receiver.class);
    
 private CountDownLatch latch0 = new CountDownLatch(5);
 private CountDownLatch latch1 = new CountDownLatch(5);
 private CountDownLatch latch2 = new CountDownLatch(5);
 private CountDownLatch latch3 = new CountDownLatch(5);
 private CountDownLatch latch4 = new CountDownLatch(5);

 @KafkaListener(id = "id0", topicPartitions = {@TopicPartition(topic = "${spring.kafka.template.default-topic}", partitions = {"0"})})
 public void listenPartition0(String message) {
    LOGGER.info("received message='{}'", message);
    LOGGER.info("thread ID:" + Thread.currentThread().getId());
    latch0.countDown();
 }

 @KafkaListener(id = "id1", topicPartitions = {@TopicPartition(topic = "${spring.kafka.template.default-topic}", partitions = {"1"})})
 public void listenPartition1(String message) {
    LOGGER.info("received message='{}'", message);
    LOGGER.info("thread ID:" + Thread.currentThread().getId());
    latch1.countDown();
 }

 @KafkaListener(id = "id2", topicPartitions = {@TopicPartition(topic = "${spring.kafka.template.default-topic}", partitions = {"2"})})
 public void listenPartition2(String message) {
    LOGGER.info("received message='{}'", message);
    LOGGER.info("thread ID:" + Thread.currentThread().getId());
    latch2.countDown();
 }

 @KafkaListener(id = "id3", topicPartitions = {@TopicPartition(topic = "${spring.kafka.template.default-topic}", partitions = {"3"})})
 public void listenPartition3(String message) {
    LOGGER.info("received message='{}'", message);
    LOGGER.info("thread ID:" + Thread.currentThread().getId());
    latch3.countDown();
 }

 @KafkaListener(id = "id4", topicPartitions = {@TopicPartition(topic = "${spring.kafka.template.default-topic}", partitions = {"4"})})
 public void listenPartition4(String message) {
    LOGGER.info("received message='{}'", message);
    LOGGER.info("thread ID:" + Thread.currentThread().getId());
    latch4.countDown();
 }
}
```

### 总结

到此kafka的搭建到使用都已结束，在消费kafka消息时，建议使用spring boot + spring kafka。
由于spring boot打包部署比较方便，同一台机器上可以开多个spring boot也就是开多个进程的consumer。
如不想开多个进程处理，在spring kafka 中@KafkaListener注解可针对不同的topic，不同的partition消费，也可开不同的线程进行消费kafka。













