# uboot流程(四)-uboot编译流程

# 一、uboot编译和生成文件

## 0、说明

现在的uboot已经做得和kernel很像，最主要的一点是，uboot也使用了dtb的方法，将设备树和代码分离开来（当然可以通过宏来控制）。 
`project-x/u-boot/configs/tiny210_defconfig`

```
CONFIG_OF_CONTROL=y
// 用于表示是否使用了dtb的方式

CONFIG_OF_SEPARATE=y
// 是否将dtb和uboot分离表一
```

所以在uboot的编译中，和spl的最大区别是还要编译dtb。

## 1、编译方法

在project X项目中，所有镜像，包括uboot、kernel、rootfs都是放在build目录下进行编译的。具体去参考该项目build的Makefile的实现。 假设config已经配置完成，在build编译命令如下：

```
make uboot
```

`Makefile`中对应的命令如下： 
`project-x/build/Makefile`

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

## 2、生成文件

最终编译完成之后，会在project-x/build/out/u-boot下生成如下文件：

```
arch   common   dts       include   net         tools       u-boot.cfg      u-boot.lds        u-boot.srec
board  disk     examples  lib       scripts  System.map  u-boot      u-boot.dtb      u-boot.map        u-boot.sym
cmd    drivers  fs        Makefile  source   test        u-boot.bin  u-boot-dtb.bin  u-boot-nodtb.bin
```

其中，arch、common、dts、include、board、drivers、fs等等目录是对应代码的编译目录，各个目录下都会生成相应的built.o，是由同目录下的目标文件连接而成。
重点说一下以下几个文件：

|        文件        |                    说明                    |
| :--------------: | :--------------------------------------: |
|      u-boot      |             初步链接后得到的uboot文件              |
| u-boot-nodtb.bin |   在u-boot的基础上，经过objcopy去除符号表信息之后的可执行程序   |
|    u-boot.dtb    |                  dtb文件                   |
|  u-boot-dtb.bin  |   将u-boot-nodtb.bin和u-boot.dtb打包在一起的文件   |
|    u-boot.bin    | 在需要dtb的情况下，直接由u-boot-dtb.bin复制而来，也就是编译u-boot的最终目标 |
|    u-boot.lds    |                uboot的连接脚本                |
|    System.map    |                连接之后的符号表文件                |
|    u-boot.cfg    |              由uboot配置生成的文件               |

# 二、uboot编译流程

## 1、编译整体流程

根据一、2生成的文件说明可知简单流程如下： 
（1）各目录下built-in.o的生成

源文件、代码文件	-->	编译、汇编	-->	目标文件	-->	同目录目标文连接	-->	built-in目标文件

（2）由所有built-in.o以u-boot.lds为连接脚本通过连接来生成u-boot

built-in目标文件	-->	以u-boot.lds为连接脚本进行统一连接	-->	u-boot

（3）由u-boot生成u-boot-nodtb.bin

u-boot	-->	objcopy动作去掉符号信息表	-->	u-boot-nodtb.bin

（4）由生成uboot的dtb文件

dts文件	-->	dtc编译、打包	-->	dtb文件u-boot.dtb

（5）由u-boot-nodtb.bin和u-boot.dtb生成u-boot-dtb.bin

u-boot-nodtb.bin和u-boot.dtb	-->	追加整合两个文件	-->	u-boot-dtb.bin

（6）由u-boot-dtb.bin复制生成u-boot.bin

u-boot-dtb.bin	-->	复制		-->	u-boot.bin

## 2、具体编译流程分析

我们直接从make uboot命令分析，也就是从uboot下的Makefile的依赖关系来分析整个编译流程。 
注意，这个分析顺序和上述的整体编译流程的顺序是反着的。

（1）入口分析

在`project-x/u-boot/Makefile`中

```
all:            $(ALL-y)
ALL-y += u-boot.srec u-boot.bin u-boot.sym System.map u-boot.cfg binary_size_check
```

