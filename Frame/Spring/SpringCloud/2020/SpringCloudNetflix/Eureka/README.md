# Eureka 注册中心

Eureka是**Netflix**在线影片公司开源的一个服务注册和发现组件，和其他的Netflix公司的服务组件（例如负载均衡**Ribbon**，熔断器**Hystrix**，网关**Zuul**等）一起，被Spring Cloud社区整合为Spring Cloud Netflix模块。

Eureka是一个用于服务注册和发现的组件，最开始主要应用与亚马逊公司的云计算服务平台AWS，**Eureka分为Eureka Server(Eureka服务注册中心)和Eureka Client(Eureka客户端)**。

![image.png](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303271903719.png)

官方文档：https://docs.spring.io/spring-cloud-netflix/docs/current/reference/html/

**注意**：Eureka2.x已经停更，解决方案推荐使用[Nacos](../../SpringCloudAlibaba/Nacos/README.md)作为替换方案

## 服务注册与发现

虽然服务拆分完成，但是没有一个比较合理的管理机制，如果单纯只是这样编写，在部署和维护起来，肯定是很麻烦的。可以想象一下，如果某一天这些微服务的端口或是地址大规模地发生改变，就不得不将服务之间的调用路径大规模的同步进行修改，这是一个很可怕的事情。需要削弱这种服务之间的强关联性，因此需要一个集中管理微服务的平台。

Eureka能够自动注册并发现微服务，然后对服务的状态、信息进行集中管理，当需要获取其他服务的信息时，只需要向Eureka进行查询就可以了。

![image-20220323145051821](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303271906347.png)

像这样的话，服务之间的强关联性就会被进一步削弱。

Eureka基本机构主要包括以下3个角色：

- **Eureka Server**：服务注册中心，提供服务注册和发现功能。
- **Provider Service**：服务提供者。
- **Consumer Service**：服务消费者。

###  EurekaServer

搭建一个Eureka服务器，只需要创建一个新的Maven项目即可。

![image-20230327190938605](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303271909664.png)

在父工程中添加一下SpringCloud的依赖：

```xml
<dependency>
	<groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-dependencies</artifactId>
	<version>2020.0.1</version>
    <type>pom</type>
    <scope>import</scope>
</dependency>
```

为新创建的项目添加依赖(首次导入请耐心等待一下)：

```xml
<dependencies>
	<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
    </dependency>
</dependencies>
```

创建主类：

```java
@SpringBootApplication
//@EnableEurekaServer，声明当前应用为Eureka Server
@EnableEurekaServer
public class EurekaServerApplication {
    public static void main(String[] args) {
        SpringApplication.run(EurekaServerApplication.class,args);
    }
}
```

修改配置文件：

```yaml
server:
  port: 8888
eureka:
  # 开启之前需要修改一下客户端设置（虽然是服务端
  client:
  	# 由于我们是作为服务端角色，所以不需要获取服务端，改为false，默认为true
	fetch-registry: false
	# 暂时不需要将自己也注册到Eureka
    register-with-eureka: false
    # 将eureka服务端指向自己
    service-url:
      # eureka 服务地址，如果是集群的话；需要指定其它集群eureka地址
      defaultZone: http://localhost:8888/eureka
```

