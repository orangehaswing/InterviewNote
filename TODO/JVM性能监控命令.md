# JVM性能监控命令

### jps：虚拟机进程状况工具

JVM Process Status Tool，显示指定系统内所有的HotSpot虚拟机进程

jps命令格式：

```
jps [ options ] [ hostid ]

```

执行样例：

```
[root@miaomiao-calculatescore ~]# jps -l
5256 /opt/CalculateScore/CalculateScore.jar
17766 sun.tools.jps.Jps

```

option参数：

| 选项   | 作用                           |
| ---- | ---------------------------- |
| -q   | 只输出LVMID，省略主类的名称             |
| -m   | 输出虚拟机进程启动时传递给主类main()函数的参数   |
| -l   | 输出主类的全名，如果进程执行的是jar包，输出Jar路径 |
| -v   | 输出虚拟机进程启动时的JVM参数             |

### jstat：虚拟机统计信息监视工具

JVM Statistics Monitoring Tool，用于监视虚拟机各种运行状态信息的命令行工具。它可以显示出虚拟机进程中的类装载、内存、垃圾收集、JIT（Just In Time）即时编译等运行数据。

jstat命令格式：

```
jstat [ option lvmid [ interval[s|ms] [count] ] ]

```

option参数：

| 选项                | 作用                                       |
| ----------------- | ---------------------------------------- |
| -class            | class loader的行为统计。监视类装载、卸载数量、总空间以及类装载所耗费的时间 |
| -gc               | 垃圾回收堆的行为统计。监视Java堆状况，包括Eden区、两个Survivor区、老年代、永久代等的容量、已用空间、GC时间合计等信息 |
| -gccapacity       | 监视内容与-gc基本相同，但输出主要关注java堆各个区域(young,old,perm)使用到的最大、最小空间 |
| -gcutil           | 监视内容与-gc基本相同，但输出主要关注已使用空间占总空间的百分比        |
| -gccause          | 与-gcutil功能一样，但是会额外输出导致上一次GC的原因           |
| -gcnew            | 新生代行为统计。监视新生代GC状况                        |
| -gcnewcapacity    | 监视内容与-gcnew基本相同，输出主要关注使用到的最大、最小空间        |
| -gcold            | 年老代和永生代行为统计。监视老年代和永久代GC状况                |
| -gcoldcapacity    | 年老代行为统计。输出主要关注使用到的最大、最小空间                |
| -gcpermcapacity   | 永生代行为统计。输出主要关注使用到的最大、最小空间                |
| -compiler         | HotSpt JIT编译器行为统计。输出JIT编译器编译过的方法、耗时等信息   |
| -printcompilation | HotSpot编译方法统计。输出已经被JIT编译的方法              |

option参数详解：

-class
监视类装载、卸载数量、总空间以及类装载所耗费的时间
执行样例：

```
[root@miaomiao-calculatescore ~]# jstat -class 5256 500 3
Loaded  Bytes  Unloaded  Bytes     Time   
 7280 15178.8       81   132.2       5.16
 7280 15178.8       81   132.2       5.16
 7280 15178.8       81   132.2       5.16

```

> Loaded：加载的class数量
> Bytes：加载的class字节大小
> Unloaded：未加载的class数量
> Bytes：未加载的class字节大小
> Time：加载时间

------

-gc
监视Java堆状况，包括Eden区、两个Survivor区、老年代、永久代等的容量、已用空间、GC时间合计等信息
执行样例：

```
[root@miaomiao-calculatescore ~]# jstat -gc 20955
S0C    S1C    S0U    S1U      EC       EU        OC         OU       PC     
PU    YGC     YGCT    FGC    FGCT     GCT   
384.0  448.0   0.0   416.0   9408.0   5963.6   62272.0    60694.8   
56960.0 56669.3  10337   37.052  55     10.115   47.167

```

C即Capacity 总容量，U即Used 已使用的容量

