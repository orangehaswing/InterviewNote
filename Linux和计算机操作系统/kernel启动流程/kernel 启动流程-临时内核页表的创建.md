# kernel 启动流程-临时内核页表的创建

## 零、说明

1、kernel启动流程第一阶段简单说明

`arch/arm/kernel/head.S`

kernel入口地址对应stext

```
ENTRY(stext)
```

第一阶段要做的事情，也就是stext的实现内容

- 设置为SVC模式，关闭所有中断
- 获取CPU ID，提取相应的proc info
- 验证tags或者dtb
- 创建临时内核页表的页表项
- 配置r13寄存器，也就是设置打开MMU之后要跳转到的函数。
- 使能MMU
- 跳转到start_kernel，也就是跳转到第二阶段

本文要介绍的是“创建临时内核页表的页表项”的部分。

## 2、疑问

主要带着以下几个问题去理解

- 什么是MMU?以及其和页表之间的关系？
- 页表内容？页表格式？如何进行创建？

## 3、对应代码实现

```
    __HEAD
ENTRY(stext)
    ldr    r8, =PLAT_PHYS_OFFSET        @ always constant in this case
    /*
     * r1 = machine no, r2 = atags or dtb,
     * r8 = phys_offset, r9 = cpuid, r10 = procinfo
     */
    bl    __create_page_tables
```

__create_page_tables的工作就是创建临时内核页表。而创建临时内核页表则是为了打开MMU功能。

# 一、MMU和页表简单介绍

以下只是为了理解很简单地进行介绍。 并且主要介绍段式管理的工作原理。

## 1、MMU简单介绍

MMU是Memory Management Unit的缩写，中文名是内存管理单元，它是中央处理器（CPU）中用来管理虚拟存储器、物理

存储器的控制线路。 

其主要功能如下：

- 将线性地址映射为物理地址 
- 所谓线性地址就是指虚拟地址，而物理地址就是指实际在RAM上的地址

提供硬件机制的内存访问授权

## 2、页表介绍

MMU工作的核心是页表，也就是其根据页表来找到映射关系以及权限。 页表由页表项构成，每一个虚拟地址映射区都会有一个对应的页表项。 arm的页表有如下分类：

- 段式管理页表 

  在arm打开MMU初期，使用的是临时内核页表，其类型就是段式页表。 

  段式页表将4GB的地址空间（32bit系统）划分成4096个1MB的段，因此段式页表有4096个页表项，每个页表项有32bit（4 byte），故段式页表需要16KB的空间 

页表项格式如下：

|    位号     |    功能    |
| :-------: | :------: |
| 31:20 bit |   段序号    |
| 20: 0 bit | MMU flag |

## 3、arm上MMU的工作原理

arm将页表基地址存放在协处理器cp15的c2寄存器上。 

如下说明： 

CP15 中的寄存器 C2 保存的是页表的基地址，即一级映射描述符表的基地址。 

arm的MMU会根据虚拟地址计算出其相应页表项的偏移，从cp15的c2寄存器中获取页表基址之后，加上偏移得到对应的页表

项地址。后续操作就是根据页表结构来做的。这些动作都是MMU硬件处理！ 

如果是段式页表的话，再根据段内偏移以及页表项中的物理段基址最终得到对应的物理地址。

```
段式管理页表工作举例（先不关心MMU flag）：
假设
(1) 页表基地址为0x0（存放在CP15的c2寄存器上）.
(2) 0xc0000000所在段（也就是段序号为0xc00）的页表项地址0x3000,
(3) 页表项地址0x3000的值为0x20000000（也就是段序号为0x300）.
当虚拟地址为0xc0001000,计算方式如下
(1) 左移20位的得到虚拟地址所在段序号为0xc00,获取低20位得到段内偏移为0x1000
(2) 计算对应页表项地址=页表基地址0x0+段序号0xc00*页表项长度4=0x3000.
(3) 0x3000地址上的值为0x20000000，提取高12位得到0x200，所以对应物理段基址为0x20000000
(4) 物理段基址加上段内偏移得到实际的物理地址0x20001000.
```

## 4、临时内核页表及其内容

为了打开MMU，内核需要创建一个临时内核页表，用于kenrel启动过程中的打开MMU的过渡阶段。 并且，使用的是段式管理的方法。 需要建立如下区域的映射

