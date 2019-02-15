# uboot流程 - 启动Kernel(一)

# 一. uImage

编译kernel之后，会生成Image或者压缩过的zImage。但是这两种镜像的格式并没有办法提供给uboot的足够的信息来进行load、jump或者验证操作等等。因此，uboot提供了mkimage工具，来将kernel制作为uboot可以识别的格式，将生成的文件称之为uImage。 


uboot支持两种类型的uImage。

## Legacy-uImage 

在kernel镜像的基础上，加上64Byte的信息提供给uboot使用。

## FIT-uImage 

以类似FDT的方式，将kernel、fdt、ramdisk等等镜像打包到一个image file中，并且加上一些需要的信息（属性）。uboot只要获得了这个image file，就可以得到kernel、fdt、ramdisk等等镜像的具体信息和内容。

Legacy-uImage实现较为简单，并且长度较小。但是实际上使用较为麻烦，需要在启动kernel的命令中额外添加fdt、ramdisk的加载信息。 而FIT-uImage实现较为复杂，但是使用起来较为简单，兼容性较好,（可以兼容多种配置）。但是需要的额外信息也较长。

# 二. Legacy-uImage

## 1. 使能需要打开的宏

```
CONFIG_IMAGE_FORMAT_LEGACY=y
```

注意，这个宏在自动生成的autoconf.mk中会自动配置，不需要额外配置。

## 2. 如何制作 & 使用

（1）工具mkimage 

编译完uboot之后会在uboot的tools目录下生成mkimage可执行文件 

（2）命令

```
mkimage -A arm -O linux -C none -T kernel -a 0x20008000 -e 0x20008040 -n Linux_Image -d zImage uImage 

各个参数意义如下
Usage: mkimage -l image
          -l ==> list image header information
       mkimage [-x] -A arch -O os -T type -C comp -a addr -e ep -n name -d data_file[:data_file...] image
          -A ==> set architecture to 'arch'  // 体系
          -O ==> set operating system to 'os' // 操作系统
          -T ==> set image type to 'type' // 镜像类型
          -C ==> set compression type 'comp' // 压缩类型
          -a ==> set load address to 'addr' (hex) // 加载地址
          -e ==> set entry point to 'ep' (hex) // 入口地址
          -n ==> set image name to 'name' // 镜像名称，注意不能超过32B
          -d ==> use image data from 'datafile' // 输入文件
          -x ==> set XIP (execute in place) 
```

（3）使用 

将生成的Legacy-uImage下载到参数中指定的load address中，使用bootm ‘实际load address’命令跳转到kernel中。 

但是注意，如果使用Legacy-uImage后面还需要跟上文件系统的ram地址以及dtb的ram地址，否则可能会导致bootm失败。 

格式如下：

```
bootm Legacy-uImage加载地址 ramdisk加载地址 dtb加载地址
```

## 3. 和zImage的区别

```
-rw-rw-rw-  1 hlos hlos 1560120 Dec  5 14:46 uImage
-rwxrwxrwx  1 hlos hlos 1560056 Nov 30 15:42 zImage*
```

可以看出uImage比zImage多了64Byte，通过对比可以发现这64Byte的数据是加载zImage的头部的。 

直接查看这64Byte的数据，通过od命令查看如下：

```
hlos@node4:boot$ od -tx1 -tc -Ax -N64 uImage
000000  27  05  19  56  5a  f3  f7  8e  58  45  0d  3d  00  17  cd  f8
         ' 005 031   V   Z 363 367 216   X   E  \r   =  \0 027 315 370
000010  20  00  80  00  20  00  80  40  e2  4b  43  b6  05  02  02  00
            \0 200  \0      \0 200   @ 342   K   C 266 005 002 002  \0
000020  4c  69  6e  75  78  5f  49  6d  61  67  65  00  00  00  00  00
         L   i   n   u   x   _   I   m   a   g   e  \0  \0  \0  \0  \0
000030  00  00  00  00  00  00  00  00  00  00  00  00  00  00  00  00
        \0  \0  \0  \0  \0  \0  \0  \0  \0  \0  \0  \0  \0  \0  \0  \0
```

Legacy-uImage是在zImage的基础上加上64Byte的头部。 下面通过Legacy-uImage的头部数据结构来解析这串数据。

## 4. 格式头说明

uboot用image_header来表示Legacy-uImage的头部

```
typedef struct image_header {
    __be32      ih_magic;   /* Image Header Magic Number    */   // 幻数头，用来校验是否是一个Legacy-uImage
    __be32      ih_hcrc;    /* Image Header CRC Checksum    */ // 头部的CRC校验值
    __be32      ih_time;    /* Image Creation Timestamp */ // 镜像创建的时间戳
    __be32      ih_size;    /* Image Data Size      */ // 镜像数据长度
    __be32      ih_load;    /* Data  Load  Address      */ // 加载地址
    __be32      ih_ep;      /* Entry Point Address      */ // 入口地址
    __be32      ih_dcrc;    /* Image Data CRC Checksum  */ // 镜像的CRC校验
    uint8_t     ih_os;      /* Operating System     */ // 操作系统类型
    uint8_t     ih_arch;    /* CPU architecture     */ // 体系 
    uint8_t     ih_type;    /* Image Type           */ // 镜像类型
    uint8_t     ih_comp;    /* Compression Type     */ // 压缩类型
    uint8_t     ih_name[IH_NMLEN];  /* Image Name       */ // 镜像名
} image_header_t;
#define IH_NMLEN        32  /* Image Name Length        */
```

