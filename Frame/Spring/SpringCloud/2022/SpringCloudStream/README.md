# Spring Cloud Stream

- [Spring Cloud Stream 概述](Introduce/README.md)
- 











这里我们创建一个新的项目来测试一下：

![image-20220421215534386](https://s2.loli.net/2023/03/08/pJefuIUXzNHhsxP.jpg)

依赖如下：

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-dependencies</artifactId>
    <version>2021.0.1</version>
    <type>pom</type>
    <scope>import</scope>
</dependency>
```

```xml
<dependencies>
    <!--  RabbitMQ的Stream实现  -->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-stream-rabbit</artifactId>
    </dependency>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>
```

首先我们来编写一下生产者，首先是配置文件：

```yaml
server:
  port: 8001
spring:
  cloud:
    stream:
      binders:   #此处配置要绑定的rabbitmq的服务信息
        local-server: #绑定名称，随便起一个就行
          type: rabbit #消息组件类型，这里使用的是RabbitMQ，就填写rabbit
          environment:  #服务器相关信息，按照下面的方式填写就行，爆红别管
            spring:
              rabbitmq:
                host: 192.168.0.6
                port: 5672
                username: admin
                password: admin
                virtual-host: /test
       bindings:
        test-out-0:
          destination: test.exchange
```

接着我们来编写一个Controller，一会访问一次这个接口，就向消息队列发送一个数据：

```java
@RestController
public class PublishController {

    @Resource
    StreamBridge bridge;  //通过bridge来发送消息

    @RequestMapping("/publish")
    public String publish(){
        //第一个参数其实就是RabbitMQ的交换机名称（数据会发送给这个交换机，到达哪个消息队列，不由我们决定）
      	//这个交换机的命名稍微有一些规则:
      	//输入:    <名称> + -in- + <index>
      	//输出:    <名称> + -out- + <index>
      	//这里我们使用输出的方式，来将数据发送到消息队列，注意这里的名称会和之后的消费者Bean名称进行对应
        bridge.send("test-out-0", "HelloWorld!");
        return "消息发送成功！"+new Date();
    }
}
```

现在我们来将生产者启动一下，访问一下接口：

![image-20220421220955906](https://s2.loli.net/2023/03/08/pvc8udVL9EwMW56.jpg)

可以看到消息成功发送，我们来看看RabbitMQ这边的情况：

![image-20220421221027145](https://s2.loli.net/2023/03/08/1fBHoQe6gc7XizO.jpg)

新增了一个`test-in-0`交换机，并且此交换机是topic类型的：

![image-20220421221107547](https://s2.loli.net/2023/03/08/mN4EfOehP8Ta2JC.jpg)

但是目前没有任何队列绑定到此交换机上，因此我们刚刚发送的消息实际上是没有给到任何队列的。

接着我们来编写一下消费者，消费者的编写方式比较特别，只需要定义一个Consumer就可以了，其他配置保持一致：

```java
@Component
public class ConsumerComponent {

    @Bean("test")   //注意这里需要填写我们前面交换机名称中"名称"，这样生产者发送的数据才会正确到达
    public Consumer<String> consumer(){
        return System.out::println;
    }
}
```

配置中需要修改一下目标交换机：

```yaml
server:
  port: 8002
spring:
  cloud:
    stream:
    	...
      bindings:
      	#因为消费者是输入，默认名称为 方法名-in-index，这里我们将其指定为我们刚刚定义的交换机
        test-in-0:
          destination: test.exchange
```

接着我们直接启动就可以了，可以看到启动之后，自动为我们创建了一个新的队列：

![image-20220421221733723](https://s2.loli.net/2023/03/08/kUelcRgb7MrGdB6.jpg)

而这个队列实际上就是我们消费者等待数据到达的队列：

![image-20220421221807577](https://s2.loli.net/2023/03/08/lzDjiI9SLH1rVY3.jpg)

可以看到当前队列直接绑定到了我们刚刚创建的交换机上，并且`routingKey`是直接写的`#`，也就是说一会消息会直接过来。

现在我们再来访问一些消息发送接口：

![image-20220421221938730](https://s2.loli.net/2023/03/08/cSPRdoY43gzVNXk.jpg)

![image-20220421221952663](https://s2.loli.net/2023/03/08/8TEv1KQGSNA9luY.jpg)

可以看到消费者成功地进行消费了：

![image-20220421222011924](https://s2.loli.net/2023/03/08/lICtpeK2oAGZynD.jpg)

这样，我们就通过使用SpringCloud Stream来屏蔽掉底层RabbitMQ来直接进行消息的操作了。
