# LoadBalancer 负载均衡

SpringCloud原有的客户端负载均衡方案Ribbon已经被废弃，取而代之的是SpringCloud LoadBalancer，LoadBalancer是Spring Cloud Commons的一个子项目，他属于上述的第二种方式，是将负载均衡逻辑封装到客户端中，并且运行在客户端的进程里。

**在Spring Cloud构件微服务系统中，LoadBalancer作为服务消费者的负载均衡器，有两种使用方式，一种是和RestTemplate相结合，另一种是和Feign相结合，Feign已经默认集成了LoadBalancer。**

**官方文档**：https://docs.spring.io/spring-cloud-commons/reference/spring-cloud-commons/loadbalancer.html

## LoadBalancer整合RestTemplate

**添加依赖**：

```xml
<!--loadbalancer-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-loadbalancer</artifactId>
</dependency>
```

可以直接向其进行查询，得到对应的微服务地址：

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

访问http://localhost:8301/borrow/1：

![image-20240118224022574](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202401182240094.png)

同一个服务器实际上是可以注册很多个的，但是它们的端口不同，比如创建多个用户查询服务，将原有的端口配置修改一下，由IDEA中设定启动参数来决定，这样就可以多创建几个不同端口的启动项了：

![image-20240118224209782](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202401182242190.png)

同一个服务出现了三个实例：

![image-20240118224356695](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202401182243090.png)

修改用户查询，然后进行远程调用，看看请求是不是均匀地分配到服务端：

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

![image-20240118225604235](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202401182256916.png)

这样，服务自动发现以及简单的负载均衡就实现完成了，并且，如果某个微服务挂掉了，只要存在其他同样的微服务实例在运行，那么就不会导致整个微服务不可用，极大地保证了安全性。

****

实际上，在添加`@LoadBalanced`注解之后，会启用拦截器对发起的服务调用请求进行拦截（注意这里是针对我们发起的请求进行拦截），叫做LoadBalancerInterceptor，它实现ClientHttpRequestInterceptor接口：

```java
@FunctionalInterface
public interface ClientHttpRequestInterceptor {
    ClientHttpResponse intercept(HttpRequest request, byte[] body, ClientHttpRequestExecution execution) throws IOException;
}
```

主要是对`intercept`方法的实现：

```java
public ClientHttpResponse intercept(final HttpRequest request, final byte[] body, final ClientHttpRequestExecution execution) throws IOException {
    URI originalUri = request.getURI();
    String serviceName = originalUri.getHost();
    Assert.state(serviceName != null, "Request URI does not contain a valid hostname: " + originalUri);
    return (ClientHttpResponse)this.loadBalancer.execute(serviceName, this.requestFactory.createRequest(request, body, execution));
}
```

实际执行：

![image-20220323220519463](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303271920691.png)

![image-20220323220548051](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303271920247.png)

服务端会在发起请求时执行这些拦截器。

请求地址，并不是一个有效的主机名称，而是服务名称，那么怎么才能得到真正需要访问的主机名称呢，肯定是得找Eureka获取的。

`loadBalancer.execute()`的具体实现为`BlockingLoadBalancerClient`：

```java
//从上面给进来了服务的名称和具体的请求实体
public <T> T execute(String serviceId, LoadBalancerRequest<T> request) throws IOException {
    String hint = this.getHint(serviceId);
    LoadBalancerRequestAdapter<T, DefaultRequestContext> lbRequest = new LoadBalancerRequestAdapter(request, new DefaultRequestContext(request, hint));
    Set<LoadBalancerLifecycle> supportedLifecycleProcessors = this.getSupportedLifecycleProcessors(serviceId);
    supportedLifecycleProcessors.forEach((lifecycle) -> {
        lifecycle.onStart(lbRequest);
    });
  	//可以看到在这里会调用choose方法自动获取对应的服务实例信息
    ServiceInstance serviceInstance = this.choose(serviceId, lbRequest);
    if (serviceInstance == null) {
        supportedLifecycleProcessors.forEach((lifecycle) -> {
            lifecycle.onComplete(new CompletionContext(Status.DISCARD, lbRequest, new EmptyResponse()));
        });
      	//没有发现任何此服务的实例就抛异常（之前的测试中可能已经遇到了）
        throw new IllegalStateException("No instances available for " + serviceId);
    } else {
      	//成功获取到对应服务的实例，这时就可以发起HTTP请求获取信息了
        return this.execute(serviceId, serviceInstance, lbRequest);
    }
}
```

