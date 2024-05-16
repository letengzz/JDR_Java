# MinIO 集群部署

纠删码模式是单机多磁盘方式，如果这台机器宕机了，那么minio就不能对外提供服务了

MinIO集群搭建步骤 (多节点多磁盘)：

1. 准备4台机器 (根据MinIO的架构设计，至少需要4个节点来构建集群，这是因为在一个N节点的分布式MinIO集群中，只要有N/2节点在线，数据就是安全的，同时，为了确保能够创建新的对象，需要至少有N/2+1个节点，因此，对于一个4节点的集群，即使有两个节点宕机，集群仍然是可读的，但需要有3个节点才能写数据)

2. 每台机器添加一块磁盘 (minio集群需要独占磁盘块，不能使用Linux的root磁盘块)

3. 将添加的磁盘格式化为xfs格式：

   ```shell
   mkfs.xfs /dev/sdb
   ```

4. 将磁盘挂载到minio的存储目录：

   ```shell
   mount /dev/sdb /opt/minio/data
   ```

5. 每台机器上安装好minio (这里是安装在 /usr/local/minio 目录下，版本统一)

6. 将防火墙关闭：

   ```shell
   systemctl stop firewalld
   ```

7. 一键部署MinIO分布式集群：

   ```
   vi start.sh
   chmod 744 start.sh
   mkdir data1 data2 data3 data4
   ```

   ```shell
   #!/bin/bash
   export MINIO_ROOT_USER=minioadmin
   export MINIO_ROOT_PASSWORD=minioadmin
   /usr/local/minio/minio server --config-dir /etc/minio --address :9000 --console-address :9001 \
   http://192.168.11.128/opt/minio/data/data1 http://192.168.11.128/opt/minio/data/data2 \
   http://192.168.11.128/opt/minio/data/data3 http://192.168.11.128/opt/minio/data/data4 \
   http://192.168.11.129/opt/minio/data/data1 http://192.168.11.129/opt/minio/data/data2 \
   http://192.168.11.129/opt/minio/data/data3 http://192.168.11.129/opt/minio/data/data4 \
   http://192.168.11.130/opt/minio/data/data1 http://192.168.11.130/opt/minio/data/data2 \
   http://192.168.11.130/opt/minio/data/data3 http://192.168.11.130/opt/minio/data/data4 \
   http://192.168.11.131/opt/minio/data/data1 http://192.168.11.131/opt/minio/data/data2 \
   http://192.168.11.131/opt/minio/data/data3 http://192.168.11.131/opt/minio/data/data4 &
   ```

   ```shell
   start.sh
   ```

8. 使用访问负载均衡来做转发：

   ![image-20240512221301406](https://cdn.jsdelivr.net/gh/letengzz/tc2@main/img202405122232752.png)

   使用Nginx来作为负载均衡：在配置文件中配置

   - 在http模块配置upstream：

     > API端口

     ```nginx
     upstream api { 
          	server  192.168.11.128:9000; 
           server  192.168.11.129:9000;  
          	server  192.168.11.130:9000;  
           server  192.168.11.131:9000;  
     } 
     ```

     > 管理后台端口

     ```nginx
     upstream webui { 
          	server  192.168.11.128:9001; 
           server  192.168.11.129:9001;  
          	server  192.168.11.130:9001;  
           server  192.168.11.131:9001;  
     } 
     ```

   - 在server模块里配置：

     ```nginx
     location / {
     	proxy_set_header Host $http_host;
       proxy_set_header Server MinIO;
       proxy_set_header Accept-Ranges bytes;
     	proxy_pass http://api;
     }
     ```

   - 配置Webui：

     ```nginx
     server {
       listen 50000;
       server_name localhost;
       
       location / {
     		proxy_pass http://webui;
     	}
     }
     ```

     

   



