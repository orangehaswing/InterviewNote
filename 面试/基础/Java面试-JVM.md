# JVM面试问题

## **垃圾回收**：

- 如何判断对象是否死亡（两种方法）。
- 简单的介绍一下强引用、软引用、弱引用、虚引用（虚引用与软引用和弱引用的区别、使用软引用能带来的好处）。
- 垃圾收集有哪些算法，各自的特点？
- HotSpot为什么要分为新生代和老年代？
- 常见的垃圾回收器有那些？
- 介绍一下CMS,G1收集器。
- Minor Gc和Full GC 有什么不同呢？
- GC的两种判定方法：引用计数与引用链。
- GC的三种收集方法：标记清除、标记整理、复制算法的原理与特点，分别用在什么地方，如果让你优化收集方法，有什么思路？
- GC收集器有哪些？CMS收集器与G1收集器的特点。

## **Java内存区域**：

- 对象的访问定位的两种方式。
- 内存模型以及分区，需要详细到每个区放什么？
- 内存泄漏和内存溢出的差别
- 堆里面的分区：Eden，survival from to，老年代，各自的特点。
- 对象创建方法，对象的内存分配，对象的访问定位。

## **性能监控和故障处理工具**

- 几种常用的内存调试工具：jmap、jstack、jconsole。

## **类文件结构**

- class类文件结构(常量池主要存放的是那两大常量？Class文件的继承关系是如何确定的？字段表、方法表、属性表主要包含那些信息？)

## **虚拟机类加载机制**

- 类加载的五个过程：加载、验证、准备、解析、初始化。
- 双亲委派模型：Bootstrap ClassLoader、Extension ClassLoader、ApplicationClassLoader。
- 反射机制，双亲委派机制，类加载器的种类


## 双亲委派模式

- 为什么要双亲委派

  场景：

  黑客自定义一个`java.lang.String`类，该`String`类具有系统的`String`类一样的功能，只是在某个函数稍作修改。比如`equals`函数，这个函数经常使用，如果在这这个函数中，黑客加入一些“病毒代码”。并且通过自定义类加载器加入到`JVM`中。此时，如果没有双亲委派模型，那么`JVM`就可能误以为黑客自定义的`java.lang.String`类是系统的`String`类，导致“病毒代码”被执行。

  而有了双亲委派模型，黑客自定义的`java.lang.String`类永远都不会被加载进内存。因为首先是最顶端的类加载器加载系统的`java.lang.String`类，最终自定义的类加载器无法加载`java.lang.String`类。

- 如何破坏双亲委派

  如果不想打破双亲委派模型，就重写ClassLoader类中的findClass()方法即可，无法被父类加载器加载的类最终会通过这个方法被加载。而如果想打破双亲委派模型则需要重写loadClass()方法

- Java9模块化

   Java 9 之前代码都是按包进行组织的，而模块则是在包的概念上又增加了一个更高层的抽象，它将多个逻辑上、功能上相关的包以及相关的资源文件，如 xml 文件等组织成一个可重用的逻辑单元，这个逻辑单元就称为模块。

  模块的类型

  - 具名模块(Named Module)
  - 无名模块(Unnamed Module)
  - 自动模块(Automatic Module)

- ClassNotFound

  1. 缺class即jar包，引入class所在的jar包即可
  2. java虚拟机即jre中一个很重要的东西classloader的执行策略问题了

- ClassNotFoundException和NoClassDefFoundErr区别

  ClassNotFoundException；

  当应用程序试图通过类的字符串名称，使用以下三种方法装入类，但却找不到指定名称的类定义时抛出该异常，是显式类装载的抛出的异常。

  1. 类 Class 中的 forName() 方法。
  2. 类 ClassLoader 中的 findSystemClass() 方法。
  3. 类 ClassLoader 中的 loadClass() 方法。

  NoClassDefFoundError：

  如果 Java 虚拟机或 ClassLoader 实例试图装入类定义（作为正常的方法调用的一部分，或者作为使用 new 表达式创建新实例的一部分），但却没有找到类定义时抛出该异常。

  实际上，这意味着 NoClassDefFoundError 的抛出，是不成功的隐式类装入的结果。简单说来，就是引用的类在类路径中没有找到。