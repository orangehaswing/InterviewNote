# springmvc学习笔记-mapping

# 第一节

## Controller

MappingController

```
@RestController
public class MappingController {

   @GetMapping("/mapping/path")
   public String byPath() {
      return "Mapped by path!";
   }

   @GetMapping("/mapping/path/*")
   public String byPathPattern(HttpServletRequest request) {
      return "Mapped by path pattern ('" + request.getRequestURI() + "')";
   }
  }
```

@GetMapping("/mapping/path")使用GET方式设定控制器。

@GetMapping("/mapping/path/*")使用正则表达式  "\*"表示在path之后匹配任意的URI名称，从Views中匹配到的是/mapping/path/wildcard。

1. params，headers；
   - params： 指定request中必须包含某些参数值时，才让该方法处理。
   - headers： 指定request中必须包含某些指定的header值，才能让该方法处理请求。
2. consumes，produces；
   - consumes： 指定处理请求的提交内容类型（Content-Type），例如application/json, text/html;
   - produces:    指定返回的内容类型，仅当request请求头中的(Accept)类型中包含该指定类型才返回；

## Views

```
<div id="mapping">
	<h2>Request Mapping</h2>
	<ul>
		<li>
			<a id="byPath" class="textLink" href="<c:url value="/mapping/path" />">By path</a>
		</li>
		<li>
			<a id="byPathPattern" class="textLink" href="<c:url value="/mapping/path/wildcard" />">By path pattern</a>
		</li>
	</ul>
</div>
```

# 第二节

## Controller

```
@GetMapping(path="/mapping/parameter", params="foo")
public String byParameter() {
	return "Mapped by path + method + presence of query parameter!";
}

@GetMapping(path="/mapping/parameter", params="!foo")
public String byParameterNegation() {
	return "Mapped by path + method + not presence of query parameter!";
}
```

@GetMapping注解用params设定参数，params="foo"表示只有参数为foo的时候，才可以匹配。

对应Views的/mapping/parameter?foo=bar。params="!foo"表示除了参数为foo以外的所有内容，匹配到/mapping/parameter，但是匹配不到/mapping/parameter?foo=bar，因为含有foo参数的地址被过滤了。

## Views

```
<li>
	<a id="byParameter" class="textLink" href="<c:url value="/mapping/parameter?foo=bar" />">By path, method, and presence of parameter</a>
</li>
<li>
	<a id="byNotParameter" class="textLink" href="<c:url value="/mapping/parameter" />">By path, method, and not presence of parameter</a>
</li>
```

# 第三节

## Controller

```
@GetMapping(path="/mapping/header", headers="FooHeader=foo")
public String byHeader() {
	return "Mapped by path + method + presence of header!";
}

@GetMapping(path="/mapping/header", headers="!FooHeader")
public String byHeaderNegation() {
	return "Mapped by path + method + absence of header!";
}
```

@GetMapping注解使用headers进行限定，上述两条对应头部分别包含FooHeader和不包含。

## Views

```
<li>
	<a id="byHeader" href="<c:url value="/mapping/header" />">By presence of header</a>
</li>
<li>
	<a id="byHeaderNegation" class="textLink" href="<c:url value="/mapping/header" />">By absence of header</a>
</li>
```

# 第四节

## Controller

```
@PostMapping(path="/mapping/consumes", consumes=MediaType.APPLICATION_JSON_VALUE)
public String byConsumes(@RequestBody JavaBean javaBean) {
	return "Mapped by path + method + consumable media type (javaBean '" + javaBean + "')";
}

@GetMapping(path="/mapping/produces", produces=MediaType.APPLICATION_JSON_VALUE)
public JavaBean byProducesJson() {
	return new JavaBean();
}

@GetMapping(path="/mapping/produces", produces=MediaType.APPLICATION_XML_VALUE)
public JavaBean byProducesXml() {
	return new JavaBean();
}
```

@PostMapping和@GetMapping用到consumes和produces两种内容类型，只接收JSON或者XML类型的内容。

PostMapping用form表单方式提交。

## Views

```
<li>
	<form id="byConsumes" class="readJsonForm" action="<c:url value="/mapping/consumes" />" method="post">
	<input id="byConsumesSubmit" type="submit" value="By consumes" />
	</form>
</li>
<li>
	<a id="byProducesAcceptJson" class="writeJsonLink" href="<c:url value="/mapping/produces" />">By produces via Accept=application/json</a>
</li>
<li>
    <a id="byProducesAcceptXml" class="writeXmlLink" href="<c:url value="/mapping/produces" />">By produces via Accept=appilcation/xml</a>
</li>
```

# JavaBean

```
@XmlRootElement
public class JavaBean {

	private String foo = "bar";
	private String fruit = "apple";

	public String getFoo() {
		return foo;
	}

	public void setFoo(String foo) {
		this.foo = foo;
	}

	public String getFruit() {
		return fruit;
	}

	public void setFruit(String fruit) {
		this.fruit = fruit;
	}

	@Override
	public String toString() {
		return "JavaBean {foo=[" + foo + "], fruit=[" + fruit + "]}";
	}

}
```

@XmlRootElement：将类或枚举类型映射到 XML 元素。JAXB中的注解，用来根据java类生成xml内容。 
