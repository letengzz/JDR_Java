# Nacos 服务配置管理

**Nacos 配置中心工作原理**：

![image-20231113224413025](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202311132244345.png)

**优点**：

- Config Client 通知自动感知配置中心中相应配置文件的更新。
- 架构简单 
- 支持的配置文件类型较多 (支持 JSON 与 YAML)

## 配置管理

Nacos支持配置远程配置文件获取，然后再进入到配置文件加载，使用方式是将服务的配置文件放到Nacos进行管理，那么这个时候就需要在Nacos中创建配置文件：

![image-20231113225747758](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202311132257817.png)

将各个服务的配置文件复制过来并将原来的 `spring.application.name` 属性删除。

**注意**：**Data ID**的格式：`应用名称.yaml`

![image-20231113234548248](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202311132345780.png)

完成之后点击发布即可：

![image-20231113234612687](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202311132346075.png)

在项目中导入依赖：

```xml
<!--nacos config 依赖-->
<dependency>
	<groupId>com.alibaba.cloud</groupId>
 	<artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
</dependency>
```

重写修改application.yaml内容：

```yaml
spring:
  application:
    # 应用名称 borrow-service
    name: borrow-service
  cloud:
    nacos:
      config:
        server-addr: localhost:8848
        file-extension: yaml
        # 用户名
        username: nacos
        # 密码
        password: nacos
  config:
    import:
      - optional:nacos:${spring.application.name}.${spring.cloud.nacos.config.file-extension}
```

重新启动程序：

![image-20231114131645348](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202311141316198.png)

****

**使用Bootstrap**：

1. 添加依赖：

   ```xml
   <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-bootstrap</artifactId>
   </dependency>
   <dependency>
       <groupId>com.alibaba.cloud</groupId>
       <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
   </dependency>
   ```

2. 添加配置文件：**Data ID**的格式：`应用名称.yaml`

   ![image-20231115133537166](assets/image-20231115133537166.png)

3. 在借阅服务中删除`applicaiton.yaml`添加`bootstrap.yaml`文件：

   > bootstrap.yaml

   ```yaml
   spring:
     application:
     	# 服务名称和配置文件保持一致
       name: borrow-service
     cloud:
       nacos:
         config:
         	# 配置文件后缀名
           file-extension: yaml
           # 配置中心服务器地址，也就是Nacos地址
           server-addr: localhost:8848
           # 用户名
           username: nacos
           # 密码
           password: nacos
   ```

4. 启动服务，可以看到成功读取配置文件并启动了：

   ![image-20231115133645869](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202311151336764.png)

## 配置文件拓展

当前服务配置、共享配置与扩展配置的加载顺序为：共享配置，扩展配置，当前服务配 置。若在三个配置中具有相同属性设置，但它们具有不同的值，那么，后加载的会将先加载 的给覆盖。即这三类配置的优先级由低到高是：共享配置，扩展配置，当前服务配置

```yaml
spring:
  application:
    # 应用名称 borrow-service
    name: borrow-service
  cloud:
    nacos:
      config:
        server-addr: localhost:8848
        file-extension: yaml
        username: nacos
        password: nacos

        # 同一个group中的不同服务可以共享以下 "共享配置"
        shared-configs[0]:
          data-id: shareconfig.yaml
          refresh: true
        # 不同group 中的不同服务可以共享以下 "扩展配置"
        extension-configs[0]:
          data-id: extconfig.yaml
          refresh: true
  config:
    import:
      - optional:nacos:${spring.application.name}.${spring.cloud.nacos.config.file-extension}
```

## 当前服务配置优先级

当前服务配置可以存在于三个地方：

- 远程配置文件 (Nacos config 中)：

  ![image-20231114205959290](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202311142100608.png)

- 快照文件：

  ![image-20231114210106968](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202311142101647.png)

- 本地配置文件(主动写入的配置文件)：

  ![image-20231114212230588](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202311142122554.png)

这三个同名文件也存在加载顺序问题，它们的加载顺序为**：本地配置文件、远程配置文件、快照配置文件**。只要系统加载到了配置文件，那么后面的就不再加载。

## 配置文件热更新

Nacos还支持配置文件的热更新，比如在配置文件中添加了一个属性，而这个时候可能需要实时修改，并在后端实时更新：

创建一个新的Controller：

```java
@RestController
@RefreshScope   //添加此注解实现自动刷新
public class TestController {
    
    @Value("${test.txt}")  //我们从配置文件中读取test.txt的字符串值，作为test接口的返回值
    String txt;
    
    @RequestMapping("/test")
    public String test(){
        return txt;
    }
}
```

修改并发布配置文件，然后重启服务：

![image-20231114132835369](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202311141328525.png)

访问 http://localhost:8301/test 显示内容：

![image-20231114133221083](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202311141332079.png)

将配置文件的值进行修改并发布：

![](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202311141333897.png)

再次访问接口：

![image-20231114133531447](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202311141335353.png)

## 长轮询模型

Nacos Config Server 中配置数据的变更，Nacos Config Client 采用的是长轮询模型。 

长轮询模型整合了 Push 与 Pull 模型的优势。Client 仍定时发起 Pull 请求，查看 Server 端数据是否更新。若发生了更新，则 Server 立即将更新数据以响应的形式 Push 给 Client 端；

若没有发生更新，Server 端并不向 Client 进行 Push，而是临时性的保持住这个连接一段时间。 若在此时间段内，Server 端数据发生了变更，则立即将变更数据 Push 给 Client。若仍未发生 变更，则放弃这个连接。等待着下一次 Client 的 Pull 请求。 长轮询模型，是 Push 与 Pull 模型的整合，**既降低了 Push 模型中长连接的维护问题，又 降低了 Push 模型实时性较低的问题**。

