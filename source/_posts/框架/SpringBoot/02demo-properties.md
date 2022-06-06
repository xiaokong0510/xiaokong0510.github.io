---
title: 02 SpringBoot 配置文件
categories:
  - 框架
  - SpringBoot 
tags:
  - SpringBoot 
  - yml
date: 2021-07-18
description: SpringBoot 是如何获取配置文件的
---

SpringBoot 默认使用两种全局的配置文件，全局配置文件可以对一些默认配置进行修改。

- application.properties
- application.yml

例：

```yaml
user:
  username: zhangsan@dev
  age: 20
  lists: [cat,dog,pig]
  maps: {k1: v1,k2: 12}

developer:
  name: kongxiao@dev
  phone-number: xxxxxx@dev
```

定义好配置属性对象后，有以下两种方式进行配置注入


## 方式一：@Value 注入属性

- Spring 支持 `@value` 注解的方式来读取配置文件中的配置值；

- 添加 `@Component` 注解将配置类注入 Spring 容器中。

- 支持 List、Map 等绑定。

```java
@Data
@Component
public class DeveloperProperty {
    /**
     * 通过配置文件注入
     */
    @Value("${developer.name}")
    private String name;
    @Value("${developer.phone-number}")
    private String phoneNumber;
}
```

## 方式二：@ConfigurationProperties

步骤：

1. 定义配置类；
2. 使用 `@ConfigurationProperties` 注解也可以获取配置文件的值：

- `prefix` 前缀定义了哪些外部属性将绑定到类的字段上；
- 类的属性名称必须与外部属性的名称匹配；
- 可以简单地用一个值初始化一个字段来定义一个默认值； 
- 类的字段必须有公共 setter 方法
- Spring Boot 具备宽松绑定规则，参考 https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/htmlsingle/#boot-features-external-config-relaxed-binding

```java
@Data
@ConfigurationProperties(prefix = "user")
public class UserInfoProperty {

    private String username;
    private Integer age;
    private List<Object> lists;
    private Map<String, Object> maps;
}
```

3. 使用 `@EnableConfigurationProperties({UserInfoProperty.class})`启用该配置

```java
@RestController
@EnableConfigurationProperties(UserInfoProperty.class)
public class PropertiesController {

    private final DeveloperProperty developerProperty;

    private final UserInfoProperty userInfoProperty;

    public PropertiesController(DeveloperProperty developerProperty, UserInfoProperty userInfoProperty) {
        this.developerProperty = developerProperty;
        this.userInfoProperty = userInfoProperty;
    }

    @RequestMapping("/property")
    public Dict getProperties() {
        return Dict.create().set("userInfoProperty", userInfoProperty).set("developerProperty", developerProperty);
    }

}
```



也可以直接在配置类上添加 `@Component` 注解将配置类注入 Spring 容器中使用



如果添加了如下依赖，可以从@ConfigurationProperties 注释的项生成的配置元数据文件

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-configuration-processor</artifactId>
	<optional>true</optional>
 </dependency>
```

![image-20210929114212357](https://image.kongxiao.top/20210929114213.png)

## 多环境配置

在Spring Boot中多环境配置文件名需要满足`application-{profile}.properties`的格式，其中`{profile}`对应环境标识，比如：

- `application-dev.properties`：开发环境
- `application-test.properties`：测试环境
- `application-prod.properties`：生产环境

需要在`application.properties`文件中通过`spring.profiles.active`属性来设置`{profile}`的值，决定哪个具体的配置文件会被加载，其值对应

例如：

```yaml
server:
  port: 8081
spring:
  profiles:
    active: prod
```

也可以通过命令行的方式：

`java -jar xxx.jar --spring.profiles.active=dev`

![image-20210929105846792](https://image.kongxiao.top/20210929105847.png)

## 更多操作

### 参数间的引用

在`application.properties`中的各个参数之间也可以直接引用，通过 `${xxx}` 即可

```yaml
blog:
  name: Spring Boot
  title: 配置文件
  desc: ${blog.name}的标题是${blog.title}

```

### 使用随机数

在一些情况下，有些参数希望它不是一个固定的值，比如密钥、服务端口等。Spring Boot的属性配置文件中可以通过`${random}`来产生 int 值、long 值或者 string 字符串，来支持属性的随机值。

-  随机字符串：  `${random.value}`
-  随机 int：  `${random.int}`
-  随机 long：  `${random.value}`
-  10以内的随机数：  `{random.int(10)}`
-  10-20的随机数：  `${random.int[10,20]}`

### 定制启动 banner

在resourece 目录下新建 banner.txt，即可更改项目启动时的 banner

![image-20210929111522875](https://image.kongxiao.top/20210929111523.png)

```txt
   _
  | |__   __ __
  | / /   \ \ /
  |_\_\   /_\_\
_|"""""|_|"""""|
"`-0-0-'"`-0-0-'

```

