# Nacos 配置

Nacos-server其实就是一个Java工程或者说是一个Springboot项目，配置文件在`nacos\conf`目录下，找到`application.properties`文件进行配置相关操作。

![image-20231110174212894](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202311101742653.png)

## 配置端口号及上下文路径

从配置文件可以看出，默认 Nacos 服务器的端口号为 8848，上下文路径为/nacos。一般都是 采用默认值，但也可以修改。

![image-20231110174334030](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202311101743650.png)

## 用户管理

登录nacos账户后，在权限控制-用户列表中，操作添加、删除普通用户，修改所有用户的密码。但不能删除管理员用户， 也不能将普通用户指定为管理员角色。

![image-20231110174939440](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202311101749306.png)

## 数据持久化

当使用默认配置启动Nacos时(单节点)，所有配置文件都被Nacos保存在自带的一个嵌入式数据库中。

> 在0.7版本之前，在单机模式时nacos使用嵌入式数据库实现数据的存储，不方便观察数据存储的基本情况。0.7版本增加了支持mysql数据源能力。

![image-20230330173535247](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/202306281454201.png)

**说明**：

- 如果使用内嵌数据库，注定会有存储上限，所以要将Nacos中的数据实现持久化。

- Nacos数据是存放在内存中的，无法持久化。使用集中式存储来保证数据的持久化，同时也为Nacos集群部署奠定了基础。

- Nacos采用了单一数据源，直接解决了分布式和集群部署中的一致性问题。

**数据库要求**：

- 安装MySQL数据库，版本要求：5.6.5+
- 初始化 mysql 数据库，数据库初始化文件：mysql-schema.sql
- 修改 conf/application.properties 文件，增加支持 mysql 数据源配置 (目前只支持 MySQL)， 添加 MySQL数据源的 url、用户名和密码

**操作步骤**：

1. 创建数据库，用于存储数据。

   ```sql
   CREATE DATABASE IF NOT EXISTS `nacos_config`;
   USE `nacos_config`
   ```

2. 直接导入到 `nacos_config`数据库即可，文件在conf目录中：

   ![image-20231111152602610](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202311111526345.png)

   将其导入到数据库，可以看到生成了很多的表：

   ![image-20231111155440423](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202311111554863.png)

3. 修改配置文件`application.properties`，在文件底部添加数据源配置：

   ```properties
   spring.datasource.platform=mysql
   
   db.num=1
   db.url.0=jdbc:mysql://127.0.0.1:3306/nacos_config?characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true
   db.user=root
   db.password=123123
   ```

   ![image-20231111155947936](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202311111559524.png)

4. 验证是否持久化到数据库中：启动Nacos，进入Nacos控制台，此时的Nacos控制台中焕然一新，之前的数据都不见了

   > 因为加入了新的数据源，Nacos从mysql中读取所有的配置文件，而刚刚初始化的数据库是干干净净的，自然不会有什么数据和信息显示。

   ![image-20231111160659180](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202311111607353.png)

5. 在公共空间(public)中新建一个配置文件DataID: `nacos-config.yaml`, 配置内容如下：

   ```yaml
   server: 
       port: 9989
   nacos:
       config: 配置文件已持久化到数据库中...
   ```

![image-20231111160306667](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202311111603712.png)

观察数据库`mynacos`中的数据库表 `config_info` ：

![image-20231111160406534](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202311111604950.png)
