# 搭建微服务工程

采用：

- Spring Boot 3.1.5
- Spring Cloud 2022.0.4
- Spring Cloud Alibaba 2022.0.0.0

**注意**：

- Spring Boot3.x 版本要求 JDK 至少是 17，Spring 6.0
- IntelliJ IDEA 从 2022.2 版本开始完全支持 Spring Boot3.x 与 Spring 6

## 微服务项目结构

将单体项目(图书管理系统)进行拆分(**项目拆分一定要尽可能保证单一职责**，相同的业务不要在多个微服务中重复出现，如果出现需要借助其他业务完成的服务，那么可以使用服务之间相互调用的形式来实现)：

- 登录验证服务：用于处理用户注册、登录、密码重置等，反正就是一切与账户相关的内容，包括用户信息获取等。
- 图书管理服务：用于进行图书添加、删除、更新等操作，图书管理相关的服务，包括图书的存储等和信息获取。
- 图书借阅服务：交互性比较强的服务，需要和登陆验证服务和图书管理服务进行交互。

将单体应用拆分为多个小型服务，需要重新设计一下整个项目目录结构，父项目中创建多个子项目，每一个子项目都是一个服务，这样由父项目统一管理依赖，就无需每个子项目都去单独管理依赖了，方便管理。

由于项目中存在消费关系，可以通过使用的 Spring 提供的 RestTemplate 实现**消费者对提供者的调用**，但并未使用到 Spring Cloud，对于 MySQL 数据库的访问，使用 Mybatis 作为持久层技术。后续 Spring Cloud 的运行测试环境就是在此基础上修改出来的。

## 创建项目

1. 创建数据库：

   ![image-20230308213712623](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303271751902.png)

   ```sql
   DROP TABLE IF EXISTS `DB_BOOK`;
   CREATE TABLE `DB_BOOK` (
     `bid` int(11) NOT NULL AUTO_INCREMENT,
     `title` varchar(255) NOT NULL,
     `desc` varchar(255) NOT NULL,
     PRIMARY KEY (`bid`)
   ) ENGINE=InnoDB AUTO_INCREMENT=5 DEFAULT CHARSET=utf8mb4;
   
   INSERT INTO `DB_BOOK` (`bid`, `title`, `desc`) VALUES (1, '深入理解Java虚拟机', '讲解JVM底层原理');
   INSERT INTO `DB_BOOK` (`bid`, `title`, `desc`) VALUES (2, 'Java并发编程的艺术', '讲解并发编程的各种技术');
   INSERT INTO `DB_BOOK` (`bid`, `title`, `desc`) VALUES (3, 'Java核心技术卷一', '讲解Java的基础知识等');
   INSERT INTO `DB_BOOK` (`bid`, `title`, `desc`) VALUES (4, '计算机网络', '讲解计算机的网络实现原理和知识');
   
   
   DROP TABLE IF EXISTS `DB_USER`;
   CREATE TABLE `DB_USER` (
     `uid` int(11) NOT NULL AUTO_INCREMENT,
     `name` varchar(255) NOT NULL,
     `age` int(11) NOT NULL,
     `sex` enum('男','女') NOT NULL,
     PRIMARY KEY (`uid`)
   ) ENGINE=InnoDB AUTO_INCREMENT=5 DEFAULT CHARSET=utf8mb4;
   
   INSERT INTO `DB_USER` (`uid`, `name`, `age`, `sex`) VALUES (1, '小明', 18, '男');
   INSERT INTO `DB_USER` (`uid`, `name`, `age`, `sex`) VALUES (2, '小红', 17, '女');
   INSERT INTO `DB_USER` (`uid`, `name`, `age`, `sex`) VALUES (3, '小芳', 18, '女');
   INSERT INTO `DB_USER` (`uid`, `name`, `age`, `sex`) VALUES (4, '小刚', 17, '男');
   
   DROP TABLE IF EXISTS `DB_BORROW`;
   CREATE TABLE `DB_BORROW` (
     `id` int(11) NOT NULL AUTO_INCREMENT,
     `uid` int(11) NOT NULL,
     `bid` int(11) NOT NULL,
     PRIMARY KEY (`id`),
     UNIQUE KEY `unique_bid_uid` (`uid`,`bid`),
     KEY `f_bid` (`bid`),
     CONSTRAINT `f_bid` FOREIGN KEY (`bid`) REFERENCES `DB_BOOK` (`bid`),
     CONSTRAINT `f_uid` FOREIGN KEY (`uid`) REFERENCES `DB_USER` (`uid`)
   ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
   
   INSERT INTO `DB_BORROW` (`id`, `uid`, `bid`) VALUES (1, 1, 1);
   INSERT INTO `DB_BORROW` (`id`, `uid`, `bid`) VALUES (2, 1, 3);
   ```

