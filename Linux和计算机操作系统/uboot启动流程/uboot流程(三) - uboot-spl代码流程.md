# uboot流程(三)-uboot-spl代码流程

# 一、说明

## 1、uboot-spl入口说明

通过uboot-spl编译脚本`project-X/u-boot/arch/arm/cpu/u-boot-spl.lds`

```
ENTRY(_start)
```

所以uboot-spl的代码入口函数是_start 
对应于路径`project-X/u-boot/arch/arm/lib/vector.S的_start`，后续就是从这个函数开始分析。

## 2、CONFIG_SPL_BUILD说明

前面说过，在编译SPL的时候，编译参数会有如下语句： 

`project-X/u-boot/scripts/Makefile.spl`

```
KBUILD_CPPFLAGS += -DCONFIG_SPL_BUILD
```

所以说在编译SPL的代码的过程中，CONFIG_SPL_BUILD这个宏是打开的。 

uboot-spl和uboot的代码是通用的，其区别就是通过CONFIG_SPL_BUILD宏来进行区分的。

# 二、uboot-spl需要做的事情

CPU初始刚上电的状态。需要小心的设置好很多状态，包括cpu状态、中断状态、MMU状态等等。 在armv7架构的uboot-spl，主要需要做如下事情:

- 关闭中断，svc模式
- 禁用MMU、TLB
- 芯片级、板级的一些初始化操作 
  - IO初始化
  - 时钟
  - 内存
  - 选项，串口初始化
  - 选项，nand flash初始化
  - 其他额外的操作
  - 加载BL2，跳转到BL2

上述工作，也就是uboot-spl代码流程的核心。

# 三、代码流程

## 1、代码整体流程

代码整体流程如下，以下列出来的就是spl核心函数。 
_start———–>reset————–>关闭中断 

………………………………| 

………………………………———->cpu_init_cp15———–>关闭MMU,TLB 

………………………………| 

………………………………———->cpu_init_crit————->lowlevel_init————->平台级和板级的初始化

………………………………| 

………………………………———->_main————–>board_init_f_alloc_reserve & board_init_f_init_reserve & board_init_f

………………………………———->加载BL2,跳转到BL2 

board_init_f执行时已经是C语言环境了。在这里需要结束掉SPL的工作，跳转到BL2中。

## 2、_start

上述已经说明了_start是整个spl的入口，其代码如下： 

`arch/arm/lib/vector.S`

```
_start:
#ifdef CONFIG_SYS_DV_NOR_BOOT_CFG
    .word   CONFIG_SYS_DV_NOR_BOOT_CFG
#endif
    b   reset
```

会跳转到reset中。 

注意，spl的流程在reset中就应该被结束，也就是说在reset中，就应该转到到BL2，也就是uboot中了。后面看reset的实现。

## 3、reset

代码如下： 

`arch/arm/cpu/armv7/start.S`

```
    .globl  reset
    .globl  save_boot_params_ret

reset:
    /* Allow the board to save important registers */
    b   save_boot_params
save_boot_params_ret:
    /*
     * disable interrupts (FIQ and IRQ), also set the cpu to SVC32 mode,
     * except if in HYP mode already
     */
    mrs r0, cpsr
    and r1, r0, #0x1f       @ mask mode bits
    teq r1, #0x1a       @ test for HYP mode
    bicne   r0, r0, #0x1f       @ clear all mode bits
    orrne   r0, r0, #0x13       @ set SVC mode
    orr r0, r0, #0xc0       @ disable FIQ and IRQ
    msr cpsr,r0
@@ 以上通过设置CPSR寄存器里设置CPU为SVC模式，禁止中断
@@ 具体操作可以参考《[kernel 启动流程] （第二章）第一阶段之——设置SVC、关闭中断》的分析

    /* the mask ROM code should have PLL and others stable */
#ifndef CONFIG_SKIP_LOWLEVEL_INIT
    bl  cpu_init_cp15
@@ 调用cpu_init_cp15，初始化协处理器CP15,从而禁用MMU和TLB。
@@ 后面会有一小节进行分析

    bl  cpu_init_crit
@@ 调用cpu_init_crit，进行一些关键的初始化动作，也就是平台级和板级的初始化
@@ 后面会有一小节进行分析
#endif

    bl  _main
@@ 跳转到主函数，也就是要加载BL2以及跳转到BL2的主体部分
```

