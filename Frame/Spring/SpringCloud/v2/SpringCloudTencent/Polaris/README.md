# Polaris

![img](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/202306301418173.png)

- [Polaris 概述]()
- [Polaris 安装]()
- [Polaris  服务注册与发现]()
- [Polaris  配置中心]()
- [Polaris 服务限流]()
- [Polaris 服务熔断]()
- [Polaris 服务路由]()
- [RPC 增强]()
- [场景化插件](https://github.com/Tencent/spring-cloud-tencent/wiki/场景化插件)



# Polaris 概述

北极星是腾讯开源的服务治理平台，致力于解决分布式和微服务架构中的服务管理、流量管理、配置管理、故障容错和可观测性问题，针对不同的技术栈和环境提供服务治理的标准方案和最佳实践。

## 北极星具备功能

![img](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/202306301336555.png)

北极星具备服务管理、流量管理、故障容错、配置管理和可观测性五大功能：

- **服务管理**：包含服务发现、服务注册、健康检查和元数据管理。
  - 服务发现：支持 HTTP、SDK 和 DNS 服务发现方式。
  - 服务注册：支持 HTTP、SDK、控制台操作和 K8s 服务注册方式。
  - 健康检查：支持服务实例上报心跳，通过心跳判断实例是否健康，及时剔除异常实例。
  - 元数据管理：支持在服务和实例上配置协议、版本和位置等标签，实现动态路由等功能。
- **流量管理**：包含动态路由、负载均衡和访问限流。
  - 动态路由：支持自定义路由策略，将服务的部分请求路由到部分实例，用于灰度发布等应用场景。
  - 负载均衡：支持权重轮训、权重随机和权重一致性 Hash 等负载均衡算法。
  - 访问限流：支持本地和分布式两种模式，被限流的请求支持排队和自定义响应。
- **故障容错**：包含服务熔断和节点熔断。
  - 服务熔断：对服务或者接口进行熔断，如果服务或者接口发生熔断，返回自定义响应。
  - 节点熔断：对服务实例进行熔断，不会将请求路由到熔断的服务实例，降低请求失败率。
  - 主动探测：服务和节点熔断除了被动探测，还支持主动探测，进一步降低请求失败率。
- **配置管理**：包含配置变更、配置校验、版本管理和灰度发布等功能。
- **可观测性**：提供业务流量、系统事件和操作记录等监控视图。

北极星的功能需要控制面和数据面配合实现：

- **控制面**：负责服务和配置数据的管理和下发，负责流量管理和熔断降级策略的管理和下发。
- **数据面**：负责全部服务发现和治理功能的客户端实现，采用插件化设计，支持按需加载和使用。

数据面功能分为三个部分：

- **服务作为被调**：当一个服务被其他服务调用时，可以使用服务注册、上报心跳、访问限流和访问鉴权功能。
- **服务作为主调**：当一个服务调用其他服务时，可以使用服务发现、动态路由、负载均衡和熔断降级功能。
- **公共部分**：支持拉取配置数据和上报监控数据。

## 北极星组件

![img](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/202306301337041.png)

北极星的系统组件分为控制台、控制面和数据面三个部分：

- **控制台**：提供简单易用的管理页面，支持用户和权限管理。
- **控制面**：包含核心组件 Polaris 和可选的功能组件，核心组件可以满足绝大部分业务需求，可选的功能组件按需部署。
- **数据面**：提供多语言 SDK、开发框架、Java Agent 和网格代理四种形态的实现，满足不同的业务场景和开发模式，支持异构服务的互联互通和统一治理。

控制面组件：

- **Polaris**：支持各种形态的数据面接入，支持服务和配置数据的管理和下发，支持流量管理和熔断降级策略的管理和下发，可以覆盖服务注册中心、服务网格控制面和配置中心的功能。
- **Polaris Controller**：可选的功能组件，支持 K8s 服务同步和网格代理注入。K8s 服务同步将 K8s 服务按需同步到北极星，用户不需要在应用程序里显式地注册服务。网格代理注入按需在应用程序 Pod 里注入北极星 Sidecar，以流量代理的方式实现服务发现和治理功能。

数据面组件：

- **SDK**：北极星提供轻量级的多语言 SDK，使用方法和绝大部分客户端软件类似，用户在应用程序里引入北极星 SDK。这种数据面形态以无流量代理的方式实现服务发现和治理功能，没有额外的性能和资源损耗，不会增加现网运维和问题定位的成本。
- **开发框架**：北极星 SDK 可以被集成到开发框架内部，如果用户使用开发框架，不需要显式地引入北极星 SDK。对于 Spring Cloud、Dubbo 和 gRPC 等开发框架，北极星提供可以无缝集成的依赖包。另外，go-micro、go-kratos、go-zero、GoFrame 和 CloudWeGo 等开发框架社区也提供北极星插件。
- **Java Agent**：对于 Spring Cloud 和 Dubbo 等 Java 开发框架，北极星支持 Java 生态常用的 Agent 接入模式。用户只需要在应用程序的启动命令中引入 Polaris Java Agent，即可将北极星的服务发现和治理功能引入应用程序，不需要改动任何代码和配置文件。
- **网格代理**：北极星网格代理在应用程序 Pod 里注入 Polaris Sidecar 和 Proxy，前者通过劫持 DNS 解析将请求转到后者，后者通过流量代理实现服务发现和治理功能。这种数据面形态适合性能和资源损耗不敏感的业务，要求业务具备网格代理的运维能力。

# Polaris 安装

Polaris 下载地址：https://github.com/polarismesh/polaris/releases

## 单机版安装

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

## 集群版安装