> S0C：survivor0区的总容量
> S1C：survivor1区的总容量
> S0U：survivor0区已使用的容量
> S1U：survivor1区已使用的容量
> EC：Eden区的总容量
> EU：Eden区已使用的容量
> OC：Old区的总容量
> OU：Old区已使用的容量
> PC：当前perm的容量 (KB)
> PU：perm的使用 (KB)
> YGC：新生代垃圾回收次数
> YGCT：新生代垃圾回收时间
> FGC：老年代垃圾回收次数
> FGCT：老年代垃圾回收时间
> GCT：垃圾回收总消耗时间

------

-gccapacity
监视内容与-gc基本相同，但输出主要关注java堆各个区域(young,old,perm)使用到的最大、最小空间
执行样例：

```
[root@miaomiao-calculatescore ~]# jstat -gccapacity 20955
 NGCMN    NGCMX     NGC     S0C   S1C       EC      OGCMN      
 OGCMX       OGC         OC      PGCMN    PGCMX     PGC       
 PC    YGC    FGC 
 21120.0 338560.0  18240.0  896.0  896.0  16320.0    42304.0  
 677248.0    66944.0    66944.0  21248.0  83968.0  56960.0  
 56960.0  10803    60

```

> NGCMN : 新生代占用的最小空间
> NGCMX : 新生代占用的最大空间
> NGC : 当前新生代的容量
> S0C：当前survivor0区的空间
> S1C：当前survivor1区的空间
> EC：当前Eden区的空间
> OGCMN : 老年代占用的最小空间
> OGCMX : 老年代占用的最大空间
> OGC：当前年老代的容量 (KB)
> OC：当前年老代的空间 (KB)
> PGCMN : perm占用的最小空间
> PGCMX : perm占用的最大空间
> PGC : 当前perm区的容量
> PC : 当前perm区的空间
> YGC：新生代垃圾回收次数
> FGC：老年代垃圾回收次数

------

-gcutil
监视内容与-gc基本相同，但输出主要关注已使用空间占总空间的百分比
执行样例：

```
[root@miaomiao-calculatescore ~]# jstat -gcutil 20955
 S0     S1     E      O      P     YGC     YGCT    FGC    FGCT     
 GCT   
 0.00  64.48  57.31  78.34  99.46  11105   39.882    61   11.202   
 51.084

```

> S0：survivor0区占用百分比
> S1：survivor1区占用百分比
> E：Eden区占用百分比
> O : 年老代占用百分比
> P : 永久代占用百分比
> YGC：新生代垃圾回收次数
> YGCT：新生代垃圾回收时间
> FGC：老年代垃圾回收次数
> FGCT：老年代垃圾回收时间
> GCT：垃圾回收总消耗时间

------

-gccause
与-gcutil功能一样，但是会额外输出导致上一次GC的原因
执行样例：

```
[root@miaomiao-calculatescore ~]# jstat -gccause 20955
S0     S1     E      O      P     YGC     YGCT    FGC    FGCT     GCT        
LGCC                 GCC                 
0.00  58.82  12.34  79.37  99.49  11125   39.959    61   11.202   
51.161 unknown GCCause      No GC 

```

> S0：survivor0区占用百分比
> S1：survivor1区占用百分比
> E：Eden区占用百分比
> O : 年老代占用百分比
> P : 永久代占用百分比
> YGC：新生代垃圾回收次数
> YGCT：新生代垃圾回收时间
> FGC：老年代垃圾回收次数
> FGCT：老年代垃圾回收时间
> GCT：垃圾回收总消耗时间
> LGCC：最近垃圾回收的原因
> GCC：当前垃圾回收的原因

------

-gcnew
监视新生代GC状况
执行样例：

```
[root@miaomiao-calculatescore ~]# jstat -gcnew 20955
 S0C    S1C    S0U    S1U   TT MTT  DSS      EC       EU     YGC         
 YGCT  
 1152.0 1152.0    0.0  696.0 15  15 1152.0  69760.0  47539.3  
 11135   40.002

```

