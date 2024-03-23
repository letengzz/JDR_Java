# 发送批量消息

Rocketmq可以一次性发送一组消息，那么这一组消息会被当做一个消息消费

## 生产者

```java
public class Producer {
    public static void main(String[] args) throws MQClientException, MQBrokerException, RemotingException, InterruptedException {
        // 创建默认的生产者
        DefaultMQProducer producer = new DefaultMQProducer("batch-producer-group");
        //设置nameServer地址
        producer.setNamesrvAddr(MqConstant.NAME_SRV_ADDR_LINUX);
        //启动生产者
        producer.start();
        //创建消息
        //第一个参数：主题的名字
        //第二个参数：消息内容
        List<Message> msgs = Arrays.asList(
                new Message("BatchTopicTest", "我是一组消息的A消息".getBytes()),
                new Message("BatchTopicTest", "我是一组消息的B消息".getBytes()),
                new Message("BatchTopicTest", "我是一组消息的C消息".getBytes())
        );

        SendResult send = producer.send(msgs);
        System.out.println("send = " + send);
        //关闭实例
        producer.shutdown();
    }
}
```

启动生产者进行测试：

![image-20240319215958367](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202403192200604.png)

## 消费者

消息会被投递(或者消费者主动拉取)给每一个组，在每一个组内是进行负载均衡还是广播模式，是看这个消费者组的配置

- 负载均衡模式：`MessageModel.CLUSTERING`
- 广播模式：`MessageModel.BROADCASTING`

![image-20231219205310716](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202312192053776.png)

```java
public class Consumer {
    public static void main(String[] args) throws MQClientException, IOException {
        //创建默认消费者组
        DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("batch-consumer-group");
        //设置nameServer地址
        consumer.setNamesrvAddr(MqConstant.NAME_SRV_ADDR_LINUX);
        //订阅一个主题来消费  *表示没有过滤参数 表示这个主题的任何消息
        consumer.subscribe("BatchTopicTest","*");
        //注册一个消费监听 MessageListenerConcurrently 是多线程消费，默认20个线程
        // 可以参看consumer.setConsumeThreadMax()
//        consumer.setMessageModel(MessageModel.CLUSTERING);
        consumer.registerMessageListener(new MessageListenerConcurrently() {
            @Override
            public ConsumeConcurrentlyStatus consumeMessage(List<MessageExt> list, ConsumeConcurrentlyContext consumeConcurrentlyContext) {
                // 这里执行消费的代码 默认是多线程消费
                System.out.println(Thread.currentThread().getName() + "----" + list);
                System.out.println("消息内容:"+new String(list.get(0).getBody()));
                System.out.println("消费上下文:"+consumeConcurrentlyContext);
                // 返回消费的状态 如果是CONSUME_SUCCESS 则成功，若为RECONSUME_LATER则该条消息会被重回队列，重新被投递
                // 重试的时间为messageDelayLevel = "1s 5s 10s 30s 1m 2m 3m 4m 5m 6m 7m 8m 9m 10m 20m 30m 1h 2h
                // 也就是第一次1s 第二次5s 第三次10s  ....  如果重试了16次 那么这个消息就会被终止发送给消费者
                // 返回状态：
                // CONSUME_SUCCESS：成功 消息会从mq出队
                // RECONSUME_LATER：失败(报错、null) 消息会重新回到队列 过一会重新投递出来 给当前消费者或者其他消费者消费的
                return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
                // return ConsumeConcurrentlyStatus.RECONSUME_LATER;
            }
        });
        // 这个start一定要写在registerMessageListener下面
        consumer.start();
        // 由于是异步回调方式，挂起当前的jvm
        System.in.read();

    }
}
```

启动消费者进行测试：

![image-20240319220532602](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202403192205924.png)
