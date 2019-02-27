# springmvc学习笔记-data

# 第一节

## Controller

custom

```
@RestController
public class CustomArgumentController {

	@ModelAttribute
	void beforeInvokingHandlerMethod(HttpServletRequest request) {
		request.setAttribute("foo", "bar");
	}
	
	@GetMapping("/data/custom")
	public String custom(@RequestAttribute("foo") String foo) {
		return "Got 'foo' request attribute value '" + foo + "'";
	}
}
```

注解`@RequestAttribute`可以被用于访问由过滤器或拦截器创建的、预先存在的请求属性。

先用@ModelAttribute定义预先存在的属性：

```
@ModelAttribute
void beforeInvokingHandlerMethod(HttpServletRequest request) {
		request.setAttribute("foo", "bar");
}
```

然后用@RequestAttribute 去请求数据

## Views

```
<ul>
	<li>
	<a id="customArg" class="textLink" href="<c:url value="/data/custom" />">Custom</a>	
	</li>
</ul>
```

# 第二节

## Controller

standard

```
@RestController
public class StandardArgumentsController {

	// request related
	@GetMapping("/data/standard/request")
	public String standardRequestArgs(HttpServletRequest request, Principal user, Locale locale) {
		StringBuilder buffer = new StringBuilder();
		buffer.append("request = ").append(request).append(", ");
		buffer.append("userPrincipal = ").append(user).append(", ");
		buffer.append("requestLocale = ").append(locale);
		return buffer.toString();
	}

	@PostMapping("/data/standard/request/reader")
	public String requestReader(Reader requestBodyReader) throws IOException {
		return "Read char request body = " + FileCopyUtils.copyToString(requestBodyReader);
	}

	@PostMapping("/data/standard/request/is")
	public String requestReader(InputStream requestBodyIs) throws IOException {
		return "Read binary request body = " + new String(FileCopyUtils.copyToByteArray(requestBodyIs));
	}
}
```

@GetMapping("/data/standard/request")中解析Servlet 原生参数：HttpServletRequest，Principal，Locale内容。

@PostMapping("/data/standard/request/reader")使用字符流方式请求数据，使用Reader类。

@PostMapping("/data/standard/request/is")使用字节流方式请求数据，使用InputStream。

## Views

```
<li>
	<a id="request" class="textLink" href="<c:url value="/data/standard/request" />">Request arguments</a>				
</li>
<li>
	<form id="requestReader" class="textForm" action="<c:url value="/data/standard/request/reader" />" method="post">
		<input id="requestReaderSubmit" type="submit" value="Request Reader" />
	</form>
</li>			
<li>
	<form id="requestIs" class="textForm" action="<c:url value="/data/standard/request/is" />" method="post">
	<input id="requestIsSubmit" type="submit" value="Request InputStream" />
	</form>
</li>
```

# 第三节

## Controller

```
// response related
@GetMapping("/data/standard/response")
public String response(HttpServletResponse response) {
	return "response = " + response;
}

@GetMapping("/data/standard/response/writer")
public void availableStandardResponseArguments(Writer responseWriter) throws IOException {
	responseWriter.write("Wrote char response using Writer");
}
	
@GetMapping("/data/standard/response/os")
public void availableStandardResponseArguments(OutputStream os) throws IOException {
	os.write("Wrote binary response using OutputStream".getBytes());
}
```

@GetMapping("/data/standard/response")输出HttpServletResponse内容

@GetMapping("/data/standard/response/writer")使用字符流，Writer输出数据。

@GetMapping("/data/standard/response/os")使用字节流，OutputStream输出数据。

## Views

```
<li>
	<a id="response" class="textLink" href="<c:url value="/data/standard/response" />">Response arguments</a>				
</li>			
<li>
	<a id="writer" class="textLink" href="<c:url value="/data/standard/response/writer" />">Response Writer</a>
</li>
	<li>
	<a id="os" class="textLink" href="<c:url value="/data/standard/response/os" />">Response OutputStream</a>				
	</li>
```

# 第四节

## Controller

```
// HttpSession
@GetMapping("/data/standard/session")
public String session(HttpSession session) {
	StringBuilder buffer = new StringBuilder();
	buffer.append("session=").append(session);
	return buffer.toString();
}
```

session就是一种保存上下文信息的机制，它是针对每一个用户的，变量的值保存在服务器端。

## Views

```
<li>
   <a id="session" class="textLink" href="<c:url value="/data/standard/session" />">Session</a>         
</li>  
```

# 第五节

## Controller

RequestDataController

```
@RestController
@RequestMapping("/data")
public class RequestDataController {

	@GetMapping("param")
	public String withParam(@RequestParam String foo) {
		return "Obtained 'foo' query parameter value '" + foo + "'";
	}

	@GetMapping("group")
	public String withParamGroup(JavaBean bean) {
		return "Obtained parameter group " + bean;
	}

	@GetMapping("path/{var}")
	public String withPathVariable(@PathVariable String var) {
		return "Obtained 'var' path variable value '" + var + "'";
	}

	@GetMapping("{path}/simple")
	public String withMatrixVariable(@PathVariable String path, @MatrixVariable String foo) {
		return "Obtained matrix variable 'foo=" + foo + "' from path segment '" + path + "'";
	}

	@GetMapping("{path1}/{path2}")
	public String withMatrixVariablesMultiple (
			@PathVariable String path1, @MatrixVariable(name="foo", pathVar="path1") String foo1,
			@PathVariable String path2, @MatrixVariable(name="foo", pathVar="path2") String foo2) {

		return "Obtained matrix variable foo=" + foo1 + " from path segment '" + path1
				+ "' and variable 'foo=" + foo2 + " from path segment '" + path2 + "'";
	}
}
```

