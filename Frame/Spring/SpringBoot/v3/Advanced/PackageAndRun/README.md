# SpringBoot 项目打包和运行

## 添加打包插件

在Spring Boot项目中添加`spring-boot-maven-plugin`插件是为了支持将项目打包成可执行的可运行jar包。如果不添加`spring-boot-maven-plugin`插件配置，使用常规的`java -jar`命令来运行打包后的Spring Boot项目是无法找到应用程序的入口点，因此导致无法运行。

```xml
<!--    SpringBoot应用打包插件-->
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <configuration>
                <excludes>
                    <exclude>
                        <groupId>org.projectlombok</groupId>
                        <artifactId>lombok</artifactId>
                    </exclude>
                </excludes>
            </configuration>
        </plugin>
    </plugins>
</build>
```

或使用

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <version>3.8.1</version>
            <configuration>
                <source>17</source>
                <target>17</target>
                <encoding>UTF-8</encoding>
            </configuration>
        </plugin>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <version>${spring-boot.version}</version>
            <configuration>
                <mainClass>com.hjc.demo.SpringbootActuatorApplication</mainClass>
                <!-- 
                issue：SpringBoot-0.0.1-SNAPSHOT.jar 中没有主清单属性

                resolve：① 注释掉，或者 ② 将 skip 值改为 false 
                -->
                <!--<skip>true</skip>-->
                <excludes>
                    <exclude>
                        <groupId>org.projectlombok</groupId>
                        <artifactId>lombok</artifactId>
                    </exclude>
                </excludes>
            </configuration>
            <executions>
                <execution>
                    <id>repackage</id>
                    <goals>
                        <goal>repackage</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

## 执行打包

在IDEA点击package或使用 `mvn package`命令进行打包。

![image-20231216230527114](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202312162305163.png)

可以在编译的target文件中查看jar包：

![image-20231216231634785](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202312162317560.png)

## 命令启动和参数说明

`java -jar`命令用于在Java环境中执行可执行的JAR文件。

**命令格式**：

```xml
java -jar  [选项] [参数] <jar文件名>
```

**说明**：

1.  `-D<name>=<value>`：设置系统属性，可以通过`System.getProperty()`方法在应用程序中获取该属性值。例如：`java -jar -Dserver.port=8080 myapp.jar`。
2.  `-X`：设置JVM参数，例如内存大小、垃圾回收策略等。常用的选项包括：
    -   `-Xmx<size>`：设置JVM的最大堆内存大小，例如 `-Xmx512m` 表示设置最大堆内存为512MB。
    -   `-Xms<size>`：设置JVM的初始堆内存大小，例如 `-Xms256m` 表示设置初始堆内存为256MB。
3.  `-Dspring.profiles.active=<profile>`：指定Spring Boot的激活配置文件，可以通过`application-<profile>.properties`或`application-<profile>.yml`文件来加载相应的配置。例如：`java -jar -Dspring.profiles.active=dev myapp.jar`。

启动和测试：

![image-20231217010256758](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202312170102831.png)

**注意**： -D 参数必须要在jar之前！否者不生效！

## 后台启动java应用

```shell
nohup java -jar <jar文件名> > output.log 2>&1 &
```

**说明**：

- `output.log`：日志输出文件，指定为标准输出(stdout)并重定向到一个名为output.log的文件中。
- `2>&1`：将标准错误输出(stderr)重定向到标准输出(stdout)
- `&`：在后台运行进程

## 部署Jar包瘦瘦身

分析 jar，我们可以看出，jar 包里面分为以下三个模块

![图片](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202403291834471.png)

分为 `BOOT-INF`，`META-INF`，org 三个部分，打开 `BOOT-INF`

![图片](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202403291834442.webp)

可以看到有 classes，lib 两个文件夹，我们编译好的代码是放在 classes 里面的，而我们所依赖的 jar 包都是放在 lib 文件夹下

classes 部分是非常小的，lib部分是非常大的，所以上传很慢

**可以将自己写的代码部分与所依赖的 maven jar 包部分拆开上传，每次只需要上传自己写的代码部分即可**

## 瘦身部署

### 正常打包

项目的 pom.xml 文件中的打包方式：

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugins>
</build>
```

这是 SpringBoot 中默认的打包方式，先按照这种方式打包出来，得到一个 jar 包，我们将 jar 包解压，如果不能直接解压，则将后缀改为 zip 再进行解压，只需要拿到 `BOOT-INF` 中的 lib 目录即可

### 改变打包方式

对 SpringBoot 中默认的打包方式做一些配置：

- `mainClass`：指定了项目的启动类
- `layout`：指定了打包方式为 ZIP，**注意**：一定是大写的
- `includes`：有自己的依赖 jar，可以在此导入
- `repackage`：剔除其它的依赖，只需要保留最简单的结构

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <configuration>
                <mainClass>com.hjc.DeclareApplication</mainClass>
                <layout>ZIP</layout>
                <includes>
                    <include>
                        <groupId>nothing</groupId>
                        <artifactId>nothing</artifactId>
                    </include>
                </includes>
            </configuration>
            <executions>
                <execution>
                    <goals>
                        <goal>repackage</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

### 再次打包

再次点击` maven package`，得到一个 jar 包，可以看到此时的 jar 包只有几兆了

![image-20240329183319446](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202403291833159.png)

### 上传启动

将 lib 目录，以及最后打包的瘦身项目 jar 包，上传至服务器，目录如下

![图片](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202403291833893.png)

使用命令：

- `-Dloader.path`：告诉它所依赖的 maven jar 包位置
- `sbm-0.0.1-SNAPSHOT.jar`：项目 jar 包的名字
- `nohup`、`&`：使得 jar 包在服务后台运行

```sh
nohup java -Dloader.path=./lib -jar ./sbm-0.0.1-SNAPSHOT.jar &
```