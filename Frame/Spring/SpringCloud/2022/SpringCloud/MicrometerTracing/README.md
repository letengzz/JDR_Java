# Micrometer Tracing

Micrometer 提供了一套完整的分布式链路追踪 (`Distributed Tracing`)解决方案且兼容支持了zipkin展现。

将一次分布式请求还原成调用链路，进行日志记录和性能监控，并将一次分布式请求的调用情况集中web展示。

官方网站：https://micrometer.io/

官方文档：https://docs.micrometer.io/micrometer/reference/

![image-20240414141425786](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202404141414494.png)

## 分布式链路追踪原理

假定3个微服务调用链路：Service1调用Service2，Service2调用Service3和Service4

![](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202404141419334.png)

一条链路追踪会在每个服务调用的时候加上Trace ID 和 Span ID

链路通过TraceId唯一标识，Span标识发起的请求信息，各span通过parent id 关联起来 (Span:表示调用链路来源，通俗的理解span就是一次请求信息)

![](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202404141419882.png)

**总结**：一条链路通过Trace Id唯一标识，Span标识发起的请求信息，各span通过parent id 关联起来

![](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202404141420839.png)

![image-20240414142052680](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202404141619127.png)

## Zipkin 概述

Zipkin是一种分布式链路跟踪系统**图形化工具**，Zipkin 是 Twitter 开源的分布式跟踪系统，能够收集微服务运行过程中的实时调用链路信息，并能够将这些调用链路信息展示到Web图形化界面上供开发人员分析，开发人员能够从ZipKin中分析出调用链路中的性能瓶颈，识别出存在问题的应用程序，进而定位问题和解决问题。

**官方网站**：http://zipkin.io

![image-20240414143330884](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202404141433738.png)

## Zipkin 安装

官网下载：https://zipkin.io/pages/quickstart

支持3个方式：使用Java命令方式、Docker、通过源码安装

### 使用Java命令安装

下载Jar包：

![image-20240414143915512](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202404141619568.png)

使用 `java -jar zipkin-server-3.2.1-exec.jar`进行安装：

![image-20240414144408220](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202404141444368.png)

运行控制台：http://localhost:9411/zipkin

## Micrometer + ZipKin 搭建链路监控

**Micrometer + ZipKin 两者各自的分工**：

- Micrometer：数据采集
- ZipKin：图形展示

### 添加依赖

**总体依赖**：

![image-20240414151932541](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202404141519449.png)

在总体父工程中，添加依赖：

```xml
<properties>
    <micrometer-tracing.version>1.2.0</micrometer-tracing.version>
	<zipkin-reporter-brave.version>2.17.0</zipkin-reporter-brave.version>
</properties>
<dependencyManagement>
    <dependencies>
        <!--micrometer-tracing-bom导入链路追踪版本中心-->
        <dependency>
            <groupId>io.micrometer</groupId>
            <artifactId>micrometer-tracing-bom</artifactId>
            <version>${micrometer-tracing.version}</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
        <!--zipkin-reporter-brave-->
        <dependency>
            <groupId>io.zipkin.reporter2</groupId>
            <artifactId>zipkin-reporter-brave</artifactId>
            <version>${zipkin-reporter-brave.version}</version>
        </dependency>
    </dependencies>
</dependencyManagement>
```

### 服务提供者

添加依赖：

```xml
<dependencies>
    <!-- SpringBoot框架的一个模块用于监视和管理应用程序 -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
    <!--micrometer-tracing指标追踪-->
    <dependency>
        <groupId>io.micrometer</groupId>
        <artifactId>micrometer-tracing</artifactId>
    </dependency>
    <!--micrometer-tracing-bridge-brave适配zipkin的桥接包-->
    <dependency>
        <groupId>io.micrometer</groupId>
        <artifactId>micrometer-tracing-bridge-brave</artifactId>
    </dependency>
    <!--micrometer-observation-->
    <dependency>
        <groupId>io.micrometer</groupId>
        <artifactId>micrometer-observation</artifactId>
    </dependency>
    <!--feign-micrometer-->
    <dependency>
        <groupId>io.github.openfeign</groupId>
        <artifactId>feign-micrometer</artifactId>
    </dependency>
    <!--zipkin-reporter-brave-->
    <dependency>
        <groupId>io.zipkin.reporter2</groupId>
        <artifactId>zipkin-reporter-brave</artifactId>
    </dependency>
</dependencies>
```

