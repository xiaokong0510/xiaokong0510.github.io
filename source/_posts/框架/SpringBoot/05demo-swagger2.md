---
title: 05 SpringBoot 集成 Swagger2
categories:
  - 框架
  - SpringBoot 
tags:
  - SpringBoot 
  - Swagger2
date: 2021-07-28
description: 一键生成接口文档，简单学习下，实际用的比较少。
---

**Swagger2** 是一个规范和完整的框架，可以用于生成、描述、调用和可视化 RESTful 风格的 Web 服务：

1. 接口文档在线自动生成，文档随接口变动实时更新，节省维护成本
2. 支持在线接口测试，不依赖第三方工具

官网：https://swagger.io/

在线 DEMO：https://petstore.swagger.io/#/



顺便一提，目前公司使用的是 YApi 进行接口文档管理平台，每次迭代开发后，都是手动进行接口新增、修改，有点低效。

官网：https://hellosean1025.github.io/yapi/index.html

仓库：https://github.com/ymfe/yapi

## 添加 Swagger2 依赖 

```xml
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger2</artifactId>
    <version>2.9.2</version>
</dependency>
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger-ui</artifactId>
    <version>2.9.2</version>
</dependency>
```

## 创建 Swagger2 配置类

```java
@Configuration
public class Swagger2Config {
    /**
     * 构建一个DocketBean
     *
     * @return
     */
    @Bean
    public Docket createRestApi() {
        return new Docket(DocumentationType.SWAGGER_2)
                // 调用apiInfo方法,创建一个ApiInfo实例,里面是展示在文档页面信息内容
                .apiInfo(apiInfo())
                .select()
                // 控制暴露出去的路径下的实例
                // 如果某个接口不想暴露,可以使用注解 @ApiIgnore
                .apis(RequestHandlerSelectors.basePackage("com.xiao.swagger2.controller"))
                .paths(PathSelectors.any())
                .build();
    }

    /**
     * 构建 api文档的详细信息函数
     *
     * @return
     */
    private ApiInfo apiInfo() {
        return new ApiInfoBuilder()
                // 页面标题
                .title("Spring Boot Swagger2 构建restful API")
                // 条款地址
                .termsOfServiceUrl("https://www.kongxiao.top")
                // 创建人
                .contact(new Contact("kongxiao","https://www.kongxiao.top","xxx@163.com"))
                // 版本号
                .version("1.0")
                // 描述
                .description("API 描述")
                .build();
    }
}
```

- `createRestApi`函数创建`Docket`的Bean
- `apiInfo()`用来创建该Api的基本信息，这些基本信息会展现在文档页面中
- `select()`函数返回一个`ApiSelectorBuilder`实例用来控制哪些接口暴露给Swagger来展现

## 添加注解

使用注解来生成文档内容。

常用注解汇总如下：

| 作用范围           | API                | 使用位置                         |
| :----------------- | :----------------- | :------------------------------- |
| 对象属性           | @ApiModelProperty  | 用在出入参数对象的字段上         |
| 协议集描述         | @Api               | 用于controller类上               |
| 协议描述           | @ApiOperation      | 用在controller的方法上           |
| Response集         | @ApiResponses      | 用在controller的方法上           |
| Response           | @ApiResponse       | 用在 @ApiResponses里边           |
| 非对象参数集       | @ApiImplicitParams | 用在controller的方法上           |
| 非对象参数描述     | @ApiImplicitParam  | 用在@ApiImplicitParams的方法里边 |
| 描述返回对象的意义 | @ApiModel          | 用在返回对象类上                 |

### @API

**使用场景：** 在 Rest 接口类上使用，标记类为 Swagger 资源类，运行时有效

**属性：**

- `tags`： 对接口进行分组；
- `value`： 在UI界面上不显示；
- `produces`： content types，例如："application/json, application/xml"；

生成的 api 文档会根据 tags 分类，这个 controller 中的所有接口生成的接口文档都会在 tags 这个 list 下；tags 如果有多个值，会生成多个 list，每个 list 都显示所有接口

```java
@Api(tags = "列表1")
@Api(tags = {"列表1","列表2"})
```

### @ApiOperation

**使用场景：** 使用于在方法上，表示一个 http 请求的操作；具有相同路径的不同操作会被归组为同一个操作对象

**属性：**

- `value`： 用于方法描述
- `notes`： 用于提示内容

### @ApiModel

**使用场景：** 用于响应实体类上，用于说明实体作用

**属性：**

