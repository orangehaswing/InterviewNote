# kernel 启动流程(一) - 概述

**参考文档**：

- [ARMV7官方数据手册](http://download.csdn.net/detail/ooonebook/9658942)
- [ARM的CP15协处理器的寄存器](http://download.csdn.net/detail/ooonebook/9658937)

# 一. kernel启动之前的准备动作

在kernel启动之前的准备都是由bootloader来完成。所以不管是什么bootloader，例如uboot、LK、superboot等等，都需要实现以下准备动作。这里指说明概念，不涉及代码。 

## 1. kernel镜像加载到ddr的相应位置

kernel镜像一般会存在于存储设备上，比如 FLASH/ EMMC/ SDCARD. 因此，需要先将kernel镜像加载到RAM的位置上，CPU才可以去访问到kernel。 
具体实现方法由bootloader决定，可以是自动复制，也可以是根据bootloader cmdline模式下输入的命令来是否复制。 但是注意，加载的位置是有要求的，一般是加载到物理RAM偏移0x8000的位置，也就是要在前面预留出32K的RAM。kernel会从加载的位置上开始解压，而kernel前面的32K空闲RAM中，16K作为boot params，16K作为临时页表 。

例如，s5pv210的物理RAM的起始地址是0x20000000，那么kernel的加载地址就应该是0x20008000。

## 2. 硬件要求

根据arch/arm/kernel/head.S的stext（kernel的入口函数）的注释头

```
/*
 * Kernel startup entry point.
 * ---------------------------
 *
 * This is normally called from the decompressor code.  The requirements
 * are: MMU = off, D-cache = off, I-cache = dont care, r0 = 0,
 * r1 = machine nr, r2 = atags or dtb pointer.
```

所以有如下要求 

*  MMU = off 
   MMU用来处理物理地址到虚拟内存地址的映射，因此需要软件上需要先配置其映射表（也就是后续文章会说明的页表）。MMU关闭的情况下，CPU寻址的地址都是物理地址，也就是不需要经过转化直接访问相应的硬件。一旦打开之后，CPU寻址的所有地址都是虚拟地址，都会经过MMU映射到真正的物理地址上，即使你在代码中访问的是一个物理地址，也会被当作虚拟内存地址使用。 
   而映射表是由kernel自己创建的，因此，在创建映射表之前kernel访问的地址都是物理地址，所以必须保证MMU是关闭状态。 
*  D-cache = off 
     CACHE是CPU和内存之间的高速缓冲存储器，又分成数据缓冲器D-cache和指令缓冲器I-cache。具体意义这里不多说明。 
     数据Cache一定要关闭，否则可能kernel刚启动的过程中，去取数据的时候，从Cache里面取，而这时候RAM中数据还没有Cache过来，导致数据预取异常 。 
     自己的理解是，假设打开MMU之前，cache上存了一个项“地址0x20000000、数据0xffff0000”,打开MMU之后，读取0x20000000地址上（虚拟地址）数据，但是此时会直接从cache中读到项“地址0x20000000、数据0xffff0000”，但实际上对应物理地址上的数据并不是这个，所以会导致读取的数据错误。 
*  r0 = 0 
     硬性规定r0寄存器为0，没有什么意义。 
*  r1 = machine nr 
     r1寄存器里面存放machine ID。 
*  r2 = atags or dtb pointer 
     r2寄存器里面存放atags或者dtb的地址指针。

## 3. 跳转到kernel镜像入口的对应位置

bootloader需要通过设置PC指针到kernel的入口代码处（也就是kernel的加载位置）来实现kernel的跳转。 
例如 tiny210(s5pv210) 的PC设置为0x20008000来实现到kernel的跳转。

# 二. kernel启动过程入口说明

## 1. kernel入口地址的指定 

arch/arm/kernel/vmlinux.lds.S中

```
ENTRY(stext)
```

所以kernel启动的入口代码位于arch/arm/kernel/head.S。 这里也是我们后续要分析kernel启动流程的核心代码部分。 

## 2. kernel入口地址 

* 连接地址，通过System.map查看，可以看到stext的连接地址是0xc0008000

```
c0008000 T stext
```

加载地址，因为uboot把kernel加载到0x20008000的位置上，所以kernel的入口是0x20008000 根据连接脚本，stext是作为kernel的入口（上述一），所以stext就在kernel镜像的起始位置上，所以stext的加载地址应该0x20008000。

# 三. kernel启动的两个阶段说明

kernel在启动初期主要分成两个阶段。

## 1. 第一阶段：从入口跳转到start_kernel之前的阶段。

对应代码arch/arm/kernel/head.S中stext的实现：

```
ENTRY(stext)
```

这个阶段主要由汇编语言实现。 负责MMU打开之前的一些操作，以及打开MMU的操作。 

由于这个阶段MMU还没有打开，并且kernel加载地址和连接地址并一致，所以需要使用位置无关设计。在运行过程中运行地址和加载地址一致。 

其主要流程如下： 

* 设置为SVC模式，关闭所有中断 
  什么是SVC模式？为什么要设置成SVC模式？ 如何关闭所有中断？为什么要关闭所有中断呢？

  ```
      @ ensure svc mode and all interrupts masked
      safe_svcmode_maskall r9
  ```

* 获取CPU ID，提取相应的proc info 
  从哪里去提取proc_info? proc_info里面存放了什么东西？这里面的数据结构应该是很重要所以才有必要在打开MMU前就去获取。

   ```
      mrc p15, 0, r9, c0, c0      @ get processor id
      bl  __lookup_processor_type     @ r5=procinfo r9=cpuid
      movs    r10, r5             @ invalid processor (r5=0)?
   THUMB( it  eq )        @ force fixup-able long branch encoding
      beq __error_p           @ yes, error 'p'
   ```

* 验证tags或者dtb 
  如何验证dtb的有效性？

  ```
      bl  __vet_atags
  ```

* 创建临时内核页表的页表项 
  页表项的作用？基础知识？和MMU之间的关系？ 要为哪些内存区域创建页表项？为什么？

  ```
      bl  __create_page_tables
  ```

* 配置r13寄存器，也就是设置打开MMU之后要跳转到的函数。 
  为什么要在打开MMU之前配置呢

  ```
      ldr    r13, =__mmap_switched        @ address to jump to after
                          @ mmu has been enabled
  ```

  使能MMU 
  如何使能MMU？ swapper_pg_dir变量？

  ```
      ldr    r12, [r10, #PROCINFO_INITFUNC]
      add    r12, r12, r10
      ret    r12
  1:    b    __enable_mmu
  ```

* 跳转到start_kernel 
  如何跳转到start_kernel? 

分析代码的时候可以结合上述疑问去进行分析。 
后续分析启动代码的过程中会针对上述流程和疑问进行文档整理。

## 2. 第二阶段：start_kernel开始的阶段。

这个阶段主要由C语言实现。 kernel启动流程中剩余的操作都是这里实现。 

# 四. kernel准备动作在uboot中的实现

以下以tiny210为例：

## 1. kernel镜像加载到ddr的相应位置

在project-X 项目中tiny210的存储设备暂时设计为SDCARD，并且是自动将kernel加载到ddr上。 
kernel镜像有传统的uImage方式和FIT方式。 这里以传统的uImage方式做说明。 
具体实现如下： 
u-boot/board/samsung/tiny210/board.c

    #define KERNEL_TEXT_BASE                0x20008000
    void copy_kernel_to_ddr(void)
    {
            u32 sdmmc_base_addr;
            copy_sd_mmc_to_mem copy_bl2 = (copy_sd_mmc_to_mem)(*(u32*)CopySDMMCtoMem);
            sdmmc_base_addr = *(u32 *)SDMMC_BASE;
            if(sdmmc_base_addr == SDMMC_CH2_BASE_ADDR)
            {
                    copy_bl2(2, MOVI_KERNEL_SDCARD_POS, MOVI_KERNEL_BLKCNT, (u32 *)KERNEL_TEXT_BASE, 0);
            }
    }
MOVI_KERNEL_SDCARD_POS表示kernel镜像在SDCARD上的位置。 KERNEL_TEXT_BASE表示要加载到RAM上的位置，注意，就是0x20008000。 通过copy_sd_mmc_to_mem来实现SDCARD到DDR上的复制动作。

## 2. MMU和D-cache的关闭

关于arm的MMU和D-cache都是有协处理器CP15来进行管理。  C1寄存器的某些位用于配置MMU中的一些操作，C1的0位控制禁止/使能MMU，简单示例如下：

```
MRC P15，0，R0，C1，0，0
ORR R0，#01
MCR P15，0，R0，C1，0，0
```

uboot中s5pv210关闭MMU和D-cache的位置在（其实默认就是关闭的） 

u-boot/arch/arm/cpu/armv7/start.S

    ENTRY(cpu_init_cp15)
        mrc p15, 0, r0, c1, c0, 0
        bic r0, r0, #0x00002000 @ clear bits 13 (--V-)
        bic r0, r0, #0x00000007 @ clear bits 2:0 (-CAM)
        orr r0, r0, #0x00000002 @ set bit 1 (--A-) Align
        orr r0, r0, #0x00000800 @ set bit 11 (Z---) BTB
    #ifdef CONFIG_SYS_ICACHE_OFF
        bic r0, r0, #0x00001000 @ clear bit 12 (I) I-cache
    #else
        orr r0, r0, #0x00001000 @ set bit 12 (I) I-cache
    #endif
        mcr p15, 0, r0, c1, c0, 0
## 3. r0 r1 r2寄存器的设置和kernel的跳转

u-boot/arch/arm/lib/bootm.c

    static void boot_jump_linux(bootm_headers_t *images, int flag)
    {
        unsigned long machid = gd->bd->bi_arch_number;//获取mechine_id
        void (*kernel_entry)(int zero, int arch, uint params);
        kernel_entry = (void (*)(int, int, uint))images->ep;//获取了要转的入口地址
        debug("## Transferring control to Linux (at address %08lx)" \
            "...\n", (ulong) kernel_entry);
           r2 = (unsigned long)images->ft_addr;//获取dtb地址。
           kernel_entry(0, machid, r2);
    ｝
最后跳转到kernel_entry(0, machid, r2);，也就是0x20008000的位置，并且通过参数，将0传入到r0,machid传入到r1，dtb的地址传入到r2。最终就调用到kernel的入口去了。 

在串口输出中会看到如下log：

```
## Transferring control to Linux (at address 20008000)...
```
