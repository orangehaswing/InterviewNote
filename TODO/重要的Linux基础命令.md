# 重要的Linux基础命令

## 1 必不可少的基础命令和工具

### 1.1 grep

grep是Linux下通用的文本内容查找命令。

也可以利用它打印匹配的上下几行，线上查找问题的时候，可以使用下列命令，查找关键字，显示关键字出现行的后5行，并且给关键字着色。

使用方式：

```
grep -5 'parttern' INPUT_FILE #打印匹配行的前后5行
 
grep -C 5 'parttern' INPUT_FILE #打印匹配行的前后5行
 
grep -A 5 'parttern' INPUT_FILE #打印匹配行的后5行
 
$grep -B 5 'parttern' INPUT_FILE #打印匹配行的前5行

grep -A -15  --color 1010061938 * #查找后着色

```

### 1.2 find

通过文件名称查找文件的存在位置，名称查找支持模糊匹配。

使用方式：

```
find . -name FILE_NAME

```

命令输出：

```
robert@robert-ubuntu1410：~$ find . -name VestaServer.java
./working/workspace/vesta-id-generator/vesta-server/src/main/java/com/robert/vesta/server/VestaServer.java

```

### 1.3 uptime

查看机器的启动时间、登录用户、平均负载等情况，通常用在线上应急或者技术攻关的时候来确定操作系统的重启时间。

使用方式：

```
uptime

```

命令输出：

```
robert@robert-ubuntu1410：~$ uptime
 14：42：30 up  2：51,  3 users,  load average：0.03, 0.06, 0.06

```

从上面输出可以看到如下信息：

> 1. 当前时间：14：42：30

1. 系统已运行的时间：2小时51分
2. 当前在线用户：3个用户
3. 系统平均负载：0.03, 0.06, 0.06，最近1分钟、5分钟、15分钟系统的负载情况

系统平均负载指在特定时间间隔内队列中运行的平均进程数。如果一个进程满足以下条件，它其就会位于运行队列中：

> 1. 它没有在等待IO操作的结果

1. 它没有主动进入等待状态(也就是没有调用'wait'相关的系统API)
2. 没有被停止(例如：等待终止)

一般来说，每个CPU内核对应活动进程数不大于3，则系统运行良好，换句话说，也就是活动进程数小于CPU核数的3倍。

举例说明，如果你的服务器的cpu有3个核心，那么只要uptime最后输出的一串字符数值小于9，即表示系统负载正常。但是，如果系统负载超过10，那就表示当前系统负载过重，需要定位系统执行任务负载超标的原因。

### 1.4 lsof

列出系统当前打开的文件句柄，在Linux文件系统中，任何资源都是以文件句柄的形式管理的，例如：硬件设备、文件、网络套接字等，系统内部为每一种资源分配一个句柄，应用程序只能用操作系统分配的句柄来引用资源，因此，文件句柄为应用程序与基础操作系统之间的交互提供了通用的操作接口。

应用程序打开文件的描述符列表包含了大量的关于应用程序本身的运行信息，因此通过lsof工具查看这个文件句柄列表，对系统监控以及应急排错提供重要的帮助。

查看某一个进程打开的文件句柄：

```
lsof -p 2862

```

命令输出：

```
robert@robert-ubuntu1410：~$ lsof -p 2862 | less
COMMAND  PID   USER   FD   TYPE             DEVICE SIZE/OFF    NODE NAME
java    2862 robert  cwd    DIR                8,1     4096  537041 /home/robert/working/workspace/vesta-id-generator/releases/vesta-id-generator-0.0.1-release/bin/vesta-rest-0.0.1
java    2862 robert  rtd    DIR                8,1     4096       2 /
java    2862 robert  txt    REG                8,1     5730 1064639 /home/robert/working/softwares/jdk1.8.0_20/bin/java
java    2862 robert  mem    REG                8,1  7216688 1318996 /usr/lib/locale/locale-archive
java    2862 robert  mem    REG                8,1 65525265 1189622 /home/robert/working/softwares/jdk1.8.0_20/jre/lib/rt.jar
java    2862 robert  mem    REG                8,1    80460 1189581 /home/robert/working/softwares/jdk1.8.0_20/jre/lib/i386/libnio.so
java    2862 robert  mem    REG                8,1   103299 1189580 /home/robert/working/softwares/jdk1.8.0_20/jre/lib/i386/libnet.so
java    2862 robert  mem    REG                8,1    81884 1583248 /usr/share/locale-langpack/zh_CN/LC_MESSAGES/libc.mo
java    2862 robert  mem    REG                8,1  3131363 1189479 /home/robert/working/softwares/jdk1.8.0_20/jre/lib/charsets.jar
java    2862 robert  mem    REG                8,1  3500527 1189621 /home/robert/working/softwares/jdk1.8.0_20/jre/lib/resources.jar
java    2862 robert  mem    REG                8,1  1179307 1330505 /home/robert/working/softwares/jdk1.8.0_20/jre/lib/ext/localedata.jar
java    2862 robert  mem    REG                8,1   615948 1189601 /home/robert/working/softwares/jdk1.8.0_20/jre/lib/jsse.jar
java    2862 robert  mem    REG                8,1  3860522 1330502 /home/robert/working/softwares/jdk1.8.0_20/jre/lib/ext/cldrdata.jar
java    2862 robert  mem    REG                8,1  1065895 1330501 /home/robert/working/softwares/jdk1.8.0_20/jre/lib/ext/bcprov-jdk15-132.jar
......

```

查看某一个端口的使用方式：

```
lsof -i :8080

```

命令输出：

```
robert@robert-ubuntu1410：~$ lsof -i ：8080
COMMAND  PID   USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
java    2862 robert   19u  IPv6  21370      0t0  TCP *：http-alt (LISTEN)

```

### 1.5 ulimit

Linux系统对每个登录用户，都限制其最大进程数和打开的最大文件句柄数。为提高性能，可以根据硬件资源的具体情况，设置各个用户的最大进程数和打开的最大文件句柄数。

可以用ulimit -a来显示当前的各种系统对用户使用资源的限制：

```
robert@robert-ubuntu1410：~$ ulimit -a
core file size          (blocks, -c) 0
data seg size           (kbytes, -d) unlimited
scheduling priority             (-e) 0
file size               (blocks, -f) unlimited
pending signals                 (-i) 7921
max locked memory       (kbytes, -l) 64
max memory size         (kbytes, -m) unlimited
open files                      (-n) 1024
pipe size            (512 bytes, -p) 8
POSIX message queues     (bytes, -q) 819200
real-time priority              (-r) 0
stack size              (kbytes, -s) 8192
cpu time               (seconds, -t) unlimited
max user processes              (-u) 7921
virtual memory          (kbytes, -v) unlimited
file locks                      (-x) unlimited

```

设置用户的最大进程数：

```
ulimit -u 10240

```

设置用户可以打开的最大文件句柄数：

```
ulimit -n xx

```

### 1.6 curl

程序开发后，会使用junit, testng以及jmock, mockito进行单元测试，单元测试后需要进行集成测试，由于当前的线上服务较多使用restful风格，那么集成测试的时候就需要进行HTTP调用，查看返回的结果是否符合预期，curl命令当然是首选测试的方法。

使用方式：

```
curl -i  “http://www.sina.com” #打印请求响应头信息

curl -v  “http://www.sina.com” #使用post方法
curl -verbose  “http://www.sina.com” #使用post方法

curl -d ‘abc=def’ “http://www.sina.com” #使用head方法

curl -I "http://www.sina.com" #打印HTTP响应码

curl -sw '%{http_code}'  "http://www.sina.com" #打印HTTP响应码

```

### 1.7 scp

scp命令是Linux系统中功能强大的文件传输命令，可以实现从本地到远程以及远程到本地的双向文件传输，用起来非常的方便。常用来在线上定位问题时，将线上的一些文件下载到本地进行详查，或者将本地的修改上传到服务器上。

使用方式：

```
scp robert@192.168.1.1:/home/robert/test.txt .

scp ./test.txt robert@192.168.1.1:/home/robert/

```

### 1.8 vi & vim

vi和vim是Linux中最常用的命令行文本编辑工具，vim是vi的升级版本，在某些Linux版本下，vi实际上通过软连接指向vim。

笔者常用的vi/vim命令如下：

> 1. h：左移一个字符

1. l：右移一个字符，这个命令很少用，一般用w代替
2. k：上移一个字符
3. j：下移一个字符
4. set number：显示行号
5. shift + g：移动到最后一行
6. 1 + shift + g：移动到第一行
7. n + shift + g：移动到第n行
8. 0： 移动到行首
9. $：移动到行尾
10. /text：查找text，按n健查找下一个，按N健查找前一个
11. ?text：查找text，反向查找，按n健查找下一个，按N健查找前一个
12. i：在当前位置生前插入
13. I：在当前行首插入
14. a：在当前位置后插入
15. A：在当前行尾插入
16. o：在当前行之后插入一行
17. O：在当前行之前插入一行
18. %s/old/new/g：用old替换new，替换当前行的所有匹配
19. ctrl + f：向下滚动一屏
20. ctrl + b：向上滚动一屏
21. u：撤销
22. U：撤销对整行的操作
23. Ctrl + r：重做，即撤销的撤销
24. x：删除当前字符
25. dd：删除当前行
26. 10d：删除当前行开始的10行
27. yy：拷贝当前行
28. p：在当前光标后粘贴,如果之前使用了yy命令来复制一行，那么就在当前行的下一行粘贴
29. ：wq：保存并退出
30. ：q!：强制退出并忽略所有更改

