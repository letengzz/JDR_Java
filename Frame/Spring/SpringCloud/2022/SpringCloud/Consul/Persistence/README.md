# Consul 持久化

## Windows

在安装路径下创建mydata目录：

![image-20240306230934670](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202403062309415.png)

在安装路径下，创建`consul_start.bat`

```bash
@echo.服务启动......  
@echo off  
@sc create Consul binpath= "D:\consul_1.18.0_windows_386\consul.exe agent -server -ui -bind=127.0.0.1 -client=0.0.0.0 -bootstrap-expect  1  -data-dir D:\consul_1.18.0_windows_386\mydata   "
@net start Consul
@sc config Consul start= AUTO  
@echo.Consul start is OK......success
@pause
```

右键管理员权限打开：

![image-20240306233120952](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202403062331995.png)