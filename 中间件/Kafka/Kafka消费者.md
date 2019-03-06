# Kafka消费者

Kafka客户端从集群中消费消息，并透明地处理kafka集群中出现故障服务器，透明地调节适应集群中变化的数据分区。也和服务器交互，平衡均衡消费者。

```
public class KafkaConsumer<K,V>
extends Object
implements Consumer<K,V>

```

消费者TCP长连接到broker来拉取消息。故障导致的消费者关闭失败，将会泄露这些连接，消费者不是线程安全的，可以查看更多关于[Multi-threaded（多线程）](http://kafka.apache.org/0100/javadoc/org/apache/kafka/clients/consumer/KafkaConsumer.html#multithreaded)处理的细节。

## 跨版本兼容性

该客户端可以与0.10.0或更新版本的broker集群进行通信。较早的版本可能不支持某些功能。例如，`0.10.0`broker不支持`offsetsForTimes`，因为此功能是在版本`0.10.1`中添加的。 如果你调用broker版本不可用的API时，将报* UnsupportedVersionException* 异常。

## 偏移量和消费者的位置

kafka为分区中的每条消息保存一个`偏移量（offset）`，这个`偏移量`是该分区中一条消息的唯一标示符。也表示消费者在分区的位置。例如，一个位置是5的消费者(说明已经消费了0到4的消息)，下一个接收消息的偏移量为5的消息。实际上有两个与消费者相关的“位置”概念：

消费者的位置给出了下一条记录的偏移量。它比消费者在该分区中看到的最大偏移量要大一个。 它在每次消费者在调用poll(long)中接收消息时自动增长。

“已提交”的位置是已安全保存的最后偏移量，如果进程失败或重新启动时，消费者将恢复到这个偏移量。消费者可以选择定期自动提交偏移量，也可以选择通过调用commit API来手动的控制(如：commitSync 和 commitAsync)。

这个区别是消费者来控制一条消息什么时候才被认为是已被消费的，控制权在消费者，下面我们进一步更详细地讨论。

## 消费者组和主题订阅

Kafka的`消费者组`概念，通过*进程池*瓜分消息并处理消息。这些进程可以在同一台机器运行，也可分布到多台机器上，以增加可扩展性和容错性，相同group.id的消费者将视为同一个`消费者组`。

分组中的每个消费者都通过`subscribe API`动态的订阅一个topic列表。kafka将已订阅topic的消息发送到每个消费者组中。并通过平衡分区在消费者分组中所有成员之间来达到平均。因此每个分区恰好地分配1个消费者（一个消费者组中）。所有如果一个topic有4个分区，并且一个消费者分组有只有2个消费者。那么每个消费者将消费2个分区。

消费者组的成员是动态维护的：如果一个消费者故障。分配给它的分区将重新分配给同一个分组中其他的消费者。同样的，如果一个新的消费者加入到分组，将从现有消费者中移一个给它。这被称为`重新平衡分组`，并在下面更详细地讨论。当新分区添加到订阅的topic时，或者当创建与订阅的正则表达式匹配的新topic时，也将重新平衡。将通过定时刷新自动发现新的分区，并将其分配给分组的成员。

从概念上讲，你可以将消费者分组看作是由多个进程组成的单一逻辑订阅者。作为一个多订阅系统，Kafka支持对于给定topic任何数量的消费者组，而不重复。

这是在消息系统中常见的功能的略微概括。所有进程都将是单个消费者分组的一部分（类似传统消息传递系统中的队列的语义），因此消息传递就像队列一样，在组中平衡。与传统的消息系统不同的是，虽然，你可以有多个这样的组。但每个进程都有自己的消费者组（类似于传统消息系统中pub-sub的语义），因此每个进程都会订阅到该主题的所有消息。

此外，当分组重新分配自动发生时，可以通过ConsumerRebalanceListener通知消费者，这允许他们完成必要的应用程序级逻辑，例如状态清除，手动偏移提交等。有关更多详细信息，请参阅[Kafka存储的偏移](http://kafka.apache.org/0101/javadoc/org/apache/kafka/clients/consumer/KafkaConsumer.html#rebalancecallback)。

它也允许消费者通过使用`assign(Collection)`手动分配指定分区，如果使用手动指定分配分区，那么动态分区分配和协调消费者组将失效。

## 发现消费者故障

订阅一组topic后，当调用poll(long）时，消费者将自动加入到组中。只要持续的调用poll，消费者将一直保持可用，并继续从分配的分区中接收消息。此外，消费者向服务器定时发送心跳。 如果消费者崩溃或无法在session.timeout.ms配置的时间内发送心跳，则消费者将被视为死亡，并且其分区将被重新分配。

还有一种可能，消费可能遇到“活锁”的情况，它持续的发送心跳，但是没有处理。为了预防消费者在这种情况下一直持有分区，我们使用max.poll.interval.ms活跃检测机制。 在此基础上，如果你调用的poll的频率大于最大间隔，则客户端将主动地离开组，以便其他消费者接管该分区。 发生这种情况时，你会看到offset提交失败（调用commitSync（）引发的CommitFailedException）。这是一种安全机制，保障只有活动成员能够提交offset。所以要留在组中，你必须持续调用poll。

消费者提供两个配置设置来控制poll循环：

1. `max.poll.interval.ms`：增大poll的间隔，可以为消费者提供更多的时间去处理返回的消息（调用poll(long)返回的消息，通常返回的消息都是一批）。缺点是此值越大将会延迟组重新平衡。
2. `max.poll.records`：此设置限制每次调用poll返回的消息数，这样可以更容易的预测每次poll间隔要处理的最大值。通过调整此值，可以减少poll间隔，减少重新平衡分组的

对于消息处理时间不可预测地的情况，这些选项是不够的。 处理这种情况的推荐方法是将消息处理移到另一个线程中，让消费者继续调用poll。 但是必须注意确保已提交的offset不超过实际位置。另外，你必须禁用自动提交，并只有在线程完成处理后才为记录手动提交偏移量（取决于你）。 还要注意，你需要`pause暂停`分区，不会从poll接收到新消息，让线程处理完之前返回的消息（如果你的处理能力比拉取消息的慢，那创建新线程将导致你机器内存溢出）。

## 示例

这个消费者API提供了灵活性，以涵盖各种消费场景，下面是一些例子来演示如何使用它们。

## 自动提交偏移量

这是个【自动提交偏移量】的简单的kafka消费者API。

```
  Properties props = new Properties();
     props.put("bootstrap.servers", "localhost:9092");
     props.put("group.id", "test");
     props.put("enable.auto.commit", "true");
     props.put("auto.commit.interval.ms", "1000");
     props.put("key.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
     props.put("value.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
     KafkaConsumer<String, String> consumer = new KafkaConsumer<>(props);
     consumer.subscribe(Arrays.asList("foo", "bar"));
     while (true) {
         ConsumerRecords<String, String> records = consumer.poll(100);
         for (ConsumerRecord<String, String> record : records)
             System.out.printf("offset = %d, key = %s, value = %s%n", record.offset(), record.key(), record.value());
     }

```

设置`enable.auto.commit`,偏移量由`auto.commit.interval.ms`控制自动提交的频率。

集群是通过配置bootstrap.servers指定一个或多个broker。不用指定全部的broker，它将自动发现集群中的其余的borker（最好指定多个，万一有服务器故障）。

在这个例子中，客户端订阅了主题`foo`和`bar`。消费者组叫`test`。

broker通过心跳机器自动检测test组中失败的进程，消费者会自动`ping`集群，告诉进群它还活着。只要消费者能够做到这一点，它就被认为是活着的，并保留分配给它分区的权利，如果它停止心跳的时间超过`session.timeout.ms`,那么就会认为是故障的，它的分区将被分配到别的进程。

这个`deserializer`设置如何把byte转成object类型，例子中，通过指定string解析器，我们告诉获取到的消息的key和value只是简单个string类型。

## 手动控制偏移量

不需要定时的提交offset，可以自己控制offset，当消息认为已消费过了，这个时候再去提交它们的偏移量。这个很有用的，当消费的消息结合了一些处理逻辑，这个消息就不应该认为是已经消费的，直到它完成了整个处理。

```
Properties props = new Properties();
     props.put("bootstrap.servers", "localhost:9092");
     props.put("group.id", "test");
     props.put("enable.auto.commit", "false");
     props.put("auto.commit.interval.ms", "1000");
     props.put("session.timeout.ms", "30000");
     props.put("key.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
     props.put("value.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
     KafkaConsumer<String, String> consumer = new KafkaConsumer<>(props);
     consumer.subscribe(Arrays.asList("foo", "bar"));
     final int minBatchSize = 200;
     List<ConsumerRecord<String, String>> buffer = new ArrayList<>();
     while (true) {
         ConsumerRecords<String, String> records = consumer.poll(100);
         for (ConsumerRecord<String, String> record : records) {
             buffer.add(record);
         }
         if (buffer.size() >= minBatchSize) {
             insertIntoDb(buffer);
             consumer.commitSync();
             buffer.clear();
         }
     }

```

在这个例子中，我们将消费一批消息并将它们存储在内存中。当我们积累足够多的消息后，我们再将它们批量插入到数据库中。如果我们设置offset自动提交（之前说的例子），消费将被认为是已消费的。这样会出现问题，我们的进程可能在批处理记录之后，但在它们被插入到数据库之前失败了。

为了避免这种情况，我们将在相应的记录插入数据库之后再手动提交偏移量。这样我们可以准确控制消息是成功消费的。提出一个相反的可能性：在插入数据库之后，但是在提交之前，这个过程可能会失败（即使这可能只是几毫秒，这是一种可能性）。在这种情况下，进程将获取到已提交的偏移量，并会重复插入的最后一批数据。这种方式就是所谓的“至少一次”保证，在故障情况下，可以重复。

如果您无法执行这些操作，可能会使已提交的偏移超过消耗的位置，从而导致缺少记录。 使用手动偏移控制的优点是，您可以直接控制记录何时被视为“已消耗”。

注意：使用自动提交也可以“至少一次”。但是要求你必须下次调用poll（long）之前或关闭消费者之前，处理完所有返回的数据。如果操作失败，这将会导致已提交的offset超过消费的位置，从而导致丢失消息。使用手动控制offset的有点是，你可以直接控制消息何时提交。、

上面的例子使用commitSync表示所有收到的消息为”已提交"，在某些情况下，你可以希望更精细的控制，通过指定一个明确消息的偏移量为“已提交”。在下面，我们的例子中，我们处理完每个分区中的消息后，提交偏移量。

```
try {
         while(running) {
             ConsumerRecords<String, String> records = consumer.poll(Long.MAX_VALUE);
             for (TopicPartition partition : records.partitions()) {
                 List<ConsumerRecord<String, String>> partitionRecords = records.records(partition);
                 for (ConsumerRecord<String, String> record : partitionRecords) {
                     System.out.println(record.offset() + ": " + record.value());
                 }
                 long lastOffset = partitionRecords.get(partitionRecords.size() - 1).offset();
                 consumer.commitSync(Collections.singletonMap(partition, new OffsetAndMetadata(lastOffset + 1)));
             }
         }
     } finally {
       consumer.close();
     }

```

注意：已提交的offset应始终是你的程序将读取的下一条消息的offset。因此，调用commitSync（offsets）时，你应该加1个到最后处理的消息的offset。

## 订阅指定的分区

在前面的例子中，我们订阅我们感兴趣的topic，让kafka提供给我们平分后的topic分区。但是，在有些情况下，你可能需要自己来控制分配指定分区，例如：

- 如果这个消费者进程与该分区保存了某种本地状态（如本地磁盘的键值存储），则它应该只能获取这个分区的消息。
- 如果消费者进程本身具有高可用性，并且如果它失败，会自动重新启动（可能使用集群管理框架如YARN，Mesos，或者AWS设施，或作为一个流处理框架的一部分）。 在这种情况下，不需要Kafka检测故障，重新分配分区，因为消费者进程将在另一台机器上重新启动。

要使用此模式，，你只需调用`assign（Collection）`消费指定的分区即可：

```
     String topic = "foo";
     TopicPartition partition0 = new TopicPartition(topic, 0);
     TopicPartition partition1 = new TopicPartition(topic, 1);
     consumer.assign(Arrays.asList(partition0, partition1));

```

一旦手动分配分区，你可以在循环中调用poll（跟前面的例子一样）。消费者分组仍需要提交offset，只是现在分区的设置只能通过调用`assign`修改，因为手动分配不会进行分组协调，因此消费者故障不会引发分区重新平衡。每一个消费者是独立工作的（即使和其他的消费者共享GroupId）。为了避免offset提交冲突，通常你需要确认每一个consumer实例的gorupId都是唯一的。

注意，手动分配分区（即，assgin）和动态分区分配的订阅topic模式（即，subcribe）不能混合使用。

## offset存储在其他地方

消费者可以不使用kafka内置的offset仓库。可以选择自己来存储offset。要注意的是，将消费的offset和结果存储在同一个的系统中，用原子的方式存储结果和offset，但这不能保证原子，要想消费是完全原子的，并提供的“正好一次”的消费保证比kafka默认的“至少一次”的语义要更高。你需要使用kafka的offset提交功能。

这有结合的例子。

- 如果消费的结果存储在`关系数据库`中，存储在数据库的offset，让提交结果和offset在单个事务中。这样，事物成功，则offset存储和更新。如果offset没有存储，那么偏移量也不会被更新。
- 如果offset和消费结果存储在本地仓库。例如，可以通过订阅一个指定的分区并将offset和索引数据一起存储来构建一个搜索索引。如果这是以原子的方式做的，常见的可能是，即使崩溃引起未同步的数据丢失。索引程序从它确保没有更新丢失的地方恢复，而仅仅丢失最近更新的消息。

每个消息都有自己的offset，所以要管理自己的偏移，你只需要做到以下几点：

- 配置 enable.auto.commit=false
- 使用提供的 ConsumerRecord 来保存你的位置。
- 在重启时用 seek(TopicPartition, long) 恢复消费者的位置。

当分区分配也是手动完成的（像上文搜索索引的情况），这种类型的使用是最简单的。 如果分区分配是自动完成的，需要特别小心处理分区分配变更的情况。可以通过调用`subscribe（Collection，ConsumerRebalanceListener）`和`subscribe（Pattern，ConsumerRebalanceListener）`中提供的`ConsumerRebalanceListener`实例来完成的。例如，当分区向消费者获取时，消费者将通过实现`ConsumerRebalanceListener.onPartitionsRevoked（Collection）`来给这些分区提交它们offset。当分区分配给消费者时，消费者通过`ConsumerRebalanceListener.onPartitionsAssigned(Collection)`为新的分区正确地将消费者初始化到该位置。

`ConsumerRebalanceListener`的另一个常见用法是清除应用已移动到其他位置的分区的缓存。

## 控制消费的位置

大多数情况下，消费者只是简单的从头到尾的消费消息，周期性的提交位置（自动或手动）。kafka也支持消费者去手动的控制消费的位置，可以消费之前的消息也可以跳过最近的消息。

有几种情况，手动控制消费者的位置可能是有用的。

一种场景是对于时间敏感的消费者处理程序，对足够落后的消费者，直接跳过，从最近的消费开始消费。

另一个使用场景是本地状态存储系统（上一节说的）。在这样的系统中，消费者将要在启动时初始化它的位置（无论本地存储是否包含）。同样，如果本地状态已被破坏（假设因为磁盘丢失），则可以通过重新消费所有数据并重新创建状态（假设kafka保留了足够的历史）在新的机器上重新创建。

kafka使用`seek(TopicPartition, long)`指定新的消费位置。用于查找服务器保留的最早和最新的offset的特殊的方法也可用（seekToBeginning(Collection) 和 seekToEnd(Collection)）。

## 消费者流量控制

如果消费者分配了多个分区，并同时消费所有的分区，这些分区具有相同的优先级。在一些情况下，消费者需要首先消费一些指定的分区，当指定的分区有少量或者已经没有可消费的数据时，则开始消费其他分区。

例如流处理，当处理器从2个topic获取消息并把这两个topic的消息合并，当其中一个topic长时间落后另一个，则暂停消费，以便落后的赶上来。

kafka支持动态控制消费流量，分别在future的poll(long)中使用`pause(Collection)` 和 `resume(Collection)` 来暂停消费指定分配的分区，重新开始消费指定暂停的分区。

## 多线程处理

Kafka消费者不是线程安全的。所有`网络I/O`都发生在进行调用应用程序的线程中。用户的责任是确保多线程访问正确同步的。非同步访问将导致ConcurrentModificationException。

此规则唯一的例外是`wakeup()`，它可以安全地从外部线程来中断活动操作。在这种情况下，将从操作的线程阻塞并抛出一个WakeupException。这可用于从其他线程来关闭消费者。 以下代码段显示了典型模式：

```
public class KafkaConsumerRunner implements Runnable {
     private final AtomicBoolean closed = new AtomicBoolean(false);
     private final KafkaConsumer consumer;

     public void run() {
         try {
             consumer.subscribe(Arrays.asList("topic"));
             while (!closed.get()) {
                 ConsumerRecords records = consumer.poll(10000);
                 // Handle new records
             }
         } catch (WakeupException e) {
             // Ignore exception if closing
             if (!closed.get()) throw e;
         } finally {
             consumer.close();
         }
     }

     // Shutdown hook which can be called from a separate thread
     public void shutdown() {
         closed.set(true);
         consumer.wakeup();
     }
 }

```

在单独的线程中，可以通过设置关闭标志和唤醒消费者来关闭消费者。

```
      closed.set(true);
     consumer.wakeup();

```

我们没有多线程模型的例子。但留下几个操作可用来实现多线程处理消息。

1. 每个线程一个消费者

    每个线程自己的消费者实例。这里是这种方法的优点和缺点：

   - PRO: 这是最容易实现的
   - PRO: 因为它不需要在线程之间协调，所以通常它是最快的。
   - PRO: 它按顺序处理每个分区（每个线程只处理它接受的消息）。
   - CON: 更多的消费者意味着更多的TCP连接到集群（每个线程一个）。一般kafka处理连接非常的快，所以这是一个小成本。
   - CON: 更多的消费者意味着更多的请求被发送到服务器，但稍微较少的数据批次可能导致I/O吞吐量的一些下降。
   - CON: 所有进程中的线程总数受到分区总数的限制。

2. 解耦消费和处理

    另一个替代方式是一个或多个消费者线程，它来消费所有数据，其消费所有数据并将ConsumerRecords实例切换到由实际处理记录处理的处理器线程池来消费的阻塞队列。这个选项同样有利弊：

   - PRO: 可扩展消费者和处理进程的数量。这样单个消费者的数据可分给多个处理器线程来执行，避免对分区的任何限制。
   - CON: 跨多个处理器的顺序保证需要特别注意，因为线程是独立的执行，后来的消息可能比遭到的消息先处理，这仅仅是因为线程执行的运气。如果对排序没有问题，这就不是个问题。
   - CON: 手动提交变得更困难，因为它需要协调所有的线程以确保处理对该分区的处理完成。

这种方法有多种玩法，例如，每个处理线程可以有自己的队列，消费者线程可以使用`TopicPartition`hash到这些队列中，以确保按顺序消费，并且提交也将简化。

# kafka消费者API

随着0.9.0版本，我们已经增加了一个新的Java消费者替换我们现有的基于zookeeper的高级和低级消费者。这个客户端还是测试版的质量。为了确保用户平滑升级，我们仍然维护旧的0.8版本的消费者客户端继续在0.9集群上工作，两个老的0.8 API的消费者（ [高级消费者](http://orchome.com/10) 和 [低级消费者](http://orchome.com/11)）。

这个新的消费API，清除了0.8版本的高版本和低版本消费者之间的区别，你可以通过下面的maven，引入依赖到你的客户端。

```
<dependency>
      <groupId>org.apache.kafka</groupId>
      <artifactId>kafka-clients</artifactId>
      <version>0.10.1.0</version>
</dependency>
```