u-boot.bin就是我们的目标，所以后需要主要研究u-boot.bin的依赖关系。

（2）u-boot.bin的依赖关系

在project-x/u-boot/Makefile中

```
ifeq ($(CONFIG_OF_SEPARATE),y)
## CONFIG_OF_SEPARATE用于定义是否有DTB并且是否是和uboot分开编译的。
## tiny210是有定义这个宏的，所以走的是上面这路

u-boot-dtb.bin: u-boot-nodtb.bin dts/dt.dtb FORCE
        $(call if_changed,cat)
## 由u-boot-nodtb.bin和dts/dt.dtb连接在一起，先生成u-boot-dtb.bin
## $(call if_changed,cat)会调用到cmd_cat函数，具体实现我们不分析了

u-boot.bin: u-boot-dtb.bin FORCE
        $(call if_changed,copy)
## 直接将u-boot-dtb.bin复制为u-boot.bin
## $(call if_changed,copy)会调用到cmd_copy函数，具体实现我们不分析了

else
u-boot.bin: u-boot-nodtb.bin FORCE
        $(call if_changed,copy)
endif
```

对应于上述二、1（5）流程和上述二、1（6）流程。 后续有两个依赖关系要分析，分别是`u-boot-nodtb.bin和dts/dt.dtb`。 `u-boot-nodtb.bin`依赖关系参考下述二、2（3）-2（6）`dts/dt.dtb`依赖关系参考下述二、2（7） 
其中`u-boot-nodtb.bin`的依赖关系和SPL的相当类似，

（3）u-boot-nodtb.bin的依赖关系 

在`project-x/u-boot/Makefile`中

```
u-boot-nodtb.bin: u-boot FORCE
        $(call if_changed,objcopy)
        $(call DO_STATIC_RELA,$<,$@,$(CONFIG_SYS_TEXT_BASE))
        $(BOARD_SIZE_CHECK)
## $(call if_changed,objcopy)表示当依赖文件发生变化时，将依赖文件经过objcopy处理之后得到目标文件。
## 也就是通过objcopy把u-boot的符号信息以及一些无用信息去掉之后，得到了u-boot-nodtb.bin。
```

如上述Makefile代码u-boot-nodtb.bin依赖于u-boot，并且由u-boot经过objcopy操作之后得到。 
对应于上述二、1（3）流程.

（4）u-boot的依赖关系

在`project-x/u-boot/Makefile`中

```
u-boot: $(u-boot-init) $(u-boot-main) u-boot.lds FORCE
        $(call if_changed,u-boot__)
## $(call if_changed,u-boot__)来生成目标
## $(call if_changed,u-boot__)对应cmd_u-boot__命令
```

如上，u-boot依赖于`$(u-boot-init) 、$(u-boot-main)和u-boot.lds`，并且最终会调用`cmd_u-boot__`来生成`u-boot`。 
`cmd_u-boot__`实现如下 
`project-x/u-boot/Makefile`

```
 cmd_u-boot__ ?= $(LD) $(LDFLAGS) $(LDFLAGS_u-boot) -o $@ \ 
      -T u-boot.lds $(u-boot-init)                             \    
      --start-group $(u-boot-main) --end-group                 \    
      $(PLATFORM_LIBS) -Map u-boot.map
```

将cmd_u-boot__通过echo命令打印出来之后得到如下（拆分出来看的）： 
`project-x/u-boot/Makefile`

