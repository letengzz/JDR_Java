# Apollo

## 配置中心

**Apollo 配置中心工作原理**：

![image-20231114195926693](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202311141959437.png)

其 Config Client 可以自动感知配置文件的更新。

存在两个不足： 

- 系统架构复杂 
- 配置文件支持类型较少，其只支持 xml、text、properties，不支持 json、yml