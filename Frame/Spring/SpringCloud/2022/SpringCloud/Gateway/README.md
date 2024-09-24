# Gateway 路由网关

一般情况下，可能并不是所有的微服务都需要直接暴露给外部调用，这时可以使用路由机制，添加一层防护，让所有的请求全部通过路由来转发到各个微服务，并且转发给多个相同微服务实例也可以实现负载均衡。

![image-20240119001548961](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202401190015434.png)

SpringCloud Gateway 是基于 Spring，Spring Boot和 Project Reactor 等技术开发的网关，它旨在为微服务架构提供一种简单有效的统一的 API 路由管理方式，并为它们提供跨领域的关注点，例如：安全性、监控/度量和弹性。

SpringCloud Gateway 作为 Spring Cloud 生态系统中的网关，目标是替代 Zuul，在Spring Cloud 2.0以上版本中，没有对新版本的Zuul 2.0以上最新高性能版本进行集成，仍然还是使用的Zuul 2.0之前的非Reactor模式的老版本。

为了提升网关的性能，SpringCloud Gateway是基于WebFlux框架实现的，而WebFlux框架底层则使用了高性能的Reactor模式通信框架Netty。所以说，**Spring Cloud Gateway 底层使用了高性能的通信框架Netty**。

Spring Cloud Gateway 的目标，不仅提供统一的路由方式，并且基于 Filter 链的方式提供了网关基本的功能，例如：鉴权、路由、安全、监控/指标和限流。

官网地址：https://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/

## 特征

SpringCloud官方，对SpringCloud Gateway 特征：

- 基于 Spring Framework 6，Project Reactor 和 Spring Boot 3
- 集成 Spring Cloud DiscoveryClient
- Predicates 和 Filters 作用于特定路由，易于编写的Predicates 和 Filters
- 具备一些网关的高级功能：动态路由、限流、路径重写
- 集成Spring Cloud DiscoveryClient
- 集成熔断器Circuit Breaker

从以上的特征来说，和Zuul的特征差别不大。SpringCloud Gateway和Zuul主要的区别，是在底层的通信框架上。

## 基本概念

1. **Route(路由)**：

   网关配置的基本组成模块，和Zuul的路由配置模块类似。一个Route模块由一个 ID，一个目标地址 URI，一组断言和一组过滤器定义。如果断言为真，则路由匹配，则请求将经由 filter 被路由到目标 URL。

2. **Filter(过滤器)**：

   对请求进行处理的逻辑部分，可以使用它拦截和修改请求，并且对下游的响应，进行二次处理。过滤器为`org.springframework.cloud.gateway.filter.GatewayFilter`类的实例。当请求的断言为 true 时，会被路由到设置好的过滤器， 以对请求或响应进行处理。例如，可以为请求添加一个请求参数，或对请求 URI 进行修改， 或为响应添加 header 等。总之，就是对请求或响应进行处理。

3. **Predicate(断言)**：

   断言即一个条件判断，根据当前的 HTTP 请求进行指定规则的匹配，比如说 http 请求头、请求时间、参数等。断言的输入类型是一个 ServerWebExchange。只有当匹配上规则时，断言才为 true，此时请求才会被直接路由到目标地址 (目标服务器)，或先路由到某过滤器链，经过过滤器链的层层处理后，再路由到相应的目标地址（目标服务器）

## 处理流程

客户端向 Spring Cloud Gateway 发出请求。然后在 Gateway Handler Mapping 中找到与请求相匹配的路由，将其发送到 Gateway Web Handler。Handler 再通过指定的过滤器链来将请求发送到我们实际的服务执行业务逻辑，然后返回。过滤器之间用虚线分开是因为过滤器可能会在发送代理请求之前（“pre”）或之后（“post”）执行业务逻辑：



![Spring Cloud Gateway Diagram](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202311212102835.png)

## 部署网关

创建一个新的项目，作为网关，添加依赖：

```xml
<dependency>
	<groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-gateway</artifactId>
</dependency>
```

**注意**：不需要添加Web依赖，Spring Cloud Gateway使用的是WebFlux框架。

## 路由配置

路由是网关配置的基本组成模块，和Zuul的路由配置模块类似。一个**Route模块**由一个 ID，一个目标 URI，一组断言和一组过滤器定义。如果断言为真，则路由匹配，目标URI会被访问。

### 基础路由配置

#### 配置式路由配置

如果请求的目标地址，是单个的URI资源路径。路由功能进行配置：

```yaml
server:
  port: 9000
spring:
  application:
    name: gateway
  cloud:
    gateway:
    # 配置路由，注意这里是个列表，每一项都包含了很多信息
      routes:
        # 基础路由配置
        - id: baidu_route # 路由名称
          uri: https://www.baidu.com/
          predicates: # 路由规则，断言什么请求会被路由 
            - Path=/** # 只要是访问的这个路径，一律都被路由到上面指定的服务
```

配置了一个 id 为 baidu_route 的URI代理规则，路由的规则为，当访问地址 http://localhost:9000/ 时，会路由到上游地址 百度主页。

![image-20231121223714205](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202311212237066.png)

#### API式路由配置

转发功能同样可以通过代码来实现，可以在启动类 GateWayApplication 中添加方法 `customRouteLocator()` 来定制转发规则：

```java
@SpringBootApplication
public class GatewayApplication {
    public static void main(String[] args) {
        SpringApplication.run(GatewayApplication.class,args);
    }
    // 基于代码的路由配置方式
    @Bean
    public RouteLocator customRouteLocator(RouteLocatorBuilder builder) {
        return builder.routes()
                .route("baidu_route", r -> r.path("/**")
                        .uri("https://www.baidu.com/"))
                .build();
    }
}
```

![image-20231121225516362](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202311212255867.png)

### 路由优先级

可以通过配置order指定路由优先级，order越小优先级越高，以下代码执行 http://localhost:8500/port ，路由到 http://localhost:8999/，如果不写order，则会路由到 http://localhost:8081/

创建一个应用并在Controller中添加：

```java
@RequestMapping("/port")
public String port(HttpServletRequest request){
       return String.valueOf(request.getRequestURL());
}
```

在Gateway配置文件中修改代码。

**配置式路由配置**：

```yaml
spring:
  cloud:
    gateway:
      routes:
        - id: user_route1
          uri:  http://localhost:8101/
          predicates:
            - Path=/port
          order: 2
        - id: user_route2
          uri: http://localhost:8102/
          predicates:
            - Path=/port
          order: 1
```

**API式路由配置**：

```java
@SpringBootApplication
public class GatewayApplication {
    public static void main(String[] args) {
        SpringApplication.run(GatewayApplication.class,args);
    }
    // 基于代码的路由配置方式
    @Bean
    public RouteLocator customRouteLocator(RouteLocatorBuilder builder) {
        return builder.routes()
                .route("user_route1", r -> r.order(2).path("/**")
                        .uri("http://localhost:8101/"))
                .route("user_route2", r -> r.order(1).path("/**")
                        .uri("http://localhost:8102/"))
                .build();
    }
}
```

访问，查看效果：

![image-20231124210944849](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202401222001356.png)

### 注册中心路由配置

在URI的schema协议部分为自定义的`lb:`类型，表示从微服务注册中心订阅服务，并且通过负载均衡进行服务的路由。

**注意**：需要注册并配置[注册中心](../../Expand/Registry/README.md)。

**添加loadbalancer依赖**：

```xml
<dependency>
	<groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-loadbalancer</artifactId>
</dependency>
```

**配置式路由配置**：

```yaml
spring:
  cloud:
    gateway:
      # 允许gateway到注册中心定位服务
      discovery:
        locator:
          enabled: true
      # 找不到指定服务时，使状态码由503变为404
      loadbalancer:
        use404: true
      routes:
        - id: borrow-service   # 路由名称
          uri: lb://borrow-service  # 路由的地址，lb表示使用负载均衡到微服务，也可以使用http正常转发
          predicates: # 路由规则，断言什么请求会被路由
            - Path=/borrow/**  # 只要是访问的这个路径，一律都被路由到上面指定的服务
```

**API式路由配置**：

```yaml
spring:
  cloud:
    gateway:
      # 允许gateway到注册中心定位服务
      discovery:
        locator:
          enabled: true
      # 找不到指定服务时，使状态码由503变为404
      loadbalancer:
        use404: true
```

```java
@SpringBootApplication
public class GatewayApplication {
    public static void main(String[] args) {
        SpringApplication.run(GatewayApplication.class,args);
    }
    // 基于代码的路由配置方式
    @Bean
    public RouteLocator customRouteLocator(RouteLocatorBuilder builder) {
        return builder.routes()
                .route("borrow-service", r -> r.path("/borrow/**")
                        .uri("lb://borrow-service"))
                .build();
    }
}
```

注册中心相结合的路由配置方式，与单个URI的路由配置，区别其实很小，仅仅在于URI的schema协议不同。单个URI的地址的schema协议，一般为http或者https协议。

**启动多个支付微服务，会发现端口轮流出现**。

接着启动网关，搭载Arm架构芯片的Mac电脑可能会遇到这个问题：

![image-20220325150924472](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303281413920.png)

这是因为没有找到适用于此架构的动态链接库，不影响使用，无视即可。

直接通过路由来访问服务：

![image-20231202181106275](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202312021811922.png)

此时依然可以通过原有的服务地址进行访问：

![image-20231202181040642](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202312021810638.png)

这样就可以将不需要外网直接访问的微服务全部放到内网环境下，而只依靠网关来对外进行交涉。

## 路由匹配规则