```
/project-x/build/arm-none-linux-gnueabi-4.8/bin/arm-none-linux-gnueabi-ld
 -pie --gc-sections -Bstatic -Ttext 0x23E00000 
 -o u-boot 
-T u-boot.lds 
arch/arm/cpu/armv7/start.o 
--start-group 
arch/arm/cpu/built-in.o arch/arm/cpu/armv7/built-in.o arch/arm/lib/built-in.o arch/arm/mach-s5pc1xx/built-in.o board/samsung/common/built-in.o board/samsung/tiny210/built-in.o cmd/built-in.o common/built-in.o disk/built-in.o drivers/built-in.o drivers/dma/built-in.o drivers/gpio/built-in.o drivers/i2c/built-in.o drivers/mmc/built-in.o drivers/mtd/built-in.o drivers/mtd/onenand/built-in.o drivers/mtd/spi/built-in.o drivers/net/built-in.o drivers/net/phy/built-in.o drivers/pci/built-in.o drivers/power/built-in.o drivers/power/battery/built-in.o drivers/power/fuel_gauge/built-in.o drivers/power/mfd/built-in.o drivers/power/pmic/built-in.o drivers/power/regulator/built-in.o drivers/serial/built-in.o drivers/spi/built-in.o drivers/usb/common/built-in.o drivers/usb/dwc3/built-in.o drivers/usb/emul/built-in.o drivers/usb/eth/built-in.o drivers/usb/gadget/built-in.o drivers/usb/gadget/udc/built-in.o drivers/usb/host/built-in.o drivers/usb/musb-new/built-in.o drivers/usb/musb/built-in.o drivers/usb/phy/built-in.o drivers/usb/ulpi/built-in.o fs/built-in.o lib/built-in.o net/built-in.o test/built-in.o test/dm/built-in.o 
--end-group 
arch/arm/lib/eabi_compat.o 
-L /project-x/build/arm-none-linux-gnueabi-4.8/bin/../lib/gcc/arm-none-linux-gnueabi/4.8.3 -lgcc 
-Map u-boot.map
```

可以看出上述是一条连接命令，以u-boot.lds为链接脚本，把`$(u-boot-init) 、$(u-boot-main)`的指定的目标文件连接到u-boot中。 并且已经指定输出文件为u-boot，连接脚本为`u-boot.lds`。 
连接很重要的东西就是连接标识，也就是` $(LD) $(LDFLAGS) $(LDFLAGS_u-boot)`的定义。 尝试把`$(LD) \$(LDFLAGS) \$(LDFLAGS_u-boot)) `打印出来，结果如下：

```
LD=~/project-x/build/arm-none-linux-gnueabi-4.8/bin/arm-none-linux-gnueabi-ld
LDFLAGS=
LDFLAGS_u-boot=-pie --gc-sections -Bstatic -Ttext 0x23E00000
```

LDFLAGS_u-boot定义如下

```
LDFLAGS_u-boot += -pie
LDFLAGS_u-boot += $(LDFLAGS_FINAL)
ifneq ($(CONFIG_SYS_TEXT_BASE),)
LDFLAGS_u-boot += -Ttext $(CONFIG_SYS_TEXT_BASE)
endif

## 当指定CONFIG_SYS_TEXT_BASE时，会配置连接地址。在tiny210项目中，定义如下：
## ./include/configs/tiny210.h:52:#define CONFIG_SYS_TEXT_BASE            0x23E00000

## $(LDFLAGS_FINAL)在如下几个地方定义了
## ./config.mk:19:LDFLAGS_FINAL :=
## ./config.mk:80:LDFLAGS_FINAL += -Bstatic
## ./arch/arm/config.mk:16:LDFLAGS_FINAL += --gc-sections
## 通过上述LDFLAGS_u-boot=-pie --gc-sections -Bstatic -Ttext 0x23E00000也就可以理解了
## 对应于上述二、1（2）流程。
```

对应于上述二、1（2）流程。 关于u-boot依赖的说明在（5）、（6）中继续介绍

（5）u-boot-init & u-boot-main依赖关系（代码是如何被编译的）

先看一下这两个值打印出来的