有了这些命令，基本可以在Linux系统命令行下做开发了，无论是开发脚本，还是线上应急或者技术攻关过程中在Linux系统中做编辑，都没有问题，建议大家把上面这个命令列表打印出来，贴在办公桌上，需要的时候可以看一眼，久而久之就记住了。

### 1.9 dos2unix & unix2dos

用于转换windows和unix的换行符，通常在windows上开发的脚本和配置，上传到unix上都需要转换。

使用方式：

```
robert@robert-ubuntu1410：~$ dos2unix test.txt 
dos2unix：converting file test.txt to Unix format ...

robert@robert-ubuntu1410：~$ unix2dos test.txt 
unix2dos：converting file test.txt to DOS format ...

```

## 2 查看活动进程的命令

### 2.1 ps

显示系统内所有的进程。

使用方式：

```
ps -elf

```

输出：

```
robert@robert-ubuntu1410：~$ ps -elf
F S UID        PID  PPID  C PRI  NI ADDR SZ WCHAN  STIME TTY          TIME CMD
4 S root         1     0  0  80   0 -  8477 poll_s 09：56 ?        00：00：01 /sbin/init
1 S root         2     0  0  80   0 -     0 kthrea 09：56 ?        00：00：00 [kthreadd]
1 S root         3     2  0  80   0 -     0 smpboo 09：56 ?        00：00：00 [ksoftirqd/0]
1 S root         4     2  0  80   0 -     0 worker 09：56 ?        00：00：00 [kworker/0：0]
1 S root         5     2  0  60 -20 -     0 worker 09：56 ?        00：00：00 [kworker/0：0H]
1 S root         7     2  0  80   0 -     0 rcu_gp 09：56 ?        00：00：00 [rcu_sched]
1 S root         8     2  0  80   0 -     0 rcu_no 09：56 ?        00：00：00 [rcuos/0]
1 R root         9     2  0  80   0 -     0 ?      09：56 ?        00：00：00 [rcuos/1]
1 S root        10     2  0  80   0 -     0 rcu_no 09：56 ?        00：00：00 [rcuos/2]
1 S root        11     2  0  80   0 -     0 rcu_no 09：56 ?        00：00：00 [rcuos/3]
1 S root        12     2  0  80   0 -     0 rcu_gp 09：56 ?        00：00：00 [rcu_bh]
1 S root        13     2  0  80   0 -     0 rcu_no 09：56 ?        00：00：00 [rcuob/0]
1 S root        14     2  0  80   0 -     0 rcu_no 09：56 ?        00：00：00 [rcuob/1]
1 S root        15     2  0  80   0 -     0 rcu_no 09：56 ?        00：00：00 [rcuob/2]
1 S root        16     2  0  80   0 -     0 rcu_no 09：56 ?        00：00：00 [rcuob/3]
......

```

根据进程的名字或者其他信息，通过grep命令找到目标进程，也可以打印进程启动脚本的全路径。

### 2.2 top

查看活动进程的CPU和内存信息的工具命令，能够实时显示系统中各个进程的资源占用情况。可以按CPU和内存的使用情况和执行时间对进程进行排序。

使用方式：

```
top

```

命令输出：

```
    top - 10：18：49 up 22 min,  2 users,  load average：0.10, 0.31, 0.22
    Tasks：195 total,   2 running, 193 sleeping,   0 stopped,   0 zombie
    %Cpu(s)： 1.8 us,  0.2 sy,  0.0 ni, 98.0 id,  0.1 wa,  0.0 hi,  0.0 si,  0.0 st
    KiB Mem：  2049416 total,  1636620 used,   412796 free,   117652 buffers
    KiB Swap： 2095100 total,     1480 used,  2093620 free.   643848 cached Mem
    
      PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND                                                           
     1608 root      20   0  475836  74616  30232 S   4.0  3.6   0：12.21 Xorg                                                              
     2363 robert    20   0 1380660 103000  63884 S   2.7  5.0   0：12.15 compiz                                                            
     2157 robert    20   0  589920  30748  24412 S   1.3  1.5   0：00.98 unity-panel-ser                                                   
     2769 robert    20   0  597884  35820  28008 S   0.7  1.7   0：04.95 gnome-terminal                                                    
     ...

```

从输出中可以看到整体的cpu占用率、cpu负载的情况、进程占用CPU和内存等资源的情况。另外top命令的输出把cache放在了swap一行，这并不重要，实际上它和swap没有太大的关系。

我们可以用top命令的快捷键转换输出的显示信息：

> 1. t：切换显示进程和 CPU 状态信息

1. m：切换显示进程和 CPU 状态信息
2. A：分类显示各种系统资源的消耗情况。可用于快速识别系统的性能要求极高的任务
3. o：改变显示项目的顺序
4. r：重新设置进程的优先级别(系统提示用户输入需要改变的进程PID以及需要设置
   的优先级值)
5. k：终止一个进程(系统将提示用户输入需要终止的进程PID)
6. s：改变刷新的时间间隔
7. u：查看指定用户的进程

## 3 窥探内存的命令

### 3.1 free

此命令显示系统内存的使用情况，包括总体内存、已经使用的内存、以及系统核心使用的缓冲区，包括缓存(buffer)和缓冲(cache)等。

使用方式：

```
free

```

命令输出：

```
robert@robert-ubuntu1410：~$ free
             total       used       free     shared    buffers     cached
Mem：      2049416    1646480     402936      13280     118596     646288
-/+ buffers/cache：    881596    1167820
Swap：     2095100       1480    2093620

```

内存使用并不只有简单的占用和空闲两个状态，从上面的输出发现里面有buffer和cache的数据，从字面意义上来讲，都是缓存，那么弄清楚缓存什么数据才能有效的区分这两种缓存。

从上面命令的输出，我们可以看到，Buffer 118M, Cache 646M。其实，这两个内存区域都是用来缓存磁盘数据的，只不过缓存的数据是不同的：

1. buffers一般都不太大，在一个通用的Linux系统中，一般都是在几十到几百M字节，用于存储磁盘块设备的元数据，比如哪些块属于哪些文件，文件的权限，目录等信息。
2. cached一般会很大, 一般都是G字节以上, 用于存储读写文件的页, 当对一个文件进行读的时候, 会取磁盘文件页到此内存区域，然后从内存进行读取，当写入一个文件，会先写到此缓存，并将相关的页面标记为”dirty”。

buffers用于存储元数据，一般占用的空间不大，对它的关注也不多，cached一般会很大，随着读写磁盘的多少而自动的增加而减少，这也取决于物理内存是否够用，如果应用使用物理内存较多，操作系统会适当的缩小cached来保证用户进程对内存的需要。

### 3.2 pmap

此命令用来报告进程占用内存的详细情况，可以用来查出某些内存瓶颈问题的根源原因。

使用方式：

```
pmap -d 2862

```

命令输出：

```
robert@robert-ubuntu1410：~$ pmap -d 2862
2862：  java -server -Xms512m -Xmx512m -Xmn128m -XX：PermSize=128m -Xss256k -XX：+DisableExplicitGC -XX：+UseConcMarkSweepGC -XX：+CMSParallelRemarkEnabled -XX：+UseCMSCompactAtFullCollection -XX：+UseCMSInitiatingOccupancyOnly -XX：CMSInitiatingOccupancyFraction=60 -verbose：gc -XX：+PrintGCDateStamps -XX：+PrintTenuringDistribution -XX：+PrintGCDetails -Xloggc：./logs/gc.log -cp /home/robert/working/workspace/vesta-id-generator/releases/vesta-id-generator-0.0.1-release/bin/vesta-rest-0.0.1/extlib -jar ./lib/vesta-rest-0.0.
Address           Kbytes Mode  Offset           Device    Mapping
0000000008048000       4 r-x-- 0000000000000000 008：00001 java
0000000008049000       4 rw--- 0000000000000000 008：00001 java
000000000a017000     872 rw--- 0000000000000000 000：00000   [ anon ]
00000000be800000     896 rw--- 0000000000000000 000：00000   [ anon ]
00000000be8e0000     128 ----- 0000000000000000 000：00000   [ anon ]
00000000be900000    1920 rw--- 0000000000000000 000：00000   [ anon ]
00000000beae0000     128 ----- 0000000000000000 000：00000   [ anon ]
00000000beb00000     284 rw--- 0000000000000000 000：00000   [ anon ]
......

```

