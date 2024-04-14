# Nacos 服务注册与发现

**官方文档**：https://github.com/alibaba/spring-cloud-alibaba/wiki/Nacos-discovery

## 服务注册与发现

实现基于Nacos的服务注册与发现，需要导入SpringCloudAlibaba相关的依赖。

在父工程将依赖进行管理：

```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>3.0.2</version>
        </dependency>
      
      	<!-- 引入SpringCloud依赖 -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>2020.0.1</version>
          	<type>pom</type>
            <scope>import</scope>
        </dependency>

     	<!-- 引入SpringCloudAlibaba依赖 -->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-alibaba-dependencies</artifactId>
            <version>2022.0.0.0</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

在子项目中添加服务发现依赖：

```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
</dependency>
```

在配置文件中配置Nacos注册中心的地址：

```yaml
server:
  # 图书服务8201/用户服务8101/借书服务8301
  port: 8101
spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/springcloud?useSSL=false&useUnicode=true&characterEncoding=utf8&serverTimezone=Asia/Shanghai
    username: root
    password: 123123
  # 应用名称 book-service/borrow-service/user-service
  application:
    name: book-service
  cloud:
    nacos:
      discovery:
        # 开启注册与发现 默认为true
        enabled: true
        # 配置Nacos注册中心地址
        server-addr: localhost:8848
        # 用户名
        username: nacos
        # 密码
        password: nacos
```

接着启动服务，可以在Nacos的服务列表中找到：

![image-20231110214229829](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202311102142286.png)

## 调用服务并实现负载均衡

使用 [LoadBalancer](../../../SpringCloud/LoadBalancer/README.md) 调用服务提供者并实现负载均衡

1. 导入依赖：

   ```xml
   <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-loadbalancer</artifactId>
   </dependency>
   ```

2. 添加配置类，注册RestTemplate：

   ```java
   @Configuration
   public class SpringConfig {
       @LoadBalanced
       @Bean
       public RestTemplate restTemplate(){
           return new RestTemplate();
       }
   }
   ```

3. 将调用路径改为 `http://服务名/`

   ```java
   @Service
   public class BorrowServiceImpl implements BorrowService {
       @Resource
       private BorrowMapper mapper;
   
       //RestTemplate支持多种方式的远程调用
       @Resource
       private RestTemplate restTemplate;
   
       //直连方式
   //    public static final String USER_PROVIDER = "http://localhost:8101/user/";
       //微服务方式
       public static final String USER_PROVIDER = "http://user-service/user/";
       //直连方式
   //    public static final String BOOK_PROVIDER = "http://localhost:8201/book/";
       //微服务方式
       public static final String BOOK_PROVIDER = "http://book-service/book/";
   
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

   启动，并访问 http://localhost:8301/borrow/1 ：

   ![image-20231111002256579](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202311110022651.png)

4. 配置多个服务并修改端口号：

   ![image-20231118141605267](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202311191250891.png)

5. 添加测试方法：

   ```java
   @RestController
   public class BorrowController {
       @Resource
       BorrowService service;
       @Resource
       private LoadBalancerClient loadBalancerClient;
   
       //通过 DiscoveryClient API 获取服务列表
       @Resource
       private DiscoveryClient discoveryClient;
   
       @RequestMapping("/borrow/{uid}")
       public UserBorrowDetail findUserBorrows(@PathVariable("uid") Integer uid) {
           return service.getUserBorrowDetailByUid(uid);
       }
   
       @GetMapping("/test-load-balancer")
       public String testLoadBalancer() {
           //获取负载均衡的服务提供者实例信息
           ServiceInstance instance = loadBalancerClient.choose("user-service");
           return instance.getHost() + ":" + instance.getPort();
       }
   
       //获取到“服务发现 Client”，即可读取到注册中心的微服务列表
       @GetMapping("/discovery")
       public List<String> discovery() {
           
           List<String> services = discoveryClient.getServices();
           for (String serviceName :
                   services) {
               List<ServiceInstance> instances = discoveryClient.getInstances(serviceName);
               for (ServiceInstance instance :
                       instances) {
                   Map<String, Object> map = new HashMap<>();
                   map.put("serviceName",serviceName);
                   map.put("serviceId",instance.getServiceId());
                   map.put("host",instance.getHost());
                   map.put("port",instance.getPort());
                   map.put("uri",instance.getUri());
                   System.out.println(map);
               }
           }
           return services;
       }
   }
   ```

   启动，访问 http://localhost:8301/test-load-balancer 查看结果：

   ![](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202311110040292.gif)

   访问 http://localhost:8301/discovery 获取到“服务发现 Client”，读取 到注册中心的微服务列表：

   ![image-20231111114507156](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202311111145699.png)

## 服务发现远程调用

使用[OpenFeign](../../../SpringCloud/OpenFeign/README.md)，也可以实现服务发现远程调用并实现负载均衡。

导入依赖：

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-loadbalancer</artifactId>
</dependency>
```

