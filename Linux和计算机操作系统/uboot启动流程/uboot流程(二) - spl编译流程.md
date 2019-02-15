uboot流程(二) - spl编译流程

# 一. uboot-spl编译和生成文件

spl的编译是编译uboot的一部分，和uboot.bin走的是两条编译流程。 正常来说，会先编译主体uboot，也就是uboot.bin。再编译uboot-spl，也就是uboot-spl.bin，虽然编译命令是一起的，但是编译流程是分开的。

## 1. 编译方法

所有镜像，包括uboot、kernel、rootfs都是放在build目录下进行编译的。具体去参考该项目build的Makefile的实现。 假设config已经配置完成，在build编译命令如下：

```
make uboot
```

Makefile中对应的命令如下：

```
BUILD_DIR=$(shell pwd)
OUT_DIR=$(BUILD_DIR)/out
UBOOT_OUT_DIR=$(OUT_DIR)/u-boot
UBOOT_DIR=$(BUILD_DIR)/../u-boot
uboot:
        mkdir -p $(UBOOT_OUT_DIR)
        make -C $(UBOOT_DIR) CROSS_COMPILE=$(CROSS_COMPILE) KBUILD_OUTPUT=$(UBOOT_OUT_DIR) $(BOARD_NAME)_defconfig
        make -C $(UBOOT_DIR) CROSS_COMPILE=$(CROSS_COMPILE) KBUILD_OUTPUT=$(UBOOT_OUT_DIR)
## -C $(UBOOT_DIR) 指定了要在../uboot，也就是uboot的代码根目录下执行make
## CROSS_COMPILE=$(CROSS_COMPILE) 指定了交叉编译器
## KBUILD_OUTPUT=$(UBOOT_OUT_DIR) 指定了最终编译的输出目录是build/out/u-boot.
```

最终，相当于进入了uboot目录执行了make动作。 

也就是说spl的编译是编译uboot的一部分，和uboot.bin走的是两条编译流程，这个要重点注意。 正常来说，会先编译主体uboot，也就是uboot.bin.再编译uboot-spl，也就是uboot-spl.bin，虽然编译命令是一起的，但是编译流程是分开的。

## 2.  生成文件

最终编译完成之后，会在project-x/build/out/u-boot/spl下生成如下文件：

```
arch   common   dts  include          u-boot-spl      u-boot-spl.cfg  u-boot-spl.map
board  drivers  fs   tiny210-spl.bin  u-boot-spl.bin  u-boot-spl.lds  u-boot-spl-nodtb.bin
```

其中 arch、common、dts、include、board、drivers、fs 是对应代码的编译目录，各个目录下都会生成相应的built.o，是由同目录下的目标文件连接而成。

|          文件          |                    说明                    |
| :------------------: | :--------------------------------------: |
|      u-boot-spl      |              初步链接后得到的spl文件               |
| u-boot-spl-nodtb.bin | 在u-boot-spl的基础上，经过objcopy去除符号表信息之后的可执行程序 |
|    u-boot-spl.bin    | 在不需要dtb的情况下，直接由u-boot-spl-nodtb.bin复制而来，也就是编译spl的最终目标 |
|   tiny210-spl.bin    | 由s5pv210平台决定，需要在u-boot-spl.bin的基础上加上16B的header用作校验 |
|    u-boot-spl.lds    |                 spl的连接脚本                 |
|    u-boot-spl.map    |                连接之后的符号表文件                |
|    u-boot-spl.cfg    |               由spl配置生成的文件                |

# 二. uboot-spl编译流程

## 1. 编译整体流程

简单流程如下： 

（1）各目录下built-in.o的生成

源文件、代码文件     -->	编译、汇编	-->	目标文件    -->	同目录目标文件连接	-->	built-in目标文件

对应二、2（5）的实现 

（2）由所有built-in.o以u-boot-spl.lds为连接脚本通过连接来生成u-boot-spl

built-in目标文件	-->	以u-boot-spl.lds为连接脚本进行统一连接	-->	u-boot-spl

对应二、2（4）的实现 

（3）由u-boot-spl生成u-boot-spl-nodtb.bin

