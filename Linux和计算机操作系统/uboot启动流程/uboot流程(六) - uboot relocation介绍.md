# u-boot流程(六) - uboot relocation介绍

# 一. relocate介绍

## 1. uboot的relocate

uboot的relocate动作就是指uboot的重定向动作，也就是将uboot自身镜像拷贝到ddr上的另外一个位置的动作。

## 2. uboot为什么要进行relocate

考虑以下问题 

- 在某些情况下，uboot是在某些只读存储器上运行，比如ROM、nor flash等等。需要将这部分代码拷贝到DDR上才能完整运行uboot。 （当然，如果我们在spl阶段就把uboot拷贝到ddr上，就不会有这种情况。但是uboot本身就是要考虑各种可能性） 
- 一般会把kernel放在ddr的低端地址上。

考虑到以上情况，uboot的relocation动作会把自己本身relocate到ddr上（前提是在SPL的过程中或者在dram_init中已经对ddr进行初始化了），并且会relocate到ddr的顶端地址使之不会和kernel的冲突。

## 3. uboot的一些注意事项

既然uboot会把自身relocate到ddr的其他位置上，那么相当于执行地址也会发生变化。也就是要求uboot既要能在relocate正常执行，也要能在relocate之后正常执行。这就涉及到uboot需要使用“位置无关代码”技术，也就是Position independent code技术。

# 二. “位置无关代码”介绍及其原理

## 1. 什么是“位置无关代码”

“位置无关代码”是指无论代码加载到内存上的什么地址上，都可以被正常运行。也就是当加载地址和连接地址不一样时，CPU也可以通过相对寻址获得到正确的指令地址。

## 2. 如何生成“位置无关代码”

（1）生成位置无关代码分成两部分 

- 首先是编译源文件的时候，需要将其编译成位置无关代码，主要通过gcc的-fpic选项（也有可能是fPIC,fPIE, mword-relocations选项） 
- 其次是连接时要将其连接成一个完整的位置无关的可执行文件，主要通过ld的-fpie选项

（2）ARM在如何生成“位置无关代码” 

- 编译PIC代码 
  在uboot流程(四) — uboot编译流程中，我们知道gcc的编译选项如下：

```
c_flags=-Wp,-MD,arch/arm/mach-s5pc1xx/.clock.o.d -nostdinc -isystem /home/disk3/xys/temp/project-x/build/arm-none-linux-gnueabi-4.8/bin/../lib/gcc/arm-none-linux-gnueabi/4.8.3/include -Iinclude -I/home/disk3/xys/temp/project-x/u-boot/include -I/home/disk3/xys/temp/project-x/u-boot/arch/arm/include -include /home/disk3/xys/temp/project-x/u-boot/include/linux/kconfig.h -I/home/disk3/xys/temp/project-x/u-boot/arch/arm/mach-s5pc1xx -Iarch/arm/mach-s5pc1xx -D__KERNEL__ -D__UBOOT__ -Wall -Wstrict-prototypes -Wno-format-security -fno-builtin -ffreestanding -Os -fno-stack-protector -fno-delete-null-pointer-checks -g -fstack-usage -Wno-format-nonliteral -D__ARM__ -marm -mno-thumb-interwork -mabi=aapcs-linux -mword-relocations -fno-pic -mno-unaligned-access -ffunction-sections -fdata-sections -fno-common -ffixed-r9 -msoft-float -pipe -march=armv7-a -I/home/disk3/xys/temp/project-x/u-boot/arch/arm/mach-s5pc1xx/include -DKBUILD_STR(s)=#s -DKBUILD_BASENAME=KBUILD_STR(clock) -DKBUILD_MODNAME=KBUILD_STR(clock)
```

重点关注`“-mword-relocations -fno-pic”`。 由于使用pic时movt / movw指令会硬编码16bit的地址域，而uboot的relocation并不支持这个， 所以arm平台使用mword-relocations来生成位置无关代码。-fno-pic则表示不使用pic。 

