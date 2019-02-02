# springboot学习笔记-spring注解

`@Repository`, `@Service`, `@Controller`就是针对不同的使用场景所采取的特定功能化的注解组件。

`@Service`, `@Controller` , `@Repository` = {`@Component` + 一些特定的功能}。

|     注解      |            含义             |
| :---------: | :-----------------------: |
| @Component  | 最普通的组件，可以被注入到spring容器进行管理 |
| @Repository |          作用于持久层           |
|  @Service   |         作用于业务逻辑层          |
| @Controller |   作用于表现层（spring-mvc的注解）   |

注：以下两个注解不能被替换

- `@Controller` 注解的bean会被spring-mvc框架所使用。 
- `@Repository` 会被作为持久层操作（数据库）的bean来使用 

详细说明：

1. `@Controller`注解类进行前端请求的处理，转发，重定向。包括调用Service层的方法 
2. `@Service`注解类处理业务逻辑 
3. `@Repository`注解类作为DAO对象（数据访问对象，Data Access Objects），这些类可以直接对数据库进行操作 
4. `@Component`注解定义Spring管理Bean