> S0C：survivor0区的总容量
> S1C：survivor1区的总容量
> S0U：survivor0区已使用的容量
> S1U：survivor1区已使用的容量
> TT：Tenuring threshold(晋升老年代的内存阈值)
> MTT：最大的tenuring threshold(晋升老年代的年龄阈值)
> DSS：survivor区域大小 (KB)
> EC：Eden区的总容量
> EU：Eden区已使用的容量
> YGC：新生代垃圾回收次数
> YGCT：新生代垃圾回收时间

------

-gcnewcapacity
监视内容与-gcnew基本相同，输出主要关注使用到的最大、最小空间
执行样例：

```
[root@miaomiao-calculatescore ~]# jstat -gcnewcapacity 20955
 NGCMN      NGCMX       NGC      S0CMX     S0C     S1CMX     
 S1C       ECMX        EC      YGC   FGC 
 21120.0   338560.0    73536.0    960.0 112832.0 112832.0   
 1088.0   338432.0    71296.0 11160    61

```

> NGCMN : 新生代占用的最小空间
> NGCMX : 新生代占用的最大空间
> NGC : 当前新生代的容量
> S0CMX : survivor0区占用的最大空间
> S0C：当前survivor0区的空间
> S1CMX : survivor1区占用的最大空间
> S1C：当前survivor1区的空间
> ECMX : Eden区占用的最大空间
> EC：当前Eden区的空间
> YGC：新生代垃圾回收次数
> FGC：老年代垃圾回收次数

------

-gcold
监视老年代和永久代GC状况
执行样例：

```
[root@miaomiao-calculatescore ~]# jstat -gcold 20955
PC       PU        OC          OU       YGC    FGC    FGCT     GCT   
56960.0  56669.6     64512.0     53414.0  11173    61   11.202   
51.383

```

> PC：当前perm的容量 (KB)
> PU：perm的使用 (KB)
> OC：Old区的总容量
> OU：Old区已使用的容量
> YGC：新生代垃圾回收次数
> FGC：老年代垃圾回收次数
> FGCT：老年代垃圾回收时间
> GCT：垃圾回收总消耗时间

------

-gcoldcapacity
输出主要关注使用到的最大、最小空间
执行样例：

```
[root@miaomiao-calculatescore ~]# jstat -gcoldcapacity 20955
OGCMN       OGCMX        OGC         OC       YGC   FGC    FGCT     
GCT   
42304.0    677248.0     64512.0     64512.0 11185    61   11.202   
51.440

```

> OGCMN : 老年代占用的最小空间
> OGCMX : 老年代占用的最大空间
> OGC：当前年老代的容量 (KB)
> OC：当前年老代的空间 (KB)
> YGC：新生代垃圾回收次数
> FGC：老年代垃圾回收次数
> FGCT：老年代垃圾回收时间
> GCT：垃圾回收总消耗时间

------

-gcpermcapacity
永生代行为统计。输出主要关注使用到的最大、最小空间
执行样例：

```
[root@miaomiao-calculatescore ~]#jstat -gcpermcapacity 28920
 PGCMN      PGCMX       PGC         PC      YGC   FGC    FGCT     
 GCT   
 1048576.0  2097152.0  1048576.0  1048576.0     4     0    0.000    
 0.242

```

> PGCMN : 永久代占用的最小空间
> PGCMX : 永久代占用的最大空间
> PGC：当前永久代的容量 (KB)
> PC：当前永久的空间 (KB)
> YGC：新生代垃圾回收次数
> FGC：老年代垃圾回收次数
> FGCT：老年代垃圾回收时间
> GCT：垃圾回收总消耗时间

------

-compiler
HotSpt JIT编译器行为统计。输出JIT编译器编译过的方法、耗时等信息
执行样例：

```
[root@miaomiao-calculatescore ~]# jstat -compiler 20955
 Compiled Failed Invalid   Time   FailedType FailedMethod
 2847      0       0    36.17          0

```

