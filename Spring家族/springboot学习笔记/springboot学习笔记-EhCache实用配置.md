# springboot学习笔记-EhCache实用配置

# 持久化

## 配置

```
<ehcache xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:noNamespaceSchemaLocation="http://code.taobao.org/p/tgw/diff/3/tgw/src/ehcache.xsd">


    <!--<diskStore path="java.io.tmpdir"/>-->
    <diskStore path="C:\ehcache"/>

    <defaultCache
            maxElementsInMemory="10000"
            eternal="false"
            timeToIdleSeconds="120"
            timeToLiveSeconds="120"
            overflowToDisk="true"
            maxElementsOnDisk="10000000"
            diskPersistent="false"
            diskExpiryThreadIntervalSeconds="120"
            memoryStoreEvictionPolicy="LRU"
    />

    <cache name="testCache"
           maxBytesLocalHeap="50m"
           timeToLiveSeconds="100">
    </cache>

    <!--
       maxElementsInMemory设置成1，overflowToDisk设置成true，只要有一个缓存元素，就直接存到硬盘上去
       eternal设置成true，代表对象永久有效
       maxElementsOnDisk设置成0 表示硬盘中最大缓存对象数无限大
       diskPersistent设置成true表示缓存虚拟机重启期数据
    -->
    <cache
            name="disk"
            maxElementsInMemory="1"
            eternal="true"
            overflowToDisk="true"
            maxElementsOnDisk="0"
            diskPersistent="true"/>
</ehcache>

```

**web.xml**

```
<!-- ehcache 磁盘缓存 监控，持久化恢复 -->
<listener>
    <listener-class>net.sf.ehcache.constructs.web.ShutdownListener</listener-class>
</listener>

```

# EhCache单机配置

## ehcache.xml

```
<ehcache xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="../config/ehcache.xsd">   

    <diskStore path="java.io.tmpdir"/>   

    <!--   
    Mandatory Default Cache configuration. These settings will be applied to caches   
    created programmtically using CacheManager.add(String cacheName)   
    -->   
    <!--   
       name:缓存名称。   
       maxElementsInMemory：缓存最大个数。   
       eternal:对象是否永久有效，一但设置了，timeout将不起作用。   
       timeToIdleSeconds：设置对象在失效前的允许闲置时间（单位：秒）。仅当eternal=false对象不是永久有效时使用，可选属性，默认值是0，也就是可闲置时间无穷大。   
       timeToLiveSeconds：设置对象在失效前允许存活时间（单位：秒）。最大时间介于创建时间和失效时间之间。仅当eternal=false对象不是永久有效时使用，默认是0.，也就是对象存活时间无穷大。   
       overflowToDisk：当内存中对象数量达到maxElementsInMemory时，Ehcache将会对象写到磁盘中。   
       diskSpoolBufferSizeMB：这个参数设置DiskStore（磁盘缓存）的缓存区大小。默认是30MB。每个Cache都应该有自己的一个缓冲区。   
       maxElementsOnDisk：硬盘最大缓存个数。   
       diskPersistent：是否缓存虚拟机重启期数据 Whether the disk store persists between restarts of the Virtual Machine. The default value is false.   
       diskExpiryThreadIntervalSeconds：磁盘失效线程运行时间间隔，默认是120秒。   
       memoryStoreEvictionPolicy：当达到maxElementsInMemory限制时，Ehcache将会根据指定的策略去清理内存。默认策略是LRU（最近最少使用）。你可以设置为FIFO（先进先出）或是LFU（较少使用）。   
       clearOnFlush：内存数量最大时是否清除。   
    -->   
    <defaultCache   
            maxElementsInMemory="10000"  
            eternal="false"  
            timeToIdleSeconds="120"  
            timeToLiveSeconds="120"  
            overflowToDisk="true"  
            maxElementsOnDisk="10000000"  
            diskPersistent="false"  
            diskExpiryThreadIntervalSeconds="120"  
            memoryStoreEvictionPolicy="LRU"  
            />   
</ehcache>  


```

## 缓存清理策略

1 FIFO ，first in first out，先进先出，
2 LFU ， Less Frequently Used,最少被使用的。如上面所讲，缓存的元素有一个hit 属性，hit 值最小的将会被清出缓存。
2 LRU ，Least Recently Used，最近最少使用的，缓存的元素有一个时间戳，当缓存容量满了，而又需要腾出地方来缓存新的元素的时候，那么现有缓存元素中时间戳离当前时间最远的元素将被清出缓存。

## 配置节点

### diskStore

配置持久化路径，被持久化的对象需要实现序列化接口
有一个path属性，path 配置文件存储位置，如user.home，user.dir，java.io.tmpdir

### defaultCache

默认缓存配置，当未指定缓存名而对缓存进行操作的时候，都是对此缓存在进行操作

