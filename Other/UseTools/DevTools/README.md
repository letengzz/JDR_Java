# Dev-Tools

Dev-Tools只要类路径上的文件发⽣更改，使用的应用程序就会自动重新启动。

在 IDE中工作时，这可能是⼀个有用的功能，因为它为代码更改提供了非常快速的反馈循环。默认情况下，将监视类路径上指向目录的任何条目的更改。

**注意**：某些资源(例如静态资产和视图模板)不需要重新启动应用程序。

 **引入依赖**： 

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-devtools</artifactId>
	<optional>true</optional>
</dependency>
```

类路径文件发⽣更改，`CTRL+F9`重新构件，dev-tools自动重启项目

Java代码的修改，如果`devtools`热启动了，可能会引起一些bug，难以排查