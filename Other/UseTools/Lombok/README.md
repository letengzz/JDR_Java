# Lombok

Lombok 是⼀个可以通过简单的注解形式来帮助简化消除⼀些必须有但显得很臃肿的 Java代码的⼯具，通过使用对应的注解，可以在编译源码的时候生成对应的方法。简而言之就是，**通过简单的注解来精简代码达到消除冗长代码的目的**。

- Lombok官网：https://projectlombok.org/

- GitHub地址：https://github.com/projectlombok/lombok

**Lombok 优点**：

- 提⾼编码效率 

- 使代码更简洁 
- 消除冗长代码 
- 避免修改字段名字时忘记修改方法名

**注意**：使用Lombok时，当有特殊需求时也可定制自己的代码。

## Lombok 常用注解

**常用注解**：

![图片1](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202302221828074.png)

## Lombok 使用

在SpringBoot中添加依赖：

```xml
<dependency>
	<groupId>org.projectlombok</groupId>
	<artifactId>lombok</artifactId>
</dependency>
```

****

在IDEA中安装Lombok插件，安装后重启即可：

![image-20230222164401161](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202302221828921.png)

Lombok既是⼀个IDE插件，也是⼀个项目要依赖的jar包。Lombok是依赖jar包的原因是因为编译时要用它的注解。是插件的原因是他要在编译器编译时通过操作AST(抽象语法树)改变字节码生成。也就是说他可以改变java语法。他不像spring的依赖注⼊或者hibernate的orm⼀样是运行时的特性，而是编译时的特性。

### 类属性相关

#### @Getter 注解

用于自动生成Getter方法。

**注意**：`@Getter`存在一定的命名规则，如果该字段为foo，则将其直接按照字段名称命名为`getFoo`，但是如果字段的类型为boolean，则会命名为`isFoo`。

**注解定义**：

```java
@Target({ElementType.FIELD, ElementType.TYPE})
@Retention(RetentionPolicy.SOURCE)
public @interface Getter {
    AccessLevel value() default AccessLevel.PUBLIC;  //自动生成Getter的访问权限级别

    AnyAnnotation[] onMethod() default {}; //用于添加额外的注解

    boolean lazy() default false;  //懒加载功能

	...
}
```

**用法**：

```java
@Getter  //添加到类上时，将为类中所有字段生成getter方法
public class Account {
    private int id;
    @Getter //添加到字段上时，只为该字段生成getter方法
    private String name;
    private int age;
}
```

**调整**：

- 访问权限级别：默认情况下为`public`

  - `AccessLevel.PUBLIC`：对应public关键字
  - `AccessLevel.MODULE`：仅限模块内使用，与PACKAGE类似，相当于不添加任何访问权限关键字
  - `AccessLevel.PROTECTED`：对应protected关键字
  - `AccessLevel.PACKAGE`：相当于不添加任何访问权限关键字
  - `AccessLevel.PRIVATE`：对应private关键字
  - `AccessLevel.NONE`：表示不生成对应的方法，很适合对类中不需要生成的字段进行排除

  ```java
  @Getter(AccessLevel.PRIVATE)
  public class Account {
      private int id;
      private String name;
      private int age;
      private boolean gender;
  }
  ```

- 添加额外的注解：onMethod属性用于添加一些额外的注解到生成的方法上，比如：为Getter方法添加一个额外的 `@Deprecated`表示它不推荐使用：

  ```java
  @Getter
  public class Account {
      private int id;
      @Getter(onMethod_ = {@Deprecated}) 
      private String name;
      private int age;
      private boolean gender;
  }
  ```

- 懒加载：lazy属性用于控制懒加载，懒加载在一开始的时候此字段没有值，当需要的时候再将值添加到此处。

  要求：字段必须是private且final

  ```java
  @Getter
  public class Account {
      private int id;
      @Getter(lazy = true)
      private final String name = "张三";
      private int age;
      private boolean gender;
  }
  ```

  生成的代码：

  ```java
  public class Account {
      // 自动将字段修改为AtomicReference原子类型，以防止多线程环境下出现问题
      private final AtomicReference<Object> name = new AtomicReference();
  
      //当调用getName才会初始化字段的值，为了保证初始化只进行一次，整个过程与懒汉式单例模式一致
      public String getName() {
          Object value = this.name.get();
          if (value == null) {  //判断值是否为null，如果是则需要进行懒初始化
              synchronized(this.name) {  //对字段加锁，保证同时只能进一个
                  value = this.name.get();
                  if (value == null) {  //再次进行一次判断，因为有可能其他线程后进入
                      String actualValue = "张三";
                      value = "张三" == null ? this.name : "张三";
                      this.name.set(value);
                  }
              }
          }
  		//返回得到的结果
          return (String)(value == this.name ? null : value);
      }
  }
  ```

  作为使用者来说，只需要知道懒加载就是将字段的值延迟赋值给它，适合以下场景：

  ```java
  @Getter
  public class Account {
      @Getter(lazy = true)
      private final String name = initValue();
  
      private String initValue(){
          System.out.println("我不希望在对象创建时就执行");
          return "张三";
      }
  }
  ```

