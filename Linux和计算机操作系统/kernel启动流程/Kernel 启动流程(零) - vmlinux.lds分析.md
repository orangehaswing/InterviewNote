# Kernel启动流程(零) - vmlinux.lds分析

# 一. 基础部分

## 1. 段说明

- text段 

  代码段，通常是指用来存放程序执行代码的一块内存区域。这部分区域的大小在程序运行前就已经确定。

- data段 

  数据段，通常是指用来存放程序中已初始化的全局变量的一块内存区域。数据段属于静态内存分配。

- bss段 

  通常是指用来存放程序中未初始化的全局变量和静态变量的一块内存区域。BSS段属于静态内存分配。

- init段 

  linux定义的一种初始化过程中才会用到的段，一旦初始化完成，那么这些段所占用的内存会被释放掉，后续会继续说明

## 2. 各种地址说明

- 地址解释 

  - 加载地址：程序中指令和变量等加载到RAM上的地址。
  - 运行地址：CPU执行一条程序中指令时的执行地址，也就是PC寄存器中的值。更简单的讲，就是要寻址到一个指令或者变量所使用的地址。
  - 链接地址：链接过程中链接器为指令和变量分配的地址。

- 地址之间联系

  注意，运行地址并不一定完全和链接地址相同，也不一定完全和加载地址相同。

  - 如果没有打开MMU，并且使用的是位置相关设计，那么加载地址、运行地址、链接地址三者需要一致。 

    需要保证链接地址和加载地址是一致的，否则会导致程序跑飞，从uboot上可以理解。

  - 当打开MMU之前，如果使用的是位置无关设计，那么运行地址和加载地址应该是一致的 

    例如kernel在打开mmu之前，使用的是位置无关设计，其运行地址和加载地址一致。关于位置无关设计请自行度娘。

  - 如果打开了MMU，那么运行地址和链接地址相同。 

    硬件会根据运行地址进行计算并自动寻址到对应的加载地址上。


- 举例说明 

  以s5pv210为例 

  - uboot（BL2）阶段并没有打开MMU，并且其使用的是位置相关设计，所以其加载地址和链接地址都需要设置成相同， 也就是加载地址是0x23E00000，链接地址也是0x23E00000，运行地址也就和这两者一致，也就是


-   kernel启动过程中，在MMU打开之前，使用的是位置无关设计， 

    内核镜像加载地址是0x20008000，链接地址是0xc0008000,运行地址是0x20008000.

-   打开MMU之后， 内核镜像加载地址是0x20008000，链接地址是0xc0008000,运行地址是0xc0008000.

# 二. 链接脚本语言

一个简单的vmlinux.lds.S的例子 

```
SECTIONS
{
       . = 0x10000;
       .text : { *(.text) }
       . = 0x8000000;
       .data : { *(.data) }
       .bss : { *(.bss) }
}
```

- 第3行指示，链接地址为0x100000；即指定了后面的text段的链接地址 
- 第4行指示：输出文件的text段内容由所有目标文件(，理解为所有的.o文件，.o)的text段组成； 注意理解.text : { (.text) }的用法，冒号前面.text表示这个段的名称，{.text}则表示所有目标文件的text段.__ 
- 第5行指示：链接地址变了，变为0x8000000；即重新指定了后面的data段的链接地址； 
- 第6行指示：输出文件的data端由所有目标文件的data段组成； 
- 第7行指示：输出文件的bss端由所有目标文件的bss段组成；

# 三. vmlinux.lds.S分析

## 0. 一些有助于我们分析vmlinux.lds.S的东西

kernel在启动过程中会打印一些和memory信息相关的log

