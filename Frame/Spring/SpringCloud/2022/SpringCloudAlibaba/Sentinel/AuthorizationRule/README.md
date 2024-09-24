# Sentinel 授权规则

在某些场景下，需要根据调用接口的来源判断是否允许执行本次请求。此时就可以使用Sentinel提供的授权规则来实现，Sentinel的授权规则能够根据请求的来源判断是否允许本次请求通过。

在Sentinel的授权规则中，提供了 白名单与黑名单 两种授权类型 (**白放行、黑禁止**)。

**官方文档**：https://github.com/alibaba/Sentinel/wiki/%E9%BB%91%E7%99%BD%E5%90%8D%E5%8D%95%E6%8E%A7%E5%88%B6

**操作步骤**：

1. **创建Controller**：

   ```java
   @RestController
   @Slf4j
   public class EmpowerController //Empower授权规则，用来处理请求的来源
   {
       @GetMapping(value = "/empower")
       public String requestSentinel4(){
           log.info("测试Sentinel授权规则empower");
           return "Sentinel授权规则";
       }
   }
   ```

2. 获取请求来源信息：

   ```java
   @Component
   public class MyRequestOriginParser implements RequestOriginParser
   {
       @Override
       public String parseOrigin(HttpServletRequest httpServletRequest) {
           return httpServletRequest.getParameter("serverName");
       }
   }
   ```

3. 配置授权规则：

   ![image-20240427164151996](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202404271641707.png)

4. 访问 http://localhost:8201/empower ：

   ![image-20240427164237119](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202404271643387.png)

   访问 http://localhost:8201/empower?serverName=test1 ：

   ![image-20240427164456144](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202404271644236.png)

   访问 http://localhost:8201/empower?serverName=test2 ：

   ![image-20240427164528813](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202404271645341.png)

   访问 http://localhost:8201/empower?serverName=test3 ：

   ![image-20240427164601392](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202404271646717.png)