- 打开MMU的函数的代码区域的恒等映射。 

  恒等映射是指虚拟地址和物理地址一致的映射。 
  在打开MMU的过程中，CPU还是按照地址顺序一条接着一条去获取指令，也就是说此时PC指针还是指向这段代码区域的物理地址。当MMU打开之后，如果没有恒等映射的话，PC指针此时还是指向这段代码区域的物理地址，但实际上CPU会把PC指针的地址当作虚拟地址进行操作，而造成找不到对应的物理地址。因此，如果做了恒等映射，虚拟地址和物理地址一致，及时CPU会把PC指针的地址当作虚拟地址进行操作，最终仍会映射到相同的物理地址上。

- kernel镜像的映射 

  kernel的链接地址是0xc0008000。 而我们把kernel加载到0x20008000的位置上。kernel镜像的连接区域映射到实际的物理地址的区域。

- dtb区域的映射 

  在kernel启动初期需要使用到dtb的东西，因此，在临时内核页表中需要做这些区域的映射。

# 二、s5pv210映射说明

内存地址和对应段页表项的地址如下： 

(s5p210的物理RAM地址偏移是0x20000000，所以段页表项的基地址是0x20004000)

|          虚拟段          |          物理段          |  对应页表项地址   |         计算方式         |        临时页表映射的值         |
| :-------------------: | :-------------------: | :--------: | :------------------: | :---------------------: |
| 0x00000000-0x000FFFFF |           -           | 0x20004000 | (0x20004000+0x000*4) |            -            |
| 0x00100000-0x002FFFFF |           -           | 0x20004004 | (0x20004000+0x001*4) |            -            |
| 0x00200000-0x003FFFFF |           -           | 0x20004008 | (0x20004000+0x002*4) |            -            |
|           …           |                       |            |                      |                         |
| 0x20100000-0x201FFFFF | 0x20100000-0x201FFFFF | 0x20004804 | (0x20004000+0x201*4) | (0x201<<20) \| mmuflags |
|           …           |                       |            |                      |                         |
| 0xC0000000-0xC00FFFFF | 0x20000000-0x200FFFFF | 0x20007000 | (0x20004000+0xC00*4) | (0x200<<20) \| mmuflags |
| 0xC0100000-0xC01FFFFF | 0x20100000-0x201FFFFF | 0X20007004 | (0x20004000+0xC01*4) | (0x201<<20) \| mmuflags |
|           …           |                       |            |                      |                         |
| 0xC0500000-0xC05FFFFF | 0x20500000-0x205FFFFF | 0X20007014 | (0x20004000+0xC05*4) | (0x205<<20) \| mmuflags |
|           …           |                       |            |                      |                         |
| 0xDFC00000-0xDFCFFFFF |  0x3FC00000-3FCFFFFF  | 0x200077F0 | (0x20004000+0xDFC*4) | (0x205<<20) \| mmuflags |
| 0xDFD00000-0xDFDFFFFF |  0x3FD00000-3FDFFFFF  | 0x200077F4 | (0x20004000+0xDFC*4) | (0x205<<20) \| mmuflags |
|           …           |                       |            |                      |                         |
| 0xFFF00000-0xFFFFFFFF |           -           | 0x20007FFC | (0x20004000+0xFFF*4) |            -            |

其中

- [0x20100000-0x201FFFFF]段是打开MMU的函数的代码区域的恒等映射。
- [0xC0000000-0xC00FFFFF]段到[0xC0500000-0xC05FFFFF]段是kernel镜像的映射
- [0xDFC00000-0xDFCFFFFF]段到[0xDFD00000-0xDFDFFFFF]段是dtb内存区域的映射 

# 三、代码分析

## 1、代码入口

（1）分配物理地址给r8

```
ENTRY(stext)
...
#define PLAT_PHYS_OFFSET    UL(CONFIG_PHYS_OFFSET)
ldr    r8, =PLAT_PHYS_OFFSET        @ always constant in this case
```

物理地址PHYS_OFFSET的定义如下： 

arch/arm/Kconfig

```
config PHYS_OFFSET
    hex "Physical address of main memory" if MMU
    depends on !ARM_PATCH_PHYS_VIRT
    default 0x20000000 if ARCH_S5PV210
```

