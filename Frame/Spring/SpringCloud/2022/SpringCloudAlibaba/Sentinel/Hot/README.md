# 热点规则限流

热点即经常访问的数据，很多时候希望统计或者限制某个热点数据中访问频次最高的TopN数据，并对其访问进行限流或者其它操作。

![](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202404252119184.png)

**官方文档**：https://github.com/alibaba/Sentinel/wiki/%E7%83%AD%E7%82%B9%E5%8F%82%E6%95%B0%E9%99%90%E6%B5%81

可以对某一热点数据进行精准限流，比如在某一时刻，不同参数被携带访问的频率是不一样的：

- http://localhost:8201/test?a=10 访问100次
- http://localhost:8201/test?b=10 访问0次
- http://localhost:8201/test?c=10 访问3次

由于携带参数`a`的请求比较多，可以只对携带参数`a`的请求进行限流。

创建一个新的测试请求映射：

> BookController.java

```java
@RequestMapping("/hot-test")
@SentinelResource(value = "hotTest",fallback = "dealHandler")   //注意这里需要添加@SentinelResource才可以，用户资源名称就使用这里定义的资源名称
String findUserBorrows2(@RequestParam(value = "a", required = false) Integer a,
                        @RequestParam(value = "b", required = false) Integer b,
                        @RequestParam(value = "c",required = false) Integer c) {
    return "请求成功！a = "+a+", b = "+b+", c = "+c;
}
    
public String dealHandler(Integer a,Integer b,Integer c,Throwable throwable){
	return "请求失败！a = "+a+", b = "+b+", c = "+c+"，异常信息："+throwable.getMessage();
}
```

启动之后，在Sentinel里面进行热点配置：

![image-20240427162513806](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202404271625283.png)

然后开始访问测试接口，可以看到在携带参数a时，当访问频率超过设定值，就会直接被限流，这里是直接在后台抛出异常：

![image-20240427162539097](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202404271625363.png)

而使用其他参数或是不带`a`参数，那么就不会出现这种问题了：

![image-20240427162605616](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202404271626127.png)

除了直接对某个参数精准限流外，还可以对参数携带的指定值单独设定阈值，比如不仅希望对参数`a`限流，而且还希望当参数`a`的值为10时，QPS达到5再进行限流，那么就可以设定例外：

![image-20240427162743917](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202404271627276.png)

这样，当请求携带参数`a`，且参数`a`的值为10时，阈值将按照指定的特例进行计算。

![image-20240427163011888](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202404271630281.png)