所以，实际上在进行负载均衡的时候，会向注册中心发起请求，选择一个可用的对应服务，然后会返回此服务的主机地址等信息：

![image-20220324120741736](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303271920282.png)

## LoadBlancerClient

负载均衡的核心类为LoadBalancerClient，LoadBalancerClient可以获取负载均衡的服务提供者实例信息。

在BorrowController增加代码：

```java
@Resource
private LoadBalancerClient loadBalancerClient;

@GetMapping("/test-load-balancer")
public String testLoadBalancer() {
    ServiceInstance instance = loadBalancerClient.choose("userservice");
    return instance.getHost() + ":" + instance.getPort();
}
```

> BorrowController.java

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

重启工程，浏览器访问http://localhost:8301/test-load-balancer，刷新发现浏览器轮流显示：

![image-20240118230632809](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202401182306359.png)

## LoadBalancer源码解析 

### 类的调用顺序 

当时使用有`@LoadBalanced`注解的RestTemplate时，设计的类的调用关系：

![image-20230310132143078](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303271920910.png)

### 关键类解析

- **LoadBalancerRequestFactory**： 一个工厂, 包装一个为HttpRequest对象，回调对象LoadBalancerRequest。


- **LoadBalancerClient**： 用于根据 serviceId 选取一个 ServiceInstance, 执行从 LoadBalancerRequestFactory 获得的那个回调。


- **LoadBalancerInterceptor**：RestTemplate 的拦截器, 拦截后调用 LoadBalancerClient 修改 HttpRequest 对象(主要是 url), 且传入调用 LoadBalancerRequestFactory 生成的回调给 LoadBalancerClient。


- **RestTemplateCustomizer**：为 restTemplate 加上一个拦截器(也可以干点别的, 默认就这一个用处)  。


- **SmartInitializingSingleton**：容器初始化是，调用 RestTemplateCustomizer 为容器中所有加了。@LoadBalanced 的 RestTemplate 加上一个拦截器。


- **ReactiveLoadBalancer**：定义了 choose 方法, 即如何选取一个 ServiceInstance, 如轮播, 随机。


### 源码部分 

#### LoadBalancerInterceptor

RestTemplate 的拦截器, 拦截后调用 LoadBalancerClient 修改 HttpRequest 对象(主要是 url), 且传入调用 LoadBalancerRequestFactory 生成的回调给 LoadBalancerClient。

```java
	@Override
	public ClientHttpResponse intercept(final HttpRequest request, final byte[] body,
			final ClientHttpRequestExecution execution) throws IOException {
		final URI originalUri = request.getURI();
		String serviceName = originalUri.getHost();
		Assert.state(serviceName != null, "Request URI does not contain a valid hostname: " + originalUri);
		return this.loadBalancer.execute(serviceName, this.requestFactory.createRequest(request, body, execution));
	}
```

#### BlockingLoadBalancerClient

用于根据 serviceId 选取一个 ServiceInstance, 执行从 LoadBalancerRequestFactory 获得的那个回调，发送HTTP请求，远程调用REST ful API：

