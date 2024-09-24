# Spring Cloud Bus

**官方文档：** https://cloud.spring.io/spring-cloud-bus/reference/html/

实际上它就相当于是一个消息总线，可用于向各个服务广播某些状态的更改（比如云端配置更改，可以结合Config组件实现动态更新配置，当然我们前面学习的Nacos其实已经包含这个功能了）或其他管理指令。

这里我们也是简单使用一下吧，Bus需要基于一个具体的消息队列实现，比如RabbitMQ或是Kafka，这里我们依然使用RabbitMQ。

我们将最开始的微服务拆分项目继续使用，比如现在我们希望借阅服务的某个接口调用时，能够给用户服务和图书服务发送一个通知，首先是依赖：

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-bus-amqp</artifactId>
</dependency>

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

接着我们只需要在配置文件中将RabbitMQ的相关信息配置：

```yaml
spring:
  rabbitmq:
    addresses: 192.168.0.6
    username: admin
    password: admin
    virtual-host: /test
management:
  endpoints:
    web:
      exposure:
        include: "*"    #暴露端点，一会用于提醒刷新
```

然后启动我们的三个服务器，可以看到在管理面板中：

![image-20220421232118952](https://s2.loli.net/2023/03/08/UfTVhAiOnMqoPX7.jpg)

新增了springCloudBug这样一个交换机，并且：

![image-20220421232146646](https://s2.loli.net/2023/03/08/2VdCOuPLAb9Qhfx.jpg)

自动生成了各自的消息队列，这样就可以监听并接收到消息了。

现在我们访问一个端口：

![image-20220421233200950](https://s2.loli.net/2023/03/08/H3szAX82xhpWw6j.jpg)

此端口是用于通知别人进行刷新，可以看到调用之后，消息队列中成功出现了一次消费：

![image-20220421233302328](https://s2.loli.net/2023/03/08/LoviBfecC1DbMOg.jpg)

现在结合之前使用的Config配置中心，来看看是不是可以做到通知之后所有的配置动态刷新了。

