# Consul 服务配置管理

## 服务配置

**添加依赖**：

```xml
<!--SpringCloud consul config-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-consul-config</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-bootstrap</artifactId>
</dependency>
```

新建bootstrap.yml：

Spring Cloud会创建一个“Bootstrap Context”，作为Spring应用的`Application Context`的父上下文。初始化的时候，`Bootstrap Context`负责从**外部源**加载配置属性并解析配置。这两个上下文共享一个从外部获取的`Environment`。

`Bootstrap`属性有高优先级，默认情况下，它们不会被本地配置覆盖。 `Bootstrap context`和`Application Context`有着不同的约定，所以新增了一个`bootstrap.yml`文件，保证`Bootstrap Context`和`Application Context`配置的分离。

application.yml文件改为bootstrap.yml，这是很关键的或者两者共存

因为bootstrap.yml是比application.yml先加载的。bootstrap.yml优先级高于application.yml

applicaiton.yml 与 bootstrap.yml **区别**：

- applicaiton.yml是用户级的资源配置项

- bootstrap.yml是系统级的，优先级更加高

> bootstrap.yml

```yaml
spring:
  application:
    name: provider-user
    ####Spring Cloud Consul for Service Discovery
  cloud:
    consul:
      host: localhost
      port: 8500
      discovery:
        service-name: ${spring.application.name}
      config:
        profile-separator: '-' # default value is ","，we update '-'
        format: YAML

# config/provider-user/data
#       /provider-user-dev/data
#       /provider-user-prod/data
```

> application.yml

```yaml
server:
  port: 8101
spring:
  profiles:
    active: dev # 多环境配置加载内容dev/prod,不写就是默认default配置
```

在Consul上key/value配置填写：

**填写规则**：

![image-20240306154123295](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202403061541480.png)

创建config文件夹(以`/`结尾)：

![image-20240306154306078](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202403061543910.png)

在config文件夹下分别创建其他3个文件夹(以`/`结尾)：

![image-20240306154523697](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202403061545125.png)

在上述3个文件夹下分别创建data内容，data不再是文件夹：

![image-20240306155303557](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202403061553453.png)

运行项目：

![image-20240306155741461](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202403061557639.png)

## 配置文件热更新

Consul支持配置文件的热更新，比如在配置文件中添加了一个属性，而这个时候可能需要实时修改，并在后端实时更新：

创建一个新的Controller：

```java
@RestController
@RefreshScope   //添加此注解实现自动刷新
public class TestController {
    
    @Value("${test.txt}")  //从配置文件中读取test.txt的字符串值，作为test接口的返回值
    String txt;
    
    @RequestMapping("/test")
    public String test(){
        return txt;
    }
}
```

修改并发布配置文件，然后重启服务：

![image-20240306160537341](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202403061605598.png)

访问 http://localhost:8101/test 显示内容：

![image-20240306160504774](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202403061605357.png)

将配置文件的值进行修改并发布：

![image-20240306160605874](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202403061606339.png)

再次访问接口：

![image-20240306160627252](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202403061606062.png)
