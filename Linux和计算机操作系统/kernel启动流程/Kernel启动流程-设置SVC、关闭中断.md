# Kernel启动流程-设置SVC、关闭中断

## 零、说明

### 1、kernel启动流程第一阶段简单说明

arch/arm/kernel/head.S

- kernel入口地址对应stext

```
ENTRY(stext)
```

第一阶段要做的事情，也就是stext的实现内容

设置为SVC模式，关闭所有中断
获取CPU ID，提取相应的proc info
验证tags或者dtb
创建页表项
配置r13寄存器，也就是设置打开MMU之后要跳转到的函数。
使能MMU

跳转到start_kernel，也就是跳转到第二阶段