编写接口：

```java
@FeignClient("user-service")
public interface UserClient {
    
    @RequestMapping("/user/{uid}")
    User getUserById(@PathVariable("uid") int uid);
}
```

```java
@FeignClient("book-service")
public interface BookClient {

    @RequestMapping("/book/{bid}")
    Book getBookById(@PathVariable("bid") int bid);
}
```

```java
@Service
public class BorrowServiceImpl implements BorrowService{

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

```java
@EnableFeignClients
@SpringBootApplication
public class BorrowApplication {
    public static void main(String[] args) {
        SpringApplication.run(BorrowApplication.class, args);
    }
}
```

启动，并访问 http://localhost:8301/borrow/1 ：

![image-20231111002256579](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202311110022651.png)

配置多个服务并修改端口号：

![image-20231111002959060](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202311110030304.png)

添加测试方法：

```java
@RestController
public class BorrowController {
    @Resource
    BorrowService service;
    @Resource
    private LoadBalancerClient loadBalancerClient;

    //通过 DiscoveryClient API 获取服务列表
    @Resource
    private DiscoveryClient discoveryClient;

    @RequestMapping("/borrow/{uid}")
    public UserBorrowDetail findUserBorrows(@PathVariable("uid") Integer uid) {
        return service.getUserBorrowDetailByUid(uid);
    }

    @GetMapping("/test-load-balancer")
    public String testLoadBalancer() {
        //获取负载均衡的服务提供者实例信息
        ServiceInstance instance = loadBalancerClient.choose("user-service");
        return instance.getHost() + ":" + instance.getPort();
    }

    //获取到“服务发现 Client”，即可读取到注册中心的微服务列表
    @GetMapping("/discovery")
    public List<String> discovery() {
        
        List<String> services = discoveryClient.getServices();
        for (String serviceName :
                services) {
            List<ServiceInstance> instances = discoveryClient.getInstances(serviceName);
            for (ServiceInstance instance :
                    instances) {
                Map<String, Object> map = new HashMap<>();
                map.put("serviceName",serviceName);
                map.put("serviceId",instance.getServiceId());
                map.put("host",instance.getHost());
                map.put("port",instance.getPort());
                map.put("uri",instance.getUri());
                System.out.println(map);
            }
        }
        return services;
    }
}
```

启动，访问 http://localhost:8301/test-load-balancer 查看结果：

![](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202311110040292.gif)

访问 http://localhost:8301/discovery 获取到“服务发现 Client”，读取 到注册中心的微服务列表：

![image-20231111114507156](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202311111145699.png)

## 注册表缓存

服务在启动后，当**发生调用**时会自动从 Nacos 注册中心下载并缓存注册表到本地。所以， 即使 Nacos 发生宕机，会发现消费者仍然是可以调用到提供者的。只不过此时已经不能再有 服务进行注册了，服务中缓存的注册列表信息无法更新。

## 临时实例与持久实例

Nacos区分了临时实例和非临时实例(持久实例)：

![image-20231111140112861](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202311111401568.png)

临时实例与持久实例的实例存储的位置与健康检测机制是不同的：

- **临时实例**：默认情况。服务实例仅会注册在 Nacos 内存，不会持久化到 Nacos 磁盘。其 健康检测机制为 Client 模式，即 Client 主动向 Server 上报其健康状态。默认心跳间隔为 5 秒。在 15 秒内 Server 未收到 Client 心跳，则会将其标记为"不健康"状态；在 30 秒 内若收到了 Client 心跳，则重新恢复“健康”状态，否则该实例将从 Server 端内存清除(**采用心跳机制向Nacos发送请求保持在线状态，一旦心跳停止，代表实例下线，不保留实例信息**)。
- **非临时实例(持久实例)**：服务实例不仅会注册到 Nacos 内存，同时也会被持久化到 Nacos 磁盘。其 健康检测机制为 Server 模式，即 Server 会主动去检测 Client 的健康状态，默认每 20 秒检 测一次。健康检测失败后服务实例会被标记为"不健康"状态，但不会被清除，因为其 是持久化在磁盘的(**由Nacos主动进行联系，如果连接失败，那么不会移除实例信息，而是将健康状态设定为false，相当于会对某个实例状态持续地进行监控**)。

可以通过配置文件进行修改临时实例为非临时实例：

```yaml
spring:
  application:
    name: borrow-service
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848
        # 将ephemeral修改为false，表示非临时实例
        ephemeral: false
