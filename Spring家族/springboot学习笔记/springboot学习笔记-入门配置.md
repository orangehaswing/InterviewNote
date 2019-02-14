# SpringBoot学习笔记-入门配置

Spring Boot使用“习惯优于配置”（项目中存在大量的配置，此外还内置一个习惯性的配置，让你无需手动进行配置）的理念让你的项目快速运行起来。

## 引入Web模块

当前的pom.xml内容如下，仅引入了两个模块：

- spring-boot-starter：核心模块，包括自动配置支持、日志和YAML
- spring-boot-starter-test：测试模块，包括JUnit、Hamcrest、Mockito

```
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter</artifactId>
	</dependency>

	<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-test</artifactId>
		<scope>test</scope>
	</dependency>
</dependencies>
```

引入 Web 模块，需添加 spring-boot-starter-web 模块：

```
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

## HelloWorld服务

- 创建package：在com.orangehaswing.springboot下创建web包
- 创建HelloWorld类

```
@RestController
public class HelloController {
    @GetMapping("/hello")
    public String index() {
        return "Hello World";
    }
}
```

- 启动主程序，打开浏览器访问`http://localhost:8080/hello`，可以看到页面输出`Hello World`

## 属性配置文件

SpringBoot中默认的从application.properties文件中加载参数。也可以使用yml类型的配置文件代替properties文件。

1. 应用配置文件(.properties或.yml)
2. 当前目录下的/config子目录
3. 当前目录
4. 一个classpath下的/config包
5. classpath根路径（root）

按优先级排序，位置高的将覆盖位置低的

在application.properties下自定义一些属性

```
test.name=mrbird's blog
test.age=18
test.tel=Spring Boot
```

2. 加载到类文件中

   在类上添加@Component注解，在类域属性上通过@Value("${xxx}")指定关联属性，Spring Application会自动加载。

   ```
   @Component
   public class Configurations {
       @Value("${test.name}")
       private String name;

       @Value("${test.age}")
       private String age;

       @Value("${test.tel}")
       private Long tel;
   }
   ```



## 定制Banner

Spring Boot项目在启动的时候会有一个默认的启动图案：

```
 .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::        (v1.5.9.RELEASE)
```

我们可以把这个图案修改为自己想要的。

- 在src/main/resources目录下新建banner.txt文件
- ASCII图案可通过网站[http://www.network-science.de/ascii/](http://www.network-science.de/ascii/)一键生成
- 将生成的图案黏贴进txt文档即可

banner也可以关闭，在main方法中：

```
public static void main(String[] args) {
    SpringApplication app = new SpringApplication(DemoApplication.class);
    app.setBannerMode(Mode.OFF);
    app.run(args);
}
```

























