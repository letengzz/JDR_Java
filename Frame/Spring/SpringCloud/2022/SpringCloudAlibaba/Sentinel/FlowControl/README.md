# Sentinel 流量控制

我们的机器不可能无限制的接受和处理客户端的请求，如果不加以限制，当发生高并发情况时，系统资源将很快被耗尽。为了避免这种情况，可以添加流量控制(也可以说是限流)当一段时间内的流量到达一定的阈值的时候，新的请求将不再进行处理，这样不仅可以合理地应对高并发请求，同时也能在一定程度上保护服务器不受到外界的恶意攻击。

Sentinel能够对流量进行控制，主要是监控应用的QPS流量或者并发线程数等指标，如果达到指定的阈值时，就会被流量进行控制，以避免服务被瞬时的高并发流量击垮，保证服务的高可靠性。

## 流量阈值算法

针对于是否超过流量阈值的判断的4种算法：

1. **漏桶算法**
2. **令牌桶算法**
3. **固定时间窗口算法**
4. **滑动时间窗口算法**

### 漏桶算法

顾名思义，就像一个桶开了一个小孔，水流进桶中的速度肯定是远大于水流出桶的速度的，这也是最简单的一种限流思路：

![image-20220327172014949](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202304020130553.png)

桶是有容量的，所以当桶的容量已满时，就装不下水了，这时就只有丢弃请求了。

利用这种思想，可以写出一个简单的限流算法。

### 令牌桶算法

只能说有点像信号量机制。现在有一个令牌桶，这个桶是专门存放令牌的，每隔一段时间就向桶中丢入一个令牌（速度由我们指定）当新的请求到达时，将从桶中删除令牌，接着请求就可以通过并给到服务，但是如果桶中的令牌数量不足，那么不会删除令牌，而是让此数据包等待。

![image-20220327173323339](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202304020130381.png)

可以试想一下，当流量下降时，令牌桶中的令牌会逐渐积累，这样如果突然出现高并发，那么就能在短时间内拿到大量的令牌。

### 固定时间窗口算法

可以对某一个时间段内的请求进行统计和计数，比如在`14:15`到`14:16`这一分钟内，请求量不能超过`100`，也就是一分钟之内不能超过`100`次请求，那么就可以像下面这样进行划分：

![image-20220327174027199](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202304020130019.png)

虽然这种模式看似比较合理，但是试想一下这种情况：

- 14:15:59的时候来了100个请求
- 14:16:01的时候又来了100个请求

出现上面这种情况，符合固定时间窗口算法的规则，所以这200个请求都能正常接受，但是，如果你反应比较快，应该发现了，其实希望的是60秒内只有100个请求，但是这种情况却是在3秒内出现了200个请求，很明显已经违背了初衷。因此，当遇到临界点时，固定时间窗口算法存在安全隐患。

### 滑动时间窗口算法

相对于固定窗口算法，滑动时间窗口算法更加灵活，它会动态移动窗口，重新进行计算：

![image-20220327174906227](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202304020130632.png)

虽然这样能够避免固定时间窗口的临界问题，但是这样显然是比固定窗口更加耗时的。

## 限流策略

Sentinel 实现限流采取策略的方式：

1. **快速拒绝**：既然不再接受新的请求，那么可以直接返回一个拒绝信息，告诉用户访问频率过高。
2. **预热**：依然基于方案一，但是由于某些情况下高并发请求是在某一时刻突然到来，可以缓慢地将阈值提高到指定阈值，形成一个缓冲保护。
3. **排队等待**：不接受新的请求，但是也不直接拒绝，而是进队列先等一下，如果规定时间内能够执行，那么就执行，要是超时就算了。

## 流控设置

Sentinel能够对流量进行控制，主要是监控应用的QPS流量或者并发线程数等指标，如果达到指定的阈值时，就会被流量进行控制，以避免服务被瞬时的高并发流量击垮，保证服务的高可靠性。

**创建流控规则**：

![image-20240415212459527](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202404152125250.png)

**通过簇点链路创建流控规则**：

![image-20240427135615579](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202404271356577.png)

**参数说明**：

