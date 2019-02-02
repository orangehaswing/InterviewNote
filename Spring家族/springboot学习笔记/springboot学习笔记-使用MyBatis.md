# springboot学习笔记-使用MyBatis

## MyBatis配置

整合MyBatis之前，先搭建一个基本的Spring Boot项目。

然后 `pom.xml`中引入依赖

- 这里用到spring-boot-starter基础和spring-boot-starter-test用来做单元测试验证数据访问
- 引入连接mysql的必要依赖mysql-connector-java
- 引入整合MyBatis的核心依赖mybatis-spring-boot-starter
- 这里不引入spring-boot-starter-jdbc依赖，是由于mybatis-spring-boot-starter中已经包含了此依赖

```
	
	<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter</artifactId>
	</dependency>

	<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-test</artifactId>
		<scope>test</scope>
	</dependency>

	<dependency>
		<groupId>org.mybatis.spring.boot</groupId>
		<artifactId>mybatis-spring-boot-starter</artifactId>
		<version>1.1.1</version>
	</dependency>

	<dependency>
		<groupId>mysql</groupId>
		<artifactId>mysql-connector-java</artifactId>
		<version>5.1.21</version>
	</dependency>
```

在`application.properties`中配置mysql的连接配置

```
spring.datasource.url=jdbc:mysql://localhost:3306/test
spring.datasource.username=root
spring.datasource.password=123456
spring.datasource.driver-class-name=com.mysql.jdbc.Driver
```

同其他Spring Boot工程一样，简单且简洁的的完成了基本配置

## 使用MyBatis

### 使用注解

- 在Mysql中创建User表，包含id(BIGINT)、name(INT)、age(VARCHAR)字段。同时，创建映射对象User

```
public class User {

    private Long id;
    private String name;
    private Integer age;

    // 省略getter和setter
}
```

- 创建User映射的操作UserMapper，为了后续单元测试验证，实现插入和查询操作

```
@Mapper
public interface UserMapper {

    @Select("SELECT * FROM USER WHERE NAME = #{name}")
    User findByName(@Param("name") String name);

    @Insert("INSERT INTO USER(NAME, AGE) VALUES(#{name}, #{age})")
    int insert(@Param("name") String name, @Param("age") Integer age);

}
```

- 创建单元测试
  - 测试逻辑：插入一条name=AAA，age=20的记录，然后根据name=AAA查询，并判断age是否为20
  - 测试结束回滚数据，保证测试单元每次运行的数据环境独立

```
@RunWith(SpringJUnit4ClassRunner.class)
@SpringApplicationConfiguration(classes = Application.class)
public class ApplicationTests {

	@Autowired
	private UserMapper userMapper;

	@Test
	@Rollback
	public void findByName() throws Exception {
		userMapper.insert("AAA", 20);
		User u = userMapper.findByName("AAA");
		Assert.assertEquals(20, u.getAge().intValue());
	}

}
```

### 使用XML

引入依赖：

```
<dependency>
	<groupId>org.mybatis.spring.boot</groupId>
	<artifactId>mybatis-spring-boot-starter</artifactId>
	<version>1.1.1</version>
</dependency>
<dependency>
	<groupId>mysql</groupId>
	<artifactId>mysql-connector-java</artifactId>
</dependency>
```

application.properties

```
mybatis.config-locations=classpath:mybatis/mybatis-config.xml
mybatis.mapper-locations=classpath:mybatis/mapper/UserMapper.xml
mybatis.type-aliases-package=com.orangehaswing.mybatisxml.entity

spring.datasource.driverClassName = com.mysql.jdbc.Driver
spring.datasource.url = jdbc:mysql://localhost:3306/test?useUnicode=true&characterEncoding=utf-8&serverTimezone=GMT
spring.datasource.username = root
spring.datasource.password = 123456
```

resource.mybatis.mybatis-config.xml

```
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN" "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <typeAliases>
        <typeAlias alias="Integer" type="java.lang.Integer" />
        <typeAlias alias="Long" type="java.lang.Long" />
        <typeAlias alias="HashMap" type="java.util.HashMap" />
        <typeAlias alias="LinkedHashMap" type="java.util.LinkedHashMap" />
        <typeAlias alias="ArrayList" type="java.util.ArrayList" />
        <typeAlias alias="LinkedList" type="java.util.LinkedList" />
    </typeAliases>
</configuration>
```

resource.mybatis.mapper.UserMapper.xml

