# Zuul

Zuul 是 Netflix 的开源 API 网关。Zuul 是基于 Servlet 的，使用同步阻塞 IO，不支持长连接。Zuul 是 Spring Cloud 生态中的一员。 

Zuul2.x 使用 Netty 实现了异步非阻塞 IO，支持长连接。但其未整合到 Spring Cloud。