2. **父项目**(创建一个普通的Maven项目，用作统一管理)：

   ![image-20231110175707136](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202311101757670.png)

3. 不需要勾选任何依赖，直接创建即可，项目创建完成并初始化后，删除父工程的无用文件，只保留必要文件：

   ![image-20231110175734108](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202311101757532.png)

4. 添加依赖：

   ```xml
   <parent>
   	<groupId>org.springframework.boot</groupId>
   	<artifactId>spring-boot-starter-parent</artifactId>
   	<version>3.1.5</version>
   	<relativePath/> <!-- lookup parent from repository -->
   </parent>   
   <dependencies>
   	<dependency>
           <groupId>org.springframework.boot</groupId>
           <artifactId>spring-boot-starter</artifactId>
   	</dependency>
   	<dependency>
           <groupId>org.springframework.boot</groupId>
           <artifactId>spring-boot-starter-test</artifactId>
           <scope>test</scope>
   	</dependency>
   </dependencies>
   <dependencyManagement>
       <!-- Spring Cloud 依赖管理模块 -->
       <dependencies>
           <dependency>
               <groupId>org.springframework.cloud</groupId>
               <artifactId>spring-cloud-dependencies</artifactId>
               <version>2022.0.4</version>
               <type>pom</type>
               <scope>import</scope>
           </dependency>
       </dependencies>
   </dependencyManagement>
   <build>
       <plugins>
       	<plugin>
           	<groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-maven-plugin</artifactId>
               <configuration>
   				<excludes>
    					<exclude>
    						<groupId>org.projectlombok</groupId>
    						<artifactId>lombok</artifactId>
    					</exclude>
    				</excludes>
    			</configuration>
           </plugin>
   	</plugins>
   </build>
   ```

5. 按照划分的服务，进行子工程创建了，创建一个新的Maven项目：

   **注意**：父项目要指定为一开始创建的的项目，子项目命名随意。

   ![image-20231110175904153](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202311101759686.png)

6. 添加web依赖并刷新maven依赖：

   ![image-20231109220948356](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202311092209807.png)

   ```xml
   <dependencies>
   	<dependency>
       	<groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-starter-web</artifactId>
       </dependency>
   </dependencies>
   ```

   修改端口号：

   ```yaml
   server:
     port: 8101
   spring:
     application:
       name: provider-user
   ```

7. 在子项目中创建SpringBoot的启动主类：

   ![image-20231109221052128](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202311092213268.png)

8. 点击运行，即可启动子项目，实际上这个子项目就是一个最简单的SpringBoot web项目

   **注意**：启动之后最下方有弹窗，点击"使用 服务"添加SpringBoot项目，可以实时查看当前整个大项目中有哪些微服务：

   ![image-20231110180729820](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202311101807641.png)

9. 以同样的方法，创建其他的子项目，注意最好将其他子项目的端口设置得不一样，不然会导致端口占用，分别为它们创建`application.yml`文件：

   ![image-20231110181100792](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202311101811412.png)

10. 分别运行在不同的端口上，这样，就方便不同的程序员编写不同的服务了，提交当前项目代码时的冲突率也会降低。

    ![image-20231110181222492](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202311101812038.png)

11. 编写用户信息查询业务，先把数据库相关的依赖进行导入，使用Mybatis框架，首先在父项目中添加MySQL驱动和Lombok依赖：

    ```xml
    <!--mysql5.7:mysql-connector-java-->
    <dependency>
    	<groupId>com.mysql</groupId>
        <artifactId>mysql-connector-j</artifactId>
    </dependency>
    
    <dependency>
         <groupId>org.projectlombok</groupId>
         <artifactId>lombok</artifactId>
    </dependency>
    ```

    不是所有的子项目都需要用到Mybatis，在父项目中只进行版本管理即可：

    ```xml
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.mybatis.spring.boot</groupId>
                <artifactId>mybatis-spring-boot-starter</artifactId>
                <version>3.0.2</version>
            </dependency>
        </dependencies>
    </dependencyManagement>
    ```