```
u-boot-init=arch/arm/cpu/armv7/start.o
u-boot-main= arch/arm/cpu/built-in.o arch/arm/cpu/armv7/built-in.o arch/arm/lib/built-in.o arch/arm/mach-s5pc1xx/built-in.o board/samsung/common/built-in.o board/samsung/tiny210/built-in.o cmd/built-in.o common/built-in.o disk/built-in.o drivers/built-in.o drivers/dma/built-in.o drivers/gpio/built-in.o drivers/i2c/built-in.o drivers/mmc/built-in.o drivers/mtd/built-in.o drivers/mtd/onenand/built-in.o drivers/mtd/spi/built-in.o drivers/net/built-in.o drivers/net/phy/built-in.o drivers/pci/built-in.o drivers/power/built-in.o drivers/power/battery/built-in.o drivers/power/fuel_gauge/built-in.o drivers/power/mfd/built-in.o drivers/power/pmic/built-in.o drivers/power/regulator/built-in.o drivers/serial/built-in.o drivers/spi/built-in.o drivers/usb/common/built-in.o drivers/usb/dwc3/built-in.o drivers/usb/emul/built-in.o drivers/usb/eth/built-in.o drivers/usb/gadget/built-in.o drivers/usb/gadget/udc/built-in.o drivers/usb/host/built-in.o drivers/usb/musb-new/built-in.o drivers/usb/musb/built-in.o drivers/usb/phy/built-in.o drivers/usb/ulpi/built-in.o fs/built-in.o lib/built-in.o net/built-in.o test/built-in.o test/dm/built-in.o
```

可以观察到是一堆目标文件的路径。这些目标文件最终都要被连接到`u-boot`中。 `u-boot-init & u-boot-main`的定义如下代码： 
`project-x/u-boot/Makefile`

```
u-boot-init := $(head-y)
## head-y定义在如下位置
## ./arch/arm/Makefile:73:head-y := arch/arm/cpu/$(CPU)/start.o

libs-y += lib/
libs-y += fs/
libs-y += net/
libs-y += disk/
libs-y += drivers/
libs-y += drivers/dma/
libs-y += drivers/gpio/
libs-y += drivers/i2c/
...
u-boot-dirs     := $(patsubst %/,%,$(filter %/, $(libs-y))) tools examples
## 过滤出路径之后，加上tools目录和example目录

libs-y          := $(patsubst %/, %/built-in.o, $(libs-y)) 
## 先加上后缀built-in.o

u-boot-main := $(libs-y)
```

那么u-boot-init & u-boot-main是如何生成的呢？ 

需要看一下对应的依赖如下：

```
$(sort $(u-boot-init) $(u-boot-main)): $(u-boot-dirs) ;
## 也就是说$(u-boot-init) $(u-boot--main)依赖于$(u-boot-dirs) 
## sort函数根据首字母进行排序并去除掉重复的。
##u-boot-dirs     := $(patsubst %/,%,$(filter %/, $(libs-y))) tools examples
## $(filter %/, $(libs-y)过滤出'/'结尾的字符串，注意，此时$(libs-y)的内容还没有加上built-in.o文件后缀
## patsubst去掉字符串中最后的'/'的字符。
## 最后u-boot-dirs打印出来如下：
## u-boot-dirs=arch/arm/cpu arch/arm/cpu/armv7 arch/arm/lib arch/arm/mach-s5pc1xx board/samsung/common board/samsung/tiny210 cmd common disk drivers drivers/dma drivers/gpio drivers/i2c drivers/mmc drivers/mtd drivers/mtd/onenand drivers/mtd/spi drivers/net drivers/net/phy drivers/pci drivers/power drivers/power/battery drivers/power/fuel_gauge drivers/power/mfd drivers/power/pmic drivers/power/regulator drivers/serial drivers/spi drivers/usb/common drivers/usb/dwc3 drivers/usb/emul drivers/usb/eth drivers/usb/gadget drivers/usb/gadget/udc drivers/usb/host drivers/usb/musb-new drivers/usb/musb drivers/usb/phy drivers/usb/ulpi fs lib net test test/dm tools examples
```

u-boot-dirs依赖规则如下：

