# springboot学习笔记-使用MongoDB配置

# MongoDB简介

MongoDB是一个基于分布式文件存储的数据库，它是一个介于关系数据库和非关系数据库之间的产品，其主要目标是在键/值存储方式（提供了高性能和高度伸缩性）和传统的RDBMS系统（具有丰富的功能）之间架起一座桥梁，它集两者的优势于一身。

较常见的，我们可以直接用MongoDB来存储键值对类型的数据，如：验证码、Session等；由于MongoDB的横向扩展能力，也可以用来存储数据规模会在未来变的非常巨大的数据，如：日志、评论等；由于MongoDB存储数据的弱类型，也可以用来存储一些多变json数据，如：与外系统交互时经常变化的JSON报文。而对于一些对数据有复杂的高事务性要求的操作，如：账户交易等就不适合使用MongoDB来存储。

# 配置工程

## 引入依赖：

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-mongodb</artifactId>
</dependency>
```

## 快速开始

若MongoDB的安装配置采用默认端口，那么在自动配置的情况下，我们不需要做任何参数配置，就能马上连接上本地的MongoDB。下面直接使用`spring-data-mongodb`来尝试对mongodb的存取操作。

- 创建要存储的User实体，包含属性：id、username、age

```
public class User {

    @Id
    private Long id;

    private String username;
    private Integer age;

    public User(Long id, String username, Integer age) {
        this.id = id;
        this.username = username;
        this.age = age;
    }

    // 省略getter和setter
}
```

- 实现User的数据访问对象：UserRepository

```
public interface UserRepository extends MongoRepository<User, Long> {
    
    User findByUsername(String username);
}
```

- 在单元测试中调用

```
@RunWith(SpringJUnit4ClassRunner.class)
@SpringApplicationConfiguration(Application.class)
public class ApplicationTests {

	@Autowired
	private UserRepository userRepository;

	@Before
	public void setUp() {
		userRepository.deleteAll();
	}

	@Test
	public void test() throws Exception {

		// 创建三个User，并验证User总数
		userRepository.save(new User(1L, "didi", 30));
		userRepository.save(new User(2L, "mama", 40));
		userRepository.save(new User(3L, "kaka", 50));
		Assert.assertEquals(3, userRepository.findAll().size());

		// 删除一个User，再验证User总数
		User u = userRepository.findOne(1L);
		userRepository.delete(u);
		Assert.assertEquals(2, userRepository.findAll().size());

		// 删除一个User，再验证User总数
		u = userRepository.findByUsername("mama");
		userRepository.delete(u);
		Assert.assertEquals(1, userRepository.findAll().size());
	}
}
```

# 参数配置

应用服务器与MongoDB通常不会部署于同一台设备之上，这样就无法使用自动化的本地配置来进行使用。这个时候，我们也可以方便的配置来完成支持，只需要在application.properties中加入mongodb服务端的相关配置，具体示例如下：

```
spring.data.mongodb.uri=mongodb://name:pass@localhost:27017/test
```

在尝试此配置时，记得在mongo中对test库创建具备读写权限的用户（用户名为name，密码为pass），不同版本的用户创建语句不同，注意查看文档做好准备工作

# 增强MongoDB

## 引入依赖

在使用了`spring-boot-starter-data-mongodb`的项目中，增加以下依赖

```
<dependency>
    <groupId>com.spring4all</groupId>
    <artifactId>mongodb-plus-spring-boot-starter</artifactId>
    <version>1.0.0.RELEASE</version>
</dependency>
```

## 增加注解

在应用主类上增加`@EnableMongoPlus`注解，比如：

```
@EnableMongoPlus
@SpringBootApplication
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

## 可用配置参数

```
spring.data.mongodb.option.min-connection-per-host=0
spring.data.mongodb.option.max-connection-per-host=100
spring.data.mongodb.option.threads-allowed-to-block-for-connection-multiplier=5
spring.data.mongodb.option.server-selection-timeout=30000
spring.data.mongodb.option.max-wait-time=120000
spring.data.mongodb.option.max-connection-idle-time=0
spring.data.mongodb.option.max-connection-life-time=0
spring.data.mongodb.option.connect-timeout=10000
spring.data.mongodb.option.socket-timeout=0

spring.data.mongodb.option.socket-keep-alive=false
spring.data.mongodb.option.ssl-enabled=false
spring.data.mongodb.option.ssl-invalid-host-name-allowed=false
spring.data.mongodb.option.always-use-m-beans=false

spring.data.mongodb.option.heartbeat-socket-timeout=20000
spring.data.mongodb.option.heartbeat-connect-timeout=20000
spring.data.mongodb.option.min-heartbeat-frequency=500
spring.data.mongodb.option.heartbeat-frequency=10000
spring.data.mongodb.option.local-threshold=15
```

上述配置值均为默认值

























