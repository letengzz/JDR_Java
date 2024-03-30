# SpringMVC 前后端分离开发

现在推荐前后端分离的开发模式，而不是所有的内容全部交给后端渲染再发送给浏览器，也就是说，整个Web页面的内容在一开始就编写完成了，而其中的数据由前端执行JS代码来向服务器动态获取，再到前端进行渲染（填充），这样可以大幅度减少后端的压力，并且后端只需要传输关键数据即可

##  传输格式

既然要实现前后端分离，就必须约定一种更加高效的数据传输模式，来向前端页面传输后端提供的数据。因此JSON横空出世，它非常容易理解，并且与前端的兼容性极好，因此现在比较主流的数据传输方式则是通过JSON格式承载的。

- [JSON](../../../../Other/DataExchangeFormat/JSON/README.md)

## 后端操作

后端可以以JSON字符串的形式向前端返回数据，这样前端在拿到数据之后，就可以快速获取，非常方便。

后端使用 [JSON解析框架](../../../../../../Other/DataExchangeFormat/JSON/Frame/README.md) 快速创建一个JSON格式的数据。

JSON解析框架有很多种，比较常用的是Jackson和FastJSON，这里使用阿里巴巴的FastJSON进行解析，这是目前号称最快的JSON解析框架，并且现在已经强势推出FastJSON 2版本。

### 使用阿里巴巴的FastJSON进行解析

- 导入依赖：

  ```xml
  <dependency>
        <groupId>com.alibaba.fastjson2</groupId>
        <artifactId>fastjson2</artifactId>
        <version>2.0.34</version>
  </dependency>
  ```

