# 微服务负载均衡

负载均衡是指将负载分摊到多个执行单元上，常见的负载均衡有两种方式。一种独立进程单元，通过负载均衡策略，将请求转发到不同的执行单元上，例如Nginx。另一种是将负载均衡逻辑以代码的形式封装到服务消费者的客户端上，服务消费者客户端维护了一份服务提供者的信息列表，有了信息表，通过负载均衡策略将请求分摊给多个服务提供者，从而达到负载均衡的目的。

****

**Spring Cloud分布式微服务负载均衡解决方案**：

- [Spring Cloud Ribbon](../../SpringCloudNetflix/Ribbon/README.md)
- [Spring Cloud LoadBalancer](../../SpringCloud/LoadBalancer/README.md)