# Spring Cloud Stream 概述

Spring Cloud Stream 是一个用来为微服务应用构建**消息驱动**能力的框架。通过使用 Spring Cloud Stream，可以有效简化开发人员对消息中间件的使用复杂度，降低代码与消息中间件间的耦合度，屏蔽消息中间件之间的差异性，让开发人员可以有更多的精力关注于核心业务逻辑的处理。

当遇到不同的系统在用不同的消息队列，比如系统A用的Kafka、系统B用的RabbitMQ，SpringCloud Stream能够屏蔽底层实现，使用统一的消息队列操作方式就能操作多种不同类型的消息队列。

它屏蔽了MQ底层操作，可以使用统一的Input和Output形式，以Binder为中间件，这样就算切换了不同的消息队列，也无需修改代码，而具体某种消息队列的底层实现是交给Stream在做的。

![SCSt with binder](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202404292207200.png)

**官方文档：** https://docs.spring.io/spring-cloud-stream/docs/3.2.2/reference/html/

## 编程模型

![SCSt overview](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202404292219762.png)

### Destination Binders 

目的地绑定器：负责提供与外部消息系统集成的组件(应用程序和MQ之间进行消息通信的规范、接口，用来屏蔽各个MQ的差异性，具体的MQ有不同的实现)。 

### Bindings

固定器：介于外部消息系统与应用程序间的桥梁，这个应用程序提供了生产者和消费者的消息（由 Destination Binders 创建）

#### Input Bindings

输入管道，消费者通过 Input Bindings 连接 Binder，而 Binder 与 MQ 连接，即消费者通过 Input Bindings 从 MQ 读取数据。 

#### Output Bindings

输出管道，生产者通过 Output Bindings 连接 Binder，而 Binder 与 MQ 连接，即生产者通过 Output Bindings 向 MQ 写入数据。 

### Message 

消息：生产者和消费者使用的规范数据结构，用于与 Binders 通信（从而通过外部消息系统 与其他应用程序通信）。