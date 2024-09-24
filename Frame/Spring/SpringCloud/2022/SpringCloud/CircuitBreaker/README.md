# Circuit Breaker

CircuitBreaker的目的是保护分布式系统免受故障和异常，**提高系统的可用性和健壮性**。

当一个组件或服务出现故障时，Circuit Breaker会迅速切换到开放OPEN状态(保险丝跳闸断电)，阻止请求发送到该组件或服务从而避免更多的请求发送到该组件或服务。这可以减少对该组件或服务的负载，防止该组件或服务进一步崩溃，并使整个系统能够继续正常运行。同时，Circuit Breaker还可以提高系统的可用性和健壮性，因为它可以在分布式系统的各个组件之间自动切换，从而避免单点故障的问题。

**官方网站**：https://spring.io/projects/spring-cloud-circuitbreaker#overview

![](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202404101345025.png)

Circuit Breaker 只是一套规范和接口，落地实现者是[Resilience4J](Resilience4J/README.md)