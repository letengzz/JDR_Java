# 默认效果

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