#### @Setter 注解

用于生成字段对应的Setter方法。

**说明**：@Setter 注解同样会根据字段名称来生成Setter方法，其他参数与`@Getter`用法一致。唯一不一样的参数为 `onParam`，它可以在形参上额外添加的自定义注解。

**用法**：

```java
@Setter
public class Account {
    private int id;
    private String name;
    private int age;
    private boolean gender;
}
```

**注意**：如果手动编写了对应字段的Getter或是Setter方法(按照上述命名规则进行判断)，那么Lombok提供的注解将不会生效，也不会覆盖我们自己编写的方法。

如果需要对原本内容进行覆盖，也可以使用 `@Tolerate` 注解使Lombok忽略它的存在，直接覆盖掉。

### 构造方法相关

Lombok可以自动生成对应的构造方法，它提供了三个用于处理构造方法的注解。

#### 生成所有字段

使用`@AllArgsConstructor`为类中所有字段生成一个构造方法：

**注解定义**：

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.SOURCE)
public @interface AllArgsConstructor {
    //用于生成一个静态构造方法
    String staticName() default "";

    //用于在构造方法上添加额外的注解
    AnyAnnotation[] onConstructor() default {};

    //设置构造方法的访问权限级别
    AccessLevel access() default AccessLevel.PUBLIC;
    ...
}
```

**用法**：

```java
@AllArgsConstructor
public class Account {
    private int id;
    private String name;
    private int age;
    private boolean gender;
}
```

静态构造方法：这种方法非常适合用作泛型的类型推断，简化代码，比如 `MapEntry.of("foo",5)` 而不是更长的`new MapEntry<String,Integer>("foo",5)`

```java
@AllArgsConstructor(staticName = "of")
public class Account {
    private int id;
    private String name;
    private int age;
    private boolean gender;
}
```

```java
Account account = Account.of(1, "zhangsan",2, true);
```

#### 无参构造

使用 `@NoArgsConstructor`来生成无参构造。

```java
@NoArgsConstructor
public class User {
	private String name;
	private Integer age;
}
```

**注意**：当使用 `@NoArgsConstructor` 无参构造时不允许存在final类型的字段，否则会出现错误，可以使用 force属性，可以在创建无参构造时，为final类型的字段给一个默认值

```java
@NoArgsConstructor(force = true) //强制生成无参构造器
public class User {
	private String name;  //字段必须初始化
	private Integer age;
}
```

#### 需要初始化参数

通过 `@RequiredArgsConstructor` 来生成需要初始化的参数的构造方法，也就是说类中哪些字段为final，就针对这些字段来生成对应的构造方法。

```java
@RequiredArgsConstructor //生成包含所有final字段和非final字段（如果存在）的构造器
public class User {
	private final String name; 
	private Integer age;
}
```

### 打印对象

在类上添加 `@ToString`注解即可生成`toString()`方法。

**注解定义**：

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.SOURCE)
public @interface ToString {
    //是否在打印的内容中带上对应字段的名字
    boolean includeFieldNames() default true;

    //用于排除不需要打印的字段 (这种用法很快会被移除，不建议使用)
    String[] exclude() default {};

    //和上面相反，设置哪些字段需要打印，默认打印所有(这种用法很快会被移除，不建议使用)
    String[] of() default {};

    //不仅为当前类中所有字段生成，同时还调用父类toString进行拼接
    boolean callSuper() default false;

    //默认情况下生成的toString会尽可能使用get方法获取字段值，也可以手动关闭这个功能
    boolean doNotUseGetters() default false;

    //开启后将只为字段或get方法上添加了@ToString.Include注解的内容生成toString方法，白名单模式
    boolean onlyExplicitlyIncluded() default false;

    /**
    * 用于排除toString中的字段
    */
    @Target({ElementType.FIELD})
    @Retention(RetentionPolicy.SOURCE)
    public @interface Exclude {
    }

    /**
    * 用于手动包含toString中的字段
    */
    @Target({ElementType.FIELD, ElementType.METHOD})
    @Retention(RetentionPolicy.SOURCE)
    public @interface Include {
        //配置字段打印顺序的优先级
        int rank() default 0;

        //配置一个自定义的字段名称进行打印
        String name() default "";
    }
}
```

**用法**：

```java
@AllArgsConstructor
@ToString
public class Account {
    private int id;
    private String name;
    private int age;
}
```

