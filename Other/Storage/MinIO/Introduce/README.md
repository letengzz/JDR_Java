# MinIO 概述

MinIO 是 人工智能数据基础设施的对象存储，为人工智能系统提供数据支持，数据存储。

对象存储 (`Object Storage`)：是一种数据**存储架构**，它以**对象**为单位来处理、存储和检索数据，每个对象都包含了**数据本身**以及**元数据**

MinIO存储的元数据主要包括对象的描述信息，如用户 (account)、存储桶 (bucket)以及存储桶索引 (bucket index)等。

对象存储系统通常通过HTTP或HTTPS协议的API (应用程序编程接口)进行数据读写。

![](https://cdn.jsdelivr.net/gh/letengzz/tc2@main/img202405041215524.svg)

- 官方网站：https://min.io/


- 中文网站：http://minio.org.cn

- GitHub：https://github.com/minio

- Gitee：https://gitee.com/mirrors/minio

## 版权许可

MinIO具有双重许可：

- 开源GNU AGPL v3：完全免费
- 商业企业许可证：收费

兼容亚马逊S3云存储服务接口