所以config是0x20000000，和s5pv210的ddr起始ram一致。 

（2）调用 bl __create_page_tables

```
ENTRY(stext)
...
    bl    __create_page_tables
```



create_page_tables主要用于创建临时内核页表。

create_page_tables代码总览，后续小节会详细分析： 

arch/arm/kernel/head.S 

移除掉CONFIG_ARM_LPAE和CONFIG_DEBUG_LL的无关部分

```
/*
 * Setup the initial page tables.  We only setup the barest
* amount which are required to get the kernel running, which
* generally means mapping in the kernel code.
*
* r8 = phys_offset, r9 = cpuid, r10 = procinfo
*
* Returns:
 *  r0, r3, r5-r7 corrupted
 *  r4 = physical page table address
*/
__create_page_tables:
    pgtbl    r4, r8                @ page table address
        @============上述代码见下<详解1>

    /*
     * Clear the swapper page table
     */
    mov    r0, r4
    mov    r3, #0
    add    r6, r0, #PG_DIR_SIZE
1:    str    r3, [r0], #4
    str    r3, [r0], #4
    str    r3, [r0], #4
    str    r3, [r0], #4
    teq    r0, r6
    bne    1b
        @============上述代码见下<详解2>

    ldr    r7, [r10, #PROCINFO_MM_MMUFLAGS] @ mm_mmuflags
        @============上述代码见下<详解3>

    /*
     * Create identity mapping to cater for __enable_mmu.
     * This identity mapping will be removed by paging_init().
     */
    adr    r0, __turn_mmu_on_loc
    ldmia    r0, {r3, r5, r6}
    sub    r0, r0, r3            @ virt->phys offset
    add    r5, r5, r0            @ phys __turn_mmu_on
    add    r6, r6, r0            @ phys __turn_mmu_on_end
    mov    r5, r5, lsr #SECTION_SHIFT
    mov    r6, r6, lsr #SECTION_SHIFT

1:    orr    r3, r7, r5, lsl #SECTION_SHIFT    @ flags + kernel base
    str    r3, [r4, r5, lsl #PMD_ORDER]    @ identity mapping
    cmp    r5, r6
    addlo    r5, r5, #1            @ next section
    blo    1b
        @============上述代码见下<详解4>

    /*
     * Map our RAM from the start to the end of the kernel .bss section.
     */
    add    r0, r4, #PAGE_OFFSET >> (SECTION_SHIFT - PMD_ORDER)
    ldr    r6, =(_end - 1)
    orr    r3, r8, r7
    add    r6, r4, r6, lsr #(SECTION_SHIFT - PMD_ORDER)
1:    str    r3, [r0], #1 << PMD_ORDER
    add    r3, r3, #1 << SECTION_SHIFT
    cmp    r0, r6
    bls    1b
        @============上述代码见下<详解5>

    /*
     * Then map boot params address in r2 if specified.
     * We map 2 sections in case the ATAGs/DTB crosses a section boundary.
     */
    mov    r0, r2, lsr #SECTION_SHIFT
    movs    r0, r0, lsl #SECTION_SHIFT
    subne    r3, r0, r8
    addne    r3, r3, #PAGE_OFFSET
    addne    r3, r4, r3, lsr #(SECTION_SHIFT - PMD_ORDER)
    orrne    r6, r7, r0
    strne    r6, [r3], #1 << PMD_ORDER
    addne    r6, r6, #1 << SECTION_SHIFT
    strne    r6, [r3]
        @============上述代码见下<详解6>

    ret    lr
ENDPROC(__create_page_tables)
```

## 2、详解1

```
    pgtbl     r4, r8                @ page table address
```

**pgtbl 宏用于通过DRAM物理地址来获取页表的物理地址。** 

前面我们已经知道r8用于存放DRAM的起始物理地址，r4则是要存放计算得到的页表物理地址。 

pgtbl 宏如下：

```
arch/arm/kernel/head.S
    .macro    pgtbl, rd, phys
    add    \rd, \phys, #TEXT_OFFSET
    sub    \rd, \rd, #PG_DIR_SIZE
    .endm
```

kernel在放在DRAM上偏移TEXT_OFFSET的位置上。 而linux规定将TEXT_OFFSET之前的PG_DIR_SIZE大小的空间用作临时页表。 所以计算方式如下： 

