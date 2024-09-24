# SpringBoot 整合 Kafka



官方：https://kafka.apache.org/documentation/ 

 消息队列-场景 

 1异步 

场景1

注册

发送邮件

发送短信

注册信息入库

50MS

50MS

100MS

异步改造:

发送短信

50MS

注册

注册信息入库

100MS

发送邮件

50MS

消息改造:

发送短信

50MS

文

注册

监听消息

注册信息入库

发送成功消息

100MS

5MS

发送邮件

50MS

![image.png](https://cdn.nlark.com/yuque/0/2023/png/1613913/1683508559265-41439f88-8a77-421b-8072-64ac18836e14.png?x-oss-process=image%2Fwatermark%2Ctype_d3F5LW1pY3JvaGVp%2Csize_24%2Ctext_5bCa56GF6LC3IGF0Z3VpZ3UuY29t%2Ccolor_FFFFFF%2Cshadow_50%2Ct_80%2Cg_se%2Cx_10%2Cy_10)





 2解耦 

场景2

调用库存接口

库存系统

订单系统

消息队列

发送消息

监听消息

库存系统

订单系统

![image.png](https://cdn.nlark.com/yuque/0/2023/png/1613913/1683509053856-89249929-3fd0-46c9-bef0-05fdf1d4a57a.png?x-oss-process=image%2Fwatermark%2Ctype_d3F5LW1pY3JvaGVp%2Csize_22%2Ctext_5bCa56GF6LC3IGF0Z3VpZ3UuY29t%2Ccolor_FFFFFF%2Cshadow_50%2Ct_80%2Cg_se%2Cx_10%2Cy_10)







 3削峰 

场景3

按照能力处理

写入

消息队列

用户请求

业务处理

![image.png](https://cdn.nlark.com/yuque/0/2023/png/1613913/1683509270156-4f382e30-4c48-48f4-be07-502178966813.png?x-oss-process=image%2Fwatermark%2Ctype_d3F5LW1pY3JvaGVp%2Csize_26%2Ctext_5bCa56GF6LC3IGF0Z3VpZ3UuY29t%2Ccolor_FFFFFF%2Cshadow_50%2Ct_80%2Cg_se%2Cx_10%2Cy_10)





 4缓冲 

场景4

APP

主机

订阅消费

写入

日志采集客户端

日志分析处理

消息队列

容器

LOT

![image.png](https://cdn.nlark.com/yuque/0/2023/png/1613913/1683509501803-432efaf3-227a-488c-9751-67bdb8cbeb5e.png?x-oss-process=image%2Fwatermark%2Ctype_d3F5LW1pY3JvaGVp%2Csize_28%2Ctext_5bCa56GF6LC3IGF0Z3VpZ3UuY29t%2Ccolor_FFFFFF%2Cshadow_50%2Ct_80%2Cg_se%2Cx_10%2Cy_10)







 消息队列-Kafka 

 1. 消息模式 

消息点对点模式

MESSAGE QUEUE

1.发送消息

2.接受消息

PRODUCER

CONSUMER

3

4

1

2

4.删除已处理消息

3.确认收到

消息发布订阅模式

CONSUMER-1

订阅主题

MESSAGE QUEUE

TOPIC(主题)

5

4

3

1

2

1.发布消息

NEWS

订阅主题

CONSUMER-2

服务端不删除

PRODUCER

1

TOPIC(主题)

B

P

C

A

E

TECHNOLOGY

CONSUMER-3

订阅主题

A

![image.png](https://cdn.nlark.com/yuque/0/2023/png/1613913/1682662791504-6cf21127-d9da-4602-a076-ae38c298f0ac.png?x-oss-process=image%2Fwatermark%2Ctype_d3F5LW1pY3JvaGVp%2Csize_34%2Ctext_5bCa56GF6LC3IGF0Z3VpZ3UuY29t%2Ccolor_FFFFFF%2Cshadow_50%2Ct_80%2Cg_se%2Cx_10%2Cy_10)





 2. Kafka工作原理 

KAFKA原理

同一个消费者组里面的消费者是队列竞争模式

分区:海量数据分散存储

不同消费者组里面的消费者是发布/订阅模式

副本:每个数据区都有备份

3分区2副本;故障切换

CONSUMER GROUP

KAFKA CLUSTER

FOLLOWER

FOLLOWER

LEADER

CONSUMER1

BROKER-O

100T-NEWS数据

CONSUMER2

TOPIC-NEWS

TOPIC-NEWS

TOPIC-NEWS

CONSUMER3

PUSH

PARTITION-1

PARTITION-2

PARTITION-0

PULL

PRODUCER

CONSUMER GROUP

BROKER-1

PUSH

TOPIC-NEWS

PULL

TOPIC-NEWS

TOPIC-NEWS

CONSUMER4

PRODUCER

PARTITION-1

PARTITION-2

PARTITION-0

CONSUMER5

获取LEADER分区,给LEADER发送消息

BROKER-2

CONSUMER6

PULL

PUSH

PRODUCER

TOPIC-NEWS

TOPIC-NEWS

TOPIC-NEWS

PARTITION-1

PARTITION-0

PARTITION-2

CONSUMER7

一个消费者可以消费多个分区数据

ZOOKEEPER

记录KAFKA集群数据

![image.png](https://cdn.nlark.com/yuque/0/2023/png/1613913/1683170677428-6ffa28b6-d522-435f-9e50-20fe3ddfd024.png?x-oss-process=image%2Fwatermark%2Ctype_d3F5LW1pY3JvaGVp%2Csize_36%2Ctext_5bCa56GF6LC3IGF0Z3VpZ3UuY29t%2Ccolor_FFFFFF%2Cshadow_50%2Ct_80%2Cg_se%2Cx_10%2Cy_10%2Fresize%2Cw_937%2Climit_0)







 3. SpringBoot整合 

参照：https://docs.spring.io/spring-kafka/docs/current/reference/html/#preface





配置





修改C:\Windows\System32\drivers\etc\hosts文件，配置8.130.32.70 kafka

 4. 消息发送 







Java

复制代码

1

2

3

4

5

6

7

8

9

10

11

12

13

14

15

16

17

import org.springframework.kafka.core.KafkaTemplate;

import org.springframework.stereotype.Component;

@Component

public class MyBean {

​    private final KafkaTemplate<String, String> kafkaTemplate;

​    public MyBean(KafkaTemplate<String, String> kafkaTemplate) {

​        this.kafkaTemplate = kafkaTemplate;

​    }

​    public void someMethod() {

​        this.kafkaTemplate.send("someTopic", "Hello");

​    }

}

 5. 消息监听 





Java

复制代码

1

2

3

4

5

6

7

8

9

10

11

12

13

14

15

16

17

@Component

public class OrderMsgListener {

​    @KafkaListener(topics = "order",groupId = "order-service")

​    public void listen(ConsumerRecord record){

​        System.out.println("收到消息："+record); //可以监听到发给kafka的新消息，以前的拿不到

​    }

​    @KafkaListener(groupId = "order-service-2",topicPartitions = {

​            @TopicPartition(topic = "order",partitionOffsets = {

​                    @PartitionOffset(partition = "0",initialOffset = "0")

​            })

​    })

​    public void listenAll(ConsumerRecord record){

​        System.out.println("收到partion-0消息："+record);

​    }

}

 6. 参数配置 

消费者





Properties

复制代码

1

2

3

spring.kafka.consumer.value-deserializer=org.springframework.kafka.support.serializer.JsonDeserializer

spring.kafka.consumer.properties[spring.json.value.default.type]=com.example.Invoice

spring.kafka.consumer.properties[spring.json.trusted.packages]=com.example.main,com.example.another



生产者





Properties

复制代码

1

2

spring.kafka.producer.value-serializer=org.springframework.kafka.support.serializer.JsonSerializer

spring.kafka.producer.properties[spring.json.add.type.headers]=false

 7. 自动配置原理 

kafka 自动配置在KafkaAutoConfiguration

1容器中放了 KafkaTemplate 可以进行消息收发

2容器中放了KafkaAdmin 可以进行 Kafka 的管理，比如创建 topic 等

3kafka 的配置在KafkaProperties中

4@EnableKafka可以开启基于注解的模式

1 人点赞

- ![it](https://cdn.nlark.com/yuque/0/2023/png/38373744/1688975784520-avatar/11f3fe31-a3ef-4e4b-8653-893da4c7151b.png?x-oss-process=image%2Fresize%2Cm_fill%2Cw_64%2Ch_64%2Fformat%2Cpng)

1