Spring Cloud Gateway的主要功能之一是**转发请求**，转发规则的定义主要包含三个部分：

![图片1](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303281413466.png)

### 路由断言工厂

**拓展**：断言(`Predicate` )来源于 Java 8，是 Java 8 中引入的一个函数，Predicate 接受一个输入参数，返回一个布尔值结果。该接口包含多种默认方法来将 Predicate 组合成其他复杂的逻辑（比如：与，或，非）。可以用于接口请求参数校验、判断新老数据是否有变化需要进行更新操作。

****

Spring Cloud Gataway 内置了很多断言功能。可以指定多种类型，包括指定时间段、Cookie携带情况、Header携带情况、访问的域名地址、访问的方法、路径、参数、访问者IP等。

Predicate 就是为了实现一组匹配规则，方便让请求过来找到对应的 Route 进行处理。

路由规则的详细列表 (断言工厂列表)：https://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/#gateway-request-predicates-factories

Spring Cloud Gateway 是通过 Spring WebFlux 的 HandlerMapping 做为底层支持来匹配到转发路由，Spring Cloud Gateway 内置了很多 Predicates 工厂，这些 Predicates 工厂通过不同的 HTTP 请求参数来匹配，多个 Predicates 工厂可以组合使用：

![image.png](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303281413555.png)

RoutePredicateFactory类：

![image.png](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303281413392.png)

Spring Cloud 内置Predicate实现：

![图片2](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303281414885.png)

#### 通过After匹配

该断言工厂(`After RoutePredicate`)通过接收一个 UTC 格式的时间参数，将请求访问到 Gateway 的时间与该参数时间相比，若请求时间在参数时间之后，则匹配成功会执行路由(断言为 true)，如果没有匹配上则不执行。

**配置式路由配置**：

```yaml
spring:
  cloud:
    gateway:
      routes:
        #TODO 路由断言工厂:通过时间匹配
        - id: baidu_route
          uri: https://www.baidu.com/
          predicates:
            - After=2020-04-20T15:42:47.789-07:00
```

**API式路由配置**：

```java
@SpringBootApplication
public class GatewayApplication {
    public static void main(String[] args) {
        SpringApplication.run(GatewayApplication.class,args);
    }
    // 基于代码的路由配置方式
    @Bean
    public RouteLocator customRouteLocator(RouteLocatorBuilder builder) {
        ZonedDateTime dateTime = LocalDateTime.now().minusDays(5)   //当前时间减5天
                .atZone(ZoneId.systemDefault());    //将系统的默认时区设置为当前时区
        return builder.routes()
                .route("baidu_route", r -> r.after(dateTime)
                        .uri("https://www.baidu.com/"))
                .build();
    }
}
```

**运行结果**：

![image-20231203164244533](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202312031642506.png)

#### 通过Before匹配

该断言工厂(`Before RoutePredicate` )通过接收一个 UTC 格式的时间参数，其会将请求访问到 Gateway 的时间与该参数时间相比，若请求时间在参数时间之前，则匹配成功会执行路由(断言为 true)，如果没有匹配上则不执行。

**配置式路由配置**：

```yaml
spring:
  cloud:
    gateway:
      routes:
        #TODO 路由断言工厂:通过时间匹配
        - id: baidu_route
          uri: https://www.baidu.com/
          predicates:
            - Before=2025-04-20T15:42:47.789-07:00
```

**API式路由配置**：

```java
@SpringBootApplication
public class GatewayApplication {
    public static void main(String[] args) {
        SpringApplication.run(GatewayApplication.class,args);
    }
    // 基于代码的路由配置方式
    @Bean
    public RouteLocator customRouteLocator(RouteLocatorBuilder builder) {
        ZonedDateTime dateTime = LocalDateTime.now().plusDays(5)   //当前时间加5天
                .atZone(ZoneId.systemDefault());    //将系统的默认时区设置为当前时区
        return builder.routes()
                .route("baidu_route", r -> r.before(dateTime)
                        .uri("https://www.baidu.com/"))
                .build();
    }
}
```

**运行结果**：

![image-20231203164244533](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202312031642506.png)

#### 通过Between匹配

该断言工厂(`Between RoutePredicate` )通过接收两个 UTC 格式的时间参数，将请求访问到 Gateway 的时间与这两个参数时间相比，若请求时间在这两个参数时间之间，则匹配成功会执行路由(断言为 true)，如果没有匹配上则不执行。

**配置式路由配置**：

```yaml
spring:
  cloud:
    gateway:
      routes:
        #TODO 路由断言工厂:通过时间匹配
        - id: baidu_route
          uri: https://www.baidu.com/
          predicates:
            - Between=2020-04-20T15:42:47.789-07:00,2025-04-20T15:42:47.789-07:00
```

**API式路由配置**：

```java
@SpringBootApplication
public class GatewayApplication {
    public static void main(String[] args) {
        SpringApplication.run(GatewayApplication.class,args);
    }
    // 基于代码的路由配置方式
    @Bean
    public RouteLocator customRouteLocator(RouteLocatorBuilder builder) {
        ZonedDateTime plusTime = LocalDateTime.now().plusDays(5)   //当前时间加5天
                .atZone(ZoneId.systemDefault());    //将系统的默认时区设置为当前时区
        ZonedDateTime minusTime = LocalDateTime.now().minusDays(5)   //当前时间减5天
                .atZone(ZoneId.systemDefault());    //将系统的默认时区设置为当前时区
        return builder.routes()
                .route("baidu_route", r -> r.between(minusTime,plusTime)
                        .uri("https://www.baidu.com/"))
                .build();
    }
}
```

**运行结果**：

![image-20231203164244533](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202312031642506.png)

#### 通过Header匹配

该断言工厂(`Header RoutePredicate` )通过接收两个参数，分别是请求头 header 的 key(属性名称) 与 value(正则表达式)。当请求中携带了指定 key 与 value 的 header 时，匹配成功会执行路由(断言为 true)，如果没有匹配上则不执行。

**配置式路由配置**：

```yaml
spring:
  cloud:
    gateway:
      routes:
        #TODO 路由断言工厂:通过请求头匹配
        - id: baidu_route
          uri: https://www.baidu.com/
          predicates:
            - Header=X-Request-Id, \d+ #匹配数字类型
```

**API式路由配置**：

```java
@SpringBootApplication
public class GatewayApplication {
    public static void main(String[] args) {
        SpringApplication.run(GatewayApplication.class,args);
    }
    // 基于代码的路由配置方式
    @Bean
    public RouteLocator customRouteLocator(RouteLocatorBuilder builder) {
        return builder.routes()
                .route("baidu_route", r -> r.header("X-Request-Id","\\d+")
                        .uri("https://www.baidu.com/"))
                .build();
    }
}
```

使用接口测试工具添加请求头 `X-Request-Id`：

![image-20231206211526771](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202312062116801.png)

当参数设置为字符串时，则会报错：

![image-20231206211935569](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202312062119154.png)

#### 通过Cookie匹配

该断言工厂(`Cookie RoutePredicate`)通过接收两个参数，分别是 cookie 的 key(Cookie name) 与 value(正则表达式)。当请求中携带了指定 key 与 value 的 cookie 时，匹配成功会执行路由(断言为 true)，如果没有匹配上则不执行。

·**配置式路由配置**：

```yaml
spring:
  cloud:
    gateway:
      routes:
        #TODO 路由断言工厂:通过Cookie匹配
        - id: baidu_route
          uri: https://www.baidu.com/
          predicates:
            - Cookie=sessionId, test
```

**API式路由配置**：

```java
@SpringBootApplication
public class GatewayApplication {
    public static void main(String[] args) {
        SpringApplication.run(GatewayApplication.class,args);
    }
    //TODO 路由断言工厂:通过Cookie匹配
    @Bean
    public RouteLocator customRouteLocator(RouteLocatorBuilder builder) {
        return builder.routes()
                .route("baidu_route", r -> r.cookie("sessionId", "test")
                        .uri("https://www.baidu.com/"))
                .build();
    }
}
```

使用接口测试工具添加cookie：`sessionId=test`，则会返回页面代码，如果去掉`sessionId=test`或设置的 Cookie 的 key 与 value 与指定的不同，则无法访问到，报 404 错误

![image-20231206214221518](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202312062144143.png)

#### 通过Host匹配

该断言工厂(`Host RoutePredicate` )通过接收一组参数，一组匹配的域名列表，这个模板是一个 ant 分隔的模板，用`.`号作为分隔符。当请求中携带了指定的 Host 属性值时，匹配成功会执行路由(断言为 true)，如果没有匹配上则不执行。

**配置式路由配置**：

```yaml
spring:
  cloud:
    gateway:
      routes:
        #TODO 路由断言工厂:通过Host匹配
        - id: baidu_route
          uri: https://www.baidu.com
          predicates:
            - Host=**.baidu.com
```

**API式路由配置**：

```java
@SpringBootApplication
public class GatewayApplication {
    public static void main(String[] args) {
        SpringApplication.run(GatewayApplication.class,args);
    }
    //TODO 路由断言工厂:通过Host匹配
    @Bean
    public RouteLocator customRouteLocator(RouteLocatorBuilder builder) {
        return builder.routes()
                .route("baidu_route", r -> r.host("**.baidu.com")
                        .uri("https://www.baidu.com/"))
                .build();
    }
}
```

使用接口测试工具添加Host：`www.baidu.com`或`md.baidu.com`，则会匹配到路由，返回页面代码，如果去掉或设置与指定的不同，则无法访问到，报 404 错误

![image-20231206220437106](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202312062207634.png)

