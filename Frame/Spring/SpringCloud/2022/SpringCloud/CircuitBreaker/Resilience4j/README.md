# Resilience4j

Resilience4j 是一个专为函数式编程设计的轻量级容错库。Resilience4j 提供高阶函数 (装饰器)，以通过断路器、速率限制器、重试或隔板增强任何功能接口、lambda表达式或方法引用。可以在任何函数式接口、lambda表达式或方法引用上堆叠多个装饰器。

- [熔断 (CircuitBreaker)](CircuitBreaker/README.md)
- [隔离 (BulkHead)]()
- [限流 (RateLimiter)]()

**整合操作**：

- [Resilience4j 整合 OpenFeign](OpenFeign/README.md)
- [Resilience4j 整合 RestTemplate](RestTemplate/README.md)
- [Resilience4j 整合 Gateway](Gateway/README.md)

**在borrow添加依赖**：

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```

在启动类添加`@EnableFeignClients`注解：

```java
@SpringBootApplication
@EnableDiscoveryClient
@EnableFeignClients
public class BorrowApplication {
    public static void main(String[] args) {
        SpringApplication.run(BorrowApplication.class,args);
    }
}
```

需要调用其他微服务提供的接口可以直接创建一个对应服务的接口类：

> UserClient.java

```java
@FeignClient("userservice")   //声明为userservice服务的HTTP请求客户端
public interface UserClient {
  	//路径保证和其他微服务提供的一致即可
    @RequestMapping("/user/{uid}")
    User getUserById(@PathVariable("uid") Integer uid);  //参数和返回值也保持一致
}
```

> BookClient.java

```java
@FeignClient("bookservice") //声明为userservice服务的HTTP请求客户端
public interface BookClient {
    //路径保证和其他微服务提供的一致即可
    @RequestMapping("/book/{bid}")
    Book getBookById(@PathVariable("bid") Integer bid);  //参数和返回值也保持一致
}
```

直接注入使用：

> BorrowServiceImpl.java

```java
@Service
public class BorrowServiceImpl implements BorrowService {

    @Resource
    BorrowMapper mapper;

    @Resource
    UserClient userClient;

    @Resource
    BookClient bookClient;

    @Override
    public UserBorrowDetail getUserBorrowDetailByUid(Integer uid) {
        List<Borrow> borrow = mapper.getBorrowsByUid(uid);

        User user = userClient.getUserById(uid);
        List<Book> bookList = borrow
                .stream()
                .map(b -> bookClient.getBookById(b.getBid()))
                .collect(Collectors.toList());
        return new UserBorrowDetail(user, bookList);
    }
}
```

重启工程，浏览器访问http://localhost:8082/test-load-balancer，刷新发现浏览器轮流显示：

![image-20230310182914337](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303281407516.png)

这样，实现了OpenFeign负载均衡。

## 实现降级

Hystrix也可以配合Feign进行降级，可以对应接口中定义的远程调用单独进行降级操作。

比如用户服务挂掉，那么这个时候肯定是会远程调用失败的，也就是说Controller中的方法在执行过程中会直接抛出异常，进而被Hystrix监控到并进行服务降级。

而实际上导致方法执行异常的根源就是远程调用失败，既然用户服务调用失败，那么就给这个远程调用添加一个替代方案，如果此远程调用失败，那么就直接上替代方案。Feign都是以接口的形式来声明远程调用，那么既然远程调用已经失效，可以自行对其进行实现，创建一个实现类，对原有的接口方法进行替代方案实现：

```java
@Component   //注意，需要将其注册为Bean，Feign才能自动注入
public class UserFallbackClient implements UserClient{
    @Override
    public User getUserById(Integer uid) {   //这里我们自行对其进行实现，并返回我们的替代方案
        User user = new User();
        user.setName("我是替代方案");
        return user;
    }
}
```

在原有的接口中指定失败替代实现：

```java
//fallback参数指定为刚刚编写的实现类
@FeignClient(value = "userservice", fallback = UserFallbackClient.class)
public interface UserClient {

    @RequestMapping("/user/{uid}")
    User getUserById(@PathVariable("uid") Integer uid);
}
```

去掉`BorrowController`的`@HystrixCommand`注解和备选方法：

```java
@RestController
public class BorrowController {
    @Resource
    BorrowService service;

    @Resource
    private LoadBalancerClient loadBalancerClient;


