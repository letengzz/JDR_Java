# MinIO 常见错误

当出现`The difference between the request time and the server‘s time is too large`

**原因**：客户端和服务端时间不同步导致的

**解决办法**：linux同步网络时间

```shell
yum install ntpdate -y
ntpdate pool.ntp.org
date
```

