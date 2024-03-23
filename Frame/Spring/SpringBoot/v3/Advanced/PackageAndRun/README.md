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