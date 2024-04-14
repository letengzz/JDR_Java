# Consul

Consul 是一套开源的分布式服务发现和配置管理系统，由 HashiCorp 公司用 Go 语言开发。部署起来非常容易，只需要极少的可执行程序和配置文件，具有绿色、轻量级的特点。

Consul是分布式的、高可用的、 可横向扩展的用于实现分布式系统的服务发现与配置。提供了微服务系统中的服务治理、配置中心、控制总线等功能。这些功能中的每一个都可以根据需要单独使用，也可以一起使用以构建全方位的服务网格，总之Consul提供了一种完整的服务网格解决方案。它具有很多优点。包括： 基于 raft 协议，比较简洁； 支持健康检查, 同时支持 HTTP 和 DNS 协议 支持跨数据中心的 WAN 集群 提供图形界面 跨平台，支持 Linux、Mac、Windows

**官方网站**：http://www.consul.io

**Spring Cloud Consul**：https://spring.io/projects/spring-cloud-consul

Consul 具有如下特性：

- **服务发现**：提供HTTP和DNS两种发现方式
- **健康监测**：支持多种方式：HTTP、TCP、Docker、Shell脚本定制化监控
- **KV存储**：Key-Value存储方式
- **多数据中心**：支持多数据中心
- **可视化Web界面**

![image-20240405181906222](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202404051821499.png)

