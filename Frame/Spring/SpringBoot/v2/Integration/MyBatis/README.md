# SpringBoot 整合 MyBatis

- [MyBatis](../../../../../MyBatis/README.md)

- 整合mybatis的starter官网地址：https://github.com/mybatis/spring-boot-starter

## 自动配置原理

mybatis-spring-boot-starter导入 spring-boot-starter-jdbc，jdbc是操作数据库的场景

Jdbc场景的自动配置：

- `org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration`
  - 数据源的自动配置
  - 所有和数据源有关的配置都绑定在DataSourceProperties
  - 默认使用 HikariDataSource
- `org.springframework.boot.autoconfigure.jdbc.JdbcTemplateAutoConfiguration`
  - 给容器中放了JdbcTemplate操作数据库
- `org.springframework.boot.autoconfigure.jdbc.JndiDataSourceAutoConfiguration`
- `org.springframework.boot.autoconfigure.jdbc.XADataSourceAutoConfiguration`
  - 基于XA二阶提交协议的分布式事务数据源
- `org.springframework.boot.autoconfigure.jdbc.DataSourceTransactionManagerAutoConfiguration`
  - 支持事务
- 具有的底层能力：数据源、JdbcTemplate、事务

MyBatisAutoConfiguration：配置了MyBatis的整合流程

mybatis-spring-boot-starter导入 mybatis-spring-boot-autoconfigure（mybatis的自动配置包）默认加载两个自动配置类：

- `org.mybatis.spring.boot.autoconfigure.MybatisLanguageDriverAutoConfiguration`
- `org.mybatis.spring.boot.autoconfigure.MybatisAutoConfiguration`
  - 必须在数据源配置好之后才配置
  - 给容器中SqlSessionFactory组件。创建和数据库的一次会话
  - 给容器中SqlSessionTemplate组件。操作数据库

MyBatis的所有配置绑定在MybatisProperties

每个Mapper接口的代理对象是怎么创建放到容器中。详见@MapperScan原理

利用@Import(MapperScannerRegistrar.class)批量给容器中注册组件。解析指定的包路径里面的每一个类，为每一个Mapper接口类，创建Bean定义信息，注册到容器中。

## 操作步骤

可以直接从spring initializr中选择整合MyBatis：

![image-20231224205619772](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202312242056982.png)

也可导入依赖：

```xml
<!-- SpringBoot整合MyBatis -->
<dependency>
	<groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>3.0.1</version>
</dependency>
```

## 配置模式

- 导入mybatis官方starter
- 编写mapper接口。标准`@Mapper`注解
- 编写sql映射文件并绑定mapper接口
- 在application.yaml中指定Mapper配置文件的位置，以及指定全局配置文件的信息 （建议：**配置在mybatis.configuration**）

**配置参数**：

> application.yaml

```yaml
# 配置数据源
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/springboot
    driver-class-name: com.mysql.cj.jdbc.Driver
    username: root
    password: 123123
    type: com.zaxxer.hikari.HikariDataSource
# 配置mybatis规则、使⽤MyBatisPlus则此项配置⽆效
mybatis:
  # config-location: classpath:mybatis/mybatis-config.xml #全局配置文件位置
  mapper-locations: classpath:mappers/*.xml #sql映射文件位置
  configuration: # 指定mybatis全局配置⽂件中的相关配置项
    map-underscore-to-camel-case: true
  type-aliases-package: com.hjc.demo.mybatis.pojo
```

### 注解模式

```java
@Mapper
public interface CityMapper {

    @Select("select * from city where id=#{id}")
    public City getById(Long id);

    public void insert(City city);

}
```

### 混合模式

- 引入mybatis-starter
- **配置application.yaml中，指定mapper-location位置即可**
- 编写Mapper接口并标注@Mapper注解
- 简单方法直接注解方式
- 复杂方法编写mapper.xml进行绑定映射
- `@MapperScan("com.hjc.mapper")` 简化，其他的接口就可以不用标注`@Mapper`注解

```java
@Mapper
public interface CityMapper {

    @Select("select * from city where id=#{id}")
    public City getById(Long id);

    public void insert(City city);

}
```

