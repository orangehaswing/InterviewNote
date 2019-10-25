# 虚拟化 - KVM管理工具

## Libvirt

### 概要

Libvirt是目前使用最广泛的对KVM虚拟机进行管理的工具和应用程序接口，而且一些常用的虚拟机管理工具(virsh，virt-install，virt-manager等)和云计算框架平台(OpenStack，OpenNebula，Eucalyptus)都在底层使用libvirt的应用接口。

libvirt支持多种虚拟化，包括KVM，QEMU，Xen，VMware，VirtualBox等在内的平台虚拟化方案，又支持OpenVZ，LXC等Linux容器虚拟化。还支持用户态Linux虚拟化。

### 管理功能

Libvirt管理功能主要包含以下五个部分：

- 域的管理：包括对节点上域的各个生命周期的管理，如启动，停止，暂停，保存，恢复和动态迁移，也包括对多种设备类型的热插拔操作，包括磁盘，网卡，内存和CPU。
- 远程节点的管理：只要物理节点上运行了Libvirtd这个守护进程，经过认证和授权之后，所有的libvirt功能都可以被访问和使用。远程传输类型，如SSH，TCP套接字，Unix domain socket，支持TLS的加密传输等。
- 存储的管理：通过Libvirt管理不同类型的存储，如创建不同格式的客户机镜像(qcow2，raw，qde，vmdk等)、挂载NFS共享存储系统，查看现有的LVM卷组，创建新的LVM卷组和逻辑卷，对磁盘设备分区，挂载iSCSI共享存储等。
- 网络管理：任何运行Libvirt守护进程的主机，都可以通过libvirt来管理物理和逻辑的网络接口。包括列出现有的网络接口卡，配置网络接口，创建虚拟网络接口，网络接口的桥接，VLAN管理，NAT网络设置，为客户机分配虚拟网络接口等
- 提供一个稳定、可靠、高效的应用程序接口：
  - 应用程序编程接口库
  - 守护基础(libvirt)
  - 一个默认命令行管理工具(virsh)

### Libvird API

连接 Hypervisor 相关的API：以virConnect 开头的一系列函数。

1. 只有与 Hypervisor 建立了连接之后，才能进行虚拟机管理操作，所以连接 Hypervisor 的API是其他所有API使用的前提条件。与 Hypervisor 建立的连接是为其他API的执行提供了路径，是其他虚拟化管理功能的基础。通过调用 virConnectOpen 函数可以建立一个连接，其返回值是一个virConnectPtr 对象，该对象就代表到 Hypervisor 的一个连接；如果连接出错，则返回空值（NULL）。而 virConnectOpenReadOnly 函数会建立一个只读的连接，在该连接上可以使用一些查询的功能，而不使用创建、修改等功能。 virConnectOpenAuth 函数提供了更具认证建立的连接。 virConnectGetCapabilities 函数是返回对 Hypervisor 和驱动的功能的描述的 XML 格式的字符串。virConnectListDomains函数返回一列域标识符，它们代表该 Hypervisor 上的活动域。

2. 域管理的 API：以virDomain 开头的一系列函数。

   虚拟机的管理，最基本的职能就是对各个节点上的域的管理，故 libvirt API 中实现了很多针对域管理的函数。要管理域，首先就要获取virDomainPtr 这个域对象，然后才能对域进行操作。有很多种方式来获取域对象，如 virDomainPtr virDomainLookupByID (virConnectPtr conn, int id) 函数是根据域的 id 值到 conn 这个连接上去查找相应的域。类似地，virDomainLookupByName、virDomainLookupByUUID 等函数分别是根据域的名称和 UUID 去查找相应的域。在得到了某个域的对象后，就可以进行很多的操作，可以是查询域的信息（如：virDomainGetHostname、virDomainGetInfo、virDomainGetVcpus、virDomainGetVcpusFlags、virDomainGetCPUStats，等等），也可以是控制域的生命周期（如：virDomainCreate 、virDomainSuspend 、virDomainResume 、virDomainDestroy 、virDomainMigrate，等等）。

3. 节点管理的 API：以virNode 开头的一系列函数。

   域是运行在物理节点之上，libvirt也提供了对节点的信息查询和控制的功能。节点管理的多数函数都需要使用一个连接 Hypervisor 的对象作为其中的一个传入参数，以便可以查询或修改到该连接上的节点的信息。virNodeGetInfo函数是获取节点的物理硬件信息，virNodeGetCPUStats 函数可以获取节点上各个 CPU 的使用统计信息，virNodeGetMemoryStats 函数可以获取节点上的内存的使用统计信息，virNodeGetFreeMemory 函数可以获取节点上可用的空闲内存大小。也有一些设置或者控制节点的函数，如virNodeSetMemoryParameters 函数可以设置节点上的内存调度的参数，virNodeSuspendForDuration 函数可以让节点（宿主机）暂停运行一段时间。