```
Memory: 514112K/524288K available (2128K kernel code, 82K rwdata, 696K rodata, 1024K init, 204K bss, 10176K reserved, 0K cma-reserved)
Virtual kernel memory layout:
    vector  : 0xffff0000 - 0xffff1000   (   4 kB)
    fixmap  : 0xffc00000 - 0xfff00000   (3072 kB)
    vmalloc : 0xe0800000 - 0xff800000   ( 496 MB)
    lowmem  : 0xc0000000 - 0xe0000000   ( 512 MB)
    modules : 0xbf000000 - 0xc0000000   (  16 MB)
      .text : 0xc0008000 - 0xc03c228c   (3817 kB)
      .init : 0xc0400000 - 0xc0500000   (1024 kB)
      .data : 0xc0500000 - 0xc0514ba0   (  83 kB)
       .bss : 0xc0514ba0 - 0xc0547d74   ( 205 kB)
```

这部分log在mm/page_alloc.c中的mem_init_print_info函数中打印。 
这里我们着重关注连接过程中的一些段的位置：

```
      .text : 0xc0008000 - 0xc03c228c   (3817 kB)
      .init : 0xc0400000 - 0xc0500000   (1024 kB)
      .data : 0xc0500000 - 0xc0514ba0   (  83 kB)
       .bss : 0xc0514ba0 - 0xc0547d74   ( 205 kB)
```

- 编译之后生成的System.map文件 
  System.map是内核的内核符号表，在这里可以找到函数地址，变量地址，包括一些链接过程中的地址定义等等， 
  build/out/linux/System.map（这里列出一些关键部分）

```
c0008000 T _text
c0008000 T stext
c0100000 T _stext
c03c228c T _etext
c0400000 T __init_begin
c0500000 D __init_end
c0500000 D _data
c0500000 D _sdata
c0514ba0 D _edata
c0514ba0 B __bss_start
c0547d74 B __bss_stop
c0547d74 B _end
```

可以看出和上述（1）中是匹配的。

- 通过反汇编命令对vmlinux进行反汇编，可以解析出详细的汇编代码，包括了一些地址 
  指令如下：

```
./arm-none-linux-gnueabi-4.8/bin/arm-none-linux-gnueabi-objdump -D out/linux/vmlinux > vmlinux_objdump.txt
```

- 通过arm-readelf -s vmlinux查看各个段的布局

```
hlos@node4:linux$ readelf -S vmlinux
There are 39 section headers, starting at offset 0x1ed6388:

Section Headers:
  [Nr] Name              Type            Addr     Off    Size   ES Flg Lk Inf Al
  [ 0]                   NULL            00000000 000000 000000 00      0   0  0
  [ 1] .head.text        PROGBITS        80008000 008000 000220 00  AX  0   0 32
  [ 2] .text             PROGBITS        80100000 010000 1f8ba0 00  AX  0   0 64
  [ 3] .fixup            PROGBITS        802f8ba0 208ba0 000028 00  AX  0   0  4
  [ 4] .rodata           PROGBITS        80300000 210000 0952e8 00   A  0   0 64
  [ 5] __bug_table       PROGBITS        803952e8 2a52e8 002418 00   A  0   0  4
  [ 6] __ksymtab         PROGBITS        80397700 2a7700 0042d0 00   A  0   0  4
```

## 1. 整体的段结构

vmlinux.lds.S的段基本上会按照如下格式进行组织。 参考include/asm-generic/vmlinux.lds.h注释部分

```
* OUTPUT_FORMAT(...)
* OUTPUT_ARCH(...)
* ENTRY(...)
* SECTIONS
* {
 *      . = START;
 *      __init_begin = .;
 *      HEAD_TEXT_SECTION
 *      INIT_TEXT_SECTION(PAGE_SIZE)
 *      INIT_DATA_SECTION(...)
 *      PERCPU_SECTION(CACHELINE_SIZE)
 *      __init_end = .;
*
 *      _stext = .;
 *      TEXT_SECTION = 0
 *      _etext = .;
*
 *      _sdata = .;
 *      RO_DATA_SECTION(PAGE_SIZE)
 *      RW_DATA_SECTION(...)
 *      _edata = .;
*
 *      EXCEPTION_TABLE(...)
 *      NOTES
*
 *      BSS_SECTION(0, 0, 0)
 *      _end = .;
*
 *      STABS_DEBUG
 *      DWARF_DEBUG
*
 *      DISCARDS                // must be the last
* }
*
* [__init_begin, __init_end] is the init section that may be freed after init
 *      // __init_begin and __init_end should be page aligned, so that we can
 *      // free the whole .init memory
* [_stext, _etext] is the text section
* [_sdata, _edata] is the data section
*
* Some of the included output section have their own set of constants.
* Examples are: [__initramfs_start, __initramfs_end] for initramfs and
 *               [__nosave_begin, __nosave_end] for the nosave data
*/
```