#### 通过RemoteAddr匹配

该断言工厂(`RemoteAddr RoutePredicate`)用于判断提交请求的客户端 IP 地址是否在断言中指定的 IP 范围或 IP 列表中。若在列表中，匹配成功会执行路由(断言为 true)，如果没有匹配上则不执行。

**配置式路由配置**：

```yaml
spring:
  cloud:
    gateway:
      routes:
        #TODO 路由断言工厂:通过客户端 IP 地址匹配
        - id: baidu_route
          uri: https://www.baidu.com
          predicates:
            - RemoteAddr=127.0.0.1/24
```

**API式路由配置**：

```java
@SpringBootApplication
public class GatewayApplication {
    public static void main(String[] args) {
        SpringApplication.run(GatewayApplication.class,args);
    }
    //TODO 路由断言工厂:通过客户端 IP 地址匹配
    @Bean
    public RouteLocator customRouteLocator(RouteLocatorBuilder builder) {
        return builder.routes()
                .route("baidu_route", r -> r.remoteAddr("127.0.0.1/24")
                        .uri("https://www.baidu.com/"))
                .build();
    }
}
```

![image-20231208132703533](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202401222001686.png)

#### 通过Weight匹配

该断言工厂(`Weight RoutePredicate`)用于实现对同一组中的不同 uri 实现指定权重的负载均衡。路由中包含两个参数， 分别是用于表示组 group，与权重 weight。对于同一组中的多个 uri 地址，路由器会根据设置的权重，按比例将请求转发给相应的 uri。

****

指定对两个电商平台 jd.com 与 tabao.com 的访问权重为 8:2。以下两 个路由都属于电商 online-retailers 组。 

在大量请求的前提下，对于相同的请求，转发到 tabao 的机率占 8 成，转发到 jd 的机 率占 2 成。但这不是一个绝对准确的访问比例，大体差不多。

**配置式路由配置**：

```yaml
spring:
  cloud:
    gateway:
      routes:
        #TODO 路由断言工厂:通过Weight匹配
        - id: jd_route
          uri: https://www.jd.com
          predicates:
            - Weight=online-retailers,8
        - id: tb_route
          uri: https://www.taobao.com
          predicates:
            - Weight=online-retailers,2
```

**API式路由配置**：

```java
@SpringBootApplication
public class GatewayApplication {
    public static void main(String[] args) {
        SpringApplication.run(GatewayApplication.class,args);
    }
    //TODO 路由断言工厂:通过Weight匹配
    @Bean
    public RouteLocator customRouteLocator(RouteLocatorBuilder builder) {
        return builder.routes()
                .route("jd_route", r -> r.weight("online-retailers",8)
                        .uri("https://www.jd.com/"))
                .route("taobao_route", r -> r.weight("online-retailers",2)
                        .uri("https://www.taobao.com/"))
                .build();
    }
}
```

多次提交，发现大多数会跳转到 jd，但偶尔 也会跳转到 taobao

#### 通过XForwardedRemoteAddr匹配

该断言工厂(`XForwardedRemoteAddr RoutePredicate`)在当前请求头中追加入 X-Forwarded-For 的 IP 出现在路由指定的 IP 列表中，则匹配成功会执行路由(断言为 true)，如果没有匹配上则不执行。

**配置式路由配置**：

```yaml
spring:
  cloud:
    gateway:
      routes:
        #TODO 路由断言工厂:通过XForwardedRemoteAddr匹配
        - id: baidu_route
          uri: https://www.baidu.com
          predicates:
            - XForwardedRemoteAddr=192.168.1.100/24,192.168.1.129
```

**API式路由配置**：

```java
@SpringBootApplication
public class GatewayApplication {
    public static void main(String[] args) {
        SpringApplication.run(GatewayApplication.class,args);
    }
    // TODO 路由断言工厂:通过XForwardedRemoteAddr匹配
    @Bean
    public RouteLocator customRouteLocator(RouteLocatorBuilder builder) {
        return builder.routes()
                .route("baidu_route", r -> r.xForwardedRemoteAddr("192.168.1.100/24","192.168.1.129")
                        .uri("https://www.baidu.com/"))
                .build();
    }
}
```

![image-20231210002504807](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202312100025332.png)

#### 通过Query请求参数匹配

该断言工厂(`Query RoutePredicate`)用于从请求中查找指定的请求参数。其可以只查看参数名称，也可以同时查看参数名与参数值(可以是正则表达式)。当请求中包含了指定的参数名，或名值对时，匹配成功会执行路由(断言为 true)，如果没有匹配上则不执行。

**配置式路由配置**：

```yaml
spring:
  cloud:
    gateway:
      routes:
      #TODO 路由断言工厂:通过请求参数匹配
        - id: baidu_route
          uri: https://www.baidu.com
          predicates:
            - Query=smile
```

**API式路由配置**：

```java
@SpringBootApplication
public class GatewayApplication {
    public static void main(String[] args) {
        SpringApplication.run(GatewayApplication.class,args);
    }
    // TODO 路由断言工厂:通过请求参数匹配
    @Bean
    public RouteLocator customRouteLocator(RouteLocatorBuilder builder) {
        return builder.routes()
                .route("baidu_route", r -> r.query("smile")
                        .uri("https://www.baidu.com/"))
                .build();
    }
}
```

只要请求中包含 smile 属性的参数即可匹配路由，不带 smile 参数则不会匹配。

![image-20231211130547164](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202312111305387.png)

****

可以将 Query 的值以键值对的方式进行配置，这样在请求过来时会对属性值和正则进行匹配，匹配上才会走路由。

**配置式路由配置**：

```yaml
spring:
  cloud:
    gateway:
      routes:
      #TODO 路由断言工厂:通过请求参数匹配
        - id: baidu_route
          uri: https://www.baidu.com
          predicates:
            #以 pu 开头的长度为三位可以访问到
            - Query=keep, pu.
            #以 gr 开头的任意长度可以访问到
            #- Query=color, gr.+
```

**API式路由配置**：

```java
@SpringBootApplication
public class GatewayApplication {
    public static void main(String[] args) {
        SpringApplication.run(GatewayApplication.class,args);
    }
    // TODO 路由断言工厂:通过请求参数匹配
    @Bean
    public RouteLocator customRouteLocator(RouteLocatorBuilder builder) {
        return builder.routes()
                .route("baidu_route", r -> r.query("keep","pu.")
                        .uri("https://www.baidu.com/"))
                .build();
    }
}
```

只要当请求中包含 keep 属性并且参数值是以 pu 开头的长度为三位的字符串才会进行匹配和路由。`keep=pub`，测试可以返回页面代码，将 keep 的属性值改为 pubx 再次访问就会报 404，证明路由需要匹配正则表达式才会进行路由。

![image-20231211140042864](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202312111400984.png)

#### 通过Method请求方式匹配

通过 POST、GET、PUT、DELETE 等不同的请求方式来进行路由。当请求中使用了指定的请求方法时，匹配成功会执行路由(断言为 true)，如果没有匹配上则不执行。

**配置式路由配置**：

```yaml
spring:
  cloud:
    gateway:
      routes:
      #TODO 路由断言工厂:通过Method匹配
        - id: baidu_route
          uri: https://www.baidu.com
          predicates:
            - Method=GET,POST
```

**API式路由配置**：

```java
@SpringBootApplication
public class GatewayApplication {
    public static void main(String[] args) {
        SpringApplication.run(GatewayApplication.class,args);
    }
    //TODO 路由断言工厂:通过Method匹配
    @Bean
    public RouteLocator customRouteLocator(RouteLocatorBuilder builder) {
        return builder.routes()
                .route("baidu_route", r -> r.method("GET","POST")
                        .uri("https://www.baidu.com/"))
                .build();
    }
}
```

使用接口测试工具发送GET、POST请求，则会匹配到路由，返回页面代码，如果以其他方式，则无法访问到，报 404 错误

![image-20231206225851003](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202312062259790.png)

#### 通过Path请求路径匹配

该断言工厂(`Path RoutePredicate` )通过接收一个匹配路径的参数来判断是否路由。用于判断请求路径中是否包含指定的 URI。若包含，则匹配成功，断言为 true， 此时会将该匹配上的 uri 拼接到要转向的目标 URI的后面，形成一个统一的 URI。

**配置式路由配置**：

```yaml
spring:
  cloud:
    gateway:
      routes:   
      #TODO 路由断言工厂:通过Path匹配图书服务
        - id: book-route
          uri: http://localhost:8201/
          predicates:
            - Path=/book/{segment}
```

**API式路由配置**：

```java
@SpringBootApplication
public class GatewayApplication {
    public static void main(String[] args) {
        SpringApplication.run(GatewayApplication.class,args);
    }
    //TODO 路由断言工厂:通过Path匹配图书服务
    @Bean
    public RouteLocator customRouteLocator(RouteLocatorBuilder builder) {
        return builder.routes()
                .route("book-route", r -> r.path("/book/{segment}")
                        .uri("http://localhost:8201/"))
                .build();
    }
}
```

启动端口号8201图书服务，如果请求路径符合要求，则此路由将匹配，可以正常获取到页面返回值：http://localhost:9000/book/1。访问http://localhost:9000/book/1/2，报404，证明路由是通过指定路由来匹配。

![image-20231211142351260](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202312111423244.png)

#### 组合匹配

当使用多个路由断言工厂时，可以进行组合匹配，其关系包括：或、与关系。

**与关系**：

