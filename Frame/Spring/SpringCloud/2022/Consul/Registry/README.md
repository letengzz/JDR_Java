# Consul 服务注册与发现

## 服务注册与发现

**官方文档**：https://docs.spring.io/spring-cloud-consul/reference/quickstart.html

在项目中添加服务发现依赖：

```xml
<!--SpringCloud consul discovery -->
<dependency>
	<groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-consul-discovery</artifactId>
    <exclusion>
    	<groupId>commons-logging</groupId>
        <artifactId>commons</artifactId>
    </exclusion>
</dependency>
<dependency>
     <groupId>org.springframework.boot</groupId>
     <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

在配置文件`application.yaml`中配置地址：

```yaml
spring:
  application:
    name: cloud-consumer-order
  ####Spring Cloud Consul for Service Discovery
  cloud:
    consul:
      host: localhost
      port: 8500
      discovery:
        prefer-ip-address: true #优先使用服务ip进行注册
        service-name: ${spring.application.name}
```

**说明**：Consul 会占用8301和8302端口，请更改服务端口号为8301或8302的服务端口号，否则会运行失败。

![image-20240304233920457](assets/image-20240304233920457.png)

## 远程调用

使用RestTemplate调用时，需要清楚的知道被调用者(服务提供者)的地址，但是一旦地址发生变化，就需要在调用者(服务消费者)代码中修改。

但是在服务提供者和服务消费者都注册在Consul中时，可以直接根据其服务名称(服务注册中心上的微服务名称) 来获取动态的获取地址。

```java
@Service
public class BorrowServiceImpl implements BorrowService {
    @Resource
    private BorrowMapper mapper;

    //RestTemplate支持多种方式的远程调用
    @Resource
    private RestTemplate restTemplate;



    @Override
    public UserBorrowDetail getUserBorrowDetailByUid(Integer uid) {
        List<Borrow> borrow = mapper.getBorrowsByUid(uid);
        //调用其他关联信息

        //获取User信息
        //这里通过调用getForObject来请求其他服务，并将结果自动进行封装
        //这里不用再写IP，直接写服务名称provider-user
        User user = restTemplate.getForObject("http://provider-user/user/" + uid, User.class);

        //获取每一本书的详细信息
        List<Book> books = borrow.stream().map(borrow1 ->
                        restTemplate.getForObject("http://provider-book/book/" + borrow1.getBid(), Book.class))
                .collect(Collectors.toList());
        return new UserBorrowDetail(user,books);
    }
}
```

将RestTemplate添加`@LoadBalanced`注解，这样Consul就会对服务的调用进行自动发现，并提供负载均衡：

```java
@Configuration
public class BeanConfig {
    @Bean
    @LoadBalanced
    public RestTemplate restTemplate(){
        return new RestTemplate();
    }
}
```

访问http://localhost:8801/borrow/1：

![image-20230309184050142](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303271910490.png)

同一个服务器实际上是可以注册很多个的，但是它们的端口不同，比如创建多个用户查询服务，现在将原有的端口配置修改一下，由IDEA中设定启动参数来决定，这样就可以多创建几个不同端口的启动项了：

![image-20240306133056401](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202403061331423.png)

在Consul中，同一个服务出现了三个实例：

![image-20240306133345589](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202403061333938.png)

修改用户查询，然后进行远程调用，看看请求是不是均匀地分配到这两个服务端：

```java
@RestController
public class UserController {
    @Resource
    UserService service;

    @Value("${server.port}")
    private String serverPort;

    @RequestMapping("/user/{uid}")
    public User findUserById(@PathVariable("uid") Integer uid){
        System.out.println("我被调用拉！服务端口=" + serverPort);
        return service.getUserById(uid);
    }
}
```

三个实例都能够均匀地被分配请求：

![image-20240306133653923](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202403061336553.png)

这样，服务自动发现以及简单的负载均衡就实现完成了，并且，如果某个微服务挂掉了，只要存在其他同样的微服务实例在运行，那么就不会导致整个微服务不可用，极大地保证了安全性。