u-boot-spl	-->	objcopy动作去掉符号信息表	-->	u-boot-spl-nodtb.bin对应二、2（3）的实现 

（4）由u-boot-spl-nodtb.bin生成u-boot-spl.bin，也就是spl的bin文件

u-boot-spl-nodtb.bin	-->	在不需要dtb的情况下复制	-->	u-boot-spl.bin

对应二、2（2）的实现

后续的编译的核心过程就是按照上述的四个编译流程就是按照上述四个步骤来的。

## 2. 具体编译流程分析

我们直接从make uboot命令分析，也就是从uboot下的Makefile的依赖关系来分析整个编译流程。 注意，这个分析顺序和上述的整体编译流程的顺序是反着的。

（1）入口分析 

在 project-x/u-boot/Makefile 中

```
all:            $(ALL-y)

ALL-$(CONFIG_SPL) += spl/u-boot-spl.bin
## 当配置了CONFIG_SPL，make的时候就会执行spl/u-boot-spl.bin这个目标

spl/u-boot-spl.bin: spl/u-boot-spl
        @:
spl/u-boot-spl: tools prepare $(if $(CONFIG_OF_SEPARATE),dts/dt.dtb)
        $(Q)$(MAKE) obj=spl -f $(srctree)/scripts/Makefile.spl all
## obj=spl 会在out/u-boot目录下生成spl目录
## -f $(srctree)/scripts/Makefile.spl 说明执行的Makefile文件是scripts/Makefile.spl
## $(MAKE) all 相当于make的目标是all
```

综上，由CONFIG_SPL来决定是否需要编译出spl文件，也就是BL1。

 后续相当于执行了`“make -f u-boot/scripts/Makefile.spl obj=spl all”`命令。 

在 project-x/u-boot/scripts/Makefile.spl 中

```
SPL_BIN := u-boot-spl
ALL-y   += $(obj)/$(SPL_BIN).bin $(obj)/$(SPL_BIN).cfg
## 所以最终目标是spl/u-boot-spl.bin和spl/u-boot-spl.cfg
```

在project-x/u-boot/scripts/Makefile.spl中建立了spl/u-boot-spl.bin的依赖关系，后续make过程的主体都是在Makefile.spl中。

（2）spl/u-boot-spl.bin的依赖关系

在project-x/u-boot/scripts/Makefile.spl中

```
$(obj)/$(SPL_BIN).bin: $(obj)/$(SPL_BIN)-nodtb.bin FORCE
        $(call if_changed,copy)
## $(obj)/$(SPL_BIN).bin依赖于$(obj)/$(SPL_BIN)-nodtb.bin。
## $(call if_changed,copy)表示当依赖文件发生变化时，直接把依赖文件复制为目标文件，即直接把$(obj)/$(SPL_BIN)-nodtb.bin复制为$(obj)/$(SPL_BIN).bin
```

如上述Makefile代码spl/u-boot-spl.bin依赖于spl/u-boot-spl-nodtb.bin，并且由spl/u-boot-spl-nodtb.bin复制而成。 对应于上述二、1（4）流程。

（3）spl/u-boot-spl-nodtb.bin的依赖关系 

在project-x/u-boot/scripts/Makefile.spl中

```
$(obj)/$(SPL_BIN)-nodtb.bin: $(obj)/$(SPL_BIN) FORCE
        $(call if_changed,objcopy)
$(obj)/$(SPL_BIN)-nodtb.bin依赖于$(obj)/$(SPL_BIN)。也就是spl/u-boot-spl-nodtb.bin依赖于spl/u-boot-spl.
## $(call if_changed,objcopy)表示当依赖文件发生变化时，将依赖文件经过objcopy处理之后得到目标文件。
## 也就是通过objcopy把spl/u-boot-spl的符号信息以及一些无用信息去掉之后，得到了spl/u-boot-spl-nodtb.bin。
```

如上述Makefile代码spl/u-boot-spl-nodtb.bin依赖于spl/u-boot-spl，并且由spl/u-boot-spl经过objcopy操作之后得到。 对应于上述二、1（3）流程。

（4）spl/u-boot-spl的依赖关系 

