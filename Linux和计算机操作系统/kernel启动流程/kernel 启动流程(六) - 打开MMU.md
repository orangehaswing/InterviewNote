# kernel 启动流程(六) - 打开MMU

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
  - 创建临时内核页表的页表项
  - 配置r13寄存器，也就是设置打开MMU之后要跳转到的函数。
  - 使能MMU
  - 跳转到start_kernel，也就是跳转到第二阶段

## 2. 疑问

主要带着以下几个问题去理解

- 如何打开MMU？
- 打开MMU前的一些寄存器设置？

# 一. MMU控制

MMU的配置操作都是通过操作CP15协处理器来实现的。

## 1. CP15协处理器寄存器说明

寄存器说明请参考《ARM的CP15协处理器的寄存器》。 

表格如下:

| 寄存器编号 |   基本作用    |   在 MMU 中的作用    |    在 PU 中的作用     |
| :---: | :-------: | :-------------: | :--------------: |
|  c0   | ID 编码（只读） | ID 编码和 cache 类型 |                  |
|  c1   | 控制位（可读写）  |      各种控制位      |                  |
|  c2   |  存储保护和控制  |    地址转换表基地址     | Cachability 的控制位 |
|  c3   |  存储保护和控制  |     域访问控制位      | Bufferablity 控制位 |
|  c4   |  存储保护和控制  |       保留        |        保留        |
|  c5   |  存储保护和控制  |     内存失效状态      |     访问权限控制位      |
|  c6   |  存储保护和控制  |     内存失效地址      |      保护区域控制      |
|  c7   | 高速缓存和写缓存  |   高速缓存和写缓存控制    |                  |
|  c8   |  存储保护和控制  |     TLB 控制      |        保留        |
|  c9   | 高速缓存和写缓存  |     高速缓存锁定      |                  |
|  c10  |  存储保护和控制  |     TLB 锁定      |        保留        |
|  c11  |    保留     |                 |                  |
|       |           |                 |                  |

**（1）c1，MMU的控制寄存器**

| bit  | 15   | 14   | 13   | 12   | 11   | 10   | 9    | 8    | 7    | 6    | 5    | 4    | 3    | 2    | 1    | 0    |
| :--: | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
| flag | L4   | R    | V    | I    | Z    | F    | R    | S    | B    | L    | D    | P    | W    | C    | A    | M    |

具体意义如下：

|  位   |                   说 明                    |
| :--: | :--------------------------------------: |
|  M   |      0：禁止 MMU 或者 PU； 1：使能 MMU 或者 PU      |
|  A   |          0：禁止地址对齐检查； 1：使能地址对齐检查          |
|  C   |     0：禁止数据/整个 cache； 1：使能数据/整个 cache     |
|  W   |             0：禁止写缓冲； 1：使能写缓冲             |
|  P   | 0：异常中断处理程序进入 32 位地址模式； 1：异常中断处理程序进入 26 位地址模式 |
|  D   |     0：禁止 26 位地址异常检查； 1：使能 26 位地址异常检查     |
|  L   |          0：选择早期中止模型； 1：选择后期中止模型          |
|  B   |     0： little endian； 1： big endian      |
|  S   |         在基于 MMU 的存储系统中，本位用作系统保护          |
|  R   |        在基于 MMU 的存储系统中，本位用作 ROM 保护        |
|  F   |                 0：由生产商定义                 |
|  Z   |          0：禁止跳转预测功能； 1：使能跳转预测指令          |
|  I   |        0：禁止指令 cache； 1：使能指令 cache        |
|  V   | 0：选择低端异常中断向量 0x0~0x1c； 1：选择高端异常中断向量 0xffff0000~ 0xffff001c |

**通过上述可知，c1的bit 0位用于控制MMU的开关。**

**（2）c2，MMU页表基地址存储器**

|   bit    |         功能         |
| :------: | :----------------: |
| 31:00:00 | 一级映射描述符表的基地址（物理地址） |

