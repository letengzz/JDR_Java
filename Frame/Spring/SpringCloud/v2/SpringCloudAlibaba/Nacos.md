# Nacos 更加全能的注册中心

Nacos(**Na**ming **Co**nfiguration **S**ervice)是一款阿里巴巴开源的服务注册与发现、配置管理的组件，相当于是[Eureka](../Netflix/Eureka.md)+[Config](../Config.md)的组合形态。

## 安装与部署

Nacos服务器是独立安装部署的，因此需要下载最新的Nacos服务端程序。

**下载地址**：https://github.com/alibaba/nacos

![image-20230316185713790](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303301649736.png)

**注意版本问题**：https://github.com/alibaba/spring-cloud-alibaba/wiki/%E7%89%88%E6%9C%AC%E8%AF%B4%E6%98%8E

下载1.4.1版本并将文件进行解压：

![image-20230316190310059](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303301649268.png)

直接将其拖入到项目文件夹下，便于在IDEA内部启动，接着添加运行配置：

![image-20230316191505185](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303301649622.png)

其中`-m standalone`表示单节点模式，Mac和Linux下记得将解释器设定为`/bin/bash`，由于Nacos在Mac/Linux默认是后台启动模式，修改一下它的bash文件，让它变成前台启动，这样IDEA关闭了Nacos就自动关闭了，否则开发环境下很容易忘记关：

```bash
# 注释掉 nohup $JAVA ${JAVA_OPT} nacos.nacos >> ${BASE_DIR}/logs/start.out 2>&1 &
# 替换成下面的
$JAVA ${JAVA_OPT} nacos.nacos
```

接着点击启动：

![image-20230316191531850](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303301650788.png)

**注意**：如果想在命令行中启动，只需要进入nacos/bin目录，输入命令即可启动：

```bash
startup.cmd -m standalone
```

启动成功，访问这个地址：http://localhost:8848/nacos/index.html

![image-20230316191644571](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303301650931.png)

默认的用户名和管理员密码都是`nacos`，直接登陆即可，可以看到进入管理页面：

![image-20230316191724638](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303301650564.png)

至此，Nacos的安装与部署完成。

## 服务注册与发现

实现基于Nacos的服务注册与发现，需要导入SpringCloudAlibaba相关的依赖。

在父工程将依赖进行管理：

```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>2.2.0</version>
        </dependency>
      
      	<!-- 引入SpringCloud依赖 -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>2020.0.1</version>
          	<type>pom</type>
            <scope>import</scope>
        </dependency>

     	  <!-- 引入SpringCloudAlibaba依赖 -->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-alibaba-dependencies</artifactId>
            <version>2021.1</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

接着在子项目中添加服务发现依赖了，以图书服务为例：

```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
</dependency>
```

和注册到Eureka一样，需要在配置文件中配置Nacos注册中心的地址：

```yaml
server:
	# 之后所有的图书服务节点就81XX端口
  port: 8101
spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/springcloud?useSSL=false&useUnicode=true&characterEncoding=utf8&serverTimezone=Asia/Shanghai
    username: root
    password: 123123
  # 应用名称 bookservice
  application:
    name: bookservice
  cloud:
    nacos:
      discovery:
        # 配置Nacos注册中心地址
        server-addr: localhost:8848
```

接着启动图书服务，可以在Nacos的服务列表中找到：

![image-20230321201102673](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303301652292.png)

按照同样的方法，接着将另外两个服务也注册到Nacos中：

![image-20230321203348753](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303301652127.png)

接着使用OpenFeign，实现服务发现远程调用以及负载均衡，导入依赖：

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
<!-- 这里需要单独导入LoadBalancer依赖 -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-loadbalancer</artifactId>
</dependency>
```

编写接口：

```java
@FeignClient("userservice")
public interface UserClient {
    
    @RequestMapping("/user/{uid}")
    User getUserById(@PathVariable("uid") int uid);
}
```

```java
@FeignClient("bookservice")
public interface BookClient {

    @RequestMapping("/book/{bid}")
    Book getBookById(@PathVariable("bid") int bid);
}
```

