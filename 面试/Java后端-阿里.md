# Java后端-阿里

## **阿里一面题目：**

1. osi七层网络模型，五层网络模型，每次层分别有哪些协议

   1. TCP四层：应用层，运输层，网络层，接口层

2. 死锁产生的条件， 以及如何避免死锁，**银行家算法**，产生死锁后如何解决

   1. 必要条件：资源互斥；等待与请求；不可抢夺；循环等待；
   2. 避免死锁：安全状态，单个资源的银行家算法，多个资源的银行家算法；自己写一个支持多线程的消息管理类，单开一个线程访问独占资源，其它线程用消息交互实现间接访问。
   3. 死锁解决：产生死锁时忽视；小心动态分配资源，避免死锁；死锁的检查和恢复：抢占资源，杀死进程，回滚；破坏死锁必要条件；
   4. 银行家算法：https://github.com/orangehaswing/InterviewNote/blob/master/Linux%E5%92%8C%E8%AE%A1%E7%AE%97%E6%9C%BA%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F/%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F%E5%8E%9F%E7%90%86/%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F%E5%8E%9F%E7%90%86-%E6%AD%BB%E9%94%81.md

3. 如何判断链表有环

   快慢指针

   hashmap

4. 虚拟机类加载机制，双亲委派模型，以及为什么要实现双亲委派模型

   1. 加载二进制码 2. 验证class文件 3. 准备静态变量初始值 4. 解析间接引用变为直接引用 5.初始化init，构造函数 6. 使用 7.卸载
   2. 双亲委派：类的加载器：启动类java home lib 扩展类java home lib ext 应用类加载器 用户指定实现路径。双亲委派每个类都由父类加载，类的启动项具有优先级。类加载请求传送到父类加载器，只有当父类加载器无法完成类加载请求时才尝试加载。
   3. 类加载器一起具有一种带有优先级的层次关系，从而使得基础类得到统一，保证了对象相等；防止恶意类被加载进，始终加载的是可信任的类；findClass和loadClass

5. 虚拟机调优参数

   堆：初始xms 最大xmx 新生代 xmn 栈：xss  初始化永久代大小 -XX:PermSize 永久代最大容量 -XX:MaxPermSize

6. 拆箱装箱的原理

   原始类型值和对应的对象互换；valueOf方法（自动装箱）和intValue()方法（自动拆箱）

7. JVM垃圾回收算法

   复制；标记-回收；标记-整理；分代收集

8. CMS G1

   G1 商业 ，面向服务端，目的希望替换CMS。G1 把堆划分成多个大小相等的独立区域（Region），新生代和老年代不再物理隔离，每个小空间可以单独进行垃圾回收；两者都支持并发使用；CMS使用标记-清除，G1使用标记-整理 + 复制算法

9. hashset和hashmap的区别，haspmap的底层实现put操作，扩容机制，currenthashmap如何解决线程安全,1.7版本以及1.8版本的不同

   hashset的底层实现是hashmap，将value值设置为常量；hashset：不重复的元素，查找都是O(1)；hashmap：key 和value都可以是null，键值对，使用键的hashcode计算，hashset使用成员对象计算。

10. md5加密的原理

11. 有多少种方法可以让线程阻塞，能说多少说多少

- 当线程执行 Thread.sleep() 时，它一直阻塞到指定的毫秒时间之后，或者阻塞被另一个线程打断；
- 当线程碰到一条 wait() 语句时，它会一直阻塞到接到通知 notify()、被中断或经过了指定毫秒时间为止（若制定了超时值的话）
- 线程阻塞与不同 I/O 的方式有多种。常见的一种方式是 InputStream 的 read() 方法，该方法一直阻塞到从流中读取一个字节的数据为止，它可以无限阻塞，因此不能指定超时时间；
- 线程也可以阻塞等待获取某个对象锁的排他性访问权限（即等待获得 synchronized 语句必须的锁时阻塞）。

12. synchronized和reetrantlock锁

   jvm jdk；对synchronized进行自旋锁优化，性能差不多；公平锁和非公平锁；可中断和不可中断；reetrantlock可以绑定多个Condition 对象

13. AQS同步器框架，countdowmlatch，cyclebarrier，semaphore，读写锁

   countdowmlatch：一个线程等待多个线程执行完；维护了一个计数器 cnt

   cyclebarrier：循环等待， 执行await() 方法之后计数器会减 1，通过调用 reset() 方法可以循环使用，所以它才叫做循环屏障。

   semaphore：信号量，对互斥资源限制访问的线程数

   AQS定义了一套多线程访问共享资源的同步器框架。整个AQS同步组件、Atomic原子类操作等等都是以CAS实现的

   读写锁：一个资源的访问分成了2个锁。一个读锁和一个写锁

## 阿里二面题目：

