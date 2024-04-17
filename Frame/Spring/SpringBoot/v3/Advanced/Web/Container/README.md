# 嵌入式容器

**Servlet容器**：管理、运行Servlet组件 (Servlet、Filter、Listener) 的环境，一般指服务器

## 自动配置原理

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

Spring容器刷新（启动）的时候，会预留一个时机，刷新子容器：`onRefresh()`

`refresh()` 容器刷新 十二大步的刷新子容器会调用 `onRefresh()`；

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

## 切换容器

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

