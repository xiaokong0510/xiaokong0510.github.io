---
title: 01 SpringBoot 入门案例 HelloWorld
categories:
  - 框架
  - SpringBoot 
tags:
  - SpringBoot 
date: 2021-07-15
description: 万物皆从 HelloWorld 开始，先学会 SpringBoot 快速开发一个 HelloWorld 接口，抛弃 Spring、SpringMVC 复杂的配置文件
---

## 项目搭建

可以通过 https://start.spring.io/ 来初始化基础项目

![image-20210928164344879](https://image.kongxiao.top/20210928164346.png)

也可以使用 IDEA 中的 Spring Initializr 来快速搭建项目

![image-20210928170132959](https://image.kongxiao.top/20210928170133.png)

Spring Boot的基础结构共三个文件

- `src/main/java`下的程序入口
- `src/main/resources`下的配置文件：`application.properties`
- `src/test/`下的测试入口

## 引入依赖

- `spring-boot-starter`：核心模块，包括自动配置支持、日志和 YAML
- `spring-boot-starter-test`：测试模块，包括 JUnit、Hamcrest、Mockito
- `lombok`
- `hutool-all`

```xml
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
        </dependency>
        <dependency>
            <groupId>cn.hutool</groupId>
            <artifactId>hutool-all</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>
```

## 编写 HelloWorld 服务

启动类：

```java
@SpringBootApplication
@Slf4j
public class HelloWorldApplication {
    public static void main(String[] args) throws UnknownHostException {
        ConfigurableApplicationContext context = SpringApplication.run(HelloWorldApplication.class, args);
        Environment env = context.getBean(Environment.class);
        String port = env.getProperty("server.port");
        String ip = InetAddress.getLocalHost().getHostAddress();
        log.info("\n---------------------------------------------------------\n" +
                "\tApplication is running! Access address: http://{}:{}" +
                "\n---------------------------------------------------------\n", ip, port);
    }
}
```

接口：

```java
@RestController
public class HelloController {
    @RequestMapping("/hello")
    public String hello(@RequestParam(required = false, name = "userName") String userName) {
       if(StrUtil.isBlank(userName)) {
           userName = "World";
       }
        return StrUtil.format("Hello,{}!", userName);
    }
}

```

## 单元测试

```java
@RunWith(SpringJUnit4ClassRunner.class)
@WebAppConfiguration
class HelloWorldApplicationTest {

    private MockMvc mvc;

    @BeforeEach
    public void setUp() {
        mvc = MockMvcBuilders.standaloneSetup(new HelloController()).build();
    }

    @Test
    void getHello() throws Exception {
        mvc.perform(MockMvcRequestBuilders.get("/hello").param("userName", "test")
                .accept(MediaType.APPLICATION_JSON))
                .andExpect(status().isOk())
                .andExpect(content().string(equalTo("Hello,test!")));
    }
}
```

