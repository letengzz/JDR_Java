# SkyWalking  调用链跟踪

生产环境下，Spring Cloud Alibaba 经常会使用 SkyWalking 作为调用链跟踪系统。

## 概述

SkyWalking 是由国内开源爱好者吴晟开源并提交到 Apache 孵化器的产品，现在已是Apache 的顶级项目。其是一个开源的 APM (Application Performance Management，应用性能管理系统) 和 OAP 平台 (Observability Analysis Platform，可观测性分析平台)。

其是通过在被监测应用中插入探针，以无侵入方式自动收集所需指标，并自动推送到OAP 系统平台。OAP 会将收集到的数据存储到指定的存储介质 Storage。UI 系统通过调用 OAP提供的接口，可以实现对相应数据的查询。

官网地址：https://skywalking.apache.org/ 

### 系统架构

SkyWalking 系统整体由四部分构成：

- Agent：探针，无侵入收集，是被插入到被监测应用中的一个进程。其会将收集到的监控指标自动推送给 OAP 系统。

- OAP：Observability Analysis Platform，可观测性分析平台，其包含一个收集器 Collector，能够将来自于 Agent 的数据收集并存储到相应的存储介质 Storage。

- Storage：数据中心。用于存储由 OAP 系统收集到的链路数据，支持 H2、MySQL、ElasticSearch 等。默认使用的是 H2（测试使用），推荐使用 ElasticSearch（生产使用）。

- UI：一个独立运行的可视化 Web 平台，其通过调用 OAP 系统提供的接口，可以对存储在 Storage 中的数据进行多维度查询。

### trace 与 span

- trace (轨迹) ：跟踪单元是从客户端所发起的请求抵达被跟踪系统的边界开始，到被跟踪系统向客户返回响应为止的过程，这个过程称为一个 trace。

- span(跨度)：每个 trace 中会调用若干个服务，为了记录调用了哪些服务，以及每次调用所消耗的时间等信息，在每次调用服务时，埋入一个调用记录，这样两个调用记录之间的区域称为一个 span。一个 trace 由若干个有序的 span 组成。

- segment(片断)：一个 trace 的一个片断，可以由多个 span 组成。

为了唯一的标识 trace、span 与 segment，跟踪系统一般会为每个 trace、span 与 segment都指定了一个唯一标识，即 traceId、spanId 与 segmentId。