## 4、cpu_init_cp15

cpu_init_cp15主要用于对cp15协处理器进行初始化，其主要目的就是关闭其MMU和TLB。 代码如下(去掉无关部分的代码)： 

`arch/arm/cpu/armv7/start.S`

```
ENTRY(cpu_init_cp15)
    /*
     * Invalidate L1 I/D
     */
    mov r0, #0          @ set up for MCR
    mcr p15, 0, r0, c8, c7, 0   @ invalidate TLBs
    mcr p15, 0, r0, c7, c5, 0   @ invalidate icache
    mcr p15, 0, r0, c7, c5, 6   @ invalidate BP array
    mcr     p15, 0, r0, c7, c10, 4  @ DSB
    mcr     p15, 0, r0, c7, c5, 4   @ ISB
@@ 这里只需要知道是对CP15处理器的部分寄存器清零即可。
@@ 将协处理器的c7\c8清零等等，各个寄存器的含义请参考《ARM的CP15协处理器的寄存器》

    /*
     * disable MMU stuff and caches
     */
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
@@ 通过上述的文章的介绍，我们可以知道cp15的c1寄存器就是MMU控制器
@@ 上述对MMU的一些位进行清零和置位，达到关闭MMU和cache的目的，具体的话去看一下上述文章吧。

ENDPROC(cpu_init_cp15)
```

## 5、cpu_init_crit

`cpu_init_crit`，进行一些关键的初始化动作，也就是平台级和板级的初始化。其代码核心就是lowlevel_init，如下 

`arch/arm/cpu/armv7/start.S`

```
ENTRY(cpu_init_crit)
/*
 * Jump to board specific initialization...
 * The Mask ROM will have already initialized
 * basic memory. Go here to bump up clock rate and handle
 * wake up conditions.
 */
b   lowlevel_init       @ go setup pll,mux,memory
ENDPROC(cpu_init_crit)
```

所以说lowlevel_init就是这个函数的核心。

 lowlevel_init一般是由板级代码自己实现的。但是对于某些平台来说，也可以使用通用的lowlevel_init，其定义在arch/arm/cpu/lowlevel_init.S中 

以tiny210为例，在移植tiny210的过程中，就需要在board/samsung/tiny210下，也就是板级目录下面创建lowlevel_init.S，在内部实现lowlevel_init。（其实只要实现了lowlevel_init了就好，没必要说在哪里是实现，但是通常规范都是创建了lowlevel_init.S来专门实现lowlevel_init函数）。

在lowlevel_init中，我们要实现如下： 

- 检查一些复位状态 
- 关闭看门狗 
- 系统时钟的初始化 
- 内存、DDR的初始化 
- 串口初始化（可选） 
- Nand flash的初始化

`board/samsung/tiny210/lowlevel_init.S`