kernel起始地址	=	DRAM起始物理地址	+	TEXT_OFFSET	=	0x20008000 

内核页表地址	=	kernel起始地址	-	PG_DIR_SIZE	=	0x20004000 

所以代码换算成如下计算： 

\rd(r4) = phys(r8) + TEXT_OFFSET 

\rd(r4) = \rd(r4) - PG_DIR_SIZE 

TEXT_OFFSET如下： 


```
./arch/arm/Makefile
# The byte offset of the kernel image in RAM from the start of RAM.
TEXT_OFFSET := $(textofs-y)
# Text offset. This list is sorted numerically by address in order to
# provide a means to avoid/resolve conflicts in multi-arch kernels.
textofs-y       := 0x00008000
```

32K的偏移 

PG_DIR_SIZE如下： 

```
arch/arm/kernel/head.S
#define PG_DIR_SIZE    0x4000
#define PMD_ORDER    2
```

前面也说过页表的大小是16K，刚好和这里是符合的。最终获得页表物理地址（s5pv210是0x20004000）并且存放在r4寄存器中。

## 3、详解2

为临时内核页表分配空间之后，接下来的任务就是清空临时内核页表分配空间。

```
    /*
     * Clear the swapper page table
     */
    mov    r0, r4   
@ 将临时内核页表物理地址放到r0上

    mov    r3, #0   
@ 在r3上存放0值

    add    r6, r0, #PG_DIR_SIZE   
@ 将临时内核页表的末尾物理地址放到r6上

1:    str    r3, [r0], #4       
@ 从r0（临时内核页表物理地址）指向的寄存器上开始写入0值，每16个字节一个循环

    str    r3, [r0], #4
    str    r3, [r0], #4
    str    r3, [r0], #4
    teq    r0, r6           
@ 比较是否已经写到了r6（临时内核页表的末尾物理地址）上

    bne    1b           
@ 如果还没有写完，进入下一个循环
```

## 4、详解3

设置MMU的标识并存放到r7寄存器中，后续需要写入到临时内核页表的页表项中

```
ldr    r7, [r10, #PROCINFO_MM_MMUFLAGS] @ mm_mmuflags
```

通过这边文章我们已经知道已经将和对应CPU配置的proc info存放到了r10寄存器中。

```
PROCINFO_MM_MMUFLAGS对应如下
DEFINE(PROCINFO_MM_MMUFLAGS,    offsetof(struct proc_info_list, __cpu_mm_mmu_flags));
```

PROCINFO_MM_MMUFLAGS对应proc_info_list中的__cpu_mm_mmu_flags，这个成员用于表示临时页表映射的内核空间的MMU标识。 

s5pv210的这个值对应为PMD_TYPE_SECT | PMD_SECT_AP_WRITE | PMD_SECT_AP_READ | \ PMD_SECT_AF |

PMD_FLAGS_UP。 

最终，将这个成员对应的值写入到r7寄存器中。

## 5、详解4

开始进行映射表的创建，首先是创建恒等映射。 

代码总览如下

```
  /*
     * Create identity mapping to cater for __enable_mmu.
     * This identity mapping will be removed by paging_init().
     */
    adr    r0, __turn_mmu_on_loc
    ldmia    r0, {r3, r5, r6}
    sub    r0, r0, r3            @ virt->phys offset
    add    r5, r5, r0            @ phys __turn_mmu_on
    add    r6, r6, r0            @ phys __turn_mmu_on_end
    mov    r5, r5, lsr #SECTION_SHIFT
    mov    r6, r6, lsr #SECTION_SHIFT

1:    orr    r3, r7, r5, lsl #SECTION_SHIFT    @ flags + kernel base
    str    r3, [r4, r5, lsl #PMD_ORDER]    @ identity mapping
    cmp    r5, r6
    addlo    r5, r5, #1            @ next section
    blo    1b
```

也就是对打开mmu的函数，也就是__turn_mmu_on进行恒等映射。 所谓恒等映射，就是将物理地址相应到相同的虚拟地址上。__

turn_mmu_on代码如下

```
ENTRY(__turn_mmu_on)
    mov    r0, r0
    instr_sync
    mcr    p15, 0, r0, c1, c0, 0        @ write control reg
    mrc    p15, 0, r3, c0, c0, 0        @ read id reg
    instr_sync
    mov    r3, r3
    mov    r3, r13
    ret    r3
__turn_mmu_on_end:
ENDPROC(__turn_mmu_on)
```