启动完成后，直接输入地址+端口(http://localhost:8888/)即可访问Eureka的管理后台：

![image-20230309153127728](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303271908990.png)

### 实现服务注册

可以看到目前还没有任何的服务注册到Eureka，接着来配置一下三个微服务。

1. 导入Eureka依赖(注意别导错了，名称里面有个starter的才是)：

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

然后修改配置文件：

```yaml
eureka:
  client:
  	# 跟上面一样，需要指向Eureka服务端地址，这样才能进行注册
    service-url:
      defaultZone: http://localhost:8888/eureka
```

无需在启动类添加注解，直接启动就可以了，然后打开Eureka的服务管理页面，看到我们刚刚开启的服务：

![image-20230309153534909](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303271909976.png)

可以看到`8082`端口上的服务器，已经成功注册到Eureka了，但是这个服务名称怎么会显示为UNKNOWN，需要修改配置文件：

```yaml
spring:
  application:
    name: userservice
```

![image-20230309153645441](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303271910036.png)

**当我们的服务启动之后，会每隔一段时间跟Eureka发送一次心跳包，这样Eureka就能够感知到我们的服务是否处于正常运行状态。**

用同样的方法，将另外两个微服务也注册进来：

![image-20230309154053235](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303271910059.png)

### 实现服务发现

之前如果需要对其他微服务进行远程调用(服务)，那么就必须要知道其他服务的地址 ([SpringBoot远程调用](../../../SpringBoot/Advanced/Remote/README.md))：

```java
User user = template.getForObject("http://localhost:8082/user/"+uid, User.class);
```

> BorrowServiceImpl.java
>

```java
@Service
public class BorrowServiceImpl implements BorrowService {
    @Resource
    private BorrowMapper mapper;

    //RestTemplate支持多种方式的远程调用
    @Resource
    private RestTemplate restTemplate;

    @Resource
    private DiscoveryClient discoveryClient;

    @Override
    public UserBorrowDetail getUserBorrowDetailByUid(Integer uid) {
        List<Borrow> borrow = mapper.getBorrowsByUid(uid);
        //调用其他关联信息

        //这里通过调用getForObject来请求其他服务，并将结果自动进行封装

        //获取User信息
        List<ServiceInstance> userService = discoveryClient.getInstances("userservice");
        ServiceInstance serviceInstance = userService.get(0);

        User user = restTemplate.getForObject("http://"+serviceInstance.getHost()+":"+serviceInstance.getPort()+"/user/" + uid, User.class);

        //获取每一本书的详细信息
        List<ServiceInstance> bookService = discoveryClient.getInstances("bookservice");
        ServiceInstance serviceInstance2 = bookService.get(0);
        List<Book> bookList = borrow
                .stream()
                .map(borrow1 -> restTemplate.getForObject("http://"+serviceInstance2.getHost()+":"+serviceInstance2.getPort()+"/book/" + borrow1.getBid(), Book.class))
                .collect(Collectors.toList());

        return new UserBorrowDetail(user,bookList);
    }
}
```

主类添加RestTemplate：

```java
@SpringBootApplication
public class BorrowApplication {
    public static void main(String[] args) {
        SpringApplication.run(BorrowApplication.class,args);
    }

    @Bean
    public RestTemplate restTemplate(){
        return new RestTemplate();
    }
}
```

访问http://localhost:8082/borrow/1：

![image-20230309155721366](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303271910295.png)

### 负载均衡

Eureka可以直接向其进行查询，得到对应的微服务地址：

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

        //这里不用再写IP，直接写服务名称userservice
        User user = restTemplate.getForObject("http://userservice/user/" + uid, User.class);
        //这里不用再写IP，直接写服务名称bookservice
        List<Book> bookList = borrow
                .stream()
                .map(b -> restTemplate.getForObject("http://bookservice/book/" + b.getBid(), Book.class))
                .collect(Collectors.toList());
        return new UserBorrowDetail(user, bookList);
    }
}
```

将RestTemplate添加`@LoadBalanced`注解，这样Eureka就会对服务的调用进行自动发现，并提供负载均衡：

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

访问http://localhost:8082/borrow/1：

![image-20230309184050142](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303271910490.png)

同一个服务器实际上是可以注册很多个的，但是它们的端口不同，比如创建多个用户查询服务，我们现在将原有的端口配置修改一下，由IDEA中设定启动参数来决定，这样就可以多创建几个不同端口的启动项了：

![image-20230327191211335](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303271912557.png)

![image-20230327191331162](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303271913128.png)

在Eureka中，同一个服务出现了三个实例：

![image-20230309184208627](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303271913384.png)

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

![image-20230309184919473](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303271916608.png)

这样，服务自动发现以及简单的负载均衡就实现完成了，并且，如果某个微服务挂掉了，只要存在其他同样的微服务实例在运行，那么就不会导致整个微服务不可用，极大地保证了安全性。

## Eureka的自我保护

当有一个新的Eureka Server出现时，他尝试从相邻的节点获取所有服务实例注册信息。如果从相邻的节点获取信息时出现了故障，Eureka Server会尝试其他的节点。如果Eureka Server能够成功获取所有的服务实例信息。则根据配置信息设置服务续约的阈值。在任何时间，如果Eureka Server接收到的服务续约低于为该值配置的百分比（默认为15分钟内低于85%），则服务器开启自我保护模式，即不再剔除注册列表的信息。

这样做的好处在于，如果Eureka Server自身的网络问题而导致Eureka Client无法续约，Eureka Client的注册列表信息不再被删除，也就是Eureka Client还可以被其他服务消费。自我保护开启后：

![image-20230309185818989](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303271913916.png)

在默认情况下，Eureka Server的自我保护模式是开启的，生产环境下这很有效，保证了大多数服务依然可用，但是这给我们的开发带来了麻烦， 因此开发阶段我们都会关闭自我保护模式。

```yaml
eureka:
  server:
    enable-self-preservation: false # 关闭自我保护模式（缺省为打开）
    eviction-interval-timer-in-ms: 1000 # 扫描失效服务的间隔时间（缺省为60*1000ms）
