# 服务配置中心

随着单体架构向微服务架构的演进，各个应用自己独立维护本地配置文件的方式开始显露出它的不足之处。

主要体现：

- **配置的动态更新**：在实际应用会有动态更新位置的需求，比如修改服务连接地址、限流配置等。在传统模式下，需要手动修改配置文件并且重启应用才能生效，这种方式效率太低，重启也会导致服务暂时不可用。
- **配置多节点维护**：在微服务架构中某些核心服务为了保证高性能会部署上百个节点，如果在每个节点中都维护一个配置文件，一旦配置文件中的某个属性需要修改，可想而知，工作量是巨大的。
- **不同部署环境下配置的管理**：前面提到通过profile机制来管理不同环境下的配置，这种方式对于日常维护来说也比较繁琐。

统一配置管理就是弥补上述不足的方法，简单说，最基本的方法是把各个应用系统中的某些配置放在一个第三方中间件上进行统一维护。然后，对于统一配置中心上的数据的变更需要推送到相应的服务节点实现动态更新。

相关产品很多，例如，Spring Cloud Config、Zookeeper、Apollo、Disconf(百度的，不再维护)等。 Spring Cloud Alibaba 官方推荐使用 Nacos 作为微服务的配置中心。

**一致性问题**：

配置中心中的配置数据一般都是持久化在第三方服务器的，例如 DBMS、Git 远程库等。 由于这些配置中心 Server 中根本就不存放数据，所以、集群中就不存在数据一致性问题。但像 Zookeeper，其作为配置中心，配置数据是存放在自己本地的。所以该集群中的节点是存在数据一致性问题的。Zookeeper 集群对于数据一致性采用的是 CP 模式。 

****

**Spring Cloud分布式微服务服务配置中心解决方案**：

- [Apollo](../../Apollo/README.md)
- [Consul](../../SpringCloud/Consul/Config/README.md)
- [Spring Cloud Config](../../SpringCloud/Config/README.md)
- [Nacos Config](../../SpringCloudAlibaba/Nacos/Config/README.md)
- [Zookeeper](../../Zookeeper/Config/README.md)

