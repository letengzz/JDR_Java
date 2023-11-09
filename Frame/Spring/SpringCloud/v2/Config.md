# Config 配置中心

**官方文档：**https://docs.spring.io/spring-cloud-config/docs/current/reference/html/

在Spring Boot项目中，默认会提供一个application.properties或者application.yml文件，把一些全局性的配置或者需要动态维护的配置写入改文件，比如数据库连接，功能开关，限流阈值，服务地址等。

为了解决不同环境下服务连接配置等信息的差异，Spring Boot还提供了基于`spring.profiles.active={profile}`的机制来实现不同的环境的切换。

随着单体架构向微服务架构的演进，各个应用自己独立维护本地配置文件的方式开始显露出它的不足之处。

主要体现：

- 配置的动态更新：在实际应用会有动态更新位置的需求，比如修改服务连接地址、限流配置等。在传统模式下，需要手动修改配置文件并且重启应用才能生效，这种方式效率太低，重启也会导致服务暂时不可用。
- 配置多节点维护：在微服务架构中某些核心服务为了保证高性能会部署上百个节点，如果在每个节点中都维护一个配置文件，一旦配置文件中的某个属性需要修改，可想而知，工作量是巨大的。
- 不同部署环境下配置的管理：前面提到通过profile机制来管理不同环境下的配置，这种方式对于日常维护来说也比较繁琐。

统一配置管理就是弥补上述不足的方法，简单说，最基本的方法是把各个应用系统中的某些配置放在一个第三方中间件上进行统一维护。然后，对于统一配置中心上的数据的变更需要推送到相应的服务节点实现动态更新。

微服务架构中，Spring Cloud Config 为分布式系统中的外部配置提供服务器端和客户端支持。使用 Config Server，您可以集中管理所有环境中应用程序的外部配置。

![image-20220325171754862](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303281421721.png)

实际上Spring Cloud Config就是一个配置中心，所有的服务都可以从配置中心取出配置，而配置中心又可以从GitHub远程仓库中获取云端的配置文件，这样只需要修改GitHub中的配置即可对所有的服务进行配置管理了。

## 部署配置中心

创建一个新的项目，导入依赖：

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-config-server</artifactId>
    </dependency>
  	<dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
    </dependency>
</dependencies>
```

启动类：

```java
@SpringBootApplication
public class ConfigApplication {
    public static void main(String[] args) {
        SpringApplication.run(ConfigApplication.class, args);
    }
}
```

配置文件：

```yaml
server:
  port: 8700
spring:
  application:
    name: configserver
eureka:
  client:
    service-url:
      defaultZone: http://eureka:8888/eureka,http://eureka01:8801/eureka,http://eureka02:8802/eureka
```

启动：

![image-20230316155235821](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303281421641.png)

以本地仓库为例，首先在项目目录下创建一个本地Git仓库，打开终端，在桌面上创建一个新的本地仓库：

![image-20230316155630431](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303281421890.png)

在文件夹中随便创建一些配置文件，注意名称最好是{服务名称}-{环境}.yml：

![image-20230316160444459](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303281421794.png)

将图书微服务中的连接数据库剪切到bookservice-dev.yml：

![image-20230316160953872](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303281421829.png)

添加文件并提交到仓库：

![image-20230316161203584](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303281421051.png)

启动类添加`@EnableConfigServer`：

```java
@SpringBootApplication
@EnableConfigServer
public class ConfigApplication {
    public static void main(String[] args) {
        SpringApplication.run(ConfigApplication.class, args);
    }
}
```

在配置文件中，添加本地仓库的一些信息（远程仓库同理），详细使用教程：https://docs.spring.io/spring-cloud-config/docs/current/reference/html/#_git_backend

**注意**：Config Server默认存储配置的⽅式是git，如果git仓库是公开仓库，username和password属性可以省略不配置，否则必须配置username和password。

```yaml
server:
  port: 8700
spring:
  application:
    name: configserver
  cloud:
    config:
      server:
        git:
          # 配置⽂件所在的git仓库 这里填写的是本地仓库地址，远程仓库直接填写远程仓库地址 http://git...
          uri: D:\Develop\JC-JavaPro\SpringCloud\Config\config-repo
          # 配置文件所在目录
          # search-paths: config
          # 默认分支设定为你自己本地或是远程分支的名称
          default-label: master
        #redis:
```

启动配置服务器，通过以下格式进行访问：

- http://localhost:8700/{服务名称}/{环境}/{Git分支}
- http://localhost:8700/{Git分支}/{服务名称}-{环境}.yml

访问图书服务的生产环境代码，可以使用 http://localhost:8700/bookservice/dev/master 链接，它会显示详细信息：

![image-20230321171435381](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303281421152.png)

也可以使用 http://localhost:8700/master/bookservice-dev.yml 链接，它仅显示配置文件原文：

![image-20230321171403606](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303281421832.png)

当然，除了使用Git来保存之外，还支持一些其他的方式，详细情况请查阅官网。

## 客户端配置

配置客户端，服务既然需要从服务器读取配置文件，那么就需要进行一些配置，我们删除原来的`application.yml`文件（也可以保留，最后无论是远端配置还是本地配置都会被加载），改用`bootstrap.yml`（在application.yml之前加载，可以实现配置文件远程获取）：

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-config</artifactId>
</dependency>

<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-bootstrap</artifactId>
    <version>3.1.0</version>
</dependency>
```

> bootstrap.yml

```yaml
spring:
  cloud:
    config:
    	# 名称，其实就是文件名称
      name: bookservice
      # 配置服务器的地址
      uri: http://localhost:8700
      # 环境
      profile: dev
      # 分支
      label: master
```

配置完成之后，启动图书服务：

![image-20230321175357108](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303281421211.png)

已经从远端获取到了配置，并进行启动。