- **配置式路由配置**：各种 Predicates 同时存在于同一个路由时，请求必须同时满足所有的条件才被这个路由匹配。

  ```yaml
  spring:
    cloud:
      gateway:
        routes:   
          - id: book-route
            uri: http://localhost:8201/
            predicates:
              - Path=/book/{segment}
              - Host=**.foo.org
              - Method=GET
              - Header=X-Request-Id, \d+
              - Query=foo, ba.
              - Query=baz
              - Cookie=chocolate, ch.p
  ```

- **API式路由配置**：

  ```java
  @SpringBootApplication
  public class GatewayApplication {
      public static void main(String[] args) {
          SpringApplication.run(GatewayApplication.class,args);
      }
      @Bean
      public RouteLocator customRouteLocator(RouteLocatorBuilder builder) {
          return builder.routes()
                  .route("book-route", r -> r.path("/book/{segment}")
                         .and()
                         .host("**.foo.org")
                         .and()
                         .method("GET")
                         .and()
                         .header("X-Request-Id","\\d+")
                         .and()
                         .query("foo","ba.")
                         .and()
                         .query("baz")
                         .and()
                         .cookie("chocolate","ch.p")
                       .uri("http://localhost:8201/"))
                  .build();
      }
  }
  ```

  ![image-20231211165814038](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202312111658032.png)

**或关系**：

- **配置式路由配置**：一个请求满足多个路由的断言条件时，请求只会被首个成功匹配的路由转发。

  ```yaml
  spring:
    cloud:
      gateway:
        routes:   
          - id: book-route
            uri: http://localhost:8201/
            predicates:
              - Path=/book/{segment}
              - Host=**.foo.org
          - id: book-route2
            uri: http://localhost:8201/
            predicates:
              - Path=/book/{segment}
              - Method=GET
          - id: book-route3
            uri: http://localhost:8201/
            predicates: 
              - Path=/book/{segment}
              - Header=X-Request-Id, \d+
  ```

- **API式路由配置**：

  ```java
  @SpringBootApplication
  public class GatewayApplication {
      public static void main(String[] args) {
          SpringApplication.run(GatewayApplication.class,args);
      }
      
          @Bean
      public RouteLocator customRouteLocator(RouteLocatorBuilder builder) {
          return builder.routes()
                  .route("book-route", r -> r.path("/book/{segment}")
                          .and()
                          .method("POST")
                          .uri("http://localhost:8201/"))
                  .route("book-route2", r -> r.path("/book/{segment}")
                          .and()
                          .header("X-Request-Id", "\\d+")
                          .uri("http://localhost:8201/"))
                  .route("book-route2", r -> r.path("/book/{segment}")
                          .and()
                          .host("**.foo.org")
                          .uri("http://localhost:8201/"))
                  .build();
      }
  }
  ```

  ![image-20231211172123667](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202312111721936.png)

  ![image-20231211172128014](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202312111721764.png)

  ![image-20231211172309464](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202312111723309.png)

#### 优先级

 当一个应用中对于相同的路由断言工厂，分别通过配置式与 API 式都进行了设置，且设置的值不同，此时它们的关系有两种：

- 或的关系：例如，Header 路由断言工厂、Query 路由断言工厂

- 优先级关系：例如，After、Before、Between 路由断言工厂。

  此时配置式的优先级要高 于 API 式的。

### 网关过滤工厂

网关过滤器工厂 GatewayFilterFactory支持以某种方式修改传入的 HTTP 请求或传出的 HTTP 响应，其作用域是某些特定路由。跟之前的断言一样，Spring Cloud Gateway 也包含许多内置的路由过滤器工厂。

在请求到达时，在请求头中添加一些信息再转发给我们的服务，那么这个时候就可以使用路由过滤器来完成。

详细列表：https://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/#gatewayfilter-factories

常见过滤器：

- PrefixPath：对请求添加前缀
- StripPrefix：跳过指定的路径
- RewritePath：访问URI替换为另一个指定的 URI
- SetPath：访问URI替换为另一个指定的 URI
- AddRequestHeader：添加请求头信息
- AddRequestHeadersIfNotPresent：添加未添加过的请求头信息
- AddRequestParameter：添加请求参数信息
- AddResponseHeader：添加响应头信息
- RemoveRequestHeader：
- RemoveResponseHeader：
- RemoveRequestParameter：
- CircuitBreaker：网关层的服务熔断与降级
- RequestRateLimiterGateway：
- SetRequestHeader：
- default-filters：

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202401232124634.png)

#### PrefixPath

PrefixPath 过滤器工厂会为指定的 URI 自动添加上一个指定的 URI 前辍。

****

对所有的请求路径添加前缀：

- 配置式：

  ```yaml
  spring:
    cloud:
      gateway:
        routes:
          - id: my-router
            uri: http://localhost:8080
            predicates:
              - Path=/{segment}
            filters:  # 添加过滤器
              - PrefixPath=/info
  ```

- API式：

  ```java
  @SpringBootApplication
  public class GatewayApplication {
      public static void main(String[] args) {
          SpringApplication.run(GatewayApplication.class, args);
      }
  
      // 基于代码的路由配置方式
  
      //TODO PrefixPath 过滤器:对匹配上的添加URI前缀
      @Bean
      public RouteLocator customRouteLocator(RouteLocatorBuilder builder) {
          return builder.routes()
                  .route("my_router", r -> r.path("/{segment}")
                          .filters(fs ->
                                  fs.prefixPath("/info"))
                          .uri("http://localhost:8080"))
                  .build();
      }
  }
  ```

在 Controller 处理器中添加如下处理器方法：

```java
@RequestMapping("/info")
@RestController
public class SomeController {
    @RequestMapping("/allHeaders")
    public String allHeaders(HttpServletRequest request) {
        // 获取请求中所有的请求头
        Enumeration<String> headerNames = request.getHeaderNames();

        //遍历所有请求头
        StringBuilder sb = new StringBuilder();
        while (headerNames.hasMoreElements()) {
            String name = headerNames.nextElement();
            sb.append(name).append(":");
            //获取指定key的header的所有值
            Enumeration<String> headers = request.getHeaders(name);
            //遍历该枚举实例
            while (headers.hasMoreElements()){
                sb.append(headers.nextElement()).append(" ");
            }
            sb.append("<br>");
        }
        return sb.toString();
    }
}
```

访问 http://localhost:9000/allHeaders 成功获取信息：

![image-20240120165850237](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202401222001446.png)

#### StripPrefix

StripPrefix 过滤器工厂会为指定的 URI 去掉指定长度的前辍。

****

跳过指定的路径：

- 配置式：

  ```yaml
  spring:
    cloud:
      gateway:
        routes:
          - id: my-router
            uri: http://localhost:8080
            predicates:
              - Path=/api/{segment}
            filters:  # 添加过滤器
              - StripPrefix=1
              - PrefixPath=/info
  ```

- API式：

  ```java
  @SpringBootApplication
  public class GatewayApplication {
      public static void main(String[] args) {
          SpringApplication.run(GatewayApplication.class, args);
      }
  
      // 基于代码的路由配置方式
  
      //TODO StripPrefix 过滤器:对匹配上的跳过指定的路径
      @Bean
      public RouteLocator customRouteLocator(RouteLocatorBuilder builder) {
          return builder.routes()
                  .route("my_router", r -> r.path("/api/{segment}")
                          .filters(fs ->
                                  fs.stripPrefix(1)
                                    .prefixPath("/info"))
                          .uri("http://localhost:8080"))
                  .build();
      }
  }
  ```

在 Controller 处理器中添加如下处理器方法：

```java
@RequestMapping("/info")
@RestController
public class SomeController {
    @RequestMapping("/allHeaders")
    public String allHeaders(HttpServletRequest request) {
        // 获取请求中所有的请求头
        Enumeration<String> headerNames = request.getHeaderNames();

        //遍历所有请求头
        StringBuilder sb = new StringBuilder();
        while (headerNames.hasMoreElements()) {
            String name = headerNames.nextElement();
            sb.append(name).append(":");
            //获取指定key的header的所有值
            Enumeration<String> headers = request.getHeaders(name);
            //遍历该枚举实例
            while (headers.hasMoreElements()){
                sb.append(headers.nextElement()).append(" ");
            }
            sb.append("<br>");
        }
        return sb.toString();
    }
}
```

访问 http://localhost:9000/api/allHeaders 首先StripPrefix过滤器去掉一个/api，然后PrefixPath过滤器加上一个`/info`，能够正确访问：

![image-20240120171606386](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202401222001276.png)

#### RewritePath

RewritePath过滤器工厂会将请求 URI 替换为另一个指定的 URI 进行访问。

RewritePath 有两个**参数**：

- 第一个是正则表达式

- 第二个是要替换为的目标表达式

**应用场景**：在某子模块中的某功能，通过其它子模块也可以访问，此时就可以通过 uri 替换方式让不同的子模块路径跳转到同一个路径。

****

访问URI替换为另一个指定的 URI：

- 无正则替换：

  - 配置式：

    ```yaml
    spring:
      cloud:
        gateway:
          routes:
            - id: my-router
              uri: http://localhost:8080
              predicates:
                - Path=/api/**
              filters:  # 添加过滤器
                - RewritePath=/api, /info
    ```

  - API式：

    ```java
    @SpringBootApplication
    public class GatewayApplication {
        public static void main(String[] args) {
            SpringApplication.run(GatewayApplication.class, args);
        }
    
        // 基于代码的路由配置方式
    
        //TODO RewritePath 过滤器:访问URI替换为另一个指定的 URI
        @Bean
        public RouteLocator customRouteLocator(RouteLocatorBuilder builder) {
            return builder.routes()
                    .route("my_router", r -> r.path("/api/**")
                            .filters(fs ->
                                    fs.rewritePath("/api", "/info"))
                            .uri("http://localhost:8080"))
                    .build();
        }
    }
    ```

