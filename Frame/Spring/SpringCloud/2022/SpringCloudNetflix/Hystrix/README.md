# Hystrix 服务熔断

为了解决分布式系统的雪崩问题，Spring Cloud提供了Hystrix熔断器组件，保证在不断地有大量的请求到达，需要各个服务进行处理，在一个服务故障时，不会导致整条链路上的服务全线崩溃。

Hystrix是一个用于处理分布式系统的延迟和容错的开源库，在分布式系统里，许多依赖不可避免的会调用失败，比如超时、异常等，Hystrix能够保证在一个依赖出问题的情况下，不会导致整体服务失败，避免级联故障，以提高分布式系统的弹性。

**注意**：Netflix的Hystrix微服务容错库已经停止更新，官方推荐使用[Resilience4j](../../SpringCloud/CircuitBreaker/Resilience4j/README.md)代替Hystrix，或者使用Spring Cloud Alibaba的[Sentinel](../../SpringCloudAlibaba/Sentinel/README.md)组件。