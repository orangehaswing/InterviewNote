# uboot流程(一)-概述

# 一、bootloader& uboot

## 1、bootloader的概念

​	Bootloader是在操作系统运行之前执行的一段小程序。而这段小程序的最终目的，正确地设置好软硬件环境，使之能够成功地引导操作系统。

## 2、bootloader的核心功能

bootloader的核心功能就是引导操作系统，部分工作如下

- 初始化部分硬件，包括时钟、内存等等
- 加载内核到内存上
- 加载文件系统、atags或者dtb到内存上
- 根据操作系统启动要求正确配置好一些硬件
- 启动操作系统

## 3、bootloader的monitor功能

bootloader的核心功能，也就是引导操作系统的功能。 但是部分bootloader还支持monitor功能，提供了更多的命令行接口，具体部分功能如下：

- 进行调试
- 读写内存
- 烧写Flash
- 配置环境变量
- 命令引导操作系统

# 二、uboot-spl & uboot

## 1、uboot-spl

由uboot编译生成，对应于BL1阶段，也就是BL1的镜像，uboot-spl.bin。 根据《[project X] tiny210(s5pv210)上电启动流程（BL0-BL2）》，其代码运行于IRAM中

主要工作有： 

- 初始化部分时钟（和SDRAM相关）
- 初始化DDR（外部SDRAM）
- 从存储介质上（比如SD\eMMC\nand flash）将BL2镜像加载到SDRAM上
- 验证BL2镜像的合法性
- 跳转到BL2镜像所在的地址上

## 2、uboot

由uboot编译生成，对应于BL2阶段，也就是BL2的镜像,uboot.bin。 根据《[project X] tiny210(s5pv210)上电启动流程（BL0-BL2）》，其代码运行于SDRAM中.

主要工作有： 

- 初始化部分硬件，包括时钟、内存等等
- 加载内核到内存上
- 加载文件系统、atags或者dtb到内存上
- 根据操作系统启动要求正确配置好一些硬件
- 启动操作系统

monitor工作，主要是处理命令行的命令，以下是部分操作： 

- flash操作
- 环境变量操作
- 启动操作

