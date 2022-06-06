---
title: 07 SpringBoot 整合 Mybatis
categories:
  - 框架
  - SpringBoot 
tags:
  - SpringBoot 
  - Mybatis
date: 2021-08-05
description: SpringBoot 整合 Mybatis，丢掉繁琐的 mybatis-config.xml 配置文件
---

MyBatis 官方中文文档：https://mybatis.org/mybatis-3/zh/

GitHub: https://github.com/mybatis/mybatis-3

参考文档：https://blog.didispace.com/spring-boot-learning-21-3-5/

## 1 数据库环境
在数据中 mybatis 中，建表 user：

```sql
CREATE TABLE `user` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `name` varchar(255) DEFAULT NULL COMMENT '用户名',
  `phone_number` int(11) DEFAULT NULL COMMENT '手机号',
  `status` bit(1) DEFAULT NULL COMMENT '状态，1 有效；0 无效',
  `create_time` datetime DEFAULT NULL,
  `update_time` datetime DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```
## 2 引入依赖

- mybatis
- mysql-connector-java

```xml
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>2.1.1</version>
</dependency>

<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
</dependency>
```

关于 mybatis-spring-boot-starter 的版本：
- 2.1.x版本适用于：MyBatis 3.5+、Java 8+、Spring Boot 2.1+
- 2.0.x版本适用于：MyBatis 3.5+、Java 8+、Spring Boot 2.0/2.1
- 1.3.x版本适用于：MyBatis 3.4+、Java 6+、Spring Boot 1.5

## 3 数据库连接参数

在 Spring Boot 自动化配置中，对于数据源的配置可以分为两类：

- 通用配置：以`spring.datasource.*`的形式存在，主要是对一些即使使用不同数据源也都需要配置的一些常规内容。比如：数据库链接地址、用户名、密码等。

- 数据源连接池配置：以`spring.datasource.<数据源名称>.*`的形式存在，比如：Hikari 的配置参数就是`spring.datasource.hikari.*`形式。

在Spring Boot 2.x中，默认数据源是 [HikariCP](https://github.com/brettwooldridge/HikariCP)。

其常用配置项如下：

- `spring.datasource.hikari.minimum-idle`: 最小空闲连接，默认值 10，小于 0 或大于 maximum-pool-size，都会重置为 maximum-pool-size
- `spring.datasource.hikari.maximum-pool-size`: 最大连接数，小于等于 0 会被重置为默认值 10 ；大于零小于 1 会被重置为 minimum-idle的值
- `spring.datasource.hikari.idle-timeout`: 空闲连接超时时间，默认值 600000（10分钟），大于等于max-lifetime 且 max-lifetime > 0，会被重置为 0；不等于 0 且小于 10 秒，会被重置为 10 秒。
- `spring.datasource.hikari.max-lifetime`: 连接最大存活时间，不等于 0 且小于 30 秒，会被重置为默认值 30 分钟。设置应该比 mysql 设置的超时时间短
- `spring.datasource.hikari.connection-timeout`: 连接超时时间：毫秒，小于 250 毫秒，否则被重置为默认值 30 秒
- `spring.datasource.hikari.connection-test-query`: 用于测试连接是否可用的查询语句

```yaml
spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://127.0.0.1:3306/mybatis?userUnicode=true&characterEncoding=UTF-8&useSSL=false&autoReconnect=true&failOverReadOnly=false&serverTimezone=GMT%2B8
    username: root
    password: root
    hikari:
      minimum-idle: 0
      maximum-pool-size: 20
      idle-timeout: 500000
      max-lifetime: 540000
      connection-timeout: 60000
      connection-test-query: SELECT 1
# mybatis配置，下划线转驼峰、打印SQL语句、mapper文件位置、包别名
mybatis:
  configuration:
    map-underscore-to-camel-case: true
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
  mapper-locations: classpath:mappers/*.xml
  type-aliases-package: com.xiao.mybatis.entity
```
## 4 实体类、Mapper接口、xml文件
1. **实体类**
```java
@Data
@Builder
public class User implements Serializable {
    public static final long serialVersionUID = 1L;

    /**
     * 主键
     */
    private Long id;

    /**
     * 用户名
     */
    private String name;

    /**
     * 手机号码
     */
    private String phoneNumber;

    /**
     * 状态，1 有效；0 无效
     */
    private Byte status;

    /**
     * 创建时间
     */
    private LocalDateTime createTime;

    /**
     * 更新时间
     */
    private LocalDateTime updateTime;
}
```

2. **Mapper 接口文件**

新建 UserMapper，在接口中定义两个数据操作，一个查询，一个插入。

支持两种方式进行 SQL 语句操作：

   - 注解形式
   - XML 文件
```java
 @Mapper
public interface UserMapper {

    /**
     * 查询所有用户
     *
     * @return 用户集合
     */
    @Select("SELECT * FROM user")
    List<User> selectAll();

    /**
     * 根据id查询
     * @param id 用户id
     * @return 用户信息
     */
    @Select("SELECT * FROM user WHERE id = #{id}")
    User selectById(@Param("id") Long id);

    /**
     * 新增用户信息
     * @param model 用户信息
     * @return 成功 ：1； 失败 ： 0
     */
    int add(User model);

    /**
     * 修改用户信息
     * @param model 用户信息
     * @return 成功 ：1； 失败 ： 0
     */
    int update(User model);


    /**
     * 删除用户
     * @param id 用户id
     * @return 成功 ：1； 失败 ： 0
     */
    @Delete("update user set status = 0 where id = #{id}")
    int deleteById(@Param("id") Long id);
}
```
3. **xml 配置文件**

insert 标签中标明` useGeneratedKeys="true" keyProperty="id"`：可以将自增主键的值设置进实体类中
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.xiao.mybatis.mapper.UserMapper">


    <insert id="add">
        INSERT INTO `user` (`name`,`phone_number`,`status`,`create_time`,`update_time`)
        VALUES (#{name}, #{phoneNumber}, #{status},#{createTime}, #{updateTime})
    </insert>

    <update id="update">
        UPDATE `user`
        <set>
            <if test="name != null">
                name = #{name},
            </if>
            <if test="phoneNumber != null">
                phone_number = #{phoneNumber},
            </if>
            <if test="status != null">
                status = #{status},
            </if>
            <if test="createTime != null">
                create_time = #{createTime},
            </if>
            <if test="updateTime != null">
                update_time = #{updateTime},
            </if>
        </set>
        WHERE id = #{id}
    </update>
</mapper>
```

4. **创建 Spring Boot 主程序类**

```java
@SpringBootApplication
@MapperScan("com.xiao.mybatis.mapper")
public class MybatisApplication {
    public static void main(String[] args) {
        SpringApplication.run(MybatisApplication.class);
        System.out.println("--------service start success ------");
    }
}
```

5. **编写单元测试**

- 使用 `@Transactional` 和 `@Rollback` 注解，测试结束回滚数据，保证测试单元每次运行的数据环境独立

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class MybatisApplicationTest {

    @Autowired
    private UserMapper userMapper;

    @Test
    @Transactional
    @Rollback
    public void test(){
        User user = User.builder()
                .name("zhangsan")
                .phoneNumber("12345678901")
                .status((byte) 1)
                .createTime(LocalDateTime.now())
                .updateTime(LocalDateTime.now())
                .build();
        userMapper.add(user);
        List<User> users = userMapper.selectAll();
        Assert.assertEquals(1,users.size());
        Assert.assertEquals("zhangsan",users.get(0).getName());
    }
}
```

