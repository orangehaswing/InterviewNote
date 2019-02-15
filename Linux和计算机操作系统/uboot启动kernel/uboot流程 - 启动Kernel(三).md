# uboot流程 - 启动Kernel(三)

# 一. 说明

bootm的过程中，从uImage解析出kernel信息主要是在bootm_find_os中实现的。 

这里注意，这些信息主要是在mkimage工具生成uImage的时候附加上去的，并不是kernel原生镜像的内容。 因为uImage有两种类型，Legacy-uImage和FIT-uImage ，这两个类型的格式是不一样的，因此，uboot从这两种uImage解析出kernel信息的方式也是不一样的。 

后面会分别介绍这两种类型的解析流程。

# 二. kernel信息的存放位置

## 1. 存放位置

kernel信息主要包括两方面内容：

- kernel的镜像信息（包括其加载地址）：放在bootm_headers_t images -> image_info_t os中
- kernel的入口地址：放在bootm_headers_t images -> ulong ep中

如下（过滤掉一些无关部分） 

include/image.h

```
typedef struct bootm_headers {
    image_info_t    os;     /* os image info */
    ulong       ep;     /* entry point of OS */
}
```

因此，bootm_find_os的主要目的是实现bootm_headers_t images中的image_info_t os和ulong ep的成员。

## 2. image_info_t

image_info_t用来描述kernel的镜像信息。包括头部信息、加载地址、kernel镜像地址和长度等等。 

其数据结构如下： 

include/image.h

```
typedef struct image_info {
    ulong       start, end;     /* start/end of blob */   // 包括附加节点信息之内的整个kernel节点的起始地址和结束地址
    ulong       image_start, image_len; /* start of image within blob, len of image */ // kernel镜像的起始地址，镜像长度
    ulong       load;           /* load addr for the image */ // 镜像的加载地址
    uint8_t     comp, type, os;     /* compression, type of image, os type */ // 镜像的压缩格式、镜像类型、操作系统类型
    uint8_t     arch;           /* CPU architecture */ // CPU体系结构
} image_info_t;
```

type对应于“kennel”，os对应于“linux”。

综上，后续我们解析uImage的时候的主要目的是填充image_info_t os和ulong ep，这句话多强调几遍。

# 三. Legacy-uImage中kernel信息的解析

Legacy-uImage中kernel信息的解析相对较为简单。

## 1. Legacy-uImage的生成

首先要知道Legacy-uImage是怎么生成的。 如下：

