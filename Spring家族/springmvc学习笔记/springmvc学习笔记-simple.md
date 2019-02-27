# springmvc学习笔记-simple

# 第一节

## Controller

SimpleController

```
@RestController
public class SimpleController {

	@GetMapping("/simple")
	public String simple() {
		return "Hello world!";
	}

}
```

使用@RestController注解，相当于@ResponseBody ＋ @Controller。Controller设置为一个控制器，ResponseBody表示返回的body是Hello world!

`@GetMapping("/simple")` 表示@RequestMapping(method = RequestMethod.GET)。

## View

home.jsp

```
<div id="simple">
<h2>Simple</h2>
	<p>
	See the <code>org.springframework.samples.mvc.simple</code> package for the @Controller code
	</p>
<ul>
	<li>
		<a id="simpleLink" class="textLink" href="<c:url value="/simple" />">GET /simple</a>
	</li>
</ul>
</div>
```

 `href="<c:url value="/simple" />"`  href 属性用于指定超链接目标的 URL  `GET /simple`使用GET方式

# 第二节

## Controller

SimpleControllerRevisited

```
@RestController
public class SimpleControllerRevisited {

	@GetMapping(path="/simple/revisited", headers="Accept=text/plain")
	public String simple() {
		return "Hello world revisited!";
	}
}
```

@GetMapping注解设定path和headers参数。

- path：指定URL访问路径。


- headers： 指定request中必须包含某些指定的header值，才能让该方法处理请求。这里包含的header值在class="textLink"中

## View

```
<div id="simple">
<h2>Simple</h2>
<p>
	See the <code>org.springframework.samples.mvc.simple</code> package for the @Controller code
</p>
<ul>
	<li>
		<a id="simpleRevisited" class="textLink" href="<c:url value="/simple/revisited" />">GET /simple/revisited</a>
	</li>
</ul>
</div>
```

```
$("a.textLink").click(function(){
	var link = $(this);
	$.ajax({ url: link.attr("href"), dataType: "text", success: function(text) { 					MvcUtil.showSuccessResponse(text, link); }, 
	error: function(xhr) { MvcUtil.showErrorResponse(xhr.responseText, link); }});
	return false;
});
```
