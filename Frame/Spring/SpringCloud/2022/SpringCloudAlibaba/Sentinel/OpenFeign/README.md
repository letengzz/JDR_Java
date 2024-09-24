# Sentinel 整合 OpenFeign

使用Sentinel，可以直接对Feign的每个接口调用单独进行服务降级。

访问者要有fallback服务降级的情况，不要持续访问加大微服务负担，但是通过feign接口调用的又方法各自不同， 如果每个不同方法都加一个fallback配对方法，会导致代码膨胀不好管理，工程埋雷。

可以通过OpenFeign接口的统一fallback服务降级配置处理，feign接口里面定义的全部方法都走统一的服务降级，**一个搞定即可**。

微服务自身还带着sentinel内部配置的流控规则，如果满足也会被触发：Sentinel访问触发了自定义的限流配置,在注解@SentinelResource里面配置的blockHandler方法。

导入依赖：

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-loadbalancer</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
</dependency>
```

首先在配置文件中开启支持：

```yml
feign:
  sentinel:
    enabled: true
```

FeignClient：

```java
@FeignClient(value = "user-service",path = "/user",fallback = UserClientFallback.class)
public interface UserClient {

    @RequestMapping("/{uid}")
    User getUserById(@PathVariable("uid") int uid);
}
```

创建实现类：

```java
@Component
public class UserClientFallback implements UserClient {
    @Override
    public User getUserById(int uid) {
        User user = new User();
        user.setName("我是替代方案");
        return user;
    }
}
```

然后直接启动就可以了，中途的时候把用户服务全部下掉，可以看到正常使用替代方案：

![image-20240428220931365](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202404282209215.png)

当服务内部限流时：

```java
@Service
public class BookServiceImpl implements BookService {
    @Resource
    private BookMapper bookMapper;
    @Override
    @SentinelResource(value = "getBook",blockHandler = "handleBlockHandler")   //监控此方法，无论被谁执行都在监控范围内，这里给的value是自定义名称，这个注解可以加在任何方法上，包括Controller中的请求映射方法
    public Book getBookById(Integer bid) {
        return bookMapper.getBookById(bid);
    }
    public Book handleBlockHandler(Integer bid, BlockException e) {
        Book book = new Book();
        book.setTitle("这是内部的错误");
        return book;
    }
}
```

![image-20240428223844316](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202404282238193.png)

当连续点击两次后，会出现限流：

![image-20240428223925834](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202404282239480.png)

同样，在使用Openfeign调用时，也会出现：

![image-20240428224012812](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202404282240534.png)