__turn_mmu_on和__turn_mmu_on_end标识了其起始代码地址和末端代码地址。 

通过System.map查看其连接地址如下：

```
c0100000 T __turn_mmu_on
c0100020 t __turn_mmu_on_end
```

kernel将这些链接地址存放到了__turn_mmu_on_loc中。

__turn_mmu_on_loc:
​    .long    .
​    .long    __turn_mmu_on
​    .long    __turn_mmu_on_end

也就是说临时内核映射表需要为__turn_mmu_on添加如下页表

|          虚拟段          |          物理段          |   对应页表位置   |         计算方式         |        临时页表映射的值         |
| :-------------------: | :-------------------: | :--------: | :------------------: | :---------------------: |
| 0x20100000-0x201FFFFF | 0x20100000-0x201FFFFF | 0x20004804 | (0x20004000+0x201*4) | (0x201<<20) \| mmuflags |

代码分析如下

```
    adr    r0, __turn_mmu_on_loc
    ldmia    r0, {r3, r5, r6}
    sub    r0, r0, r3            @ virt->phys offset
    add    r5, r5, r0            @ phys __turn_mmu_on
    add    r6, r6, r0            @ phys __turn_mmu_on_end
@ 通过以下代码将其转化为物理地址并且存放到r5和r6寄存器中，
@ 具体方法在《[kernel 启动流程] （第三章）第一阶段之——proc info的获取》中说明过了。

    mov    r5, r5, lsr #SECTION_SHIFT
    mov    r6, r6, lsr #SECTION_SHIFT
@ 以1M做为一个段，所以对应段序号是内存地址左移20位。
@ arch/arm/include/asm/pgtable-2level.h
@ #define SECTION_SHIFT        20
@ 以上获取__turn_mmu_on代码部分所对应的段序号，
@ r5存放起始地址的段序号，r6存放末地址的段序号。
@ 后续就是填充相应的段页表项

1:     orr    r3, r7, r5, lsl #SECTION_SHIFT    @ flags + kernel base
@ 填写对应段的段页表项的内容，存放在r3中。
@ 因为是恒等映射，所以映射后的段地址就是物理地址的段序号左移SECTION_SHIFT。
@ 页表项内容为段序号（r5）左移SECTION_SHIFT后或上MMU标识(r7)，

    str    r3, [r4, r5, lsl #PMD_ORDER]    @ identity mapping
@ 将段页表项值(r3)写入到对应的段页表项中
@ 段页表项的地址=段页表起始地址(r4)+段序号(r5)*段页表项的size(1<<PMD_ORDER,4K)

    cmp    r5, r6
    addlo    r5, r5, #1            @ next section
    blo    1b
@ 判断是否已经写到__turn_mmu_on末地址的对应的段页表项中，如果没有的话，继续写入下一个段。
```

通过上述，就完成了__turn_mmu_on代码部分的恒等映射。

## 6、详解5

以下对kernel的内核空间进行映射。

通过System.map中可以看出kernel的连接区域如下：

```
c0008000 T stext
c0547d74 B _end
```

其相应在物理地址上的内存区域是 0x20008000	到	0x20547d74	的区域。 

因此后续要创建物理区[0x20008000-0x20547d74]到内核映射区[0xc0008000到0xc0547d74]的内存映射。 

对应如下：

|          虚拟段          |          物理段          |   对应页表位置   |         计算方式         |        临时页表映射的值         |
| :-------------------: | :-------------------: | :--------: | :------------------: | :---------------------: |
| 0xC0000000-0xC00FFFFF | 0x20000000-0x200FFFFF | 0x20007000 | (0x20004000+0xC00*4) | (0x200<<20) \| mmuflags |
| 0xC0100000-0xC01FFFFF | 0x20100000-0x201FFFFF | 0X20007004 | (0x20004000+0xC01*4) | (0x201<<20) \| mmuflags |
|           …           |                       |            |                      |                         |
| 0xC0500000-0xC05FFFFF | 0x20500000-0x205FFFFF | 0X20007014 | (0x20004000+0xC05*4) | (0x205<<20) \| mmuflags |















