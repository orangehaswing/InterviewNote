# libvirt XML 格式

## 元素和属性

```xml
<domain type='kvm' id='1'>
  <name>MyGuest</name>
  <uuid>4dea22b3-1d52-d8f3-2516-782e98ab3fa0</uuid>
  <genid>43dc0cf8-809b-4adb-9bea-a9abb5f3d90e</genid>
  <title>A short description - title - of the domain</title>
  <description>Some human readable description</description>
  <metadata>
    <app1:foo xmlns:app1="http://app1.org/app1/">..</app1:foo>
    <app2:bar xmlns:app2="http://app1.org/app2/">..</app2:bar>
  </metadata>
  ...
```

domain：命名所有虚拟机所需的根元素`domain`。

它有两个属性， `type` 指定用于运行域的管理程序。允许的值是特定于驱动程序的，但包括“xen”，“kvm”，“qemu”和“lxc”。第二个属性是`id`运行的客户机的唯一整数标识符。非活动计算机没有id值。

name：虚拟机名称

uuid：全局唯一标识符

genid：全局唯一标识符（GUID），该值用于在虚拟机重新执行之前已执行的操作时帮助通知客户机操作系统，例如：

- VM开始执行快照
- VM从备份中恢复
- VM是灾难恢复环境中的故障​​转移
- 导入，复制或克隆VM

## 操作系统启动

```xml
...
<os firmware='efi'>
  <type>hvm</type>
  <loader readonly='yes' secure='no' type='rom'>/usr/lib/xen/boot/hvmloader</loader>
  <nvram template='/usr/share/OVMF/OVMF_VARS.fd'>/var/lib/libvirt/nvram/guest_VARS.fd</nvram>
  <boot dev='hd'/>
  <boot dev='cdrom'/>
  <bootmenu enable='yes' timeout='3000'/>
  <smbios mode='sysinfo'/>
  <bios useserial='yes' rebootTimeout='0'/>
</os>
...
```

os firmware:启动时，接受的值是 `bios`和`efi`。选择过程扫描描述指定位置的已安装固件映像的文件

type：type元素的内容指定要在虚拟机中引导的操作系统的类型。

loader：指定UEFI固件，可以选择 readonly 和 secure 两个参数

boot：该`dev`属性采用值“fd”，“hd”，“cdrom”或“network”之一，用于指定要考虑的下一个引导设备。该`boot`元素可以重复多次，以设置启动设备的优先级列表，以便依次尝试。

## 内核启动

安装新的guest虚拟机操作系统时，直接从存储在主机操作系统中的内核和initrd启动。

```xml
...
<os>
  <type>hvm</type>
  <loader>/usr/lib/xen/boot/hvmloader</loader>
  <kernel>/root/f8-i386-vmlinuz</kernel>
  <initrd>/root/f8-i386-initrd</initrd>
  <cmdline>console=ttyS0 ks=http://example.com/f8-i386/os/</cmdline>
  <dtb>/root/ppc.dtb</dtb>
  <acpi>
    <table type='slic'>/path/to/slic.dat</table>
  </acpi>
</os>
...
```

`kernel` 此元素的内容指定主机OS中内核映像的完全限定路径。

`initrd` 此元素的内容指定主机操作系统中（可选）ramdisk映像的完全限定路径。

`dtb` 此元素的内容指定主机OS中（可选）设备树二进制（dtb）映像的完全限定路径。

`cmdline` 此元素的内容指定在引导时传递给内核（或安装程序）的参数。

`acpi` 该`table`元素包含ACPI表的完全限定路径。该`type`属性包含ACPI表类型（目前仅`slic`支持）

## 容器启动

使用基于容器的虚拟化，而不是内核/启动映像启动域时，需要使用该`init`元素指向init二进制文件的路径。