在project-x/u-boot/scripts/Makefile.spl中

```
$(obj)/$(SPL_BIN): $(u-boot-spl-init) $(u-boot-spl-main) $(obj)/u-boot-spl.lds FORCE
        $(call if_changed,u-boot-spl)
## $(call if_changed,u-boot-spl)来生成目标
## $(call if_changed,u-boot-spl)对应cmd_u-boot-spl命令
```

如上，spl/u-boot-spl依赖于(u-boot-spl-main)和spl/u-boot-spl.ld，并且最终会调用cmd_u-boot-spl来生成spl/u-boot-spl。 cmd_u-boot-spl实现如下：

```
quiet_cmd_u-boot-spl = LD      $@
      cmd_u-boot-spl = (cd $(obj) && $(LD) $(LDFLAGS) $(LDFLAGS_$(@F)) \
                       $(patsubst $(obj)/%,%,$(u-boot-spl-init)) --start-group \
                       $(patsubst $(obj)/%,%,$(u-boot-spl-main)) --end-group \
                       $(PLATFORM_LIBS) -Map $(SPL_BIN).map -o $(SPL_BIN))
```

将cmd_u-boot-spl通过echo命令打印出来之后得到如下（拆分出来看的）：

```
cmd_u-boot-spl=(
cd spl && 
/build/arm-none-linux-gnueabi-4.8/bin/arm-none-linux-gnueabi-ld   
-T u-boot-spl.lds  --gc-sections -Bstatic --gc-sections 
arch/arm/cpu/armv7/start.o 
--start-group 
arch/arm/mach-s5pc1xx/built-in.o arch/arm/cpu/armv7/built-in.o arch/arm/cpu/built-in.o arch/arm/lib/built-in.o board/samsung/tiny210/built-in.o board/samsung/common/built-in.o common/init/built-in.o drivers/built-in.o dts/built-in.o fs/built-in.o 
--end-group 
arch/arm/lib/eabi_compat.o 
-L /home/disk3/xys/temp/project-x/build/arm-none-linux-gnueabi-4.8/bin/../lib/gcc/arm-none-linux-gnueabi/4.8.3 -lgcc 
-Map u-boot-spl.map 
-o u-boot-spl)
```

可以看出上述是一条连接命令，以spl/u-boot-spl.lds为链接脚本，把`$(u-boot-spl-init) 、$(u-boot-spl-main)`的指定的目标文件连接到`u-boot-spl`中。 连接很重要的东西就是连接标识，也就是`$(LD) $(LDFLAGS) $(LDFLAGS_$(@F)`的定义。 

尝试把`$(LD) \$(LDFLAGS) \$(LDFLAGS_\$(@F))` 打印出来，结果如下：

```
LD=/home/disk3/xys/temp/project-x/build/arm-none-linux-gnueabi-4.8/bin/arm-none-linux-gnueabi-ld
LDFLAGS=
LDFLAGS_u-boot-spl=-T u-boot-spl.lds --gc-sections -Bstatic --gc-sections
## $(LDFLAGS_$(@F)对应于LDFLAGS_u-boot-spl
```

也就是说在`LDFLAGS_u-boot-spl`中指定了链接脚本。 重点关注`$(LDFLAGS_$(@F))`的由来

```
## @F是一个自动化变量，提取目标的文件名，比如目标是$(obj)/$(SPL_BIN)，也就是spl/u-boot-spl,那么@F就是u-boot-spl。
## 所以LDFLAGS_$(@F)就是LDFLAGS_u-boot-spl
## 定义如下
LDFLAGS_$(SPL_BIN) += -T u-boot-spl.lds $(LDFLAGS_FINAL)
ifneq ($(CONFIG_SPL_TEXT_BASE),)
LDFLAGS_$(SPL_BIN) += -Ttext $(CONFIG_SPL_TEXT_BASE)
endif
## 当指定CONFIG_SPL_TEXT_BASE时，会配置连接地址。在tiny210项目中，因为spl是地址无关代码设计，故没有设置连接地址。

## $(LDFLAGS_FINAL)在如下几个地方定义了
## ./config.mk:19:LDFLAGS_FINAL :=
## ./config.mk:80:LDFLAGS_FINAL += -Bstatic
## ./arch/arm/config.mk:16:LDFLAGS_FINAL += --gc-sections
## ./scripts/Makefile.spl:43:LDFLAGS_FINAL += --gc-sections
## 综上：最后LDFLAGS_u-boot-spl=-T u-boot-spl.lds --gc-sections -Bstatic --gc-sections就可以理解了。
## 对应于上述二、1（2）流程。
```

