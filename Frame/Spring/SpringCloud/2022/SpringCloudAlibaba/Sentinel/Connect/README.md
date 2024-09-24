# Sentinel 服务连接到控制台

导入依赖：

```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
</dependency>
```

在配置文件中添加Sentinel相关信息(实际上Sentinel是本地在进行管理，但是可以连接到监控页面使用图形化操作)：

```yaml
spring:
  application:
    name: book-service
  cloud:
    sentinel:
      transport:
      	# 添加监控页面地址即可
        dashboard: localhost:8858
        #默认8719端口，假如被占用会自动从8719开始依次+1扫描,直至找到未被占用的端口
        port: 8719 
```

使用Sentinel对某个接口进行限流和降级等操作，一定要先访问下接口，使Sentinel检测出相应的接口 (懒加载机制，不会一上来就加载)：

![image-20230331160451879](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202304020131421.png)

![image-20240415210103778](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202404152101940.png)

可以在Sentinel控制台中对服务运行情况进行实时监控，可以看到监控的内容非常的多，包括时间点、QPS(每秒查询率)、响应时间等数据。

按照上面的方式，将所有的服务全部连接到Sentinel管理面板中。