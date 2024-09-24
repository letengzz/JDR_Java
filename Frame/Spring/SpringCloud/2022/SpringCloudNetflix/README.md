# Spring Cloud Netflix

Netflix公司是目前微服务落地中最成功的公司。它开源了诸如Eureka、Hystrix、Zuul、Feign、Ribbon等广大开发者所知的微服务套件，统称为**Netflix OSS**。在当时Netflix OSS成为了微服务组件上事实的标准。但是微服务兴起不久，也就是2018年前后Netflix公司宣布其核心组件Hystrix、Ribbon、Zuul、Eureka等进入维护状态，不再进行新特性开发，只修Bug。

这直接影响了Spring Cloud项目的发展路线，Spring官方不得不采取了应对措施，在2019年的 SpringOne 2019大会中，Spring Cloud宣布Spring Cloud Netflix项目进入维护模式，并在2020年移除相关的Netflix OSS组件。

- [Eureka 注册中心](../SpringCloudNetflix/Eureka/README.md)
- [Ribbon 负载均衡](../SpringCloudNetflix/Ribbon/README.md)
- [Zuul 路由网关](Zuul/README.md)
- [Hystrix 服务熔断](Hystrix/README.md)