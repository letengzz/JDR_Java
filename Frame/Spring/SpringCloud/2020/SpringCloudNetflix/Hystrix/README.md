# Hystrix 服务熔断

微服务之间是可以进行相互调用的，如果出现问题：

![image-20220324141230070](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303271925828.png)

由于位于最底端的服务提供者E发生故障，那么此时会直接导致服务ABCD全线崩溃，就像雪崩了一样。

![image-20220324141706946](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303271925522.png)

这种问题实际上是不可避免的，由于多种因素，比如网络卡顿、系统故障、硬件问题等，都存在一定可能，会导致这种极端的情况发生。因此，我们需要寻找一个应对这种极端情况的解决方案。

为了解决分布式系统的雪崩问题，SpringCloud提供了Hystrix熔断器组件，保证在不断地有大量的请求到达，需要各个服务进行处理，在一个服务故障时，不会导致整条链路上的服务全线崩溃。

**说明**：一定要区分开服务降级和服务熔断的区别，服务降级并不会直接返回错误，而是可以提供一个补救措施，正常响应给请求者。这样相当于服务依然可用，但是服务能力肯定是下降了的。

官方文档：https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/1.3.5.RELEASE/single/spring-cloud-netflix.html#_circuit_breaker_hystrix_clients

**注意**：Netflix的Hystrix微服务容错库已经停止更新，官方推荐使用Resilience4j代替Hystrix，或者使用Spring Cloud Alibaba的[Sentinel](../SpringCloudAlibaba/Sentinel.md)组件。

## 服务降级

通过不开启用户服务和图书服务，模拟用户服务和图书服务已经挂掉了。

导入Hystrix的依赖 (此项目已经停止维护，SpringCloud依赖中已经不自带了，所以说需要自己单独导入)：

```xml
<dependency>
	<groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
    <version>2.2.10.RELEASE</version>
</dependency>
```

在启动类添加注解开启：

> BorrowApplication.java

```java
@SpringBootApplication
@EnableHystrix   //启用Hystrix
public class BorrowApplication {
    public static void main(String[] args) {
        SpringApplication.run(BorrowApplication.class, args);
    }
}
```

由于用户服务和图书服务不可用，所以查询借阅信息的请求肯定是没办法正常响应的，可以提供一个备选方案，也就是说当服务出现异常时，返回我们的备选方案：

> BorrowController.java

```java
@RestController
public class BorrowController {

    @Resource
    BorrowService service;

    @HystrixCommand(fallbackMethod = "onError")    //使用@HystrixCommand来指定备选方案
    @RequestMapping("/borrow/{uid}")
    UserBorrowDetail findUserBorrows(@PathVariable("uid") Integer uid){
        return service.getUserBorrowDetailByUid(uid);
    }
		
  	//备选方案，这里直接返回空列表了
  	//注意参数和返回值要和上面的一致
    UserBorrowDetail onError(Integer uid){
        return new UserBorrowDetail(null, Collections.emptyList());
    }
}
```

可以看到，虽然服务无法正常运行了，但是依然可以给浏览器正常返回响应数据：

![image-20230311150105347](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303271925699.png)

![image-20230311150155274](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303271925723.png)

服务降级是一种比较温柔的解决方案，虽然服务本身的不可用，但是能够保证正常响应数据。

## 服务熔断

熔断机制是应对雪崩效应的一种微服务链路保护机制，当检测出链路的某个微服务不可用或者响应时间太长时，会进行服务的降级，进而熔断该节点微服务的调用，快速返回”错误”的响应信息。当检测到该节点微服务响应正常后恢复调用链路。

实际上，熔断就是在降级的基础上进一步升级形成的，也就是说，在一段时间内多次调用失败，那么就直接升级为熔断。

添加两条输出语句：

```java
@RestController
public class BorrowController {

    @Resource
    BorrowService service;

    @HystrixCommand(fallbackMethod = "onError")
    @RequestMapping("/borrow/{uid}")
    UserBorrowDetail findUserBorrows(@PathVariable("uid") Integer uid){
        System.out.println("开始向其他服务获取信息");
        return service.getUserBorrowDetailByUid(uid);
    }

    UserBorrowDetail onError(Integer uid){
        System.out.println("服务错误，进入备选方法！");
        return new UserBorrowDetail(null, Collections.emptyList());
    }
}
```

