# Ribbon

Spring Cloud Ribbon是基于Netflix Ribbon实现的一套负载均衡的客户端工具。

简单的说，Ribbon是Netflix发布的开源项目，主要功能是提供客户端的软件负载均衡算法和服务调用。Ribbon客户端组件提供一系列完善的配置项如连接超时，重试等。简单的说，就是在配置文件中列出Load Balancer（简称LB）后面所有的机器，Ribbon会自动的帮助你基于某种规则（如简单轮询，随机连接等）去连接这些机器。我们很容易使用Ribbon实现自定义的负载均衡算法。

**注意**：Ribbon已经被废弃，解决方案推荐使用[Spring Cloud LoadBalancer](../../SPringCloud/LoadBalancer/README.md)作为替换方案

