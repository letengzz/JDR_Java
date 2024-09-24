# 注册中心

所有提供者将自己提供服务的名称及自己主机详情 (IP、端口、版本等) 写入到另一台 主机中的一个列表中，这台主机称为**服务注册中心**，而这个表称为**服务注册表**。 所有消费者需要调用微服务时，其会从注册中心首先将服务注册表下载到本地，然后根据消费者本地设置好的负载均衡策略选择一个服务提供者进行调用。这个过程称为**服务发现**。 可以充当 Spring Cloud 服务注册中心的服务器很多，如 Zookeeper、Eureka、Consul 等。 Spring Cloud Alibaba 中使用的注册中心为 Alibaba 的中间件 Nacos。

**一致性问题**：

作为注册中心，Spring Cloud Config、Apollo、Nacos Config、Zookeeper这些 Server 集群间是存在数据一致性问题的。

它们采用的模式是不同的：Zookeeper (CP)、Eureka (AP)、Consul (AP)、Nacos (默认 AP，也支持 CP)。

****

**Spring Cloud分布式微服务注册中心解决方案**：

- [Eureka](../../SpringCloudNetflix/Eureka/README.md)
- [Zookeeper](../../SpringCloud/Zookeeper/Registry/README.md)
- [Consul](../../SpringCloud/Consul/README.md)
- [Nacos](../../SpringCloudAlibaba/Nacos/README.md)