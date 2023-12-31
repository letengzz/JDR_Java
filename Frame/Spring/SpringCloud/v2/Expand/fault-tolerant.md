# 微服务容错

在高并发访问下，比如天猫双11，流量持续不断的涌入，服务之间的相互调用频率突然增加，引发系统负载过高，这时系统所依赖的服务的稳定性对系统的影响非常大，而且还有很多不确定因素引起雪崩，如网络连接中断，服务宕机等。一般微服务容错组件提供了限流、隔离、降级、熔断等手段，可以有效保护我们的微服务系统。

## 隔离 

微服务系统A调用B，而B调用C，这时如果C出现故障，则此时调用B的大量线程资源阻塞，慢慢的B的线程数量持续增加直到CPU耗尽到100%，整体微服务不可用，这时就需要对不可用的服务进行隔离。

服务调用关系：

![image.png](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303301456380.png)

### 线程池隔离

线程池隔离是通过Java的线程池进行隔离，B服务调用C服务给予固定的线程数量比如12个线程，如果此时C服务宕机了就算大量的请求过来，调用C服务的接口只会占用12个线程不会占用其他工作线程资源，因此B服务就不会出现级联故障。

线程池隔离实际上就是对每个服务的远程调用单独开放线程池，只基于固定数量的线程池，这样即使在短时间内出现大量请求，由于没有线程可以分配，所以就不会导致资源耗尽了。

**线程池隔离原理**：

![image-20220328121932455](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303311908505.png)

### 信号量隔离

信号量隔离是使用Semaphore类实现的，思想基本上与上面是相同的，也是限定指定的线程数量能够同时进行服务调用，当拿不到信号量的时候直接拒接因此不会出现超时占用其他工作线程的情况。但是它相对于线程池隔离，开销会更小一些，使用效果同样优秀，也支持超时等。

```java
Semaphore semaphore = new Semaphore(10,true);  
//获取信号量  
semaphore.acquire();  
//do something here  
//释放信号量  
semaphore.release();  
```

### 线程池隔离和信号量隔离的区别

线程池隔离针对不同的资源分别创建不同的线程池，不同服务调用都发生在不同的线程池中，在线程池排队、超时等阻塞情况时可以快速失败。线程池隔离的好处是隔离度比较高，可以针对某个资源的线程池去进行处理而不影响其它资源，但是代价就是线程上下文切换的 overhead 比较大，特别是对低延时的调用有比较大的影响。而信号量隔离非常轻量级，仅限制对某个资源调用的并发数，而不是显式地去创建线程池，所以 overhead 比较小，但是效果不错，也支持超时失败。

二者区别：

| 类别         | 线程池隔离                               | 信号量隔离             |
| ------------ | ---------------------------------------- | ---------------------- |
| 线程         | 与调用线程不同，使用的是线程池创建的线程 | 与调用线程相同         |
| 开销         | 排队，切换，调度等开销                   | 无线程切换性能更高     |
| 是否支持异步 | 支持                                     | 不支持                 |
| 是否支持超时 | 支持超时                                 | 支持超时               |
| 并发支持     | 支持通过线程池大小控制                   | 支持通过最大信号量控制 |

## 熔断 

当下游的服务因为某种原因突然变得不可用或响应过慢，上游服务为了保证自己整体服务的可用性，不再继续调用目标服务，直接返回，快速释放资源。如果目标服务情况好转则恢复调用。

**熔断器模型**：

![image.png](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303301456720.png)

熔断器模型的状态机有3个状态：

- **关闭状态**(断路器关闭，Closed)：所有请求都正常访问。

- **打开状态**(断路器打开，Open)：所有请求都会被降级。熔断器会对请求情况计数，当一定时间内失败请求百分比达到阈值，则触发熔断，断路器会完全打开。

- **半开状态**(Half Open)：不是永久的，断路器打开后会进入休眠时间。随后断路器会自动进入半开状态。此时会释放部分请求通过，若这些请求都是健康的，则会关闭断路器，否则继续保持打开，再次进行休眠计时。 

## 降级 

