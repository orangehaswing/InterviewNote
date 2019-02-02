# kernel 启动流程-proc info的获取

# 零、说明

## 1、kernel启动流程第一阶段简单说明

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

本文要介绍的是“获取CPU ID，提取相应的proc info”的部分。

## 2、疑问

主要带着以下几个问题去理解

- 为什么要获取CPU ID和proc info？

  也就是说proc info存放了什么东西以至于有必要在第一阶段、打开MMU之前就去获取？

- 如何获取对应CPU的proc info？

## 3、对应代码实现

        __HEAD
    ENTRY(stext)
        mrc p15, 0, r9, c0, c0      @ get processor id，用于获取CPU ID，具体参考第二节
        bl  __lookup_processor_type     @ r5=procinfo r9=cpuid，根据cpu id获取proc info，具体参考第一节和第三节。
        movs    r10, r5             @ invalid processor (r5=0)?，判断proc info是否存在
     THUMB( it  eq )        @ force fixup-able long branch encoding
        beq __error_p           @ yes, error 'p'
mrc p15, 0, r9, c0, c0 @ get processor id，用于获取CPU ID，具体参考第二节 
bl __lookup_processor_type @ r5=procinfo r9=cpuid，根据cpu id获取proc info，具体参考第一节和第三节。

# 一、procinfo

## 1、说明 

procinfo使用proc_info_list结构体，用来说明一个cpu的信息，包括这个cpu的ID号，对应的内核数据映射区的MMU标识等等。

## 2、数据结构定义 

```
arch/arm/include/asm/procinfo.h

struct proc_info_list {
    unsigned int        cpu_val;
    unsigned int        cpu_mask;
    unsigned long       __cpu_mm_mmu_flags; /* used by head.S */
    unsigned long       __cpu_io_mmu_flags; /* used by head.S */
    unsigned long       __cpu_flush;        /* used by head.S */
    const char      *arch_name;
    const char      *elf_name;
    unsigned int        elf_hwcap;
    const char      *cpu_name;
    struct processor    *proc;
    struct cpu_tlb_fns  *tlb;
    struct cpu_user_fns *user;
    struct cpu_cache_fns    *cache;
};
```

几个我们重点关注的成员如下： 

- cpu_val：cpu对应的硬件id号 
- cpu_mask：cpu硬件id号的掩码 
- __cpu_mm_mmu_flags：临时页表映射的内核空间的MMU标识 
- __cpu_io_mmu_flags：IO映射区的MMU标识 
- __cpu_flush：cpu setup函数的地址，后续在打开MMU过程时候会使用到

注意： 
- 这里存在的MMU标识，也就是我们需要在打开MMU之前需要先获取procinfo的原因，因为打开MMU之前需要配置临时内核页表，而配置临时内核页表需要这里的MMU标识来进行设置。在后续创建临时内核页表的文章中会进行说明。 
  这里回答了“proc info存放了什么东西以至于有必要在第一阶段、打开MMU之前就去获取？”的疑问。 
- cpu id和procinfo是一一对应的关系，所以可以通过cpu id来获取到对应的procinfo结构体，后续的小节中会说明。 
  这里回答了“为什么要获取cpu id”的疑问。

## 3、存放位置 

所有CPU的proc info都会被存放到.init.proc.info段中 

```
arch/arm/kernel/vmlinux.lds.S

SECTIONS
{
        .init.proc.info : {
                ARM_CPU_DISCARD(PROC_INFO)
        }
}
```

```
#define PROC_INFO                                                       \
        . = ALIGN(4);                                                   \
        VMLINUX_SYMBOL(__proc_info_begin) = .;                          \
        *(.proc.info.init)                                              \   
        VMLINUX_SYMBOL(__proc_info_end) = .;
```

通过查看Systemp.map可以看到.init.proc.info段里面放了这些cpu的procinfo：

```
8041a800 T __proc_info_begin
8041a800 t __v7_ca5mp_proc_info
8041a834 t __v7_ca9mp_proc_info
8041a868 t __v7_ca8_proc_info
8041a89c t __v7_cr7mp_proc_info
8041a8d0 t __v7_ca7mp_proc_info
8041a904 t __v7_ca12mp_proc_info
8041a938 t __v7_ca15mp_proc_info
8041a96c t __v7_b15mp_proc_info
8041a9a0 t __v7_ca17mp_proc_info
8041a9d4 t __krait_proc_info
8041aa08 t __v7_proc_info
8041aa3c T __proc_info_end
```

## 4、示例 

以s5pv210为例，其arm体系是armv7，cortex-A8架构，对应procinfo定义于proc-v7.S中 


    arch/arm/mm/proc-v7.S
    
    .section ".proc.info.init", #alloc
    /*
     * ARM Ltd. Cortex A8 processor.
     */
    .type    __v7_ca8_proc_info, #object
    _v7ca8_proc_info:
    .long    0x410fc080
    .long    0xff0ffff0
    __v7_proc __v7_ca8_proc_info, __v7_setup, proc_fns = ca8_processor_functions
    .size    __v7_ca8_proc_info, . - __v7_ca8_proc_info