```xml
<os>
  <type arch='x86_64'>exe</type>
  <init>/bin/systemd</init>
  <initarg>--unit</initarg>
  <initarg>emergency.service</initarg>
  <initenv name='MYENV'>some value</initenv>
  <initdir>/my/custom/cwd</initdir>
  <inituser>tester</inituser>
  <initgroup>1000</initgroup>
</os>
```

## CPU分配 

```xml
<domain>
  ...
  <vcpu placement='static' cpuset="1-4,^3,6" current="1">2</vcpu>
  <vcpus>
    <vcpu id='0' enabled='yes' hotpluggable='no' order='1'/>
    <vcpu id='1' enabled='no' hotpluggable='yes'/>
  </vcpus>
  ...
</domain>
```

`vcpu` guest虚拟机操作系统分配的最大虚拟 CPU 数，必须介于1和虚拟机管理程序支持的最大值之间。

​	`cpuset ` 绑定域进程，虚拟 CPU 和物理 CPU 关系

​	`current` 可选属性`current`可用于指定是否应启用少于最大数量的虚拟CPU。 

​	`placement` 域进程的CPU放置模式。 值可以是 "static" 或 "auto"

`vcpus` vcpus元素允许控制各个vCPU的状态。

## IOThreads  线程分配

IOThreads是专用的事件循环线程，用于支持的磁盘设备执行块I / O请求

```xml
<domain>
  ...
  <iothreads>4</iothreads>
  ...
</domain>
```

```xml
<domain>
  ...
  <iothreadids>
    <iothread id="2"/>
    <iothread id="4"/>
    <iothread id="6"/>
    <iothread id="8"/>
  </iothreadids>
  ...
</domain>
```

`iothreads` 定义要分配给域以供受支持的目标存储设备使用的IOThread数。每个主机CPU应该只有1或2个IOThread。每个IOThread可能分配了多个受支持的设备。

`iothreadids` 提供了专门为域定义IOThread ID的功能。

## CPU调整

```xml
<domain>
  ...
  <cputune>
    <vcpupin vcpu="0" cpuset="1-4,^2"/>
    <vcpupin vcpu="1" cpuset="0,1"/>
    <vcpupin vcpu="2" cpuset="2,3"/>
    <vcpupin vcpu="3" cpuset="0,4"/>
    <emulatorpin cpuset="1-3"/>
    <iothreadpin iothread="1" cpuset="5,6"/>
    <iothreadpin iothread="2" cpuset="7,8"/>
    <shares>2048</shares>
    <period>1000000</period>
    <quota>-1</quota>
    <global_period>1000000</global_period>
    <global_quota>-1</global_quota>
    <emulator_period>1000000</emulator_period>
    <emulator_quota>-1</emulator_quota>
    <iothread_period>1000000</iothread_period>
    <iothread_quota>-1</iothread_quota>
    <vcpusched vcpus='0-4,^3' scheduler='fifo' priority='1'/>
    <iothreadsched iothreads='2' scheduler='batch'/>
    <cachetune vcpus='0-3'>
      <cache id='0' level='3' type='both' size='3' unit='MiB'/>
      <cache id='1' level='3' type='both' size='3' unit='MiB'/>
      <monitor level='3' vcpus='1'/>
      <monitor level='3' vcpus='0-3'/>
    </cachetune>
    <cachetune vcpus='4-5'>
      <monitor level='3' vcpus='4'/>
      <monitor level='3' vcpus='5'/>
    </cachetune>
    <memorytune vcpus='0-3'>
      <node id='0' bandwidth='60'/>
    </memorytune>

  </cputune>
  ...
</domain>
```

`cputune` 可选`cputune`元素

`vcpupin` 指定域vCPU将固定到哪个主机的物理CPU。

## 内存分配

```xml
<domain>
  ...
  <maxMemory slots='16' unit='KiB'>1524288</maxMemory>
  <memory unit='KiB'>524288</memory>
  <currentMemory unit='KiB'>524288</currentMemory>
  ...
</domain>
```

