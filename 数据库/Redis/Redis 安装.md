# Redis 安装

## Window 下安装

**下载地址：**[https://github.com/MSOpenTech/redis/releases](https://github.com/MSOpenTech/redis/releases)。

Redis 支持 32 位和 64 位。这个需要根据你系统平台的实际情况选择，这里我们下载 **Redis-x64-xxx.zip**压缩包到 C 盘，解压后，将文件夹重新命名为 **redis**。

![img](http://www.runoob.com/wp-content/uploads/2014/11/3B8D633F-14CE-42E3-B174-FCCD48B11FF3.jpg)

打开文件夹，内容如下：

![img](http://www.runoob.com/wp-content/uploads/2014/11/C2CEBAA0-30B9-4340-8D23-78F6FEB8CBE2.png%22)

打开一个 **cmd** 窗口 使用 cd 命令切换目录到 **C:\redis** 运行：

```
redis-server.exe redis.windows.conf
```

如果想方便的话，可以把 redis 的路径加到系统的环境变量里，这样就省得再输路径了，后面的那个 redis.windows.conf 可以省略，如果省略，会启用默认的。输入之后，会显示如下界面：

这时候另启一个 cmd 窗口，原来的不要关闭，不然就无法访问服务端了。

切换到 redis 目录下运行:

```
redis-cli.exe -h 127.0.0.1 -p 6379
```

设置键值对:

```
set myKey abc
```

取出键值对:

```
get myKey
```

![Redis 安装](http://www.runoob.com/wp-content/uploads/2014/11/redis-install2.jpg)



