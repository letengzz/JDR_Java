# Gateway 路由网关

- [微服务网关](Expand/microservice_gateway.md)

一般情况下，可能并不是所有的微服务都需要直接暴露给外部调用，这时可以使用路由机制，添加一层防护，让所有的请求全部通过路由来转发到各个微服务，并且转发给多个相同微服务实例也可以实现负载均衡。

![image-20220325130147758](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303271423626.png)

SpringCloud Gateway 是 Spring Cloud 的一个全新项目，该项目是基于 Spring 5.0，Spring Boot 2.0 和 Project Reactor 等技术开发的网关，它旨在为微服务架构提供一种简单有效的统一的 API 路由管理方式。

SpringCloud Gateway 作为 Spring Cloud 生态系统中的网关，目标是替代 Zuul，在Spring Cloud 2.0以上版本中，没有对新版本的Zuul 2.0以上最新高性能版本进行集成，仍然还是使用的Zuul 2.0之前的非Reactor模式的老版本。而为了提升网关的性能，SpringCloud Gateway是基于WebFlux框架实现的，而WebFlux框架底层则使用了高性能的Reactor模式通信框架Netty。

Spring Cloud Gateway 的目标，不仅提供统一的路由方式，并且基于 Filter 链的方式提供了网关基本的功能，例如：安全，监控/指标和限流。

**注意**：Spring Cloud Gateway 底层使用了高性能的通信框架Netty。

官网地址：https://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/

## 特征

SpringCloud官方，对SpringCloud Gateway 特征介绍：

- 基于 Spring Framework 5，Project Reactor 和 Spring Boot 2.0
- 集成 Spring Cloud DiscoveryClient
- Predicates 和 Filters 作用于特定路由，易于编写的Predicates 和 Filters
- 具备一些网关的高级功能：动态路由、限流、路径重写
- 集成Spring Cloud DiscoveryClient
- 集成熔断器CircuitBreaker

从以上的特征来说，和Zuul的特征差别不大。SpringCloud Gateway和Zuul主要的区别，还是在底层的通信框架上：

1. Filter(过滤器)：

   和Zuul的过滤器在概念上类似，可以使用它拦截和修改请求，并且对下游的响应，进行二次处理。过滤器为`org.springframework.cloud.gateway.filter.GatewayFilter`类的实例。

2. Route(路由)：

   网关配置的基本组成模块，和Zuul的路由配置模块类似。一个Route模块由一个 ID，一个目标 URI，一组断言和一组过滤器定义。如果断言为真，则路由匹配，目标URI会被访问。

3. Predicate(断言)：

   这是一个 Java 8 的 Predicate，可以使用它来匹配来自 HTTP 请求的任何内容，例如 headers 或参数。断言的输入类型是一个 ServerWebExchange。

## 部署网关

创建一个新的项目，作为我们的网关，这里需要添加两个依赖：

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-gateway</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
    </dependency>
</dependencies>
```

第一个依赖就是网关的依赖，而第二个则跟其他微服务一样，需要注册到Eureka才能生效，注意别添加Web依赖，使用的是WebFlux框架。

完善配置文件：

```yaml
server:
  port: 8500
eureka:
  client:
    service-url:
     defaultZone: http://eureka:8888/eureka,http://eureka01:8801/eureka,http://eureka02:8802/eureka
spring:
  application:
    name: gateway
```

启动：

![image-20230313190819391](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303281413865.png)

## 处理流程

客户端向 Spring Cloud Gateway 发出请求。然后在 Gateway Handler Mapping 中找到与请求相匹配的路由，将其发送到 Gateway Web Handler。Handler 再通过指定的过滤器链来将请求发送到我们实际的服务执行业务逻辑，然后返回。过滤器之间用虚线分开是因为过滤器可能会在发送代理请求之前（“pre”）或之后（“post”）执行业务逻辑：

![gateway处理流程.jpg](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303281413476.jpeg)

## 路由配置方式

路由是网关配置的基本组成模块，和Zuul的路由配置模块类似。一个**Route模块**由一个 ID，一个目标 URI，一组断言和一组过滤器定义。如果断言为真，则路由匹配，目标URI会被访问。

### 基础路由配置

如果请求的目标地址，是单个的URI资源路径。路由功能进行配置：

```yaml
spring:
  cloud:
    gateway:
      routes:
        # 基础路由配置
        - id: service1
          uri: https://www.oracle.com/
          predicates:
            - Path=/sg
