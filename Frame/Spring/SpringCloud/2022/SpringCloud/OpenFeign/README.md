# OpenFeign 远程调用

**声明式 REST 客户端**：Feign 通过使用 JAX-RS (`Java Api eXtensions of RESTful web  Servivces`) 或 SpringMVC 注解的修饰方式，生成接口的动态实现。 

**Feign是一个声明式的HTTP客户端组件，它旨在是编写Http客户端变得更加容易**。OpenFeign添加了对于Spring MVC注解的支持，同时集成了[Spring Cloud LoadBalancer](../LoadBalancer/README.md)和[Spring Cloud CircuitBreaker](../CircuitBreaker/README.md)，在使用Feign时，提供Http客户端的负载均衡和熔断降级的功能。同时可以集成阿里巴巴Sentinel来提供熔断、降级等功能。通过OpenFeign只需要定义服务绑定接口且以声明式的方法，优雅而简单的实现了服务调用。

在使用SpringCloud LoadBalancer+RestTemplate时，利用RestTemplate对http请求的封装处理形成了一套模版化的调用方法。但是在实际开发中，由于对服务依赖的调用可能不止一处，往往一个接口会被多处调用，所以通常都会针对每个微服务自行封装一些客户端类来包装这些依赖服务的调用。所以，OpenFeign在此基础上做了进一步封装，由他来帮助定义和实现依赖服务接口的定义。在OpenFeign的实现下，只需创建一个接口并使用注解的方式来配置它 (在一个微服务接口上面标注一个@FeignClient 注解将提供者提供的 Restful 服务伪装为接口进行消费)，即可完成对服务提供方的接口绑定，统一对外暴露可以被调用的接口方法，大大简化和降低了调用客户端的开发量，也即由服务提供者给出调用接口清单，消费者只需使用"**feign 接口 + 注解**"的方式即可直接调用提供者提供的 Restful 服务。 

**OpenFeign 优点**： 

- 可插拔的注解支持，包括Feign注解和JAX-RS注解，通过注解的方式实现 RESTful 请求的
- 支持可插拔的HTTP解码器和解码器
- 只涉及 Consumer，与 Provider 无关。因为其是用于 Consumer 调用 Provider 的 
- 仅仅就是一个伪客户端，其不会对请求做任务的处理
- 支持Sentinel和Fallback
- 支持Spring Cloud LoadBalancer负载均衡
- 支持HTTP请求和响应压缩 

**官方文档**：https://docs.spring.io/spring-cloud-openfeign/docs/current/reference/html/

![image-20240413135853079](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202404131358497.png)

****

**OpenFeign 与 Ribbon**：

OpenFeign 具有负载均衡功能，其可以对指定的微服务采用负载均衡方式进行消费、访问。之前老版本 Spring Cloud 所集成的 OpenFeign 默认采用了 Ribbon 负载均衡器。但由于 Netflix 已不再维护 Ribbon，所以从 Spring Cloud 2021.x 开始集成的 OpenFeign 中已彻底丢弃 Ribbon，而是采用 Spring Cloud 自行研发的 Spring Cloud Loadbalancer 作为负载均衡器。

## 远程调用及负载均衡