**（3）c3， 定义了 ARM 处理器的 16 个域的访问权限** 
每两bit定义了一个位域

| bit  | 31:30:00 | 29:28:00 | 27:26:00 | 25:24:00 | 23:22 | 21:20 | 19:18 | 17:16 | 15:14 | 13:12 | 11:10 | 9:08 | 7:06 | 5:04 | 3:02 | 1:00 |
| ---- | -------- | -------- | -------- | -------- | ----- | ----- | ----- | ----- | ----- | ----- | ----- | ---- | ---- | ---- | ---- | ---- |
| 位域   | D15      | D14      | D13      | D12      | D11   | D10   | D9    | D8    | D7    | D6    | D5    | D4   | D3   | D2   | D1   | D0   |

其中D0对应DOMAIN_KERNEL，D1对应DOMAIN_USER，D2对应DOMAIN_IO（有可能是反的） 
对应权限如下

```
#define DOMAIN_NOACCESS 0
#define DOMAIN_CLIENT   1
#ifdef CONFIG_CPU_USE_DOMAINS
#define DOMAIN_MANAGER  3
#else
#define DOMAIN_MANAGER  1
#endif
```

## 2. 打开MMU之前要配置的cp15的寄存器

通过上面1分析，我们知道c1-c10都要进行设置，但我们这里只关注c2和c3

- c2，MMU页表基地址存储器 

  因此，我们需要先将页表物理地址写入到c2寄存器中

- c3，定义了 ARM 处理器的 16 个域的访问权限 

  需要在这个寄存器中写入位域相应的权限，否则可能导致无法访问。

## 3. 如何打开MMU

经过前面分析，我们知道c1作为MMU的控制器，并且其bit0用于控制MMU的开关。 因此，只需要将c1的BIT 0设置为1，就可以打开MMU了。

# 二. 代码分析

## 1. 整体入口代码分析

```
ldr r13, =__mmap_switched       @ address to jump to after
                        @ mmu has been enabled
@ 把__mmap_switched函数的地址放在寄存器
@ __mmap_switched实现了打开MMU之后跳转到start_kernel。

    badr    lr, 1f              @ return (PIC) address
@ 把“b   __enable_mmu”这条指令的地址放在lr中
@ 当lr寄存器，简单说，就是存储了调用子程序后返回的指令地址

    mov r8, r4              @ set TTBR1 to swapper_pg_dir
@ 把临时内核页表的地址放在r8寄存器中

    ldr r12, [r10, #PROCINFO_INITFUNC]
@ 把cpu对应proc info中的__cpu_flush存放到r12寄存器中
@ 通过《[kernel 启动流程] （第三章）第一阶段之——proc info的获取》，我们知道这个__cpu_flush成员存放的是cpu对应架构的setup函数的地址
@ 对于s5pv210来说，这个值就是__v7_setup的连接地址。

    add r12, r12, r10
    ret r12
@ 这里实现为跳转到__v7_setup的物理地址上，也就是调用__v7_setup，具体怎么实现我也看不懂，希望有高人赐教下。

1:  b   __enable_mmu
@ 跳转到__enable_mmu
```

至此，我们跳转到__v7_setup中了，并且 r13存放了__mmap_switched的地址 lr存放了“b __enable_mmu”这条指令的地址

## 2. __v7_setup分析

__v7_setup实现如下 
arch/arm/mm/proc-v7.S

