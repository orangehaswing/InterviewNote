# Zookeeper基础


## 概念

### Zookeeper 

​	它包含一个简单的原语集，分布式应用程序可以基于它实现**同步服务**，配置维护和命名服务等

[![img](https://github.com/sunnyandgood/BigData/raw/master/Zookeeper/img/zookeeper1.png)](https://github.com/sunnyandgood/BigData/blob/master/Zookeeper/img/zookeeper1.png)

### 特点

- 大部分分布式应用需要一个主控、协调器或控制器来管理物理分布的子进程（如资源、任务分配等）
- 目前，大部分应用需要开发私有的协调程序，缺乏一个通用的机制
- 协调程序的反复编写浪费，且难以形成通用、伸缩性好的协调器
- ZooKeeper：提供通用的分布式锁服务，用以协调分布式应用

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
- 学习者（learner），包括跟随者（follower）和观察者（observer），follower用于接受客户端请求并返回客户端结果，在选主过程中参与投票
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

- **生产者负载均衡**：metaq发送消息的时候，生产者在发送消息的时候必须选择一台broker上的一个分区来发送消息，因此metaq在运行过程中，会把所有broker和对应的分区信息全部注册到ZK指定节点上，默认的策略是一个依次轮询的过程，生产者在通过ZK获取分区列表之后，会按照brokerId和partition的顺序排列组织成一个有序的分区列表，发送的时候按照从头到尾循环往复的方式选择一个分区来发送消息。
- **消费负载均衡**：在消费过程中，一个消费者会消费一个或多个分区中的消息，但是一个分区只会由一个消费者来消费。MetaQ的消费策略是：
  - 每个分区针对同一个group只挂载一个消费者。
  - 如果同一个group的消费者数目大于分区数目，则多出来的消费者将不参与消费。
  - 如果同一个group的消费者数目小于分区数目，则有部分消费者需要额外承担消费任务。
  - 在某个消费者故障或者重启等情况下，其他消费者会感知到这一变化（通过 zookeeper watch消费者列表），然后重新进行负载均衡，保证所有的分区都有消费者进行消费。

### 命名服务(Naming Service)

命名服务也是分布式系统中比较常见的一类场景。在分布式系统中，通过使用命名服务，客户端应用能够根据指定名字来获取资源或服务的地址，提供者等信息。被命名的实体通常可以是集群中的机器，提供的服务地址，远程对象等等——这些我们都可以统称他们为名字（Name）。其中较为常见的就是一些分布式服务框架中的服务地址列表。通过调用ZK提供的创建节点的 API，能够很容易创建一个全局唯一的path，这个path就可以作为一个名称。

阿里巴巴集团开源的分布式服务框架Dubbo中使用ZooKeeper来作为其命名服务，维护全局的服务地址列表，点击这里查看Dubbo开源项目。在Dubbo实现中：

- 服务提供者在启动的时候，向ZK上的指定节点/dubbo/${serviceName}/providers目录下写入自己的URL地址，这个操作就完成了服务的发布。
- 服务消费者启动的时候，订阅/dubbo/${serviceName}/providers目录下的提供者URL地址， 并向/dubbo/${serviceName} /consumers目录下写入自己的URL地址。

**注意，所有向ZK上注册的地址都是临时节点，这样就能够保证服务提供者和消费者能够自动感应资源的变化。**

另外，Dubbo还有针对服务粒度的监控，方法是订阅/dubbo/${serviceName}目录下所有提供者和消费者的信息。

### 分布式通知/协调

ZooKeeper中特有watcher注册与异步通知机制，能够很好的实现分布式环境下不同系统之间的通知与协调，实现对数据变更的实时处理。使用方法通常是不同系统都对ZK上同一个znode进行注册，监听znode的变化（包括znode本身内容及子节点的），其中一个系统update了znode，那么另一个系统能够收到通知，并作出相应处理。

- 心跳检测机制：检测系统和被检测系统之间并不直接关联起来，而是通过zk上某个节点关联，大大减少系统耦合。
- 系统调度模式：某系统有控制台和推送系统两部分组成，控制台的职责是控制推送系统进行相应的推送工作。管理人员在控制台作的一些操作，实际上是修改了ZK上某些节点的状态，而ZK就把这些变化通知给他们注册Watcher的客户端，即推送系统，于是，作出相应的推送任务。
- 工作汇报模式：一些类似于任务分发系统，子任务启动后，到zk来注册一个临时节点，并且定时将自己的进度进行汇报（将进度写回这个临时节点），这样任务管理者就能够实时知道任务进度。

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

  另外，这种场景演化一下，就是动态Master选举。这就要用到EPHEMERAL_SEQUENTIAL类型节点的特性了。

  上文中提到，所有客户端创建请求，最终只有一个能够创建成功。在这里稍微变化下，就是允许所有请求都能够创建成功，但是得有个创建顺序，于是所有的请求最终在ZK上创建结果的一种可能情况是这样： /currentMaster/{sessionId}-1 ,/currentMaster/{sessionId}-2 ,/currentMaster/{sessionId}-3 ….. 每次选取序列号最小的那个机器作为Master，如果这个机器挂了，由于他创建的节点会马上变小，那么之后最小的那个机器就是Master了。

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










