# RabbitMQ 安装

**下载地址：** https://www.rabbitmq.com/download.html

由于`RabbitMQ`是基于`Erlang`（面向高并发的语言）语言开发，所以在安装`RabbitMQ`之前，需要先安装`Erlang`：

- Ubuntu/Debian：

  ```sh
  sudo apt install erlang
  ```

- Centos：`Erlang`在默认的`YUM`存储库中不可用，因此需要安装`EPEL`存储库：

  ```sh
  wget --content-disposition https://packagecloud.io/rabbitmq/erlang/packages/el/7/erlang-22.3.4.12-1.el7.x86_64.rpm/download.rpm
  
  yum localinstall erlang-22.3.4.12-1.el7.x86_64.rpm
  ```

检查`Erlang`版本。

```sh
erl -version
```

接着安装RabbitMQ：

- Ubuntu/Debian：

  ```sh
  sudo apt install rabbitmq-server
  ```

- Centos：

  ```sh
  wget --content-disposition https://packagecloud.io/rabbitmq/rabbitmq-server/packages/el/7/rabbitmq-server-3.8.13-1.el7.noarch.rpm/download.rpm
  
  rpm --import https://www.rabbitmq.com/rabbitmq-release-signing-key.asc
  
  yum localinstall rabbitmq-server-3.8.13-1.el7.noarch.rpm
  ```

RabbitMQ **常见命令**：

```sh
systemctl start rabbitmq-server  #启动
systemctl enable rabbitmq-server #设置开机自启
systemctl status rabbitmq-server #查看状态
```

安装完成后，可以输入`sudo rabbitmqctl status`来查看当前的RabbitMQ运行状态，包括运行环境、内存占用、日志文件等信息：

```sh
sudo rabbitmqctl status
```

![image-20240310232416553](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202403102324977.png)

这样RabbitMQ服务器就安装完成。

可以看到默认有两个端口名被使用：使用amqp协议的那个端口`5672`来进行连接，25672是集群化端口。

![image-20240310232739381](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202403102339301.png)

可以将RabbitMQ的管理面板开启，这样话就可以在浏览器上进行实时访问和监控了：

```sh
sudo rabbitmq-plugins enable rabbitmq_management
```

由于Web管理界面访问端口为15672，所以防火墙需要放行该端口 15672 (默认 RabbitMQ 管理界面的端口)：

1. 登录到服务器上，以具有管理员权限的用户身份。

2. 检查防火墙状态，确认是否已安装 firewalld 防火墙：

   ```sh
   systemctl status firewalld
   ```

3. 如果防火墙处于开启状态，可以直接跳转到第 6 步。如果防火墙停止运行，则需要启动，请继续执行以下步骤。

4. 启动 firewalld 服务：

   ```sh
   systemctl start firewalld
   ```

5. 设置 firewalld 开机自启：

   ```sh
   systemctl enable firewalld
   1
   ```

6. 添加端口规则，允许在防火墙上开放 15672 端口和5672端口：

   ```sh
   firewall-cmd --zone=public --add-port=15672/tcp --permanent
   firewall-cmd --zone=public --add-port=5672/tcp --permanent
   ```

7. 重新加载防火墙配置，使更改生效：

   ```sh
   firewall-cmd --reload
   ```

防火墙应该已经放行了 15672 端口和5672 端口，允许对 RabbitMQ 管理界面进行访问。**注意**：为了安全起见，建议仅在需要时才开放必要的端口，并在完成使用后关闭不必要的端口。

使用`sudo rabbitmqctl status`查看状态，可以看到多了一个管理面板，使用的是HTTP协议：

![image-20240310232918575](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202403102339354.png)

直接访问：

![image-20240310233004614](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202403102339439.png)

RabbitMQ 默认的管理界面账号和密码：用户名：`guest`、密码：`guest`

这对默认凭据在 RabbitMQ 安装后可用于访问管理界面 (**只限于本地**)。然而，出于安全考虑，强烈建议在生产环境中修改默认凭据或创建新的管理员帐户，并使用更强大的密码来加强安全性。

在 RabbitMQ 中添加新用户：您需要使用 RabbitMQ 提供的命令行工具或者管理界面进行操作

- 使用 RabbitMQ 命令行工具：

  - 创建`admin`用户：

    ```sh
    sudo rabbitmqctl add_user admin admin
    ```


  - 将管理员权限给予`admin`用户：

    ```sh
    sudo rabbitmqctl set_user_tags admin administrator
    ```

- 使用 RabbitMQ 管理界面 (只能搭建在本地操作)：

  - 打开您的浏览器并访问 RabbitMQ 管理界面。默认地址为 `http://localhost:15672`

  - 使用默认的管理员账号和密码（通常是 `guest`/`guest`）登录到管理界面

  - 在管理界面上导航到 "Admin" -> "Users" 选项卡

  - 单击 "Add a user" 按钮

  - 输入用户名和密码，并选择 "Tag" 为 "Administrator"

  - 单击 "Add user" 按钮以创建新用户。

  - 注意，此处需要进行一次授权，否则在代码中连接RabbitMQ会失败

    ![image-20230616194600354](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202403110022613.png)

    ![image-20230616194636865](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202403110022681.png)

创建完成之后，登录一下页面：

![image-20240310233156858](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202403102332176.png)

进入了之后会显示当前的消息队列情况，包括版本号、Erlang版本等。