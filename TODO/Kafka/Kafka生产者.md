# Kafka生产者

### kafka客户端发布`record(消息)`到kafka集群。

新的生产者是线程安全的，在线程之间共享**单个生产者**实例，通常单例比多个实例要快。

一个简单的例子，使用producer发送一个有序的key/value(键值对)，放到java的`main`方法里就能直接运行，

```
Properties props = new Properties();
 props.put("bootstrap.servers", "localhost:9092");
 props.put("acks", "all");
 props.put("retries", 0);
 props.put("batch.size", 16384);
 props.put("linger.ms", 1);
 props.put("buffer.memory", 33554432);
 props.put("key.serializer", "org.apache.kafka.common.serialization.StringSerializer");
 props.put("value.serializer", "org.apache.kafka.common.serialization.StringSerializer");

 Producer<String, String> producer = new KafkaProducer<>(props);
 for(int i = 0; i < 100; i++)
     producer.send(new ProducerRecord<String, String>("my-topic", Integer.toString(i), Integer.toString(i)));

 producer.close();

```

生产者的缓冲空间池保留尚未发送到服务器的消息，后台I/O线程负责将这些消息转换成请求发送到集群。如果使用后不关闭生产者，则会泄露这些资源。

`send()`方法是异步的，添加消息到缓冲区等待发送，并立即返回。生产者将单个的消息批量在一起发送来提高效率。

`ack`是判别请求是否为完整的条件（就是是判断是不是成功发送了）。我们指定了“all”将会阻塞消息，这种设置性能最低，但是是最可靠的。

`retries`，如果请求失败，生产者会自动重试，我们指定是0次，如果启用重试，则会有重复消息的可能性。

`producer`(生产者)缓存每个分区未发送的消息。缓存的大小是通过 `batch.size` 配置指定的。值较大的话将会产生更大的批。并需要更多的内存（因为每个“活跃”的分区都有1个缓冲区）。

默认缓冲可立即发送，即便缓冲空间还没有满，但是，如果你想减少请求的数量，可以设置`linger.ms`大于0。这将指示生产者发送请求之前等待一段时间，希望更多的消息填补到未满的批中。这类似于TCP的算法，例如上面的代码段，可能100条消息在一个请求发送，因为我们设置了linger(逗留)时间为1毫秒，然后，如果我们没有填满缓冲区，这个设置将增加1毫秒的延迟请求以等待更多的消息。需要注意的是，在高负载下，相近的时间一般也会组成批，即使是 `linger.ms=0`。在不处于高负载的情况下，如果设置比0大，以少量的延迟代价换取更少的，更有效的请求。

`buffer.memory` 控制生产者可用的缓存总量，如果消息发送速度比其传输到服务器的快，将会耗尽这个缓存空间。当缓存空间耗尽，其他发送调用将被阻塞，阻塞时间的阈值通过`max.block.ms`设定，之后它将抛出一个TimeoutException。

`key.serializer`和`value.serializer`示例，将用户提供的key和value对象ProducerRecord转换成字节，你可以使用附带的**ByteArraySerializaer**或**StringSerializer**处理简单的string或byte类型。

## send()

```
public Future<RecordMetadata> send(ProducerRecord<K,V> record,Callback callback)

```

异步发送一条消息到topic，并调用`callback`（当发送已确认）。

send是异步的，并且一旦消息被保存在`等待发送的消息缓存`中，此方法就立即返回。这样并行发送多条消息而不阻塞去等待每一条消息的响应。

发送的结果是一个[RecordMetadata](http://kafka.apache.org/0101/javadoc/org/apache/kafka/clients/producer/RecordMetadata.html)，它指定了消息发送的分区，分配的offset和消息的时间戳。如果topic使用的是CreateTime，则使用用户提供的时间戳或发送的时间（如果用户没有指定指定消息的时间戳）如果topic使用的是LogAppendTime，则追加消息时，时间戳是broker的本地时间。

由于send调用是异步的，它将为分配消息的此消息的`RecordMetadata`返回一个[Future](http://docs.oracle.com/javase/7/docs/api/java/util/concurrent/Future.html?is-external=true)。如果future调用[get()](http://docs.oracle.com/javase/7/docs/api/java/util/concurrent/Future.html?is-external=true#get)，则将阻塞，直到相关请求完成并返回该消息的metadata，或抛出发送异常。

如果要模拟一个简单的阻塞调用，你可以调用`get()`方法。

```
 byte[] key = "key".getBytes();
 byte[] value = "value".getBytes();
 ProducerRecord<byte[],byte[]> record = new ProducerRecord<byte[],byte[]>("my-topic", key, value)
 producer.send(record).get();

```

完全无阻塞的话,可以利用回调参数提供的请求完成时将调用的回调通知。

```
 ProducerRecord<byte[],byte[]> record = new ProducerRecord<byte[],byte[]>("the-topic", key, value);
 producer.send(myRecord,
               new Callback() {
                   public void onCompletion(RecordMetadata metadata, Exception e) {
                       if(e != null)
                           e.printStackTrace();
                       System.out.println("The offset of the record we just sent is: " + metadata.offset());
                   }
               });

```

发送到同一个分区的消息回调保证按一定的顺序执行，也就是说，在下面的例子中 `callback1` 保证执行 `callback2` 之前：

```
producer.send(new ProducerRecord<byte[],byte[]>(topic, partition, key1, value1), callback1);
producer.send(new ProducerRecord<byte[],byte[]>(topic, partition, key2, value2), callback2);

```

注意：callback一般在生产者的I/O线程中执行，所以是相当的快的，否则将延迟其他的线程的消息发送。如果你需要执行阻塞或计算昂贵（消耗）的回调，建议在callback主体中使用自己的[Executor](http://docs.oracle.com/javase/7/docs/api/java/util/concurrent/Executor.html?is-external=true)来并行处理。

##### pecified by:

```
send in interface Producer<K,V>

```

##### Parameters:

record - 发送的记录（消息）
callback - 用户提供的callback，服务器来调用这个callback来应答结果（null表示没有callback）。

##### Throws:

InterruptException - 如果线程在阻塞中断。
SerializationException - 如果key或value不是给定有效配置的serializers。
TimeoutException - 如果获取元数据或消息分配内存话费的时间超过max.block.ms。
KafkaException - Kafka有关的错误（不属于公共API的异常）。

# kafka生产者API

我们鼓励所有新开发的程序使用新的`Java生产者`，新的java生产者客户端比以前的Scala的客户端更快、功能更全面。通过下面的例子，引入Maven（可以更改新的版本号）。

```
    <dependency>
        <groupId>org.apache.kafka</groupId>
        <artifactId>kafka-clients</artifactId>
        <version>0.10.1.0</version>
    </dependency>
```