```java
	@Override
	public <T> T execute(String serviceId, LoadBalancerRequest<T> request) throws IOException {
		String hint = getHint(serviceId);
		LoadBalancerRequestAdapter<T, DefaultRequestContext> lbRequest = new LoadBalancerRequestAdapter<>(request,
				new DefaultRequestContext(request, hint));
		Set<LoadBalancerLifecycle> supportedLifecycleProcessors = getSupportedLifecycleProcessors(serviceId);
		supportedLifecycleProcessors.forEach(lifecycle -> lifecycle.onStart(lbRequest));
		ServiceInstance serviceInstance = choose(serviceId, lbRequest);
		if (serviceInstance == null) {
			supportedLifecycleProcessors.forEach(lifecycle -> lifecycle.onComplete(
					new CompletionContext<>(CompletionContext.Status.DISCARD, lbRequest, new EmptyResponse())));
			throw new IllegalStateException("No instances available for " + serviceId);
		}
		return execute(serviceId, serviceInstance, lbRequest);
	}
```

choose方法，调用RoundRobinLoadBalancer的choose方法，以轮播方式获取一个serviceInstance：

```java
	@Override
	public <T> ServiceInstance choose(String serviceId, Request<T> request) {
		ReactiveLoadBalancer<ServiceInstance> loadBalancer = loadBalancerClientFactory.getInstance(serviceId);
		if (loadBalancer == null) {
			return null;
		}
		Response<ServiceInstance> loadBalancerResponse = Mono.from(loadBalancer.choose(request)).block();
		if (loadBalancerResponse == null) {
			return null;
		}
		return loadBalancerResponse.getServer();
	}
```

#### RoundRobinLoadBalancer

RoundRobinLoadBalancer的choose方法，以轮播方式获取一个serviceInstance：

```java
	public Mono<Response<ServiceInstance>> choose(Request request) {
		ServiceInstanceListSupplier supplier = serviceInstanceListSupplierProvider
				.getIfAvailable(NoopServiceInstanceListSupplier::new);
		return supplier.get(request).next()
				.map(serviceInstances -> processInstanceResponse(supplier, serviceInstances));
	}
```

## 自定义负载均衡策略

LoadBalancer默认提供了两种负载均衡策略：

- RandomLoadBalancer：随机分配策略
- RoundRobinLoadBalancer**(默认)**：轮询分配策略

修改默认的负载均衡策略可以进行指定，比如现在希望用户服务采用随机分配策略，我们需要先创建随机分配策略的配置类（不用加`@Configuration`）：

```java
public class LoadBalancerConfig {
  	//将官方提供的 RandomLoadBalancer 注册为Bean
    @Bean
    public ReactorLoadBalancer<ServiceInstance> randomLoadBalancer(Environment environment, LoadBalancerClientFactory loadBalancerClientFactory){
        //获取负载均衡客户端名称，即提供者服务名称
        String name = environment.getProperty(LoadBalancerClientFactory.PROPERTY_NAME);
        //从所有provider实例中指定名称的实力列表中随机选择一个实例
        //参数1：获取指定名称的所有provider实例列表
        //参数2：指定要获取的provider服务名称
        return new RandomLoadBalancer(loadBalancerClientFactory.getLazyProvider(name, ServiceInstanceListSupplier.class), name);
    }
}
```

为对应的服务指定负载均衡策略，直接使用注解即可：

```java
@Configuration
@LoadBalancerClient(value = "userservice",      //指定为 userservice 服务，只要是调用此服务都会使用我们指定的策略
                    configuration = LoadBalancerConfig.class)   //指定我们刚刚定义好的配置类
public class BeanConfig {
    @Bean
    @LoadBalanced
    RestTemplate template(){
        return new RestTemplate();
    }
}
```

接着在`BlockingLoadBalancerClient`中添加断点，观察是否采用指定的策略进行请求：

![image-20220324221750289](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303271921267.png)

![image-20220324221713964](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303271921299.png)

发现访问userservice服务的策略已经更改指定的策略了。

## 客户端负载和服务器端负载区别

loadbalancer本地负载均衡客户端 VS Nginx服务端负载均衡区别：

- Nginx是**服务器负载均衡**，客户端所有请求都会交给nginx，然后由nginx实现转发请求，即负载均衡是由服务端实现的。

- loadbalancer是**本地负载均衡**，在调用微服务接口时候，会在注册中心上获取注册信息服务列表之后缓存到JVM本地，从而在本地实现RPC远程服务调用技术。
