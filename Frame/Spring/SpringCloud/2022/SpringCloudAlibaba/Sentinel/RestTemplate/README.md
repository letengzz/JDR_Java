# Sentinel 整合 RestTemplate

使用`@SentinelRestTemplate`注解实现。

配置注入：

```java
@Bean
@LoadBalanced
// 让 RestTemplate 支持 Sentinel 限流
@SentinelRestTemplate(
	blockHandler = "blockHandlerFunc",
    blockHandlerClass = MyBlockHandlerClass.class,
	fallback = "fallbackFunc",
    fallbackClass = ExceptionUtil.class
) //这里同样可以设定fallback等参数
public RestTemplate restTemplate(){
	return new RestTemplate();
}
```

异常处理类：

```java
@Slf4j
public class MyBlockHandlerClass {
	public static String blockHandlerFunc(String a, BlockException e){
		log.warn("限流了",e);
		return "";
	}

	public static String fallbackFunc(String a){
		log.warn("降级了");
		return "";
	}
}
```

调用：

```java
@Autowired
RestTemplate restTemplate;

this.restTemplate.getForObject("http://user-service/user/{id}",Users.class,1);
```


配置开关：

```yaml
resttemplate:
  # 关闭 @SentinelRestTemplate 作用,开发环境可以临时关闭: 降级,限流
  sentinel:
    enabled: false
```

