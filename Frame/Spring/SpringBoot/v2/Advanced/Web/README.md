# SpringBoot-web开发

SpringBoot的Web开发能力，由[**SpringMVC**](../../../../SpringFamework/v6/MVC/README.md)提供。

## SpringMVC自动配置

大多场景都无需自定义配置，包括：

- 内容协商视图解析器和BeanName视图解析器
- 静态资源 (包括webjars)
- 自动注册 `Converter，GenericConverter，Formatter `
- 支持 `HttpMessageConverters` 
- 自动注册 `MessageCodesResolver` (国际化用)
- 静态index.html 页支持
- 自定义 `Favicon`  
- 自动使用 `ConfigurableWebBindingInitializer` (DataBinder负责将请求数据绑定到JavaBean上)

**不用`@EnableWebMvc`注解。使用** `@Configuration` **+** `WebMvcConfigurer` **自定义规则**

**声明** `WebMvcRegistrations` **改变默认底层组件**

**使用** `@EnableWebMvc`+`@Configuration`+`DelegatingWebMvcConfiguration` 全面接管SpringMVC

****

**自动配置过程**：

1. 整合web场景：

   ```xml
   <dependency>
       <groupId>org.springframework.boot</groupId>
   	<artifactId>spring-boot-starter-web</artifactId>
   </dependency>
   ```

2. 引入了 `autoconfigure`功能

3. `@EnableAutoConfiguration`注解使用`@Import(AutoConfigurationImportSelector.class)`批量导入组件

4. 加载 `META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports` 文件中配置的所有组件

5. 所有自动配置类：

   > org.springframework.boot.autoconfigure.web.client.RestTemplateAutoConfiguration
   > org.springframework.boot.autoconfigure.web.embedded.EmbeddedWebServerFactoryCustomizerAutoConfiguration
   > ====以下是响应式web场景和现在的没关系======
   > org.springframework.boot.autoconfigure.web.reactive.HttpHandlerAutoConfiguration
   > org.springframework.boot.autoconfigure.web.reactive.ReactiveMultipartAutoConfiguration
   > org.springframework.boot.autoconfigure.web.reactive.ReactiveWebServerFactoryAutoConfiguration
   > org.springframework.boot.autoconfigure.web.reactive.WebFluxAutoConfiguration
   > org.springframework.boot.autoconfigure.web.reactive.WebSessionIdResolverAutoConfiguration
   > org.springframework.boot.autoconfigure.web.reactive.error.ErrorWebFluxAutoConfiguration
   > org.springframework.boot.autoconfigure.web.reactive.function.client.ClientHttpConnectorAutoConfiguration
   > org.springframework.boot.autoconfigure.web.reactive.function.client.WebClientAutoConfiguration
   > ================以上没关系=================
   > org.springframework.boot.autoconfigure.web.servlet.DispatcherServletAutoConfiguration
   > org.springframework.boot.autoconfigure.web.servlet.ServletWebServerFactoryAutoConfiguration
   > org.springframework.boot.autoconfigure.web.servlet.error.ErrorMvcAutoConfiguration
   > org.springframework.boot.autoconfigure.web.servlet.HttpEncodingAutoConfiguration
   > org.springframework.boot.autoconfigure.web.servlet.MultipartAutoConfiguration
   > org.springframework.boot.autoconfigure.web.servlet.WebMvcAutoConfiguration

6. 绑定了配置文件的一堆配置项：
   1. SpringMVC的所有配置 `spring.mvc`
   2. Web场景通用配置 `spring.web`
   3. 文件上传配置 `spring.servlet.multipart`
   4. 服务器的配置 `server`: 比如：编码方式

## 默认效果

**默认配置**：

1. 包含了 ContentNegotiatingViewResolver 和 BeanNameViewResolver 组件，**方便视图解析**
2. **默认的静态资源处理机制**： 静态资源放在 static 文件夹下即可直接访问
3. **自动注册**了 `Converter`，`GenericConverter`，`Formatter`组件，适配常见**数据类型转换**和**格式化需求**
4. **支持** **HttpMessageConverters**，可以**方便返回JSON等数据类型**
5. **注册** MessageCodesResolver，方便**国际化**及错误消息处理
6. **支持 静态** index.html
7. **自动使用**ConfigurableWebBindingInitializer，实现**消息处理、数据绑定、类型转化、数据校验**等功能

**说明**：

- 如果想保持boot mvc 的默认配置，并且自定义更多的 mvc 配置，如：interceptors，formatters，view controllers 等。可以使用`@Configuration`注解添加一个 WebMvcConfigurer 类型的配置类，并不要标注 `@EnableWebMvc`
- 如果想保持 boot mvc 的默认配置，但要自定义核心组件实例，比如：RequestMappingHandlerMapping，RequestMappingHandlerAdapter 或 ExceptionHandlerExceptionResolver，给容器中放一个WebMvcRegistrations 组件即可
- 如果想全面接管 Spring MVC，`@Configuration` 标注一个配置类，并加上 `@EnableWebMvc`注解，实现 WebMvcConfigurer 接口

![image-20230801152505058](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202308011526218.png)

**推荐方式**：给容器中写一个配置类`@Configuration`实现 `WebMvcConfigurer`但是不要标注 `@EnableWebMvc`注解，实现手自一体的效果。

## WebMvcAutoConfiguration原理

SpringBoot启动默认加载  xxxAutoConfiguration 类（自动配置类）

SpringMVC功能的自动配置类 WebMvcAutoConfiguration生效。

### 生效条件

```java
@AutoConfiguration(after = { DispatcherServletAutoConfiguration.class, TaskExecutionAutoConfiguration.class,ValidationAutoConfiguration.class }) //在这些自动配置之后
@ConditionalOnWebApplication(type = Type.SERVLET) //如果是web应用就生效，类型：SERVLET
@ConditionalOnClass({ Servlet.class, DispatcherServlet.class, WebMvcConfigurer.class })
@ConditionalOnMissingBean(WebMvcConfigurationSupport.class) //容器中没有这个Bean，才生效。默认就是没有
@AutoConfigureOrder(Ordered.HIGHEST_PRECEDENCE + 10)//优先级
@ImportRuntimeHints(WebResourcesRuntimeHints.class)
public class WebMvcAutoConfiguration { 
}
```

### 效果

1. 放了两个Filter：
   1. `HiddenHttpMethodFilter`：页面表单提交Rest请求(GET、POST、PUT、DELETE)
   1. `FormContentFilter`： 表单内容Filter，GET(数据放URL后面)、POST(数据放请求体)请求可以携带数据，PUT、DELETE 的请求体数据会被忽略

1. 给容器中放了`WebMvcConfigurer`组件：给SpringMVC添加各种定制功能

   ```java
   @Configuration(proxyBeanMethods = false)
   @Import(EnableWebMvcConfiguration.class) //额外导入了其他配置
   @EnableConfigurationProperties({ WebMvcProperties.class, WebProperties.class })
   @Order(0)
   public static class WebMvcAutoConfigurationAdapter implements WebMvcConfigurer, ServletContextAware{        //实现了WebMvcConfigurer
   }
   ```

   1. 所有的功能最终会和配置文件进行绑定
   1. WebMvcProperties： `spring.mvc`配置文件
   1. WebProperties： `spring.web`配置文件

### WebMvcConfigurer接口

提供了配置SpringMVC底层的所有组件入口。

![img](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202308011535530.png)

### 静态资源规则源码

```java
@Override
public void addResourceHandlers(ResourceHandlerRegistry registry) {
    if (!this.resourceProperties.isAddMappings()) {
        logger.debug("Default resource handling disabled");
        return;
    }
    addResourceHandler(registry, this.mvcProperties.getWebjarsPathPattern(),
            "classpath:/META-INF/resources/webjars/");
    addResourceHandler(registry, this.mvcProperties.getStaticPathPattern(), (registration) -> {
        registration.addResourceLocations(this.resourceProperties.getStaticLocations());
        if (this.servletContext != null) {
            ServletContextResource resource = new ServletContextResource(this.servletContext, SERVLET_LOCATION);
            registration.addResourceLocations(resource);
        }
    });
}
```

