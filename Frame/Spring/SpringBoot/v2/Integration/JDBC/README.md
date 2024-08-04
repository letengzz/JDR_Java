## JDBC交互框架

除了Mybatis之外，实际上Spring官方也提供了一个非常方便的JDBC操作工具，它同样可以快速进行增删改查。

1. 通过starter将依赖导入：

```xml
<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-jdbc</artifactId>
</dependency>
```

导入完成之后就可以轻松使用了。

### JDBC模版类

Spring JDBC提供了一个非常方便的`JdbcTemplate`类，它封装了常用的JDBC操作，可以快速使用这些方法来实现增删改查。

配置MySQL数据源信息：

```xml
<dependency>
    <groupId>com.mysql</groupId>
    <artifactId>mysql-connector-j</artifactId>
</dependency>
```

```yml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/test
    username: root
    password: 123456
    driver-class-name: com.mysql.cj.jdbc.Driver
```

我们要操作数据库，最简单直接的方法就是使用JdbcTemplate来完成：

```java
@Resource
JdbcTemplate template;
```

它给我们封装了很多方法使用，比如我们要查询数据库中的一条记录：

![image-20230716000431492](https://s2.loli.net/2023/07/16/ygRp98mDKafXkw1.png)

我们可以使用`queryForMap`快速以Map为结果的形式查询一行数据：

```java
@Test
void contextLoads() {
    Map<String, Object> map = template.queryForMap("select * from user where id = ?", 1);
    System.out.println(map);
}
```

非常方便：

![image-20230720215124918](https://s2.loli.net/2023/07/20/ijczpNxh4fXoQKv.png)

也可以编写自定义的Mapper用于直接得到查询结果：

```java
@Data
@AllArgsConstructor
public class User {
    int id;
    String name;
    String email;
    String password;
}
```

```java
@Test
void contextLoads() {
    User user = template.queryForObject("select * from user where id = ?",
        (r, i) -> new User(r.getInt(1), r.getString(2), r.getString(3), r.getString(4)), 1);
    System.out.println(user);
}
```

当然除了这些之外，它还提供了`update`方法适用于各种情况的查询、更新、删除操作：

```java
 @Test
void contextLoads() {
    int update = template.update("insert into user values(2, 'admin', '654321@qq.com', '123456')");
    System.out.println("更新了 "+update+" 行");
}
```

这样，如果是那种非常小型的项目，甚至是测试用例的话，都可以快速使用JdbcTemplate快速进行各种操作。

### JDBC简单封装

对于一些插入操作，Spring JDBC为我们提供了更方便的SimpleJdbcInsert工具，它可以实现更多高级的插入功能，比如我们的表主键采用的是自增ID，那么它支持插入后返回自动生成的ID，这就非常方便了：

```java
@Configuration
public class WebConfiguration {

    @Resource
    DataSource source;

    @Test
    void contextLoads() {
      	//这个类需要自己创建对象
        SimpleJdbcInsert simple = new SimpleJdbcInsert(source)
                .withTableName("user")   //设置要操作的表名称
                .usingGeneratedKeyColumns("id");    //设置自增主键列
        Map<String, Object> user = new HashMap<>(2);  //插入操作需要传入一个Map作为数据
        user.put("name", "bob");
        user.put("email", "112233@qq.com");
        user.put("password", "123456");
        Number number = simple.executeAndReturnKey(user);   //最后得到的Numver就是得到的自增主键
        System.out.println(number);
    }
}
```

这样就可以快速进行插入操作并且返回自增主键了，还是挺方便的。

![image-20230720224314223](https://s2.loli.net/2023/07/20/xMeBEY3sdKVGmly.png)

当然，虽然SpringJDBC给我们提供了这些小工具，但是其实只适用于简单小项目，稍微复杂一点就不太适合了。

