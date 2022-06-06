---
title: 03 SpringBoot JSR-303 接口参数校验
categories:
  - 框架
  - SpringBoot 
tags:
  - SpringBoot 
  - Validation
date: 2021-07-23
description: 学会优雅的进行接口参数校验
---

JSR 是 Java Specification Requests 的缩写，意思是 Java 规范提案。

JSR-303 是 JAVA EE 6 中的一项子规范，叫做Bean Validation，Hibernate Validator 是 Bean Validation 的参考实现

## 1 问题引出

先来看看平时写的代码存在的一些问题：

```java
@RequestMapping("/addUser")
public CommonResult addUser(UserInfo userInfo ) {
    if(userInfo .getAge() == null){
        return "年龄不能为空";
    }
    if(userInfo .getAge() > 120){
        return "年龄不能超过120";
    }
    if(userInfo .getName().isEmpty()){
        return "用户名不能为空";
    }
    // 省略一堆参数校验...
    //开始业务逻辑...
    return processSuccess();
}
```

> 正常的业务逻辑还没有开始，参数校验代码就写了一堆，而且每个接口中都要写一些类似的代码，既不优雅，又不专业。

所以我们要做的是：

- 消灭不优雅的代码，使用 Spring 框架封装的校验组件：`validation`
- 对于请求参数格式错误造成的异常进行处理，即使传参错误，程序也能按预期处理响应，减少无必要的告警。

`validation` 依赖

```xml
<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-validation</artifactId>
</dependency>
```

在Spring Boot 2.1版本中，该依然其实已经包含在了spring-boot-starter-web依赖中，并不需要额外引入

## 2 几个案例

先定义好通用返回消息实体类。

```java
@Data
public class CommonResult {

    private int code;
    private Object data;
    private String msg;

    public static CommonResult success(Object data, String msg) {
        CommonResult r = new CommonResult();
        r.setCode(200);
        r.setData(data);
        r.setMsg(msg);
        return r;
    }

    public static CommonResult error( String msg) {
        CommonResult r = new CommonResult();
        r.setCode(-1);
        r.setMsg(msg);
        return r;
    }
}
```

下面总结了需要进行参数校验的几种情况：

### 2.1 Query 参数缺失

 ```java
 @GetMapping("/test")
 public CommonResult test(@RequestParam("age") Integer age) {
     return CommonResult.success(null, "操作成功！");
 }
 ```

对于如上接口，请求参数添加注解：`@RequestParam("age")`

如果调用接口时，未携带参数age，例如 http://127.0.0.1:8080/test，则会返回 400 异常页面**MissingServletRequestParameterException：Required Integer parameter 'age' is not present**

```json
{
    "timestamp": "2021-09-01T08:24:14.372+0000",
    "status": 400,
    "error": "Bad Request",
    "message": "Required Integer parameter 'age' is not present",
    "path": "/test"
}
```



为返回友好提示，添加一个 **全局异常处理类** ，针对  `MissingServletRequestParameterException` 异常进行处理，以自定义返回消息进行返回：

```java
@Component
@ControllerAdvice
@Slf4j
public class GlobalExceptionHandler {

    @Autowired
    private HttpServletRequest request;

    /**
     * 处理请求参数缺失异常
     * @param e
     * @return
     */
    @ExceptionHandler(value = MissingServletRequestParameterException.class)
    @ResponseBody
    public CommonResult exceptionHandler(Exception e){
        log.warn("异常：MissingServletRequestParameterException，原因: {},请求方式: {},请求url: {},请求参数: {}",
                e.getMessage(),request.getMethod(), request.getRequestURI(),request.getQueryString());
        return CommonResult.error(e.getMessage());
    }
}
```

再次请求返回消息：

```json
{
    "code": -1,
    "data": null,
    "msg": "Required Integer parameter 'age' is not present"
}
```

### 2.2 参数类型不正确

访问链接：http://127.0.0.1:8080/test?age=ddd

对于 Integer 类型的请求参数 age ，如果参数格式不对，如携带一个字符串进行请求，会返回 400 异常页面 **MethodArgumentTypeMismatchException：Failed to convert value of type 'java.lang.String' to required type 'java.lang.Integer'**

```json
{
    "timestamp": "2021-09-01T08:25:01.197+0000",
    "status": 400,
    "error": "Bad Request",
    "message": "Failed to convert value of type 'java.lang.String' to required type 'java.lang.Integer'; nested exception is java.lang.NumberFormatException: For input string: \"ddd\"",
    "path": "/test"
}
```



同样的，在 **全局异常处理类中添加处理 MethodArgumentTypeMismatchException的方法：**