```
PHONY += $(u-boot-dirs)
$(u-boot-dirs): prepare scripts
        $(Q)$(MAKE) $(build)=$@
## 依赖于prepare scripts
## prepare会导致prepare0、prepare1、prepare2、prepare3目标被执行，最终编译了tools目录下的东西，生成了一些工具
## 然后执行$(Q)$(MAKE) $(build)=$@
## 也就是会对每一个目标文件依次执行make \$(build)=目标文件
```

对每一个目标文件依次执行`make $(build)=目标文件 $(build)`定义如下： 
`project-x/u-boot/scripts/Kbuild.include`

```
build := -f $(srctree)/scripts/Makefile.build obj
```

以`arch/arm/mach-s5pc1xx`为例,`“$(MAKE) $(build)=$@”`展开后格式如下 
`make -f project-x/u-boot/scripts/Makefile.build obj=arch/arm/mach-s5pc1xx。`

`Makefile.build`定义`built-in.o`、`.lib`以及目标文件.o的生成规则。这个Makefile文件生成了子目录的.lib、built-in.o以及目标文件.o。 

`Makefile.build`第一个编译目标是__build，如下

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
## 以obj=arch/arm/mach-s5pc1xx为例，那么builtin-target就是arch/arm/mach-s5pc1xx/built-in.o.

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
## 当obj=arch/arm/mach-s5pc1xx时,得到对应的kbuild-file=u-boot/arch/arm/mach-s5pc1xx/Makefile
## 而在u-boot/arch/arm/mach-s5pc1xx/Makefile中定义了obj-y如下：
## obj-y   = cache.o
## obj-y   += reset.o
## obj-y   += clock.o
## 对应obj-y对应一些目标文件，由C文件编译而来，这里就不说明了。
```

后面来看目标文件的编译流程 
`./scripts/Makefile.build/scripts/Makefile.build`

```
# Built-in and composite module parts
$(obj)/%.o: $(src)/%.c $(recordmcount_source) FORCE
        $(call cmd,force_checksrc)
        $(call if_changed_rule,cc_o_c)
## 调用cmd_cc_o_c对.c文件进行编译

## cmd_cc_o_c格式如下：
cmd_cc_o_c = $(CC) $(c_flags) -c -o $@ $<
## $(CC) $(c_flags)打印出来如下：
## CC=/home/disk3/xys/temp/project-x/build/arm-none-linux-gnueabi-4.8/bin/arm-none-linux-gnueabi-gcc
## c_flags=-Wp,-MD,arch/arm/mach-s5pc1xx/.clock.o.d -nostdinc -isystem /home/disk3/xys/temp/project-x/build/arm-none-linux-gnueabi-4.8/bin/../lib/gcc/arm-none-linux-gnueabi/4.8.3/include -Iinclude -I/home/disk3/xys/temp/project-x/u-boot/include -I/home/disk3/xys/temp/project-x/u-boot/arch/arm/include -include /home/disk3/xys/temp/project-x/u-boot/include/linux/kconfig.h -I/home/disk3/xys/temp/project-x/u-boot/arch/arm/mach-s5pc1xx -Iarch/arm/mach-s5pc1xx -D__KERNEL__ -D__UBOOT__ -Wall -Wstrict-prototypes -Wno-format-security -fno-builtin -ffreestanding -Os -fno-stack-protector -fno-delete-null-pointer-checks -g -fstack-usage -Wno-format-nonliteral -D__ARM__ -marm -mno-thumb-interwork -mabi=aapcs-linux -mword-relocations -fno-pic -mno-unaligned-access -ffunction-sections -fdata-sections -fno-common -ffixed-r9 -msoft-float -pipe -march=armv7-a -I/home/disk3/xys/temp/project-x/u-boot/arch/arm/mach-s5pc1xx/include -DKBUILD_STR(s)=#s -DKBUILD_BASENAME=KBUILD_STR(clock) -DKBUILD_MODNAME=KBUILD_STR(clock)
```

对应于上述二、1（1）流程。

（6）u-boot.lds依赖关系

这里主要是为了找到一个匹配的连接文件。

```
u-boot.lds: $(LDSCRIPT) prepare FORCE
        $(call if_changed_dep,cpp_lds)