这个命令显示比较底层的进程模块占用内存的信息，并且可以打印内存的起止地址等，用于定位深层次JVM或者操作系统的内存问题。

## 4 CPU使用情况监控命令

### 4.1 vmstat

此命令显示关于内核线程、虚拟内存、磁盘IO、陷阱和CPU占用率的统计信息。

使用方式：

```
vmstat

```

命令输出：

```
robert@robert-ubuntu1410：~$ vmstat
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
 2  0   1480 404300 118252 646216    0    0    78    31   63  145  2  0 97  1  0

```

这里面需要注意的是：

> 1. buff是IO系统用来做存储磁盘块文件元数据的信息

1. cache是操作系统用来缓存磁盘数据的缓冲区，操作系统会自动调节这个参数，在内存紧张的时候操作系统会减少cache的占用空间来保证其他进程可用
2. cs参数表达线程切换次数，此数据太大表明线程同步机制有问题
3. si和so如果较大说明系统频繁使用交换区，应该查看操作系统内存是否够用
4. bi和bo代表IO活动，根据大小可以知道磁盘的负载

### 4.2 mpstat

实时监控系统CPU的一些统计信息，这些信息存放在/proc/stat文件中，在多核CPU系统里，其不但能查看所有CPU的平均使用信息，而且能够查看某一个特定CPU的信息。

使用方式：

```
mpstat -P ALL

```

命令输出：

```
robert@robert-ubuntu1410：~$ mpstat -P ALL
Linux 3.16.0-30-generic (robert-ubuntu1410)     2017年04月23日     _x86_64_    (4 CPU)

11时12分38秒  CPU    %usr   %nice    %sys %iowait    %irq   %soft  %steal  %guest  %gnice   %idle
11时12分38秒  all    0.54    0.55    0.23    0.57    0.00    0.04    0.00    0.00    0.00   98.07
11时12分38秒    0    0.75    1.41    0.35    1.06    0.00    0.11    0.00    0.00    0.00   96.32
11时12分38秒    1    0.51    0.22    0.22    0.48    0.00    0.01    0.00    0.00    0.00   98.57
11时12分38秒    2    0.52    0.58    0.18    0.29    0.00    0.02    0.00    0.00    0.00   98.41
11时12分38秒    3    0.40    0.01    0.15    0.45    0.00    0.01    0.00    0.00    0.00   98.98

```

我们可以看到每个CPU核心的占用率、IO等待、软中断、硬中断等。

## 5 磁盘IO监控命令

### 5.1 iostat

监视CPU占用率和平均负载值，以及IO读写速度等。

这个命令的输出的每个字段都非常有用，r/s和w/s指的是IOPS，rkB/s和wkB/s指的是每秒的存取速度，await是平均等待时间，一般都在10ms左右。

另外，iotop、ioprofiler、blktrace可以监控更多底层的IO活动信息，本文就不展开介绍，vmstat、mpstat也有一些IO相关的信息输出。

使用方式：

```
iostat -x

```

命令输出：

```
robert@robert-ubuntu1410：~$ iostat -x
Linux 3.16.0-30-generic (robert-ubuntu1410)     2017年04月23日     _x86_64_    (4 CPU)

avg-cpu： %user   %nice %system %iowait  %steal   %idle
           0.61    0.68    0.31    0.70    0.00   97.72

Device：        rrqm/s   wrqm/s     r/s     w/s    rkB/s    wkB/s avgrq-sz avgqu-sz   await r_await w_await  svctm  %util
sda               2.69     3.43   11.92    1.56   217.23   118.91    49.87     0.21   15.45    9.39   61.69   2.18   2.94

```

从命令输出可以看出：

> 1. iowait较大说明IO负载大，IO等待比较严重，磁盘读写遇到瓶颈。

1. 可以直接看到每秒读写速度的最大峰值。
2. 也可以看到CPU的占用率情况。

### 5.2 swapon

查看交换分区的使用情况。

使用方式：

```
/sbin/swapon -s

```

命令输出：

```
robert@robert-Latitude-E6440：~/tmp$ /sbin/swapon -s
Filename                Type        Size    Used    Priority
/dev/sda6              partition   4094972 708384   -1

```

由输出可见交换分区共4G，已使用大约708M。

### 5.3 df

查看文件系统的硬盘挂载点和空间使用情况。

使用方式：

```
df

```

命令输出：

```
robert@robert-Latitude-E6440：~/tmp$ df -h
文件系统        容量  已用  可用 已用% 挂载点
/dev/sda5       220G   84G  125G   40% /
none            4.0K     0  4.0K    0% /sys/fs/cgroup
udev            2.0G  4.0K  2.0G    1% /dev
tmpfs           395M  1.3M  393M    1% /run
none            5.0M     0  5.0M    0% /run/lock
none            2.0G   52M  1.9G    3% /run/shm
none            100M   60K  100M    1% /run/user

```

## 6 查看网络信息和网络监控命令

## 6.1 ifconfig

ifconfig可以查看机器挂载的网卡情况。

使用方式：

```
ifconfig -a

```

命令输出：

```
robert@robert-ubuntu1410：~$ ifconfig -a
eth0      Link encap：以太网  硬件地址 08：00：27：2f：70：b6  
          inet 地址：192.168.1.102  广播：192.168.1.255  掩码：255.255.255.0
          inet6 地址：fe80：：a00：27ff：fe2f：70b6/64 Scope：Link
          UP BROADCAST RUNNING MULTICAST  MTU：1500  跃点数：1
          接收数据包：14392 错误：0 丢弃：0 过载：0 帧数：0
          发送数据包：8665 错误：0 丢弃：0 过载：0 载波：0
          碰撞：0 发送队列长度：1000 
          接收字节：15021524 (15.0 MB)  发送字节：858553 (858.5 KB)

lo        Link encap：本地环回  
          inet 地址：127.0.0.1  掩码：255.0.0.0
          inet6 地址：：：1/128 Scope：Host
          UP LOOPBACK RUNNING  MTU：65536  跃点数：1
          接收数据包：4161 错误：0 丢弃：0 过载：0 帧数：0
          发送数据包：4161 错误：0 丢弃：0 过载：0 载波：0
          碰撞：0 发送队列长度：0 
          接收字节：331544 (331.5 KB)  发送字节：331544 (331.5 KB)

```

可见机器有两个网卡，一个是eth0，一个是本地回环虚拟网卡。

## 6.2 ping

ping命令是检测网络故障常用的命令，可以用来测试一台主机到另外一台主机的网络是否联通。

使用方式：

```
ping www.baidu.com

```

命令输出：

```
robert@robert-ubuntu1410：~$ ping www.baidu.com
PING www.a.shifen.com (111.13.100.92) 56(84) bytes of data.
64 bytes from localhost (111.13.100.92)：icmp_seq=1 ttl=54 time=4.91 ms
64 bytes from localhost (111.13.100.92)：icmp_seq=2 ttl=54 time=8.76 ms
^C
--- www.a.shifen.com ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1001ms
rtt min/avg/max/mdev = 4.917/6.838/8.760/1.923 ms

```

## 6.3 telnet

telnet是TCP/IP协议族的一员，是网络远程登陆服务的标准协议，它为用户提供了在本地计算机上连接远程主机的能力和主要方式。

使用方式：

```
telnet IP PORT

```

命令输出：

```
robert at robert in ~/working/softwares/redis-3.0.5/src 
$ telnet localhost 6379
Trying ：：1...
Connected to localhost.
Escape character is '^]'.
get hello
$3
world

```

从上面输出可以看到，使用telnet协议可以直接连接redis端口，并发送redis命令。

## 6.4 nc

nc是NetCat的简称，在网络调试工具享有“瑞士军刀”的美誉，此命令功能丰富、短小精悍、简单实用，被设计成为一个易用的网络工具，可通过TCP/UDP协议传输数据。同时，它也是一个网络应用调试分析器，因为它可以根据需要创建各种不同类型的网络服务和连接，在调试Restful服务的时候，经常会发生不可预期的结果，这种情况下可以使用nc模拟启动服务器，把HTTP客户端连接到nc上，nc上会打印出Restful服务提供的所有参数，然后一一检查参数，找到问题。

当然，也可用于传输二进制或者文本文件。

传输文件端：

```
robert@robert-ubuntu1410：~$ nc localhost 8888 < test.txt

```

接受文件端：

```
robert@robert-ubuntu1410：~$ nc -l 8888
12345678

```

## 6.5 mtr

Linux系统中的网络连通性测试工具，也可以用来检测丢包率。

使用方式：

```
mtr -r sina.com

```

命令输出：

