# springmvc学习笔记-async异步操作

按照原来的web.xml配置，运行异步，会出现报错：

```
Handled exception :Async support must be enabled on a servlet and for all filters involved in async request processing. This is done in Java code using the Servlet API or by adding "true" to servlet and filter declarations in web.xml.
```

提示很清楚，不支持异步，需要在web.xml文件中设置 async 为 true。配置web.xml如下：

```
<servlet>
        <servlet-name>spring_mvc</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>/WEB-INF/spring/springmvc-servlet.xml</param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
        <async-supported>true</async-supported>
</servlet>
```

只需要添加 `<async-supported>true</async-supported>` ，将默认值修改就可以实现异步操作。

# 第一节

## Controller

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

	@GetMapping("/view")
	public Callable<String> callableWithView(final Model model) {
		return () -> {
			Thread.sleep(2000);
			model.addAttribute("foo", "bar");
			model.addAttribute("fruit", "apple");
			return "views/html";
		};
	}
```

使用了J.U.C中的 `Callable<T>` 当 `new Callable<String>`时，可以返回一个线程，对比Runnable的run()，Callable实现的方法是call()。这里设置睡眠时间，在页面上内容显示将会延迟2-3s。

## Views

```
		<li>
			<a id="callableResponseBodyLink" class="textLink"
				href="<c:url value="/async/callable/response-body" />">GET /async/callable/response-body</a>
		</li>
		<li>
			<a id="callableViewLink" class="textLink"
				href="<c:url value="/async/callable/view" />">GET /async/callable/view</a>
		</li>
```

# 第二节

## Controller

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

使用lambda表达式，线程睡眠时间2000毫秒，判断handled并抛出异常。

## Views

```
<li>
   <a id="callableExceptionLink" class="textLink"
      href="<c:url value="/async/callable/exception" />">GET /async/callable/exception</a>
</li>
```

# 第三节

## Controller

```
 @GetMapping("/custom-timeout-handling")
    public @ResponseBody
    WebAsyncTask<String> callableWithCustomTimeoutHandling() {
        Callable<String> callable = () -> {
            Thread.sleep(2000);
            return "Callable result";
        };
        return new WebAsyncTask<String>(1000, callable);
    }
```

返回WebAsyncTask来实现“异步”，返回WebAsyncTask的话是不需要我们主动去调用Callback的。直接返回`Callable<String>` 是可以的，但我们这里包装了一层，以便做到“超时处理”。这个Callable的call方法并不是我们直接调用的，而是在longTimeTask返回后，由Spring MVC用一个工作线程来调用执行。

处理超时：如果“长时间处理任务”一直没返回，那我们也不应该让客户端无限等下去，“超时”用来提醒用户。超时并不会打断正常执行流程，但注意，出现超时后我们给客户端返回了“超时”的结果，那接下来即便正常处理流程成功，客户端也收不到正常处理成功所产生的结果了，这带来的问题就是：客户端看到了“超时”，实际上操作到底有没有成功，客户端并不知道，但通常这也不是什么大问题，因为用户在浏览器上再刷新一下就好了。

## Views

```
<li>
			<a id="callableCustomTimeoutLink" class="textLink"
				href="<c:url value="/async/callable/custom-timeout-handling" />">GET /async/callable/custom-timeout-handling</a>
</li>
```

# 第四节

## Controller

```
private final Queue<DeferredResult<String>> responseBodyQueue = new ConcurrentLinkedQueue<>();
private final Queue<DeferredResult<ModelAndView>> mavQueue = new ConcurrentLinkedQueue<>();
private final Queue<DeferredResult<String>> exceptionQueue = new ConcurrentLinkedQueue<>();
```

设置线程安全队列

```
@GetMapping("/deferred-result/response-body")
public @ResponseBody DeferredResult<String> deferredResult() {
   DeferredResult<String> result = new DeferredResult<>();
   this.responseBodyQueue.add(result);
   return result;
}
```

Callable和Deferredresult做的是同样的事情——释放容器线程，在另一个线程上异步运行长时间的任务。

```
@GetMapping("/deferred-result/exception")
	public @ResponseBody DeferredResult<String> deferredResultWithException() {
		DeferredResult<String> result = new DeferredResult<>();
		this.exceptionQueue.add(result);
		return result;
	}
```

与上一部分相似，返回的result由String变成了一个对象。

## Views

```
<li>
   <a id="deferredResultSuccessLink" class="textLink"
    href="<c:url value="/async/deferred-result/response-body" />">GET /async/deferred-result/response-body</a>
</li>
<li>
	<a id="deferredResultModelAndViewLink" class="textLink"
	href="<c:url value="/async/deferred-result/model-and-view" />">GET /async/deferred-result/model-and-view</a>
</li>
```

# 第五节

## Controller

```
@GetMapping("/deferred-result/timeout-value")
	public @ResponseBody DeferredResult<String> deferredResultWithTimeoutValue() {

		// Provide a default result in case of timeout and override the timeout value
		// set in src/main/webapp/WEB-INF/spring/appServlet/servlet-context.xml

		return new DeferredResult<>(1000L, "Deferred result after timeout");
	}
```

设定一个超时时间，但是返回值还是正常执行。

## Views

```
<li>
	<a id="deferredResultTimeoutValueLink" class="textLink"
	href="<c:url value="/async/deferred-result/timeout-value" />">GET /async/deferred-result/timeout-value</a>
</li>
```

# 第六节

## Scheduled

@Scheduled(fixedRate=2000)

	@Scheduled(fixedRate=2000)
		public void processQueues() {
			for (DeferredResult<String> result : this.responseBodyQueue) {
				result.setResult("Deferred result");
				this.responseBodyQueue.remove(result);
			}
			for (DeferredResult<String> result : this.exceptionQueue) {
				result.setErrorResult(new IllegalStateException("DeferredResult error"));
				this.exceptionQueue.remove(result);
			}
			for (DeferredResult<ModelAndView> result : this.mavQueue) {
				result.setResult(new ModelAndView("views/html", "javaBean", new JavaBean("bar", "apple")));
				this.mavQueue.remove(result);
			}
		}
@Scheduled定时任务，设定时间2000ms。将ConcurrentLinkedQueue设定响应结果。