如下`./arch/arm/config.mk`

```
# The movt / movw can hardcode 16 bit parts of the addresses in the
# instruction. Relocation is not supported for that case, so disable
# such usage by requiring word relocations.
PLATFORM_CPPFLAGS += $(call cc-option, -mword-relocations)
PLATFORM_CPPFLAGS += $(call cc-option, -fno-pic)
```

- 生成PIE可执行文件 
  在 uboot流程(四) — uboot编译流程中，我们知道ld的连接选项如下：

```
LDFLAGS_u-boot=-pie --gc-sections -Bstatic -Ttext 0x23E00000
```

-pie选项用于生成PIE位置无关可执行文件。

## 3. “位置无关代码”原理

“位置无关代码”主要是通过使用一些只会使用相对地址的指令实现，比如“b”、“bl”、“ldr”、“adr”等等。 对于一些绝对地址符号（例如已经初始化的全局变量），会将其以label的形式放在每个函数的代码实现的末端。 同时，在链接的过程中，会把这些label的地址统一维护在.rel.dyn段中，当relocation的时候，方便对这些地址的fix。

## 4. .rel.dyn段介绍和使用

前面也说了： 对于一些绝对地址符号（例如已经初始化的全局变量），会将其以label的形式放在每个函数的代码实现的末端。 同时，在链接的过程中，会把这些label的地址统一维护在.rel.dyn段中，当relocation的时候，方便对这些地址的fix。 
这边简单的给个例子： 

`u-boot/common/board_f.c`中

```
static init_fnc_t init_sequence_f[] = {
// 这里定义了全局变量init_sequence_f
｝

void board_init_f(ulong boot_flags)
{
    if (initcall_run_list(init_sequence_f))
// 这里使用了全局变量init_sequence_f
        hang();
｝
```

通过如下命令对编译生成的u-boot

```
arm-none-linux-gnueabi-objdump -D u-boot > uboot_objdump.txt
```

board_init_f和init_sequence_f相关的连接地址如下：

```
Disassembly of section .text:
23e08428 <board_init_f>:
23e08438:       e59f000c        ldr     r0, [pc, #12]   ; 23e0844c <board_init_f+0x24> 
// 通过ldr     r0, [pc, #12]，相当于是ldr r0,[23e0844c] ,
// 也就是通过后面的label项，获得了init_sequence_f的地址。

23e0844c:       23e35dcc        mvncs   r5, #204, 26    ; 0x3300
// 23e0844c:       23e35dcc 是一个label项，23e0844c表示这个label的地址，23e35dcc表示这个label里面的值，也就是全局变量23e35dcc的地址。


Disassembly of section .data:
23e35dcc <init_sequence_f>:
// 全局变量init_sequence_f的地址在23e35dcc 

Disassembly of section .rel.dyn:
23e37b88:       23e0844c        mvncs   r8, #76, 8      ; 0x4c000000
23e37b8c:       00000017        andeq   r0, r0, r7, lsl r0 
// 把init_sequence_f的label的地址存在.rel.dyn段中，方便后续relocation的时候，对label中的绝对变量地址进行整理修改。
```

**各个符号的地址意义**

- 23e08428，是board_init_f的地址
- 23e35dcc，是init_sequence_f的地址
- 23e0844c，是board_init_f为init_sequence_f做的label的地址，所以其值是init_sequence_f的地址，也就是23e35dcc
- 23e37b88，把init_sequence_f的label的地址存放在.rel.dyn段中的这个位置

**根据上述对全局变量的寻址进行简单的说明** 

当board_init_f读取init_sequence_f时，会通过相对偏移获取init_sequence_f的label的地址（23e0844c），再从23e0844c中获取到init_sequence_f的地址（23e35dcc）。