handler method 参数绑定常用的注解,我们根据他们处理的Request的不同内容部分分为四类：

- 处理requet uri 部分（这里指uri template中variable，不含queryString部分）的注解： @PathVariable;
- 处理request header部分的注解：@RequestHeader, @CookieValue;
- 处理request body部分的注解：@RequestParam,  @RequestBody;
- 处理attribute类型是注解：@SessionAttributes, @ModelAttribute;

1. 当使用@RequestMapping URI template 样式映射时， 即 someUrl/{paramId}, 这时的paramId可通过 @Pathvariable注解绑定它传过来的值到方法的参数上。

2. @RequestHeader 注解，可以把Request请求header部分的值绑定到方法的参数上。

3. @CookieValue 可以把Request header中关于cookie的值绑定到方法的参数上。

4. @RequestParam：A） 常用来处理简单类型的绑定  B）用来处理Content-Type  C) 该注解有两个属性： value、

   required； value用来指定要传入值的id名称，required用来指示参数是否必须绑定；

5. @RequestBody注解常用来处理Content-Type

6. @SessionAttributes注解用来绑定HttpSession中的attribute对象的值，便于在方法中的参数里使用。

   该注解有value、types两个属性，可以通过名字和类型指定要使用的attribute 对象

7. @ModelAttribute注解有两个用法，一个是用于方法上，一个是用于参数上； 

   A) 在处理@RequestMapping之前，为请求绑定需要从后台查询的model；

   B) 通过名称对应，把相应名称的值绑定到注解的参数bean上；


注意，方法的参数在不给定参数的情况下：

- 若要绑定的对象时简单类型：调用@RequestParam来处理的。  
- 若要绑定的对象时复杂类型：调用@ModelAttribute来处理的。

简单类型是指原始类型(boolean, int 等)、原始类型对象（Boolean, Int等）、String、Date等ConversionService里可以直接String转换成目标对象的类型

{path1}/{path2}表示指定为含有某变量的一类值


## Views

```
<li>
	<a id="param" class="textLink" href="<c:url value="/data/param?foo=bar" />">Query parameter</a>
</li>
<li>
	<a id="group" class="textLink" href="<c:url value="/data/group?param1=foo&param2=bar&param3=baz" />">Group of query parameters</a>
</li>
<li>
	<a id="var" class="textLink" href="<c:url value="/data/path/foo" />">Path variable</a>
</li>
<li>
	<a id="matrixVar" class="textLink" href="<c:url value="/data/matrixvars;foo=bar/simple" />">Matrix variable</a>
</li>
<li>
	<a id="matrixVarMultiple" class="textLink" href="<c:url value="/data/matrixvars;foo=bar1/multiple;foo=bar2" />">Matrix variables (multiple)</a>
</li>
```

# 第六节

## Controller

```
@GetMapping("header")
public String withHeader(@RequestHeader String Accept) {
	return "Obtained 'Accept' header '" + Accept + "'";
}

@GetMapping("cookie")
public String withCookie(@CookieValue String openid_provider) {
	return "Obtained 'openid_provider' cookie '" + openid_provider + "'";
}

@PostMapping("body")
public String withBody(@RequestBody String body) {
	return "Posted request body '" + body + "'";
}

@PostMapping("entity")
public String withEntity(HttpEntity<String> entity) {
	return "Posted request body '" + entity.getBody() + "'; headers = " + entity.getHeaders();
}
```

参考第五节的@RequestHeader，@CookieValue，@RequestBody

`HttpEntity<String>` 表示在HTTP报文中获取的实体

## Views

```
<li>
	<a id="header" class="textLink" href="<c:url value="/data/header" />">Header</a>
</li>
<li>
	<form id="requestBody" class="textForm" action="<c:url value="/data/body" />" method="post">
	<input id="requestBodySubmit" type="submit" value="Request Body" />
	</form>
</li>				
<li>
	<form id="requestBodyAndHeaders" class="textForm" action="<c:url value="/data/entity" />" method="post">
	<input id="requestBodyAndHeadersSubmit" type="submit" value="Request Body and Headers" />
	</form>
</li>
```

# JavaBean

```
public class JavaBean {

	private String param1;
	private String param2;
	private String param3;

	public String getParam1() {
		return param1;
	}

	public void setParam1(String param1) {
		this.param1 = param1;
	}

	public String getParam2() {
		return param2;
	}

	public void setParam2(String param2) {
		this.param2 = param2;
	}

	public String getParam3() {
		return param3;
	}

	public void setParam3(String param3) {
		this.param3 = param3;
	}
}
```

# RequestAttribute注解

```
@Target(ElementType.PARAMETER)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface RequestAttribute {
	String value();
}
```
