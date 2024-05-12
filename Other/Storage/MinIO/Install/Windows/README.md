# Windows 安装 MinIO

**说明**：基于开源协议下载安装，单机版可能会有数据丢失的情况。

## 下载MinIO

复制命令到浏览器下载：

```shell
https://dl.min.io/server/minio/release/windows-amd64/minio.exe
```

## 启动MinIO

使用管理员进入到minio.exe所在的目录，执行 `minio.exe server D:\Data\minio`，其中D:\Data\minio为MinIO存储数据的目录路径

```shell
setx MINIO_ROOT_USER admin 
setx MINIO_ROOT_PASSWORD password 
minio.exe server D:\Data\minio --console-address ":9001"
```

访问：http://localhost:9000/

默认账号密码都是 `minioadmin`

![image-20240504141924742](https://cdn.jsdelivr.net/gh/letengzz/tc2@main/img202405041419090.png)

## 关闭MinIO

关闭启动：CTRL + C
