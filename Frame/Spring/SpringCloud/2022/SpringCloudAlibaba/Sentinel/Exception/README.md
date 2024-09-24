# 异常处理

针对限流状态下的返回结果修改：

先创建好被限流状态下需要返回的内容，定义一个请求映射：

> BookController.java

```java
@RequestMapping("/blocked")
JSONObject blocked(){
    JSONObject object = new JSONObject();
    object.put("code", 403);
    object.put("success", false);
    object.put("massage", "您的请求频率过快，请稍后再试！");
    return object;
}
```

在配置文件中将此页面设定为限流页面：

```yaml
spring:
  cloud:
    sentinel:
      transport:
        dashboard: localhost:8858
      # 将刚刚编写的请求映射设定为限流页面
      block-page: /blocked
```

当被限流时，就会被重定向到指定页面：

![image-20240427155927746](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202404271559274.png)

当某个方法被限流时，会直接在后台抛出异常，可以添加一个替代方案，出现异常时会直接执行替代方法并返回。

在BookController上进行配置：

> BookController.java

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

一旦被限流将执行替代方案，返回的结果：

![image-20240427155707819](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202404271557014.png)

**注意**：blockHandler只能处理限流情况下抛出的异常，包括热点参数限流也是同理，如果是方法本身抛出的其他类型异常，不在管控范围内，但是可以通过其他参数进行处理：

```java
@RequestMapping("/test5")
@SentinelResource(value = "test5",
        fallback = "except",    //fallback指定出现异常时的替代方案
        exceptionsToIgnore = IOException.class)  //忽略那些异常，也就是说这些异常出现时不使用替代方案
public String test(){
    throw new RuntimeException("HelloWorld！");
}

//替代方法必须和原方法返回值和参数一致，最后可以添加一个Throwable作为参数接受异常
public String except(Throwable t){
    return t.getMessage();
}
```

这样，其他的异常也可以有替代方案了：

![image-20240427160831102](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202404271608423.png)

**注意**：这种方式会在没有配置`blockHandler`的情况下，将Sentinel机制内（也就是限流的异常）的异常也一并处理了，如果配置了`blockHandler`，那么在出现限流时，依然只会执行`blockHandler`指定的替代方案（因为限流是在方法执行之前进行的）

```java
@RequestMapping("/test6/{id}")
@SentinelResource(value = "test6",
        fallback = "except",    //fallback指定出现异常时的替代方案
        blockHandler = "block",
        exceptionsToIgnore = IOException.class)  //忽略那些异常，也就是说这些异常出现时不使用替代方案
public String test(@PathVariable("id") Integer id) {
    if(id == 0){
        throw new RuntimeException("HelloWorld！");
    }
    return "test";
}

//替代方法必须和原方法返回值和参数一致，最后可以添加一个Throwable作为参数接受异常
public String except(@PathVariable("id") Integer id,Throwable t) {
    return "异常了...";
}

public String block(@PathVariable("id") Integer id,BlockException b) {
    return "限流了...";
}
```

![image-20240427161234302](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202404271612808.png)

![image-20240427161121580](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202404271611191.png)

![image-20240427161134570](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202404271611342.png)

