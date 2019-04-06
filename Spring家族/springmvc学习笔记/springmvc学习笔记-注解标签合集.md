# springmvc学习笔记-注解标签合集

## Controller

在SpringMVC 中，控制器Controller 负责处理由DispatcherServlet 分发的请求，它把用户请求的数据经过业务处理层处理之后封装成一个Model ，然后再把该Model 返回给对应的View 进行展示。在Spring MVC 中提供了一个非常简便的定义Controller 的方法，你无需继承特定的类或实现特定的接口，只需使用@Controller 标记一个类是Controller ，然后使用@RequestMapping 和@RequestParam 等一些注解用以定义URL 请求和Controller 方法之间的映射，这样的Controller 就能被外界访问到。

想自动检测生效，需在XML头文件下引入 spring-context:

```
<context:component-scan base-package="org.springframework.samples.petclinic.web"/>
```

例子：

```
@Controller
@RequestMapping("/async/callable")
public class CallableController {

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
}
```

## RestController

@RestController注解相当于@ResponseBody ＋ @Controller合在一起的作用。

```
@RestController
public class MappingController {

	@GetMapping("/mapping/path")
	public String byPath() {
		return "Mapped by path!";
	}
}
```

返回的Body是 "Mapped by path!"

## RequestMapping

RequestMapping是一个用来处理请求地址映射的注解，可用于类或方法上。用于类上，表示类中的所有响应请求的方法都是以该地址作为父路径。

RequestMapping注解有六个属性:

-  value， method；
   -  value： 指定请求的实际地址，指定的地址可以是URI Template 模式（后面将会说明）；
   -  method： 指定请求的method类型， GET、POST、PUT、DELETE等；
-  consumes，produces；
   - consumes： 指定处理请求的提交内容类型（Content-Type），例如application/json, text/html;

   - produces: 指定返回的内容类型，仅当request请求头中的(Accept)类型中包含该指定类型才返回；

-  params，headers；
   - params： 指定request中必须包含某些参数值是，才让该方法处理。
   - headers： 指定request中必须包含某些指定的header值，才能让该方法处理请求。

value/method示例：

```
@Controller
@RequestMapping("/favsoft")
public class AnnotationController {
    // A） 可以指定为普通的具体值；
    @RequestMapping(method = RequestMethod.GET)  
    public Map<String, Appointment> get() {  
       ...  
    }  
     // B)  可以指定为含有某变量的一类值
 	@RequestMapping(value="/owners/{ownerId}", method=RequestMethod.GET)  
	public String findOwner(@PathVariable String ownerId, Model model) {  
		...
	}  
 
 	// C) 可以指定为含正则表达式的一类值
 	@RequestMapping("/spring-web/{symbolicName:[a-z-]+}-{version:\d\.\d\.\d}.{extension:\.[a-	z]}")  
  	public void handle(@PathVariable String version, @PathVariable String extension) {      
    // ...  
  }  
}  
```

 comsumes/produces示例：

```
cousumes的样例：
方法仅处理request Content-Type为“application/json”类型的请求。

@Controller  
@RequestMapping(value = "/pets", method = RequestMethod.POST, consumes="application/json")  
public void addPet(@RequestBody Pet pet, Model model) {      
   ...
}  
 
produces的样例：
 方法仅处理request请求中Accept头中包含了"application/json"的请求，同时暗示了返回的内容类型为application/json;
 
@Controller  
@RequestMapping(value = "/pets/{petId}", method = RequestMethod.GET, produces="application/json")  
public Pet getPet(@PathVariable String petId, Model model) {      
   ... 
}  
```

 params/headers示例：

```
params的样例：
仅处理请求中包含名为“myParam”，值为“myValue”的请求；

@RequestMapping(value = "/pets/{petId}", method = RequestMethod.GET, params="myParam=myValue")  
 public void findPet(@PathVariable String ownerId, @PathVariable String petId, Model model) { 
  ...
 }  
 
headers的样例：
仅处理request的header中包含了指定“Accept”请求头和对应文本为text/plain的请求；
 
@RequestMapping(value = "/pets", method = RequestMethod.GET, headers="Accept=text/plain")  
  public void findPet(@PathVariable String ownerId, @PathVariable String petId, Model model) { 
  ...
  }  

```

