# kernel 启动流程(四) - dtb的验证

# 零. 说明

## 1. kernel启动流程第一阶段简单说明

`arch/arm/kernel/head.S`

- kernel入口地址对应stext

```
ENTRY(stext)
```

- 第一阶段要做的事情，也就是stext的实现内容
  - 设置为SVC模式，关闭所有中断
  - 获取CPU ID，提取相应的proc info
  - 验证tags或者dtb
  - 创建页表项
  - 配置r13寄存器，也就是设置打开MMU之后要跳转到的函数。
  - 使能MMU
- 跳转到start_kernel，也就是跳转到第二阶段

因为现在基本上很少使用atags，所以这里我们只说dtb的部分。

## 2. 疑问

主要带着以下几个问题去理解

- dtb是什么？为什么要验证dtb的合法性？
- 如何验证dtb的合法性？

3. 对应代码实现
--------------------- 
```
    __HEAD
ENTRY(stext)
    /*
     * r1 = machine no, r2 = atags or dtb,
     * r8 = phys_offset, r9 = cpuid, r10 = procinfo
     */
    bl  __vet_atags
```

# 一. DTB说明

这部分建议直接参考wowo的dtb的文章 

- [Device Tree（一）：背景介绍](http://www.wowotech.net/device_model/why-dt.html) 
  [Device Tree（二）：基本概念](http://www.wowotech.net/device_model/dt_basic_concept.html) 
  [Device Tree（三）：代码分析](http://www.wowotech.net/device_model/why-dt.html) 

简单说明，dtb里面存放了各种硬件信息，如果dtb有问题，会导致后续开机过程中读取的设备信息有问题而导致无法开机。

# 二. 如何验证了一个dtb是否合法

## 1. 原理说明 

在生成dtb的时候会在头部上添加一个幻数magic，而验证dtb是否合法主要也就是看这个dtb的magic是否和预期的值一致。 

2. dtb结构如下
--------------------- 
|         结构体如下         |
| :-------------------: |
|      DTB header       |
|     alignment gap     |
|  memory reserve map   |
|     alignment gap     |
| device-tree structure |
|     alignment gap     |
|  device-tree string   |

## 3. dtb header结构如下：

|     结构体如下      |
| :------------: |
|     magic      |
|   totalsize    |
| off_dt_struct  |
| off_dt_strings |
| off_mem_rsvmap |
|    version     |
|       ……       |

其中，magic是一个固定的值，0xd00dfeed（大端）或者0xedfe0dd0（小端）。 

以s5pv210-tiny210.dtb为例： 执行”hexdump -C s5pv210-tiny210.dtb | more”命令

```
@:dts$ hexdump -C s5pv210-tiny210.dtb | more
00000000  d0 0d fe ed 00 00 5a cc  00 00 00 38 00 00 58 14  |......Z....8..X.|
00000010  00 00 00 28 00 00 00 11  00 00 00 10 00 00 00 00  |...(............|
```

可以看到dtb的前面4个字节就是0xd00dfeed，也就是magic。 

综上，我们只要提取待验证dtb的地址上的数据的前四个字节，与0xd00dfeed（大端）或者0xedfe0dd0（小端）进行比较，如果匹配的话，就说明对应待验证dtb就是一个合法的dtb。

# 三. 代码分析

具体就是分析__vet_atags的实现。 

r2上存放的是dtb的地址指针，而代码中所要做的，就是要通过这个地址指针，获取前四个字节，去和dtb应有的幻数，也就是0xd00dfeed（大端）或者0xedfe0dd0（小端）进行比较。匹配的话，则说明这是一个合法的dtb。 
代码如下(省略了验证atags的部分)： 

```
arch/arm/kernel/head-common.S

__vet_atags:
    tst    r2, #0x3            @ aligned?保证dtb的地址是四字节对齐的
    bne    1f

    ldr    r5, [r2, #0]    @获取dtb的前四个字节，存放在r5寄存器中
#ifdef CONFIG_OF_FLATTREE
    ldr    r6, =OF_DT_MAGIC    @ is it a DTB?，获取dtb的幻数，0xd00dfeed（大端）或者0xedfe0dd0（小端）
    cmp    r5, r6    @前四个字节和幻数进行对比
    beq    2f    @匹配，则说明是一个合法的dtb文件，跳到2
#endif
    bne    1f @不匹配，跳到1

2:    ret    lr                @ atag/dtb pointer is ok，直接返回，此时r2存放了dtb的地址

1:    mov    r2, #0@错误返回，此时，r2上是0
    ret    lr
ENDPROC(__vet_atags)
```

DTB的幻数，也就是OF_DT_MAGIC定义如下： 

```
arch/arm/kernel/head-common.S

#ifdef CONFIG_CPU_BIG_ENDIAN
#define OF_DT_MAGIC 0xd00dfeed
#else
#define OF_DT_MAGIC 0xedfe0dd0 /* 0xd00dfeed in big-endian */
#endif
```

综上，验证dtb的工作完成。
