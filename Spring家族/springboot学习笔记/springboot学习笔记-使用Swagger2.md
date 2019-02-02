# springboot学习笔记-使用Swagger2

# 添加Swagger2依赖

在 pom.xml 中加入Swagger2的依赖

```
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger2</artifactId>
    <version>2.2.2</version>
</dependency>
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger-ui</artifactId>
    <version>2.2.2</version>
</dependency>
```

# 创建User类

```
public class User {

    private Long id;
    private String name;
    private Integer age;

    public Long getId() {
        return id;
    }
    public void setId(Long id) {
        this.id = id;
    }
    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
    public Integer getAge() {
        return age;
    }
    public void setAge(Integer age) {
        this.age = age;
    }
}
```

# 创建Swagger2配置类

在`Application.java`同级创建Swagger2的配置类`Swagger2`。

```
@Configuration
@EnableSwagger2
public class Swagger2 {

    @Bean
    public Docket createRestApi() {
        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(apiInfo())
                .select()
                //包位置一定要正确
                .apis(RequestHandlerSelectors.basePackage("com.orangehaswing.swagger2.web")) 
                .paths(PathSelectors.any())
                .build();
    }

    private ApiInfo apiInfo() {
        return new ApiInfoBuilder()
                .title("Spring BootAPIs")
                .description("springboot technology")
                .termsOfServiceUrl("http://github.com/")
                .contact("orangehaswig")
                .version("1.0")
                .build();
    }

}
```

通过 `@Configuration`注解，让Spring来加载该类配置。再通过`@EnableSwagger2`注解来启用Swagger2。

再通过`createRestApi`函数创建`Docket`的Bean之后，`apiInfo()`用来创建该Api的基本信息（这些基本信息会展现在文档页面中）。`select()`函数返回一个`ApiSelectorBuilder`实例用来控制哪些接口暴露给Swagger来展。

本例采用指定扫描的包路径来定义，Swagger会扫描该包下所有Controller定义的API，并产生文档内容（除了被`@ApiIgnore`指定的请求）。

# 添加文档内容

通过`@ApiOperation`注解来给API增加说明、通过`@ApiImplicitParams`、`@ApiImplicitParam`注解来给参数增加说明。

```
@RestController
@RequestMapping(value="/users")
public class UserController {
    static Map<Long, User> users = Collections.synchronizedMap(new HashMap<Long, User>());

    @ApiOperation(value="获取用户列表", notes="")
    @RequestMapping(value={""}, method=RequestMethod.GET)
    public List<User> getUserList() {
        List<User> r = new ArrayList<User>(users.values());
        return r;
    }

    @ApiOperation(value="创建用户", notes="根据User对象创建用户")
    @ApiImplicitParam(name = "user", value = "用户详细实体user", required = true, dataType = "User")
    @RequestMapping(value="", method=RequestMethod.POST)
    public String postUser(@RequestBody User user) {
        users.put(user.getId(), user);
        return "success";
    }

    @ApiOperation(value="获取用户详细信息", notes="根据url的id来获取用户详细信息")
    @ApiImplicitParam(name = "id", value = "用户ID", required = true, dataType = "Long", paramType = "path")
    @RequestMapping(value="/{id}", method=RequestMethod.GET)
    public User getUser(@PathVariable Long id) {
        return users.get(id);
    }

    @ApiOperation(value="更新用户详细信息", notes="根据url的id来指定更新对象，并根据传过来的user信息来更新用户详细信息")
    @ApiImplicitParams({
            @ApiImplicitParam(name = "id", value = "用户ID", required = true, dataType = "Long", paramType = "path"),
            @ApiImplicitParam(name = "user", value = "用户详细实体user", required = true, dataType = "User")
    })
    @RequestMapping(value="/{id}", method=RequestMethod.PUT)
    public String putUser(@PathVariable Long id, @RequestBody User user) {
        User u = users.get(id);
        u.setName(user.getName());
        u.setAge(user.getAge());
        users.put(id, u);
        return "success";
    }

    @ApiOperation(value="删除用户", notes="根据url的id来指定删除对象")
    @ApiImplicitParam(name = "id", value = "用户ID", required = true, dataType = "Long", paramType = "path")
    @RequestMapping(value="/{id}", method=RequestMethod.DELETE)
    public String deleteUser(@PathVariable Long id) {
        users.remove(id);
        return "success";
    }
}
```

# 不使用Swagger2

```
@RestController
public class HelloController {

    @ApiIgnore  //使用注释忽略
    @RequestMapping(value = "/hello",method = RequestMethod.GET)
    public String index(){
        return "hello world";
    }
}
```

# 测试

启动Spring Boot程序，访问：`http://localhost:8080/swagger-ui.html`。就能看到前文所展示的RESTful API的页面。

# Swagger常用注解

- `@Api`：修饰整个类，描述Controller的作用；
- `@ApiOperation`：描述一个类的一个方法，或者说一个接口；
- `@ApiParam`：单个参数描述；
- `@ApiModel`：用对象来接收参数；
- `@ApiProperty`：用对象接收参数时，描述对象的一个字段；
- `@ApiResponse`：HTTP响应其中1个描述；
- `@ApiResponses`：HTTP响应整体描述；
- `@ApiIgnore`：使用该注解忽略这个API；
- `@ApiError` ：发生错误返回的信息；
- `@ApiImplicitParam`：一个请求参数；
- `@ApiImplicitParams`：多个请求参数。































