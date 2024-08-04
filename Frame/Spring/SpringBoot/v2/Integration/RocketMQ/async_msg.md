# 发送异步消息

## 准备工作

引入starter依赖：

```xml
<dependencies>
       <dependency>
            <groupId>org.apache.rocketmq</groupId>
            <artifactId>rocketmq-spring-boot-starter</artifactId>
            <version>2.2.3</version>
        </dependency>
</dependencies>
```

## 创建生产者

配置文件：

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

## 创建消费者

