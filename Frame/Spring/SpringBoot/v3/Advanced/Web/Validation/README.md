# 参数校验

在项目里面，很有可能用户发送的数据存在一些问题，所以需要对前端传入的参数做一个简单的简单的校验，**避免出现脏数据和业务逻辑错误**。如果每个接口单独写校验逻辑的话，需要在controller层做逻辑判断。参数较少时，还勉强能够接受，如果参数和接口较多，无形中加重了工作量，也多了很多重复代码。所以引入注解式参数校验很有必要。

SpringBoot提供了很方便的接口校验框架来解决上述问题。

## 导入依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
</dependency>
```

## 使用注解完成接口校验

直接使用注解完成全部接口的校验：

```java
@Validated   //首先在Controller上开启接口校验
@Controller
public class TestController {

    ...

    @ResponseBody
    @PostMapping("/submit")
    public String submit(@Length(min = 3,message = "少于3位") String username,  //使用@Length注解判断长度，message自定义提示信息
                         @Length(min = 10) String password){
        System.out.println(username.substring(3));
        System.out.println(password.substring(2, 10));
        return "请求成功!";
    }
}
```

效果：

![image-20240417230841348](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202404172308444.png)

## 对象类型校验

对于对象类型接收前端发送的表单数据的，可以校验参数中的每个属性进行验证：

```java
@ResponseBody
@PostMapping("/submit")  //在参数上添加@Valid注解表示需要验证
public String submit(@Valid Account account){
    System.out.println(account.getUsername().substring(3));
    System.out.println(account.getPassword().substring(2, 10));
    return "请求成功!";
}
```

```java
@Data
public class Account {
    @Length(min = 3)   //只需要在对应的字段上添加校验的注解即可
    String username;
    @Length(min = 10)
    String password;
}
```

## 常用注解

![](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202404172339378.png)

## 异常捕获器的作用

### MissingServletRequestParameterException

加了`@RequestParam`注解，但是接口调用时没有传指定的参数（注意：是没有传，而不是传了，但是值是null）。

### MethodArgumentNotValidException

当校验的参数放在对象中，接口的请求方式是post请求，用`@Valid` `@RequestBody`方式接受参数时，如果报错，会被该捕获器捕获。

### BindException

当校验参数写在类中，接口请求方式是get请求时，报错会被该捕获器捕获。

### ConstraintViolationException

传了值，但是不符合要求。`@NotNull(message = "最大值不能为空")`、`@Min(value = 10,message = "参数必须大于10")`，要求传非null值，且值必须大于10，否则会返回错误信息。经过测试，当校验参数直接写在接口上，而不是写在类中，报错会被该捕获器捕获。

## 全局异常捕获

自定义接口响应类：

```java
@Data
public class ResponseVO<T> implements Serializable {
    // 状态码: 0-成功，其他-失败
    private final Integer code;
    // 返回信息
    private final String message;
    //返回值
    private final T data;
    //是否成功
    private final Boolean success;
    // 成功返回
    public static <T> ResponseVO<T> success(T data) {
        return new ResponseVO<>(data);
    }
    // 失败返回
    public static <T> ResponseVO<T> error(Integer code, String message, T data) {
        return new ResponseVO<>(code, message, data);
    }
    public ResponseVO(T data) {
        this.code = ResponseConstant.SUCCESS_CODE;
        this.message = ResponseConstant.OK;
        this.data = data;
        this.success = true;
    }
    public ResponseVO(Integer code, String message, T data) {
        this.code = code;
        this.message = message;
        this.data = data;
        this.success = code == ResponseConstant.SUCCESS_CODE;
    }
}
```

自定义常量类：

```java
public class ResponseConstant {
    public static final String OK = "OK";
    public static final String ERROR = "error";
    public static final int SUCCESS_CODE = 200;
    public static final int ERROR_CODE = 500;
    public static final String ERROR_MESSAGE = "操作失败！！";
}
```

全局异常捕获：

```java
@Slf4j
@RestControllerAdvice
public class GlobalExceptionHandler {
	//捕获器1
    @ExceptionHandler(value = {MissingServletRequestParameterException.class})
    public ResponseVO<String> handleMissingServletRequestParameterException(MissingServletRequestParameterException ex) {
        if (log.isErrorEnabled()) {
            log.error(ex.getMessage(), ex);
        }
        return ResponseVO.error(ResponseConstant.ERROR_CODE, String.format("缺少必要参数[%s]", ex.getParameterName()), "");
    }
    //捕获器2
    @ExceptionHandler(value = {MethodArgumentNotValidException.class})
    public ResponseVO<String> handleMethodArgumentNotValidException(MethodArgumentNotValidException ex) {
        if (log.isErrorEnabled()) {
            log.error(ex.getMessage(), ex);
        }
        BindingResult result = ex.getBindingResult();
        FieldError error = result.getFieldError();
        return ResponseVO.error(ResponseConstant.ERROR_CODE, null == error ? ResponseConstant.ERROR_MESSAGE : error.getDefaultMessage(), "");
    }
    //捕获器3
    @ExceptionHandler(value = {BindException.class})
    public ResponseVO<String> handleBindException(BindException ex) {
        if (log.isErrorEnabled()) {
            log.error(ex.getMessage(), ex);
        }
        BindingResult result = ex.getBindingResult();
        FieldError error = result.getFieldError();
        return ResponseVO.error(ResponseConstant.ERROR_CODE, null == error ? ResponseConstant.ERROR_MESSAGE : error.getDefaultMessage(), "");
    }
    //捕获器4
    @ExceptionHandler(value = {ConstraintViolationException.class})
    public ResponseVO<String> handleConstraintViolationException(ConstraintViolationException ex) {
        if (log.isErrorEnabled()) {
            log.error(ex.getMessage(), ex);
        }
        Optional<ConstraintViolation<?>> first = ex.getConstraintViolations().stream().findFirst();
        return ResponseVO.error(ResponseConstant.ERROR_CODE, first.isPresent() ? first.get().getMessage() : ResponseConstant.ERROR_MESSAGE, "");
    }
    //其他所有异常捕获器
    @ExceptionHandler(Exception.class)
    public ResponseVO<String> otherErrorDispose(Exception e) {
        // 打印错误日志
        log.error("错误代码({}),错误信息({})", ResponseConstant.ERROR_CODE, e.getMessage());
        e.printStackTrace();
        return ResponseVO.error(ResponseConstant.ERROR_CODE, ResponseConstant.ERROR_MESSAGE, e.getMessage());
    }
}
```