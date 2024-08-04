# 接口文档

## OpenAPI 3 与 Swagger 

Swagger 可以快速生成实时接口文档，方便前后开发人员进行协调沟通。遵循 OpenAPI 规范。

文档：https://springdoc.org/v2/

## OpenAPI 3 架构 

![image.png](https://cdn.nlark.com/yuque/0/2023/png/1613913/1681977849796-4b79ef53-8969-488d-aaa8-73fc333e8be7.png)

## SpringBoot 整合 

导入依赖：

```xml
<dependency>
    <groupId>org.springdoc</groupId>
    <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
    <version>2.3.0</version>
</dependency>
```

配置：

```properties
# /api-docs endpoint custom path 默认 /v3/api-docs
springdoc.api-docs.path=/api-docs

# swagger 相关配置在  springdoc.swagger-ui
# swagger-ui custom path
springdoc.swagger-ui.path=/swagger-ui.html

springdoc.show-actuator=true
```

### 常用注解 

| 注解         | 标注位置            | 作用                   |
| ------------ | ------------------- | ---------------------- |
| @Tag         | controller 类       | 标识 controller 作用   |
| @Parameter   | 参数                | 标识参数作用           |
| @Parameters  | 参数                | 参数多重说明           |
| @Schema      | model 层的 JavaBean | 描述模型作用及每个属性 |
| @Operation   | 方法                | 描述方法作用           |
| @ApiResponse | 方法                | 描述响应状态码等       |

### Docket配置 

如果有多个Docket配置：

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

如果只有一个Docket，可以配置如下：

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

| 原注解                                      | 现注解                                                       | 作用           |
| ------------------------------------------- | ------------------------------------------------------------ | -------------- |
| @Api                                        | @Tag                                                         | 描述Controller |
| @ApiIgnore                                  | @Parameter(hidden = true)  @Operation(hidden = true) @Hidden | 描述忽略操作   |
| @ApiImplicitParam                           | @Parameter                                                   | 描述参数       |
| @ApiImplicitParams                          | @Parameters                                                  | 描述参数       |
| @ApiModel                                   | @Schema                                                      | 描述对象       |
| @ApiModelProperty(hidden = true)            | @Schema(accessMode = READ_ONLY)                              | 描述对象属性   |
| @ApiModelProperty                           | @Schema                                                      | 描述对象属性   |
| @ApiOperation(value = "foo", notes = "bar") | @Operation(summary = "foo", description = "bar")             | 描述方法       |
| @ApiParam                                   | @Parameter                                                   | 描述参数       |
| @ApiResponse(code = 404, message = "foo")   | @ApiResponse(responseCode = "404", description = "foo")      | 描述响应       |

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

#### 添加OpenAPI组件 

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

# SpringBoot 整合 Swagger

Swaager的常用注解如下：

在Java类中添加Swagger的注解即可生成Swagger接口，常用Swagger注解如下：

-  @Api：修饰整个类，描述Controller的作用  
-  @ApiOperation：描述一个类的一个方法，或者说一个接口  
-  @ApiParam：单个参数描述  
-  @ApiModel：用对象来接收参数 
-  @ApiModelProperty：用对象接收参数时，描述对象的一个字段  
-  @ApiResponse：HTTP响应其中1个描述  
-  @ApiResponses：HTTP响应整体描述  
-  @ApiIgnore：使用该注解忽略这个API  
-  @ApiError ：发生错误返回的信息  
-  @ApiImplicitParam：一个请求参数  
-  @ApiImplicitParams：多个请求参数

@ApiImplicitParam属性如下：

| 属性         | 取值   | 作用                                          |
| ------------ | ------ | --------------------------------------------- |
| paramType    |        | 查询参数类型                                  |
|              | path   | 以地址的形式提交数据                          |
|              | query  | 直接跟参数完成自动映射赋值                    |
|              | body   | 以流的形式提交 仅支持POST                     |
|              | header | 参数在request headers 里边提交                |
|              | form   | 以form表单的形式提交 仅支持POST               |
| dataType     |        | 参数的数据类型 只作为标志说明，并没有实际验证 |
|              | Long   |                                               |
|              | String |                                               |
| name         |        | 接收参数名                                    |
| value        |        | 接收参数的意义描述                            |
| required     |        | 参数是否必填                                  |
|              | true   | 必填                                          |
|              | false  | 非必填                                        |
| defaultValue |        | 默认值                                        |

使用Swagger可以进行接口的测试。