（5）u-boot-spl-init & u-boot-spl-main依赖关系（代码是如何被编译的） 先看一下这两个值打印出来的

```
u-boot-spl-init=spl/arch/arm/cpu/armv7/start.o

u-boot-spl-main= spl/arch/arm/mach-s5pc1xx/built-in.o spl/arch/arm/cpu/armv7/built-in.o spl/arch/arm/cpu/built-in.o spl/arch/arm/lib/built-in.o spl/board/samsung/tiny210/built-in.o spl/board/samsung/common/built-in.o spl/common/init/built-in.o spl/drivers/built-in.o spl/dts/built-in.o spl/fs/built-in.o
```

可以观察到是一堆目标文件的路径。这些目标文件最终都要被连接到u-boot-spl中。 

`u-boot-spl-init & u-boot-spl-main`的定义如下代码： `project-x/u-boot/scripts/Makefile.spl`

```
u-boot-spl-init := $(head-y)
head-y          := $(addprefix $(obj)/,$(head-y))
## 加spl路径
## ./arch/arm/Makefile定义了如下：
## head-y := arch/arm/cpu/$(CPU)/start.o

u-boot-spl-main := $(libs-y)
libs-y += $(if $(BOARDDIR),board/$(BOARDDIR)/)
libs-$(HAVE_VENDOR_COMMON_LIB) += board/$(VENDOR)/common/
libs-$(CONFIG_SPL_FRAMEWORK) += common/spl/
libs-y += common/init/
libs-$(CONFIG_SPL_LIBCOMMON_SUPPORT) += common/ cmd/
libs-$(CONFIG_SPL_LIBDISK_SUPPORT) += disk/
libs-y += drivers/
libs-y += dts/
libs-y += fs/ 
libs-$(CONFIG_SPL_LIBGENERIC_SUPPORT) += lib/
libs-$(CONFIG_SPL_POST_MEM_SUPPORT) += post/drivers/
libs-$(CONFIG_SPL_NET_SUPPORT) += net/
libs-y          := $(addprefix $(obj)/,$(libs-y))
## 加spl路径
u-boot-spl-dirs := $(patsubst %/,%,$(filter %/, $(libs-y)))
## 这里注意一下，u-boot-spl-dir是libs-y没有加built-in.o后缀的时候被定义的。
libs-y := $(patsubst %/, %/built-in.o, $(libs-y))
## 加built-in.o文件后缀
```

那么u-boot-spl-init & u-boot-spl-main是如何生成的呢？ 需要看一下对应的依赖如下：

```
$(sort $(u-boot-spl-init) $(u-boot-spl-main)): $(u-boot-spl-dirs) ;
## 也就是说$(u-boot-spl-init) $(u-boot-spl-main)依赖于$(u-boot-spl-dirs) 
## sort函数根据首字母进行排序并去除掉重复的。
## u-boot-spl-dirs := $(patsubst %/,%,$(filter %/, $(libs-y)))
## $(filter %/, $(libs-y)过滤出'/'结尾的字符串，注意，此时$(libs-y)的内容还没有加上built-in.o文件后缀
## patsubst去掉字符串中最后的'/'的字符。
## 最后u-boot-spl-dirs打印出来如下：
## u-boot-spl-dirs=spl/arch/arm/mach-s5pc1xx spl/arch/arm/cpu/armv7 spl/arch/arm/cpu spl/arch/arm/lib spl/board/samsung/tiny210 spl/board/samsung/common spl/common/init spl/drivers spl/dts spl/fs
## 也就是从libs-y改造而来的。

## $(u-boot-spl-dirs) 的依赖规则如下：
$(u-boot-spl-dirs):
        $(Q)$(MAKE) $(build)=$@
```

