# OpenFeign 实现负载均衡

**Feign是一个声明式的HTTP客户端组件，它旨在是编写Http客户端变得更加容易**。OpenFeign添加了对于Spring MVC注解的支持，同时集成了Spring Cloud LoadBalancer和Spring Cloud CircuitBreaker，在使用Feign时，提供负载均衡和熔断降级的功能。

官方文档：https://docs.spring.io/spring-cloud-openfeign/docs/current/reference/html/

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

## Feign日志 

可以配置打开Feign日志，显示Feign调用的详细信息，比如请求和响应的headers、body和metadata。

**设置日志级别**：

Feign Logging只响应debug级别，在application.yml中配置：

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

![image-20230313155607396](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303281410397.png)

Logger.Level取值：

- NONE：无日志记录（默认）。
- BASIC：只记录请求方法和 URL 以及响应状态码和执行时间。
- HEADERS：记录基本信息以及请求和响应标头。
- FULL：记录请求和响应的标头、正文和元数据。

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