`memory` 启动时guest虚拟机的最大内存分配。内存分配包括在启动时指定的可能的附加内存设备或稍后的热插拔。

`maxMemory` guest虚拟机的运行时最大内存分配。通过将内存热插入到此元素指定的限制，可以增加元素或NUMA单元大小配置指定的初始内存。

`currentMemory` 为 guest 实际分配内存。此值可能小于最大分配，以允许动态增加 guest 虚拟机内存。

## 内存支持

```xml
<域>
  ...
  <memoryBacking>
    <大页面>
      <page size =“1”unit =“G”nodeset =“0-3,5”/>
      <page size =“2”unit =“M”nodeset =“4”/>
    </大页面>
    <nosharepages />
    <锁定/>
    <source type =“file | anonymous | memfd”/>
    <access mode =“shared | private”/>
    <allocation mode =“immediate | ondemand”/>
    <丢弃/>
  </ memoryBacking>
  ...
</域>
```

`hugepages` 虚拟机应使用hugepages而不是正常的本机页面大小分配其内存。

`nosharepages` 指示管理程序禁用此域的共享页面（内存合并，KSM）。

`locked` 当虚拟机管理程序设置和支持时，属于该域的内存页将被锁定在主机的内存中，并且不允许主机将它们交换出来，这可能是某些工作负载（如实时）所必需的。

`access` 使用该`mode`属性，指定内存是“共享”还是“私有”。

`allocation` 使用该`mode`属性，通过提供“immediate”或“ondemand”指定何时分配内存。

## 内存调整

```xml
<domain>
  ...
  <memtune>
    <hard_limit unit='G'>1</hard_limit>
    <soft_limit unit='M'>128</soft_limit>
    <swap_hard_limit unit='G'>2</swap_hard_limit>
    <min_guarantee unit='bytes'>67108864</min_guarantee>
  </memtune>
  ...
</domain>
```

`memtune` 提供有关域的内存可调参数的详细信息。

`hard_limit` 虚拟机可以使用的最大内存。

`soft_limit` 是在内存争用期间强制执行的内存限制。

`swap_hard_limit` 是guest虚拟机可以使用的最大内存加交换。

`min_guarantee` 是guest虚拟机的保证最小内存分配。

## NUMA节点调整

```xml
<domain>
  ...
  <numatune>
    <memory mode="strict" nodeset="1-4,^3"/>
    <memnode cellid="0" mode="strict" nodeset="1"/>
    <memnode cellid="2" mode="preferred" nodeset="2"/>
  </numatune>
  ...
</domain>
```

## Block I/O 调整

```xml
<domain>
  ...
  <blkiotune>
    <weight>800</weight>
    <device>
      <path>/dev/sda</path>
      <weight>1000</weight>
    </device>
    <device>
      <path>/dev/sdb</path>
      <weight>500</weight>
      <read_bytes_sec>10000</read_bytes_sec>
      <write_bytes_sec>10000</write_bytes_sec>
      <read_iops_sec>20000</read_iops_sec>
      <write_iops_sec>20000</write_iops_sec>
    </device>
  </blkiotune>
  ...
</domain>
```

## Resource partitioning

```xml
...
<resource>
  <partition>/virtualmachines/production</partition>
</resource>
...
```

## CPU模型和拓扑

可以使用以下元素集合指定CPU模型，其功能和拓扑的要求。

```
...
<cpu match='exact'>
  <model fallback='allow'>core2duo</model>
  <vendor>Intel</vendor>
  <topology sockets='1' cores='2' threads='1'/>
  <cache level='3' mode='emulate'/>
  <feature policy='disable' name='lahf_lm'/>
</cpu>
...
```

```
<cpu mode='host-model'>
  <model fallback='forbid'/>
  <topology sockets='1' cores='2' threads='1'/>
</cpu>
...
```

```
<cpu mode='host-passthrough'>
  <cache mode='passthrough'/>
  <feature policy='disable' name='lahf_lm'/>
...
```

