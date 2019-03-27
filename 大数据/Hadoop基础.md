# Hadoop基础

## HDFS架构

### 特点

- 适用于分布式存储和处理。

- Hadoop提供了与HDFS交互的命令界面。

- namenode和datanode的内置服务器可以帮助用户轻松检查集群的状态。

- 流式访问文件系统数据。

- HDFS提供文件权限和身份验证。

- Web界面

- 安全模式

- 负载均衡

  ​

以下是Hadoop文件系统的体系结构。

### 架构

![img](https://upload-images.jianshu.io/upload_images/6871092-8377a620c560a2b7.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/547/format/webp)

HDFS遵循主 - 从架构，一个HDFS集群包含一个单独的Master节点和多个Slave节点服务器。单独的Master节点是指逻辑上的Master组件。可以包含两台物理主机，即两台Master服务器。

**Namenode**
namenode是包含GNU / Linux操作系统和namenode软件的商品硬件。具有namenode的系统充当主服务器，它执行以下任务：

- 管理文件系统命名空间。
- 规范客户对文件的访问。
- 它还执行文件系统操作，如重命名，关闭和打开文件和目录。

**Datanode**
数据库是具有GNU / Linux操作系统和数据库软件的商品硬件。对于集群中的每个节点（商品硬件/系统），将有一个数据库。这些节点管理其系统的数据存储。

- Datanodes根据客户端请求对文件系统执行读写操作。
- 他们还根据namenode的说明执行块创建，删除和复制等操作。

**Block**
通常，用户数据存储在HDFS的文件中。文件系统中的文件将被分成一个或多个片段和/或存储在各个数据节点中。这些文件段被称为块。换句话说，HDFS可以读取或写入的最小数据量称为块。默认块大小为64MB，但可根据需要在HDFS配置中更改。

### NameNode和Secondary NameNode通信模型

NameNode将对文件系统的改动追加保存到本地文件系统上的一个日志文件edits。当一个NameNode启动时，它首先从一个映像文件（fsimage）中读取HDFS的状态，接着执行日志文件中的编辑操作。然后将新的HDFS状态写入fsimage中，并使用一个空的edits文件开始正常操作。因为NameNode只有在启动阶段才合并fsimage和edits，久而久之日志文件可能会变得非常庞大，特别是对于大型的集群。日志文件太大的另一个副作用是下一次NameNode启动会花很长时间，NameNode和Secondary NameNode之间的通信示意图如图3-2所示。

[![img](http://s4.51cto.com/wyfs02/M01/6B/A3/wKiom1UzUmrBCnBBAABzkI4Lyp4757.jpg)](http://s4.51cto.com/wyfs02/M01/6B/A3/wKiom1UzUmrBCnBBAABzkI4Lyp4757.jpg)

如图3-2所示，NameNode和Secondary NameNode间数据的通信使用的是HTTP协议，Secondary NameNode定期合并fsimage和edits日志，将edits日志文件大小控制在一个限度下。因为内存需求和NameNode在一个数量级上，所以通常Secondary NameNode和NameNode运行在不同的机器上。Secondary NameNode通过bin/start-dfs.sh在conf/masters中指定的节点上启动。

### 文件存取机制

**HDFS读文件数据流**

首先客户端调用FileSystem的open( )函数打开文件，DistributedFileSystem用RPC调用元数据节点，得到文件的数据块信息。对于每一个数据块，元数据节点返回保存数据块的数据节点的地址。DistributedFileSystem返回FSDataInputStream给客户端，用来读取数据。客户端调用stream的read()函数开始读取数据。DFSInputStream连接保存此文件第一个数据块的最近的数据节点。Data从数据节点读到客户端，当此数据块读取完毕时，DFSInputStream关闭和此数据节点的连接，然后连接此文件下一个数据块的最近的数据节点。当客户端读取完数据的时候，调用FSDataInputStream的close函数。客户端读取HDFS中的文件访问数据流的整个过程如图3-3所示。

读取文件的数据流步骤如下：

1. 调用FileSystem的open()打开文件，见序号1：open。
2. DistributedFileSystem使用RPC调用NameNode，得到文件的数据块元数据信息，并返回FSDataInputStream给客户端，见序号2：get block locations。
3. HDFS客户端调用stream的read()函数开始读取数据，见序号3：read。
4. 调用FSDataInputStream直接从DataNode获取文件数据块，见序号4、5：read。
5. 读完文件时，调用FSDataInputStream的close函数，见序号6：close。

![img](http://s5.51cto.com/wyfs02/M01/6B/A3/wKiom1UzUs6hnSSWAABVDaSV-3w938.jpg)

**HDFS写文件数据流**

写文件数据需要同时写到多个数据块文件中，这就相对比较复杂了。HDFS的写机制可以通过图3-4进行简单描述。

![img](http://s9.51cto.com/wyfs02/M02/6B/9E/wKioL1UzVCzxX6O5AAAnJ7NDdsc598.jpg)

数据流从客户端开始，流经一系列的数据节点，到达最后一个DataNode。图3-4中的所有DataNode只需要写一次硬盘，DataNode1和DataNode2会从socket上接收到数据，将它直接写到下个节点的socket上。

如果出错，抛出的异常会导致receiveBlock关闭相关的输出流，并终止传输。同时，数据校验出错还会上报到NameNode上。

最后一个DataNode由于没有后续节点，PacketResponder的ackQueue每收到一项，表明对应的数据块已经处理完毕，那么就可以发送成功应答。

从客户端开始，直到在HDFS上完成写一个文件的整体数据流程图

![img](http://s5.51cto.com/wyfs02/M02/6B/A3/wKiom1UzUtmQok8UAABgqyum7_U781.jpg)

### HDFS核心设计

**Block大小**

Block的大小默认的参数是64MB，远远大于一般文件系统的Blocksize。

选择较大的Block尺寸有几个优点：

- 减少了客户端和NameNode通信的需求，因为只需要一次和NameNode节点的通信就可以获取Block的位置信息，之后就可以对同一个Block进行多次的读写操作。
- 客户端能够对一个块进行多次操作，这样就可以通过与Block服务器保持较长时间的TCP连接来减少网络负载。
- 减少了NameNode节点需要保存的元数据的数量，从而很容易把所有元数据全部放在内存中。

缺点：小文件包含较少的Block，甚至只有一个Block，当有许多的客户端对同一个小文件进行多次访问时，存储这些Block的DataNode服务器就会变成访问热点。

当一个可执行文件保存在HDFS上时或许是一个Block的文件，当这个可执行文件在数百台机器上同时启动时，数百个客户端的并发请求访问会导致系统局部过载，解决这个问题可以通过自定义更大的HDFS复制因子数来保存可执行文件。

**数据复制**

HDFS被设计成在一个大集群中可以跨机器可靠地存储海量的文件。它将每个文件存储成Block序列，除了最后一个Block，所有的Block都是同样的大小。

文件的所有Block为了容错都会被冗余复制存储。每个文件的Block大小和Replication因子都是可配置的。Replication因子在文件创建的时候会默认读取客户端的HDFS配置，然后创建，以后也可以改变。HDFS中的文件是write-one，并且严格要求在任何时候只有一个writer。HDFS数据冗余复制示意图如3-6图所示。

![img](http://s5.51cto.com/wyfs02/M01/6B/9F/wKioL1UzVI_DpsWGAABj-QCa23Y150.jpg)

part-1的复制因子Replication值是2，块的ID列表包括1和3，可以看到块1和块3分别被冗余备份了两份数据块；

part-2的复制因子Replication值是3，块的ID列表包括2、4、5，可以看到块2、4、5分别被冗余复制了三份。

文件所有块的复制会全权由NameNode进行管理，NameNode周期性地从集群中的每个DataNode接收心跳包和一个Blockreport。心跳包的接收表示该DataNode节点正常工作，而Blockreport包括了该DataNode上所有的Block组成的列表。

**数据副本存放策略**

HDFS采用一种称为机架感知（rack-aware）的策略来改进数据的可靠性、可用性和网络带宽的利用率。

默认的副本系数是3，这适用于大多数情况。副本存放策略是将第一个副本存放在本地机架的节点上，将第二个副本放在同一机架的另一个节点上，将第三个副本放在不同机架的节点上。

为了降低整体的带宽消耗和读取延时，HDFS会尽量让读取程序读取离它最近的副本。如果读取程序的同一个机架上有一个副本，那么就读取该副本；如果一个HDFS集群跨越多个数据中心，那么客户端也将首先读取本地数据中心的副本。

### 数据组织

**数据块**

一个典型的Block大小是64MB，因此文件总是按照64MB切分成Chunk，每个Chunk存储于不同的DataNode服务器中。

**Staging**

HDFS客户端会将文件数据缓存到本地的一个临时文件中，当这个临时文件累积的数据超过一个Block的大小（默认为64MB)，客户端才会联系NameNode。NameNode将文件名插入文件系统的层次结构中，并且分配一个数据块给它，然后返回DataNode的标识符和目标数据块给客户端。

**流水线式的复制**

当客户端将数据写入复制因子为 `r = 3` 的 `HDFS` 文件时，`NameNode` 使用 `replication target choosing algorithm` 检索 `DataNode` 列表。此列表包含将承载该块副本的 `DataNode`。

然后客户端向第一个 `DataNode` 写入，第一个 `DataNode` 开始分批接收数据，将每个部分写入其本地存储，并将该部分传输到列表中的第二个 `DataNode`。第二个 `DataNode` 又开始接收数据块的每个部分，将该部分写入其存储，然后将该部分刷新到第三个 `DataNode`。最后，第三个 `DataNode` 将数据写入其本地存储。

可见，`DataNode` 是从流水线中的前一个接收数据，同时将数据转发到流水线中的下一个，数据是从一个 `DataNode` 流水线到下一个 `DataNode`。

### 空间回收

**文件的删除和恢复**

在文件被用户删除和HDFS空闲空间的增加之间会有一个等待时间延迟。并没有立刻从HDFS中被删除，HDFS将这个文件重命名，并转移到trash目录下，可以被迅速恢复。当超过设定的时间后，将该文件从namespace中删除。

**减小副本系数**

在减小某个文件的副本系数后，NameNode会选择要删除的过剩的副本。下次心跳检测就将该信息传递给DataNode，DataNode就会移除相应的Block并释放空间。

### 通信协议

所有的HDFS通信协议都是构建在TCP/IP协议上的。NameNode不会主动发起RPC，而是响应来自客户端和DataNode的RPC请求。

### 安全模式

当系统处于安全状态时，不接受任何对名称空间的修改，同时也不会对数据块进行复制或删除。

### 机架感知

大型Hadoop集群是以机架的形式来组织的，同一个机架上不同节点间的网络状况比不同机架之间的更为理想。另外，NameNode设法将数据块副本保存在不同的机架上，以提高容错性。

### 健壮性

实现在失败情况下数据存储的可靠性。

失败情况：

- NameNode failures
- DataNode failures
- 网络分割（network partitions)

健壮性设计

1. 数据错误

   网络切割可能会导致部分DataNode与NameNode失去联系。NameNode可通过心跳包的缺失检测到这一情况，并将这些DataNode标记为dead，不会向它们发送IO请求，寄存在dead DataNode上的任何数据将不再有效。DataNode的“死亡”可能引起一些Block的副本数目低于指定值，NameNode不断地跟踪需要复制的Block，并在需要的情况下启动复制。

2. 集群均衡

   如果某个DataNode上的空闲空间低于特定的临界点，那么就会启动一个计划——自动将数据从一个DataNode搬移到空闲的DataNode。当对某个文件的请求突然增加，也可能会启动一个计划——创建该文件新的副本，并将此副本分布到集群中以满足应用的要求。

3. 数据完整性

   当某个客户端创建一个新的HDFS文件，会计算这个文件的每个Block的校验和，并作为一个单独的隐藏文件保存这些校验和在同一个HDFS namespace下。当客户端检索文件内容，它会确认从DataNode获取的数据跟相应的校验和文件中的校验和是否匹配，如果不匹配，客户端可以选择从其他DataNode获取该Block的副本。

4. 元数据磁盘错误

   FsImage和Editlog是HDFS的核心数据结构。这些文件如果损坏了，整个HDFS实例都将失效。因而，NameNode可以配置成支持维护多个FsImage和Editlog的副本。任何对FsImage或Editlog的修改，都将同步到它们的副本上。

5. 快照

   快照支持某个时间的数据副本，当HDFS数据损坏的时候，可以恢复到过去一个已知的正确时间点。

### 负载均衡

**负载均衡遵循原则**

- 在执行数据重分布的过程中，必须保证数据不能出现丢失，不能改变数据的备份数，不能改变每一个机架中所具备的Block数量。
- 系统管理员可以通过一条命令启动数据重分布程序或停止数据重分布程序。
- Block在移动的过程中，不能占用过多的资源，如网络带宽。
- 数据重分布程序在执行的过程中，不能影响NameNode的正常工作。

![img](http://s9.51cto.com/wyfs02/M02/6B/9F/wKioL1UzWNeT2FZeAABOTmqzZ1Y994.jpg)

**负载均衡的处理步骤**

1. 负载均衡服务Rebalancing Server从NameNode中获取所有的DataNode情况，具体包括每一个DataNode磁盘使用情况，见图3-7中的流程1.get datanode report。
2. Rebalancing Server计算哪些机器需要将数据移动，哪些机器可以接受移动的数据，以及从NameNode中获取需要移动数据的分布情况，见图3-7中的流程2.get partial blockmap。
3. Rebalancing Server计算出来可以将哪一台机器的Block移动到另一台机器中去，见图3-7中流程3.copy a block。
4. 需要移动Block的机器将数据移动到目标机器上，同时删除自己机器上的Block数据，见图3-7中的流程4、5、6。
5. Rebalancing Server获取本次数据移动的执行结果，并继续执行这个过程，一直到没有数据可以移动或HDFS集群已经达到平衡的标准为止，见图3-7中的流程7。

### 升级和回滚机制

Hadoop内部实现了一套升级机制。如果失败了，那就用rollback进行回滚；如果过了一段时间，系统运行正常，那就可以通过finalize正式提交这次升级。

状态转移示意图

![img](http://s4.51cto.com/wyfs02/M00/6B/A3/wKiom1UzWBWhBxz-AABXCJWH8Jk346.jpg)

## MapReduce计算框架

基于 java 的并行分布式计算框架，`MapReduce` 可以利用数据的位置，在存储的位置附近处理数据，以最大限度地减少通信开销。

### 框架

三个操作组成：

1. **Map**：每个工作节点将 `map` 函数应用于本地数据，并将输出写入临时存储。主节点确保仅处理冗余输入数据的一个副本。
2. **Shuffle**：工作节点根据输出键（由 `map` 函数生成）重新分配数据，对数据映射排序、分组、拷贝，目的是属于一个键的所有数据都位于同一个工作节点上。
3. **Reduce**：工作节点现在并行处理每个键的每组输出数据。

MapReduce 流程图：

![img](https://user-gold-cdn.xitu.io/2018/10/4/1663d77230e1bbd5?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

`MapReduce` 允许分布式运行 `Map` 操作，只要每个 `Map` 操作独立于其他 `Map` 操作就可以并行执行。

#### Mapper

一个 `Map` 函数就是对一些独立元素组成的概念上的列表的每一个元素进行指定的操作，所以每个元素都是被独立操作的，而原始列表没有被更改，`Map` 操作是可以高度并行的

#### Reducer

 `Reduce` 是对一个列表的元素进行适当的合并。`Reduce` 函数并行应用于每个组，从而在同一个数据域中生成一组值

#### Partitioner

`Map` 阶段有一个分割成组的操作，这个划分数据的过程就是 `Partition`，而负责分区的 java 类就是 `Partitioner`。

`Partitioner` 组件可以让 `Map` 对 `Key` 进行分区，从而将不同分区的 `Key` 交由不同的 `Reduce`处理。具有多个分割总是有好处的，因为与处理整个输入所花费的时间相比，处理分割所花费的时间很短。当分割较小时，可以更好的处理负载平衡

![img](https://user-gold-cdn.xitu.io/2018/10/4/1663d77232a288e4?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

### 优势

- 并行处理：由于数据由多台机器而不是单台机器并行处理，因此处理数据所需的时间会减少很多
- 数据位置：将计算移动到 `MapReduce` 框架中的数据，而不是将数据移动到计算部分。数据分布在多个节点中，其中每个节点处理驻留在其上的数据部分。

以下优势

- 将处理单元移动到数据所在位置可以降低网络成本；
- 由于所有节点并行处理其部分数据，因此处理时间缩短；
- 每个节点都会获取要处理的数据的一部分，因此节点不会出现负担过重的可能性。

限制

1. 不能进行流式计算和实时计算，只能计算离线数据；
2. 中间结果存储在磁盘上，加大了磁盘的 I/O 负载，且读取速度比较慢；
3. 开发麻烦，例如 `wordcount` 功能就需要很多的设置和代码量，而 `Spark` 将会非常简单。







### 作业运行

![img](https://upload-images.jianshu.io/upload_images/10865890-92201b71f0b6c45a.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/689/format/webp)

#### 五个核心的实体

- 客户端，提交 MapReduce 作业
- YARN 资源管理器（YARN resource manager），负责协调集群上计算机资源的分配
- YARN 节点管理器（YARN node manager），负责启动和监视集群中机器上的计算容器（container）
- MRAppMaster（MapReduce application master），负责协调 MapReduce 作业的任务（tasks）的运行。MRAppMaster 和 MapReduce 任务运行在容器中，该容器由资源管理器进行调度（schedule）[此处理解为划分、分配更为合适] 且由节点管理器进行管理
- 分布式文件系统（通常是 HDFS），用来在其他实体间共享作业文件

#### 作业提交（Job Submission）

作业的提交过程由 JobSubmitter 实现，执行如下事情：

- 向资源管理器请求一个 application ID，该 ID 被用作 MapReduce 作业的 ID`(step 2)`
- 检查作业指定的输出（output）目录。例如，如果该输出目录没有被指定或者已经存在，作业不会被提交且一个错误被抛出给 MapReduce 程序
- 为作业计算输入分片（input splits）。如果分片不能被计算（可能因为输入路径（input paths）不存在），该作业不会被提交且一个错误被抛出给 MapReduce 程序
- 拷贝作业运行必备的资源，包括作业 JAR 文件，配置文件以及计算的输入分片，到一个以作业 ID 命名的共享文件系统目录中`(step 3)`。作业 JAR 文件以一个高副本因子（a high replication factor）进行拷贝（由 mapreduce.client.submit.file.replication 属性控制，默认值为 10），所以在作业任务运行时，在集群中有很多的作业 JAR 副本供节点管理器来访问
- 通过在资源管理器上调用 submitApplication 来提交作业`(step 4)`

### 输入格式

1. 输入分片与记录 
2. 文件输入 
3. 文本输入 
4. 二进制输入 
5. 多文件输入 
6. 数据库格式输入

![img](https://img-blog.csdn.net/20150610173108636)

### 输出格式

- 文本输出
- 二进制输出
- 多文件输出
- 数据库格式输出

![img](https://img-blog.csdn.net/20150610215643330)

### 特性

- 提交作业到队列：队列是作业的集合，允许系统添加特定的功能
- Counters：全局计数器
- DistributedCache：将具体应用相关的，Size大的、只读的文件有效地分布放置。可以缓存应用程序所需的文件
- Profiling：Profiling是一个工具，它使用内置的java profiler工具进行分析获得（2-3个）map和reduce样例运行分析报告。
- 数据压缩：工具可以为map输出的中间数据和作业结果输出数据（例如reduce的输出）提供支持。
- Skipping Bad Records：在处理map输入的时候，允许跳过一定数量的记录集。有时map任务一遇到某些特定的输入集合就会崩溃，这主要是map函数的一些BUG导致的。

### Shuffle 和排序（Shuffle and Sort）

MapReduce 确保每个 reducer 的输入都是按键排过序的。由系统来执行排序，将 map 的输出转换成 reducer 的输入，这个过程称之为 shuffle。在这部分，我们探究一下 shuffle 如何工作，这个基本的理解对于你优化 MapReduce 程序是很有帮助的。shuffle 是一个持续不断改进和提升的代码库（codebase），所以下面的描述隐藏了很多细节。从多方面看，shuffle 是 MapReduce 的心脏和奇迹发生的地方

#### Map 端（The Map Side）

当 map 函数开始产生输出，它不是简单的写入到磁盘。这个过程更加复杂，利用缓冲写入到内存，为了效率执行一些预排序（presorting）。如下图所示

![img](https://upload-images.jianshu.io/upload_images/10865890-e84296c46f97c56a.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp)

每个 map 任务有一个环形内存缓冲用于输出的写入。这个缓冲默认为 100 MB（可以通过 mapreduce.task.io.sort.mb 属性进行调整）。当缓冲中的内容达到一个确定的阈值大小（mapreduce.map.sort.spill.percent，默认值为 0.80 或者 80%），后台有一个线程将内容溢出（spill）到磁盘。当溢出操作发生时，Map 输出继续写入到缓冲中，如果此时缓冲被填满（fill up），map 将会阻塞直到溢出操作完成。溢出按照轮询的方式写入到通过 mapreduce.cluster.local.dir 属性指定的目录（在一个作业特定的子目录下）

在内容被写入到磁盘之前，首先线程根据数据最终被发送的 reducers 来将数据进行分区（partition）。在每个分区中，后台线程在内存中按键进行排序，如果有一个 combiner 函数，它会在排过序的输出上运行。运行 combiner 函数得到更为紧凑的 map 输出，所以更少的数据写入到本地磁盘和传输到 reducer

每次内存缓冲达到溢出阈值，一个新的溢出文件被创建，因此 map 任务在写完最后一条记录之后，可能会有几个溢出文件。在任务完成之前，这些溢出文件被合并成单个被分区过的且排序过的文件。配置属性 mapreduce.task.io.sort.factor 控制最大一次能够合并多少流（stream），默认是 10

如果至少有三个溢出文件（由 mapreduce.map.combine.minspills 属性设置），combiner 在输出文件写入之前再次运行。combiners 可能会被重复在输入上运行而不会影响最终的结果。如果只有 1 或 2 个溢出文件，潜在的调用 combiner 的开销来减少 map 的输出大小是不值得的，所以不会为 map 的输出在调用 combiner。[此处如果超过三个，将会被合并为一个文件，少于三个就不会进行合并]

通常压缩 map 输出写入到磁盘是一个好注意，因为带来更快的写入以及节省磁盘空间，减少传输到 reducer 的数据量。默认，输出是不压缩的，可用很容易通过设置 mapreduce.map.output.compress 为 true 来启用压缩。压缩的类库通过 mapreduce.map.output.compress.codec 来制定。

Reducer 很容易通过 HTTP 得到输出文件的部分（partitions）。通过设置 mapreduce.shuffle.max.threads 属性为每个节点管理器（而不是每个 map 任务）来控制用于服务文件部分的工作线程数。默认值 0 是设置最大线程数为机器上处理器数目的两倍

#### Reduce 端（The Reduce Side）

map 输出文件坐落在运行 map 任务的机器本地磁盘上（注意虽然 map 输出总是写到本地磁盘，但是 reduce 输出可能不会），这些文件被运行 reduce 任务的机器所需要。再者，reduce 任务需要集群上若干个 map 任务的 map 输出作为其特殊的分区（partition）文件。map 任务可能在不同时间点完成，所以每个 map 任务完成，reduce 任务就开始拷贝它们的输出。这就是 reduce 任务的拷贝阶段（copy phase）。reduce 任务拥有少量的拷贝线程以至于能够并行抓取 map 输出。默认是 5 个线程，这个值可以通过 mapreduce.reduce.shuffle.parallelcopies 属性设置

**注意**
Reducers 怎么知道从哪台机器上面抓取 map 输出呢？

当 map 任务成功完成，它们通过心跳机制通知 application master。因此，对于一个特定的作业，application master 知道 map 输出和主机之间的映射。在 reducer 中有一个线程周期性想 master 所询问有的 map 输出主机，直到所有的主机都得到。

主机不会在首个 reducer 拿到 map 输出就将其从磁盘上删除掉，因为 reducer 后续可能失败。在作业已经完成之后，主机会等待 application master 的通知来从磁盘上删除这些输出文件。

Map 输出如果足够小（缓冲的大小由），会被拷贝到 reduce 任务的 JVM 内存中（这个缓冲的大小由 mapreduce.reduce.shuffle.input.buffer.percent 控制，指定了对堆内存使用比率）。否则，这些输出将会拷贝到磁盘。当内存中的缓冲达到一个阈值大小（由mapreduce.reduce.shuffle.merge.percent）或者达到 map 输出阈值大小（mapreduce.reduce.merge.inmen.threshold）,则合并溢出到磁盘中。如果一个 combiner 被指定，在合并的过程中它可能被运行以至来减少写入磁盘的数据量

随着磁盘上的副本持续累积，一个后台线程用来将它们合并成一个更大的，排过序的文件。这会为后面的合并节省一些时间。注意，为了合并，任何在 map 压缩的输出必须在内存中进行解压缩

当所有的 map 输出都被拷贝，reduce 任务进入到排序阶段（应该被称为合并阶段，因为排序已经在 map 端执行了），合并 map 输出，维持其排序顺序。这个是循环执行的。例如，如果有 50 个 map 输出且合并因子（merge factor）是 10（有 mapreduce.task.io.sort.factor 属性控制），因此有 5 个循环。每个循环合并 10 个文件到 1 个文件中，所以最后有 5 个中间文件。

并不是有最后一个循环来合并这 5 个文件到 1 个单独排序过的文件，在最后阶段（reduce phase）合并通过直接向 reduce 函数提供数据，从而节省了一次磁盘的往返。最后的合并可能来自内存和磁盘的混合（a mixture of in-memory and on-disk segments）

![img](https://upload-images.jianshu.io/upload_images/10865890-9980a28aec868437.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/340/format/webp)

在 reduce 阶段，在已排序的输出中，为每个键调用 reduce 函数。该阶段的输出直接写入到输出文件系统中，典型的是 HDFS。在是 HDFS 的情况下，因为节点管理器也运行在数据节点上，所以第一个块副本被写入到本地磁盘上

## YARN

###  初识

Apache Hadoop YARN 是开源 Hadoop 分布式处理框架中的资源管理和作业调度技术。作为 Apache Hadoop 的核心组件之一，YARN 负责将系统资源分配给在 Hadoop 集群中运行的各种应用程序，并调度要在不同集群节点上执行的任务。

YARN 的基本思想是将资源管理和作业调度/监视的功能分解为单独的 daemon(守护进程)，其拥有一个全局 ResourceManager(RM) 和每个应用程序的 ApplicationMaster(AM)。应用程序可以是单个作业，也可以是作业的 DAG。

ResourceManager和 NodeManager构成了数据计算框架。 ResourceManager 是在系统中的所有应用程序之间仲裁资源的最终权限。NodeManager 是每台机器框架代理，负责 Containers，监视其资源使用情况（CPU，内存，磁盘，网络）并将其报告给 ResourceManager。

每个应用程序 ApplicationMaster 实际上是一个框架特定的库，其任务是协调来自 ResourceManager 的资源，并与 NodeManager 一起执行和监视任务。



在 YARN 体系结构中，ResourceManager 作为守护程序运行，作为架构中的全局的 master 角色，通常在专用计算机上运行，它在各种竞争应用程序之间仲裁可用的群集资源。ResourceManager 跟踪群集上可用的活动节点和资源的数量，并协调用户提交的应用程序应获取哪些资源以及事件。ResourceManager 是具有此信息的单个进程，因此它可以以共享，安全和多租户的方式进行调度决策（例如，根据应用程序优先级，队列容量，ACL，数据位置等）。

当用户提交应用程序时，将启动名为 ApplicationMaster 的轻量级进程实例，以协调应用程序中所有任务的执行。这包括监视任务，重新启动失败的任务，推测性地运行慢速任务以及计算应用程序计数器的总值。ApplicationMaster 和属于其应用程序的任务在 NodeManagers 控制的资源容器中运行。

NodeManager 有许多动态创建的资源容器。容器的大小取决于它包含的资源量，例如内存、CPU、磁盘和网络IO。目前，仅支持内存和CPU。节点上的容器数是配置参数和用于守护程序及OS的资源之外的节点资源总量（例如总CPU和总内存）的乘积。

ApplicationMaster 可以在容器内运行任何类型的任务。例如，MapReduce ApplicationMaster 请求容器启动 map 或 reduce 任务，而 Giraph ApplicationMaster 请求容器运行 Giraph 任务。您还可以实现运行特定任务的自定义 ApplicationMaster

在 YARN 中，MapReduce 简单地降级为分布式应用程序的角色（但仍然是非常流行且有用的），现在称为MRv2。

此外，YARN 通过 ReservationSystem 支持资源预留的概念，ReservationSystem 允许用户通过配置文件来指定资源的时间和时间约束（例如，截止日期）的，并保留资源以确保重要作业的可预测执行。ReservationSystem 可跟踪资源超时，执行预留的准入控制，并动态指示基础调度程序确保预留已满。

###  基本服务组件

YARN 总体上是 master/slave 结构，在整个资源管理框架中，ResourceManager 为 master，NodeManager 是 slave。

YARN的基本组成结构，YARN 主要由 ResourceManager、NodeManager、ApplicationMaster 和 Container 等几个组件构成。

- ResourceManager是Master上一个独立运行的进程，负责集群统一的资源管理、调度、分配等等；
- NodeManager是Slave上一个独立运行的进程，负责上报节点的状态；
- ApplicationMaster相当于这个Application的监护人和管理者，负责监控、管理这个Application的所有Attempt在cluster中各个节点上的具体运行，同时负责向Yarn ResourceManager申请资源、返还资源等；
- Container是yarn中分配资源的一个单位，包涵内存、CPU等等资源，YARN以Container为单位分配资源；

ResourceManager 负责对各个 NadeManager 上资源进行统一管理和调度。当用户提交一个应用程序时，需要提供一个用以跟踪和管理这个程序的 ApplicationMaster，它负责向 ResourceManager 申请资源，并要求 NodeManger 启动可以占用一定资源的任务。由于不同的 ApplicationMaster 被分布到不同的节点上，因此它们之间不会相互影响。

Client 向 ResourceManager 提交的每一个应用程序都必须有一个 ApplicationMaster，它经过 ResourceManager 分配资源后，运行于某一个 Slave 节点的 Container 中，具体做事情的 Task，同样也运行与某一个 Slave 节点的 Container 中。

#### 2.1 **ResourceManager**

RM是一个全局的资源管理器，集群只有一个，负责整个系统的资源管理和分配，包括处理客户端请求、启动/监控 ApplicationMaster、监控 NodeManager、资源的分配与调度。它主要由两个组件构成：调度器（Scheduler）和应用程序管理器（Applications Manager，ASM）。

（1） 调度器

调度器根据容量、队列等限制条件（如每个队列分配一定的资源，最多执行一定数量的作业等），将系统中的资源分配给各个正在运行的应用程序。需要注意的是，该调度器是一个“纯调度器”，它从事任何与具体应用程序相关的工作，比如不负责监控或者跟踪应用的执行状态等，也不负责重新启动因应用执行失败或者硬件故障而产生的失败任务，这些均交由应用程序相关的ApplicationMaster完成。

调度器仅根据各个应用程序的资源需求进行资源分配，而资源分配单位用一个抽象概念“资源容器”（Resource Container，简称Container）表示，Container是一个动态资源分配单位，它将内存、CPU、磁盘、网络等资源封装在一起，从而限定每个任务使用的资源量。

（2） 应用程序管理器

应用程序管理器主要负责管理整个系统中所有应用程序，接收job的提交请求，为应用分配第一个 Container 来运行 ApplicationMaster，包括应用程序提交、与调度器协商资源以启动 ApplicationMaster、监控 ApplicationMaster 运行状态并在失败时重新启动它等。

#### 2.2 **ApplicationMaster**

管理 YARN 内运行的一个应用程序的每个实例。关于 job 或应用的管理都是由 ApplicationMaster 进程负责的，Yarn 允许我们以为自己的应用开发 ApplicationMaster。

功能：

- 数据切分；
- 为应用程序申请资源并进一步分配给内部任务（TASK）；
- 任务监控与容错；
- 负责协调来自ResourceManager的资源，并通过NodeManager监视容易的执行和资源使用情况。

可以说，ApplicationMaster 与 ResourceManager 之间的通信是整个 Yarn 应用从提交到运行的最核心部分，是 Yarn 对整个集群进行动态资源管理的根本步骤，Yarn 的动态性，就是来源于多个Application 的 ApplicationMaster 动态地和 ResourceManager 进行沟通，不断地申请、释放、再申请、再释放资源的过程。

#### 2.3 **NodeManager**

NodeManager 整个集群有多个，负责每个节点上的资源和使用。

NodeManager 是一个 slave 服务：它负责接收 ResourceManager 的资源分配请求，分配具体的 Container 给应用。同时，它还负责监控并报告 Container 使用信息给 ResourceManager。通过和ResourceManager 配合，NodeManager 负责整个 Hadoop 集群中的资源分配工作。

功能：NodeManager 本节点上的资源使用情况和各个 Container 的运行状态（cpu和内存等资源）

- 接收及处理来自 ResourceManager 的命令请求，分配 Container 给应用的某个任务；
- 定时地向RM汇报以确保整个集群平稳运行，RM 通过收集每个 NodeManager 的报告信息来追踪整个集群健康状态的，而 NodeManager 负责监控自身的健康状态；
- 处理来自 ApplicationMaster 的请求；
- 管理着所在节点每个 Container 的生命周期；
- 管理每个节点上的日志；
- 执行 Yarn 上面应用的一些额外的服务，比如 MapReduce 的 shuffle 过程；

当一个节点启动时，它会向 ResourceManager 进行注册并告知 ResourceManager 自己有多少资源可用。在运行期，通过 NodeManager 和 ResourceManager 协同工作，这些信息会不断被更新并保障整个集群发挥出最佳状态。

NodeManager 只负责管理自身的 Container，它并不知道运行在它上面应用的信息。负责管理应用信息的组件是 ApplicationMaster

#### 2.4 **Container**

Container 是 YARN 中的资源抽象，它封装了某个节点上的多维度资源，如内存、CPU、磁盘、网络等，当 AM 向 RM 申请资源时，RM 为 AM 返回的资源便是用 Container 表示的。YARN 会为每个任务分配一个 Container，且该任务只能使用该 Container 中描述的资源。

Container 和集群节点的关系是：一个节点会运行多个 Container，但一个 Container 不会跨节点。任何一个 job 或 application 必须运行在一个或多个 Container 中，在 Yarn 框架中，ResourceManager 只负责告诉 ApplicationMaster 哪些 Containers 可以用，ApplicationMaster 还需要去找 NodeManager 请求分配具体的 Container。

需要注意的是，Container 是一个动态资源划分单位，是根据应用程序的需求动态生成的。目前为止，YARN 仅支持 CPU 和内存两种资源，且使用了轻量级资源隔离机制 Cgroups 进行资源隔离。

功能：

- 对task环境的抽象；
- 描述一系列信息；
- 任务运行资源的集合（cpu、内存、io等）；
- 任务运行环境

###  应用提交过程

Application在Yarn中的执行过程，整个执行过程可以总结为三步：

1. 应用程序提交
2. 启动应用的ApplicationMaster实例
3. ApplicationMaster 实例管理应用程序的执行

具体提交过程为：

1. 客户端程序向 ResourceManager 提交应用并请求一个 ApplicationMaster 实例；
2. ResourceManager 找到一个可以运行一个 Container 的 NodeManager，并在这个 Container 中启动 ApplicationMaster 实例；
3. ApplicationMaster 向 ResourceManager 进行注册，注册之后客户端就可以查询 ResourceManager 获得自己 ApplicationMaster 的详细信息，以后就可以和自己的 ApplicationMaster 直接交互了（这个时候，客户端主动和 ApplicationMaster 交流，应用先向 ApplicationMaster 发送一个满足自己需求的资源请求）；
4. 在平常的操作过程中，ApplicationMaster 根据 `resource-request协议` 向 ResourceManager 发送 `resource-request请求`；
5. 当 Container 被成功分配后，ApplicationMaster 通过向 NodeManager 发送 `container-launch-specification信息` 来启动Container，`container-launch-specification信息`包含了能够让Container 和 ApplicationMaster 交流所需要的资料；
6. 应用程序的代码以 task 形式在启动的 Container 中运行，并把运行的进度、状态等信息通过 `application-specific协议` 发送给ApplicationMaster；
7. 在应用程序运行期间，提交应用的客户端主动和 ApplicationMaster 交流获得应用的运行状态、进度更新等信息，交流协议也是 `application-specific协议`；
8. 一旦应用程序执行完成并且所有相关工作也已经完成，ApplicationMaster 向 ResourceManager 取消注册然后关闭，用到所有的 Container 也归还给系统。

精简版的：

- 步骤1：用户将应用程序提交到 ResourceManager 上；
- 步骤2：ResourceManager 为应用程序 ApplicationMaster 申请资源，并与某个 NodeManager 通信启动第一个 Container，以启动ApplicationMaster；
- 步骤3：ApplicationMaster 与 ResourceManager 注册进行通信，为内部要执行的任务申请资源，一旦得到资源后，将于 NodeManager 通信，以启动对应的 Task；
- 步骤4：所有任务运行完成后，ApplicationMaster 向 ResourceManager 注销，整个应用程序运行结束。

### Resource Request 及 Container

Yarn的设计目标就是允许我们的各种应用以共享、安全、多租户的形式使用整个集群。并且，为了保证集群资源调度和数据访问的高效性，Yarn还必须能够感知整个集群拓扑结构。

为了实现这些目标，ResourceManager的调度器Scheduler为应用程序的资源请求定义了一些灵活的协议，通过它就可以对运行在集群中的各个应用做更好的调度，因此，这就诞生了**Resource Request**和**Container**。

一个应用先向ApplicationMaster发送一个满足自己需求的资源请求，然后ApplicationMaster把这个资源请求以resource-request的形式发送给ResourceManager的Scheduler，Scheduler再在这个原始的resource-request中返回分配到的资源描述Container。

每个ResourceRequest可看做一个可序列化Java对象，包含的字段信息如下：

```
<resource-name, priority, resource-requirement, number-of-containers>

- resource-name：资源名称，现阶段指的是资源所在的host和rack，后期可能还会支持虚拟机或者更复杂的网络结构
- priority：资源的优先级
- resource-requirement：资源的具体需求，现阶段指内存和cpu需求的数量
- number-of-containers：满足需求的Container的集合

```

ApplicationMaster在得到这些Containers后，还需要与分配Container所在机器上的NodeManager交互来启动Container并运行相关任务。当然Container的分配是需要认证的，以防止ApplicationMaster自己去请求集群资源。