- **资源名**：资源的唯一名称，默认就是请求的接口路径，可以自行修改，但是要**保证唯一**
- **针对来源**：具体针对某个微服务进行限流，默认值为default，表示不区分来源，全部限流
- **阈值类型**：
  - QPS：每秒钟的请求数量，通过QPS进行限流
  - 并发线程数：按服务当前使用的线程数据进行统计的，通过并发线程数限流
- **单机阈值**：与阈值类型组合使用
  - 如果阈值类型选择QPS，表示当调用接口的QPS达到阈值时，进行限流操作
  - 如果阈值类型选择并发线程数，表示当调用接口的并发线程数达到阈值时，进行限流操作
- **是否集群**：选中则表示集群环境，不选中则表示非集群环境
- **流控效果**：**快速拒绝**、**预热**、**排队等待**

## 流控模式

**流控模式的区别**：

- 直接：只针对于当前接口。
- 关联：当其他接口超过阈值时，会导致当前接口被限流。
- 链路：更细粒度的限流，能精确到具体的方法。

### 直接流控模式

**默认的流控模式**，当接口达到限流条件时，直接开启限流功能。

当选择**QPS**、阈值设定为**1**，流控模式选择**直接**、流控效果选择**快速失败**，表示1秒钟内查询1次就是OK，若超过次数1，就直接快速失败，报默认错误。当快速地进行请求时，会直接返回失败信息：

![image-20240427135659415](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202404271357032.png)

![image-20240427135748777](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202404271357668.png)

### 关联流控模式

当关联的资源达到阈值时，一旦发现关联资源超过阈值，那么就会对当前的资源进行限流。

对自带的`/error`接口进行限流：

![image-20240427135927315](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202404271359857.png)

测试：使用JMeter连续对关联资源发起请求：

![image-20240427150314922](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202404271503920.png)

开启JMeter，然后会发现服务已经出错：

![image-20240427135748777](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202404271357668.png)

当关闭掉JMeter的任务后，恢复正常。

### 链路流控模式

链路模式能够更加精准的进行流量控制，链路流控模式指的是，来自不同链路的请求对同一个目标访问时，实施针对性的不同限流措施，从指定接口过来的资源请求达到限流条件时，开启限流。

对某一个方法进行限流控制，无论是谁在何处调用了它，这里需要使用到[`@SentinelResource`](../SentinelResource/README.md)，一旦方法被标注，那么就会进行监控，比如创建两个请求映射，都来调用Service的被监控方法：

> BookController.java

```java
@RestController
public class BookController {
    @Resource
    BookService service;

    //这里以RESTFul风格为例
    @RequestMapping("/book/{uid}")
    public Book findUserById(@PathVariable("uid") Integer uid){
        return service.getBookById(uid);
    }

    @RequestMapping("/book2/{uid}")
    public Book findUserById2(@PathVariable("uid") Integer uid){
        return service.getBookById(uid);
    }
}
```

> BookServiceImpl.java

```java
@Service
public class BookServiceImpl implements BookService {
    @Resource
    private BookMapper bookMapper;
    @Override
    @SentinelResource("getBook")   //监控此方法，无论被谁执行都在监控范围内，这里给的value是自定义名称，这个注解可以加在任何方法上，包括Controller中的请求映射方法
    public Book getBookById(Integer bid) {
        return bookMapper.getBookById(bid);
    }
}
```

添加配置：

```yaml
spring:
  application:
    name: book-service
  cloud:
    sentinel:
      transport:
        dashboard: localhost:8858
      # 关闭Context收敛，这样被监控方法可以进行不同链路的单独控制 
      # controller层的方法对service层调用不认为是同一个根链路
      web-context-unify: false
```

在Sentinel控制台中添加流控规则，注意是针对此方法，可以看到已经自动识别到book接口下调用了这个方法：

![image-20240427140728918](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202404271407339.png)

在浏览器中对这两个接口都进行测试，会发现，无论请求哪个接口，只要调用了Service中的`getBookById`这个方法，都会被限流。注意限流的形式是后台直接抛出异常。

![image-20240427140918887](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202404271409385.png)

这个链路选项实际上就是决定只限流从哪个方向来的调用，比如只对`book2`这个接口对`getBookById`方法的调用进行限流，那么可以为其指定链路：