- 有正则替换：

  - 配置式：

    ```yaml
    spring:
      cloud:
        gateway:
          routes:
            - id: my-router
              uri: http://localhost:8080
              predicates:
                - Path=/api/{segment}
              filters:  # 添加过滤器
                - RewritePath=/api/(?<segment>.*), /info/$\{segment}
    ```

  - API式：

    ```java
    @SpringBootApplication
    public class GatewayApplication {
        public static void main(String[] args) {
            SpringApplication.run(GatewayApplication.class, args);
        }
    
        // 基于代码的路由配置方式
    
        //TODO RewritePath 过滤器:访问URI替换为另一个指定的 URI
        @Bean
        public RouteLocator customRouteLocator(RouteLocatorBuilder builder) {
            return builder.routes()
                    .route("my_router", r -> r.path("/api/{segment}")
                            .filters(fs ->
                                    fs.rewritePath("/api/(?<segment>.*)", "/info/$\\{segment}"))
                            .uri("http://localhost:8080"))
                    .build();
        }
        
    }
    ```


在 Controller 处理器中添加如下处理器方法：

```java
@RequestMapping("/info")
@RestController
public class SomeController {
    @RequestMapping("/allHeaders")
    public String allHeaders(HttpServletRequest request) {
        // 获取请求中所有的请求头
        Enumeration<String> headerNames = request.getHeaderNames();

        //遍历所有请求头
        StringBuilder sb = new StringBuilder();
        while (headerNames.hasMoreElements()) {
            String name = headerNames.nextElement();
            sb.append(name).append(":");
            //获取指定key的header的所有值
            Enumeration<String> headers = request.getHeaders(name);
            //遍历该枚举实例
            while (headers.hasMoreElements()){
                sb.append(headers.nextElement()).append(" ");
            }
            sb.append("<br>");
        }
        return sb.toString();
    }
}
```

访问 http://localhost:9000/api/allHeaders RewritePath过滤器将路径重写为http://localhost:8080/info/allHeaders，能够正确访问：

![image-20240122212320416](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202401222123206.png)

#### SetPath

SetPath和Rewrite类似：

- 配置式：

  ```yaml
  spring:
    cloud:
      gateway:
        routes:           
          - id: my-router
            uri: http://localhost:8080
            predicates:
              - Path=/api/{segment}
            filters:
              - SetPath=/info/{segment}
  ```

- API式：

  ```java
  @SpringBootApplication
  public class GatewayApplication {
      public static void main(String[] args) {
          SpringApplication.run(GatewayApplication.class, args);
      }
  
      // 基于代码的路由配置方式
  
      @Bean
      public RouteLocator customRouteLocator(RouteLocatorBuilder builder) {
          return builder.routes()
                  .route("my_router", r -> r.path("/api/{segment}")
                          .filters(fs ->
                                  fs.setPath("/info/{segment}"))
                          .uri("http://localhost:8080"))
                  .build();
      }
      
  }
  ```

在 Controller 处理器中添加如下处理器方法：

```java
@RequestMapping("/info")
@RestController
public class SomeController {
    @RequestMapping("/allHeaders")
    public String allHeaders(HttpServletRequest request) {
        // 获取请求中所有的请求头
        Enumeration<String> headerNames = request.getHeaderNames();

        //遍历所有请求头
        StringBuilder sb = new StringBuilder();
        while (headerNames.hasMoreElements()) {
            String name = headerNames.nextElement();
            sb.append(name).append(":");
            //获取指定key的header的所有值
            Enumeration<String> headers = request.getHeaders(name);
            //遍历该枚举实例
            while (headers.hasMoreElements()){
                sb.append(headers.nextElement()).append(" ");
            }
            sb.append("<br>");
        }
        return sb.toString();
    }
}
```

请求http://localhost:9000/api/allHeaders 路径，SetPath过滤器将路径设置为 http://localhost:8080/borrow/1，能够正确访问。

![image-20240122212320416](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202401222123206.png)

#### AddRequestHeader

AddRequestHeader 过滤器工厂会对匹配上的请求添加指定的 header。

添加请求头信息：

- 配置式：

  ```yaml
  spring:
    cloud:
      gateway:
        routes:
          - id: my-router
            uri: http://localhost:8080
            predicates:
              - Path=/**
            filters:  # 添加过滤器
              - AddRequestHeader=X-Request-Color,blue
  ```

- API式：

  ```java
  @SpringBootApplication
  public class GatewayApplication {
      public static void main(String[] args) {
          SpringApplication.run(GatewayApplication.class, args);
      }
  
      // 基于代码的路由配置方式
  
      //TODO AddRequestHeader:对匹配上的请求添加指定的 header
      @Bean
      public RouteLocator customRouteLocator(RouteLocatorBuilder builder) {
          return builder.routes()
                  .route("my_router", r -> r.path("/**")
                          .filters(fs -> fs.addRequestHeader("X-Request-Color","blue"))
                          .uri("http://localhost:8080"))
                  .build();
      }
  }
  ```

在 Controller 处理器中添加如下处理器方法：

```java
@RequestMapping("/info")
@RestController
public class SomeController {
    @RequestMapping("/header")
    public String header(HttpServletRequest request) {
        String header = request.getHeader("X-Request-Color");
        return "X-Request-Color:" + header;
    }
}
```

访问 http://localhost:9000/info/header 成功获取到由网关添加的请求头信息：

![image-20240119224053235](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202401222001063.png)

#### AddRequestHeadersIfNotPresent

请求中允许出现多个同名的请求头。即同一名称的请求头可以被多次添加到请求中。

有些场景下为了保证请求中对于某请求头只有一个值，就可以使用该过滤工厂。 该过滤器工厂会对匹配上的请求添加指定的 header，前提是该 header 在当前请求中尚未出现过。

添加未添加过的请求头信息：

- 配置式：

  ```yaml
  spring:
    cloud:
      gateway:
        routes:
          - id: my-router
            uri: http://localhost:8080
            predicates:
              - Path=/**
            filters:  # 添加过滤器
              - AddRequestHeadersIfNotPresent=X-Request-Color:blue,City:beijing
  ```

- API式：

  ```java
  @SpringBootApplication
  public class GatewayApplication {
      public static void main(String[] args) {
          SpringApplication.run(GatewayApplication.class, args);
      }
  
      // 基于代码的路由配置方式
  
      //TODO AddRequestHeader:对匹配上的请求添加指定的 header
      @Bean
      public RouteLocator customRouteLocator(RouteLocatorBuilder builder) {
          return builder.routes()
                  .route("my_router", r -> r.path("/**")
                  .filters(fs -> fs.addRequestHeadersIfNotPresent("X-Request-Color:blue","City:beijing"))
                          .uri("http://localhost:8080"))
                  .build();
      }
  }
  ```

在 Controller 处理器中添加如下处理器方法，该方法遍历了当前请求中的所有 header：

```java
@RequestMapping("/info")
@RestController
public class SomeController {
    @RequestMapping("/allHeaders")
    public String allHeaders(HttpServletRequest request) {
        // 获取请求中所有的请求头
        Enumeration<String> headerNames = request.getHeaderNames();

        //遍历所有请求头
        StringBuilder sb = new StringBuilder();
        while (headerNames.hasMoreElements()) {
            String name = headerNames.nextElement();
            sb.append(name).append(":");
            //获取指定key的header的所有值
            Enumeration<String> headers = request.getHeaders(name);
            //遍历该枚举实例
            while (headers.hasMoreElements()){
                sb.append(headers.nextElement()).append(" ");
            }
            sb.append("<br>");
        }
        return sb.toString();
    }
}
```

访问 http://localhost:9000/info/allHeaders 成功获取到由网关添加的未添加的请求头信息：

![image-20240120001707385](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202401222001528.png)

#### AddRequestParameter

AddRequestParameter 过滤器工厂会对匹配上的请求添加指定的请求参数。

添加请求参数信息：

- 配置式：

  ```yaml
  spring:
    cloud:
      gateway:
        routes:
          - id: my-router
            uri: http://localhost:8080
            predicates:
              - Path=/**
            filters:  # 添加过滤器
            # 添加了一个两个 param，其中 color 还有两个值。
              - AddRequestParameter=color,blue
              - AddRequestParameter=color,red
              - AddRequestParameter=size,23
  ```

- API式：

  ```java
  @SpringBootApplication
  public class GatewayApplication {
      public static void main(String[] args) {
          SpringApplication.run(GatewayApplication.class, args);
      }
  
      // 基于代码的路由配置方式
  
      //TODO AddRequestParameter:对匹配上的请求添加指定的 请求参数
      @Bean
      public RouteLocator customRouteLocator(RouteLocatorBuilder builder) {
          return builder.routes()
                  .route("my_router", r -> r.path("/**")
                          .filters(fs ->
                                  fs.addRequestParameter("color", "blue")
                                    .addRequestParameter("color", "red")
                                    .addRequestParameter("size", "23"))
                          .uri("http://localhost:8080"))
                  .build();
      }
  }
  ```

在 Controller 处理器中添加如下处理器方法，该方法遍历了当前请求中的所有请求参数：

