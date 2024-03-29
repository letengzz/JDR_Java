# 标准方言

Thymeleaf是非常非常可扩展的，它允许自定义的名字来定义一组模板属性(或者甚至是标签)，用自定语法评估计算表达式和应用逻辑。它更像是一个模板引擎框架。

它还带有一些称为标准方言(称为*Standard*和*SpringStandard*)的东西，它们定义了一组功能，这些功能应该足以满足大多数情况。可以识别这些标准方言在模板中的使用，因为它将包含以`th`前缀开头的属性，如`<span th:text="...">`。

请注意，*Standard*和*SpringStandard*方言几乎完全相同，只是*SpringStandard*包含了集成到Spring MVC应用程序中的特定功能(例如，使用Spring表达式语言进行表达式评估而不是OGNL)。

## 标准表达式语法

大多数Thymeleaf属性允许将它们的值设置为或包含表达式，由于它们使用的方言，将其称为**标准表达式**。这些表达式可以有五种类型：

- [**变量表达式**](#变量表达式)：`${...}` 
- [**选择表达式**](#选择表达式)：`*{...}` 
- [**消息 (i18n) 表达式**](#消息(i18n)表达式)：`#{...}`
- [**链接 (URL) 表达式**](#链接(URL)表达式)：`@{...}` 
- [**片段表达式**](#片段表达式)：`~{...}`

### 变量表达式

变量表达式是OGNL表达式 - 如果将*Thymeleaf* 与*Spring* - 集成在上下文变量上(也称为Spring术语中的模型属性)，则为*Spring EL*。 它们看起来像这样：

```jsp
${session.user.name}
```

它们作为属性值或作为它们的一部分，取决于属性：

```jsp
<span th:text="${book.author.name}">
```

上面的表达式与下面是相同的(在OGNL和SpringEL中)：

```java
((Book)context.getVariable("book")).getAuthor().getName()
```

但是不仅在涉及输出的场景中找到变量表达式，而且还可以使用更复杂的处理方式，如:条件，迭代…等等。

```jsp
<li th:each="book : ${books}">
```

这里`${books}`从上下文中选择名为`books`的变量，并在`th:each`中使用循环将其评估为迭代器。

**${}中的表达式本质是OGNL**：

OGNL：Object-Graph Navigation Language对象-图 导航语言

**对象图**：

从根对象触发，通过特定的语法，逐层访问对象的各种属性。

![./images](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202307312155371.png)

**OGNL语法**：

**起点**：

在Thymeleaf环境下，`${}`中的表达式可以从下列元素开始：

- 访问属性域的起点
  - 请求域属性名
  - session
  - application
- param
- 内置对象
  - \#request
  - \#session
  - \#lists
  - \#strings

**属性访问语法**：

- 访问对象属性：使用getXxx()、setXxx()方法定义的属性
  - 对象.属性名
- 访问List集合或数组
  - 集合或数组[下标]
- 访问Map集合
  - Map集合.key
  - Map集合['key']

### 选择表达式

选择表达式就像变量表达式一样，它们不是整个上下文变量映射上执行，而是在先前选择的对象。 它们看起来像这样：

```java
*{customer.name}
```

它们所作用的对象由`th:object`属性指定：

```jsp
<div th:object="${book}">
  ...
  <span th:text="*{title}">...</span>
  ...
</div>
```

所以这相当于:

```java
{
  // th:object="${book}"
  final Book selection = (Book) context.getVariable("book");
  // th:text="*{title}"
  output(selection.getTitle());
}
```

### 消息(i18n)表达式

消息表达式(通常称为文本外部化，国际化或i18n)允许从外部源(如:`.properties`)文件中检索特定于语言环境的消息，通过键来引用这引用消息。

在Spring应用程序中，它将自动与Spring的MessageSource机制集成：

```jsp
#{main.title}
#{message.entrycreated(${entryId})}
```

在模板中使用它们的方式：

```html
<table>
  ...
  <th th:text="#{header.address.city}">...</th>
  <th th:text="#{header.address.country}">...</th>
  ...
</table>
```

**注意**：如果希望消息键由上下文变量的值确定，或者希望将变量指定为参数，则可以在消息表达式中使用变量表达式：

```java
#{${config.adminWelcomeKey}(${session.user.name})}
```

### 链接(URL)表达式

链接表达式在构建URL并向其添加有用的上下文和会话信息(通常称为URL重写的过程)。
因此，对于部署在Web服务器的`/myapp`上下文中的Web应用程序，可以使用以下表达式：

```jsp
<a th:href="@{/order/list}">...</a>
```

可以转换成如下的东西：

```jsp
<a href="/myapp/order/list">...</a>
```

甚至，如果需要保持会话，并且cookie未启用(或者服务器还不知道)，那么生成的格式：

```html
<a href="/myapp/order/list;jsessionid=s2ds3fa31abd241e2a01932">...</a>
```

网址也可以带参数：

```jsp
<a th:href="@{/order/details(id=${orderId},type=${orderType})}">...</a>
```

这将产生类似以下的结果：

```html
<!-- 注意＆符号会在标签属性中进行HTML转义... -->
<a href="/myapp/order/details?id=23&type=online">...</a>
```

链接表达式可以是相对的，在这种情况下，应用程序上下文将不会被加到URL的前面：

```jsp
<a th:href="@{../documents/report}">...</a>
```

也是服务器相对的(同样，没有应用程序上下文的前缀)：

```html
<a th:href="@{~/contents/main}">...</a>
```

和协议相关(就像绝对URL一样，但浏览器将使用与正在显示的页面相同的HTTP或HTTPS协议)：

```html
<a th:href="@{//static.mycompany.com/res/initial}">...</a>
```

当然，链接表达式也可以是绝对的：

```html
<a th:href="@{http://www.my.com/main}">...</a>
```

绝对(或协议相对)URL 在Thymeleaf链接表达式中由响应过滤器定义URL重写：在基于Servlet的Web应用程序中，对于每个输出的URL(上下文相对，相对，绝对…)，在显示URL之前，Thymeleaf总是调用`HttpServletResponse.encodeUrl(...)`机制。 这意味着一个过滤器可以通过包装HttpServletResponse对象来为应用程序执行自定义的URL重写。

### 片段表达式

片段表达式是一种简单的方法用来表示标记的片段并将其移动到模板中。 由于这些表达式，片段可以被复制，传递给其他模板的参数等等。

最常见的是使用`th:insert`或`th:replace`来插入片段：

```html
<div th:insert="~{commons :: main}">...</div>
```

但是它们可以在任何地方使用，就像任何其他变量一样：

```jsp
<div th:with="frag=~{footer :: #main/text()}">
  <p th:insert="${frag}">
</div>
```

片段表达式可以有参数。

### 文字和操作

有很多类型的文字和操作可用：

- **文字**：
  - 文本文字：`'one text'`, `'Another one!'`,`…`
  - 数字文字：`0`,`10`, `314`, `31.01`, `112.83`,`…`
  - 布尔文字：`true`,`false`
  - Null文字：`Null`
  - 文字标记：`one`, `sometext`, `main`,`…`
- **文本操作**：
  - 字符串连接：`+`
  - 文字替换：`|The name is ${name}|`
- **算术运算**：
  - 二进制操作：`+`, `-`, `*`, `/`, `%`
  - 减号(一元运算符)：`-`
- **布尔运算**：
  - 二进制运算符：`and`,`or`
  - 布尔否定(一元运算符)：`!`,`not`
- **比较和相等**：
  - 比较运算符：`>`,`<`,`>=`,`<=`(`gt`,`lt`,`ge`,`le`)
  - 相等运算符：`==`, `!=` (`eq`, `ne`)
- **条件操作符**：
  - If-then：`(if) ? (then)`
  - If-then-else：`(if) ? (then) : (else)`
  - Default：`(value) ?: (defaultvalue)`

## 表达式预处理

表达式预处理，在`__`之间指定：

```java
#{selection.__${sel.code}__}
```

第一个被执行的变量表达式是：`${sel.code}`，并且将使用它的结果作为表达式的一部分 (假设`${sel.code}`的结果为:`ALL`)，在此处执行国际化的情况下(这将查找与关键`selection.ALL`消息)。

## 基本的属性

 `th:`文本代替了标签的主体：

```jsp
<p th:text="#{msg.welcome}">Welcome everyone!</p>
```

`th:each`重复它所在元素的次数，由它的表达式返回的数组或列表所指定的次数，为迭代元素创建一个内部变量，其语法与Java的foreach表达式相同:

```html
<li th:each="book : ${books}" th:text="${book.title}">En las Orillas del Sar</li>
```

最后，Thymeleaf为特定的XHTML和HTML5属性提供了许多`th`属性，这些属性只评估它们的表达式，并将这些属性的值设置为结果。

```html
<form th:action="@{/createOrder}">
<input type="button" th:value="#{form.submit}" />
<a th:href="@{/admin/users}">
```