通过.section “.proc.info.init”将后面的数据结构定义在了.proc.info.init段中，最终在连接过程中被连接到.init.proc.info段中，也就是__proc_info_begin和__proc_info_end之间的位置中。 

__v7_proc是一个宏，其定义如下

```
.macro __v7_proc name, initfunc, mm_mmuflags = 0, io_mmuflags = 0, hwcaps = 0, proc_fns = v7_processor_functions
    ALT_SMP(.long    PMD_TYPE_SECT | PMD_SECT_AP_WRITE | PMD_SECT_AP_READ | \
            PMD_SECT_AF | PMD_FLAGS_SMP | \mm_mmuflags)
    ALT_UP(.long    PMD_TYPE_SECT | PMD_SECT_AP_WRITE | PMD_SECT_AP_READ | \
            PMD_SECT_AF | PMD_FLAGS_UP | \mm_mmuflags)
    .long    PMD_TYPE_SECT | PMD_SECT_AP_WRITE | \
        PMD_SECT_AP_READ | PMD_SECT_AF | \io_mmuflags
    initfn    \initfunc, \name
    .long    cpu_arch_name
    .long    cpu_elf_name
    .long    HWCAP_SWP | HWCAP_HALF | HWCAP_THUMB | HWCAP_FAST_MULT | \
        HWCAP_EDSP | HWCAP_TLS | \hwcaps
    .long    cpu_v7_name
    .long    \proc_fns
    .long    v7wbi_tlb_fns
    .long    v6_user_fns
    .long    v7_cache_fns
.endm
```

根据数据结构定义，__v7_ca8_proc_info对应如下结果（只列出我们重点关注的部分） 

- cpu_val：0x410fc080 
- cpu_mask：0xff0ffff0 
- __cpu_mm_mmu_flags：PMD_TYPE_SECT | PMD_SECT_AP_WRITE | PMD_SECT_AP_READ | \ 
  PMD_SECT_AF | PMD_FLAGS_UP | \mm_mmuflags 
- __cpu_io_mmu_flags： PMD_TYPE_SECT | PMD_SECT_AP_WRITE | \ 
  PMD_SECT_AP_READ | PMD_SECT_AF | \io_mmuflags 
- __cpu_flush ： __v7_setup 
  上述几个成员我们在后续启动过程都会使用到。

# 二、如何获取CPU ID

## 1、原理 

- arm体系上可支持最多16个协处理器。 
- arm体系将CPU ID（处理器标识符，主标识符）存放在协处理器cp15的c0寄存器中。 

## 2、协处理器cp15 

这部分建议参考《ARM 的 CP15 协处理器的寄存器》 

- 介绍 
  主要用于内存系统控制和测试控制。也被称之为系统控制协处理器。 
- 寄存器说明 
  cp15有16个寄存器。其中和处理器标识符（CPU ID）相关的是c0寄存器。 
  cp15 中寄存器 c0 对应两个标识符寄存器，分别是处理器标识符寄存器（主标识符寄存器）和cache类型标识符寄存器。当操作码2（opcode2）是0的时候，访问的是处理器标识符寄存器，当操作码2（opcode2）是1的时候的时候访问的是cache类型标识符寄存器。

## 3、协处理器指令说明 

（1）MCR 指令：ARM寄存器到协处理器寄存器的数据传送

```
   MCR{<cond>} <p>，<opcode_1>，<Rd>,<CRn>,<CRm>{,<opcode_2>}
   MCR{<cond>} p15，0，<Rd>,<CRn>,<CRm>{,<opcode_2>}
```

（2）MRC指令： 协处理器寄存器到ARM寄存器的数据传送

```
MRC{<cond>} <p>，<opcode_1>，<Rd>,<CRn>,<CRm>{,<opcode_2>}
MRC{<cond>} p15，0，<Rd>,<CRn>,<CRm>{,<opcode_2>}
```

- cond为指令执行的条件码。当忽略时指令为无条件执行。
- opcode_1为协处理器将执行的操作的操作码。对于CP15协处理器来说，< opcode_1>永远为0b000，当< opcode_1>不为0b000时，该指令操作结果不可预知。
- Rd作为源寄存器的ARM寄存器，其值将被传送到协处理器寄存器中。
- CRn作为目标寄存器的协处理器寄存器，其编号可能是C0，C1，…，C15。
- CRm和opcode_2两者组合决定对协处理器寄存器进行所需要的操作，如果没有指定，则将为为C0，opcode_2为0，否则可能导致不可预知的结果。

## 4、获取cpu id的指令 

通过上述，我们通过mrc指令从p15中的c0寄存器中获取CPU ID,并且获取获取CPU ID的参数如下： 

- p->p15, 
- opcode_1->0 
- Rd->r9（我们要存放在r9寄存器其中） 
- CRn->c0 
- CRm->c0 

最终获取cpu id的指令如下：

```
mrc    p15, 0, r9, c0, c0        @ get processor id
```

