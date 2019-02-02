# springboot学习笔记-统一异常处理

Spring Boot提供了一个默认的映射：`/error`，当处理中抛出异常之后，会转到该请求中处理，并且该请求有一个全局的错误页面用来展示异常内容。

在实际应用中，错误页面对用户来说并不够友好，我们通常需要去实现我们自己的异常提示。

步骤：

- 创建wev应用，启动该应用，访问URL地址。

```
@RequestMapping("/hello")
public class hello {

    public String helloworld() throws Exception{
        throw new Exception("error happened");
    }
}
```

出现错误：

```
Whitelabel Error Page

This application has no explicit mapping for /error, so you are seeing this as a fallback.
Wed Jan 09 14:22:39 CST 2019
There was an unexpected error (type=Not Found, status=404).
No message available
```

- 创建全局异常处理类：通过使用`@ControllerAdvice`定义统一的异常处理类，而不是在每个Controller中逐个定义。`@ExceptionHandler`用来定义函数针对的异常类型，最后将Exception对象和请求URL映射到`error.html`中

```
@ControllerAdvice
public class GlobalExceptionHandler {

    public static final String DEFAULT_ERROR_VIEW = "error";

    @ExceptionHandler(value = Exception.class)
    public ModelAndView defaultErrorHandler(HttpServletRequest req,Exception e)throws Exception{
        ModelAndView mav = new ModelAndView();
        mav.addObject("exception",e);
        mav.addObject("url",req.getRequestURL());
        mav.setViewName(DEFAULT_ERROR_VIEW);
        return mav;
    }
}
```

- 实现`error.html`页面展示：在`templates`目录下创建`error.html`，将请求的URL和Exception对象的message输出。

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>exceptionhandler</title>
</head>
<body>
    <h1>Error Handler</h1>
    <div th:text="${url}"></div>
    <div th:text="${exception.message}"></div>
</body>
</html>
```

- 启动该应用，访问：`http://localhost:8080/hello`

  显示：

  ```
  Error Handler
  ```


可能会有多种不同的Exception。在@ControllerAdvice类中，根据抛出的具体Exception类型匹配@ExceptionHandler中配置的异常类型来匹配错误映射和处理。实际实现还是依靠Spring MVC的注解。




