# SpringMVC 构建入门程序

**环境要求**：

- JDK：Java17
- Maven：3.6.x
- Spring：6.0.0
- Tomcat 10.1.x

**注意**：Spring6之后要求必须使用Tomcat10或更高版本

## 1. 创建Jakarta EE项目

![image-20230901202249805](https://cdn.jsdelivr.net/gh/letengzz/tc2@main/img/Java/202309181621621.png)

选择Web Profile：

![image-20230901202556677](https://cdn.jsdelivr.net/gh/letengzz/tc2@main/img/Java/202309012026676.png)

创建完成后会自动生成相关文件，但是还是请注意检查运行配置中的URL和应用程序上下文名称是否一致。

![image-20230830171526961](https://cdn.jsdelivr.net/gh/letengzz/tc2@main/img/Java/202309181623289.png)

## 2. 配置SpringMvc

SpringMvc项目依然支持多种配置形式。

首先需要添加Spring MVC相关的依赖：

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-webmvc</artifactId>
    <version>6.0.0</version>
</dependency>
```

**配置web.xml**：将DispatcherServlet替换掉Tomcat自带的Servlet，这里url-pattern需要写为`/`，即可完成替换：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="https://jakarta.ee/xml/ns/jakartaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="https://jakarta.ee/xml/ns/jakartaee https://jakarta.ee/xml/ns/jakartaee/web-app_5_0.xsd"
         version="5.0">
    <servlet>
        <servlet-name>mvc</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    </servlet>
    <servlet-mapping>
        <servlet-name>mvc</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>
</web-app>
```

### 传统XML配置形式

1. **为整个Web应用程序配置一个Spring上下文环境 (也就是容器)**：因为SpringMVC是基于Spring开发的，它直接利用Spring提供的容器来实现各种功能。所以需要编写一个配置文件：

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <beans xmlns="http://www.springframework.org/schema/beans"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://www.springframework.org/schema/beans
           https://www.springframework.org/schema/beans/spring-beans.xsd">
   </beans>
   ```

2. 为DispatcherServlet配置一些初始化参数来指定刚刚创建的配置文件：

   ```xml
   <servlet>
       <servlet-name>mvc</servlet-name>
       <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
       <init-param>
         	<!--     指定刚刚创建在类路径下的XML配置文件       -->
           <param-name>contextConfigLocation</param-name>
           <param-value>classpath:application.xml</param-value>
       </init-param>
   </servlet>
   ```
   
3. 删除项目自带的Servlet类，创建一个Mvc中使用的Controller类测试：

   ```java
   @Controller
   public class HelloController {
       @ResponseBody
       @RequestMapping("/")
       public String hello(){
           return "HelloWorld!";
       }
   }
   ```

4. 将这个类注册为Bean才能正常使用，编写Spring的配置文件，直接配置包扫描，XML下的包扫描需要这样开启：

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <beans xmlns="http://www.springframework.org/schema/beans"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xmlns:context="http://www.springframework.org/schema/context"
          xsi:schemaLocation="http://www.springframework.org/schema/beans
           https://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd">
     	<!-- 需要先引入context命名空间，然后直接配置base-package属性就可以了 -->
       <context:component-scan base-package="com.hjc.demo"/>
   </beans>
   ```

5. 如果可以成功在浏览器中出现HelloWorld则说明配置成功：

   ![image-20230830200525660](https://cdn.jsdelivr.net/gh/letengzz/tc2@main/img/Java/202309181624358.png)

实际上上面编写的Controller就是负责Servlet基本功能的，比如返回的是HelloWorld字符串，那么在访问这个地址的时候，得到的就是这里返回的字符串，可以看到写法非常简洁。

### 全注解配置形式

如果希望完完全全丢弃配置文件，使用纯注解开发，可以直接添加一个类，Tomcat会在类路径中查找实现ServletContainerInitializer 接口的类，如果发现的话，就用它来配置Servlet容器，Spring提供了这个接口的实现类 SpringServletContainerInitializer , 通过`@HandlesTypes(WebApplicationInitializer.class)`设置，这个类反过来会查找实现WebApplicationInitializer 的类，并将配置的任务交给他们来完成，因此直接实现接口即可：

```java
/**
 * web工程的初始化类，用来代替web.xml
 *
 * @author hjc
 */
public class MainInitializer extends AbstractAnnotationConfigDispatcherServletInitializer {

    /**
     * 指定基本的Spring配置类，一般用于业务层配置
     *
     * @return
     */
    @Override
    protected Class<?>[] getRootConfigClasses() {
        return new Class[]{WebConfiguration.class};
    }

    /**
     * 配置DispatcherServlet的配置类、
     * 主要用于Controller等配置，这里只使用上面的基本配置类
     * @return
     */
    @Override
    protected Class<?>[] getServletConfigClasses() {
        return new Class[0];
    }

    /**
     * 指定DispatcherServlet的映射规则，即url-pattern
     * @return
     */
    @Override
    protected String[] getServletMappings() {
        return new String[]{"/"};    //匹配路径
    }
}
```

接着需要再配置类中添加一些必要的注解：

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

**注意**：需要删除web.xml

这样同样可以正常访问：

![image-20230830202428723](https://cdn.jsdelivr.net/gh/letengzz/tc2@main/img/Java/202309181624068.png)

## 3. 使用日志

如果日志有报错无法显示Mvc相关的日志，请添加依赖：

```xml
<dependency>
      <groupId>org.slf4j</groupId>
      <artifactId>slf4j-api</artifactId>
      <version>1.7.33</version>
</dependency>
<dependency>
      <groupId>org.slf4j</groupId>
      <artifactId>slf4j-jdk14</artifactId>
      <version>1.7.33</version>
</dependency>
```

添加后就可以正常打印日志了：

![image-20230918162730388](https://cdn.jsdelivr.net/gh/letengzz/tc2@main/img/Java/202309181627960.png)
