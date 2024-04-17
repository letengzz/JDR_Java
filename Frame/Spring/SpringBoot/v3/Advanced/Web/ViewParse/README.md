# 视图解析与模板引擎

由于 **SpringBoot** 使用了**嵌入式 Servlet 容器**。所以 **JSP** 默认是**不能使用**的。

如果需要**服务端页面渲染**，优先考虑使用 模板引擎。

![img](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202307312026519.png)

模板引擎页面默认放在 `src/main/resources/templates`

**SpringBoot** 包含以下模板引擎的自动配置：

- FreeMarker
- Groovy
- [Thymeleaf](../../../Integration/Thymeleaf/README.md)
- Mustache