- `value`： 实体值
- `description`： 描述实体的作用

### @ApiModelProperty

**使用场景：**用在属性上，描述实体类的属性

**属性：**

- `value`： 描述参数的意义
- `name`： 参数的变量名
- `required=true`：参数是否必选

### @ApiImplicitParams

**使用场景：** 用在请求的方法上，包含多 `@ApiImplicitParam`

**属性：**

- `value`： 用于方法描述
- `description`： 描述实体的作用

### @ApiImplicitParam

**使用场景：** 用于方法上，表示单独的请求参数

**属性：**

- `name`：参数名

- `value`： 参数说明
- `dataType`：数据类型
- `paramType`： 表示参数放在哪里，包括
  -  header 请求参数的获取：@RequestHeader
  - query   请求参数的获取：@RequestParam
  -  path（用于restful接口） 请求参数的获取：@PathVariable
  - body
  - form

- `defaultValue`：参数默认值
- `required=true`：参数是否必选

### @ApiResponses

**使用场景：** 用于请求的方法上，根据响应码表示不同响应。一个`@ApiResponses`包含多个`@ApiResponse`

### @ApiResponse

**使用场景：**用在请求的方法上，表示不同的响应

**属性：**

- `code`： 表示响应码(int型)，可自定义
- `message`： 状态码对应的响应信息

## RESTful API 文档

### 统一返回格式

```java
@Data
@ApiModel(value = "CommonResult",description = "统一返回结果")
public class CommonResult<T> {
    @ApiModelProperty(value = "成功标识；true：成功；false:失败")
    private boolean success;
    @ApiModelProperty(value = "返回状态码；200:成功")
    private int code;
    @ApiModelProperty(value = "描述信息")
    private String msg;
    @ApiModelProperty(value = "数据")
    private T data;

    public static <T> CommonResult<T> success(T data) {
        CommonResult<T> r = new CommonResult<>();
        r.setCode(200);
        r.setData(data);
        return r;
    }

    public static <T> CommonResult<T> error(String msg) {
        CommonResult<T> r = new CommonResult<>();
        r.setCode(500);
        r.setMsg(msg);
        return r;
    }
}
```

### RESTFul风格API

![img](https://image.kongxiao.top/20211018113252.png)

```java
@RestController
@RequestMapping("/users")
@Api(tags = "User",produces = "application/json")
public class UserController {

    static Map<Integer, User> users = Collections.synchronizedMap(new HashMap<>());

    @ApiOperation(value = "获取用户列表",notes = "获取所有的用户信息列表")
    @ApiResponses({@ApiResponse(code = 200, message = "操作成功"),
            @ApiResponse(code = 500, message = "服务器内部异常"),
            @ApiResponse(code = 401, message = "权限不足")})
    @GetMapping
    public CommonResult<List<User>> list() {
        List<User> userList = new ArrayList<>(UserController.users.values());
        return CommonResult.success(userList);
    }

    @ApiOperation(value = "获取单个用户信息", notes = "根据用户id查询")
    @ApiImplicitParam(name = "id", value = "用户id", required = true, dataType = "Integer")
    @GetMapping(value = "/{id}")
    public User getUserById(@PathVariable("id") Integer id) {
        return users.get(id);
    }

    @ApiOperation(value = "创建用户")
    @ApiImplicitParam(name = "user", value = "用户详细信息", required = true, dataType = "User")
    @PostMapping
    public User createUser(@RequestBody User user) {
        users.put(user.getId(), user);
        return user;
    }

    @ApiOperation(value = "修改用户信息")
    @ApiImplicitParams({
            @ApiImplicitParam(name = "user", value = "用户详细信息", required = true, dataType = "User"),
            @ApiImplicitParam(name = "id", value = "用户id", required = true, dataType = "Integer")
    })
    @PutMapping("/{id}")
    public User updateUser(@RequestBody User user, @PathVariable("id") Integer id) {
        User u = users.get(id);
        u.setUsername(user.getUsername());
        u.setAge(user.getId());
        users.put(user.getId(), user);
        return user;
    }

    @ApiOperation(value = "删除单个用户信息", notes = "根据用户id删除")
    @ApiImplicitParam(name = "id", value = "用户id", required = true, dataType = "Integer")
    @DeleteMapping(value = "/{id}")
    public User deleteUserById(@PathVariable("id") Integer id) {
        users.remove(id);
        return users.get(id);
    }

}
```

### 生成API文档

访问文档地址：http://localhost:8084/swagger-ui.html#/