综上，当uboot对自身进行relocate之后，此时全局变量的绝对地址已经发生变化，如果函数按照原来的label去获取全局变量的地址的时候，这个地址其实是relocate之前的地址。因此，在relocate的过程中需要对全局变量的label中的地址值进行修改，所以uboot将这些label的地址全部维护在.rel.dyn段中，然后再统一对.rel.dyn段指向的label进行修改。后续代码可以看出来。

# 三. uboot relocate代码介绍

## 1. uboot relocate地址和布局。

前面已经说明，uboot的relocation动作会把自己本身relocate到ddr上（前提是在SPL的过程中或者在dram_init中已经对ddr进行初始化了），并且会relocate到ddr的顶端地址使之不会和kernel的冲突。 但是relocate过程中，并不是直接把uboot直接放到ddr的顶端位置，而是会有一定的布局，预留一些空间给其他一些需要固定空间的功能使用。

- uboot relocate从高地址到低地址布局如下（并不是所有的区域都是需要的，可以根据宏定义来确定），注意，对应区域的size在这个时候都是确定的，不会发生变化了。

|   relocate区域   |           size           |
| :------------: | :----------------------: |
|    prom页表区域    |         8192byte         |
|   logbuffer    |     LOGBUFF_RESERVE      |
|     pram区域     |     CONFIG_PRAM<<10      |
|    round_4k    |         用于4kb对齐          |
|    mmu页表区域     |       PGTABLE_SIZE       |
|  video buffer  |   不关心。但是是确定的。不会随着代码变化    |
|   lcd buffer   |   不关心。但是是确定的。不会随着代码变化    |
|  trace buffer  | CONFIG_TRACE_BUFFER_SIZE |
|   uboot代码区域    |  gd->mon_len，并且对齐4KB对齐   |
|   malloc内存池    |     TOTAL_MALLOC_LEN     |
|  Board Info区域  |       sizeof(bd_t)       |
| 新global_data区域 |       sizeof(gd_t)       |
|     fdt区域      |       gd->fdt_size       |
|       对齐       |          16b对齐           |
|      堆栈区域      |           无限制            |

## 2. relocate代码流程

主要是分成如下流程 

- 对relocate进行空间规划 
- 计算uboot代码空间到relocation的位置的偏移 
- relocate旧的global_data到新的global_data的空间上 
- relocate旧的uboot代码空间到新的空间上去 
- 修改relocate之后全局变量的label。（不懂的话参考第二节） 
- relocate中断向量表

（1）首先看一下relocate的整体代码 

去掉无关代码的代码如下： 

`arch/arm/lib/crt0.S`

```
ENTRY(_main)
    bl  board_init_f
@@ 在board_init_f里面实现了
@@                             （1）对relocate进行空间规划
@@                             （2）计算uboot代码空间到relocation的位置的偏移
@@                             （3）relocate旧的global_data到新的global_data的空间上

    ldr sp, [r9, #GD_START_ADDR_SP] /* sp = gd->start_addr_sp */
    bic sp, sp, #7  /* 8-byte alignment for ABI compliance */
    ldr r9, [r9, #GD_BD]        /* r9 = gd->bd */
    sub r9, r9, #GD_SIZE        /* new GD is below bd */
@@ 把新的global_data地址放在r9寄存器中

    adr lr, here
    ldr r0, [r9, #GD_RELOC_OFF]     /* r0 = gd->reloc_off */
    add lr, lr, r0
@@ 计算返回地址在新的uboot空间中的地址。b调用函数返回之后，就跳到了新的uboot代码空间中。

    ldr r0, [r9, #GD_RELOCADDR]     /* r0 = gd->relocaddr */
@@ 把uboot的新的地址空间放到r0寄存器中，作为relocate_code的参数
    b   relocate_code
@@ 跳转到relocate_code中，在这里面实现了
@@                                       （1）relocate旧的uboot代码空间到新的空间上去
@@                                       （2）修改relocate之后全局变量的label
@@ 注意，由于上述已经把lr寄存器重定义到uboot新的代码空间中了，所以返回之后，就已经跳到了新的代码空间了！！！！！！

    bl  relocate_vectors
@@ relocate中断向量表
```

