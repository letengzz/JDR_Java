# Docker 安装 MinIO

**说明**：基于开源协议下载安装，单机版可能会有数据丢失的情况。

搜索MinIO镜像：

```dockerfile
docker search minio
```

拉取MinIO镜像：

```dockerfile
docker pull minio/minio
```

启动MinIO容器：

```dockerfile
docker run -p 9000:9000 -p 9001:9001 minio/minio server /data --console-address ":9001"
```

查看已安装镜像：

```dockerfile
docker images
```

删除镜像：

```dockerfile
docker rmi minio/minio
```

