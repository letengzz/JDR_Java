# SpringMVC 介绍

Spring Web MVC是基于Servlet API构建的原始Web框架，从一开始就包含在Spring Framework中。正式名称"Spring Web MVC"来自其源模块的名称 (`spring-webmvc`) ，但它通常被称为 "Spring MVC"。

在控制层框架历经Strust、WebWork、Strust2等诸多产品的历代更迭之后，目前业界普遍选择了SpringMVC作为Java EE项目表述层开发的**首选方案**。

**SpringMVC 具备显著优势**：

-   **Spring 家族原生产品**，与IOC容器等基础设施无缝对接
-   表述层各细分领域需要解决的问题**全方位覆盖**，提供**全面解决方案**
-   **代码清新简洁**，大幅度提升开发效率
-   内部组件化程度高，可插拔式组件**即插即用**，想要什么功能配置相应组件即可
-   **性能卓著**，尤其适合现代大型、超大型互联网项目要求

官方：https://docs.spring.io/spring-framework/reference/web/webmvc.html

****

原生Servlet API开发代码片段：

```java
protected void doGet(HttpServletRequest request, HttpServletResponse response) 
                                                        throws ServletException, IOException {  
    String userName = request.getParameter("userName");
    
    System.out.println("userName="+userName);
}
```

基于SpringMVC开发代码片段

```java
@RequestMapping("/user/login")
public String login(@RequestParam("userName") String userName,Sting password){
    
    log.debug("userName="+userName);
    //调用业务即可
    
    return "result";
}
```

## 主要作用

主要是SSM框架构建起单体项目的技术栈需求。SpringMVC负责表述层 (控制层)实现简化。

![](https://cdn.jsdelivr.net/gh/letengzz/tc2@main/img/Java/202309181614857.png)

因此，SpringMVC正是为了解决这种问题而生的，它是一个非常优秀的表示层框架，采用MVC思想设计实现。

SpringMVC的作用主要覆盖的是**表述层**：

-   请求映射
-   数据输入
-   视图界面
-   请求分发
-   表单回显
-   会话控制
-   过滤拦截
-   异步交互
-   文件上传
-   文件下载
-   数据校验
-   类型转换
-   等等等

**主要作用**：

1.  简化前端参数接收 (形参列表)
2.  简化后端数据响应 (返回值)
3.  以及其他......

## 核心组件和调用流程

Spring MVC与许多其他Web框架一样，是围绕前端控制器模式设计的，其中中央 `Servlet`  `DispatcherServlet` 做整体请求处理调度。

除了`DispatcherServlet`SpringMVC还会提供其他特殊的组件协作完成请求处理和响应呈现。

**SpringMVC处理请求流程**：

当请求到达Tomcat服务器之后，会交给当前的Web应用程序进行处理，而SpringMVC使用`DispatcherServlet`来处理所有的请求，也就是说它被作为一个统一的访问点，所有的请求全部由它来进行调度。

当一个请求经过`DispatcherServlet`之后，会先走`HandlerMapping`，它会将请求映射为`HandlerExecutionChain`，依次经过`HandlerInterceptor`有点类似于过滤器，不过在SpringMVC中使用的是拦截器，然后再交给`HandlerAdapter`，根据请求的路径选择合适的控制器进行处理，控制器处理完成之后，会返回一个`ModelAndView`对象，包括数据模型和视图，通俗的讲就是页面中数据和页面本身（只包含视图名称即可）。

返回`ModelAndView`之后，会交给`ViewResolver`（视图解析器）进行处理，视图解析器会对整个视图页面进行解析，SpringMVC自带了一些视图解析器，但是只适用于JSP页面，也可以像之前一样使用Thymeleaf作为视图解析器，这样就可以根据给定的视图名称，直接读取HTML编写的页面，解析为一个真正的View。

解析完成后，就需要将页面中的数据全部渲染到View中，最后返回给`DispatcherServlet`一个包含所有数据的成形页面，再响应给浏览器，完成整个过程。

因此，实际上整个过程只需要编写对应请求路径的的Controller以及配置好需要的ViewResolver即可，之后还可以继续补充添加拦截器，而其他的流程已经由SpringMVC帮助完成。

![SpringMVC 运行过程 (2)](https://cdn.jsdelivr.net/gh/letengzz/tc2@main/img/Java/202309011350852.png)

**说明**：

1. DispatcherServlet :  SpringMVC提供，需要使用web.xml配置使其生效，它是整个流程处理的核心，所有请求都经过它的处理和分发 (**设置处理所有请求**)

2. HandlerMapping :  SpringMVC提供，需要进行IoC配置使其加入IoC容器方可生效，它内部缓存handler(controller方法)和handler访问路径数据，被DispatcherServlet调用，用于查找路径对应的handler (**需要加入到IoC容器，供DispatcherServlet 调用**)

3. HandlerAdapter : SpringMVC提供，需要进行IoC配置使其加入IoC容器方可生效，它可以处理请求参数和处理响应数据数据，每次DispatcherServlet都是通过handlerAdapter间接调用handler，他是handler和DispatcherServlet之间的适配器 (**需要加入到IoC容器，供DispatcherServlet 调用**)****

4. Handler : handler又称处理器，他是Controller类内部的方法简称，是由我们自己定义，用来接收参数，向后调用业务，最终返回响应结果 (**需要配置到HandlerMapping中供DispatcherServlet 查找**)

5. ViewResovler : SpringMVC提供，需要进行IoC配置使其加入IoC容器方可生效。视图解析器主要作用简化模版视图页面查找的

   **注意**：前后端分离项目，后端只返回JSON数据，不返回页面，那就不需要视图解析器。所以，视图解析器，相对其他的组件不是必须的。