**注意上面的注释，从relocate_code返回之后就已经在新的uboot代码空间中运行了。**

这里简单地说明一下`board_init_f`：

```
static init_fnc_t init_sequence_f[] = {
#ifdef CONFIG_SANDBOX
    setup_ram_buf,
#endif
    setup_mon_len,
#ifdef CONFIG_OF_CONTROL
    fdtdec_setup,
#endif
#ifdef CONFIG_TRACE
    trace_early_init,
...
}
// 可以看出init_sequence_f是一个函数指针数组

void board_init_f(ulong boot_flags)
{
    if (initcall_run_list(init_sequence_f))
// 在这里会init_sequence_f里面的函数
        hang();
}
```

（2）对relocate进行空间规划 

布局已经在上面说过了。 其规划只要体现在gd一些指针的设置，如下面所示

——————————————————— <—–(gd->ram_top) 
| 最高的区域 
——————————————————— 
| …… 
——————————————————— 
| uboot代码区域 
——————————————————— <—–(gd->relocaddr) 
| …… 
——————————————————— 
| Board Info区域 
——————————————————— <—–(gd->bd) 
| 新global_data区域 
——————————————————— <—–(gd->new_gd) 
| fdt区域 
——————————————————— <—–(gd->new_fdt) 
| ….. 
——————————————————— <—–(gd->start_addr_sp) 
| 堆栈区域 

在board_init_f中，会依次执行init_sequence_f数组里面函数。其中，和relocate空间规划的函数如下：

```
static init_fnc_t init_sequence_f[] = {
    setup_dest_addr,
#if defined(CONFIG_SPARC)
    reserve_prom,
#endif
#if defined(CONFIG_LOGBUFFER) && !defined(CONFIG_ALT_LB_ADDR)
    reserve_logbuffer,
#endif
#ifdef CONFIG_PRAM
    reserve_pram,
#endif
    reserve_round_4k,
#if !(defined(CONFIG_SYS_ICACHE_OFF) && defined(CONFIG_SYS_DCACHE_OFF)) && \
        defined(CONFIG_ARM)
    reserve_mmu,
#endif
#ifdef CONFIG_DM_VIDEO
    reserve_video,
#else
# ifdef CONFIG_LCD
    reserve_lcd,
# endif
    /* TODO: Why the dependency on CONFIG_8xx? */
# if defined(CONFIG_VIDEO) && (!defined(CONFIG_PPC) || defined(CONFIG_8xx)) && \
        !defined(CONFIG_ARM) && !defined(CONFIG_X86) && \
        !defined(CONFIG_BLACKFIN) && !defined(CONFIG_M68K)
    reserve_legacy_video,
# endif
#endif /* CONFIG_DM_VIDEO */
    reserve_trace,
#if !defined(CONFIG_BLACKFIN)
    reserve_uboot,
#endif
#ifndef CONFIG_SPL_BUILD
    reserve_malloc,
    reserve_board,
#endif
    setup_machine,
    reserve_global_data,
    reserve_fdt,
    reserve_arch,
    reserve_stacks,
```

代码里面都是一些简单的减法以及指针的设置。可以参考上述“区域布局”和指针设置自己看一下代码，这里不详细说明。 

这里说明一下setup_dest_addr，也就是一些指针的初始化。

