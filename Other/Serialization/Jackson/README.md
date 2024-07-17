# Jackson

Jackson是一个用于处理JSON数据的开源`Java`库。JSON (`JavaScript Object Notation`) 是一种轻量级的数据交换格式，易于阅读和编写，同时也易于计算机解析和生成。在`Java`领域，Jackson已经成为处理JSON数据的事实标准库。它提供了丰富的功能，包括将Java对象转换为JSON字符串 (序列化)以及将JSON字符串转换为Java对象 (反序列化)

`Jackson`主要由三个核心包组成：

- jackson-databind：提供了通用的数据绑定功能 (将Java对象与JSON数据相互转换)
- jackson-core：提供了核心的低级JSON处理API (例如JsonParser和JsonGenerator)
- jackson-annotations：提供了用于配置数据绑定的注解

## Jackson 优势

尽管Java生态系统中有其他处理JSON数据的库（如Gson和JSON-java），但Jackson仍然是许多开发者的首选，原因包括：

- 性能：Jackson性能优越，对内存和CPU的使用都相对较低。许多性能基准测试表明，Jackson在序列化和反序列化方面都比其他库更快。
- 功能丰富：Jackson提供了许多功能，包括注解、自定义序列化和反序列化、动态解析等，使其非常灵活和强大。
- 易于使用：Jackson的API简单易用，使得开发者可以轻松地在他们的应用程序中集成和使用。
- 社区支持：Jackson拥有庞大的开发者社区，这意味着有更多的文档、教程和问题解答可供参考。
- 模块化：Jackson支持通过模块扩展其功能，例如Java 8时间库、Joda-Time和Kotlin等。
- 兼容性：Jackson可以很好地与其他流行的Java框架（如Spring）集成。

综上所述，Jackson是一个强大且易于使用的库，值得Java开发者在处理JSON数据时使用

## Jackson 基本功能

Jackson库的核心功能是将Java对象转换为JSON字符串 (序列化) 以及将JSON字符串转换为Java对象 (反序列化)

### 序列化

序列化：将Java对象转换为JSON字符串的过程。这在许多场景中非常有用，例如在将数据发送到Web客户端时，或者在将数据存储到文件或数据库时。Jackson通过ObjectMapper类来实现序列化。

```java
import com.fasterxml.jackson.databind.ObjectMapper;

public class Person {
    public String name;
    public int age;

    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }
    public static void main(String[] args) {
        ObjectMapper objectMapper = new ObjectMapper();
        Person person = new Person("Alice", 30);

        try {
            String jsonString = objectMapper.writeValueAsString(person);
            System.out.println(jsonString); // 输出：{"name":"Alice","age":30}
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

### 反序列化

反序列化：将JSON字符串转换回Java对象的过程。这在从Web客户端接收数据或从文件或数据库读取数据时非常有用。同样，Jackson使用ObjectMapper类来实现反序列化。

```java
import com.fasterxml.jackson.databind.ObjectMapper;