如果不需要对CPU模型及其功能进行限制，`cpu`可以使用更简单的元素。

```
...
<cpu>
  <topology sockets='1' cores='2' threads='1'/>
</cpu>
...
```

## 事件配置

有时需要覆盖对各种事件采取的默认操作。

```
...
<on_poweroff>destroy</on_poweroff>
<on_reboot>restart</on_reboot>
<on_crash>restart</on_crash>
<on_lockfailure>poweroff</on_lockfailure>
...
```

## 电源管理

```
...
<pm>
  <suspend-to-disk enabled='no'/>
  <suspend-to-mem enabled='yes'/>
</pm>
...
```

可以强制启用或禁用BIOS

## Hypervisor features

```xml
...
<features>
  <pae/>
  <acpi/>
  <apic/>
  <hap/>
  <privnet/>
  <hyperv>
    <relaxed state='on'/>
    <vapic state='on'/>
    <spinlocks state='on' retries='4096'/>
    <vpindex state='on'/>
    <runtime state='on'/>
    <synic state='on'/>
    <stimer state='on'>
      <direct state='on'/>
    </stimer>
    <reset state='on'/>
    <vendor_id state='on' value='KVM Hv'/>
    <frequencies state='on'/>
    <reenlightenment state='on'/>
    <tlbflush state='on'/>
    <ipi state='on'/>
    <evmcs state='on'/>
  </hyperv>
  <kvm>
    <hidden state='on'/>
    <hint-dedicated state='on'/>
  </kvm>
  <pvspinlock state='on'/>
  <gic version='2'/>
  <ioapic driver='qemu'/>
  <hpt resizing='required'>
    <maxpagesize unit='MiB'>16</maxpagesize>
  </hpt>
  <vmcoreinfo state='on'/>
  <smm state='on'>
    <tseg unit='MiB'>48</tseg>
  </smm>
  <htm state='on'/>
  <msrs unknown='ignore'/>
</features>
...
```

## Time keeping

大多数操作系统都希望硬件时钟保持在UTC状态，这是默认设置。然而，Windows希望它处于所谓的“本地时间”。

```xml
...
<clock offset='localtime'>
  <timer name='rtc' tickpolicy='catchup' track='guest'>
    <catchup threshold='123' slew='120' limit='10000'/>
  </timer>
  <timer name='pit' tickpolicy='delay'/>
</clock>
...
```

## Performance monitoring events

某些平台允许监视虚拟机的性能和内部执行的代码。

```xml
...
<perf>
  <event name='cmt' enabled='yes'/>
  <event name='mbmt' enabled='no'/>
  <event name='mbml' enabled='yes'/>
  <event name='cpu_cycles' enabled='no'/>
  <event name='instructions' enabled='yes'/>
  <event name='cache_references' enabled='no'/>
  <event name='cache_misses' enabled='no'/>
  <event name='branch_instructions' enabled='no'/>
  <event name='branch_misses' enabled='no'/>
  <event name='bus_cycles' enabled='no'/>
  <event name='stalled_cycles_frontend' enabled='no'/>
  <event name='stalled_cycles_backend' enabled='no'/>
  <event name='ref_cpu_cycles' enabled='no'/>
  <event name='cpu_clock' enabled='no'/>
  <event name='task_clock' enabled='no'/>
  <event name='page_faults' enabled='no'/>
  <event name='context_switches' enabled='no'/>
  <event name='cpu_migrations' enabled='no'/>
  <event name='page_faults_min' enabled='no'/>
  <event name='page_faults_maj' enabled='no'/>
  <event name='alignment_faults' enabled='no'/>
  <event name='emulation_faults' enabled='no'/>
</perf>
...
```

## 设备

```
...
<devices>
  <emulator>/usr/lib/xen/bin/qemu-dm</emulator>
</devices>
...
```

`emulator` 指定设备模型仿真器二进制文件的完全限定路径。

