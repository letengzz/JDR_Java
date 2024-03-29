# 基本语法

## th名称空间

动态渲染指定的 html 标签属性值、或者th指令(遍历、判断等)

![images](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202301031103270.png)

## 表达式语法

### 修改标签文本值

- `th:text`：标签体内文本值渲染
- `th:utext`：不会转义，显示为html原本的样子。

**基本语法**：

```html
<p th:text="标签体新值">标签体原始值</p>
```

`th:text`作用：

- 不经过服务器解析，直接用浏览器打开HTML文件，看到的是『标签体原始值』
- 经过服务器解析，Thymeleaf引擎根据th:text属性指定的『标签体新值』去**替换**『标签体原始值』

**字面量**：

『字面量』是一个经常会遇到的概念，我们可以对照『变量』来理解它的含义。

```java
// a是变量，100是字面量
int a = 100;
System.out.println("a = " + a);
```

- 变量：变量名字符串本身不是它的值，它指向的才是它的值
- 字面量：它就是字面上的含义，我们从『字面』上看到的直接就是它的值

在th:text属性中使用的就是『字面量』，它**不指代任何其他值**。

### 修改指定属性值

`th:属性`：标签指定属性渲染

**基本语法**：

```html
<input type="text" name="username" th:value="文本框新值" value="文本框旧值" />
```

**语法**：任何HTML标签原有的属性，前面加上`th:`就都可以通过Thymeleaf来设定新值。

### 修改标签任意属性

`th:attr`：标签任意属性渲染

```jsp
<img src="../../images/gtvglogo.png" 
     th:attr="src=@{/images/gtvglogo.png},title=#{logo},alt=#{logo}" />
```

### 解析URL地址

**基本语法**：

```html
<p th:text="@{/aaa/bbb/ccc}">标签体原始值</p>
```

经过解析后得到：

> /view/aaa/bbb/ccc

所以`@{}`的作用是**在字符串前附加『上下文路径』**

这个语法的好处是：实际开发过程中，项目在不同环境部署时，Web应用的名字有可能发生变化。所以上下文路径不能写死。而通过`@{}`动态获取上下文路径后，不管怎么变都不怕啦！

### 首页使用URL地址解析

![./images](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202301031101914.png)

如果直接访问index.html本身，那么index.html是不需要通过Servlet，当然也不经过模板引擎，所以index.html上的Thymeleaf的任何表达式都不会被解析。

解决办法：通过Servlet访问index.html，这样就可以让模板引擎渲染页面了：

![./images](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202301031103444.png)

**优点**：所有和业务功能相关的请求都能够确保它们通过Servlet来处理，这样就方便我们统一对这些请求进行特定规则的限定。

### 给URL地址后面附加请求参数

参照官方文档说明：

![./images](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202301031103922.png)

**基本语法**：

```html
<p th:text="@{/aaa/bbb/ccc(id=${id})}">标签体原始值</p>
```

### 直接执行表达式

Servlet代码：

```java
request.setAttribute("reqAttrName", "<span>hello-value</span>");
```

页面代码：

```html
<p>有转义效果：[[${reqAttrName}]]</p>
<p>无转义效果：[(${reqAttrName})]</p>
```

执行效果：

> 有转义效果：`&lt;span&gt;hello-value&lt;/span&gt;`
>
>
> 无转义效果：`<span>hello-value</span>`

## 字符串与表达式拼接

**普通字符串与表达式拼接**：

```html
<span th:text="'欢迎您:' + ${user.name} + '!'"></span>
```

字符串字面值需要用`''`，拼接起来非常麻烦，Thymeleaf对此进行了简化，使用一对`|`即可：

```html
<span th:text="|欢迎您:${user.name}|"></span>
```

## 运算

注意：`${}`内部的是通过OGNL表达式引擎解析的，外部的才是通过Thymeleaf的引擎解析，因此运算符尽量放在`${}`外进行。

- 算术运算

  支持的算术运算符：`+ - * / %`

  ```java
  <span th:text="${user.age}"></span>         //21
  <span th:text="${user.age}%2 == 0"></span>  //false
  ```