```
static int setup_dest_addr(void)
{
    debug("Monitor len: %08lX\n", gd->mon_len);
// gd->mon_len表示了整个uboot代码空间的大小，如下
// gd->mon_len = (ulong)&__bss_end - (ulong)_start;
// 在uboot代码空间relocate的时候，relocate的size就是由这里决定

    debug("Ram size: %08lX\n", (ulong)gd->ram_size);
// gd->ram_size表示了ram的size，也就是可使用的ddr的size，在board.c中定义如下
// int dram_init(void)
// {
//  gd->ram_size = PHYS_SDRAM_1_SIZE;也就是0x2000_0000
//  return 0;
// }


#ifdef CONFIG_SYS_SDRAM_BASE
    gd->ram_top = CONFIG_SYS_SDRAM_BASE;
#endif
    gd->ram_top += get_effective_memsize();
    gd->ram_top = board_get_usable_ram_top(gd->mon_len);
// gd->ram_top计算ddr的顶端地址
// CONFIG_SYS_SDRAM_BASE(0x2000_0000+0x2000_0000=0x4000_0000)

    gd->relocaddr = gd->ram_top;
// 从gd->ram_top的位置开始分配
    debug("Ram top: %08lX\n", (ulong)gd->ram_top);
    return 0;
}
```

（3）计算uboot代码空间到relocation的位置的偏移

同样在board_init_f中，调用init_sequence_f数组里面的setup_reloc实现。

```
static int setup_reloc(void)
{
#ifdef CONFIG_SYS_TEXT_BASE
    gd->reloc_off = gd->relocaddr - CONFIG_SYS_TEXT_BASE;
// gd->relocaddr表示新的uboot代码空间的起始地址，CONFIG_SYS_TEXT_BASE表示旧的uboot代码空间的起始地址，二者算起来就是偏移了。
#endif
}
```

（4）relocate旧的global_data到新的global_data的空间上

同样在board_init_f中，调用init_sequence_f数组里面的setup_reloc实现。

```
static int setup_reloc(void)
{
    memcpy(gd->new_gd, (char *)gd, sizeof(gd_t));
// 直接把gd的地址空间拷贝到gd->new_gd中
}
```

（5）relocate旧的uboot代码空间到新的空间上去 

代码在relocate_code中，上述（1）中可以知道此时的r0是uboot的新的地址空间。 

主要目的是把__image_copy_start到__image_copy_end的代码空间拷贝到新的uboot地址空间中。 

```
ENTRY(relocate_code)
    ldr r1, =__image_copy_start /* r1 <- SRC &__image_copy_start */
// 获取uboot代码空间的首地址
    subs    r4, r0, r1      /* r4 <- relocation offset */
// 计算新旧uboot代码空间的偏移
    beq relocate_done       /* skip relocation */
    ldr r2, =__image_copy_end   /* r2 <- SRC &__image_copy_end */
// 获取uboot代码空间的尾地址

copy_loop:
    ldmia   r1!, {r10-r11}      /* copy from source address [r1]    */
    stmia   r0!, {r10-r11}      /* copy to   target address [r0]    */
    cmp r1, r2          /* until source end address [r2]    */
    blo copy_loop
// 把旧代码空间复制到新代码空间中。
```

（6）修改relocate之后全局变量的label 

需要先完全理解第二节““位置无关代码”介绍及其原理” 主要目的是修改label中的地址。 

这里复习一下： 

- 绝对地址符号的地址会放在label中提供位置无关代码使用 
- label的地址会放在.rel.dyn段中 
  综上，当uboot对自身进行relocate之后，此时全局变量的绝对地址已经发生变化，如果函数按照原来的label去获取全局变量的地址的时候，这个地址其实是relocate之前的地址。因此，在relocate的过程中需要对全局变量的label中的地址值进行修改，所以uboot将这些label的地址全部维护在.rel.dyn段中，然后再统一对.rel.dyn段指向的label进行修改。后续代码可以看出来。 
  `.rel.dyn`段部分示例如下：

