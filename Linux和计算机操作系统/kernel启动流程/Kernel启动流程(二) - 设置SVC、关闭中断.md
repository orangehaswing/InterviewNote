# Kernel启动流程(二) - 设置SVC、关闭中断

# 零. 说明

## 1. kernel启动流程第一阶段简单说明

arch/arm/kernel/head.S

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

  本文要介绍的是“设置为SVC模式，关闭所有中断”的部分。分成两部分说明，设置为SVC模式，关闭所有中断。

## 2. 疑问

主要带着以下几个问题去理解

- 设置SVC模式 
  - 什么是SVC模式？
  - 为什么要设置成SVC模式？
  - 如何设置成SVC模式
- 关闭所有中断 
  - 为什么要关闭所有中断？
  - 如何关闭所有中断？ 

对应代码

```
ENTRY(stext)
    @ ensure svc mode and all interrupts masked
    safe_svcmode_maskall r9
```

safe_svcmode_maskall实现代码如下： 
arch/arm/include/asm/assembler.h

```
.macro safe_svcmode_maskall reg:req
#if __LINUX_ARM_ARCH__ >= 6 && !defined(CONFIG_CPU_V7M)
    mrs \reg , cpsr
    eor \reg, \reg, #HYP_MODE
    tst \reg, #MODE_MASK
    bic \reg , \reg , #MODE_MASK
    orr \reg , \reg , #PSR_I_BIT | PSR_F_BIT | SVC_MODE
THUMB(  orr \reg , \reg , #PSR_T_BIT    )
    bne 1f
    orr \reg, \reg, #PSR_A_BIT
    badr    lr, 2f
    msr spsr_cxsf, \reg
    __MSR_ELR_HYP(14)
    __ERET
1:  msr cpsr_c, \reg
2:
```

这部分代码也就是后面要讲述的“设置为SVC模式，关闭所有中断”的核心

# 一. 设置为SVC模式

## 1. ARM7体系的几种工作模式（什么是SVC模式？）

ARM7体系的cpu有如下七种工作模式。

* 用户模式（USR） 
  普通模式 
  正常程序工作模式，此模式下程序不能够访问一些受操作系统保护的系统资源，应用程序也不能直接进行处理器模式的切换。 
* 系统模式（SYS） 
  特权模式 
  操作系统特权任务模式，用于支持操作系统的特权任务等，可以访问系统保护的系统资源，也可以直接切换到其它模式等特权。 
* 管理模式（SVC） 
  特权模式、异常模式 
  操作系统保护模式 
* 快中断模式（FIQ） 
  特权模式、异常模式 
  支持高速数据传输及通道处理，FIQ异常响应时进入此模式。 
* 中断模式（IRQ） 
  特权模式、异常模式 
  用于通用中断处理，IRQ异常响应时进入此模式。 
* 中止模式（ABT） 
  特权模式、异常模式 
  用于支持虚拟内存和/或存储器保护。 
* 未定义模式（UND） 
  特权模式、异常模式 
  支持硬件协处理器的软件仿真，未定义指令异常响应时进入此模式。

## 2. 为什么要设置成SVC模式？

除了用户模式之外的其他6种处理器模式称为特权模式。 
特权模式下，程序可以访问所有的系统资源（除了特定模式下的影子寄存器），也可以任意地进行处理器模式的切换。 
特权模式中，除系统模式外，其他5种模式又称为异常模式。 

* 用户模式下访问的资源受限，故不能使用用户模式 
* 系统模式的优先级低于异常模式，故不使用系统模式 
* 快中断模式、中断模式、中止模式、未定义模式用于特殊场景下由CPU自动切入，故不使用 
  所以需要使用SVC模式。

## 3. 如何设置成SVC模式？

（1）CPSR & SPSR 

ARM工作模式的切换由CPSR寄存器控制。 CPSR表示当前程序状态寄存器，SPSR表示备份的程序状态寄存器。 

CPSR:程序状态寄存器(current program status register) (当前程序状态寄存器)，在任何处理器模式下被访问。它包含了条件标志位、中断禁止位、当前处理器模式标志以及其他的一些控制和状态位。如下所示： 

