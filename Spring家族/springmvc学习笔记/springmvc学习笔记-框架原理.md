# springmvc学习笔记(1)_框架原理

## Bean定义

每一个被Spring管理的Java对象都称之为Bean；而Spring提供了一个IOC容器用来初始化对象，解决对象间的依赖管理和对象的使用

## springmvc框架原理

组件及其作用

- 前端控制器(DispatcherServlet)：接收请求，响应结果，相当于转发器，中央处理器。减少了其他组件之间的耦合度
- 处理器映射器(HandlerMapping)：根据请求的url查找Handler
- **Handler处理器**：按照HandlerAdapter的要求编写
- 处理器适配器(HandlerAdapter)：按照特定规则(HandlerAdapter要求的规则)执行Handler。
- 视图解析器(ViewResolver)：进行视图解析，根据逻辑视图解析成真正的视图(View)
- **视图(View)**：View是一个接口实现类试吃不同的View类型（jsp,pdf等等）

*注：其中加粗的为需要程序员开发的，没加粗的为不需要程序员开发的*



![spring-struct](https://camo.githubusercontent.com/8ebb2bb00ed83a8fa5d289f07e29b9c0a5d78da4/687474703a2f2f3778706836642e636f6d312e7a302e676c622e636c6f7564646e2e636f6d2f737072696e676d76635f2545362541302542382545352542462538332545362539452542362545362539452538342545352539422542452e6a7067)

步骤：

1. 发起请求到前端控制器(`DispatcherServlet`)
2. 前端控制器请求处理器映射器(`HandlerMapping`)查找`Handler`(可根据xml配置、注解进行查找)
3. 处理器映射器(`HandlerMapping`)向前端控制器返回`Handler`
4. 前端控制器调用处理器适配器(`HandlerAdapter`)执行`Handler`
5. 处理器适配器(HandlerAdapter)去执行Handler
6. Handler执行完，给适配器返回ModelAndView(Springmvc框架的一个底层对象)
7. 处理器适配器(`HandlerAdapter`)向前端控制器返回`ModelAndView`
8. 前端控制器(`DispatcherServlet`)请求视图解析器(`ViewResolver`)进行视图解析，根据逻辑视图名解析成真正的视图(jsp)
9. 视图解析器(ViewResolver)向前端控制器(`DispatcherServlet`)返回View
10. 前端控制器进行视图渲染，即将模型数据(在`ModelAndView`对象中)填充到request域
11. 前端控制器向用户响应结果