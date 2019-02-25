# uboot流程(五) - uboot启动流程

# 一. uboot说明

## 1. uboot要做的事情

CPU初始刚上电的状态。需要小心的设置好很多状态，包括cpu状态、中断状态、MMU状态等等。其次，就是要根据硬件资源进行板级的初始化，代码重定向等等。最后，就是进入命令行状态，等待处理命令。 

在armv7架构的uboot，主要需要做如下事情:

- arch级的初始化
  - 关闭中断，设置svc模式
  - 禁用MMU、TLB
  - 关键寄存器的设置，包括时钟、看门狗的寄存器
- 板级的初始化
  - 堆栈环境的设置
  - 代码重定向之前的板级初始化，包括串口、定时器、环境变量、I2C/SPI等等的初始化
  - 进行代码重定向
  - 代码重定向之后的板级初始化，包括板级代码中定义的初始化操作、emmc、nand flash、网络、中断等等的初始化。
  - 进入命令行状态，等待终端输入命令以及对命令进行处理

## 2. 疑问

- 在前面的文章中虽然已经说明了，在spl的阶段中已经对arch级进行了初始化了，为什么uboot里面还要对arch再初始化一遍？ 

  回答：spl对于启动uboot来说并不是必须的，在某些情况下，上电之后uboot可能在ROM上或者flash上开始执行而并没有使用spl。这些都是取决于平台的启动机制。因此uboot并不会考虑spl是否已经对arch进行了初始化操作，uboot会完整的做一遍初始化动作，以保证cpu处于所要求的状态下。

- 和spl在启动过程的差异在哪里？ 

  回答：以tiny210而言，前期arch的初始化流程基本上是一致的，出现本质区别的是在board_init_f开始的。

  - spl的board_init_f是由board自己实现相应的功能，例如tiny210则是在board/samsung/tiny210/board.c中。其主要实现了复制uboot到ddr中，并且跳转到uboot的对应位置上。一般spl在这里就可以完成自己的工作了。

  - uboot的board_init_f是在common下实现的，其主要实现uboot relocate前的板级初始化以及relocate的区域规划，其还需要往下走其他初始化流程。

## 3. 代码入口

`project-X/u-boot/arch/arm/cpu/u-boot.lds`

```
ENTRY(_start)
```

所以uboot-spl的代码入口函数是 `_start` ,对应于路径`project-X/u-boot/arch/arm/lib/vector.S`的_start，后续就是从这个函数开始分析。

# 二. 代码整体流程

## 1. 首先看一下主枝干的流程（包含了arch级的初始化）

在arch级初始化是和spl完全一致的 

_start———–>reset————–>关闭中断 

………………………………| 

………………………………———->cpu_init_cp15———–>关闭MMU,TLB 

………………………………| 

………………………………———->cpu_init_crit————->lowlevel_init————->关键寄存器的配置和初始化

………………………………| 

………………………………———->_main————–>进入板级初始化

具体看下面

## 2. 板级初始化的流程

_main————–>board_init_f_alloc_reserve —————>堆栈、GD、early malloc空间的分配 

…………| 

…………————->board_init_f_init_reserve —————>堆栈、GD、early malloc空间的初始化 

…………| 

…………————->board_init_f —————>uboot relocate前的板级初始化以及relocate的区域规划 

…………| 

…………————->relocate_code、relocate_vectors —————>进行uboot和异常中断向量表的重定向 

…………| 

…………————->旧堆栈的清空 

…………| 

…………————->board_init_r —————>uboot relocate后的板级初始化 

…………| 

…………————->run_main_loop —————>进入命令行状态，等待终端输入命令以及对命令进行处理

# 三. arch级初始化代码分析

## 1. _start

上述已经说明了_start是整个uboot的入口，其代码如下： 

`arch/arm/lib/vector.S`

```
_start:
#ifdef CONFIG_SYS_DV_NOR_BOOT_CFG
    .word   CONFIG_SYS_DV_NOR_BOOT_CFG
#endif
    b   reset
```

会跳转到reset中。

