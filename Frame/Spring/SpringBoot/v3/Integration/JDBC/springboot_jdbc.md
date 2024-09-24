## SpringBoot整合JDBC

**说明**：SpringBoot默认使用HikariDataSource数据源。

**导入依赖**：

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jdbc</artifactId>
</dependency>
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <!-- 想要修改版本
            1、直接依赖引入具体版本（maven的就近依赖原则）
            2、重新声明版本（maven的属性的就近优先原则） -->
</dependency>
```

![image-20230223151248976](./assets/image-20230223151248976.png)

**配置连接参数**：

> application.yaml

```
# 配置连接参数
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/springboot
    username: root
    password: 123123
    driver-class-name: com.mysql.cj.jdbc.Driver
```

**实体类**：

```
@Data
public class User {
    //id
    private Long id;
    
    // ⽤户名
    //⾃动转换下换线到驼峰命名user_name -> userName
    private String userName;
    
    // 密码
    private String password;
    
    // 姓名
    private String name;
    
    // 年龄
    private Integer age;
    
    // 性别，1男性，2⼥性
    private Integer sex;
    
    // 出⽣⽇期
    private Date birthday;
    
    // 创建时间
    private Date created;
    
    // 更新时间
    private Date updated;
    
    // 备注
    private String note;
}
```

**JdbcDao**：

```
@Repository
public class JdbcDao {
    @Autowired
    private JdbcTemplate jdbcTemplate;
    public List<User> findAll() {
        //BeanPropertyRowMapper:可以把同名字段赋值给属性
        return jdbcTemplate.query("select * from tb_user", new BeanPropertyRowMapper<>(User.class));
    }
}
```