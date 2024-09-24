# 改造 Sentinel 控制台

基于 Sentinel 控制台的源码改造目的的不同，有两种不同的改造方式：

- 以测试目的的改造
- 以生产目的的改造

## 下载源码并导入

Sentinel Dashboard 的源码是包含在 Sentinel 框架整个源码中的，所以需要从 Sentienl 官 网下载整个 Sentinel 源码。不过，下载源码时需要注意下载的版本为 1.8.6。 源码下载解压后，可以直接导入到 Idea 中。

## 测试目的改造

该改造方式仅供程序员自己在开发时测试使用，不能用于生产中。

### 修改 pom

直接在 Idea 中就可以打开该下载的 Sentinel 源码工程。在工程中找到 sentinel-dashboard 模块中的 pom 文件，将其中的 sentinel-datasource-nacos 依赖中的 test 依赖范围注释掉，以 保证打包时可以将该依赖打包到工程 jar 包中。

![image-20240429204838606](assets/image-20240429204838606.png)

### 目录复制 

将 sentinel-dashboard 模块的 test 下的 rule/nacos 目录整体复制到 main 下的如下 rule 目 录中。

![image-20240429204914912](assets/image-20240429204914912.png)

![image-20240429204921677](assets/image-20240429204921677.png)

### 修改配置文件

在sentinel-dashboard模块的配置文件application.properties中添加nacos服务器的地址、 username 与 password。

![image-20240429204935496](assets/image-20240429204935496.png)

### 修改 NacosConfig 类

找到前面刚添加的 rule.nacos 包中的 NacosConfig 类，该类用于获取 Nacos 配置中心中 的数据。对其进行修改，指定 Nacos 配置中心的地址。 找到前面刚添加的 rule.nacos 包中的 NacosConfig 类，该类用于获取 Nacos 配置中心中 的数据。对其进行修改，指定 Nacos 配置中心的地址、username 与 password。

![image-20240429204953850](assets/image-20240429204953850.png)

### 修改控制器类

1. 找到实例名称 

找到前面刚添加的 rule.nacos 包中的 FlowRuleNacosProvider 与 FlowRuleNacosPublisher 类，在其中复制出其实例名称。

![image-20240429205014327](assets/image-20240429205014327.png)

![image-20240429205020005](assets/image-20240429205020005.png)

2. 修改实例名称

   直接在 controller.v2.FlowControllerV2 类中修改第 52 与 55 行中要注入实例的名称为前面 复制的两个名称。

   ![image-20240429205035377](assets/image-20240429205035377.png)

3. 修改页面
   修改 src/main/webapp/resources/app/scripts/directives/sidebar/sidebar.html 中的代码：将 原本注释掉的第 42-45 行的注释去掉。

   ![image-20240429205049321](assets/image-20240429205049321.png)

4. 重新打包 dashboard

   对 Sentinel Dashboard 进行重新打包。

   ![image-20240429205108920](assets/image-20240429205108920.png)

   ![image-20240429205113866](assets/image-20240429205113866.png)

5. 启动新的 dashboard

   重新打包修改过的 sentinel-dashboard 模块，然后重新启动新的 dashboard 的 jar 包。

   ![image-20240429205128229](assets/image-20240429205128229.png)

### 修改微服务应用

通过前面的官网文档可以看出，其要求 groupId 必须是 SENTINEL_GROUP，流控规则的 dataId 名称必须为${spring.application.name}-flow-rules，所以需要修改应用中相应的配置项。 直接修改 06-consumer-persist-8080 工程的配置文件即可。

![image-20240429205151588](assets/image-20240429205151588.png)



## 生产目的改造

该改造方式可应用于 FlowRule、DegradeRule、SystemRule、AuthorityRule、ParamFlowRule。

### 修改 pom

直接在 Idea 中就可以打开该下载的 Sentinel 源码工程。在工程中找到 sentinel-dashboard 模块中的 pom 文件，将其中的 sentinel-datasource-nacos 依赖中的 test 依赖范围注释掉，以 保证打包时可以将该依赖打包到工程 jar 包中。

![image-20240429205213591](assets/image-20240429205213591.png)

### 类复制/移动

1. 从 test 到 main 复制 

   将 sentinel-dashboard 模块的 test 下的 rule/nacos 目录整体复制到 main 下的如下 rule 目 录中。

   ![image-20240429205238982](assets/image-20240429205238982.png)

   ![image-20240429205244824](assets/image-20240429205244824.png)

2. 移动读写类

   在main下的rule.nacos包下新建子包flow，并将rule.nacos包下的FlowRuleNacosProvider 类与 FlowRuleNacosPublisher 类移动到 rule.nacos.flow 包下。

   ![image-20240429205301949](assets/image-20240429205301949.png)

