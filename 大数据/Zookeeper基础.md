# Zookeeper基础

## 功能

- ZooKeeper是一个高可用的分布式数据管理与系统协调框架。基于对Paxos算法的实现，使该框架保证了分布式环境中数据的强一致性，也正是基于这样的特性，使得ZooKeeper解决很多分布式问题。
- 值得注意的是，ZK并非天生就是为这些应用场景设计的，都是后来众多开发者根据其框架的特性，利用其提供的一系列API接口（或者称为原语集），摸索出来的典型使用方法。

### 数据发布与订阅（配置中心）

发布与订阅模型，即所谓的配置中心，顾名思义就是发布者将数据发布到ZK节点上，供订阅者动态获取数据，实现配置信息的集中式管理和动态更新。例如全局的配置信息，服务式服务框架的服务地址列表等就非常适合使用。

- 应用中用到的一些配置信息放到ZK上进行集中管理。这类场景通常是这样：应用在启动的时候会主动来获取一次配置，同时，在节点上注册一个Watcher，这样一来，以后每次配置有更新的时候，都会实时通知到订阅的客户端，从而达到获取最新配置信息的目的。
- 分布式搜索服务中，索引的元信息和服务器集群机器的节点状态存放在ZK的一些指定节点，供各个客户端订阅使用。
- 分布式日志收集系统。这个系统的核心工作是收集分布在不同机器的日志。收集器通常是按照应用来分配收集任务单元，因此需要在ZK上创建一个以应用名作为path的节点P，并将这个应用的所有机器ip，以子节点的形式注册到节点P上，这样一来就能够实现机器变动的时候，能够实时通知到收集器调整任务分配。
- 系统中有些信息需要动态获取，并且还会存在人工手动去修改这个信息的发问。通常是暴露出接口，例如JMX接口，来获取一些运行时的信息。引入ZK之后，就不用自己实现一套方案了，只要将这些信息存放到指定的ZK节点上即可。

**注意：在上面提到的应用场景中，有个默认前提是：数据量很小，但是数据更新可能会比较快的场景。**

### 负载均衡

这里说的负载均衡是指软负载均衡。在分布式环境中，为了保证高可用性，通常同一个应用或同一个服务的提供方都会部署多份，达到对等服务。而消费者就须要在这些对等的服务器中选择一个来执行相关的业务逻辑，其中比较典型的是消息中间件中的生产者，消费者负载均衡。

消息中间件中发布者和订阅者的负载均衡，linkedin开源的KafkaMQ和阿里开源的metaq都是通过zookeeper来做到生产者、消费者的负载均衡。这里以metaq为例如讲下：

- 1、**生产者负载均衡**：metaq发送消息的时候，生产者在发送消息的时候必须选择一台broker上的一个分区来发送消息，因此metaq在运行过程中，会把所有broker和对应的分区信息全部注册到ZK指定节点上，默认的策略是一个依次轮询的过程，生产者在通过ZK获取分区列表之后，会按照brokerId和partition的顺序排列组织成一个有序的分区列表，发送的时候按照从头到尾循环往复的方式选择一个分区来发送消息。
- 2、**消费负载均衡**：在消费过程中，一个消费者会消费一个或多个分区中的消息，但是一个分区只会由一个消费者来消费。MetaQ的消费策略是：
  - 每个分区针对同一个group只挂载一个消费者。
  - 如果同一个group的消费者数目大于分区数目，则多出来的消费者将不参与消费。
  - 如果同一个group的消费者数目小于分区数目，则有部分消费者需要额外承担消费任务。
  - 在某个消费者故障或者重启等情况下，其他消费者会感知到这一变化（通过 zookeeper watch消费者列表），然后重新进行负载均衡，保证所有的分区都有消费者进行消费。

### 命名服务(Naming Service)

命名服务也是分布式系统中比较常见的一类场景。在分布式系统中，通过使用命名服务，客户端应用能够根据指定名字来获取资源或服务的地址，提供者等信息。被命名的实体通常可以是集群中的机器，提供的服务地址，远程对象等等——这些我们都可以统称他们为名字（Name）。其中较为常见的就是一些分布式服务框架中的服务地址列表。通过调用ZK提供的创建节点的 API，能够很容易创建一个全局唯一的path，这个path就可以作为一个名称。

阿里巴巴集团开源的分布式服务框架Dubbo中使用ZooKeeper来作为其命名服务，维护全局的服务地址列表，点击这里查看Dubbo开源项目。在Dubbo实现中：

- 1、服务提供者在启动的时候，向ZK上的指定节点/dubbo/${serviceName}/providers目录下写入自己的URL地址，这个操作就完成了服务的发布。
- 2、服务消费者启动的时候，订阅/dubbo/${serviceName}/providers目录下的提供者URL地址， 并向/dubbo/${serviceName} /consumers目录下写入自己的URL地址。

**注意，所有向ZK上注册的地址都是临时节点，这样就能够保证服务提供者和消费者能够自动感应资源的变化。**

