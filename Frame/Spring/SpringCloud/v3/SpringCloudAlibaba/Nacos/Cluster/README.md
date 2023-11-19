# Nacos 集群搭建

单机版 Nacos 都存在单点问题，需要搭建Nacos集群，实现高可用。

官方方案：https://nacos.io/zh-cn/docs/cluster-mode-quick-start.html

![deployDnsVipMode.jpg](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202303301730370.png)

Nacos提供了三种集群部署方案：

- http://ip1:port/openAPI 直连ip模式，机器挂则需要修改ip才可以使用。


- http://SLB:port/openAPI 挂载SLB模式(内网SLB，不可暴露到公网，以免带来安全风险)，直连SLB即可，下面挂server真实ip，可读性不好。


- http://nacos.com:port/openAPI 域名 + SLB模式(内网SLB，不可暴露到公网，以免带来安全风险)，可读性好，而且换ip方便，推荐模式

Nacos推荐在所有的服务端之前建立一个负载均衡，通过访问负载均衡服务器来间接访问到各个Nacos服务器。实际上就是比如有三个Nacos服务器做集群，但是每个服务不可能把每个Nacos都去访问一次进行注册，实际上只需要在任意一台Nacos服务器上注册即可，Nacos服务器之间会自动同步信息，但是如果随便指定一台Nacos服务器进行注册，如果这台Nacos服务器挂了，但是其他Nacos服务器没挂，这样就没办法完成注册了，但是实际上整个集群还是可用的状态。

所以就需要在所有Nacos服务器之前搭建一个SLB(服务器负载均衡)，这样就可以避免上面的问题了。但是如果要实现外界对服务访问的负载均衡，就得用比如Gateway来实现，而这里实际上可以用一个更加方便的工具：Nginx来实现。

SLB最上方还有一个DNS是因为SLB是裸IP，如果SLB服务器修改了地址，那么所有微服务注册的地址也得改，所以这里是通过加域名，通过域名来访问，让DNS去解析真实IP，这样就算改变IP，只需要修改域名解析记录即可，域名地址是不会变化的。