```

**注意**：

- 当已经注册过临时实例，在转为非临时实例时，会报错：`当前服务DEFAULT_GROUP@@borrow-service是临时服务，不能注册持久实例`

  ![image-20231111141655274](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202311111416115.png)

  **解决办法**：可以通过更改服务名来解决：

  ```yaml
  spring:
    application:
      # 应用名称 borrow-service
      name: borrow-service-new
    cloud:
      nacos:
        discovery:
          # 配置Nacos注册中心地址
          server-addr: localhost:8848
          # 用户名
          username: nacos
          # 密码
          password: nacos
          # 将ephemeral修改为false，表示非临时实例
          ephemeral: false
  ```

- 当出现 `The Raft Group [naming_persistent_service_v2] did not find the Leader node;`时

  **解决办法**：删除 Nacos 根目录下 data 文件夹下的 protocol 文件夹即可

  ![image-20231111143633402](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202311111436634.png)

****

在Nacos中查看，可以发现实例已经不是临时的：

![image-20231111145253314](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202311131400584.png)

当关闭此实例，只是将健康状态变为false，而不会删除实例的信息：

![image-20231111145351776](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202311131400665.png)

**注销实例**：

- 当实例为临时实例时，只需要关闭服务，就会注销实例。

- 当实例为持久实例时，需要在Linux 环境下执行注销命令：

  ```shell
  curl -d 'serviceName=borrow-service-new' \
    -d 'ip=192.168.0.111' \
    -d 'port=8301' \
    -d 'username=nacos' \
    -d 'password=nacos' \
    -d 'ephemeral=false' \
    -X DELETE 'http://127.0.0.1:8848/nacos/v2/ns/instance'
  ```

  ![image-20231111151145854](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202311111511755.png)

## 集群分区

在一个分布式应用中，相同服务的实例可能会在不同的机器、位置上启动，比如用户管理服务，可能在北京有1台服务器部署、上海有一台服务器部署，而这时，在北京的服务器上启动了借阅服务，如果借阅服务要调用用户服务，就应该优先选择同一个区域的用户服务进行调用，这样会使得响应速度更快。

![image-20220326150024118](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303301702028.png)

因此，可以对部署在不同机房的服务进行分区，可以看到实例的分区是默认：

![image-20231113131125579](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202311131311148.png)

直接在配置文件中进行修改：

```yaml
spring:
  application:
    name: borrow-service
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848
        # 修改为上海地区的集群
        cluster-name: Shanghai
```

由于使用的是不同的启动配置，直接在启动配置中添加环境变量`spring.cloud.nacos.discovery.cluster-name`，将用户服务和图书服务都分配Beijing和Shanghai两个区域，借阅服务8302端口下配置为北京地区：

![image-20231113131514626](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202311131315194.png)

修改完成之后，重新启动服务（Nacos也要重启），观察Nacos中集群分布情况：

![image-20230321212629691](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303301703735.png)

现在有三个集群，并且都有一个实例正在运行。接着去调用借阅服务，但是发现并没有按照区域进行优先调用，而依然使用的是轮询模式的负载均衡调用。

区域优先调用必须要提供Nacos的负载均衡实现才能开启区域优先调用机制，只需要在配制文件中进行修改即可：

```yaml
spring:
  application:
    name: borrow-service
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848
        cluster-name: Beijing
    # 将loadbalancer的nacos支持开启，集成Nacos负载均衡
    loadbalancer:
      nacos:
        enabled: true
```

现在重启借阅服务，会发现优先调用的是同区域的用户和图书服务。

![image-20231113135640356](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202311131400246.png)

将北京地区的服务下线：

![image-20231113140021805](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202311131400234.png)

可以看到，在下线之后，由于本区域内没有可用服务了，借阅服务将会调用上海区域的用户服务。

![image-20231113135747293](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202311131357531.png)

除了根据区域优先调用之外，同一个区域内的实例也可以单独设置权重，Nacos会优先选择权重更大的实例进行调用，可以直接在管理页面中进行配置：

![image-20230321214135160](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303301704177.png)

或是在配置文件中进行配置：

```yml
spring:
  application:
    name: borrowservice
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848
        cluster-name: Chengdu
        # 权重大小，越大越优先调用，默认为1
        weight: 0.5
```

通过配置权重，某些性能不太好的机器就能够更少地被使用，而更多的使用那些网络良好性能更高的主机上的实例。