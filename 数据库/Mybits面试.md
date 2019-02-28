# Mybits面试

**问题：JDBC编程有哪些不足之处，Mybatis是如何解决这些问题的？**

1） 数据库连接的创建、释放频繁造成系统资源浪费从而影响了性能，如果使用数据库连接池就可以解决这个问题。当然JDBC同样能够使用数据源。
解决：在SQLMapConfig.xml中配置数据连接池，使用数据库连接池管理数据库连接。

2） SQL语句在写代码中不容易维护，事件需求中SQL变化的可能性很大，SQL变动需要改变JAVA代码。解决：将SQL语句配置在mapper.xml文件中与java代码分离。

3） 向SQL语句传递参数麻烦，因为SQL语句的where条件不一定，可能多，也可能少，占位符需要和参数一一对应。解决：Mybatis自动将java对象映射到sql语句。

4） 对结果集解析麻烦，sql变化导致解析代码变化，且解析前需要遍历，如果能将数据库记录封装成pojo对象解析比较方便。解决：Mbatis自动将SQL执行结果映射到java对象。

**问题： Mybatis编程步骤 ？**

Step1：创建SQLSessionFactory

 Step2：通过SQLSessionFactory创建SQLSession 

Step3：通过SQLSession执行数据库操作 

Step4：调用session.commit()提交事物 

Step5：调用session.close()关闭会话

**问题： MyBatis与hibernate有哪些不同 ？**

 1）Mybatis MyBatis 是支持定制化 SQL、存储过程以及高级映射的一种持久层框架。MyBatis 避免了几乎所有的 JDBC 代码和手动设置参数以及获取结果集。Mybatis它不完全是一个ORM(对象关系映射)框架；它需要程序员自己编写部分SQL语句。 mybatis可以通过xml或者注解的方式灵活的配置要运行的SQL语句，并将java对象和SQL语句映射生成最终的执行的SQL，最后将SQL执行的结果在映射生成java对象。 Mybatis程序员可以直接的编写原生态的SQL语句，可以控制SQL执行性能，灵活度高，适合软件需求变换频繁的企业。 缺点：Mybatis无法做到数据库无关性，如果需要实现支持多种数据库的软件，则需要自定义多套SQL映射文件，工作量大。

 2） Hibernate Hibernate是支持定制化 SQL、存储过程以及高级映射的一种持久层框架。 Hibernate对象-关系映射能力强，数据库的无关性好，Hirberate可以自动生成SQL语句，对于关系模型要求高的软件，如果用HIrbernate开发可以节省很多时间。

**问题： 使用Mybatis的mapper接口调用时候有哪些要求？**

1) Mapper接口方法名和Mapper.xml中定义的每个SQL的id相同；

 2) Mapper接口方法的输入参数类型和mapper.xml中定义的每个sqlparameterType类型相同 3) Mapper接口方法的输入输出参数类型和mapper.xml中定义的每个sql的resultType的类型相同 4) Mapper.xml文件中的namespace，就是接口的类路径。

**问题： Mybatis的一级缓存和二级缓存？**

1）一级缓存 Mybatis的一级缓存是指SQLSession，一级缓存的作用域是SQlSession, Mabits默认开启一级缓存。 在同一个SqlSession中，执行相同的SQL查询时；第一次会去查询数据库，并写在缓存中，第二次会直接从缓存中取。 当执行SQL时候两次查询中间发生了增删改的操作，则SQLSession的缓存会被清空。 每次查询会先去缓存中找，如果找不到，再去数据库查询，然后把结果写到缓存中。 Mybatis的内部缓存使用一个HashMap，key为hashcode+statementId+sql语句。Value为查询出来的结果集映射成的java对象。 SqlSession执行insert、update、delete等操作commit后会清空该SQLSession缓存。

2）二级缓存 二级缓存是mapper级别的，Mybatis默认是没有开启二级缓存的。 第一次调用mapper下的SQL去查询用户的信息，查询到的信息会存放代该mapper对应的二级缓存区域。 第二次调用namespace下的mapper映射文件中，相同的sql去查询用户信息，会去对应的二级缓存内取结果。 如果调用相同namespace下的mapepr映射文件中增删改sql，并执行了commit操作，此时会情况该

**问题： Mapper编写有几种方式 ？** 方式1：接口实现类集成SQLSessionDaoSupport 此方法需要编写mapper接口，mapper接口的实现类,mapper.xml文件。 方式2：使用org.mybatis.spring.mapper.MapperFactoryBean 此方法需要在SqlMapConfig.xml中配置mapper.xml的位置，还需定义mapper接口。 方式3：使用mapper扫描器 需要编写mapper.xml文件，需要mapper接口，配置mapper扫描器，使用扫描器从spring容器中获取mapper的实现对象。

**问题： Mybatis的映射文件 ？**

Mybatis 真正强大的在于它的映射文件，它和JDBC代码进行比较，可以省掉95%的代码，Mybatis就是针对SQL进行构建。 SQL映射文件中几个顶级的元素： • cache – 给定命名空间的缓存配置。 • cache-ref – 其他命名空间缓存配置的引用。 • resultMap – 是最复杂也是最强大的元素，用来描述如何从数据库结果集中来加载对象。 • sql – 可被其他语句引用的可重用语句块。 • insert – 映射插入语句 • update – 映射更新语句 • delete – 映射删除语句 • select – 映射查询语句

**问题: Mybatis动态SQL？**

1) 传统的JDBC的方法，在组合SQL语句的时候需要去拼接，稍微不注意就会少少了一个空格，标点符号，都会导致系统错误。Mybatis的动态SQL就是为了解决这种问题而产生的；Mybatis的动态SQL语句值基于OGNL表达式的，方便在SQL语句中实现某些逻辑；可以使用标签组合成灵活的sql语句，提供开发的效率。 2) Mybatis的动态SQL标签主要由以下几类： If语句（简单的条件判断） Choose(when/otherwise),相当于java语言中的switch，与jstl中choose类似 Trim(对包含的内容加上prefix，或者suffix) Where(主要是用来简化SQL语句中where条件判断，能智能的处理and/or 不用担心多余的语法导致的错误) Set(主要用于更新时候) Foreach(一般使用在mybatis in语句查询时特别有用)

**问题： Mybais 常用注解 ？**

```
@Insert ： 插入sql , 和xml insert sql语法完全一样
@Select ： 查询sql,  和xml select sql语法完全一样
@Update ： 更新sql,  和xml update sql语法完全一样
@Delete ： 删除sql,  和xml delete sql语法完全一样
@Param ：  入参@Results ：结果集合@Result ： 结果
```

