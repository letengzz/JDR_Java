# Windows 安装与启动

Nacos服务器是独立安装部署的，因此需要下载最新的Nacos服务端程序。

## 下载安装

**下载地址**：https://github.com/alibaba/nacos 

有两种资源下载方式：源码下载与打过包的工程下载。点击"最新稳定版本"(下载2.2.1及以上版本)

，可以选择性地下载最新版的这两种资源。这里选择编译过的 zip 压缩资源。

![image-20231110160337665](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202311101603128.png)



下载2.2.3版本并将文件进行解压即可：

![image-20231110162151059](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202311101621482.png)

## 启动

从 nacos2.2.0.1 版本开始，nacos 配置文件中去掉了默认的鉴权配置，需要手动添加，否则无法启动 nacos。

官方解释：https://nacos.io/zh-cn/docs/v2/guide/user/auth.html?spm=a2c6h.13066369.question.19.318936e3fJ3kqO

打开conf/application.properties，修改配置：

![image-20231110164644428](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202311101646231.png)

### IDEA内部启动

直接将其拖入到项目文件夹下，便于在IDEA内部启动，接着添加运行配置：

![image-20231110170945007](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202311101709467.png)

其中`-m standalone`表示单节点模式，接着点击启动：

![image-20231110171002566](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202311101710189.png)

**注意**：当出现表达式或语句中包含意外的标记“standalone”。

![image-20231110171928526](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202311101719355.png)

**解决办法**：IDEA2022.1默认的Terminal是用powershell运行的，进入Settings-Tools-Terminal， 将`Shell path`修改为cmd的路径修改为cmd的路径

![image-20231110172055420](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202311101720203.png)

![image-20231110172153096](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202311101721698.png)

### 命令行启动

由于其默认是集群方式启动，所以若要单机启动在命令行中启动，只需要进入nacos/bin目录，输入命令即可启动：

```bash
startup.cmd -m standalone
```

![image-20231110164847686](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202311101648325.png)

## 访问Nacos

启动成功，访问 http://localhost:8848/nacos/#/login 打开Nacos控制台的登录页面：

![image-20231110165238148](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202311101652314.png)

默认的用户名和管理员密码都是`nacos`，该默认账号与密码存放在 nacos 内置 mysql数据库中的，而非存放在某配置文件中。若要添加账号或修改密码，可在登录后通过页面修改。直接登陆即可，可以看到进入管理页面：

![image-20231110165303299](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202311101653716.png)
