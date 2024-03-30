# SpringMVC Controller控制器

有了SpringMVC之后，不必再像之前那样一个请求地址创建一个Servlet了，它使用`DispatcherServlet`替代Tomcat为我们提供的默认的静态资源Servlet，也就是说，现在所有的请求（除了jsp，因为Tomcat还提供了一个jsp的Servlet）都会经过`DispatcherServlet`进行处理。

## 配置视图解析器和控制器

SpringMVC 使用[Thymeleaf](../../../../Other/TemplateEngine/Thymeleaf/README.md)作为视图解析器实现最基本的页面解析并返回。

1. 导入依赖：

   ```xml
   <dependency>
       <groupId>org.thymeleaf</groupId>
       <artifactId>thymeleaf-spring6</artifactId>
       <version>3.1.1.RELEASE</version>
   </dependency>
   ```

2. 在配置类中将对应的`ViewResolver`注册为Bean：

   ```java
   @Configuration //让当前类成为配置类
   @EnableWebMvc   //快速配置SpringMvc注解，如果不添加此注解会导致后续无法通过实现WebMvcConfigurer接口进行自定义配置
   @ComponentScan("com.hjc.demo") //开启扫描组件
   public class WebConfiguration {
       //需要使用ThymeleafViewResolver作为视图解析器，并解析HTML页面
       @Bean
       public ThymeleafViewResolver thymeleafViewResolver(SpringTemplateEngine springTemplateEngine){
           ThymeleafViewResolver resolver = new ThymeleafViewResolver();
           resolver.setOrder(1);   //可以存在多个视图解析器，并且可以为他们设定解析顺序
           resolver.setCharacterEncoding("UTF-8");   //设置编码格式
           resolver.setTemplateEngine(springTemplateEngine);   //和之前JavaWeb阶段一样，需要使用模板引擎进行解析，所以这里也需要设定一下模板引擎
           return resolver;
       }
   
       //配置模板解析器
       @Bean
       public SpringResourceTemplateResolver templateResolver(){
           SpringResourceTemplateResolver resolver = new SpringResourceTemplateResolver();
           resolver.setSuffix(".html");   //需要解析的后缀名称
           resolver.setPrefix("/");   //需要解析的HTML页面文件存放的位置，默认是webapp目录下，如果是类路径下需要添加classpath:前缀
           resolver.setCharacterEncoding("UTF-8");
           return resolver;
       }
   
       //配置模板引擎Bean
       @Bean
       public SpringTemplateEngine springTemplateEngine(ITemplateResolver resolver){
           SpringTemplateEngine engine = new SpringTemplateEngine();
           engine.setTemplateResolver(resolver);   //模板解析器，默认即可
           return engine;
       }
   }
   ```

3. 创建一个Controller，只需在一个类上添加一个`@Controller`注解即可，它会被Spring扫描并自动注册为Controller类型的Bean，然后只需要在类中编写方法用于处理对应地址的请求即可：

   ```java
   @Controller   //直接添加注解即可
   public class HelloController {
   
       @RequestMapping("/index")   //直接填写访问路径
       public ModelAndView index(){
           return new ModelAndView("index");  //返回ModelAndView对象，这里填入了视图的名称
         	//返回后会经过视图解析器进行处理
       }
   }
   ```

4. 在类路径根目录下创建一个简单html文件：

   > index.html

   ```html
   <!DOCTYPE html>
   <html lang="en">
   <head>
       <meta charset="UTF-8">
       <title>测试</title>
   </head>
   <body>
       <p>Hello World</p>
   </body>
   </html>
   ```

