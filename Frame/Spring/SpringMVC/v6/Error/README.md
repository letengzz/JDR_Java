# SpringMVC 常见问题

## 乱码问题

### 字符串乱码

当使用`@ResponseBody`返回的字符串带有中文时，返回类型为String会被StringHttpMessageConverter处理，当时查看源码发现默认的Charset DEFAULT_CHARSET使用的是ISO-8859-1，需要修改为UTF-8，否则会显示为乱码：

![image-20230902211458827](https://cdn.jsdelivr.net/gh/letengzz/tc2@main/img/Java/202309022115339.png)

**解决**：

xml形式编写：

> application.xml

```xml
<!-- 开启mvc注解驱动 -->
<mvc:annotation-driven>
    <mvc:message-converters>
        <!-- 处理响应中文内容乱码 -->
        <bean class="org.springframework.http.converter.StringHttpMessageConverter">
            <property name="defaultCharset" value="UTF-8" />
            <property name="supportedMediaTypes">
                <list>
                    <value>text/html</value>
                    <value>application/json</value>
                </list>
            </property>
        </bean>
    </mvc:message-converters>
</mvc:annotation-driven>
```

全注解形式编写：

```java
@Configuration
@EnableWebMvc   //快速配置SpringMvc注解，如果不添加此注解会导致后续无法通过实现WebMvcConfigurer接口进行自定义配置
@ComponentScan("com.hjc.demo")
public class WebConfiguration {

    //处理响应中文内容乱码
    @Bean
    public RequestMappingHandlerAdapter requestMappingHandlerAdapter(){
        StringHttpMessageConverter stringHttpMessageConverter = new StringHttpMessageConverter(StandardCharsets.UTF_8);
        stringHttpMessageConverter.setSupportedMediaTypes(
                Arrays.asList(
                        MediaType.parseMediaType("text/html"),
                        MediaType.parseMediaType("application/json")));
        RequestMappingHandlerAdapter requestMappingHandlerAdapter = new RequestMappingHandlerAdapter();
        requestMappingHandlerAdapter.setMessageConverters(Arrays.asList(stringHttpMessageConverter));
        return requestMappingHandlerAdapter;
    }
}

```

****

### 控制台乱码

当输出到控制台时，可能会出现乱码问题：

![image-20230902220541615](https://cdn.jsdelivr.net/gh/letengzz/tc2@main/img/Java/202309022210721.png)

**解决**：

1. 在IDEA中设置控制台编码为 UTF-8：

   ![image-20230902220815219](https://cdn.jsdelivr.net/gh/letengzz/tc2@main/img/Java/202309022210426.png)

2. 在Tomcat中设置：

   ```bash
   -Dfile.encoding=utf-8 
   ```

   ![image-20230902220903538](https://cdn.jsdelivr.net/gh/letengzz/tc2@main/img/Java/202309022210135.png)

3. 显示正常：

   ![image-20230902221004755](https://cdn.jsdelivr.net/gh/letengzz/tc2@main/img/Java/202309022210270.png)