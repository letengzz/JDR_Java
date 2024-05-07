# Nacos 服务隔离

实际开发中，通常一个系统会准备：

- dev开发环境

- test测试环境

- prod生产环境。

一个大型分布式微服务系统会有很多微服务子项目，每个微服务项目又都会有相应的开发环境、测试环境、预发环境、正式环境。

**官方教程**：https://nacos.io/zh-cn/docs/architecture.htm

## 数据模型

Nacos 中的服务是由三元组唯一确定的：工作空间(namespace)、工作组(group )与服务名称(service)。

namespace 与 group 的作用是相同的，用于划分不同的区域范围，隔离服务。不同的是， namespace 的范围更大，不同的 namespace 中可以包含相同的 group。不同的 group 中可以 包含相同的 service。

namespace 的默认值为公共命名空间`public` (ID值为空串)，group 的默认值为 `DEFAULT_GROUP`。

之间的关系：

![image-20231113221556539](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202311132218676.png)

## 多环境配置和服务

在开发应用时，通常同一套程序会被运行在多个不同的环境，例如，开发环境、测试环境、预发环境、正式环境等。

每个环境的数据库地址、服务器端口号等配置都会不同。若在不同环境下运行时将配置文件修改为不同内容，那么，这种做法不仅非常繁琐，而且很容易发生错误。此时就需要定义出不同的配置信息，在不同的环境中选择不同的配置，保证指定环境启动时服务能正确读取到Nacos上相应环境的配置文件。

**多环境下的配置读取方式选择**：

- DataID：适用于项目不多，服务量少的情况
- Group：实现方式简单，但是容易与DataID方案发生冲突，仅适合于本地调试
- Namespace：实现方式简单，配置管理简单灵活，同时可以结合DataID共同使用

### Data ID方案

该方案通过指定`spring.profile.active`和配置文件的DataID来使不同环境下读取不同的配置。

Data ID它的定义规则是：`${prefix}-${spring.profile.active}.${file-extension}` (**应用名称-环境.数据格式**)

- prefix：默认为 spring.application.name 的值，也可以通过配置项 `spring.cloud.nacos.config.prefix` 来配置。
- spring.profile.active：即为当前环境对应的 profile，可以通过配置项 `spring.profile.active` 来配置。
- file-exetension：为配置内容的数据格式，可以通过配置项 `spring.cloud.nacos.config.file-extension` 来配置。目前只支持 properties 和 yaml 类型。

**注意**：当 `spring.profile.active` 为空时，对应的连接符 `-` 也将不存在，Data ID 的拼接格式变成 `prefix.file-extension`，代表此配置文件无论在什么环境下都会使用。

1. 借阅服务-生产环境：

   ![image-20231115210034140](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202311152100359.png)

2. 借阅服务-测试环境：

   ![image-20231115210157235](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202311152101050.png)

3. 使用Bootstrap时，在`application.yaml`中添加`spring-profiles-active: prod`：

   ![image-20240414122649502](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202404141226054.png)

   ****

   不使用Bootstrap时，需要修改：

   ```yaml
   spring:
     application:
       # 应用名称 borrow-service
       name: borrow-service
     profiles:
       active: prod
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
         - optional:nacos:${spring.application.name}-${spring.profiles.active}.${spring.cloud.nacos.config.file-extension}
   ```

4. 访问显示`prod` 说明访问的是正式环境的数据：

   ![image-20231115210806927](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202311152108666.png)

### Group方案

通过指定`spring.profile.active`和配置文件的`DataID`来使不同环境下读取不同的配置，也可以不用`DataID`，直接通过`Group`实现环境区分

**注意：这种方式不太推荐，切换不灵活，需要切换环境时要改Gruop配置**

**说明**：

只通过Group来进行多环境的区分的方式不推荐使用，因为涉及到了多环境自然就会改变`spring.profile.active`，而profile一旦生效，配置文件就会依据DataID的规则进行查找。所以Group的方式仅作参考。

**Group的合理用法**：配合namespace进行服务列表和配置列表的隔离和管理。