```
__v7_setup:
    adr r0, __v7_setup_stack_ptr
    ldr r12, [r0]
    add r12, r12, r0            @ the local stack
    stmia   r12, {r1-r6, lr}        @ v7_invalidate_l1 touches r0-r6
    bl      v7_invalidate_l1
    ldmia   r12, {r1-r6, lr}

__v7_setup_cont:
    and r0, r9, #0xff000000     @ ARM?
    teq r0, #0x41000000
    bne __errata_finish
    and r3, r9, #0x00f00000     @ variant
    and r6, r9, #0x0000000f     @ revision
    orr r6, r6, r3, lsr #20-4       @ combine variant and revision
    ubfx    r0, r9, #4, #12         @ primary part number

    /* Cortex-A8 Errata */
    ldr r10, =0x00000c08        @ Cortex-A8 primary part number
    teq r0, r10
    beq __ca8_errata

    /* Cortex-A9 Errata */
    ldr r10, =0x00000c09        @ Cortex-A9 primary part number
    teq r0, r10
    beq __ca9_errata

    /* Cortex-A15 Errata */
    ldr r10, =0x00000c0f        @ Cortex-A15 primary part number
    teq r0, r10
    beq __ca15_errata

__errata_finish:
    mov r10, #0
    mcr p15, 0, r10, c7, c5, 0      @ I+BTB cache invalidate
#ifdef CONFIG_MMU
    mcr p15, 0, r10, c8, c7, 0      @ invalidate I + D TLBs
    v7_ttb_setup r10, r4, r5, r8, r3    @ TTBCR, TTBRx setup
    ldr r3, =PRRR           @ PRRR
    ldr r6, =NMRR           @ NMRR
    mcr p15, 0, r3, c10, c2, 0      @ write PRRR
    mcr p15, 0, r6, c10, c2, 1      @ write NMRR
#endif
    dsb                 @ Complete invalidations
#ifndef CONFIG_ARM_THUMBEE
    mrc p15, 0, r0, c0, c1, 0       @ read ID_PFR0 for ThumbEE
    and r0, r0, #(0xf << 12)        @ ThumbEE enabled field
    teq r0, #(1 << 12)          @ check if ThumbEE is present
    bne 1f
    mov r3, #0
    mcr p14, 6, r3, c1, c0, 0       @ Initialize TEEHBR to 0
    mrc p14, 6, r0, c0, c0, 0       @ load TEECR
    orr r0, r0, #1          @ set the 1st bit in order to
    mcr p14, 6, r0, c0, c0, 0       @ stop userspace TEEHBR access
1:
#endif
    adr r3, v7_crval
    ldmia   r3, {r3, r6}
 ARM_BE8(orr    r6, r6, #1 << 25)       @ big-endian page tables
#ifdef CONFIG_SWP_EMULATE
    orr     r3, r3, #(1 << 10)              @ set SW bit in "clear"
    bic     r6, r6, #(1 << 10)              @ clear it in "mmuset"
#endif
    mrc p15, 0, r0, c1, c0, 0       @ read control register
    bic r0, r0, r3          @ clear bits them
    orr r0, r0, r6          @ set them
 THUMB( orr r0, r0, #1 << 30    )   @ Thumb exceptions
    ret lr              @ return to head.S:__ret

    .align  2
__v7_setup_stack_ptr:
    .word   PHYS_RELATIVE(__v7_setup_stack, .)
ENDPROC(__v7_setup)
```

主要是armv7打开MMU前的一些架构性的准备动作。

```
    mrc p15, 0, r0, c1, c0, 0       @ read control register
@ 通过如上指令，将cp15协处理器的c1寄存器的值读写到r0寄存器中
@ 如前面所说，c1寄存器就是MMU的控制器寄存器

    bic r0, r0, r3          @ clear bits them
    orr r0, r0, r6          @ set them
 THUMB( orr r0, r0, #1 << 30    )   @ Thumb exceptions
@ 根据前面armv7的一些计算结果，清除和设置要写入MMU控制器的值(r0)的某些位。
@ 注意，虽然前面代码没有分析，但是可以确认的是，bit0是被置位为1的，也就enable_mmu bit为1.
@ 这样才能打开MMU功能。

    ret lr              @ return to head.S:__ret
@ 因为前面在进入__v7_setup之前，设置了badr   lr, 1f  
@ 1:    b   __enable_mmu
@ 所以在这里会跳转到__enable_mmu中
```

