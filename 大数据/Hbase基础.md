# Hbase基础

## Hbase概念

HBase是建立在Hadoop文件系统之上的分布式面向列的数据库。

HBase是一个数据模型，类似于谷歌的大表设计，可以提供快速随机访问海量结构化数据。它利用了Hadoop的文件系统（HDFS）提供的容错能力。提供对数据的随机实时读/写访问，是Hadoop文件系统的一部分。

可以直接或通过HBase的存储HDFS数据。使用HBase在HDFS读取消费/随机访问数据。 HBase在Hadoop的文件系统之上，并提供了读写访问。 

![HBase Flow](https://www.yiibai.com/uploads/allimg/141225/1-1412250H30NV.jpg)

### HBase 和 HDFS 

|        HDFS        |                  HBase                   |
| :----------------: | :--------------------------------------: |
|  存储大容量文件的分布式文件系统   |              建立在HDFS之上的数据库               |
|    不支持快速单独记录查找     |                在较大的表快速查找                 |
| 提供了高延迟批量处理;没有批处理概念 |        提供了数十亿条记录低延迟访问单个行记录（随机存取）         |
|       只能顺序访问       | 内部使用哈希表和提供随机接入，并且其存储索引，可将在HDFS文件中的数据进行快速查找 |

### HBase 和 RDBMS 

|            HBase            |         RDBMS          |
| :-------------------------: | :--------------------: |
| HBase无模式，它不具有固定列模式的概念;仅定义列族 | RDBMS有它的模式，描述表的整体结构的约束 |
|    它专门创建为宽表。 HBase是横向扩展     |   这些都是细而专为小表。很难形成规模    |
|       没有任何事务存在于HBase        |       RDBMS是事务性的       |
|          它反规范化的数据           |       它具有规范化的数据        |
|     它用于半结构以及结构化数据是非常好的      |       用于结构化数据非常好       |

### HBase的存储机制 

HBase是一个面向列的数据库，在表中它由行排序。表模式定义只能列族，也就是键值对。一个表有多个列族以及每一个列族可以有任意数量的列。后续列的值连续地存储在磁盘上。表中的每个单元格值都具有时间戳。

在一个HBase： 

- 表是行的集合。 
- 行是列族的集合。 
- 列族是列的集合。 
- 列是键值对的集合。

 下面给出的表中是HBase模式的一个例子。

| Rowide |      | Column Family |      |      | Column Family |      | Column Family |      |      | Column Family |      |      |
| :----: | :--: | :-----------: | :--: | :--: | :-----------: | :--: | :-----------: | :--: | :--: | :-----------: | ---- | ---- |
|        | col1 |     col2      | col3 | col1 |     col2      | col3 |     col1      | col2 | col3 |     col1      | col2 | col3 |
|   1    |      |               |      |      |               |      |               |      |      |               |      |      |
|   2    |      |               |      |      |               |      |               |      |      |               |      |      |
|   3    |      |               |      |      |               |      |               |      |      |               |      |      |

### 面向列和面向行

 面向列的数据库是存储数据表作为数据列的部分，而不是作为行数据。总之它们拥有列族。

|       行式数据库       |      列式数据库       |
| :---------------: | :--------------: |
| 它适用于联机事务处理（OLTP）  | 它适用于在线分析处理（OLAP） |
| 这样的数据库被设计为小数目的行和列 |  面向列的数据库设计的巨大表   |

下图显示了列族在面向列的数据库：

![Table](https://www.yiibai.com/uploads/allimg/141225/1-1412250I41J23.jpg)

### HBase的特点 

- HBase线性可扩展。 
- 它具有自动故障支持。
- 它提供了一致的读取和写入。 
- 它集成了Hadoop，作为源和目的地。
- 客户端方便的Java API。 
- 它提供了跨集群数据复制。

### 使用HBase

- Apache HBase曾经是随机，实时的读/写访问大数据。 它承载在集群普通硬件的顶端是非常大的表。 
- Apache HBase是此前谷歌Bigtable模拟非关系型数据库。 Bigtable对谷歌文件系统操作，同样类似Apache HBase工作在Hadoop HDFS的顶部。
- 它是用来当有需要写重的应用程序。 HBase使用于当我们需要提供快速随机访问的数据。

## Hbase架构

在HBase中，表被分割成区域，并由区域服务器提供服务。区域被列族垂直分为“Stores”。Stores被保存在HDFS文件。下面显示的是HBase的结构。

**注意：** 术语“store”是用于区域来解释存储结构。

![img](https://upload-images.jianshu.io/upload_images/6871092-5819356096368e09.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/893/format/webp)

HBase有三个主要组成部分：客户端库，主服务器和区域服务器。区域服务器可以按要求添加或删除。

**Client**

- 包含访问HBase的接口并维护cache来加快对HBase的访问

**Zookeeper**

- 保证任何时候，集群中只有一个master
- 存贮所有Region的寻址入口。
- 实时监控Region server的上线和下线信息。并实时通知Master
- 存储HBase的schema和table元数据

**Master**

- 为Region server分配region
- 负责Region server的负载均衡
- 发现失效的Region server并重新分配其上的region
- 管理用户对table的增删改操作

**RegionServer**

- Region server维护region，处理对这些region的IO请求
- Region server负责切分在运行过程中变得过大的region

**HLog(WAL log)：**

- HLog文件就是一个普通的Hadoop Sequence File，Sequence File 的Key是HLogKey对象，HLogKey中记录了写入数据的归属信息，除了table和 region名字外，同时还包括sequence number和timestamp，timestamp是” 写入时间”，sequence number的起始值为0，或者是最近一次存入文件系 统中sequence number。
- HLog SequeceFile的Value是HBase的KeyValue对象，即对应HFile中的 KeyValue

**Region**

- HBase自动把表水平划分成多个区域(region)，每个region会保存一个表里面某段连续的数据；每个表一开始只有一个region，随着数据不断插入表，region不断增大，当增大到一个阀值的时候，region就会等分会 两个新的region（裂变）；
- 当table中的行不断增多，就会有越来越多的region。这样一张完整的表 被保存在多个Regionserver上。

**Memstore 与 storefile**

- 一个region由多个store组成，一个store对应一个CF（列族）
- store包括位于内存中的memstore和位于磁盘的storefile写操作先写入memstore，当memstore中的数据达到某个阈值，hregionserver会启动flashcache进程写入storefile，每次写入形成单独的一个storefile
- 当storefile文件的数量增长到一定阈值后，系统会进行合并（minor、 major compaction），在合并过程中会进行版本合并和删除工作 （majar），形成更大的storefile。
- 当一个region所有storefile的大小和超过一定阈值后，会把当前的region分割为两个，并由hmaster分配到相应的regionserver服务器，实现负载均衡。
- 客户端检索数据，先在memstore找，找不到再找storefile
- HRegion是HBase中分布式存储和负载均衡的最小单元。最小单元就表示不同的HRegion可以分布在不同的HRegion server上。
- HRegion由一个或者多个Store组成，每个store保存一个columns family。
- 每个Strore又由一个memStore和0至多个StoreFile组成。

## Hbase优化

**1.预先分区**

默认情况下，在创建 HBase 表的时候会自动创建一个 Region 分区，当导入数据的时候，所有的 HBase 客户端都向这一个 Region 写数据，直到这个 Region 足够大了才进行切分。一种可以加快批量写入速度的方法是通过预先创建一些空的 Regions，这样当数据写入 HBase 时，会按照 Region 分区情况，在集群内做数据的负载均衡。

**2.Rowkey优化**

HBase 中 Rowkey 是按照字典序存储，因此，设计 Rowkey 时，要充分利用排序特点，将经常一起读取的数据存储到一块，将最近可能会被访问的数据放在一块。

此外，Rowkey 若是递增的生成，建议不要使用正序直接写入 Rowkey，而是采用 reverse 的方式反转Rowkey，使得 Rowkey 大致均衡分布，这样设计有个好处是能将 RegionServer 的负载均衡，否则容易产生所有新数据都在一个 RegionServer 上堆积的现象，这一点还可以结合 table 的预切分一起设计。

**3.减少列族数量**

不要在一张表里定义太多的 ColumnFamily。目前 Hbase 并不能很好的处理超过 2~3 个 ColumnFamily 的表。因为某个 ColumnFamily 在 flush 的时候，它邻近的 ColumnFamily 也会因关联效应被触发 flush，最终导致系统产生更多的 I/O。

**4.缓存策略**

创建表的时候，可以通过 HColumnDescriptor.setInMemory(true) 将表放到 RegionServer 的缓存中，保证在读取的时候被 cache 命中。

**5.设置存储生命期**

创建表的时候，可以通过 `HColumnDescriptor.setTimeToLive(int timeToLive)` 设置表中数据的存储生命期，过期数据将自动被删除。

**6.硬盘配置**

每台 RegionServer 管理 10~1000 个 Regions，每个 Region 在 1~2G，则每台 Server 最少要 10G，最大要1000*2G=2TB，考虑 3 备份，则要 6TB。方案一是用 3 块 2TB 硬盘，二是用 12 块 500G 硬盘，带宽足够时，后者能提供更大的吞吐率，更细粒度的冗余备份，更快速的单盘故障恢复。

**7.分配合适的内存给RegionServer服务**

在不影响其他服务的情况下，越大越好。例如在 HBase 的 conf 目录下的 hbase-env.sh 的最后添加 `export HBASE_REGIONSERVER_OPTS="-Xmx16000m$HBASE_REGIONSERVER_OPTS”`

其中 16000m 为分配给 RegionServer 的内存大小。

**8.写数据的备份数**

备份数与读性能成正比，与写性能成反比，且备份数影响高可用性。有两种配置方式，一种是将 hdfs-site.xml拷贝到 hbase 的 conf 目录下，然后在其中添加或修改配置项 dfs.replication 的值为要设置的备份数，这种修改对所有的 HBase 用户表都生效，另外一种方式，是改写 HBase 代码，让 HBase 支持针对列族设置备份数，在创建表时，设置列族备份数，默认为 3，此种备份数只对设置的列族生效。

**9.WAL（预写日志）**

可设置开关，表示 HBase 在写数据前用不用先写日志，默认是打开，关掉会提高性能，但是如果系统出现故障(负责插入的 RegionServer 挂掉)，数据可能会丢失。配置 WAL 在调用 JavaAPI 写入时，设置 Put 实例的WAL，调用 Put.setWriteToWAL(boolean)。

**10. 批量写**

HBase 的 Put 支持单条插入，也支持批量插入，一般来说批量写更快，节省来回的网络开销。在客户端调用JavaAPI 时，先将批量的 Put 放入一个 Put 列表，然后调用 HTable 的 Put(Put 列表) 函数来批量写。

**11. 客户端一次从服务器拉取的数量**

通过配置一次拉去的较大的数据量可以减少客户端获取数据的时间，但是它会占用客户端内存。有三个地方可进行配置：

1）在 HBase 的 conf 配置文件中进行配置 `hbase.client.scanner.caching`；

2）通过调用`HTable.setScannerCaching(intscannerCaching)` 进行配置；

3）通过调用`Scan.setCaching(intcaching)` 进行配置。三者的优先级越来越高。

**12. RegionServer的请求处理I/O线程数**

较少的 IO 线程适用于处理单次请求内存消耗较高的 Big Put 场景 (大容量单次 Put 或设置了较大 cache 的Scan，均属于 Big Put) 或 ReigonServer 的内存比较紧张的场景。

较多的 IO 线程，适用于单次请求内存消耗低，TPS 要求 (每秒事务处理量 (TransactionPerSecond)) 非常高的场景。设置该值的时候，以监控内存为主要参考。

在 hbase-site.xml 配置文件中配置项为 `hbase.regionserver.handler.count`。

**13. Region的大小设置**

配置项为 `hbase.hregion.max.filesize`，所属配置文件为 `hbase-site.xml`.，默认大小 **256M**。

在当前 ReigonServer 上单个 Reigon 的最大存储空间，单个 Region 超过该值时，这个 Region 会被自动 split成更小的 Region。小 Region 对 split 和 compaction 友好，因为拆分 Region 或 compact 小 Region 里的StoreFile 速度很快，内存占用低。缺点是 split 和 compaction 会很频繁，特别是数量较多的小 Region 不停地split, compaction，会导致集群响应时间波动很大，Region 数量太多不仅给管理上带来麻烦，甚至会引发一些Hbase 的 bug。一般 512M 以下的都算小 Region。大 Region 则不太适合经常 split 和 compaction，因为做一次 compact 和 split 会产生较长时间的停顿，对应用的读写性能冲击非常大。

此外，大 Region 意味着较大的 StoreFile，compaction 时对内存也是一个挑战。如果你的应用场景中，某个时间点的访问量较低，那么在此时做 compact 和 split，既能顺利完成 split 和 compaction，又能保证绝大多数时间平稳的读写性能。compaction 是无法避免的，split 可以从自动调整为手动。只要通过将这个参数值调大到某个很难达到的值，比如 100G，就可以间接禁用自动 split(RegionServer 不会对未到达 100G 的 Region 做split)。再配合 RegionSplitter 这个工具，在需要 split 时，手动 split。手动 split 在灵活性和稳定性上比起自动split 要高很多，而且管理成本增加不多，比较推荐 online 实时系统使用。内存方面，小 Region 在设置memstore 的大小值上比较灵活，大 Region 则过大过小都不行，过大会导致 flush 时 app 的 IO wait 增高，过小则因 StoreFile 过多影响读性能。

**14.操作系统参数**

Linux系统最大可打开文件数一般默认的参数值是1024,如果你不进行修改并发量上来的时候会出现“Too Many Open Files”的错误，导致整个HBase不可运行，你可以用ulimit -n 命令进行修改，或者修改/etc/security/limits.conf和/proc/sys/fs/file-max 的参数，具体如何修改可以去Google 关键字 “linux limits.conf ”

**15.Jvm配置**

修改 hbase-env.sh 文件中的配置参数，根据你的机器硬件和当前操作系统的JVM(32/64位)配置适当的参数

`HBASE_HEAPSIZE 4000` HBase使用的 JVM 堆的大小

`HBASE_OPTS "‐server ‐XX:+UseConcMarkSweepGC"JVM GC 选项`

`HBASE_MANAGES_ZKfalse` 是否使用Zookeeper进行分布式管理

**16. 持久化**

重启操作系统后HBase中数据全无，你可以不做任何修改的情况下，创建一张表，写一条数据进行，然后将机器重启，重启后你再进入HBase的shell中使用 list 命令查看当前所存在的表，一个都没有了。是不是很杯具？没有关系你可以在hbase/conf/hbase-default.xml中设置hbase.rootdir的值，来设置文件的保存位置指定一个文件夹，例如：<value>file:///you/hbase-data/path</value>，你建立的HBase中的表和数据就直接写到了你的磁盘上，同样你也可以指定你的分布式文件系统HDFS的路径例如:[hdfs://NAMENODE_SERVER:PORT/HBASE_ROOTDIR](https://link.jianshu.com?t=hdfs://NAMENODE_SERVER:PORT/HBASE_ROOTDIR)，这样就写到了你的分布式文件系统上了。

**17. 缓冲区大小**

`hbase.client.write.buffer`

这个参数可以设置写入数据缓冲区的大小，当客户端和服务器端传输数据，服务器为了提高系统运行性能开辟一个写的缓冲区来处理它，这个参数设置如果设置的大了，将会对系统的内存有一定的要求，直接影响系统的性能。

**18. 扫描目录表**

`hbase.master.meta.thread.rescanfrequency`

定义多长时间HMaster对系统表 root 和 meta 扫描一次，这个参数可以设置的长一些，降低系统的能耗。

**19. split/compaction时间间隔**

`hbase.regionserver.thread.splitcompactcheckfrequency`

这个参数是表示多久去RegionServer服务器运行一次split/compaction的时间间隔，当然split之前会先进行一个compact操作.这个compact操作可能是minorcompact也可能是major compact.compact后,会从所有的Store下的所有StoreFile文件最大的那个取midkey.这个midkey可能并不处于全部数据的mid中.一个row-key的下面的数据可能会跨不同的HRegion。

**20. 缓存在JVM堆中分配的百分比**

`hfile.block.cache.size`

指定HFile/StoreFile 缓存在JVM堆中分配的百分比，默认值是0.2，意思就是20%，而如果你设置成0，就表示对该选项屏蔽。

**21. ZooKeeper客户端同时访问的并发连接数**

`hbase.zookeeper.property.maxClientCnxns`

这项配置的选项就是从zookeeper中来的，表示ZooKeeper客户端同时访问的并发连接数，ZooKeeper对于HBase来说就是一个入口这个参数的值可以适当放大些。

**22. memstores占用堆的大小参数配置**

`hbase.regionserver.global.memstore.upperLimit`

在RegionServer中所有memstores占用堆的大小参数配置，默认值是0.4，表示40%，如果设置为0，就是对选项进行屏蔽。

**23. Memstore中缓存写入大小**

`hbase.hregion.memstore.flush.size`

Memstore中缓存的内容超过配置的范围后将会写到磁盘上，例如：删除操作是先写入MemStore里做个标记，指示那个value, column 或 family等下是要删除的，HBase会定期对存储文件做一个major compaction，在那时HBase会把MemStore刷入一个新的HFile存储文件中。如果在一定时间范围内没有做major compaction，而Memstore中超出的范围就写入磁盘上了。

