## 2. reset

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
@@ 跳转到主函数，也就是板级初始化函数
@@ 下一节中进行说明。
```

## 3. cpu_init_cp15

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

## 4. cpu_init_crit

cpu_init_crit，进行一些关键寄存器的初始化动。其代码核心就是lowlevel_init，如下 
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

所以说lowlevel_init就是这个函数的核心。 lowlevel_init一般是由板级代码自己实现的。但是对于某些平台来说，也可以使用通用的lowlevel_init，其定义在arch/arm/cpu/lowlevel_init.S中 

在lowlevel_init中，我们要实现如下：

- 检查一些复位状态
- 关闭看门狗
- 系统时钟的初始化
- 内存、DDR的初始化
- 串口初始化（可选）
- Nand flash的初始化

下面以tiny210的lowlevel_init为例。这部分代码和平台相关性很强，简单介绍一下即可 

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
@@ 其实，在tiny210的项目中，已经在spl里面对ddr初始化了一遍，这里还是又重新初始化了一遍，从实际测试结果来看，并不影响正常的使用。

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

当串口中打印出‘OK’的字符的时候，说明lowlevel_init已经执行完成。

# 三. 板级初始化代码分析

## 1. _main

板级初始化代码的入口就是_main。从这里开始分析。 去除无关代码部分 

`arch/arm/lib/crt0.S`

```
ENTRY(_main)

/*
 * Set up initial C runtime environment and call board_init_f(0).
 */
    ldr sp, =(CONFIG_SYS_INIT_SP_ADDR)
    bic sp, sp, #7  /* 8-byte alignment for ABI compliance */

    mov r0, sp
    bl  board_init_f_alloc_reserve
    mov sp, r0
    /* set up gd here, outside any C code */
    mov r9, r0
    bl  board_init_f_init_reserve
@@ 以上是堆栈、GD、early malloc空间的分配，具体参考《[uboot] （番外篇）global_data介绍》

    mov r0, #0
    bl  board_init_f
@@ uboot relocate前的板级初始化以及relocate的区域规划，后续小节继续说明
@@ 其中relocate区域规划也可以参考一下《[uboot] （番外篇）uboot relocation介绍》

