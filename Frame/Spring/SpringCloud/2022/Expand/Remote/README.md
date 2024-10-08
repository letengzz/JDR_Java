# 微服务远程调用

消费者对于微服务的消费是通过 RestTemplate 完成的方式**弊端**： 

- 消费者对提供者的调用无法与业务接口完全吻合。例如，原本 Service 接口中的方法是 有返回值的，但经过 相关 API 调用后没有了其返回值，最终执行是否成功 用户并不清楚。再例如 对数据的删除与修改操作方法都没有返回值。


- 代码编写不方便，不直观。提供者原本是按照业务接口提供服务的，而经过一转手，变为了 URL，使得程序员在编写消费者对提供者的调用代码时，变得不直接、 不明了。没有直接通过业务接口调用方便、清晰。

****

- [远程调用 概念](Introduce/README.md)

**Spring Cloud分布式微服务远程调用解决方案**：

- [Spring Cloud OpenFeign](../../SpringCloud/OpenFeign/README.md)