- 比较运算

  支持的比较运算：`>`, `<`, `>=` and `<=` ，但是`>`, `<`不能直接使用，因为xml会解析为标签，要使用别名。

  **注意**： `==` and `!=`不仅可以比较数值，类似于equals的功能。

  可以使用的别名：`gt (>), lt (<), ge (>=), le (<=), not (!), Also eq (==), neq/ne (!=)。`

  ```java
  <span th:text="${age} gt 200"></span>
  ```

- 条件运算

  - 三元运算

  ```html
  <span th:text="${user.sex} ? '男':'女'"></span>
  ```

  - 默认值

    有的时候，我们取一个值可能为空，这个时候需要做非空判断，可以使用 `表达式 ?: 默认值`简写：

  ```html
  <span th:text="${user.name} ?: '二狗'"></span>
  ```

  当前面的表达式值为null时，就会使用后面的默认值。

  **注意**：`?:`之间没有空格。

## 访问域对象

### 域对象

#### 请求域

在请求转发的场景下，可以借助HttpServletRequest对象内部给提供的存储空间，帮助携带数据，把数据发送给转发的目标资源。

**请求域**：HttpServletRequest对象内部给我们提供的存储空间

![./images](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202301031102212.png)

#### 会话域

![./images](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202301031102896.png)

#### 应用域

![./images](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202301031103427.png)

> PS：在使用的视图是JSP的时候，域对象有4个
>
> - pageContext
> - request：请求域
> - session：会话域
> - application：应用域
>
> 所以在JSP的使用背景下，可以说域对象有4个，现在使用Thymeleaf了，没有pageContext。

### 在Servlet中将数据存入属性域

#### 操作请求域

Servlet中代码：

```java
String requestAttrName = "helloRequestAttr";
String requestAttrValue = "helloRequestAttr-VALUE";

request.setAttribute(requestAttrName, requestAttrValue);
```

Thymeleaf表达式：

```html
<p th:text="${helloRequestAttr}">request field value</p>
```

#### 操作会话域

Servlet中代码：

```java
// ①通过request对象获取session对象
HttpSession session = request.getSession();

// ②存入数据
session.setAttribute("helloSessionAttr", "helloSessionAttr-VALUE");
```

Thymeleaf表达式：

```html
<p th:text="${session.helloSessionAttr}">这里显示会话域数据</p>
```

#### 操作应用域

Servlet中代码：

```java
// ①通过调用父类的方法获取ServletContext对象
ServletContext servletContext = getServletContext();

// ②存入数据
servletContext.setAttribute("helloAppAttr", "helloAppAttr-VALUE");
```

Thymeleaf表达式：

```html
<p th:text="${application.helloAppAttr}">这里显示应用域数据</p>
```

## 获取请求参数

在页面上（模板页面）获取请求参数。底层机制是：

![./images](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202301031104581.png)

### 一个名字一个值

页面代码：

```html
<p th:text="${param.username}">这里替换为请求参数的值</p>
```

页面显示效果：

![image-20230103110431621](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202301031104057.png)

### 一个名字多个值

页面代码：

```html
<p th:text="${param.team}">这里替换为请求参数的值</p>
```

页面显示效果：

![image-20230103110448865](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202301031104720.png)

如果想要精确获取某一个值，可以使用数组下标。页面代码：

```html
<p th:text="${param.team[0]}">这里替换为请求参数的值</p>
<p th:text="${param.team[1]}">这里替换为请求参数的值</p>
```

页面显示效果：

![image-20230103110502480](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202301031105918.png)

## 内置对象

内置对象其实就是在表达式中**可以直接使用**的对象。

### 基本内置对象