在多节点集群模式下，数据肯定是不能各存各的，所以，需要先[实现数据持久化](../Configuration/README.md#数据持久化)统一存储支持，只需要让所有的Nacos服务器连接MySQL进行数据存储即可。

## Windows 搭建集群

搭建 Nacos 集群包含三台 Nacos 服务器，由于这些 Nacos 都在同一台主机， 所以这里创建的集群实际只有端口号不同，是个伪集群。

创建一个 nacos-server-cluster 目录，用于存放三个 Nacos 服务器。复制原来配置好的单机版的 Nacos 到这个目录，并重命名为 nacos8850

![image-20231112151144989](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202311121511730.png)

打开 nacos8850/conf，重命名其中的 cluster.conf.example 为 cluster.conf。然后打开该文件，在其中写入三个 nacos 的 ip:port。

**注意**：不能写为 localhost 与 127.0.0.1，且这三个端口 号不能连续。否则会报地址被占用异常。 

![image-20231112150811161](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202311121508284.png)

打开 nacos8850/conf/application.properties 文件，修改端口号为 8850。 

![image-20231112151031923](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202311121510451.png)

复制目录 将 nacos8850 目录复制三份，分别命名为 nacos8852、nacos8854。 并修改各自目录中 conf/application.properties 文件中 nacos 的端口号为 8852 与 8854。 

![image-20231112151913558](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202311121519389.png)

启动集群：逐个双击三个 nacos 目录中的 bin/startup.cmd 命令，逐个启动三台 nacos。 从启动日志上可以看到其提供的 SLB 服务访问地址及三台 Nacos 节点地址。

![image-20231112152134325](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202311121521761.png)

查看 nacos 平台：在浏览器中使用SLB的VIP访问地址即可打开[nacos服务器平台](http://localhost:8850/nacos/#/login)，查看到集群节点列表。 点击"节点元数据"可查看 nacos 的节点元数据。

![image-20231112152313770](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202311121523026.png)

Client 连接 Nacos 集群：直接将微服务配置文件 application.yaml 中的 nacos 地址更换为 Nacos 集群的 VIP 地址。 

```yaml
spring:
  application:
    # 应用名称 borrow-service
    name: borrow-service
  cloud:
    nacos:
      discovery:
        # 配置Nacos注册中心地址
        server-addr: 192.168.0.111:8850,192.168.0.111:8852,192.168.0.111:8854
```

微服务重启后便可以 Nacos 平台中查看到它们的信息了。

![image-20231112152826515](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202311121528803.png)

## Linux 搭建集群

创建两个Nacos服务器，做一个迷你的集群，将nacos服务端上传到Linux服务器(注意需要提前安装好JRE 8或更高版本的环境)：

![image-20231112164516701](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202311121645433.png)

解压之后，对其配置文件进行修改，首先是`application.properties`配置文件，修改以下内容，包括MySQL服务器的信息：

```properties
### Default web server port:
server.port=8850

#*************** Config Module Related Configurations ***************#
### If use MySQL as datasource:
spring.datasource.platform=mysql
spring.sql.init.platform=mysql

### Count of DB:
db.num=1

### Connect URL of DB:
db.url.0=jdbc:mysql://192.168.0.111/nacos?characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true&useUnicode=true&useSSL=false&serverTimezone=UTC
db.user.0=nacos
db.password.0=123123
```

修改`conf/`下的`cluster.conf.example`文件，将其命名为`cluster.conf`：

![image-20231112165311486](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202311121653978.png)

端口记得使用内网IP地址：

![image-20231112173702635](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202311121737307.png)

修改Nacos的内存分配以及前台启动，直接修改`startup.sh`文件：

![image-20231112170339038](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202311121703431.png)

保存之后，将nacos复制一份，并将端口修改为8852、8853：

![image-20230706211707505](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202307062117059.png)

```shell
mkdir nacos2
mkdir nacos3
cp -r nacos/* nacos2
cp -r nacos/* nacos3
```

接着启动三个Nacos服务器：

![image-20231112171441400](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202311121714117.png)

```bash
bash nacos/bin/startup.sh
bash nacos2/bin/startup.sh
bash nacos3/bin/startup.sh
```

然后打开管理面板，可以看到三个节点都已经启动了：

![image-20231112221921270](assets/image-20231112221921270.png)

接着需要添加一个SLB，用Nginx做反向代理：

> *Nginx* (engine x) 是一个高性能的[HTTP](https://baike.baidu.com/item/HTTP)和[反向代理](https://baike.baidu.com/item/反向代理/7793488)web服务器，同时也提供了IMAP/POP3/SMTP服务。它相当于在内网与外网之间形成一个网关，所有的请求都可以由Nginx服务器转交给内网的其他服务器。

直接安装：

> Ubuntu

```sh
sudo apt install nginx
```

> Centos

```shell
yum install nginx -y
```

可以看到直接请求80端口之后得到，表示安装成功：

![image-20231112223125093](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202311161301261.png)

让其代理刚刚启动的三个Nacos服务器，需要对其进行一些配置。配置文件位于`/etc/nginx/nginx.conf`，添加以下内容：

```nginx
#添加在上游刚刚创建好的三个nacos服务器
upstream nacos-server {
        server 192.168.56.182:8850;
        server 192.168.56.182:8852;
        server 192.168.56.182:8854;
}

server {
        listen   80;
        server_name  localhost;

        location /nacos {
                proxy_pass http://nacos-server;
        }
}
```

重启Nginx服务器(`nginx -s reload`)，成功连接：

```nginx
nginx -s reload
```

![image-20231112223943052](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202311122239422.png)

然后将所有的服务全部修改为云服务器上Nacos的地址，启动：

```yaml
spring:
  application:
    # 应用名称 borrow-service
    name: borrow-service
  cloud:
    nacos:
      discovery:
        # 配置Nacos注册中心地址
        server-addr: 192.168.56.182:8850,192.168.56.182:8852,192.168.56.182:8854
```

![image-20231112224750657](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202311122247749.png)

这样就搭建好了Nacos集群。