如上述描述，主要分成了几个区间 

* __init_begin - __init_end区间： 
  内核把一些初始化才会使用到的段（并不局限于数据段或者代码段，也可以是自己定义的段），简称初始化相关段，放在这个区间里，一旦初始化完成，那么这个区间里的数据或代码在后面就不会被使用，内核会把这部分内存释放出来。 
* _stext - _etext区间： 
  存放内核的代码段，正文 
* _sdata - _edata区间： 
  存放data段，包括只读data段和可读可写数据段。 
* bss段
## 2. __init_begin - __init_end区间定义段

arch/arm/kernel/vmlinux.lds.S

```
     __init_begin = .;
...
        INIT_TEXT_SECTION(8)
        .exit.text : {
                ARM_EXIT_KEEP(EXIT_TEXT)
        }
        .init.proc.info : {
                ARM_CPU_DISCARD(PROC_INFO)
        }
        .init.arch.info : {
                __arch_info_begin = .;
                *(.arch.info.init)
                __arch_info_end = .;
        }
...
        .exit.data : {
                ARM_EXIT_KEEP(EXIT_DATA)
        }
        __init_end = .;
```

在__init_begin和__init_end之间定义了很多初始化过程中会使用到的段。具体例子在后面会说明。 

从System.map可以看到对应地址如下：

```
c0400000 T __init_begin
c0500000 D __init_end
```

- *疑问：为什么exit也放在这里？*
- *补充知识*： 
  这个区间的内存会在初始化完成后被free，具体代码在init/main.c

```
static int __ref kernel_init(void *unused)
{
    free_initmem();
}
void free_initmem(void)
{
    poison_init_mem(__init_begin, __init_end - __init_begin);
}
```

## 3. _stext - _etext区间定义段

```
   .head.text : {
                _text = .;
                HEAD_TEXT
        }
        .text : {                       /* Real text segment            */
                _stext = .;             /* Text and read-only data      */
                        IRQENTRY_TEXT
                        SOFTIRQENTRY_TEXT
                        TEXT_TEXT
                        SCHED_TEXT
                        LOCK_TEXT
                        HYPERVISOR_TEXT
                        KPROBES_TEXT
        _etext = .;                     /* End of text and rodata section */
```

注意_stext和_etext的定义位置。但是真正的文本段是从_text开始的。 各部分的代码段都被放到了这个区间，注意，只读数据段也放到这里来了。 

从System.map可以看到对应地址如下：

```
c0008000 T _text
c0008000 T stext
c0100000 T _stext
c03c228c T _etext
```

## 4. _sdata - _edata区间

```
        __data_loc = .;
        .data : AT(__data_loc) {
                _data = .;              /* address in memory */
                _sdata = .;
                INIT_TASK_DATA(THREAD_SIZE)
                NOSAVE_DATA
                CACHELINE_ALIGNED_DATA(L1_CACHE_BYTES)
                READ_MOSTLY_DATA(L1_CACHE_BYTES)
                DATA_DATA
                CONSTRUCTORS
                _edata = .;
        }
        _edata_loc = __data_loc + SIZEOF(.data);
```

注意_sdata和_edata的定义位置。 各部分的数据段都被放到了这个区间。 

从System.map可以看到对应地址如下：

```
c0500000 D _data
c0500000 D _sdata
c0514ba0 D _edata
```

## 5. bss段定义

