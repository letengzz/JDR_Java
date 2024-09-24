# Sentinel 流量防卫兵

随着微服务的流行，服务和服务之间的稳定性变得越来越重要。Sentinel 是 SpringCloud Alibaba 微服务容错组件。面向分布式、 多语言异构化服务架构的流量治理组件，主要以流量为切入点，从流量路由、流量控制、流 量整形、熔断降级、系统自适应过载保护、热点流量防护等多个维度来帮助开发者保障微服务的稳定性。 Sentinel 是分布式系统的防御系统。 

- 官网: https://sentinelguard.io/zh-cn 
- Github：https://github.com/alibaba/Sentinel

![Sentinel Logo](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202404161256292.png)

- [Sentinel 概述](Introduce/README.md)
- [Sentinel 安装与部署](Install/README.md)
- [Sentinel 服务连接到控制台](Connect/README.md)
- [Sentinel 流量控制](FlowControl/README.md)
- [Sentinel 服务熔断和降级](FuseDown/README.md)
- [Sentinel 异常处理](Exception/README.md)
- [Sentinel 热点规则限流](Hot/README.md)
- [Sentinel 授权规则](AuthorizationRule/README.md)
- [`@SentinelResource` 注解](SentinelResource/README.md)
- [规则持久化](Persistence/README.md)
- [改造 Sentinel 控制台](RetrofitConsole/README.md)
- [集群流控](ClusterFlowControl/README.md)

**整合操作**：

- [Sentinel 整合 OpenFeign](OpenFeign/README.md)
- [Sentinel 整合 RestTemplate](RestTemplate/README.md)
- [Sentinel 整合 Gateway](Gateway/README.md)

