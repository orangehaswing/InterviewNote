# Jackson序列化反序列化和注解使用

# 自定义ObjectMapper

我们都知道，在Spring中使用`@ResponseBody`注解可以将方法返回的对象序列化成JSON，比如：

```
@RequestMapping("getuser")
@ResponseBody
public User getUser() {
    User user = new User();
    user.setUserName("mrbird");
    user.setBirthday(new Date());
    return user;
}

```

User类：

```
public class User implements Serializable {
    private static final long serialVersionUID = 6222176558369919436L;
    
    private String userName;
    private int age;
    private String password;
    private Date birthday;
    ...
}

```

访问`getuser`页面输出：

```
{"userName":"mrbird","age":0,"password":null,"birthday":1522634892365}

```

可看到时间默认以时间戳的形式输出，如果想要改变这个默认行为，我们可以自定义一个ObjectMapper来替代：

```
import java.text.SimpleDateFormat;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import com.fasterxml.jackson.databind.ObjectMapper;

@Configuration
public class JacksonConfig {
    @Bean
    public ObjectMapper getObjectMapper(){
        ObjectMapper mapper = new ObjectMapper();
        mapper.setDateFormat(new SimpleDateFormat("yyyy-MM-dd HH:mm:ss"));
        return mapper;
    }
}

```

上面配置获取了ObjectMapper对象，并且设置了时间格式。再次访问`getuser`，页面输出：

```
{"userName":"mrbird","age":0,"password":null,"birthday":"2019-01-07 21:24:38"}
```

# 序列化

Jackson通过使用mapper的`writeValueAsString`方法将Java对象序列化为JSON格式字符串：

```
@Autowired
ObjectMapper mapper;

@RequestMapping("serialization")
@ResponseBody
public String serialization() {
    try {
        User user = new User();
        user.setUserName("mrbird");
        user.setBirthday(new Date());
        String str = mapper.writeValueAsString(user);
        return str;
    } catch (Exception e) {
        e.printStackTrace();
    }
    return null;
}

```

# 反序列化

使用`@ResponseBody`注解可以使对象序列化为JSON格式字符串，除此之外，Jackson也提供了反序列化方法。

## 树遍历

当采用树遍历的方式时，JSON被读入到JsonNode对象中，可以像操作XML DOM那样读取JSON。比如：

```
@Autowired
ObjectMapper mapper;

@RequestMapping("readjsonstring")
@ResponseBody
public String readJsonString() {
    try {
        String json = "{\"name\":\"mrbird\",\"age\":26}";
        JsonNode node = this.mapper.readTree(json);
        String name = node.get("name").asText();
        int age = node.get("age").asInt();
        return name + " " + age;
    } catch (Exception e) {
        e.printStackTrace();
    }
    return null;
}

```

`readTree`方法可以接受一个字符串或者字节数组、文件、InputStream等， 返回JsonNode作为根节点，你可以像操作XML DOM那样操作遍历JsonNode以获取数据。

解析多级JSON例子：

```
String json = "{\"name\":\"mrbird\",\"hobby\":{\"first\":\"sleep\",\"second\":\"eat\"}}";;
JsonNode node = this.mapper.readTree(json);
JsonNode hobby = node.get("hobby");
String first = hobby.get("first").asText();

```

## 绑定对象

我们也可以将Java对象和JSON数据进行绑定，如下所示：

```
@Autowired
ObjectMapper mapper;

@RequestMapping("readjsonasobject")
@ResponseBody
public String readJsonAsObject() {
    try {
        String json = "{\"name\":\"mrbird\",\"age\":26}";
        User user = mapper.readValue(json, User.class);
        String name = user.getUserName();
        int age = user.getAge();
        return name + " " + age;
    } catch (Exception e) {
        e.printStackTrace();
    }
    return null;
}
```

# Jackson注解

@JsonProperty

`@JsonProperty`，作用在属性上，用来为JSON Key指定一个别名。

@Jsonlgnore

`@Jsonlgnore`，作用在属性上，用来忽略此属性。

@JsonIgnoreProperties

`@JsonIgnoreProperties`，忽略一组属性，作用于类上，比如`JsonIgnoreProperties({ "password", "age" })`。

@JsonFormat

`@JsonFormat`，用于日期格式化，如：

```
@JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss")
private Date birthday;
```

@JsonNaming

`@JsonNaming`，用于指定一个命名策略，作用于类或者属性上。Jackson自带了多种命名策略，你可以实现自己的命名策略，比如输出的key 由Java命名方式转为下面线命名方法 —— userName转化为user-name。

@JsonSerialize

`@JsonSerialize`，指定一个实现类来自定义序列化。类必须实现`JsonSerializer`接口。

@JsonDeserialize

`@JsonDeserialize`，用户自定义反序列化，同`@JsonSerialize` ，类需要实现`JsonDeserializer`接口。

@JsonView

`@JsonView`，作用在类或者属性上，用来定义一个序列化组。 比如对于User对象，某些情况下只返回userName属性就行，而某些情况下需要返回全部属性。

```
public class User implements Serializable {
    private static final long serialVersionUID = 6222176558369919436L;
    
    public interface UserNameView {};
    public interface AllUserFieldView extends UserNameView {};
    
    @JsonView(UserNameView.class)
    private String userName;
    
    @JsonView(AllUserFieldView.class)
    private int age;
    
    @JsonView(AllUserFieldView.class)
    private String password;
    
    @JsonView(AllUserFieldView.class)
    private Date birthday;
    ...	
}
```

















