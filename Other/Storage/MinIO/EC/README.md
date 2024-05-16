# MinIO 纠删码

**纠删码** (`Erasure Code`，EC)是一种数据保护方法，也是一种算法。

MinIO对纠删码模式的算法进行了实现，采用Reed-Solomon code (简称RScode) 纠错码将对象拆分成N/2数据和N/2奇偶校验块，Reed Solomon利用范德蒙矩阵 (Vandermonde matrix)、柯西矩阵 (Cauchy matrix) 的特性来实现。

即将数据拆分为多个数据块和多个校验块，分散存储在不同的磁盘上，即使在部分磁盘损坏或丢失的情况下，也可以通过剩余的数据块和校验块恢复出原始数据。

举个例子，现在有12块磁盘，一个对象数据会被分成6个数据块、6个奇偶校验块，你可以损坏或丢失任意6块磁盘（不管其是存放的数据块还是奇偶校验块），仍可以从剩下的磁盘中恢复数据。

**纠删码模式相当于单机多磁盘的部署**。

## 启动纠删码模式

启动命令：

```shell
/usr/local/minio/minio server --console-address ":9001" /opt/minio/data/data{1...12}
```

![image-20240512212535303](https://cdn.jsdelivr.net/gh/letengzz/tc2@main/img202405122125293.png)

minio 集群部署被强制性安装在独占的磁盘分区，不能在 root 根盘符下建立目录，如在 /opt/data1 等等建目录文件夹代替，会抛出错误提示。因此可以在系统已挂载的磁盘下建 data 目录。

![image-20240512212423199](https://cdn.jsdelivr.net/gh/letengzz/tc2@main/img202405122124817.png)

**解决办法**：

1. 列出所有可用的块设备 (数据存储设备，如磁盘、闪存驱动器)的信息，如设备名称、大小、挂载点等

   ```shell
   lsblk
   ```

2. 添加一块磁盘，将添加的磁盘格式化为xfs格式：

   ```shell
   mkfs.xfs /dev/sdb
   ```

3. 将磁盘挂载到minio的存储目录：

   ```shell
   mount /dev/sdb /opt/minio/data
   ```

4. 启动纠删码模式：

   ```
   /usr/local/minio/minio server --console-address ":9001" /opt/minio/data/data{1...12}	
   ```

   ![image-20240512212328430](https://cdn.jsdelivr.net/gh/letengzz/tc2@main/img202405122123797.png)

## 启动纠删码模式(后台启动)

- nohup:  这是一个Unix命令，用于运行另一个命令在后台，并且忽略挂起（HUP）信号，也就是即使退出了终端或关闭了会话，该命令也会继续运行
- /opt/minio/data/minio.log:  这部分是将标准输出（stdout）重定向到 /opt/minio/data/minio.log 文件，这意味着 MinIO 服务器的所有正常输出（如启动信息、状态更新等）都会被写入到这个日志文件中；
- 2>&1:  这部分是将标准错误输出（stderr）重定向到标准输出（stdout），即输出到 /opt/minio/data/minio.log 文件，这样，无论是标准输出还是错误输出，都会被写入到同一个日志文件中
- &:  这个符号是在命令的末尾，用于将命令放到后台执行，也就是即使你启动了 MinIO 服务器，你的终端或 shell 会话也不会被阻塞，你可以继续执行其他命令；

```shell
MINIO_ROOT_USER=admin MINIO_ROOT_PASSWORD=password nohup /usr/local/minio/minio server --console-address ":9001" /opt/minio/data/data{1...12} > /opt/minio/data/minio.log 2>&1 &
```