```java
@RequestMapping("/info")
@RestController
public class SomeController {
    @RequestMapping("/params")
    public String params(HttpServletRequest request) {
        StringBuilder sb = new StringBuilder();
        // 获取请求中所有的请求参数及其值
        Map<String, String[]> map = request.getParameterMap();
        // 遍历所有请求参数
        for (String key : map.keySet()) {
            sb.append(key).append(":");
            // 遍历当前请求参数的所有值
            for (String value : map.get(key)){
                sb.append(value).append(" ");
            }
            sb.append("<br>");
        }
        return sb.toString();
    }
}
```

访问 http://localhost:9000/info/params 成功获取到由网关添加的请求参数信息：

![image-20240120154737165](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202401201558986.png)

#### AddResponseHeader

AddResponseHeader 过滤器工厂会给从网关返回的响应添加上指定的 header。

添加响应头信息：

- 配置式：

  ```yaml
  spring:
    cloud:
      gateway:
        routes:
          - id: my-router
            uri: http://localhost:8080
            predicates:
              - Path=/**
            filters:  # 添加过滤器
              - AddResponseHeader=Color,Blue
              - AddResponseHeader=Color,Red
              - AddResponseHeader=User,Tom
  ```

- API式：

  ```java
  @SpringBootApplication
  public class GatewayApplication {
      public static void main(String[] args) {
          SpringApplication.run(GatewayApplication.class, args);
      }
  
      // 基于代码的路由配置方式
  
      //TODO AddResponseHeader:对匹配上的响应添加指定的响应头
      @Bean
      public RouteLocator customRouteLocator(RouteLocatorBuilder builder) {
          return builder.routes()
                  .route("my_router", r -> r.path("/**")
                          .filters(fs ->
                                  fs.addResponseHeader("Color","Blue")
                                    .addResponseHeader("Color","Red")
                                    .addResponseHeader("User","Tom"))
                          .uri("http://localhost:8080"))
                  .build();
      }
  }
  ```

在 Controller 处理器中添加如下处理器方法，该方法遍历了当前请求中的所有 header：

```java
@RequestMapping("/info")
@RestController
public class SomeController {
    @RequestMapping("/allHeaders")
    public String allHeaders(HttpServletRequest request) {
        // 获取请求中所有的请求头
        Enumeration<String> headerNames = request.getHeaderNames();

        //遍历所有请求头
        StringBuilder sb = new StringBuilder();
        while (headerNames.hasMoreElements()) {
            String name = headerNames.nextElement();
            sb.append(name).append(":");
            //获取指定key的header的所有值
            Enumeration<String> headers = request.getHeaders(name);
            //遍历该枚举实例
            while (headers.hasMoreElements()){
                sb.append(headers.nextElement()).append(" ");
            }
            sb.append("<br>");
        }
        return sb.toString();
    }
}
```

访问 http://localhost:9000/info/allHeaders 成功获取到由网关添加的响应头信息：

![image-20240120161202442](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202401222001974.png)

#### RemoveRequestHeader

去掉请求头信息：

```yaml
spring:
  cloud:
    gateway:
      routes:           
        - id: my-router
          uri: http://localhost:8080
          predicates:
            - Path=/info/{segment}
          filters:
            - RemoveRequestHeader=Test
```

在 Controller 处理器中添加如下处理器方法，该方法遍历了当前请求中的所有 header：

```java
@RequestMapping("/info")
@RestController
public class SomeController {
    @RequestMapping("/allHeaders")
    public String allHeaders(HttpServletRequest request) {
        // 获取请求中所有的请求头
        Enumeration<String> headerNames = request.getHeaderNames();

        //遍历所有请求头
        StringBuilder sb = new StringBuilder();
        while (headerNames.hasMoreElements()) {
            String name = headerNames.nextElement();
            sb.append(name).append(":");
            //获取指定key的header的所有值
            Enumeration<String> headers = request.getHeaders(name);
            //遍历该枚举实例
            while (headers.hasMoreElements()){
                sb.append(headers.nextElement()).append(" ");
            }
            sb.append("<br>");
        }
        return sb.toString();
    }
}
```

![image-20240122230849502](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202401222308138.png)

#### RemoveResponseHeader

去掉响应头信息：

```yaml
spring:
  cloud:
    gateway:
      routes:           
        - id: my-router
          uri: http://localhost:8080
          predicates:
            - Path=/info/{segment}
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
        - id: my-router
          uri: http://localhost:8080
          predicates:
            - Path=/info/{segment}
          filters:
            - RemoveRequestParameter=test
```

在 Controller 处理器中添加如下处理器方法，该方法遍历了当前请求中的所有 header：

```java
@RequestMapping("/info")
@RestController
public class SomeController {
    @RequestMapping("/params")
    public String params(HttpServletRequest request) {
        StringBuilder sb = new StringBuilder();
        // 获取请求中所有的请求参数及其值
        Map<String, String[]> map = request.getParameterMap();
        // 遍历所有请求参数
        for (String key : map.keySet()) {
            sb.append(key).append(":");
            // 遍历当前请求参数的所有值
            for (String value : map.get(key)){
                sb.append(value).append(" ");
            }
            sb.append("<br>");
        }
        return sb.toString();
    }
}
```

![image-20240122233131537](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202401222331236.png)

#### DedupeResponseHeader

`DedupeResponseHeader` 剔除重复的响应头，接受一个name参数和一个可选strategy参数。name可以包含以空格分隔的标题名称列表。

该DedupeResponseHeader过滤器还接受一个可选的strategy参数：

- `RETAIN_FIRST`：(默认)默认值，保留第一个

- `RETAIN_LAST`：保留最后一个

- `RETAIN_UNIQUE`：保留唯一的，出现重复的属性值，会保留一个

  例如有两个My:aaa的属性，最后会保留一个。

```yaml
spring:
  cloud:
    gateway:
      routes:
       - id: my-router
         uri: http://localhost:8080
         filters:
          # DedupeResponseHeader=响应头名称 响应头参数,strategy
          - DedupeResponseHeader=Access-Control-Allow-Credentials Access-Control-Allow-Origin, RETAIN_LAST
```

如果网关 CORS 逻辑和下游逻辑都添加了响应头Access-Control-Allow-Credentials和Access-Control-Allow-Origin响应头的重复值，这将删除它们。

#### CircuitBreaker

CircuitBreaker 过滤器工厂完成网关层的服务熔断与降级。

添加依赖：在API接口项目和Gateway工程中均需要添加上 resilience4j 依赖。

```xml
<!--resilience4j 依赖-->
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-circuitbreaker-reactor-resilience4j</artifactId>
</dependency>
```

定义降级处理器：在API接口项目和Gateway工程中均需要添加上该降级处理器。

```java
@RestController
public class FallbackController {
    @RequestMapping("/fb")
    public String fallbackHandle(){
        return "This is the Gateway Fallback";
    }
}
```

添加服务熔断与降级：

- 配置式：

  ```yaml
  spring:
    cloud:
      gateway:
        routes:
          - id: my_router
            uri: http://localhost:8080
            predicates:
              - Path=/info/**
            filters:
             # 指定要使用的 filter 类型，不能随意写。
              -  name: CircuitBreaker
              # args 是对指定 filter 的配置参数。
                 args:
                   # 用于指定当前所使用断路器的名称，可以随意。
                   name: myCircuitBreaker
                   # fallbackUri 属性用于指定发生断路后要提交的降级 URI。
                   fallbackUri: forward:/fb
  ```

- API式：

  ```java
  @SpringBootApplication
  public class GatewayApplication {
      public static void main(String[] args) {
          SpringApplication.run(GatewayApplication.class, args);
      }
  
      // 基于代码的路由配置方式
  
      //TODO CircuitBreaker  过滤器:服务熔断降级
      @Bean
      public RouteLocator customRouteLocator(RouteLocatorBuilder builder) {
          return builder.routes()
                  .route("my_router", r -> r.path("/info/**")
                          .filters(fs ->
                                  fs.circuitBreaker(config -> {
                                      config.setName("myCircuitBreaker");
                                      config.setFallbackUri("forward:/fb");
                                  }))
                          .uri("http://localhost:8080"))
                  .build();
      }
  }
  ```

通过不启动API接口项目来模拟宕机场景：

![image-20240122200041181](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202401222000433.png)

#### RequestRateLimiter

RequestRateLimiter 过滤器工厂通过**令牌桶算法**对进来的请求进行限流。

限流器是基于 Redis，所以需要导入 Redis 的 Starter 依赖：

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-data-redis-reactive</artifactId>
</dependency>
```

添加限流键解析器：

在启动类中添加一个限流键解析器，其用于从请求中解析出需要限流的 key。限流 key 可以是用户、URI，也可以是目标服务器的 host 或 IP。 

- 指定的是根据请求要访问的目标服务器的 host 进行限流：

  ```java
  @SpringBootConfiguration
  public class GatewayConfig {
  
      /**
       * 根据请求要访问的目标服务器的 host 进行限流
       * @return KeyResolver
       */
      @Bean
      KeyResolver hostKeyResolver() {
          return exchange -> Mono.just(exchange.getRequest().getRemoteAddress().getHostName());
      }
  }
  ```

- 指定的是根据请求要访问的目标服务器的 URI 进行限流：

  ```java
  @SpringBootConfiguration
  public class GatewayConfig {
      /**
       * 根据请求要访问的目标服务器的 URI 进行限流
       * @return KeyResolver
       */
      @Bean
      KeyResolver pathKeyResolver() {
          return exchange -> Mono.just(exchange.getRequest().getPath().value());
      }
  }
  ```

- 指定的是根据请求要访问的目标服务器的请求参数进行限流：

  ```java
  @SpringBootConfiguration
  public class GatewayConfig {
      /**
       * 根据请求要访问的目标服务器的请求参数进行限流
       * @return KeyResolver
       */
      @Bean
      KeyResolver userKeyResolver() {
          return exchange -> Mono.just(exchange.getRequest().getQueryParams().getFirst("user"));
      }
  }
  ```
  

修改配置文件：

```yaml
server:
  port: 9000