```java
@Service
public class BorrowServiceImpl implements BorrowService{

    @Resource
    BorrowMapper mapper;

    @Resource
    UserClient userClient;

    @Resource
    BookClient bookClient;

    @Override
    public UserBorrowDetail getUserBorrowDetailByUid(int uid) {
        List<Borrow> borrow = mapper.getBorrowsByUid(uid);
        User user = userClient.getUserById(uid);
        List<Book> bookList = borrow
                .stream()
                .map(b -> bookClient.getBookById(b.getBid()))
                .collect(Collectors.toList());
        return new UserBorrowDetail(user, bookList);
    }
}
```

```java
@EnableFeignClients
@SpringBootApplication
public class BorrowApplication {
    public static void main(String[] args) {
        SpringApplication.run(BorrowApplication.class, args);
    }
}
```

测试：

![image-20230321203411697](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303301652462.png)

测试正常，可以自动发现服务，接着多配置几个实例，图书服务和用户服务的端口配置：

![image-20230321204338767](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303301653743.png)

> BorrowController.java
>

```java
@RestController
public class BorrowController {
    @Resource
    BorrowService service;

    @Resource
    private LoadBalancerClient loadBalancerClient;
    

    @RequestMapping("/borrow/{uid}")
    public UserBorrowDetail findUserBorrows(@PathVariable("uid") Integer uid){
        return service.getUserBorrowDetailByUid(uid);
    }


    @GetMapping("/test-load-balancer")
    public String testLoadBalancer() {
        ServiceInstance instance = loadBalancerClient.choose("userservice");
        return instance.getHost() + ":" + instance.getPort();
    }
}
```

现在将全部服务启动：

![image-20230330165835594](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303301658605.png)

可以看到Nacos中的实例数量已经显示为3：

![image-20230321205814610](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303301653254.png)

浏览器访问http://localhost:8301/test-load-balancer，刷新发现浏览器轮流显示：

![image-20230321205723401](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303301653104.png)

这样就实现了基于Nacos的服务的注册与发现，实际上大致流程与Eureka一致。

**注意**：Nacos区分了临时实例和非临时实例：

![image-20220326155010841](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303301701860.png)

- **临时实例**：和Eureka一样，采用心跳机制向Nacos发送请求保持在线状态，一旦心跳停止，代表实例下线，不保留实例信息。
- **非临时实例**：由Nacos主动进行联系，如果连接失败，那么不会移除实例信息，而是将健康状态设定为false，相当于会对某个实例状态持续地进行监控。

可以通过配置文件进行修改临时实例：

```yaml
spring:
  application:
    name: borrowservice
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848
        # 将ephemeral修改为false，表示非临时实例
        ephemeral: false
```

在Nacos中查看，可以发现实例已经不是临时的：

![image-20230321210336483](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303301701057.png)

关闭此实例：

![image-20230321210414605](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303301701216.png)

只是将健康状态变为false，而不会删除实例的信息。

## 集群分区

在一个分布式应用中，相同服务的实例可能会在不同的机器、位置上启动，比如我们的用户管理服务，可能在北京有1台服务器部署、上海有一台服务器部署，而这时，在北京的服务器上启动了借阅服务，如果借阅服务要调用用户服务，就应该优先选择同一个区域的用户服务进行调用，这样会使得响应速度更快。

![image-20220326150024118](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303301702028.png)

因此，可以对部署在不同机房的服务进行分区，可以看到实例的分区是默认：

![image-20230321211323808](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303301702799.png)

直接在配置文件中进行修改：

```yaml
spring:
  application:
    name: borrowservice
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848
        # 修改为北京地区的集群
        cluster-name: Beijing
```

由于使用的是不同的启动配置，直接在启动配置中添加环境变量`spring.cloud.nacos.discovery.cluster-name`，将用户服务和图书服务两个区域都分配一个，借阅服务就配置为北京地区：

![image-20230330170316076](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303301703634.png)

修改完成之后，重新启动服务（Nacos也要重启），观察Nacos中集群分布情况：

![image-20230321212629691](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303301703735.png)

现在有三个集群，并且都有一个实例正在运行。接着去调用借阅服务，但是发现并没有按照区域进行优先调用，而依然使用的是轮询模式的负载均衡调用。

必须要提供Nacos的负载均衡实现才能开启区域优先调用机制，只需要在配制文件中进行修改即可：