* 条件标志位（这里可以暂时不关心）

  |  对应位  |  对应标识  |               功能说明                |      |      |      |
  | :---: | :----: | :-------------------------------: | ---- | ---- | ---- |
  | 31bit | N flag | 用两个补码表示的带符号的运算结果标识，为1表示负数，为0表示非负数 |      |      |      |
  | 30bit | Z flag |      为1表示运算结果为0，为0表示运算结果为非0       |      |      |      |
  | 29bit | C flag |            加法运算结果进位标志位            |      |      |      |
  | 28bit | V flag |          加减法运算指令符号位溢出标志位          |      |      |      |

* 控制位

  |  对应位  |  对应标识  |            功能说明             |      |      |      |
  | :---: | :----: | :-------------------------: | ---- | ---- | ---- |
  | 7bit  |  flag  |        为1表示禁止IRQ中断。         |      |      |      |
  | 6bit  | F flag |        为1表示禁止FIQ中断。         |      |      |      |
  | 5bit  | T flag | 为1表示程序运行于Thumb状态，否则处于ARM状态。 |      |      |      |
  | [4:0] | 模式控制位  |       用于配置CPU当前的工作模式。       |      |      |      |

（2）工作模式编码 
七种工作模式对应在CPSR中的工作模式编码如下：

| 工作模式 | 工作模式编码 |
| :--: | :----: |
| USR  | 10000  |
| FIQ  | 10001  |
| IRQ  | 10010  |
| SVC  | 10011  |
| ABT  | 10111  |
| UND  | 11011  |
| SYS  | 11111  |

综上，需要将CPSR的[4:0]设置成10011就可以切换到SVC模式。

# 二. 关闭所有中断

## 1. 为什么要关闭所有中断？

在启动过程中，中断环境并没有完全准备好，也就是中断向量表和中断处理函数并没有完成设置，一旦有中断产生，可能会导致预想不到的问题，或者是程序跑飞。因此，在准备好中断环境之前，需要关闭所有中断。

## 2. 如何关闭所有中断？

中断的关闭同样由CPSR寄存器来进行控制

| 对应位  |  对应标识  |     功能说明     |
| :--: | :----: | :----------: |
| 7bit |  flag  | 为1表示禁止IRQ中断。 |
| 6bit | F flag | 为1表示禁止FIQ中断。 |

综上，需要将CPSR的bit6和bit7设置为1。

# 三. 代码分析

设置为SVC模式和关闭所有中断都是在safe_svcmode_maskall中完成。

```
ENTRY(stext)
    @ ensure svc mode and all interrupts masked
    safe_svcmode_maskall r9
```

safe_svcmode_maskall实现和分析如下:

arch/arm/include/asm/assembler.h

```
.macro safe_svcmode_maskall reg:req
#if __LINUX_ARM_ARCH__ >= 6 && !defined(CONFIG_CPU_V7M)
    mrs \reg , cpsr                                  
    eor \reg, \reg, #HYP_MODE
    tst \reg, #MODE_MASK
    bic \reg , \reg , #MODE_MASK                                         
    orr \reg , \reg , #PSR_I_BIT | PSR_F_BIT | SVC_MODE  
THUMB(  orr \reg , \reg , #PSR_T_BIT    )
    bne 1f
    orr \reg, \reg, #PSR_A_BIT
    badr    lr, 2f
    msr spsr_cxsf, \reg                    
    __MSR_ELR_HYP(14)
    __ERET
1:  msr cpsr_c, \reg
2:
```

其中主要的关闭中断设置SVC的核心代码如下：

```
    mrs \reg , cpsr                                          @获取CPSR寄存器的值到临时寄存器中
    bic \reg , \reg , #MODE_MASK                                                @清除模式位[4:0]位
    orr \reg , \reg , #PSR_I_BIT | PSR_F_BIT | SVC_MODE        @设置BIT6\BIT7,关闭中断，设置[4:0]为SVC_MODE
1:  msr cpsr_c, \reg                @存储到CPSR的低八位中，CPSR_C表示CRSR寄存器的低8位，也就是控制域屏蔽字节。
```


其中 
arch/arm/include/uapi/asm/ptrace.h中

```
#define SVC_MODE    0x00000013
#define PSR_F_BIT   0x00000040
#define PSR_I_BIT   0x00000080
```

到这里就实现了工作模式的切换和中断的关闭。