```
        BSS_SECTION(0, 0, 0)
        _end = .;
include/asm-generic/vmlinux.lds.h
#define BSS_SECTION(sbss_align, bss_align, stop_align)                  \
        . = ALIGN(sbss_align);                                          \
        VMLINUX_SYMBOL(__bss_start) = .;                                \
        SBSS(sbss_align)                                                \
        BSS(bss_align)                                                  \
        . = ALIGN(stop_align);                                          \
        VMLINUX_SYMBOL(__bss_stop) = .;
```

从System.map看出对应地址如下：

```
c0514ba0 B __bss_start
c0547d74 B __bss_stop
```

# 四. vmlinux.lds.S更多说明

## 1. 入口

有很多不同的方法来设置入口点.链接器会通过按顺序尝试一下方法来设置入口点,如果成功了,就会停止. 

- ’-e’ 入口命令行选项 
- 链接脚本中的ENTRY(SYMBOL)命令 
- 如果定义了start,就使用start的值 
- 如果存在就使用’.text’段的首地址 
- 地址’0’ 

arm/arch/kernel/vmlinux.lds.S指定入口地址如下：

```
ENTRY(stext)
```

说明其入口地址是stext，在arch/arm/kernel/head.S中。 
注意：也就是说kernel启动的入口在这里，后续分析kernel启动流程就是从这里开始分析的。

## 2. 连接地址

为什么stext的地址是0xc0008000呢？（通过System.map查看的）。 

链接器是通过vmlinux.lds.S链接脚本来进行地址定义的。但是如果起始地址不为0的话，我们需要在链接脚本中为其指定一个起始地址。 

arm/arch/kernel/vmlinux.lds.S指定起始连接地址如下（所谓的起始连接地址就是在入口的时候对’.’进行赋值）：

```
      . = PAGE_OFFSET + TEXT_OFFSET;
```

- PAGE_OFFSET表示内核空间的起始地址。 
  定义位置如下： 
  ./arch/arm/include/asm/memory.h

```
/* PAGE_OFFSET - the virtual address of the start of the kernel image */
#define PAGE_OFFSET        UL(CONFIG_PAGE_OFFSET)
```

- CONFIG_PAGE_OFFSET在配置Kconfig的时候会被设置 

  arch/arm/Kconfig

```
config PAGE_OFFSET
        hex 
        default PHYS_OFFSET if !MMU
        default 0x40000000 if VMSPLIT_1G
        default 0x80000000 if VMSPLIT_2G
        default 0xB0000000 if VMSPLIT_3G_OPT
        default 0xC0000000
```

默认情况下是0xC0000000。可以通过配置VMSPLIT来进行修改。 

* TEXT_OFFSET表示内核在RAM中的起始位置相对于RAM起始地址偏移。 
  定义位置如下： 
  ./arch/arm/Makefile


```
# The byte offset of the kernel image in RAM from the start of RAM.
TEXT_OFFSET := $(textofs-y)
# Text offset. This list is sorted numerically by address in order to
# provide a means to avoid/resolve conflicts in multi-arch kernels.
textofs-y       := 0x00008000
```

也就是说默认情况下是0x00008000。 

拓展：为什么要有0x8000的偏移？ 

因为kernel镜像的前16K需要预留出来给初始化页表项使用。这里先暂时了解一下，后续研究kernel启动流程会遇到，再学习。 对应代码arch/arm/kernel/head.S，这里的注释也提到了。

```
/*
* swapper_pg_dir is the virtual address of the initial page table.
 * We place the page tables 16K below KERNEL_RAM_VADDR.  Therefore, we must
 * make sure that KERNEL_RAM_VADDR is correctly set.  Currently, we expect
* the least significant 16 bits to be 0x8000, but we could probably
* relax this restriction to KERNEL_RAM_VADDR >= PAGE_OFFSET + 0x4000.
*/
#define KERNEL_RAM_VADDR    (PAGE_OFFSET + TEXT_OFFSET)
#if (KERNEL_RAM_VADDR & 0xffff) != 0x8000
#error KERNEL_RAM_VADDR must start at 0xXXXX8000
#endif
```
