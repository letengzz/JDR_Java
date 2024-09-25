# 微服务链路跟踪

在微服务框架中，随着分布式系统规模的越来越大，各微服务间的调用关系也变得越来越复杂。一个由客户端发起的请求在后端系统中会经过多个不同的的服务节点调用来协同产生最后的请求结果，每一个前段请求都会形成一条复杂的分布式服务调用链路，链路中的任何一环出现高延时或错误都会引起整个请求最后的失败。

**分布式链路追踪** (`Distributed Tracing`) 将一次分布式请求还原成调用链路，进行日志记录，性能监控并将一次分布式请求的调用情况集中展示，比如各个服务节点上的耗时、请求具体到达哪台机器上、每个服务节点的请求状态等等。

在大规模分布式与微服务集群下，通过分布式服务跟踪系统可以实时观测系统的整体调用链路情况、快速发现并定位到问题、尽可能精确的判断故障对系统的影响范围与影响程度、尽可能精确的梳理出服务之间的依赖关系，并判断出服务之间的依赖关系是否合理、尽可能精确的分析整个系统调用链路的性能与瓶颈点。尽可能精确的分析系统的存储瓶颈与容量规划。

国内的分布式跟踪系统：Hydra(京东)、CAT(大众点评)、Watchman(新浪)、Micoscope(唯品会) 及 Eagleeye(淘宝，未开源)。

国际上使用较多的是 Zipkin(Twitter)及 Skywalking。

****

**Spring Cloud分布式微服务链路跟踪解决方案**：

- [Sleuth](../../SpringCloud/Sleuth/README.md)
- [Micrometer Tracing](../../SpringCloud/MicrometerTracing/README.md)
- [SkyWalking](SkyWalking/README.md)