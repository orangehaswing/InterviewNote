# kafka消息格式

##  kafka消息格式

消息（又名记录）始终是按批次写入。一批消息用技术术语表达就是`记录批次`，记录批次包含一个或多个记录。 在低性能的情况下，一个批次只有单条消息。记录批次和记录都有自己的头文件。下面介绍了Kafka版本0.11.0及更高版本（消息格式版本v2或magic = 2）的格式。点击[此处](#https://cwiki.apache.org/confluence/display/KAFKA/A+Guide+To+The+Kafka+Protocol#AGuideToTheKafkaProtocol-Messagesets)查看有关邮件格式0和1的详细信息。

#### 5.3.1 消息批次

以下是RecordBatch的磁盘格式。

```
baseOffset: int64
batchLength: int32
partitionLeaderEpoch: int32
magic: int8 (current magic value is 2)
crc: int32
attributes: int16
    bit 0~2:
        0: no compression
        1: gzip
        2: snappy
        3: lz4
    bit 3: timestampType
    bit 4: isTransactional (0 means not transactional)
    bit 5: isControlBatch (0 means not a control batch)
    bit 6~15: unused
lastOffsetDelta: int32
firstTimestamp: int64
maxTimestamp: int64
producerId: int64
producerEpoch: int16
baseSequence: int32
records: [Record]

```

请注意，当启用压缩时，压缩记录数据将按照记录数的计数直接序列化的。

CRC涵盖从属性到批次结束的数据（即。CRC之后的所有的字节）。它位于magic字节之后，也就是说，在决定如何解析批次长度和magic字节之前，客户端必须解析magic字节。分区leader epoch字段不包括在CRC计算中，以避免broker接收的每个批次分配该字段时，重新计算CRC。CRC-32C (Castagnoli)多项式被用于计算。

压缩：与老的消息格式不同，magic v2及以上版本在日志清理时，保留原始批次中的第一个和最后一个offset/sequence号。为了当日志重新加载时能恢复生产者的状态。这个功能是必须的，假设，如果我们不保留最后的序列数。如果分区leader故障，则生产者将看到OutOfSequence错误。必须保留基本序列号以进行重复检查（broker通过验证传入批次的第一个和最后一个序列号与该生产者的最后一个序列号相匹配）。因此，当清除批次中的所有消息，以保留生产者的最后序列号时，可以在日志中具有空批次。这里有一个奇怪的是，在压缩过程中，baseTimestamp字段不保留。所以如果批次中第一条消息被压缩，那么它将改变。

#### 5.3.1.1 控制批次

控制批次包含一个称为控制记录的记录。控制记录不传递给应用程序。相反，它们被消费者用来过滤掉中止的事务消息。

控制记录的关键是符合以下模式：

```
version: int16 (current version is 0)
type: int16 (0 indicates an abort marker, 1 indicates a commit)

```

控制记录的值的模式取决于类型。 该值对客户端是不透明的。

#### 5.3.2 记录

记录级header在Kafka 0.11.0中引入。具有Header的记录的磁盘格式如下所示。

```
length: varint
attributes: int8
    bit 0~7: unused
timestampDelta: varint
offsetDelta: varint
keyLength: varint
key: byte[]
valueLen: varint
value: byte[]
Headers => [Header]
```

#### 5.4.2.1 记录的header

```
headerKeyLength: varint
headerKey: String
headerValueLength: varint
Value: byte[]

```

我们使用与Protobuf相同的varint编码。有关后者的更多信息，请参见[这里](https://developers.google.com/protocol-buffers/docs/encoding#varints)。记录中的header计数也被编码为一个varint。