```yaml
spring:
  application:
    name: borrowservice
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848
        cluster-name: Beijing
    # 将loadbalancer的nacos支持开启，集成Nacos负载均衡
    loadbalancer:
      nacos:
        enabled: true
```

现在重启借阅服务，会发现优先调用的是同区域的用户和图书服务，现在可以将北京地区的服务下线：

![image-20230321213948924](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303301704657.png)

可以看到，在下线之后，由于本区域内没有可用服务了，借阅服务将会调用上海区域的用户服务。

除了根据区域优先调用之外，同一个区域内的实例也可以单独设置权重，Nacos会优先选择权重更大的实例进行调用，可以直接在管理页面中进行配置：

![image-20230321214135160](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303301704177.png)

或是在配置文件中进行配置：

```yml
spring:
  application:
    name: borrowservice
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848
        cluster-name: Chengdu
        # 权重大小，越大越优先调用，默认为1
        weight: 0.5
```

通过配置权重，某些性能不太好的机器就能够更少地被使用，而更多的使用那些网络良好性能更高的主机上的实例。

## 配置中心

[Spring Cloud Config](../Config.md)在`bootstrap.yml`中配置远程配置文件获取，然后再进入到配置文件加载环节，而Nacos也支持这样的操作，使用方式与Config比较类似，比如想要将借阅服务的配置文件放到Nacos进行管理，那么这个时候就需要在Nacos中创建配置文件：

![image-20230324190930429](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303301705971.png)

将借阅服务的配置文件全部复制过来注意**Data ID**的格式跟之前一样，`应用名称-环境.yml`，如果只编写应用名称，那么代表此配置文件无论在什么环境下都会使用，然后每个配置文件都可以进行分组，也算是一种分类方式：

![image-20230324191144386](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303301705393.png)

完成之后点击发布即可：

![image-20220326162122134](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303301705265.png)

在项目中导入依赖：

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

在借阅服务中添加`bootstrap.yml`文件：

```yaml
spring:
  application:
  	# 服务名称和配置文件保持一致
    name: borrowservice
  profiles:
  	# 环境也是和配置文件保持一致
    active: dev
  cloud:
    nacos:
      config:
      	# 配置文件后缀名
        file-extension: yml
        # 配置中心服务器地址，也就是Nacos地址
        server-addr: localhost:8848
```

启动服务：

![image-20230330170752677](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303301707107.png)

可以看到成功读取配置文件并启动了，实际上使用上来说跟Config是基本一致的。

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

修改配置文件，然后重启服务器：

![image-20230324192733766](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303301708021.png)

显示内容：

![image-20230324192811371](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303301708849.png)

将配置文件的值进行修改：

![image-20230324192836776](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303301708585.png)

再次访问接口：

![image-20230324193012675](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303301708425.png)

## 数据持久化

当使用默认配置启动Nacos时(单节点)，所有配置文件都被Nacos保存在自带的一个嵌入式数据库中。

> 在0.7版本之前，在单机模式时nacos使用嵌入式数据库实现数据的存储，不方便观察数据存储的基本情况。0.7版本增加了支持mysql数据源能力。

![image-20230330173535247](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/202306281454201.png)

如果使用内嵌数据库，注定会有存储上限，所以要将Nacos中的数据实现持久化。

Nacos通过集中式存储来保证数据的持久化，同时也为Nacos集群部署奠定了基础

Nacos采用了单一数据源，直接解决了分布式和集群部署中的一致性问题。

**操作步骤**：

1. 创建数据库，用于存储数据。

2. 直接导入到数据库即可，文件在conf目录中：

   ![image-20230330173934813](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303301739626.png)

   将其导入到数据库，可以看到生成了很多的表：

   ![image-20230628150410241](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/202306281504358.png)

3. 修改配置文件，Nacos-server其实就是一个Java工程或者说是一个Springboot项目，他的配置文件在`nacos\conf`目录下，名为 `application.properties`，在文件底部添加数据源配置：

   ```properties
   spring.datasource.platform=mysql
   
   db.num=1
   db.url.0=jdbc:mysql://127.0.0.1:3306/nacos?characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true
   db.user=root
   db.password=123456
   ```

