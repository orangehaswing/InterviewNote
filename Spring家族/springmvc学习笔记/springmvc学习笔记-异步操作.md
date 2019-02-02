# springmvc学习笔记-异步操作

# 概念

## 同步和异步

同步和异步关注的是结果消息的通信机制

- 同步: 调用方需要主动等待结果的返回
- 异步: 不需要主动等待结果的返回，而是通过其他手段比如，状态通知，回调函数等。

阻塞和非阻塞主要关注的是等待结果返回调用方的状态

- 阻塞: 是指结果返回之前，当前线程被挂起，不做任何事
- 非阻塞: 是指结果在返回之前，线程可以做一些其他事，不会被挂起。

**同步例子**

```
int n = func();
next();
// func() 的结果没有返回，next() 就不会执行，直到 func() 运行完。
```

**异步例子**

```
func(callback);
next();
...

void callback(int n)     // func 结果回调
{
  int k = n;
}
// func() 执行后，还没得出结果就立即返回，然后执行 next() 了
// 等到结果出来，func() 回调 callback() 通知调用者结果。
```

## 同步通信方式与异步通信

- 同步通信方式要求通信双方以相同的时钟频率进行，而且准确协调，通过共享一个单个时钟或定时脉冲源保证发送方和接收方的准确同步，效率较高； 
- 异步通信方式不要求双方同步，收发方可采用各自的时钟源，双方遵循异步的通信协议，以字符为数据传输单位，发送方传送字符的时间间隔不确定，发送效率比同步传送效率低。

# 什么是异步模式

要知道什么是异步模式，就先要知道什么是同步模式，先看最典型的同步模式：