# 三、如何获取cpu对应的procinfo

## 1、原理 

在上述第二节中，可知所有cpu的procinfo结构体都被连接到了.init.proc.info段中，也就是__proc_info_begin和__proc_info_end之间的位置中。 

启动过程中，获取cpu id，在依次将cpu id和这个区间内的proc_info_list的cpu_val成员进行比较，如果匹配则对应proc_info_list结构体就是所需的proc info。

## 2、代码实现 

- 首先将__proc_info_begin的区间信息存放在__lookup_processor_type_data位置上 
```
arch/arm/kernel/head-common.S

/*
* Look in <asm/procinfo.h> for information about the __proc_info structure.
*/
    .align    2
    .type    __lookup_processor_type_data, %object
__lookup_processor_type_data:
    .long    .
    .long    __proc_info_begin
    .long    __proc_info_end
    .size    __lookup_processor_type_data, . - __lookup_processor_type_data
```

- 在stext中调用__lookup_processor_type来获取cpu对应的proc info 
  其中r9存放的是cpu id，r5存放的是获取到的proc info的地址

```
    __HEAD
ENTRY(stext)
    bl  __lookup_processor_type     @ r5=procinfo r9=cpuid
```

- __lookup_processor_type实现如下 

```
arch/arm/kernel/head-common.S

/*
* Read processor ID register (CP#15, CR0), and look up in the linker-built
 * supported processor list.  Note that we can't use the absolute addresses
* for the __proc_info lists since we aren't running with the MMU on
 * (and therefore, we are not in the correct address space).  We have to
* calculate the offset.
*
 *    r9 = cpuid
* Returns:
 *    r3, r4, r6 corrupted
 *    r5 = proc_info pointer in physical address space
 *    r9 = cpuid (preserved)
*/
__lookup_processor_type:
    adr    r3, __lookup_processor_type_data
    ldmia    r3, {r4 - r6}
    sub    r3, r3, r4            @ get offset between virt&phys
    add    r5, r5, r3            @ convert virt addresses to
    add    r6, r6, r3            @ physical address space
1:    ldmia    r5, {r3, r4}            @ value, mask
    and    r4, r4, r9            @ mask wanted bits
    teq    r3, r4
    beq    2f
    add    r5, r5, #PROC_INFO_SZ        @ sizeof(proc_info_list)
    cmp    r5, r6
    blo    1b
    mov    r5, #0                @ unknown processor
2:    ret    lr
ENDPROC(__lookup_processor_type)
```

解析如下： 

（1）获取proc_info区间的连接地址

```
    adr    r3, __lookup_processor_type_data
    ldmia    r3, {r4 - r6}
```

通过上述 

- r3存放的是__lookup_processor_type_data的真实的物理地址，也就是在RAM上的位置 
- r4存放的是__lookup_processor_type_data的连接地址 
- r5存放的是__proc_info_begin的连接地址 
- r6存放的是__proc_info_end的连接地址 

（2）计算出proc_info区间的内存地址

    sub    r3, r3, r4            @ get offset between virt&phys
    add    r5, r5, r3            @ convert virt addresses to
    add    r6, r6, r3            @ physical address space
因为此时MMU是没有打开的，所以并不能直接使用连接地址来访问对应区域，而需要计算出对应区域的内存地址。 

1. 首先计算出__lookup_processor_type_data的真实物理地址(r4)和连接地址(r3)的偏移， 
2. 然后根据偏移计算出__proc_info_begin(r5)和__proc_info_end(r6)的真实物理地址，也就是内存地址。 

通过上述步骤之后 

- r5存放的是__proc_info_begin的物理地址，也就是内存地址 
- r6存放的是__proc_info_end的物理地址，也就是内存地址 

可以直接r5和r6访问到proc info的区间。 

（3）提取结构体信息并进行比较

    1:    ldmia    r5, {r3, r4}            @ value, mask
    and    r4, r4, r9            @ mask wanted bits
    teq    r3, r4
    beq    2f
“ldmia r5, {r3, r4}”获取[__proc_info_begin-__proc_info_end]中第一个proc_info_list结构体的，因为cpu_val和cpu_mask被存放在proc_info_list结构体的前16个字节，所以可以直接这样获取。 

r3上就存放了cpu_val，r4上存放了cpu_mask。 将cpu_mask(r4)和获得的cpuid(r9)进行掩码后和cpu_val(r3)进行比较，相等则返回退出,此时的r5上存放了对应的proc_info地址。否则进入下一个循环。 

（4）获取下一个proc_info_list结构体，

    add    r5, r5, #PROC_INFO_SZ        @ sizeof(proc_info_list)
    cmp    r5, r6
    blo    1b
    mov    r5, #0                @ unknown processor
如果还没有到__proc_info_end(r6)，则继续下一个循环。如果已经搜索到结尾，将r5设置为0（也就表示非法值）后直接退出。

**通过上述步骤之后，就可以获取到了cpu id对应的proc_info_list结构体，并且其地址存放在r5寄存器中。**