4. 验证是否持久化到数据库中：启动Nacos，进入Nacos控制台，此时的Nacos控制台中焕然一新，之前的数据都不见了

   > 因为加入了新的数据源，Nacos从mysql中读取所有的配置文件，而刚刚初始化的数据库是干干净净的，自然不会有什么数据和信息显示。

   在公共空间(public)中新建一个配置文件DataID: `nacos-config.yml`, 配置内容如下：

   ```yaml
   server: 
       port: 9989
   nacos:
       config: 配置文件已持久化到数据库中...
   ```

   ![image-20230628151454965](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/202306281514070.png)

   观察数据库`mynacos`中的数据库表 `config_info` ：

   ![image-20230628151542840](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/202306281515927.png)

## 多环境配置和服务

在实际开发中，通常一个系统会准备开发环境、测试环境、预发环境、正式环境。

需要保证指定环境启动时服务能正确读取到Nacos上相应环境的配置文件。

**多环境下的配置读取方式选择**：

- DataID：适用于项目不多，服务量少的情况。
- Group：实现方式简单，但是容易与DataID方案发生冲突，仅适合于本地调试
- Namespace：实现方式简单，配置管理简单灵活，同时可以结合DataID共同使用

### Data ID方案

Data ID它的定义规则是：`${prefix}-${spring.profile.active}.${file-extension}`(`应用名称-环境.数据格式`)

- prefix：默认为 spring.application.name 的值，也可以通过配置项 `spring.cloud.nacos.config.prefix` 来配置。
- spring.profile.active：即为当前环境对应的 profile，可以通过配置项 `spring.profile.active` 来配置。
- file-exetension：为配置内容的数据格式，可以通过配置项 `spring.cloud.nacos.config.file-extension` 来配置。目前只支持 properties 和 yaml 类型。

**注意**：当 `spring.profile.active` 为空时，对应的连接符 `-` 也将不存在，Data ID 的拼接格式变成 `prefix.file-extension`，代表此配置文件无论在什么环境下都会使用。

1. 借阅服务-生产环境：![image-20230629211009426](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202306292224096.png)

2. 借阅服务-测试环境：

   ![image-20230629210934953](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202306292223610.png)

3. 修改spring-profiles-active：prod

   ![image-20230629222136359](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202306292224824.png)

4. 访问则变为prod：

   ![image-20230629222333316](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202306292224614.png)

### Group方案

通过指定`spring.profile.active`和配置文件的`DataID`来使不同环境下读取不同的配置，也可以不用`DataID`，直接通过`Group`实现环境区分

**注意：这种方式不太推荐，切换不灵活，需要切换环境时要改Gruop配置**

**说明**：

只通过Group来进行多环境的区分的方式不推荐使用，因为涉及到了多环境自然就会改变`spring.profile.active`，而profile一旦生效，配置文件就会依据DataID的规则进行查找。所以Group的方式仅作参考。

Group的合理用法应该是配合namespace进行服务列表和配置列表的隔离和管理。

**操作步骤**：

1. 新建配置，创建配置文件Data ID为：`bookservice-dev.yml`, Group为：`DEV_GROUP`, 其配置：

   ![image-20230630211640213](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202307032310557.png)

2. 继续创建配置文件Data ID为：`bookservice-dev.yml`, Group为：`TEST_GROUP`, 其配置：

   ![image-20230630211834507](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202307032310124.png)

> 这里的两个配置文件他们的DataID相同但是Group不同

3. 修改项目中的配置文件bootstrap.yml

   在config下增加一条group的配置，指定配置文件所在的group，可配置为`DEV_GROUP`或`TEST_GROUP`

   ```yaml
   server:
     port: 8101
   spring:
     application:
       # 服务名称和配置文件保持一致
       name: bookservice
     profiles:
       # 环境也是和配置文件保持一致
       active: dev
     cloud:
       nacos:
         discovery:
           server-addr: localhost:8848
         config:
           # 配置文件后缀名
           file-extension: yml
           # 配置中心服务器地址，也就是Nacos地址
           server-addr: localhost:8848
           group: DEV_GROUP
   ```

   ![image-20230630211909762](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202307032310764.png)

