---
title: 07 SpringBoot 整合 Spring Data JPA
categories:
  - 框架
  - SpringBoot 
tags:
  - SpringBoot 
  - Mybatis
date: 2021-08-10
description: SpringBoot 整合 JPA，解放双手，单表查询不再写 SQL，但是不适用复杂的连表查询
---

官方文档：https://docs.spring.io/spring-data/jpa/docs/2.4.2/reference/html/#repository-query-keywords

GitHub: https://github.com/spring-projects/spring-data-jpa

## 1 引入依赖

- jpa
- mysql-connector-java

```xml

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>

<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
</dependency>
```

## 2 配置文件

```yaml
spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://127.0.0.1:3306/mybatis?userUnicode=true&characterEncoding=UTF-8&useSSL=false&autoReconnect=true&failOverReadOnly=false&serverTimezone=GMT%2B8
    username: root
    password: root

  jpa:
    properties:
      hibernate:
        hbm2ddl:
          auto: update
```

`spring.jpa.properties.hibernate.hbm2ddl.auto` 是 hibernate 的配置属性，其主要作用是：自动创建、更新、验证数据库表结构。该参数的几种配置如下：

- create：每次加载hibernate时都会删除上一次的生成的表，然后根据 model 类再重新来生成新表。谨慎使用
- create-drop：每次加载hibernate时根据model类生成表，但是 sessionFactory 一关闭,表就自动删除
- update：最常用的属性，第一次加载hibernate时根据model类会自动建立起表的结构（前提是先建立好数据库），以后加载hibernate时根据model类自动更新表结构，即使表结构改变了但表中的行仍然存在不会删除以前的行。要注意的是当部署到服务器后，表结构是不会被马上建立起来的，是要等应用第一次运行起来后才会。
- validate：每次加载hibernate时，验证创建数据库表结构，只会和数据库中的表进行比较，不会创建新表，但是会插入新值。

## 3 创建实体类

```java
@Data
@Builder
@AllArgsConstructor
@NoArgsConstructor
@Entity
@Table(name = "user")
public class User implements Serializable {
    public static final long serialVersionUID = 1L;

    /**
     * 主键
     */
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    /**
     * 用户名
     */
    @Column(name = "name")
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

注解说明：

- `@Entity`: 标识了 User 类是一个持久化的实体
- `@Id` 和 `@GeneratedValue` 用来标识 User 对应对应数据库表中的主键，可选四种策略。
    - TABLE：使用一个特定的数据库表格来保存主键。
    - SEQUENCE：根据底层数据库的序列来生成主键，条件是数据库支持序列。
    - IDENTITY：主键由数据库自动生成（主要是自动增长型）
    - AUTO：主键由程序控制。
- `@Column`：用于成员变量名与表字段名的映射，JPA 默认使用的命名策略是下划线分隔的字段命名
- `@DynamicInsert`: 设置为 true,表示 insert 对象的时候,生成动态的 insert 语句,如果这个字段的值是 null 就不会加入到 insert 语句当中。默认 false
- `@DynamicUpdate`: 设置为true,表示 update 对象的时候,生成动态的 update 语句,如果这个字段的值是 null 就不会被加入到 update 语句中,默认 false

## 4 创建数据访问接口 Repository
```java
@Repository
public interface UserRepository extends JpaRepository<User, Long> {

    /**
     * 根据用户名查找
     *
     * @param name
     * @return
     */
    User findByName(String name);

    /**
     * 根据用户名和手机号查询
     *
     * @param name
     * @param phoneNumber
     * @return
     */
    User findByNameAndPhoneNumber(String name, String phoneNumber);

    /**
     * @param name
     * @return
     */
    @Query("from User u where u.name=:name")
    User findUser(@Param("name") String name);
    
}
```
在 Spring Data JPA中，只需要编写接口继承 JpaRepository, 就可实现数据访问。

JpaRepository 接口本身已经实现了创建（save）、更新（save）、删除（delete）、查询（findAll、findOne）等基本操作的函数，因此对于这些基础操作的数据访问就不需要开发者再自己定义。

支持通过解析方法名创建查询，例如 findByNameAndPhoneNumber

支持使用 `@Query` 注解来创建查询，编写 JPQL 语句，并通过类似“:name”来映射 @Param 指定的参数