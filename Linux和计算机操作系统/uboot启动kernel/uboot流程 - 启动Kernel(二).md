# uboot流程 - 启动Kernel(二)

# 一. bootm说明

bootm这个命令用于启动一个操作系统映像。它会从映像文件的头部取得一些信息，这些信息包括：映像文件的基于的cpu架构、其操作系统类型、映像的类型、压缩方式、映像文件在内存中的加载地址、映像文件运行的入口地址、映像文件名等。

 紧接着bootm将映像加载到指定的地址，如果需要的话，还会解压映像并传递必要有参数给内核，最后跳到入口地址进入内核。 

[这里的描述参考](http://blog.chinaunix.net/uid-20799298-id-99666.html)

需要打开的宏

```
CONFIG_BOOTM_LINUX=y
CONFIG_CMD_BOOTM=y
```

# 二. bootm使用方式

uImage有两种格式。

- Legacy-uImage ,对于Legacy-uImage，我们需要另外加载ramdisk和fdt到RAM上面。 

  执行的命令如下

```
假设Legacy-uImage的加载地址是0x20008000，ramdisk的加载地址是0x21000000，fdt的加载地址是0x22000000

(1) 只加载kernel的情况下
bootm 0x20008000

(2) 加载kernel和ramdisk
bootm 0x20008000 0x21000000

(3) 加载kernel和fdt
bootm 0x20008000 - 0x22000000

(4) 加载kernel、ramdisk、fdt
bootm 0x20008000 0x21000000 0x22000000
```

- FIT-uImage ,对于FIT-uImage，kernel镜像、ramdisk镜像和fdt都已经打包到FIT-uImage的镜像中了。 

  执行的命令如下

```
假设FIT-uImage的加载地址是0x30000000，启动kernel的命令如下：
bootm 0x30000000
```

# 三. bootm执行流程

- 对应U_BOOT_CMD 我们找到bootm命令对应的U_BOOT_CMD如下： 
  cmd/bootm.c

```
U_BOOT_CMD(
    bootm,  CONFIG_SYS_MAXARGS, 1,  do_bootm,
    "boot application image from memory", bootm_help_text
);
```

- do_bootm参数说明,当执行bootm命令时，do_bootm会被调用，参数如下：

```
当执行‘bootm 0x20008000 0x21000000 0x22000000’
int do_bootm(cmd_tbl_t *cmdtp, int flag, int argc, char * const argv[])
cmdtp：传递的是bootm的命令表项指针，也就是_u_boot_list_2_cmd_2_bootm的指针
argc=4
argv[0]="bootm", argv[1]=0x20008000, arv[2]=0x21000000, argv[3]=0x22000000
```

- do_bootm实现 
  do_bootm实现如下：

```
/*******************************************************************/
/* bootm - boot application image from image in memory */
/*******************************************************************/

int do_bootm(cmd_tbl_t *cmdtp, int flag, int argc, char * const argv[])
{
    /* determine if we have a sub command */
    argc--; argv++;
    if (argc > 0) {
        char *endp;

        simple_strtoul(argv[0], &endp, 16);
        /* endp pointing to NULL means that argv[0] was just a
         * valid number, pass it along to the normal bootm processing
         *
         * If endp is ':' or '#' assume a FIT identifier so pass
         * along for normal processing.
         *
         * Right now we assume the first arg should never be '-'
         */
        if ((*endp != 0) && (*endp != ':') && (*endp != '#'))
            return do_bootm_subcommand(cmdtp, flag, argc, argv);
    }
        // 以上会判断是否有子命令，这里我们不管
        // 到这里，参数中的bootm参数会被去掉，
        // 也就是当'bootm 0x20008000 0x21000000 0x22000000'时
        // argc=3, argv[0]=0x20008000 , argv[1]=0x21000000, argv[2]=0x22000000
        // 当‘bootm 0x30000000’时
        // argc=1, argv[0]=0x30000000

    return do_bootm_states(cmdtp, flag, argc, argv, BOOTM_STATE_START |
        BOOTM_STATE_FINDOS | BOOTM_STATE_FINDOTHER |
        BOOTM_STATE_LOADOS |
        BOOTM_STATE_OS_PREP | BOOTM_STATE_OS_FAKE_GO |
        BOOTM_STATE_OS_GO, &images, 1);
        // 最终对调用到do_bootm_states，在do_bootm_states中执行的操作如states标识所示：
        // BOOTM_STATE_START
        // BOOTM_STATE_FINDOS 
        // BOOTM_STATE_FINDOTHER 
        // BOOTM_STATE_LOADOS 
        // BOOTM_STATE_OS_PREP 
        // BOOTM_STATE_OS_FAKE_GO 
        // BOOTM_STATE_OS_GO
}
```

所以bootm的核心是do_bootm_states，以全局变量bootm_headers_t images作为do_bootm_states的参数。 

# 四. do_bootm_states软件流程

## 1. 数据结构说明

- bootm_headers_t 

  bootm_headers_t用来表示bootm启动kernel的一些信息的结构体，其中包括了os/initrd/fdt images的信息。 bootm会根据参数以及参数指向的镜像来填充这个结构题里面的成员。 最终再使用这个结构体里面的信息来填充kernel启动信息并且到跳转到kernel中。 

在uboot中使用了一个全局的bootm_headers_t images。

```
typedef struct bootm_headers {
    /*
     * Legacy os image header, if it is a multi component image
     * then boot_get_ramdisk() and get_fdt() will attempt to get
     * data from second and third component accordingly.
     */
    image_header_t  *legacy_hdr_os;     /* image header pointer */  // Legacy-uImage的镜像头
    image_header_t  legacy_hdr_os_copy; /* header copy */ // Legacy-uImage的镜像头备份
    ulong       legacy_hdr_valid; // Legacy-uImage的镜像头是否存在的标记

#if IMAGE_ENABLE_FIT
    const char  *fit_uname_cfg; /* configuration node unit name */ // 配置节点名

    void        *fit_hdr_os;    /* os FIT image header */ // FIT-uImage中kernel镜像头
    const char  *fit_uname_os;  /* os subimage node unit name */ // FIT-uImage中kernel的节点名
    int     fit_noffset_os; /* os subimage node offset */ // FIT-uImage中kernel的节点偏移

    void        *fit_hdr_rd;    /* init ramdisk FIT image header */ // FIT-uImage中ramdisk的镜像头
    const char  *fit_uname_rd;  /* init ramdisk subimage node unit name */ // FIT-uImage中ramdisk的节点名
    int     fit_noffset_rd; /* init ramdisk subimage node offset */ // FIT-uImage中ramdisk的节点偏移

    void        *fit_hdr_fdt;   /* FDT blob FIT image header */ // FIT-uImage中FDT的镜像头
    const char  *fit_uname_fdt; /* FDT blob subimage node unit name */ // FIT-uImage中FDT的节点名
    int     fit_noffset_fdt;/* FDT blob subimage node offset */ // FIT-uImage中FDT的节点偏移
#endif

    image_info_t    os;     /* os image info */ // 操作系统信息的结构体
    ulong       ep;     /* entry point of OS */ // 操作系统的入口地址

    ulong       rd_start, rd_end;/* ramdisk start/end */ // ramdisk在内存上的起始地址和结束地址

    char        *ft_addr;   /* flat dev tree address */ // fdt在内存上的地址
    ulong       ft_len;     /* length of flat device tree */ // fdt在内存上的长度

    ulong       initrd_start; // 
    ulong       initrd_end; // 
    ulong       cmdline_start; // 
    ulong       cmdline_end; // 
    bd_t        *kbd; // 

    int     verify;     /* getenv("verify")[0] != 'n' */ // 是否需要验证
    int     state; // 状态标识，用于标识对应的bootm需要做什么操作，具体看下面2.

#ifdef CONFIG_LMB
    struct lmb  lmb;        /* for memory mgmt */
#endif

} bootm_headers_t;
```

## 2. 状态说明

- BOOTM_STATE_START 

  define BOOTM_STATE_START (0x00000001)

  开始执行bootm的一些准备动作。

- BOOTM_STATE_FINDOS 

  define BOOTM_STATE_FINDOS (0x00000002)

  查找操作系统镜像

- BOOTM_STATE_FINDOTHER 

  define BOOTM_STATE_FINDOTHER (0x00000004)

  查找操作系统镜像外的其他镜像，比如FDT\ramdisk等等

- BOOTM_STATE_LOADOS 

  define BOOTM_STATE_LOADOS (0x00000008)

  加载操作系统

- BOOTM_STATE_RAMDISK 

  define BOOTM_STATE_RAMDISK (0x00000010)

  操作ramdisk

- BOOTM_STATE_FDT 

  define BOOTM_STATE_FDT (0x00000020)

  操作FDT

- BOOTM_STATE_OS_CMDLINE 

  define BOOTM_STATE_OS_CMDLINE (0x00000040)

  操作commandline

- BOOTM_STATE_OS_BD_T 

  define BOOTM_STATE_OS_BD_T (0x00000080)

- BOOTM_STATE_OS_PREP 

  define BOOTM_STATE_OS_PREP (0x00000100)

  跳转到操作系统的前的准备动作

- BOOTM_STATE_OS_FAKE_GO 

  define BOOTM_STATE_OS_FAKE_GO (0x00000200) /* ‘Almost’ run the OS */

  伪跳转，一般都能直接跳转到kernel中去

- BOOTM_STATE_OS_GO 

  define BOOTM_STATE_OS_GO (0x00000400)

  跳转到kernel中去

## 3. 软件流程说明

do_bootm_states根据states来判断要执行的操作。

主要流程简单说明如下： 

- bootm的准备动作 

  BOOTM_STATE_START

- 获取kernel信息 

  BOOTM_STATE_FINDOS

- 获取ramdisk和fdt的信息 

  BOOTM_STATE_FINDOTHER

- 加载kernel到对应的位置上（有可能已经就在这个位置上了） 

  BOOTM_STATE_LOADOS

- 重定向ramdisk和fdt（不一定需要） 

  BOOTM_STATE_RAMDISK、BOOTM_STATE_FDT

- 执行跳转前的准备动作 

  BOOTM_STATE_OS_PREP

- 设置启动参数，跳转到kernel所在的地址上 

  BOOTM_STATE_OS_GO

在这些流程中，起传递作用的是bootm_headers_t images这个数据结构，有些流程是解析镜像，往这个结构体里写数据。 
而跳转的时候，则需要使用到这个结构体里面的数据。

软件代码如下 

common/bootm.c

```
int do_bootm_states(cmd_tbl_t *cmdtp, int flag, int argc, char * const argv[],
            int states, bootm_headers_t *images, int boot_progress)
{
    boot_os_fn *boot_fn;
    ulong iflag = 0;
    int ret = 0, need_boot_fn;

    images->state |= states;
       // 把states放到bootm_headers_t images内部

    /*
     * Work through the states and see how far we get. We stop on
     * any error.
     */
        // 判断states是否需要BOOTM_STATE_START动作，也就是bootm的准备动作，需要的话则调用bootm_start
    if (states & BOOTM_STATE_START)
        ret = bootm_start(cmdtp, flag, argc, argv);

        // 判断states是否需要BOOTM_STATE_FINDOS动作，也就是获取kernel信息，需要的话在调用bootm_find_os
    if (!ret && (states & BOOTM_STATE_FINDOS))
        ret = bootm_find_os(cmdtp, flag, argc, argv);

    ·   // 判断states是否需要BOOTM_STATE_FINDOTHER动作，也就是获取ramdisk和fdt等其他镜像的信息，需要的话则调用bootm_find_other
    if (!ret && (states & BOOTM_STATE_FINDOTHER)) {
        ret = bootm_find_other(cmdtp, flag, argc, argv);
        argc = 0;   /* consume the args */
    }

        /* 这里要重点注意，前面的步骤都是在解析uImage镜像并填充bootm_headers_t images */
        /* 也就是说解析uImage的部分在此之前 */
        /* 而后续则是使用bootm_headers_t images 里面的内容来进行后续动作*/



    /* Load the OS */
        // 判断states是否需要BOOTM_STATE_LOADOS动作，也就是加载操作系统的动作，需要的话则调用bootm_load_os
    if (!ret && (states & BOOTM_STATE_LOADOS)) {
        ulong load_end;

        iflag = bootm_disable_interrupts();
        ret = bootm_load_os(images, &load_end, 0);
        if (ret == 0)
            lmb_reserve(&images->lmb, images->os.load,
                    (load_end - images->os.load));
        else if (ret && ret != BOOTM_ERR_OVERLAP)
            goto err;
        else if (ret == BOOTM_ERR_OVERLAP)
            ret = 0;
#if defined(CONFIG_SILENT_CONSOLE) && !defined(CONFIG_SILENT_U_BOOT_ONLY)
        if (images->os.os == IH_OS_LINUX)
            fixup_silent_linux();
#endif
    }

        // 是否需要重定向ramdinsk，do_bootm流程的话是不需要的
    /* Relocate the ramdisk */
#ifdef CONFIG_SYS_BOOT_RAMDISK_HIGH
    if (!ret && (states & BOOTM_STATE_RAMDISK)) {
        ulong rd_len = images->rd_end - images->rd_start;

        ret = boot_ramdisk_high(&images->lmb, images->rd_start,
            rd_len, &images->initrd_start, &images->initrd_end);
        if (!ret) {
            setenv_hex("initrd_start", images->initrd_start);
            setenv_hex("initrd_end", images->initrd_end);
        }
    }
#endif

        // 是否需要重定向fdt，do_bootm流程的话是不需要的
#if IMAGE_ENABLE_OF_LIBFDT && defined(CONFIG_LMB)
    if (!ret && (states & BOOTM_STATE_FDT)) {
        boot_fdt_add_mem_rsv_regions(&images->lmb, images->ft_addr);
        ret = boot_relocate_fdt(&images->lmb, &images->ft_addr,
                    &images->ft_len);
    }
#endif

    /* From now on, we need the OS boot function */
    if (ret)
        return ret;

        // 获取对应操作系统的启动函数，存放到boot_fn中
    boot_fn = bootm_os_get_boot_func(images->os.os);
    need_boot_fn = states & (BOOTM_STATE_OS_CMDLINE |
            BOOTM_STATE_OS_BD_T | BOOTM_STATE_OS_PREP |
            BOOTM_STATE_OS_FAKE_GO | BOOTM_STATE_OS_GO);
    if (boot_fn == NULL && need_boot_fn) {
        if (iflag)
            enable_interrupts();
        printf("ERROR: booting os '%s' (%d) is not supported\n",
               genimg_get_os_name(images->os.os), images->os.os);
        bootstage_error(BOOTSTAGE_ID_CHECK_BOOT_OS);
        return 1;
    }

    /* Call various other states that are not generally used */
    if (!ret && (states & BOOTM_STATE_OS_CMDLINE))
        ret = boot_fn(BOOTM_STATE_OS_CMDLINE, argc, argv, images);
    if (!ret && (states & BOOTM_STATE_OS_BD_T))
        ret = boot_fn(BOOTM_STATE_OS_BD_T, argc, argv, images);

        // 跳转到操作系统前的准备动作，会直接调用启动函数，但是标识是BOOTM_STATE_OS_PREP
    if (!ret && (states & BOOTM_STATE_OS_PREP))
        ret = boot_fn(BOOTM_STATE_OS_PREP, argc, argv, images);


    /* Check for unsupported subcommand. */
    if (ret) {
        puts("subcommand not supported\n");
        return ret;
    }

        // BOOTM_STATE_OS_GO标识，跳转到操作系统中，并且不应该再返回了
    /* Now run the OS! We hope this doesn't return */
    if (!ret && (states & BOOTM_STATE_OS_GO))
        ret = boot_selected_os(argc, argv, BOOTM_STATE_OS_GO,
                images, boot_fn);

    /* Deal with any fallout */
err:
    if (iflag)
        enable_interrupts();

    if (ret == BOOTM_ERR_UNIMPLEMENTED)
        bootstage_error(BOOTSTAGE_ID_DECOMP_UNIMPL);
    else if (ret == BOOTM_ERR_RESET)
        do_reset(cmdtp, flag, argc, argv);

    return ret;
}
```

主要用到如下依次几个函数来实现：

- bootm_start
- bootm_find_os
- bootm_find_other
- bootm_load_os
- bootm_os_get_boot_func
- boot_fn(BOOTM_STATE_OS_PREP, argc, argv, images);
- boot_selected_os
- boot_selected_os(argc, argv, BOOTM_STATE_OS_GO,images, boot_fn);

## 4. bootm_start & bootm_find_os & bootm_find_other

主要负责解析环境变量、参数、uImage，来填充bootm_headers_t images这个数据结构。 最终目的是实现bootm_headers_t images中的这几个成员：

```
typedef struct bootm_headers {
    image_info_t    os;     /* os image info */
    ulong       ep;     /* entry point of OS */

    ulong       rd_start, rd_end;/* ramdisk start/end */

    char        *ft_addr;   /* flat dev tree address */
    ulong       ft_len;     /* length of flat device tree */

    ulong       initrd_start;
    ulong       initrd_end;
    ulong       cmdline_start;
    ulong       cmdline_end;
    bd_t        *kbd;
    int     verify;     /* getenv("verify")[0] != 'n' */
#ifdef CONFIG_LMB
    struct lmb  lmb;        /* for memory mgmt */
#endif
}
```

- bootm_start 

  实现verify和lmb

- bootm_find_os 

  实现os和ep。也就是说，不管是Legacy-uImage还是FIT-uImage，最终解析出来要得到的都是这两个成员。 


- bootm_find_other 

  实现rd_start, rd_end，ft_addr和initrd_end。 也就是说，不管是Legacy-uImage还是FIT-uImage，最终解析出来要得到的都是这两个成员。 

## 5. bootm_load_os

简单说明一下，在bootm_load_os中，会对kernel镜像进行load到对应的位置上，并且如果kernel镜像是被mkimage压缩过的，那么会先经过解压之后再进行load。（这里要注意，这里的压缩和Image压缩成zImage并不是同一个，而是uboot在Image或者zImage的基础上进行的压缩）

```
static int bootm_load_os(bootm_headers_t *images, unsigned long *load_end,
             int boot_progress)
{
    image_info_t os = images->os;
    ulong load = os.load; // kernel要加载的地址
    ulong blob_start = os.start;
    ulong blob_end = os.end;
    ulong image_start = os.image_start; // kernel实际存在的位置
    ulong image_len = os.image_len; // kernel的长度
    bool no_overlap;
    void *load_buf, *image_buf;
    int err;

    load_buf = map_sysmem(load, 0);
    image_buf = map_sysmem(os.image_start, image_len);

// 调用bootm_decomp_image，对image_buf的镜像进行解压缩，并load到load_buf上
    err = bootm_decomp_image(os.comp, load, os.image_start, os.type,
                 load_buf, image_buf, image_len,
                 CONFIG_SYS_BOOTM_LEN, load_end);
```

结果上述步骤之后，kernel镜像就被load到对应位置上了。

## 6. bootm_os_get_boot_func

bootm_os_get_boot_func用于获取到对应操作系统的启动函数，被存储到boot_fn 中。 如下：

```
int do_bootm_states(cmd_tbl_t *cmdtp, int flag, int argc, char * const argv[],
            int states, bootm_headers_t *images, int boot_progress)
{
...
    boot_fn = bootm_os_get_boot_func(images->os.os);
...
}

boot_os_fn *bootm_os_get_boot_func(int os)
{
    return boot_os[os];
// 根据操作系统类型获得到对应的操作函数
}

static boot_os_fn *boot_os[] = {
...
#ifdef CONFIG_BOOTM_LINUX
    [IH_OS_LINUX] = do_bootm_linux,
#endif
}
```

可以看出最终启动linux的核心函数是do_bootm_linux。 另外几个函数最终也是调用到boot_fn，对应linux也就是do_bootm_linux，所以这里不在说明了。 

# 五. do_bootm_linux

arch/arm/lib/bootm.c

```
int do_bootm_linux(int flag, int argc, char * const argv[],
           bootm_headers_t *images)
{
    /* No need for those on ARM */
    if (flag & BOOTM_STATE_OS_BD_T || flag & BOOTM_STATE_OS_CMDLINE)
        return -1;

        // 当flag为BOOTM_STATE_OS_PREP，则说明只需要做准备动作boot_prep_linux
    if (flag & BOOTM_STATE_OS_PREP) {
        boot_prep_linux(images);
        return 0;
    }

        // 当flag为BOOTM_STATE_OS_GO ，则说明只需要做跳转动作 
        if (flag & (BOOTM_STATE_OS_GO | BOOTM_STATE_OS_FAKE_GO)) {
        boot_jump_linux(images, flag);
        return 0;
    }

    boot_prep_linux(images); // 以全局变量bootm_headers_t images为参数传递给boot_prep_linux
    boot_jump_linux(images, flag);// 以全局变量bootm_headers_t images为参数传递给boot_jump_linux
    return 0;
}
```

boot_prep_linux用于实现跳转到linux前的准备动作。 而boot_jump_linux用于跳转到linux中。 都是以全局变量bootm_headers_t images为参数，这样就可以直接获取到前面步骤中得到的kernel镜像、ramdisk以及fdt的信息了。

- boot_prep_linux 

  首先要说明一下LMB的概念。LMB是指logical memory blocks，主要是用于表示内存的保留区域，主要有fdt的区域，ramdisk的区域等等。 

  boot_prep_linux,主要的目的是修正LMB，并把LMB填入到fdt中。 

实现如下：

```
static void boot_prep_linux(bootm_headers_t *images)
{
    char *commandline = getenv("bootargs");

    if (IMAGE_ENABLE_OF_LIBFDT && images->ft_len) {
#ifdef CONFIG_OF_LIBFDT
        debug("using: FDT\n");
        if (image_setup_linux(images)) {
            printf("FDT creation failed! hanging...");
            hang();
        }
#endif
    }
```

这里没有深入学习image_setup_linux，等后续有需要的话再进行深入。

- boot_jump_linux 
  以arm为例： 
  arch/arm/lib/bootm.c

```
static void boot_jump_linux(bootm_headers_t *images, int flag)
{
    unsigned long machid = gd->bd->bi_arch_number; // 从bd中获取machine-id，machine-id在uboot启动流程的文章中有说明过了
    char *s;
    void (*kernel_entry)(int zero, int arch, uint params); // kernel入口函数，也就是kernel的入口地址，对应kernel的_start地址。
    unsigned long r2;
    int fake = (flag & BOOTM_STATE_OS_FAKE_GO); // 伪跳转，并不真正地跳转到kernel中

    kernel_entry = (void (*)(int, int, uint))images->ep; 
         // 将kernel_entry设置为images中的ep（kernel的入口地址），后面直接执行kernel_entry也就跳转到了kernel中了
        // 这里要注意这种跳转的方法

    debug("## Transferring control to Linux (at address %08lx)" \
        "...\n", (ulong) kernel_entry);
    bootstage_mark(BOOTSTAGE_ID_RUN_OS);
    announce_and_cleanup(fake);

        // 把images->ft_addr（fdt的地址）放在r2寄存器中
    if (IMAGE_ENABLE_OF_LIBFDT && images->ft_len)
        r2 = (unsigned long)images->ft_addr;
    else
        r2 = gd->bd->bi_boot_params;

    if (!fake) {
            kernel_entry(0, machid, r2);
        // 这里通过调用kernel_entry，就跳转到了images->ep中了，也就是跳转到kernel中了，具体则是kernel的_start符号的地址。
        // 参数0则传入到r0寄存器中，参数machid传入到r1寄存器中，把images->ft_addr（fdt的地址）放在r2寄存器中
        // 满足了kernel启动的硬件要求
    }
}
```