```

配置了一个 id 为 service1的URI代理规则，路由的规则为，当访问地址http://localhost:8500/sg时，会路由到上游地址 https://www.oracle.com/sg

![image-20230314173954192](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202307051853220.png)

### 路由优先级

可以通过配置order指定路由优先级，order越小优先级越高，以下代码执行http://localhost:8500/port，路由到http://localhost:8999/，如果不写order，则会路由到http://localhost:8081/

> UserController.java

```java
@RequestMapping("/port")
public String port(HttpServletRequest request){
       return String.valueOf(request.getRequestURL());
}
```

> gateway：application.yaml

```yaml
spring:
  cloud:
    gateway:
      routes:
        - id: service2
          uri: http://localhost:8081/
          order: 2
          predicates:
            - Path=/port
        - id: service3
          uri: http://localhost:8999/
          order: 1
          predicates:
            - Path=/port
```

![image-20230314184104669](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303281413696.png)

## 注册中心中的路由配置

在URI的schema协议部分为自定义的`lb:`类型，表示从微服务注册中心(如Eureka)订阅服务，并且通过负载均衡进行服务的路由：

```yaml
spring:
  cloud:
    gateway:
    	# 配置路由，注意这里是个列表，每一项都包含了很多信息
      routes:
        - id: borrow-service   # 路由名称
          uri: lb://borrowservice  # 路由的地址，lb表示使用负载均衡到微服务，也可以使用http正常转发
          predicates: # 路由规则，断言什么请求会被路由
            - Path=/borrow/**  # 只要是访问的这个路径，一律都被路由到上面指定的服务
```

注册中心相结合的路由配置方式，与单个URI的路由配置，区别其实很小，仅仅在于URI的schema协议不同。单个URI的地址的schema协议，一般为http或者https协议。启动多个支付微服务，会发现端口9000,8999轮流出现。

接着启动网关，搭载Arm架构芯片的Mac电脑可能会遇到这个问题：

![image-20220325150924472](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303281413920.png)

这是因为没有找到适用于此架构的动态链接库，不影响使用，无视即可。

直接通过路由来访问服务：

![image-20230313190912124](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303281413048.png)

注意：此时依然可以通过原有的服务地址进行访问：

![image-20230313190943561](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303281413382.png)

这样我们就可以将不需要外网直接访问的微服务全部放到内网环境下，而只依靠网关来对外进行交涉。

## 基于代码的路由配置方式

转发功能同样可以通过代码来实现，我们可以在启动类 GateWayApplication 中添加方法 `customRouteLocator()` 来定制转发规则：

```java
@SpringBootApplication
@EnableEurekaClient
public class GatewayApplication {
    public static void main(String[] args) {
        SpringApplication.run(GatewayApplication.class,args);
    }
    // 基于代码的路由配置方式
    @Bean
    public RouteLocator customRouteLocator(RouteLocatorBuilder builder) {
        return builder.routes()
                .route("path_route", r -> r.path("/user/**")
                        .uri("lb://userservice"))
                .build();
    }
}
```

## 路由匹配规则

Spring Cloud Gateway的主要功能之一是转发请求，转发规则的定义主要包含三个部分：

![图片1](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303281413466.png)

 Spring Cloud Gataway 内置了很多 Predicates 功能。可以指定多种类型，包括指定时间段、Cookie携带情况、Header携带情况、访问的域名地址、访问的方法、路径、参数、访问者IP等。

路由规则的详细列表(断言工厂列表)：https://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/#gateway-request-predicates-factories

Spring Cloud Gateway 是通过 Spring WebFlux 的 HandlerMapping 做为底层支持来匹配到转发路由，Spring Cloud Gateway 内置了很多 Predicates 工厂，这些 Predicates 工厂通过不同的 HTTP 请求参数来匹配，多个 Predicates 工厂可以组合使用：

![image.png](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303281413555.png)

### Predicate 断言条件 

Predicate 来源于 Java 8，是 Java 8 中引入的一个函数，Predicate 接受一个输入参数，返回一个布尔值结果。该接口包含多种默认方法来将 Predicate 组合成其他复杂的逻辑（比如：与，或，非）。可以用于接口请求参数校验、判断新老数据是否有变化需要进行更新操作。

在 Spring Cloud Gateway 中 Spring 利用 Predicate 的特性实现了各种路由匹配规则，有通过 Header、请求参数等不同的条件来进行作为条件匹配到对应的路由。

Spring Cloud 内置的几种 Predicate 的实现：

![image.png](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303281413392.png)

Predicate 就是为了实现一组匹配规则，方便让请求过来找到对应的 Route 进行处理。

**常见Predicate**：

![图片2](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303281414885.png)

#### 通过请求参数匹配

Query Route Predicate 支持传入两个参数，一个是属性名一个为属性值，属性值可以是正则表达式：

```yaml
spring:
  cloud:
    gateway:
      routes:
        - id: service3
          uri: https://www.baidu.com
          order: 0
          predicates:
            - Query=smile
```

只要请求中包含 smile 属性的参数即可匹配路由。使用 curl 测试，命令行输入：`curl localhost:8500?smile=x`，经过测试发现只要请求汇总带有 smile 参数即会匹配路由，不带 smile 参数则不会匹配。

还可以将 Query 的值以键值对的方式进行配置，这样在请求过来时会对属性值和正则进行匹配，匹配上才会走路由：

```yaml
spring:
  cloud:
    gateway:
      routes:
        - id: service3
          uri: https://www.baidu.com
          order: 0
          predicates:
            - Query=keep, pu.
```

只要当请求中包含 keep 属性并且参数值是以 pu 开头的长度为三位的字符串才会进行匹配和路由。

使用 curl 测试，命令行输入：`curl localhost:8500?keep=pub`，测试可以返回页面代码，将 keep 的属性值改为 pubx 再次访问就会报 404，证明路由需要匹配正则表达式才会进行路由。

#### 通过Header匹配

Header Route Predicate 和 Query Route Predicate 一样，也是接收 2 个参数，一个 header 中属性名称和一个正则表达式，这个属性值和正则表达式匹配则执行：

```yaml
spring:
  cloud:
    gateway:
      routes:        
        - id: service4
          uri: http://localhost:8083/book/2
          order: 0
          predicates:
            - Header=X-Request-Id, \d+
```

使用 curl 测试，命令行输入：curl http://localhost:8500 -H "X-Request-Id:88"，则返回页面代码证明匹配成功。将参数`-H "X-Request-Id:88"`改为`-H "X-Request-Id:spring"`，再次执行时返回404证明没有匹配。

#### 通过Cookie匹配

Cookie Route Predicate 可以接收两个参数，一个是 Cookie name ,一个是正则表达式，路由规则会通过获取对应的 Cookie name 值和正则表达式去匹配，如果匹配上就会执行路由，如果没有匹配上则不执行：

```yaml
spring:
  cloud:
    gateway:
      routes:        
        - id: service5
          uri: http://localhost:8083/book/2
          predicates:
            - Cookie=sessionId, test
```

使用 curl 测试，命令行输入，curl http://localhost:8500 --cookie "sessionId=test"，则会返回页面代码，如果去掉`--cookie "sessionId=test"`，后台汇报 404 错误。

#### 通过Host匹配

Host Route Predicate 接收一组参数，一组匹配的域名列表，这个模板是一个 ant 分隔的模板，用`.`号作为分隔符。它通过参数中的主机地址作为匹配规则。

```yaml
spring:
  cloud:
    gateway:
      routes:    
        - id: service6
          uri: https://www.baidu.com
          predicates:
            - Host=**.baidu.com      
```

使用 curl 测试，命令行输入，curl http://localhost:8500 -H "Host: www.baidu.com"或者curl http://localhost:8080 -H "Host: md.baidu.com"，经测试以上两种 host 均可匹配到 host_route 路由，去掉 host 参数则会报 404 错误。

#### 通过请求方式匹配

通过 POST、GET、PUT、DELETE 等不同的请求方式来进行路由：

```yaml
spring:
  cloud:
    gateway:
      routes:    
        - id: service7
          uri: https://www.baidu.com
          predicates:
            - Method=PUT
```

使用 curl 测试，命令行输入，curl -X PUT http://localhost:8500，测试返回页面代码，证明匹配到路由，以其他方式，返回 404 没有找到，证明没有匹配上路由

#### 通过请求路径匹配

Path RoutePredicate 接收一个匹配路径的参数来判断是否路由：

```yaml
spring:
  cloud:
    gateway:
      routes:    
        - id: service8
          uri: https://www.baidu.com
          predicates:
            - Path=/baidu/{segment}
```

如果请求路径符合要求，则此路由将匹配， curl 测试，命令行输入，curl http://localhost:8500/baidu/1，

可以正常获取到页面返回值，curl http://localhost:8500/baidu/1/2，报404，证明路由是通过指定路由来匹配。

#### 组合匹配

```yaml
spring:
  cloud:
    gateway:
      routes:    
        - id: service9
          uri: https://www.baidu.com
          order: 0
          predicates:
            - Host=**.foo.org
            - Path=/headers
            - Method=GET
            - Header=X-Request-Id, \d+
            - Query=foo, ba.
            - Query=baz
            - Cookie=chocolate, ch.p
```

各种 Predicates 同时存在于同一个路由时，请求必须同时满足所有的条件才被这个路由匹配。

一个请求满足多个路由的断言条件时，请求只会被首个成功匹配的路由转发。

### 过滤器规则

路由过滤器支持以某种方式修改传入的 HTTP 请求或传出的 HTTP 响应，路由过滤器的范围是某一个路由，跟之前的断言一样，Spring Cloud Gateway 也包含许多内置的路由过滤器工厂。

详细列表：https://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/#gatewayfilter-factories

常见过滤器：

![图片1](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303281414760.png)

比如在请求到达时，在请求头中添加一些信息再转发给我们的服务，那么这个时候就可以使用路由过滤器来完成，只需要对配置文件进行修改：

```yaml
spring:
  application:
    name: gateway
  cloud:
    gateway:
      routes:
      - id: borrow-service
        uri: lb://borrowservice
        predicates:
        - Path=/borrow/**
      # 继续添加新的路由配置，这里就以书籍管理服务为例
      # 注意-要对齐routes:
      - id: book-service
        uri: lb://bookservice
        predicates:
        - Path=/book/**
        filters:   # 添加过滤器
        - AddRequestHeader=Test, HelloWorld!
        # AddRequestHeader 就是添加请求头信息，其他工厂请查阅官网
```

在BookController中获取并输出：

```java
@RestController
@EnableDiscoveryClient
public class BookController {

    @Resource
    BookService service;

    @RequestMapping("/book/{bid}")
    Book findBookById(@PathVariable("bid") int bid,
                      HttpServletRequest request){
        System.out.println(request.getHeader("Test"));
        return service.getBookById(bid);
    }
}
```

现在我们通过Gateway访问我们的图书管理服务：

![image-20230313191704905](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303281414522.png)

![image-20230313191744337](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303281414088.png)

成功获取到由网关添加的请求头信息。

#### PrefixPath

对所有的请求路径添加前缀：

```yaml
spring:
  cloud:
    gateway:
      routes:    
        - id: service10
          uri: http://127.0.0.1:8082
          predicates:
            - Path=/{segment}
          filters:
            - PrefixPath=/borrow
```

#### StripPrefix

跳过指定的路径：

```yaml
spring:
  cloud:
    gateway:
      routes:    
        - id: service11
          uri: http://127.0.0.1:8082
          predicates:
            - Path=/api/{segment}
          filters:
            - StripPrefix=1
            - PrefixPath=/borrow
```

此时访问http://localhost:9005/api/1，首先StripPrefix过滤器去掉一个/api，然后PrefixPath过滤器加上一个/borrow，能够正确访问到借阅微服务。

#### RewritePath

```yaml
spring:
  cloud:
    gateway:
      routes:        
        - id: service12
          uri: http://127.0.0.1:8082
          predicates:
            - Path=/api/payment/**
          filters:
            - RewritePath=/api/(?<segment>.*), /$\{segment}
```

请求http://localhost:8500/api/borrow/1路径，RewritePath过滤器将路径重写为http://localhost:8500/borrow/1，能够正确访问支付微服务。

#### SetPath

SetPath和Rewrite类似：

```yaml
spring:
  cloud:
    gateway:
      routes:           
        - id: service13
          uri: http://127.0.0.1:8082
          predicates:
            - Path=/api/borrow/{segment}
          filters:
            - SetPath=/borrow/{segment}
```

请求http://localhost:8500/api/borrow/1路径，SetPath过滤器将路径设置为http://localhost:8500/borrow/1，能够正确访问借阅微服务。

#### AddRequestHeader

添加请求头信息：

```yaml
spring:
  cloud:
    gateway:
      routes:           
        - id: service13
          uri: http://127.0.0.1:8082
          predicates:
            - Path=/borrow/{segment}
          filters:
            - AddRequestHeader=Test,HelloWorld!
```

#### RemoveRequestHeader

去掉请求头信息：

```yaml
spring:
  cloud:
    gateway:
      routes:           
        - id: service13
          uri: http://127.0.0.1:8082
          predicates:
            - Path=/borrow/{segment}
          filters:
            - RemoveRequestHeader=Test
```

#### RemoveResponseHeader

去掉回执头信息：

```yaml
spring:
  cloud:
    gateway:
      routes:           
        - id: service13
          uri: http://127.0.0.1:8082
          predicates:
            - Path=/borrow/{segment}
          filters:
            - RemoveResponseHeader=Test
```

#### RemoveRequestParameter

去掉请求参数信息：

```yaml
spring:
  cloud:
    gateway:
      routes:           
        - id: service13
          uri: http://127.0.0.1:8082
          predicates:
            - Path=/borrow/{segment}
          filters:
          - RemoveRequestParameter=a
```

#### SetRequestHeader

设置请求头信息：

```yaml
spring:
  cloud:
    gateway:
      routes:           
        - id: service13
          uri: http://127.0.0.1:8082
          predicates:
            - Path=/borrow/{segment}
          filters:
          - SetRequestHeader=X-Request-Red, Blue
```

#### default-filters

对所有的请求添加过滤器：

```yaml
spring:
  cloud:
    gateway:
      routes:        
        - id: service14
          uri: http://127.0.0.1:8999
          predicates:
            - Path=/8999/{segment}
        - id: service15
          uri: http://127.0.0.1:9000
          predicates:
            - Path=/9000/{segment}
      default-filters:
      	- StripPrefix=1
        - PrefixPath=/user
```

## 自定义过滤器

### 过滤器执行次序

Spring-Cloud-Gateway 基于过滤器实现，同 zuul 类似，有pre和post两种方式的 filter，分别处理前置逻辑和后置逻辑。客户端的请求先经过pre类型的 filter，然后将请求转发到具体的业务服务，收到业务服务的响应之后，再经过post类型的 filter 处理，最后返回响应到客户端。

**过滤器执行流程**：

![image.png](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303281414860.png)

**注意**：Order的值越小优先级越高，并且无论是在配置文件中编写的单个路由过滤器还是全局路由过滤器，都会受到Order值影响（单个路由的过滤器Order值按从上往下的顺序从1开始递增），最终是按照Order值决定哪个过滤器优先执行，当Order值一样时 全局路由过滤器执行 **优于** 单独的路由过滤器执行。

过滤器分为全局过滤器和局部过滤器：

- 全局过滤器：对所有路由生效。
- 局部过滤器：对指定的路由生效。

### 全局过滤器

自定义全局过滤器能够作用于全局。但是需要通过代码的方式进行编写，比如要实现拦截没有携带指定请求参数的请求：

```java
@Component   //需要注册为Bean
public class TestFilter implements GlobalFilter {
    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {   //只需要实现此方法
        return null;
    }
}
```

编写判断：

```java
@Override
public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
    //先获取ServerHttpRequest对象，注意不是HttpServletRequest
    ServerHttpRequest request = exchange.getRequest();
    //打印一下所有的请求参数
    System.out.println(request.getQueryParams());
    //判断是否包含test参数，且参数值为1
    List<String> value = request.getQueryParams().get("test");
    if(value != null && value.contains("1")) {
        //将ServerWebExchange向过滤链的下一级传递（跟JavaWeb中介绍的过滤器其实是差不多的）
        return chain.filter(exchange);
    }else {
        //直接在这里不再向下传递，然后返回响应
        return exchange.getResponse().setComplete();
    }
}
```

可以看到结果：

![image-20230313192529567](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303281414922.png)

成功实现规则判断和拦截操作。

当然，过滤器肯定是可以存在很多个的，所以我们可以手动指定过滤器之间的顺序：

```java
@Component
public class TestFilter implements GlobalFilter, Ordered {   //实现Ordered接口
  
    @Override
    public int getOrder() {
        return 0;
    }
}
```

定义了4个全局过滤器，顺序为A>B>C>MyAuthFilter，其中全局过滤器MyAuthFilter中判断令牌是否存在，如果令牌不存在，则返回401状态码，表示没有权限访问：

```java
@Configuration
public class FilterConfig
{

    @Bean
    public GlobalFilter a()
    {
        return new AFilter();
    }

    @Bean
    public GlobalFilter b()
    {
        return new BFilter();
    }

    @Bean
    public GlobalFilter c()
    {
        return new CFilter();
    }

    @Bean
    public GlobalFilter myAuthFilter()
    {
        return new MyAuthFilter();
    }


    @Slf4j
    static class AFilter implements GlobalFilter, Ordered
    {

        @Override
        public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain)
        {
            log.info("AFilter前置逻辑");
            return chain.filter(exchange).then(Mono.fromRunnable(() ->
            {
                log.info("AFilter后置逻辑");
            }));
        }

        //   值越小，优先级越高
        @Override
        public int getOrder()
        {
            return HIGHEST_PRECEDENCE + 100;
        }
    }

    @Slf4j
    static class BFilter implements GlobalFilter, Ordered
    {
        @Override
        public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain)
        {
            log.info("BFilter前置逻辑");
            return chain.filter(exchange).then(Mono.fromRunnable(() ->
            {
                log.info("BFilter后置逻辑");
            }));
        }

        @Override
        public int getOrder()
        {
            return HIGHEST_PRECEDENCE + 200;
        }
    }

    @Slf4j
    static class CFilter implements GlobalFilter, Ordered
    {

        @Override
        public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain)
        {
            log.info("CFilter前置逻辑");
            return chain.filter(exchange).then(Mono.fromRunnable(() ->
            {
                log.info("CFilter后置逻辑");
            }));
        }

        @Override
        public int getOrder()
        {
            return HIGHEST_PRECEDENCE + 300;
        }
    }


    @Slf4j
    static class MyAuthFilter implements GlobalFilter, Ordered {

        @Override
        public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
            log.info("MyAuthFilter权限过滤器");
            String token = exchange.getRequest().getHeaders().getFirst("token");
            if (StringUtils.isBlank(token)) {
                exchange.getResponse().setStatusCode(HttpStatus.UNAUTHORIZED);
                return exchange.getResponse().setComplete();
            }

            return chain.filter(exchange);
        }

        @Override
        public int getOrder() {
            return HIGHEST_PRECEDENCE + 400;
        }
    }


}
```

### 局部过滤器

定义局部过滤器步骤：

1. 需要实现GatewayFilter, Ordered，实现相关的方法
2. 包装GatewayFilter，产生GatewayFilterFactory
3. GatewayFilterFactory加入到过滤器工厂，并且注册到spring容器中。
4. 在配置文件中进行配置，如果不配置则不启用此过滤器规则。

定义局部过滤器，对于请求头user-id校验，如果不存在user-id请求头，直接返回状态码406：

```java
@Component
public class UserIdCheckGatewayFilterFactory extends AbstractGatewayFilterFactory<Object>
{
    @Override
    public GatewayFilter apply(Object config)
    {
        return new UserIdCheckGateWayFilter();
    }

    @Slf4j
    static class UserIdCheckGateWayFilter implements GatewayFilter, Ordered
    {
        @Override
        public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain)
        {
            String url = exchange.getRequest().getPath().pathWithinApplication().value();
            log.info("请求URL:" + url);
            log.info("method:" + exchange.getRequest().getMethod());
            //获取header
            String userId = exchange.getRequest().getHeaders().getFirst("user-id");
            log.info("userId：" + userId);

            if (StringUtils.isBlank(userId))
            {
                log.info("*****头部验证不通过，请在头部输入  user-id");
                //终止请求，直接回应
                exchange.getResponse().setStatusCode(HttpStatus.NOT_ACCEPTABLE);
                return exchange.getResponse().setComplete();
            }
            return chain.filter(exchange);
        }

        //   值越小，优先级越高
        @Override
        public int getOrder()
        {
            return HIGHEST_PRECEDENCE;
        }
    }
}
```

> application.yml

```yaml
spring:
  cloud:
    gateway:
      routes:        
        - id: service14
          uri: http://127.0.0.1:8083
          predicates:
            - Path=/{segment}
      default-filters:
        - PrefixPath=/borrow
        - UserIdCheck
```

## 高级特性 

### 熔断降级 

在分布式系统中，网关作为流量的入口，因此会有大量的请求进入网关，向其他服务发起调用，其他服务不可避免的会出现调用失败（超时、异常），失败时不能让请求堆积在网关上，需要快速失败并返回给客户端，想要实现这个要求，就必须在网关上做熔断、降级操作。

当一个客户端请求发生故障的时候，这个请求会一直堆积在网关上，当然只有一个这种请求，网关肯定没有问题（如果一个请求就能造成整个系统瘫痪，那这个系统可以下架了），但是网关上堆积多了就会给网关乃至整个服务都造成巨大的压力，甚至整个服务宕掉。因此要对一些服务和页面进行有策略的降级，以此缓解服务器资源的的压力，以保证核心业务的正常运行，同时也保持了客户和大部分客户的得到正确的相应，所以需要网关上请求失败需要快速返回给客户端。

CircuitBreaker过滤器使用Spring Cloud CircuitBreaker API 将网关路由包装在断路器中。Spring Cloud CircuitBreaker 支持多个可与 Spring Cloud Gateway 一起使用熔断器库。比如，Spring Cloud 支持开箱即用的 Resilience4J。

启用Spring Cloud CircuitBreaker过滤器步骤：

**添加依赖**：

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-circuitbreaker-reactor-resilience4j</artifactId>
</dependency>
```

**配置文件**：

```yaml
server:
  port: 9005
spring:
  application:
    name: gateway
  cloud:
    gateway:
      routes:
        - id: borrow
          uri: lb://borrowservice
          predicates:
            - Path=/b/{segment}
          filters:
            - SetPath=/borrow/{segment}
            - name: CircuitBreaker
              args:
                 name: backendA
                 fallbackUri: forward:/fallbackA

eureka:
  client:
    service-url:
      defaultZone: http://eureka:8888/eureka,http://eureka01:8801/eureka,http://eureka02:8802/eureka

resilience4j:
  circuitbreaker:
    configs:
      default:
        failureRateThreshold: 30 #失败请求百分比，超过这个比例，CircuitBreaker变为OPEN状态
        slidingWindowSize: 10 #滑动窗口的大小，配置COUNT_BASED,表示10个请求，配置TIME_BASED表示10秒
        minimumNumberOfCalls: 5 #最小请求个数，只有在滑动窗口内，请求个数达到这个个数，才会触发CircuitBreader对于断路器的判断
        slidingWindowType: TIME_BASED #滑动窗口的类型
        permittedNumberOfCallsInHalfOpenState: 3 #当CircuitBreaker处于HALF_OPEN状态的时候，允许通过的请求个数
        automaticTransitionFromOpenToHalfOpenEnabled: true #设置true，表示自动从OPEN变成HALF_OPEN，即使没有请求过来
        waitDurationInOpenState: 2s #从OPEN到HALF_OPEN状态需要等待的时间
        recordExceptions: #异常名单
          - java.lang.Exception
    instances:
      backendA:
        baseConfig: default
      backendB:
        failureRateThreshold: 50
        slowCallDurationThreshold: 2s #慢调用时间阈值，高于这个阈值的呼叫视为慢调用，并增加慢调用比例。
        slowCallRateThreshold: 30 #慢调用百分比阈值，断路器把调用时间大于slowCallDurationThreshold，视为慢调用，当慢调用比例大于阈值，断路器打开，并进行服务降级
        slidingWindowSize: 10
        slidingWindowType: TIME_BASED
        minimumNumberOfCalls: 2
        permittedNumberOfCallsInHalfOpenState: 2
        waitDurationInOpenState: 120s #从OPEN到HALF_OPEN状态需要等待的时间
```

**全局过滤器**：创建一个全局过滤器，打印熔断器状态

```java
@Component
@Slf4j
public class CircuitBreakerLogFilter implements GlobalFilter, Ordered {

    @Autowired
    private CircuitBreakerRegistry circuitBreakerRegistry;

    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        String url = exchange.getRequest().getPath().pathWithinApplication().value();
        log.info("url : {} status : {}", url, circuitBreakerRegistry.circuitBreaker("backendA").getState().toString());
        return chain.filter(exchange);
    }

    @Override
    public int getOrder() {
        return HIGHEST_PRECEDENCE;
    }
}
```

**降级方法**：

```java
@RestController
@Slf4j
public class FallbackController {

    @GetMapping("/fallbackA")
    public ResponseEntity fallbackA() {
        return ResponseEntity.ok("服务不可用，降级");
    }
}
```

使用JMeter工具同时1秒内并发发送20次请求，触发断路器backendA熔断条件，断路器打开，之后2秒后熔断器自动进入进入半开状态（`automaticTransitionFromOpenToHalfOpenEnabled: true，waitDurationInOpenState: 2s`）：

![image.png](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303281415016.png)

接下来2秒后再次请求熔断器处于半开状态，接下来启动借阅微服务，服务能够正常访问熔断器关闭。

### 统一跨域请求 

**跨域请求**：当前发起请求的域与该请求指向的资源所在的域不一样。

**域**：若协议 + 域名 + 端口号均相同，那么就是同域。

举个例子：假如一个域名为aaa.cn的网站，它发起一个资源路径为aaa.cn/books/getBookInfo的 Ajax 请求，那么这个请求是同域的，因为资源路径的协议、域名以及端口号与当前域一致（例子中协议名默认为http，端口号默认为80）。但是，如果发起一个资源路径为bbb.com/pay/purchase的 Ajax 请求，那么这个请求就是跨域请求，因为域不一致，与此同时由于安全问题，这种请求会受到同源策略限制。

**创建index.html，发送ajax测试跨域**：

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Title</title>
  <script src="https://code.jquery.com/jquery-3.0.0.min.js"></script>
  <script>
    function sendAjax() {
      $.ajax({
        method: 'GET',
        url: "http://127.0.0.1:8082/borrow/1",
        contentType: 'application/json; charset=UTF-8',
        success: function() {
          alert("成功");
        }
      });
    }
  </script>
</head>
<body>
<button onclick="sendAjax();" >send ajax</button>
</body>
</html>
```

当访问http://localhost:8082/会产生异常：

![image-20230315184458311](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303281415487.png)

访问http://127.0.0.1:8082/则会正常访问：

![image-20230315184738668](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303281415759.png)

因为浏览器同源策略，就会出现跨域访问问题。

虽然在安全层面上同源限制是必要的，但有时同源策略会对我们的合理用途造成影响，为了避免开发的应用受到限制，有多种方式可以绕开同源策略，常用的做法JSONP, CORS。可以使用`@CrossOrigin`：

> BorrowController

```java
@RestController
@CrossOrigin
public class BorrowController {
   ...
}
```

现在请求经过gatway网关，可以通过网关统一配置跨域访问。

**跨域配置**：

可以在配置文件中设置CORS：

将html中的url修改为：http://localhost:8500/borrow/1，并去掉`@CrossOrigin`注解：

![image-20230315185712598](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303281415449.png)

> gateway：application.yml

```yaml
spring:
  cloud:
    gateway:
      globalcors:
        cors-configurations:
          '[/**]':
            max-age: 3600
            allowed-origin-patterns: "*" # spring boot2.4配置跨域访问
#            allowed-origins: "*" 
            allowed-headers: "*"
            allow-credentials: true
            allowed-methods:
              - GET
              - POST
              - DELETE
              - PUT
              - OPTION
```