GetMapping和PostMapping与RequestMapping类似，提前定义好method。其中的path值等于RequestMapping中的value

## GetMapping,PostMapping...

- GetMapping
- PostMapping
- PutMapping
- DeleteMapping
- PatchMapping

Spring4.3中引进的上述注解来帮助简化常用的HTTP方法的映射，并更好地表达被注解方法的语义。

GetMapping：

@GetMapping是一个组合注解，是@RequestMapping(method = RequestMethod.GET)的缩写。该注解将HTTP Get 映射到 特定的处理方法上。

```
@RestController
@RequestMapping(value="/response", method=RequestMethod.GET)
public class ResponseController {

	@GetMapping("/annotation")
	public String responseBody() {
		return "The String ResponseBody";
	}

	@GetMapping("/charset/accept")
	public String responseAcceptHeaderCharset() {
		return "\u3053\u3093\u306b\u3061\u306f\u4e16\u754c\uff01 (\"Hello world!\" in Japanese)";
	}
}
```

## 声明Bean

- @Component组件，没有明确的角色
- @Service在业务逻辑层（service层）使用
- @Repository在数据访问层（dao层）使用
- @Controller在展现层（MVC -> Spring MVC）使用

## 注入Bean注解

**Resource,Autowired,Inject**

| ANNOTATION |             PACKAGE              |    SOURCE    |
| :--------: | :------------------------------: | :----------: |
| @Resource  |         javax.annotation         | Java JSR-250 |
|  @Inject   |           javax.inject           | Java JSR-330 |
| @Autowired | org.springframework.bean.factory | Spring 2.5+  |

区别：

- @Resource

两个关键的属性：name－名称，type－类型

1. 如果指定了name,type，则从Spring容器中找一个名称和类型相当应的一个bean，找不到则报错。
2. 如果只指定了name，则从Spring容器中找一个名称和name一样的bean，找不到则报错。
3. 如果只指定了type，则从Spring容器中找一个类型和type一样的bean，找不到或者找到多个则报错。
4. 如果没有指定参数，则默认找字段名称装配，找不到则按类型装配，找不到则报错。

- @Autowired

1. 默认按类型装配，找不到或者找到多个则报错。 
2. 如果要按名称装配，需要结合Spring另外一个注解Qualifier("name")使用。
3. 默认必须装配requred=true，如果可以为空，可以设置为false，在Spring4+结合jdk8+的情况下还可以使用Optional和false同等的效果，如

```
@Autowired
private Optional<UserService> userService;
```

- @Inject

和@Autowired类似，可以完全代替@Autowired，但这个没有required属性，要求bean必须存在。

如果要按名称装配，需要结合javax另外一个注解N("name")使用。

@Autowired和@Inject基本是一样的，因为两者都是使用AutowiredAnnotationBeanPostProcessor来处理依赖注入。但是@Resource是个例外，它使用的是CommonAnnotationBeanPostProcessor来处理依赖注入。当然，两者都是BeanPostProcessor。

## PathVariable

可以使用@PathVariable注解方法参数并将其绑定到URI模板变量的值上。

- @PathVariable中的参数可以是任意的简单类型，如int,long,Date等等。Spring会自动将其转换成合适的类型或者抛出TypeMismatchException异常。当然，我们也可以注册支持额外的数据类型。
- @PathVariable使用`Map<String, String>`类型的参数时， Map会填充到所有的URI模板变量中。
- @PathVariable支持使用正则表达式，这就决定了它的超强大属性，它能在路径模板中使用占位符，可以设定特定的前缀匹配，后缀匹配等自定义格式。 
- @PathVariable还支持矩阵变量。

```
@RequestMapping(value="/owners/{ownerId}/pets/{petId}", method=RequestMethod.GET) 
public String findPet(@PathVariable String ownerId, @PathVariable String petId, Model model) { 		
	Owner owner = ownerService.findOwner(ownerId); 
	Pet pet = owner.getPet(petId); model.addAttribute("pet", pet); 
	return "displayPet"; 
}
```

## RequestParam

