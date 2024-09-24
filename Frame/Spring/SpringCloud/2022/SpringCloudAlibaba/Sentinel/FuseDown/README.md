# Sentinel熔断和降级

在整个微服务调用链路出现问题的时候，及时对服务进行降级，以防止问题进一步恶化。

![image-20220324141706946](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202304020128684.png)

如果在某一时刻，服务B出现故障（可能就卡在那里了），而这时服务A依然有大量的请求，在调用服务B，那么，由于服务A没办法再短时间内完成处理，新来的请求就会导致线程数不断地增加，这样，CPU的资源很快就会被耗尽。那么要防止这种情况，就只能进行隔离了。

Sentinel 熔断降级会在调用链路中某个资源出现不稳定状态时（例如调用超时或异常比例升高），对这个资源的调用进行限制，让请求快速失败，避免影响到其它的资源而导致级联错误。当资源被降级后，在接下来的降级时间窗口之内，对该资源的调用都自动熔断（默认行为是抛出 DegradeException）。

Sentinel采用的[信号量隔离](../../../Expand/Tolerant/isolation/README.md#信号量隔离)实现隔离的。

**官方教程**：https://github.com/alibaba/Sentinel/wiki/%E7%86%94%E6%96%AD%E9%99%8D%E7%BA%A7

![image-20240418212732900](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202404182127360.png)

## 新建熔断规则

打开管理页面，可以自由新增熔断规则：

![image-20230401174556756](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202304020128725.png)

## 熔断策略

### 慢调用比例

如果出现那种半天都处理不完的调用，有可能就是服务出现故障，导致卡顿，这个选项是按照最大响应时间(RT)进行判定，如果一次请求的处理时间超过了指定的RT，那么就被判定为**慢调用**，在一个统计时长内，如果请求数目大于最小请求数目，并且被判定为慢调用的请求比例已经超过阈值，将触发熔断。经过熔断时长之后，将会进入到半开状态进行试探

**进入熔断状态判断依据**：在统计时长内，实际请求数目＞设定的最小请求数  且   实际慢调用比例＞比例阈值 ，进入熔断状态。

**参数说明**：

1. **调用**：一个请求发送到服务器，服务器给与响应，一个响应就是一个调用。

2. **最大RT**：即最大的响应时间，指系统对请求作出响应的业务处理时间。

3. **慢调用**：处理业务逻辑的实际时间>设置的最大RT时间，这个调用叫做慢调用。

4. **慢调用比例**：在所以调用中，慢调用占有实际的比例＝慢调用次数➗总调用次数

5. **比例阈值**：自己设定的 ， 比例阈值＝慢调用次数➗调用次数

6. **统计时长**：时间的判断依据

7. **最小请求数**：设置的调用最小请求数

**过程**：

1. 熔断状态(保险丝跳闸断电，不可访问)：在接下来的熔断时长内请求会自动被熔断

2. 探测恢复状态(探路先锋)：熔断时长结束后进入探测恢复状态

3. 结束熔断(保险丝闭合恢复，可以访问)：在探测恢复状态，如果接下来的一个请求响应时间小于设置的慢调用 RT，则结束熔断，否则继续熔断。

![](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202404182229414.png)

模拟慢调用：10个线程，在一秒的时间内发送完。又因为服务器响应时长设置：暂停1秒，所以响应一个请求的时长都大于1秒综上符合熔断条件，所以当线程开启1秒后，进入熔断状态

```java
@RequestMapping("/test2")
public String test2() throws InterruptedException {
    //暂停几秒钟线程
    try {
        TimeUnit.SECONDS.sleep(1);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
    return "hello";
}
```

重启，创建一个新的熔断规则：

![image-20240427153029409](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202404271530082.png)

![image-20240427153235352](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202404271532752.png)

可以看到，多次循环，一秒钟打进来10个线程(大于5个了)调用，希望200毫秒处理完一次调用，超时直接触发了熔断，进入到阻止页面：

![image-20240427153313899](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202404271533485.png)

**假如在统计时长内，实际请求数目＞最小请求数且慢调用比例＞比例阈值 ，断路器打开(保险丝跳闸)微服务不可用(Blocked by Sentinel (flow limiting))，进入熔断状态5秒；**后续停止，没有这么大的访问量了，单独用浏览器访问rest地址，断路器关闭(保险丝恢复，合上闸口)，微服务恢复

### 异常比例

这个与慢调用比例类似，不过这里判断的是出现异常的次数

![](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202404182232400.png)

测试：

```java
@RequestMapping("/test3")
public String test3() {
    throw new RuntimeException();
}
```

![image-20240427153343432](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202404271533880.png)

启动服务器，接着添加熔断规则：

![image-20240427153553264](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202404271535603.png)

![image-20240427153644132](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202404271536430.png)

进行访问，会发现后台疯狂报错，断路器开启(保险丝跳闸)，微服务不可用了，不再报错error而是服务熔断+服务降级，出提示：

![image-20240427153824793](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202404271538287.png)

![image-20240427153917888](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202404271539927.png)

### 异常数

这个和上面的唯一区别就是，只要达到指定的异常数量，就熔断，修改熔断规则：

![image-20240427154550614](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202404271545929.png)

现在再次不断访问此接口，可以发现，效果跟之前其实是差不多的，只是判断的策略稍微不同罢了：

![image-20240427153917888](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202404271539927.png)

## 自定义服务降级

自定义服务降级只需要在`@SentinelResource`中配置`blockHandler`参数（因为如果添加了`@SentinelResource`注解，那么这里会进行方法级别细粒度的限制，和之前方法级别限流一样，会在降级之后直接抛出异常，如果不添加则返回默认的限流页面，`blockHandler`的目的就是处理这种Sentinel机制上的异常，所以这里其实和之前的限流配置是一个道理，因此熔断配置也应该对`value`自定义名称的资源进行配置，才能作用到此方法上）：

```java
@RequestMapping("/test4/{bid}")
@SentinelResource(value = "test4", blockHandler = "test4BlockHandler")  //指定blockHandler，也就是被限流之后的替代解决方案，这样就不会使用默认的抛出异常的形式了
public String test4(@PathVariable("bid") Integer bid) {
    throw new RuntimeException();
}
//替代方案，注意参数和返回值需要保持一致，并且参数最后还需要额外添加一个BlockException
public String test4BlockHandler(@PathVariable("bid") Integer bid,BlockException e){
    e.printStackTrace();
    return "熔断降级了....";
}
```

接着对进行熔断配置，注意是对添加的`@SentinelResource`中指定名称的`test4`进行配置：

![image-20240427155635652](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202404271556647.png)

OK，可以看到熔断之后，服务降级之后的效果：

![image-20240427155707819](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202404271557014.png)