```
 必须属性：
        name:设置缓存的名称，用于标志缓存,惟一
        maxElementsInMemory:在内存中最大的对象数量
        maxElementsOnDisk：在DiskStore中的最大对象数量，如为0，则没有限制
        eternal：设置元素是否永久的，如果为永久，则timeout忽略
        overflowToDisk：是否当memory中的数量达到限制后，保存到Disk

可选的属性：
    timeToIdleSeconds：设置元素过期前的空闲时间
    timeToLiveSeconds：设置元素过期前的活动时间
    diskPersistent：是否disk store在虚拟机启动时持久化。默认为false
    diskExpiryThreadIntervalSeconds:运行disk终结线程的时间，默认为120秒
    memoryStoreEvictionPolicy：策略关于Eviction

```

[EhCache详细功能](https://link.jianshu.com?t=http%3A%2F%2Fblog.csdn.net%2Ftang06211015%2Farticle%2Fdetails%2F52281551)

# EhCache集群配置

## 为什么要配置集群

大型网站往往采用集群部署，EhCache是进程中的缓存系统
也就是说，每台机器上的应用都会有各自的缓存系统，但是对于客户而言，访问入口只有一个
所以需要保证每台机器上的缓存同步

## EhCache集群配置实战

### 验证思路

在两个机器中配置缓存，分别为A、B
先访问A，将数据写入缓存
再访问B，看日志判断是新的查询还是读取的缓存
再访问B，写入新的数据
再访问A，看日志判断A能否同步B中的缓存数据

### 配置

ehcache的三种最为常用集群方式，分别是 RMI、JGroups 以及 EhCache Server 。

主要步骤
1、配置主从节点
2、配置事件监听器
3、配置缓存初始化工厂类

A的ehcache.xml

```
<ehcache>

    <cacheManagerPeerProviderFactory
            class="net.sf.ehcache.distribution.RMICacheManagerPeerProviderFactory"
            properties="peerDiscovery=automatic, multicastGroupAddress=224.1.1.1,
            multicastGroupPort=1000, timeToLive=32"/>
    <cacheManagerPeerListenerFactory
            class="net.sf.ehcache.distribution.RMICacheManagerPeerListenerFactory"
            properties="hostName=127.0.0.1,port=1000,socketTimeoutMillis=120000"/>

    <cache name="testCache"
           maxBytesLocalHeap="50m"
           timeToLiveSeconds="100">
        <cacheEventListenerFactory class="net.sf.ehcache.distribution.RMICacheReplicatorFactory"
                                   properties="replicateAsynchronously=true,
                                   replicatePuts=true,
                                   replicateUpdates=true,
                                   replicateUpdatesViaCopy= false,
                                   replicateRemovals= true "/>
        <bootstrapCacheLoaderFactory
                class="net.sf.ehcache.distribution.RMIBootstrapCacheLoaderFactory"/>
    </cache>
</ehcache>

```

B的ehcache.xml

```
<ehcache>

    <cacheManagerPeerProviderFactory
            class="net.sf.ehcache.distribution.RMICacheManagerPeerProviderFactory"
            properties="peerDiscovery=automatic, multicastGroupAddress=224.1.1.1,
            multicastGroupPort=1000, timeToLive=32"/>
    <cacheManagerPeerListenerFactory
            class="net.sf.ehcache.distribution.RMICacheManagerPeerListenerFactory"
            properties="hostName=127.0.0.1,port=2000,socketTimeoutMillis=120000"/>

    <cache name="testCache"
           maxBytesLocalHeap="50m"
           timeToLiveSeconds="100">
        <cacheEventListenerFactory class="net.sf.ehcache.distribution.RMICacheReplicatorFactory"
                                   properties="replicateAsynchronously=true,
                                   replicatePuts=true,
                                   replicateUpdates=true,
                                   replicateUpdatesViaCopy= false,
                                   replicateRemovals= true "/>
        <bootstrapCacheLoaderFactory
                class="net.sf.ehcache.distribution.RMIBootstrapCacheLoaderFactory"/>
    </cache>
</ehcache>

```

```
peerDiscovery 方式：atutomatic 为自动 ；
mulicastGroupAddress 广播组地址：230.0.0.1；
mulicastGroupPort 广播组端口：40001；
timeToLive 搜索某个网段上的缓存：0是限制在同一个服务器，1是限制在同一个子网，32是限制在同一个网站，64是限制在同一个region，128是同一块大陆，还有个256；
hostName：主机名或者ip，用来接受或者发送信息的接口。同时组播地址可以指定 D 类 IP 地址空间，范围从 224.0.1.0 到 238.255.255.255 中的任何一个地址。
```

[EhCache集群配置](https://link.jianshu.com?t=http%3A%2F%2Fblog.csdn.net%2Ftang06211015%2Farticle%2Fdetails%2F52281551)