另外，Dubbo还有针对服务粒度的监控，方法是订阅/dubbo/${serviceName}目录下所有提供者和消费者的信息。

### 分布式通知/协调

ZooKeeper中特有watcher注册与异步通知机制，能够很好的实现分布式环境下不同系统之间的通知与协调，实现对数据变更的实时处理。使用方法通常是不同系统都对ZK上同一个znode进行注册，监听znode的变化（包括znode本身内容及子节点的），其中一个系统update了znode，那么另一个系统能够收到通知，并作出相应处理。

- 另一种心跳检测机制：检测系统和被检测系统之间并不直接关联起来，而是通过zk上某个节点关联，大大减少系统耦合。
- 另一种系统调度模式：某系统有控制台和推送系统两部分组成，控制台的职责是控制推送系统进行相应的推送工作。管理人员在控制台作的一些操作，实际上是修改了ZK上某些节点的状态，而ZK就把这些变化通知给他们注册Watcher的客户端，即推送系统，于是，作出相应的推送任务。
- 另一种工作汇报模式：一些类似于任务分发系统，子任务启动后，到zk来注册一个临时节点，并且定时将自己的进度进行汇报（将进度写回这个临时节点），这样任务管理者就能够实时知道任务进度。

总之，使用zookeeper来进行分布式通知和协调能够大大降低系统之间的耦合

### 集群管理与Master选举

- 1、集群机器监控：这通常用于那种对集群中机器状态，机器在线率有较高要求的场景，能够快速对集群中机器变化作出响应。这样的场景中，往往有一个监控系统，实时检测集群机器是否存活。过去的做法通常是：监控系统通过某种手段（比如ping）定时检测每个机器，或者每个机器自己定时向监控系统汇报“我还活着”。 这种做法可行，但是存在两个比较明显的问题：

  - 集群中机器有变动的时候，牵连修改的东西比较多。
  - 有一定的延时。

  利用ZooKeeper有两个特性，就可以实时另一种集群机器存活性监控系统：

  - 客户端在节点 x 上注册一个Watcher，那么如果 x 的子节点变化了，会通知该客户端。

  - 创建EPHEMERAL类型的节点，一旦客户端和服务器的会话结束或过期，那么该节点就会消失。

    例如，监控系统在 /clusterServers 节点上注册一个Watcher，以后每动态加机器，那么就往 /clusterServers 下创建一个 EPHEMERAL类型的节点：/clusterServers/{hostname}. 这样，监控系统就能够实时知道机器的增减情况，至于后续处理就是监控系统的业务了。

- 2、Master选举则是zookeeper中最为经典的应用场景了。

  在分布式环境中，相同的业务应用分布在不同的机器上，有些业务逻辑（例如一些耗时的计算，网络I/O处理），往往只需要让整个集群中的某一台机器进行执行，其余机器可以共享这个结果，这样可以大大减少重复劳动，提高性能，于是这个master选举便是这种场景下的碰到的主要问题。

  利用ZooKeeper的强一致性，能够保证在分布式高并发情况下节点创建的全局唯一性，即：同时有多个客户端请求创建 /currentMaster 节点，最终一定只有一个客户端请求能够创建成功。利用这个特性，就能很轻易的在分布式环境中进行集群选取了。

  另外，这种场景演化一下，就是动态Master选举。这就要用到?EPHEMERAL_SEQUENTIAL类型节点的特性了。

  上文中提到，所有客户端创建请求，最终只有一个能够创建成功。在这里稍微变化下，就是允许所有请求都能够创建成功，但是得有个创建顺序，于是所有的请求最终在ZK上创建结果的一种可能情况是这样： /currentMaster/{sessionId}-1 ,?/currentMaster/{sessionId}-2 ,?/currentMaster/{sessionId}-3 ….. 每次选取序列号最小的那个机器作为Master，如果这个机器挂了，由于他创建的节点会马上小时，那么之后最小的那个机器就是Master了。

- 3、在搜索系统中，如果集群中每个机器都生成一份全量索引，不仅耗时，而且不能保证彼此之间索引数据一致。因此让集群中的Master来进行全量索引的生成，然后同步到集群中其它机器。另外，Master选举的容灾措施是，可以随时进行手动指定master，就是说应用在zk在无法获取master信息时，可以通过比如http方式，向一个地方获取master。

- 4、在Hbase中，也是使用ZooKeeper来实现动态HMaster的选举。在Hbase实现中，会在ZK上存储一些ROOT表的地址和 HMaster的地址，HRegionServer也会把自己以临时节点（Ephemeral）的方式注册到Zookeeper中，使得HMaster可以随时感知到各个HRegionServer的存活状态，同时，一旦HMaster出现问题，会重新选举出一个HMaster来运行，从而避免了 HMaster的单点问题

### 分布式锁

分布式锁，这个主要得益于ZooKeeper为我们保证了数据的强一致性。锁服务可以分为两类，一个是保持独占，另一个是控制时序。