也就是会对每一个目标文件依次执行`make $(build)=目标文件 $(build)`定义如下： `project-x/u-boot/scripts/Kbuild.include`

```
build := -f $(srctree)/scripts/Makefile.build obj
```

以`arch/arm/mach-s5pc1xx`为例

`“$(MAKE) $(build)=$@”`展开后格式如下

 `make -f ~/code/temp/project-x/u-boot/scripts/Makefile.build obj=spl/arch/arm/mach-s5pc1xx`。

`Makefile.build`定义`built-in.o`,`.lib`以及目标文件.o的生成规则。这个Makefile文件生成了子目录的`.lib,built-in.o`以及目标文件`.o。 Makefile.build`第一个编译目标是__build，如下

```
PHONY := __build
__build:
## 所以会直接编译执行__build这个目标，其依赖如下
__build: $(if $(KBUILD_BUILTIN),$(builtin-target) $(lib-target) $(extra-y)) \
         $(if $(KBUILD_MODULES),$(obj-m) $(modorder-target)) \
         $(subdir-ym) $(always)
        @:
## 和built-in.o相关的是依赖builtin-target。下面来看这个依赖。

builtin-target := $(obj)/built-in.o
## 以obj=spl/arch/arm/mach-s5pc1xx为例，那么builtin-target就是spl/arch/arm/mach-s5pc1xx/built-in.o.

## 依赖关系如下：
$(builtin-target): $(obj-y) FORCE
        $(call if_changed,link_o_target)
## $(call if_changed,link_o_target)将所有依赖连接到$(builtin-target)，也就是相应的built-in.o中了。
## 具体实现可以查看cmd_link_o_target的实现，这里不详细说明了。


## 那么$(obj-y)是从哪里来的呢？是从相应目录下的Makefile中include得到的。
# The filename Kbuild has precedence over Makefile
kbuild-dir := $(if $(filter /%,$(src)),$(src),$(srctree)/$(src))
kbuild-file := $(if $(wildcard $(kbuild-dir)/Kbuild),$(kbuild-dir)/Kbuild,$(kbuild-dir)/Makefile)
include $(kbuild-file)
## 当obj=spl/arch/arm/mach-s5pc1xx时,得到对应的kbuild-file=u-boot/arch/arm/mach-s5pc1xx/Makefile
## 而在u-boot/arch/arm/mach-s5pc1xx/Makefile中定义了obj-y如下：
## obj-y   = cache.o
## obj-y   += reset.o
## obj-y   += clock.o
## 对应obj-y对应一些目标文件，由C文件编译而来，这里就不说明了。
```

对应于上述二、1（1）流程。

（6）spl/u-boot-spl.lds依赖关系 

这里主要是为了找到一个匹配的连接文件。

```
$(obj)/u-boot-spl.lds
$(obj)/u-boot-spl.lds: $(LDSCRIPT) FORCE
        $(call if_changed_dep,cpp_lds)
## 依赖于$(LDSCRIPT)，$(LDSCRIPT)定义了连接脚本所在的位置，
## 然后把链接脚本经过cpp_lds处理之后复制到$(obj)/u-boot-spl.lds中，也就是spl/u-boot-spl.lds中。
## cpp_lds处理具体实现看cmd_cpu_lds定义，具体是对应连接脚本里面的宏定义进行展开。

## $(LDSCRIPT)定义如下
ifeq ($(wildcard $(LDSCRIPT)),)
        LDSCRIPT := $(srctree)/board/$(BOARDDIR)/u-boot-spl.lds
endif
ifeq ($(wildcard $(LDSCRIPT)),)
        LDSCRIPT := $(srctree)/$(CPUDIR)/u-boot-spl.lds
endif
ifeq ($(wildcard $(LDSCRIPT)),)
        LDSCRIPT := $(srctree)/arch/$(ARCH)/cpu/u-boot-spl.lds
endif
ifeq ($(wildcard $(LDSCRIPT)),)
$(error could not find linker script)
endif
## 也就是说依次从board/板级目录、cpudir目录、arch/架构/cpu/目录下去搜索u-boot-spl.lds文件。
## 例如，tiny210(s5vp210 armv7)最终会在./arch/arm/cpu/下搜索到u-boot-spl.lds
```