```
lowlevel_init:
    push    {lr}

    /* check reset status  */

    ldr r0, =(ELFIN_CLOCK_POWER_BASE+RST_STAT_OFFSET)
    ldr r1, [r0]
    bic r1, r1, #0xfff6ffff
    cmp r1, #0x10000
    beq wakeup_reset_pre
    cmp r1, #0x80000
    beq wakeup_reset_from_didle
@@ 读取复位状态寄存器0xE010_a000的值，判断复位状态。

    /* IO Retention release */
    ldr r0, =(ELFIN_CLOCK_POWER_BASE + OTHERS_OFFSET)
    ldr r1, [r0]
    ldr r2, =IO_RET_REL
    orr r1, r1, r2
    str r1, [r0]
@@ 读取混合状态寄存器E010_e000的值，对其中的某些位进行置位，复位后需要对某些wakeup位置1，具体我也没搞懂。

    /* Disable Watchdog */
    ldr r0, =ELFIN_WATCHDOG_BASE    /* 0xE2700000 */
    mov r1, #0
    str r1, [r0]
@@ 关闭看门狗

@@ 这里忽略掉一部分对外部SROM操作的代码

    /* when we already run in ram, we don't need to relocate U-Boot.
     * and actually, memory controller must be configured before U-Boot
     * is running in ram.
     */
    ldr r0, =0x00ffffff
    bic r1, pc, r0      /* r0 <- current base addr of code */
    ldr r2, _TEXT_BASE      /* r1 <- original base addr in ram */
    bic r2, r2, r0      /* r0 <- current base addr of code */
    cmp     r1, r2                  /* compare r0, r1                  */
    beq     1f          /* r0 == r1 then skip sdram init   */
@@ 判断是否已经在SDRAM上运行了，如果是的话，就跳过以下两个对ddr初始化的步骤
@@ 判断方法如下：
@@ 1、获取当前pc指针的地址，屏蔽其低24bit，存放与r1中
@@ 2、获取_TEXT_BASE（CONFIG_SYS_TEXT_BASE）地址，也就是uboot代码段的链接地址，后续在uboot篇的时候会说明，并屏蔽其低24bit
@@ 3、如果相等的话，就跳过DDR初始化的部分

    /* init system clock */
    bl system_clock_init
@@ 初始化系统时钟，后续有时间再研究一下具体怎么配置的

    /* Memory initialize */
    bl mem_ctrl_asm_init
@@ 重点注意：在这里初始化DDR的！！！后续会写一篇文章说明一下s5pv210平台如何初始化DDR.

1:
    /* for UART */
    bl uart_asm_init
@@ 串口初始化，到这里串口会打印出一个'O'字符，后续通过写字符到UTXH_OFFSET寄存器中，就可以在串口上输出相应的字符。

    bl tzpc_init

#if defined(CONFIG_NAND)
    /* simple init for NAND */
    bl nand_asm_init
@@ 简单地初始化一下NAND flash，有可能BL2的镜像是在nand  flash上面的。
#endif

    /* Print 'K' */
    ldr r0, =ELFIN_UART_CONSOLE_BASE
    ldr r1, =0x4b4b4b4b
    str r1, [r0, #UTXH_OFFSET]
@@ 再串口上打印‘K’字符，表示lowlevel_init已经完成

    pop {pc}
@@ 弹出PC指针，即返回。
```

## 6、_main

spl的main的主要目标是调用board_init_f进行先前的板级初始化动作，在tiny210中，主要设计为，加载BL2到DDR上并且跳转到BL2中。DDR在上述lowlevel_init中已经初始化好了。 

由于`board_init_f`是以C语言的方式实现，所以需要先构造C语言环境。 注意：uboot-spl和uboot的代码是通用的，其区别就是通过`CONFIG_SPL_BUILD`宏来进行区分的。 所以以下代码中，我们只列出spl相关的部分，也就是被`CONFIG_SPL_BUILD`包含的部分。 

`arch/arm/lib/crt0.S`

```
ENTRY(_main)

/*
 * Set up initial C runtime environment and call board_init_f(0).
 */
@ 注意看这里的注释，也说明了以下代码的主要目的是，初始化C运行环境，调用board_init_f。
    ldr sp, =(CONFIG_SPL_STACK)
    bic sp, sp, #7  /* 8-byte alignment for ABI compliance */
    mov r0, sp
    bl  board_init_f_alloc_reserve
    mov sp, r0
    /* set up gd here, outside any C code */
    mov r9, r0
    bl  board_init_f_init_reserve

    mov r0, #0
    bl  board_init_f

ENDPROC(_main)
```

代码拆分如下： 

（1）因为后面是C语言环境，首先是设置堆栈

```
    ldr sp, =(CONFIG_SPL_STACK)
@@ 设置堆栈为CONFIG_SPL_STACK

    bic sp, sp, #7  /* 8-byte alignment for ABI compliance */
@@ 堆栈是8字节对齐，2^7bit=2^3byte=8byte

    mov r0, sp
@@ 把堆栈地址存放到r0寄存器中
```

我们已经知道s5pv210的BL1(spl)是运行在IRAM的，并且IRAM的地址空间是0xD002_0000-0xD003_7FFF，IRAM前面的部分放的是BL1的代码部分，所以把IRAM最后的空间用来当作堆栈。 
所以CONFIG_SPL_STACK定义如下： 