### 修改 NacosConfigUtil 类

在main下的rule.nacos包下新建子包flow，并将rule.nacos包下的FlowRuleNacosProvider 类与 FlowRuleNacosPublisher 类移动到 rule.nacos.flow 包下。

![image-20240429205325606](assets/image-20240429205325606.png)

### 修改 NacosConfigUtil 类

打开 NacosConfigUtil 类，除了 GROUP_ID 外，只能找到我们需要的 5 种规则中的 2 种规 则文件后缀。

![image-20240429205339186](assets/image-20240429205339186.png)

添加另外三种规则文件的后缀

![image-20240429205349388](assets/image-20240429205349388.png)

### 修改配置文件

在sentinel-dashboard模块的配置文件application.properties中添加nacos服务器的地址。

![image-20240429205405266](assets/image-20240429205405266.png)

### 修改 NacosConfig 类

对于 NacosConfig 类的修改，主要有两类：Nacos 服务器相关属性，添加另外 4 种 规则的编码转换器与解码转换器。

1. Nacos 属性相关

   ![image-20240429205429176](assets/image-20240429205429176.png)

2.  添加 Authority 规则转换器

   复制自带的 Flow 规则的两个转换器方法，在此基础上修改

   ![image-20240429205444614](assets/image-20240429205444614.png)

3. 添加 Degrade 规则转换器

   ![image-20240429205453799](assets/image-20240429205453799.png)

4. 添加 System 规则转换器

   ![image-20240429205503744](assets/image-20240429205503744.png)

5. 添加 ParamFlow 规则转换器：

   ![image-20240429205514809](assets/image-20240429205514809.png)

### 添加读写类

#### 添加 Authority 规则读写类

在 rule.nacos 包下新建一个子包 authority，然后将 rule.nacos.flow 包中的两个类复制到 该子包中，再分别重命名为 AuthorityRuleNacosProvider 与 AuthorityRuleNacosPublisher。

![image-20240429205551351](assets/image-20240429205551351.png)

1. 修改 AuthorityRuleNacosProvider 类

   将原来所有的 Flow 全部替换为 Authrotiry 即可。该类用于将 Nacos 中 JSON 格式的规则 文件转换为规则实体对象。即用于“读取”Nacos 中的数据的。

   ![image-20240429205713055](assets/image-20240429205713055.png)

2.  修改 AuthorityRuleNacosPublisher 类

   将原来所有的 Flow 全部替换为 Authrotiry 即可。该类用于将规则实体对象转换为 JSON 字符串后写入到 Nacos 中相应的规则文件中。即，其充当的是规则的“发布者”。

   ![image-20240429205729485](assets/image-20240429205729485.png)

####  添加 Degrade 规则读写类

在 rule.nacos 包下新建一个子包 degrade，然后将 rule.nacos.flow 包中的两个类复制到该 子包中，再分别重命名为 DegradeRuleNacosProvider 与 DegradeRuleNacosPublisher。

1.  修改 DegradeRuleNacosProvider 类

   ![image-20240429205805271](assets/image-20240429205805271.png)

2. 修改 DegradeRuleNacosPublisher 类

   ![image-20240429205815184](assets/image-20240429205815184.png)

3.  添加 System 规则读写类

   在 rule.nacos 包下新建一个子包 system，然后将 rule.nacos.flow 包中的两个类复制到该 子包中，再分别重命名为 SystemRuleNacosProvider 与 SystemRuleNacosPublisher。

   1. 修改 SystemRuleNacosProvider 类

      ![image-20240429205857110](assets/image-20240429205857110.png)

   2. 修改 SystemRuleNacosPublisher 类

      ![image-20240429205906431](assets/image-20240429205906431.png)

#### 添加 ParamFlow 规则读写类

在 rule.nacos 包下新建一个子包 param，然后将 rule.nacos.flow 包中的两个类复制到该 子包中，再分别重命名为 ParamRuleNacosProvider 与 ParamRuleNacosPublisher。

1.  修改 ParamRuleNacosProvider 类

   ![image-20240429205941883](assets/image-20240429205941883.png)

2.  修改 ParamRuleNacosPublisher 类

   ![image-20240429205953993](assets/image-20240429205953993.png)

###  修改处理器类

#### 修改 FlowControllerV2 类

直接在 controller.v2.FlowControllerV2 类中修改第 52 与 55 行中要注入的实例的名称为 Nacos 的 provider 与 publisher。

![image-20240429210026215](assets/image-20240429210026215.png)

#### 修改 AuthorityRuleController 类

修改 controller 包中的 AuthorityRuleController 类。

1. 添加/删除成员变量

   在类声明中添加自动注入的 ruleProvider 与 rulePublisher，并将 sentinelApiClient 删除。

   ![image-20240429210104863](assets/image-20240429210104863.png)

