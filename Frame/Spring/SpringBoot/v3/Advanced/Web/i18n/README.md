# 国际化

国际化也称作i18n，其来源是英文单词 internationalization的首末字符i和n，18为中间的字符数。由于软件发行可能面向多个国家，对于不同国家的用户，软件显示不同语言的过程就是国际化。通常来讲，软件中的国际化是通过配置文件来实现的，假设要支撑两种语言，那么就需要两个版本的配置文件。

国际化的自动配置参照`MessageSourceAutoConfiguration`

**实现步骤**：

1. Spring Boot 在类路径根下查找messages资源绑定文件。文件名为：`messages.properties`
2. 多语言可以定义多个消息文件，命名为`messages_区域代码.properties`。如：

3. 1. `messages.properties`：默认
   2. `messages_zh_CN.properties`：中文环境
   3. `messages_en_US.properties`：英语环境

4. 在**程序中**可以自动注入 `MessageSource`组件，获取国际化的配置项值
5. 在**页面中**可以使用表达式 ` #{}`获取国际化的配置项值

```java
@Autowired  
MessageSource messageSource;  //国际化取消息用的组件

@GetMapping("/haha")
public String haha(HttpServletRequest request){

	Locale locale = request.getLocale();
    //利用代码的方式获取国际化配置文件中指定的配置项的值
    String login = messageSource.getMessage("login", null, locale);
    return login;
}
```