@RequestParam将请求的参数绑定到方法中的参数上。如果想自定义指定参数的话，将@RequestParam的 required 属性设置为false（如@RequestParam（value="id",required=false））。

@RequestParam和@PathVariable的区别就在于请求时当前参数是在url路由上还是在请求的body上

```
@RequestMapping(value="", method=RequestMethod.POST)
    public String postUser(@RequestParam(value="phoneNum", required=true) String phoneNum ) String userName) {
        userService.create(phoneNum, userName);
        return "success";
    }
```

请求url为 localhost:8080/xxx?phoneNum=13387654321。

## RequestHeader

@RequestHeader从Http请求头中提取指定的某个请求头。

下面是一个请求头：

```
Host        localhost:8080
Accept       text/html,application/xhtml+xml,application/xml;q=0.9
Accept-Language  fr,en-gb;q=0.7,en;q=0.3
Accept-Encoding   gzip,deflate
Accept-Charset   ISO-8859-1,utf-8;q=0.7,*;q=0.7
Keep-Alive     300
```

获取请求头

```
@GetMapping("header")
public String withHeader(@RequestHeader String Accept) {
   return "Obtained 'Accept' header '" + Accept + "'";
}
```

这是获取请求头中的Accept。如果目标方法参数不是字符串，那么就会自动进行类型转换。当注解`@RequestHeader`用在一个`Map`、`MultiValueMap`或者`HttpHeaders`参数上的时候，这个 map 就是收集所有请求头的值。

获取请求头，带参数

```
@RequestMapping("/displayHeaderInfo.do")
public void displayHeaderInfo(
        @RequestHeader("Accept-Encoding") String encoding,
        @RequestHeader("Keep-Alive") long keepAlive) 
{
    //...
}
```

## RequestBody

@RequestBody是指方法参数应该被绑定到HTTP请求Body上。该注解用于读取Request请求的body部分数据，根据HTTP Request Header的`content-Type`的内容，使用系统默认配置的HttpMessageConverter进行解析，然后把相应的数据绑定到要返回的对象上；再把HttpMessageConverter返回的对象数据绑定到 controller中方法的参数上。

```
@RequestMapping(value = "/something", method = RequestMethod.PUT)
public void handle(@RequestBody String body, Writer writer) throws IOException {
    writer.write(body);
}
```

## ResponseBody

它的作用是将返回类型直接输入到HTTP response body中。该注解用于将Controller的方法返回的对象，根据HTTP Request Header的`Accept`的内容，通过适当的`HttpMessageConverter`转换为指定格式后，写入到Response对象的body数据区。

```
@RequestMapping(value = "/something", method = RequestMethod.PUT)
@ResponseBody
public String helloWorld() {    
    return "Hello World";
}
```

## CookieValue

注解@CookieValue允许一个方法参数允许把一个方法参数绑定到一个 HTTP cookie 值上。

```
	@GetMapping("cookie")
	public String withCookie(@CookieValue String openid_provider) {
		return "Obtained 'openid_provider' cookie '" + openid_provider + "'";
	}
```

如果目标方法参数不是字符串，那么就会自动进行类型转换。

## ModelAttribute

@ModelAttribute可以作用在方法或方法参数上，当它作用在方法上时，标明该方法的目的是添加一个或多个模型属性（model attributes）。

```
@ModelAttribute
public Account addAccount(@RequestParam String number) {
    return accountManager.findAccount(number);
}

@ModelAttribute
public void populateModel(@RequestParam String number, Model model) {
    model.addAttribute(accountManager.findAccount(number));    
    // add more ...
}
```

- @ModelAttribute方法用来在model中填充属性，如填充下拉列表、宠物类型或检索一个命令对象比如账户（用来在HTML表单上呈现数据）。
- @ModelAttribute方法有两种风格：一种是添加隐形属性并返回它。另一种是该方法接受一个模型并添加任意数量的模型属性。用户可以根据自己的需要选择对应的风格。
- 当@ModelAttribute作用在方法参数上时，表明该参数可以在方法模型中检索到。如果该参数不在当前模型中，该参数先被实例化然后添加到模型中。一旦模型中有了该参数，该参数的字段应该填充所有请求参数匹配的名称中。
- @ModelAttribute是一种很常见的从数据库中检索属性的方法，它通过@SessionAttributes使用request请求存储。在一些情况下，可以很方便的通过URI模板变量和类型转换器检索属性。