4. 创建TestController：

   ```java
   @RestController
   @RefreshScope   //添加此注解就能实现自动刷新了
   public class TestController {
   
       @Value("${test.txt}")  //我们从配置文件中读取test.txt的字符串值，作为test接口的返回值
       String txt;
   
       @RequestMapping("/test")
       public String test(){
           return txt;
       }
   }
   ```

5. 启动测试，分别将group配置为`DEV_GROUP`、`TEST_GROUP` 启动进行测试：

   ![image-20230630213305226](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202307032310173.png)

### Namespace方案

Namespace命名空间进行环境隔离也是官方推荐的一种方式。Namespace的常用场景之一是不同环境的配置的区分隔离，例如：开发测试环境和生产环境的资源（如配置、服务）隔离等。

![image-20230324193205881](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303301709165.png)

创建一个新的命名空间：

![image-20230324193150966](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303301709558.png)

可以看到在dev命名空间下，没有任何配置文件和服务：

![image-20230324193324963](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303301730758.png)

在不同的命名空间下，实例和配置都是相互之间隔离的。

可以**在配置文件中指定当前的命名空间**：

```yaml
spring:
  application:
    name: borrowservice
  profiles:
    active: dev   
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848
        # 配置命名空间 namespace: 命名空间ID
        namespace: 18943d38-b1c1-4192-a466-aae4cce6f46d
      config:
        file-extension: yml
        server-addr: localhost:8848
        # 配置命名空间 namespace: 命名空间ID
        namespace: 18943d38-b1c1-4192-a466-aae4cce6f46d
```

### Maven方案

使用Maven+namespace+DataID+profile的方式结合，大大提高多环境配置管理的灵活性。

**Nacos操作**：

主要采用namespace+group+dataId的格式完成三个环境的设置。

- 创建3个命名空间：

  ![image-20230630221644649](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202307032310013.png)

- 分别在不同的命名空间中创建配置，代表3个环境，并设置好需要读取的配置：

  ![image-20230630223029257](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202307032310972.png)

  ![image-20230630223227892](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202307032311119.png)

**Maven操作**：

配合maven实现环境的切换。

- pom.xml：首先在maven配置文件里面，当然了分布式在总的模块pom里面设置就可以了

  ```xml
  <!-- 统一配置测试开发生产环境 -->
  <profiles>
  	<profile>
  		<!--开发环境-->
          <id>dev</id>
          <properties>
          	<profileActive>dev</profileActive>
          	<nacosNamespace>b13163d6-ccec-4437-ac6e-f50696365d0e</nacosNamespace>
          	<nacosGroup>DEV_GROUP</nacosGroup>
          </properties>
          <activation>
              <!-- 默认情况下使用dev开发配置 如 打包时不包含 -p 参数-->
              <activeByDefault>true</activeByDefault>
          </activation>
      </profile>
      <!-- 打包命令package -P prod -->
      <profile>
          <id>prod</id>
          <properties>
              <profileActive>prod</profileActive>
              <nacosNamespace>c54bde49-1325-4b80-bb9b-c3ff76558e19</nacosNamespace>
              <nacosGroup>PROD_GROUP</nacosGroup>
          </properties>
      </profile>
      <profile>
          <id>test</id>
          <properties>
              <profileActive>test</profileActive>
              <nacosNamespace>54677409-3dd2-441d-a166-49f3f709fb85</nacosNamespace>
          	<nacosGroup>TEST_GROUP</nacosGroup>
          </properties>
      </profile>
  </profiles>
  ```
  
- application.yml

  ```yaml
  spring:
    profiles:
      active: @profileActive@
  ```

- bootstrap.yml

  ```yaml
  server:
    port: 8301
  spring:
    application:
      # 服务名称和配置文件保持一致
      name: userservice
    cloud:
      nacos:
        discovery:
          server-addr: localhost:8848
        config:
          # 配置文件后缀名
          file-extension: yml
          # 配置中心服务器地址，也就是Nacos地址
          server-addr: localhost:8848
          namespace: @nacosNamespace@
          group: @nacosGroup@
  ```

- 分布式项目里面单个服务模块的pom.xml，**不加的话@xx@这个符号无法替换成maven配置里面的数据**，这一点要注意：

  ```xml
  <build>
      <resources>
      	<resource>
              <directory>src/main/resources</directory>
          	<filtering>true</filtering>
          </resource>
  	</resources>
  </build>
  ```

