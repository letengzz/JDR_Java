# Zookeeper 配置中心

![image-20231114202924529](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202312021214048.png)

Zookeeper 作为配置中心，其工作原理与[SpringCloud Config](../../SpringCloud/Config/README.md)、[Nacos Config](../../SpringCloudAlibaba/Nacos/Config/README.md)、[Apollo](../../Apollo/README.md)都不同。其没有第三方服务器去存 储配置数据，而是将配置数据存放在自己的 Znode 中了。当配置中心中的配置数据发生了变 更，Config Client 也是可以自动感知到的 (Watcher 监听机制)。
