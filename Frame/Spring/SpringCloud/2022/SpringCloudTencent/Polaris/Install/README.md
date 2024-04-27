# Polaris 安装

Polaris 下载地址：https://github.com/polarismesh/polaris/releases

北极星支持单机版的安装架构，适用于用户在开发测试阶段，通过本机快速拉起北极星服务进行验证。

![img](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/202306301422207.png)

**单机版包含以下4个组件**：

- polaris-console：可视化控制台，提供服务治理管控页面
- polaris-server：控制面，提供数据面组件及控制台所需的后台接口
- polaris-limiter: 分布式限流服务端，提供全局配额统计的功能
- prometheus：服务治理监控所需的指标汇聚统计组件

**单机版默认占用以下端口**：

- polaris-console：8080(http/tcp)
- polaris-server：8090(http/tcp，注册中心端口)、8091(grpc/tcp，注册中心端口)、8093(grpc/tcp，配置中心端口)
- polaris-limiter：8101(grpc/tcp)、8100(http/tcp)
- prometheus：9090(http/tcp)、9091(http/tcp)

**操作步骤**：

1. 下载软件包：

   单机版的安装需要依赖单机版软件包，单机版软件包的命名格式为`polaris-standalone-release_*.zip`：

   ![单机版](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/202306301422407.png)

2. 下载后需要进行解压，如果有需要自定义单机版相关组件的监听端口，需修改压缩包内的**port.properties**文件。

   > port.properties

   ```properties
   polaris_eureka_port=8761
   polaris_open_api_port=8090
   polaris_service_grpc_port=8091
   polaris_config_grpc_port=8093
   polaris_prometheus_sd_port=9000
   polaris_xdsv3_port=15010
   polaris_console_port=8080
   prometheus_port=9090
   pushgateway_port=9091
   ```

### 使用 Linux 安装

下载Linux单机版软件包（`polaris-standalone-release_$version.linux.$arch.zip`），执行安装命令：

```bash
unzip polaris-standalone-release_$version.linux.$arch.zip

cd polaris-standalone-release_$version.linux.$arch

bash install.sh
```

### 使用 Window 安装

**注意**：

- 依赖powershell 5.0及以上版本（Windows 10及以上版本默认安装）
- 需要以管理员身份运行安装脚本，执行powershell需要进行授权操作
- 安装脚本可能遭到系统安全软件的误杀，请在安全软件中执行信任操作

下载Windows单机版软件包（`polaris-standalone-release_$version.windows.$arch.zip`），执行安装命令：

```bash
执行解压：polaris-standalone-release_$version.windows.$arch.zip

进入目录：polaris-standalone-release_$version.windows.$arch

执行脚本：install.bat
```

### 使用 Mac 安装

**注意**：

- 请在【关于本机】设置中查看Mac机器的芯片类型（Intel/Apple）
- Intel芯片请使用amd64的软件包，Apple芯片请使用arm64的软件包

下载Mac单机版软件包（`polaris-standalone-release_$version.darwin.$arch.zip`），执行安装命令：

```bash
unzip polaris-standalone-release_$version.darwin.$arch.zip

cd polaris-standalone-release_$version.darwin.$arch

bash install.sh
```

### 使用 Docker 安装

查看当前镜像需要暴露的端口信息：

```bash
docker image ls -a | grep "polarismesh/polaris-standalone" | awk '{print $3}' | xargs docker inspect --format='{{range $key, $value := .Config.ExposedPorts}}{{ $key }}{{end}}'| awk '{ gsub(/\/tcp/, " "); print $0 }'
```

执行以下命令启动：

```bash
# Publish a container's port(s) to the host
docker run -d --privileged=true \
-p 15010:15010 \
-p 8101:8101 \
-p 8100:8100 \
-p 8080:8080 \
-p 8090:8090 \
-p 8091:8091 \
-p 8093:8093 \
-p 8761:8761 \
-p 9090:9090 polarismesh/polaris-standalone:latest
```

**说明：** 8101、8100 端口不得通过 `--publish` 映射为别的端口

### 使用 Docker Compose 安装

下载 Docker Compose 安装包: `polaris-standalone-release_$version.docker-compose.zip`

创建mysql、redis 存储卷，方便数据持久化：

```shell
docker volume create --name=vlm_data_mysql
docker volume create --name=vlm_data_redis
```

启动服务：

```bash
执行解压：polaris-standalone-release_$version.docker-compose.zip

进入目录：polaris-standalone-release_$version.docker-compose

执行命令：docker-compose up
```

### 安装验证

1. 打开控制台：在浏览器里输入北极星控制台地址（127.0.0.1:8080），非容器化场景127.0.0.1可替换成安装北极星的机器host。

2. 登录控制台的默认登录账户信息：默认的用户名和管理员密码都是`polaris`，直接登陆即可，可以看到进入管理页面：

   ![控制台](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/202306301422607.png)

3. 新建服务：进入服务列表页面，点击【新建】按钮，确认是否可以新建服务。新建服务成功表示安装成功

   ![新建服务](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/202306301433530.png)