不显示字段的名字：

```java
@AllArgsConstructor
@ToString(includeFieldNames = false)
public class Account {
    private int id;
    private String name;
    private int age;
}
```

![image-20240811151242536](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202408111512334.png)

显示父类toString进行拼接：

```java
@AllArgsConstructor
@ToString(callSuper = true)
public class Account {
    private int id;
    private String name;
    private int age;
}
```

![image-20240811151906038](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202408111519481.png)

`@ToString`对于get方法的特殊机制，会尽可能使用自定义的get方法获取字段的值，命名规则判定和之前一样：配置`doNotUseGetters`，就不会采用getter方法。

```java
@ToString(doNotUseGetters = true)
@AllArgsConstructor
public class Account {
    private int id;
    private String name;

    public String getName() {
        return name + "同学";  //编写一个自定义的getName方法
    }
    
    private int age;
}
```

![image-20240811152348799](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202408111523384.png)

使用 `@ToString.Exclude` 注解排除某些字段：

```java
@ToString
@AllArgsConstructor
public class Account {
    private int id;
    private String name;
    @ToString.Exclude
    private int age;
}
```

![image-20240811154052926](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202408111540875.png)

使用白名单模式：需要使用 `onlyExplicitlyIncluded = true`，将需要打印的字段添加 ` @ToString.Include`：

```java
@ToString(onlyExplicitlyIncluded = true)
@AllArgsConstructor
public class Account {
    private int id;
    @ToString.Include
    private String name;
    @ToString.Include(rank = 1,name = "年龄")
    private int age;
}
```

![image-20240811154501159](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202408111545401.png)

`@ToString.Include`不仅可以对字段生效，还可以对方法生效，可以将某些方法执行后的结果也包含在toString中：

```java
@ToString(onlyExplicitlyIncluded = true)
@AllArgsConstructor
public class Account {
    ...
    
    @ToString.Include
    public String test(){
        return "测试";
    } 
}
```

### 比较相关

![image-20240811160531924](assets/image-20240811160531924.png)

![image-20240811161352101](assets/image-20240811161352101.png)

![image-20240811161358398](assets/image-20240811161358398.png)

![image-20240811161515687](assets/image-20240811161515687.png)

![image-20240811161521433](assets/image-20240811161521433.png)

![image-20240811161534513](assets/image-20240811161534513.png)

![image-20240811161617589](assets/image-20240811161617589.png)

![image-20240811161641371](assets/image-20240811161641371.png)

### 生成上述注解

![image-20240811162012238](assets/image-20240811162012238.png)

![image-20240811162027879](assets/image-20240811162027879.png)



### 简化日志开发

![image-20240811174841369](assets/image-20240811174841369.png)

![image-20240811174900782](assets/image-20240811174900782.png)

![image-20240811174937679](assets/image-20240811174937679.png)

![image-20240811174949460](assets/image-20240811174949460.png)

![image-20240811175000767](assets/image-20240811175000767.png)

![image-20240811175008048](assets/image-20240811175008048.png)



```java
@Slf4j
@RestController
public class HelloController {
    @RequestMapping("/hello")
    public String handle01(@RequestParam("name") String name){
        
        log.info("请求进来了....");
        
        return "Hello, Spring Boot"+"你好："+name;
    }
}
```