```
23e37b88:       23e0844c        mvncs   r8, #76, 8      ; 0x4c000000
23e37b8c:       00000017        andeq   r0, r0, r7, lsl r0 
23e37b90:       23e084b4        mvncs   r8, #180, 8     ; 0xb4000000
23e37b94:       00000017        andeq   r0, r0, r7, lsl r0 
23e37b98:       23e084d4        mvncs   r8, #212, 8     ; 0xd4000000
23e37b9c:       00000017        andeq   r0, r0, r7, lsl r0 
23e37ba0:       23e0854c        mvncs   r8, #76, 10     ; 0x13000000
23e37ba4:       00000017        andeq   r0, r0, r7, lsl r0 
```

可以看出.rel.dyn段用了8个字节来描述一个label，其中，高4字节是label地址标识0x17，低4字节就是label的地址。 

所以需要先判断label地址标识是否正确，然后再根据第四字节获取label，对label中的符号地址进行修改。

代码如下：

```
ENTRY(relocate_code)
    /*
     * fix .rel.dyn relocations
     */
    ldr r2, =__rel_dyn_start    /* r2 <- SRC &__rel_dyn_start */
    ldr r3, =__rel_dyn_end  /* r3 <- SRC &__rel_dyn_end */
// __rel_dyn段是由链接器生成的。
// 把__rel_dyn_start放到r2中，把__rel_dyn_end放到r3中

fixloop:
    ldmia   r2!, {r0-r1}        /* (r0,r1) <- (SRC location,fixup) */
// 从__rel_dyn_start开始，加载两个字节到r0和r1中，高字节存在r1中表示标志，低字节存在r0中，表示label地址。
    and r1, r1, #0xff
    cmp r1, #23         /* relative fixup? */
// 比较高4字节是否等于0x17
    bne fixnext
// 不等于的话，说明不是描述label地址，进行下一次循环

// label在relocate uboot的时候也已经复制到了新的uboot地址空间了！！！
// 这里要注意，是对新的uboot地址空间label进行修改！！！
    /* relative fix: increase location by offset */
    add r0, r0, r4
// 获取新的uboot地址空间的label地址，
// 因为r0存的是旧地址空间的label地址，而新地址空间的label地址就是在旧地址空间的label地址加上偏移得到
// r4就是relocate offset，也就是新旧地址空间的偏移

    ldr r1, [r0]
// 从label中获取绝对地址符号的地址，存放在r1中

    add r1, r1, r4
    str r1, [r0]
// 根据前面的描述，我们的目的就是要fix label中绝对地址符号的地址，也就是将其修改为新地址空间的地址
// 所以为r1加上偏移之后，重新存储到label中。
// 后面CPU就可以根据LABEL在新uboot的地址空间中寻址到正确的符号。

fixnext:
    cmp r2, r3
    blo fixloop
```

（7）relocate中断向量表

异常中断向量表的定义如下 

`arch/arm/lib/vectors.S`

```
    .globl  _undefined_instruction
    .globl  _software_interrupt
    .globl  _prefetch_abort
    .globl  _data_abort
    .globl  _not_used
    .globl  _irq
    .globl  _fiq

_undefined_instruction: .word undefined_instruction
_software_interrupt:    .word software_interrupt
_prefetch_abort:    .word prefetch_abort
_data_abort:        .word data_abort
_not_used:      .word not_used
_irq:           .word irq
_fiq:           .word fiq
```

我们知道arm的异常中断向量表需要复制到0x00000000处或者0xFFFF0000处。 当uboot进行relocate之后，其异常处理函数的地址也发生了变化，因此，我们需要把新的异常中断向量表复制到0x00000000处或者0xFFFF0000处。 这部分操作就是在relocate_vectors中进行。

异常中断向量表在uboot代码空间中的地址如下：