## 硬盘，软盘，CD-ROM

```xml
...
<devices>
  <disk type='file' snapshot='external'>
    <driver name="tap" type="aio" cache="default"/>
    <source file='/var/lib/xen/images/fv0' startupPolicy='optional'>
      <seclabel relabel='no'/>
    </source>
    <target dev='hda' bus='ide'/>
    <iotune>
      <total_bytes_sec>10000000</total_bytes_sec>
      <read_iops_sec>400000</read_iops_sec>
      <write_iops_sec>100000</write_iops_sec>
    </iotune>
    <boot order='2'/>
    <encryption type='...'>
      ...
    </encryption>
    <shareable/>
    <serial>
      ...
    </serial>
  </disk>
    ...
  <disk type='network'>
    <driver name="qemu" type="raw" io="threads" ioeventfd="on" event_idx="off"/>
    <source protocol="sheepdog" name="image_name">
      <host name="hostname" port="7000"/>
    </source>
    <target dev="hdb" bus="ide"/>
    <boot order='1'/>
    <transient/>
    <address type='drive' controller='0' bus='1' unit='0'/>
  </disk>
  <disk type='network'>
    <driver name="qemu" type="raw"/>
    <source protocol="rbd" name="image_name2">
      <host name="hostname" port="7000"/>
      <snapshot name="snapname"/>
      <config file="/path/to/file"/>
      <auth username='myuser'>
        <secret type='ceph' usage='mypassid'/>
      </auth>
    </source>
    <target dev="hdc" bus="ide"/>
  </disk>
  <disk type='block' device='cdrom'>
    <driver name='qemu' type='raw'/>
    <target dev='hdd' bus='ide' tray='open'/>
    <readonly/>
  </disk>
  <disk type='network' device='cdrom'>
    <driver name='qemu' type='raw'/>
    <source protocol="http" name="url_path">
      <host name="hostname" port="80"/>
    </source>
    <target dev='hde' bus='ide' tray='open'/>
    <readonly/>
  </disk>
  <disk type='network' device='cdrom'>
    <driver name='qemu' type='raw'/>
    <source protocol="https" name="url_path">
      <host name="hostname" port="443"/>
    </source>
    <target dev='hdf' bus='ide' tray='open'/>
    <readonly/>
  </disk>
  <disk type='network' device='cdrom'>
    <driver name='qemu' type='raw'/>
    <source protocol="ftp" name="url_path">
      <host name="hostname" port="21"/>
    </source>
    <target dev='hdg' bus='ide' tray='open'/>
    <readonly/>
  </disk>
  <disk type='network' device='cdrom'>
    <driver name='qemu' type='raw'/>
    <source protocol="ftps" name="url_path">
      <host name="hostname" port="990"/>
    </source>
    <target dev='hdh' bus='ide' tray='open'/>
    <readonly/>
  </disk>
  <disk type='network' device='cdrom'>
    <driver name='qemu' type='raw'/>
    <source protocol="tftp" name="url_path">
      <host name="hostname" port="69"/>
    </source>
    <target dev='hdi' bus='ide' tray='open'/>
    <readonly/>
  </disk>
  <disk type='block' device='lun'>
    <driver name='qemu' type='raw'/>
    <source dev='/dev/sda'>
      <reservations managed='no'>
        <source type='unix' path='/path/to/qemu-pr-helper' mode='client'/>
      </reservations>
    <target dev='sda' bus='scsi'/>
    <address type='drive' controller='0' bus='0' target='3' unit='0'/>
  </disk>
  <disk type='block' device='disk'>
    <driver name='qemu' type='raw'/>
    <source dev='/dev/sda'/>
    <geometry cyls='16383' heads='16' secs='63' trans='lba'/>
    <blockio logical_block_size='512' physical_block_size='4096'/>
    <target dev='hdj' bus='ide'/>
  </disk>
  <disk type='volume' device='disk'>
    <driver name='qemu' type='raw'/>
    <source pool='blk-pool0' volume='blk-pool0-vol0'/>
    <target dev='hdk' bus='ide'/>
  </disk>
  <disk type='network' device='disk'>
    <driver name='qemu' type='raw'/>
    <source protocol='iscsi' name='iqn.2013-07.com.example:iscsi-nopool/2'>
      <host name='example.com' port='3260'/>
      <auth username='myuser'>
        <secret type='iscsi' usage='libvirtiscsi'/>
      </auth>
    </source>
    <target dev='vda' bus='virtio'/>
  </disk>
  <disk type='network' device='lun'>
    <driver name='qemu' type='raw'/>
    <source protocol='iscsi' name='iqn.2013-07.com.example:iscsi-nopool/1'>
      <host name='example.com' port='3260'/>
      <auth username='myuser'>
        <secret type='iscsi' usage='libvirtiscsi'/>
      </auth>
    </source>
    <target dev='sdb' bus='scsi'/>
  </disk>
  <disk type='network' device='lun'>
    <driver name='qemu' type='raw'/>
    <source protocol='iscsi' name='iqn.2013-07.com.example:iscsi-nopool/0'>
      <host name='example.com' port='3260'/>
      <initiator>
        <iqn name='iqn.2013-07.com.example:client'/>
      </initiator>
    </source>
    <target dev='sdb' bus='scsi'/>
  </disk>
  <disk type='volume' device='disk'>
    <driver name='qemu' type='raw'/>
    <source pool='iscsi-pool' volume='unit:0:0:1' mode='host'/>
    <target dev='vdb' bus='virtio'/>
  </disk>
  <disk type='volume' device='disk'>
    <driver name='qemu' type='raw'/>
    <source pool='iscsi-pool' volume='unit:0:0:2' mode='direct'/>
    <target dev='vdc' bus='virtio'/>
  </disk>
  <disk type='file' device='disk'>
    <driver name='qemu' type='qcow2' queues='4'/>
    <source file='/var/lib/libvirt/images/domain.qcow'/>
    <backingStore type='file'>
      <format type='qcow2'/>
      <source file='/var/lib/libvirt/images/snapshot.qcow'/>
      <backingStore type='block'>
        <format type='raw'/>
        <source dev='/dev/mapper/base'/>
        <backingStore/>
      </backingStore>
    </backingStore>
    <target dev='vdd' bus='virtio'/>
  </disk>
</devices>
...
```