/*
 * Set up intermediate environment (new sp and gd) and call
 * relocate_code(addr_moni). Trick here is that we'll return
 * 'here' but relocated.
 */

    ldr sp, [r9, #GD_START_ADDR_SP] /* sp = gd->start_addr_sp */
    bic sp, sp, #7  /* 8-byte alignment for ABI compliance */
    ldr r9, [r9, #GD_BD]        /* r9 = gd->bd */
    sub r9, r9, #GD_SIZE        /* new GD is below bd */
    adr lr, here
    ldr r0, [r9, #GD_RELOC_OFF]     /* r0 = gd->reloc_off */
    add lr, lr, r0
    ldr r0, [r9, #GD_RELOCADDR]     /* r0 = gd->relocaddr */
    b   relocate_code
here:
/*
 * now relocate vectors
 */
    bl  relocate_vectors
@@ GD、uboot、异常中断向量表的relocate，可以参考《[uboot] （番外篇）uboot relocation介绍》，这里不详细说明

/* Set up final (full) environment */
    bl  c_runtime_cpu_setup /* we still call old routine here */
@@ 通过操作协处理器的c7寄存器来关闭Icache

    ldr r0, =__bss_start    /* this is auto-relocated! */
    ldr r3, =__bss_end      /* this is auto-relocated! */
    mov r1, #0x00000000     /* prepare zero to clear BSS */
    subs    r2, r3, r0      /* r2 = memset len */
    bl  memset
@@ 因为堆栈段已经被relocate，所以这里需要清空原来的堆栈段的内容

    bl coloured_LED_init
    bl red_led_on
@@ LED灯的初始化，可以不实现，想要实现的话，可以在board里重新实现一个函数定义。

    /* call board_init_r(gd_t *id, ulong dest_addr) */
    mov     r0, r9                  /* gd_t */
    ldr r1, [r9, #GD_RELOCADDR] /* dest_addr */
    /* call board_init_r */
    ldr pc, =board_init_r   /* this is auto-relocated! */
    /* we should not return here. */
@@ uboot relocate后的板级初始化，注意，uboot必须在这里就完成工作，或者在里面实现死循环，不应该返回。
ENDPROC(_main)
```

通过上述，有两个很重要的初始化函数，board_init_f和board_init_r，后续继续说明。

## 2. board_init_f

代码如下： 
`common/board_f.c`

```
void board_init_f(ulong boot_flags)
{
    gd->flags = boot_flags;
    gd->have_console = 0;
// 设置global_data里面的一些标志位

    if (initcall_run_list(init_sequence_f))
        hang();
// 调用initcall_run_list依次执行init_sequence_f函数数组里面的函数，initcall_run_list这里不深究
// 一旦init_sequence_f的函数出错，会导致initcall_run_list返回不为0，而从卡掉
}
```

打开DEBUG宏之后，可以通过log观察哪些init函数被调用，如下log：

```
uboot log中有如下log：
initcall: 23e005a4
根据u-boot.map可以发现对应
 .text.print_cpuinfo
                0x23e005a4        0x8 arch/arm/cpu/armv7/built-in.o
                0x23e005a4                print_cpuinfo
也就是说print_cpuinfo被initcall调用了。
```

所以uboot relocate之前的板级初始化的核心就是init_sequence_f中定义的函数了。如下，这里只做简单的说明，需要的时候再具体分析：

```
static init_fnc_t init_sequence_f[] = {
    setup_mon_len,
// 计算整个镜像的长度gd->mon_len
    initf_malloc,
// early malloc的内存池的设定
    initf_console_record,
// console的log的缓存
    arch_cpu_init,      /* basic arch cpu dependent setup */
// cpu的一些特殊的初始化
    initf_dm,
    arch_cpu_init_dm,
    mark_bootstage,     /* need timer, go after init dm */
    /* TODO: can any of this go into arch_cpu_init()? */
    env_init,       /* initialize environment */
// 环境变量的初始化，后续会专门研究一下关于环境变量的内容
    init_baud_rate,     /* initialze baudrate settings */
// 波特率的初始化
    serial_init,        /* serial communications setup */
// 串口的初始化
    console_init_f,     /* stage 1 init of console */
// console的初始化
    print_cpuinfo,      /* display cpu info (and speed) */
// 打印CPU的信息
    init_func_i2c,
    init_func_spi,
// i2c和spi的初始化

    dram_init,      /* configure available RAM banks */
// ddr的初始化，最重要的是ddr ram size的设置！！！！gd->ram_size
// 如果说uboot是在ROM、flash中运行的话，那么这里就必须要对DDR进行初始化
//========================================
    setup_dest_addr,
    reserve_round_4k,
    reserve_trace,
    setup_machine,
    reserve_global_data,
    reserve_fdt,
    reserve_arch,
    reserve_stacks,
// ==以上部分是对relocate区域的规划，具体参考《[uboot] （番外篇）uboot relocation介绍》
    setup_dram_config,
    show_dram_config,
    display_new_sp,
    reloc_fdt,
    setup_reloc,
// relocation之后gd一些成员的设置
    NULL,
};
```

注意，必须保证上述的函数都正确地返回0值，否则会导致hang。

## 3. board_init_r

代码如下： 
`common/board_r.c`

```
void board_init_r(gd_t *new_gd, ulong dest_addr)
{
    if (initcall_run_list(init_sequence_r))
        hang();
// 调用initcall_run_list依次执行init_sequence_r函数数组里面的函数，initcall_run_list这里不深究
// 一旦init_sequence_r的函数出错，会导致initcall_run_list返回不为0，而从卡掉

    /* NOTREACHED - run_main_loop() does not return */
    hang();
// uboot要求在这个函数里面终止一切工作，或者进入死循环，一旦试图返回，则直接hang。
}
```

所以uboot relocate之前的板级初始化的核心就是init_sequence_r中定义的函数了。
如下，这里只做简单的说明，需要的时候再具体分析： 
`common/board_r.c`

```
init_fnc_t init_sequence_r[] = {
    initr_trace,
// trace相关的初始化
    initr_reloc,
// gd中一些关于relocate的标识的设置
    initr_reloc_global_data,
// relocate之后，gd中一些的成员的重新设置
    initr_malloc,
// malloc内存池的设置
    initr_console_record,
    bootstage_relocate,
    initr_bootstage,
#if defined(CONFIG_ARM) || defined(CONFIG_NDS32)
    board_init, /* Setup chipselects */
// 板级自己需要的特殊的初始化函数，如board/samsung/tiny210/board.c中定义了board_init这个函数
#endif
    stdio_init_tables,
    initr_serial,
// 串口初始化
    initr_announce,
// 打印uboot运行位置的log
    initr_logbuffer,
// logbuffer的初始化
    power_init_board,
#ifdef CONFIG_CMD_NAND
    initr_nand,
// 如果使用nand flash，那么这里需要对nand进行初始化
#endif
#ifdef CONFIG_GENERIC_MMC
    initr_mmc,
// 如果使用emmc，那么这里需要对nand进行初始化
#endif
    initr_env,
// 初始化环境变量
    initr_secondary_cpu,
    stdio_add_devices,
    initr_jumptable,
    console_init_r,     /* fully init console as a device */
    interrupt_init,
// 初始化中断
#if defined(CONFIG_ARM) || defined(CONFIG_AVR32)
    initr_enable_interrupts,
// 使能中断
#endif
    run_main_loop,
// 进入一个死循环，在死循环里面处理终端命令。
};
```

最终，uboot运行到了run_main_loop，并且在run_main_loop进入命令行状态，等待终端输入命令以及对命令进行处理。 
到此，uboot流程也就完成了。



