![./images](https://heavy_code_industry.gitee.io/code_heavy_industry/assets/img/img021.98446d22.png)

**例**：

```html
<h3>表达式的基本内置对象</h3>
<p th:text="${#request.getClass().getName()}">这里显示#request对象的全类名</p>
<p th:text="${#request.getContextPath()}">调用#request对象的getContextPath()方法</p>
<p th:text="${#request.getAttribute('helloRequestAttr')}">调用#request对象的getAttribute()方法，读取属性域</p>
```

**基本思路**：

- 如果不清楚这个对象有哪些方法可以使用，那么就通过`getClass().getName()`获取全类名，再回到Java环境查看这个对象有哪些方法
- 内置对象的方法可以直接调用
- 调用方法时需要传参的也可以直接传入参数

### 公共内置对象

![./images](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202301031105216.png)

Servlet中将List集合数据存入请求域：

```java
request.setAttribute("aNotEmptyList", Arrays.asList("aaa","bbb","ccc"));
request.setAttribute("anEmptyList", new ArrayList<>());
```

页面代码：

```html
<p>#list对象isEmpty方法判断集合整体是否为空aNotEmptyList：<span th:text="${#lists.isEmpty(aNotEmptyList)}">测试#lists</span></p>
<p>#list对象isEmpty方法判断集合整体是否为空anEmptyList：<span th:text="${#lists.isEmpty(anEmptyList)}">测试#lists</span></p>
```

公共内置对象对应的源码位置：

![./images](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202301031105175.png)

## 分支与迭代

### 分支

#### if和unless

标记`th:if`、`th:unless`的标签根据条件决定是否显示。

示例的实体类：

```java
public class Employee {
    private Integer empId;
    private String empName;
    private Double empSalary;
}
```

示例的Servlet代码：

```java
protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {

    // 1.创建ArrayList对象并填充
    List<Employee> employeeList = new ArrayList<>();

    employeeList.add(new Employee(1, "tom", 500.00));
    employeeList.add(new Employee(2, "jerry", 600.00));
    employeeList.add(new Employee(3, "harry", 700.00));

    // 2.将集合数据存入请求域
    request.setAttribute("employeeList", employeeList);

    // 3.调用父类方法渲染视图
    super.processTemplate("list", request, response);
}
```

HTML代码：

```html
<table>
    <tr>
        <th>员工编号</th>
        <th>员工姓名</th>
        <th>员工工资</th>
    </tr>
    <tr th:if="${#lists.isEmpty(employeeList)}">
        <td colspan="3">抱歉！没有查询到你搜索的数据！</td>
    </tr>
    <tr th:if="${not #lists.isEmpty(employeeList)}">
        <td colspan="3">有数据！</td>
    </tr>
    <tr th:unless="${#lists.isEmpty(employeeList)}">
        <td colspan="3">有数据！</td>
    </tr>
</table>
```

if配合not关键词和unless配合原表达式效果是一样的，看自己的喜好。

#### switch

使用`th:switch` 和 `th:case`

**注意**：一旦有一个th:case成立，其它的则不再判断。与java中的switch是一样的。

另外`th:case="*"`表示默认，放最后。

```html
<h3>测试switch</h3>
<div th:switch="${user.memberLevel}">
    <p th:case="level-1">银牌会员</p>
    <p th:case="level-2">金牌会员</p>
    <p th:case="level-3">白金会员</p>
    <p th:case="level-4">钻石会员</p>
    <p th:case="*">不是会员哦</p>
</div>
```

### 迭代

`${users}` 是要遍历的集合，支持类型：

- Iterable，实现了Iterable接口的类
- Enumeration，枚举
- Interator，迭代器
- Map，遍历得到的是Map.Entry
- Array，数组及其它一切符合数组结果的对象

在迭代的同时，我们也可以获取迭代的状态对象：

```html
<tr th:each="user,state : ${users}">
    <td th:text="${user.name}">Onions</td>
    <td th:text="${user.age}">2.41</td>
</tr>
```

state对象包含以下属性：

- index：从0开始的角标
- count：元素的个数，从1开始
- size：总元素个数
- current：当前遍历到的元素
- even/odd：返回是否为奇偶，boolean值
- first/last：返回是否为第一或最后，boolean值

```html
<h3>测试each</h3>
<table>
    <thead>
        <tr>
            <th>员工编号</th>
            <th>员工姓名</th>
            <th>员工工资</th>
        </tr>
    </thead>
    <tbody th:if="${#lists.isEmpty(employeeList)}">
        <tr>
            <td colspan="3">抱歉！没有查询到你搜索的数据！</td>
        </tr>
    </tbody>
    <tbody th:if="${not #lists.isEmpty(employeeList)}">
        <!-- 遍历出来的每一个元素的名字 : ${要遍历的集合} -->
        <tr th:each="employee : ${employeeList}">
            <td th:text="${employee.empId}">empId</td>
            <td th:text="${employee.empName}">empName</td>
            <td th:text="${employee.empSalary}">empSalary</td>
        </tr>
    </tbody>
</table>
```

在迭代过程中，可以参考下面的说明使用迭代状态：

![./images](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202301031105395.png)

```html
<h3>测试each</h3>
<table>
    <thead>
        <tr>
            <th>员工编号</th>
            <th>员工姓名</th>
            <th>员工工资</th>
            <th>迭代状态</th>
        </tr>
    </thead>
    <tbody th:if="${#lists.isEmpty(employeeList)}">
        <tr>
            <td colspan="3">抱歉！没有查询到你搜索的数据！</td>
        </tr>
    </tbody>
    <tbody th:if="${not #lists.isEmpty(employeeList)}">
        <!-- 遍历出来的每一个元素的名字 : ${要遍历的集合} -->
        <tr th:each="employee,empStatus : ${employeeList}">
            <td th:text="${employee.empId}">empId</td>
            <td th:text="${employee.empName}">empName</td>
            <td th:text="${employee.empSalary}">empSalary</td>
            <td th:text="${empStatus.count}">count</td>
        </tr>
    </tbody>
</table>
```

## 包含其他模板文件

**应用场景**：

抽取各个页面的公共部分：

![image-20230103110533309](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202310292033947.png)

**创建页面的代码片段**：

使用`th:fragment`来给这个片段命名：

```html
<div th:fragment="header">
    <p>被抽取出来的头部内容</p>
</div>
```

**包含到有需要的页面**：

| 语法       | 效果                                                     |
| ---------- | -------------------------------------------------------- |
| th:insert  | 把目标的代码片段整个插入到当前标签内部                   |
| th:replace | 用目标的代码替换当前标签                                 |
| th:include | 把目标的代码片段去除最外层标签，然后再插入到当前标签内部 |

**例**：

```html
<!-- 代码片段所在页面的逻辑视图 :: 代码片段的名称 -->
<div id="badBoy" th:insert="segment :: header">
    div标签的原始内容
</div>

<div id="worseBoy" th:replace="segment :: header">
    div标签的原始内容
</div>

<div id="worstBoy" th:include="segment :: header">
    div标签的原始内容
</div>
```

## 预处理JS模板

```html
<!--点击进入del()函数-->
<button th:onclick="'del('+${f.id}+')'">按钮</button>
<button th:onclick="|del(${f.id})|">按钮</button>

<script>
	function del(id){
        if(confirm('是否确认删除?')){
            window.location.href='del.do?id='+id;
        }
    }
</script>
```

模板引擎不仅可以渲染html，也可以对JS中的进行预处理。而且为了在纯静态环境下可以运行，其Thymeleaf代码可以被注释起来：

```javascript
<script th:inline="javascript">
    const user = /*[[${user}]]*/ {};
    const age = /*[[${user.age}]]*/ 20;
    console.log(user);
    console.log(age)
</script>
```

- 在script标签中通过`th:inline="javascript"`来声明这是要特殊处理的js脚本

- 语法结构：

  ```javascript
  const user = /*[[Thymeleaf表达式]]*/ "静态环境下的默认值";
  ```

  因为Thymeleaf被注释起来，因此即便是静态环境下， js代码也不会报错，而是采用表达式后面跟着的默认值。且User对象会被直接处理为json格式。