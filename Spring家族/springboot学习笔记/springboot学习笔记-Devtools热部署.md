# springboot学习笔记-Devtools热部署

所谓的热部署就是在你修改了后端代码后不需要手动重启，工具会帮你快速的自动重启是修改生效。

其深层原理是使用了两个`ClassLoader`，一个`Classloader`加载那些不会改变的类（第三方Jar包），另一个 `ClassLoader`加载会更改的类，称为`restart ClassLoader`，这样在有代码更改的时候，原来的`restart ClassLoader` 被丢弃，重新创建一个`restart ClassLoader`，由于需要加载的类相比较少，所以实现了较快的重启时间。

# IDEA配置

## 开启idea自动make功能

1. Settings --> Build --> Compiler -->  查找make project automatically /Build project automatically--> 选中 

![img](https://images2015.cnblogs.com/blog/824490/201704/824490-20170404213116003-699199821.png)

2. CTRL + SHIFT + A --> 输入Registry --> 找到并勾选compiler.automake.allow.when.app.running 

![img](https://images2015.cnblogs.com/blog/824490/201704/824490-20170404213933222-1782823544.png)

3. 重启idea 

## Chrome禁用缓存  

F12（或Ctrl+Shift+J或Ctrl+Shift+I）--> NetWork --> Disable Cache(while DevTools is open) 

![img](https://images2015.cnblogs.com/blog/824490/201704/824490-20170404213518191-1967052809.png)

至此，在idea中就可以愉快的修改代码了，修改后可以及时看到效果，无须手动重启和清除浏览器缓存。

# 引入Devtools

搭建一个简单的Spring Boot项目，然后引入Spring-Boot-devtools依赖：

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
    <optional>true</optional>
</dependency>
```

开启热部署

```
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <-输入该配置->
            <configuration>
                <fork>true</fork>
            </configuration>
        </plugin>
    </plugins>
</build>
```

devtools会监听classpath下的文件变动，并且会立即重启应用（发生在保存时机），因为其采用的虚拟机机制，该项重启是很快的。

# 测试热部署

在入口类中添加一个方法，用于热部署测试：

```
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@SpringBootApplication
public class DemoApplication {
    @RequestMapping("/")
    String index() {
        return "hello spring boot";
    }
    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }
}

```

启动项目访问[http://localhost:8080/](http://localhost:8080/)，页面输出hello spring boot。

将方法的返回值修改为hello world并在保存的瞬间，应用便重启好了，刷新页面，内容也将得到更改。

# 所有配置

下面是所有Devtools在Spring Boot中的可选配置:

```
# Whether to enable a livereload.com-compatible server.
spring.devtools.livereload.enabled=true 

# Server port.
spring.devtools.livereload.port=35729 

# Additional patterns that should be excluded from triggering a full restart.
spring.devtools.restart.additional-exclude= 

# Additional paths to watch for changes.
spring.devtools.restart.additional-paths= 

# Whether to enable automatic restart.
spring.devtools.restart.enabled=true

# Patterns that should be excluded from triggering a full restart.
spring.devtools.restart.exclude=META-INF/maven/**,META-INF/resources/**,resources/**,static/**,public/**,templates/**,**/*Test.class,**/*Tests.class,git.properties,META-INF/build-info.properties

# Whether to log the condition evaluation delta upon restart.
spring.devtools.restart.log-condition-evaluation-delta=true 

# Amount of time to wait between polling for classpath changes.
spring.devtools.restart.poll-interval=1s 

# Amount of quiet time required without any classpath changes before a restart is triggered.
spring.devtools.restart.quiet-period=400ms 

# Name of a specific file that, when changed, triggers the restart check. If not specified, any classpath file change triggers the restart.
spring.devtools.restart.trigger-file=
```

























