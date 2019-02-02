# springboot学习笔记-使用模板引擎渲染web视图

# 静态资源访问

在我们开发Web应用的时候，需要引用大量的js、css、图片等静态资源。在Spring Boot中，默认的html页面地址为src/main/resources/templates，默认的静态资源地址为src/main/resources/static。目录名需符合如下规则：

- resources
  - static
  - templates

通过@RestController来处理请求，所以返回的内容为json对象。需要渲染html页面的时候，使用模板引擎也可以完美解决。

# 模板引擎

Spring Boot提供了默认配置的模板引擎主要有以下几种：

- Thymeleaf
- FreeMarker
- Velocity
- Groovy
- Mustache

Spring Boot建议使用这些模板引擎，避免使用JSP。当使用上述模板引擎中的任何一个，它们默认的模板配置路径为：src/main/resources/templates。

Spring Boot使用模板引擎主要步骤为：

1. pom.xml 添加依赖

```
<!-- thymeleaf 模板引擎-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```

2. 如需修改 thymeleaf 的默认配置，可以在application.properties中添加

```
# 是否启用thymeleaf模板解析
spring.thymeleaf.enabled=true
# 是否开启模板缓存（建议：开发环境下设置为false，生产环境设置为true）
spring.thymeleaf.cache=false 
# Check that the templates location exists.
spring.thymeleaf.check-template-location=true 
# 模板的媒体类型设置，默认为text/html
spring.thymeleaf.content-type=text/html
# 模板的编码设置，默认UTF-8
spring.thymeleaf.encoding=UTF-8
......
```

3. 编写controller

```
@Controller
public class HomeController {

    @RequestMapping("/home")
    public String home(ModelMap modelMap) {
    	 modelMap.addAttribute("host", "http://github.com");
        // return模板文件的名称，对应src/main/resources/templates/home.html
        return "home";  
    }
}
```

4. 编写html代码

```
<html xmlns="http://www.w3.org/1999/xhtml" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8"/>
    <title>Home</title>
</head>
<body>
    <h1 th:text="${host}">Hello World</h1>
</body>
</html>
```

5. 启动应用，访问`http://localhost:8080/home `


# Thymeleaf默认配置

在Spring Boot配置文件中可对Thymeleaf的默认配置进行修改：

```
#开启模板缓存（默认值：true）
spring.thymeleaf.cache=true 
#Check that the template exists before rendering it.
spring.thymeleaf.check-template=true 
#检查模板位置是否正确（默认值:true）
spring.thymeleaf.check-template-location=true
#Content-Type的值（默认值：text/html）
spring.thymeleaf.content-type=text/html
#开启MVC Thymeleaf视图解析（默认值：true）
spring.thymeleaf.enabled=true
#模板编码
spring.thymeleaf.encoding=UTF-8
#要被排除在解析之外的视图名称列表，用逗号分隔
spring.thymeleaf.excluded-view-names=
#要运用于模板之上的模板模式。另见StandardTemplate-ModeHandlers(默认值：HTML5)
spring.thymeleaf.mode=HTML5
#在构建URL时添加到视图名称前的前缀（默认值：classpath:/templates/）
spring.thymeleaf.prefix=classpath:/templates/
#在构建URL时添加到视图名称后的后缀（默认值：.html）
spring.thymeleaf.suffix=.html
#Thymeleaf模板解析器在解析器链中的顺序。默认情况下，它排第一位。顺序从1开始，只有在定义了额外的TemplateResolver Bean时才需要设置这个属性。
spring.thymeleaf.template-resolver-order=
#可解析的视图名称列表，用逗号分隔
spring.thymeleaf.view-names=
```

一般开发中将`spring.thymeleaf.cache`设置为false，其他保持默认值即可。