```
robert@robert-ubuntu1410：~$ mtr -r sina.com
Start：Sun Apr 23 16：40：27 2017
HOST：robert-ubuntu1410           Loss%   Snt   Last   Avg  Best  Wrst StDev
  1.|-- 192.168.1.1                0.0%    10    2.0   2.5   0.9  10.4   2.7
  2.|-- 172.30.44.1                0.0%    10    6.4   7.5   5.8  13.8   2.3
  3.|-- 10.1.10.201                0.0%    10    3.0   3.4   3.0   4.2   0.0
  4.|-- 111.63.14.97               0.0%    10    5.5   6.6   5.1  16.4   3.4
  5.|-- 111.11.74.9               90.0%    10   10.8  10.8  10.8  10.8   0.0
  6.|-- 111.11.65.117             90.0%    10    7.9   7.9   7.9   7.9   0.0
  7.|-- 221.183.26.205            80.0%    10    8.0   9.1   8.0  10.1   1.4
  8.|-- 221.176.16.250            80.0%    10   11.9  12.8  11.9  13.8   1.0
  9.|-- 221.176.21.194            90.0%    10   11.6  11.6  11.6  11.6   0.0
 10.|-- 202.97.15.177             90.0%    10   25.1  25.1  25.1  25.1   0.0
 11.|-- 202.97.88.237             90.0%    10   14.1  14.1  14.1  14.1   0.0
 12.|-- 202.97.53.110              0.0%    10   20.4  16.0  13.7  20.4   2.1
 13.|-- 202.97.58.114              0.0%    10   14.4  17.9  14.4  21.4   2.4
 14.|-- 202.97.51.86              40.0%    10  211.2 207.4 204.9 211.2   2.5
 15.|-- 203.14.186.34              0.0%    10  224.7 201.3 194.9 224.7  10.3
 16.|-- 218.30.41.234              0.0%     9  218.1 219.6 215.3 238.7   7.3
 17.|-- ???                       100.0     9    0.0   0.0   0.0   0.0   0.0

```

其中第二列为丢包率，可以用来判断网络中两台机器连通性的质量。

## 6.6 nslookup

是一款检测网络中DNS服务器的是否能够正确解析域名的工具命令，并且可以输出。

使用方式：

```
nslookup sina.com

```

命令输出：

```
robert@robert-ubuntu1410：~$ nslookup sina.com
Server：     127.0.1.1
Address：    127.0.1.1#53

Non-authoritative answer：
Name：   sina.com
Address：66.102.251.33

```

从输出可以看到，sina.com域名被正确解析到IP地址66.102.251.33。

## 6.7 traceroute

traceroute可以提供从你的主机到互联网另一端的主机走的什么路径，然而，每次数据包由某一同样的出发点到达某一同样的目的地走的路径可能会不一样，但通常来说大部分时候所走的路径是相同的。

使用方式：

```
traceroute sina.com 

```

命令输出：

```
robert@robert-ubuntu1410：~$ traceroute sina.com 
traceroute to sina.com (66.102.251.33), 30 hops max, 60 byte packets
 1  192.168.1.1 (192.168.1.1)  4.373 ms  4.351 ms  4.337 ms
 2  172.30.44.1 (172.30.44.1)  9.573 ms  10.107 ms  10.422 ms
 3  10.1.1.2 (10.1.1.2)  4.696 ms  4.473 ms  4.637 ms
 4  111.63.14.97 (111.63.14.97)  6.118 ms  6.929 ms  6.904 ms
 5  * * *
 6  * * *
 7  * * *
 8  * * *
 9  * * *
10  * * 221.176.23.54 (221.176.23.54)  22.312 ms
11  * * *
12  202.97.53.86 (202.97.53.86)  17.421 ms 202.97.53.34 (202.97.53.34)  29.006 ms 202.97.53.114 (202.97.53.114)  15.464 ms
13  202.97.58.114 (202.97.58.114)  17.840 ms 202.97.58.122 (202.97.58.122)  16.655 ms  20.011 ms
14  202.97.51.86 (202.97.51.86)  207.216 ms  207.157 ms  211.004 ms
15  203.14.186.34 (203.14.186.34)  199.606 ms  196.477 ms  195.614 ms
16  218.30.41.234 (218.30.41.234)  215.134 ms  214.705 ms  220.728 ms
17  66.102.251.33 (66.102.251.33)  209.436 ms  210.263 ms  208.335 ms

```

上面输出中记录按序列号从1开始，每个纪录就是网络一跳，每跳一次表示经过一个网关或者路由，我们看到每行有三个时间，单位是毫秒，指的是这一跳每次需要的时间。

### 6.8 sar

sar是一个多功能的监控工具，使用简单方便，不需要管理员权限，可以输出每秒的磁盘存取速度，适合线上排查问题时使用，命令小巧实用。

使用方式：

```
sar -n DEV 1 1

```

命令输出：

```
robert@robert-ubuntu1410：~$ sar -n DEV 1 1
Linux 3.16.0-30-generic (robert-ubuntu1410)     2017年04月23日     _x86_64_    (4 CPU)

11时02分43秒     IFACE   rxpck/s   txpck/s    rxkB/s    txkB/s   rxcmp/s   txcmp/s  rxmcst/s   %ifutil
11时02分44秒      eth0      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00
11时02分44秒        lo      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00

平均时间：    IFACE   rxpck/s   txpck/s    rxkB/s    txkB/s   rxcmp/s   txcmp/s  rxmcst/s   %ifutil
平均时间：     eth0      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00
平均时间：       lo      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00

```

从上面的用法可以看到网卡的读写速度和流量，在现实的应急过程中，可以用来判断服务器是否上量。

此命令除了可以查看网卡的信息以外，sar可以用来收集更多的服务的状态信息：

> 1. -A：所有报告的总和

1. -u：CPU利用率
2. -v：进程、I节点、文件和锁表状态
3. -d：硬盘使用报告
4. -r：没有使用的内存页面和硬盘块
5. -g：串口 I/O的情况
6. -b：缓冲区使用情况
7. -a：文件读写情况
8. -c：系统调用情况
9. -R：进程的活动情况
10. -y：终端设备活动情况
11. -w：系统交换活动

### 6.9 netstat(ss)

此命令显示网络连接、端口信息等，另外一个命令ss与netstat命令类似，就不单独介绍。

#### 6.9.1 根据进程查找端口

1. 根据进程名字查找进程ID：

   ```
   ps -elf | grep 进程

   ```

   输出为：

   ```
   robert@robert-ubuntu1410：~$ ps -elf | grep vesta
   0 S robert    2862  1988 10  80   0 - 233215 futex_ 10：00 pts/0   00：00：22 java -server -Xms512m -Xmx512m -Xmn128m -XX：PermSize=128m -Xss256k -XX：+DisableExplicitGC -XX：+UseConcMarkSweepGC -XX：+CMSParallelRemarkEnabled -XX：+UseCMSCompactAtFullCollection -XX：+UseCMSInitiatingOccupancyOnly -XX：CMSInitiatingOccupancyFraction=60 -verbose：gc -XX：+PrintGCDateStamps -XX：+PrintTenuringDistribution -XX：+PrintGCDetails -Xloggc：./logs/gc.log -cp /home/robert/working/workspace/vesta-id-generator/releases/vesta-id-generator-0.0.1-release/bin/vesta-rest-0.0.1/extlib -jar ./lib/vesta-rest-0.0.1.jar
   0 R robert    2963  2778  0  80   0 -  3993 -      10：04 pts/0    00：00：00 grep --color=auto vesta

   ```

   获得进程ID为2862。

2. 根据进程ID查找进程开启的端口：

   ```
   netstat -nap | grep 6588

   ```

   输出为：

   ```
   robert@robert-ubuntu1410：~$ netstat -nap | grep 2862
   tcp6       0      0 ：：：8080                 ：：：*                    LISTEN      2862/java       
   unix  2      [ ]         流        已连接     21371    2862/java   

   ```

   获得监听端口为8080。

## 6.9.2 根据端口查找进程

1. 查找使用端口的进程号：

   ```
   netstat －nap | grep 8080

   ```

   输出为：

   ```
   robert@robert-ubuntu1410：~$ netstat -nap | grep 8080

   ```

tcp6       0      0 ：：：8080                 ：：：*                    LISTEN      2862/java

```
​```

获得进程ID为2862。

```

1. 根据进程ID查找进程的详细信息。

   ```
   ps -elf | grep 2862

   ```

   输出为：

   ```
   robert@robert-ubuntu1410：~$ ps -elf | grep 2862
   0 S robert    2862  1988  3  80   0 - 233215 futex_ 10：00 pts/0   00：00：23 java -server -Xms512m -Xmx512m -Xmn128m -XX：PermSize=128m -Xss256k -XX：+DisableExplicitGC -XX：+UseConcMarkSweepGC -XX：+CMSParallelRemarkEnabled -XX：+UseCMSCompactAtFullCollection -XX：+UseCMSInitiatingOccupancyOnly -XX：CMSInitiatingOccupancyFraction=60 -verbose：gc -XX：+PrintGCDateStamps -XX：+PrintTenuringDistribution -XX：+PrintGCDetails -Xloggc：./logs/gc.log -cp /home/robert/working/workspace/vesta-id-generator/releases/vesta-id-generator-0.0.1-release/bin/vesta-rest-0.0.1/extlib -jar ./lib/vesta-rest-0.0.1.jar

   ```

### 6.10 iptraf

iptraf是一个实时查看网络流量的交互式的彩色的文本屏幕界面的监控工具。监控的数据比较全面，可输出TCP连接、网络接口、协议、端口、包大小等信息，但是耗费系统资源比较多，需要管理员权限。

使用方式：

```
sudo iptraf