5. 打开浏览器之后就可以直接访问HTML页面：

   ![image-20230901141539943](https://cdn.jsdelivr.net/gh/letengzz/tc2@main/img/Java/202309011415274.png)

## ModelAndView

使用Thymeleaf解析后端的一些数据时，需要通过Context进行传递，而使用SpringMvc后，数据可以直接向Model模型层进行提供：

```java
@RequestMapping(value = "/index")
public ModelAndView index(ModelAndView modelAndView){
	//返回ModelAndView对象，这里填入了视图的名称
//        ModelAndView modelAndView = new ModelAndView("index1");
//        ModelAndView modelAndView = new ModelAndView();
    modelAndView.getModel().put("name", "hjc");   //将name传递给Model
    modelAndView.setViewName("index");
    return modelAndView;
}
```

这样Thymeleaf就能收到传递的数据进行解析：

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
    HelloWorld！
    <div th:text="${name}"></div>
</body>
</html>
```

打开浏览器之后就可以直接访问HTML页面：

![image-20230901142120218](https://cdn.jsdelivr.net/gh/letengzz/tc2@main/img/Java/202309011421271.png)

## View

可以直接返回View名称，SpringMVC会将其自动包装为ModelAndView对象：

```java
@RequestMapping(value = "/index")
public String index(){
    return "index";
}
```

可以单独添加一个Model作为形参进行设置，SpringMVC通过依赖注入会自动传递实例对象：

```java
@RequestMapping(value = "/index")
public String index(Model model){  //这里不仅仅可以是Model，还可以是Map、ModelMap
    HashMap<String, Object> map = new HashMap<>();
    map.put("name", "yyds");
    model.addAllAttributes(map);
//        model.addAttribute("name", "yyds");
	return "index";
}
```

![image-20230901143446859](https://cdn.jsdelivr.net/gh/letengzz/tc2@main/img/Java/202309011434598.png)

## 静态资源

页面中可能还会包含一些静态资源，比如js、css，因此还需要配置让静态资源通过Tomcat提供的默认Servlet进行解析，需要让配置类实现一下`WebMvcConfigurer`接口，这样在Web应用程序启动时，会根据重写方法里面的内容进行进一步的配置：

```java
@Override
public void configureDefaultServletHandling(DefaultServletHandlerConfigurer configurer) {
    configurer.enable();   //开启默认的Servlet
}

@Override
public void addResourceHandlers(ResourceHandlerRegistry registry) {
    registry.addResourceHandler("/static/**").addResourceLocations("/static/");
    //配置静态资源的访问路径
}
```

编写前端内容：

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>测试</title>
    <!-- 引用静态资源，这里使用Thymeleaf的网址链接表达式，Thymeleaf会自动添加web应用程序的名称到链接前面 -->
    <script th:src="@{/static/test.js}"></script>
</head>
<body>
    <p>Hello World</p>
    <p>你好 世界</p>
    <div th:text="${name}"></div>
</body>
</html>
```

创建`test.js`并编写如下内容：

```javascript
window.alert("你好 欢迎")
```

最后访问页面，页面在加载时就会显示一个弹窗，这样就完成了最基本的页面配置。相比之前的方式，这样就简单很多了，直接避免了编写大量的Servlet来处理请求。

![image-20230901144046854](https://cdn.jsdelivr.net/gh/letengzz/tc2@main/img/Java/202309011440389.png)

## @RequestMapping

以后需要在控制器添加一个方法用于处理对应的请求即可，之前需要完整地编写一个Servlet来实现，而现在只需要添加一个`@RequestMapping`即可实现，**此注解就是将请求和处理请求的方法建立一个映射关系**，当收到请求时就可以根据映射关系调用对应的请求处理方法。

`@RequestMapping`注解定义：

```java
@Mapping
public @interface RequestMapping {
    String name() default "";

    @AliasFor("path")
    String[] value() default {};

    @AliasFor("value")
    String[] path() default {};

    RequestMethod[] method() default {};

    String[] params() default {};

    String[] headers() default {};

    String[] consumes() default {};

    String[] produces() default {};
}
```

### path/value属性

其中最关键的是path属性 (等价于value)，它决定了当前方法处理的请求路径，**注意路径必须全局唯一**，任何路径只能有一个方法进行处理，它是一个数组，也就是说此方法不仅仅可以只用于处理某一个请求路径，可以使用此方法处理多个请求路径：

```java
@RequestMapping({"/index", "/test"})
public ModelAndView index(){
    return new ModelAndView("index");
}
```

现在访问`/index`或是`/test`都会经过此方法进行处理。

****

可以直接将`@RequestMapping`添加到类名上，表示为此类中的所有请求映射添加一个路径前缀：

```java
@Controller
@RequestMapping("/yyds")
public class MainController {

    @RequestMapping({"/index", "/test"})
    public ModelAndView index(){
        return new ModelAndView("index");
    }
}
```

那么现在需要访问`/yyds/index`或是`/yyds/test`才可以得到此页面。

可以直接在IDEA下方的端点板块中查看当前Web应用程序定义的所有请求映射，并且可以通过IDEA为我们提供的内置Web客户端直接访问某个路径。

![image-20230901152310519](https://cdn.jsdelivr.net/gh/letengzz/tc2@main/img/Java/202309011523990.png)

**路径支持使用通配符进行匹配**：

- `?`：表示任意一个字符，比如`@RequestMapping("/index/x?")`可以匹配/index/xa、/index/xb等等。
- `*`：表示任意0-n个字符，比如`@RequestMapping("/index/*")`可以匹配/index/lbwnb、/index/yyds等。
- `**`：表示当前目录或基于当前目录的多级目录，比如`@RequestMapping("/index/**")`可以匹配/index、/index/xxx等。

### method属性

method属性规定请求的方法类型，限定请求方式：

```java
@RequestMapping(value = "/index", method = RequestMethod.POST)
public ModelAndView index(){
    return new ModelAndView("index");
}
```

如果直接使用浏览器访问此页面，会显示405方法不支持，因为浏览器默认是直接使用GET方法获取页面，而路径指定为POST方法访问此地址，所以访问失败，再去端点中用POST方式去访问，成功得到页面。
![image-20230901153059377](https://cdn.jsdelivr.net/gh/letengzz/tc2@main/img/Java/202309011531776.png)

![image-20230901153041421](https://cdn.jsdelivr.net/gh/letengzz/tc2@main/img/Java/202309011530807.png)

### params属性

可以使用`params`属性来指定请求必须携带哪些请求参数：

```java
@RequestMapping(value = "/index", params = {"username", "password"})
public ModelAndView index(){
    return new ModelAndView("index");
}
```

请求中必须携带`username`和`password`属性，否则无法访问。它还支持表达式：

```java
@RequestMapping(value = "/index", params = {"!username", "password"})
public ModelAndView index(){
    return new ModelAndView("index");
}
```

在username之前添加一个感叹号表示请求的不允许携带此参数，否则无法访问，可以直接设定一个固定值：

```java
@RequestMapping(value = "/index", params = {"username!=test", "password=123"})
public ModelAndView index(){
    return new ModelAndView("index");
}
```

请求参数username不允许为test，并且password必须为123，否则无法访问。

### headers属性

`headers`属性用法与`params`一致，但是它要求的是请求头中需要携带什么内容，比如：

```java
@RequestMapping(value = "/index", headers = "!Connection")
public ModelAndView index(){
    return new ModelAndView("index");
}
```

那么，如果请求头中携带了`Connection`属性，将无法访问。其他两个属性：

- consumes： 指定处理请求的提交内容类型（Content-Type），例如application/json, text/html;
- produces:  指定返回的内容类型，仅当request请求头中的(Accept)类型中包含该指定类型才返回；

### 衍生注解

除了使用[method属性](#method属性)，还可以使用衍生注解来指定请求方式。

衍生注解主要有：

- `@GetMapping`：直接指定为GET请求方式
- `@PostMapping`：直接指定为POST请求方式
- `@PutMapping`：直接指定为PUT请求方式
- `@DeleteMapping`：直接指定为DELETE请求方式
- `@PatchMapping`：直接指定为PATCH请求方式

可以使用衍生注解直接设定为指定类型的请求映射：

```java
@PostMapping(value = "/index")
public ModelAndView index(){
    return new ModelAndView("index");
}
```

使用`@PostMapping`直接指定为POST请求类型的请求映射，同样的，还有`@GetMapping`可以直接指定为GET请求方式，这里就不一一列举了。

## @RequestParam和@RequestHeader

获取到请求中的参数只需要为方法添加一个形式参数，并在形式参数前面添加`@RequestParam`注解：

```java
@RequestMapping(value = "/index")
public ModelAndView index(@RequestParam("username") String username){
    System.out.println("接受到请求参数："+username);
    return new ModelAndView("index");
}
```

需要在`@RequestParam`中填写参数名称，参数的值会自动传递给形式参数，可以直接在方法中使用。

**注意**：如果参数名称与形式参数名称相同，即使不添加`@RequestParam`也能获取到参数值。

```java
@RequestMapping(value = "/index")
public ModelAndView index(String username){
    System.out.println("接受到请求参数："+username);
    return new ModelAndView("index");
}
```

一旦添加`@RequestParam`，那么此请求必须携带指定参数，可以将require属性设定为false来将属性设定为非必须：

```java
@RequestMapping(value = "/index")
public ModelAndView index(@RequestParam(value = "username", required = false) String username){
    System.out.println("接受到请求参数："+username);
    return new ModelAndView("index");
}
```

可以直接设定一个默认值，当请求参数缺失时，可以直接使用默认值：

```java
@RequestMapping(value = "/index")
public ModelAndView index(@RequestParam(value = "username", required = false, defaultValue = "hjc") String username){
    System.out.println("接受到请求参数："+username);
    return new ModelAndView("index");
}
```

如果需要使用Servlet原本的一些类：

```java
@RequestMapping(value = "/index")
public ModelAndView index(HttpServletRequest request){
    System.out.println("接受到请求参数："+request.getParameterMap().keySet());
    return new ModelAndView("index");
}
```

直接添加`HttpServletRequest`为形式参数即可，SpringMVC会自动传递该请求原本的`HttpServletRequest`对象，同理，我们也可以添加`HttpServletResponse`作为形式参数，甚至可以直接将HttpSession也作为参数传递：

```java
@RequestMapping(value = "/index")
public ModelAndView index(HttpSession session){
    System.out.println(session.getAttribute("test"));
    session.setAttribute("test", "hjc");
    return new ModelAndView("index");
}
```

可以直接将请求参数传递给一个实体类：

```java
@Data
public class User {
    String username;
    String password;
}
```

**注意**：必须携带set方法或是构造方法中包含所有参数，请求参数会自动根据类中的字段名称进行匹配：

```java
@RequestMapping(value = "/index")
public ModelAndView index(User user){
    System.out.println("获取到值为："+user);
    return new ModelAndView("index");
}
```

****

`@RequestHeader`与`@RequestParam`用法一致，不过它是用于获取请求头参数的。

## @CookieValue和@SessionAttrbutie

通过使用`@CookieValue`注解，可以快速获取请求携带的Cookie信息：

```java
@RequestMapping(value = "/index")
public ModelAndView index(HttpServletResponse response,
                          @CookieValue(value = "test", required = false) String test){
    System.out.println("获取到cookie值为："+test);
    response.addCookie(new Cookie("test", "hjc"));
    return new ModelAndView("index");
}
```

同样的，Session也能使用注解快速获取：

```java
@RequestMapping(value = "/index")
public ModelAndView index(@SessionAttribute(value = "test", required = false) String test, HttpSession session){
    session.setAttribute("test", "hjc");
    System.out.println(test);
    return new ModelAndView("index");
}
```

## 重定向和请求转发

使用SpringMVC，只需要在视图名称前面添加一个前缀就可以实现重定向和请求转发。

通过添加`redirect:`前缀，就可以很方便地实现重定向：

```java
@RequestMapping("/index")
public String index(){
    return "redirect:home";
}

@RequestMapping("/home")
public String home(){
    return "home";
}
```

使用`forward:`前缀表示转发给其他请求映射：

```java
@RequestMapping("/index")
public String index(){
    return "forward:home";
}

@RequestMapping("/home")
public String home(){
    return "home";
}
```

## Bean的Web作用域

SpringMVCBean的作用域除了`singleton`和`prototype`还存在：

- request：对于每次HTTP请求，使用request作用域定义的Bean都将产生一个新实例，请求结束后Bean也消失。
- session：对于每一个会话，使用session作用域定义的Bean都将产生一个新实例，会话过期后Bean也消失。
- global session：不常用，不做讲解。

**例**：

1. 创建一个测试类：

   ```java
   public class TestBean {
   
   }
   ```

2. 接着将其注册为Bean，注意这里需要添加`@RequestScope`或是`@SessionScope`表示此Bean的Web作用域：

   ```java
   @Bean
   @RequestScope
   public TestBean testBean(){
       return new TestBean();
   }
   ```

3. 将其自动注入到Controller中：

   ```java
   @Controller
   public class MainController {
   
       @Autowired
       TestBean bean;
   
       @RequestMapping(value = "/index")
       public ModelAndView index(){
           System.out.println(bean);
           return new ModelAndView("index");
       }
   }
   ```

每次发起得到的Bean实例都不同，接着将其作用域修改为`@SessionScope`，这样作用域就上升到Session，只要清理浏览器的Cookie，那么都会被认为是同一个会话，只要是同一个会话，那么Bean实例始终不变。

实际上，它也是通过代理实现的，调用Bean中的方法会被转发到真正的Bean对象去执行。