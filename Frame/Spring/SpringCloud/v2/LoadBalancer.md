# LoadBalancer 负载均衡

负载均衡是指将负载分摊到多个执行单元上，常见的负载均衡有两种方式。一种独立进程单元，通过负载均衡策略，将请求转发到不同的执行单元上，例如Nginx。另一种是将负载均衡逻辑以代码的形式封装到服务消费者的客户端上，服务消费者客户端维护了一份服务提供者的信息列表，有了信息表，通过负载均衡策略将请求分摊给多个服务提供者，从而达到负载均衡的目的。

SpringCloud原有的客户端负载均衡方案Ribbon已经被废弃，取而代之的是SpringCloud LoadBalancer，LoadBalancer是Spring Cloud Commons的一个子项目，他属于上述的第二种方式，是将负载均衡逻辑封装到客户端中，并且运行在客户端的进程里。

**在Spring Cloud构件微服务系统中，LoadBalancer作为服务消费者的负载均衡器，有两种使用方式，一种是和RestTemplate相结合，另一种是和Feign相结合，Feign已经默认集成了LoadBalancer。**

## LoadBalancer整合RestTemplate

[LoadBalancer整合RestTemplate](Netflix/Eureka.md#负载均衡)

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

所以，实际上在进行负载均衡的时候，会向Eureka发起请求，选择一个可用的对应服务，然后会返回此服务的主机地址等信息：

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

重启工程，浏览器访问http://localhost:8082/test-load-balancer，刷新发现浏览器轮流显示：

![image-20230310131903865](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303271920023.png)

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
        String name = environment.getProperty(LoadBalancerClientFactory.PROPERTY_NAME);
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

接着我们在`BlockingLoadBalancerClient`中添加断点，观察是否采用指定的策略进行请求：

![image-20220324221750289](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303271921267.png)

![image-20220324221713964](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303271921299.png)

发现访问userservice服务的策略已经更改指定的策略了。
