# springmvc学习笔记(8)-forward和redirect

## redirect

这是重定向例子：

控制器

```
@Controller
@RequestMapping("/redirect")
public class RedirectController {
	
	private final ConversionService conversionService;

	@Inject
	public RedirectController(ConversionService conversionService) {
		this.conversionService = conversionService;
	}

	@GetMapping("/uriTemplate")
	public String uriTemplate(RedirectAttributes redirectAttrs) {
		redirectAttrs.addAttribute("account", "a123");  // Used as URI template variable
		redirectAttrs.addAttribute("date", new LocalDate(2011, 12, 31));  // Appended as a query parameter
		return "redirect:/redirect/{account}";
	}

	@GetMapping("/uriComponentsBuilder")
	public String uriComponentsBuilder() {
		String date = this.conversionService.convert(new LocalDate(2011, 12, 31), String.class);
		UriComponents redirectUri = UriComponentsBuilder.fromPath("/redirect/{account}").queryParam("date", date)
				.build().expand("a123").encode();
		return "redirect:" + redirectUri.toUriString();
	}

	@GetMapping("/{account}")
	public String show(@PathVariable String account, @RequestParam(required=false) LocalDate date) {
		return "redirect/redirectResults";
	}

}
```

视图

home.jsp

```
<div id="redirect">
		<h2>Redirecting</h2>
		<p>
			See the <code>org.springframework.samples.mvc.redirect</code> package for the @Controller code	
		</p>
		<ul>
			<li>
				<a href="<c:url value="/redirect/uriTemplate" />">URI Template String</a>
			</li>
			<li>
				<a href="<c:url value="/redirect/uriComponentsBuilder" />">UriComponentsBuilder</a>
			</li>
		</ul>
	</div>
```

redirect.jsp

```
<html>
<head>
	<title>Redirect Results</title>
	<link href="<c:url value="/resources/form.css" />" rel="stylesheet"  type="text/css" />		
</head>
<body>
<div class="success">
	<h3>Path variable 'account': ${account}</h3>
	<h3>Query param 'date': ${param.date}</h3>
</div>
</body>
</html>
```

## forward

```
public String handle() {  
    // return "forward:/hello" => 转发到能够匹配 /hello 的 controller 上  
    // return "hello" => 实际上还是转发，只不过是框架会找到该逻辑视图名对应的 View 并渲染  
    // return "/hello" => 同 return "hello"  
    return "forward:/hello";  
}  
```

## RedirectAttributes

这是专门用于重定向之后还能带参数跳转的。

**第一种：**

`redirectAttributes.addAttributie("param",value);` 这种方法相当于在**重定向链接地址追加传递的参数**，例如:

```
redirectAttributes.addAttributie("param1",value1);

redirectAttributes.addAttributie("param2",value2);

return:"redirect：/path/list" 
```

以上重定向的方法等同于 `return:"redirect：/path/list？param1=value1&param2=value2 "` ，注意这种方法直接将传递的参数暴露在链接地址上，非常的不安全，慎用。

**第二种：**

`redirectAttributes.addFlashAttributie("param",value);` 这种方法是隐藏了参数，链接地址上不直接暴露，但是能且只能在重定向的 “页面” 获取param参数值。其原理就是放到session中，session在跳到页面后马上移除对象。

如果是重定向一个controller中是获取不到该param属性值的。 
除非在`controller`中用`(@RequestPrama(value = "param")String param)`注解，采用传参的方式。

页面获值例如：

```
redirectAttributes.addFlashAttributie("param1",value1);

redirectAttributes.addFlashAttributie("param2",value2);

return:"redirect：/path/list.jsp" 12345
```

在以上参数均可在list.jsp页面使用EL表达式获取到参数值${param}

controller获得redirectAttributes重定向的值例如：

```
redirectAttributes.addFlashAttributie("param1",value1);

redirectAttributes.addFlashAttributie("param2",value2);

return:"redirect：/path/list/"

@RequestMapping("list")
public List<Student> list(@RequestPrama(value = "param1")String  param1,
   @RequestPrama(value = "param2")String  param2,...
){
    //TODO
    //your code

}1234567891011121314
```

通过在controller中的list方法体中可以获取到参数值。

## forward与redirect区别

**1.从地址栏显示来说**

- forward是服务器请求资源，即服务器直接访问目标地址的URL，把那个URL的响应内容读取过来,然后把这些内容再发给浏览器，浏览器根本不知道服务器发送的内容从哪里来的，所以它的地址栏还是原来的地址.
- redirect是服务端根据逻辑,发送一个状态码,告诉浏览器重新去请求那个地址.所以地址栏显示的是新的URL.

**2.从数据共享来说**

- forward：转发页面和转发到的页面可以共享request里面的数据.
- redirect：不能共享数据.

**3.从运用地方来说**

- forward：一般用于用户登陆的时候,根据角色转发到相应的模块.
- redirect：一般用于用户注销登陆时返回主页面和跳转到其它的网站等.

**4.从效率来说**

- forward：高.
- redirect：低.

## 内部机制区别

1. 请求转发只能将请求转发给同一个WEB应用中的组件，而重定向还可以重新定向到同一站点不同应用程序中的资源，甚至可以定向到一绝对的URL。   
2. 重定向可以看见目标页面的URL，转发只能看见第一次访问的页面URL，以后的工作都是有服务器来做的。  
3. 请求响应调用者和被调用者之间共享相同的request对象和response对象，重定向调用者和被调用者属于两个独立访问请求和响应过程。   
4. 重定向跳转后必须加上return，要不然页面虽然跳转了，但是还会执行跳转后面的语句，转发是执行了跳转页面，下面的代码就不会在执行了。