12. 在用户服务子项目中添加依赖：

    ```xml
    <dependencies>
        <dependency>
         	<groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
        </dependency>
    </dependencies>
    ```

13. 添加数据源信息：

    > application.yaml

    ```yaml
    spring:
      datasource:
        url: jdbc:mysql://localhost:3306/springcloud?useSSL=false&useUnicode=true&characterEncoding=utf8&serverTimezone=Asia/Shanghai
        driver-class-name: com.mysql.cj.jdbc.Driver
        username: root
        password: 123123
    ```

14. 用户查询相关的业务：

    > User.java

    ```java
    @Data
    public class User {
        private Integer uid;
        private String name;
        private Integer age;
        private String sex;
    }
    ```

    > UserMapper.java

    ```java
    @Mapper
    public interface UserMapper {
        @Select("select * from db_user where uid = #{uid}")
        User getUserById(Integer uid);
    }
    ```

    > UserService.java

    ```java
    public interface UserService {
        User getUserById(Integer uid);
    }
    ```
    
    > UserServiceImpl.java
    
    ```java
    @Service
    public class UserServiceImpl implements UserService {
        @Resource
        private UserMapper mapper;
    
        @Override
        public User getUserById(Integer uid) {
            return mapper.getUserById(uid);
        }
    }
    ```
    
    > UserController.java
    
    ```java
    @RestController
    public class UserController {
        @Resource
        UserService service;
    
        //这里以RESTFul风格为例
        @RequestMapping("/user/{uid}")
        public User findUserById(@PathVariable("uid") Integer uid){
            return service.getUserById(uid);
        }
    }
    ```
    
    访问 http://localhost:8101/user/1 即可：
    
    ![image-20231110203246465](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202311102032906.png)
    
15. 同样的方式，创建图书查询业务：

    > Book.java

    ```java
    @Data
    public class Book {
        private Integer bid;
        private String title;
        private String desc;
    }
    ```

    > BookMapper.java

    ```java
    @Mapper
    public interface BookMapper {
        @Select("select * from db_book where bid = #{bid}")
        Book getBookById(Integer bid);
    }
    ```

    > BookService.java

    ```java
    @Service
    public interface BookService {
        Book getBookById(Integer bid);
    }
    ```

    > BookServiceImpl.java

    ```java
    @Service
    public class BookServiceImpl implements BookService {
        @Resource
        private BookMapper mapper;
    
        @Override
        public Book getBookById(Integer uid) {
            return mapper.getBookById(uid);
        }
    }
    ```

    > BookController.java

    ```java
    @RestController
    public class BookController {
        @Resource
        BookService service;
    
        //这里以RESTFul风格为例
        @RequestMapping("/book/{uid}")
        public Book findUserById(@PathVariable("uid") Integer uid){
            return service.getBookById(uid);
        }
    }
    ```

    访问 http://localhost:8201/book/1 即可：

    ![image-20231110203347889](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202311102033210.png)

这样，一个完整项目的就拆分成了多个微服务，不同微服务之间是独立进行开发和部署的。

## 服务间的调用

借阅服务是一个关联性比较强的服务，它不仅仅需要查询借阅信息，同时可能还需要获取借阅信息下的详细信息，比如具体那个用户借阅了哪本书，并且用户和书籍的详情也需要同时出现，那么这种情况下，就需要去访问除了借阅表以外的用户表和图书表：

![image-20220323140053749](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303271331918.png)

但是这显然是违反之前所说的单一职责的，相同的业务功能不应该重复出现，但是现在由需要在此服务中查询用户的信息和图书信息，可以让一个服务去调用另一个服务来获取信息：

![image-20220323140322502](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303271331550.png)

这样，图书管理微服务和用户管理微服务相对于借阅记录，就形成了一个生产者和消费者的关系，前者是生产者，后者便是消费者。

**完善借阅关联信息查询**：

> Borrow.java

```java
@Data
public class Borrow {
    private Integer id;
    private Integer uid;
    private Integer bid;
}
```

> BorrowMapper.java

```java
@Mapper
public interface BorrowMapper {
    @Select("select * from db_borrow where uid = #{uid}")
    List<Borrow> getBorrowsByUid(Integer uid);

    @Select("select * from db_borrow where bid = #{bid}")
    List<Borrow> getBorrowsByBid(Integer bid);

    @Select("select * from db_borrow where bid = #{bid} and uid = #{uid}")
    Borrow getBorrow(Integer uid, int bid);
}
```

