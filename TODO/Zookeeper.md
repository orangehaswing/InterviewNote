# Zookeeper

zookeeper在分布式系统中作为协调员的角色，可应用于Leader选举、分布式锁、配置管理等服务的实现。

## ZK API

ZK以Unix文件系统树结构的形式管理存储的数据，图示如下：

![img](https://images0.cnblogs.com/blog2015/116770/201504/142337054797921.png)

其中每个树节点被称为**znode**，每个znode类似一个文件，包含文件元信息(meta data)和数据。

有两种类型的znode：

- **Regular**: 该类型znode只能由client端显式创建或删除
- **Ephemeral**: client端可创建或删除该类型znode；当session终止时，ZK亦会删除该类型znode

ZK提供了以下API，供client操作znode和znode中存储的数据：

- create(path, data, flags)：创建路径为path的znode，在其中存储data[]数据，flags可设置为Regular或Ephemeral，并可选打上sequential标志。
- delete(path, version)：删除相应path/version的znode
- exists(path,watch)：如果存在path对应znode，则返回true；否则返回false，watch标志可设置监听事件
- getData(path, watch)：返回对应znode的数据和元信息（如version等）
- setData(path, data, version)：将data[]数据写入对应path/version的znode
- getChildren(path, watch)：返回指定znode的子节点集合

## ZK应用场景

基于以上ZK提供的znode和znode数据的操作，可轻松实现Leader选举、分布式锁、配置管理等服务。

### Leader选举

利用打上sequential标志的Ephemeral，我们可以实现Leader选举。假设需要从三个client中选取Leader

**1**、各自创建Ephemeral类型的znode，并打上sequential标志

**2**、检查 /master 路径下的所有znode，如果自己创建的znode序号最小，则认为自己是Leader；否则记录序号比自己次小的znode

**3**、非Leader在次小序号znode上设置监听事件，并重复执行以上步骤2

### 配置管理

znode可以存储数据，基于这一点，我们可以用ZK实现分布式系统的配置管理，假设有服务A，A扩容设备时需要将相应新增的ip/port同步到全网服务器的A.conf配置

1. A扩容时，相应在ZK上新增znode
2. 全网机器监听 /A，当该znode下有新节点加入时，调用相应处理函数，将服务A的新增ip/port加入A.conf
3. 完成步骤2后，继续设置对 /A监听

服务缩容的步骤类似，机器下线时将ZK相应节点删除，全网机器监听到该事件后将配置中的设备剔除。

## ZK监控

ZK自身提供了一些“四字命令”，通过这些四字命令，我们可以获得ZK集群中，某台ZK的角色、znode数、健康状态等信息

常用的四字命令有：

- **mntr**：显示自身角色、znode数、平均调用耗时、收包发包数等信息
- **ruok**：诊断自身状态是否ok
- **cons**：展示当前的client连接

















