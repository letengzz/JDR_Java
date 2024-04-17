# 路径匹配

**Spring5.3** 之后加入了更多的请求路径匹配的实现策略：以前只支持 AntPathMatcher 策略, 现在提供了 **PathPatternParser** 策略。并且可以指定到底使用那种策略。

## Ant风格路径用法

Ant 风格的路径模式**语法规则**：

- `*`：表示**任意数量**的字符。
- `?`：表示任意**一个字符**。
- `**`：表示 **任意数量的目录**。
- `{}`：表示一个命名的模式**占位符**。
- `[]`：表示**字符集合**，例如[a-z]表示小写字母。

例如：

- `*.html`：匹配任意名称，扩展名为.html的文件。
- `/folder1/*/*.java`：匹配在folder1目录下的任意两级目录下的.java文件。
- `/folder2/**/*.jsp`：匹配在folder2目录下任意目录深度的.jsp文件。
- `/{type}/{id}.html`：匹配任意文件名为{id}.html，在任意命名的{type}目录下的文件。

**注意**：Ant 风格的路径模式语法中的**特殊字符需要转义**：

- 要匹配文件路径中的星号，则需要转义为`\\*`
- 要匹配文件路径中的问号，则需要转义为`\\?`

## 模式切换

**修改路径匹配策略**：

```properties
# 改变路径匹配策略：
# ant_path_matcher 老版策略；
# path_pattern_parser 新版策略；
spring.mvc.pathmatch.matching-strategy=ant_path_matcher
```

## AntPathMatcher 与 PathPatternParser

- PathPatternParser 在 jmh 基准测试下，有 6~8 倍吞吐量提升，降低 30%~40%空间分配率
- PathPatternParser 兼容 AntPathMatcher语法，并支持更多类型的路径模式
- PathPatternParser  "`**`" **多段匹配**的支持**仅允许在模式末尾使用**

```java
/**
* 默认使用 PathPatternParser 进行路径匹配
* 不能匹配 **在中间的情况，剩下的和 AntPathMatcher 语法兼容
*/
@GetMapping("/a*/b?/{p1:[a-f]+}")
public String hello(HttpServletRequest request, 
                        @PathVariable("p1") String path) {

	log.info("路径变量p1： {}", path);
    //获取请求路径
    String uri = request.getRequestURI();
    return uri;
}
```

![image-20230804215103710](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202308042151831.png)

**注意**：

- 当使用`**`不在末尾时，使用PathPatternParser模式会提示使用AntPathMatcher，所以如果路径中间需要有 `**`，替换成ant风格路径

  ![image-20230804215403804](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202308042154951.png)

  ```properties
  spring.mvc.pathmatch.matching-strategy=ant_path_matcher
  ```

  ```yaml
  # 改变路径匹配策略：ant_path_matcher 老版策略；path_pattern_parser 新版策略
  spring:
    mvc:
      pathmatch:
        matching-strategy: ant_path_matcher
  ```

  ![image-20230804220011048](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202308042200968.png)

- 使用默认的路径匹配规则，是由 PathPatternParser  提供的

  ![image-20230804222136420](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202308042221159.png)

  ![image-20230804222217814](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202308042222032.png)