**规则**：

1. 访问： `/webjars/**`路径就去 `classpath:/META-INF/resources/webjars/`下找资源：

   - maven 导入依赖

     官方网站：https://www.webjars.org/

     ```xml
     <dependency>
     	<groupId>org.webjars</groupId>
         <artifactId>jquery</artifactId>
         <version>3.5.1</version>
     </dependency>
     ```

1. 访问： `/**`路径就去 静态资源默认的四个位置找资源：

   1. `classpath:/META-INF/resources/`
   1. `classpath:/resources/`
   1. `classpath:/static/`
   1. `classpath:/public/`

1. **静态资源默认都有缓存规则的设置**：

   1. 所有缓存的设置，直接通过**配置文件**： `spring.web`
   1. cachePeriod： 缓存周期。 默认没有，以s(秒)为单位
   1. cacheControl： **HTTP缓存**控制 ([https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Caching](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Caching#概览))
   1. **useLastModified**：是否使用最后一次修改。配合HTTP Cache规则


如果浏览器访问了一个静态资源 `index.js`，如果服务这个资源没有发生变化，下次访问的时候就可以直接让浏览器用缓存中的东西，而不用给服务器发请求。

```java
registration.setCachePeriod(getSeconds(this.resourceProperties.getCache().getPeriod()));
registration.setCacheControl(this.resourceProperties.getCache().getCachecontrol().toHttpCacheControl());
registration.setUseLastModified(this.resourceProperties.getCache().isUseLastModified());
```

### EnableWebMvcConfiguration 源码

```java
//SpringBoot 给容器中放 WebMvcConfigurationSupport 组件。
//如果自己放了 WebMvcConfigurationSupport 组件，Boot的WebMvcAutoConfiguration都会失效。
@Configuration(proxyBeanMethods = false)
@EnableConfigurationProperties(WebProperties.class)
public static class EnableWebMvcConfiguration extends DelegatingWebMvcConfiguration implements ResourceLoaderAware 
{ 
}
```

`HandlerMapping`： 根据请求路径 ` /a` 找那个handler能处理请求
1. `WelcomePageHandlerMapping`： 访问 `/**`路径下的所有请求，都在以前静态资源路径下找，欢迎页也一样(`classpath:/META-INF/resources/`、`classpath:/resources/`、`classpath:/static/`、`classpath:/public/`)
1. 找`index.html`：只要静态资源的位置有一个 `index.html`页面，项目启动默认访问


### WebMvcConfigurationSupport

提供了很多的默认设置。判断系统中是否有相应的类：如果有，就加入相应的`HttpMessageConverter`

```java
jackson2Present = ClassUtils.isPresent("com.fasterxml.jackson.databind.ObjectMapper", classLoader) &&
				ClassUtils.isPresent("com.fasterxml.jackson.core.JsonGenerator", classLoader);
jackson2XmlPresent = ClassUtils.isPresent("com.fasterxml.jackson.dataformat.xml.XmlMapper", classLoader);
jackson2SmilePresent = ClassUtils.isPresent("com.fasterxml.jackson.dataformat.smile.SmileFactory", classLoader);
```

## 静态资源访问

### 静态资源规则

只要静态资源放在类路径下如下⽬录： `/static`、 `/public`、`/resources`、`/META-INF/resources` ，就可以使⽤静态资源⽅式访问： **当前项目根路径/ + 静态资源名**

**说明**：默认情况下， 静态映射是`/**`，可以使⽤属性 `spring.mvc.static-path-pattern` 属性进行调整，也可以使⽤ `spring.mvc.resources.static-locations` 配置⾃定义静态资源存放目录。

```yaml
spring:
  mvc:
    static-path-pattern: /res/** #静态资源映射路径，默认/**
  web:
    resources:
      static-locations: [classpath:/hjc/] #静态资源自定义路径，设置有默认路径失效
```

**注意**：⾃定义资源路径后，默认资源路径失效。

当请求进来，先去找Controller看能不能处理。不能处理的所有请求⼜都交给静态资源处理器。静态资源也找不到则响应404页面。

### 欢迎页

SpringBoot 支持静态和模板的欢迎页静态资源路径`classpath:/static`下添加 index.html。

欢迎页规则在 WebMvcAutoConfiguration 中进行了定义：

1. 在**静态资源**目录下找 index.html
2. 没有就在 templates下找index模板页

**注意**：

- 不可以配置静态资源的访问前缀，否则导致 index.html不能被默认访问。

- 模板路径`classpath:/templates`下添加index.html，只有static不存在欢迎页才会访问。

- 需要配置thymleaf模板引擎：

  ```xml
  <dependency>
  	<groupId>org.springframework.boot</groupId>
  	<artifactId>spring-boot-starter-thymeleaf</artifactId>
  </dependency>
  ```

![image-20230804205109114](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202308042052242.png)

### Favorites Icon

favicon.ico 放在静态资源目录下即可。

**注意**：⾃定义资源路径后，Favicon 功能失效。

![image-20230804205715095](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202308042057630.png)

### webjar

自动映射 `/webjars/**`

官方网站：https://www.webjars.org/

```xml
<dependency>
	<groupId>org.webjars</groupId>
    <artifactId>jquery</artifactId>
    <version>3.5.1</version>
</dependency>
```

访问地址：http://localhost:8080/webjars/jquery/3.5.1/jquery.js  后面地址要按照依赖里面的包路径

![image-20230804210209928](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202308042102577.png)

### 自定义静态资源规则

自定义静态资源路径、自定义缓存规则

#### 配置方式

`spring.mvc`： 静态资源访问前缀路径

`spring.web`：

- 静态资源目录
- 静态资源缓存策略

```properties
# spring.web：
# spring.web.locale: 配置国际化的区域信息
# spring.web.resources: 静态资源策略(staticLocations: 静态资源的路径、addMapping: 是否开启静态资源映射、chain: 静态资源处理链、cache: 缓存规则)

#开启静态资源映射规则
spring.web.resources.add-mappings=true

#设置缓存时间(s)
spring.web.resources.cache.period=3600
##缓存详细合并项控制，覆盖period配置：
## 浏览器第一次请求服务器，服务器告诉浏览器此资源缓存7200秒，7200秒以内的所有此资源访问不用发给服务器请求，7200秒以后发请求给服务器
spring.web.resources.cache.cachecontrol.max-age=7200
#使用资源 last-modified 时间，来对比服务器和浏览器的资源是否相同没有变化。相同返回 304
spring.web.resources.cache.use-last-modified=true


# 共享缓存
spring.web.resources.cache.cachecontrol.cache-public=true
#自定义静态资源文件夹位置
spring.web.resources.static-locations=classpath:/a/,classpath:/b/,classpath:/static/

#spring.mvc
# 自定义webjars路径前缀
spring.mvc.webjars-path-pattern=/wj/**
# 静态资源访问路径前缀
spring.mvc.static-path-pattern=/static/**
```

#### 代码方式

容器中只要有一个 WebMvcConfigurer 组件，配置的底层行为都会生效。

**原理**：

1. WebMvcAutoConfiguration 是一个自动配置类，它里面有一个 `EnableWebMvcConfiguration`
2. `EnableWebMvcConfiguration`继承与 `DelegatingWebMvcConfiguration`，这两个都生效
3. `DelegatingWebMvcConfiguration`利用 DI 把容器中 所有 `WebMvcConfigurer `注入进来
4. 别人调用 `DelegatingWebMvcConfiguration` 的方法配置底层规则，而它调用所有 `WebMvcConfigurer`的配置底层方法。

```java
@EnableWebMvc //禁用boot的默认配置
@Configuration //这是一个配置类
public class MyConfig implements WebMvcConfigurer {


    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        //保留以前规则
        WebMvcConfigurer.super.addResourceHandlers(registry);
        //自己写新的规则。
        registry.addResourceHandler("/static/**")
                .addResourceLocations("classpath:/a/","classpath:/b/")
                .setCacheControl(CacheControl.maxAge(1180, TimeUnit.SECONDS));
    }
}
```

或

```java
@Configuration //这是一个配置类,给容器中放一个 WebMvcConfigurer 组件，就能自定义底层
public class MyConfig {


    @Bean
    public WebMvcConfigurer webMvcConfigurer(){
        return new WebMvcConfigurer() {
            @Override
            public void addResourceHandlers(ResourceHandlerRegistry registry) {
                registry.addResourceHandler("/static/**")
                        .addResourceLocations("classpath:/a/", "classpath:/b/")
                        .setCacheControl(CacheControl.maxAge(1180, TimeUnit.SECONDS));
            }
        };
    }
}
```

![image-20230804213433129](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202308042134269.png)

### 缓存

```properties
server.port=9000

# spring.web：
# spring.web.locale: 配置国际化的区域信息
# spring.web.resources: 静态资源策略(staticLocations: 静态资源的路径、addMapping: 是否开启静态资源映射、chain: 静态资源处理链、cache: 缓存规则)

#开启静态资源映射规则
spring.web.resources.add-mappings=true

#设置缓存时间(s)
#spring.web.resources.cache.period=3600
##缓存详细合并项控制，覆盖period配置：
## 浏览器第一次请求服务器，服务器告诉浏览器此资源缓存7200秒，7200秒以内的所有此资源访问不用发给服务器请求，7200秒以后发请求给服务器
spring.web.resources.cache.cachecontrol.max-age=7200
#使用资源 last-modified 时间，来对比服务器和浏览器的资源是否相同没有变化。相同返回 304
spring.web.resources.cache.use-last-modified=true
```

## 路径匹配

**Spring5.3** 之后加入了更多的请求路径匹配的实现策略：以前只支持 AntPathMatcher 策略, 现在提供了 **PathPatternParser** 策略。并且可以指定到底使用那种策略。

### Ant风格路径用法

Ant 风格的路径模式**语法规则**：

- `*`：表示**任意数量**的字符。
- `?`：表示任意**一个字符**。
- `**`：表示 **任意数量的目录**。
- `{}`：表示一个命名的模式**占位符**。
- `[]`：表示**字符集合**，例如[a-z]表示小写字母。

例如：

- `*.html`：匹配任意名称，扩展名为.html的文件。
- `/folder1/*/*.java`：匹配在folder1目录下的任意两级目录下的.java文件。
- `/folder2/**/*.jsp`：匹配在folder2目录下任意目录深度的.jsp文件。
- `/{type}/{id}.html`：匹配任意文件名为{id}.html，在任意命名的{type}目录下的文件。

**注意**：Ant 风格的路径模式语法中的**特殊字符需要转义**：

- 要匹配文件路径中的星号，则需要转义为`\\*`
- 要匹配文件路径中的问号，则需要转义为`\\?`

### 模式切换

**修改路径匹配策略**：

```properties
# 改变路径匹配策略：
# ant_path_matcher 老版策略；
# path_pattern_parser 新版策略；
spring.mvc.pathmatch.matching-strategy=ant_path_matcher
```

#### AntPathMatcher 与 PathPatternParser

- PathPatternParser 在 jmh 基准测试下，有 6~8 倍吞吐量提升，降低 30%~40%空间分配率
- PathPatternParser 兼容 AntPathMatcher语法，并支持更多类型的路径模式
- PathPatternParser  "`**`" **多段匹配**的支持**仅允许在模式末尾使用**

```java
/**
* 默认使用 PathPatternParser 进行路径匹配
* 不能匹配 **在中间的情况，剩下的和 AntPathMatcher 语法兼容
*/
@GetMapping("/a*/b?/{p1:[a-f]+}")
public String hello(HttpServletRequest request, 
                        @PathVariable("p1") String path) {

	log.info("路径变量p1： {}", path);
    //获取请求路径
    String uri = request.getRequestURI();
    return uri;
}
```

![image-20230804215103710](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202308042151831.png)

**注意**：

- 当使用`**`不在末尾时，使用PathPatternParser模式会提示使用AntPathMatcher，所以如果路径中间需要有 `**`，替换成ant风格路径

  ![image-20230804215403804](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202308042154951.png)

  ```properties
  spring.mvc.pathmatch.matching-strategy=ant_path_matcher
  ```

  ```yaml
  # 改变路径匹配策略：ant_path_matcher 老版策略；path_pattern_parser 新版策略
  spring:
    mvc:
      pathmatch:
        matching-strategy: ant_path_matcher
  ```

  ![image-20230804220011048](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202308042200968.png)

- 使用默认的路径匹配规则，是由 PathPatternParser  提供的

  ![image-20230804222136420](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202308042221159.png)

  ![image-20230804222217814](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202308042222032.png)

## 内容协商

一套系统适配多端数据返回

![img](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/202308021001821.png)

### 多端内容适配

####  默认规则

**SpringBoot 多端内容适配**：

1. **基于请求头内容协商(默认开启)**：

    客户端向服务端发送请求，携带HTTP标准的**Accept请求头**。

   **Accept**: `application/json`、`text/xml`、`text/yaml`

   服务端根据**客户端请求头期望的数据类型进行动态返回**

   ![image-20230804222924773](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202308042229351.png)

2. **基于请求参数内容协商(需要开启)**：

   1.  送请求 GET /projects/spring-boot?format=json 

   1. 匹配到 `@GetMapping("/projects/spring-boot")` 

   1. 根据**参数协商**，优先返回 json 类型数据 (**需要开启参数匹配设置**)

   1. 发送请求 GET /projects/spring-boot?format=xml,优先返回 xml 类型数据

      ![image-20230804223135250](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202308042231999.png)

默认支持把对象转化为JSON，因为默认web场景导入了Jackson处理的包：jackson-core

```java
@Data
public class Person {
    private Long id;
    private String userName;
    private String email;
    private Integer age;
}
```

```java
@RestController
public class PersonController {
    @RequestMapping("/person")
    public Person person(){
        Person person = new Person();
        person.setId(1L);
        person.setUserName("张三");
        person.setEmail("111@qq.com");
        person.setAge(22);
        return person;
    }
}
```

![image-20230805152218210](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202308051522439.png)

**例**：请求同一个接口，可以返回json和xml不同格式数据：

1. 引入支持写出xml内容依赖 (jackson 支持把数据写为xml)：

   ```xml
   <dependency>
       <groupId>com.fasterxml.jackson.dataformat</groupId>
       <artifactId>jackson-dataformat-xml</artifactId>
   </dependency>
   ```

2. 标注注解：

   ```java
   @JacksonXmlRootElement  // 可以写出为xml文档
   @Data
   public class Person {
       private Long id;
       private String userName;
       private String email;
       private Integer age;
   }
   ```

3. 进行了内容协商 默认返回了文本：

   ![image-20230805152754450](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202308051527240.png)

4. 使用**基于请求头内容协商**：

   ![image-20230805153315918](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202308051533147.png)

   ![image-20230805153423806](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202308051534407.png)

5. 使用**基于请求参数内容协商**：开启基于请求参数的内容协商

   ```properties
   # 开启基于请求参数的内容协商功能。 默认参数名：format。 默认此功能不开启
   spring.mvc.contentnegotiation.favor-parameter=true
   # 指定内容协商时使用的参数名。默认是 format
   spring.mvc.contentnegotiation.parameter-name=type
   ```

   ```yaml
   spring:
     mvc:
       contentnegotiation:
         # 开启基于请求参数的内容协商功能。 默认参数名：format。 默认此功能不开启
         favor-parameter: true
         # 指定内容协商时使用的参数名。默认是 format
         parameter-name: type
   ```

   ![image-20230805154210686](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202308051542142.png)

#### 配置协商规则与支持类型

修改**内容协商方式**：

```properties
#使用参数进行内容协商
spring.mvc.contentnegotiation.favor-parameter=true  
#自定义参数名，默认为format
spring.mvc.contentnegotiation.parameter-name=myparam 
```

```yaml
spring:
  mvc:
    contentnegotiation:
      # 开启基于请求参数的内容协商功能。 默认参数名：format。 默认此功能不开启
      favor-parameter: true
      # 指定内容协商时使用的参数名。默认是 format
      parameter-name: myparam
```

大多数 MediaType 都是开箱即用的。也可以**自定义内容类型**：

```properties
spring.mvc.contentnegotiation.media-types.yaml=text/yaml
```

```yaml
spring:
  mvc:
    contentnegotiation:
      # 新增一种媒体类型
      media-types:
        yaml: text/yaml
```

### 自定义内容返回

**步骤**：

- 配置媒体类型支持: 
  - `spring.mvc.contentnegotiation.media-types.yaml=text/yaml`

- 编写对应的`HttpMessageConverter`，要告诉Boot这个支持的媒体类型

- 把MessageConverter组件加入到底层
  - 容器中放一个``WebMvcConfigurer`` 组件，并配置底层的`MessageConverter`

#### 增加yaml返回支持

导入依赖：

```xml
<!-- 支持返回YAML格式数据-->
<dependency>
    <groupId>com.fasterxml.jackson.dataformat</groupId>
    <artifactId>jackson-dataformat-yaml</artifactId>
</dependency>
```

把对象写出成YAML：

```java
public static void main(String[] args) throws JsonProcessingException {
	Person person = new Person();
    person.setId(1L);
    person.setUserName("张三");
    person.setEmail("aaa@qq.com");
    person.setAge(18);

    //YAMLGenerator.Feature.WRITE_DOC_START_MARKER：yaml开头“---”
    YAMLFactory factory = new YAMLFactory().disable(YAMLGenerator.Feature.WRITE_DOC_START_MARKER);
    ObjectMapper mapper = new ObjectMapper(factory);

    String s = mapper.writeValueAsString(person);
    System.out.println(s);
}
```

编写配置：

```properties
#新增一种媒体类型
spring.mvc.contentnegotiation.media-types.yaml=text/yaml
```

```yaml
#新增一种媒体类型
spring:
  mvc:
    contentnegotiation:
      media-types:
        yaml: text/yaml
```

增加`HttpMessageConverter`组件，专门负责把对象写出为yaml格式：

```java
@Bean
public WebMvcConfigurer webMvcConfigurer(){
    return new WebMvcConfigurer() {
        @Override //配置一个能把对象转为yaml的messageConverter
        public void configureMessageConverters(List<HttpMessageConverter<?>> converters) {
            converters.add(new MyYamlHttpMessageConverter());
        }
    };
}
```

#### HttpMessageConverter写法

```java
public class MyYamlHttpMessageConverter extends AbstractHttpMessageConverter<Object> {

    private ObjectMapper objectMapper = null; //把对象转成yaml

    public MyYamlHttpMessageConverter(){
        //告诉SpringBoot这个MessageConverter支持哪种媒体类型  //媒体类型
        super(new MediaType("text", "yaml", Charset.forName("UTF-8")));
        YAMLFactory factory = new YAMLFactory()
                .disable(YAMLGenerator.Feature.WRITE_DOC_START_MARKER);
        this.objectMapper = new ObjectMapper(factory);
    }

    //支持某种类型
    @Override
    protected boolean supports(Class<?> clazz) {
        //只要是对象类型，不是基本类型 此判断忽略 可自行添加
        return true;
    }

    //重写读指定对象的操作
    @Override  //@RequestBody
    protected Object readInternal(Class<?> clazz, HttpInputMessage inputMessage) throws IOException, HttpMessageNotReadableException {
        return null;
    }

    //重写写指定对象的操作
    @Override //@ResponseBody 
    protected void writeInternal(Object methodReturnValue, HttpOutputMessage outputMessage) throws IOException, HttpMessageNotWritableException {

        //try-with写法，自动关流
        try(OutputStream os = outputMessage.getBody()){
            this.objectMapper.writeValue(os,methodReturnValue);
        }

    }
}
```

![image-20230805230556304](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202308052305785.png)

![image-20230805230615593](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202308052306842.png)

### 内容协商原理-`HttpMessageConverter`

通过定制 `HttpMessageConverter`  来实现多端内容协商

编写`WebMvcConfigurer`提供的`configureMessageConverters`底层，修改底层的`MessageConverter`

#### `@ResponseBody`由`HttpMessageConverter`处理

标注了`@ResponseBody`的返回值 将会由支持它的 `HttpMessageConverter`写给浏览器

1. 如果controller方法的返回值标注了 `@ResponseBody `注解
   1. 请求进来先来到`DispatcherServlet`的`doDispatch()`进行处理
   1. 找到一个 `HandlerAdapter `适配器。利用适配器执行目标方法
   1. `RequestMappingHandlerAdapter`来执行，调用`invokeHandlerMethod()`来执行目标方法
   1. 目标方法执行之前，准备好两个东西：
      1. `HandlerMethodArgumentResolver`：参数解析器，确定目标方法每个参数值
      1. `HandlerMethodReturnValueHandler`：返回值处理器，确定目标方法的返回值改怎么处理

   1. `RequestMappingHandlerAdapter` 里面的`invokeAndHandle()`真正执行目标方法
   1. 目标方法执行完成，会返回**返回值对象**
   1. **找到一个合适的返回值处理器** `HandlerMethodReturnValueHandler`
   1. 最终找到 `RequestResponseBodyMethodProcessor`能处理 标注了 `@ResponseBody`注解的方法
   1. `RequestResponseBodyMethodProcessor` 调用`writeWithMessageConverters `，利用`MessageConverter`把返回值写出去


上面解释：`@ResponseBody`由`HttpMessageConverter`处理

1. `HttpMessageConverter` 会**先进行内容协商**
   1. 遍历所有的`MessageConverter`看谁支持这种**内容类型的数据**
   
   1. 默认`MessageConverter`：
   
      ![img](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202308051623295.png)
   
   1. 最终因为要`json`所以`MappingJackson2HttpMessageConverter`支持写出json
   
   1. jackson用`ObjectMapper`把对象写出去


#### `WebMvcAutoConfiguration`提供几种默认`HttpMessageConverters`

- `EnableWebMvcConfiguration`通过 `addDefaultHttpMessageConverters`添加了默认的`MessageConverter`：
  - `ByteArrayHttpMessageConverter`： 支持字节数据读写
  - `StringHttpMessageConverter`： 支持字符串读写
  - `ResourceHttpMessageConverter`：支持资源读写
  - `ResourceRegionHttpMessageConverter`: 支持分区资源写出
  - `AllEncompassingFormHttpMessageConverter`：支持表单xml/json读写
  - `MappingJackson2HttpMessageConverter`： 支持请求响应体Json读写


默认8个：

![img](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202308052235292.png)

系统提供默认的MessageConverter 功能有限，仅用于json或者普通返回数据。额外增加新的内容协商功能，必须增加新的`HttpMessageConverter`

### WebMvcConfigurationSupport

提供了很多的默认配置。判断系统中是否有相应的类：如果有，就加入相应的HttpMessageConverter

![image-20230805224337466](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202308052243134.png)

## Restful请求

REST 是 Representational State Transfer的缩写，如果⼀个架构符合REST原则，就称它为 RESTful架构。RESTful架构可以充分的利用 HTTP 协议的各种功能，是 HTTP 协议的最佳实践。 RESTful API 是⼀种软件架构风格、设计风格，可以让软件更加清晰，更简洁，更有层次，可维护性更好。

![image-20230301091133844](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202312241441195.png)

## 视图解析与模板引擎

由于 **SpringBoot** 使用了**嵌入式 Servlet 容器**。所以 **JSP** 默认是**不能使用**的。

如果需要**服务端页面渲染**，优先考虑使用 模板引擎。

![img](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202307312026519.png)

模板引擎页面默认放在 `src/main/resources/templates`

**SpringBoot** 包含以下模板引擎的自动配置：

- FreeMarker
- Groovy
- [Thymeleaf](../../Integration/Thymeleaf/README.md)
- Mustache

## 国际化

国际化也称作i18n，其来源是英文单词 internationalization的首末字符i和n，18为中间的字符数。由于软件发行可能面向多个国家，对于不同国家的用户，软件显示不同语言的过程就是国际化。通常来讲，软件中的国际化是通过配置文件来实现的，假设要支撑两种语言，那么就需要两个版本的配置文件。

国际化的自动配置参照`MessageSourceAutoConfiguration`

**实现步骤**：

1. Spring Boot 在类路径根下查找messages资源绑定文件。文件名为：`messages.properties`
2. 多语言可以定义多个消息文件，命名为`messages_区域代码.properties`。如：

1. 1. `messages.properties`：默认
   2. `messages_zh_CN.properties`：中文环境
   3. `messages_en_US.properties`：英语环境

1. 在**程序中**可以自动注入 `MessageSource`组件，获取国际化的配置项值
2. 在**页面中**可以使用表达式 ` #{}`获取国际化的配置项值

```java
@Autowired  
MessageSource messageSource;  //国际化取消息用的组件

@GetMapping("/haha")
public String haha(HttpServletRequest request){

	Locale locale = request.getLocale();
    //利用代码的方式获取国际化配置文件中指定的配置项的值
    String login = messageSource.getMessage("login", null, locale);
    return login;
}
```

## 文件上传

文件上传要求form表单的请求方式必须为post，并且添加属性`enctype="multipart/form-data"`

SpringBoot中将上传的文件封装到MultipartFile对象中，通过此对象可以获取文件相关信息

> index.html

```html
<form th:action="@{/upload}" method="post" enctype="multipart/form-data">
    <!-- 单文件上传 -->
    <input type="file" name="file"><br>
    <!-- 多文件上传 -->
    <input type="file" name="photos" multiple><br>
    <input type="submit" value="提交">
</form>
```

> FileController.java

```java
@Slf4j
@Controller
public class FileController {

    @PostMapping("/upload")
    public String upload(HttpSession session, @RequestPart("file")MultipartFile file,
                         @RequestPart("photos")MultipartFile[] photos) throws IOException {
        log.info("上传的信息:file={},photos={}",file.getSize(),photos.length);
        if (!file.isEmpty()){
            //保存到文件服务器，OSS服务器
            String originalFilename = file.getOriginalFilename();
            //获取上传的文件的后缀名
            assert originalFilename != null;
            String suffixName = originalFilename.substring(originalFilename.lastIndexOf("."));
            //使用UUID防止重名 将UUID作为文件名
            String uuid = UUID.randomUUID().toString().replaceAll("-", "");
            //将uuid和后缀名拼接后的结果作为最终的文件名
            String fileName = uuid + suffixName;
            ServletContext servletContext = session.getServletContext();
            String path = servletContext.getRealPath("/");
            File filePath = new File(path,"img\\");
            if (!filePath.exists()){
                filePath.mkdirs();
            }
            file.transferTo(new File(filePath+fileName));
        }
        //上传多个文件
        if (photos.length>0){
            for (MultipartFile photo :
                    photos) {
                String originalFilename = photo.getOriginalFilename();
                //获取上传的文件的后缀名
                assert originalFilename != null;
                String suffixName = originalFilename.substring(originalFilename.lastIndexOf("."));
                //使用UUID防止重名 将UUID作为文件名
                String uuid = UUID.randomUUID().toString().replaceAll("-", "");
                //将uuid和后缀名拼接后的结果作为最终的文件名
                String fileName = uuid + suffixName;

                ServletContext servletContext = session.getServletContext();
                String path = servletContext.getRealPath("/");
                File filePath = new File(path,"img\\");
                if (!filePath.exists()){
                    filePath.mkdirs();
                }
                System.out.println(filePath+fileName);
                photo.transferTo(new File(filePath,fileName));
            }
        }
        return "index";
    }
}
```

## 拦截器

### 拦截器原理

1. 根据当前请求，找到`HandlerExecutionChain`【可以处理请求的handler以及handler的所有 拦截器】

2. 先来**顺序执行** 所有拦截器的 preHandle方法

   1. 如果当前拦截器prehandler返回为true。则执行下一个拦截器的preHandle

   2. 如果当前拦截器返回为false。直接倒序执行所有已经执行了的拦截器的  afterCompletion

3. 如果任何一个拦截器返回false。直接跳出不执行目标方法

4. 所有拦截器都返回true。执行目标方法

5. 倒序执行所有拦截器的postHandle方法。

6. 前面的步骤有任何异常都会直接倒序触发 afterCompletion

7. 页面成功渲染完成以后，也会倒序触发 afterCompletion

![image-20230222192104145](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202312241543815.png)

### HandlerInterceptor 接口

```java
/**
 * 登录检查
 * 1、配置好拦截器要拦截哪些请求
 * 2、把这些配置放在容器中
 */
@Slf4j
public class LoginInterceptor implements HandlerInterceptor {

    /**
     * 目标方法执行之前
     * @param request
     * @param response
     * @param handler
     * @return
     * @throws Exception
     */
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {

        String requestURI = request.getRequestURI();
        log.info("preHandle拦截的请求路径是{}",requestURI);

        //登录检查逻辑
        HttpSession session = request.getSession();

        Object loginUser = session.getAttribute("loginUser");

        if(loginUser != null){
            //放行
            return true;
        }

        //拦截住。未登录。跳转到登录页
        request.setAttribute("msg","请先登录");
//        re.sendRedirect("/");
        request.getRequestDispatcher("/").forward(request,response);
        return false;
    }

    /**
     * 目标方法执行完成以后
     * @param request
     * @param response
     * @param handler
     * @param modelAndView
     * @throws Exception
     */
    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        log.info("postHandle执行{}",modelAndView);
    }

    /**
     * 页面渲染以后
     * @param request
     * @param response
     * @param handler
     * @param ex
     * @throws Exception
     */
    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        log.info("afterCompletion执行异常{}",ex);
    }
}
```

### 配置拦截器

```java
/**
 * 1、编写一个拦截器实现HandlerInterceptor接口
 * 2、拦截器注册到容器中（实现WebMvcConfigurer的addInterceptors）
 * 3、指定拦截规则【如果是拦截所有，静态资源也会被拦截】
 */
@Configuration
public class AdminWebConfig implements WebMvcConfigurer {

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new LoginInterceptor())
                .addPathPatterns("/**")  //所有请求都被拦截包括静态资源
                .excludePathPatterns("/","/login","/css/**","/fonts/**","/images/**","/js/**"); //放行的请求
    }
}
```

## 异常处理

### 默认错误处理

默认情况下，Spring Boot提供`/error`处理所有错误的映射

对于机器客户端，它将生成JSON响应，其中包含错误，HTTP状态和异常消息的详细信息。对于浏览器客户端，响应一个"whitelabel"错误视图，以HTML格式呈现相同的数据

![image-20230223110956415](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202307312050534.png)

![image-20230223110751895](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202307312050313.png)

**错误处理的自动配置**都在`ErrorMvcAutoConfiguration`中，两大核心机制：

1. SpringBoot 会**自适应**处理错误**，**响应页面或JSON数据

2. **SpringMVC的错误处理机制**依然保留，**MVC处理不了**，才会**交给boot进行处理**

![img](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202307312059215.png)

发生错误以后，转发给/error路径，SpringBoot在底层写好一个 BasicErrorController的组件，专门处理这个请求：

```java
	@RequestMapping(produces = MediaType.TEXT_HTML_VALUE) //返回HTML
	public ModelAndView errorHtml(HttpServletRequest request, HttpServletResponse response) {
		HttpStatus status = getStatus(request);
		Map<String, Object> model = Collections
			.unmodifiableMap(getErrorAttributes(request, getErrorAttributeOptions(request, MediaType.TEXT_HTML)));
		response.setStatus(status.value());
		ModelAndView modelAndView = resolveErrorView(request, response, status, model);
		return (modelAndView != null) ? modelAndView : new ModelAndView("error", model);
	}

	@RequestMapping  //返回 ResponseEntity, JSON
	public ResponseEntity<Map<String, Object>> error(HttpServletRequest request) {
		HttpStatus status = getStatus(request);
		if (status == HttpStatus.NO_CONTENT) {
			return new ResponseEntity<>(status);
		}
		Map<String, Object> body = getErrorAttributes(request, getErrorAttributeOptions(request, MediaType.ALL));
		return new ResponseEntity<>(body, status);
	}
```

错误页面是这么解析到的：

```java
//1、解析错误的自定义视图地址
ModelAndView modelAndView = resolveErrorView(request, response, status, model);
//2、如果解析不到错误页面的地址，默认的错误页就是 error
return (modelAndView != null) ? modelAndView : new ModelAndView("error", model);
```

容器中专门有一个错误视图解析器：

```java
@Bean
@ConditionalOnBean(DispatcherServlet.class)
@ConditionalOnMissingBean(ErrorViewResolver.class)
DefaultErrorViewResolver conventionErrorViewResolver() {
    return new DefaultErrorViewResolver(this.applicationContext, this.resources);
}
```

SpringBoot解析自定义错误页的默认规则：

```java
	@Override
	public ModelAndView resolveErrorView(HttpServletRequest request, HttpStatus status, Map<String, Object> model) {
		ModelAndView modelAndView = resolve(String.valueOf(status.value()), model);
		if (modelAndView == null && SERIES_VIEWS.containsKey(status.series())) {
			modelAndView = resolve(SERIES_VIEWS.get(status.series()), model);
		}
		return modelAndView;
	}

	private ModelAndView resolve(String viewName, Map<String, Object> model) {
		String errorViewName = "error/" + viewName;
		TemplateAvailabilityProvider provider = this.templateAvailabilityProviders.getProvider(errorViewName,
				this.applicationContext);
		if (provider != null) {
			return new ModelAndView(errorViewName, model);
		}
		return resolveResource(errorViewName, model);
	}

	private ModelAndView resolveResource(String viewName, Map<String, Object> model) {
		for (String location : this.resources.getStaticLocations()) {
			try {
				Resource resource = this.applicationContext.getResource(location);
				resource = resource.createRelative(viewName + ".html");
				if (resource.exists()) {
					return new ModelAndView(new HtmlResourceView(resource), model);
				}
			}
			catch (Exception ex) {
			}
		}
		return null;
	}
```

容器中有一个默认的名为 error 的 view； 提供了默认白页功能：

```java
@Bean(name = "error")
@ConditionalOnMissingBean(name = "error")
public View defaultErrorView() {
    return this.defaultErrorView;
}
```

封装了JSON格式的错误信息：

```java
	@Bean
	@ConditionalOnMissingBean(value = ErrorAttributes.class, search = SearchStrategy.CURRENT)
	public DefaultErrorAttributes errorAttributes() {
		return new DefaultErrorAttributes();
	}
```

规则：**解析一个错误页**

1. 如果发生了500、404、503、403 这些错误
   1. 如果有**模板引擎**，默认在 `classpath:/templates/error/精确码.html`
   1. 如果没有模板引擎，在静态资源文件夹下找  `精确码.html`

1. 如果匹配不到`精确码.html`这些精确的错误页，就去找`5xx.html`，`4xx.html`**模糊匹配**
   1. 如果有模板引擎，默认在 `classpath:/templates/error/5xx.html`
   1. 如果没有模板引擎，在静态资源文件夹下找  `5xx.html`
   1. 如果模板引擎路径`templates`下有 `error.html`页面，就直接渲染

### 自定义异常

#### 自定义json响应

使用`@ControllerAdvice` + `@ExceptionHandler` 进行统一异常处理

#### 自定义页面响应 

根据boot的错误页面规则，要对其进行自定义页面模板。

可以在error/下增加4xx，5xx页面，error/下的4xx，5xx页面会被自动解析。

解析顺序如果发生404错误，先找 404.html，找不到则查找4xx.html，如果都没有找到，就触发白页

![image-20230223112150202](./assets/image-20230223112150202.png)

要完全替换默认行为，可以实现 ErrorController并注册该类型的Bean定义，或添加ErrorAttributes类型的组件以使用现有机制但替换其内容。

## 嵌入式容器

**Servlet容器**：管理、运行Servlet组件（Servlet、Filter、Listener）的环境，一般指服务器

### 自动配置原理

**SpringBoot 默认嵌入Tomcat作为Servlet容器**。自动配置类是ServletWebServerFactoryAutoConfiguration，EmbeddedWebServerFactoryCustomizerAutoConfiguration

ServletWebServerFactoryAutoConfiguration 自动配置了嵌入式容器场景：

```java
@AutoConfiguration
@AutoConfigureOrder(Ordered.HIGHEST_PRECEDENCE)
@ConditionalOnClass(ServletRequest.class)
@ConditionalOnWebApplication(type = Type.SERVLET)
@EnableConfigurationProperties(ServerProperties.class)
@Import({ ServletWebServerFactoryAutoConfiguration.BeanPostProcessorsRegistrar.class,
		ServletWebServerFactoryConfiguration.EmbeddedTomcat.class,
		ServletWebServerFactoryConfiguration.EmbeddedJetty.class,
		ServletWebServerFactoryConfiguration.EmbeddedUndertow.class })
public class ServletWebServerFactoryAutoConfiguration {
    
}
```

绑定了ServerProperties配置类，所有和服务器有关的配置 server：

ServletWebServerFactoryAutoConfiguration 导入了 嵌入式的三大服务器 Tomcat、Jetty、Undertow

![image-20231224163631649](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202312241636902.png)

- 导入 Tomcat、Jetty、Undertow 都有条件注解。系统中有这个类才行 (也就是导了包)

- 默认  Tomcat配置生效。给容器中放 TomcatServletWebServerFactory

- 都给容器中 ServletWebServerFactory放了一个 web服务器工厂 (造web服务器的)

- web服务器工厂 都有一个功能，getWebServer获取web服务器

- TomcatServletWebServerFactory 创建了 tomcat。

ServletWebServerFactory 什么时候会创建 webServer出来。

ServletWebServerApplicationContextioc容器，启动的时候会调用创建web服务器

Spring容器刷新（启动）的时候，会预留一个时机，刷新子容器：onRefresh()

refresh() 容器刷新 十二大步的刷新子容器会调用 onRefresh()；

```java
@Override
protected void onRefresh() {
	super.onRefresh();
	try {
		createWebServer();
	}
	catch (Throwable ex) {
		throw new ApplicationContextException("Unable to start web server", ex);
	}
}
```

Web场景的Spring容器启动，在onRefresh的时候，会调用创建web服务器的方法。
Web服务器的创建是通过WebServerFactory搞定的。容器中又会根据导了什么包条件注解，启动相关的 服务器配置，默认EmbeddedTomcat会给容器中放一个 TomcatServletWebServerFactory，导致项目启动，自动创建出Tomcat。

### 切换容器

切换服务器只需要将默认的Tomcat依赖排除，引入需要的嵌入式容器即可：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <exclusions>
        <!-- Exclude the Tomcat dependency -->
        <exclusion>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-tomcat</artifactId>
        </exclusion>
    </exclusions>
</dependency>
<!-- Use Jetty instead -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jetty</artifactId>
</dependency>
```

## 全面接管SpringMVC

SpringBoot 默认配置好了 SpringMVC 的所有常用特性。

如果需要全面接管SpringMVC的所有配置并禁用默认配置，仅需要编写一个WebMvcConfigurer配置类，并标注 `@EnableWebMvc` 即可

**全手动模式**：

- @EnableWebMvc : 禁用默认配置
- WebMvcConfigurer组件：定义MVC的底层行为

### WebMvcAutoConfiguration 自动配置规则

SpringMVC自动配置场景配置了如下所有默认行为：**WebMvcAutoConfigurationweb场景的自动配置类**

1. 支持RESTful的filter：HiddenHttpMethodFilter
2. 支持非POST请求，请求体携带数据：FormContentFilter
3. 导入EnableWebMvcConfiguration：
   1. RequestMappingHandlerAdapter
   2. WelcomePageHandlerMapping： 欢迎页功能支持（模板引擎目录、静态资源目录放index.html），项目访问/ 就默认展示这个页面.
   3. RequestMappingHandlerMapping：找每个请求由谁处理的映射关系
   4. ExceptionHandlerExceptionResolver：默认的异常解析器 
   5. LocaleResolver：国际化解析器
   6. ThemeResolver：主题解析器
   7. FlashMapManager：临时数据共享
   8. FormattingConversionService： 数据格式化 、类型转化
   9. Validator： 数据校验JSR303提供的数据校验功能
   10. WebBindingInitializer：请求参数的封装与绑定
   11. ContentNegotiationManager：内容协商管理器
4. WebMvcAutoConfigurationAdapter配置生效，它是一个WebMvcConfigurer，定义mvc底层组件
   1. 定义好 WebMvcConfigurer 底层组件默认功能
   2. 视图解析器：InternalResourceViewResolver
   3. 视图解析器：BeanNameViewResolver,视图名（controller方法的返回值字符串）就是组件名
   4. 内容协商解析器：ContentNegotiatingViewResolver
   5. 请求上下文过滤器：RequestContextFilter: 任意位置直接获取当前请求
   6. 静态资源链规则
   7. ProblemDetailsExceptionHandler：错误详情，SpringMVC内部场景异常被它捕获
5. 定义了MVC默认的底层行为: WebMvcConfigurer

### @EnableWebMvc 禁用默认行为

`@EnableWebMVC` 禁用了 Mvc的自动配置、WebMvcConfigurer 定义SpringMVC底层组件的功能类

1. @EnableWebMvc给容器中导入 DelegatingWebMvcConfiguration组件，是 WebMvcConfigurationSupport
2. WebMvcAutoConfiguration有一个核心的条件注解，@ConditionalOnMissingBean(WebMvcConfigurationSupport.class)，容器中没有WebMvcConfigurationSupport，WebMvcAutoConfiguration才生效
3. @EnableWebMvc 导入 WebMvcConfigurationSupport 导致 WebMvcAutoConfiguration 失效。导致禁用了默认行为

### WebMvcConfigurer 功能

定义扩展SpringMVC底层功能。

| 提供方法                           | 核心参数                                | 功能                                                         | 默认                                                         |
| ---------------------------------- | --------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| addFormatters                      | FormatterRegistry                       | 格式化器：支持属性上@NumberFormat和@DatetimeFormat的数据类型转换 | GenericConversionService                                     |
| getValidator                       | 无                                      | 数据校验：校验 Controller 上使用@Valid标注的参数合法性。需要导入starter-validator | 无                                                           |
| addInterceptors                    | InterceptorRegistry                     | 拦截器：拦截收到的所有请求                                   | 无                                                           |
| configureContentNegotiation        | ContentNegotiationConfigurer            | 内容协商：支持多种数据格式返回。需要配合支持这种类型的HttpMessageConverter | 支持 json                                                    |
| configureMessageConverters         | `List<HttpMessageConverter<?>>`         | 消息转换器：标注@ResponseBody的返回值会利用MessageConverter直接写出去 | 8 个，支持byte，string,multipart,resource，json              |
| addViewControllers                 | ViewControllerRegistry                  | 视图映射：直接将请求路径与物理视图映射。用于无 java 业务逻辑的直接视图页渲染 | 无 `<mvc:view-controller>`                                   |
| configureViewResolvers             | ViewResolverRegistry                    | 视图解析器：逻辑视图转为物理视图                             | ViewResolverComposite                                        |
| addResourceHandlers                | ResourceHandlerRegistry                 | 静态资源处理：静态资源路径映射、缓存控制                     | ResourceHandlerRegistry                                      |
| configureDefaultServletHandling    | DefaultServletHandlerConfigurer         | 默认 Servlet：可以覆盖 Tomcat 的DefaultServlet。让DispatcherServlet拦截/ | 无                                                           |
| configurePathMatch                 | PathMatchConfigurer                     | 路径匹配：自定义 URL 路径匹配。可以自动为所有路径加上指定前缀，比如 /api | 无                                                           |
| configureAsyncSupport              | AsyncSupportConfigurer                  | 异步支持                                                     | TaskExecutionAutoConfiguration                               |
| addCorsMappings                    | CorsRegistry                            | 跨域                                                         | 无                                                           |
| addArgumentResolvers               | `List<HandlerMethodArgumentResolver>`   | 参数解析器                                                   | mvc 默认提供                                                 |
| addReturnValueHandlers             | `List<HandlerMethodReturnValueHandler>` | 返回值解析器                                                 | mvc 默认提供                                                 |
| configureHandlerExceptionResolvers | `List<HandlerExceptionResolver>`        | 异常处理器                                                   | 默认 3 个 ExceptionHandlerExceptionResolver ResponseStatusExceptionResolver DefaultHandlerExceptionResolver |
| getMessageCodesResolver            | 无                                      | 消息码解析器：国际化使用                                     | 无                                                           |

## 开发模式

1. **前后分离模式**： @RestController 响应JSON数据
2. **前后不分离模式**：@Controller + Thymeleaf模板引擎

## Web新特性

### Problemdetails

RFC 7807: https://www.rfc-editor.org/rfc/rfc7807

**错误信息**返回新格式

**原理**：

```java
@Configuration(proxyBeanMethods = false)
//配置过一个属性 spring.mvc.problemdetails.enabled=true
@ConditionalOnProperty(prefix = "spring.mvc.problemdetails", name = "enabled", havingValue = "true")
static class ProblemDetailsErrorHandlingConfiguration {

    @Bean
    @ConditionalOnMissingBean(ResponseEntityExceptionHandler.class)
    ProblemDetailsExceptionHandler problemDetailsExceptionHandler() {
        return new ProblemDetailsExceptionHandler();
    }

}
```

1. `ProblemDetailsExceptionHandler `是一个 `@ControllerAdvice`**集中处理系统异常**
2. 处理以下异常。如果系统出现以下异常，会被SpringBoot支持以 `RFC 7807`规范方式返回错误数据

```java
@ExceptionHandler({
    //请求方式不支持
	HttpRequestMethodNotSupportedException.class, 
    //媒体类型不支持
	HttpMediaTypeNotSupportedException.class,
	HttpMediaTypeNotAcceptableException.class,
	MissingPathVariableException.class,
	MissingServletRequestParameterException.class,
	MissingServletRequestPartException.class,
	ServletRequestBindingException.class,
	MethodArgumentNotValidException.class,
	NoHandlerFoundException.class,
	AsyncRequestTimeoutException.class,
	ErrorResponseException.class,
	ConversionNotSupportedException.class,
	TypeMismatchException.class,
	HttpMessageNotReadableException.class,
	HttpMessageNotWritableException.class,
	BindException.class
})
```

**例**：

1. problemdetails 默认是关闭的

   ```properties
   # problemdetails 默认是关闭的
   spring.mvc.problemdetails.enabled=false
   ```

2. 默认响应错误的json。状态码 405：

   > {
   >     "timestamp": "2023-04-18T11:13:05.515+00:00",
   >     "status": 405,
   >     "error": "Method Not Allowed",
   >     "trace": "org.springframework.web.HttpRequestMethodNotSupportedException: Request method 'POST' is not supported\r\n\tat org.springframework.web.servlet.mvc.method.RequestMappingInfoHandlerMapping.handleNoMatch(RequestMappingInfoHandlerMapping.java:265)\r\n\tat org.springframework.web.servlet.handler.AbstractHandlerMethodMapping.lookupHandlerMethod(AbstractHandlerMethodMapping.java:441)\r\n\tat org.springframework.web.servlet.handler.AbstractHandlerMethodMapping.getHandlerInternal(AbstractHandlerMethodMapping.java:382)\r\n\tat org.springframework.web.servlet.mvc.method.RequestMappingInfoHandlerMapping.getHandlerInternal(RequestMappingInfoHandlerMapping.java:126)\r\n\tat org.springframework.web.servlet.mvc.method.RequestMappingInfoHandlerMapping.getHandlerInternal(RequestMappingInfoHandlerMapping.java:68)\r\n\tat org.springframework.web.servlet.handler.AbstractHandlerMapping.getHandler(AbstractHandlerMapping.java:505)\r\n\tat org.springframework.web.servlet.DispatcherServlet.getHandler(DispatcherServlet.java:1275)\r\n\tat org.springframework.web.servlet.DispatcherServlet.doDispatch(DispatcherServlet.java:1057)\r\n\tat org.springframework.web.servlet.DispatcherServlet.doService(DispatcherServlet.java:974)\r\n\tat org.springframework.web.servlet.FrameworkServlet.processRequest(FrameworkServlet.java:1011)\r\n\tat org.springframework.web.servlet.FrameworkServlet.doPost(FrameworkServlet.java:914)\r\n\tat jakarta.servlet.http.HttpServlet.service(HttpServlet.java:563)\r\n\tat org.springframework.web.servlet.FrameworkServlet.service(FrameworkServlet.java:885)\r\n\tat jakarta.servlet.http.HttpServlet.service(HttpServlet.java:631)\r\n\tat org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:205)\r\n\tat org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:149)\r\n\tat org.apache.tomcat.websocket.server.WsFilter.doFilter(WsFilter.java:53)\r\n\tat org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:174)\r\n\tat org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:149)\r\n\tat org.springframework.web.filter.RequestContextFilter.doFilterInternal(RequestContextFilter.java:100)\r\n\tat org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:116)\r\n\tat org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:174)\r\n\tat org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:149)\r\n\tat org.springframework.web.filter.FormContentFilter.doFilterInternal(FormContentFilter.java:93)\r\n\tat org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:116)\r\n\tat org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:174)\r\n\tat org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:149)\r\n\tat org.springframework.web.filter.CharacterEncodingFilter.doFilterInternal(CharacterEncodingFilter.java:201)\r\n\tat org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:116)\r\n\tat org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:174)\r\n\tat org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:149)\r\n\tat org.apache.catalina.core.StandardWrapperValve.invoke(StandardWrapperValve.java:166)\r\n\tat org.apache.catalina.core.StandardContextValve.invoke(StandardContextValve.java:90)\r\n\tat org.apache.catalina.authenticator.AuthenticatorBase.invoke(AuthenticatorBase.java:493)\r\n\tat org.apache.catalina.core.StandardHostValve.invoke(StandardHostValve.java:115)\r\n\tat org.apache.catalina.valves.ErrorReportValve.invoke(ErrorReportValve.java:93)\r\n\tat org.apache.catalina.core.StandardEngineValve.invoke(StandardEngineValve.java:74)\r\n\tat org.apache.catalina.connector.CoyoteAdapter.service(CoyoteAdapter.java:341)\r\n\tat org.apache.coyote.http11.Http11Processor.service(Http11Processor.java:390)\r\n\tat org.apache.coyote.AbstractProcessorLight.process(AbstractProcessorLight.java:63)\r\n\tat org.apache.coyote.AbstractProtocol$ConnectionHandler.process(AbstractProtocol.java:894)\r\n\tat org.apache.tomcat.util.net.NioEndpoint$SocketProcessor.doRun(NioEndpoint.java:1741)\r\n\tat org.apache.tomcat.util.net.SocketProcessorBase.run(SocketProcessorBase.java:52)\r\n\tat org.apache.tomcat.util.threads.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1191)\r\n\tat org.apache.tomcat.util.threads.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:659)\r\n\tat org.apache.tomcat.util.threads.TaskThread$WrappingRunnable.run(TaskThread.java:61)\r\n\tat java.base/java.lang.Thread.run(Thread.java:833)\r\n",
   >     "message": "Method 'POST' is not supported.",
   >     "path": "/list"
   > }

3. 开启ProblemDetails返回, 使用新的MediaType：

   ```properties
   spring.mvc.problemdetails.enabled=true
   ```

   ```yaml
   spring:
     mvc:
       problemdetails:
         enabled: true
   ```

4. 新的错误返回格式：

   使用新的类型：`Content-Type: application/problem+json`+ 额外扩展返回

   ```json
   {
       "type": "about:blank",
       "title": "Method Not Allowed",
       "status": 405,
       "detail": "Method 'POST' is not supported.",
       "instance": "/list"
   }
   ```

### 函数式Web

`SpringMVC 5.2` 以后 允许使用**函数式**的方式(函数式接口)，**定义Web的请求处理流程**。

**Web请求处理的方式**：

1. `@Controller` + `@RequestMapping`：**耦合式** （**路由**、**业务**耦合）
2. **函数式Web**：分离式（路由、业务分离）

**函数式核心类**：

- **RouterFunction**：定义路由信息。发送什么请求，谁来处理
- **RequestPredicate**：定义请求：请求谓语。请求方式(GET、POST)、请求参数
- **ServerRequest**：封装请求完整数据
- **ServerResponse**：封装响应完整数据

**例**：

1. **场景**：User RESTful - CRUD

   - GET /user/1  获取1号用户

   - GET /users   获取所有用户

   - POST /user  **请求体**携带JSON，新增一个用户

   - PUT /user/1 **请求体**携带JSON，修改1号用户

   - DELETE /user/1 **删除**1号用户 

2. 容器中放入一个Bean：类型是`RouterFunction<ServerResponse>`：

   ```java
   import org.springframework.context.annotation.Bean;
   import org.springframework.context.annotation.Configuration;
   import org.springframework.http.MediaType;
   import org.springframework.web.servlet.function.RequestPredicate;
   import org.springframework.web.servlet.function.RouterFunction;
   import org.springframework.web.servlet.function.ServerResponse;
   
   import static org.springframework.web.servlet.function.RequestPredicates.accept;
   import static org.springframework.web.servlet.function.RouterFunctions.route;
   
   @Configuration(proxyBeanMethods = false)
   public class MyRoutingConfiguration {
   
       private static final RequestPredicate ACCEPT_JSON = accept(MediaType.APPLICATION_JSON);
   
       @Bean
       public RouterFunction<ServerResponse> routerFunction(MyUserHandler userHandler) {
           //链式调用
           return Rouroute()  //开始定义路由信息
                   .GET("/user/{id}", ACCEPT_JSON, userHandler::getUser)
                   .GET("/{user}/customers", ACCEPT_JSON, userHandler::getUserCustomers)
                   .DELETE("/{user}", ACCEPT_JSON, userHandler::deleteUser)
                   .build();
       }
   
   }
   ```

3. 给每个业务定义一个自己的Handler：

   ```java
   import org.springframework.stereotype.Component;
   import org.springframework.web.servlet.function.ServerRequest;
   import org.springframework.web.servlet.function.ServerResponse;
   /**
   * 专门处理User有关的业务
   */
   @Component
   public class MyUserHandler {
   
       public ServerResponse getUser(ServerRequest request) {
           ...
           return ServerResponse.ok().build();
       }
   
       public ServerResponse getUserCustomers(ServerRequest request) {
           ...
           return ServerResponse.ok().build();
       }
   
       public ServerResponse deleteUser(ServerRequest request) {
           ...
           return ServerResponse.ok().build();
       }
   
   }
   ```