1. B-Tree索引，myisam和innodb中索引的区别

   索引：不再需要进行全表扫描，只需要对树进行搜索即可，所以查找速度快很多。除了用于查找，还可以用于排序和分组。

   索引分类：B+Tree 哈希索引 全文索引 空间数据索引

   区别：

   - 事务：InnoDB 是事务型的，可以使用 Commit 和 Rollback 语句。
   - 并发：MyISAM 只支持表级锁，而 InnoDB 还支持行级锁。
   - 外键：InnoDB 支持外键。
   - 备份：InnoDB 支持在线热备份。
   - 崩溃恢复：MyISAM 崩溃后发生损坏的概率比 InnoDB 高很多，而且恢复的速度也更慢。
   - 其它特性：MyISAM 支持压缩表和空间数据索引。

2. BIO和NIO的应用场景

3. 讲讲threadlocal

4. 数据库隔离级别，每层级别分别用什么方法实现，三级封锁协议,共享锁排它锁，mvcc多版本并发控制协议，间隙锁

   未提交读（READ UNCOMMITTED）: 事务中的修改，即使没有提交，对其它事务也是可见的。

   提交读（READ COMMITTED）: 一个事务只能读取已经提交的事务所做的修改。换句话说，一个事务所做的修改在提交之前对其它事务是不可见的。

   可重复读（REPEATABLE READ）: 保证在同一个事务中多次读取同样数据的结果是一样的。

   可串行化（SERIALIZABLE）: 强制事务串行执行。

   ​

5. 数据库索引？B+树？为什么要建索引？什么样的字段需要建索引，建索引的时候一般考虑什么？索引会不会使插入、删除作效率变低，怎么解决？

6. 数据库表怎么设计的？数据库范式？设计的过程中需要注意什么？

   ​

7. 共享锁与非共享锁、一个事务锁住了一条数据，另一个事务能查吗？

8. Spring bean的生命周期？默认创建的模式是什么？不想单例怎么办？

## **阿里三面题：**

1. 高并发时怎么限流

2. 线程池的拒接任务策略

   1. workerCountOf方法根据ctl的低29位，得到线程池的当前线程数，如果线程数小于corePoolSize，则执行addWorker方法创建新的线程执行任务；否则执行步骤（2）；
   2. 如果线程池处于RUNNING状态，且把提交的任务成功放入阻塞队列中，则执行步骤（3），否则执行步骤（4）；
   3. 再次检查线程池的状态，如果线程池没有RUNNING，且成功从阻塞队列中删除任务，则执行reject方法处理任务；
   4. 执行addWorker方法创建新的线程执行任务，如果addWoker执行失败，则执行reject方法处理任务；

   线程池中的核心线程数，当提交一个任务时，线程池创建一个新线程执行任务，直到当前线程数等于corePoolSize；如果当前线程数为 corePoolSize，继续提交的任务被保存到阻塞队列中，等待被执行；如果阻塞队列满了，那就创建新的线程执行当前任务；直到线程池中的线程数达到 maxPoolSize，这时再有任务来，只能执行 reject() 处理该任务。

3. HashMap和Hashtable的区别

   Hashtable线程安全，使用线程安全的类用ConcurrentHashMap，这个类被废弃；HashMap可以存放null值；容器初始值：HashMap 16 扩展 2的倍数，Hashtable是11，扩展是2n + 1；底层实现是 数组 + 链表/大于8转变成红黑树

   hashMap使用快速失败机制

4. 实现一个保证迭代顺序的HashMap

   LinkedHashMap保证顺序，LRU

5. 说一说排序算法，稳定性，复杂度

   选择n2，冒泡n2，插入n-n2，希尔N 的若干倍乘于递增序列的长度，归并nlogn，快速nlogn，堆nlogn；

6. 说一说GC

   minor GC：eden满的时候回收

   full GC：System.gc();老年代空间不足；空间担保失败；JDK 1.7 及以前的永久代空间不足

   JVM内存：程序计数器，虚拟机栈，本地方法栈；堆，方法区

   算法：复制，标记-清除，标记-整理，分代回收：新生代：复制；老年代：标记-清除，标记-整理

   CMS和G1：G1 商业 ，面向服务端，目的希望替换CMS。G1 把堆划分成多个大小相等的独立区域（Region），新生代和老年代不再物理隔离，每个小空间可以单独进行垃圾回收；两者都支持并发使用；CMS使用标记-清除，G1使用标记-整理 + 复制算法

7. JVM如何加载一个类的过程，双亲委派模型中有哪些方法？

8. TCP如何保证可靠传输？三次握手过程？

   可靠性：三次握手，四次挥手，超时重传，滑动窗口，流量控制；拥塞控制：慢开始，拥塞避免，快恢复，快启动