ifndef LDSCRIPT
        ifeq ($(wildcard $(LDSCRIPT)),)
                LDSCRIPT := $(srctree)/board/$(BOARDDIR)/u-boot.lds
        endif
        ifeq ($(wildcard $(LDSCRIPT)),)
                LDSCRIPT := $(srctree)/$(CPUDIR)/u-boot.lds
        endif
        ifeq ($(wildcard $(LDSCRIPT)),)
                LDSCRIPT := $(srctree)/arch/$(ARCH)/cpu/u-boot.lds
        endif
endif
## 也就是说依次从board/板级目录、cpudir目录、arch/架构/cpu/目录下去搜索u-boot.lds文件。
## 例如，tiny210(s5vp210 armv7)最终会在./arch/arm/cpu/下搜索到u-boot.lds
```

综上，最终指定了`project-X/u-boot/arch/arm/cpu/u-boot.lds`作为连接脚本。

（7）dts/dt.dtb依赖关系 

该依赖关系的主要目的是生成dtb文件。 首先了解dts文件被放在了`arch/arm/dts`里面，并通过dts下的Makefile进行选择。 
Makefile如下

`project-X/u-boot/arch/arm/dts/Makefile`

```
dtb-$(CONFIG_S5PC110) += s5pc1xx-goni.dtb
dtb-$(CONFIG_EXYNOS5) += exynos5250-arndale.dtb \
        exynos5250-snow.dtb \
        exynos5250-spring.dtb \
        exynos5250-smdk5250.dtb \
        exynos5420-smdk5420.dtb \
        exynos5420-peach-pit.dtb \
        exynos5800-peach-pi.dtb \
        exynos5422-odroidxu3.dtb
dtb-$(CONFIG_TARGET_TINY210) += \
        s5pv210-tiny210.dtb
## 填充选择dtb-y

targets += $(dtb-y)

# Add any required device tree compiler flags here
DTC_FLAGS +=
## 用于添加DTC编译选项

PHONY += dtbs
dtbs: $(addprefix $(obj)/, $(dtb-y))
        @:
## 伪目标，其依赖为$(dtb-y)加上了源路径，如下
## arch/arm/dts/s5pc1xx-goni.dtb
## arch/arm/dts/s5pv210-tiny210.dtb
## 后续会使用到这个伪目标
```

接下来看一下`dts/dt.dtb`的依赖关系

```
dtbs dts/dt.dtb: checkdtc u-boot
        $(Q)$(MAKE) $(build)=dts dtbs
## checkdtc依赖用于检查dtc的版本
## u-boot一旦发生变化那么就重新编译一遍dtb
## 重点关注命令 $(Q)$(MAKE) $(build)=dts dtbs
## 展开来就是make -f ~/project-x/u-boot/scripts/Makefile.build obj=dts dtbs
## 我们相当于值在/scripts/Makefile.build下执行了目标dtbs
```

在scripts/Makefile.build中dtbs的目标定义在哪里呢 

`project-X/u-boot/scripts/Makefile.build`

```
kbuild-file := $(if $(wildcard $(kbuild-dir)/Kbuild),$(kbuild-dir)/Kbuild,$(kbuild-dir)/Makefile)
include $(kbuild-file)
## 把对应的Makefile路径包含了进去，也就是arch/arm/dts/Makefile
## 如前面所说，arch/arm/dts/Makefile中定义了dtbs的目标
## dtbs: $(addprefix $(obj)/, $(dtb-y))
##        @:
## 这里我们就找到对应的依赖关系了，依赖就是$(obj)/, $(dtb-y)，举个例子就是arch/arm/dts/s5pv210-tiny210.dtb

include scripts/Makefile.lib
## 包含了scripts/Makefile.lib，在编译dts的时候会用到
```

接下来就是`$(obj)/, $(dtb-y)`的依赖关系了 
`project-X/u-boot/scripts/Makefile.lib`

```
$(obj)/%.dtb: $(src)/%.dts FORCE
        $(call if_changed_dep,dtc)