spring:
  data:
    redis:
      host: 123.57.66.197
      port: 6379
  application:
    name: gateway
  cloud:
    gateway:
      routes:
       - id: my-router
         uri: http://localhost:8080
         predicates:
           - Path=/info/{segment}
         filters:
           - name: RequestRateLimiter
             args:
               key-resolver: "#{@userKeyResolver}"
               # 填充令牌速率 每秒两个
               redis-rate-limiter.replenishRate: 2
               # 突发生成令牌容量
               redis-rate-limiter.burstCapacity: 5
               # 请求处理使用令牌数量
               redis-rate-limiter.requestedTokens: 1
```

在 Controller 处理器中添加如下处理器方法，该方法遍历了当前请求中的所有请求参数：

```java
@RequestMapping("/info")
@RestController
public class SomeController {
    @RequestMapping("/params")
    public String params(HttpServletRequest request) {
        StringBuilder sb = new StringBuilder();
        // 获取请求中所有的请求参数及其值
        Map<String, String[]> map = request.getParameterMap();
        // 遍历所有请求参数
        for (String key : map.keySet()) {
            sb.append(key).append(":");
            // 遍历当前请求参数的所有值
            for (String value : map.get(key)){
                sb.append(value).append(" ");
            }
            sb.append("<br>");
        }
        return sb.toString();
    }
}
```

正常访问时是没有问题的：

![image-20240123090647988](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202401230906747.png)

但如果快速连续刷新页面，则会看到如下异常。这是限流后的结果：

![image-20240123194451290](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202401231945454.png)

#### SetRequestHeader

与 AddRequestHeader不同的是，这是替换 Header 而不是添加。

设置请求头信息：          

- 配置式：

  ```yaml
  spring:
    cloud:
      gateway:
        routes:
          - id: my-router
            uri: http://localhost:8080
            predicates:
              - Path=/**
            filters:  # 添加过滤器
              - SetRequestHeader=X-Request-Color, blue
  ```

- API式：

  ```java
  @SpringBootApplication
  public class GatewayApplication {
      public static void main(String[] args) {
          SpringApplication.run(GatewayApplication.class, args);
      }
  
      // 基于代码的路由配置方式
  
      //TODO SetRequestHeader
      @Bean
      public RouteLocator customRouteLocator(RouteLocatorBuilder builder) {
          return builder.routes()
                  .route("my_router", r -> r.path("/api/{segment}")
                          .filters(fs ->
                                  fs.setRequestHeader("X-Request-Color", "blue"))
                          .uri("http://localhost:8080"))
                  .build();
      }
  }
  ```

在 Controller 处理器中添加如下处理器方法：

```java
@RequestMapping("/info")
@RestController
public class SomeController {
    @RequestMapping("/header")
    public String header(HttpServletRequest request) {
        String header = request.getHeader("X-Request-Color");
        return "X-Request-Color:" + header;
    }
}
```

访问 http://localhost:9000/info/header 成功获取到由网关添加的请求头信息：

![image-20240119224053235](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202401222001063.png)

#### Default Filters

GatewayFilter 工厂是在某一特定路由策略中设置的，仅对这一种路由生效。若要使某些过滤效果应用到所有路由策略中，就可以将该 GatewayFilter 工厂定义在默认 Filters 中。

对所有的请求添加过滤器：

```yaml
spring:
  cloud:
    gateway:
      routes:        
        - id: my-router1
          uri: http://localhost:8080
          predicates:
            - Path=/info/{segment}
        - id: my-router2
          uri: http://localhost:8080
          predicates:
            - Path=/{segment}
          filters:  # 添加过滤器
            - PrefixPath=/info
      default-filters:
      	- AddRequestHeader=X-Request-Color,blue
```

在 Controller 处理器中添加如下处理器方法：

```java
@RequestMapping("/info")
@RestController
public class SomeController {
    @RequestMapping("/header")
    public String header(HttpServletRequest request) {
        String header = request.getHeader("X-Request-Color");
        return "X-Request-Color:" + header;
    }
}
```

访问 http://localhost:9000/info/header 成功获取到由网关添加的请求头信息：

![image-20240119224053235](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202401222001063.png)

访问 http://localhost:9000/header 成功获取到由网关添加的请求头信息：

![image-20240123225919000](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202401232259768.png)

#### 优先级

对于相同 filter 工厂，在不同位置设置了不同的值，则优先级为：

- 局部 filter 的优先级高于默认 filter 的 
- API 式的 filter 优先级高于配置式 filter 的

## 自定义异常处理器

当路由断言不匹配时会报出 404 异常信息，对用户非常不友好。所以需要自定义异常信 息，以更好的形式给出用户提示。

![image-20231212125800735](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202312121308526.png)

请求默认是由`DefaultErrorWebExceptionHandler`处理，它继承自`AbstractErrorWebExceptionHandler`：

![](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202312121308526.png)

通过 `getRoutingFunction()`方法 调用 `renderErrorView()`方法处理异常信息：

![image-20231212131044423](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202312121310789.png)

页面显示内容：`DefaultErrorAttributes`

![image-20231212131742174](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202312121317538.png)

默认展示页面：`AbstractErrorWebExceptionHandler`

![image-20231212132018533](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202312121320361.png)

### 自定义异常处理器类

通过继承 `AbstractErrorWebExceptionHandler` 实现自定义异常处理器类：

```java
/**
 * 自定义异常处理器
 * @author hjc
 */
@Component
@Order(-1)
public class CustomErrorWebExceptionHandler extends AbstractErrorWebExceptionHandler {


    public CustomErrorWebExceptionHandler(ErrorAttributes errorAttributes,
                                          ApplicationContext applicationContext,
                                          ServerCodecConfigurer serverCodecConfigurer) {
        super(errorAttributes,new WebProperties.Resources(), applicationContext);
        super.setMessageWriters(serverCodecConfigurer.getWriters());
        super.setMessageReaders(serverCodecConfigurer.getReaders());
    }

    @Override
    protected RouterFunction<ServerResponse> getRoutingFunction(ErrorAttributes errorAttributes) {
        //如果predicate为true,则执行handlerFunction
        return RouterFunctions.route(RequestPredicates.all(), this::renderErrorResponse);
    }

    private Mono<ServerResponse> renderErrorResponse(final ServerRequest serverRequest) {
        Map<String, Object> map = getErrorAttributes(request, ErrorAttributeOptions.defaults());
        //构建响应
        return ServerResponse.status(HttpStatus.NOT_FOUND)  //404状态码
                .contentType(MediaType.APPLICATION_JSON)  //指定以JSON格式响应
                .body(BodyInserters.fromValue(map));    //响应体(相应内容)
    }
}
```

### 更换异常信息

如果想更换系统给出的异常信息，则可定义一个 DefaultErrorAttributes 的子类，覆盖其 `getErrorAttributes()`方法。在该重写的方法中更换新的信息。

```java
import org.springframework.boot.web.error.ErrorAttributeOptions;
import org.springframework.boot.web.reactive.error.DefaultErrorAttributes;
import org.springframework.stereotype.Component;
import org.springframework.web.reactive.function.server.ServerRequest;
/**
 * 更换异常信息
 * @author hjc
 */
@Component
public class CustomErrorAttributes extends DefaultErrorAttributes {
    @Override
    public Map<String, Object> getErrorAttributes(ServerRequest request, ErrorAttributeOptions options) {
//        Map<String, Object> map = super.getErrorAttributes(request, options);
        HashMap<String, Object> map = new HashMap<>();
        map.put("code", HttpStatus.NOT_FOUND);
        map.put("message", "对不起，没有找到该资源");
        return map;
    }
}
```

![image-20231212152448075](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202312121524644.png)

## 自定义路由断言工厂

当官方提供的路由断言工厂无法满足业务需求时，可以根据需求自定义路由断言工厂。 

### Auth 认证

当请求头中携带有用户名与密码的 key-value 对，且其用户名与配置文件中 Auth 路由断 言工厂中指定的 username 相同，密码中包含 Auth 路由断言工厂中指定的 password 时才能 通过认证，允许访问系统。

#### 定义Factory

该类类名由两部分构成：后面必须是 RoutePredicateFactory，前面为功能前缀，该前缀 将来要用在配置文件中。

```java
/**
 * Auth 认证
 * @author hjc
 */
@Component
public class AuthRoutePredicateFactory extends AbstractRoutePredicateFactory<AuthRoutePredicateFactory.Config> {

    public AuthRoutePredicateFactory() {
        super(Config.class);
    }

    @Override
    public Predicate<ServerWebExchange> apply(Config config) {
        return exchange -> {
            //获取请求中的所有header
            HttpHeaders headers = exchange.getRequest().getHeaders();
            //一个请求头可以包含多个值
            //获取请求头中username的密码
            List<String> pwds = headers.get(config.getUsername());
            //只要请求头中指定的多个密码中包含了配置文件中指定的密码 即可通过
            if (pwds.contains(config.getPassword())) {
                return true;
            }
            return false;
        };
    }

    /**
     * 读取的配置封装到Config内部类
     */
    @Data
    public static class Config {
        private String username;
        private String password;
    }