> Compiled : 编译数量
> Failed : 编译失败数量
> Invalid : 无效数量
> Time : 编译耗时
> FailedType : 失败类型
> FailedMethod : 失败方法的全限定名

------

-printcompilation
HotSpot编译方法统计。输出已经被JIT编译的方法
执行样例：

```
[root@miaomiao-calculatescore ~]# jstat -printcompilation 20955
 Compiled  Size  Type Method
  2848   2373    1 java/util/Arrays mergeSort

```

> Compiled：被执行的编译任务的数量
> Size：方法字节码的字节数
> Type：编译类型
> Method：编译方法的类名和方法名。类名使用”/” 代替 “.” 作为空间分隔符. 方法名是给出类的方法名. 格式是与- XX:+PrintComplation选项一致

### jinfo：Java配置信息工具

Configuration Info for Java，显示虚拟机配置信息，实时查看和调整虚拟机各项参数。 jps命令的-v选项可以查看虚拟机启动时显示指定的参数，如果想要查看未被显示指定的参数的系统默认值就要使用jinfo命令的-flag选项进行查询了。

jinfo命令格式：

```
jinfo [ option ] [ args ] lvmid

```

执行样例：

```
[root@miaomiao-calculatescore ~]# jinfo -flag CMSInitiatingOccupancyFraction 20955
 -XX:CMSInitiatingOccupancyFraction=-1

```

option参数：

> -flag : 输出指定args参数的值
> -flags : 不需要args参数，输出所有JVM参数的值
> -sysprops : 输出系统属性，等同于System.getProperties()

### jmap：Java内存影像工具

Memory Map for Java，生成堆转储快照（一般称为heapdump或dump文件）。 jmap的作用不仅仅是为了获取dump文件，还可以查询finalize执行队列、Java堆和永久代的详细信息，如空间使用率、当前使用的是哪种收集器等。

jmap命令格式：

```
jmap [ option ] lvmid

```

option参数：

| 选项             | 作用                                       |
| -------------- | ---------------------------------------- |
| -dump          | 生成Java堆转储快照。格式为：-dump:[live, ]format=b,file=<filename>其中live子参数说明是否只dump出存活的对象 |
| -finalizerinfo | 显示在F-Queue队列中等待Finalizer线程执行finalize方法的对象 |
| -heap          | 显示Java堆详细信息。如使用哪种垃圾收集器、参数配置、分代状况等        |
| -histo         | 显示堆中对象的统计信息，包括类、实例数量、合计容器                |
| -permstat      | 以ClassLoader为统计口径显示永久代内存状态               |
| -F             | 当虚拟机进程对-dump没有响应时，可使用此选项强制生成dump快照       |

option参数详解：

-dump
生成Java堆转储快照。格式为：-dump:[live, ]format=b,file=<filename> live指明是否只dump出存活的对象,format指定输出格式，file指定文件名

执行样例：

```
[root@miaomiao-calculatescore CalculateScore]# jmap -F -dump:live,format=b,file=dump.hprof 20995
 Attaching to process ID 20995, please wait...
 Debugger attached successfully.
 Server compiler detected.
 JVM version is 20.45-b01
 Dumping heap to dump.hprof ...

```

dump.hprof这个后缀是为了后续可以直接用MAT(Memory Anlysis Tool)打开。

------

-finalizerinfo
显示在F-Queue队列中等待Finalizer线程执行finalize方法的对象

执行样例：

```
[root@miaomiao-calculatescore CalculateScore]# jmap -finalizerinfo 20995
 Attaching to process ID 28920, please wait...
 Debugger attached successfully.
 Server compiler detected.
 JVM version is 24.71-b01
 Number of objects pending for finalization: 0

```

可以看到当前F-Queue队列中并没有等待Finalizer线程执行finalize方法的对象。

------

-heap
显示Java堆详细信息。如使用哪种垃圾收集器、参数配置、分代状况等

执行样例：

