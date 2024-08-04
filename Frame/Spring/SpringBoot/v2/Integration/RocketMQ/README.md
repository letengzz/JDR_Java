# SpringBoot 整合 RocketMQ

RocketMQ是阿里巴巴2016年MQ中间件，使用Java语言开发，RocketMQ 是一款开源的分布式消息系统，基于高可用分布式集群技术，提供低延时的、高可靠、可伸缩的消息发布与订阅服务。是实现业务削峰，分布式事务的优秀框架。RocketMQ支持集群模式、消费者负载均衡、水平扩展和广播模式。采用零拷贝原理，顺序写盘、支持亿级消息堆积能力。同时提供了丰富的消息机制，比如顺序消息、事务消息等。同时，广泛应用于多个领域，包括异步通信解耦、企业解决方案、金融支付、电信、电子商务、快递物流、广告营销、社交、即时通信、移动应用、手游、视频、物联网、车联网等。

**官方网站**：http://rocketmq.apache.org/

- [RocketMQ](../../../../../../MQ/RocketMQ/README.md)

## 准备工作

生产者、消费者都需要引入starter依赖：

```xml
<dependencies>
       <dependency>
            <groupId>org.apache.rocketmq</groupId>
            <artifactId>rocketmq-spring-boot-starter</artifactId>
            <version>2.2.3</version>
        </dependency>
</dependencies>
```

## 发送同步消息

发送同步消息，发送过后会有一个返回值，也就是MQ服务器接收到消息后返回的一个确认，这种方式非常安全，但是性能上并没有这么高，而且在MQ集群中，也是要等到所有的从机都复制了消息以后才会返回，所以针对重要的消息可以选择这种方式。

消息由消费者发送到broker后，会得到一个确认，是具有可靠性的。这种可靠性同步地发送方式使用的比较广泛，比如：**重要的消息通知**，**短信通知**等。

![image-20230822155329307](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202308221553977.png)

### 创建生产者

**配置文件 `application.yml`**：

```yaml
spring:
  application:
    name: rocketmq-producer
rocketmq:
  name-server: 127.0.0.1:9876  # rocketMq的nameServer地址
  producer:
    group: producer-group  # 生产者组别
    send-message-timeout: 3000  # 消息发送的超时时间
    retry-times-when-send-failed: 2  # 异步消息发送失败重试次数
    max-message-size: 4194304       # 消息的最大长度
```

**编写测试类**：

使用RocketMQTemplate来操作MQ

**说明**：三种发送消息的方法，底层都是调用syncSend，发送的是同步消息

- `rocketMQTemplate.syncSend()`
- `rocketMQTemplate.send()`
- `rocketMQTemplate.convertAndSend()`

```java
@SpringBootTest
public class Producer {

    /**
     * 注入rocketMQTemplate，使用它来操作mq
     */
    @Resource
    private RocketMQTemplate rocketMQTemplate;

    /**
     * 发送同步消息
     * @throws Exception
     */
    @Test
    public void testSyncSend(){
        // 往simpleTopicTest的主题里面发送一个简单的字符串消息
        SendResult sendResult = rocketMQTemplate.syncSend("SyncTopicTest", "发送一个同步消息");
        // 拿到消息的发送状态
        System.out.println(sendResult.getSendStatus());
        // 拿到消息的id
        System.out.println(sendResult.getMsgId());
    }
}
```

**运行结果**：

- 运行后查看控制台：

  ![image-20230822160732218](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202308221607125.png)

- 查看RocketMQ的控制台：

  ![image-20230822160815470](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202308221608905.png)

- 查看消息的细节：

  ![image-20230822160836892](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202308221608852.png)

## 创建消费者

**配置文件 `application.yml`**：

```yaml

```

**编写监听类**：

**启动 rocketmq-consumer，并查看运行结果**：

查看控制台，发现已经监听到消息了

![image-20230822161048478](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202308221610852.png)

- [发送同步消息](sync_msg.md)
- [发送异步消息](async_msg.md)
- [发送单向消息]()
- [发送延迟消息]()
- [发送顺序消息]()
- [发送批量消息(集合消息)]()
- [发送事务消息]()
- [发送对象消息]()
- [消息过滤(tag&key)]()



## 发送单向消息

### 创建生产者

### 创建消费者

## 发送延迟消息

### 创建生产者

### 创建消费者

## 发送顺序消息

### 创建生产者

### 创建消费者

## 发送批量消息(集合消息)

### 创建生产者

### 创建消费者

## 发送事务消息

### 创建生产者

### 创建消费者

## 发送对象消息

### 创建生产者

### 创建消费者

## 消息过滤(tag&key)

### 创建生产者

### 创建消费者
