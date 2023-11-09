##  Spring Cloud简介

Spring Cloud是Spring提供的微服务框架。它利用Spring Boot的开发特性简化了微服务开发的复杂性，例如：**服务发现注册**、**配置中心**、**消息总线**、**负载均衡**、**断路器**、**数据监控**等，这些工作都可以借助Spring Boot的开发风格做到一键启动和部署。

Spring Cloud的目标是通过一系列组件，帮助开发者迅速构件一个分布式系统，Spring Cloud 是通过包装其它公司产品来实现的，比如Spring Cloud整合了开源的Netflix很多产品。Spring Cloud提供了微服务治理的诸多组件，例如：**服务注册和发现**、**配置中心**、**熔断器**、**智能路由**、**微代理**、**控制总线**、**全局锁**、**分布式会话**等。Spring Cloud组件非常多，涉及微服务开发的诸多场景。

**Spring Cloud的架构图**：

![image.png](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303271139276.png)

![image_rR3xK4zw_I](https://cdn.jsdelivr.net/gh/letengzz/tc2@main/img/202310051300183.png)

Spring Cloud实现微服务的治理功能产品很多，下面简单介绍下Spring Cloud各个产品的作用，以及采用的原则：

![cloud产品列表.png](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303271139585.png)

例：基于Netflix（奈飞）的开源分布式解决方案提供的组件：

- Eureka：实现服务治理(服务注册与发现)，可以对所有的微服务进行集中管理，包括他们的运行状态、信息等。
- Ribbon：为服务之间相互调用提供负载均衡算法（现在被SpringCloudLoadBalancer取代）
- Hystrix：断路器，保护系统，控制故障范围。暂时可以跟家里电闸的保险丝类比，当触电危险发生时能够防止进一步的发展。
- Zuul：api网关，路由，负载均衡等多种作用，就像我们的路由器，可能有很多个设备都连接了路由器，但是数据包要转发给谁则是由路由器在进行（已经被SpringCloudGateway取代）
- Config：配置管理，可以实现配置文件集中管理

## Spring Cloud 和Spring Boot版本对应关系

| Spring Cloud                                                 | Spring Boot                           |
| ------------------------------------------------------------ | ------------------------------------- |
| [2022.0.x](https://github.com/spring-cloud/spring-cloud-release/wiki/Spring-Cloud-2022.0-Release-Notes) aka Kilburn | 3.0.x                                 |
| [2021.0.x](https://github.com/spring-cloud/spring-cloud-release/wiki/Spring-Cloud-2021.0-Release-Notes) aka Jubilee | 2.6.x, 2.7.x (Starting with 2021.0.3) |
| [2020.0.x](https://github.com/spring-cloud/spring-cloud-release/wiki/Spring-Cloud-2020.0-Release-Notes) aka Ilford | 2.4.x, 2.5.x (Starting with 2020.0.3) |
| [Hoxton](https://github.com/spring-cloud/spring-cloud-release/wiki/Spring-Cloud-Hoxton-Release-Notes) | 2.2.x, 2.3.x (Starting with SR5)      |
| [Greenwich](https://github.com/spring-projects/spring-cloud/wiki/Spring-Cloud-Greenwich-Release-Notes) | 2.1.x                                 |
| [Finchley](https://github.com/spring-projects/spring-cloud/wiki/Spring-Cloud-Finchley-Release-Notes) | 2.0.x                                 |
| [Edgware](https://github.com/spring-projects/spring-cloud/wiki/Spring-Cloud-Edgware-Release-Notes) | 1.5.x                                 |
| [Dalston](https://github.com/spring-projects/spring-cloud/wiki/Spring-Cloud-Dalston-Release-Notes) | 1.5.x                                 |

官网对版本进行解释：

- Spring Cloud Dalston, Edgware, Finchley, and Greenwich have all reached end of life status and are no longer supported.

  Spring Cloud Dalston、Edgware、Finchley 和 Greenwich 都已达到生命周期终止状态，不再受支持