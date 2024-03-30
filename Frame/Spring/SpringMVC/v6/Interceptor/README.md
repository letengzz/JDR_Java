# SpringMVC 拦截器

拦截器(Interceptor)是整个SpringMVC的一个重要内容，拦截器与过滤器类似，都是用于拦截一些非法请求，JavaWeb的过滤器是作用于Servlet之前，只有经过层层的过滤器才可以成功到达Servlet，而拦截器并不是在Servlet之前，它在Servlet与RequestMapping之间，相当于DispatcherServlet在将请求交给对应Controller中的方法之前进行拦截处理，它只会拦截所有Controller中定义的请求映射对应的请求（不会拦截静态资源）。

![image-20230630194651686](https://cdn.jsdelivr.net/gh/letengzz/tc2@main/img/Java/202309191209417.png)

## 创建拦截器

创建一个拦截器需要实现一个`HandlerInterceptor`接口：

```java
public class MainInterceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        System.out.println("我是处理之前！");
        return true;   //只有返回true才会继续，否则直接结束
    }

    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        System.out.println("我是处理之后！");
    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
      	//在DispatcherServlet完全处理完请求后被调用
        System.out.println("我是完成之后！");
    }
}
```

接着需要在配置类中进行注册：

```java
@Configuration
@EnableWebMvc   //快速配置SpringMvc注解，如果不添加此注解会导致后续无法通过实现WebMvcConfigurer接口进行自定义配置
@ComponentScan("com.hjc.demo")
public class WebConfiguration implements WebMvcConfigurer {
	@Override
	public void addInterceptors(InterceptorRegistry registry) {
    	registry.addInterceptor(new MainInterceptor())
      	.addPathPatterns("/**")    //添加拦截器的匹配路径，只要匹配一律拦截
      	.excludePathPatterns("/home");   //拦截器不进行拦截的路径
	}
}
```

在浏览器中访问index页面，拦截器已经生效。

得到拦截器的执行顺序：

![image-20230919122819986](https://cdn.jsdelivr.net/gh/letengzz/tc2@main/img/Java/202309191228064.png)

处理前和处理后，包含了真正的请求映射的处理，在整个流程结束后还执行了一次`afterCompletion`方法，在处理前，只需要返回true或是false表示是否被拦截即可。

当处理前返回false：

```java
@Override
public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
	System.out.println("我是处理之前！");
    return false;   //只有返回true才会继续，否则直接结束
}
```

![image-20230919123012949](https://cdn.jsdelivr.net/gh/letengzz/tc2@main/img/Java/202309191230628.png)

通过结果发现一旦返回false，之后的所有流程全部取消。

****

如果处理过程中抛出异常，那么就不会执行处理后`postHandle`方法，但是会执行`afterCompletion`方法，可以在此方法中获取到抛出的异常：

```java
@RequestMapping("/index")
public String index(){
    System.out.println("我是处理！");
    if(true) throw new RuntimeException("");
    return "index";
}
```

运行结果：

![image-20230919123317274](https://cdn.jsdelivr.net/gh/letengzz/tc2@main/img/Java/202309191233424.png)

## 多级拦截器

如果存在多个拦截器会如何执行，以同样的方式创建二号拦截器：

```java
public class SubInterceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        System.out.println("二号拦截器：我是处理之前！");
        return true;
    }

    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        System.out.println("二号拦截器：我是处理之后！");
    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        System.out.println("二号拦截器：我是完成之后！");
    }
}
```

注册二号拦截器：

```java
@Override
public void addInterceptors(InterceptorRegistry registry) {
  	//一号拦截器
    registry.addInterceptor(new MainInterceptor()).addPathPatterns("/**").excludePathPatterns("/home");
  	//二号拦截器
    registry.addInterceptor(new SubInterceptor()).addPathPatterns("/**");
}
```

**注意**：拦截顺序就是注册的顺序，因此拦截器会根据注册顺序依次执行。

运行结果：

![image-20230919123805606](https://cdn.jsdelivr.net/gh/letengzz/tc2@main/img/Java/202309191238620.png)

和多级Filter相同，在处理之前，是按照顺序从前向后进行拦截的，但是处理完成之后，就按照倒序执行处理后方法，而完成后是在所有的`postHandle`执行之后再同样的以倒序方式执行。

****

与单个拦截器的情况一样，一旦拦截器返回false，那么之后无论有无拦截器，都不再继续：

```java
@Override
public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
    System.out.println("一号拦截器：我是处理之前！");
    return false;
}
```

运行结果：

![image-20230919123904035](https://cdn.jsdelivr.net/gh/letengzz/tc2@main/img/Java/202309191239323.png)
