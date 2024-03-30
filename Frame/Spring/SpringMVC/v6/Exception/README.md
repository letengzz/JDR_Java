# SpringMVC 异常处理

当请求映射方法中出现异常时，会直接展示在前端页面，这是因为SpringMVC提供了默认的异常处理页面，当出现异常时，请求会被直接转交给专门用于异常处理的控制器进行处理。

## 自定义异常处理

自定义一个异常处理控制器，一旦出现指定异常，就会转接到此控制器执行：

```java
@ControllerAdvice
public class ErrorController {

    @ExceptionHandler(Exception.class)
    public String error(Exception e, Model model){  //可以直接添加形参来获取异常
        e.printStackTrace();
        model.addAttribute("e", e);
        return "500";
    }
}
```

接着编写一个专门显示异常的页面：

**说明**：需要[配置视图解析器](../Controller/README.md#配置视图解析器和控制器)

> 500.html

```java
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