# 集群流控

有这样一种场景：某微服务由 10 台主机构成的集群提供，现需要控制该微服务的某 API 总 QPS 为 50，如何实现？为每台主机分配单机 QPS 阈值为 5 可以实现吗？不可以。因为对 各个主机的访问可能并不均衡，可能有些尚未到达阈值，而有些由于已经超过了阈值而出现 了限流。 这种场景下就需要有一个专门的主机，用于统计、分配集群中 QPS，以达到最大限度的 降低由于流控而拒绝的请求数量。这就是集群流控。



## 集群流控构成

集群流控系统由两种角色构成：  Token Client：集群流控客户端，用于向所属 Token Server 通信，以请求 token。集群限 流服务端会返回给客户端结果，决定是否限流。  Token Server：集群流控服务端，处理来自 Token Client 的令牌请求，根据配置的集群规 则判断是否应该发放 Token（是否允许通过）。

## Token Server 启动模式

根据 Token Server 启动方式的不同，可以分为两种：

1. 独立模式：Token Server 作为独立的进程启动，独立部署，隔离性好，但是需要额外的部署操作

   ![image-20240429211159714](assets/image-20240429211159714.png)

2. 嵌入模式：

   Token Client 中的某个被指定为 Token Server，该主机同时具备 Server 与 Client 两种身份。该 模式下集群中各个主机都是对等的，Token Server 和 Client 可以随时通过提交一个 HTTP 请求 进行转变，无需单独部署，灵活性比较好。但隔离性不好，可能会对充当 Server 的 Client 产生影响，且 Token Server 的性能也会由于访问量的增加而受影响。

   ![image-20240429211216656](assets/image-20240429211216656.png)

## 独立模式

### 定义 TokenServer 

这里采用独立模式来启动 Token Server。至于嵌入模式，就是将下面的操作直接应用到 一个 Token Client 即可。

定义工程 

复制 06-consumer-persist-8080 工程，重命名为 06-cluster-token-server。

修改配置文件

![image-20240429211306825](assets/image-20240429211306825.png)

![image-20240429211312769](assets/image-20240429211312769.png)

 删除代码

将启动类之外的所有 java 代码全部删除。因为采用的是 Token Server 的独立启动模式， 该工程仅是一个 Token Server。

修改启动类代码

![image-20240429211339877](assets/image-20240429211339877.png)

### 定义 Token Client

定义工程 

复制 06-consumer-persist-8080 工程，重命名为 06-cluster-token-client。

 修改配置文件

![image-20240429211506843](assets/image-20240429211506843.png)

![image-20240429211511689](assets/image-20240429211511689.png)

修改启动类

![image-20240429211518984](assets/image-20240429211518984.png)

![image-20240429211522894](assets/image-20240429211522894.png)

### 启动 Token Server

首先要启动 Nacos 与 Sentinel Dashboard。注意，为了方便与 Nacos 间的数据同步，这 里启动前面修改过的 Sentinel Dashboard。然后再启动 Token Server。 Token Server 启动后，在 Sentinel 控制台的“集群流控”的 Token Server 列表中即可立即 看到该 Server。只不过，该 Server 目录的连接数为 0。

![image-20240429211537435](assets/image-20240429211537435.png)

### 启动 Token Client

启动三个 Token Client，修改端口号分别为 8080、7070、6060。再启动一个提供者工程 02-provider-nacos-8081。

### 查看 Sentinel 控制台

#### 查看 Token Client 列表

查看 Sentinel 控制台中的“集群流控”，点击“Token Client 列表”，可以查看到当前所有 的 Client 信息。并显示，它们已经连接到了 localhost:11111 的 Server。

![image-20240429211605539](assets/image-20240429211605539.png)

####  查看 Token Server 列表

点击“Token Server 列表”，在 Token Server 列表中可以看到新增了一个 ServerID 为 “localhost:11111（自主指定）”的 Token Server。该 Token Server 是一个临时 Server，是由三 个 Token Client 在启动时“自主指定”的，即三个 consumer 在应用中指定的要连接的那个 Token Server。该 Token Server 除了 port 外，其它都是“未知”。

![image-20240429211618789](assets/image-20240429211618789.png)

虽然在 ID 为 192.168.0.106@8720 的 Server 中可以看到“总连接数”为 3，但点击“管 理”按钮却会发现其没有一个 client。原因是，启动的 3 个 client 连接在了临时 Server 上。

![image-20240429211629283](assets/image-20240429211629283.png)

点击临时 Server 的“管理”可以看到该 Token Server 中包含了三个 client。

![image-20240429211638087](assets/image-20240429211638087.png)

由于该 Token Server 是一个临时 Server，没有用，所以，在该窗口中选中这三个 client， 将它们从中“移出(左箭头按钮) ”。

![image-20240429211647315](assets/image-20240429211647315.png)

点击“保存”后自动返回 Token Server 列表，发现该临时 Token Server 已经自动消失了。

![image-20240429211655566](assets/image-20240429211655566.png)

#### 为 Token Server 分配 Client

点击 Server 的“管理”按钮，可以看到在 client 选取区中已经具有了 client。将它们移 动到“已选取区”。

![image-20240429211751608](assets/image-20240429211751608.png)

![image-20240429211819595](assets/image-20240429211819595.png)

此时 Server 中的总连接数发生了变化。

![image-20240429211827630](assets/image-20240429211827630.png)

#### 新建流控规则

![image-20240429211837652](assets/image-20240429211837652.png)

此时打开 nacos 查看其配置列表，发现自动增加了一个配置文件。

![image-20240429211845973](assets/image-20240429211845973.png)

#### 为 Token Server 分配 Client

此时发现一个奇怪的情况：真正 Token Server 的连接数变为了 0（原来是 3）。再打开“连 接详情”，发现没有了连接的 Client。

![image-20240429211858853](assets/image-20240429211858853.png)

这是为什么？打开“管理”就发现了奥妙。
![image-20240429211907025](assets/image-20240429211907025.png)

原来，这三个 Client 已经出现在了真正的 Token Server 中，但还没有被选取。

![image-20240429211914358](assets/image-20240429211914358.png)

将这三个 Client 选取后保存，即可看到正常的如下所示的连接数。

![image-20240429211921611](assets/image-20240429211921611.png)

## 嵌入模式

在独立模式基础上可直接建立嵌入模式的集群。无需修改任何代码，只需修改 Sentinel 控制台的配置即可。

###  停掉 Token Server

将原来独立模式时的 Token Server 停掉。

### 新增 Token Server

![image-20240429212001683](assets/image-20240429212001683.png)

选择“应用内机器”，从“选择机器”中任意选择一个 Token Client 同时作为 Token Server。

![image-20240429212009483](assets/image-20240429212009483.png)

再将剩余的两个 client 移动到“已选取区”。

![image-20240429212016876](assets/image-20240429212016876.png)

![image-20240429212020650](assets/image-20240429212020650.png)

此时的嵌入模式的集群已经搭建完成。

![image-20240429212030182](assets/image-20240429212030182.png)

### 修改流控规则

仍可保持原来的流控规则，当然，也可修改。

