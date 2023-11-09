# Spring Cloud Tencent

Spring Cloud Tencent 是腾讯开源的一站式微服务解决方案。

![rectangle-white](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/202306301109129.png)

Spring Cloud Tencent 实现了Spring Cloud 标准微服务 SPI，开发者可以基于 Spring Cloud Tencent 快速开发 Spring Cloud 云原生分布式应用。

Spring Cloud Tencent 的核心依托腾讯开源的一站式服务发现与治理平台 [Polaris](https://github.com/polarismesh/polaris)，实现各种分布式微服务场景。

- [Polaris Github home page](https://github.com/polarismesh/polaris)
- [Polaris official website](https://polarismesh.cn/)

Spring Cloud Tencent提供的能力包括但不限于：

![image](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202307080020981.png)

- **服务注册和发现**：服务注册与发现是 Spring Cloud Tencent 最为核心的功能之一，通过实现 Spring Cloud 的服务注册与发现的标准接口，提供微服务应用快速接入北极星服务注册中心的能力。开发者通过简单的引入 Spring Cloud Tencent 服务注册与发现的依赖，即可使用北极星的服务注册与发现功能。

  接入服务注册与发现之后，还能按需使用北极星提供的强大服务治理能力，例如场景化的服务路由能力、服务熔断能力等。方便开发者针对微服务的实际生产场景作出个性化的服务治理配置。

  北极星的服务模型包括命名空间、服务和服务实例。

- **动态配置管理**：

  北极星配置中心核心配置三元组模型为：

  - **Namespace**：用于逻辑隔离集群的能力，例如常用于隔离环境。
  - **FileGroup**：配置文件组，一组配置文件的集合。在 Spring Cloud Tencent 里，我们推荐的最佳实践是一个应用为一个 FileGroup。对于框架类的配置，以框架名作为一个 FileGroup， 例如 dubbo。
  - **File**：配置文件，例如 properties 、yml 格式的配置文件。配置文件为最小管理单元，而不是配置文件里的配置项。

- **服务治理**

  - 服务限流
  - 服务熔断
  - 服务路由
  - ...

- **标签透传**