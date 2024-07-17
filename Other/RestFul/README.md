# RestFul风格

REST 是 Representational State Transfer的缩写，如果⼀个架构符合REST原则，就称它为 RESTful架构。RESTful架构可以充分的利用 HTTP 协议的各种功能，是 HTTP 协议的最佳实践。 RESTful API 是⼀种软件架构风格、设计风格，可以让软件更加清晰，更简洁，更有层次，可维护性更好。

中文释义为"**表现层状态转换**"。它的主要作用是充分并正确利用HTTP协议的特性，规范资源获取的URI路径。通俗的讲，RESTful风格的设计允许将参数通过URL拼接传到服务端，目的是让URL看起来更简洁实用，并且可以充分使用多种HTTP请求方式（POST/GET/PUT/DELETE），来执行相同请求地址的不同类型操作。

因此，这种风格的连接，可以直接从请求路径中读取参数：`http://localhost:8080/mvc/index/123456`

![image-20230301091133844](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202312241441195.png)

可以直接将index的下一级路径作为请求参数进行处理，也就是说现在的请求参数包含在了请求路径中：

```java
@RequestMapping("/index/{str}")
public String index(@PathVariable String str) {
    System.out.println(str);
    return "index";
}
```

**注意**：请求路径可以手动添加类似占位符一样的信息，这样占位符位置的所有内容都会被作为请求参数，而方法的形参列表中必须包括一个与占位符同名的并且添加了`@PathVariable`注解的参数，或是由`@PathVariable`注解指定为占位符名称：

```java
@RequestMapping("/index/{str}")
public String index(@PathVariable("str") String text){
    System.out.println(text);
    return "index";
}
```

如果没有配置正确，方法名称上会出现黄线。

****

可以按照不同功能进行划分：

- 添加用户信息，携带表单数据：`POST http://localhost:8080/mvc/index`
- 获取用户信息，id直接放在请求路径中：`GET http://localhost:8080/mvc/index/{id}`
- 修改用户信息，携带表单数据：`PUT http://localhost:8080/mvc/index`  
- 删除用户信息，id直接放在请求路径中：`DELETE http://localhost:8080/mvc/index/{id}`

分别编写四个请求映射：

```java
@Controller
public class MainController {

    @RequestMapping(value = "/index/{id}", method = RequestMethod.GET)
    public String get(@PathVariable("id") String text){
        System.out.println("获取用户："+text);
        return "index";
    }

    @RequestMapping(value = "/index", method = RequestMethod.POST)
    public String post(String username){
        System.out.println("添加用户："+username);
        return "index";
    }

    @RequestMapping(value = "/index/{id}", method = RequestMethod.DELETE)
    public String delete(@PathVariable("id") String text){
        System.out.println("删除用户："+text);
        return "index";
    }

    @RequestMapping(value = "/index", method = RequestMethod.PUT)
    public String put(String username){
        System.out.println("修改用户："+username);
        return "index";
    }
}
```
