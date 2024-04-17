# 异常处理

## 默认错误处理

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

## 自定义异常

### 自定义json响应

使用`@ControllerAdvice` + `@ExceptionHandler` 进行统一异常处理

```java
@ControllerAdvice
public class ErrorController {

    @ExceptionHandler(Exception.class)
    public String error(Exception e, Model model){  //可以直接添加形参来获取异常
        e.printStackTrace();
        model.addAttribute("e", e);
        return "error";
    }
}
```

> templates/error.html

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
  500 - 服务器出现了一个内部错误QAQ
  <div th:text="${e}"></div>
</body>
</html>
```

添加异常：

```java
@RequestMapping("/index")
public String index(){
    System.out.println("我是处理！");
    if(true) throw new RuntimeException("您的氪金力度不足，无法访问！");
    return "index";
}
```

访问后，发现控制台会输出异常信息，同时页面也是自定义的一个页面。

![image-20230919130750542](https://cdn.jsdelivr.net/gh/letengzz/tc2@main/img/Java/202309191308819.png)

### 自定义页面响应

根据boot的错误页面规则，要对其进行自定义页面模板。

可以在error/下增加4xx，5xx页面，error/下的4xx，5xx页面会被自动解析。

解析顺序如果发生404错误，先找 404.html，找不到则查找4xx.html，如果都没有找到，就触发白页

![image-20230223112150202](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202403231846413.png)

要完全替换默认行为，可以实现 ErrorController并注册该类型的Bean定义，或添加ErrorAttributes类型的组件以使用现有机制但替换其内容。