# 规则持久化

一旦重启微服务应用，Sentinel规则将消失，生产环境需要将配置规则进行持久化。

将限流配置规则持久化进Nacos保存，只要刷新某个rest地址，Sentinel控制台的流控规则就能看到，只要Nacos里面的配置不删除，Sentinel的流控规则持续有效。

## 操作步骤

1. 添加依赖：

   ```xml
   <!--SpringCloud ailibaba sentinel-datasource-nacos -->
   <dependency>
       <groupId>com.alibaba.csp</groupId>
   	<artifactId>sentinel-datasource-nacos</artifactId>
   </dependency>
   ```

2. 配置文件：

   > application.yaml

   ```yaml
   server:
     port: 8401
   spring:
     cloud:
       nacos:
         discovery:
           server-addr: localhost:8848         #Nacos服务注册中心地址
       sentinel:
         transport:
           # 添加监控页面地址即可
           dashboard: localhost:8858 #配置Sentinel dashboard控制台服务地址
           port: 8719 #默认8719端口，假如被占用会自动从8719开始依次+1扫描,直至找到未被占用的端口
         web-context-unify: false # controller层的方法对service层调用不认为是同一个根链路
         datasource: #Sentinel持久化配置
           ds1: #自定义key 也可以叫做限流类型等等
             nacos:
               server-addr: localhost:8848 #nacos地址
               dataId: flow #nacos配置文件dataId
               groupId: DEFAULT_GROUP #nacos配置GroupId
               data-type: json #nacos配置数据类型
               rule-type: flow #流控规则  具体类型见com.alibaba.cloud.sentinel.datasource.RuleType
               # 用户名
               username: nacos
               # 密码
               password: nacos
           ds2: #自定义key 也可以叫做限流类型等等
             nacos:
               server-addr: localhost:8848 #nacos地址
               dataId: degrade #nacos配置文件dataId
               groupId: DEFAULT_GROUP #nacos配置GroupId
               data-type: json #nacos配置数据类型
               rule-type: degrade #流控规则  具体类型见com.alibaba.cloud.sentinel.datasource.RuleType
               # 用户名
               username: nacos
               # 密码
               password: nacos
   ```
   
3. Nacos创建配置：

   ```json
   [
     {
       "resource": "/book/{uid}",
       "limitApp": "default",
       "grade": 1,
       "count": 1,
       "strategy": 0,
       "controlBehavior": 0,
       "clusterMode": false
     }
   ]
   ```

   ![image-20240427165835109](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202404271658817.png)

4. Sentinel 使用懒加载方式对规则进行加载：

   ![image-20240427165940720](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202404271659180.png)

## 规则类型

![](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202404182329840.png)

## Nacos参数 

- resource：资源名称

- limitApp：来源应用
- grade：阈值类型，0表示线程数，1表示QPS
- count：单机阈值
- strategy：流控模式，0表示直接，1表示关联，2表示链路
- controlBehavior：流控效果，0表示快速失败，1表示Warm Up，2表示排队等待
- clusterMode：是否集群。
