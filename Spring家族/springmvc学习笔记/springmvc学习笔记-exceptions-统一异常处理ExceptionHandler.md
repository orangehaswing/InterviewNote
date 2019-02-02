# springmvc学习笔记-exceptions

# 第一节

## Controller

```
@GetMapping("/exception")
public String exception() {
   throw new IllegalStateException("Sorry!");
}

@GetMapping("/global-exception")
public String businessException() throws BusinessException {
   throw new BusinessException();
}
```

@GetMapping("/exception")直接抛出异常。使用统一异常处理@ExceptionHandler，在接收到IllegalStateException异常类型时，ResponseBody将会返回IllegalStateException handled!

```
@ExceptionHandler
    public String hanle(IllegalStateException e){
        return "IllegalStateException handled!";
    }
```

@GetMapping("/global-exception")中的BusinessException为自定义异常，然后在URI请求中将自定义异常抛出。这里的BusinessException放在GlobalExceptionHandler类中，使用@ExceptionHandler捕获BusinessException ex异常类型。

```
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler
    public String handleBusinessException(BusinessException ex){
        return "Handled BusinessException";
    }
}
```

## Views

```
<li>
   <a id="exception" class="textLink" href="<c:url value="/exception" />">@ExceptionHandler in Controller</a>
</li>
<li>
   <a id="globalException" class="textLink" href="<c:url value="/global-exception" />">Global @ExceptionHandler</a>
</li>
```

# 第二节

## Controller

```
@ExceptionHandler
public String handle(IllegalStateException e) {
   return "IllegalStateException handled!";
}
```

用@ExceptionHandler统一处理某一类异常，从而能够减少代码重复率和复杂度

@RestControllerAdvice：是一个组合标签，允许类通过类路径扫描被自动检测到。注解的类可以包含带有`@ExceptionHandler`、`@InitBinder`和`@ModelAttribute`注解的方法。

## Views

# BusinessException

```
@SuppressWarnings("serial")
public class BusinessException extends Exception {

}
```