- 所谓保持独占，就是所有试图来获取这个锁的客户端，最终只有一个可以成功获得这把锁。通常的做法是把zk上的一个znode看作是一把锁，通过 create znode的方式来实现。所有客户端都去创建 /distribute_lock 节点，最终成功创建的那个客户端也即拥有了这把锁。
- 控制时序，就是所有视图来获取这个锁的客户端，最终都是会被安排执行，只是有个全局时序了。做法和上面基本类似，只是这里 /distribute_lock 已经预先存在，客户端在它下面创建临时有序节点（这个可以通过节点的属性控制：CreateMode.EPHEMERAL_SEQUENTIAL来指定）。Zk的父节点（/distribute_lock）维持一份sequence,保证子节点创建的时序性，从而也形成了每个客户端的全局时序。

### 分布式队列

队列方面，简单地讲有两种，一种是常规的先进先出队列，另一种是要等到队列成员聚齐之后的才统一按序执行。

- 对于第一种先进先出队列，和分布式锁服务中的控制时序场景基本原理一致。
- 第二种队列其实是在FIFO队列的基础上作了一个增强。通常可以在 /queue 这个znode下预先建立一个/queue/num 节点，并且赋值为n（或者直接给/queue赋值n），表示队列大小，之后每次有队列成员加入后，就判断下是否已经到达队列大小，决定是否可以开始执行了。这种用法的典型场景是，分布式环境中，一个大任务Task A，需要在很多子任务完成（或条件就绪）情况下才能进行。这个时候，凡是其中一个子任务完成（就绪），那么就去 /taskList 下建立自己的临时时序节点（CreateMode.EPHEMERAL_SEQUENTIAL），当 /taskList 发现自己下面的子节点满足指定个数，就可以进行下一步按序进行处理了。



## 概念

### Zookeeper 

- Zookeeper 是 Google 的 Chubby一个开源的实现，是 Hadoop 的分布式协调服务
- 它包含一个简单的原语集，分布式应用程序可以基于它实现**同步服务**，配置维护和命名服务等