```
[root@miaomiao-calculatescore CalculateScore]# jmap -heap 17986
 Attaching to process ID 17986, please wait...
 Debugger attached successfully.
 Server compiler detected.
 JVM version is 20.45-b01

 using thread-local object allocation.
 Parallel GC with 2 thread(s) //GC收集器

 Heap Configuration: //堆内存初始化配置
   MinHeapFreeRatio = 40 //对应jvm启动参数-XX:MinHeapFreeRatio设置JVM堆最小空闲比率(default 40)
   MaxHeapFreeRatio = 70 //对应jvm启动参数 -XX:MaxHeapFreeRatio设置JVM堆最大空闲比率(default 70)
   MaxHeapSize      = 1040187392 (992.0MB) //对应jvm启动参数-XX:MaxHeapSize设置JVM堆的最大大小
   NewSize          = 1310720 (1.25MB)//对应jvm启动参数-XX:NewSize设置JVM堆的新生代的默认大小
   MaxNewSize       = 17592186044415 MB//对应jvm启动参数-XX:MaxNewSize设置JVM堆的新生代的最大大小
   OldSize          = 5439488 (5.1875MB)//对应jvm启动参数-XX:OldSize设置JVM堆的老生代的大小
   NewRatio         = 2 //对应jvm启动参数-XX:NewRatio设置新生代和老生代的大小比率
   SurvivorRatio    = 8 //对应jvm启动参数-XX:SurvivorRatio设置新生代中Eden区与Survivor区的大小比值 
   PermSize         = 21757952 (20.75MB) //对应jvm启动参数-XX:PermSize设置JVM堆的永久代的初始大小
   MaxPermSize      = 85983232 (82.0MB) //对应jvm启动参数-XX:MaxPermSize设置JVM堆的永久代的最大大小

 Heap Usage: //堆内存使用情况
 PS Young Generation
 Eden Space: //Eden区内存分布
   capacity = 130023424 (124.0MB) //Eden区总容量
   used     = 37682272 (35.936614990234375MB)  //Eden区已使用
   free     = 92341152 (88.06338500976562MB) //Eden区剩余容量
   28.981141121156753% used //Eden区使用比率
 From Space: //Survivor0区的内存分布
   capacity = 786432 (0.75MB)
   used     = 720928 (0.687530517578125MB)
   free     = 65504 (0.062469482421875MB)
   91.67073567708333% used
 To Space: //Survivor1区的内存分布
   capacity = 6422528 (6.125MB)
   used     = 0 (0.0MB)
   free     = 6422528 (6.125MB)
   0.0% used
 PS Old Generation //当前的Old区内存分布
   capacity = 43319296 (41.3125MB)
   used     = 20436424 (19.48969268798828MB)
   free     = 22882872 (21.82280731201172MB)
   47.176260666840015% used
 PS Perm Generation //当前的永久代内存分布
   capacity = 56164352 (53.5625MB)
   used     = 56004904 (53.410438537597656MB)
   free     = 159448 (0.15206146240234375MB)
   99.71610462095245% used

```

可以很清楚的看到Java堆中各个区域目前的情况。

------

-histo
显示堆中对象的统计信息，包括类、实例数量、合计容器。（因为在dump:live前会进行full gc，如果带上live则只统计活对象，因此不加live的堆大小要大于加live堆的大小 ）

执行样例：

```
[root@miaomiao-calculatescore CalculateScore]# jmap -histo:live 17986 | more

 num     #instances         #bytes  class name
 ----------------------------------------------
 1:         84343       12060272  <constMethodKlass>
 2:         84343       11481208  <methodKlass>
 3:         18176        8998776  [B
 4:          7130        8247248  <constantPoolKlass>
 5:        115421        6567664  <symbolKlass>
 6:          7130        5664136  <instanceKlassKlass>
 7:          5920        4854144  <constantPoolCacheKlass>
 8:         42489        3981136  [C
 9:         13065        1855784  [I
10:         52831        1690592  java.util.HashMap$Entry
--More--

```