**在borrow添加依赖**：

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
<!--由于SpringCloud Feign在Hoxton.M2 版本之后不再使用Ribbon而是使用spring-cloud-loadbalancer，所以不引入spring-cloud-loadbalancer会报错-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-loadbalancer</artifactId>
</dependency>
```

在启动类添加`@EnableFeignClients`注解来开启OpenFeign：

```java
@SpringBootApplication
@EnableFeignClients  //开启OpenFeign
public class BorrowApplication {
    public static void main(String[] args) {
        SpringApplication.run(BorrowApplication.class,args);
    }
}
```

需要调用其他微服务提供的接口可以直接创建一个对应服务的接口类：这里的接口名可以是任意的名称，接口中的方法名也可以是任意的名称。但 `@FeignClient` 参数指定的提供者服务名称是不能修改的，接口与方法上添加的`@XxxMapping` 中的参数是不能修改的，必须与提供者相应的请求 URI 相同。 由于其充当的是业务接口，所以一般其定义在 service 包中。

**注意**：FeignClient和XxxMapping注解不能连用，会出异常。

**常见错误**：

![image-20240413143219831](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202404131432164.png)

> UserClient.java

```java
@FeignClient(value = "user-service",path = "/user")   //声明为user-service服务的HTTP请求客户端
public interface UserClient {
    //路径保证和其他微服务提供的一致即可
    @RequestMapping("/{uid}")
    User getUserById(@PathVariable("uid") Integer uid);  //参数和返回值也保持一致
}
```

> BookClient.java

```java
@FeignClient(value = "book-service",path = "/book") //声明为user-service服务的HTTP请求客户端
public interface BookClient {
    //路径保证和其他微服务提供的一致即可
    @RequestMapping("/{bid}")
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

> BorrowController

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

重启工程，访问 http://localhost:8301/test-load-balancer 刷新发现浏览器轮流显示：

![](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202311110040292.gif)

这样，实现了OpenFeign负载均衡。

## 超时配置

在Spring Cloud微服务架构中，大部分公司都是利用OpenFeign进行服务间的调用，而比较简单的业务使用默认配置是不会有多大问题的，但是如果是业务比较复杂，服务要进行比较繁杂的业务计算，那后台很有可能会出现Read Timeout这个异常，因此需要定制化配置超时时间。

OpenFeign集成了负载均衡组件LoadBalancer，OpenFeign提供了2个超时参数：

- connectTimeout：防止由于服务器处理时间长而阻塞调用者。


- readTimeout：从连接建立时开始应用，在返回响应时间过长时触发。

服务提供方图书服务添加暂停62秒 (默认等待60秒钟)：

```java
@RequestMapping("/book/{uid}")
public Book findUserById(@PathVariable("uid") Integer uid){
    //测试feign超时时间
    try {
        TimeUnit.SECONDS.sleep(62);
    } catch (InterruptedException e) {
        throw new RuntimeException(e);
    }
    return service.getBookById(uid);
}
```

服务调用方借阅服务捕捉超时异常：

```java
public UserBorrowDetail getUserBorrowDetailByUid(Integer uid) {
    List<Borrow> borrow = mapper.getBorrowsByUid(uid);
    User user = userClient.getUserById(uid);

    List<Book> bookList = null;
    try {
        System.out.println("调用开始-----:"+  LocalDateTime.now());
        bookList = borrow
                .stream()
                .map(b -> bookClient.getBookById(b.getBid()))
                .collect(Collectors.toList());
    } catch (Exception e) {
        e.printStackTrace();
        System.out.println("调用结束-----:"+ LocalDateTime.now());
    }
    return new UserBorrowDetail(user, bookList);
}
```

![image-20240306214140885](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202403062141624.png)

### 全局超时设置


对于所有的FeignClient配置，可以使用"default"假名设置全局超时：

```yaml
spring:
  cloud:
    openfeign:
      client:
        config:
          default:
            # 连接超时: consumer连接上provider的时间阈值，起决定作用的是网络状况
            connect-timeout: 5000 #防止由于服务器处理时间长而阻塞调用者 单位：毫秒
            # 读超时: consumer发出请求到接收到provider响应的时间阈值，其决定作用的是provider的业务逻辑
            read-timeout: 5000 #从连接建立时开始应用，在返回响应时间过长时触发 单位：毫秒
```

### 局部超时设置

**说明**：局部设置的优先级要高于全局设置

****

如果只对于具体FeignClient配置，把default换成具体的FeignClient的名字即可完成局部超时的设置：

```yaml
spring:
  cloud:
    openfeign:
      client:
        config:
          book-service:
            connect-timeout: 5000 #防止由于服务器处理时间长而阻塞调用者
            read-timeout: 5000 #从连接建立时开始应用，在返回响应时间过长时触发
```

## 重试机制

默认重试是关闭的，系统调用一次就会结束：

![](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202403061737221.png)

**开启Retryer功能**：

> FeignConfig.java

```java
@Configuration
public class FeignConfig
{
    @Bean
    public Retryer myRetryer()
    {
        //return Retryer.NEVER_RETRY; //Feign默认配置是不走重试策略的

        //最大请求次数为3(1+2)，初始间隔时间为100ms，重试间最大间隔时间为1s
        return new Retryer.Default(100,1,3);
    }
}
```

## 集成熔断器 

Feign可以集成Spring Cloud CircuitBreaker熔断器，集成后，Feign将使用断路器包装的所有方法，配合Feign进行降级，可以对应接口中定义的远程调用单独进行降级操作。

比如用户服务挂掉，那么这个时候肯定是会远程调用失败的，也就是说Controller中的方法在执行过程中会直接抛出异常，进而被熔断器监控到并进行服务降级。

而实际上导致方法执行异常的根源就是远程调用失败，既然用户服务调用失败，那么就给这个远程调用添加一个替代方案，如果此远程调用失败，那么就直接上替代方案。

****

**添加依赖**：

在借阅工程pom.xml中增加resilience4j熔断组件依赖：

```xml
<dependency>		
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-circuitbreaker-resilience4j</artifactId>
</dependency>
```

**开启Feign的熔断器支持**：

在application.yml中增加配置，开启熔断支持：

```yaml
spring:
  cloud:
    openfeign:
      circuitbreaker:
        enabled: true
```

添加`UserFallbackClient`用作替代方案：

**注意**：服务降级类需要在spring容器中注册

```java
@Component   //注意，需要将其注册为Bean，Feign才能自动注入
public class UserFallbackClient implements UserClient{
    @Override
    public User getUserById(Integer uid) {   //自行对其进行实现，并返回替代方案
        User user = new User();
        user.setName("熔断降级方法返回");
        return user;
    }
}
```

**Feign熔断降级类**：

Spring Cloud CircuitBreaker支持降级概念，当熔断器打开，或者调用是出现错误，则执行降级方法。`@FeignClient`的fallback属性指定失败替代实现：

```java
//fallback参数指定为刚刚编写的实现类
@FeignClient(value = "userservice", fallback = UserFallbackClient.class) //声明为user-service服务的HTTP请求客户端
public interface UserClient {

    @RequestMapping("/user/{uid}")
    User getUserById(@PathVariable("uid") Integer uid);
}
```

启动借阅服务，关闭用户服务，调用接口已经采用替代方案作为结果。：

![image-20231116224734648](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202311162247183.png)

![image-20220325122301779](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303281409614.png)

****

如果想要获得熔断降级的异常信息，比如打印异常日志，则可以使用fallbackFactory属性指定：

```java
@Component
public class UserFallbackFactory implements FallbackFactory<UserFallbackClient> {
    @Override
    public UserFallbackClient create(Throwable cause) {
        cause.printStackTrace();
        return new UserFallbackClient();
    }
}
```

```java
//fallback参数指定为刚刚编写的实现类
@FeignClient(value = "user-service",path = "/user", fallbackFactory = UserFallbackFactory.class)   //声明为user-service服务的HTTP请求客户端
public interface UserClient {
    //路径保证和其他微服务提供的一致即可
    @RequestMapping("/{uid}")
    User getUserById(@PathVariable("uid") Integer uid);  //参数和返回值也保持一致
}
```

## 请求和响应压缩

**对请求和响应进行GZIP压缩**：Spring Cloud OpenFeign支持对请求和响应进行GZIP压缩，以减少通信过程中的性能损耗。

通过参数设置，就能开启请求与相应的压缩功能：

- ```properties
  spring.cloud.openfeign.compression.request.enabled=true 
  ```

- ```properties
  spring.cloud.openfeign.compression.response.enabled=true 
  ```

**细粒度化设置**：对请求压缩做一些更细致的设置，比如下面的配置内容指定压缩的请求数据类型并设置了请求压缩的大小下限，只有超过这个大小的请求才会进行压缩：

```properties
spring.cloud.openfeign.compression.request.enabled=true
spring.cloud.openfeign.compression.request.mime-types=text/xml,application/xml,application/json #触发压缩数据类型
spring.cloud.openfeign.compression.request.min-request-size=2048 #最小触发压缩的大小
```

使用配置对Feign请求和响应进行压缩：

```yaml
spring:
  cloud:
    openfeign:
      compression:
        request:
          enabled: true  # 请求压缩
          mime-types: ["text/xml", "application/xml", "application/json"]  # 压缩的类型
          min-request-size: 1024  # 请求最小压缩的阈值
        response:
          enabled: true  # 响应压缩
```

## 底层实现选择

feign 的远程调用底层实现技术默认采用的是 JDK 的 URLConnection，同时还支持 HttpClient 与 OkHttp。

由于 JDK 的 URLConnection 不支持连接池，通信效率很低，所以生产中是不会使用该默认实现的。所以在 Spring Cloud OpenFeign 中直接将默认实现变为了 HttpClient，同时也支持 OkHttp。

**可以根据业务需求选择要使用的远程调用底层实现技术**，单例模式推荐HttpClient性能更好，非单例模式下OkHttp性能更好。

**配置说明**：

在 `spring.cloud.openfeign.httpclient` 下有大量 HttpClient 的相关属性设置：

![image-20231119154200522](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202311191542616.png)

在 `spring.cloud.openfeign.httpclient.enabled` 默认为 true：

![image-20231119154235313](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202311191542397.png)

在 `spring.cloud.openfeign.httpclient.hc5.enabled` 默认为false，表明默认没有启动HttpClient5 (当前默认是HttpClient4，官方建议使用HttpClient5)：

![image-20231119154512791](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202311191545136.png)

```yaml
#  Apache HttpClient5 配置开启
spring:
  cloud:
    openfeign:
      httpclient:
        hc5:
          enabled: true
```

添加依赖：

```xml
<!-- httpclient5-->
<dependency>
    <groupId>org.apache.httpcomponents.client5</groupId>
    <artifactId>httpclient5</artifactId>
</dependency>
<!-- feign-hc5-->
<dependency>
    <groupId>io.github.openfeign</groupId>
    <artifactId>feign-hc5</artifactId>
</dependency>	
```

在 `spring.cloud.openfeign.okhttp.enabled` 默认值为 false，表明默认没有启动 OkHttp：

![image-20231119154305061](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202311191543129.png)

OkHttp 的读超时设置共用了 HttpClient 的读超时设置属性：

![image-20231119154347232](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202311191543202.png)

## Feign日志 

可以配置打开Feign日志，显示Feign调用的详细信息，比如请求和响应的headers、body和metadata。

**设置日志级别**：

Feign Logging只响应debug级别，在application.yaml中配置：

```yaml
logging:
  level:
    com.hjc: debug
```

**配置FeignLoggerLevel**：

在配置类中配置Logger.Level，告诉配置类Feign需要打印的内容：

```java
@Configuration
public class FooConfiguration {
    @Bean
    Logger.Level feignLoggerLevel() {
        return Logger.Level.FULL;
    }
}
```

![image-20231119164114685](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202311191641310.png)

Logger.Level取值：

- NONE：无日志记录（默认）。
- BASIC：只记录请求方法和 URL 以及响应状态码和执行时间。
- HEADERS：记录基本信息以及请求和响应标头。
- FULL：除了 HEADERS 中定义的信息之外，记录请求和响应的标头、正文和元数据。

## 源码解析 

### 自动配置

FeignAutoConfiguration，启用了两个配置类：FeignClientProperties和FeignHttpClientProperties。

```java
@Configuration(proxyBeanMethods = false)
@ConditionalOnClass(Feign.class)
@EnableConfigurationProperties({ FeignClientProperties.class,
    FeignHttpClientProperties.class })
@Import(DefaultGzipDecoderConfiguration.class)
public class FeignAutoConfiguration {
  。。。
}
```

defaultToProperties就是说默认配置文件的优先级更高，config是一个map，默认的key是default，对所有的feign客户端都生效，可以单独指定一个feign客户端的名字对其做配置。

```java
@ConfigurationProperties("feign.client")
public class FeignClientProperties {
  private boolean defaultToProperties = true;
  private Map<String, FeignClientConfiguration> config = new HashMap<>();
  private String defaultConfig = "default";
}
```

再看下FeignHttpClientProperties：

回到FeignAutoConfiguration，configurations是注入系统中所有的FeignClientSpecification，这个类代表的是一个feign客户端，里面有名字和配置类，每一个feign客户端都会有一个与之对应的FeignClientSpecification。

```java
@Autowired(required = false)
private List<FeignClientSpecification> configurations = new ArrayList<>();

  @Bean
  public FeignContext feignContext() {
    FeignContext context = new FeignContext();
    context.setConfigurations(this.configurations);
    return context;
  }
```

当执行FeignContext的`setConfiguration()`的时候，实际上就是把所有的feign客户端配置类实例保存到configurations成员map中，key是client的名字，value是client本身。

```java
private Map<String, C> configurations = new ConcurrentHashMap<>();
public void setConfigurations(List<C> configurations) {
  for (C client : configurations) {
    this.configurations.put(client.getName(), client);
  }
}
```

这个类还有一个非常核心的方法`createContext()`：

```java
protected AnnotationConfigApplicationContext createContext(String name) {
    AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext();
    //判断configurations是否包含那个client
    if (this.configurations.containsKey(name)) {
      //如果包含，拿到client的所有的配置类，注册到spring容器
      for (Class<?> configuration : this.configurations.get(name)
          .getConfiguration()) {
        context.register(configuration);
      }
    }
    //注册default配置, 每一个feign客户端的context中都会有默认配置
    for (Map.Entry<String, C> entry : this.configurations.entrySet()) {
      if (entry.getKey().startsWith("default.")) {
        for (Class<?> configuration : entry.getValue().getConfiguration()) {
          context.register(configuration);
        }
      }
    }
    //然后注册PropertyPlaceholderAutoConfiguration和defaultConfigType
    context.register(PropertyPlaceholderAutoConfiguration.class,
        this.defaultConfigType);
    // 然后注册了个PropertySource，名字是feign，
    // 里面是有一个kv，key是feign.client.name，value是name，就是feign客户端的名字
    context.getEnvironment().getPropertySources().addFirst(new MapPropertySource(
        this.propertySourceName,
        Collections.<String, Object>singletonMap(this.propertyName, name)));
    if (this.parent != null) {
      // 这里设置了当前context的父context
      context.setParent(this.parent);01
      context.setClassLoader(this.parent.getClassLoader());
    }
    //设置context的名字是FeignContext-feign客户端的名字
    context.setDisplayName(generateDisplayName(name));
    context.refresh();
    return context;
  }
```

也就是说每一个feign客户端都有一个AnnotationConfigApplicationContext，这个context里面注册了这个feign客户端的自己的配置类、全局默认的配置类、FeignClientsConfiguration这个配置类、用于占位符解析的配置类，添加了环境变量feign.client.name = feign客户端的名字，设置了父类Context和本Context的名字。

FeignContext构造函数中传递了FeignClientsConfiguration配置类，这个类定义了对于Web解析的组件：

```java
public class FeignContext extends NamedContextFactory<FeignClientSpecification> {

	public FeignContext() {
		super(FeignClientsConfiguration.class, "feign", "feign.client.name");
	}
}
```

```java
@Configuration
public class FeignClientsConfiguration {

  //这是响应解码器
  @Bean
  @ConditionalOnMissingBean
  public Decoder feignDecoder() {
    return new OptionalDecoder(new ResponseEntityDecoder(new SpringDecoder(this.messageConverters)));
  }
  //这是请求编码器
  @Bean
  @ConditionalOnMissingBean
  public Encoder feignEncoder() {
    return new SpringEncoder(this.messageConverters);
  }
  //这是解析注解用的Contract
  @Bean
  @ConditionalOnMissingBean
  public Contract feignContract(ConversionService feignConversionService) {
    return new SpringMvcContract(this.parameterProcessors, feignConversionService);
  }
  //ConversionService
  @Bean
  public FormattingConversionService feignConversionService() {
    FormattingConversionService conversionService = new DefaultFormattingConversionService();
    for (FeignFormatterRegistrar feignFormatterRegistrar : feignFormatterRegistrars) {
      feignFormatterRegistrar.registerFormatters(conversionService);
    }
    return conversionService;
  }
  // 这是创建HystrixFeign.builder，
  // 只有没有Feign.Builder并且启用了hystrix才会创建
  @Configuration
  @ConditionalOnClass({ HystrixCommand.class, HystrixFeign.class })
  protected static class HystrixFeignConfiguration {
    @Bean
    @Scope("prototype")
    @ConditionalOnMissingBean
    @ConditionalOnProperty(name = "feign.hystrix.enabled")
    public Feign.Builder feignHystrixBuilder() {
      return HystrixFeign.builder();
    }
  }
  // 永远不重试
  @Bean
  @ConditionalOnMissingBean
  public Retryer feignRetryer() {
    return Retryer.NEVER_RETRY;
  }
  。。。
}
```

### 注册FeignClient

首先从`@EnableFeignClient`注解开始

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@Documented
@Import(FeignClientsRegistrar.class)
public @interface EnableFeignClients {
}
```

`@Import`中FeignClientsRegistrar完成注册，这里面主要就干了2件事，注册默认的配置和注册feign客户端，当然在注册客户端的同时也会注册客户端上的自定义的配置类。先看下registerDefaultConfiguration：

```java
class FeignClientsRegistrar
    implements ImportBeanDefinitionRegistrar, ResourceLoaderAware, EnvironmentAware {
  @Override
  public void registerBeanDefinitions(AnnotationMetadata metadata,
      BeanDefinitionRegistry registry)
    // 注册默认的配置
    registerDefaultConfiguration(metadata, registry);
    // 注册feign客户端
    registerFeignClients(metadata, registry);
  }
}
```

注册feign客户端本身registerFeignClient：

```java
private void registerFeignClient(BeanDefinitionRegistry registry,
    AnnotationMetadata annotationMetadata, Map<String, Object> attributes) {
  String className = annotationMetadata.getClassName();
  //实际注册到Spring容器的类型是FeignClientFactoryBean
  BeanDefinitionBuilder definition = BeanDefinitionBuilder
      .genericBeanDefinition(FeignClientFactoryBean.class);
  validate(attributes);
  definition.addPropertyValue("url", getUrl(attributes));
  definition.addPropertyValue("path", getPath(attributes));
  String name = getName(attributes);
  //设置name
  definition.addPropertyValue("name", name);
  String contextId = getContextId(attributes);
  //设置contextId
  definition.addPropertyValue("contextId", contextId);
  //设置类型是接口类型
  definition.addPropertyValue("type", className);
  definition.addPropertyValue("decode404", attributes.get("decode404"));
  definition.addPropertyValue("fallback", attributes.get("fallback"));
  definition.addPropertyValue("fallbackFactory", attributes.get("fallbackFactory"));
  definition.setAutowireMode(AbstractBeanDefinition.AUTOWIRE_BY_TYPE);
  
  String alias = contextId + "FeignClient";
  AbstractBeanDefinition beanDefinition = definition.getBeanDefinition();
  beanDefinition.setAttribute(FactoryBean.OBJECT_TYPE_ATTRIBUTE, className);
  // has a default, won't be null
  boolean primary = (Boolean) attributes.get("primary");
  //primary默认是true，这里设置了primary
  beanDefinition.setPrimary(primary);
  // 设置alias
  String qualifier = getQualifier(attributes);
  if (StringUtils.hasText(qualifier)) {
    alias = qualifier;
  }
  BeanDefinitionHolder holder = new BeanDefinitionHolder(beanDefinition, className,
      new String[] { alias });
  // 注册bean到Spring容器
  BeanDefinitionReaderUtils.registerBeanDefinition(holder, registry);
}
```

### 注入FeignClient

容器启动的时候，首先是走`@EnableFeignClients`去注册默认的配置类、注册FeignClient和FeignClient的配置类，然后走`@FeignAutoConfiguration`，创建FeignContext，把配置类都放进去。当`@Autowired`注入feign客户端的时候，实际注入的是FactoryBean的`getObject()`返回的那个类：

```java
class FeignClientFactoryBean
    implements FactoryBean<Object>, InitializingBean, ApplicationContextAware {
      @Override
  public Object getObject() throws Exception {
    return getTarget();
  }
  <T> T getTarget() {
    //先去拿到FeignContext
    FeignContext context = this.applicationContext.getBean(FeignContext.class);
    //构造builder
    Feign.Builder builder = feign(context);
    //如果没有配置url，走负载均衡
    if (!StringUtils.hasText(this.url)) {
      if (!this.name.startsWith("http")) {
        this.url = "http://" + this.name;
      }
      else {
        this.url = this.name;
      }
      this.url += cleanPath();
      return (T) loadBalance(builder, context,
          new HardCodedTarget<>(this.type, this.name, this.url));
    }
    if (StringUtils.hasText(this.url) && !this.url.startsWith("http")) {
      this.url = "http://" + this.url;
    }
    String url = this.url + cleanPath();
    //获取一个Client，默认是没有设置的
    Client client = getOptional(context, Client.class);
    if (client != null) {
      if (client instanceof LoadBalancerFeignClient) {
        // not load balancing because we have a url,
        // but ribbon is on the classpath, so unwrap
        client = ((LoadBalancerFeignClient) client).getDelegate();
      }
      if (client instanceof FeignBlockingLoadBalancerClient) {
        // not load balancing because we have a url,
        // but Spring Cloud LoadBalancer is on the classpath, so unwrap
        client = ((FeignBlockingLoadBalancerClient) client).getDelegate();
      }
      builder.client(client);
    }
    //创建target，这里面就是feign自己去创建动态代理了
    Targeter targeter = get(context, Targeter.class);
    return (T) targeter.target(this, builder, context,
        new HardCodedTarget<>(this.type, this.name, url));
  }
}
```

FeignContext在FeignAutoConfiguration里面已经注册到容器了，构造那个builder：

```java
protected Feign.Builder feign(FeignContext context) {
  FeignLoggerFactory loggerFactory = get(context, FeignLoggerFactory.class);
  Logger logger = loggerFactory.create(this.type);
  // @formatter:off
  //拿到Feign.Builder
  Feign.Builder builder = get(context, Feign.Builder.class)
      // required values
      .logger(logger)
      //encoder
      .encoder(get(context, Encoder.class))
      //decoder
      .decoder(get(context, Decoder.class))
      //contract
      .contract(get(context, Contract.class));
  // @formatter:on
  configureFeign(context, builder);
  return builder;
}
```

这个其实就是拿到feign客户端的name对应的那个ApplicationContext，因此，最终就是从这个Context中去拿Type，如果这个Context不存在就去创建，前面已经分析过了，创建的时候回去注册很多配置类，其中有一个FeignClientsConfiguration，这里会创建默认的encoder、decoder一大堆。


### 总结

1. `@EnableFeignClients`会做bean扫描，向Spring容器注册全局默认配置、FeignClient、FeignClient配置，其中FeignClient不是普通的Bean，而是一个FeignClientFactoryBean

2. FeignAutoConfiguration向Spring容器注册了FeignContext，FeignContext里面包含了所有的FeignClient，每一个FeignClient都关联了一个ApplicationContext，这个ApplicationContext中就包含了encoder、decoder、contract那些核心组件，SpringMvcContract就是用来解析web相关注解的。

3. 当`@Autowired`注入FeignClient接口的时候，实际注入的是FeignClientFactoryBean的`getObject()`返回的bean，在`getObject()`里面调用了`buidler.target()`返回了FeignClient实例。

4. 如果在子线程中调用feign接口，需要注意子线程中是无法获取HttpServletRequest的，此时就算是feign接口配置了拦截器，在拦截器里面一样是无法读取到http header的，对于某些使用拦截器统一设置http header的情况尤其要注意，feign说白了就是个http util，仅此而已。

## 多个同名Feign的报错问题

产生原因：

```java
@FeignClient(value = "userservice",fallback = UserFallbackClient.class)   //声明为userservice服务的HTTP请求客户端
public interface HelloClient {
    //路径保证和其他微服务提供的一致即可
    @RequestMapping("/user/{uid}")
    User getUserById(@PathVariable("uid") Integer uid);  //参数和返回值也保持一致
}
```

```java
@FeignClient(value = "userservice",fallback = UserFallbackClient.class)   //声明为userservice服务的HTTP请求客户端
public interface UserClient {
    //路径保证和其他微服务提供的一致即可
    @RequestMapping("/user/{uid}")
    User getUserById(@PathVariable("uid") Integer uid);  //参数和返回值也保持一致
}
```

启动抛出：

![image-20230313155813145](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303281411054.png)

解决办法：

1. 解决方法1：application.yml配置文件中配置：

   ```yaml
   spring
     main:
       allow-bean-definition-overriding: true
   ```

2. 解决方法2：

   ![image-20230313160551987](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303281411062.png)