需要查询用户的借阅详细信息，也就是说需要查询某个用户具体借了那些书，并且需要此用户的信息和所有已借阅的书籍信息一起返回。

借阅详细信息实体：

> BorrowService.java

```java
@Service
public interface BorrowService {
}
```

但是User和Book实体实际上是在另外两个微服务中定义的，相当于当前项目并没有定义这些实体类，可以将所有服务需要用到的实体类单独放入另一个一个项目中，然后让这些项目引用集中存放实体类的那个项目，这样就可以保证每个微服务的实体类信息都可以共用了：

![image-20231110201855679](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202311102019606.png)

```java
@Data
@AllArgsConstructor
public class UserBorrowDetail {
    User user;
    List<Book> bookList;
}
```

![image-20231110124932444](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202311101249813.png)

在对应的类中引用此项目作为依赖即可：

```xml
<dependency>
	<groupId>com.hjc.demo</groupId>
    <artifactId>commons</artifactId>
    <version>1.0-SNAPSHOT</version>
</dependency>
```

继续编写业务代码：

> BorrowService.java

```java
@Service
public interface BorrowService {
    UserBorrowDetail getUserBorrowDetailByUid(Integer uid);
}
```

> BorrowServiceImpl.java

```java
@Service
public class BorrowServiceImpl implements BorrowService {
    @Resource
    private BorrowMapper mapper;

    @Override
    public UserBorrowDetail getUserBorrowDetailByUid(Integer uid) {
        List<Borrow> borrow = mapper.getBorrowsByUid(uid);
        //调用其他关联信息

        return null;
    }
}
```

进行服务远程调用需要用到`RestTemplate`来进行：

1. 将RestTemplate声明为一个Bean，添加到IoC容器中：

   > SpringConfig.java

   ```java
   @Configuration
   public class SpringConfig {
       @Bean
       public RestTemplate restTemplate(){
           return new RestTemplate();
       }
   }
   ```

2. 修改BorrowServiceImpl：

   > BorrowServiceImpl.java

   ```java
   @Service
   public class BorrowServiceImpl implements BorrowService {
       @Resource
       private BorrowMapper mapper;
   
       //RestTemplate支持多种方式的远程调用
       @Resource
       private RestTemplate restTemplate;
   
       public static final String USER_PROVIDER = "http://localhost:8101/user/";
       public static final String BOOK_PROVIDER = "http://localhost:8201/book/";
   
       @Override
       public UserBorrowDetail getUserBorrowDetailByUid(Integer uid) {
           List<Borrow> borrow = mapper.getBorrowsByUid(uid);
           //调用其他关联信息
   
           //获取User信息
           //这里通过调用getForObject来请求其他服务，并将结果自动进行封装
           User user = restTemplate.getForObject(USER_PROVIDER + uid, User.class);
   
           //获取每一本书的详细信息
           List<Book> books = borrow.stream().map(borrow1 ->
                           restTemplate.getForObject(BOOK_PROVIDER + borrow1.getBid(), Book.class))
                   .collect(Collectors.toList());
           return new UserBorrowDetail(user, books);
       }
   }
   ```

3. 编写Controller：

   > BorrowController.java

   ```java
   @RestController
   public class BorrowController {
       @Resource
       BorrowService service;
       
       @RequestMapping("/borrow/{uid}")
       public UserBorrowDetail findUserBorrows(@PathVariable("uid") Integer uid){
           return service.getUserBorrowDetailByUid(uid);
       }
   }
   ```

访问测试是否可用：http://localhost:8301/borrow/1

**注意**：一定要保证三个服务都处于开启状态，否则远程调用会失败。

![image-20231110203608451](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202311102036670.png)

结果正常，没有问题，远程调用成功。

## 问题

微服务所在的IP地址和端口号硬编码到借阅微服务中，会存在非常多的问题：

1. 如果用户微服务和图书微服务的IP地址或者端口号发生了变化，则借阅微服务将变得不可用，需要同步修改借阅微服务中调用微服务的IP地址和端口号。

2. 如果系统中提供了多个用户微服务和图书微服务，则无法实现微服务的负载均衡功能。

3. 如果系统需要支持更高的并发，需要部署更多的用户微服务和图书微服务，硬编码借阅微服务则后续的维护会变得异常复杂。