4. 网络管理的 API：以 virNetwork 开头的一系列函数和部分以 virInterface 开头的函数。

   libvirt 对虚拟化环境中的网络管理也提供了丰富的API。libvirt 首先需要创建virNetworkPtr 对象，然后才能查询或控制虚拟网络。一些查询网络相关信息的函数，如：virNetworkGetName 函数可以获取网络的名称，virNetworkGetBridgeName 函数可以获取该网络中网桥的名称，virNetworkGetUUID 函数可以获取网络的 UUID 标识，virNetworkGetXMLDesc 函数可以获取网络的以 XML 格式的描述信息，virNetworkIsActive 函数可以查询网络是否正在使用中。一些控制或更改网络设置的函数，有：virNetworkCreateXML 函数可以根据提供的 XML 格式的字符串创建一个网络（返回 virNetworkPtr 对象），virNetworkDestroy 函数可以销毁一个网络（同时也会关闭使用该网络的域），virNetworkFree 函数可以回收一个网络（但不会关闭正在运行的域），virNetworkUpdate 函数可根据提供的 XML 格式的网络配置来更新一个已存在的网络。另外，virInterfaceCreate、virInterfaceFree、virInterfaceDestroy、virInterfaceGetName、virInterfaceIsActive 等函数可以用于创建、释放和销毁网络接口，以及查询网络接口的名称和激活状态。

5. 存储卷管理的 API：以 virStorageVol 开头的一系列函数。

   libvirt 对存储卷（volume）的管理，主要是对域的镜像文件的管理，这些镜像文件可能是 raw、qcow2、vmdk、qed等各种格式。libvirt 对存储卷的管理，首先需要创建virStorageVolPtr 这个存储卷的对象，然后才能对其进行查询或控制操作。libvirt 提供了3个函数来分别通过不同的方式来获取存储卷对象，如：virStorageVolLookupByKey 函数可以根据全局唯一的键值来获得一个存储卷对象，virStorageVolLookupByName 函数可以根据名称在一个存储资源池（storage pool）中获取一个存储卷对象，virStorageVolLookupByPath 函数可以根据它在节点上路径来获取一个存储卷对象。有一些函数用于查询存储卷的信息，如：virStorageVolGetInfo 函数可以查询某个存储卷的使用情况，virStorageVolGetName 函数可以获取存储卷的名称，virStorageVolGetPath 函数可以获取存储卷的路径，virStorageVolGetConnect 函数可以查询存储卷的连接。一些函数用于创建和修改存储卷，如：virStorageVolCreateXML 函数可以根据提供的 XML 描述来创建一个存储卷，virStorageVolFree 函数可以释放存储卷的句柄（但是存储卷依然存在），virStorageVolDelete 函数可以删除一个存储卷，virStorageVolResize 函数可以调整存储卷的大小。

6. 存储池管理的 API：以virStoragePool 开头的一系列函数。

   libvirt 对存储池（pool）的管理，包括对本地的基本文件系统、普通网络共享文件系统、iSCSI共享文件系统、LVM分区等的管理。libvirt 需要基于 virStoragePoolPtr 这个存储池对象才能进行查询和控制操作。一些函数可以通过查询获取一个存储池对象，如：virStoragePoolLookupByName 函数可以根据存储池的名称来获取一个存储池对象，virStoragePoolLookupByVolume 可以根据一个存储卷返回其对应的存储池对象。virStoragePoolCreateXML 函数可以根据 XML 描述来创建一个存储池（默认已激活），virStoragePoolDefineXML 函数可以根据 XML 描述信息静态地定义个存储池（尚未激活），virStoragePoolCreate 函数可以激活一个存储池。virStoragePoolGetInfo、virStoragePoolGetName、virStoragePoolGetUUID等函数可以分别获取存储池的信息、名称和 UUID 标识。virStoragePoolIsActive函数可以查询存储池是否处于使用中状态。virStoragePoolFree 函数可以释放存储池相关的内存（但是不改变其在宿主机中的状态），virStoragePoolDestroy 函数可以用于销毁一个存储池（但并没有释放virStoragePoolPtr 对象，之后还可以用virStoragePoolCreate 函数重新激活它），virStoragePoolDelete 函数可以物理删除一个存储池资源（该操作不可恢复）。

7. 事件管理的API：以virEvent 开头的一系列函数。

   libvirt 支持事件机制，使用该机制注册之后，可以在发生特定的事件（如：域的启动、暂停、恢复、停止等）之时，得到自己定义的一些通知。

8. 数据流管理的API：以virStream 开头的一系列函数。

   libvirt 还提供了一系列函数用于数据流的传输。

## virsh

Libvirt项目的源代码中包含了virsh这个虚拟化管理工具的代码。virsh用于管理虚拟机环境中的客户机和Hypervisor的命令行工具，通过调用libvirt API来实现虚拟化的管理。

### virsh常用命令