综上，最终指定了project-X/u-boot/arch/arm/cpu/u-boot-spl.lds作为连接脚本。

# 三. 一些重点定义

## 1. CONFIG_SPL

在二、2（1）中说明。 

用于指定是否需要编译SPL，也就是是否需要编译出uboot-spl.bin文件

## 2. 连接脚本的位置

在二、2（6）中说明。 

对于`tiny210(s5pv210 armv7)`来说，连接脚本的位置在 `project-x/u-boot/arch/arm/cpu/u-boot-spl.lds`

## 3. CONFIG_SPL_TEXT_BASE

在二、2（4）中说明。 用于指定SPL的连接地址，可以定义在板子对应的config文件中。

## 4. CONFIG_SPL_BUILD

在编译spl过程中，会配置 `project-x/scripts/Makefile.spl`中定义了如下

```
KBUILD_CPPFLAGS += -DCONFIG_SPL_BUILD
```

也就是说在编译`uboot-spl.bin`的过程中，`CONFIG_SPL_BUILD`这个宏是被定义的。

# 四. uboot-spl链接脚本说明

## 1. 连接脚本整体分析

相对比较简单，直接看连接脚本的内容project-x/u-boot/arch/arm/cpu/u-boot-spl.lds 

```
ENTRY(_start)
//定义了地址为_start的地址，所以我们分析代码就是从这个函数开始分析的！！！
SECTIONS
{
    . = 0x00000000;

//以下定义文本段
    . = ALIGN(4);
    .text :
    {
        __image_copy_start = .;
//定义__image_copy_start这个标号地址为当前地址
        *(.vectors)
//所有目标文件的vectors段，也就是中断向量表连接到这里来
        CPUDIR/start.o (.text*)
//start.o文件的.text段链接到这里来
        *(.text*)
//所有目标文件的.text段链接到这里来
    }

//以下定义只读数据段
    . = ALIGN(4);
    .rodata : { *(SORT_BY_ALIGNMENT(SORT_BY_NAME(.rodata*))) }

//以下定义数据段
    . = ALIGN(4);
    .data : {
        *(.data*)
//所有目标文件的.data段链接到这里来
    }

//以下定义u_boot_list段，具体功能未知
    . = ALIGN(4);
    .u_boot_list : {
        KEEP(*(SORT(.u_boot_list*)));
    }

    . = ALIGN(4);

    __image_copy_end = .;
//定义__image_copy_end符号的地址为当前地址
//从__image_copy_start 到__image_copy_end的区间，包含了代码段和数据段。

//以下定义rel.dyn段，这个段用于uboot资源重定向的时候使用，后面会说明
    .rel.dyn : {
        __rel_dyn_start = .;
//定义__rel_dyn_start 符号的地址为当前地址，后续在代码中会使用到
        *(.rel*)
        __rel_dyn_end = .;
//定义__rel_dyn_end 符号的地址为当前地址，后续在代码中会使用到
//从__rel_dyn_start 到__rel_dyn_end 的区间，应该是在代码重定向的过程中会使用到，后续遇到再说明。
    }

    .end :
    {
        *(.__end)
    }

    _image_binary_end = .;
//定义_image_binary_end 符号的地址为当前地址

//以下定义bss段
    .bss __rel_dyn_start (OVERLAY) : {
        __bss_start = .;
        *(.bss*)
         . = ALIGN(4);
        __bss_end = .;
    }
    __bss_size = __bss_end - __bss_start;
    .dynsym _image_binary_end : { *(.dynsym) }
    .dynbss : { *(.dynbss) }
    .dynstr : { *(.dynstr*) }
    .dynamic : { *(.dynamic*) }
    .hash : { *(.hash*) }
    .plt : { *(.plt*) }
    .interp : { *(.interp*) }
    .gnu : { *(.gnu*) }
    .ARM.exidx : { *(.ARM.exidx*) }
}
```

## 2. 符号表中需要注意的符号

project-x/build/out/u-boot/spl/u-boot-spl.map

