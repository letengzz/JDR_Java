# 静态资源访问

## 静态资源规则

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

## 欢迎页

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

## Favorites Icon

favicon.ico 放在静态资源目录下即可。

**注意**：⾃定义资源路径后，Favicon 功能失效。

![image-20230804205715095](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202308042057630.png)

## webjar

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

## 自定义静态资源规则

自定义静态资源路径、自定义缓存规则

### 配置方式

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

### 代码方式

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

## 缓存

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