```
23e00000 <__image_copy_start>:
23e00000:   ea0000be    b   23e00300 <reset>
23e00004:   e59ff014    ldr pc, [pc, #20]   ; 23e00020 <_undefined_instruction>                                                                                                              
23e00008:   e59ff014    ldr pc, [pc, #20]   ; 23e00024 <_software_interrupt>
23e0000c:   e59ff014    ldr pc, [pc, #20]   ; 23e00028 <_prefetch_abort>
23e00010:   e59ff014    ldr pc, [pc, #20]   ; 23e0002c <_data_abort>
23e00014:   e59ff014    ldr pc, [pc, #20]   ; 23e00030 <_not_used>
23e00018:   e59ff014    ldr pc, [pc, #20]   ; 23e00034 <_irq> 
23e0001c:   e59ff014    ldr pc, [pc, #20]   ; 23e00038 <_fiq> 

// 可以看出以下是异常终端向量表
23e00020 <_undefined_instruction>:
23e00020:   23e00060    mvncs   r0, #96 ; 0x60 
// 其中，23e00020存放的是未定义指令处理函数的地址，也就是23e00060
// 以下以此类推

23e00024 <_software_interrupt>:
23e00024:   23e000c0    mvncs   r0, #192    ; 0xc0 

23e00028 <_prefetch_abort>:
23e00028:   23e00120    mvncs   r0, #8 

23e0002c <_data_abort>:
23e0002c:   23e00180    mvncs   r0, #32

23e00030 <_not_used>:
23e00030:   23e001e0    mvncs   r0, #56 ; 0x38 

23e00034 <_irq>:
23e00034:   23e00240    mvncs   r0, #4 

23e00038 <_fiq>:
23e00038:   23e002a0    mvncs   r0, #10
23e0003c:   deadbeef    cdple   14, 10, cr11, cr13, cr15, {7}

23e00040 <IRQ_STACK_START_IN>:
```

所以异常中断向量表就是从偏移0x20开始的32个字节。

代码如下（去除掉无关代码部分）：

```
ENTRY(relocate_vectors)
    /*
     * Copy the relocated exception vectors to the
     * correct address
     * CP15 c1 V bit gives us the location of the vectors:
     * 0x00000000 or 0xFFFF0000.
     */
@@ 注意看注释，通过cp15协处理器的c1寄存器的V标志来判断cpu从什么位置获取中断向量表，
@@ 换句话说，就是中断向量表应该被复制到什么地方！！！

    ldr r0, [r9, #GD_RELOCADDR] /* r0 = gd->relocaddr */
@@ 获取uboot新地址空间的起始地址，存放到r0寄存器中

    mrc p15, 0, r2, c1, c0, 0   /* V bit (bit[13]) in CP15 c1 */
    ands    r2, r2, #(1 << 13)
    ldreq   r1, =0x00000000     /* If V=0 */
    ldrne   r1, =0xFFFF0000     /* If V=1 */
@@ 获取cp15协处理器的c1寄存器的V标志，当V=0时，cpu从0x00000000获取中断向量表，当V=1时，cpu从0xFFFF0000获取中断向量表
@@ 将该地址存在r1中

    ldmia   r0!, {r2-r8,r10}
    stmia   r1!, {r2-r8,r10}
@@ 前面说了异常中断向量表就是从偏移0x20开始的32个字节。
@@ 所以这里是过滤掉前面的0x20个字节（32个字节，8*4）
@@ 但是不明白为什么还要stmia  r1!, {r2-r8,r10}，理论上只需要让r0的值产生0x20的偏移就可以了才对？？？不明白。

@@ 经过上述两行代码之后，此时r0的值已经偏移了0x20了
    ldmia   r0!, {r2-r8,r10}
    stmia   r1!, {r2-r8,r10}
@@ 继续从0x20开始，获取32个字节，存储到r1指向的地址，也就是cpu获取中断向量表的地址
@@ r2-r8,r10表示从r2到r8寄存器和r10寄存器，一个8个寄存器，每个寄存器有4个字节，所以就从r0指向的地址处获取到了32个字节
@@ 再把 {r2-r8,r10}的值存放到r1指向的地址，也就是cpu获取中断向量表的地址

    bx  lr
@@ 返回
ENDPROC(relocate_vectors)
```

经过上述，uboot relocate就完成了。