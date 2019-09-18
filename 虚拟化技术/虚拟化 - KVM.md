# 虚拟化 - KVM

## 云计算模式

### 软件即服务(SaaS)

在 SaaS模式中，厂商将应用软件统一部署在自己的服务器上，客户可以根据自己实际需求，通过互联网向厂商定购所需的应用软件服务，按定购的服务多少和时间 长短向厂商支付费用，并通过互联网获得厂商提供的服务。

### 平台即服务(PaaS)

通过PaaS这种模式，用户可以在一个包括SDK，文档和测试环境等在内的开发平台上非常方便地编写应用，而且不论是在部署，或者在运行的时候，用户都无需 为服务器，操作系统，网络和存储等资源的管理操心。

### 基础设施即服务(IaaS)

通过IaaS这种模式，用户可以从供应商那里获得他所需要的虚拟机或者存储等资源来装载相关的应用，同时这些基础设施的繁琐的管理工作将由IaaS供应商来处理。

## 虚拟化技术

虚拟化及时是一种资源管理(优化)技术。根据不用对象类型，可以细分为：

- **平台虚拟化（Platform Virtualization）：**针对计算机和操作系统的虚拟化。
- **资源虚拟化（Resource Virtualization）：**针对特定的系统资源的虚拟化，如内存、存储、网络资源等。
- **应用程序虚拟化（Application Virtualization）：**包括仿真、模拟、解释技术等，如 Java 虚拟机（JVM）。

### 平台虚拟化

VMM 运行在传统的操作系统上，就像其他计算机程序那样运行。