## RequestAttribute

@RequestAttribute可以被用于访问由过滤器或拦截器创建的、预先存在的请求属性 

```
@RequestMapping("/reqAttr")
    public String handle(@RequestAttribute("reqStr") String str, Model model)
    {
        System.out.println("--> reqStr : " + str);
        model.addAttribute("sth", str);
        return "/examples/targets/test1";
    }
```

这是过滤拦截foo参数，无论传入的foo值是什么，最终都会转变成bar

```
	@ModelAttribute
	void beforeInvokingHandlerMethod(HttpServletRequest request) {
		request.setAttribute("foo", "bar");
	}
	
	@GetMapping("/data/custom")
	public String custom(@RequestAttribute("foo") String foo) {
		return "Got 'foo' request attribute value '" + foo + "'";
	}
```

## MatrixVariable

@MatrixVariable注解表明了在一个路径段中的方法参数应该被绑定到一个名值对中。矩阵变量可以出现在任何路径段,每个矩阵变量用“;”分隔。例如:“/汽车;颜色=红;年=2012”。多个值可以是“,”分隔“颜色=红,绿,蓝”或变量名称可以重复“颜色=红;颜色=绿色;颜色=蓝”

例子1：

```
@GetMapping("{path}/simple")
public String withMatrixVariable(@PathVariable String path, @MatrixVariable String foo) {
   return "Obtained matrix variable 'foo=" + foo + "' from path segment '" + path + "'";
}
```

请求格式："/data/matrixvars;foo=bar/simple" 对应关系为 matrixvars - > {path}; foo=bar -> String foo。

例子2：

```
@GetMapping("{path1}/{path2}")
public String withMatrixVariablesMultiple (
		@PathVariable String path1, @MatrixVariable(name="foo", pathVar="path1") String foo1,
		@PathVariable String path2, @MatrixVariable(name="foo", pathVar="path2") String foo2) {

	return "Obtained matrix variable foo=" + foo1 + " from path segment '" + path1
			+ "' and variable 'foo=" + foo2 + " from path segment '" + path2 + "'";
}
```

请求格式："/data/matrixvars;foo=bar1/multiple;foo=bar2" />" 

对应关系：matrixvars - > {path1}; foo=bar1 - > String foo1

​		    multiple -> {path2}；foo=bar2 - > String foo2

## HttpEntity

HttpEntity除了能获得request请求和response响应之外，它还能访问请求和响应头，如下所示：

```
@RequestMapping("/something") 
public ResponseEntity handle(HttpEntity<byte[]> requestEntity) throws UnsupportedEncodingException { 
	String requestHeader = requestEntity.getHeaders().getFirst("MyRequestHeader"));
	byte[] requestBody = requestEntity.getBody(); // do something with request header and body 	   HttpHeaders responseHeaders = new HttpHeaders(); 
	responseHeaders.set("MyResponseHeader", "MyValue");
	return new ResponseEntity("Hello World", responseHeaders, HttpStatus.CREATED); 
}
```

## Valid

@Valid注解进行数据验证功能，配合BindingResult可以直接提供参数验证结果。

```
	@PostMapping("/json")
	public String readJson(@Valid @RequestBody JavaBean bean) {
		return "Read from JSON: " + bean;
	}
```

例子：

```
@GetMapping("/validate")
	public String validate(@Valid JavaBean bean, BindingResult result) {
		if (result.hasErrors()) {
			return "Object has validation errors";
		} else {
			return "No errors";
		}
	}
```

```
public class JavaBean {
	
	@NotNull
	@Max(5)
	private Integer number;

	@NotNull
	@Future
	@DateTimeFormat(iso=ISO.DATE)
	private Date date;
}
```

@Valid：对JavaBean进行后台数据校验。@Valid 和 BindingResult 是一一对应的，否则spring会在校验不通过时直接抛出异常。如果有多个@Valid，那么每个@Valid后面跟着的BindingResult就是这个@Valid的验证结果，顺序不能乱。

## RedirectAttributes

