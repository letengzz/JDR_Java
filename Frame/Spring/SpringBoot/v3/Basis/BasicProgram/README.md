# SpringBoot构建入门程序

**环境要求**：

- JDK：Java17
- Maven：3.6.8
- SpringBoot：3.0.9

**官方教程**：https://docs.spring.io/spring-boot/docs/3.0.9/reference/html/

## 1. 构建SpringBoot项目

**SpringBoot 项目结构**：

![image-20230722213944365](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202307222143259.png)

### 使用Maven父工程构建

**说明**：下列操作基于模块进行开发，仅创建SpringBoot项目时，将父工程\子模块中的依赖全部添加即可。

**构建basic_program_parent工程**：

![image-20230722231822302](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202307222318010.png)

**创建子模块`springboot_hello`**：

![image-20230722232002367](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202307222320525.png)

**父工程添加依赖**：

```xml
<!--    所有springboot项目都必须继承自 spring-boot-starter-parent -->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>3.0.9</version>
</parent>
```

**子工程添加依赖**：

```xml
<!-- 导入依赖：springboot通过自动版本仲裁 不需要关注版本问题 -->
<dependencies>
    <dependency>
    	<groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
	</dependency>
</dependencies>
```

**查看依赖**：

![image-20230722210039241](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202307222100838.png)

### 使用Maven依赖管理器构建

**说明**：下列操作仅创建SpringBoot项目，创建基于模块进行开发，可先创建空项目，再进行创建模块操作下列步骤。

**构建basic_program_dM项目**：

![image-20230722233424396](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202307222334335.png)

**添加依赖**：

```xml
<!-- 导入依赖：springboot通过自动版本仲裁 不需要关注版本问题 -->
<dependencies>
    <dependency>
    	<groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
	</dependency>
</dependencies>
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-dependencies</artifactId>
            <version>3.0.9</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

### 使用Spring  Initailizr构建

使用 Spring Initailizr 可以大量简化SpringBoot项目的开发。

**选择开发场景**：

- 官方脚手架：https://start.spring.io/
- 阿里脚手架：https://start.aliyun.com/

![image-20230808144710182](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202308081447047.png)

在网站中可构建项目来进行开发，也可在IDEA中进行开发。

**创建`springboot_initailizr`项目**：

![image-20230722234225212](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202307222342152.png)

**选择版本及所需依赖**：

![image-20230722233927410](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202307222339835.png)

**自动引入依赖**：

![image-20230722235517116](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202307222355431.png)

**自动创建项目结构**：

![image-20230722235620962](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202307222356930.png)

**项目启动器**：

```java
@SpringBootApplication
public class SpringbootInitailizrApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringbootInitailizrApplication.class, args);
    }

}
```

## 2. 创建主程序类

> com.hjc.demo.MainApplication

```java
/**
 * 主程序类
 * SpringBootApplication等同于EnableAutoConfiguration+SpringBootConfiguration+ComponentScan("com.hjc.demo")
 * scanBasePackages扫描指定包
 * 主程序所在包及其下面的所有子包里面的组件都会被默认扫描进来
 * @author hjc
 */
@SpringBootApplication(scanBasePackages = "com.hjc.demo")
public class MainApplication {

    public static void main(String[] args) {
        //返回IoC容器
        ConfigurableApplicationContext run = SpringApplication.run(MainApplication.class, args);
        //查看容器中的组件
        String[] names = run.getBeanDefinitionNames();
        for (String name :
                names) {
            System.out.println("name = " + name);
        }

    }
}
```

## 3. 创建业务层

> com.hjc.demo.controller.HelloController

```java
/**
 * RestController 就是Controller 和ResponseBody的合体
 * @author hjc
 */
@RestController
public class HelloController {

    @RequestMapping("/hello")
    public String handle01() {
        return "Hello, Spring Boot!";
    }
}
```

## 4. 测试运行

执行运行主程序类中的main方法。

访问http://localhost:8080/hello即可。

![image-20230722212035522](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202307222120147.png)

## 5. 配置参数

**配置文件名称必须为`application.properties`**：

- 集中式管理配置。只需要修改这个文件就行 
- 配置基本都有默认值

**更多参数**：https://docs.spring.io/spring-boot/docs/3.0.9/reference/html/application-properties.html#appendix.application-properties

> application.properties

```properties
# 修改服务器的端口号 默认为8080
server.port=8888
```

## 6. 部署运行

在pom.xml中添加插件：

```xml
<!-- 打包格式 -->
<packaging>jar</packaging>

<!-- 打包插件 -->
<build>
	<plugins>
    	<!-- 可以打成jar包：包括所有运行环境，可以直接运行jar包 -->
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <configuration>
                    <executable>true</executable>
                    <layout>JAR</layout>
            </configuration>
            <executions>
                <execution>
                    <goals>
                        <goal>repackage</goal>
                    </goals>
                    <configuration>
                        <attach>false</attach>
                    </configuration>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

![image-20230722212253815](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202307222122164.png)

把项目打成jar包，直接在目标服务器执行即可。

**项目打成可执行的jar包**：

```bash
mvn clean package
```

**运行**：

```bash
java -jar 包名.jar
```