### 链式调用

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
@Accessors(chain = true) //支持链式编程
public class User {
    private String name;
    private Integer age;
}
```

```java
@Test
void chainTest() {
    User user = new User().setName("张三").setAge(15);
	System.out.println("user = " + user);
}
```

### 建造者模式

![image-20240811162101095](assets/image-20240811162101095.png)

![image-20240811162313566](assets/image-20240811162313566.png)

![image-20240811162343321](assets/image-20240811162343321.png)

![image-20240811162415138](assets/image-20240811162415138.png)

![image-20240811162440421](assets/image-20240811162440421.png)

![image-20240811162510062](assets/image-20240811162510062.png)

![image-20240811162519551](assets/image-20240811162519551.png)

![image-20240811162607720](assets/image-20240811162607720.png)

![image-20240811162638027](assets/image-20240811162638027.png)

![image-20240811162713907](assets/image-20240811162713907.png)

![image-20240811162806897](assets/image-20240811162806897.png)

![image-20240811162811255](assets/image-20240811162811255.png)



```java
@Data
@AllArgsConstructor
@NoArgsConstructor
@Builder
public class User {
    private String name;
    private Integer age;
}
```

```java
@Test
void builderTest(){
    User user = User.builder().name("李四").age(18).build();
	System.out.println("user = " + user);
}
```

### 变量相关

![image-20240811162859372](assets/image-20240811162859372.png)

![image-20240811163014260](assets/image-20240811163014260.png)

![image-20240811163037007](assets/image-20240811163037007.png)

### 资源释放和异常处理

![image-20240811163100003](assets/image-20240811163100003.png)

![image-20240811163300830](assets/image-20240811163300830.png)

![image-20240811163356243](assets/image-20240811163356243.png)

![image-20240811163406084](assets/image-20240811163406084.png)

![image-20240811163420442](assets/image-20240811163420442.png)

![image-20240811163441413](assets/image-20240811163441413.png)

![image-20240811163511496](assets/image-20240811163511496.png)

![image-20240811163516958](assets/image-20240811163516958.png)

### 非空判断

![image-20240811163552398](assets/image-20240811163552398.png)

![image-20240811163647726](assets/image-20240811163647726.png)

![image-20240811163707404](assets/image-20240811163707404.png)

![image-20240811163722751](assets/image-20240811163722751.png)

### 锁处理

![image-20240811163826420](assets/image-20240811163826420.png)

![image-20240811174527463](assets/image-20240811174527463.png)

![image-20240811174533748](assets/image-20240811174533748.png)

![image-20240811174600691](assets/image-20240811174600691.png)

![image-20240811174605928](assets/image-20240811174605928.png)

![image-20240811174611743](assets/image-20240811174611743.png)

![image-20240811174625707](assets/image-20240811174625707.png)

![image-20240811174634003](assets/image-20240811174634003.png)

![image-20240811174707898](assets/image-20240811174707898.png)

![image-20240811174747622](assets/image-20240811174747622.png)

![image-20240811174754562](assets/image-20240811174754562.png)

![image-20240811174800145](assets/image-20240811174800145.png)

​	![image-20240811174818859](assets/image-20240811174818859.png)



## 试验性功能

![image-20240811175643043](assets/image-20240811175643043.png)

![image-20240811175902468](assets/image-20240811175902468.png)

![image-20240811175918076](assets/image-20240811175918076.png)

![image-20240811175936476](assets/image-20240811175936476.png)

![image-20240811180032243](assets/image-20240811180032243.png)

![image-20240811180132815](assets/image-20240811180132815.png)

![image-20240811180203168](assets/image-20240811180203168.png)

![image-20240811180219252](assets/image-20240811180219252.png)



### 强化Getter和Setter

### 添加新方法到现有类

![image-20240811181028203](assets/image-20240811181028203.png)

![image-20240811181228544](assets/image-20240811181228544.png)

![image-20240811181318911](assets/image-20240811181318911.png)

![image-20240811181328723](assets/image-20240811181328723.png)

![image-20240811181337188](assets/image-20240811181337188.png)

![image-20240811181439184](assets/image-20240811181439184.png)

![image-20240811181500050](assets/image-20240811181500050.png)

![image-20240811181555308](assets/image-20240811181555308.png)

![image-20240811181603192](assets/image-20240811181603192.png)

![image-20240811181610493](assets/image-20240811181610493.png)

![image-20240811181640157](assets/image-20240811181640157.png)

![image-20240811181643807](assets/image-20240811181643807.png)

![image-20240811181650500](assets/image-20240811181650500.png)

![image-20240811181655866](assets/image-20240811181655866.png)

![image-20240811181705713](assets/image-20240811181705713.png)

![image-20240811181816669](assets/image-20240811181816669.png)

![image-20240811181848902](assets/image-20240811181848902.png)

![image-20240811181902841](assets/image-20240811181902841.png)

![image-20240811182021065](assets/image-20240811182021065.png)



### 默认字段修饰符

默认字段修饰可以为类中字段快速生成对应的修饰符，避免手动编写，只需要添加 `@FieldDefaults`即可：

```java
@FieldDefaults(level= AccessLevel.PRIVATE)
public class Account {
    int id;
    String name;
    int age;
}
```

生成的代码中所有字段就自动变成private：

```java
public class Account {
    private int id;
    private String name;
    private int age;
}
```

### 委托属性

![image-20240811180521731](assets/image-20240811180521731.png)

![image-20240811180533026](assets/image-20240811180533026.png)

![image-20240811180702663](assets/image-20240811180702663.png)

![image-20240811180817367](assets/image-20240811180817367.png)

![image-20240811180830457](assets/image-20240811180830457.png)

![image-20240811180912021](assets/image-20240811180912021.png)

![image-20240811180916054](assets/image-20240811180916054.png)



### 工具类

![image-20240811180421604](assets/image-20240811180421604.png)

![image-20240811180453061](assets/image-20240811180453061.png)



### 标准异常

在项目中常常需要创建各种各样的自定义类型，便于快速定位问题原因。



![image-20240811180312850](assets/image-20240811180312850.png)

![image-20240811180403206](assets/image-20240811180403206.png)

![image-20240811180411158](assets/image-20240811180411158.png)