RedirectAttributes 是专门用于重定向之后还能带参数跳转的工具类。

两种带参的方式：

- 第一种：redirectAttributes.addAttributie("prama",value); 这种方法相当于在重定向链接地址追加传递的参数，

```
redirectAttributes.addAttributie("prama1",value1);
redirectAttributes.addAttributie("prama2",value2);
return:"redirect：/path/list" 
```

​	以上重定向的方法等同于 return:"redirect：/path/list?prama1=value1&prama2=value2 "

- 第二种 redirectAttributes.addFlashAttributie("prama",value); 这种方法是隐藏了参数，链接地址上不直接暴露，但是能且只能在重定向的 “页面” 获取parma参数值。

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

## Model

Spring mvc支持如下的返回方式：

- ModelAndView
- Model
- ModelMap
- Map
- View
- String
- void

Model 是一个接口， 其实现类为ExtendedModelMap，继承了ModelMap类。

```
@RequestMapping("/demo2/show") 
    public Map<String, String> getMap() { 
        Map<String, String> map = new HashMap<String, String>(); 
        map.put("key1", "value-1"); 
        map.put("key2", "value-2"); 
        return map; 
    } 
```

在jsp页面中可直通过 ${key1} 获得到值, map.put()相当于request.setAttribute方法。

```
@GetMapping("html")
	public String prepare(Model model) {
		model.addAttribute("foo", "bar");
		model.addAttribute("fruit", "apple");
		return "views/html";
	}
```

## DateTimeFormat

@DateTimeFormat注解把2010-07-04转换成标准格式:

- pattern 属性：类型为字符串。指定解析/格式化字段数据的模式，如：”yyyy-MM-dd hh:mm:ss”
- iso 属性：类型为 DateTimeFormat.ISO。指定解析/格式化字段数据的ISO模式，包括四种：ISO.NONE（不使用） -- 默认、ISO.DATE(yyyy-MM-dd) 、ISO.TIME(hh:mm:ss.SSSZ)、ISO.DATE_TIME(yyyy-MM-dd hh:mm:ss.SSSZ)
- style 属性：字符串类型。通过样式指定日期时间的格式，由两位字符组成，第一位表示日期的格式，第二位表示时间的格式：S：短日期/时间格式、M：中日期/时间格式、L：长日期/时间格式、F：完整日期/时间格式、-：忽略日期或时间格式

当存在value值是values=1&values=2&values=3&values=4&values=5或者values=1,2,3,4,5，都可以先放入集合中。

values=2010-07-04,2011-07-04，把时间放入集合中。

```
	@GetMapping("date/{value}")
	public String date(@PathVariable @DateTimeFormat(iso=ISO.DATE) Date value) {
		return "Converted date " + value;
	}
```

请求格式：/convert/date/2010-07-04

## ExceptionHandler

统一异常处理

 1. 在 @ExceptionHandler 方法的入参中可以加入 Exception 类型的参数, 该参数即对应发生的异常对象
 2. @ExceptionHandler 方法的入参中不能传入 Map. 若希望把异常信息传导页面上, 需要使用 ModelAndView 作为返回值
 3. @ExceptionHandler 方法标记的异常有优先级的问题
 4. @ControllerAdvice: 如果在当前 Handler 中找不到 @ExceptionHandler 方法来出来当前方法出现的异常, 
     则将去 @ControllerAdvice 标记的类中查找 @ExceptionHandler 标记的方法来处理异常

```
    @ExceptionHandler({ArithmeticException.class})
    public ModelAndView handleArithmeticException(Exception ex){
        System.out.println("出异常了: " + ex);
        ModelAndView mv = new ModelAndView("error");
        mv.addObject("exception", ex);
        return mv;
    }
```

## SessionAttributes

默认情况下Spring MVC将模型中的数据存储到request域中。当一个请求结束后，数据就失效了。如果要跨页面使用。那么需要使用到session。而@SessionAttributes注解就可以使得模型中的数据存储一份到session域中。

参数：

1. names：这是一个字符串数组。里面应写需要存储到session中数据的名称。
2. types：根据指定参数的类型，将模型中对应类型的参数存储到session中 
3. value：其实和names是一样的。