9. springboot的启动流程

   springmvc启动流程：

   ​	运行：DispatcherServlet  init方法：对WebApplicationContext、组件和外部资源的初始化。service()处于侦听模式。

   ​	继承：DispatcherServlet继承FrameworkServlet，HttpServletBean，HttpServlet。

   HttpServletBean中，Spring会将这个Servlet视作是一个Spring的bean；FrameworkServlet真正初始化了一个Spring的容器（WebApplicationContext），并引入到Servlet对象。

   ​	运行体系：整个运行体系由DispatcherServlet、组件、容器构成。DispatcherServlet是逻辑处理的调度中心，组件则是被调度的操作对象。容器是协助DispatcherServlet更好地对组件进行管理。

   - 初始化主线的驱动要素 —— servlet中的init方法
   - 初始化主线的执行次序 —— HttpServletBean -> FrameworkServlet -> DispatcherServlet
   - 初始化主线的操作对象 —— Spring容器（WebApplicationContext）和组件

   DispatcherServlet步骤：

   - 1. 发起请求到前端控制器(`DispatcherServlet`)
   - 2. 前端控制器请求处理器映射器(`HandlerMapping`)查找`Handler`(可根据xml配置、注解进行查找)
   - 3. 处理器映射器(`HandlerMapping`)向前端控制器返回`Handler`
   - 4. 前端控制器调用处理器适配器(`HandlerAdapter`)执行`Handler`
   - 5. 处理器适配器(HandlerAdapter)去执行Handler
   - 6. Handler执行完，给适配器返回ModelAndView(Springmvc框架的一个底层对象)
   - 7. 处理器适配器(`HandlerAdapter`)向前端控制器返回`ModelAndView`
   - 8. 前端控制器(`DispatcherServlet`)请求视图解析器(`ViewResolver`)进行视图解析，根据逻辑视图名解析成真正的视图(jsp)
   - 9. 视图解析器(ViewResolver)向前端控制器(`DispatcherServlet`)返回View
   - 10. 前端控制器进行视图渲染，即将模型数据(在`ModelAndView`对象中)填充到request域
   - 11. 前端控制器向用户响应结果

   **springboot启动流程：**

   ​

10. 集群、负载均衡、分布式、数据一致性的区别与关系

11. 数据库如果让你来垂直和水平拆分，谁先拆分，拆分的原则有哪些(单表数据量多大拆)

12. 最后谈谈Redis、Kafka、 Dubbo，各自的设计原理和应用场景

## **面试总结：**

**通过这次面试题和之前发的阿里面试题来看，可以总结出目前互联网公司面试考点为：**

1. 性能调优、算法数据机构
2. 高并发下数据安全、接口冪等性、原子性等
3. 分布式下协同、已经锁的处理
4. 数据库的分库分表、项目之间的垂直拆分

**详细技术点为：**

- HashMap
- JVM  【必问】
- Dubbo
- Mybatis
- Zookeeper
- http tcp/ip


# 面试

## Part.1

- 进程线程 

  进程和线程的关系

  线程之间变量能否互相访问 

  线程能否访问进程变量 


- 数据库 

  mysql语句

  平衡二叉树 

- http

  对称和非对称加密 以及应用场景 

  https过程 

  http协议2.0和1.1的区别

- 两个有序数组 怎么找到相同的元素 

## Part.2

- jvm

  什么时候发生stackOverflow

  一个线程的工作栈是多大？

  哪些区域会发生OOM

  Jvm的线程和操作系统线程的关系

- Hashmap和TreeMap

  hashmap的实现，怎么解决冲突，其他解决冲突的方法，使用过哪些线程安全的集合，优先队列的实现（怎么实现排序），hashmap的插入时间复杂度。

  TreeMap的实现，红黑树的优点，介绍一下其他的平衡树，数据库索引为什么采用B+树，插入节点的时间复杂度。

- 多线程

  死锁的条件 

  消息队列的使用

  多线程怎么使用消息队列

  生产者消费者模型的实现

  synchronize关键字的作用，修饰方法、变量和类的区别


# 阿里内推

## Part.1



1. Hashmap的原理   Hashmap为什么大小是 2的幂次   
2. 介绍一下红黑树  
3. Arraylist 的原理  
4. 场景题：设计判断论文抄袭的系统   
5. 堆排序的原理   
6. 抽象工厂和工厂方法模式的区别；工厂模式的思想；哪里用到了工厂模式
7. object 类你知道的方法
8. Forward 和redirect 的区别 

## Part.2 

2. 项目介绍；项目架构； 项目难点    
3. Synchronize 关键字为什么jdk1.5 后效率提高了    
4. 线程池的使用时的注意事项    
5. Spring 中 autowire 和resourse 关键字的区别    
6. Hashmap的原理；Hashmap的大小为什么指定为 2的幂次    
7. 讲一下线程状态转移图   
8. 消息队列   
9. 分布式