- 在maven上面出现了这样的可选记号，可以选择自己的环境，然后启动项目：

  ![image-20230703200950659](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202307032012213.png)

  ![image-20230703201510809](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202307032310414.png)

- 切换为dev，重启项目：

  ![image-20230703201551809](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202307032015673.png)

  ![image-20230703201852139](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202307032018260.png)



## 多环境下管理及隔离配置和服务

当服务达到一定的数量，集中式的管理许多服务会十分不便，可以将这些具有相同特征或属性的服务进行分组管理，服务对应的配置也进行分组隔离，将服务和配置纳入相同的Namespace进行管理，不同Namespace下的服务和配置之间就隔离开来。

- 单租户方案：适合小型项目，服务数量不多时，单租户方案完全够用
- 多租户方案：适合项目量多，有一定的团队规模，且服务数量较多时，可以相对条理清晰的管理和隔离配置及服务。

### 面向一个租户

从一个租户(用户)的角度来看，如果有多套不同的环境，那么这个时候可以根据指定的环境来创建不同的 namespce，以此来实现多环境的隔离。

有dev，test和prod三个不同的环境，那么使用一套 nacos 集群可以分别建以下三个不同的 namespace：

![方案1](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202307032309888.png)

**这里的单租户同样也适于小型项目，或者是项目不太多时的实施方案**。

通过定义不同的环境，不同环境的项目在不同的Namespace下进行管理，不同环境之间通过Namespace进行隔离

当多个项目同时使用该Nacos集群时，还可以通过Group进行Namespace内的细化分组

以Namespace：dev为例，在Namespace中通过不同Group进行同一环境中不同项目的**再分类**：

![方案1内部Group分组](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202307032309892.png)

**通过Namespace来隔离不同的环境（dev\test）,在具体的环境Namespace中通过Group来管理不同的项目**

### 面向多个租户

从多个租户(用户)的角度来看，每个租户(用户)可能会有自己的 namespace,每个租户(用户)的配置数据以及注册的服务数据都会归属到自己的 namespace 下，以此来实现多租户间的数据隔离。

例如超级管理员分配了三个租户，分别为张三、李四和王五。张三负责A项目，李四负责B项目，王五负责C项目

分配好了之后，各租户用自己的账户名和密码登录后，创建自己的命名空间：

![多租户namespace](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202307032309803.png)

> 方案2通过Namespace来隔离多租户之间的服务和配置，但不仅于此，他有很好的扩展性

在该方案中，Group同样也有用武之地。

**需求改变下**，公司发展迅速业务调整，张三负责A项目、B项目、C项目，李四负责D项目、E项目、F项目，王五负责G项目、H项目、I项目,

而每个项目又分了dev、test、prod三个环境，继续沿用之前的Namespace隔离租户方案，显得有些管理不便，这时候可以在NameSpace中加入Group进行项目环境分组：

![group环境分组](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202307032311051.png)

但是当业务规模更大的时候（不考虑Nacos集群能否支持的因素），张三、李四、王五每人都负责10多个项目时，即**项目数>环境数**时，可以通过Group进行项目分组：

![Group项目分组](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202307032309863.png)

通过上面的理论分析，可以看出方案二有很好的扩展性

**Namespace隔离租户 + group环境分组**

## 实现高可用:集群

搭建Nacos集群，实现高可用。

官方方案：https://nacos.io/zh-cn/docs/cluster-mode-quick-start.html

![deployDnsVipMode.jpg](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303301730370.png)

Nacos提供了三种集群部署方案：

- http://ip1:port/openAPI 直连ip模式，机器挂则需要修改ip才可以使用。


- http://SLB:port/openAPI 挂载SLB模式(内网SLB，不可暴露到公网，以免带来安全风险)，直连SLB即可，下面挂server真实ip，可读性不好。


- http://nacos.com:port/openAPI 域名 + SLB模式(内网SLB，不可暴露到公网，以免带来安全风险)，可读性好，而且换ip方便，推荐模式

