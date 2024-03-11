# Lombok

Lombok 是⼀个可以通过简单的注解形式来帮助简化消除⼀些必须有但显得很臃肿的 Java代码的⼯具，通过使用对应的注解，可以在编译源码的时候生成对应的方法。简而言之就是，**通过简单的注解来精简代码达到消除冗长代码的目的**。

- Lombok官网：https://projectlombok.org/

- GitHub地址：https://github.com/projectlombok/lombok

**Lombok 优点**：

- 提⾼编码效率 

- 使代码更简洁 
- 消除冗长代码 
- 避免修改字段名字时忘记修改方法名

**Lombok 常用注解**：

![图片1](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202302221828074.png)

**Lombok 使用**：

在SpringBoot中添加依赖：

```xml
<dependency>
	<groupId>org.projectlombok</groupId>
	<artifactId>lombok</artifactId>
</dependency>
```

****

在IDEA中安装Lombok插件，安装后重启即可：

![image-20230222164401161](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202302221828921.png)

Lombok既是⼀个IDE插件，也是⼀个项目要依赖的jar包。Lombok是依赖jar包的原因是因为编译时要用它的注解。是插件的原因是他要在编译器编译时通过操作AST(抽象语法树)改变字节码生成。也就是说他可以改变java语法。他不像spring的依赖注⼊或者hibernate的orm⼀样是运行时的特性，而是编译时的特性。

****

重构pojo类：

如果觉得`@Data`这个注解有点简单粗暴的话，Lombok提供⼀些更精细的注解，比如 `@Getter`、`@Setter`，(这两个是field注解)、`@ToString`，`@AllArgsConstructor`(这两个是类注解)

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class User {
	private String name;
	private Integer age;
}
```

简化日志开发：

```java
@Slf4j
@RestController
public class HelloController {
    @RequestMapping("/hello")
    public String handle01(@RequestParam("name") String name){
        
        log.info("请求进来了....");
        
        return "Hello, Spring Boot"+"你好："+name;
    }
}
```

链式调用：

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
@Accessors(chain = true) //支持链式编程
public class User {
    private String name;
    private Integer age;
}
```

```java
@Test
void chainTest() {
    User user = new User().setName("张三").setAge(15);
	System.out.println("user = " + user);
}
```

建造者模式：

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
@Builder
public class User {
    private String name;
    private Integer age;
}
```

```java
@Test
void builderTest(){
    User user = User.builder().name("李四").age(18).build();
	System.out.println("user = " + user);
}
```

**注意**：使用Lombok时，当有特殊需求时也可定制自己的代码。