## 使用了通配符的方式
## 这样就通过dtc对dts编译生成了dtb文件
```

对应于上述二、1（4）流程。

# 三、一些重点定义

1、连接标志
在二、2（4）中说明。 连接命令在`cmd_u-boot__`中，如下

```
     cmd_u-boot__ ?= $(LD) $(LDFLAGS) $(LDFLAGS_u-boot) -o $@ \ 
      -T u-boot.lds $(u-boot-init)                             \    
      --start-group $(u-boot-main) --end-group                 \    
      $(PLATFORM_LIBS) -Map u-boot.map
```

连接标识如下：

```
LD=~/project-x/build/arm-none-linux-gnueabi-4.8/bin/arm-none-linux-gnueabi-ld
LDFLAGS=
LDFLAGS_u-boot=-pie --gc-sections -Bstatic -Ttext 0x23E00000
```

LDFLAGS_u-boot定义如下

```
LDFLAGS_u-boot += -pie
LDFLAGS_u-boot += $(LDFLAGS_FINAL)
ifneq ($(CONFIG_SYS_TEXT_BASE),)
LDFLAGS_u-boot += -Ttext $(CONFIG_SYS_TEXT_BASE)
endif
```

‘-o’指定了输出文件是u-boot，’-T’是指定了连接脚本是当前目录下的u-boot.lds, -Ttext指定了连接地址是`CONFIG_SYS_TEXT_BASE`。

2、连接地址

在二、2（4）中说明。 `CONFIG_SYS_TEXT_BASE`指定了`u-boot.bin`的连接地址。这个地址也就是uboot的起始运行地址。 
对于tiny210，其定义如下（可以进行修改） 

`/include/configs/tiny210.h`

```
#define CONFIG_SYS_TEXT_BASE            0x23E00000
```

3、连接脚本

在二、2（6）中说明。

 `u-boot/arch/arm/cpu/u-boot.lds`

```
u-boot.lds: $(LDSCRIPT) prepare FORCE
        $(call if_changed_dep,cpp_lds)
ifndef LDSCRIPT
        ifeq ($(wildcard $(LDSCRIPT)),)
                LDSCRIPT := $(srctree)/board/$(BOARDDIR)/u-boot.lds
        endif
        ifeq ($(wildcard $(LDSCRIPT)),)
                LDSCRIPT := $(srctree)/$(CPUDIR)/u-boot.lds
        endif
        ifeq ($(wildcard $(LDSCRIPT)),)
                LDSCRIPT := $(srctree)/arch/$(ARCH)/cpu/u-boot.lds
        endif
endif
```

综上，最终指定了`project-X/u-boot/arch/arm/cpu/u-boot.lds`作为连接脚本。

# 四、uboot链接脚本说明

## 1、连接脚本整体分析

相对比较简单，直接看连接脚本的内容`project-x/u-boot/arch/arm/cpu/u-boot.lds `

参考如下，只提取了一部分：

```
ENTRY(_start)
//定义了地址为_start的地址，所以我们分析代码就是从这个函数开始分析的！！！
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

    . = ALIGN(4);

//以下定义u_boot_list段，具体功能未知
    . = ALIGN(4);
    .u_boot_list : {
        KEEP(*(SORT(.u_boot_list*)));
    }

    . = ALIGN(4);

    .image_copy_end :
    {
        *(.__image_copy_end)
    }
//定义__image_copy_end符号的地址为当前地址
//从__image_copy_start 到__image_copy_end的区间，包含了代码段和数据段。

    .rel_dyn_start :
    {
        *(.__rel_dyn_start)
    }
//定义__rel_dyn_start 符号的地址为当前地址，后续在代码中会使用到

    .rel.dyn : {
        *(.rel*)
    }

    .rel_dyn_end :
    {
        *(.__rel_dyn_end)
    }