**操作步骤**：

1. 新建配置，创建配置文件Data ID为：`bookservice-dev.yaml`, Group为：`DEV_GROUP`, 其配置：

   ![image-20231115213335230](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202311152133271.png)

2. 继续创建配置文件Data ID为：`bookservice-dev.yaml`, Group为：`TEST_GROUP`, 其配置：

   ![image-20231115213544367](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202311152135540.png)

> 这里的两个配置文件他们的DataID相同但是Group不同

3. 修改项目中的配置文件`bootstrap.yaml`

   在config下增加一条group的配置，指定配置文件所在的group，可配置为`DEV_GROUP`或`TEST_GROUP`

   ```yaml
   server:
     port: 8101
   spring:
     application:
       # 服务名称和配置文件保持一致
       name: book-service
     profiles:
       # 环境也是和配置文件保持一致
       active: dev
     cloud:
       nacos:
         discovery:
           server-addr: localhost:8848
         config:
           # 配置文件后缀名
           file-extension: yaml
           # 配置中心服务器地址，也就是Nacos地址
           server-addr: localhost:8848
           group: DEV_GROUP
   ```

   ****

   不使用Bootstrap下：

   ```yaml
   spring:
     application:
       # 应用名称 borrow-service
       name: borrow-service
     profiles:
       active: dev
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
           group: DEV_GROUP
     config:
       import:
         - optional:nacos:${spring.application.name}-${spring.profiles.active}.${spring.cloud.nacos.config.file-extension}
   ```

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

   ![image-20231115214121997](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202311152141506.png)

### Namespace方案

Namespace命名空间进行环境隔离也是官方推荐的一种方式。Namespace的常用场景之一是不同环境的配置的区分隔离，例如：开发测试环境和生产环境的资源 (如配置、服务)隔离等。

![image-20231115211027020](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202311152110940.png)

创建一个新的命名空间：

![image-20231115211124341](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202311152111614.png)

可以看到在dev命名空间下，没有任何配置文件和服务：

![image-20231115211203794](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202311152112748.png)

在不同的命名空间下，实例和配置都是相互之间隔离的。

**在配置文件中指定当前的命名空间**：

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
        # 注册中心配置命名空间 namespace: 命名空间ID
        namespace: 909f6d09-f498-4b5b-be71-4d10f651f997
      config:
        file-extension: yaml
        server-addr: localhost:8848
        # 配置中心配置命名空间 namespace: 命名空间ID
        namespace: 909f6d09-f498-4b5b-be71-4d10f651f997
```

![image-20231115211924937](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202311152119701.png)

### Maven方案

使用Maven+namespace+DataID+profile的方式结合，大大提高多环境配置管理的灵活性。

**Nacos操作**：

主要采用namespace+group+dataId的格式完成三个环境的设置。

- 创建3个命名空间：

  ![image-20231115214407899](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202311152144347.png)

- 分别在不同的命名空间中创建配置，代表3个环境，并设置好需要读取的配置：

  ![image-20231115214942767](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202311152149067.png)

  ![image-20231115215733349](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202311152157716.png)

**Maven操作**：

配合maven实现环境的切换。

1. 在父工程maven配置文件设置：

  > pom.xml

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

2. application.yaml

  ```yaml
  spring:
    profiles:
      active: @profileActive@
  ```

3. bootstrap.yaml

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
          # 账号
          username: nacos
          # 密码
          password: nacos
          namespace: @nacosNamespace@
          group: @nacosGroup@
  ```

4. **注意**：分布式项目里面单个服务模块的pom.xml，**不加的话@xx@这个符号无法替换成maven配置里面的数据**：

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

5. 在maven上面出现了这样的可选记号，可以选择自己的环境，然后启动项目：

  ![image-20231115220535936](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202311152205475.png)

  ![image-20231115221354338](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202404272153879.png)

6. 切换为prod，重启项目：

  ![image-20231115220724174](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202311152207684.png)

  ![](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202311152213407.png)



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