可以发现这个头部就是64Byte。根据上述3来说明一下这个数据结构

- ih_magic 

  27 05 19 56 

  头部幻数，用来判断这个是否是Legacy-uImage的头部。为0x27051956则表示是一个Legacy-uImage。

- ih_size 

  00 17 cd f8 

  数据镜像长度。这里是0x17cdf8，也就是1560056，和我们前面测得的zImage的长度一致。

- ih_load 

  20 00 80 00 

  加载地址，也就是0x20008000， 和我们执行mkimage使用的参数一致。

- ih_ep 

  20 00 80 40 

  入口地址，也就是0x20008040， 和我们执行mkimage使用的参数一致。


# 三. FIT-uImage

## 0. 原理说明

flattened image tree，类似于FDT(flattened device tree)的一种实现机制。其通过一定语法和格式将一些需要使用到的镜像（例如kernel、dtb以及文件系统）组合到一起生成一个image文件。其主要使用四个组件。

- its文件 

  image source file，类似于dts文件，负责描述要声称的image的的信息。需要自行进行构造。

- itb文件 

  最终得到的image文件，类似于dtb文件，也就是uboot可以直接对其进行识别和解析的FIT-uImage。

- mkimage 

  mkimage则负责dtc的角色，用于通过解析its文件、获取对应的镜像，最终生成一个uboot可以直接进行识别和解析的itb文件。

- image data file 

  实际使用到的镜像文件。

mkimage将its文件以及对应的image data file，打包成一个itb文件，也就是uboot可以识别的image file（FIT-uImage）。我们将这个文件下载到么memory中，使用bootm命令就可以执行了。

## 1. 使能需要打开的宏

```
CONFIG_FIT=y
```

## 2. 如何制作 & 使用

（1）its文件制作 

因为mkimage是根据its文件中的描述来打包镜像生成itb文件（FIT-uImage），所以首先需要制作一个its文件，在its文件中描述需要被打包的镜像，主要是kernel镜像，dtb文件，ramdisk镜像。 

简单的例子如下：

```
/*
 * U-Boot uImage source file for "X project"
 */

/dts-v1/;

/ {
    description = "U-Boot uImage source file for X project";
    #address-cells = <1>;

    images {
        kernel@tiny210 {
            description = "Unify(TODO) Linux kernel for project-x";
            data = /incbin/("/home/hlos/code/xys/temp/project-x/build/out/linux/arch/arm/boot/zImage");
            type = "kernel";
            arch = "arm";
            os = "linux";
            compression = "none";
            load = <0x20008000>;
            entry = <0x20008000>;
        };
        fdt@tiny210 {
            description = "Flattened Device Tree blob for project-x";
            data = /incbin/("/home/hlos/code/xys/temp/project-x/build/out/linux/arch/arm/boot/dts/s5pv210-tiny210.dtb");
            type = "flat_dt";
            arch = "arm";
            compression = "none";
        };
        ramdisk@tiny210 {
            description = "Ramdisk for project-x";
            data = /incbin/("/home/hlos/code/xys/temp/project-x/build/out/rootfs/initramfs.gz");
            type = "ramdisk";
            arch = "arm";
            os = "linux";
            compression = "gzip";
        };
    };

    configurations {
        default = "conf@tiny210";
        conf@tiny210 {
            description = "Boot Linux kernel with FDT blob";
            kernel = "kernel@tiny210";
            fdt = "fdt@tiny210";
            ramdisk = "ramdisk@tiny210";
        };
    };
};
```

注意，可以有多个kernel节点或者fdt节点等等，兼容性更强。同时，可以有多种configurations，来对kernel、fdt、ramdisk来进行组合，使FIT-uImage可以兼容于多种板子，而无需重新进行编译烧写。

（2）生成FIT-uImage 

生成的命令相对Legacy-uImage较为简单，因为信息都在its文件里面描述了

````
mkimage -f ITS文件 要生成的ITB文件，如下：
${UBOOT_OUT_DIR}/tools/mkimage -f ${UIMAGE_ITS_FILE} ${UIMAGE_ITB_FILE}
````

最终在project X项目上生成了xprj_uImage.itb，这就是我们需要的FIT-uImage文件。

（3）使用 
将生成的FIT-uImage下载到参数中指定的load address中，使用bootm ‘实际load address’命令跳转到kernel中。 uboot会自动解析出FIT-uImage中，kernel、ramdisk、dtb的信息，使用起来相当方便。

## 3. 简单说明

FIT-uImage的格式类似于DTB。uboot会去解析出FIT-uImage中的configurations节点，根据节点选择出需要的kernel、dtb、rootfs。因此，在节点的基础上，添加很多节点信息，提供uboot使用。





















