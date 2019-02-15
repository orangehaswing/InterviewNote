# kernel 启动流程(七) - 跳转到start_kernel

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
  - 创建临时内核页表的页表项
  - 配置r13寄存器，也就是设置打开MMU之后要跳转到的函数。
  - 使能MMU
  - 跳转到start_kernel，也就是跳转到第二阶段

本文要介绍的是“跳转到start_kernel”的部分。

## 2. 疑问

主要带着以下几个问题去理解

- 跳转到start_kernel的准备动作？环境设置？

# 一. 跳转到start_kernel的准备动作

经过前面几章，我们已经知道MMU已经打开，后续都是在虚拟地址上执行。并且kernel代码段的链接地址都已经映射到对应的物理地址上了。 
接下来的已经调用到__mmap_switched，作跳转到start_kernel的动作。 在跳转到start_kernel前，需要做如下准备动作：

- 数据段的准备 

  通过System.map,数据段对应的连接区域如下：

  ```
  c0500000 D __data_loc
  c0500000 D _data
  c0500000 D _sdata
  c0514ba0 D _edata
  c0514ba0 D _edata_loc
  ```

- 堆栈段的准备 

  因为后续start_kernel之后都是在C语言环境下运行，所以需要对堆栈段进行设置并清空。

  ```
  c0514ba0 B __bss_start
  c0547d74 B __bss_stop
  c0547d74 B _end
  ```

  ​



- 一些后续会访问到的变量的设置 

  因为后续C语言代码会访问到一些变量，并且这些变量的值在启动过程中是存储到寄存器中的。那么就需要把寄存器上的值搬移到对应的变量的地址上。有如下这些变量

  - cpu id（processor ID）
  - machine id（machine type）
  - dtb的指针（atags pointer）
  - 当前进程堆栈指针的设置

- 当前进程堆栈指针的设置 

  因为后续start_kernel之后都是在C语言环境下运行，需要完成其堆栈环境。

然后就可以跳转到start_kernel中了。 后续__mmap_switched的代码也是根据这些准备动作来的。

# 二. 代码分析

直接介绍__mmap_switched的实现。

## 1. __mmap_switched_data

首先了解一下__mmap_switched_data这个数据结构，存放了__mmap_switched过程中使用的变量的地址。 

arch/arm/kernel/head-common.S

```
__mmap_switched_data:
    .long   __data_loc          @ r4
    .long   _sdata              @ r5
    .long   __bss_start         @ r6
    .long   _end                @ r7
    .long   processor_id            @ r4
    .long   __machine_arch_type     @ r5
    .long   __atags_pointer         @ r6
#ifdef CONFIG_CPU_CP15
    .long   cr_alignment            @ r7
#else
    .long   0               @ r7
#endif
    .long   init_thread_union + THREAD_START_SP @ sp
    .size   __mmap_switched_data, . - __mmap_switched_data
```

解释如下：

- __data_loc：数据段存储地址
- _sdata：数据段起始地址
- __bss_start：堆栈段起始地址
- _end：堆栈段结束地址
- processor_id：cpu处理器ID地址，其变量定义在arch/arm/kernel/setup.c中
- __machine_arch_type：machine id地址，其变量定义在arch/arm/kernel/setup.c中
- __atags_pointer：dtb指针的地址，其变量定义在arch/arm/kernel/setup.c中
- cr_alignment：cp15的c1寄存器的值的地址，也就是mmu控制寄存器的值，其变量定义在arch/arm/kernel/entry-armv.S中

## 2. 进入前的寄存器说明

通过前面几章的分析，有几个寄存器专门存放了如下值：

- r0，存放了cp15协处理器c1寄存器的值，也就是MMU控制器的值
- r1，存放了由uboot传过来的mechine id
- r2，存放了dtb的地址
- r9，存放了cpu处理器id

3. 代码分析
--------------------- 
```
/*
 * The following fragment of code is executed with the MMU on in MMU mode,
 * and uses absolute addresses; this is not position independent.
 *
 *  r0  = cp#15 control register
 *  r1  = machine ID
 *  r2  = atags/dtb pointer
 *  r9  = processor ID
 */
    __INIT
__mmap_switched:
    adr r3, __mmap_switched_data
@ 将__mmap_switched_data的地址加载到r3中

    ldmia   r3!, {r4, r5, r6, r7}
@ 将__mmap_switched_data(r3)上的值分别加载到r4、r5、r6、r7寄存器中，__mmap_switched_data前面说明了
@ 经过上述动作，r4、r5、r6、r7寄存器分别存放了如下值
@ r4 -> __data_loc：数据段存储地址
@ r5 -> _sdata：数据段起始地址
@ r6 -> __bss_start：堆栈段起始地址
@ r7 -> _end：堆栈段结束地址

    cmp r4, r5              @ Copy data segment if needed
1:  cmpne   r5, r6
    ldrne   fp, [r4], #4
    strne   fp, [r5], #4
    bne 1b
@ 判断数据段存储地址(r4)和数据段起始地址(r5)
@ 如果不一样的话需要搬移到数据段起始地址(r5)上。

    mov fp, #0              @ Clear BSS (and zero fp)
1:  cmp r6, r7
    strcc   fp, [r6],#4
    bcc 1b
@ 清空堆栈段。
@ 从堆栈段起始地址(r6)开始写入0，一直写到地址为堆栈段结束地址(r7)

 ARM(   ldmia   r3, {r4, r5, r6, r7, sp})
 THUMB( ldmia   r3, {r4, r5, r6, r7}    )
 THUMB( ldr sp, [r3, #16]       )
@ 继续将__mmap_switched_data(r3)上的值分别加载到r4、r5、r6、r7、sp寄存器中，注意是前面r3已经加载过一部分了，地址和__mmap_switched_data已经不一样了。
@ 经过上述动作，r4、r5、r6、r7寄存器分别存放了如下值
@ r4 -> processor_id变量地址：其内容是cpu处理器ID
@ r5 -> __machine_arch_type变量地址：其内容是machine id
@ r6 -> __atags_pointer变量地址：其内容是dtb的地址
@ r7 -> cr_alignment变量地址：其内容是cp15的c1的寄存器的值
@ sp->  init_thread_union + THREAD_START_SP，设置了当前进程的堆栈

    str r9, [r4]            @ Save processor ID
@ 把cpu处理器id(r9)放到processor_id变量中([r4])

    str r1, [r5]            @ Save machine type
@ 把mechine id(r1)存放到__machine_arch_type变量中([r5])

    str r2, [r6]            @ Save atags pointer
@ 把dtb的地址指针(r2)存放到__atags_pointer变量中([r6])

    cmp r7, #0
    strne   r0, [r7]            @ Save control register values
@ 把cp15的c1的寄存器的值(r0)存放到cr_alignment变量中([r7])

    b   start_kernel
@ 跳转到start_kernel中，也就是启动流程的第二阶段。
ENDPROC(__mmap_switched)
```

通过上述，最终跳转到start_kernel，kernel启动过程的第一阶段结束。

———————————————-分割线——————————————————- 
下面对sp指针的设置进行说明 
前面可知 sp-> init_thread_union + THREAD_START_SP 

init_thread_union表示init进程的起始地址，在init/init_task.c中

```
union thread_union init_thread_union __init_task_data =
    { INIT_THREAD_INFO(init_task) };
```

将init_thread_union加上THREAD_START_SP后得到其堆栈地址. 
这部分代码要到进程管理的部分才会更加了解。