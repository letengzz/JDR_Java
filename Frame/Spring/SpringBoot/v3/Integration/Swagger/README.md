# SpringBoot 整合 Swagger

## OpenAPI 3 与 Swagger

**OpenAPI规范** (OpenAPI Specification 简称OAS) 是Linux基金会的一个项目，试图通过定义一种用来描述API格式或API定义的语言，来规范RESTful服务开发过程，目前版本是V3.0，并且已经发布并开源在github上。

**官方网站**：https://github.com/OAI/OpenAPI-Specification

****

Swagger是全球最大的OpenAPI规范 (OAS) API开发工具框架，遵循 OpenAPI 规范。

Swagger 可以快速**生成实时接口文档**，方便前后端开发人员进行协调沟通，依据接口文档进行开发。。

**官方网站**： https://swagger.io/
**官方文档**：https://springdoc.org/

## OpenAPI 3 架构

![image.png](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202312240109155.png)

## 整合 Swagger

Spring Boot 可以集成Swagger，Swaager根据Controller类中的注解生成接口文档 。

导入依赖：

```xml
<dependency>
    <groupId>org.springdoc</groupId>
    <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
    <version>2.3.0</version>
</dependency>
```

配置：

> application.properties

```properties
# /api-docs endpoint custom path 默认 /v3/api-docs
springdoc.api-docs.path=/api-docs

# swagger 相关配置在  springdoc.swagger-ui
# swagger-ui custom path
springdoc.swagger-ui.path=/swagger-ui.html

springdoc.show-actuator=true
```

> application.yaml

```yaml
springdoc:
  # /api-docs endpoint custom path 默认 /v3/api-docs
  api-docs.path: /api-docs
  # swagger 相关配置在  springdoc.swagger-ui
  # swagger-ui custom path
  swagger-ui.path: /swagger-ui.html
  show-actuator: true
```

## 使用 

### 常用注解

在Java类中添加Swagger的注解即可生成Swagger接口，常用Swagger注解：

![image-20231224005519126](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202312240055116.png)

### Docket 配置 

如果有多个Docket，配置：

```java
  @Bean
  public GroupedOpenApi publicApi() {
      return GroupedOpenApi.builder()
              .group("springshop-public")
              .pathsToMatch("/public/**")
              .build();
  }
  @Bean
  public GroupedOpenApi adminApi() {
      return GroupedOpenApi.builder()
              .group("springshop-admin")
              .pathsToMatch("/admin/**")
              .addMethodFilter(method -> method.isAnnotationPresent(Admin.class))
              .build();
  }
```

如果只有一个Docket，可以配置：

```properties
springdoc.packagesToScan=package1, package2
springdoc.pathsToMatch=/v1, /api/balance/**
```

### OpenAPI配置 

```java
@Bean
public OpenAPI springShopOpenAPI() {
	return new OpenAPI()
    	      .info(new Info().title("SpringShop API")
              .description("Spring shop sample application")
              .version("v0.0.1")
              .license(new License().name("Apache 2.0").url("http://springdoc.org")))
              .externalDocs(new ExternalDocumentation()
              .description("SpringShop Wiki Documentation")
              .url("https://springshop.wiki.github.org/docs"));
}
```

## Springfox 迁移 

### 注解变化 

![image-20231224010724087](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202312240107498.png)

### Docket配置 

#### 以前写法 

```java
@Bean
public Docket publicApi() {
    return new Docket(DocumentationType.SWAGGER_2)
              .select()
              .apis(RequestHandlerSelectors.basePackage("org.github.springshop.web.public"))
              .paths(PathSelectors.regex("/public.*"))
              .build()
              .groupName("springshop-public")
              .apiInfo(apiInfo());
}

@Bean
public Docket adminApi() {
    return new Docket(DocumentationType.SWAGGER_2)
              .select()
              .apis(RequestHandlerSelectors.basePackage("org.github.springshop.web.admin"))
              .paths(PathSelectors.regex("/admin.*"))
              .apis(RequestHandlerSelectors.withMethodAnnotation(Admin.class))
              .build()
              .groupName("springshop-admin")
              .apiInfo(apiInfo());
}
```

#### 新写法

```java
@Bean
public GroupedOpenApi publicApi() {
    return GroupedOpenApi.builder()
              .group("springshop-public")
              .pathsToMatch("/public/**")
              .build();
}
@Bean
public GroupedOpenApi adminApi() {
    return GroupedOpenApi.builder()
              .group("springshop-admin")
              .pathsToMatch("/admin/**")
              .addOpenApiMethodFilter(method -> method.isAnnotationPresent(Admin.class))
              .build();
}
```

### 添加OpenAPI组件 

```java
@Bean
public OpenAPI springShopOpenAPI() {
    return new OpenAPI()
              .info(new Info().title("SpringShop API")
              .description("Spring shop sample application")
              .version("v0.0.1")
              .license(new License().name("Apache 2.0").url("http://springdoc.org")))
              .externalDocs(new ExternalDocumentation()
              .description("SpringShop Wiki Documentation")
              .url("https://springshop.wiki.github.org/docs"));
}
```


