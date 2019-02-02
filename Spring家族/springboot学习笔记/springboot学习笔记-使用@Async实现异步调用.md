# springboot学习笔记-使用@Async实现异步调用

# 异步与同步

“异步调用”对应的是“同步调用”，同步调用指程序按照定义顺序依次执行，每一行程序都必须等待上一行程序执行完成之后才能执行；异步调用指程序在顺序执行时，不等待异步调用的语句返回结果就执行后面的程序。

# 同步调用

下面通过一个简单示例来直观的理解什么是同步调用：

- 定义Task类，创建三个处理函数分别模拟三个执行任务的操作，操作消耗时间随机取（10秒内）

```
@Component
public class Task {
    public static Random random =new Random();

    public void doTaskOne() throws Exception {
        System.out.println("开始做任务一");
        long start = System.currentTimeMillis();
        Thread.sleep(random.nextInt(10000));
        long end = System.currentTimeMillis();
        System.out.println("完成任务一，耗时：" + (end - start) + "毫秒");
    }

    public void doTaskTwo() throws Exception {
        System.out.println("开始做任务二");
        long start = System.currentTimeMillis();
        Thread.sleep(random.nextInt(10000));
        long end = System.currentTimeMillis();
        System.out.println("完成任务二，耗时：" + (end - start) + "毫秒");
    }

    public void doTaskThree() throws Exception {
        System.out.println("开始做任务三");
        long start = System.currentTimeMillis();
        Thread.sleep(random.nextInt(10000));
        long end = System.currentTimeMillis();
        System.out.println("完成任务三，耗时：" + (end - start) + "毫秒");
    }
}
```

- 在单元测试用例中，注入Task对象，并在测试用例中执行`doTaskOne`、`doTaskTwo`、`doTaskThree`三个函数。

```
@RunWith(SpringJUnit4ClassRunner.class)
@SpringApplicationConfiguration(classes = Application.class)
public class ApplicationTests {

	@Autowired
	private Task task;

	@Test
	public void test() throws Exception {
		task.doTaskOne();
		task.doTaskTwo();
		task.doTaskThree();
	}
}
```

- 执行单元测试，可以看到类似如下输出：

```
开始做任务一
完成任务一，耗时：4256毫秒
开始做任务二
完成任务二，耗时：4957毫秒
开始做任务三
完成任务三，耗时：7173毫秒
```

任务一、任务二、任务三顺序的执行完了，换言之`doTaskOne`、`doTaskTwo`、`doTaskThree`三个函数顺序的执行完成。

# 异步调用

在Spring Boot中，我们只需要通过使用`@Async`注解就能简单的将原来的同步函数变为异步函数，Task类改在为如下模式：

```
@Component
public class Task {
    @Async
    public void doTaskOne() throws Exception {
        // 同上内容，省略
    }

    @Async
    public void doTaskTwo() throws Exception {
        // 同上内容，省略
    }

    @Async
    public void doTaskThree() throws Exception {
        // 同上内容，省略
    }
}
```

为了让@Async注解能够生效，还需要在Spring Boot的主程序中配置@EnableAsync，如下所示：

```
@SpringBootApplication
@EnableAsync
public class Application {
	public static void main(String[] args) {
		SpringApplication.run(Application.class, args);
	}
}
```

此时可以反复执行单元测试, 比如：

- 没有任何任务相关的输出
- 有部分任务相关的输出
- 乱序的任务相关的输出

原因是目前`doTaskOne`、`doTaskTwo`、`doTaskThree`三个函数的时候已经是异步执行了。主程序在异步调用之后，主程序并不会理会这三个函数是否执行完成了，由于没有其他需要执行的内容，所以程序就自动结束了，导致了不完整或是没有输出任务相关内容的情况。

**注： @Async所修饰的函数不要定义为static类型，这样异步调用不会生效**

# 异步回调

为了让`doTaskOne`、`doTaskTwo`、`doTaskThree`能正常结束，假设我们需要统计一下三个任务并发执行共耗时多少，这就需要等到上述三个函数都完成调动之后记录时间，并计算结果。

那么我们如何判断上述三个异步调用是否已经执行完成呢？我们需要使用`Future`来返回异步调用的结果，就像如下方式改造`doTaskOne`函数：

```
@Async
public Future<String> doTaskOne() throws Exception {
    System.out.println("开始做任务一");
    long start = System.currentTimeMillis();
    Thread.sleep(random.nextInt(10000));
    long end = System.currentTimeMillis();
    System.out.println("完成任务一，耗时：" + (end - start) + "毫秒");
    return new AsyncResult<>("任务一完成");
}
```

按照如上方式改造一下其他两个异步函数之后，下面我们改造一下测试用例，让测试在等待完成三个异步调用之后来做一些其他事情。

```
@Test
public void test() throws Exception {
	long start = System.currentTimeMillis();

	Future<String> task1 = task.doTaskOne();
	Future<String> task2 = task.doTaskTwo();
	Future<String> task3 = task.doTaskThree();

	while(true) {
		if(task1.isDone() && task2.isDone() && task3.isDone()) {
			// 三个任务都调用完成，退出循环等待
			break;
		}
		Thread.sleep(1000);
	}

	long end = System.currentTimeMillis();
	System.out.println("任务全部完成，总耗时：" + (end - start) + "毫秒");
}
```

看看我们做了哪些改变：

- 在测试用例一开始记录开始时间
- 在调用三个异步函数的时候，返回`Future`类型的结果对象
- 在调用完三个异步函数之后，开启一个循环，根据返回的`Future`对象来判断三个异步函数是否都结束了。若都结束，就结束循环；若没有都结束，就等1秒后再判断。
- 跳出循环之后，根据结束时间 - 开始时间，计算出三个任务并发执行的总耗时。

执行一下上述的单元测试，可以看到如下结果：

```
开始做任务一
开始做任务二
开始做任务三
完成任务三，耗时：37毫秒
完成任务二，耗时：3661毫秒
完成任务一，耗时：7149毫秒
任务全部完成，总耗时：8025毫秒
```

可以看到，通过异步调用，让任务一、二、三并发执行，有效的减少了程序的总运行时间。