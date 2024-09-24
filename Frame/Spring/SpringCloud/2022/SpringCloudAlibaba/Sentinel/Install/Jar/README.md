# Jar包 安装部署

下载地址：https://github.com/alibaba/Sentinel/releases

**参考选择版本**：[https://github.com/alibaba/spring-cloud-alibaba/wiki/%E7%89%88%E6%9C%AC%E8%AF%B4%E6%98%8E](https://github.com/alibaba/spring-cloud-alibaba/wiki/版本说明)

![image-20230331135523246](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202304020131285.png)

下载`jar`文件(其实就是个SpringBoot项目)，需要在IDEA中添加一些运行配置：

![image-20240415200910917](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202404152009227.png)

Sentinel组件由两部分构成：后台默认端口号8719、前台默认端口号8080

https://github.com/alibaba/Sentinel/wiki/%E4%BB%8B%E7%BB%8D

![](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202404152029736.png)

直接启动，默认端口占用8080，如果需要修改，可以添加环境变量：

![image-20240415201134239](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202404152011009.png)

启动之后，就可以访问到Sentinel的监控页面了，用户名和密码都是`sentinel`，地址：http://localhost:8858/#/dashboard

![image-20240415201223817](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202404152012501.png)

![image-20240415203503717](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202404152035663.png)

这样就成功开启监控页面了。