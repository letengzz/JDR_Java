# 微服务容错

在高并发访问下，比如天猫双11，流量持续不断的涌入，服务之间的相互调用频率突然增加，引发系统负载过高，这时系统所依赖的服务的稳定性对系统的影响非常大，而且还有很多不确定因素 (比如网络卡顿、系统故障、硬件问题等都存在一定可能，会导致极端的情况发生) 引起[雪崩](Avalanche/README.md)、[服务超时](Overtime/README.md)、网络连接中断、服务宕机等。因此，需要寻找一个应对这种极端情况的解决方案。

**解决方案**：向调用方返回一个符合预期的、可处理的备选响应(FallBack)，而不是长时间的等待或者抛出调用方无法处理的异常，这样就保证了服务调用方的线程不会被长时间、不必要地占用，从而避免了故障在分布式系统中的蔓延，乃至雪崩。一般微服务容错组件提供了[限流](CurrentLimiting/README.md)、[隔离](isolation/README.md)、[降级](degradation/README.md)、[熔断](fusing/README.md)等手段，可以有效保护微服务系统。

**说明**：一定要区分开服务降级和服务熔断的区别，服务降级并不会直接返回错误，而是可以提供一个补救措施，正常响应给请求者。这样相当于服务依然可用，但是服务能力肯定是下降了的。

****

**Spring Cloud分布式微服务容错解决方案**：

- [Hystrix](../../SpringCloudNetflix/Hystrix/README.md)
- [CircuitBreaker](../../SpringCloud/CircuitBreaker/README.md)
- [Sentinel](../../SpringCloudAlibaba/Sentinel/README.md)