    @RequestMapping("/borrow/{uid}")
    public UserBorrowDetail findUserBorrows(@PathVariable("uid") Integer uid){
        return service.getUserBorrowDetailByUid(uid);
    }


    @GetMapping("/test-load-balancer")
    public String testLoadBalancer() {
        ServiceInstance instance = loadBalancerClient.choose("userservice");
        return instance.getHost() + ":" + instance.getPort();
    }
}
```

最后在配置文件中开启熔断支持：

```yaml
feign:
  circuitbreaker:
    enabled: true
```

启动服务，调用接口：

![image-20230313150139015](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303281409566.png)

![image-20220325122301779](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303281409614.png)

已经采用替代方案作为结果。

## 超时配置

OpenFeign集成了负载均衡组件LoadBalancer，OpenFeign提供了2个超时参数。

- connectTimeout防止由于服务器处理时间长而阻塞调用者。


- readTimeout 从连接建立时开始应用，在返回响应时间过长时触发。


对于所有的FeignClient配置，可以使用"default"假名：

```yaml
feign:
  client:
    config:
      default:
        connectTimeout: 5000 #防止由于服务器处理时间长而阻塞调用者
        readTimeout: 5000 #从连接建立时开始应用，在返回响应时间过长时触发
```

如果只对于具体FeignClient配置，可以把default换成具体的FeignClient的名字：

```yaml
feign:
  client:
    config:
      userservice:
        connectTimeout: 5000 #防止由于服务器处理时间长而阻塞调用者
        readTimeout: 5000 #从连接建立时开始应用，在返回响应时间过长时触发
```

## 集成熔断器 

Feign可以集成Spring Cloud CircuitBreaker熔断器，集成后，Feign将使用断路器包装的所有方法：

**添加依赖**：

在借阅工程pom.xml中增加resilience4j熔断组件依赖：

```xml
<dependency>		
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-circuitbreaker-resilience4j</artifactId>
</dependency>
```

**开启Feign的熔断器支持**：

在application.yml中增加配置：

```yaml
feign:
  circuitbreaker:
    enabled: true
```

**Feign熔断降级类**：

Spring Cloud CircuitBreaker支持降级概念，当熔断器打开，或者调用是出现错误，则执行降级方法。`@FeignClient`的fallback属性指定讲解的类，注意服务降级类需要在spring容器中注册：

```java
@FeignClient(value = "userservice",fallback = UserClient.Fallback.class)   //声明为userservice服务的HTTP请求客户端
public interface UserClient {
    //路径保证和其他微服务提供的一致即可
    @RequestMapping("/user/{uid}")
    User getUserById(@PathVariable("uid") Integer uid);  //参数和返回值也保持一致

    @Component
    static class Fallback implements UserClient {
        @Override
        public  User getUserById(Integer uid) {
            //这里我们自行对其进行实现，并返回我们的替代方案
            User user = new User();
            user.setName("熔断降级方法返回");
            return user;
        }
    }
}
```

如果想要获得熔断降级的异常信息，比如打印异常日志，则可以使用fallbackFactory属性指定：

```java
@FeignClient(value = "userservice",fallback = UserClient.Fallback.class,fallbackFactory = UserClient.FallBackFactory.class)   //声明为userservice服务的HTTP请求客户端
public interface UserClient {
    //路径保证和其他微服务提供的一致即可
    @RequestMapping("/user/{uid}")
    User getUserById(@PathVariable("uid") Integer uid);  //参数和返回值也保持一致

    @Component
    static class Fallback implements UserClient {
        @Override
        public  User getUserById(Integer uid) {
            //这里我们自行对其进行实现，并返回我们的替代方案
            User user = new User();
            user.setName("熔断降级方法返回");
            return user;
        }
    }

    @Component
    static class FallBackFactory implements FallbackFactory<Fallback> {

        @Override
        public Fallback create(Throwable cause) {
            cause.printStackTrace();
            return new Fallback();
        }
    }

}
```

**启动并测试**：

启动订单服务和Eureka，此时因为没有服务提供者支付服务。执行发生异常，熔断降级方法执行：

![image-20230313173433895](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303281410884.png)

## 请求和响应压缩 

使用配置对Feign请求和响应进行压缩：

```yaml
feign:
  compression:
    request:
      enabled: true # 请求压缩
      mime-types: text/xml,application/xml,application/json # 压缩的类型
      min-request-size: 2048 # 请求最小压缩的阈值
    response:
      enabled: true #响应压缩
      useGzipDecoder: true #使用gzip解码器解码响应数据
```