```java
@ExceptionHandler(MethodArgumentTypeMismatchException.class)
@ResponseBody
public CommonResult mismatchErrorHandler(MethodArgumentTypeMismatchException e, HttpServletRequest request) {
    log.warn("异常：MissingServletRequestParameterException，原因: {},请求方式: {},请求url: {},请求参数: {}",
            e.getMessage(),request.getMethod(), request.getRequestURI(),request.getQueryString());
    return CommonResult.error( e.getMessage());
}
```

### 2.3 处理 query 参数校验

校验 `@RequestParam` 类型的请求参数：

- 首先需要添加 `@Validated` 注解来启用控制器中的 `@RequestParams` 的验证。

- 在请求参数中增加 **校验规则注解**

任何与这些条件不匹配的请求都将返回 HTTP 状态 500 ，并显示默认错误消息。

```java
@RestController
@Validated
public class UserController {

    @GetMapping("/test02")
    public CommonResult test02(@RequestParam("age") @NotNull(message = "age不能为空") Integer age) {
        return CommonResult.success(null, "操作成功！");
    }
}
```

`@RequestParam` 上 validate 失败后抛出的异常是：**ConstraintViolationException**

在全局异常处理类中新增处理 ConstraintViolationException 的方法。

```java
/**
 * 处理请求参数格式错误 @RequestParam上validate失败后抛出的异常是javax.validation.ConstraintViolationException
 * @param e
 * @return
 */
@ExceptionHandler(ConstraintViolationException.class)
@ResponseBody
public CommonResult constraintViolationExceptionHandler(ConstraintViolationException e) {
    String message = e.getConstraintViolations().stream().map(ConstraintViolation::getMessage).collect(Collectors.joining(";"));
    return CommonResult.error(message);
}
```

调用 http://127.0.0.1:8080/test02?age=

在调用接口时，如果 age 为 null ，则会返回注解 @NotNull(message = "age不能为空") 中的自定义的 message提示信息。

```json
{
    "code": -1,
    "data": null,
    "msg": "age不能为空"
}
```



>  如果 GET 请求的接口的请求参数用一个实体类接收，则需要将 **校验规则注解标注在实体类对于的成员变量上**，此时校验不通过抛出的异常是  **BindException **

```java
@Data
public class UserInfo {
    @NotNull(message = "userName不能为空")
    private String userName;
    @NotNull
    @Max(value = 10, message = "age最大值为10")
    @Min(value = 1, message = "age最小值为1")
    private Integer age;
}
```

接口，在实体类参数前加上注解 `@Valid` 或者 `@Validated`

```java
@GetMapping("/test03")
public CommonResult test03(@Valid UserInfo userInfo) {
    return CommonResult.success(null, "操作成功！");
}
```

访问  http://127.0.0.1/test03?userName= 直接返回了异常信息：

```json
{
    "timestamp": "2021-09-01T08:26:53.978+0000",
    "status": 400,
    "error": "Bad Request",
    "errors": [
        {
            "codes": [
                "NotNull.userInfo.age",
                "NotNull.age",
                "NotNull.java.lang.Integer",
                "NotNull"
            ],
            "arguments": [
                {
                    "codes": [
                        "userInfo.age",
                        "age"
                    ],
                    "arguments": null,
                    "defaultMessage": "age",
                    "code": "age"
                }
            ],
            "defaultMessage": "不能为null",
            "objectName": "userInfo",
            "field": "age",
            "rejectedValue": null,
            "bindingFailure": false,
            "code": "NotNull"
        }
    ],
    "message": "Validation failed for object='userInfo'. Error count: 1",
    "path": "/test03"
}
```

### 2.4 处理 Body 请求体验

```java
@PostMapping("/test04")
public CommonResult test04(@RequestBody @Valid UserInfo userInfo) {
    return CommonResult.success(null, "操作成功！");
}
```

对于携带 RequestBody 类型的请求，

@RequestBody 上 validate 失败后抛出的异常是 **MethodArgumentNotValidException** 异常。

在全局异常处理类中添加上对于此类型异常的处理即可。

## 3 内置的校验注解

内置的校验注解有很多，罗列如下：

| **注解**         | **校验功能**                       |
| ---------------- | ---------------------------------- |
| @AssertFalse     | 必须是false                        |
| @AssertTrue      | 必须是true                         |
| @DecimalMax      | 小于等于给定的值                   |
| @DecimalMin      | 大于等于给定的值                   |
| @Digits          | 可设定最大整数位数和最大小数位数   |
| @Email           | 校验是否符合Email格式              |
| @Future          | 必须是将来的时间                   |
| @FutureOrPresent | 当前或将来时间                     |
| @Max             | 最大值                             |
| @Min             | 最小值                             |
| @Negative        | 负数（不包括0）                    |
| @NegativeOrZero  | 负数或0                            |
| @NotBlank        | 不为null并且包含至少一个非空白字符 |
| @NotEmpty        | 不为null并且不为空                 |
| @NotNull         | 不为null                           |
| @Null            | 为null                             |
| @Past            | 必须是过去的时间                   |
| @PastOrPresent   | 必须是过去的时间，包含现在         |
| @PositiveOrZero  | 正数或0                            |
| @Size            | 校验容器的元素个数                 |