[![img](https://github.com/sunnyandgood/BigData/raw/master/Zookeeper/img/zookeeper1.png)](https://github.com/sunnyandgood/BigData/blob/master/Zookeeper/img/zookeeper1.png)

### 特点

- 大部分分布式应用需要一个主控、协调器或控制器来管理物理分布的子进程（如资源、任务分配等）
- 目前，大部分应用需要开发私有的协调程序，缺乏一个通用的机制
- 协调程序的反复编写浪费，且难以形成通用、伸缩性好的协调器
- ZooKeeper：提供通用的分布式锁服务，用以协调分布式应用

### 功能

- Hadoop2.0,使用Zookeeper的事件处理确保整个集群只有一个活跃的NameNode,存储配置信息等.
- HBase,使用Zookeeper的事件处理确保整个集群只有一个HMaster,察觉HRegionServer联机和宕机,存储访问控制列表等.

### 特性

- Zookeeper是简单的
- Zookeeper是富有表现力的
- Zookeeper具有高可用性
- Zookeeper采用松耦合交互方式
- Zookeeper是一个资源库

## 结构

### 数据模型

- 层次化的目录结构，命名符合常规文件系统规范
- 每个节点在zookeeper中叫做znode,并且其有一个唯一的路径标识
- 节点Znode可以包含数据和子节点，但是EPHEMERAL类型的节点不能有子节点
- Znode中的数据可以有多个版本，比如某一个路径下存有多个数据版本，那么查询这个路径下的数据就需要带上版本
- 客户端应用可以在节点上设置监视器
- 节点不支持部分读写，而是一次性完整读写

### 节点

- Znode有两种类型，短暂的（ephemeral）和持久的（persistent）
- Znode的类型在创建时确定并且之后不能再修改
- 短暂znode的客户端会话结束时，zookeeper会将该短暂znode删除，短暂znode不可以有子节点
- 持久znode不依赖于客户端会话，只有当客户端明确要删除该持久znode时才会被删除
- Znode有四种形式的目录节点，PERSISTENT、PERSISTENT_SEQUENTIAL、EPHEMERAL、EPHEMERAL_SEQUENTIAL

### 角色

- 领导者（leader），负责进行投票的发起和决议，更新系统状态
- 学习者（learner），包括跟随者（follower）和观察者（observer），follower用于接受客户端请求并想客户端返回结果，在选主过程中参与投票
- 观察者（Observer），可以接受客户端连接，将写请求转发给leader，但observer不参加投票过程，只同步leader的状态，observer的目的是为了扩展系统，提高读取速度
- 客户端（client），请求发起方

[![img](https://github.com/sunnyandgood/BigData/raw/master/Zookeeper/img/zookeeper2.png)](https://github.com/sunnyandgood/BigData/blob/master/Zookeeper/img/zookeeper2.png)

### 顺序号

- 创建znode时设置顺序标识，znode名称后会附加一个值
- 顺序号是一个单调递增的计数器，由父节点维护
- 在分布式系统中，顺序号可以被用于为所有的事件进行全局排序，这样客户端可以通过顺序号推断事件的顺序

### 读写机制

- Zookeeper是一个由多个server组成的集群
- 一个leader，多个follower
- 每个server保存一份数据副本
- 全局数据一致
- 分布式读写
- 更新请求转发，由leader实施

### 保证

- 更新请求顺序进行，来自同一个client的更新请求按其发送顺序依次执行
- 数据更新原子性，一次数据更新要么成功，要么失败
- 全局唯一数据视图，client无论连接到哪个server，数据视图都是一致的
- 实时性，在一定事件范围内，client能读到最新数据

### API接口

- String create(String path, byte[] data, List acl, CreateMode createMode)
  - 创建一个给定的目录节点 path, 并给它设置数据，CreateMode 标识有四种形式的目录节点，分别是 PERSISTENT：持久化目录节点，这个目录节点存储的数据不会丢失；PERSISTENT_SEQUENTIAL：顺序自动编号的目录节点，这种目录节点会根据当前已近存在的节点数自动加 1，然后返回给客户端已经成功创建的目录节点名；EPHEMERAL：临时目录节点，一旦创建这个节点的客户端与服务器端口也就是 session 超时，这种节点会被自动删除；EPHEMERAL_SEQUENTIAL：临时自动编号节点
- Stat exists(String path, boolean watch)
  - 判断某个 path 是否存在，并设置是否监控这个目录节点，这里的 watcher 是在创建 ZooKeeper 实例时指定的 watcher，exists方法还有一个重载方法，可以指定特定的 watcher
- Stat exists(String path, Watcher watcher)
  - 重载方法，这里给某个目录节点设置特定的 watcher，Watcher 在 ZooKeeper 是一个核心功能，Watcher 可以监控目录节点的数据变化以及子目录的变化，一旦这些状态发生变化，服务器就会通知所有设置在这个目录节点上的 Watcher，从而每个客户端都很快知道它所关注的目录节点的状态发生变化，而做出相应的反应
- void delete(String path, int version)
  - 删除 path 对应的目录节点，version 为 -1 可以匹配任何版本，也就删除了这个目录节点所有数据
- List getChildren(String path, boolean watch)
  - 获取指定 path 下的所有子目录节点，同样 getChildren方法也有一个重载方法可以设置特定的 watcher 监控子节点的状态
- Stat setData(String path, byte[] data, int version)
  - 给 path 设置数据，可以指定这个数据的版本号，如果 version 为 -1 怎可以匹配任何版本
- byte[] getData(String path, boolean watch, Stat stat)
  - 获取这个 path 对应的目录节点存储的数据，数据的版本等信息可以通过 stat 来指定，同时还可以设置是否监控这个目录节点数据的状态
- void addAuthInfo(String scheme, byte[] auth)
  - 客户端将自己的授权信息提交给服务器，服务器将根据这个授权信息验证客户端的访问权限。
- Stat setACL(String path, List acl, int version)
  - 给某个目录节点重新设置访问权限，需要注意的是 Zookeeper 中的目录节点权限不具有传递性，父目录节点的权限不能传递给子目录节点。目录节点 ACL 由两部分组成：perms 和 id。
  - Perms 有 ALL、READ、WRITE、CREATE、DELETE、ADMIN 几种 , id 标识了访问目录节点的身份列表，默认情况下有以下两种：
    - ANYONE_ID_UNSAFE = new Id("world", "anyone") 和
    - AUTH_IDS = new Id("auth", "") 分别表示任何人都可以访问和创建者拥有访问权限。
- List getACL(String path, Stat stat)
  - 获取某个目录节点的访问权限列表

### 观察（watcher）

- Watcher 在 ZooKeeper 是一个核心功能，Watcher 可以监控目录节点的数据变化以及子目录的变化，一旦这些状态发生变化，服务器就会通知所有设置在这个目录节点上的 Watcher，从而每个客户端都很快知道它所关注的目录节点的状态发生变化，而做出相应的反应
- 可以设置观察的操作：exists,getChildren,getData
- 可以触发观察的操作：create,delete,setData

写操作与zookeeper内部事件的对应关系

[![img](https://github.com/sunnyandgood/BigData/raw/master/Zookeeper/img/%E5%86%99%E6%93%8D%E4%BD%9C%E4%B8%8Ezookeeper%E5%86%85%E9%83%A8%E4%BA%8B%E4%BB%B6%E7%9A%84%E5%AF%B9%E5%BA%94%E5%85%B3%E7%B3%BB.png)](https://github.com/sunnyandgood/BigData/blob/master/Zookeeper/img/%E5%86%99%E6%93%8D%E4%BD%9C%E4%B8%8Ezookeeper%E5%86%85%E9%83%A8%E4%BA%8B%E4%BB%B6%E7%9A%84%E5%AF%B9%E5%BA%94%E5%85%B3%E7%B3%BB.png)

内部事件与watcher的对应关系

[![img](https://github.com/sunnyandgood/BigData/raw/master/Zookeeper/img/zookeeper%E5%86%85%E9%83%A8%E4%BA%8B%E4%BB%B6%E4%B8%8Ewatcher%E7%9A%84%E5%AF%B9%E5%BA%94%E5%85%B3%E7%B3%BB.png)](https://github.com/sunnyandgood/BigData/blob/master/Zookeeper/img/zookeeper%E5%86%85%E9%83%A8%E4%BA%8B%E4%BB%B6%E4%B8%8Ewatcher%E7%9A%84%E5%AF%B9%E5%BA%94%E5%85%B3%E7%B3%BB.png)

写操作与watcher的对应关系

[![img](https://github.com/sunnyandgood/BigData/raw/master/Zookeeper/img/%E5%86%99%E6%93%8D%E4%BD%9C%E4%B8%8Ewatcher%E7%9A%84%E5%AF%B9%E5%BA%94%E5%85%B3%E7%B3%BB.png)](https://github.com/sunnyandgood/BigData/blob/master/Zookeeper/img/%E5%86%99%E6%93%8D%E4%BD%9C%E4%B8%8Ewatcher%E7%9A%84%E5%AF%B9%E5%BA%94%E5%85%B3%E7%B3%BB.png)

每个znode被创建时都会带有一个ACL列表，用于决定谁可以对它执行何种操作

[![img](https://github.com/sunnyandgood/BigData/raw/master/Zookeeper/img/znode%E8%A2%AB%E5%88%9B%E5%BB%BA%E6%97%B6ACL%E5%88%97%E8%A1%A8.png)](https://github.com/sunnyandgood/BigData/blob/master/Zookeeper/img/znode%E8%A2%AB%E5%88%9B%E5%BB%BA%E6%97%B6ACL%E5%88%97%E8%A1%A8.png)

ACL 访问控制列表（Access Control List，ACL）

- 身份验证模式有三种：
- digest:用户名，密码
- host:通过客户端的主机名来识别客户端
- ip： 通过客户端的ip来识别客户端
- new ACL(Perms.READ,new Id("host","example.com"));这个ACL对应的身份验证模式是host，符合该模式的身份是example.com，权限的组合是：READ

Znode的节点状态

[![img](https://github.com/sunnyandgood/BigData/raw/master/Zookeeper/img/Znode%E7%9A%84%E8%8A%82%E7%82%B9%E7%8A%B6%E6%80%81.png)](https://github.com/sunnyandgood/BigData/blob/master/Zookeeper/img/Znode%E7%9A%84%E8%8A%82%E7%82%B9%E7%8A%B6%E6%80%81.png)

### 工作原理

- Zookeeper的核心是原子广播，这个机制保证了各个server之间的同步。实现这个机制的协议叫做Zab协议。Zab协议有两种模式，它们分别是恢复模式和广播模式。当服务启动或者在领导者崩溃后，Zab就进入了恢复模式，当领导者被选举出来，且大多数server的完成了和leader的状态同步以后，恢复模式就结束了。状态同步保证了leader和server具有相同的系统状态。
- 一旦leader已经和多数的follower进行了状态同步后，他就可以开始广播消息了，即进入广播状态。这时候当一个server加入zookeeper服务中，它会在恢复模式下启动，发现leader，并和leader进行状态同步。待到同步结束，它也参与消息广播。Zookeeper服务一直维持在Broadcast状态，直到leader崩溃了或者leader失去了大部分的followers支持。
- 广播模式需要保证proposal被按顺序处理，因此zk采用了递增的事务id号(zxid)来保证。所有的提议(proposal)都在被提出的时候加上了zxid。实现中zxid是一个64为的数字，它高32位是epoch用来标识leader关系是否改变，每次一个leader被选出来，它都会有一个新的epoch。低32位是个递增计数。
- 当leader崩溃或者leader失去大多数的follower，这时候zk进入恢复模式，恢复模式需要重新选举出一个新的leader，让所有的server都恢复到一个正确的状态。

### Leader选举

- 每个Server启动以后都询问其它的Server它要投票给谁。
- 对于其他server的询问，server每次根据自己的状态都回复自己推荐的leader的id和上一次处理事务的zxid（系统启动时每个server都会推荐自己）
- 收到所有Server回复以后，就计算出zxid最大的哪个Server，并将这个Server相关信息设置成下一次要投票的Server。
- 计算这过程中获得票数最多的的sever为获胜者，如果获胜者的票数超过半数，则改server被选为leader。否则，继续这个过程，直到leader被选举出来。
- leader就会开始等待server连接
- Follower连接leader，将最大的zxid发送给leader
- Leader根据follower的zxid确定同步点
- 完成同步后通知follower 已经成为uptodate状态
- Follower收到uptodate消息后，又可以重新接受client的请求进行服务了

[![img](https://github.com/sunnyandgood/BigData/raw/master/Zookeeper/img/Leader%E9%80%89%E4%B8%BE1.png)](https://github.com/sunnyandgood/BigData/blob/master/Zookeeper/img/Leader%E9%80%89%E4%B8%BE1.png)

[![img](https://github.com/sunnyandgood/BigData/raw/master/Zookeeper/img/Leader%E9%80%89%E4%B8%BE2.png)](https://github.com/sunnyandgood/BigData/blob/master/Zookeeper/img/Leader%E9%80%89%E4%B8%BE2.png)

### 示例代码

```
  // 创建一个与服务器的连接
   ZooKeeper zk = new ZooKeeper("localhost:" + CLIENT_PORT, 
          ClientBase.CONNECTION_TIMEOUT, new Watcher() { 
              // 监控所有被触发的事件
              public void process(WatchedEvent event) { 
                  System.out.println("已经触发了" + event.getType() + "事件！"); 
              } 
          }); 
   // 创建一个目录节点
   zk.create("/testRootPath", "testRootData".getBytes(), Ids.OPEN_ACL_UNSAFE,
     CreateMode.PERSISTENT); 
   // 创建一个子目录节点
   zk.create("/testRootPath/testChildPathOne", "testChildDataOne".getBytes(),
     Ids.OPEN_ACL_UNSAFE,CreateMode.PERSISTENT); 
   System.out.println(new String(zk.getData("/testRootPath",false,null))); 
   // 取出子目录节点列表
   System.out.println(zk.getChildren("/testRootPath",true)); 
   // 修改子目录节点数据
   zk.setData("/testRootPath/testChildPathOne","modifyChildDataOne".getBytes(),-1); 
   System.out.println("目录节点状态：["+zk.exists("/testRootPath",true)+"]"); 
   // 创建另外一个子目录节点
   zk.create("/testRootPath/testChildPathTwo", "testChildDataTwo".getBytes(), 
     Ids.OPEN_ACL_UNSAFE,CreateMode.PERSISTENT); 
   System.out.println(new String(zk.getData("/testRootPath/testChildPathTwo",true,null))); 
   // 删除子目录节点
   zk.delete("/testRootPath/testChildPathTwo",-1); 
   zk.delete("/testRootPath/testChildPathOne",-1); 
   // 删除父目录节点
   zk.delete("/testRootPath",-1); 
   // 关闭连接
   zk.close();

```

- 输出的结果如下：

  ```
  已经触发了 None 事件！
  testRootData	[testChildPathOne] 
  目录节点状态：[5,5,1281804532336,1281804532336,0,1,0,0,12,1,6] 
  已经触发了 NodeChildrenChanged 事件！
  testChildDataTwo
  已经触发了 NodeDeleted 事件！
  已经触发了 NodeDeleted 事件！

  ```

### 应用场景

统一命名服务

- 分布式应用中，通常需要有一套完整的命名规则，既能够产生唯一的名称又便于人识别和记住，通常情况下用树形的名称结构是一个理想的选择，树形的名称结构是一个有层次的目录结构，既对人友好又不会重复。
- Name Service 是 Zookeeper 内置的功能，只要调用 Zookeeper 的 API 就能实现。如调用 create 接口就可以很容易创建一个目录节点。

配置管理

- 配置的管理在分布式应用环境中很常见，例如同一个应用系统需要多台 PC Server 运行，但是它们运行的应用系统的某些配置项是相同的，如果要修改这些相同的配置项，那么就必须同时修改每台运行这个应用系统的 PC Server，这样非常麻烦而且容易出错。
- 将配置信息保存在 Zookeeper 的某个目录节点中，然后将所有需要修改的应用机器监控配置信息的状态，一旦配置信息发生变化，每台应用机器就会收到 Zookeeper 的通知，然后从 Zookeeper 获取新的配置信息应用到系统中。

[![img](https://github.com/sunnyandgood/BigData/raw/master/Zookeeper/img/%E5%BA%94%E7%94%A8%E5%9C%BA%E6%99%AF2%EF%BC%8D%E9%85%8D%E7%BD%AE%E7%AE%A1%E7%90%86.png)](https://github.com/sunnyandgood/BigData/blob/master/Zookeeper/img/%E5%BA%94%E7%94%A8%E5%9C%BA%E6%99%AF2%EF%BC%8D%E9%85%8D%E7%BD%AE%E7%AE%A1%E7%90%86.png)

集群管理

- Zookeeper 能够很容易的实现集群管理的功能，如有多台 Server 组成一个服务集群，那么必须要一个“总管”知道当前集群中每台机器的服务状态，一旦有机器不能提供服务，集群中其它集群必须知道，从而做出调整重新分配服务策略。同样当增加集群的服务能力时，就会增加一台或多台 Server，同样也必须让“总管”知道。
- Zookeeper 不仅能够维护当前的集群中机器的服务状态，而且能够选出一个“总管”，让这个总管来管理集群，这就是 Zookeeper 的另一个功能 Leader Election。
- 它们的实现方式都是在 Zookeeper 上创建一个 EPHEMERAL 类型的目录节点，然后每个 Server 在它们创建目录节点的父目录节点上调用 getChildren(String path, boolean watch) 方法并设置 watch 为 true，由于是 EPHEMERAL 目录节点，当创建它的 Server 死去，这个目录节点也随之被删除，所以 Children 将会变化，这时 getChildren上的 Watch 将会被调用，所以其它 Server 就知道已经有某台 Server 死去了。新增 Server 也是同样的原理。
- Zookeeper 如何实现 Leader Election，也就是选出一个 Master Server。和前面的一样每台 Server 创建一个 EPHEMERAL 目录节点，不同的是它还是一个 SEQUENTIAL 目录节点，所以它是个 EPHEMERAL_SEQUENTIAL 目录节点。之所以它是 EPHEMERAL_SEQUENTIAL 目录节点，是因为我们可以给每台 Server 编号，我们可以选择当前是最小编号的 Server 为 Master，假如这个最小编号的 Server 死去，由于是 EPHEMERAL 节点，死去的 Server 对应的节点也被删除，所以当前的节点列表中又出现一个最小编号的节点，我们就选择这个节点为当前 Master。这样就实现了动态选择 Master，避免了传统意义上单 Master 容易出现单点故障的问题。

[![img](https://github.com/sunnyandgood/BigData/raw/master/Zookeeper/img/%E5%BA%94%E7%94%A8%E5%9C%BA%E6%99%AF3%EF%BC%8D%E9%9B%86%E7%BE%A4%E7%AE%A1%E7%90%86.png)](https://github.com/sunnyandgood/BigData/blob/master/Zookeeper/img/%E5%BA%94%E7%94%A8%E5%9C%BA%E6%99%AF3%EF%BC%8D%E9%9B%86%E7%BE%A4%E7%AE%A1%E7%90%86.png)

```
  zk.create("/testRootPath/testChildPath1","1".getBytes(), 
                                Ids.OPEN_ACL_UNSAFE,CreateMode.EPHEMERAL_SEQUENTIAL);
  zk.create(“/testRootPath/testChildPath2”,“2”.getBytes(), 
                                Ids.OPEN_ACL_UNSAFE,CreateMode.EPHEMERAL_SEQUENTIAL);
  zk.create("/testRootPath/testChildPath3","3".getBytes(), 
                                Ids.OPEN_ACL_UNSAFE,CreateMode.EPHEMERAL_SEQUENTIAL);
  zk.create("/testRootPath/testChildPath4","4".getBytes(), 
                                Ids.OPEN_ACL_UNSAFE,CreateMode.EPHEMERAL_SEQUENTIAL);
  System.out.println(zk.getChildren("/testRootPath", false));

```

打印结果：

```
    [testChildPath10000000000, testChildPath20000000001, testChildPath40000000003, testChildPath30000000002]

```

- 规定编号最小的为master,所以当我们对SERVERS节点做监控的时候，得到服务器列表，只要所有集群机器逻辑认为最小编号节点为master，那么master就被选出，而这

- 个master宕机的时候，相应的znode会消失，然后新的服务器列表就被推送到客户端，然后每个节点逻辑认为最小编号节点为master，这样就做到动态master选举。

- Leader Election 关键代码

  ```
       void findLeader() throws InterruptedException { 
               byte[] leader = null; 
               try { 
                   leader = zk.getData(root + "/leader", true, null); 
               } catch (Exception e) { 
                   logger.error(e); 
               } 
               if (leader != null) { 
                   following(); 
               } else { 
                   String newLeader = null; 
                   try { 
                       byte[] localhost = InetAddress.getLocalHost().getAddress(); 
                       newLeader = zk.create(root + "/leader", localhost, 
                       ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.EPHEMERAL); 
                   } catch (Exception e) { 
                       logger.error(e); 
                   } 
                   if (newLeader != null) { 
                       leading(); 
                   } else { 
                       mutex.wait(); 
                   } 
               } 
           }

  ```

共享锁

- 共享锁在同一个进程中很容易实现，但是在跨进程或者在不同 Server 之间就不好实现了。Zookeeper 却很容易实现这个功能，实现方式也是需要获得锁的 Server 创建一个 EPHEMERAL_SEQUENTIAL 目录节点，然后调用 getChildren方法获取当前的目录节点列表中最小的目录节点是不是就是自己创建的目录节点，如果正是自己创建的，那么它就获得了这个锁，如果不是那么它就调用 exists(String path, boolean watch) 方法并监控 Zookeeper 上目录节点列表的变化，一直到自己创建的节点是列表中最小编号的目录节点，从而获得锁，释放锁很简单，只要删除前面它自己所创建的目录节点就行了。

[![img](https://github.com/sunnyandgood/BigData/raw/master/Zookeeper/img/%E5%BA%94%E7%94%A8%E5%9C%BA%E6%99%AF4%EF%BC%8D%E5%85%B1%E4%BA%AB%E9%94%81.png)](https://github.com/sunnyandgood/BigData/blob/master/Zookeeper/img/%E5%BA%94%E7%94%A8%E5%9C%BA%E6%99%AF4%EF%BC%8D%E5%85%B1%E4%BA%AB%E9%94%81.png)

- 同步锁的关键代码

  ```
  void getLock() throws KeeperException, InterruptedException{ 
          List<String> list = zk.getChildren(root, false); 
          String[] nodes = list.toArray(new String[list.size()]); 
          Arrays.sort(nodes); 
          if(myZnode.equals(root+"/"+nodes[0])){ 
              doAction(); 
          } 
          else{ 
              waitForLock(nodes[0]); 
          } 
      } 
      void waitForLock(String lower) throws InterruptedException, KeeperException {
          Stat stat = zk.exists(root + "/" + lower,true); 
          if(stat != null){ 
              mutex.wait(); 
          } 
          else{ 
              getLock(); 
          } 
      }

  ```

队列管理

- Zookeeper 可以处理两种类型的队列：
  - 当一个队列的成员都聚齐时，这个队列才可用，否则一直等待所有成员到达，这种是同步队列；
  - 队列按照 FIFO 方式进行入队和出队操作，例如实现生产者和消费者模型。
- 创建一个父目录 /synchronizing，每个成员都监控目录 /synchronizing/start 是否存在，然后每个成员都加入这个队列（创建 /synchronizing/member_i 的临时目录节点），然后每个成员获取 / synchronizing 目录的所有目录节点，判断 i 的值是否已经是成员的个数，如果小于成员个数等待 /synchronizing/start 的出现，如果已经相等就创建 /synchronizing/start。

[![img](https://github.com/sunnyandgood/BigData/raw/master/Zookeeper/img/%E5%BA%94%E7%94%A8%E5%9C%BA%E6%99%AF5%EF%BC%8D%E9%98%9F%E5%88%97%E7%AE%A1%E7%90%86.png)](https://github.com/sunnyandgood/BigData/blob/master/Zookeeper/img/%E5%BA%94%E7%94%A8%E5%9C%BA%E6%99%AF5%EF%BC%8D%E9%98%9F%E5%88%97%E7%AE%A1%E7%90%86.png)

- 同步队列的关键代码

  - 同步队列

    ```
    void addQueue() throws KeeperException, InterruptedException{ 
       zk.exists(root + "/start",true); 
       zk.create(root + "/" + name, new byte[0], Ids.OPEN_ACL_UNSAFE, 
       CreateMode.EPHEMERAL_SEQUENTIAL); 
       synchronized (mutex) { 
           List<String> list = zk.getChildren(root, false); 
           if (list.size() < size) { 
               mutex.wait(); 
           } else { 
               zk.create(root + "/start", new byte[0], Ids.OPEN_ACL_UNSAFE,
                CreateMode.PERSISTENT); 
           } 
       } 
    }

    ```

  当队列没满是进入 wait()，然后会一直等待 Watch 的通知，Watch 的代码如下：

  ```
       public void process(WatchedEvent event) { 
          if(event.getPath().equals(root + "/start") &&
           event.getType() == Event.EventType.NodeCreated){ 
              System.out.println("得到通知"); 
              super.process(event); 
              doAction(); 
          } 
       }

  ```

- FIFO 队列用 Zookeeper 实现思路：

  在特定的目录下创建 SEQUENTIAL 类型的子目录 /queue_i，这样就能保证所有成员加入队列时都是有编号的，出队列时通过 getChildren( ) 方法可以返回当前所有的队列中的元素，然后消费其中最小的一个，这样就能保证 FIFO。

  - 生产者代码

    ```
     boolean produce(int i) throws KeeperException, InterruptedException{ 
             ByteBuffer b = ByteBuffer.allocate(4); 
             byte[] value; 
             b.putInt(i); 
             value = b.array(); 
             zk.create(root + "/element", value, ZooDefs.Ids.OPEN_ACL_UNSAFE, 
                         CreateMode.PERSISTENT_SEQUENTIAL); 
             return true; 
     }

    ```

  - 消费者代码

    ```
    int consume() throws KeeperException, InterruptedException{ 
       int retvalue = -1; 
       Stat stat = null; 
       while (true) { 
           synchronized (mutex) { 
               List<String> list = zk.getChildren(root, true); 
               if (list.size() == 0) { 
                   mutex.wait(); 
               } else { 
                   Integer min = new Integer(list.get(0).substring(7)); 
                   for(String s : list){ 
                       Integer tempValue = new Integer(s.substring(7)); 
                       if(tempValue < min) min = tempValue; 
                   } 
                   byte[] b = zk.getData(root + "/element" + min,false, stat); 
                   zk.delete(root + "/element" + min, 0); 
                   ByteBuffer buffer = ByteBuffer.wrap(b); 
                   retvalue = buffer.getInt(); 
                   return retvalue; 
               } 
           } 
       } 
    }

    ```