![image-20240427141342747](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202404271413138.png)

限流效果只对配置的链路接口有效，而其他链路是不会被限流的。

![image-20240427141424052](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202404271414584.png)

![image-20240427141406883](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202404271414417.png)

## 流控效果

### 快速失败

**默认的流控处理**，会直接失败，抛出异常：`Blocked by Sentinel (flow limiting)`

当选择**QPS**、阈值设定为**1**，流控模式选择**直接**、流控效果选择**快速失败**，表示1秒钟内查询1次就是OK，若超过次数1，就直接快速失败，报默认错误。当快速地进行请求时，会直接返回失败信息：

![image-20240427135659415](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202404271357032.png)

![image-20240427135748777](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202404271357668.png)

### 预热

官方文档：

- 流量控制：https://github.com/alibaba/Sentinel/wiki/%E6%B5%81%E9%87%8F%E6%8E%A7%E5%88%B6
- 限流 冷启动：https://github.com/alibaba/Sentinel/wiki/%E9%99%90%E6%B5%81---%E5%86%B7%E5%90%AF%E5%8A%A8

**公式**：阈值除以冷却因子coldFactor(默认值为3)，经过预热时长后才会达到阈值。即请求QPS从`threshold/3`开始，经预热逐渐升至设定的QPS阈值。

**源码**：`com.alibaba.csp.sentinel.slots.block.flow.controller.WarmUpController`

![image-20240416131050798](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202404161310858.png)

**应用场景**：秒杀系统在开启的瞬间，会有很多流量上来，很有可能把系统打死，预热方式就是把为了保护系统，可慢慢的把流量放进来，慢慢的把阈值增长到设置的阈值。

当选择**QPS**、阈值设定为**10** (系统初始化的阈值为10 / 3 约等于3,即单机阈值刚开始为3，Sentinel计算后QPS判定为3开始)，流控模式选择**直接**、流控效果选择**Warm Up**，预热时长设定为5，可以看到，从刚开始时，直接返回失败信息，过了5秒后阀值才慢慢升高恢复到设置的单机阈值10，也就是说5秒钟内QPS为3，过了保护期5秒后QPS为10，请求没超过十次时，会正常响应：

![image-20240427141647538](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202404271416508.png)

### 排队等待

![](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202404161345190.png)

添加请求：

```java
@GetMapping("/test1")
public String test1()
{
    System.out.println(System.currentTimeMillis()+"      test,排队等待");
    return "------test";
}
```

当选择**QPS**、阈值设定为**1**，流控模式选择**直接**、流控效果选择**排队等待**，超时时间10000ms，等待10000ms后，等待到就会执行：

![image-20240427141920609](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202404271419465.png)

一秒钟通过一个请求，10秒后的请求作为超时处理，放弃：

![image-20240427150858081](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202404271508913.png)

![image-20240427150800883](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202404271508471.png)

## 并发线程数

Sentinel配置：

![image-20240427142409318](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202404271424053.png)

测试：

![image-20240427151109644](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202404271511718.png)

![image-20240427151126672](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202404271511504.png)

## 系统规则设置

除了直接对接口进行限流规则控制之外，可以根据当前系统的资源使用情况，决定是否进行限流。

系统规则支持以下的模式：

- **Load 自适应**(仅对 Linux/Unix-like 机器生效)：系统的 load1 作为启发指标，进行自适应系统保护。当系统 load1 超过设定的启发值，且系统当前的并发线程数超过估算的系统容量时才会触发系统保护（BBR 阶段）。系统容量由系统的 `maxQps * minRt` 估算得出。设定参考值一般是 `CPU cores * 2.5`。
- **CPU usage**(1.5.0+ 版本)：当系统 CPU 使用率超过阈值即触发系统保护(取值范围 0.0-1.0)，比较灵敏。
- **平均 RT**：当单台机器上所有入口流量的平均 RT 达到阈值即触发系统保护，单位是毫秒。
- **并发线程数**：当单台机器上所有入口流量的并发线程数达到阈值即触发系统保护。
- **入口 QPS**：当单台机器上所有入口流量的 QPS 达到阈值即触发系统保护。

![image-20220328235217680](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202304020129004.png)