## Filesystems

```xml
...
<devices>
  <filesystem type='template'>
    <source name='my-vm-template'/>
    <target dir='/'/>
  </filesystem>
  <filesystem type='mount' accessmode='passthrough'>
    <driver type='path' wrpolicy='immediate'/>
    <source dir='/export/to/guest'/>
    <target dir='/import/from/host'/>
    <readonly/>
  </filesystem>
  <filesystem type='file' accessmode='passthrough'>
    <driver name='loop' type='raw'/>
    <driver type='path' wrpolicy='immediate'/>
    <source file='/export/to/guest.img'/>
    <target dir='/import/from/host'/>
    <readonly/>
  </filesystem>
  ...
</devices>
...
```

主机上可以直接从guest虚拟机访问的目录。

## 设备地址

## Virtio相关选项

## 控制器 

## 主机设备分配 

```xml
...
<devices>
  <hostdev mode='subsystem' type='usb'>
    <source startupPolicy='optional'>
      <vendor id='0x1234'/>
      <product id='0xbeef'/>
    </source>
    <boot order='2'/>
  </hostdev>
</devices>
...
```

## 网络接口

```xml
...
<devices>
  <interface type='direct' trustGuestRxFilters='yes'>
    <source dev='eth0'/>
    <mac address='52:54:00:5d:c7:9e'/>
    <boot order='1'/>
    <rom bar='off'/>
  </interface>
</devices>
...
```

















