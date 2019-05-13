# MyBatis整体架构解析

MyBatis的整体分为三层，分别是基础支持层，核心处理层和接口层，如图所示

![img](https://upload-images.jianshu.io/upload_images/13467292-84d8279f21aeff9f?imageMogr2/auto-orient/strip%7CimageView2/2/w/640/format/webp)

## 1. 基础支持层

基础支持层包含整个MyBatis的基础模块，这些模块为核心处理层的功能提供了良好的支撑，下面简单描述下各个模块。

### 1 数据源模块

数据源是实际开发中常用的组件之一，现在开源的数据源都提供了比较丰富的功能，例如连接池功能、检测链接状态等，选择性能优秀的数据源组件对于提升ORM框架乃至整个应用的性能都是非常重要的。MyBatis自身提供了相应的数据源实现。当然MyBatis也提供了与第三方接口数据源集成的接口，这些功能都位于数据源模块之中。

### 2 事务管理模块

MyBatis对数据库中的事务进行了抽象，其自身提供了相应的事务接口和简单的实现，在很多场景中，MyBatis会与Spring框架集成，并由Spring框架管理事务相关配置。

### 3 缓存模块

在优化系统性能时，优化数据库性能是非常重要的一个环节，而添加缓存则是优化数据库时最有效的手段之一。正确、合理的使用缓存可以将一部分数据库请求拦截在缓存这一层，这就能够减少相当一部分数据库的压力。

MyBatis中提供了一级缓存和二级缓存，而这两级缓存都依赖于基础支持层中的缓存模块实现，这里需要读者注意的是MyBatis中自带的两级缓存以及整个应用是运行在一个JVM中的，共享一块堆内存，如果这两级缓存中的数据量较大，则可能影响系统中其他功能的运行，所以当需要缓存大量数据时，优先考虑使用Redis、Mongodb、Memcache等缓存产品。

### 4 Binding模块

在调用 SqISession 相应方法执行数据库操作时，需要指定映射文件中定义的 SQL 节点，如果出现拼写错误，我们只能在运行时才能发现相应的 异常 。 为了尽早发现这种错误， MyBatis 通过 Binding 模块将用户自定义的 Mapper 接 口与映射配置文件关联起来，系统可以通过调用自定义 Mapper 接口中的方法执行相应的 SQL 语句完成数据库操作，从而避免上述问题。值得读者注意的是，开发人员无须编写自定义 Mapper接口的实现， MyBatis会自动为 其创建动态代理对象。在有些场景中，自定义 Mapper接口可以完全代替映射配置文件， 但有的映射规则和 SQL 语句的定义还是写在映射配置文件中比较方便，例如动态 SQL 语句的定义 。

### 5 反射模块

Java中的反射功能虽然强大，但对大多数开发人员来说，写出高质量的反射代码还是有一定难度的。MyBatis中专门提供了反射模块，该模块对Java原生的反射进行了一系列优化，例如缓存了类的元数据，提高了反射的性能。

### 6 类型转换模块

MyBatis 为简化配置文件提供了别名机制 ， 该机制是类型转换模 块的主要功能之一 。 类型转换模块的另一个功能是实现 JDBC 类型与 Java 类型之间的 转换，该功能在为 SQL 语句绑定实参以及 映射查询结果集 时都会涉及。在为 SQL 语 句绑定实参时， 会将数据由 Java类型转换成 JDBC 类型;而在映射结果集时，会将数 据由 JDBC类型转换成 Java类型。

### 7 日志模块

无论在开发测试环境中，还是在线上生产环境中，日志在整个系统中的地位都是非常重要的。良好的日志功能可以帮助开发人员和测试人员快速定位 Bug代码，也可以帮助运维人员快速定位性能瓶颈、等问题 。 目前的 Java 世界中存在很多优秀的日志框架，例如 Log4j、 Log4j2, slf4j等。 MyBatis作为一个设计优良的框架，除了提供详细的日志输出信息，还要能够集成多种日志框架，其日志模块的 一个主要功能就是集成第三方日志框架。

### 8 资源加载模块

资源加载模块主要是对类加载器进行封装，确定类的使用顺序，并提供了加载类文件以及其他资源文件的功能。

### 9 解析器模块

解析器模块主要提供了两个功能：一个功能是对XPath进行封装，为MyBatis初始化时解析mybatis-config.xml配置文件以及映射配置文件提供支持；另一个功能是为处理动态sql语句中的占位符提供支持。

## 2. 核心处理层

在MyBatis的核心处理层中实现了MyBatis的核心处理流程，其中包括MyBatis的初始化以及完成一次数据库操作的全部流程，而这些都是基于基础支持层实现的。

### 1 配置解析

在 MyBatis 初始化过程中，会加载 mybatis-config.xml 配置文件、映射配置文件以及 Mapper 接口中的注解信息，解析后的配置信息会形成相应的对象并保存到 Configuration 对象中 。例如，节点(即ResultSet 的映射规则) 会被解析成 ResultMap 对象，定义的节点(即属性映射)会被解析成 ResultMapping对象。之后，利用该 Configuration对象创SqlSessionFactor对象。待 MyBatis 初始化之后，开发人员可以通过初始化得到SqlSessionFactory创建 SqlSession 对象并完成数据库操作。

### 2 参数映射-SQL解析

拼凑 SQL 语句是一件烦琐且易出错的过程，为了将开发人员从这项枯燥无趣的工作中解脱出来，MyBatis实现动态SQL语句的功能，提供了多种动态 SQL语句对应的节点， 例如，节点、节点、节点等 。通过这些节点的组合使用，开发人员可以写出几乎满足所有需求的动态 SQL语句。 MyBatis 中的scripting模块会根据用户传入的实参，解析映射文件中定义的动态SQL节点，并形成数据库可执行的SQL 语句 。之后会处理 SQL 语句中的占位符，绑定用户传入的实参。

### 3 SQL执行

SQL语句的执行涉及多个组件，其中比较重要的是Executor、StatementHandler、ParameterHandler和ResultSetHandler。Executor主要负责维护一级缓存和二级缓存，并提供事务管理的相关操作，它会将数据库相关操作委托给StatementHandler完成。StatementHandler首先通过ParamHandler完成SQL语句的实参绑定，然后通过java.sql.Statement对象执行SQL语句并得到结果集，最后通过ResultSetHandler完成结果集的映射，得到结果对象并返回。如图展示一条sql的执行过程：

![img](https://upload-images.jianshu.io/upload_images/13467292-529eaf66d99826da?imageMogr2/auto-orient/strip%7CimageView2/2/w/640/format/webp)

### 4 插件

Mybatis 自身的功能虽然强大，但是并不能完美切 合所有 的应用场景，因此 MyBatis 提供了插件接口，我们可以通过添加用户自定义插件的方式对 MyBatis 进行扩展。用 户自定义插件也可以改变 Mybatis 的默认行为 ，例如，我们可以 拦截 SQL 语句并对其 进行重写。由于用户自定义插件会影响 MyBatis 的核心行为，在使用自定义插件之前， 开发人员需要了解 MyBatis 内部的原理，这样才能编写出安全、高效的插件。

## 3. MyBatis 核心类介绍

- SqlSession: 作为MyBatis工作的主要顶层API，表示和数据库交互的会话，完成必要数据库增删改查功能
- Executor: MyBatis执行器，是MyBatis 调度的核心，负责SQL语句的生成和查询缓存的维护
- StatementHandler: 封装了JDBC Statement操作，负责对JDBC statement 的操作，如设置参数、将Statement结果集转换成List集合。
- ParameterHandler: 负责对用户传递的参数转换成JDBC Statement 所需要的参数，
- ResultSetHandler: 负责将JDBC返回的ResultSet结果集对象转换成List类型的集合；
- TypeHandler:负责java数据类型和jdbc数据类型之间的映射和转换
- MappedStatement: 维护了一条节点的封装，
- SqlSource: 负责根据用户传递的parameterObject，动态地生成SQL语句，将信息封装到BoundSql对象中，并返回
- BoundSql: 表示动态生成的SQL语句以及相应的参数信息
- Configuration: MyBatis所有的配置信息都维持在Configuration对象之中。

## 4. MyBatis执行流程

![img](https://upload-images.jianshu.io/upload_images/13467292-17ec3cb3a0970ca3?imageMogr2/auto-orient/strip%7CimageView2/2/w/361/format/webp)

