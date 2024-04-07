# Eureka 注册中心

Eureka是**Netflix**在线影片公司开源的一个服务注册和发现组件，和其他的Netflix公司的服务组件（例如负载均衡**Ribbon**，熔断器**Hystrix**，网关**Zuul**等）一起，被Spring Cloud社区整合为Spring Cloud Netflix模块。

Eureka是一个用于服务注册和发现的组件，最开始主要应用与亚马逊公司的云计算服务平台AWS，**Eureka分为Eureka Server(Eureka服务注册中心)和Eureka Client(Eureka客户端)**。

官方文档：https://docs.spring.io/spring-cloud-netflix/docs/current/reference/html/

**注意**：Eureka2.x已经停更，解决方案推荐使用[Consul](../../Consul/README.md)、[Nacos](../../SpringCloudAlibaba/Nacos/README.md)作为替换方案