- 使用JSONObject (表示JSON对象)，它和Map的使用方法一样，并且是有序的 (实现了LinkedHashMap接口)：

  ```java
  @RequestMapping(value = "/index")
  public String index(){
      JSONObject object = new JSONObject();
      object.put("name", "杰哥");
      object.put("age", 18);
      System.out.println(object.toJSONString());   //以JSON格式输出JSONObject字符串
      return "index";
  }
  ```

  ![image-20230920132738608](https://cdn.jsdelivr.net/gh/letengzz/tc2@main/img/Java/202309201356492.png)

- 使用 JSONArray (表示JSON数组)，它和List的使用方法一样，可以嵌套其他的JSONObject或是JSONArray：

  ```java
  @RequestMapping(value = "/index")
  public String index(){
      JSONObject object = new JSONObject();
      object.put("name", "杰哥");
      object.put("age", 18);
      JSONArray array = new JSONArray();
      array.add(object);
      System.out.println(array.toJSONString());
      return "index";
  }
  ```

  ![image-20230920134358376](https://cdn.jsdelivr.net/gh/letengzz/tc2@main/img/Java/202309201356503.png)

当出现循环引用时，会按照以下语法来解析：

![img](https://cdn.jsdelivr.net/gh/letengzz/tc2@main/img/Java/202309201412739.png)

可以也直接创建一个实体类，将实体类转换为JSON格式的数据：

```java
@RequestMapping(value = "/index", produces = "application/json")
@ResponseBody
public String data(){
    Student student = new Student();
    student.setName("杰哥");
    student.setAge(18);
    return JSON.toJSONString(student);
}
```

这里修改了`produces`的值，将返回的内容类型设定为`application/json`，表示服务器端返回了一个JSON格式的数据（当然不设置也行，也能展示，这样是为了规范）然后在方法上添加一个`@ResponseBody`表示方法返回（也可以在类上添加`@RestController`表示此Controller默认返回的是字符串数据）的结果不是视图名称而是直接需要返回一个字符串作为页面数据，这样，返回给浏览器的就是直接返回的字符串内容。

接着使用JSON工具类将其转换为JSON格式的字符串，打开浏览器，得到JSON格式数据。

****

SpringMVC非常智能，可以直接返回一个对象类型，它会被自动转换为JSON字符串格式：

```java
@RequestMapping(value = "/data", produces = "application/json")
@ResponseBody
public Student data(){
    Student student = new Student();
    student.setName("杰哥");
    student.setAge(18);
    return student;
}
```

**注意**：需要在配置类中添加一下FastJSON转换器，这里需要先添加一个依赖：

```xml
<dependency>
    <groupId>com.alibaba.fastjson2</groupId>
    <artifactId>fastjson2-extension-spring6</artifactId>
    <version>2.0.34</version>
</dependency>
```

编写配置：

```java
@Override
public void configureMessageConverters(List<HttpMessageConverter<?>> converters) {
    converters.add(new FastJsonHttpMessageConverter());
}
```

再次尝试，内容就会自动转换为JSON格式响应给客户端了。

## 前端操作

![img](https://cdn.jsdelivr.net/gh/letengzz/tc2@main/img/Java/202309201622519.png)

前端需要用到异步请求，因为网页是动态的（这里的动态不是指有动画效果，而是能够实时更新内容）比如点击一个按钮会弹出新的内容、或是跳转到新的页面、更新页面中的数据等等，这些都需要通过JS完成异步请求来实现。

> 前端异步请求指的是在前端中发送请求至服务器或其他资源，并且不阻塞用户界面或其他操作。在传统的同步请求中，当发送请求时，浏览器会等待服务器响应，期间用户无法进行其他操作。而异步请求通过将请求发送到后台，在等待响应的同时，允许用户继续进行其他操作。这种机制能够提升用户体验，并且允许页面进行实时更新。常见的前端异步请求方式包括使用XMLHttpRequest对象、Fetch API、以及使用jQuery库中的AJAX方法，以及目前最常用的Axios框架等。

### 使用 Axios 异步请求

假设后端有一个需要实时刷新的数据（随时间而变化）现在需要再前端实时更新展示，以axios框架的使用，发起异步请求并更新页面中的数据。

前端页面：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>测试</title>
    <script src="https://unpkg.com/axios@1.1.2/dist/axios.min.js"></script>
</head>
<body>
  <p>欢迎来到GayHub全球最大交友网站</p>
  <p>用户名: <span id="username"></span></p>
  <p>密码: <span id="password"></span></p>
</body>
</html>
```

接着使用axios框架直接对后端请求JSON数据 (**注意路径**)：

```html
<script>
    function getInfo() {
        axios.get('/mvc/test').then(({data}) => {
            document.getElementById('username').innerText = data.username
            document.getElementById('password').innerText = data.password
        })
    }
</script>
```

这样就实现了从服务端获取数据并更新到页面中，前端开发者利用JS发起异步请求，可以实现各种各样的效果，而后端开发者只需要关心接口返回正确的数据即可。

****

向服务端发送一个JS对象数据并进行解析，这里以简单的登录为例：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>测试</title>
    <script src="https://unpkg.com/axios@1.1.2/dist/axios.min.js"></script>
</head>
<body>
  <p>欢迎来到GayHub全球最大交友网站</p>
  <button onclick="login()">立即登录</button>
</body>
</html>
```

这里依然使用axios发送POST请求：

```html
<script>
    function login() {
        axios.post('/mvc/test', {
            username: 'test',
            password: '123456'
        }, {
            headers: {
                'Content-Type': 'application/x-www-form-urlencoded'
            }
        }).then(({data}) => {
            if(data.success) {
                alert('登录成功')
            } else {
                alert('登录失败')
            }
        })
    }
</script>
```

服务器端只需要在请求参数位置添加一个对象接收即可（和前面是一样的，因为这里也是提交的表单数据）：

```java
@ResponseBody
@PostMapping(value = "/test", produces = "application/json")
public String hello(String username, String password){
    boolean success = "test".equals(user.getUsername()) && "123456".equals(user.getPassword());
    JSONObject object = new JSONObject();
    object.put("success", success);
    return object.toString();
}
```

也可以将js对象转换为JSON字符串的形式进行传输，这里需要使用ajax方法来处理：

```html
<script>
    function login() {
        axios.post('/mvc/test', {
            username: 'test',
            password: '123456'
        }).then(({data}) => {
            if(data.success) {
                alert('登录成功')
            } else {
                alert('登录失败')
            }
        })
    }
</script>
```

如果需要读取前端发送JSON格式数据，那么这个时候就需要添加`@RequestBody`注解：

```java
@ResponseBody
@PostMapping(value = "/test", produces = "application/json")
public String hello(@RequestBody User user){
    boolean success = "test".equals(user.getUsername()) && "123456".equals(user.getPassword());
    JSONObject object = new JSONObject();
    object.put("success", success);
    return object.toString();
}
```

这样，就实现了前后端使用JSON字符串进行通信。