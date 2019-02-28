# springboot学习笔记-使用JdbcTemplate

在Spring Boot开启JdbcTemplate很简单，只需要引入`spring-boot-starter-jdbc`依赖即可。JdbcTemplate封装了许多SQL操作，

具体可查阅官方档

[JdbcTemplate](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/jdbc/core/JdbcTemplate.html)。

## 引入依赖

spring-boot-starter-jdbc：

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jdbc</artifactId>
</dependency>
```

以MySQL数据库为例，引入MySQL连接的依赖包，在`pom.xml`中加入：

```
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>5.1.21</version>
</dependency>
```

## 代码编写

### 直接使用JdbcTemplate类

数据准备：

```
CREATE TABLE user (
   name NOT NULL ,
   age NOT NULL 
);
```

User类

```
@Component
public class User {

    private String name;
    private int age;
	get & set...
}
```

StudentMapper类代码：

```
@Repository("studentDao")
public class StudentMapper implements StudentDao{

    @Autowired
    private JdbcTemplate jdbcTemplate;

    @Autowired
    private User user;

    @Override
    public void create(String name, Integer age) {
        String sql = "INSERT  INTO USER(NAME ,AGE) VALUES (?,?)";
        int c = jdbcTemplate.update(sql,name,age);
        System.out.println("c:"+ c);
    }

    @Override
    public void deleteByName(String name) {
        String sql = "delete from USER where NAME = ?";
        int temp = jdbcTemplate.update(sql,name);
        System.out.println("temp :" + temp);
    }
```

在引入`spring-boot-starter-jdbc`驱动后，可直接在类中注入JdbcTemplate。由上面代码可发现，对于保存操作有两种不同的方法，当插入的表字段较多的情况下，推荐使用`NamedParameterJdbcTemplate`。

对于返回结果，可以直接使用`List<>`来接收，这也是个人比较推荐使用的方式，毕竟比较简单方便；也可以使用库表对应的实体对象来接收，不过这时候我们就需要手动创建一个实现了`org.springframework.jdbc.core.RowMapper`的对象，用于将实体对象属性和库表字段一一对应：

```
@Repository("studentDao")
public class StudentMapper implements StudentDao{

    @Autowired
    private JdbcTemplate jdbcTemplate;

    @Autowired
    private User user;

    @Override
    public void create(String name, Integer age) {
        String sql = "INSERT  INTO USER(NAME ,AGE) VALUES (?,?)";
        int c = jdbcTemplate.update(sql,name,age);
        System.out.println("c:"+ c);
    }

    @Override
    public void deleteByName(String name) {
        String sql = "delete from USER where NAME = ?";
        int temp = jdbcTemplate.update(sql,name);
        System.out.println("temp :" + temp);
    }
}
```

测试类：

```
@RunWith(SpringJUnit4ClassRunner.class)
@SpringBootTest
public class StudentMapperTest {

    @Autowired
    private StudentMapper studentMapper;

    @Test
    public void test() throws Exception{
        studentMapper.create("red",10);
        studentMapper.create("green",15);

        studentMapper.deleteByName("b");
    }
}
```

### 使用注解操作数据库

- 引入依赖：

```
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-jdbc</artifactId>
</dependency>
<dependency>
	<groupId>mysql</groupId>
	<artifactId>mysql-connector-java</artifactId>
	<version>5.1.21</version>
</dependency>
```



- 在`src/main/resources/application.properties`中配置数据源信息

```
spring.datasource.url=jdbc:mysql://localhost:3306/test
spring.datasource.username=dbuser
spring.datasource.password=dbpass
spring.datasource.driver-class-name=com.mysql.jdbc.Driver
```

- 操作数据库

Spring的JdbcTemplate是自动配置的，你可以直接使用`@Autowired`来注入到你自己的bean中来使用。

- 定义包含有插入、删除、查询的抽象接口UserService

```
public interface UserService {
    /**
     * @param name
     * @param age
     */
    void create(String name,Integer age);
    /**
     * @param name
     */
    void deleteByName(String name);
    /**
     * @return Integer
     */
    Integer getAllUsers();

    void deleteAllUsers();
}
```

- 通过JdbcTemplate实现UserService中定义的数据访问操作

```
@Service
public class UserServiceImpl implements UserService {

    @Autowired
    private JdbcTemplate jdbcTemplate;

    @Override
    public void create(String name, Integer age) {
        jdbcTemplate.update("insert into USER(NAME, AGE) values(?, ?)", name, age);
    }

    @Override
    public void deleteByName(String name) {
        jdbcTemplate.update("delete from USER where NAME = ?", name);
    }

    @Override
    public Integer getAllUsers() {
        return jdbcTemplate.queryForObject("select count(1) from USER", Integer.class);
    }

    @Override
    public void deleteAllUsers() {
        jdbcTemplate.update("delete from USER");
    }
}

```

- 创建对UserService的单元测试用例，通过创建、删除和查询来验证数据库操作的正确性。

```
@RunWith(SpringJUnit4ClassRunner.class)
@SpringBootTest
public class UserServiceImplTest {

    @Autowired
    private UserService userService;

    @Before
    public void setUp(){
        userService.deleteAllUsers();
    }

    @Test
    public void test() throws Exception{
        userService.create("a",1);
        userService.create("b",2);
        userService.create("c",3);
        userService.create("d",4);
        userService.create("e",5);

        Assert.assertEquals(5,userService.getAllUsers().intValue());

        userService.deleteByName("a");
        userService.deleteByName("e");

        Assert.assertEquals(3,userService.getAllUsers().intValue());
    }
}
```



















