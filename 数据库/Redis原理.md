# Redis原理

# 一、概述

Redis 是速度非常快的非关系型（NoSQL）内存键值数据库，可以存储键和五种不同类型的值之间的映射。

键的类型只能为字符串，值支持五种数据类型：字符串、列表、集合、散列表、有序集合。

Redis 支持很多特性，例如将内存中的数据持久化到硬盘中，使用复制来扩展读性能，使用分片来扩展写性能。可以应用在

缓存系统、排行版、计数器社交网络、消息队列系统、实时系统等场景。

单线程为什么这么快？

1. 纯内存
2. 非阻塞IO
3. 避免线程切换和竞争消耗

单线程Redis注意事项

1. 一次只运行一条命令
2. 拒绝长（慢）命令，例如：keys、flushall、flushdb、slow lua script、mutil/exec、operate big value(collection)
3. Redis其实不是单线程，fysnc file descriptor进行持久化

特性

- 速度快
- 持久化
- 多钟数据结构
- 支持多种编程语言
- 功能丰富
- 简单
- 主从复制
- 高可用，分布式

# 二、数据类型

[![img](https://github.com/orangehaswing/fullstack-tutorial/raw/master/notes/pics/redis-data-structure-types.jpeg)](https://github.com/orangehaswing/fullstack-tutorial/blob/master/notes/pics/redis-data-structure-types.jpeg)

| 结构类型   | 结构存储的值                                   | 结构的读写能力                                  |
| ------ | ---------------------------------------- | ---------------------------------------- |
| STRING | 可以是字符串、整数或者浮点数                           | 对整个字符串或者字符串的其中一部分执行操作对整数和浮点数执行自增或自减操作    |
| LIST   | 一个链表，链表上的每个节点都包含了一个字符串                   | 从两端压入或者弹出元素 对单个或者多个元素进行修剪，只保留一个范围内的元素    |
| SET    | 包含字符串的无序收集器（unordered collection），并且被包含的每个字符串都是独一无二、各不相同的 | 添加、获取、移除单个元素检查一个元素是否存在于集合中计算交集、并集、差集从集合里面随机获取元素 |
| HAST   | 包含键值对的无序散列表                              | 添加、获取、移除单个键值对获取所有键值对检查某个键是否存在            |
| ZSET   | 字符串成员（member）与浮点数分值（score）之间的有序映射，元素的排列顺序由分值的大小决定 | 添加、获取、删除元素根据分值范围或者成员来获取元素计算一个键的排名        |

### STRING

[![img](https://github.com/orangehaswing/fullstack-tutorial/raw/master/notes/assets/redis-string.png)](https://github.com/orangehaswing/fullstack-tutorial/blob/master/notes/assets/redis-string.png)

**设置语法**

```
set key value [EX seconds] [PX ms] [nx|xx]
```

- key: 键名
- value: 键值
- ex seconds: 键秒级过期时间
- ex ms: 键毫秒及过期时间
- nx: 键不存在才能设置，setnx和nx选项作用一样，用于添加，分布式锁的实现
- xx: 键存在才能设置，setxx和xx选项作用一样，用于更新

**常用命令**

```
> set hello world
OK
> get hello
"world"
> del hello
(integer) 1
> get hello
(nil)
```

书中提到一个有趣的概念，批量操作mget可以提供效率节省时间

逐条 get/se t的时间消耗公式：

```
n次get/set时间 = n次网络时间 + n次命令时间
```

批量get/set的时间消耗公式： `n次get/set时间 = 1次网络时间 + n次命令时间`

合理的使用批量操作可以提高Redis性能，但是注意不要量太大，**如果过量的话可能会导致Redis阻塞**

**时间复杂度**

- set: O(1)
- get: O(1)
- del: O(k)，k为键的个数
- mget: O(k)，k为键的个数
- mset: O(k)，k为键的个数
- append: O(1)
- str: O(1)
- getrange: O(n), n为字符串的长度

**内部编码**

- int: 8字节长整型
- embstr: 小于39字节值
- raw: 大于39字节的值

**典型场景**

- 缓存
- 计算器
- 分布式锁

**场景**

- 缓存
- 计算器
- 分布式锁

### LIST

[![img](https://github.com/orangehaswing/fullstack-tutorial/raw/master/notes/assets/1536026733016.png)](https://github.com/orangehaswing/fullstack-tutorial/blob/master/notes/assets/1536026733016.png)

```
> rpush list-key item
(integer) 1
> rpush list-key item2
(integer) 2
> rpush list-key item
(integer) 3
> lrange list-key 0 -1
1) "item"
2) "item2"
3) "item"
> lindex list-key 1
"item2"
> lpop list-key
"item"
> lrange list-key 0 -1
1) "item2"
2) "item"
```

### SET

[![img](https://github.com/orangehaswing/fullstack-tutorial/raw/master/notes/assets/1536026799672.png)](https://github.com/orangehaswing/fullstack-tutorial/blob/master/notes/assets/1536026799672.png)

```
> sadd set-key item
(integer) 1
> sadd set-key item2
(integer) 1
> sadd set-key item3
(integer) 1
> sadd set-key item
(integer) 0
> smembers set-key
1) "item"
2) "item2"
3) "item3"
> sismember set-key item4
(integer) 0
> sismember set-key item
(integer) 1
> srem set-key item2
(integer) 1
> srem set-key item2
(integer) 0
> smembers set-key
1) "item"
2) "item3"
```

### HASH

[![img](https://github.com/orangehaswing/fullstack-tutorial/raw/master/notes/assets/1536026823413.png)](https://github.com/orangehaswing/fullstack-tutorial/blob/master/notes/assets/1536026823413.png)

**创建哈希类型的键值**

```
127.0.0.1:6379> hset user name LotusChing 
(integer) 1
127.0.0.1:6379> hset user age 21
(integer) 1
127.0.0.1:6379> hset user gender "Male"
(integer) 1
```

HSET 不支持创建一次性创建多field

```
127.0.0.1:6379> hset user name "LotusChing" age 21
(error) ERR wrong number of arguments for 'hset' command
```

**获取哈希键中的field值**

```
127.0.0.1:6379> hget user name
"LotusChing"
127.0.0.1:6379> hget user age
"21"
127.0.0.1:6379> hget user gender
"Male"
```

HGET 不支持一次获取多个field

**获取哈希键中的fields**

```
127.0.0.1:6379> hekys user
1) "name"
2) "age"
```

**获取哈希键中的所有field的value**

```
127.0.0.1:6379> hvals user
1) "LotusChing"
2) "21"
```

**删除哈希键中某个field**

```
127.0.0.1:6379> hdel user age
(integer) 1
127.0.0.1:6379> hkeys user
1) "name"
```

**统计哈希中field的个数**

```
127.0.0.1:6379> hkeys user
1) "name"
2) "age"
3) "gender"
127.0.0.1:6379> hlen user
(integer) 3
```

**批量设置哈希键的field**

```
127.0.0.1:6379> hmset user name "LotusChing" age 21 gender "Male"
OK
127.0.0.1:6379> hkeys user
1) "name"
2) "age"
3) "gender"
127.0.0.1:6379> hvals user
1) "LotusChing"
2) "21"
3) "Male"
```

**批量获取哈希键中field的value**

```
127.0.0.1:6379> hmget user name age gender
1) "LotusChing"
2) "21"
3) "Male"
```

**判断哈希键中field是否存在**

```
127.0.0.1:6379> hexists user name
(integer) 1
127.0.0.1:6379> hexists user hobbies
(integer) 0
```

**一次性获取哈希键中所有的fields和values**

注意：尽量避免使用`hgetall`，因为如果哈希键field过多的话，可能会导致Redis阻塞，建议使用`hmget`获取所需哈希键中的field值，或者采用`hscan`

```
127.0.0.1:6379> hgetall user
1) "name"
2) "LotusChing"
3) "age"
4) "21"
5) "gender"
6) "Male"
```

### ZSET

[![img](https://github.com/orangehaswing/fullstack-tutorial/raw/master/notes/assets/1536026839475.png)](https://github.com/orangehaswing/fullstack-tutorial/blob/master/notes/assets/1536026839475.png)

```
> zadd zset-key 728 member1
(integer) 1
> zadd zset-key 982 member0
(integer) 1
> zadd zset-key 982 member0
(integer) 0
> zrange zset-key 0 -1 withscores
1) "member1"
2) "728"
3) "member0"
4) "982"
> zrangebyscore zset-key 0 800 withscores
1) "member1"
2) "728"
> zrem zset-key member1
(integer) 1
> zrem zset-key member1
(integer) 0
> zrange zset-key 0 -1 withscores
1) "member0"
2) "982"
```

参考资料：

- [Chapter 1: Getting to know Redis | Redis Labs](https://redislabs.com/ebook/part-1-getting-started/chapter-1-getting-to-know-redis/)

# 三、数据结构

## 字典

dictht 是一个散列表结构，使用拉链法保存哈希冲突。

```
/* This is our hash table structure. Every dictionary has two of this as we
 * implement incremental rehashing, for the old to the new table. */
typedef struct dictht {
    dictEntry **table;
    unsigned long size;
    unsigned long sizemask;
    unsigned long used;
} dictht;
```

```
typedef struct dictEntry {
    void *key;
    union {
        void *val;
        uint64_t u64;
        int64_t s64;
        double d;
    } v;
    struct dictEntry *next;
} dictEntry;
```

Redis 的字典 dict 中包含两个哈希表 dictht，这是为了方便进行 rehash 操作。在扩容时，将其中一个 dictht 上的键值对 rehash 到另一个 dictht 上面，完成之后释放空间并交换两个 dictht 的角色。

```
typedef struct dict {
    dictType *type;
    void *privdata;
    dictht ht[2];
    long rehashidx; /* rehashing not in progress if rehashidx == -1 */
    unsigned long iterators; /* number of iterators currently running */
} dict;
```

rehash 操作不是一次性完成，而是采用渐进方式，这是为了避免一次性执行过多的 rehash 操作给服务器带来过大的负担。

渐进式 rehash 通过记录 dict 的 rehashidx 完成，它从 0 开始，然后每执行一次 rehash 都会递增。例如在一次 rehash 中，要把 dict[0] rehash 到 dict[1]，这一次会把 dict[0] 上 table[rehashidx] 的键值对 rehash 到 dict[1] 上，dict[0] 的 table[rehashidx] 指向 null，并令 rehashidx++。

在 rehash 期间，每次对字典执行添加、删除、查找或者更新操作时，都会执行一次渐进式 rehash。

采用渐进式 rehash 会导致字典中的数据分散在两个 dictht 上，因此对字典的查找操作也需要到对应的 dictht 去执行。

## 跳跃表

是有序集合的底层实现之一。

跳跃表是基于多指针有序链表实现的，可以看成多个有序链表。

[![img](https://github.com/CyC2018/CS-Notes/raw/master/docs/notes/pics/beba612e-dc5b-4fc2-869d-0b23408ac90a.png)](https://github.com/CyC2018/CS-Notes/blob/master/docs/notes/pics/beba612e-dc5b-4fc2-869d-0b23408ac90a.png)

在查找时，从上层指针开始查找，找到对应的区间之后再到下一层去查找。下图演示了查找 22 的过程。

[![img](https://github.com/CyC2018/CS-Notes/raw/master/docs/notes/pics/0ea37ee2-c224-4c79-b895-e131c6805c40.png)](https://github.com/CyC2018/CS-Notes/blob/master/docs/notes/pics/0ea37ee2-c224-4c79-b895-e131c6805c40.png)

与红黑树等平衡树相比，跳跃表具有以下优点：

- 插入速度非常快速，因为不需要进行旋转等操作来维护平衡性；
- 更容易实现；
- 支持无锁操作。

# 四、使用场景

## 计数器

可以对 String 进行自增自减运算，从而实现计数器功能。

Redis 这种内存型数据库的读写性能非常高，很适合存储频繁读写的计数量。

## 缓存

将热点数据放到内存中，设置内存的最大使用量以及淘汰策略来保证缓存的命中率。

## 查找表

例如 DNS 记录就很适合使用 Redis 进行存储。

查找表和缓存类似，也是利用了 Redis 快速的查找特性。但是查找表的内容不能失效，而缓存的内容可以失效，因为缓存不作为可靠的数据来源。

## 消息队列

List 是一个双向链表，可以通过 lpop 和 lpush 写入和读取消息。

不过最好使用 Kafka、RabbitMQ 等消息中间件。

## 会话缓存

可以使用 Redis 来统一存储多台应用服务器的会话信息。

当应用服务器不再存储用户的会话信息，也就不再具有状态，一个用户可以请求任意一个应用服务器，从而更容易实现高可用性以及可伸缩性。

## 分布式锁实现

在分布式场景下，无法使用单机环境下的锁来对多个节点上的进程进行同步。

可以使用 Reids 自带的 SETNX 命令实现分布式锁，除此之外，还可以使用官方提供的 RedLock 分布式锁实现。

## 其它

Set 可以实现交集、并集等操作，从而实现共同好友等功能。

ZSet 可以实现有序性操作，从而实现排行榜等功能。

# 五、Redis 与 Memcached

两者都是非关系型内存键值数据库，主要有以下不同：

## 数据类型

Memcached 仅支持字符串类型，而 Redis 支持五种不同的数据类型，可以更灵活地解决问题。

## 数据持久化

Redis 支持两种持久化策略：RDB 快照和 AOF 日志，而 Memcached 不支持持久化。

## 内存管理机制

- 在 Redis 中，并不是所有数据都一直存储在内存中，可以将一些很久没用的 value 交换到磁盘，而 Memcached 的数据则会一直在内存中。
- Memcached 将内存分割成特定长度的块来存储数据，以完全解决内存碎片的问题。但是这种方式会使得内存的利用率不高，例如块的大小为 128 bytes，只存储 100 bytes 的数据，那么剩下的 28 bytes 就浪费掉了。

# 六、键的过期时间

Redis 可以为每个键设置过期时间，当键过期时，会自动删除该键。

对于散列表这种容器，只能为整个键设置过期时间（整个散列表），而不能为键里面的单个元素设置过期时间。

# 七、数据淘汰策略

可以设置内存最大使用量，当内存使用量超出时，会施行数据淘汰策略。

Reids 具体有 6 种淘汰策略：

| 策略              | 描述                         |
| --------------- | -------------------------- |
| volatile-lru    | 从已设置过期时间的数据集中挑选最近最少使用的数据淘汰 |
| volatile-ttl    | 从已设置过期时间的数据集中挑选将要过期的数据淘汰   |
| volatile-random | 从已设置过期时间的数据集中任意选择数据淘汰      |
| allkeys-lru     | 从所有数据集中挑选最近最少使用的数据淘汰       |
| allkeys-random  | 从所有数据集中任意选择数据进行淘汰          |
| noeviction      | 禁止驱逐数据                     |

作为内存数据库，出于对性能和内存消耗的考虑，Redis 的淘汰算法实际实现上并非针对所有 key，而是抽样一小部分并且从中选出被淘汰的 key。

使用 Redis 缓存数据时，为了提高缓存命中率，需要保证缓存数据都是热点数据。可以将内存最大使用量设置为热点数据占用的内存量，然后启用 allkeys-lru 淘汰策略，将最近最少使用的数据淘汰。

Redis 4.0 引入了 volatile-lfu 和 allkeys-lfu 淘汰策略，LFU 策略通过统计访问频率，将访问频率最少的键值对淘汰。

# 八、持久化

Redis 是内存型数据库，为了保证数据在断电后不会丢失，需要将内存中的数据持久化到硬盘上。

## RDB 持久化

将某个时间点的所有数据都存放到硬盘上。

可以将快照复制到其它服务器从而创建具有相同数据的服务器副本。

如果系统发生故障，将会丢失最后一次创建快照之后的数据。

如果数据量很大，保存快照的时间会很长。

## AOF 持久化

将写命令添加到 AOF 文件（Append Only File）的末尾。

使用 AOF 持久化需要设置同步选项，从而确保写命令什么时候会同步到磁盘文件上。这是因为对文件进行写入并不会马上将内容同步到磁盘上，而是先存储到缓冲区，然后由操作系统决定什么时候同步到磁盘。有以下同步选项：

| 选项       | 同步频率         |
| -------- | ------------ |
| always   | 每个写命令都同步     |
| everysec | 每秒同步一次       |
| no       | 让操作系统来决定何时同步 |

- always 选项会严重减低服务器的性能；
- everysec 选项比较合适，可以保证系统崩溃时只会丢失一秒左右的数据，并且 Redis 每秒执行一次同步对服务器性能几乎没有任何影响；
- no 选项并不能给服务器性能带来多大的提升，而且也会增加系统崩溃时数据丢失的数量。

随着服务器写请求的增多，AOF 文件会越来越大。Redis 提供了一种将 AOF 重写的特性，能够去除 AOF 文件中的冗余写命令。

# 十、事件

Redis 服务器是一个事件驱动程序。

一个事务包含了多个命令，服务器在执行事务期间，不会改去执行其它客户端的命令请求。

事务中的多个命令被一次性发送给服务器，而不是一条一条发送，这种方式被称为流水线，它可以减少客户端与服务器之间的网络通信次数从而提升性能。

Redis 最简单的事务实现方式是使用 MULTI 和 EXEC 命令将事务操作包围起来。

## 文件事件

服务器通过套接字与客户端或者其它服务器进行通信，文件事件就是对套接字操作的抽象。

Redis 基于 Reactor 模式开发了自己的网络事件处理器，使用 I/O 多路复用程序来同时监听多个套接字，并将到达的事件传送给文件事件分派器，分派器会根据套接字产生的事件类型调用相应的事件处理器。

[![img](https://github.com/CyC2018/CS-Notes/raw/master/docs/notes/pics/9ea86eb5-000a-4281-b948-7b567bd6f1d8.png)](https://github.com/CyC2018/CS-Notes/blob/master/docs/notes/pics/9ea86eb5-000a-4281-b948-7b567bd6f1d8.png)

## 时间事件

服务器有一些操作需要在给定的时间点执行，时间事件是对这类定时操作的抽象。

时间事件又分为：

- 定时事件：是让一段程序在指定的时间之内执行一次；
- 周期性事件：是让一段程序每隔指定时间就执行一次。

Redis 将所有时间事件都放在一个无序链表中，通过遍历整个链表查找出已到达的时间事件，并调用相应的事件处理器。

## 事件的调度与执行

服务器需要不断监听文件事件的套接字才能得到待处理的文件事件，但是不能一直监听，否则时间事件无法在规定的时间内执行，因此监听时间应该根据距离现在最近的时间事件来决定。

事件调度与执行由 aeProcessEvents 函数负责，伪代码如下：

```
def aeProcessEvents():
    # 获取到达时间离当前时间最接近的时间事件
    time_event = aeSearchNearestTimer()
    # 计算最接近的时间事件距离到达还有多少毫秒
    remaind_ms = time_event.when - unix_ts_now()
    # 如果事件已到达，那么 remaind_ms 的值可能为负数，将它设为 0
    if remaind_ms < 0:
        remaind_ms = 0
    # 根据 remaind_ms 的值，创建 timeval
    timeval = create_timeval_with_ms(remaind_ms)
    # 阻塞并等待文件事件产生，最大阻塞时间由传入的 timeval 决定
    aeApiPoll(timeval)
    # 处理所有已产生的文件事件
    procesFileEvents()
    # 处理所有已到达的时间事件
    processTimeEvents()
```

将 aeProcessEvents 函数置于一个循环里面，加上初始化和清理函数，就构成了 Redis 服务器的主函数，伪代码如下：

```
def main():
    # 初始化服务器
    init_server()
    # 一直处理事件，直到服务器关闭为止
    while server_is_not_shutdown():
        aeProcessEvents()
    # 服务器关闭，执行清理操作
    clean_server()
```

从事件处理的角度来看，服务器运行流程如下：

[![img](https://github.com/CyC2018/CS-Notes/raw/master/docs/notes/pics/c0a9fa91-da2e-4892-8c9f-80206a6f7047.png)](https://github.com/CyC2018/CS-Notes/blob/master/docs/notes/pics/c0a9fa91-da2e-4892-8c9f-80206a6f7047.png)

# 十一、复制

通过使用 slave of host port 命令来让一个服务器成为另一个服务器的从服务器。

一个从服务器只能有一个主服务器，并且不支持主主复制。

## 连接过程

1. 主服务器创建快照文件，发送给从服务器，并在发送期间使用缓冲区记录执行的写命令。快照文件发送完毕之后，开始向从服务器发送存储在缓冲区中的写命令；
2. 从服务器丢弃所有旧数据，载入主服务器发来的快照文件，之后从服务器开始接受主服务器发来的写命令；
3. 主服务器每执行一次写命令，就向从服务器发送相同的写命令。

## 主从链

随着负载不断上升，主服务器可能无法很快地更新所有从服务器，或者重新连接和重新同步从服务器将导致系统超载。为了解决这个问题，可以创建一个中间层来分担主服务器的复制工作。中间层的服务器是最上层服务器的从服务器，又是最下层服务器的主服务器。

[![img](https://github.com/CyC2018/CS-Notes/raw/master/docs/notes/pics/395a9e83-b1a1-4a1d-b170-d081e7bb5bab.png)](https://github.com/CyC2018/CS-Notes/blob/master/docs/notes/pics/395a9e83-b1a1-4a1d-b170-d081e7bb5bab.png)

## 复制实现

1. 主服务器会维护一个偏移量，每次向服务器传播N个字节的数据时，该偏移量就会加上N，比如说一开始是0，接受到一条`set key1 value1`后，其偏移量就为13
2. 从服务器也维护一个偏移量，当从服务器收到到主服务器的N个字节数据时，该偏移量会加上N。
3. 主服务器维护一个固定大小的缓冲区，每次接受到客户端写命令后，都会将对应`命令`往这个缓冲区写入。
4. 主服务器有一个唯一id
5. 从服务器连接上主服务时，会向主服务器发送上一次连接的主服务器的id以及偏移量，这里又分几种情况：
   1. 如果从服务器没传id或者id与当前主服务器不匹配，那主服务器将传送全量数据
   2. 如果从服务器的offset在缓冲区中不能找到（落后太多导致缓冲区已经被新数据覆盖了），那也会进行全量同步
   3. 如果offset能在缓冲区找到，则主服务从offset开始，将缓冲区的数据依次发送给从服务器。

# 分布式锁

解决方案通常是借助于一个第三方组件并利用它自身的排他性来达到多进程的互斥。如：

- 基于 DB 的唯一索引。
- 基于 ZK 的临时有序节点。
- 基于 Redis 的 `NX EX` 参数。

## 实现

既然是选用了 Redis，那么它就得具有排他性才行。同时它最好也有锁的一些基本特性：

- 高性能(加、解锁时高性能)
- 可以使用阻塞锁与非阻塞锁。
- 不能出现死锁。
- 可用性(不能出现节点 down 掉后加锁失败)。

这里利用 `Redis set key` 时的一个 NX 参数可以保证在这个 key 不存在的情况下写入成功。并且再加上 EX 参数可以让该 key 在超时之后自动删除。

所以利用以上两个特性可以保证在同一时刻只会有一个进程获得锁，并且不会出现死锁(最坏的情况就是超时自动删除 key)。

### 加锁

实现代码如下：

```
private static final String SET_IF_NOT_EXIST = "NX";
private static final String SET_WITH_EXPIRE_TIME = "PX";
public  boolean tryLock(String key, String request) {
String result = this.jedis.set(LOCK_PREFIX + key, request, SET_IF_NOT_EXIST, SET_WITH_EXPIRE_TIME, 10 * TIME);
    if (LOCK_MSG.equals(result)){
        return true ;
    }else {
        return false ;
    }
}
```

注意这里使用的 jedis 的api。

```
String set(String key, String value, String nxxx, String expx, long time);
```

该命令可以保证 NX EX 的原子性。一定不要把两个命令(NX EX)分开执行，如果在 NX 之后程序出现问题就有可能产生死锁。

### 阻塞锁

同时也可以实现一个阻塞锁：

```
//一直阻塞
public void lock(String key, String request) throws InterruptedException {

    for (;;){
        String result = this.jedis.set(LOCK_PREFIX + key, request, SET_IF_NOT_EXIST, SET_WITH_EXPIRE_TIME, 10 * TIME);
        if (LOCK_MSG.equals(result)){
            break ;
        }

 //防止一直消耗 CPU 	
        Thread.sleep(DEFAULT_SLEEP_TIME) ;
    }

}

 //自定义阻塞时间
 public boolean lock(String key, String request,int blockTime) throws InterruptedException {

    while (blockTime >= 0){

        String result = this.jedis.set(LOCK_PREFIX + key, request, SET_IF_NOT_EXIST, SET_WITH_EXPIRE_TIME, 10 * TIME);
        if (LOCK_MSG.equals(result)){
            return true ;
        }
        blockTime -= DEFAULT_SLEEP_TIME ;

        Thread.sleep(DEFAULT_SLEEP_TIME) ;
    }
    return false ;
}
```

### 解锁

解锁也很简单，其实就是把这个 key 删掉就万事大吉了，比如使用 `del key` 命令。

如果进程 A 获取了锁设置了超时时间，但是由于执行周期较长导致到了超时时间之后锁就自动释放了。这时进程 B 获取了该锁执行很快就释放锁。这样就会出现进程 B 将进程 A 的锁释放了。

所以最好的方式是在每次解锁时都需要判断锁**是否是自己**的。

这时就需要结合加锁机制一起实现了。

加锁时需要传递一个参数，将该参数作为这个 key 的 value，这样每次解锁时判断 value 是否相等即可。

所以解锁代码就不能是简单的 `del`了。

```
public  boolean unlock(String key,String request){
    //lua script
    String script = "if redis.call('get', KEYS[1]) == ARGV[1] then return redis.call('del', KEYS[1]) else return 0 end";
    Object result = null ;
    if (jedis instanceof Jedis){
        result = ((Jedis)this.jedis).eval(script, Collections.singletonList(LOCK_PREFIX + key), Collections.singletonList(request));
    }else if (jedis instanceof JedisCluster){
        result = ((JedisCluster)this.jedis).eval(script, Collections.singletonList(LOCK_PREFIX + key), Collections.singletonList(request));
    }else {
        //throw new RuntimeException("instance is error") ;
        return false ;
    }
    if (UNLOCK_MSG.equals(result)){
        return true ;
    }else {
        return false ;
    }
}
```

这里使用了一个 `lua` 脚本来判断 value 是否相等，相等才执行 del 命令。

使用 `lua` 也可以保证这里两个操作的原子性。

因此上文提到的四个基本特性也能满足了：

- 使用 Redis 可以保证性能。
- 阻塞锁与非阻塞锁见上文。
- 利用超时机制解决了死锁。
- Redis 支持集群部署提高了可用性。

## RedLock

**设计模型**

1. 安全性(Safety):在任意时刻，只有一个客户端可以获得锁(排他性)。
2. 避免死锁：客户端最终一定可以获得锁，即使锁住某个资源的客户端在释放锁之前崩溃或者网络不可达。
3. 容错性：只要Redsi集群中的大部分节点存活，client就可以进行加锁解锁操作。

**RedLock算法**

1. 得到本地时间

2. Client使用相同的key和随机数,按照顺序在每个Master实例中尝试获得锁。在获得锁的过程中，为每一个锁操作设置一个**快速失败时间**(如果想要获得一个10秒的锁，  那么每一个锁操作的失败时间设为5-50ms)。

   这样可以避免客户端与一个已经故障的Master通信占用太长时间，通过快速失败的方式尽快的与集群中的其他节点完成锁操作。

3. 客户端计算出与master获得锁操作过程中消耗的时间，当且仅当Client获得锁消耗的时间小于锁的存活时间，并且在一半以上的master节点中获得锁。才认为client成功的获得了锁。

4. 如果已经获得了锁，Client执行任务的时间窗口是锁的存活时间减去获得锁消耗的时间。

5. 如果Client获得锁的数量不足一半以上，或获得锁的时间超时，那么认为获得锁失败。客户端需要尝试在所有的master节点中释放锁，  即使在第二步中没有成功获得该Master节点中的锁，仍要进行释放操作。

**失败重试机制**

如果一个Client无法获得锁，它将在一个随机延时后开始重试。无论Client认为在指定的Master中有没有获得锁，都需要执行释放锁操作。

# 集群

## 集群基础实现

一个集群由多个Redis节点组成，不同的节点通过`CLUSTOR MEET`命令进行连接：

`CLUSTOR MEET  `

收到命令的节点会与命令中指定的目标节点进行握手，握手成功后目标节点会加入到集群中

![image](https://camo.githubusercontent.com/3e1b7153337b7a673a995142ecdb8dd41bf3acc1/68747470733a2f2f757365722d676f6c642d63646e2e786974752e696f2f323031382f392f31302f313635633366353436353539376631383f773d34353826683d32323226663d706e6726733d3231333936)

![image](https://camo.githubusercontent.com/0197a3ebff847578def4e31a7a8443d2cbe651bb/68747470733a2f2f757365722d676f6c642d63646e2e786974752e696f2f323031382f392f31302f313635633366353364353862303365653f773d35353226683d33373126663d706e6726733d3332333134)

![image](https://camo.githubusercontent.com/fc74b028da7e134817a9d83dd8af0ae41c3d03a7/68747470733a2f2f757365722d676f6c642d63646e2e786974752e696f2f323031382f392f31302f313635633366353464653737303761383f773d36363426683d33363526663d706e6726733d3330343532)

![image](https://camo.githubusercontent.com/1ac3eb414eb41fcd002906e608024dfa35224787/68747470733a2f2f757365722d676f6c642d63646e2e786974752e696f2f323031382f392f31302f313635633366353366306233333133363f773d35383926683d33363526663d706e6726733d3334313337)

![image](https://camo.githubusercontent.com/5e9927df7d0b5743e40cf422d5af974230a440cf/68747470733a2f2f757365722d676f6c642d63646e2e786974752e696f2f323031382f392f31302f313635633366353364353562313235303f773d36323626683d33323126663d706e6726733d3238303939)

### 槽分配

一个集群的所有数据被分为16384个槽，可以通过`CLUSTER ADDSLOTS`命令将槽指派给对应的节点。当所有的槽都有节点负责时，集群处于上线状态，否则处于下线状态不对外提供服务。

clusterNode的位数组slots代表一个节点负责的槽信息。

```
struct clusterNode {


    unsigned char slots[16384/8]; /* slots handled by this node */

    int numslots;   /* Number of slots handled by this node */

    ...
}

```

看个例子，下图中1、3、5、8、9、10位的值为1，代表该节点负责槽1、3、5、8、9、10。

每个Redis Server上都有一个ClusterState的对象，代表了该Server所在集群的信息，其中字段slots记录了集群中所有节点负责的槽信息。

```
typedef struct clusterState {

    // 负责处理各个槽的节点
    // 例如 slots[i] = clusterNode_A 表示槽 i 由节点 A 处理
    // slots[i] = null 代表该槽目前没有节点负责
    clusterNode *slots[REDIS_CLUSTER_SLOTS];

}
```

### 槽重分配

可以通过redis-trib工具对槽重新分配，重分配的实现步骤如下：

1. 通知目标节点准备好接收槽
2. 通知源节点准备好发送槽
3. 向源节点发送命令：`CLUSTER GETKEYSINSLOT  `从源节点获取最多count个槽slot的key
4. 对于步骤3的每个key，都向源节点发送一个`MIGRATE    0 `命令，将被选中的键原子的从源节点迁移至目标节点。
5. 重复步骤3、4。直到槽slot的所有键值对都被迁移到目标节点
6. 将槽slot指派给目标节点的信息发送到整个集群。

在槽重分配的过程中，槽中的一部分数据保存着源节点，另一部分保存在目标节点。这时如果要客户端向源节点发送一个命令，且相关数据在一个正在迁移槽中，源节点处理步骤如图:
[![image](https://camo.githubusercontent.com/9012272647179aa0817fe5181ccf89f794fdc248/68747470733a2f2f757365722d676f6c642d63646e2e786974752e696f2f323031382f392f31302f313635633366353365663764623564623f773d39393626683d34373426663d706e6726733d313838343836)](https://camo.githubusercontent.com/9012272647179aa0817fe5181ccf89f794fdc248/68747470733a2f2f757365722d676f6c642d63646e2e786974752e696f2f323031382f392f31302f313635633366353365663764623564623f773d39393626683d34373426663d706e6726733d313838343836)

当客户端收到一个ASK错误的时候，会根据返回的信息向目标节点重新发起一次请求。

ASK和MOVED的区别主要是ASK是一次性的，MOVED是永久性的，有点像Http协议中的301和302。

### 一次命令执行过程

我们来看clustor下一次命令的请求过程,假设执行命令 `get testKey`

1. clustor client在运行前需要配置若干个server节点的ip和port。我们称这些节点为种子节点。
2. clustor的客户端在执行命令时，会先通过计算得到key的槽信息，计算规则为：`getCRC16(key) & (16384 - 1)`，得到槽信息后，会从一个缓存map中获得槽对应的redis server信息，如果能获取到，则调到第4步
3. 向种子节点发送`slots`命令以获得整个集群的槽分布信息，然后跳转到第2步重试命令
4. 向负责该槽的server发起调用
   server处理如图：
   [![image](https://camo.githubusercontent.com/8537bb4aa988931a34eac0636a378f813479d8bb/68747470733a2f2f757365722d676f6c642d63646e2e786974752e696f2f323031382f392f31302f313635633366353465376538376535303f773d3131313626683d35323626663d706e6726733d323134323931)](https://camo.githubusercontent.com/8537bb4aa988931a34eac0636a378f813479d8bb/68747470733a2f2f757365722d676f6c642d63646e2e786974752e696f2f323031382f392f31302f313635633366353465376538376535303f773d3131313626683d35323626663d706e6726733d323134323931)
5. 客户端如果收到MOVED错误，则根据对应的地址跳转到第4步重新请求，
6. 客户段如果收到ASK错误，则根据对应的地址跳转到第4步重新请求，并在请求前带上ASKING标识。

以上步骤大致就是redis clustor下一次命令请求的过程，但忽略了一个细节，如果要查找的数据锁所在的槽正在重分配怎么办？

### Redis故障转移

#### 疑似下线与已下线

集群中每个Redis节点都会定期的向集群中的其他节点发送PING消息，如果目标节点没有在有效时间内回复PONG消息，则会被标记为疑似下线。同时将该信息发送给其他节点。当一个集群中有半数负责处理槽的主节点都将某个节点A标记为疑似下线后，那么A会被标记为已下线，将A标记为已下线的节点会将该信息发送给其他节点。

比如说有A,B,C,D,E 5个主节点。E有F、G两个从节点。
当E节点发生异常后，其他节点发送给A的PING消息将不能得到正常回复。当过了最大超时时间后，假设A,B先将E标记为疑似下线；之后C也会将E标记为疑似下线，这时C发现集群中由3个节点（A、B、C）都将E标记为疑似下线，超过集群复制槽的主节点个数的一半(>2.5)则会将E标记为已下线，并向集群广播E下线的消息。

#### 选取新的主节点

当F、G（E的从节点）收到E被标记已下线的消息后，会根据Raft算法选举出一个新的主节点，新的主节点会将E复制的所有槽指派给自己，然后向集群广播消息，通知其他节点新的主节点信息。

选举新的主节点算法与选举Sentinel头节点的[过程](http://www.farmerjohn.top/2018/08/20/redis-sentinel/#%E9%80%89%E4%B8%BE%E9%A2%86%E5%A4%B4Sentinel)很像：

1. 集群的配置纪元是一个自增计数器，它的初始值为0.
2. 当集群里的某个节点开始一次故障转移操作时，集群配置纪元的值会被增一。
3. 对于每个配置纪元，集群里每个负责处理槽的主节点都有一次投票的机会，而第一个向主节点要求投票的从节点将获得主节点的投票。
4. 档从节点发现自己正在复制的主节点进入已下线状态时，从节点会想集群广播一条CLUSTER_TYPE_FAILOVER_AUTH_REQUEST消息，要求所有接收到这条消息、并且具有投票权的主节点向这个从节点投票。
5. 如果一个主节点具有投票权（它正在负责处理槽），并且这个主节点尚未投票给其他从节点，那么主节点将向要求投票的从节点返回一条CLUSTERMSG_TYPE_FAILOVER_AUTH_ACK消息，表示这个主节点支持从节点成为新的主节点。
6. 每个参与选举的从节点都会接收CLUSTERMSG_TYPE_FAILOVER_AUTH_ACK消息，并根据自己收到了多少条这种消息来同济自己获得了多少主节点的支持。
7. 如果集群里有N个具有投票权的主节点，那么当一个从节点收集到大于等于N/2+1张支持票时，这个从节点就会当选为新的主节点。
8. 因为在每一个配置纪元里面，每个具有投票权的主节点只能投一次票，所以如果有N个主节点进行投票，那么具有大于等于N/2+1张支持票的从节点只会有一个，这确保了新的主节点只会有一个。
9. 如果在一个配置纪元里面没有从节点能收集到足够多的支持票，那么集群进入一个新的配置纪元，并再次进行选举，知道选出新的主节点为止。

# 十二、Sentinel

Sentinel（哨兵）可以监听集群中的服务器，并在主服务器进入下线状态时，自动从从服务器中选举出新的主服务器。

![image](https://camo.githubusercontent.com/b6538cd7bda024df06081f82b6b67402889f83a0/68747470733a2f2f757365722d676f6c642d63646e2e786974752e696f2f323031382f392f332f313635396434643135376663303739363f773d3130373626683d39303426663d706e6726733d313432313336)

### 与主服务器建立连接

Sentinel启动后，会与配置文件中提供的所有主服务器建立两个连接，一个是命令连接，一个是订阅连接。

命令连接用于向服务器发送命令。

订阅连接则是用于订阅服务器的`_sentinel_:hello`频道，用于获取其他Sentinel信息，下文会详细说。

### 获取主服务器信息

Sentinel会以一定频率向主服务器发送`Info`命令获取信息，包括主服务器自身的信息比如说服务器id等，以及对应的从服务器信息，包括ip和port。Sentinel会根据info命令返回的信息更新自己保存的服务器信息，并会与从服务器建立连接。

### 获取从服务器信息

与和主服务器的交互相似，Sentinel也会以一定频率通过`Info`命令获取从服务器信息，包括：从服务器ID，从服务器与主服务器的连接状态，从服务器的优先级，从服务器的复制偏移等等。

### 向服务器订阅和发布消息

用一个监视服务器集群（也就是Sentinel集群）。如何实现，如何保证监视服务器的一致性暂且先不说，我们只要记住需要用若干台Sentinel来保障高可用，那一个Sentinel是如何感知其他的Sentinel的呢？

前面说过，Sentinel在与服务器建立连接时，会建立两个连接，其中一个是订阅连接。Sentinel会定时的通过订阅连接向`_sentinel_:hello`频道频道发送消息（对Redis发布订阅功能不太了解的同学可以去去了解下），其中包括：

- Sentinel本身的信息，如ip地址、端口号、配置纪元（见下文）等
- Sentinel监视的主服务器的信息，包括ip、端口、配置纪元（见下文）等

同时，Sentinel也会订阅`_sentinel_:hello`频道的消息，也就是说Sentinel即向该频道发布消息，又从该频道订阅消息。
[![image](https://camo.githubusercontent.com/5e652ac5f12f60f93c297167c4e1884e71422808/68747470733a2f2f757365722d676f6c642d63646e2e786974752e696f2f323031382f392f332f313635396434643135376538653361643f773d37393826683d32363426663d706e6726733d3235383131)](https://camo.githubusercontent.com/5e652ac5f12f60f93c297167c4e1884e71422808/68747470733a2f2f757365722d676f6c642d63646e2e786974752e696f2f323031382f392f332f313635396434643135376538653361643f773d37393826683d32363426663d706e6726733d3235383131)

Sentinel有一个字典对象`sentinels`，保存着监视同一主服务器的其他所有Sentinel服务器，当一个Sentinel接收到来自`_sentinel_:hello`频道的消息时，会先比较发送该消息的是不是自己，如果是则忽略，否则将更新`sentinels`中的内容，并对新的Sentinel建立连接。

### 主观下线

Sentinel默认会以每秒一次的频率向所有建立连接的服务器（主服务器，从服务器，Sentinel服务器）发送`PING`命令，如果在`down-after-milliseconds`内都没有收到有效回复,Sentinel会将该服务器标记为主观下线，代表该Sentinel认为这台服务器已经下线了。需要注意的是不同Sentinel的`down-after-milliseconds`是可以不同的。

### 客观下线

为了确保服务器真的已经下线，当Sentinel将某个服务器标记为主观下线后，它会向其他的Sentinel实例发送`Sentinel is-master-down-by-addr`命令，接收到该命令的Sentinel实例会回复主服务器的状态，代表该Sentinel对该主服务器的连接情况。

Sentinel会统计发出的所有`Sentinel is-master-down-by-addr`命令的回复，并统计同意将主服务器下线的数量，如果该数量超出了某个阈值，就会将该主服务器标记为客观下线。

### 选举领头Sentinel

当Sentinel将一个主服务器标记为客观下线后，监视该服务器的各个Sentinel会通过`Raft`算法进行协商，选举出一个领头的Sentinel。

规则：

- 所有的Sentinel都有可能成为领头Sentinel的资格
- 每次选举后，无论有没有选出领头Sentinel，配置纪元都会+1
- 在某个纪元里，每个Sentinel都有为投票的机会
- 我们称要求其他人选举自己的Sentinel称为源Sentinel，将被要求投票的Sentinel称为目标Sentinel
- 每个发现主服务器被标记为客观下线且还没有被其他Sentinel要求投票的Sentinel都会要求其他Sentinel将自己设置为头
- 目标Sentinel在一个配置纪元里，一旦为某个Sentinel（也可能是它自己）投票后，对于之后收到的要求投票的命令，将拒绝
- 目标Sentinel对于要求投票的命令将回复自己选举的Sentinel的id以及当前配置纪元
- 源Sentinel在接收到要求投票的回复后：如果回复的配置纪元与自己的相同，则再检测目标Sentinel选举的头Sentinel是不是自己
- 如果某个Sentinel被半数以上的Sentinel设置成了领头Sentinel，那它将称为领头Sentinel
- 一个配置纪元只会选出一个头（因为一个头需要半数以上的支持）
- 如果在给定时间内，还没有选出头，则过段时间再次选举（配置纪元会+1）

保证Redis服务器高可用的问题

使用若干台Sentinel服务器，通过`Raft`一致性算法来保障集群的高可用，只要Sentinel服务器有一半以上的节点都正常，那集群就是可用的。

### 故障转移

领头Sentinel将会进行以下3个步骤进行故障转移：

1. 在已下线主服务器的所有从服务器中，挑选出一个作为新的主服务器


2. 将其他从服务器的主服务器设置成新的


3. 将已下线的主服务器的role改成从服务器，并将其主服务器设置成新的，当该服务器重新上线后，就会一个从服务器的角色继续工作

第一步中挑选新的主服务器的规则如下：

1. 过滤掉所有已下线的从服务器


2. 过滤掉最近5秒没有回复过Sentinel命令的从服务器


3. 过滤掉与原主服务器断开时间超过down-after-milliseconds*10的从服务器


4. 根据从服务器的优先级进行排序，选择优先级最高的那个


5. 如果有多个从服务器优先级相同，则选取复制偏移量最大的那个


6. 如果上一步的服务器还有多个，则选取id最小的那个



# 十三、分片

分片是将数据划分为多个部分的方法，可以将数据存储到多台机器里面，这种方法在解决某些问题时可以获得线性级别的性能提升。

假设有 4 个 Reids 实例 R0，R1，R2，R3，还有很多表示用户的键 user:1，user:2，... ，有不同的方式来选择一个指定的键存储在哪个实例中。

- 最简单的方式是范围分片，例如用户 id 从 0~1000 的存储到实例 R0 中，用户 id 从 1001~2000 的存储到实例 R1 中，等等。但是这样需要维护一张映射范围表，维护操作代价很高。
- 还有一种方式是哈希分片，使用 CRC32 哈希函数将键转换为一个数字，再对实例数量求模就能知道应该存储的实例。

根据执行分片的位置，可以分为三种分片方式：

- 客户端分片：客户端使用一致性哈希等算法决定键应当分布到哪个节点。
- 代理分片：将客户端请求发送到代理上，由代理转发请求到正确的节点上。
- 服务器分片：Redis Cluster。

# 保证幂等性

- 全局唯一ID：业务的操作和内容生成一个全局ID，在执行操作前先根据这个全局唯一ID是否存在，来判断这个操作是否已经执行。
- 去重表：以建一张去重表，并且把唯一标识作为唯一索引，在我们实现时，把创建支付单据和写入去去重表，放在一个事务中，如果重复创建，数据库会抛出唯一约束异常，操作就会回滚。
- 插入或更新：插入并且有唯一索引的情况，比如要关联商品品类，其中商品的ID和品类的ID可以构成唯一索引，并且在数据表中也增加了唯一索引。
- 多版本控制：适合在更新的场景中，比如我们要更新商品的名字，这时我们就可以在更新的接口中增加一个版本号，来做幂等
- 状态机控制：有状态机流转的情况下，比如就会订单的创建和付款，订单的付款肯定是在之前，这时我们可以通过在设计状态字段时，使用int类型，并且通过值类型的大小来做幂等，比如订单的创建为0，付款成功为100。付款失败为99。

# 十四、一个简单的论坛系统分析

该论坛系统功能如下：

- 可以发布文章；
- 可以对文章进行点赞；
- 在首页可以按文章的发布时间或者文章的点赞数进行排序显示。

## 文章信息

文章包括标题、作者、赞数等信息，在关系型数据库中很容易构建一张表来存储这些信息，在 Redis 中可以使用 HASH 来存储每种信息以及其对应的值的映射。

Redis 没有关系型数据库中的表这一概念来将同种类型的数据存放在一起，而是使用命名空间的方式来实现这一功能。键名的前面部分存储命名空间，后面部分的内容存储 ID，通常使用 : 来进行分隔。例如下面的 HASH 的键名为 article:92617，其中 article 为命名空间，ID 为 92617。

[![img](https://github.com/CyC2018/CS-Notes/raw/master/docs/notes/pics/7c54de21-e2ff-402e-bc42-4037de1c1592.png)](https://github.com/CyC2018/CS-Notes/blob/master/docs/notes/pics/7c54de21-e2ff-402e-bc42-4037de1c1592.png)

## 点赞功能

当有用户为一篇文章点赞时，除了要对该文章的 votes 字段进行加 1 操作，还必须记录该用户已经对该文章进行了点赞，防止用户点赞次数超过 1。可以建立文章的已投票用户集合来进行记录。

为了节约内存，规定一篇文章发布满一周之后，就不能再对它进行投票，而文章的已投票集合也会被删除，可以为文章的已投票集合设置一个一周的过期时间就能实现这个规定。

[![img](https://github.com/CyC2018/CS-Notes/raw/master/docs/notes/pics/485fdf34-ccf8-4185-97c6-17374ee719a0.png)](https://github.com/CyC2018/CS-Notes/blob/master/docs/notes/pics/485fdf34-ccf8-4185-97c6-17374ee719a0.png)

## 对文章进行排序

为了按发布时间和点赞数进行排序，可以建立一个文章发布时间的有序集合和一个文章点赞数的有序集合。（下图中的 score 就是这里所说的点赞数；下面所示的有序集合分值并不直接是时间和点赞数，而是根据时间和点赞数间接计算出来的）

[![img](https://github.com/CyC2018/CS-Notes/raw/master/docs/notes/pics/f7d170a3-e446-4a64-ac2d-cb95028f81a8.png)](https://github.com/CyC2018/CS-Notes/blob/master/docs/notes/pics/f7d170a3-e446-4a64-ac2d-cb95028f81a8.png)

























