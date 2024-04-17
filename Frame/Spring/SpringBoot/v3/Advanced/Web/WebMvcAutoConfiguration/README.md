# WebMvcAutoConfiguration原理

SpringBoot启动默认加载  xxxAutoConfiguration 类（自动配置类）

SpringMVC功能的自动配置类 WebMvcAutoConfiguration生效。

## 生效条件

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

## 效果

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

## WebMvcConfigurer接口

提供了配置SpringMVC底层的所有组件入口。

![img](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202308011535530.png)

## 静态资源规则源码

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

## EnableWebMvcConfiguration 源码

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


## WebMvcConfigurationSupport

提供了很多的默认设置。判断系统中是否有相应的类：如果有，就加入相应的`HttpMessageConverter`

```java
jackson2Present = ClassUtils.isPresent("com.fasterxml.jackson.databind.ObjectMapper", classLoader) &&
				ClassUtils.isPresent("com.fasterxml.jackson.core.JsonGenerator", classLoader);
jackson2XmlPresent = ClassUtils.isPresent("com.fasterxml.jackson.dataformat.xml.XmlMapper", classLoader);
jackson2SmilePresent = ClassUtils.isPresent("com.fasterxml.jackson.dataformat.smile.SmileFactory", classLoader);
```