至此，r0上已经存放了初步设置完的、准备写入cp15协处理器的c1寄存器的值。 并且跳转到了__enable_mmu中。 

## 3. __enable_mmu代码分析

经过前面的分析，我们知道r0上已经存放了初步设置完的、准备写入cp15协处理器的c1寄存器的值。 

这里主要做的动作是：

- 需要先将页表物理地址写入到cp15的c2寄存器中
- 需要在cp15的c3寄存器中写入位域相应的权限
- 配置cp15的c1寄存器，用来控制MMU的相应功能

对应代码如下： 

```
__enable_mmu:
#if defined(CONFIG_ALIGNMENT_TRAP) && __LINUX_ARM_ARCH__ < 6
    orr r0, r0, #CR_A
#else
    bic r0, r0, #CR_A
#endif
@ 配置MMU的地址对齐检查是否使能，也就是c1的bit 1位，先暂存在r0中。

#ifdef CONFIG_CPU_DCACHE_DISABLE
    bic r0, r0, #CR_C
#endif
@ 配置数据/整个 cache是否打开，也就是c1的bit 2位，先暂存在r0中。

#ifdef CONFIG_CPU_BPREDICT_DISABLE
    bic r0, r0, #CR_Z
#endif
@ 配置跳转预测功能是否使能，也就是c1的bit 11位，先暂存在r0中。

#ifdef CONFIG_CPU_ICACHE_DISABLE
    bic r0, r0, #CR_I
#endif
@ 配置指令 cache是否使能，也就是c1的bit 12位，先暂存在r0中。

#ifdef CONFIG_ARM_LPAE
    mcrr    p15, 0, r4, r5, c2      @ load TTBR0
#else
    mov r5, #DACR_INIT
@ 设置位域权限，先存放到r5中。

    mcr p15, 0, r5, c3, c0, 0       @ load domain access register
@ 将位域访问权限写入cp15的c3寄存器中，理由第一节已经说明了

    mcr p15, 0, r4, c2, c0, 0       @ load page table pointer
@ 将页表物理地址写入cp15的c2寄存器中，理由第一节已经说明了
#endif

    b   __turn_mmu_on
@ 跳转到__turn_mmu_on中
ENDPROC(__enable_mmu)
```

至此，MMU打开前的准备动作已经完成，但是还没有写入MMU的控制寄存器c1，也就是说MMU还没有打开。 
最终跳转到__turn_mmu_on中做MMU打开的动作，主要的目标也就是向cp15的c1进行写入。

## 4. __turn_mmu_on代码分析

__turn_mmu_on是真正做MMU打开的动作。打开之后，CPU会把所有地址都当作虚拟地址处理。 ____但是因为在前面已经对

__turn_mmu_on的代码区域进行恒等映射，所以无需担心打开MMU之后程序跑飞的问题。 

前面已经说明，r0上已经存放了设置完成、准备写入cp15协处理器的c1寄存器的值。 __turn_mmu_on的主要动作就是向cp15的c1寄存器写入r0上的值。 

具体代码如下：

```
ENTRY(__turn_mmu_on)
    mov r0, r0
    instr_sync
@ 前面是一些同步操作，具体还没有完全搞清楚

    mcr p15, 0, r0, c1, c0, 0       @ write control reg
@ 这里就是这段代码的核心，也就是把r0的值写入到c1中，这时候，MMU就已经打开了

    mrc p15, 0, r3, c0, c0, 0       @ read id reg
    instr_sync
    mov r3, r3
@ 前面是一些同步操作，具体还没有完全搞清楚

    mov r3, r13
    ret r3
@ 这里会跳转到__mmap_switched中。
@ 因为前面在代码入口有如下指令“ldr    r13, =__mmap_switched”，所以r13存放的是__mmap_switched
@ __mmap_switched主要实现start_kernel的动作，下一篇会说明
__turn_mmu_on_end:
ENDPROC(__turn_mmu_on)
```

通过上述，MMU已经打开完成，并且跳转到了__mmap_switched中。
