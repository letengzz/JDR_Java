# SpringBoot 整合 Thymeleaf

- [Thymeleaf 详解](../../../../../../Other/TemplateEngine/Thymeleaf/README.md)

**导入依赖**：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```

**自动配置原理**：

1. 开启了 org.springframework.boot.autoconfigure.thymeleaf.ThymeleafAutoConfiguration 自动配置
2. 属性绑定在 ThymeleafProperties 中，对应配置文件 `spring.thymeleaf` 内容
3. 所有的模板页面默认在 `classpath:/templates`文件夹下
4. **默认效果**：
   1. 所有的模板页面在 `classpath:/templates/`下面找
   2. 找后缀名为`.html`的页面


**操作步骤**：

- 导入依赖后，在classpath:/templates文件夹下创建一个hello.html

  ![image-20230805232317490](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202308052323262.png)

- 创建一个controller：

  ```java
  @Controller
  public class HelloController {
      @GetMapping("/hello")
      public String hello(){
          return "hello";
      }
  }
  ```

- 启动并访问：http://localhost:8080/hello

  ![image-20230805234225081](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202401061913735.png)

- 修改hello.html

  ```html
  <h1>Hello <span th:text="${name}"></span></h1>
  ```

- 修改HelloController：

  ```java
  @Controller // 适配 服务端渲染 前后不分离模式开始
  public class HelloController {
      /**
       * 利用模板引擎跳转到指定页面
       * @param name
       * @param model
       * @return
       */
      @GetMapping("/hello")
      public String hello(@RequestParam("name") String name,
                          Model model){
          //把需要给页面共享的数据放到model中
          model.addAttribute("name",name);
          //模板的逻辑视图名
          //物理视图 = 前缀 + 逻辑视图名 + 后缀
          //真实地址: classpath:/templates/hello.html
          return "hello";
      }
  }
  ```

- 启动并访问：http://localhost:8080/hello?name=zhangsan

  ![image-20230805234754395](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202308052347373.png)