public class Person {
    public String name;
    public int age;
    public Person() {
    }
    public static void main(String[] args) {
        ObjectMapper objectMapper = new ObjectMapper();
        String jsonString = "{\"name\":\"Alice\",\"age\":30}";

        try {
            Person person = objectMapper.readValue(jsonString, Person.class);
            System.out.println("Name: " + person.name + ", Age: " + person.age); // 输出：Name: Alice, Age: 30
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

## Jackson 主要方法

Jackson主要API和组件：

- ObjectMapper：Jackson库的核心类，用于序列化和反序列化操作。

  主要方法有：

  - `writeValueAsString(Object)`：将Java对象序列化为JSON字符串
  - `readValue(String, Class)`：将JSON字符串反序列化为Java对象

- JsonParser：用于从JSON数据源 (如文件、输入流或字符串) 解析JSON数据。

  主要方法有：

  - `nextToken()`：获取下一个JSON令牌(如START_OBJECT、FIELD_NAME等)
  - `getValueAsString()`：将当前令牌作为字符串返回。
  - `getValueAsInt()`：将当前令牌作为整数返回。

- JsonGenerator：用于将`JSON`数据写入数据源 (如文件、输出流或字符串缓冲区)。

  主要方法有：

  - `writeStartObject()`：写入开始对象标记 (`{`)
  - `writeFieldName(String)`：写入字段名称
  - `writeString(String)`：写入字符串值
  - `writeEndObject()`：写入结束对象标记(`}`)

- `JsonNode`：用于表示JSON树模型中的节点，可以是对象、数组、字符串、数字等。

  主要方法有：

  - `get(String)`：获取指定字段的子节点
  - `path(String)`：获取指定字段的子节点，如果不存在则返回一个"missing"节点
  - `isObject()`：检查当前节点是否是一个对象
  - `isArray()`：检查当前节点是否是一个数组

## Jackson 注解

Jackson提供了一系列注解来配置序列化和反序列化过程。一些常用注解包括：

- `@JsonProperty`：指定在JSON数据中的名称。
- `@JsonIgnore`：指定字段在序列化和反序列化过程中被忽略。
- `@JsonCreator`：指定用于反序列化的构造函数或工厂方法。
- `@JsonSerialize`：指定用于序列化特定字段或类的自定义序列化器。
- `@JsonDeserialize`：指定用于反序列化特定字段或类的自定义反序列化器。

### @JsonProperty

指定 Java 属性与 JSON 属性之间的映射关系。

常用的参数：

- `value`：用于指定 JSON 属性的名称，当 Java 属性和 JSON 属性名称不一致时使用。
- `access`：用于指定该属性的访问方式，常用的取值有 `JsonAccess.READ_ONLY` (只读)，`JsonAccess.WRITE_ONLY` (只写) 和 `JsonAccess.READ_WRITE` (可读可写)

```java
public class Person {
    @JsonProperty(value = "name")
    private String fullName;
    
    @JsonProperty(access = JsonProperty.Access.WRITE_ONLY)
    private String password;
    
    // getters and setters
}

Person person = new Person();
person.setFullName("John Smith");
person.setPassword("123456");

ObjectMapper mapper = new ObjectMapper();
String json = mapper.writeValueAsString(person);
// {"name":"John Smith"}

Person person2 = mapper.readValue(json, Person.class);
System.out.println(person2.getFullName());
// John Smith
System.out.println(person2.getPassword());
// null
```

### @JsonIgnore

用于禁用 Java 属性的序列化和反序列化

```java
public class Person {
    private String fullName;
    
    @JsonIgnore
    private String password;
    
    // getters and setters
}

Person person = new Person();
person.setFullName("John Smith");
person.setPassword("123456");

ObjectMapper mapper = new ObjectMapper();
String json = mapper.writeValueAsString(person);
// {"fullName":"John Smith"}

Person person2 = mapper.readValue("{\"fullName\":\"John Smith\",\"password\":\"123456\"}", Person.class);
System.out.println(person2.getFullName());
// John Smith
System.out.println(person2.getPassword());
// null
```

### @JsonFormat

用于指定 `Java` 属性的日期和时间格式。

常用的参数：

- `shape`：用于指定日期和时间的格式，可选的取值有 `JsonFormat.Shape.STRING` (以字符串形式表示) 和 `JsonFormat.Shape.NUMBER` (以时间戳形式表示)
- `pattern`：用于指定日期和时间的格式模板，例：`yyyy-MM-dd HH:mm:ss`

```java
public class Person {
    private String fullName;
    
    @JsonFormat(shape = JsonFormat.Shape.STRING, pattern = "yyyy-MM-dd HH:mm:ss")
    private Date birthDate;
    
    // getters and setters
}

Person person = new Person();
person.setFullName("John Smith");
person.setBirthDate(new Date());

ObjectMapper mapper = new ObjectMapper();
String json = mapper.writeValueAsString(person);
// {"fullName":"John Smith","birthDate":"2022-05-16 10:38:30"}

Person person2 = mapper.readValue(json, Person.class);
System.out.println(person2.getFullName());
// John Smith
System.out.println(person2.getBirthDate());
// Mon May 16 10:38:30 CST 2022
```

### @JsonInclude

用于指定序列化 `Java` 对象时包含哪些属性。

常用的参数：

- `value`：用于指定包含哪些属性，可选的取值有 `JsonInclude.Include.ALWAYS`（始终包含）、`JsonInclude.Include.NON_NULL`（值不为 null 时包含）、`JsonInclude.Include.NON_DEFAULT`（值不为默认值时包含）、`JsonInclude.Include.NON_EMPTY`（值不为空时包含）和 `JsonInclude.Include.CUSTOM`（自定义条件）。
- `content`：用于指定自定义条件的实现类。

```java
@JsonInclude(JsonInclude.Include.NON_NULL)
public class Person {
    private String fullName;
    
    private Integer age;
    
    // getters and setters
}

Person person = new Person();
person.setFullName("John Smith");
// person.setAge(null);

ObjectMapper mapper = new ObjectMapper();
String json = mapper.writeValueAsString(person);
// {"fullName":"John Smith"}

Person person2 = mapper.readValue("{\"fullName\":\"John Smith\",\"age\":null}", Person.class);
System.out.println(person2.getFullName());
// John Smith
System.out.println(person2.getAge());
// null
```

### @JsonCreator

用于指定反序列化时使用的构造方法或工厂方法。

```java
public class Person {
    private String fullName;
    
    private Integer age;
    
    @JsonCreator
    public Person(@JsonProperty("fullName") String fullName, @JsonProperty("age") Integer age) {
        this.fullName = fullName;
        this.age = age;
    }
    
    // getters and setters
}

ObjectMapper mapper = new ObjectMapper();
Person person = mapper.readValue("{\"fullName\":\"John Smith\",\"age\":30}", Person.class);
System.out.println(person.getFullName());
// John Smith
System.out.println(person.getAge());
// 30
```

### @JsonSetter

用于指定反序列化时使用的方法，常用的参数有：

- `value`：用于指定 JSON 属性的名称，当方法名和 JSON 属性名称不一致时使用

```java
public class Person {
    private String fullName;
    
    private Integer age;
    
    @JsonSetter("name")
    public void setFullName(String fullName) {
        this.fullName = fullName;
    }
    
    // getters and setters
}

ObjectMapper mapper = new ObjectMapper();
Person person = mapper.readValue("{\"name\":\"John Smith\",\"age\":30}", Person.class);
System.out.println(person.getFullName());
// John Smith
System.out.println(person.getAge());
// 30
```

### @JsonGetter

用于指定序列化时使用的方法，常用的参数有：

- `value`：用于指定 `JSON` 属性的名称，当方法名和 `JSON` 属性名称不一致时使用。

```java
public class Person {
    private String fullName;
    
    private Integer age;
    
    @JsonGetter("name")
    public String getFullName() {
        return fullName;
    }
    
    // getters and setters
}
```

### @JsonAnySetter

用于指定反序列化时使用的方法，用于处理 `JSON` 中未知的属性

```java
public class Person {
    private String fullName;
    
    private Map<String, Object> otherProperties = new HashMap<>();
    
    @JsonAnySetter
    public void setOtherProperties(String key, Object value) {
        otherProperties.put(key, value);
    }
    
    // getters and setters
}

ObjectMapper mapper = new ObjectMapper();
Person person = mapper.readValue("{\"fullName\":\"John Smith\",\"age\":30}", Person.class);
System.out.println(person.getFullName());
// John Smith
System.out.println(person.getOtherProperties());
// {age=30}
```

### @JsonAnyGetter

用于指定序列化时使用的方法，用于处理 Java 对象中未知的属性

```java
public class Person {
    private String fullName;
    
    private Map<String, Object> otherProperties = new HashMap<>();
    
    public void addOtherProperty(String key, Object value) {
        otherProperties.put(key, value);
    }
    
    @JsonAnyGetter
    public Map<String, Object> getOtherProperties() {
        return otherProperties;
    }
    
    // getters and setters
}

Person person = new Person();
person.setFullName("John Smith");
person.addOtherProperty("age", 30);

ObjectMapper mapper = new ObjectMapper();
String json = mapper.writeValueAsString(person);
// {"fullName":"John Smith","age":30}
```

### @JsonTypeInfo

用于指定 Java 对象在序列化和反序列化时的类型信息，常用的参数有：

- `use`：用于指定类型信息的使用方式，可选的取值有 `JsonTypeInfo.Id.CLASS` (使用 Java 类的全限定名)、`JsonTypeInfo.Id.NAME` (使用名称)和 `JsonTypeInfo.Id.NONE` (不使用类型信息)。
- `include`：用于指定类型信息的包含方式，可选的取值有 `JsonTypeInfo.As.PROPERTY` (作为 JSON 属性) 和 `JsonTypeInfo.As.EXTERNAL_PROPERTY` (作为外部属性)。
- `property`：用于指定包含类型信息的属性名，当 include 的值为 `JsonTypeInfo.As.PROPERTY` 时使用。
- `visible`：用于指定类型信息是否可见。

```java
@JsonTypeInfo(use = JsonTypeInfo.Id.NAME, include = JsonTypeInfo.As.PROPERTY, property = "type")
@JsonSubTypes({
    @JsonSubTypes.Type(value = Rectangle.class, name = "rectangle"),
    @JsonSubTypes.Type(value = Circle.class, name = "circle")
})
public abstract class Shape {
    // ...
}

public class Rectangle extends Shape {
    // ...
}

public class Circle extends Shape {
    // ...
}

Shape shape = new Rectangle();

ObjectMapper mapper = new ObjectMapper();
String json = mapper.writeValueAsString(shape);
// {"type":"rectangle"}

Shape shape2 = mapper.readValue(json, Shape.class);
System.out.println(shape2.getClass().getSimpleName());
// Rectangle
```

## Jackson 基本使用

**添加依赖**：

```xml
<dependencies>
  <!-- Jackson core -->
  <dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-core</artifactId>
    <version>2.13.0</version>
  </dependency>
  <!-- Jackson databind -->
  <dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.13.0</version>
  </dependency>
  <!-- Jackson annotations -->
  <dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-annotations</artifactId>
    <version>2.13.0</version>
  </dependency>
</dependencies>
```

**创建Java对象模型**：在使用Jackson之前，需要创建一个Java对象模型，该模型表示要序列化和反序列化的JSON数据

```java
public class Person {
    private String name;
    private int age;

    public Person() {
    }

    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }
}
```

**使用ObjectMapper进行序列化和反序列化**：使用ObjectMapper类，可以轻松地将Java对象序列化为JSON字符串以及将JSON字符串反序列化为Java对象

- 序列化：

  ```java
  import com.fasterxml.jackson.databind.ObjectMapper;
  
  public class Main {
      public static void main(String[] args) {
          ObjectMapper objectMapper = new ObjectMapper();
          Person person = new Person("Alice", 30);
  
          try {
              String jsonString = objectMapper.writeValueAsString(person);
              System.out.println(jsonString); // 输出：{"name":"Alice","age":30}
          } catch (Exception e) {
              e.printStackTrace();
          }
      }
  }
  ```

- 反序列化：

  ```java
  import com.fasterxml.jackson.databind.ObjectMapper;
  
  public class Main {
      public static void main(String[] args) {
          ObjectMapper objectMapper = new ObjectMapper();
          String jsonString = "{\"name\":\"Alice\",\"age\":30}";
  
          try {
              Person person = objectMapper.readValue(jsonString, Person.class);
              System.out.println("Name: " + person.getName() + ", Age: " + person.getAge()); // 输出：Name: Alice, Age: 30
          } catch (Exception e) {
              e.printStackTrace();
          }
      }
  }
  ```

## 自定义序列化和反序列化

创建自定义序列化器和反序列化器以自定义特定字段或类的序列化和反序列化行为。为此，创建一个实现`JsonSerializer`或`JsonDeserializer`接口的类，并在需要自定义的字段或类上使用`@JsonSerialize`和`@JsonDeserialize`注解。

```java
public class CustomDateSerializer extends JsonSerializer<Date> {
    private SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd");

    @Override
    public void serialize(Date value, JsonGenerator gen, SerializerProvider serializers) throws IOException {
        gen.writeString(dateFormat.format(value));
    }
}
public class Person {
    private String name;

    @JsonSerialize(using = CustomDateSerializer.class)
    private Date birthdate;

    // ...其他代码...
}
```

## 使用JsonNode进行动态解析

可以使用`JsonNode`类来动态地解析和操作`JSON`数据

```java
String jsonString = "{\"name\":\"Alice\",\"age\":30,\"address\":{\"street\":\"Main St\",\"city\":\"New York\"}}";

ObjectMapper objectMapper = new ObjectMapper();
JsonNode rootNode = objectMapper.readTree(jsonString);

String name = rootNode.get("name").asText(); // Alice
int age = rootNode.get("age").asInt(); // 30
String street = rootNode.get("address").get("street").asText(); // Main St
```

## 处理日期和时间类型

`Jackson`可以处理Java日期和时间类型，例如java.util.Date和Java 8时间库中的类型。可以通过配置`ObjectMapper`来指定日期和时间格式

```java
ObjectMapper objectMapper = new ObjectMapper();
objectMapper.setDateFormat(new SimpleDateFormat("yyyy-MM-dd"));
```

## 处理泛型

`Jackson`可以处理泛型类型，例如`List<T>`和`Map<String, T>`。在反序列化时，需要使用`TypeReference`来指定泛型类型

```java
String jsonString = "[{\"name\":\"Alice\",\"age\":30},{\"name\":\"Bob\",\"age\":25}]";

ObjectMapper objectMapper = new ObjectMapper();
List<Person> persons = objectMapper.readValue(jsonString, new TypeReference<List<Person>>() {});
```