Nacos推荐在所有的服务端之前建立一个负载均衡，通过访问负载均衡服务器来间接访问到各个Nacos服务器。实际上就是比如有三个Nacos服务器做集群，但是每个服务不可能把每个Nacos都去访问一次进行注册，实际上只需要在任意一台Nacos服务器上注册即可，Nacos服务器之间会自动同步信息，但是如果我们随便指定一台Nacos服务器进行注册，如果这台Nacos服务器挂了，但是其他Nacos服务器没挂，这样就没办法完成注册了，但是实际上整个集群还是可用的状态。

所以就需要在所有Nacos服务器之前搭建一个SLB(服务器负载均衡)，这样就可以避免上面的问题了。但是如果要实现外界对服务访问的负载均衡，就得用比如Gateway来实现，而这里实际上可以用一个更加方便的工具：Nginx来实现。

SLB最上方还有一个DNS是因为SLB是裸IP，如果SLB服务器修改了地址，那么所有微服务注册的地址也得改，所以这里是通过加域名，通过域名来访问，让DNS去解析真实IP，这样就算改变IP，只需要修改域名解析记录即可，域名地址是不会变化的。

在多节点集群模式下，数据肯定是不能各存各的，所以，需要先[实现数据持久化](#数据持久化)统一存储支持，只需要让所有的Nacos服务器连接MySQL进行数据存储即可。

创建两个Nacos服务器，做一个迷你的集群，将nacos服务端上传到Linux服务器(注意需要提前安装好JRE 8或更高版本的环境)：

![image-20230706210700261](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202307070912776.png)

解压之后，对其配置文件进行修改，首先是`application.properties`配置文件，修改以下内容，包括MySQL服务器的信息：

```properties
### Default web server port:
server.port=8850

#*************** Config Module Related Configurations ***************#
### If use MySQL as datasource:
spring.datasource.platform=mysql

### Count of DB:
db.num=1

### Connect URL of DB:
db.url.0=jdbc:mysql://cloudstudy.mysql.cn-chengdu.rds.aliyuncs.com:3306/nacos?characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true&useUnicode=true&useSSL=false&serverTimezone=UTC
db.user.0=nacos
db.password.0=nacos
```

修改`conf/`下的`cluster.conf.example`文件，将其命名为`cluster.conf`：

![image-20230706210909522](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202307070912683.png)

端口记得使用内网IP地址：

![image-20230706210940856](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202307070913522.png)

修改Nacos的内存分配以及前台启动，直接修改`startup.sh`文件：

![image-20220327125049013](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202307062125509.png)

保存之后，将nacos复制一份，并将端口修改为8851：

![image-20230706211707505](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202307062117059.png)

接着启动这两个Nacos服务器：

![image-20230706212005098](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202307062125390.png)

```bash
bash nacos/bin/startup.sh
bash nacos2/bin/startup.sh
```

然后打开管理面板，可以看到两个节点都已经启动了：

![image-20230707164246230](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202307072219646.png)

接着需要添加一个SLB，用Nginx做反向代理：

> *Nginx* (engine x) 是一个高性能的[HTTP](https://baike.baidu.com/item/HTTP)和[反向代理](https://baike.baidu.com/item/反向代理/7793488)web服务器，同时也提供了IMAP/POP3/SMTP服务。它相当于在内网与外网之间形成一个网关，所有的请求都可以由Nginx服务器转交给内网的其他服务器。

直接安装：

```sh
sudo apt install nginx
```

可以看到直接请求80端口之后得到，表示安装成功：

![image-20230707153801159](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202307072218142.png)

让其代理刚刚启动的两个Nacos服务器，需要对其进行一些配置。配置文件位于`/etc/nginx/nginx.conf`，添加以下内容：

```conf
#添加我们在上游刚刚创建好的两个nacos服务器
upstream nacos-server {
        server 192.168.56.101:8850;
        server 192.168.56.101:8851;
}

server {
        listen   80;
        server_name  192.168.56.101;

        location /nacos {
                proxy_pass http://nacos-server;
        }
}
```

重启Nginx服务器(`nginx -s reload`)，成功连接：

![image-20230707165637389](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202307072218729.png)

然后将所有的服务全部修改为云服务器上Nacos的地址，启动：

![image-20230707221802665](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202307072218401.png)

这样就搭建好了Nacos集群。