//定义__rel_dyn_end 符号的地址为当前地址，后续在代码中会使用到
//从__rel_dyn_start 到__rel_dyn_end 的区间，应该是在代码重定向的过程中会使用到，后续遇到再说明。

    .end :
    {
        *(.__end)
    }

    _image_binary_end = .;
//定义_image_binary_end 符号的地址为当前地址

// 以下定义堆栈段
    .bss_start __rel_dyn_start (OVERLAY) : {
        KEEP(*(.__bss_start));
        __bss_base = .;
    }

    .bss __bss_base (OVERLAY) : {
        *(.bss*)
         . = ALIGN(4);
         __bss_limit = .;
    }

    .bss_end __bss_limit (OVERLAY) : {
        KEEP(*(.__bss_end));
    }
｝
```

## 2、以下以.vectors段做说明，

`.vectors是uboo`t链接脚本第一个链接的段，也就是_start被链接进来的部分，也负责链接异常中断向量表 

先看一下代码`project-x/u-boot/arch/arm/lib/vectors.S`

```
.globl _start
    .section ".vectors", "ax"
@@ 定义在.vectors段中

_start:

    b   reset
    ldr pc, _undefined_instruction
    ldr pc, _software_interrupt
    ldr pc, _prefetch_abort
    ldr pc, _data_abort
    ldr pc, _not_used
    ldr pc, _irq
    ldr pc, _fiq

    .globl  _undefined_instruction
    .globl  _software_interrupt
    .globl  _prefetch_abort
    .globl  _data_abort
    .globl  _not_used
    .globl  _irq
    .globl  _fiq
@@ 定义了异常中断向量表
```

通过`“arm-none-linux-gnueabi-objdump -D u-boot > uboot_objdump.txt”`进行反编译之后，得到了如下指令

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
```

## 3、符号表中需要注意的符号

前面我们说过了在tiny210中把连接地址设置为0x23e00000。 

`project-x/build/out/u-boot/spl/u-boot.map`

```
Linker script and memory map

Address of section .text set to 0x23e00000
.text           0x23e00000    0x29b28
 *(.__image_copy_start)
 .__image_copy_start
                0x23e00000        0x0 arch/arm/lib/built-in.o
                0x23e00000                __image_copy_start
 *(.vectors)
 .vectors       0x23e00000      0x300 arch/arm/lib/built-in.o
                0x23e00000                _start
                0x23e00020                _undefined_instruction
                0x23e00024                _software_interrupt
                0x23e00028                _prefetch_abort
                0x23e0002c                _data_abort
                0x23e00030                _not_used
                0x23e00034                _irq
                0x23e00038                _fiq
                0x23e00040                IRQ_STACK_START_IN
 *(.__image_copy_end)
 .__image_copy_end
                0x23e36b78        0x0 arch/arm/lib/built-in.o
 *(.__rel_dyn_start)
 .__rel_dyn_start
                0x23e36b78        0x0 arch/arm/lib/built-in.o
 *(.__rel_dyn_end)
 .__rel_dyn_end
                0x23e3cbb8        0x0 arch/arm/lib/built-in.o
                0x23e3cbb8                _image_binary_end = .
 *(.__bss_start)
 .__bss_start   0x23e36b78        0x0 arch/arm/lib/built-in.o
                0x23e36b78                __bss_start
 .__bss_end     0x23e6b514        0x0 arch/arm/lib/built-in.o
                0x23e6b514                __bss_end
```

重点关注 

_image_copy_start & __image_copy_end 
界定了代码空间的位置，用于重定向代码的时候使用，在uboot relocate的过程中，需要把这部分拷贝到uboot的新的地址空间中，后续在新地址空间中运行。 

_start 
在u-boot-spl.lds中ENTRY(_start)，也就规定了代码的入口函数是_start。所以后续分析代码的时候就是从这里开始分析。 

_rel_dyn_start & __rel_dyn_end 
由链接器生成，存放了绝对地址符号的label的地址，用于修改uboot relocate过程中修改绝对地址符号的label的值。 

_image_binary_end