降级是指当自身服务压力增大时，系统将某些不重要的业务或接口的功能降低，可以只提供部分功能，也可以完全停止所有不重要的功能。比如，下线非核心服务以保证核心服务的稳定、降低实时性、降低数据一致性，降级的思想是丢车保帅。

举个例子，比如，目前很多人想要下订单，但是我的服务器除了处理下订单业务之外，还有一些其他的服务在运行，比如，搜索、定时任务、支付、商品详情、日志等等服务。然而这些不重要的服务占用了JVM的不少内存和CPU资源，为了应对很多人要下订单的需求，设计了一个动态开关，把这些不重要的服务直接在最外层拒绝掉。这样就有跟多的资源来处理下订单服务（下订单速度更快了）

## 限流 

限流，就是限制最大流量。系统能提供的最大并发有限，同时来的请求又太多，就需要限流，比如商城秒杀业务，瞬时大量请求涌入，服务器服务不过来，就只好排队限流了，就跟去景点排队买票和去银行办理业务排队等号道理相同。下面介绍下四种常见的限流算法。

### 漏桶算法

漏桶算法的思路，一个固定容量的漏桶，按照常量固定速率流出水滴。如果桶是空的，则不需流出水滴。可以以任意速率流入水滴到漏桶。如果流入水滴超出了桶的容量，则流入的水滴溢出了（被丢弃），而漏桶容量是不变的。

漏桶限流原理：

![4-4漏桶限流.png](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303301455743.png)

### 令牌桶算法

令牌桶算法：假设限制2r/s，则按照500毫秒的固定速率往桶中添加令牌。桶中最多存放b个令牌，当桶满时，新添加的令牌被丢弃或拒绝。当一个n个字节大小的数据包到达，将从桶中删除n个令牌，接着数据包被发送到网络上。如果桶中的令牌不足n个，则不会删除令牌，且该数据包将被限流（要么丢弃，要么缓冲区等待）。

令牌桶限流原理：

![4-5令牌桶.png](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303301456243.png)

令牌桶限流服务器端可以根据实际服务性能和时间段改变生成令牌的速度和水桶的容量。 一旦需要提高速率,则按需提高放入桶中的令牌的速率。

生成令牌的速度是恒定的，而请求去拿令牌是没有速度限制的。这意味着当面对瞬时大流量，该算法可以在短时间内请求拿到大量令牌，而且拿令牌的过程并不是消耗很大。

### 固定时间窗口算法

在固定的时间窗口内，可以允许固定数量的请求进入。超过数量就拒绝或者排队，等下一个时间段进入。这种实现计数器限流方式由于是在一个时间间隔内进行限制，如果用户在上个时间间隔结束前请求（但没有超过限制），同时在当前时间间隔刚开始请求（同样没超过限制），在各自的时间间隔内，这些请求都是正常的，但是将间隔临界的一段时间内的请求就会超过系统限制，可能导致系统被压垮。

固定时间窗口算法原理：

![4-6固定时间窗口.png](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303301456023.png)

由于计数器算法存在时间临界点缺陷，因此在时间临界点左右的极短时间段内容易遭到攻击。比如设定每分钟最多可以请求100次某个接口，如12:00:00-12:00:59时间段内没有数据请求，而12:00:59-12:01:00时间段内突然并发100次请求，而紧接着跨入下一个计数周期，计数器清零，在12:01:00-12:01:01内又有100次请求。那么也就是说在时间临界点左右可能同时有2倍的阀值进行请求，从而造成后台处理请求过载的情况，导致系统运营能力不足，甚至导致系统崩溃。

### 滑动时间窗口算法

滑动窗口算法是把固定时间片进行划分，并且随着时间移动，移动方式为开始时间点变为时间列表中的第二时间点，结束时间点增加一个时间点，不断重复，通过这种方式可以巧妙的避开计数器的临界点的问题。

滑动窗口算法可以有效的规避计数器算法中时间临界点的问题，但是仍然存在时间片段的概念。同时滑动窗口算法计数运算也相对固定时间窗口算法比较耗时。

滑动时间窗口算法：

![4-6滑动时间窗口.png](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303301456585.png)