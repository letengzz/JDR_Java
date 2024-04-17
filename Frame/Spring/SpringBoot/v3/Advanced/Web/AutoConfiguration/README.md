# SpringMVC自动配置

SpringBoot 大多场景都无需自定义配置，包括：

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

## 自动配置过程

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