```

## 注册中心高可用

Eureka Server不但需要接收服务的心跳，用来检测服务是否可用，而且每个服务会定期会去Eureka申请服务列表的信息，当服务实例很多时，Eureka中的负载就会很大，所以必须实现Eureka服务注册中心的高可用，避免Eureka服务器崩溃了，所有需要用到服务发现的微服务也会崩溃。搭建Eureka集群，存在多个Eureka服务器，这样就算挂掉其中一个，其他的也还在正常运行，就不会使得服务注册与发现不可用。

![image-20220323205531185](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303271913816.png)

修改一下Eureka服务端的配置文件：

> application.yml

```yaml
server:
  port: 8888
spring:
  application:
    name: eurekaserver
eureka:
  instance:
    # 由于不支持多个localhost的Eureka服务器，但是又只有本地测试环境，所以就只能自定义主机名称了
    # 主机名称改为eureka
    hostname: eureka
  # 开启之前需要修改一下客户端设置（虽然是服务端
  client:
    # 由于我们是作为服务端角色，所以不需要获取服务端，改为false，默认为true
    fetch-registry: false
    # 暂时不需要将自己也注册到Eureka
#    register-with-eureka: false
    # 去掉register-with-eureka选项，让Eureka服务器自己注册到其他Eureka服务器，这样才能相互启用
    service-url:
    # 将eureka服务端指向自己
#    service-url:
#      defaultZone: http://localhost:8888/eureka
      # eureka 服务地址，如果是集群的话；需要指定其它集群eureka地址
      # 注意这里填写其他Eureka服务器的地址，不用写自己的
      defaultZone: http://eureka01:8801/eureka,http://eureka02:8802/eureka
  server:
    eviction-interval-timer-in-ms: 1000 # 扫描失效服务的间隔时间（缺省为60*1000ms）
    enable-self-preservation: false # 关闭自我保护模式（缺省为打开）
```

> application-eureka1.yml

```yaml
server:
  port: 8801
spring:
  application:
    name: eurekaserver
eureka:
  instance:
    hostname: eureka01
  client:
    fetch-registry: false
    service-url:
      defaultZone: http://eureka:8888/eureka,http://eureka02:8802/eureka
  server:
    eviction-interval-timer-in-ms: 1000
    enable-self-preservation: false
```

> application-eureka2.yml

```yaml
server:
  port: 8802
spring:
  application:
    name: eurekaserver
eureka:
  instance:
    hostname: eureka02
  client:
    fetch-registry: false
    service-url:
      defaultZone: http://eureka:8888/eureka,http://eureka01:8801/eureka
  server:
    eviction-interval-timer-in-ms: 1000
    enable-self-preservation: false
```

由于修改成自定义的地址，需要在hosts文件中将其解析到127.0.0.1才能回到localhost，Mac下文件路径为`/etc/hosts`，Windows下为`C:\Windows\system32\drivers\etc\hosts`：

![image-20230310095658812](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303271914261.png)

对创建的两个配置文件分别添加启动配置，直接使用`spring.profiles.active`指定启用的配置文件即可：

![image-20230313101439222](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303271914113.png)

接着启动这三个注册中心，这三个Eureka管理页面都可以被访问，我们访问http://eureka02:8802/：

![image-20230310095052008](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303271914033.png)

![image-20230310095135327](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303271914479.png)

可以看到下方`replicas`中已经包含了另一个Eureka服务器的地址，并且是可用状态。

接着需要将微服务配置也进行修改：

```yaml
eureka:
  client:
    	# 将三个Eureka的地址都加入，这样就算有一个Eureka挂掉，也能完成注册
    service-url:
      defaultZone: http://eureka:8888/eureka,http://eureka01:8801/eureka,http://eureka02:8802/eureka
