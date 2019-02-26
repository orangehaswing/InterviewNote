# Java并发

# 线程状态转换

[![img](https://github.com/orangehaswing/OrdinaryNote/blob/master/Java%E5%B9%B6%E5%8F%91/resource/ace830df-9919-48ca-91b5-60b193f593d2.png?raw=true)](https://github.com/CyC2018/CS-Notes/blob/master/pics/ace830df-9919-48ca-91b5-60b193f593d2.png?raw=true)

## 新建（New）

创建后尚未启动。

## 可运行（Runnable）

可能正在运行，也可能正在等待 CPU 时间片。

包含了操作系统线程状态中的 Running 和 Ready。

## 阻塞（Blocking）

等待获取一个互斥锁，如果其线程释放了锁就会结束此状态。

## 无限期等待（Waiting）

等待其它线程显式地唤醒，否则不会被分配 CPU 时间片。

| 进入方法                              | 退出方法                                 |
| --------------------------------- | ------------------------------------ |
| 没有设置 Timeout 参数的 Object.wait() 方法 | Object.notify() / Object.notifyAll() |
| 没有设置 Timeout 参数的 Thread.join() 方法 | 被调用的线程执行完毕                           |
| LockSupport.park() 方法             | -                                    |

## 限期等待（Timed Waiting）

无需等待其它线程显式地唤醒，在一定时间之后会被系统自动唤醒。

调用 Thread.sleep() 方法使线程进入限期等待状态时，常常用“使一个线程睡眠”进行描述。

调用 Object.wait() 方法使线程进入限期等待或者无限期等待时，常常用“挂起一个线程”进行描述。

睡眠和挂起是用来描述行为，而阻塞和等待用来描述状态。

阻塞和等待的区别在于，阻塞是被动的，它是在等待获取一个排它锁。而等待是主动的，通过调用 Thread.sleep() 和 Object.wait() 等方法进入。

| 进入方法                             | 退出方法                                     |
| -------------------------------- | ---------------------------------------- |
| Thread.sleep() 方法                | 时间结束                                     |
| 设置了 Timeout 参数的 Object.wait() 方法 | 时间结束 / Object.notify() / Object.notifyAll() |
| 设置了 Timeout 参数的 Thread.join() 方法 | 时间结束 / 被调用的线程执行完毕                        |
| LockSupport.parkNanos() 方法       | -                                        |
| LockSupport.parkUntil() 方法       | -                                        |

## 死亡（Terminated）

可以是线程结束任务之后自己结束，或者产生了异常而结束。

# 使用线程

有三种使用线程的方法：

- 实现 Runnable 接口；
- 实现 Callable 接口；
- 继承 Thread 类。

实现 Runnable 和 Callable 接口的类只能当做一个可以在线程中运行的任务，不是真正意义上的线程，因此最后还需要通过 Thread 来调用。可以说任务是通过线程驱动从而执行的。

## 实现 Runnable 接口

需要实现 run() 方法。

通过 Thread 调用 start() 方法来启动线程。

```
public class MyRunnable implements Runnable {
    public void run() {
        // ...
    }
}
```

```
public static void main(String[] args) {
    MyRunnable instance = new MyRunnable();
    Thread thread = new Thread(instance);
    thread.start();
}
```

## 实现 Callable 接口

与 Runnable 相比，Callable 可以有返回值，返回值通过 FutureTask 进行封装。

```
public class MyCallable implements Callable<Integer> {
    public Integer call() {
        return 123;
    }
}
```

```
public static void main(String[] args) throws ExecutionException, InterruptedException {
    MyCallable mc = new MyCallable();
    FutureTask<Integer> ft = new FutureTask<>(mc);
    Thread thread = new Thread(ft);
    thread.start();
    System.out.println(ft.get());
}
```

## 继承 Thread 类

同样也是需要实现 run() 方法，因为 Thread 类也实现了 Runable 接口。

当调用 start() 方法启动一个线程时，虚拟机会将该线程放入就绪队列中等待被调度，当一个线程被调度时会执行该线程的 run() 方法。

```
public class MyThread extends Thread {
    public void run() {
        // ...
    }
}
```

```
public static void main(String[] args) {
    MyThread mt = new MyThread();
    mt.start();
}
```

## 实现接口 VS 继承 Thread

实现接口会更好一些，因为：

- Java 不支持多重继承，因此继承了 Thread 类就无法继承其它类，但是可以实现多个接口；
- 类可能只要求可执行就行，继承整个 Thread 类开销过大。

## 三种方式的区别

- 实现 Runnable 接口可以避免 Java 单继承特性而带来的局限；增强程序的健壮性，代码能够被多个线程共享，代码与数据是独立的；适合多个相同程序代码的线程区处理同一资源的情况。
- 继承 Thread 类和实现 Runnable 方法启动线程都是使用 start() 方法，然后 JVM 虚拟机将此线程放到就绪队列中，如果有处理机可用，则执行 run() 方法。
- 实现 Callable 接口要实现 call() 方法，并且线程执行完毕后会有返回值。其他的两种都是重写 run() 方法，没有返回值。

# 基础线程机制

## Executors

Executor 管理多个异步任务的执行，而无需程序员显式地管理线程的生命周期。这里的异步是指多个任务的执行互不干扰，不需要进行同步操作。

**为什么引入Executor线程池框架？**

new Thread() 的缺点

- 每次 new Thread() 耗费性能
- 调用 new Thread() 创建的线程缺乏管理，被称为野线程，而且可以无限制创建，之间相互竞争，会导致过多占用系统资源导致系统瘫痪。
- 不利于扩展，比如如定时执行、定期执行、线程中断

采用线程池的优点

- 重用存在的线程，减少对象创建、消亡的开销，性能佳
- 可有效控制最大并发线程数，提高系统资源的使用率，同时避免过多资源竞争，避免堵塞
- 提供定时执行、定期执行、单线程、并发数控制等功能

### 线程池工作原理

[![ThrealpoolExecutor_framework](https://github.com/orangehaswing/fullstack-tutorial/raw/master/notes/JavaArchitecture/assets/ThrealpoolExecutor_framework.jpg)](https://github.com/orangehaswing/fullstack-tutorial/blob/master/notes/JavaArchitecture/assets/ThrealpoolExecutor_framework.jpg)

#### 并发队列

**入队**

非阻塞队列：当队列中满了时候，放入数据，数据丢失

阻塞队列：当队列满了的时候，进行等待，什么时候队列中有出队的数据，那么第11个再放进去

**出队**

非阻塞队列：如果现在队列中没有元素，取元素，得到的是null

阻塞队列：等待，什么时候放进去，再取出来

线程池使用的是阻塞队列

线程池中的核心线程数，当提交一个任务时，线程池创建一个新线程执行任务，直到当前线程数等于corePoolSize；如果当前线程数为 corePoolSize，继续提交的任务被保存到阻塞队列中，等待被执行；如果阻塞队列满了，那就创建新的线程执行当前任务；直到线程池中的线程数达到 maxPoolSize，这时再有任务来，只能执行 reject() 处理该任务。

主要有三种 Executor：

#### newCachedThreadPool：

```
public static ExecutorService newCachedThreadPool(){
  return new ThreadPoolExecutor(0, Integer.MAX_VAUE,
  								60L, TimeUnit.SECONDS,
                                new SynchoronousQueue<Runnable>());
}
```

​

1. 初始化一个可以缓存线程的线程池，默认缓存60s，线程池的线程数可达到Integer.MAX_VALUE，即2147483647，内部使用SynchronousQueue作为阻塞队列；

2. 和newFixedThreadPool创建的线程池不同，newCachedThreadPool在没有任务执行时，当线程的空闲时间超过keepAliveTime，会自动释放线程资源，当提交新任务时，如果没有空闲线程，则创建新线程执行任务，会导致一定的系统开销；

所以，使用该线程池时，一定要注意控制并发的任务数，否则创建大量的线程可能导致严重的性能问题。

#### newFixedThreadPool: 

```
public static ExecutorService newFixedThreadPool(int nThreads){
  return new ThreadPoolExecutor(nThreads, nThreads,
  								0L, TimeUnit.SECONDS,
                                new LinkedBlockingQueue<Runnable>());
}
```

初始化一个指定线程数的线程池，其中corePoolSize == maximumPoolSize，使用LinkedBlockingQuene作为阻塞队列，不过当线程池没有可执行任务时，也不会释放线程;

#### newSingleThreadExecutor: 

```
public static ExecutorService newSingleThreadExecutor(ThreadFactory threadFactory){
  return new FinalizableDelegatedExecutorService(
  		 new ThreadPoolExecutor(1, 1,
  								0L, TimeUnit.SECONDS,
                                new LinkedBlockingQueue<Runnable>(),
                                threadFactory));

}
```

初始化的线程池中只有一个线程，如果该线程异常结束，会重新创建一个新的线程继续执行任务，唯一的线程可以保证所提交任务的顺序执行，内部使用LinkedBlockingQueue作为阻塞队列。

#### newScheduledThreadPool: 

```
public static ScheduledExecutorService newScheduledThreadPool(int corePoolSize){
  		return new ScheduledThreadPoolExecutor(corePoolSize);
}
```

初始化的线程池可以在指定的时间内周期性的执行所提交的任务，在实际的业务场景中可以使用该线程池定期的同步数据。

```
public static void main(String[] args) {
    ExecutorService executorService = Executors.newCachedThreadPool();
    for (int i = 0; i < 5; i++) {
        executorService.execute(new MyRunnable());
    }
    executorService.shutdown();
}
```

其本质是通过不同的参数初始化一个ThreadPoolExecutor对象，

#### 参数描述

**corePoolSize**

线程池中的核心线程数，当提交一个任务时，线程池创建一个新线程执行任务，直到当前线程数等于corePoolSize；如果当前线程数为corePoolSize，继续提交的任务被保存到阻塞队列中，等待被执行；如果执行了线程池的prestartAllCoreThreads()方法，线程池会提前创建并启动所有核心线程。

**maximumPoolSize**

线程池中允许的最大线程数。如果当前阻塞队列满了，且继续提交任务，则创建新的线程执行任务，前提是当前线程数小于maximumPoolSize；

**keepAliveTime**

线程空闲时的存活时间，即当线程没有任务执行时，继续存活的时间；默认情况下，该参数只在线程数大于corePoolSize时才有用；

**unit**

keepAliveTime的单位；

**workQueue**

用来保存等待被执行的任务的阻塞队列，且任务必须实现Runable接口，在JDK中提供了如下阻塞队列：

1. ArrayBlockingQueue：基于数组结构的有界阻塞队列，按FIFO排序任务；
2. LinkedBlockingQuene：基于链表结构的阻塞队列，按FIFO排序任务，吞吐量通常要高于ArrayBlockingQuene；
3. SynchronousQuene：一个不存储元素的阻塞队列，每个插入操作必须等到另一个线程调用移除操作，否则插入操作一直处于阻塞状态，吞吐量通常要高于LinkedBlockingQuene；
4. priorityBlockingQuene：具有优先级的无界阻塞队列；

**threadFactory**

创建线程的工厂，通过自定义的线程工厂可以给每个新建的线程设置一个具有识别度的线程名。

**handler**

线程池的饱和策略，当阻塞队列满了，且没有空闲的工作线程，如果继续提交任务，必须采取一种策略处理该任务，线程池提供了4种策略：

1. AbortPolicy：直接抛出异常，默认策略；
2. CallerRunsPolicy：用调用者所在的线程来执行任务；
3. DiscardOldestPolicy：丢弃阻塞队列中靠最前的任务，并执行当前任务；
4. DiscardPolicy：直接丢弃任务；

当然也可以根据应用场景实现RejectedExecutionHandler接口，自定义饱和策略，如记录日志或持久化存储不能处理的任务。

**实现原理**

除了newScheduledThreadPool的内部实现特殊一点之外，其它几个线程池都是基于ThreadPoolExecutor类实现的。

#### 线程池内部状态

![img](https://upload-images.jianshu.io/upload_images/2184951-5a620e0f56cbb008.png?raw=true?imageMogr2/auto-orient/strip%7CimageView2/2/w/684/format/webp)

其中AtomicInteger变量ctl的功能非常强大：利用低29位表示线程池中线程数，通过高3位表示线程池的运行状态：
1、RUNNING：`-1 << COUNT_BITS`，即高3位为111，该状态的线程池会接收新任务，并处理阻塞队列中的任务；
2、SHUTDOWN： `0 << COUNT_BITS`，即高3位为000，该状态的线程池不会接收新任务，但会处理阻塞队列中的任务；
3、STOP ： `1 << COUNT_BITS`，即高3位为001，该状态的线程不会接收新任务，也不会处理阻塞队列中的任务，而且会中断正在运行的任务；
4、TIDYING ： `2 << COUNT_BITS`，即高3位为010；
5、TERMINATED： `3 << COUNT_BITS`，即高3位为011；

#### 任务提交

线程池框架提供了两种方式提交任务，根据不同的业务需求选择不同的方式。

**Executor.execute()**

![img](https://upload-images.jianshu.io/upload_images/2184951-834971c24d085d31.png?raw=true?imageMogr2/auto-orient/strip%7CimageView2/2/w/735/format/webp)

通过Executor.execute()方法提交的任务，必须实现Runnable接口，该方式提交的任务不能获取返回值，因此无法判断任务是否执行成功。

**ExecutorService.submit()**

![img](https://upload-images.jianshu.io/upload_images/2184951-ea9fe289ca3de89f.png?raw=true?imageMogr2/auto-orient/strip%7CimageView2/2/w/702/format/webp)

通过ExecutorService.submit()方法提交的任务，可以获取任务执行完的返回值。

#### 任务执行

当向线程池中提交一个任务，线程池会如何处理该任务？

##### execute实现

![img](https://upload-images.jianshu.io/upload_images/2184951-7b6f0840be17799d.png?raw=true?imageMogr2/auto-orient/strip%7CimageView2/2/w/650/format/webp)

具体的执行流程如下：

1. workerCountOf方法根据ctl的低29位，得到线程池的当前线程数，如果线程数小于corePoolSize，则执行addWorker方法创建新的线程执行任务；否则执行步骤（2）；
2. 如果线程池处于RUNNING状态，且把提交的任务成功放入阻塞队列中，则执行步骤（3），否则执行步骤（4）；
3. 再次检查线程池的状态，如果线程池没有RUNNING，且成功从阻塞队列中删除任务，则执行reject方法处理任务；
4. 执行addWorker方法创建新的线程执行任务，如果addWoker执行失败，则执行reject方法处理任务；

![img](https://upload-images.jianshu.io/upload_images/2184951-18c425ebd3877453.png?raw=true?imageMogr2/auto-orient/strip%7CimageView2/2/w/639/format/webp)

##### addWorker实现

从方法execute的实现可以看出：addWorker主要负责创建新的线程并执行任务，代码实现如下：

![img](https://upload-images.jianshu.io/upload_images/2184951-93ee534ab66761cc.png?raw=true?imageMogr2/auto-orient/strip%7CimageView2/2/w/690/format/webp)

这只是addWoker方法实现的前半部分：

1. 判断线程池的状态，如果线程池的状态值大于或等SHUTDOWN，则不处理提交的任务，直接返回；
2. 通过参数core判断当前需要创建的线程是否为核心线程，如果core为true，且当前线程数小于corePoolSize，则跳出循环，开始创建新的线程，具体实现如下：

![img](https://upload-images.jianshu.io/upload_images/2184951-b36984b791e99464.png?raw=true?imageMogr2/auto-orient/strip%7CimageView2/2/w/746/format/webp)

线程池的工作线程通过Woker类实现，在ReentrantLock锁的保证下，把Woker实例插入到HashSet后，并启动Woker中的线程，其中Worker类设计如下：

1. 继承了AQS类，可以方便的实现工作线程的中止操作；
2. 实现了Runnable接口，可以将自身作为一个任务在工作线程中执行；
3. 当前提交的任务firstTask作为参数传入Worker的构造方法；

![img](https://upload-images.jianshu.io/upload_images/2184951-61b08d34ae1aaf49.png?raw=true?imageMogr2/auto-orient/strip%7CimageView2/2/w/679/format/webp)

从Woker类的构造方法实现可以发现：线程工厂在创建线程thread时，将Woker实例本身this作为参数传入，当执行start方法启动线程thread时，本质是执行了Worker的runWorker方法。

##### runWorker实现

![img](https://upload-images.jianshu.io/upload_images/2184951-1e8ed00138c189ea.png?raw=true?imageMogr2/auto-orient/strip%7CimageView2/2/w/715/format/webp)

runWorker方法是线程池的核心：

1. 线程启动之后，通过unlock方法释放锁，设置AQS的state为0，表示运行中断；
2. 获取第一个任务firstTask，执行任务的run方法，不过在执行任务之前，会进行加锁操作，任务执行完会释放锁；
3. 在执行任务的前后，可以根据业务场景自定义beforeExecute和afterExecute方法；
4. firstTask执行完成之后，通过getTask方法从阻塞队列中获取等待的任务，如果队列中没有任务，getTask方法会被阻塞并挂起，不会占用cpu资源；

##### getTask实现

![img](https://upload-images.jianshu.io/upload_images/2184951-a63a6646c456f715.png?raw=true?imageMogr2/auto-orient/strip%7CimageView2/2/w/738/format/webp)

整个getTask操作在自旋下完成：

1. workQueue.take：如果阻塞队列为空，当前线程会被挂起等待；当队列中有任务加入时，线程被唤醒，take方法返回任务，并执行；
2. workQueue.poll：如果在keepAliveTime时间内，阻塞队列还是没有任务，则返回null；

所以，线程池中实现的线程可以一直执行由用户提交的任务。

### 线程池其他常用方法

如果执行了线程池的 prestartAllCoreThreads() 方法，线程池会提前创建并启动所有核心线程。 ThreadPoolExecutor 提供了动态调整线程池容量大小的方法：setCorePoolSize() 和 setMaximumPoolSize()。

### 合理设置线程池的大小

一般需要根据任务的类型来配置线程池大小： 如果是 CPU 密集型任务，就需要尽量压榨 CPU，参考值可以设为 NCPU+1 如果是 IO 密集型任务，参考值可以设置为 2*NCPU

## Daemon

守护线程是程序运行时在后台提供服务的线程，不属于程序中不可或缺的部分。

当所有非守护线程结束时，程序也就终止，同时会杀死所有守护线程。

main() 属于非守护线程。

使用 setDaemon() 方法将一个线程设置为守护线程。

```
public static void main(String[] args) {
    Thread thread = new Thread(new MyRunnable());
    thread.setDaemon(true);
}
```

**另外有几点需要注意：**

- setDaemon(true) 必须在调用线程的 start() 方法之前设置，否则会跑出 IllegalThreadStateException 异常。
- 在守护线程中产生的新线程也是守护线程。
- 不要认为所有的应用都可以分配给守护线程来进行服务，比如读写操作或者计算逻辑。

## sleep()

Thread.sleep(millisec) 方法会休眠当前正在执行的线程，millisec 单位为毫秒。

sleep() 可能会抛出 InterruptedException，因为异常不能跨线程传播回 main() 中，因此必须在本地进行处理。线程中抛出的其它异常也同样需要在本地进行处理。

```
public void run() {
    try {
        Thread.sleep(3000);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
}
```

## yield()

对静态方法 Thread.yield() 的调用声明了当前线程已经完成了生命周期中最重要的部分，可以切换给其它线程来执行。该方法只是对线程调度器的一个建议，而且也只是建议具有相同优先级的其它线程可以运行。

```
public void run() {
    Thread.yield();
}
```

# 线程之间的协作

当多个线程可以一起工作去解决某个问题时，如果某些部分必须在其它部分之前完成，那么就需要对线程进行协调。

## join()

在线程中调用另一个线程的 join() 方法，会将当前线程挂起，而不是忙等待，直到目标线程结束。

对于以下代码，虽然 b 线程先启动，但是因为在 b 线程中调用了 a 线程的 join() 方法，b 线程会等待 a 线程结束才继续执行，因此最后能够保证 a 线程的输出先于 b 线程的输出。

```
public class JoinExample {

    private class A extends Thread {
        @Override
        public void run() {
            System.out.println("A");
        }
    }

    private class B extends Thread {

        private A a;

        B(A a) {
            this.a = a;
        }

        @Override
        public void run() {
            try {
                a.join();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("B");
        }
    }

    public void test() {
        A a = new A();
        B b = new B(a);
        b.start();
        a.start();
    }
}
```

```
public static void main(String[] args) {
    JoinExample example = new JoinExample();
    example.test();
}
```

```
A
B

```

## wait() notify() notifyAll()

调用 wait() 使得线程等待某个条件满足，线程在等待时会被挂起，当其他线程的运行使得这个条件满足时，其它线程会调用 notify() 或者 notifyAll() 来唤醒挂起的线程。

它们都属于 Object 的一部分，而不属于 Thread。

只能用在同步方法或者同步控制块中使用，否则会在运行时抛出 IllegalMonitorStateException。

使用 wait() 挂起期间，线程会释放锁。这是因为，如果没有释放锁，那么其它线程就无法进入对象的同步方法或者同步控制块中，那么就无法执行 notify() 或者 notifyAll() 来唤醒挂起的线程，造成死锁。

```
public class WaitNotifyExample {

    public synchronized void before() {
        System.out.println("before");
        notifyAll();
    }

    public synchronized void after() {
        try {
            wait();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("after");
    }
}
```

```
public static void main(String[] args) {
    ExecutorService executorService = Executors.newCachedThreadPool();
    WaitNotifyExample example = new WaitNotifyExample();
    executorService.execute(() -> example.after());
    executorService.execute(() -> example.before());
}
```

```
before
after
```

**wait() 和 sleep() 的区别**

- wait() 是 Object 的方法，而 sleep() 是 Thread 的静态方法；
- wait() 会释放锁，sleep() 不会。

## await() signal() signalAll()

java.util.concurrent 类库中提供了 Condition 类来实现线程之间的协调，可以在 Condition 上调用 await() 方法使线程等待，其它线程调用 signal() 或 signalAll() 方法唤醒等待的线程。

相比于 wait() 这种等待方式，await() 可以指定等待的条件，因此更加灵活。

使用 Lock 来获取一个 Condition 对象。

```
public class AwaitSignalExample {

    private Lock lock = new ReentrantLock();
    private Condition condition = lock.newCondition();

    public void before() {
        lock.lock();
        try {
            System.out.println("before");
            condition.signalAll();
        } finally {
            lock.unlock();
        }
    }

    public void after() {
        lock.lock();
        try {
            condition.await();
            System.out.println("after");
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }
}
public static void main(String[] args) {
    ExecutorService executorService = Executors.newCachedThreadPool();
    AwaitSignalExample example = new AwaitSignalExample();
    executorService.execute(() -> example.after());
    executorService.execute(() -> example.before());
}
before
after
```

# 中断

一个线程执行完毕之后会自动结束，如果在运行过程中发生异常也会提前结束。

## InterruptedException

通过调用一个线程的 interrupt() 来中断该线程，如果该线程处于阻塞、限期等待或者无限期等待状态，那么就会抛出 InterruptedException，从而提前结束该线程。但是不能中断 I/O 阻塞和 synchronized 锁阻塞。

对于以下代码，在 main() 中启动一个线程之后再中断它，由于线程中调用了 Thread.sleep() 方法，因此会抛出一个 InterruptedException，从而提前结束线程，不执行之后的语句。

```
public class InterruptExample {

    private static class MyThread1 extends Thread {
        @Override
        public void run() {
            try {
                Thread.sleep(2000);
                System.out.println("Thread run");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}
```

```
public static void main(String[] args) throws InterruptedException {
    Thread thread1 = new MyThread1();
    thread1.start();
    thread1.interrupt();
    System.out.println("Main run");
}
```

```
Main run
java.lang.InterruptedException: sleep interrupted
    at java.lang.Thread.sleep(Native Method)
    at InterruptExample.lambda$main$0(InterruptExample.java:5)
    at InterruptExample$$Lambda$1/713338599.run(Unknown Source)
    at java.lang.Thread.run(Thread.java:745)
```

## interrupted()

如果一个线程的 run() 方法执行一个无限循环，并且没有执行 sleep() 等会抛出 InterruptedException 的操作，那么调用线程的 interrupt() 方法就无法使线程提前结束。

但是调用 interrupt() 方法会设置线程的中断标记，此时调用 interrupted() 方法会返回 true。因此可以在循环体中使用 interrupted() 方法来判断线程是否处于中断状态，从而提前结束线程。

```
public class InterruptExample {

    private static class MyThread2 extends Thread {
        @Override
        public void run() {
            while (!interrupted()) {
                // ..
            }
            System.out.println("Thread end");
        }
    }
}
```

```
public static void main(String[] args) throws InterruptedException {
    Thread thread2 = new MyThread2();
    thread2.start();
    thread2.interrupt();
}
```

```
Thread end
```

## Executor 的中断操作

调用 Executor 的 shutdown() 方法会等待线程都执行完毕之后再关闭，但是如果调用的是 shutdownNow() 方法，则相当于调用每个线程的 interrupt() 方法。

以下使用 Lambda 创建线程，相当于创建了一个匿名内部线程。

```
public static void main(String[] args) {
    ExecutorService executorService = Executors.newCachedThreadPool();
    executorService.execute(() -> {
        try {
            Thread.sleep(2000);
            System.out.println("Thread run");
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    });
    executorService.shutdownNow();
    System.out.println("Main run");
}
```

```
Main run
java.lang.InterruptedException: sleep interrupted
    at java.lang.Thread.sleep(Native Method)
    at ExecutorInterruptExample.lambda$main$0(ExecutorInterruptExample.java:9)
    at ExecutorInterruptExample$$Lambda$1/1160460865.run(Unknown Source)
    at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1142)
    at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:617)
    at java.lang.Thread.run(Thread.java:745)
```

如果只想中断 Executor 中的一个线程，可以通过使用 submit() 方法来提交一个线程，它会返回一个 Future<?> 对象，通过调用该对象的 cancel(true) 方法就可以中断线程。

```
Future<?> future = executorService.submit(() -> {
    // ..
});
future.cancel(true);
```

# 线程阻塞

线程可以阻塞于四种状态：

- 当线程执行 Thread.sleep() 时，它一直阻塞到指定的毫秒时间之后，或者阻塞被另一个线程打断；
- 当线程碰到一条 wait() 语句时，它会一直阻塞到接到通知 notify()、被中断或经过了指定毫秒时间为止（若制定了超时值的话）
- 线程阻塞与不同 I/O 的方式有多种。常见的一种方式是 InputStream 的 read() 方法，该方法一直阻塞到从流中读取一个字节的数据为止，它可以无限阻塞，因此不能指定超时时间；
- 线程也可以阻塞等待获取某个对象锁的排他性访问权限（即等待获得 synchronized 语句必须的锁时阻塞）。

> 注意，并非所有的阻塞状态都是可中断的，以上阻塞状态的前两种可以被中断，后两种不会对中断做出反应

# J.U.C - AQS

java.util.concurrent（J.U.C）大大提高了并发性能，AQS 被认为是 J.U.C 的核心。

## CountdownLatch

用来控制一个线程等待多个线程。

维护了一个计数器 cnt，每次调用 countDown() 方法会让计数器的值减 1，减到 0 的时候，那些因为调用 await() 方法而在等待的线程就会被唤醒。

[![img](https://github.com/orangehaswing/OrdinaryNote/blob/master/Java%E5%B9%B6%E5%8F%91/resource/CountdownLatch.png?raw=true)](https://github.com/CyC2018/CS-Notes/blob/master/pics/CountdownLatch.png?raw=true)

```
public class CountdownLatchExample {

    public static void main(String[] args) throws InterruptedException {
        final int totalThread = 10;
        CountDownLatch countDownLatch = new CountDownLatch(totalThread);
        ExecutorService executorService = Executors.newCachedThreadPool();
        for (int i = 0; i < totalThread; i++) {
            executorService.execute(() -> {
                System.out.print("run..");
                countDownLatch.countDown();
            });
        }
        countDownLatch.await();
        System.out.println("end");
        executorService.shutdown();
    }
}
```

```
run..run..run..run..run..run..run..run..run..run..end
```

## CyclicBarrier

用来控制多个线程互相等待，只有当多个线程都到达时，这些线程才会继续执行。

和 CountdownLatch 相似，都是通过维护计数器来实现的。线程执行 await() 方法之后计数器会减 1，并进行等待，直到计数器为 0，所有调用 await() 方法而在等待的线程才能继续执行。

CyclicBarrier 和 CountdownLatch 的一个区别是，CyclicBarrier 的计数器通过调用 reset() 方法可以循环使用，所以它才叫做循环屏障。

CyclicBarrier 有两个构造函数，其中 parties 指示计数器的初始值，barrierAction 在所有线程都到达屏障的时候会执行一次。

```
public CyclicBarrier(int parties, Runnable barrierAction) {
    if (parties <= 0) throw new IllegalArgumentException();
    this.parties = parties;
    this.count = parties;
    this.barrierCommand = barrierAction;
}

public CyclicBarrier(int parties) {
    this(parties, null);
}
```

[![img](https://github.com/orangehaswing/OrdinaryNote/blob/master/Java%E5%B9%B6%E5%8F%91/resource/CyclicBarrier.png?raw=true)](https://github.com/CyC2018/CS-Notes/blob/master/pics/CyclicBarrier.png?raw=true)

```
public class CyclicBarrierExample {

    public static void main(String[] args) {
        final int totalThread = 10;
        CyclicBarrier cyclicBarrier = new CyclicBarrier(totalThread);
        ExecutorService executorService = Executors.newCachedThreadPool();
        for (int i = 0; i < totalThread; i++) {
            executorService.execute(() -> {
                System.out.print("before..");
                try {
                    cyclicBarrier.await();
                } catch (InterruptedException | BrokenBarrierException e) {
                    e.printStackTrace();
                }
                System.out.print("after..");
            });
        }
        executorService.shutdown();
    }
}
```

```
before..before..before..before..before..before..before..before..before..before..after..after..after..after..after..after..after..after..after..after..
```

## Semaphore

Semaphore 类似于操作系统中的信号量，可以控制对互斥资源的访问线程数。

[![img](https://github.com/orangehaswing/OrdinaryNote/blob/master/Java%E5%B9%B6%E5%8F%91/resource/Semaphore.png?raw=true)](https://github.com/CyC2018/CS-Notes/blob/master/pics/Semaphore.png?raw=true)

以下代码模拟了对某个服务的并发请求，每次只能有 3 个客户端同时访问，请求总数为 10。

```
public class SemaphoreExample {

    public static void main(String[] args) {
        final int clientCount = 3;
        final int totalRequestCount = 10;
        Semaphore semaphore = new Semaphore(clientCount);
        ExecutorService executorService = Executors.newCachedThreadPool();
        for (int i = 0; i < totalRequestCount; i++) {
            executorService.execute(()->{
                try {
                    semaphore.acquire();
                    System.out.print(semaphore.availablePermits() + " ");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } finally {
                    semaphore.release();
                }
            });
        }
        executorService.shutdown();
    }
}
```

```
2 1 2 2 2 2 2 1 2 2
```

# J.U.C - 其它组件

## FutureTask

在介绍 Callable 时我们知道它可以有返回值，返回值通过 Future 进行封装。FutureTask 实现了 RunnableFuture 接口，该接口继承自 Runnable 和 Future 接口，这使得 FutureTask 既可以当做一个任务执行，也可以有返回值。

```
public class FutureTask<V> implements RunnableFuture<V>
```

```
public interface RunnableFuture<V> extends Runnable, Future<V>
```

FutureTask 可用于异步获取执行结果或取消执行任务的场景。当一个计算任务需要执行很长时间，那么就可以用 FutureTask 来封装这个任务，主线程在完成自己的任务之后再去获取结果。

```
public class FutureTaskExample {

    public static void main(String[] args) throws ExecutionException, InterruptedException {
        FutureTask<Integer> futureTask = new FutureTask<Integer>(new Callable<Integer>() {
            @Override
            public Integer call() throws Exception {
                int result = 0;
                for (int i = 0; i < 100; i++) {
                    Thread.sleep(10);
                    result += i;
                }
                return result;
            }
        });

        Thread computeThread = new Thread(futureTask);
        computeThread.start();

        Thread otherThread = new Thread(() -> {
            System.out.println("other task is running...");
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });
        otherThread.start();
        System.out.println(futureTask.get());
    }
}
```

```
other task is running...
4950
```

## 阻塞队列（BlockingQueue）

阻塞队列（BlockingQueue）是一个支持两个附加操作的队列。这两个附加的操作是：在队列为空时，获取元素的线程会等待队列变为非空。当队列满时，存储元素的线程会等待队列可用。

阻塞队列提供了四种处理方法:

| 方法\处理方式 | 抛出异常      | 返回特殊值    | 一直阻塞   | 超时退出               |
| ------- | --------- | -------- | ------ | ------------------ |
| 插入方法    | add(e)    | offer(e) | put(e) | offer(e,time,unit) |
| 移除方法    | remove()  | poll()   | take() | poll(time,unit)    |
| 检查方法    | element() | peek()   | 不可用    | 不可用                |

- 抛出异常：是指当阻塞队列满时候，再往队列里插入元素，会抛出 IllegalStateException("Queue full") 异常。当队列为空时，从队列里获取元素时会抛出 NoSuchElementException 异常 。
- 返回特殊值：插入方法会返回是否成功，成功则返回 true。移除方法，则是从队列里拿出一个元素，如果没有则返回 null
- 一直阻塞：当阻塞队列满时，如果生产者线程往队列里 put 元素，队列会一直阻塞生产者线程，直到拿到数据，或者响应中断退出。当队列空时，消费者线程试图从队列里 take 元素，队列也会阻塞消费者线程，直到队列可用。
- 超时退出：当阻塞队列满时，队列会阻塞生产者线程一段时间，如果超过一定的时间，生产者线程就会退出。

JDK8 提供了 7 个阻塞队列：

- ArrayBlockingQueue ：一个由数组结构组成的有界阻塞队列  [源码](https://www.jianshu.com/p/0a0b58934401)
- LinkedBlockingQueue ：一个由链表结构组成的有界阻塞队列  [源码](https://www.jianshu.com/p/f1b2c053c103)
- PriorityBlockingQueue ：一个支持优先级排序的无界阻塞队列  [源码](https://www.jianshu.com/p/43954715aa28)
- DelayQueue：一个使用优先级队列实现的无界阻塞队列。
- SynchronousQueue：一个不存储元素的阻塞队列  [源码](https://www.jianshu.com/p/9d2c706e45b7)
- LinkedTransferQueue：一个由链表结构组成的无界阻塞队列  [源码](https://www.jianshu.com/p/bd708cb3ea91)
- LinkedBlockingDeque：一个由链表结构组成的双向阻塞队列

这是[详解](https://www.jianshu.com/p/4af8ab00c587)

**使用 BlockingQueue 实现生产者消费者问题**

```
public class ProducerConsumer {

    private static BlockingQueue<String> queue = new ArrayBlockingQueue<>(5);

    private static class Producer extends Thread {
        @Override
        public void run() {
            try {
                queue.put("product");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.print("produce..");
        }
    }

    private static class Consumer extends Thread {

        @Override
        public void run() {
            try {
                String product = queue.take();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.print("consume..");
        }
    }
}
```

```
public static void main(String[] args) {
    for (int i = 0; i < 2; i++) {
        Producer producer = new Producer();
        producer.start();
    }
    for (int i = 0; i < 5; i++) {
        Consumer consumer = new Consumer();
        consumer.start();
    }
    for (int i = 0; i < 3; i++) {
        Producer producer = new Producer();
        producer.start();
    }
}
```

```
produce..produce..consume..consume..produce..consume..produce..consume..produce..consume..
```

## ForkJoin

主要用于并行计算中，和 MapReduce 原理类似，都是把大的计算任务拆分成多个小任务并行计算。

```
public class ForkJoinExample extends RecursiveTask<Integer> {

    private final int threshold = 5;
    private int first;
    private int last;

    public ForkJoinExample(int first, int last) {
        this.first = first;
        this.last = last;
    }

    @Override
    protected Integer compute() {
        int result = 0;
        if (last - first <= threshold) {
            // 任务足够小则直接计算
            for (int i = first; i <= last; i++) {
                result += i;
            }
        } else {
            // 拆分成小任务
            int middle = first + (last - first) / 2;
            ForkJoinExample leftTask = new ForkJoinExample(first, middle);
            ForkJoinExample rightTask = new ForkJoinExample(middle + 1, last);
            leftTask.fork();
            rightTask.fork();
            result = leftTask.join() + rightTask.join();
        }
        return result;
    }
}
```

```
public static void main(String[] args) throws ExecutionException, InterruptedException {
    ForkJoinExample example = new ForkJoinExample(1, 10000);
    ForkJoinPool forkJoinPool = new ForkJoinPool();
    Future result = forkJoinPool.submit(example);
    System.out.println(result.get());
}
```

ForkJoin 使用 ForkJoinPool 来启动，它是一个特殊的线程池，线程数量取决于 CPU 核数。

```
public class ForkJoinPool extends AbstractExecutorService
```

ForkJoinPool 实现了工作窃取算法来提高 CPU 的利用率。每个线程都维护了一个双端队列，用来存储需要执行的任务。工作窃取算法允许空闲的线程从其它线程的双端队列中窃取一个任务来执行。窃取的任务必须是最晚的任务，避免和队列所属线程发生竞争。例如下图中，Thread2 从 Thread1 的队列中拿出最晚的 Task1 任务，Thread1 会拿出 Task2 来执行，这样就避免发生竞争。但是如果队列中只有一个任务时还是会发生竞争。

[![img](https://github.com/orangehaswing/OrdinaryNote/blob/master/Java%E5%B9%B6%E5%8F%91/resource/15b45dc6-27aa-4519-9194-f4acfa2b077f.jpg?raw=true)](https://github.com/CyC2018/CS-Notes/blob/master/pics/15b45dc6-27aa-4519-9194-f4acfa2b077f.jpg?raw=true)

# 互斥同步

Java 提供了两种锁机制来控制多个线程对共享资源的互斥访问，第一个是 JVM 实现的 synchronized，而另一个是 JDK 实现的 ReentrantLock。

## 对象头和monitor

**对象头**

在hotspot虚拟机中，对象在内存的分布分为3个部分：

- 对象头
- 实例数据
- 对齐填充。

synchronized用的锁是存在Java对象头里的，对象头主要包括两部分数据：Mark Word（标记字段）、Klass Pointer（类型指针）。Mark Word用于存储对象自身的运行时数据，如哈希码（HashCode）、GC分代年龄、锁状态标志、线程持有的锁、偏向线程 ID、偏向时间戳等等

下图是Java对象头的存储结构（32位虚拟机）：

|    25Bit    |  4Bit   |  1Bit  | 2Bit |
| :---------: | :-----: | :----: | :--: |
| 对象的hashcode | 对象的分代年龄 | 是否是偏向锁 | 锁标志位 |

mark word 被设计为非固定的数据结构，以便在及小的空间内存储更多的信息。

比如：在32位的hotspot虚拟机中：如果对象处于未被锁定的情况下。mark  word 的32bit空间中有25bit存储对象的哈希码、4bit存储对象的分代年龄、2bit存储锁的标记位、1bit固定为0。而在其他的状态下（轻量级锁、重量级锁、GC标记、可偏向）下对象的存储结构为

![img](https://upload-images.jianshu.io/upload_images/2184951-96c64ed6c9f3316e.png?raw=true?imageMogr2/auto-orient/strip%7CimageView2/2/w/879/format/webp)

**monitor**

monitor，把它理解为一个同步工具，也可以描述为一种同步机制，它通常被描述为一个对象。



![img](https://upload-images.jianshu.io/upload_images/2184951-c1fc7a8eee6d5d64.png?raw=true?imageMogr2/auto-orient/strip%7CimageView2/2/w/347/format/webp)

- Owner：初始时为NULL表示当前没有任何线程拥有该monitor，当线程成功拥有该锁后保存线程唯一标识，当锁被释放时又设置为NULL；
- EntryQ：关联一个系统互斥锁（semaphore），阻塞所有试图锁住monitor失败的线程。
- RcThis：表示blocked或waiting在该monitor上的所有线程的个数。
- Nest：用来实现重入锁的计数。
- HashCode：保存从对象头拷贝过来的HashCode值（可能还包含GC age）。
- Candidate：用来避免不必要的阻塞或等待线程唤醒，因为每一次只有一个线程能够成功拥有锁，如果每次前一个释放锁的线程唤醒所有正在阻塞或等待的线程，会引起不必要的上下文切换（从阻塞到就绪然后因为竞争锁失败又被阻塞）从而导致性能严重下降。Candidate只有两种可能的值：0表示没有需要唤醒的线程，1表示要唤醒一个继任线程来竞争锁。

那么monitor的作用是什么呢？在 java 虚拟机中，线程一旦进入到被synchronized修饰的方法或代码块时，指定的锁对象通过某些操作将对象头中的LockWord指向monitor 的起始地址与之关联，同时monitor 中的Owner存放拥有该锁的线程的唯一标识，确保一次只能有一个线程执行该部分的代码，线程在获取锁之前不允许执行该部分的代码。

## synchronized

1. 同步代码块，锁是括号里面的对象
2. 普通同步方法，锁是当前实例对象
3. 同步类，锁的是 对象.class
4. 静态同步方法，锁是当前类的class对象

**1. 同步一个代码块**

```
public void func() {
    synchronized (this) {
        // ...
    }
}
```

它只作用于同一个对象，如果调用两个对象上的同步代码块，就不会进行同步。

对于以下代码，使用 ExecutorService 执行了两个线程，由于调用的是同一个对象的同步代码块，因此这两个线程会进行同步，当一个线程进入同步语句块时，另一个线程就必须等待。

```
public class SynchronizedExample {

    public void func1() {
        synchronized (this) {
            for (int i = 0; i < 10; i++) {
                System.out.print(i + " ");
            }
        }
    }
}
```

```
public static void main(String[] args) {
    SynchronizedExample e1 = new SynchronizedExample();
    ExecutorService executorService = Executors.newCachedThreadPool();
    executorService.execute(() -> e1.func1());
    executorService.execute(() -> e1.func1());
}
```

```
0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9
```

对于以下代码，两个线程调用了不同对象的同步代码块，因此这两个线程就不需要同步。从输出结果可以看出，两个线程交叉执行。

```
public static void main(String[] args) {
    SynchronizedExample e1 = new SynchronizedExample();
    SynchronizedExample e2 = new SynchronizedExample();
    ExecutorService executorService = Executors.newCachedThreadPool();
    executorService.execute(() -> e1.func1());
    executorService.execute(() -> e2.func1());
}
```

```
0 0 1 1 2 2 3 3 4 4 5 5 6 6 7 7 8 8 9 9
```

**2. 同步一个方法**

```
public synchronized void func () {
    // ...
}
```

它和同步代码块一样，作用于同一个对象。

**3. 同步一个类**

```
public void func() {
    synchronized (SynchronizedExample.class) {
        // ...
    }
}
```

作用于整个类，也就是说两个线程调用同一个类的不同对象上的这种同步语句，也会进行同步。

```
public class SynchronizedExample {

    public void func2() {
        synchronized (SynchronizedExample.class) {
            for (int i = 0; i < 10; i++) {
                System.out.print(i + " ");
            }
        }
    }
}
```

```
public static void main(String[] args) {
    SynchronizedExample e1 = new SynchronizedExample();
    SynchronizedExample e2 = new SynchronizedExample();
    ExecutorService executorService = Executors.newCachedThreadPool();
    executorService.execute(() -> e1.func2());
    executorService.execute(() -> e2.func2());
}
```

```
0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9
```

**4. 同步一个静态方法**

```
public synchronized static void fun() {
    // ...
}
```

作用于整个类。

**实现原理**：

```
public class SynchronizedTest {
    private static Object object = new Object();
    public static void main(String[] args) throws Exception{
        synchronized(object) {
            
        }
    }
    public static synchronized void m() {}
}
```

上述代码中，使用了同步代码块和同步方法，通过使用javap工具查看生成的class文件信息来分析synchronized关键字的实现细节。

```
public static void main(java.lang.String[]) throws java.lang.Exception;
    descriptor: ([Ljava/lang/String;)V
    flags: ACC_PUBLIC, ACC_STATIC
    Code:
      stack=2, locals=3, args_size=1
         0: getstatic     #2                  // Field object:Ljava/lang/Object

         3: dup
         4: astore_1
         5: monitorenter            //监视器进入，获取锁
         6: aload_1
         7: monitorexit              //监视器退出，释放锁
         8: goto          16
        11: astore_2
        12: aload_1
        13: monitorexit
        14: aload_2
        15: athrow
        16: return
     
   public static synchronized void m();
   descriptor: ()V
   flags: ACC_PUBLIC, ACC_STATIC, ACC_SYNCHRONIZED
   Code:
     stack=0, locals=0, args_size=0
        0: return
     LineNumberTable:
       line 9: 0
```

从生成的class信息中看到

1. 同步代码块使用了 **monitorenter** 和 **monitorexit** 指令实现。
2. 同步方法中依靠方法修饰符上的 **ACC_SYNCHRONIZED** 实现。

无论哪种实现，本质上都是对指定对象相关联的monitor的获取，这个过程是互斥性的，也就是说同一时刻只有一个线程能够成功，其它失败的线程会被阻塞，并放入到同步队列中，进入BLOCKED状态。

## ReentrantLock

ReentrantLock 是 java.util.concurrent（J.U.C）包中的锁。

```
public class LockExample {

    private Lock lock = new ReentrantLock();

    public void func() {
        lock.lock();
        try {
            for (int i = 0; i < 10; i++) {
                System.out.print(i + " ");
            }
        } finally {
            lock.unlock(); // 确保释放锁，从而避免发生死锁。
        }
    }
}
```

```
public static void main(String[] args) {
    LockExample lockExample = new LockExample();
    ExecutorService executorService = Executors.newCachedThreadPool();
    executorService.execute(() -> lockExample.func());
    executorService.execute(() -> lockExample.func());
}
```

```
0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9
```

这是[详解](https://www.jianshu.com/p/508412a6ffdc)

## ReadWriteLock

```
public interface ReadWriteLock {  
    Lock readLock();       //获取读锁  
    Lock writeLock();      //获取写锁  
}  
```

一个用来获取读锁，一个用来获取写锁。也就是说将文件的读写操作分开，分成2个锁来分配给线程，从而使得多个线程可以同时进行读操作。ReentrantReadWirteLock实现了ReadWirteLock接口，并未实现Lock接口。不过要注意的是：

如果有一个线程已经占用了读锁，则此时其他线程如果要申请写锁，则申请写锁的线程会一直等待释放读锁。如果有一个线程已经占用了写锁，则此时其他线程如果申请写锁或者读锁，则申请的线程会一直等待释放写锁。

## ReetrantReadWriteLock

ReetrantReadWriteLock同样支持公平性选择，支持重进入，锁降级。

```
public class RWLock {
    static Map<String, Object> map = new HashMap<String, Object>();
    static ReentrantReadWriteLock rwLock = new ReentrantReadWriteLock();
    static Lock r = rwLock.readLock();
    static Lock w = rwLock.writeLock();
    //读
    public static final Object get(String key){
        r.lock();
        try {
            return map.get(key);
        } finally {
            r.unlock();
        }
    }
    //写
    public static final Object put(String key, Object value){
        w.lock();
        try {
            return map.put(key, value);
        } finally {
            w.unlock();
        }
    }
}
```

只需在读操作时获取读锁，写操作时获取写锁。当写锁被获取时，后续的读写操作都会被阻塞，写锁释放后，所有操作继续执行。

这是[详解](https://www.jianshu.com/p/d47fe1ec1bb3)

## 比较

**1. 锁的实现**

synchronized 是 JVM 实现的，而 ReentrantLock 是 JDK 实现的。

**2. 性能**

新版本 Java 对 synchronized 进行了很多优化，例如自旋锁等，synchronized 与 ReentrantLock 大致相同。

**3. 等待可中断**

当持有锁的线程长期不释放锁的时候，正在等待的线程可以选择放弃等待，改为处理其他事情。

ReentrantLock 可中断，而 synchronized 不行。

**4. 公平锁**

公平锁是指多个线程在等待同一个锁时，必须按照申请锁的时间顺序来依次获得锁。

synchronized 中的锁是非公平的，ReentrantLock 默认情况下也是非公平的，但是也可以是公平的。

**5. 锁绑定多个条件**

一个 ReentrantLock 可以同时绑定多个 Condition 对象。

## 使用选择

除非需要使用 ReentrantLock 的高级功能，否则优先使用 synchronized。这是因为 synchronized 是 JVM 实现的一种锁机制，JVM 原生地支持它，而 ReentrantLock 不是所有的 JDK 版本都支持。并且使用 synchronized 不用担心没有释放锁而导致死锁问题，因为 JVM 会确保锁的释放。

## synchronized与lock

**区别，使用场景**

- （用法）synchronized（隐式锁）：在需要同步的对象中加入此控制，synchronized 可以加在方法上，也可以加在特定代码块中，括号中表示需要锁的对象。
- （用法）lock（显示锁）：需要显示指定起始位置和终止位置。一般使用 ReentrantLock 类做为锁，多个线程中必须要使用一个 ReentrantLock 类做为对象才能保证锁的生效。且在加锁和解锁处需要通过 lock() 和 unlock() 显示指出。所以一般会在 finally 块中写 unlock() 以防死锁。
- （性能）synchronized 是托管给 JVM 执行的，而 lock 是 Java 写的控制锁的代码。在 Java1.5 中，synchronize 是性能低效的。因为这是一个重量级操作，需要调用操作接口，导致有可能加锁消耗的系统时间比加锁以外的操作还多。相比之下使用 Java 提供的 Lock 对象，性能更高一些。但是到了 Java1.6 ，发生了变化。synchronize 在语义上很清晰，可以进行很多优化，有适应自旋，锁消除，锁粗化，轻量级锁，偏向锁等等。导致 在 Java1.6 上 synchronize 的性能并不比 Lock 差。
- （机制）**synchronized 原始采用的是 CPU 悲观锁机制，即线程获得的是独占锁**。独占锁意味着其他线程只能依靠阻塞来等待线程释放锁。**Lock 用的是乐观锁方式**。所谓乐观锁就是，每次不加锁而是假设没有冲突而去完成某项操作，如果因为冲突失败就重试，直到成功为止。乐观锁实现的机制就是 CAS 操作（Compare and Swap）。

# Java 内存模型

Java 内存模型试图屏蔽各种硬件和操作系统的内存访问差异，以实现让 Java 程序在各种平台下都能达到一致的内存访问效果。

## 主内存与工作内存

处理器上的寄存器的读写的速度比内存快几个数量级，为了解决这种速度矛盾，在它们之间加入了高速缓存。

加入高速缓存带来了一个新的问题：缓存一致性。如果多个缓存共享同一块主内存区域，那么多个缓存的数据可能会不一致，需要一些协议来解决这个问题。

[![img](https://github.com/orangehaswing/OrdinaryNote/blob/master/Java%E5%B9%B6%E5%8F%91/resource/68778c1b-15ab-4826-99c0-3b4fd38cb9e9.png?raw=true)](https://github.com/CyC2018/CS-Notes/blob/master/pics/68778c1b-15ab-4826-99c0-3b4fd38cb9e9.png?raw=true)

所有的变量都存储在主内存中，每个线程还有自己的工作内存，工作内存存储在高速缓存或者寄存器中，保存了该线程使用的变量的主内存副本拷贝。

线程只能直接操作工作内存中的变量，不同线程之间的变量值传递需要通过主内存来完成。

[![img](https://github.com/orangehaswing/OrdinaryNote/blob/master/Java%E5%B9%B6%E5%8F%91/resource/47358f87-bc4c-496f-9a90-8d696de94cee.png?raw=true)](https://github.com/CyC2018/CS-Notes/blob/master/pics/47358f87-bc4c-496f-9a90-8d696de94cee.png?raw=true)

## 内存间交互操作

Java 内存模型定义了 8 个操作来完成主内存和工作内存的交互操作。

[![img](https://github.com/orangehaswing/OrdinaryNote/blob/master/Java%E5%B9%B6%E5%8F%91/resource/536c6dfd-305a-4b95-b12c-28ca5e8aa043.png?raw=true)](https://github.com/CyC2018/CS-Notes/blob/master/pics/536c6dfd-305a-4b95-b12c-28ca5e8aa043.png?raw=true)

- read：把一个变量的值从主内存传输到工作内存中
- load：在 read 之后执行，把 read 得到的值放入工作内存的变量副本中
- use：把工作内存中一个变量的值传递给执行引擎
- assign：把一个从执行引擎接收到的值赋给工作内存的变量
- store：把工作内存的一个变量的值传送到主内存中
- write：在 store 之后执行，把 store 得到的值放入主内存的变量中
- lock：作用于主内存的变量
- unlock

## 内存模型三大特性

### 1. 原子性

Java 内存模型保证了 read、load、use、assign、store、write、lock 和 unlock 操作具有原子性，例如对一个 int 类型的变量执行 assign 赋值操作，这个操作就是原子性的。但是 Java 内存模型允许虚拟机将没有被 volatile 修饰的 64 位数据（long，double）的读写操作划分为两次 32 位的操作来进行，即 load、store、read 和 write 操作可以不具备原子性。

有一个错误认识就是，int 等原子性的类型在多线程环境中不会出现线程安全问题。前面的线程不安全示例代码中，cnt 属于 int 类型变量，1000 个线程对它进行自增操作之后，得到的值为 997 而不是 1000。

为了方便讨论，将内存间的交互操作简化为 3 个：load、assign、store。

下图演示了两个线程同时对 cnt 进行操作，load、assign、store 这一系列操作整体上看不具备原子性，那么在 T1 修改 cnt 并且还没有将修改后的值写入主内存，T2 依然可以读入旧值。可以看出，这两个线程虽然执行了两次自增运算，但是主内存中 cnt 的值最后为 1 而不是 2。因此对 int 类型读写操作满足原子性只是说明 load、assign、store 这些单个操作具备原子性。

[![img](https://github.com/orangehaswing/OrdinaryNote/blob/master/Java%E5%B9%B6%E5%8F%91/resource/ef8eab00-1d5e-4d99-a7c2-d6d68ea7fe92.png?raw=true)](https://github.com/CyC2018/CS-Notes/blob/master/pics/ef8eab00-1d5e-4d99-a7c2-d6d68ea7fe92.png?raw=true)

AtomicInteger 能保证多个线程修改的原子性。

[![img](https://github.com/orangehaswing/OrdinaryNote/blob/master/Java%E5%B9%B6%E5%8F%91/resource/952afa9a-458b-44ce-bba9-463e60162945.png?raw=true)](https://github.com/CyC2018/CS-Notes/blob/master/pics/952afa9a-458b-44ce-bba9-463e60162945.png?raw=true)

使用 AtomicInteger 重写之前线程不安全的代码之后得到以下线程安全实现：

```
public class AtomicExample {
    private AtomicInteger cnt = new AtomicInteger();

    public void add() {
        cnt.incrementAndGet();
    }

    public int get() {
        return cnt.get();
    }
}
```

```
public static void main(String[] args) throws InterruptedException {
    final int threadSize = 1000;
    AtomicExample example = new AtomicExample(); // 只修改这条语句
    final CountDownLatch countDownLatch = new CountDownLatch(threadSize);
    ExecutorService executorService = Executors.newCachedThreadPool();
    for (int i = 0; i < threadSize; i++) {
        executorService.execute(() -> {
            example.add();
            countDownLatch.countDown();
        });
    }
    countDownLatch.await();
    executorService.shutdown();
    System.out.println(example.get());
}
```

```
1000
```

除了使用原子类之外，也可以使用 synchronized 互斥锁来保证操作的原子性。它对应的内存间交互操作为：lock 和 unlock，在虚拟机实现上对应的字节码指令为 monitorenter 和 monitorexit。

```
public class AtomicSynchronizedExample {
    private int cnt = 0;

    public synchronized void add() {
        cnt++;
    }

    public synchronized int get() {
        return cnt;
    }
}
```

```
public static void main(String[] args) throws InterruptedException {
    final int threadSize = 1000;
    AtomicSynchronizedExample example = new AtomicSynchronizedExample();
    final CountDownLatch countDownLatch = new CountDownLatch(threadSize);
    ExecutorService executorService = Executors.newCachedThreadPool();
    for (int i = 0; i < threadSize; i++) {
        executorService.execute(() -> {
            example.add();
            countDownLatch.countDown();
        });
    }
    countDownLatch.await();
    executorService.shutdown();
    System.out.println(example.get());
}
```

```
1000
```

### 2. 可见性

可见性指当一个线程修改了共享变量的值，其它线程能够立即得知这个修改。Java 内存模型是通过在变量修改后将新值同步回主内存，在变量读取前从主内存刷新变量值来实现可见性的。

主要有有三种实现可见性的方式：

- volatile
- synchronized，对一个变量执行 unlock 操作之前，必须把变量值同步回主内存。
- final，被 final 关键字修饰的字段在构造器中一旦初始化完成，并且没有发生 this 逃逸（其它线程通过 this 引用访问到初始化了一半的对象），那么其它线程就能看见 final 字段的值。

对前面的线程不安全示例中的 cnt 变量使用 volatile 修饰，不能解决线程不安全问题，因为 volatile 并不能保证操作的原子性。

### 3. 有序性

有序性是指：在本线程内观察，所有操作都是有序的。在一个线程观察另一个线程，所有操作都是无序的，无序是因为发生了指令重排序。在 Java 内存模型中，允许编译器和处理器对指令进行重排序，重排序过程不会影响到单线程程序的执行，却会影响到多线程并发执行的正确性。

volatile 关键字通过添加内存屏障的方式来禁止指令重排，即重排序时不能把后面的指令放到内存屏障之前。

也可以通过 synchronized 来保证有序性，它保证每个时刻只有一个线程执行同步代码，相当于是让线程顺序执行同步代码。

## 先行发生原则

上面提到了可以用 volatile 和 synchronized 来保证有序性。除此之外，JVM 还规定了先行发生原则，让一个操作无需控制就能先于另一个操作完成。

### 1. 单一线程原则

> Single Thread rule

在一个线程内，在程序前面的操作先行发生于后面的操作。

[![img](https://github.com/orangehaswing/OrdinaryNote/blob/master/Java%E5%B9%B6%E5%8F%91/resource/single-thread-rule.png?raw=true)](https://github.com/CyC2018/CS-Notes/blob/master/pics/single-thread-rule.png?raw=true)

### 2. 管程锁定规则

> Monitor Lock Rule

一个 unlock 操作先行发生于后面对同一个锁的 lock 操作。

[![img](https://github.com/orangehaswing/OrdinaryNote/blob/master/Java%E5%B9%B6%E5%8F%91/resource/monitor-lock-rule.png?raw=true)](https://github.com/CyC2018/CS-Notes/blob/master/pics/monitor-lock-rule.png?raw=true)

### 3. volatile 变量规则

> Volatile Variable Rule

对一个 volatile 变量的写操作先行发生于后面对这个变量的读操作。

[![img](https://github.com/orangehaswing/OrdinaryNote/blob/master/Java%E5%B9%B6%E5%8F%91/resource/volatile-variable-rule.png?raw=true)](https://github.com/CyC2018/CS-Notes/blob/master/pics/volatile-variable-rule.png?raw=true)

### 4. 线程启动规则

> Thread Start Rule

Thread 对象的 start() 方法调用先行发生于此线程的每一个动作。

[![img](https://github.com/orangehaswing/OrdinaryNote/blob/master/Java%E5%B9%B6%E5%8F%91/resource/thread-start-rule.png?raw=true)](https://github.com/CyC2018/CS-Notes/blob/master/pics/thread-start-rule.png?raw=true)

### 5. 线程加入规则

> Thread Join Rule

Thread 对象的结束先行发生于 join() 方法返回。

[![img](https://github.com/orangehaswing/OrdinaryNote/blob/master/Java%E5%B9%B6%E5%8F%91/resource/thread-join-rule.png?raw=true)](https://github.com/CyC2018/CS-Notes/blob/master/pics/thread-join-rule.png?raw=true)

### 6. 线程中断规则

> Thread Interruption Rule

对线程 interrupt() 方法的调用先行发生于被中断线程的代码检测到中断事件的发生，可以通过 interrupted() 方法检测到是否有中断发生。

### 7. 对象终结规则

> Finalizer Rule

一个对象的初始化完成（构造函数执行结束）先行发生于它的 finalize() 方法的开始。

### 8. 传递性

> Transitivity

如果操作 A 先行发生于操作 B，操作 B 先行发生于操作 C，那么操作 A 先行发生于操作 C。

# 线程安全

多个线程不管以何种方式访问某个类，并且在主调代码中不需要进行同步，都能表现正确的行为。

线程安全有以下几种实现方式：

## 不可变

不可变（Immutable）的对象一定是线程安全的，不需要再采取任何的线程安全保障措施。只要一个不可变的对象被正确地构建出来，永远也不会看到它在多个线程之中处于不一致的状态。多线程环境下，应当尽量使对象成为不可变，来满足线程安全。

不可变的类型：

- final 关键字修饰的基本数据类型
- String
- 枚举类型
- Number 部分子类，如 Long 和 Double 等数值包装类型，BigInteger 和 BigDecimal 等大数据类型。但同为 Number 的原子类 AtomicInteger 和 AtomicLong 则是可变的。

对于集合类型，可以使用 Collections.unmodifiableXXX() 方法来获取一个不可变的集合。

```
public class ImmutableExample {
    public static void main(String[] args) {
        Map<String, Integer> map = new HashMap<>();
        Map<String, Integer> unmodifiableMap = Collections.unmodifiableMap(map);
        unmodifiableMap.put("a", 1);
    }
}
```

```
Exception in thread "main" java.lang.UnsupportedOperationException
    at java.util.Collections$UnmodifiableMap.put(Collections.java:1457)
    at ImmutableExample.main(ImmutableExample.java:9)
```

Collections.unmodifiableXXX() 先对原始的集合进行拷贝，需要对集合进行修改的方法都直接抛出异常。

```
public V put(K key, V value) {
    throw new UnsupportedOperationException();
}
```

## 互斥同步

synchronized 和 ReentrantLock。

## 非阻塞同步

互斥同步最主要的问题就是线程阻塞和唤醒所带来的性能问题，因此这种同步也称为阻塞同步。

互斥同步属于一种悲观的并发策略，总是认为只要不去做正确的同步措施，那就肯定会出现问题。无论共享数据是否真的会出现竞争，它都要进行加锁（这里讨论的是概念模型，实际上虚拟机会优化掉很大一部分不必要的加锁）、用户态核心态转换、维护锁计数器和检查是否有被阻塞的线程需要唤醒等操作。

### 1. CAS

随着硬件指令集的发展，我们可以使用基于冲突检测的乐观并发策略：先进行操作，如果没有其它线程争用共享数据，那操作就成功了，否则采取补偿措施（不断地重试，直到成功为止）。这种乐观的并发策略的许多实现都不需要将线程阻塞，因此这种同步操作称为非阻塞同步。

乐观锁需要操作和冲突检测这两个步骤具备原子性，这里就不能再使用互斥同步来保证了，只能靠硬件来完成。硬件支持的原子性操作最典型的是：比较并交换（Compare-and-Swap，CAS）。

#### Compare And Swap

CAS 指的是现代 CPU 广泛支持的一种对内存中的共享数据进行操作的一种特殊指令。这个指令会对内存中的共享数据做原子的读写操作。

简单介绍一下这个指令的操作过程：

- 首先，CPU 会将内存中将要被更改的数据与期望的值做比较。
- 然后，当这两个值相等时，CPU 才会将内存中的数值替换为新的值。否则便不做操作。
- 最后，CPU 会将旧的数值返回。

​这一系列的操作是原子的。它们虽然看似复杂，但却是 Java 5 并发机制优于原有锁机制的根本。简单来说，CAS 的含义是：我认为原有的值应该是什么，如果是，则将原有的值更新为新值，否则不做修改，并告诉我原来的值是多少。  简单的来说，CAS 有 3 个操作数，内存值 V，旧的预期值 A，要修改的新值 B。当且仅当预期值 A 和内存值 V 相同时，将内存值 V 修改为 B，否则返回 V。这是一种乐观锁的思路，它相信在它修改之前，没有其它线程去修改它；而 Synchronized 是一种悲观锁，它认为在它修改之前，一定会有其它线程去修改它，悲观锁效率很低。

整个AQS同步组件、Atomic原子类操作等等都是以CAS实现的，甚至ConcurrentHashMap在1.8的版本中也调整为了CAS+Synchronized。可以说CAS是整个JUC的基石。

![img](https://upload-images.jianshu.io/upload_images/2251324-6aa6051b693594c1.png?raw=true?imageMogr2/auto-orient/strip%7CimageView2/2/w/623/format/webp)

CAS 指令需要有 3 个操作数，分别是内存地址 V、旧的预期值 A 和新值 B。当执行操作时，只有当 V 的值等于 A，才将 V 的值更新为 B。

JUC下的atomic类都是通过CAS来实现的，下面我么就以AtomicInteger为例来阐述CAS的实现。如下：

```
    private static final Unsafe unsafe = Unsafe.getUnsafe();
    private static final long valueOffset;

    static {
        try {
            valueOffset = unsafe.objectFieldOffset
                (AtomicInteger.class.getDeclaredField("value"));
        } catch (Exception ex) { throw new Error(ex); }
    }
    private volatile int value;
```

Unsafe是CAS的核心类，Java无法直接访问底层操作系统，而是通过本地（native）方法来访问。不过尽管如此，JVM还是开了一个后门：Unsafe，它提供了硬件级别的原子操作。

valueOffset为变量值在内存中的偏移地址，unsafe就是通过偏移地址来得到数据的原值的。value当前值，使用volatile修饰，保证多线程环境下看见的是同一个。

我们就以AtomicInteger的addAndGet()方法来做说明，先看源代码：

```
    public final int addAndGet(int delta) {
        return unsafe.getAndAddInt(this, valueOffset, delta) + delta;
    }

    public final int getAndAddInt(Object var1, long var2, int var4) {
        int var5;
        do {
            var5 = this.getIntVolatile(var1, var2);
        } while(!this.compareAndSwapInt(var1, var2, var5, var5 + var4));

        return var5;
    }
```

内部调用unsafe的getAndAddInt方法，在getAndAddInt方法中主要是看compareAndSwapInt方法：

```
    public final native boolean compareAndSwapInt(Object var1, long var2, int var4, int var5);
```

该方法为本地方法，有四个参数，分别代表：对象、对象的地址、预期值、修改值（有位伙伴告诉我他面试的时候就问到这四个变量是啥意思...+_+）。该方法的实现这里就不做详细介绍了，有兴趣的伙伴可以看看openjdk的源码。

CAS可以保证一次的读-改-写操作是原子操作，在单处理器上该操作容易实现，但是在多处理器上实现就有点儿复杂了。

CPU提供了两种方法来实现多处理器的原子操作:

- 总线加锁
- 缓存加锁。

总线加锁：总线加锁就是就是使用处理器提供的一个LOCK#信号，当一个处理器在总线上输出此信号时，其他处理器的请求将被阻塞住,那么该处理器可以独占使用共享内存。但是这种处理方式显得有点儿霸道，不厚道，他把CPU和内存之间的通信锁住了，在锁定期间，其他处理器都不能其他内存地址的数据，其开销有点儿大。所以就有了缓存加锁。

缓存加锁：其实针对于上面那种情况我们只需要保证在同一时刻对某个内存地址的操作是原子性的即可。缓存加锁就是缓存在内存区域的数据如果在加锁期间，当它执行锁操作写回内存时，处理器不在输出LOCK#信号，而是修改内部的内存地址，利用缓存一致性协议来保证原子性。缓存一致性机制可以保证同一个内存区域的数据仅能被一个处理器修改，也就是说当CPU1修改缓存行中的i时使用缓存锁定，那么CPU2就不能同时缓存了i的缓存行。

**CAS缺陷**

主要表现在三个方法:

- 循环时间太长
- 只能保证一个共享变量原子操作
- ABA问题。

**循环时间太长**

如果CAS一直不成功呢？这种情况绝对有可能发生，如果自旋CAS长时间地不成功，则会给CPU带来非常大的开销。

**只能保证一个共享变量原子操作**

看了CAS的实现就知道这只能针对一个共享变量，如果是多个共享变量就只能使用锁了，当然如果你有办法把多个变量整成一个变量，利用CAS也不错。

**ABA问题**

CAS需要检查操作值有没有发生改变，如果没有发生改变则更新。但是存在这样一种情况如果一个值原来是A，变成了B，然后又变成了A，那么在CAS检查的时候会发现没有改变，但是实质上它已经发生了改变，这就是所谓的ABA问题。对于ABA问题其解决方案是加上版本号，即在每个变量都加上一个版本号，每次改变时加1，即A —> B  —> A，变成1A —> 2B  —> 3A。

#### AQS

AQS 是 AbstractQueuedSynchronizer 的简称，java.util.concurrent（J.U.C）大大提高了并发性能。它提供了一个基于 FIFO 队列，这个队列可以用来构建锁或者其他相关的同步装置的基础框架。下图是 AQS 底层的数据结构：

[![img](https://github.com/orangehaswing/fullstack-tutorial/raw/master/notes/JavaArchitecture/assets/616953-20160403170136176-573839888.png)](https://github.com/orangehaswing/fullstack-tutorial/blob/master/notes/JavaArchitecture/assets/616953-20160403170136176-573839888.png)

它底层使用的是双向列表，是队列的一种实现 , 因此也可以将它当成一种队列。

- Sync queue 是同步列表，它是双向列表 , 包括 head，tail 节点。其中 head 节点主要用来后续的调度 ;
- Condition queue 是单向链表 , 不是必须的 , 只有当程序中需要 Condition 的时候，才会存在这个单向链表 , 并且可能会有多个 Condition queue。

简单的来说：

- AQS其实就是一个可以给我们实现锁的**框架**
- 内部实现的关键是：**先进先出的队列、state 状态**
- 定义了内部类 ConditionObject
- 拥有两种线程模式
- - 独占模式
  - 共享模式
- 在 LOCK 包中的相关锁（常用的有 ReentrantLock、 ReadWriteLock ）都是基于 AQS 来构建
- 一般我们叫 AQS 为同步器。

### 2. AtomicInteger

J.U.C 包里面的整数原子类 AtomicInteger 的方法调用了 Unsafe 类的 CAS 操作。

以下代码使用了 AtomicInteger 执行了自增的操作。

```
private AtomicInteger cnt = new AtomicInteger();

public void add() {
    cnt.incrementAndGet();
}
```

以下代码是 incrementAndGet() 的源码，它调用了 Unsafe 的 getAndAddInt() 。

```
public final int incrementAndGet() {
    return unsafe.getAndAddInt(this, valueOffset, 1) + 1;
}
```

以下代码是 getAndAddInt() 源码，var1 指示对象内存地址，var2 指示该字段相对对象内存地址的偏移，var4 指示操作需要加的数值，这里为 1。通过 getIntVolatile(var1, var2) 得到旧的预期值，通过调用 compareAndSwapInt() 来进行 CAS 比较，如果该字段内存地址中的值等于 var5，那么就更新内存地址为 var1+var2 的变量为 var5+var4。

可以看到 getAndAddInt() 在一个循环中进行，发生冲突的做法是不断的进行重试。

```
public final int getAndAddInt(Object var1, long var2, int var4) {
    int var5;
    do {
        var5 = this.getIntVolatile(var1, var2);
    } while(!this.compareAndSwapInt(var1, var2, var5, var5 + var4));

    return var5;
}
```

### 3. ABA

如果一个变量初次读取的时候是 A 值，它的值被改成了 B，后来又被改回为 A，那 CAS 操作就会误认为它从来没有被改变过。

CAS的ABA隐患问题，解决方案则是版本号，Java提供了AtomicStampedReference来解决。AtomicStampedReference通过包装[E,Integer]的元组来对对象标记版本戳stamp，从而避免ABA问题。

AtomicStampedReference的compareAndSet()方法定义如下：

```
    public boolean compareAndSet(V   expectedReference,
                                 V   newReference,
                                 int expectedStamp,
                                 int newStamp) {
        Pair<V> current = pair;
        return
            expectedReference == current.reference &&
            expectedStamp == current.stamp &&
            ((newReference == current.reference &&
              newStamp == current.stamp) ||
             casPair(current, Pair.of(newReference, newStamp)));
    }


```

compareAndSet有四个参数，分别表示：预期引用、更新后的引用、预期标志、更新后的标志。源码部门很好理解预期的引用 == 当前引用，预期的标识 == 当前标识，如果更新后的引用和标志和当前的引用和标志相等则直接返回true，否则通过Pair生成一个新的pair对象与当前pair CAS替换。Pair为AtomicStampedReference的内部类，主要用于记录引用和版本戳信息（标识），定义如下：

```
    private static class Pair<T> {
        final T reference;
        final int stamp;
        private Pair(T reference, int stamp) {
            this.reference = reference;
            this.stamp = stamp;
        }
        static <T> Pair<T> of(T reference, int stamp) {
            return new Pair<T>(reference, stamp);
        }
    }

    private volatile Pair<V> pair;

```

Pair记录着对象的引用和版本戳，版本戳为int型，保持自增。同时Pair是一个不可变对象，其所有属性全部定义为final，对外提供一个of方法，该方法返回一个新建的Pari对象。pair对象定义为volatile，保证多线程环境下的可见性。在AtomicStampedReference中，大多方法都是通过调用Pair的of方法来产生一个新的Pair对象，然后赋值给变量pair。如set方法：

```
    public void set(V newReference, int newStamp) {
        Pair<V> current = pair;
        if (newReference != current.reference || newStamp != current.stamp)
            this.pair = Pair.of(newReference, newStamp);
    }
```

## 无同步方案

要保证线程安全，并不是一定就要进行同步。如果一个方法本来就不涉及共享数据，那它自然就无须任何同步措施去保证正确性。

### 1. 栈封闭

多个线程访问同一个方法的局部变量时，不会出现线程安全问题，因为局部变量存储在虚拟机栈中，属于线程私有的。

```
public class StackClosedExample {
    public void add100() {
        int cnt = 0;
        for (int i = 0; i < 100; i++) {
            cnt++;
        }
        System.out.println(cnt);
    }
}
```

```
public static void main(String[] args) {
    StackClosedExample example = new StackClosedExample();
    ExecutorService executorService = Executors.newCachedThreadPool();
    executorService.execute(() -> example.add100());
    executorService.execute(() -> example.add100());
    executorService.shutdown();
}
```

```
100
100
```

### 2. 线程本地存储（Thread Local Storage）

如果一段代码中所需要的数据必须与其他代码共享，那就看看这些共享数据的代码是否能保证在同一个线程中执行。如果能保证，我们就可以把共享数据的可见范围限制在同一个线程之内，这样，无须同步也能保证线程之间不出现数据争用的问题。

符合这种特点的应用并不少见，大部分使用消费队列的架构模式（如“生产者-消费者”模式）都会将产品的消费过程尽量在一个线程中消费完。其中最重要的一个应用实例就是经典 Web 交互模型中的“一个请求对应一个服务器线程”（Thread-per-Request）的处理方式，这种处理方式的广泛应用使得很多 Web 服务端应用都可以使用线程本地存储来解决线程安全问题。

可以使用 java.lang.ThreadLocal 类来实现线程本地存储功能。

#### ThreadLocal

ThreadLocal为变量在每个线程中都创建了一个副本，所以每个线程可以访问自己内部的副本变量，不同线程之间不会互相干扰。

set(T value) 和 get()方法的源码：

```
 public void set(T value) {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null)
        map.set(this, value);
    else
        createMap(t, value);
}

public T get() {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null) {
        ThreadLocalMap.Entry e = map.getEntry(this);
        if (e != null) {
            @SuppressWarnings("unchecked")
            T result = (T)e.value;
            return result;
        }
    }
    return setInitialValue();
}

ThreadLocalMap getMap(Thread t) {
    return t.threadLocals;
}

```

可以发现，每个线程中都有一个`ThreadLocalMap`数据结构，当执行set方法时，其值是保存在当前线程的`threadLocals`变量中，当执行get方法中，是从当前线程的`threadLocals`变量获取。

所以在线程1中set的值，对线程2来说是摸不到的，而且在线程2中重新set的话，也不会影响到线程1中的值，保证了线程之间不会相互干扰。

那每个线程中的`ThreadLocalMap`究竟是什么？

#### ThreadLocalMap

从名字上看，可以猜到它也是一个类似HashMap的数据结构，但是在ThreadLocal中，并没实现Map接口。

在ThreadLoalMap中，也是初始化一个大小16的Entry数组，Entry对象用来保存每一个key-value键值对，只不过这里的key永远都是ThreadLocal对象。通过ThreadLocal对象的set方法，结果把ThreadLocal对象自己当做key，放进了ThreadLoalMap中。

![img](https://upload-images.jianshu.io/upload_images/2184951-9611b7b31c9b2e20.png?raw=true?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp)

这里需要注意的是，ThreadLoalMap的Entry是继承WeakReference，和HashMap很大的区别是，Entry中没有next字段，所以就不存在链表的情况了。

#### hash冲突

没有链表结构，那发生hash冲突了怎么办？先看看ThreadLoalMap中插入一个key-value的实现

```
private void set(ThreadLocal<?> key, Object value) {
    Entry[] tab = table;
    int len = tab.length;
    int i = key.threadLocalHashCode & (len-1);

    for (Entry e = tab[i];
         e != null;
         e = tab[i = nextIndex(i, len)]) {
        ThreadLocal<?> k = e.get();

        if (k == key) {
            e.value = value;
            return;
        }

        if (k == null) {
            replaceStaleEntry(key, value, i);
            return;
        }
    }

    tab[i] = new Entry(key, value);
    int sz = ++size;
    if (!cleanSomeSlots(i, sz) && sz >= threshold)
        rehash();
}

```

每个ThreadLocal对象都有一个hash值`threadLocalHashCode`，每初始化一个ThreadLocal对象，hash值就增加一个固定的大小`0x61c88647`。

在插入过程中，根据ThreadLocal对象的hash值，定位到table中的位置i，过程如下：

1. 如果当前位置是空的，那么正好，就初始化一个Entry对象放在位置i上；
2. 不巧，位置i已经有Entry对象了，如果这个Entry对象的key正好是即将设置的key，那么重新设置Entry中的value；
3. 很不巧，位置i的Entry对象，和即将设置的key没关系，那么只能找下一个空位置；

这样的话，在get的时候，也会根据ThreadLocal对象的hash值，定位到table中的位置，然后判断该位置Entry对象中的key是否和get的key一致，如果不一致，就判断下一个位置。

可以发现，set和get如果冲突严重的话，效率很低，因为ThreadLoalMap是Thread的一个属性，所以即使在自己的代码中控制了设置的元素个数，但还是不能控制其它代码的行为。

#### 内存泄露

ThreadLocal可能导致内存泄漏，为什么？先看看Entry的实现：

```
static class Entry extends WeakReference<ThreadLocal<?>> {
    /** The value associated with this ThreadLocal. */
    Object value;

    Entry(ThreadLocal<?> k, Object v) {
        super(k);
        value = v;
    }
}

```

通过之前的分析已经知道，当使用ThreadLocal保存一个value时，会在ThreadLocalMap中的数组插入一个Entry对象，按理说key-value都应该以强引用保存在Entry对象中，但在ThreadLocalMap的实现中，key被保存到了WeakReference对象中。

这就导致了一个问题，ThreadLocal在没有外部强引用时，发生GC时会被回收，如果创建ThreadLocal的线程一直持续运行，那么这个Entry对象中的value就有可能一直得不到回收，发生内存泄露。

#### 如何避免内存泄露

既然已经发现有内存泄露的隐患，自然有应对的策略，在调用ThreadLocal的get()、set()可能会清除ThreadLocalMap中key为null的Entry对象，这样对应的value就没有GC Roots可达了，下次GC的时候就可以被回收，当然如果调用remove方法，肯定会删除对应的Entry对象。

如果使用ThreadLocal的set方法之后，没有显示的调用remove方法，就有可能发生内存泄露，所以养成良好的编程习惯十分重要，使用完ThreadLocal之后，记得调用remove方法。

```
ThreadLocal<String> localName = new ThreadLocal();
try {
    localName.set("占小狼");
    // 其它业务逻辑
} finally {
    localName.remove();
}
```

对于以下代码，thread1 中设置 threadLocal 为 1，而 thread2 设置 threadLocal 为 2。过了一段时间之后，thread1 读取 threadLocal 依然是 1，不受 thread2 的影响。

```
public class ThreadLocalExample {
    public static void main(String[] args) {
        ThreadLocal threadLocal = new ThreadLocal();
        Thread thread1 = new Thread(() -> {
            threadLocal.set(1);
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(threadLocal.get());
            threadLocal.remove();
        });
        Thread thread2 = new Thread(() -> {
            threadLocal.set(2);
            threadLocal.remove();
        });
        thread1.start();
        thread2.start();
    }
}
```

```
1
```

为了理解 ThreadLocal，先看以下代码：

```
public class ThreadLocalExample1 {
    public static void main(String[] args) {
        ThreadLocal threadLocal1 = new ThreadLocal();
        ThreadLocal threadLocal2 = new ThreadLocal();
        Thread thread1 = new Thread(() -> {
            threadLocal1.set(1);
            threadLocal2.set(1);
        });
        Thread thread2 = new Thread(() -> {
            threadLocal1.set(2);
            threadLocal2.set(2);
        });
        thread1.start();
        thread2.start();
    }
}
```

它所对应的底层结构图为：

[![img](https://github.com/orangehaswing/OrdinaryNote/blob/master/Java%E5%B9%B6%E5%8F%91/resource/3646544a-cb57-451d-9e03-d3c4f5e4434a.png?raw=true)](https://github.com/CyC2018/CS-Notes/blob/master/pics/3646544a-cb57-451d-9e03-d3c4f5e4434a.png?raw=true)

每个 Thread 都有一个 ThreadLocal.ThreadLocalMap 对象。

```
/* ThreadLocal values pertaining to this thread. This map is maintained
 * by the ThreadLocal class. */
ThreadLocal.ThreadLocalMap threadLocals = null;
```

当调用一个 ThreadLocal 的 set(T value) 方法时，先得到当前线程的 ThreadLocalMap 对象，然后将 ThreadLocal->value 键值对插入到该 Map 中。

```
public void set(T value) {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null)
        map.set(this, value);
    else
        createMap(t, value);
}
```

get() 方法类似。

```
public T get() {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null) {
        ThreadLocalMap.Entry e = map.getEntry(this);
        if (e != null) {
            @SuppressWarnings("unchecked")
            T result = (T)e.value;
            return result;
        }
    }
    return setInitialValue();
}
```

ThreadLocal 从理论上讲并不是用来解决多线程并发问题的，因为根本不存在多线程竞争。

在一些场景 (尤其是使用线程池) 下，由于 ThreadLocal.ThreadLocalMap 的底层数据结构导致 ThreadLocal 有内存泄漏的情况，应该尽可能在每次使用 ThreadLocal 后手动调用 remove()，以避免出现 ThreadLocal 经典的内存泄漏甚至是造成自身业务混乱的风险。

### 3. 可重入代码（Reentrant Code）

这种代码也叫做纯代码（Pure Code），可以在代码执行的任何时刻中断它，转而去执行另外一段代码（包括递归调用它本身），而在控制权返回后，原来的程序不会出现任何错误。

可重入代码有一些共同的特征，例如不依赖存储在堆上的数据和公用的系统资源、用到的状态量都由参数中传入、不调用非可重入的方法等。

# 锁的种类与锁优化

## 锁的种类

包括自旋锁、自旋锁的其他种类、阻塞锁、可重入锁、读写锁、互斥锁、悲观锁、乐观锁、公平锁、可重入锁等等。重点看如下几种：可重入锁、读写锁、可中断锁、公平锁。

### 可重入锁

如果锁具备可重入性，则称作为可重入锁。synchronized和ReentrantLock都是可重入锁，可重入性实际上表明了锁的分配机制：基于线程的分配，而不是基于方法调用的分配。比如说，当一个线程执行到method1 的synchronized方法时，而在method1中会调用另外一个synchronized方法method2，此时该线程不必重新去申请锁，而是可以直接执行方法method2。

### 读写锁

读写锁将对一个资源的访问分成了2个锁，如文件，一个读锁和一个写锁。正因为有了读写锁，才使得多个线程之间的读操作不会发生冲突。`ReadWriteLock`就是读写锁，它是一个接口，ReentrantReadWriteLock实现了这个接口。可以通过readLock()获取读锁，通过writeLock()获取写锁。

### 可中断锁

可中断锁，即可以中断的锁。在Java中，synchronized就不是可中断锁，而Lock是可中断锁。如果某一线程A正在执行锁中的代码，另一线程B正在等待获取该锁，可能由于等待时间过长，线程B不想等待了，想先处理其他事情，我们可以让它中断自己或者在别的线程中中断它，这种就是可中断锁。

Lock接口中的lockInterruptibly()方法就体现了Lock的可中断性。

### 公平锁

公平锁即尽量以请求锁的顺序来获取锁。同时有多个线程在等待一个锁，当这个锁被释放时，等待时间最久的线程（最先请求的线程）会获得该锁，这种就是公平锁。

非公平锁即无法保证锁的获取是按照请求锁的顺序进行的，这样就可能导致某个或者一些线程永远获取不到锁。

`synchronized`是非公平锁，它无法保证等待的线程获取锁的顺序。对于`ReentrantLock`和`ReentrantReadWriteLock`，默认情况下是非公平锁，但是可以设置为公平锁。

## 锁优化

这里的锁优化主要是指 JVM 对 synchronized 的优化。

### 自旋锁

互斥同步进入阻塞状态的开销都很大，应该尽量避免。在许多应用中，共享数据的锁定状态只会持续很短的一段时间。自旋锁的思想是让一个线程在请求一个共享数据的锁时执行忙循环（自旋）一段时间，如果在这段时间内能获得锁，就可以避免进入阻塞状态。

自旋锁虽然能避免进入阻塞状态从而减少开销，但是它需要进行忙循环操作占用 CPU 时间，它只适用于共享数据的锁定状态很短的场景。

在 JDK 1.6 中引入了自适应的自旋锁。自适应意味着自旋的次数不再固定了，而是由前一次在同一个锁上的自旋次数及锁的拥有者的状态来决定。

### 锁消除

锁消除是指对于被检测出不可能存在竞争的共享数据的锁进行消除。

锁消除主要是通过逃逸分析来支持，如果堆上的共享数据不可能逃逸出去被其它线程访问到，那么就可以把它们当成私有数据对待，也就可以将它们的锁进行消除。

对于一些看起来没有加锁的代码，其实隐式的加了很多锁。例如下面的字符串拼接代码就隐式加了锁：

```
public static String concatString(String s1, String s2, String s3) {
    return s1 + s2 + s3;
}
```

String 是一个不可变的类，编译器会对 String 的拼接自动优化。在 JDK 1.5 之前，会转化为 StringBuffer 对象的连续 append() 操作：

```
public static String concatString(String s1, String s2, String s3) {
    StringBuffer sb = new StringBuffer();
    sb.append(s1);
    sb.append(s2);
    sb.append(s3);
    return sb.toString();
}
```

每个 append() 方法中都有一个同步块。虚拟机观察变量 sb，很快就会发现它的动态作用域被限制在 concatString() 方法内部。也就是说，sb 的所有引用永远不会逃逸到 concatString() 方法之外，其他线程无法访问到它，因此可以进行消除。

### 锁粗化

如果一系列的连续操作都对同一个对象反复加锁和解锁，频繁的加锁操作就会导致性能损耗。

上一节的示例代码中连续的 append() 方法就属于这类情况。如果虚拟机探测到由这样的一串零碎的操作都对同一个对象加锁，将会把加锁的范围扩展（粗化）到整个操作序列的外部。对于上一节的示例代码就是扩展到第一个 append() 操作之前直至最后一个 append() 操作之后，这样只需要加锁一次就可以了。

### 轻量级锁

JDK 1.6 引入了偏向锁和轻量级锁，从而让锁拥有了四个状态：无锁状态（unlocked）、偏向锁状态（biasble）、轻量级锁状态（lightweight locked）和重量级锁状态（inflated）。

以下是 HotSpot 虚拟机对象头的内存布局，这些数据被称为 Mark Word。其中 tag bits 对应了五个状态，这些状态在右侧的 state 表格中给出。除了 marked for gc 状态，其它四个状态已经在前面介绍过了。

[![img](https://github.com/orangehaswing/OrdinaryNote/blob/master/Java%E5%B9%B6%E5%8F%91/resource/bb6a49be-00f2-4f27-a0ce-4ed764bc605c.png?raw=true)](https://github.com/CyC2018/CS-Notes/blob/master/pics/bb6a49be-00f2-4f27-a0ce-4ed764bc605c.png?raw=true)

下图左侧是一个线程的虚拟机栈，其中有一部分称为 Lock Record 的区域，这是在轻量级锁运行过程创建的，用于存放锁对象的 Mark Word。而右侧就是一个锁对象，包含了 Mark Word 和其它信息。

[![img](https://github.com/orangehaswing/OrdinaryNote/blob/master/Java%E5%B9%B6%E5%8F%91/resource/051e436c-0e46-4c59-8f67-52d89d656182.png?raw=true)](https://github.com/CyC2018/CS-Notes/blob/master/pics/051e436c-0e46-4c59-8f67-52d89d656182.png?raw=true)

轻量级锁是相对于传统的重量级锁而言，它使用 CAS 操作来避免重量级锁使用互斥量的开销。对于绝大部分的锁，在整个同步周期内都是不存在竞争的，因此也就不需要都使用互斥量进行同步，可以先采用 CAS 操作进行同步，如果 CAS 失败了再改用互斥量进行同步。

当尝试获取一个锁对象时，如果锁对象标记为 0 01，说明锁对象的锁未锁定（unlocked）状态。此时虚拟机在当前线程的虚拟机栈中创建 Lock Record，然后使用 CAS 操作将对象的 Mark Word 更新为 Lock Record 指针。如果 CAS 操作成功了，那么线程就获取了该对象上的锁，并且对象的 Mark Word 的锁标记变为 00，表示该对象处于轻量级锁状态。

[![img](https://github.com/orangehaswing/OrdinaryNote/blob/master/Java%E5%B9%B6%E5%8F%91/resource/baaa681f-7c52-4198-a5ae-303b9386cf47.png?raw=true)](https://github.com/CyC2018/CS-Notes/blob/master/pics/baaa681f-7c52-4198-a5ae-303b9386cf47.png?raw=true)

如果 CAS 操作失败了，虚拟机首先会检查对象的 Mark Word 是否指向当前线程的虚拟机栈，如果是的话说明当前线程已经拥有了这个锁对象，那就可以直接进入同步块继续执行，否则说明这个锁对象已经被其他线程线程抢占了。如果有两条以上的线程争用同一个锁，那轻量级锁就不再有效，要膨胀为重量级锁。

### 偏向锁

偏向锁的思想是偏向于让第一个获取锁对象的线程，这个线程在之后获取该锁就不再需要进行同步操作，甚至连 CAS 操作也不再需要。

当锁对象第一次被线程获得的时候，进入偏向状态，标记为 1 01。同时使用 CAS 操作将线程 ID 记录到 Mark Word 中，如果 CAS 操作成功，这个线程以后每次进入这个锁相关的同步块就不需要再进行任何同步操作。

当有另外一个线程去尝试获取这个锁对象时，偏向状态就宣告结束，此时撤销偏向（Revoke Bias）后恢复到未锁定状态或者轻量级锁状态。

[![img](https://github.com/orangehaswing/OrdinaryNote/blob/master/Java%E5%B9%B6%E5%8F%91/resource/390c913b-5f31-444f-bbdb-2b88b688e7ce.jpg?raw=true)](https://github.com/CyC2018/CS-Notes/blob/master/pics/390c913b-5f31-444f-bbdb-2b88b688e7ce.jpg?raw=true)