```

命令输出：

![img](./iptraf.png)

在进入主界面之前可以选择不同的选项，在不同的选项下，可以查看不同维度的网络信息。

### 6.11 tcpdump

网络状况分析跟踪工具，可以用来抓包的一个实用的命令。要使用该工具，需要对TCP/IP协议有所熟悉，因为过滤使用的信息都来自TCP/IP协议的格式。

显示来源IP或者目的IP为192.168.1.102的网络通信：

```
sudo tcpdump  -i eth0 host 192.168.1.102

```

显示去往102.168.1.102的所有ftp会话信息：

```
tcpdump -i eth1 'dst 192.168.1.102 and (port 21 or 20)'

```

显示去往102.168.1.102的所有HTTP会话信息：

```
tcpdump -ni eth0 'dst 192.168.1.102 and tcp and port 8080' 

```

### 6.12 nmap

扫描某一主机打开的端口以及端口提供的服务信息，通常用于查看本机哪些端口对外提供服务，或者确定服务器哪些端口对外开放。

使用方式：

```
nmap -v -A localhost

```

命令输出：

```
robert@robert-ubuntu1410：~$ nmap -v -A localhost

Starting Nmap 6.40 ( http://nmap.org ) at 2017-04-23 12：11 CST
NSE：Loaded 110 scripts for scanning.
NSE：Script Pre-scanning.
Initiating Ping Scan at 12：11
Scanning localhost (127.0.0.1) [2 ports]
Completed Ping Scan at 12：11, 0.00s elapsed (1 total hosts)
Initiating Connect Scan at 12：11
Scanning localhost (127.0.0.1) [1000 ports]
Discovered open port 22/tcp on 127.0.0.1
Discovered open port 8080/tcp on 127.0.0.1
Discovered open port 25/tcp on 127.0.0.1
Discovered open port 3306/tcp on 127.0.0.1
Discovered open port 631/tcp on 127.0.0.1
Completed Connect Scan at 12：11, 0.01s elapsed (1000 total ports)
Initiating Service scan at 12：11
Scanning 5 services on localhost (127.0.0.1)
Completed Service scan at 12：11, 6.04s elapsed (5 services on 1 host)
NSE：Script scanning 127.0.0.1.
Initiating NSE at 12：11
Completed NSE at 12：11, 0.22s elapsed
Nmap scan report for localhost (127.0.0.1)
Host is up (0.00025s latency).
Not shown：995 closed ports
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     (protocol 2.0)
| ssh-hostkey：1024 95：41：c2：46：25：8d：bc：2d：d1：15：c6：90：ca：a7：8b：bc (DSA)
| 2048 47：32：93：bf：49：df：9c：e7：d7：c5：f8：ef：92：e3：28：c2 (RSA)
|_256 bd：ef：f2：21：01：b1：cb：78：c7：42：a8：f3：5f：40：e3：37 (ECDSA)
25/tcp   open  smtp    Postfix smtpd
|_smtp-commands：robert-ubuntu1410, PIPELINING, SIZE 10240000, VRFY, ETRN, STARTTLS, ENHANCEDSTATUSCODES, 8BITMIME, DSN, 
| ssl-cert：Subject：commonName=ubuntu-kylin
| Issuer：commonName=ubuntu-kylin
| Public Key type：rsa
| Public Key bits：2048
| Not valid before：2015-10-24T08：56：26+00：00
| Not valid after： 2025-10-21T08：56：26+00：00
| MD5：  2458 afb6 3955 335a b4ad 171e 3917 b222
|_SHA-1：eb49 e335 4352 ccd7 4582 aa2d 1002 7eb3 725e 9045
|_ssl-date：2103-09-27T17：18：12+00：00; +86y157d13h06m52s from local time.
631/tcp  open  ipp     CUPS 1.7
| http-methods：GET HEAD OPTIONS POST PUT
| Potentially risky methods：PUT
|_See http://nmap.org/nsedoc/scripts/http-methods.html
| http-robots.txt：1 disallowed entry 
|_/
|_http-title：Home - CUPS 1.7.2
3306/tcp open  mysql   MySQL 5.5.54-0ubuntu0.14.04.1
| mysql-info：Protocol：10
| Version：5.5.54-0ubuntu0.14.04.1
| Thread ID：38
| Some Capabilities：Long Passwords, Connect with DB, Compress, ODBC, Transactions, Secure Connection
| Status：Autocommit
|_Salt：yB|ixB~v
8080/tcp open  http    Apache Tomcat/Coyote JSP engine 1.1
|_http-favicon：Unknown favicon MD5：0488FACA4C19046B94D07C3EE83CF9D6
| http-methods：GET HEAD POST PUT DELETE TRACE OPTIONS PATCH
| Potentially risky methods：PUT DELETE TRACE PATCH
|_See http://nmap.org/nsedoc/scripts/http-methods.html
|_http-title：Site doesn't have a title (application/json;charset=UTF-8).
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at http://www.insecure.org/cgi-bin/servicefp-submit.cgi ：
SF-Port22-TCP：V=6.40%I=7%D=4/23%Time=58FC2968%P=x86_64-pc-linux-gnu%r(NULL
SF：,2B,"SSH-2\.0-OpenSSH_6\.6\.1p1\x20Ubuntu-2ubuntu2\.8\r\n");
Service Info：Host： robert-ubuntu1410

NSE：Script Post-scanning.
Initiating NSE at 12：11
Completed NSE at 12：11, 0.00s elapsed
Read data files from：/usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at http://nmap.org/submit/ .
Nmap done：1 IP address (1 host up) scanned in 6.49 seconds

```

从上面的输出可以看到，多个端口对外提供服务：

> 1. Discovered open port 22/tcp on 127.0.0.1

1. Discovered open port 8080/tcp on 127.0.0.1
2. Discovered open port 25/tcp on 127.0.0.1
3. Discovered open port 3306/tcp on 127.0.0.1
4. Discovered open port 631/tcp on 127.0.0.1

其中，8080是[Vesta发号器](https://link.jianshu.com?t=http://vesta.cloudate.net/)对外提供的服务，3306是mysql对外提供的服务。

## 7 Linux系统高级工具

### 7.1 pstack

pstack命令用来显示每个进程的调用栈。可以使用pstack来查看进程正在挂起的执行方法，也可以用来查看进程的本地线程堆栈，与JVM的jstack配合可以看到JVM线程运行的全景。

使用方式：

```
pstack 2862

```

命令输出：

```
pstack 9040 >> /tmp/pstack.log

Thread 289 (Thread 0x7f8928bdb700 (LWP 9041)):
#0  0x00000032a480ea5d in accept () from /lib64/libpthread.so.0
#1  0x00007f88735eaad7 in NET_Accept () from /apps/product/jdk1.6.0_19/jre/lib/amd64/libnet.so
#2  0x00007f88735e6ad0 in Java_java_net_PlainSocketImpl_socketAccept () from /apps/product/jdk1.6.0_19/jre/lib/amd64/libnet.so
#3  0x00007f8921010c48 in ?? ()
#4  0x00007f88fca90bd8 in ?? ()
#5  0x00007f88fca90c20 in ?? ()
#6  0x0000000000000001 in ?? ()
#7  0x00007f8928bd9c28 in ?? ()
#8  0x0000000000000000 in ?? ()

Thread 288 (Thread 0x7f88809fe700 (LWP 9042)):
#0  0x00000032a480b5bc in pthread_cond_wait@@GLIBC_2.3.2 () from /lib64/libpthread.so.0
#1  0x00007f89291b6757 in os::PlatformEvent::park() () from /apps/product/jdk1.6.0_19/jre/lib/amd64/server/libjvm.so
#2  0x00007f892918fc45 in Monitor::IWait(Thread*, long) () from /apps/product/jdk1.6.0_19/jre/lib/amd64/server/libjvm.so
#3  0x00007f892919040e in Monitor::wait(bool, long, bool) () from /apps/product/jdk1.6.0_19/jre/lib/amd64/server/libjvm.so
#4  0x00007f8928f413b5 in GCTaskManager::get_task(unsigned int) () from /apps/product/jdk1.6.0_19/jre/lib/amd64/server/libjvm.so
#5  0x00007f8928f42663 in GCTaskThread::run() () from /apps/product/jdk1.6.0_19/jre/lib/amd64/server/libjvm.so
#6  0x00007f89291b702f in java_start(Thread*) () from /apps/product/jdk1.6.0_19/jre/lib/amd64/server/libjvm.so
#7  0x00000032a48079d1 in start_thread () from /lib64/libpthread.so.0
#8  0x00000032a40e886d in clone () from /lib64/libc.so.6
......

```

### 7.1 strace

系统调用工具，是Linux系统下的一款程序调试工具，用来监控一个应用程序所使用的
系统调用，通过它可以跟踪系统调用，让你熟悉一个Linux程序在背后是怎么工作的。

适用于想研究Linux底层的工作机制，或者JVM和Linux系统本身的bug导致的技术攻关的场景。

> 由于虚拟机有问题，没有收集到这部分的输出信息 ：(

## 8 /Proc文件系统

Linux系统内核提供了通过/proc文件系统查看运行时内核内部数据结构的能力，也可以改变内核参数设置。

显示CPU信息：

```
cat /proc/cpuinfo

```

显示内存信息：

```
cat /proc/meminfo

```

显示详细的内存映射信息：

```
cat /proc/zoneinfo

```

显示磁盘映射信息：

```
cat /proc/mounts 

```

查看系统平均负载命令：

```
cat /proc/loadavg

```

## 9 性能和压测工具

### 9.1 ab

ab是一款针对HTTP协议实现的服务进行性能压测的工具，它本来是设计用来测量apache服务器的性能指标，特别是测试阿帕奇服务器每秒能够处理多少请求的指标，以及响应的时间等，但是此命令也可以用来测试一切通用的HTTP协议服务器的性能。

测量HTTP GET协议的接口：

```
robert@robert-ubuntu1410：~$ ab -c10 -n100000 "http://localhost：8080/genid"
This is ApacheBench, Version 2.3 <$Revision：1528965 $>
Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
Licensed to The Apache Software Foundation, http://www.apache.org/

Benchmarking localhost (be patient)
Completed 10000 requests
Completed 20000 requests
Completed 30000 requests
Completed 40000 requests
Completed 50000 requests
Completed 60000 requests
Completed 70000 requests
Completed 80000 requests
Completed 90000 requests
Completed 100000 requests
Finished 100000 requests


Server Software：       Apache-Coyote/1.1
Server Hostname：       localhost
Server Port：           8080

Document Path：         /genid
Document Length：       19 bytes

Concurrency Level：     10
Time taken for tests：  30.728 seconds
Complete requests：     100000
Failed requests：       0
Total transferred：     16700000 bytes
HTML transferred：      1900000 bytes
Requests per second：   3254.33 [#/sec] (mean)
Time per request：      3.073 [ms] (mean)
Time per request：      0.307 [ms] (mean, across all concurrent requests)
Transfer rate：         530.74 [Kbytes/sec] received

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect：       0    0   0.8      0      24
Processing：    0    3   3.1      2      88
Waiting：       0    2   2.7      1      80
Total：         0    3   3.3      2      88

Percentage of the requests served within a certain time (ms)
  50%      2
  66%      3
  75%      4
  80%      4
  90%      6
  95%      9
  98%     13
  99%     16
 100%     88 (longest request)

```

从输出中可以看出，开源的[Vesta发号器](https://link.jianshu.com?t=http://vesta.cloudate.net/)QPS达到3254.33，平均响应时间是3毫秒，所有请求在88毫秒内返回，99%的请求在16毫秒返回。

也可以对使用POST协议的服务进行压测：

```
ab -c 10 -n 1000 -p post -T  'application/x-www-form-urlencoded'  http://localhost：8080/billing/account/update

```

POST文件内容：

```
accountId=1149983321489408&clientDesc=1

```

### 9.2 jmeter

jmeter是apache组织开发的基于Java的性能压力测试工具。用于对Java开发的软件做压力测试，它最初被设计用于Web应用测试，但后来扩展到通用的性能测试领域。它可以用于测试静态和动态资源，例如静态文件、Java Applet、CGI脚本、Java类库、数据库、FTP服务器，HTTP服务器等等。

jmeter可以用于对服务器、网络或对象模拟巨大的负载，在不同类别的压力下，测试它们的强度和分析整体性能。另外，jmeter能够对应用程序做功能和回归测试，通过创建带有断言的脚本来自动化的验证你的程序满足你期望的结果。为了最大限度的灵活性，jmeter允许使用正则表达式创建断言。

jemeter是一个复杂性能测试工具和平台，开发者需要在自己的平台下集成jmeter，并且开发jemeter的测试用例才能使用，本文不对jmeter做展开，读者可自行通过阅读[jmeter主页](https://link.jianshu.com?t=http://jmeter.apache.org/)的文档学习。

### 9.3 mysqlslap

这是mysql自带的一款性能压测工具，通过模拟多个并发客户端访问mysql来执行压力测试，同时提供了详细的的数据性能报告。此工具可以自动生成测试表和数据，并且可以模拟读、写、混合读写、查询等不同的使用场景，并且能够很好的对比多个存储引擎在相同环境下的并发压力下性能上的差别。

#### 9.3.1 使用单线程测试

使用方式：

```
mysqlslap -a -uroot -pyouarebest

```

命令输出：

```
robert@robert-ubuntu1410：~$ mysqlslap -a -uroot -pyouarebest
Benchmark
    Average number of seconds to run all queries：0.108 seconds
    Minimum number of seconds to run all queries：0.108 seconds
    Maximum number of seconds to run all queries：0.108 seconds
    Number of clients running queries：1
    Average number of queries per client：0

```

这里可以看到，使用单线程连接一次服务器需要108毫秒。

#### 9.3.2 使用100个多线程测试

使用方式：

```
mysqlslap -a -c 100 -uroot -pyouarebest

```

命令输出：

```
robert@robert-ubuntu1410：~$ mysqlslap -a -c 100 -uroot -pyouarebest
Benchmark
    Average number of seconds to run all queries：0.504 seconds
    Minimum number of seconds to run all queries：0.504 seconds
    Maximum number of seconds to run all queries：0.504 seconds
    Number of clients running queries：100
    Average number of queries per client：0

```

这里可以看到，使用多线程连接一次服务器需要504毫秒/100，大约为5毫秒，可见增加并发提高了吞吐量指标。

#### 9.3.3 多次测试对测试结果求平均值

使用方式：

```
mysqlslap -a -i 10 -uroot -pyouarebest

```

命令输出：

```
robert@robert-ubuntu1410：~$ mysqlslap -a -i 10 -uroot -pyouarebest
Benchmark
    Average number of seconds to run all queries：0.108 seconds
    Minimum number of seconds to run all queries：0.098 seconds
    Maximum number of seconds to run all queries：0.132 seconds
    Number of clients running queries：1
    Average number of queries per client：0

```

这里可以看到，多次测试求平均值，可以看到不同次的测试的结果稍有不同，平均为108ms，这与第一个测试结果是相同的。

#### 9.3.4 测试读操作的性能指标

使用方式：

```
mysqlslap -a -c10 --number-of-queries=1000 --auto-generate-sql-load-type=read -uroot -pyouarebest

```

命令输出：

```
robert@robert-ubuntu1410：~$ mysqlslap -a -c10 --number-of-queries=1000 --auto-generate-sql-load-type=read -uroot -pyouarebest
Benchmark
    Average number of seconds to run all queries：0.048 seconds
    Minimum number of seconds to run all queries：0.048 seconds
    Maximum number of seconds to run all queries：0.048 seconds
    Number of clients running queries：10
    Average number of queries per client：100

```

可以算出，平均每个查询需要48毫秒/1000，为0.048毫秒。数据库服务器处理SQL的QPS为1000/0.048秒, 平均QPS为20833次/每秒。

#### 9.3.5 测试写操作的性能指标

使用方式：

```
mysqlslap -a -c10 --number-of-queries=1000 --auto-generate-sql-load-type=write -uroot -pyouarebest

```

命令输出：

```
robert@robert-ubuntu1410：~$ mysqlslap -a -c10 --number-of-queries=1000 --auto-generate-sql-load-type=write -uroot -pyouarebest
Benchmark
    Average number of seconds to run all queries：3.460 seconds
    Minimum number of seconds to run all queries：3.460 seconds
    Maximum number of seconds to run all queries：3.460 seconds
    Number of clients running queries：10
    Average number of queries per client：100

```

可以算出，平均每个写操作需要3460毫秒/1000，为3.4毫秒。数据库服务器处理SQL的QPS为1000/3.46秒, 平均QPS为289次/每秒。

#### 9.3.6 测试读写混合操作的性能指标

使用方式：

```
mysqlslap -a -c10 --number-of-queries=1000 --auto-generate-sql-load-type=mixed -uroot -pyouarebest

```

命令输出：

```
robert@robert-ubuntu1410：~$ mysqlslap -a -c10 --number-of-queries=1000 --auto-generate-sql-load-type=mixed -uroot -pyouarebest
Benchmark
    Average number of seconds to run all queries：1.944 seconds
    Minimum number of seconds to run all queries：1.944 seconds
    Maximum number of seconds to run all queries：1.944 seconds
    Number of clients running queries：10
    Average number of queries per client：100

```

可以算出，平均每个读或写操作需要1944毫秒/1000，为1.9毫秒。数据库服务器处理SQL的QPS为1000/1.944秒, 平均QPS为514次/每秒。

#### 9.3.7 多次不同并发数混合操作的性能指标

测试不同的存储引擎的性能进行对比，执行一次测试，分别50和100个并发，共执行1000次总查询，50和100个并发分别得到一次测试结果，并发数越多，执行完所有查询的时间越长，为了准确起见，可以多次迭代测试后求多次平均值。

使用方式：

```
mysqlslap -a --concurrency=50,100 --number-of-queries 1000 --debug-info --engine=myisam,innodb --iterations=5 -uroot -pyouarebest

```

命令输出：

```
robert@robert-ubuntu1410：~$ mysqlslap -a --concurrency=50,100 --number-of-queries 1000 --debug-info --engine=myisam,innodb --iterations=5 -uroot -pyouarebest
Benchmark
    Running for engine myisam
    Average number of seconds to run all queries：0.080 seconds
    Minimum number of seconds to run all queries：0.070 seconds
    Maximum number of seconds to run all queries：0.106 seconds
    Number of clients running queries：50
    Average number of queries per client：20


Benchmark
    Running for engine myisam
    Average number of seconds to run all queries：0.100 seconds
    Minimum number of seconds to run all queries：0.075 seconds
    Maximum number of seconds to run all queries：0.156 seconds
    Number of clients running queries：100
    Average number of queries per client：10

Benchmark
    Running for engine innodb
    Average number of seconds to run all queries：0.527 seconds
    Minimum number of seconds to run all queries：0.437 seconds
    Maximum number of seconds to run all queries：0.801 seconds
    Number of clients running queries：50
    Average number of queries per client：20

Benchmark
    Running for engine innodb
    Average number of seconds to run all queries：0.608 seconds
    Minimum number of seconds to run all queries：0.284 seconds
    Maximum number of seconds to run all queries：0.991 seconds
    Number of clients running queries：100
    Average number of queries per client：10


User time 0.85, System time 1.28
Maximum resident set size 14200, Integral resident set size 0
Non-physical pagefaults 36206, Physical pagefaults 0, Swaps 0
Blocks in 0 out 0, Messages in 0 out 0, Signals 0
Voluntary context switches 61355, Involuntary context switches 1244

```

从这次测试可以看到，并发数增多，由于有并发就有同步操作的损耗，100并发的响应时间性能指标要略小于50并发的性能指标。

### 9.4 sysbench

#### 9.4.1 CPU性能测试

使用方式：

```
sysbench --test=cpu --cpu-max-prime=20000 run

```

命令输出：

```
robert@robert-ubuntu1410：~$ sysbench --test=cpu --cpu-max-prime=20000 run
sysbench 0.4.12： multi-threaded system evaluation benchmark

Running the test with following options：
Number of threads：1

Doing CPU performance benchmark

Threads started!
Done.

Maximum prime number checked in CPU test：20000


Test execution summary：
    total time：                         26.0836s
    total number of events：             10000
    total time taken by event execution：26.0795
    per-request statistics：
         min：                                 2.41ms
         avg：                                 2.61ms
         max：                                 6.29ms
         approx.  95 percentile：              2.93ms

Threads fairness：
    events (avg/stddev)：          10000.0000/0.00
    execution time (avg/stddev)：  26.0795/0.00

```

从这里可以看出做一次素数加法运算平均时间是2.61毫秒。

#### 9.4.2 线程锁性能测试

使用方式：

```
robert@robert-ubuntu1410：~$ sysbench --test=threads --num-threads=64 --thread-yields=100 --thread-locks=2 run

```

命令输出：

```
robert@robert-ubuntu1410：~$ sysbench --test=threads --num-threads=64 --thread-yields=100 --thread-locks=2 run
sysbench 0.4.12： multi-threaded system evaluation benchmark

Running the test with following options：
Number of threads：64

Doing thread subsystem performance test
Thread yields per test：100 Locks used：2
Threads started!
Done.


Test execution summary：
    total time：                         0.6559s
    total number of events：             10000
    total time taken by event execution：41.5442
    per-request statistics：
         min：                                 0.02ms
         avg：                                 4.15ms
         max：                               114.28ms
         approx.  95 percentile：             23.35ms

Threads fairness：
    events (avg/stddev)：          156.2500/36.13
    execution time (avg/stddev)：  0.6491/0.00

```

可见，在64个线程中，每个线程yield 100次，并且上锁2次，每次事件需要4毫秒的时间。

#### 9.4.3 磁盘随机IO性能测试

用sysbench工具可以测试顺序读，顺序写，随机读，随机写等磁盘IO性能：

```
sysbench --test=fileio --file-num=16 --file-total-size=100M prepare

sysbench --test=fileio --file-total-size=100M --file-test-mode=rndrd --max-time=180 --max-requests=100000000 --num-threads=16 --init-rng=on --file-num=16 --file-extra-flags=direct --file-fsync-freq=0 --file-block-size=16384 run

sysbench --test=fileio --file-num=16 --file-total-size=2G cleanup

```

命令输出：

```
robert@robert-Latitude-E6440：~/tmp$ sysbench --test=fileio --file-num=16 --file-total-size=100M prepare
sysbench 0.4.12： multi-threaded system evaluation benchmark

16 files, 6400Kb each, 100Mb total
Creating files for the test...

robert@robert-Latitude-E6440：~/tmp$ sysbench --test=fileio --file-total-size=100M --file-test-mode=rndrd --max-time=180 --max-requests=100000000 --num-threads=16 --init-rng=on --file-num=16 --file-extra-flags=direct --file-fsync-freq=0 --file-block-size=16384 run
sysbench 0.4.12： multi-threaded system evaluation benchmark

Running the test with following options：
Number of threads：16
Initializing random number generator from timer.


Extra file open flags：16384
16 files, 6.25Mb each
100Mb total file size
Block size 16Kb
Number of random requests for random IO：100000000
Read/Write ratio for combined random IO test：1.50
Calling fsync() at the end of test, Enabled.
Using synchronous I/O mode
Doing random read test
Threads started!
Time limit exceeded, exiting...
(last message repeated 15 times)
Done.

Operations performed： 43923 Read, 0 Write, 0 Other = 43923 Total
Read 686.3Mb  Written 0b  Total transferred 686.3Mb  (3.8104Mb/sec)
  243.86 Requests/sec executed

Test execution summary：
    total time：                         180.1126s
    total number of events：             43923
    total time taken by event execution：2880.7789
    per-request statistics：
         min：                                 0.13ms
         avg：                                65.59ms
         max：                              1034.24ms
         approx.  95 percentile：            223.33ms

Threads fairness：
    events (avg/stddev)：          2745.1875/64.44
    execution time (avg/stddev)：  180.0487/0.03

robert@robert-Latitude-E6440：~/tmp$ sysbench --test=fileio --file-num=16 --file-total-size=2G cleanup
sysbench 0.4.12： multi-threaded system evaluation benchmark

Removing test files...

```

上面测试显示，这台机器随机IO速度3M/s，IOPS高达243.86。

#### 9.4.4 内存性能测试

使用方式：

```
robert@robert-ubuntu1410：~$ sysbench --test=memory --memory-block-size=16k --memory-total-size=16K run

```

命令输出：

> 由于虚拟机有问题，没有收集到这部分的输出信息 ：(

#### 9.4.4 MYSQL事务性操作测试

由于prepare阶段不能自动创建schema，需要手工预先创建测试使用的schema，并且使用--mysql-db=test来指定。

使用方式：

```
sysbench --test=oltp --mysql-table-engine=myisam --oltp-table-size=1000 --mysql-user=root --mysql-host=localhost --mysql-password=youarebest --mysql-db=test  run

```

命令输出：

```
robert@robert-ubuntu1410：/etc/mysql$ sysbench --test=oltp --mysql-table-engine=myisam --oltp-table-size=1000 --mysql-user=root --mysql-host=localhost --mysql-password=youarebest --mysql-db=test  prepare
sysbench 0.4.12： multi-threaded system evaluation benchmark

No DB drivers specified, using mysql
Creating table 'sbtest'...
Creating 1000 records in table 'sbtest'...

robert@robert-ubuntu1410：/etc/mysql$ sysbench --test=oltp --mysql-table-engine=myisam --oltp-table-size=1000 --mysql-user=root --mysql-host=localhost --mysql-password=youarebest --mysql-db=test  run
sysbench 0.4.12： multi-threaded system evaluation benchmark

No DB drivers specified, using mysql
Running the test with following options：
Number of threads：1

Doing OLTP test.
Running mixed OLTP test
Using Special distribution (12 iterations,  1 pct of values are returned in 75 pct cases)
Using "LOCK TABLES WRITE" for starting transactions
Using auto_inc on the id column
Maximum number of requests for OLTP test is limited to 10000
Threads started!
Done.

OLTP test statistics：
    queries performed：
        read：                           140000
        write：                          50000
        other：                          20000
        total：                          210000
    transactions：                       10000  (689.49 per sec.)
    deadlocks：                          0      (0.00 per sec.)
    read/write requests：                190000 (13100.32 per sec.)
    other operations：                   20000  (1378.98 per sec.)

Test execution summary：
    total time：                         14.5035s
    total number of events：             10000
    total time taken by event execution：14.4567
    per-request statistics：
         min：                                 0.92ms
         avg：                                 1.45ms
         max：                                19.34ms
         approx.  95 percentile：              2.52ms

Threads fairness：
    events (avg/stddev)：          10000.0000/0.00
    execution time (avg/stddev)：  14.4567/0.00
    
robert@robert-ubuntu1410：/etc/mysql$ sysbench --test=oltp --mysql-table-engine=myisam --oltp-table-size=1000 --mysql-user=root --mysql-host=localhost --mysql-password=youarebest --mysql-db=test  cleanup
sysbench 0.4.12： multi-threaded system evaluation benchmark

No DB drivers specified, using mysql
Dropping table 'sbtest'...
Done.

```

从测试结果中得出事务TPS为689次/秒，读写的QPS为13100次/秒，每个请求处理的平均时间是1.45毫秒。

### 9.5 dd

dd可以用于测试磁盘顺序IO的存取速度，在应用场景中，打印日志通常表现为顺序IO的写操作，而数据库查询多为磁盘随机IO。

在磁盘上放一个文件，然后使用如下命令：

```
dd if=/home/robert/test-file of=/dev/null bs=512 count=10240000

```

从结果中就能看出这个磁盘的顺序IO的读取速度：

```
robert@robert-Latitude-E6440：~/working/multimedia-test$ dd if=./bigfile.tar of=/dev/null bs=512 count=10240000
记录了160+0 的读入
记录了160+0 的写出
81920字节(82 kB)已复制，0.0277534 秒，3.0 MB/秒
robert@robert-Latitude-E6440：~/working/multimedia-test$ dd if=./bigfile.tar of=/dev/null bs=512 count=10240000
记录了160+0 的读入
记录了160+0 的写出
81920字节(82 kB)已复制，0.000345242 秒，237 MB/秒
robert@robert-Latitude-E6440：~/working/multimedia-test$ dd if=./bigfile.tar of=/dev/null bs=512 count=10240000
记录了160+0 的读入
记录了160+0 的写出
81920字节(82 kB)已复制，0.000238306 秒，344 MB/秒

```

从上面的测试中发现，文件的顺序读取可以达到上百兆字节，第一次只有3兆，这是因为第一次操作系统的IO缓存没有命中导致的。普通x86机器上顺序读在100M左右，IBM或者华为的高端机器可以达到1G/s。

## 10 摘要命令

### 10.1 md5sum

用于生成md5摘要，通常用于文件上传和下载操作校验内容的正确性，或者通过加盐的hmac做对称数据签名。

为文件生成md5摘要：

```
robert@robert-ubuntu1410：~$ md5sum test.txt 
23cdc18507b52418db7740cbb5543e54  test.txt

```

### 10.2 sha256

由于md5摘要算法可以通过碰撞的方法进行破解，虽然，碰撞后数据还能符合业务规则的可能性比较小，但是安全无小事，大家都倾向于使用更安全的sha256算法。

通常也用于文件上传和下载操作校验正确性，或者通过加盐的sha256-hmac做对称数据签名。

为文件生成sha256摘要：

```
robert@robert-ubuntu1410：~$ sha256sum test.txt 
2634c3097f98e36865f0c572009c4ffd73316bc8b88ccfe8d196af35f46e2394  test.txt

```

### 10.3 base64

base64编码是网络上最常见的用于传输8位字节代码的编码方式之一，这种编码可以保证所输出的编码位全都是可读字符，base64制定了一个编码表，以便进行统一转换。编码表共有64个字符，因此称为base64编码。

base64编码把3个8位字节（3*8=24）转化为4个6位的字节（4*6=24），之后在6位的前面补两个0，形成8位一个字节的形式。如果剩下的字符不足3个字节，则用0填充，输出字符使用'='，因此编码后输出的文本末尾可能会出现1个或者2个'='。

把文件内容转化成base64编码：

```
robert@robert-ubuntu1410：~$ base64 test.txt 
MTIzNDU2NzgK

```

另外，区块链里面存储秘钥的时候并没有使用base64编码，而是使用了base58编码，去除了肉眼容易混淆的可见字符，例如：去除了'I'，因为它和数字'1'相似，去掉了字母'o'，因为它和数字0相似，从这里可以看到一款产品是如何从用户的角度思考和设计的。

## 11 命令与场景汇总表

本节把本文中介绍的所有的命令收集在一个表格中，称为“命令与场景汇总表”，便于大家随时参考和使用，并推荐大家把这个表格打印出来放在自己的办公桌上，需要的时候看一眼，便可快速发现和解决问题的命令和工具。

| 序号   | 命令                  | 使用场景                                     |
| ---- | ------------------- | ---------------------------------------- |
| 1    | grep                | 超级强悍的文本查找命令，常用于在大量文件中查找相关的关键词            |
| 2    | find                | 查找某些文件，常用来在众多项目中根据文件名查找某些文件              |
| 3    | uptime              | 查看操作系统启动的时间、用户、负载等                       |
| 4    | lsof                | 查看某个进程打开的文件句柄                            |
| 5    | ulimit              | 查看系统配置的用户对资源使用的限制，例如：打开的最大文件句柄、创建的最大线程数等 |
| 6    | curl                | 模拟HTTP协议调用                               |
| 7    | scp                 | 从服务器上下载文件或者上传文件到服务器上                     |
| 8    | vi/vim              | 在服务器上编辑文件，或者作为开发脚本程序的编辑环境                |
| 9    | dos2unix & unix2dos | 转换windows和unix/linux的换行符                 |
| 10   | ps                  | 查看系统内进程列表，并可以看到内存、CPU的信息                 |
| 11   | top                 | 按照资源使用情况排序显示系统内进程的列表                     |
| 12   | free                | 查看系统的内存使用情况                              |
| 13   | pmap                | 查看进程详细的内存分配情况                            |
| 14   | vmstat              | 查看系统的CPU利用率、负载、内存等信息                     |
| 15   | mpstat              | 查看系统的CPU利用率、负载，并可以按照CPU核心分别显示信息          |
| 16   | iostat              | 查看磁盘IO的信息以及传输速度                          |
| 17   | swapon              | 查看系统的交换区的使用情况                            |
| 18   | df                  | 显示磁盘挂载的信息                                |
| 19   | ifconfig            | 显示网卡挂载的信息                                |
| 20   | ping                | 检测服务器到其他服务器网络连接情况                        |
| 21   | telnet              | 可以检测某一个服务器的端口是否在正常对外服务                   |
| 22   | nc                  | 模拟开启TCP/IP的服务器，通常用于拦截HTTP协议传递的参数，帮助定位Restful服务的问题 |
| 23   | mtr                 | 检测网络连通性问题，并可以获取某一个域名或者IP的丢包率             |
| 24   | nslookup            | 判断DNS是否能够正确解析域名，以及域名解析到哪个IP地址            |
| 25   | traceroute          | 跟踪网络传输的详细路径，显示每一级网关的信息                   |
| 26   | sar                 | 全面的监控网络、磁盘、CPU、内存等信息的轻量级工具               |
| 27   | netstat(ss)         | 通常用于查看网络端口的连接情况                          |
| 28   | iptraf              | 用来获得网络IO的传输速度以及其他的网络状态信息                 |
| 29   | tcpdump             | 可以拦截本机网卡任何协议的通讯内容，用来调试网络问题               |
| 30   | nmap                | 扫描某一服务器打开的端口                             |
| 31   | pstack              | 打印进程内调用堆栈                                |
| 32   | strace              | 跟踪进程内工作机制                                |
| 33   | /Proc文件系统           | 另外一种方法实时查看系统的CPU、内存、IO等信息                |
| 34   | ab                  | 简单好用的HTTP协议的压测工具                         |
| 35   | jmeter              | 用于复杂的Java程序的测试工具                         |
| 36   | mysqlslap           | 用于测试mysql性能的弓弩                           |
| 37   | sysbench            | 可以用于测试系统IO、网络、CPU、内存等的性能指标，也可以用来测试mysql的各项性能指标 |
| 38   | dd                  | 磁盘文件拷贝操作                                 |
| 39   | md5sum              | 生成md5摘要                                  |
| 40   | sha256              | 生成sha256摘要                               |
| 41   | base64              | 生成base64编码                               |

## 12 总结经验

本文全面介绍了线上应急和技术攻关必不可少的基础Linux命令和工具，包括：查看活动进程的命令、内存监控命令、CPU使用情况监控命令、磁盘IO监控命令、网络查看和监控命令、Linux系统高级工具、Proc文件系统、性能压测工具、生成摘要的命令和工具等。

在文章末尾，对本文介绍的命令进行了总结，并且汇入了一个表格，表格可以用来帮助查找不同场景使用哪些命令能够解决特定的问题，读者可以把此表格打印出来，放在桌面上，需要的时候瞄一眼，就可以找到相应命令来定位问题。