## 4 分组校验

直接定义在JavaBean 中的校验注解，在不同场景下，每个 Controller 方法对该 JavaBean 都具有不同的校验规则。

比如：新注册用户还没起名字，允许`name`字段为空，但是不允许将名字更新为空字符。

此时需要用到 **分组校验，将不同的校验规则分给不同的组，在使用时，指定不同的校验规则**



步骤：

1. 定义一个分组接口；

```java
//校验分组 Update
public interface  Update{ 
    //接口中不需要定义任何方法，仅对不同的校验规则进行分组 
} 
```

2. 在校验注解上添加 `groups` 属性指定分组；

```java
@Data
public class UserInfo {
    @NotEmpty(message = "userName不能为空",groups = Update.class)
    private String userName;
    
    @NotNull
    @Max(value = 10, message = "age最大值为10")
    @Min(value = 1, message = "age最小值为1")
    private Integer age;
}
```

3. `Controller` 方法的 `@Validated` 注解中添加一个 value 值，该 value 值指定校验规则所在的接口；此时则只会校验指定分组下的字段

```java
@GetMapping("/test05")
public CommonResult test05(@RequestBody @Validated(value = Update.class) UserInfo userInfo) {
    return CommonResult.success(null, "操作成功！");
}
```

**注意：**自定义的分组接口可以继承`Default`接口。校验注解和`@validated`默认都属于`Default.class`分组。

## 5 嵌套属性校验

如果 UserInfo 类中增加一个 OrderInfo 类的属性，而 OrderInfo中的属性也需要校验，就用到递归校验了，只要在相应属性上增加`@Valid` 注解即可实现

## 6 自定义校验

可以参照 @NotEmpty 注解；步骤如下：

1. 自定义校验注解，通过`@Constraint`来指明使用哪个类进行校验；
2. 编写自定义校验的逻辑实体类，这个类必须实现 ConstraintValidator 这个接口，这样才可以被注解用来校验。 
3. 编写具体的校验逻辑。

```java
@Target({METHOD, FIELD, ANNOTATION_TYPE, CONSTRUCTOR, PARAMETER})
@Retention(RUNTIME)
@Documented
//指定了当前注解使用哪个类来进行校验。
@Constraint(validatedBy = {HaveNoBlankValidator.class})
public @interface HaveNoBlank {
 
    // 校验出错时默认返回的消息
    String message() default "字符串中不能含有空格";
    // default 关键字 接口中被default修饰的方法，在类实现这个接口时不必必须实现这个方法
    Class<?>[] groups() default { };
    // Class<?> 表示不确定的java类型 
    // Class<T> 表示java类型 
    // Class<K,V> 分别代表java键值中的key value 
    // Class<E> 代表Element
    Class<? extends Payload>[] payload() default { };

    /**
     * 同一个元素上指定多个该注解时使用
     */
    @Target({ METHOD, FIELD, ANNOTATION_TYPE, CONSTRUCTOR, PARAMETER, TYPE_USE })
    @Retention(RUNTIME)
    @Documented
    public @interface List {
        NotBlank[] value();
    }
}
```



```java
public class HaveNoBlankValidator implements ConstraintValidator<HaveNoBlank, String> {
    @Override
    public boolean isValid(String value, ConstraintValidatorContext context) {
        // null 不做检验
        if (value == null) {
            return true;
        }
        if (value.contains(" ")) {
            // 校验失败
            return false;
        }
        // 校验成功
        return true;
    }
}
```

## 7 小结

在全局异常处理类中，我们可以写多个异常处理方法，几种参数校验时可能引发的异常：

1. Query 参数校验异常抛出 `ConstraintViolationException`

2. 使用实体类接收Query 参数，或者使用 form data 方式传参，校验异常抛出 `BindException`
3. 使用 Body 参数校验异常抛出  `MethodArgumentNotValidException`



**校验 `@RequestParam`单个请求参数时：**

- 首先需要在类上添加`@Validated`注解来启用控制器中的 @RequestParams 的验证。
- 在请求参数中增加**校验规则注解**



**校验 `@RequestBody` 请求体参数时：**

- 在请求参数实体类中的成员变量上增加 **校验规则注解**
- 在控制器方法的请求参数中加上`@Valid` 或者`@Validated`  注解
- 如果用到分组校验、嵌套属性校验时，需要使用`@Validated`

