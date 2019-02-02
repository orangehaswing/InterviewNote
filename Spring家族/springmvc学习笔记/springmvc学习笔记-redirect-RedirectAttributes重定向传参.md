# springmvc学习笔记-redirect

# 第一节

## Controller

RedirectController

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
```

@Inject 处理依赖注入，把conversionService自动注入。

RedirectAttributes 是专门用于重定向之后还能带参数跳转的的工具类。

两种带参的方式：

- 第一种：redirectAttributes.addAttributie("prama",value); 这种方法相当于在重定向链接地址追加传递的参数，

```
redirectAttributes.addAttributie("prama1",value1);
redirectAttributes.addAttributie("prama2",value2);
return:"redirect：/path/list" 
```

​	以上重定向的方法等同于 return:"redirect：/path/list？prama1=value1**&**prama2=value2 "

- 第二种 redirectAttributes.addFlashAttributie("prama",value); 这种方法是隐藏了参数，链接地址上不直接暴露，但是能且只能在重定向的 “页面” 获取prama参数值。

  其原理就是放到session中，session在跳到页面后马上移除对象。

  如果是重定向一个controller中是获取不到该prama属性值的。除非在controller中用(@RequestPrama(value = "prama")String prama)注解，

```
redirectAttributes.addFlashAttributie("prama1",value1);
redirectAttributes.addFlashAttributie("prama2",value2);
return:"redirect：/path/list/"

@RequestMapping("list")
public List<Student> list(@RequestPrama(value = "prama1")String  prama1,
   						  @RequestPrama(value = "prama2")String  prama2,...){
    //TODO
    //your code

}
```

## View

home.jsp

```
<div id="redirect">
		<h2>Redirecting</h2>
		<ul>
			<li>
				<a href="<c:url value="/redirect/uriTemplate" />">URI Template String</a>
			</li>
		</ul>
	</div>
```

redirectResults.jsp

```
<div class="success">
	<h3>Path variable 'account': ${account}</h3>
	<h3>Query param 'date': ${param.date}</h3>
</div>
```

这里将显示传入的参数account和param.date

# 第二节

## Controller

RedirectController

```
@GetMapping("/uriComponentsBuilder")
	public String uriComponentsBuilder() {
		String date = this.conversionService.convert(new LocalDate(2011, 12, 31), String.class);
		UriComponents redirectUri = 		UriComponentsBuilder.fromPath("/redirect/{account}").queryParam("date", date)
				.build().expand("a123").encode();
		return "redirect:" + redirectUri.toUriString();
	}
```

spring是通过ConversionService进行类型转换

UriComponentsBuilder构建器与UriComponents类一起工作。通过细粒度的控制URI的各个要素，如构建、扩展模板变量以及编码，UriComponentsBuilder 类用于创建UriComponents实例。expand和encode方法可以返回新的URIComponents

## View

home.jsp

```
<div id="redirect">
		<h2>Redirecting</h2>
		<ul>
			<li>
				<a href="<c:url value="/redirect/uriComponentsBuilder" />">
				UriComponentsBuilder</a>
			</li>
		</ul>
	</div>

```

redirect.jsp同上一节

# 第三节

## Controller

RedirectController

```
@GetMapping("/{account}")
	public String show(@PathVariable String account, @RequestParam(required=false) LocalDate date) {
		return "redirect/redirectResults";
	}
```

上两节中，return "redirect:/redirect/{account}"都将先跳转地址到第三节的@GetMapping("/{account}")方法中，然后统一返回到redirect/redirectResults。

@PathVariable：通过 @PathVariable 可以将 URL 中占位符参数绑定到控制器处理方法的入参中：URL 中的 {xxx} 占位符可以通过@PathVariable("xxx") 绑定到操作方法的入参中。

```
 @RequestMapping("/pathVariable/{name}")  
    public String pathVariable(@PathVariable("name")String name){  
        System.out.println("hello "+name);  
        return "helloworld";  
    }  
```

@RequestParam：是传递参数的，将请求参数区数据映射到功能处理方法的参数上。

- value：参数名字，即入参的请求参数名字，如username表示请求的参数区中的名字为username的参数的值将传入；
- required：是否必须，默认是true，表示请求中一定要有相应的参数，否则将报404错误码；
- defaultValue：默认值，表示如果请求中没有同名参数时的默认值，默认值可以是SpEL表达式，如“#{systemProperties['java.vm.version']}”。

## View

同上

