配置文件：`application.yaml`

```yaml
# ========================zipkin===================
management:
  zipkin:
    tracing:
      endpoint: http://localhost:9411/api/v2/spans
  tracing:
    sampling:
      probability: 1.0 #采样率默认为0.1(0.1就是10次只能有一次被记录下来)，值越大收集越及时。
```

添加业务类：

```java
@RestController
public class MicrometerController
{
    /**
     * Micrometer(Sleuth)进行链路监控的例子
     * @param id
     * @return
     */
    @GetMapping(value = "/user/micrometer/{id}")
    public String myMicrometer(@PathVariable("id") Integer id)
    {
        return "Hello, 欢迎到来myMicrometer inputId:  "+id+" \t    服务返回:" + UUID.randomUUID().toString();
    }
}
```

### 服务调用者

添加依赖：

```xml
<dependencies>
    <!-- SpringBoot框架的一个模块用于监视和管理应用程序 -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
    <!--micrometer-tracing指标追踪-->
    <dependency>
        <groupId>io.micrometer</groupId>
        <artifactId>micrometer-tracing</artifactId>
    </dependency>
    <!--micrometer-tracing-bridge-brave适配zipkin的桥接包-->
    <dependency>
        <groupId>io.micrometer</groupId>
        <artifactId>micrometer-tracing-bridge-brave</artifactId>
    </dependency>
    <!--micrometer-observation-->
    <dependency>
        <groupId>io.micrometer</groupId>
        <artifactId>micrometer-observation</artifactId>
    </dependency>
    <!--feign-micrometer-->
    <dependency>
        <groupId>io.github.openfeign</groupId>
        <artifactId>feign-micrometer</artifactId>
    </dependency>
    <!--zipkin-reporter-brave-->
    <dependency>
        <groupId>io.zipkin.reporter2</groupId>
        <artifactId>zipkin-reporter-brave</artifactId>
    </dependency>
</dependencies>
```

配置文件：`application.yaml`

```yaml
server:
  port: 8401
# zipkin图形展现地址和采样率设置
management:
  zipkin:
    tracing:
      endpoint: http://localhost:9411/api/v2/spans
  tracing:
    sampling:
      probability: 1.0 #采样率默认为0.1(0.1就是10次只能有一次被记录下来)，值越大收集越及时。
```

调用服务提供者：

```java
@FeignClient(value = "user-service",path = "/user")
public interface UserClient {
    /**
     * Micrometer(Sleuth)进行链路监控的例子
     * @param id
     * @return
     */
    @GetMapping(value = "/micrometer/{id}")
    public String myMicrometer(@PathVariable("id") Integer id);
}
```

添加业务类：

```java
@RestController
@Slf4j
public class MicrometerController
{
    @Resource
    private UserClient userClient;

    @GetMapping(value = "/feign/micrometer/{id}")
    public String myMicrometer(@PathVariable("id") Integer id)
    {
        return userClient.myMicrometer(id);
    }
}
```

### 测试效果

启动ZipKin、服务提供者、服务调用者，打开地址：http://localhost:8401/feign/micrometer/1

![image-20240414154917251](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202404141549013.png)

访问：http://localhost:9411

![image-20240414161103107](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202404141611206.png)

点击"SHOW"查看链路：

![image-20240414161253283](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202404141612035.png)

查看依赖关系：

![image-20240414161741753](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202404141617453.png)