`include/configs/tiny210.h`

```
#define CONFIG_SPL_STACK    0xD0037FFF
```

注意：上述还不是最终的堆栈地址，只是暂时的堆栈地址！

（2）为GD分配空间

```
 bl  board_init_f_alloc_reserve
@@ 把堆栈的前面一部分空间分配给GD使用

    mov sp, r0
@@ 重新设置堆栈指针SP

    /* set up gd here, outside any C code */
    mov r9, r0
@@ 保存GD的地址到r9寄存器中
```

注意：虽然sp的地址和GD的地址是一样的，但是堆栈是向下增长的，而GD则是占用该地址后面的部分，所以不会有冲突的问题。 

关于GD，也就是`struct global_data`，可以简单的理解为uboot的全局变量都放在了这里，比较重要，所以后续有会写篇文章说明一下global_data。这里只需要知道在开始C语言环境的时候需要先为这个结构体分配空间。`board_init_f_alloc_reserve`实现如下 

`common/init/board_init.c`

```
ulong board_init_f_alloc_reserve(ulong top)
{
    /* Reserve early malloc arena */
    /* LAST : reserve GD (rounded up to a multiple of 16 bytes) */
    top = rounddown(top-sizeof(struct global_data), 16);
// 现将top（也就是r0寄存器，前面说过存放了暂时的指针地址），减去sizeof(struct global_data)，也就是预留出一部分空间给sizeof(struct global_data)使用。
// rounddown表示向下16个字节对其

    return top;
// 到这里，top就存放了GD的地址，也是SP的地址
//把top返回，注意，返回后，其实还是存放在了r0寄存器中。
}
```

**还有一点，其实GD在spl中没什么使用，主要是用在uboot中，但在uboot中的时候还需要另外分配空间，在讲述uboot流程的时候会说明。**

（3）初始化GD空间 

前面说了，此时r0寄存器存放了GD的地址。

```
  bl  board_init_f_init_reserve
```

board_init_f_init_reserve实现如下 
`common/init/board_init.c `
编译SPL的时候  USE_MEMCPY 宏没有打开，所以我们去掉了_USE_MEMCPY的无关部分。

```
void board_init_f_init_reserve(ulong base)
{
    struct global_data *gd_ptr;
    int *ptr;
    /*
     * clear GD entirely and set it up.
     * Use gd_ptr, as gd may not be properly set yet.
     */

    gd_ptr = (struct global_data *)base;
// 从r0获取GD的地址
    /* zero the area */
    for (ptr = (int *)gd_ptr; ptr < (int *)(gd_ptr + 1); )
        *ptr++ = 0;
// 对GD的空间进行清零
}
```

（4）跳转到板级前期的初始化函数中，如下代码

```
    bl  board_init_f
```

board_init_f需要由板级代码自己实现。 
在这个函数中，tiny210主要是实现了从SD卡上加载了BL2到ddr上，然后跳转到BL2的相应位置上 
tiny210的实现如下： 
board/samsung/tiny210/board.c

```
#ifdef CONFIG_SPL_BUILD
void board_init_f(ulong bootflag)
{
    __attribute__((noreturn)) void (*uboot)(void);
    int val;
#define DDR_TEST_ADDR 0x30000000
#define DDR_TEST_CODE 0xaa
    tiny210_early_debug(0x1);
    writel(DDR_TEST_CODE, DDR_TEST_ADDR);
    val = readl(DDR_TEST_ADDR);
    if(val == DDR_TEST_CODE)
        tiny210_early_debug(0x3);
    else
    {
        tiny210_early_debug(0x2);
        while(1);
    }
// 先测试DDR是否完成

    copy_bl2_to_ddr();
// 加载BL2的代码到ddr上

    uboot = (void *)CONFIG_SYS_TEXT_BASE;
// uboot函数设置为BL2的加载地址上
    (*uboot)();
// 调用uboot函数，也就跳转到BL2的代码中
}
#endif
```

**到此，SPL的任务就完成了，也已经跳到了BL2也就是uboot里面去了。**