2.  修改 GET 方法

   将 sentinelApiClient 删除后报错的用于读取规则的语句删除，然后替换为 ruleProvider 的 getRules()方法，用于从 Nacos 配置中心读取规则配置文件，并转换为规则实体。

   ![image-20240429210122911](assets/image-20240429210122911.png)

3. 修改 publisRules()方法

   对 sentinelApiClient 删除后报错的 publishRules()方法进行修改。将报错的用于进行规则 实体持久化的语句删除，替换为 rulePublisher 的 publish()方法，用于将规则持久化到 Nacos 配置中心。 不过需要注意的是，rulePublisher.publish()方法没有返回值，且会抛出异常，所以需要 将 publishRules()方法的返回值修改为 void，并抛出 Exception 异常。

   ![image-20240429210139646](assets/image-20240429210139646.png)

4. 修改 POST 方法

   publishRules()方法的修改会引发 POST、PUT 与 DELETE 方法中对 publishRules()方法引用 处的报错。将这些报错语句修改为变更后的 publishRules()方法引用。注意，这些引用所在方 法需要抛出异常。

   ![image-20240429210157259](assets/image-20240429210157259.png)

5. 修改 PUT 方法

   ![image-20240429210208973](assets/image-20240429210208973.png)

6. 修改 DELETE 方法

   ![image-20240429210219208](assets/image-20240429210219208.png)

#### 修改 DegradeRuleController 类

修改 controller 包中的 DegradeRuleController 类

1.  添加/删除成员变量

   ![image-20240429210311726](assets/image-20240429210311726.png)

2. 修改 GET 方法

   ![image-20240429210321510](assets/image-20240429210321510.png)

3. 修改 publisRules()方法

   ![image-20240429210332726](assets/image-20240429210332726.png)

4. 、 修改 POST 方法

   ![image-20240429210344150](assets/image-20240429210344150.png)

5.  修改 PUT 方法

   ![image-20240429210356536](assets/image-20240429210356536.png)

6. 修改 DELETE 方法

   ![image-20240429210433130](assets/image-20240429210433130.png)

7. 修改 checkEntityInternal()

   ![image-20240429210441505](assets/image-20240429210441505.png)

#### 修改 SystemRuleController 类

修改 controller 包中的 SystemRuleController 类。

1. 、 添加/删除成员变量

   ![image-20240429210514395](assets/image-20240429210514395.png)

2.  修改 checkBasicParams()方法

   ![image-20240429210524240](assets/image-20240429210524240.png)

3.  修改 query 方法

   ![image-20240429210533262](assets/image-20240429210533262.png)

4. 修改 publishRules()方法

   ![image-20240429210544319](assets/image-20240429210544319.png)

5. 修改 add 方法

   ![image-20240429210554994](assets/image-20240429210554994.png)

6. 修改 update 方法

   ![image-20240429210608301](assets/image-20240429210608301.png)

7.  修改 delete 方法

   ![image-20240429210618711](assets/image-20240429210618711.png)

#### 修改 ParamFlowRuleController 类

修改 controller 包中的 ParamFlowRuleController 类。

1. 添加/删除成员变量

   ![image-20240429210701517](assets/image-20240429210701517.png)

2. 删除 checkIfSuppported()方法

   ![image-20240429210712513](assets/image-20240429210712513.png)

3. 修改 GET 方法

   ![image-20240429210724232](assets/image-20240429210724232.png)

   ![image-20240429210729447](assets/image-20240429210729447.png)

4. 修改 publishRules()方法：

   ![image-20240429210742401](assets/image-20240429210742401.png)

5. 修改 POST 方法：

   ![image-20240429210805071](assets/image-20240429210805071.png)

6. 修改 PUT 方法

   ![image-20240429210816461](assets/image-20240429210816461.png)

7. 修改 DELETE 方法

   ![image-20240429210827637](assets/image-20240429210827637.png)

### 修改页面

修改 src/main/webapp/resources/app/scripts/directives/sidebar/sidebar.html 中的代码：将 57 行中的 flowV1()修改为 flow()。

![image-20240429210849269](assets/image-20240429210849269.png)

### 重新打包 dashboard

对 Sentinel Dashboard 进行重新打包。

![image-20240429210902463](assets/image-20240429210902463.png)

![image-20240429210906728](assets/image-20240429210906728.png)

### 启动新的 dashboard

重新打包修改过的 sentinel-dashboard 模块，然后重新启动新的 dashboard 的 jar 包。

![image-20240429210918893](assets/image-20240429210918893.png)

### 运行应用

运行 06-consumer-persist-8080 工程应用。此时就可以实现 Sentinel Dashboard 与 Nacos 配置中心间修改的互通了。