```
mkimage -A arm -O linux -C none -T kernel -a 0x20008000 -e 0x20008040 -n Linux_Image -d zImage uImage 

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

通过上述，可以知道kernel的信息都是在这个时候定义的（而不是kernel原生镜像原来就有的）。 生成uImage会包含64Byte的格式头，如下：

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

所以，uboot解析Legacy-uImage主要就是解析这64Byte的头部的内容。

## 2. Legacy-uImage头部数据结构

uboot使用了struct image_header来对应这uImage的64Byte的头部。

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

所以，只需要将image_header的指针对应到uImage的起始地址，就可以得到image_header的实体了。uImage的这个头部指针image_header会被存储在bootm_headers 中，如下：

```
typedef struct bootm_headers {
    /*
     * Legacy os image header, if it is a multi component image
     * then boot_get_ramdisk() and get_fdt() will attempt to get
     * data from second and third component accordingly.
     */
    image_header_t  *legacy_hdr_os;     /* image header pointer */  // 头部指针
    image_header_t  legacy_hdr_os_copy; /* header copy */   // 头部信息的备份的指针
    ulong       legacy_hdr_valid; // 用于表示Legacy-uImage的头部指针是否可用，也就是这是否是一个Legacy-uImage。
```

## 3. 解析Legacy-uImage中kernel信息的代码流程

从bootm_find_os入口开始说明。 

代码如下，过滤掉无关部分： 
common/bootm.c

```
static int bootm_find_os(cmd_tbl_t *cmdtp, int flag, int argc,
             char * const argv[])
{
// uImage的地址对应传进来的参数argv[0].
    const void *os_hdr;
    bool ep_found = false;
    int ret;

    /* get kernel image header, start address and length */
    os_hdr = boot_get_kernel(cmdtp, flag, argc, argv,
            &images, &images.os.image_start, &images.os.image_len);
        // 通过调用boot_get_kernel获取到Legacy-uImage的头部，并将其指针返回给os_hdr。
        // 以images.os.image_start和images.os.image_len为其参数，在boot_get_kernel中会自动设置其值

    /* get image parameters */
    switch (genimg_get_format(os_hdr)) {
#if defined(CONFIG_IMAGE_FORMAT_LEGACY)
    case IMAGE_FORMAT_LEGACY:
        images.os.type = image_get_type(os_hdr); // 从头部image_header 中获得镜像类型并存储到images.os.type中
        images.os.comp = image_get_comp(os_hdr);  // 从头部image_header 中获得压缩类型类型并存储到images.os.comp中
        images.os.os = image_get_os(os_hdr); // 从头部image_header 中获得操作系统类型并存储到images.os.os中

        images.os.end = image_get_image_end(os_hdr); //获得uImage的结束地址
        images.os.load = image_get_load(os_hdr); // 从头部image_header 中获得加载地址并存储到images.os.load中
        images.os.arch = image_get_arch(os_hdr); // 从头部image_header 中获得cpu体系结构类型并存储到images.os.arch中
        break;
#endif
    if (images.os.arch == IH_ARCH_I386 ||
        images.os.arch == IH_ARCH_X86_64) {
    } else if (images.legacy_hdr_valid) {
        images.ep = image_get_ep(&images.legacy_hdr_os_copy); //从头部image_header 中获得加载地址并存储到images.ep中
        }
    images.os.start = map_to_sysmem(os_hdr);

    return 0;
}
```

通过上述代码就完成了bootm_headers_t images中的image_info_t os和ulong ep的成员的实现。 而这里的代码的核心是boot_get_kernel，会实现uImage的类型的判断、和Legacy-uImage的头部image_header的设置，并且将image_header和bootm_headers进行关联。

## 4. boot_get_kernel

解析uImage的头部的核心函数。 

代码如下，过滤掉无关部分： 

common/bootm.c

```
static const void *boot_get_kernel(cmd_tbl_t *cmdtp, int flag, int argc,
                   char * const argv[], bootm_headers_t *images,
                   ulong *os_data, ulong *os_len)
{
#if defined(CONFIG_IMAGE_FORMAT_LEGACY)
    image_header_t  *hdr;
#endif
    ulong       img_addr;
    const void *buf;
    const char  *fit_uname_config = NULL;
    const char  *fit_uname_kernel = NULL;


    img_addr = genimg_get_kernel_addr_fit(argc < 1 ? NULL : argv[0],
                          &fit_uname_config,
                          &fit_uname_kernel);
        // 因为传进来的是kernel地址argv[0]是一个字符串类型，这里会将其转换成地址类型（长整型）

    /* check image type, for FIT images get FIT kernel node */
    *os_data = *os_len = 0;
    buf = map_sysmem(img_addr, 0);
        // 映射到物理地址，因为MMU没有一般在uboot中没有打开，所以这里img_addr一般不会发生变化

    switch (genimg_get_format(buf)) {
        // 在这里会根据幻数来判断uImage的类型，假设这里已经判断出是Legacy-uImage类型

#if defined(CONFIG_IMAGE_FORMAT_LEGACY)
    case IMAGE_FORMAT_LEGACY:
        printf("## Booting kernel from Legacy Image at %08lx ...\n",
               img_addr);
                // 这里会在log中打印出“## Booting kernel from Legacy Image at 0x20008000”

        hdr = image_get_kernel(img_addr, images->verify);
        if (!hdr)
            return NULL;
                // 在这里会将头部指针hdr设置成img_addr，然后判断幻数，CRC值是否正确，
               // 通过这一步，就已经成功设置了Legacy-uImage的头部指针，其内容也就相应确定下来了！！！

        /* get os_data and os_len */
        switch (image_get_type(hdr)) {
        case IH_TYPE_KERNEL:
        case IH_TYPE_KERNEL_NOLOAD:
            *os_data = image_get_data(hdr); 
                        // 通过image_header获得kernel镜像的地址，存储在os_data中，也就是images.os.image_start中
            *os_len = image_get_data_size(hdr);
                        // 通过image_header获得kernel镜像的长度，存储在os_len中，也就是images.os.image_len中
            break;
        }

        memmove(&images->legacy_hdr_os_copy, hdr,
            sizeof(image_header_t)); // 为uImage的头部做一个备份
        images->legacy_hdr_os = hdr;  // 存储uImage的头部指针到images->legacy_hdr_os中
        images->legacy_hdr_valid = 1; // 说明uImage的头部指针指向的头部是可用的，这是一个Legacy-uImage类型
        break;
#endif
    }
    return buf;
}
```

# 四. FIT-uImage中kernel信息的解析

## 1. 原理简单介绍

flattened image tree，类似于FDT(flattened device tree)的一种实现机制。其通过一定语法和格式将一些需要使用到的镜像（例如kernel、dtb以及文件系统）组合到一起生成一个image文件。 
而kernel镜像也是作为FIT的configure中的一个节点，其信息则是以节点中的属性来进行描述的。 

而uboot的工作，就是要从FIT中提取相应的kernel节点，在节点中获取相应的属性，从而得到kernel的信息。其方式和FDT相当类似。 得到的kernel的信息之后填入bootm_headers_t images中的image_info_t os和ulong ep中即可。

2. 生成说明
--------------------- 
kernel信息则是在its中进行说明，简单例子如下：

```
/ {
    images {
        kernel@tiny210 {
            description = "Unify(TODO) Linux kernel for project-x";
            data = /incbin/("/home/hlos/code/xys/temp/project-x/build/out/linux/arch/arm/boot/zImage");
            type = "kernel"; // 镜像类型是kernel
            arch = "arm";    // 体系
            os = "linux";    // 操作系统
            compression = "none";    // 压缩类型
            load = <0x20008000>;    // 加载地址
            entry = <0x20008040>;    // 入口地址
        };
    configurations {
        default = "conf@tiny210";
        conf@tiny210 {
            description = "Boot Linux kernel with FDT blob";
            kernel = "kernel@tiny210"; // 指定kernel为节点“kernel@tiny210”描述的信息
        };
    };
```

可以看出kernel信息都在“kernel@tiny210”节点中进行了描述，注意，就连kernel镜像，也是作为这个节点的属性“data”。 

所以uboot解析FIT-uImage中的kernel信息的原理是：

- 从itb（FIT-uImage）文件中解析出configurations节点
- 从configurations获取要使用的kernel的节点的路径（偏移）
- 从kernel的节点中获取各种属性，这些属性就是kernel的信息。
- 包括kernel的镜像也是在data属性中的。

## 3. 数据结构说明

与struct bootm_headers之间的关系 。FIT-uImage中kernel是以节点的方式进行描述的，其节点也有自己的头部。 

其节点信息存储在struct bootm_headers中，如下：

```
typedef struct bootm_headers {
#if IMAGE_ENABLE_FIT
    const char  *fit_uname_cfg; /* configuration node unit name */ // configuration节点名称

    void        *fit_hdr_os;    /* os FIT image header */  // itb的头部，对应就是FIT-uImage的起始地址
    const char  *fit_uname_os;  /* os subimage node unit name */ // kernel节点的名称
    int     fit_noffset_os; /* os subimage node offset */ // kernel节点的节点偏移，直接代表了kernel节点
} bootm_headers_t;
```

## 4. 解析FIT-uImage中kernel信息的代码流程

从bootm_find_os入口开始说明。 代码如下，过滤掉无关部分： 
common/bootm.c

```
static int bootm_find_os(cmd_tbl_t *cmdtp, int flag, int argc,
             char * const argv[])
{
// uImage的地址对应传进来的参数argv[0].
    const void *os_hdr;
    bool ep_found = false;
    int ret;

    /* get kernel image header, start address and length */
    os_hdr = boot_get_kernel(cmdtp, flag, argc, argv,
            &images, &images.os.image_start, &images.os.image_len);
    if (images.os.image_len == 0) {
        puts("ERROR: can't get kernel image!\n");
        return 1;
    }

        // 通过调用boot_get_kernel，来获取itb（FIT-uImage）的头部指针，存储在os_hdr中
        // 以images.os.image_start和images.os.image_len为其参数，在boot_get_kernel中会自动设置其值
        // 同时，在boot_get_kernel中会设置images关于itb中kernel节点的信息fit_hdr_os、fit_uname_os和fit_noffset_os
        // 后续继续说明

    /* get image parameters */
    switch (genimg_get_format(os_hdr)) {
#if IMAGE_ENABLE_FIT
    case IMAGE_FORMAT_FIT:
        if (fit_image_get_type(images.fit_hdr_os,
                       images.fit_noffset_os,
                       &images.os.type)) {
            return 1;
        }
                // 调用fit_image_get_type从itb的kerne节点中解析出“type”属性，存储在images.os.type中。

        if (fit_image_get_comp(images.fit_hdr_os,
                       images.fit_noffset_os,
                       &images.os.comp)) {
            return 1;
        }
                // 调用fit_image_get_comp从itb的kerne节点中解析出“comp”属性，存储在images.os.comp中。

        if (fit_image_get_os(images.fit_hdr_os, images.fit_noffset_os,
                     &images.os.os)) {
            return 1;
        }
                // 调用fit_image_get_comp从itb的kerne节点中解析出“os”属性，存储在images.os.os中。

        if (fit_image_get_arch(images.fit_hdr_os,
                       images.fit_noffset_os,
                       &images.os.arch)) {
            return 1;
        }
                // 调用fit_image_get_comp从itb的kerne节点中解析出“arch”属性，存储在images.os.arch中。

        images.os.end = fit_get_end(images.fit_hdr_os);

        if (fit_image_get_load(images.fit_hdr_os, images.fit_noffset_os,
                       &images.os.load)) {
            return 1;
        }
                // 调用fit_image_get_comp从itb的kerne节点中解析出“load”属性，存储在images.os.load中。
        break;
#endif
    if (images.os.arch == IH_ARCH_I386 ||
        images.os.arch == IH_ARCH_X86_64) {
...
#if IMAGE_ENABLE_FIT
    } else if (images.fit_uname_os) {
        int ret;

        ret = fit_image_get_entry(images.fit_hdr_os,
                      images.fit_noffset_os, &images.ep);
                // 调用fit_image_get_comp从itb的kerne节点中解析出“ep”属性，存储在images.os.ep中。
        if (ret) {
            puts("Can't get entry point property!\n");
            return 1;
        }
#endif
    } 
```

通过上述代码就完成了bootm_headers_t images中的image_info_t os和ulong ep的成员的实现。 

而这里的代码的核心是boot_get_kernel，会实现uImage的类型的判断、和FIT-uImage的头部节点信息的设置，并且将FIT-uImage的kernel的节点信息和bootm_headers进行关联。

## 5. boot_get_kernel

解析uImage的头部的核心函数。 代码如下，过滤掉无关部分： 

common/bootm.c

```
static const void *boot_get_kernel(cmd_tbl_t *cmdtp, int flag, int argc,
                   char * const argv[], bootm_headers_t *images,
                   ulong *os_data, ulong *os_len)
{
    ulong       img_addr;
    const void *buf;
    const char  *fit_uname_config = NULL;
    const char  *fit_uname_kernel = NULL;
#if IMAGE_ENABLE_FIT
    int     os_noffset;
#endif

    img_addr = genimg_get_kernel_addr_fit(argc < 1 ? NULL : argv[0],
                          &fit_uname_config,
                          &fit_uname_kernel);
        // 因为传进来的是kernel地址argv[0]是一个字符串类型，这里会将其转换成地址类型（长整型）

    /* check image type, for FIT images get FIT kernel node */
    *os_data = *os_len = 0;
    buf = map_sysmem(img_addr, 0);
        // 映射到物理地址，因为MMU没有一般在uboot中没有打开，所以这里img_addr一般不会发生变化

    switch (genimg_get_format(buf)) {
        // 在这里会根据幻数来判断uImage的类型，假设这里已经判断出是FIT-uImage类型

#if IMAGE_ENABLE_FIT
    case IMAGE_FORMAT_FIT:
        os_noffset = fit_image_load(images, img_addr,
                &fit_uname_kernel, &fit_uname_config,
                IH_ARCH_DEFAULT, IH_TYPE_KERNEL,
                BOOTSTAGE_ID_FIT_KERNEL_START,
                FIT_LOAD_IGNORED, os_data, os_len);
                // 在fit_image_load中会去查找IH_TYPE_KERNEL指定的节点
                // 对应kernel = "kernel@tiny210"; 指定的节点，返回其节点偏移，并且将data属性的值（代表了kernel镜像）的地址和长度
                // 设置到os_data和os_len中，也就是images.os.image_start和images.os.image_len中
                // 具体自己参考代码
        if (os_noffset < 0)
            return NULL;

        images->fit_hdr_os = map_sysmem(img_addr, 0);
                // 设置 itb的头部，对应就是FIT-uImage的起始地址
        images->fit_uname_os = fit_uname_kernel;
                // 设置kernel节点的名称
        images->fit_uname_cfg = fit_uname_config;
                // 设置configuration节点名称
        images->fit_noffset_os = os_noffset;
                // 设置kernel节点的节点偏移，直接代表了kernel节点
        break;
#endif
    return buf;
}
```

通过上述代码，就得到了itb的地址和itb（FIT-uImage）中kernel的节点偏移，类似于fdt的操作，后续就可以通过这两个itb的地址和itb（FIT-uImage）中kernel的节点偏移来获得kernel节点的属性。
