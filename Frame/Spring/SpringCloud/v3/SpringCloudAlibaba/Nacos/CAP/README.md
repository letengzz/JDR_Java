# Nacos  CAP模式切换

默认情况下，Nacos Discovery 集群的数据一致性采用的是 AP模式。但其也支持 CP模式， 需要进行转换。

若要转换为 CP 的，可以提交如下 PUT 请求，完成 AP 到 CP 的转换： http://localhost:8848/nacos/v1/ns/operator/switches?entry=serverMode&value=CP

![image-20231112234453546](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202311122345860.png)