class name是对象类型，说明如下：

> B  byte
> C  char
> D  double
> F  float
> I  int
> J  long
> Z  boolean
> [  数组，如[I表示int[]
> [L+类名 其他对象

------

-permstat
以ClassLoader为统计口径显示永久代内存状态。对于每个类加载器而言，它的名称、活跃度、地址、父类加载器、它所加载的类的数量和大小都会显示。此外，包含的字符串数量和大小也会显示。

执行样例：

```
[root@miaomiao-calculatescore CalculateScore]# jmap -permstat 17986 |more
 Attaching to process ID 17986, please wait...
 Debugger attached successfully.
 Server compiler detected.
 JVM version is 20.45-b01
 finding class loader instances ..16401 intern Strings occupying 
 1810240 bytes.
 Finding object size using Printezis bits and skipping over...
 Finding object size using Printezis bits and skipping over...
 Finding object size using Printezis bits and skipping over...
 done.
 computing per loader stat ..done.
 please wait.. computing liveness...........................liveness analysis may be inaccurate ...
 class_loader   classes bytes   parent_loader   alive?  type

 <bootstrap>    1938    11332152      null      live    <internal>
 0x00000000c2bdb1b8 1   3176    0x00000000c2112748  dead    sun/reflect/DelegatingClassLoader@0x00000000bce67830
 0x00000000c2c74c40 1   3120    0x00000000c2112748  dead    sun/reflect/DelegatingClassLoader@0x00000000bce67830
 --More--

```

------

-F
强制模式。如果指定的lvmid没有响应，请使用jmap -F -dump或jmap  -F -histo选项。此模式下，不支持live子选项。

### jhat：虚拟机堆转储快照分析工具

JVM Heap Analysis Tool，与jmap搭配使用，用来分析jmap生成的heapdump文件，它会建立一个HTTP/HTML服务器，让用户可以在浏览器上查看分析结果。在实际工作中，一般不会直接使用jhat命令分析dump文件行，因为分析工作是一个耗时而且耗费硬件资源的过程，另一个原因是jhat的分析功能相对于专业的分析工具（VisualVM、IBM HeapAnalyzer等）比较简陋。

jhat命令格式：

```
jhat [ option ] [ heapdumpfile ]

```

执行样例：

```
[root@miaomiao-calculatescore CalculateScore]# jhat -J-Xmx256M dump.hprof
 Reading from dump.hprof...
 Dump file created Sun Mar 25 11:20:43 CST 2018
 Snapshot read, resolving...
 Resolving 593966 objects...
 Chasing references, expect 118 dots..........................
 Eliminating duplicate references.............................
 Snapshot resolved.
 Started HTTP server on port 7000
 Server is ready.

```

-J-Xmx256M是在dump快照很大的情况下分配内存去启动HTTP服务器，运行完之后就可在浏览器打开[Http://localhost:7000](https://link.jianshu.com?t=Http%3A%2F%2Flocalhost%3A7000)进行快照分析。堆快照分析主要在最后面的Heap Histogram里，里面根据class列出了dump的时候所有存活对象。

分析：
打开浏览器[Http://localhost:7000](https://link.jianshu.com?t=Http%3A%2F%2Flocalhost%3A7000)，该页面提供了几个查询功能可供使用。如图二：

![img](//upload-images.jianshu.io/upload_images/11232455-8f8af21b780deb64.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/559/format/webp)

图二

一般查看堆异常情况主要看这个两个部分：
Show instance counts for all classes (excluding platform)，平台外的所有对象信息。如图三：

![img](//upload-images.jianshu.io/upload_images/11232455-5cf922ed914fc473.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp)

图三

Show heap histogram 以树状图形式展示堆情况。如图四：

![img](//upload-images.jianshu.io/upload_images/11232455-94a6809a31f1b03a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp)

图四

具体排查时需要结合代码，看是否存在大量应该被回收的对象一直被引用或者是否有占用内存特别大的对象无法被回收。

###### 通常情况下，是将dump文件down下来，通过专业的工具（VisualVM、IBM HeapAnalyzer等）来进行分析。

option参数：

| 选项        | 值            | 作用                                       |
| --------- | ------------ | ---------------------------------------- |
| -stack    | false、true   | 关闭对象分配调用栈跟踪。 如果分配位置信息在堆转储中不可用，则必须将此标志设置为 false。默认值为 true。 |
| -refs     | false、true   | 关闭对象引用跟踪。默认值为 true。默认情况下, 返回的指针是指向其他特定对象的对象，如反向链接或输入引用，会统计/计算堆中的所有对象。 |
| -port     | port-number  | 设置 jhat HTTP server 的端口号。默认值 7000        |
| -exclude  | exclude-file | 指定对象查询时需要排除的数据成员列表文件。 例如，如果文件列出了 java.lang.String.value，那么当从某个特定对象 Object o 计算可达的对象列表时，引用路径涉及 java.lang.String.value 的都会被排除。 |
| -baseline | exclude-file | 指定一个基准堆转储(baseline heap dump)。 在两个 heap dumps 中有相同 object ID 的对象会被标记为不是新的。其他对象被标记为新的(new)。在比较两个不同的堆转储时很有用。 |
| -debug    | int          | 设置 debug 级别。0 表示不输出调试信息。 值越大则表示输出更详细的 debug 信息。 |
| -version  |              | 启动后只显示版本信息就退出。                           |
| -J        | < flag >     | 因为 jhat 命令实际上会启动一个JVM来执行，通过 -J 可以在启动JVM时传入一些启动参数。例如， -J-Xmx512M 则指定运行 jhat 的Java虚拟机使用的最大堆内存为 512M。如果需要使用多个JVM启动参数，则传入多个 -Jxxxxxx。 |

### jstack：Java堆栈跟踪工具

Stack Trace for Java，显示虚拟机的线程快照。线程快照是当前虚拟机内每一条线程正在执行的方法堆栈的集合，生成线程快照的主要目的是定位线程出现长时间停顿的原因，如线程间死锁、死循环、请求外部资源导致的长时间等待等。 线程出现停顿的时候通过jstack来查看各个线程的调用堆栈，就可以知道没有响应的线程到底在后台做什么事情，或者等待什么资源。

jstack命令格式：

```
jstack [ option ] lvmid

```

执行样例：

```
[root@miaomiao-calculatescore CalculateScore]# jstack -l 17986|more
 2018-03-25 15:18:23
 Full thread dump Java HotSpot(TM) 64-Bit Server VM (20.45-b01 mixed mode):

 "hconnection-0x74bfed5a-shared--pool1-t11268" daemon prio=10 tid=0x0000000040f88000 nid=0xc42 waiting on condition
 [0x00007fe1c5642000]
  java.lang.Thread.State: TIMED_WAITING (parking)
   at sun.misc.Unsafe.park(Native Method)
   - parking to wait for  <0x00000000c228a1c8> (a 
  java.util.concurrent.locks.AbstractQueuedSynchronizer$Condi 
  tionObject) at java.util.concurrent.locks.LockSupport.parkNanos(LockSupport.java
  :196)  at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionO
   bject.awaitNanos(AbstractQueuedSynchron
   izer.java:2025)
   at java.util.concurrent.LinkedBlockingQueue.poll(LinkedBlockingQueue.java:424)
   at java.util.concurrent.ThreadPoolExecutor.getTask(ThreadPoolExecutor.java:955)
   at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:917)
   at java.lang.Thread.run(Thread.java:662)

  Locked ownable synchronizers:
  - None

```

option参数：

> -F : 当正常输出请求不被响应时，强制输出线程堆栈
> -l : 除堆栈外，显示关于锁的附加信息
> -m : 如果调用到本地方法的话，可以显示C/C++的堆栈