![img](https://images2015.cnblogs.com/blog/379997/201605/379997-20160504115915138-347834786.png)

(图1)

浏览器发起请求，Web服务器开一个线程处理，处理完把处理结果返回浏览器。好像没什么好说的了，绝大多数Web服务器都如此般处理。现在想想如果处理的过程中需要调用后端的一个业务逻辑服务器，会是怎样呢？

![img](https://images2015.cnblogs.com/blog/379997/201605/379997-20160504125034466-64438905.png)

(图2)

调就调吧，上图所示，请求处理线程会在Call了之后等待Return，自身处于阻塞状态。这也是绝大多数Web服务器的做法，一般来说这样做也够了，为啥？一来“长时间处理服务”调用通常不多，二来请求数其实也不多。要不是这样的话，这种模式会出现什么问题呢？——会出现的问题就是请求处理线程的短缺！因为请求处理线程的总数是有限的，如果类似的请求多了，所有的处理线程处于阻塞的状态，那新的请求也就无法处理了，也就所谓影响了服务器的吞吐能力。要更加好地发挥服务器的全部性能，就要使用异步，这也是标题上所说的“高性能的关键”。接下来我们来看看异步是怎么一回事：

 ![img](https://images2015.cnblogs.com/blog/379997/201605/379997-20160504125905513-2084771213.png)

(图3)

最大的不同在于请求处理线程对后台处理的调用使用了“invoke”的方式，就是说调了之后直接返回，而不等待，这样请求处理线程就“自由”了，它可以接着去处理别的请求，当后端处理完成后，会钩起一个回调处理线程来处理调用的结果，这个回调处理线程跟请求处理线程也许都是线程池中的某个线程，相互间可以完全没有关系，由这个回调处理线程向浏览器返回内容。这就是异步的过程。

带来的改进是显而易见的，请求处理线程不需要阻塞了，它的能力得到了更充分的使用，带来了服务器吞吐能力的提升。

# Spring MVC的使用

## Callable<>()

```
@GetMapping("/response-body")
public @ResponseBody Callable<String> callable() {

   return new Callable<String>() {
      @Override
      public String call() throws Exception {
         Thread.sleep(2000);
         return "Callable result";
      }
   };
}
```

使用callable线程回调方式，产生2000ms的延迟。

## DefferedResult

要使用Spring MVC的异步功能，你得先确保你用的是Servlet 3.0或以上的版本，Maven中如此配置：

```
    <dependency>
      <groupId>javax.servlet</groupId>
      <artifactId>javax.servlet-api</artifactId>
      <version>3.1.0</version>
      <scope>provided</scope>
    </dependency>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-webmvc</artifactId>
      <version>4.2.3.RELEASE</version>
    </dependency>
```

我这里使用的Servlet版本是3.1.0，Spring MVC版本是4.2.3，建议使用最新的版本。

由于Spring MVC的良好封装，异步功能使用起来出奇的简单。**传统的同步模式的Controller是返回ModelAndView，而异步模式则是返回DeferredResult**。

看这个例子：

```
@RequestMapping(value="/asynctask", method = RequestMethod.GET)
public DeferredResult<ModelAndView> asyncTask(){
    DeferredResult<ModelAndView> deferredResult = new DeferredResult<ModelAndView>();
    System.out.println("/asynctask 调用！thread id is : " + Thread.currentThread().getId());
    longTimeAsyncCallService.makeRemoteCallAndUnknownWhenFinish(new LongTermTaskCallback() {
        @Override
        public void callback(Object result) {
            System.out.println("异步调用执行完成, thread id is : " + Thread.currentThread().getId());
            ModelAndView mav = new ModelAndView("remotecalltask");
            mav.addObject("result", result);
            deferredResult.setResult(mav);
        }
    });
}
```

longTimeAsyncCallService是我写的一个模拟长时间异步调用的服务类，调用之，立即返回，当它处理完成时候，就钩起一个线程调用我们提供的回调函数，这跟“图3”描述的一样，它的代码如下：

```
public interface LongTermTaskCallback {
    void callback(Object result);
}

public class LongTimeAsyncCallService {
    private final int CorePoolSize = 4;
    private final int NeedSeconds = 3;
    private Random random = new Random();
    private ScheduledExecutorService scheduler = Executors.newScheduledThreadPool(CorePoolSize);
    public void makeRemoteCallAndUnknownWhenFinish(LongTermTaskCallback callback){
        System.out.println("完成此任务需要 : " + NeedSeconds + " 秒");
        scheduler.schedule(new Runnable() {
            @Override
            public void run() {
                callback.callback("长时间异步调用完成.");
            }
        }, "这是处理结果:)", TimeUnit.SECONDS);
    }
}
```

输出的结果是：

/asynctask 调用！thread id is : 46
完成此任务需要 : 3 秒
异步调用执行完成, thread id is : 47

由此可见返回结果的线程和请求处理线程不是同一线程。

## WebAsyncTask

返回 `DefferedResult<ModelAndView>` 并非唯一做法，还可以返回WebAsyncTask来实现“异步”，但略有不同，不同之处在于返回WebAsyncTask的话是不需要我们主动去调用Callback的，看例子：

```
@RequestMapping(value="/longtimetask", method = RequestMethod.GET)
public WebAsyncTask longTimeTask(){
    System.out.println("/longtimetask被调用 thread id is : " + Thread.currentThread().getId());
    Callable<ModelAndView> callable = new Callable<ModelAndView>() {
        public ModelAndView call() throws Exception {
            Thread.sleep(3000); //假设是一些长时间任务
            ModelAndView mav = new ModelAndView("longtimetask");
            mav.addObject("result", "执行成功");
            System.out.println("执行成功 thread id is : " + Thread.currentThread().getId());
            return mav;
        }
    };
    return new WebAsyncTask(callable);
}
```

其核心是一个Callable<ModelAndView>，事实上，直接返回Callable<ModelAndView>都是可以的，但我们这里包装了一层，以便做后面提到的“超时处理”。和前一个方案的差别在于这个Callable的call方法并不是我们直接调用的，而是在longTimeTask返回后，由Spring MVC用一个工作线程来调用，执行，打印出来的结果：

/longtimetask被调用 thread id is : 56
执行成功 thread id is : 57

可见确实由不同线程执行的，但这个WebAsyncTask可不太符合“图3”所描述的技术规格，它仅仅是简单地把请求处理线程的任务转交给另一工作线程而已。

# 处理超时

如果“长时间处理任务”一直没返回，那我们也不应该让客户端无限等下去啊，总归要弄个“超时”出来。如图：

 ![img](https://images2015.cnblogs.com/blog/379997/201605/379997-20160504154249294-1156293670.png)

(图4)

其实“超时处理线程”和“回调处理线程”可能都是线程池中的某个线程，我为了清晰点把它们分开画而已。增加这个超时处理在Spring MVC中非常简单，先拿WebAsyncTask那段代码来改一下：

```
@RequestMapping(value="/longtimetask", method = RequestMethod.GET)
public WebAsyncTask longTimeTask(){
    System.out.println("/longtimetask被调用 thread id is : " + Thread.currentThread().getId());
    Callable<ModelAndView> callable = new Callable<ModelAndView>() {
        public ModelAndView call() throws Exception {
            Thread.sleep(3000); //假设是一些长时间任务
            ModelAndView mav = new ModelAndView("longtimetask");
            mav.addObject("result", "执行成功");
            System.out.println("执行成功 thread id is : " + Thread.currentThread().getId());
            return mav;
        }
    };
    
    
    WebAsyncTask asyncTask = new WebAsyncTask(2000, callable);
    asyncTask.onTimeout(
            new Callable<ModelAndView>() {
                public ModelAndView call() throws Exception {
                    ModelAndView mav = new ModelAndView("longtimetask");
                    mav.addObject("result", "执行超时");
                    System.out.println("执行超时 thread id is ：" + Thread.currentThread().getId());
                    return mav;
                }
            }
    );
    return new WebAsyncTask(3000, callable);
}
```

注意看红色字体部分代码，这就是前面提到的为什么Callable还要外包一层的缘故，给WebAsyncTask设置一个超时回调，即可实现超时处理，在这个例子中，正常处理需要3秒钟，而超时设置为2秒，所以肯定会出现超时，执行打印log如下：

/longtimetask被调用 thread id is : 59
执行超时 thread id is ：61
执行成功 thread id is : 80

嗯？明明超时了，怎么还会“执行成功”呢？超时归超时，超时并不会打断正常执行流程，但注意，出现超时后我们给客户端返回了“超时”的结果，那接下来即便正常处理流程成功，客户端也收不到正常处理成功所产生的结果了，这带来的问题就是：客户端看到了“超时”，实际上操作到底有没有成功，客户端并不知道，但通常这也不是什么大问题，因为用户在浏览器上再刷新一下就好了。:D

好，再来看DefferedResult方式的超时处理：

```
    @RequestMapping(value = "/asynctask", method = RequestMethod.GET)
    public DeferredResult<ModelAndView> asyncTask() {
        DeferredResult<ModelAndView> deferredResult = new DeferredResult<ModelAndView>(2000L);
        System.out.println("/asynctask 调用！thread id is : " + Thread.currentThread().getId());
        longTimeAsyncCallService.makeRemoteCallAndUnknownWhenFinish(new LongTermTaskCallback() {
            @Override
            public void callback(Object result) {
                System.out.println("异步调用执行完成, thread id is : " + Thread.currentThread().getId());
                ModelAndView mav = new ModelAndView("remotecalltask");
                mav.addObject("result", result);
                deferredResult.setResult(mav);
            }
        });

        deferredResult.onTimeout(new Runnable() {
            @Override
            public void run() {
                System.out.println("异步调用执行超时！thread id is : " + Thread.currentThread().getId());
                ModelAndView mav = new ModelAndView("remotecalltask");
                mav.addObject("result", "异步调用执行超时");
                deferredResult.setResult(mav);
            }
        });

        return deferredResult;
    }
```

非常类似，对吧，我把超时设置为2秒，而正常处理需要3秒，一定会超时，执行结果如下：

/asynctask 调用！thread id is : 48
完成此任务需要 : 3 秒
异步调用执行超时！thread id is : 51
异步调用执行完成, thread id is : 49

完全在我们预料之中。

# 异常处理

貌似没什么差别，在Controller中的处理和之前同步模式的处理是一样的：

```
	@GetMapping("/exception")
	public @ResponseBody Callable<String> callableWithException(
			final @RequestParam(required=false, defaultValue="true") boolean handled) {

		return () -> {
			Thread.sleep(2000);
			if (handled) {
				// see handleException method further below
				throw new IllegalStateException("Callable error");
			}
			else {
				throw new IllegalArgumentException("Callable error");
			}
		};
	}
```

统一异常处理

```
	@ExceptionHandler
	@ResponseBody
	public String handleException(IllegalStateException ex) {
		return "Handled exception: " + ex.getMessage();
	}
```

还要再弄个全局的异常处理啥的，和过去的做法都一样，在此不表了。