```
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd" >
<mapper namespace="com.orangehaswing.mybatisxml.mapper.UserMapper" >
    <resultMap id="BaseResultMap" type="com.orangehaswing.mybatisxml.entity.UserEntity" >
        <id column="id" property="id" jdbcType="BIGINT" />
        <result column="userName" property="userName" jdbcType="VARCHAR" />
        <result column="passWord" property="passWord" jdbcType="VARCHAR" />
        <result column="user_sex" property="userSex" javaType="com.orangehaswing.mybatisxml.enums.UserSexEnum"/>
        <result column="nick_name" property="nickName" jdbcType="VARCHAR" />
    </resultMap>

    <sql id="Base_Column_List" >
        id, userName, passWord, user_sex, nick_name
    </sql>

    <select id="getAll" resultMap="BaseResultMap"  >
        SELECT
        <include refid="Base_Column_List" />
        FROM users
    </select>

    <select id="getOne" parameterType="java.lang.Long" resultMap="BaseResultMap" >
        SELECT
        <include refid="Base_Column_List" />
        FROM users
        WHERE id = #{id}
    </select>

    <insert id="insert" parameterType="com.orangehaswing.mybatisxml.entity.UserEntity" >
        INSERT INTO
        users
        (userName,passWord,user_sex)
        VALUES
        (#{userName}, #{passWord}, #{userSex})
    </insert>

    <update id="update" parameterType="com.orangehaswing.mybatisxml.entity.UserEntity" >
        UPDATE
        users
        SET
        <if test="userName != null">userName = #{userName},</if>
        <if test="passWord != null">passWord = #{passWord},</if>
        nick_name = #{nickName}
        WHERE
        id = #{id}
    </update>

    <delete id="delete" parameterType="java.lang.Long" >
        DELETE FROM
        users
        WHERE
        id =#{id}
    </delete>

</mapper>
```

entity.UserEntity

```
public class UserEntity {
    private static final long serialVersionUID = 1L;
    private Long id;
    private String userName;
    private String passWord;
    private UserSexEnum userSex;
    private String nickName;

	constructor...
	get & set...
	toString...
}
```

enums.UserSexEnum

```
public enum UserSexEnum {
    MAN,WOMAN
}
```

mapper.UserMapper

```
@Service
public interface UserMapper {

    List<UserEntity> getAll();

    UserEntity getOne(Long id);

    void insert(UserEntity user);

    void update(UserEntity user);

    void delete(Long id);
}

```

web.UserController

```
@RestController
public class UserController {
    @Autowired
    private UserMapper userMapper;

    @RequestMapping("/getUsers")
    public List<UserEntity> getUsers() {
        List<UserEntity> users=userMapper.getAll();
        return users;
    }

    @RequestMapping("/getUser")
    public UserEntity getUser(Long id) {
        UserEntity user=userMapper.getOne(id);
        return user;
    }

    @RequestMapping("/add")
    public void save(UserEntity user) {
        userMapper.insert(user);
    }

    @RequestMapping(value="update")
    public void update(UserEntity user) {
        userMapper.update(user);
    }

    @RequestMapping(value="/delete/{id}")
    public void delete(@PathVariable("id") Long id) {
        userMapper.delete(id);
    }
}
```

MybatisxmlApplication

```
@SpringBootApplication
@MapperScan("com.orangehaswing.mybatisxml.mapper")
public class MybatisxmlApplication {

	public static void main(String[] args) {
		SpringApplication.run(MybatisxmlApplication.class, args);
	}
}
```

测试：

UserMapperTest

```
@RunWith(SpringRunner.class)
@SpringBootTest
public class UserMapperTest {

    @Autowired
    private UserMapper userMapper;

    @Test
    public void testInsert() throws Exception{
        userMapper.insert(new UserEntity("aa", "a123456", UserSexEnum.MAN));
        userMapper.insert(new UserEntity("bb", "b123456", UserSexEnum.WOMAN));
        userMapper.insert(new UserEntity("cc", "b123456", UserSexEnum.WOMAN));

        Assert.assertEquals(3, userMapper.getAll().size());
    }

    @Test
    public void testQuery() throws Exception {
        List<UserEntity> users = userMapper.getAll();
        if(users==null || users.size()==0){
            System.out.println("is null");
        }else{
            System.out.println(users.toString());
        }
    }

    @Test
    public void testUpdate() throws Exception {
        UserEntity user = userMapper.getOne(6l);
        System.out.println(user.toString());
        user.setNickName("neo");
        userMapper.update(user);
        Assert.assertTrue(("neo".equals(userMapper.getOne(6l).getNickName())));
    }
}
```

UserControllerTest

```
@RunWith(SpringRunner.class)
@SpringBootTest
public class UserControllerTest {

    @Autowired
    private WebApplicationContext wac;
    private MockMvc mockMvc;

    @Before
    public void setUp() throws Exception {
        mockMvc = MockMvcBuilders.webAppContextSetup(wac).build(); //初始化MockMvc对象
    }

    @Test
    public void getUsers() throws Exception {
        mockMvc.perform(MockMvcRequestBuilders.post("/getUsers")
                .accept(MediaType.APPLICATION_JSON_UTF8)).andDo(print());
    }
}
```

创建项目过程中报错：

Q: mysql java.sql.SQLException: The server time zone value‘XXXXXX' is unrecognized or represents...

A: 这是由于数据库和系统时区差异所造成的，在jdbc连接的url后面加上serverTimezone=GMT即可解决问题

```
spring.datasource.url = jdbc:mysql://localhost:3306/test?useUnicode=true&characterEncoding=utf-8&serverTimezone=GMT
```

Q: Unsatisfied dependency expressed through bean property 'sqlSessionFactory';

A: `UserMapper.xml` 文件中 `namespace` 和`reusltType` 对应 package 错误

Q: 使用 `@Autowire` 出现 `Could not autowire. No beans of 'xxxx' type found`

A: 在注入的接口上加上 `@Service`