接着，我们在浏览器中疯狂点击刷新按钮，对此服务疯狂发起请求，可以看到后台：

![image-20230311150748619](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303271924461.png)

一开始的时候，会正常地去调用Controller对应的方法`findUserBorrows`，发现失败然后进入备选方法，但是我们发现在持续请求一段时间之后，没有再调用这个方法，而是直接调用备选方案，这便是升级到了熔断状态。

我们可以继续不断点击，继续不断地发起请求：

![image-20230311150902750](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303271926087.png)

可以看到，过了一段时间之后，会尝试正常执行一次`findUserBorrows`，但是依然是失败状态，所以继续保持熔断状态。

它能够对一段时间内出现的错误进行侦测，当侦测到出错次数过多时，熔断器会打开，所有的请求会直接响应失败，一段时间后，只执行一定数量的请求，如果还是出现错误，那么则继续保持打开状态，否则说明服务恢复正常运行，关闭熔断器。

我们可以测试一下，开启另外两个服务之后，继续点击：

![image-20220324153044583](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303271924314.png)

可以看到，当另外两个服务正常运行之后，当再次尝试调用`findUserBorrows`之后会成功，于是熔断机制就关闭了，服务恢复运行：

![image-20220324153935858](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303271926375.png)

## 监控页面部署

除了对服务的降级和熔断处理，我们也可以对其进行实时监控，只需要安装监控页面即可。

创建一个新的项目，导入依赖：

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-hystrix-dashboard</artifactId>
    <version>2.2.10.RELEASE</version>
</dependency>
```

接着添加配置文件：

```yaml
server:
  port: 8900
hystrix:
  dashboard:
    # 将localhost添加到白名单，默认是不允许的
    proxy-stream-allow-list: "localhost"
```

接着创建主类，注意需要添加`@EnableHystrixDashboard`注解开启管理页面：

```java
@SpringBootApplication
@EnableHystrixDashboard
public class HystrixDashBoardApplication {
    public static void main(String[] args) {
        SpringApplication.run(HystrixDashBoardApplication.class, args);
    }
}
```

启动Hystrix管理页面服务，需要在要进行监控的服务中添加Actuator依赖：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

> Actuator是SpringBoot程序的监控系统，可以实现健康检查，记录信息等。在使用之前需要引入spring-boot-starter-actuator，并做简单的配置即可。

添加此依赖后，可以在IDEA中查看运行情况：

![image-20230311153813375](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303271924823.png)

然后在配置文件中配置Actuator添加暴露：

```yaml
management:
  endpoints:
    web:
      exposure:
        include: '*'
```

打开刚刚启动的管理页面地址：http://localhost:8900/hystrix/

![image-20230311153728522](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303271924657.png)

在中间填写要监控的服务：比如借阅服务：http://localhost:8082/actuator/hystrix.stream，注意后面要添加`/actuator/hystrix.stream`，然后点击Monitor Stream即可进入监控页面：

![image-20230311154058682](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202307072359061.png)

可以看到现在都是Loading状态，这是因为还没有开始统计，我们现在尝试调用几次我们的服务：

![image-20230311154203792](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303271924666.png)

可以看到，在调用之后，监控页面出现了信息：

![image-20230311154343079](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303271924724.png)

可以看到13次访问都是正常的，所以显示为绿色，接着尝试将图书服务关闭，这样就会导致服务降级甚至熔断，然后再多次访问此服务看看监控会如何变化：

![image-20230311154717908](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303271924259.png)

可以看到，错误率直接飙升到100%，并且一段时间内持续出现错误，中心的圆圈也变成了红色，我们继续进行访问：

![image-20230311154752061](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303271924558.png)

在出现大量错误的情况下保持持续访问，可以看到此时已经将服务熔断，`Circuit`更改为Open状态，并且图中的圆圈也变得更大，表示压力在持续上升。