```

可以看到，服务全部成功注册，并且两个Eureka服务端都显示为已注册：

![image-20230310100159970](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303271904703.png)

接着模拟将其中一个Eureka服务器关闭掉，可以看到它会直接变成不可用状态：

![image-20230310100328630](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303271904841.png)

如果这个时候重启刚刚关闭的Eureka服务器，会自动同步其他Eureka服务器的数据。

## 源码解析

### Eureka相关概念

#### Register：服务注册

当Eureka Client向Eureka Server注册时，Eureka Client提供自身的元数据，比如IP地址、端口、运行状况指标的URL，主页地址等信息。

#### Renew：服务续约

Eureka Client在默认情况下会每隔30秒发送一次心跳来进行服务续约，通过服务续约来告知Eureka Server该Eureka Client依然可用，正常情况下，如果Eureka Server在90秒内没有收到Eureka Client的心跳，Eureka Server会将Eureka Client实例从注册列表中删除，注意：官网建议不要更改服务续约的间隔时间。

#### Fetch Registries：获取服务注册列表信息

Eureka Client从Eureka Server获取服务注册表信息，并将其缓存到本地。Eureka Client 会使用服务注册列表信息查找其他服务的信息，从而进行远程调用，改注册列表信息定时（每隔30秒）更新一次，每次返回的注册列表信息可能与Eureka Client的缓存信息不同，Erueka Client会重新获取整个注册表信息。Eureka Server缓存了所有的服务注册表信息，并且进行了压缩。Eureka Client和Eureka Server可以使用json和xml的数据格式进行通信，默认，Eureka Client使用JSON格式方式来获取服务器注册列表信息。

#### Cancel：服务下线

Eureka Client在程序关闭时可以向Eureka Server发送下线请求，发送请求后，该客户端的实例信息将从Eureka Server的服务注册列表信息中删除。改下线请求不会自动完成，需要在程序关闭时调用以下代码

> DiscoveryManager.getInstance().shutdownComponent();

#### Eviction：服务

在默认情况下，Eureka Client连续90秒没有想Eureka Server发送服务续约（心跳）时，Eureka Server会将该服务实例从服务列表中删除。即服务剔除。

### Register 服务注册 

#### Eureka Client源码

Eureka Client向Eureka Server提交自己的服务信息，包括IP、端口、ServiceId等信息。如果Eureka Client没有配置ServiceId，则默认为配置文件中的配置的服务名，即`${spring.application.name}`的值。

1. **DiscoveryClient初始化方法initScheduledTasks方法**：

   该方法主要开启了获取服务注册列表的信息，如果需要向Eureka Server注册，则开启注册，同时开启了定时任务向Eureka Server服务续约：

   ```java
       /**
        * Initializes all scheduled tasks.
        */
       private void initScheduledTasks() {
           if (clientConfig.shouldFetchRegistry()) {
   			...//省略了任务调度获取注册列表的代码。
           }
   
           if (clientConfig.shouldRegisterWithEureka()) {
   			...
               // Heartbeat timer
               heartbeatTask = new TimedSupervisorTask(
                       "heartbeat",
                       scheduler,
                       heartbeatExecutor,
                       renewalIntervalInSecs,
                       TimeUnit.SECONDS,
                       expBackOffBound,
                       new HeartbeatThread()
               );
               scheduler.schedule(
                       heartbeatTask,
                       renewalIntervalInSecs, TimeUnit.SECONDS);
   
               // InstanceInfo replicator
               instanceInfoReplicator = new InstanceInfoReplicator(
                       this,
                       instanceInfo,
                       clientConfig.getInstanceInfoReplicationIntervalSeconds(),
                       2); // burstSize
               ...
           }
       }
   ```

2. **instanceInfoReplicator类**：

   initScheduledTasks方法中，定时任务调用instanceInfoReplicator类，instanceInfoReplicator类继承Runable接口，run方法代码如下：

   ```java
       public void run() {
           try {
               discoveryClient.refreshInstanceInfo();
   
               Long dirtyTimestamp = instanceInfo.isDirtyWithTime();
               if (dirtyTimestamp != null) {
                   discoveryClient.register();
                   instanceInfo.unsetIsDirty(dirtyTimestamp);
               }
           } catch (Throwable t) {
               logger.warn("There was a problem with the instance info replicator", t);
           } finally {
               Future next = scheduler.schedule(this, replicationIntervalSeconds, TimeUnit.SECONDS);
               scheduledPeriodicRef.set(next);
           }
       }
   ```

3. **DiscoveryClient的register方法**：

   在com.netflix.discovery包下的DiscoveryClient类中有一个register()方法，该方法通过Http请求向Eureka Server注册：

   ```java
       boolean register() throws Throwable {
           logger.info(PREFIX + "{}: registering service...", appPathIdentifier);
           EurekaHttpResponse<Void> httpResponse;
           try {
               httpResponse = eurekaTransport.registrationClient.register(instanceInfo);
           } catch (Exception e) {
               logger.warn(PREFIX + "{} - registration failed {}", appPathIdentifier, e.getMessage(), e);
               throw e;
           }
           if (logger.isInfoEnabled()) {
               logger.info(PREFIX + "{} - registration status: {}", appPathIdentifier, httpResponse.getStatusCode());
           }
           return httpResponse.getStatusCode() == Status.NO_CONTENT.getStatusCode();
       }
   ```

#### Eureka Server端源码

1. **ApplicationResource类**：

   ApplicationResource类的addInstance方法，接收Eureka Client客户端注册请求，完成注册：

   ```java
       /**
        * Registers information about a particular instance for an
        * {@link com.netflix.discovery.shared.Application}.
        *
        * @param info
        *            {@link InstanceInfo} information of the instance.
        * @param isReplication
        *            a header parameter containing information whether this is
        *            replicated from other nodes.
        */
       @POST
       @Consumes({"application/json", "application/xml"})
       public Response addInstance(InstanceInfo info,
                                   @HeaderParam(PeerEurekaNode.HEADER_REPLICATION) String isReplication) {
           ...
           registry.register(info, "true".equals(isReplication));
           return Response.status(204).build();  // 204 to be backwards compatible
       }
   ```

2. **PeerAwareInstanceRegistryImpl类**：

   上面addInstance方法调用PeerAwareInstanceRegistryImpl类的register方法进行注册：

   ```java
       @Override
       public void register(final InstanceInfo info, final boolean isReplication) {
           int leaseDuration = Lease.DEFAULT_DURATION_IN_SECS;
           if (info.getLeaseInfo() != null && info.getLeaseInfo().getDurationInSecs() > 0) {
               leaseDuration = info.getLeaseInfo().getDurationInSecs();
           }
           super.register(info, leaseDuration, isReplication);
           //高可用，多节点同步数据
           replicateToPeers(Action.Register, info.getAppName(), info.getId(), info, null, isReplication);
       }
   ```

### Renew 服务续约 

服务续约和服务注册非常类似，服务注册在Eureka Client程序启动后开启，并且同时开启服务续约定时任务。

#### Eureka Client端

在DiscoveryClient类下又renew()方法，完成续约：

```java
    /**
     * Renew with the eureka service by making the appropriate REST call
     */
    boolean renew() {
        EurekaHttpResponse<InstanceInfo> httpResponse;
        try {
            httpResponse = eurekaTransport.registrationClient.sendHeartBeat(instanceInfo.getAppName(), instanceInfo.getId(), instanceInfo, null);
            logger.debug(PREFIX + "{} - Heartbeat status: {}", appPathIdentifier, httpResponse.getStatusCode());
            if (httpResponse.getStatusCode() == Status.NOT_FOUND.getStatusCode()) {
                REREGISTER_COUNTER.increment();
                logger.info(PREFIX + "{} - Re-registering apps/{}", appPathIdentifier, instanceInfo.getAppName());
                long timestamp = instanceInfo.setIsDirtyWithTime();
                boolean success = register();
                if (success) {
                    instanceInfo.unsetIsDirty(timestamp);
                }
                return success;
            }
            return httpResponse.getStatusCode() == Status.OK.getStatusCode();
        } catch (Throwable e) {
            logger.error(PREFIX + "{} - was unable to send heartbeat!", appPathIdentifier, e);
            return false;
        }
    }
```

#### Eureka Server端

在com.netflix.eureka.InstanceResource类下，接口方法renewLease()，它是一个RESTful API接口。完成服务器断续：

```java
    @PUT
    public Response renewLease(
            @HeaderParam(PeerEurekaNode.HEADER_REPLICATION) String isReplication,
            @QueryParam("overriddenstatus") String overriddenStatus,
            @QueryParam("status") String status,
            @QueryParam("lastDirtyTimestamp") String lastDirtyTimestamp) {
        ...
        boolean isSuccess = registry.renew(app.getName(), id, isFromReplicaNode);
		...
    }
```

此外，服务续约的两个参数是可以配置的，即Eureka Client发送续约心跳间隔时间参数，和Eureka Server多长时间内没有收到心跳将实例剔除的时间参数，默认情况下这两个参数分别是30秒和90秒

```yaml
eureka:
  instance:
  	# 心跳间隔时间
    lease-renewal-interval-in-seconds: 30
    # 没收到心跳多长时间剔除
    lease-expiration-duration-in-seconds: 90
```

