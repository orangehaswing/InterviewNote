# 虚拟化 - CGroups

## CGroups简介

CGroup 是 Control Groups 的缩写，是Linux 内核提供的一种可以限制、记录、隔离进程组 (process groups) 所使用的物理资源 (如 cpu memory i/o等等)的机制，这种机制可以根据需求把一系列系统任务及其子任务整合（或分隔）到按资源划分等级的不同组内，从而为系统资源管理提供一个统一的框架。Libvirt将创建的虚拟机放到Cgroups 划分的资源区中，从而实现所有虚拟机、部分虚拟机、单个虚拟机的资源控制，例如控制虚拟机的磁盘IO带宽。容器也是利用CGroups 来限制cpu、io、内存、 网络等资源的。

## CGroups的四大功能

- 资源限制：cgroups 可以对任务是要的资源总额进行限制。比如设定任务运行时使用的内存上限，一旦超出就发 OOM。
- 优先级分配：通过分配的 CPU 时间片数量和磁盘 IO 带宽，实际上就等同于控制了任务运行的优先级。
- 资源统计：cgoups 可以统计系统的资源使用量，比如 CPU 使用时长、内存用量等。这个功能非常适合当前云端产品按使用量计费的方式。
- 任务控制：cgroups 可以对任务执行挂起、恢复等操作。

## CGroups的4个重要概念

- 任务（task）。在 Cgroups 中，任务就是系统的一个进程或线程，在 linux 系统中，内核本身的调度和管理并不对进程和线程进行区分，只是根据 clone 时传入的参数的不同来从概念上区分进程和线程；
- 控制组（Cgroup）。Cgroups 中的资源控制都是以Cgroup为单位实现。一个进程可以加入到某个Cgroup，也从一个进程组迁移到另一个Cgroup。一个Cgroup的进程可以使用 CGroups 以Cgroup为单位分配的资源，同时受到 CGroups 以 Cgroup 为单位设定的限制；
- 层级（hierarchy）。层级有一系列 cgroup 以一个树状结构排列而成，每个层级通过绑定对应的子系统进行资源控制。层级中的 cgroup 节点可以包含零个或多个子节点，子节点继承父节点挂载的子系统。一个操作系统中可以有多个层级
- 子系统（subsystem）。一个子系统就是一个资源控制器，比如 cpu 子系统就是控制 cpu 时间分配的一个控制器，内存子系统可以限制内存的使用量。子系统必须附加（attach）到一个层级上才能起作用，一个子系统附加到某个层级以后，这个层级上的所有控制族群都受到这个子系统的控制。

**CGroups和CGroup的区别**

- CGroups：cgroups是Linux内核中的一个机制，我们用它来作容器资源的限制等功能。
- CGroup：cgroup中文叫做控制组。它是cgroups实现资源控制的一个基本单位。cgroup表示按某种资源控制标准划分而成的一个任务组。它其中包含有一个或多个任务。

**4个重要概念之间的相互关系**

1. 每次在系统中创建新层级时，该系统中的所有任务都是那个层级的默认 cgroup（我们称之为  root cgroup，此 cgroup 在创建层级时自动创建，后面在该层级中创建的 cgroup 都是此 cgroup 的后代）的初始成员
2. 一个子系统最多只能附加到一个层级；
3. 一个层级可以附加多个子系统；
4. 一个任务可以是多个 cgroup 的成员，但是这些 cgroup 必须在不同的层级；
5. 系统中的进程（任务）创建子进程（任务）时，该子任务自动成为其父进程所在 cgroup 的成员。然后可根据需要将该子任务移动到不同的 cgroup 中，但开始时它总是继承其父任务的 cgroup。

**CGroups 层级图示例**

![img](https://upload-images.jianshu.io/upload_images/17870162-5d3f994b6b03c232.png?imageMogr2/auto-orient/strip|imageView2/2/w/704/format/webp)

如图所示的 CGroup 层级关系显示，CPU 和 Memory 两个子系统有自己独立的层级系统，而又通过 Task Group 取得关联关系。

## Cgroups子系统介绍

- cpuset：这个子系统为 cgroup 中的任务分配独立 CPU（在多核系统）和内存节点。
- cpu：     这个子系统使用调度程序提供对 CPU 的 cgroup 任务访问。
- cpuacct：这个子系统自动生成 cgroup 中任务所使用的 CPU 报告。
- blkio：     这个子系统为块设备设定I/O访问控制，比如物理设备（磁盘，固态硬盘，USB 等等）。
- memory：这个子系统设定 cgroup 中任务使用的内存限制，并自动生成由那些任务使用的内存资源报告。
- devices： 这个子系统可允许或者拒绝 cgroup 中的任务访问设备。
- freezer：  这个子系统挂起或者恢复 cgroup 中的任务。
- net_cls：  这个子系统使用等级识别符（classid）标记网络数据包，这让linux流量控制器（tc）可以识别来自特定cgroup任务的数据包，并进行网络限制。
- perf_event：这个子系统可以使cgroup中的任务进行统一的性能测试。

可以在/proc/cgroups查看当前内核支持的子系统

![img](https://upload-images.jianshu.io/upload_images/17870162-51e5f78d60a3391f.png?imageMogr2/auto-orient/strip|imageView2/2/w/406/format/webp)

## CGroups典型应用架构

![img](https://upload-images.jianshu.io/upload_images/17870162-bbbe2ff5491e1ae7.png?imageMogr2/auto-orient/strip|imageView2/2/w/855/format/webp)

如图所示，CGroups 技术可以被用来在操作系统底层限制物理资源，起到 Container 的作用。图中每一个 JVM 进程对应一个 Container Cgroup 层级，通过 CGroup 提供的各类子系统，可以对每一个 JVM 进程对应的线程级别进行物理限制，这些限制包括 CPU、内存等等许多种类的资源。

## CGroups的相关文件

CGroups 以文件的方式提供应用接口，我们可以通过 mount 命令来查看 cgroups 的挂载点：

![img](https://upload-images.jianshu.io/upload_images/17870162-b5ca29cf6bf1a951.png?imageMogr2/auto-orient/strip|imageView2/2/w/737/format/webp)

查看下cgroups的子系统：

![img](https://upload-images.jianshu.io/upload_images/17870162-8b75f7a086dd04d8.png?imageMogr2/auto-orient/strip|imageView2/2/w/675/format/webp)

查看下cpu子系统下的控制组：

![img](https://upload-images.jianshu.io/upload_images/17870162-ce618295e73c49aa.png?imageMogr2/auto-orient/strip|imageView2/2/w/1007/format/webp)

如图目录machine、system.slice、user.slice是现在已有的控制组。