```
Linker script and memory map 
.text           0x00000000      0xd10
                0x00000000                __image_copy_start = . 
 *(.vectors)
                0x00000000                _start
                0x00000020                _undefined_instruction
                0x00000024                _software_interrupt
                0x00000028                _prefetch_abort
                0x0000002c                _data_abort
                0x00000030                _not_used
                0x00000034                _irq
                0x00000038                _fiq
...
                0x00000d10                __image_copy_end = .
.rel.dyn        0x00000d10        0x0
                0x00000d10                __rel_dyn_start = .
 *(.rel*)
 .rel.iplt      0x00000000        0x0 arch/arm/cpu/armv7/start.o
                0x00000d10                __rel_dyn_end = .
.end
 *(.__end)
                0x00000d10                _image_binary_end = .

.bss            0x00000d10        0x0
                0x00000d10                __bss_start = .
 *(.bss*)
                0x00000d10                . = ALIGN (0x4)
                0x00000d10                __bss_end = .
```

重点关注 

* __image_copy_start & __image_copy_end 
  界定了代码的位置，用于重定向代码的时候使用，后面碰到了再分析。 
* _start 
  在u-boot-spl.lds中ENTRY(_start)，也就规定了代码的入口函数是_start。所以后续分析代码的时候就是从这里开始分析。 
* __rel_dyn_start & __rel_dyn_end 
* _image_binary_end
# 五. tiny210（s5pv210）的额外操作

## 1. 为什么tiny210的spl需要额外操作？需要什么额外操作？

当使用SD启动的方式和NAND FLASH启动的方式，也就是BL1镜像存放在SD上或者nand flash上时，s5pv210中固化的BL0，都需要对BL1的前16B的header做校验。BL1就是我们所说的uboot-spl.bin，但是默认编译出来的uboot-spl.bin就是一个纯粹的可执行文件，并没有加上特别的header。 因此，我们需要在生成uboot-spl.bin之后，再为其加上16B的header后生成tiny210-spl.bin。 

16B的header格式如下：

|     地址      |        数据         |
| :---------: | :---------------: |
| 0xD002_0000 | BL1镜像包括header的长度  |
| 0xD002_0004 |      保留，设置为0      |
| 0xD002_0008 | BL1镜像除去header的校验和 |
| 0xD002_000c |      保留，设置为0      |

## 2. 如何生成header？（如何生成tiny210-spl.bin）

project-x/u-boot/scripts/Makefile.spl

```
ifdef CONFIG_SAMSUNG
ALL-y   += $(obj)/$(BOARD)-spl.bin
endif
## 当平台是SAMSUNG平台的时候，也就是CONFIG_SAMSUNG被定义的时候，就需要生成对应的板级spl.bin文件，例如tiny210的话，就应该生成对应的spl/tiny210-spl.bin文件。

ifdef CONFIG_S5PC110
$(obj)/$(BOARD)-spl.bin: $(obj)/u-boot-spl.bin
        $(objtree)/tools/mks5pc1xxspl $< $@
## 如果是S5PC110系列的cpu的话，则使用如上方法打上header。tiny210的cpu是s5pv210的，属于S5PC110系列，所以走的是这路。
## $(objtree)/tools/mks5pc1xxspl对应于编译uboot时生成的build/out/u-boot/tools/mks5pc1xxspl
## 其代码路径位于u-boot/tools/mks5pc1xxspl.c，会根据s5pc1xx系列的header规则为输入bin文件加上16B的header，具体参考代码。
## 这里就构成了u-boot-spl.bin到tiny210-spl.bin的过程了。
else
$(obj)/$(BOARD)-spl.bin: $(obj)/u-boot-spl.bin
        $(if $(wildcard $(objtree)/spl/board/samsung/$(BOARD)/tools/mk$(BOARD)spl),\
        $(objtree)/spl/board/samsung/$(BOARD)/tools/mk$(BOARD)spl,\
        $(objtree)/tools/mkexynosspl) $(VAR_SIZE_PARAM) $< $@
endif
endif
```

这里就构成了u-boot-spl.bin到tiny210-spl.bin的过程了。