    /**
     * 返回关于参数数量和快捷解析顺序
     * @return 关于参数数量和快捷解析顺序
     */
    @Override
    public List<String> shortcutFieldOrder() {
        //与Config内容同名
        return Arrays.asList("username", "password");
    }
}
```

#### 修改配置文件

```yaml
spring:
  cloud:
    gateway:
      routes:        
        - id: baidu_route
          uri: https://www.baidu.com
          predicates:
            - Auth=zhangsan,123456
```

#### 访问

![image-20231213135650469](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202312131356395.png)

### Token 认证

当请求中携带有一个 token 请求参数，且参数值包含配置文件中 Token 路由断言工厂指定的 token 值时才能通过认证，允许访问系统。

#### 定义Factory

该类类名由两部分构成：后面必须是 RoutePredicateFactory，前面为功能前缀，该前缀 将来要用在配置文件中。

```java
@Component
public class TokenRoutePredicateFactory extends AbstractRoutePredicateFactory<TokenRoutePredicateFactory.Config> {
    public TokenRoutePredicateFactory() {
        super(Config.class);
    }

    @Override
    public Predicate<ServerWebExchange> apply(Config config) {
        return exchange ->{
            MultiValueMap<String, String> map = exchange.getRequest().getQueryParams();
            List<String> token = map.get("token");
            if (token.contains(config.getToken())){
                return true;
            }
            return false;
        };
    }

    @Data
    public static class Config {
        private String token;
    }

    @Override
    public List<String> shortcutFieldOrder() {
        return Collections.singletonList("token");
    }
}
```

#### 修改配置文件

```yaml
spring:
  application:
    name: gateway
  cloud:
    gateway:
      routes:
        #TODO 自定义路由断言-token
        - id: baidu_route
          uri: https://www.baidu.com
          predicates:
            - Token=123
```

#### 访问

![image-20240119204516499](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202401192045966.png)

![image-20240119204555713](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202401192045703.png)

## 自定义网关过滤工厂

过滤器分为全局过滤器和局部过滤器：

- 全局过滤器：对所有路由生效。
- 局部过滤器：对指定的路由生效。

### 过滤器执行次序

Spring-Cloud-Gateway 基于过滤器实现，同 zuul 类似，有pre和post两种方式的 filter，分别处理前置逻辑和后置逻辑。客户端的请求先经过pre类型的 filter，然后将请求转发到具体的业务服务，收到业务服务的响应之后，再经过post类型的 filter 处理，最后返回响应到客户端。

**过滤器执行流程**：

![image.png](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303281414860.png)

**注意**：Order的值越小优先级越高，并且无论是在配置文件中编写的单个路由过滤器还是全局路由过滤器，都会受到Order值影响（单个路由的过滤器Order值按从上往下的顺序从1开始递增），最终是按照Order值决定哪个过滤器优先执行，当Order值一样时 全局路由过滤器执行 **优于** 单独的路由过滤器执行。

### 全局过滤器

全局过滤器 Global Filter 是应用于所有路由策略上的 Filter。Spring Cloud Gateway 中已经 定义好了很多 GlobalFilter，但这些 GlobalFilter 无需任何的配置与声明，在符合应用条件时 就会自动生效。

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

### 局部过滤器

定义局部过滤器步骤：

1. 需要实现GatewayFilter, Ordered，实现相关的方法
2. 包装GatewayFilter，产生GatewayFilterFactory
3. GatewayFilterFactory加入到过滤器工厂，并且注册到spring容器中。
4. 在配置文件中进行配置，如果不配置则不启用此过滤器规则。

定义局部过滤器，对于请求头user-id校验，如果不存在user-id请求头，直接返回状态码406：

```java
/**
* 该类类名由两部分构成：后面必须是 GatewayFilterFactory，前面为功能前缀，该前缀将来要用在配置文件中。
*/
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
        - id: my_router
          uri: http://localhost:8080
          predicates:
            - Path=/{segment}
      default-filters:
        - PrefixPath=/info
        - UserIdCheck
```

![image-20240124223628352](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202401242238055.png)

在自定义的 Filter 中为请求添加指定的请求头：

```java
@Component
public class AddHeaderGatewayFilterFactory extends AbstractNameValueGatewayFilterFactory {

    @Override
    public GatewayFilter apply(NameValueConfig config) {
        return (exchange, chain) -> {
            ServerHttpRequest changeRequest = exchange.getRequest()
                    .mutate()
                    .header(config.getName(), config.getValue())
                    .build();
            ServerWebExchange webExchange = exchange.mutate()
                    .request(changeRequest)
                    .build();
            return chain.filter(webExchange);
        };
    }
}
```

```yaml
spring:
  application:
    name: gateway
  cloud:
    gateway:
      routes:
        - id: my_router
          uri: http://localhost:8080
          predicates:
            - Path=/info/{segment}
          filters:
            - AddHeader=new-color,red
```

![image-20240124230418153](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202401242304054.png)

### 多个过滤器

当有多个 Filter时，每个 Filter 都具有 pre 与 post 两部分。

Spring Cloud Gateway 同 zuul 类似，有"pre"和"post"两种方式的 filter。客户端的请求先按照 filter 的优先级顺序 (优先级相同，则按注册顺序)，执行 filter 的"pre"部分。然 后将请求转发到相应的目标服务器，收到目标服务器的响应之后，再按照 filter 的优先级顺 序的逆序 (优先级相同，则按注册顺序逆序)，执行 filter 的“post”部分，最后返回响应到 客户端。

filter 的"pre"部分指的是 `chain.filter()`方法执行之前的代码，而"post"部分，则是定义在 `chain.filter().then()`方法中的代码。

将所有 Filter 注册到路由中，以查看它们执行的顺序。 

 定义第一个 FilterFactory：



过滤器肯定是可以存在很多个的，可以手动指定过滤器之间的顺序：

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

## 熔断降级

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
  port: 9000
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

## 统一跨域请求

为了安全，浏览器中启用了一种称为**同源策略**的安全机制，禁止从一个域名页面中请求另一个域名下的资源。

**跨域请求**：当前发起请求的域与该请求指向的资源所在的域不一样。

**域**：若请求的访问协议 + 域名 + 端口号均相同，那么就是同域。

举个例子：假如一个域名为aaa.cn的网站，它发起一个资源路径为aaa.cn/books/getBookInfo的 Ajax 请求，那么这个请求是同域的，因为资源路径的协议、域名以及端口号与当前域一致（例子中协议名默认为http，端口号默认为80）。但是，如果发起一个资源路径为bbb.com/pay/purchase的 Ajax 请求，那么这个请求就是跨域请求，因为域不一致，与此同时由于安全问题，这种请求会受到同源策略限制。

**例**：

- 源：http://sports.abc.com/content/kb 


- 跨源：https://sports.abc.com/content/kb 


- 跨源：http://news.abc.com/content/kb 


- 跨源：http://sports.abc.com:8080/content/kb 


- 同源：http://sports.abc.com/content/abc

### CORS 

CORS (`Cross-Origin Resource Sharing`，跨域资源共享)，是一种允许当前域的资源(例如， html、js、web service)被其他域的脚本请求访问的机制。

虽然在安全层面上同源限制是必要的，但有时同源策略会对我们的合理用途造成影响，为了避免开发的应用受到限制，有多种方式可以绕开同源策略，常用的做法JSONP, CORS。

### 项目模拟

定义gateway 配置：

```yaml
server:
  port: 9000
spring:
  application:
    name: gateway
  cloud:
    gateway:
      routes:
        - id: my-route
          uri: http://localhost:8080
          predicates:
            - Path=/**
```

在任意工程中定义一个 html 页面，该页面要通过 JS 提交跨域访问请求：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>统一跨域问题</title>
    <script src="https://s3.pstatp.com/cdn/expire-1-M/jquery/3.3.1/jquery.min.js"></script>
    <script>
        function access() {
            $.get("http://localhost:9000/info/header", function (data) {
                alert(data.name)
            })
        }

    </script>
</head>
<body>
    <h1>统一跨域问题</h1>
    <input type="button" value="访问数据" onclick="access()">
</body>
</html>
```

通过IDEA 打开，点击按钮会控制台会报错误：因为浏览器同源策略，就会出现跨域访问问题。

![image-20240123235105197](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202401232351378.png)

![image-20240123235618133](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202401232356934.png)



### 全局解决

修改Gateway工程中的配置文件，在其中添加全局 cors 配置。该解决方案对当前配置文件中的所有路由均起作用。

```yaml
server:
  port: 9000
spring:
  application:
    name: gateway
  cloud:
    gateway:
      globalcors:
        cors-configurations:
          '[/**]':
            allowedOrigins: "*"
            allowedMethods:
              - "GET"
              - "POST"
            allowedHeaders: "*"
            maxAge: 30
      routes:
        - id: my-route
          uri: http://localhost:8080
          predicates:
            - Path=/**
```

此时在浏览器上刷新页面后，再点击按钮即可看到 alert 提示框：

![image-20240124000315121](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202401240003075.png)

### 局部解决

修改Gateway工程中的配置文件，在某个具体的路由中添加局部 cors 配 置。该解决方案仅对当前路由起作用。

```yaml
server:
  port: 9000
spring:
  application:
    name: gateway
  cloud:
    gateway:
      routes:
        - id: my-route
          uri: http://localhost:8080
          predicates:
            - Path=/**
          metadata:
            cors:
              allowedOrigins: "*"
              allowedMethods:
                - "GET"
                - "POST"
              allowedHeaders: "*"
              maxAge: 30
```

此时在浏览器上刷新页面后，再点击按钮即可看到 alert 提示框：

![image-20240124000315121](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202401240003075.png)
