# 服务配置中心

集群中每一台主机的配置文件都是相同的，对配置文件的更新维护就成为了一个棘手的问题。此时就出现了配置中心，将集群中每个节点的配置文件交由配置中心统一管理。相关 产品很多，例如，Spring Cloud Config、Zookeeper、Apollo、Disconf(百度的，不再维护)等。 Spring Cloud Alibaba 官方推荐使用 Nacos 作为微服务的配置中心。

**一致性问题**：

配置中心中的配置数据一般都是持久化在第三方服务器的，例如 DBMS、Git 远程库等。 由于这些配置中心 Server 中根本就不存放数据，所以、集群中就不存在数据一致性问 题。但像 Zookeeper，其作为配置中心，配置数据是存放在自己本地的。所以该集群中的节点是存在数据一致性问题的。Zookeeper 集群对于数据一致性采用的是 CP 模式。 

****

- [Spring Cloud Config](../../SpringCloud/Config/README.md)
- [Apollo](../../Apollo/README.md)
- [Nacos Config](../../SpringCloudAlibaba/Nacos/README.md)
- [Zookeeper](../../Zookeeper/Config/README.md)