![img](https://ask.qcloudimg.com/http-save/yehe-4895051/0net8avhv6.jpeg?imageView2/2/w/1620)

### 全虚拟化

全虚拟化是指虚拟机模拟了完整的底层硬件，包括处理器、物理内存、时钟、外设等，使得为原始硬件设计的操作系统或其它系统软件完全不做任何修改就可以在虚拟机中运行。

一般的，CPU 都会划分为用户态和内核态，而 x86 CPU 更是细分为了 Ring 0~3 四种执行状态。

Ring0 核心态（Kernel Mode）：是操作系统内核的执行状态（运行模式），运行在核心态的代码可以无限制的对系统内存、设备驱动程序、网卡接口、显卡接口等外部设备进行访问。

Ring3 用户态（User Mode）：运行在用户态的程序代码，需要受到 CPU 的检查，受限的内存访问。

![img](https://ask.qcloudimg.com/http-save/yehe-4895051/27chw7gl5z.jpeg?imageView2/2/w/1620)

工作原理：全虚拟化需要在 VMM 中模拟出一颗包含了控制单元、运算单元、存储单元、IS（指令集）的 CPU；此外，还需要模拟一张进行虚拟存储地址和物理存储地址转换的页表；此外，还需要在 VMM 模拟磁盘设备控制器、网络适配器等等各种 I/O 外设接口。如此依赖，Guest OS 就不知道自己其实是个虚拟机了呀，它受到了欺骗。

### 半虚拟化

半虚拟化是一种通过修改 Guest OS 部分访问特权状态的代码以便直接与 VMM 交互的技术。部分硬件接口以软件的形式提供给 Guest OS，这可以通过 Hypercall（VMM 提供给 Guest OS 的直接调用，与系统调用类似）的方式来提供。

![img](https://ask.qcloudimg.com/http-save/yehe-4895051/6d8j7581ts.jpeg?imageView2/2/w/1620)

相较于全虚拟化，半虚拟化 VMM 只需要模拟部分底层硬件，因此 Guest OS 不做修改是无法在虚拟机中运行的，甚至运行在虚拟机中的其它程序也需要进行修改，如此代价，换来的就是接近于物理机的虚拟机性能。

### 硬件辅助全虚拟化

![img](https://ask.qcloudimg.com/http-save/yehe-4895051/4c7izurj7y.jpeg?imageView2/2/w/1620)

- Intel-VT（Intel Virtualization Technology）
- AMD-V 

### 操作系统虚拟化(容器)

操作系统虚拟化（OS-level virtualization） 是一种在服务器操作系统中使用的、没有 VMM 层的轻量级虚拟化技术，内核通过创建多个虚拟的操作系统实例（内核和库）来隔离不同的进程（容器），不同实例中的进程完全不了解对方的存在。![img](https://ask.qcloudimg.com/http-save/yehe-4895051/rqoucj9zht.jpeg?imageView2/2/w/1620)

## KVM

kvm是一种用于Linux内核中的虚拟化基础设施，可以将Linux内核转化为一个hypervisor。包含一个为处理器提供底层虚拟化 可加载的核心模块kvm.ko（kvm-intel.ko或kvm-AMD.ko）还需要一个经过修改的QEMU软件（qemu-kvm），作为虚拟机上层控制和界面。

### 功能特性

- 内存管理：kvm在支持Intel的扩展页表(EPT)和AMD的嵌套页表(NPT)，内存共享通过KSM的内核功能来支持。
- 存储：KVM能够使用Linux支持的任何存储来存储虚拟机镜像。包括IDE,SCS和SATA的本地磁盘，NAS，iSCSI和光纤通信。
- 设备驱动程序：KVM支持混合虚拟机，允许虚拟机使用优化的I/O接口而不使用模拟的设备，从而为网络和块设备提供高性能的I/O。
- 性能和可伸缩性：继承来Linux的性能和可伸缩性。

### KVM架构

![img](https://ask.qcloudimg.com/http-save/yehe-1043708/rvd3d5f8k8.png?imageView2/2/w/1620)

- **/dev/kvm** 接口是 Qemu 和 KVM 交互的“桥梁”


- 基本的原理：/dev/kvm 本身是一个设备文件，这就意味着可以通过 ioctl 函数来对该文件进行控制和管理，从而可以完成用户空间与内核空间的数据交互。在 KVM 与 Qemu 的通信过程主要就是一系列针对该设备文件的 ioctl 调用。

- KVM模块：是KVM虚拟机的核心部分，主要功能是初始化CPU硬件，打开虚拟模式，然后将虚拟客户机运行在虚拟机模式下，并对虚拟客户机的运行提供一定的支持。

  - 初始化：先初始化内部数据结构；检测当前CPU，打开控制寄存器CR4中的虚拟模式开关；执行VMXON指令，将宿主操作系统至于虚拟化模式的根模式；KVM模块创建特殊设备文件/dev/kvm/并等待来自用户控件的命令
  - KVM与QEMU通信接口：主要是一系列针对特殊设备的IOCTL调用。可以对虚拟机做相应的管理，比如创建用户控件虚拟地址，物理地址的映射关系；创建多个可供运行的虚拟处理器(vcpu)。

- 内存虚拟化

  处理器中的内存管理单元(MMU)是通过页表的形式将程序运行的虚拟地址转换成物理内存地址。在虚拟机模式下，内存管理单元的页表必须在一次查询的时候完成两次地址转换。这是因为，除了要将客户机程序的虚拟地址转换成为客户机物理内存地址，还必须将客户机物理地址转换成真实物理地址。

  因此做了EPT技术，通过引入第二级页表来描述客户机虚拟地址和真实物理地址转换，硬件可以自动进行两级转换生成正确的内存访问地址。



### QEMU

Qemu 是纯软件实现的虚拟化模拟器，几乎可以模拟任何硬件设备，我们最熟悉的就是能够模拟一台能够独立运行操作系统的虚拟机，虚拟机认为自己和硬件打交道，但其实是和 Qemu 模拟出来的硬件打交道，Qemu 将这些指令转译给真正的硬件。

![img](https://ask.qcloudimg.com/http-save/yehe-1043708/oz83fzquuj.png?imageView2/2/w/1620)

从本质上看，虚拟出的每个虚拟机对应 host 上的一个 Qemu 进程，而虚拟机的执行线程（如 CPU 线程、I/O 线程等）对应 Qemu 进程的一个线程。

## KVM核心

### CPU虚拟化

QEMU/KVM中，QEMU提供对CPU的模拟，展现一定的CPU数目和CPU的特性；在KVM打开的情况下，客户机中CPU指令的执行由硬件处理器的虚拟化功能来辅助执行。

#### VCPU

在KVM环境中，每个客户机都是一个标准的Linux进程(QEMU进程)，而每一个vcpu是QEMU派生出的一个普通线程。

vcpu执行模式：

1. 用户模式：主要处理I/O的模拟和管理，由QEMU的代码实现；
2. 内核模式：处理需要高性能和安全相关的指令，如客户模式到内核模式的转换，处理器模式下的I/O，影子内存管理；
3. 客户模式：执行guest的大部分指令，I/O和一些特权指令除外；

![vCPU在KVM中的三种执行模式](https://img-blog.csdn.net/20150510150314072)

#### SMP

SMP技术就是对称多处理结构，这种结构的最大特点就是CPU共享所有资源，比如总线，内存，IO系统等等。缺点是扩展性有限，支持的CPU数量个数有限。

![img](https://ask.qcloudimg.com/http-save/yehe-3636591/gg51emxmb8.png?imageView2/2/w/1620)

#### NUMA

NUMA模式，每个CPU都有雨自己的存储器，每个处理器也可以访问其他CPU的存储器。访问自己的存储器要比访问别的存储器快很多。

![img](https://ask.qcloudimg.com/http-save/yehe-3636591/tmsyzpq6jk.png?imageView2/2/w/1620)

#### CPU过载

KVM允许客户机过载使用物理资源，分配的CPU和内存数量多于物理实际的资源。QEMU会启动更多的线程来为客户机提供服务，这些线程也是被Linux内核调度运行在物理CPU硬件上。

最推荐的做法是对多个单CPU的客户机使用over-commit，比如：在拥有4个逻辑CPU的宿主机中，同时运行多于4个（如8个、16个）客户机，其中每个客户机都被分配一个vCPU。这时，如果每个宿主机的负载不很大的情况下，宿主机Linux对每个客户机的调度是非常有效的，这样的过载使用并不会带来客户机中的性能损失。

最不推荐的做法是让某一个客户机的vCPU数量超过物理CPU数量。

#### 亲和性

进程的处理器亲和性，即是CPU的绑定设置，是指将进程绑定到特定的一个或多个CPU上去执行，而不允许调度到其他的CPU上。好处是可以减少进程在多个CPU之间交换运行带来的缓存命中失效，带来一定程度上的性能提升。但是破坏各个CPU的负载均衡。

每个虚拟机的vcpu在宿主机上，都表现为一个普通的qemu线程，可以使用taskset工具对其进行设置处理亲和性，使其绑定到某一个或几个固定的cpu上去调度。尽管Linux内核的进程调度算法已经非常高效了，在多数情况下是不需要对进程的调度进行干预，不过，在虚拟化环境中，有时有必要将客户机的qemu进程或线程绑定到固定的cpu上执行。

### 内存虚拟化

通过内存虚拟化共享物理系统内存，动态分配给虚拟机。

KVM 实现客户机内存的方式是，利用mmap系统调用，在QEMU主线程的虚拟地址空间中申明一段连续的大小的空间用于客户机物理内存映射。

在有两个虚机的情况下，情形是这样的：

![img](https://ask.qcloudimg.com/http-save/yehe-1220175/p19q9pgt21.jpeg?imageView2/2/w/1620)

KVM 为了在一台机器上运行多个虚拟机，需要增加一个新的内存虚拟化层，也就是说，必须虚拟 MMU 来支持客户操作系统，来实现 VA -> PA -> MA 的翻译。

#### EPT

EPT(扩展页表)，作为CPU中新的一层，用来将客户机的物理地址翻译为主机的物理地址。![img](https://ask.qcloudimg.com/http-save/yehe-1220175/kup99hdvof.jpeg?imageView2/2/w/1620)

它的两阶段记忆体转换，特点就是将 Guest Physical Address → System Physical Address

#### 影子页表

KVM需要为每个客户机的每个进程的页表都要维护一套相应的影子页表， 这会带来较大内存上的额外开销，此外，客户机页表和和影子页表的同步也比较复杂。

#### ![img](https://ask.qcloudimg.com/http-save/yehe-1220175/9y2k0t3pt.jpeg?imageView2/2/w/1620)

#### KSM

KSM 作为内核中的守护进程（称为 ksmd）存在，它定期执行页面扫描，识别副本页面并合并副本，释放这些页面以供它用。因此，在多个进程中，Linux将内核相似的内存页合并成一个内存页。减少多个相似的虚拟机内存占用，提高内存的使用效率。

合并过程

- 初始状态

![img](https://ask.qcloudimg.com/http-save/yehe-1220175/ceklctyv2m.jpeg?imageView2/2/w/1620)

- 合并后

  ![img](https://ask.qcloudimg.com/http-save/yehe-1220175/or5sqz41vz.jpeg?imageView2/2/w/1620)

- Guest 1 写内存后

  ![img](https://ask.qcloudimg.com/http-save/yehe-1220175/pjdceucrad.jpeg?imageView2/2/w/1620)

#### Huge Page

 HugePage，就是指的大页内存管理方式。与传统的4kb的普通页管理方式相比，HugePage为管理大内存(8GB以上)更为高效。

- Regular Pages

图中物理地址page size大小4kb。也可以看到进程1和进程2在system-wide table中都指向了page2，也就是同一个物理地址。

![img](https://ask.qcloudimg.com/http-save/yehe-2746053/u6l5orbntb.jpeg?imageView2/2/w/1620)

- Huge Pages

  page table中的任意一个page可能使用了常规的page，
  也有可能使用了huge page。同样进程1和进程2都共享了其中的Hpage2。图中的物理内存常规的page size是4kb，huge page size 是4mb。

  ![img](https://ask.qcloudimg.com/http-save/yehe-2746053/3dqn40zdgm.jpeg?imageView2/2/w/1620)

#### 内存过载

有如下三种方式来实现内存的过载使用

- 内存交换：用交换空间来弥补内存的不足
- 内存气球：通过virio_ballon驱动来实现宿主机和客户机的协作
- 页共享：通过KSM合并多个客户机进程使用的相同内存页

### 存储配置

QEMU提供了对多块存储设备的模拟，包括IDE，SCSI，软盘，U盘，virtio磁盘等。而且对设备的启动顺序提供了灵活的配置。

#### QEMU支持镜像

qemu-img支持非常多种的文件格式，包括：cow,ftps,https,http,dmg,qcow,qcow2,raw,sheepdog,vdi,vpc等

qcow2:是QEMU目前推荐的镜像格式，支持稀疏文件以节省存储空间，支持可选的AES加密以提高镜像文件安全性，支持基于zlib的压缩，支持在一个镜像文件中有多个虚拟机快照。

#### 镜像存储方式

客户机镜像存储文件，其中几种：

- 本地存储的客户机镜像文件
- 物理磁盘或磁盘分区
- LVM，逻辑分区
- NFS，网络文件系统
- iSCSI，基于Internet的小型计算机系统接口
- 本地或光纤通道连接的LUN
- GFS2

### 网络配置

#### QEMU支持的网络模式

- 基于网桥的虚拟网卡：可以让客户机和宿主机共享一个物理网络设备连接网络，客户机有独立的IP
- 基于NAT的虚拟网络：将内网地址转换为外网的合法IP地址，被广泛应用于各种类型的Internet接入方式
- QEMU内置的用户模式网络：完全由QEMU自身实现，使用Slirp实现一整套TCP/IP协议栈，并使用这个协议栈实现一套虚拟的NAT网络
- 直接分配网络设备的网络(VT-d,SR-IOV)

### 图形显示




























































