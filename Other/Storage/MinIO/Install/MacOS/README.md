# MacOS 安装 MinIO

**说明**：基于开源协议下载安装，单机版可能会有数据丢失的情况。

## 下载MinIO

命令：

```shell
curl --progress-bar -O https://dl.min.io/server/minio/release/darwin-amd64/minio
```

## 赋予权限

命令：

```
chmod +x minio
```

## 启动MinIO

前台启动：

```shell
MINIO_ROOT_USER=admin MINIO_ROOT_PASSWORD=password ./minio server /mnt/data --console-address ":9001"
```

后台启动 (`&`结束)：

```shell
MINIO_ROOT_USER=admin MINIO_ROOT_PASSWORD=password ./minio server /mnt/data --console-address ":9001" &
```

- `MINIO_ROOT_USER`：指定minio的用户名
- `MINIO_ROOT_PASSWORD`：指定minio的密码
- `/mnt/data`：指定minio服务器用于存储数据的目录
- `--console-address ":9001"`：指定minio控制台的监听地址和端口

![image-20240504141023117](https://cdn.jsdelivr.net/gh/letengzz/tc2@main/img202405041410457.png)

当不设置用户名密码 有默认的账号和密码：

```shell
./minio server /mnt/data --console-address ":9001"
```

![image-20240504142058276](https://cdn.jsdelivr.net/gh/letengzz/tc2@main/img202405041421096.png)

访问：http://localhost:9001/

![image-20240504141924742](https://cdn.jsdelivr.net/gh/letengzz/tc2@main/img202405041419090.png)

## 关闭MinIO

关闭前台启动：Control + C

关闭后台启动：

```shell
ps -ef | grep minio
kill pid
```



