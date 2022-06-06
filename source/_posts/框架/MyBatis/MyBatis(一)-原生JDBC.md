---
title: MyBatis (一) - 原生 JDBC
categories:
  - 框架
  - MyBatis
tags:
  - MyBatis
  - JDBC
date: 2021-05-31
description: SSM 入门之 MyBatis，先学会原生 JDBC。
---

MyBatis 学习笔记 (一)， 内容包括：

1. 原生 JDBC 编程步骤；
2. JDBC 关键对象

## 1 JDBC 简介

> JDBC (Java Database Connectivity) API，即 Java 数据库编程接口。

是一组标准的Java语言中的接口和类，使用这些接口和类，Java 客户端程序可以访问各种不同类型的数据库。比如建立数据库连接、执行 SQL 语句进行数据的存取操作。

JDBC 库中所包含的 API 任务通常与数据库使用：

- 连接到数据库
- 创建 SQL 或 MySQL 语句
- 在数据库中执行 SQL 或 MySQL 查询
- 查看和修改记录

## 2 JDBC 编程步骤

1. 加载数据库驱动 `Class.forName`
2. 创建并获取数据库链接 `DriverManage.getConnection`
3. 创建 jdbc statement对象 `connection.createStatement`
4. 设置 SQL 语句
5. 设置 SQL 语句中的参数 (使用 preparedStatement )
6. 通过 statement 执行 SQL 并获取结果
7. 对 SQL 执行结果进行解析处理
8. 释放资源 (resultSet、preparedstatement、connection)

## 3 几个关键对象

### （1）Connection

数据库的链接 Connection，创建方法为：

```java
Connection conn = DriverManager.getConnection(url,user,pass); 
```

常用方法：

| 方法                              | 说明                         |
| --------------------------------- | ---------------------------- |
| createStatement( )                | 执行SQL的对象                |
| prepareStatement(sql)             | 执行预编译的SQL              |
| prepaceCall(sql)                  | 执行存储过程                 |
| setAutoCommit(boolean autoCommit) | 设置事务是否自动提交，默认是 |
| commit()                          | 提交事务                     |
| rollback()                        | 回滚事务                     |

### （2）Statement

Statement，用于向数据库发送SQL语句，创建方法为：

```java
Statement st = conn.createStatement();
```

常用方法：

| 方法                      | 说明                                       |
| ------------------------- | ------------------------------------------ |
| executeQuery(String sql)  | 向数据发送查询语句                         |
| executeUpdate(String sql) | 向数据库发送 insert、update 或 delete 语句 |
| execute(String sql)       | 向数据库发送任意 sql 语句                  |
| addBatch(String sql)      | 把多条 sql 语句放到一个批处理中            |
| executeBatch()            | 向数据库发送一批 sql 语句执行              |

### （3）PreparedStatement

PreparedStatement，可执行预编译的 SQL 语句，避免 SQL 注入的问题，创建方法为：

```java
PreparedStatement preparedStatement = connection.prepareStatement(sql);
```

常用方法：

| 方法                                   | 说明                                        |
| -------------------------------------- | ------------------------------------------- |
| setString(int parameterIndex,String x) | 设置参数，其他数据类型同理，setInt、setLong |

### （4）ResultSet

ResultSet，结果集对象。

Resultset 封装执行结果时，维护了一个指向表格数据行的游标，初始游标在第一行之前，调用 ResultSet.next() 方法，可以使游标指向具体的数据行，进行调用方法获取该行的数据。

常用方法：

| 方法                         | 说明                                 |
| ---------------------------- | ------------------------------------ |
| next()                       | 移动到下一行                         |
| previous()                   | 移动到前一行                         |
| absolute(int row)            | 移动到指定行                         |
| beforeFirst()                | 移动resultSet的最前面                |
| afterLast()                  | 移动到resultSet的最后面              |
| getString(int index)         | 获取指定类型的数据，其余数据类型同理 |
| getString(String columnName) | 获取指定类型的数据                   |

## 4 代码示例

数据库环境：

```sql
CREATE TABLE `user_info` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `user_name` varchar(255) DEFAULT NULL,
  `age` int(11) DEFAULT NULL,
  `create_time` datetime DEFAULT NULL,
  `update_time` datetime DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=3 DEFAULT CHARSET=utf8;

INSERT INTO `demo`.`user_info`(`id`, `user_name`, `age`, `create_time`, `update_time`) VALUES (1, '张三', 20, '2021-05-31 15:24:55', '2021-05-31 15:24:58');
INSERT INTO `demo`.`user_info`(`id`, `user_name`, `age`, `create_time`, `update_time`) VALUES (2, '李四', 30, '2021-05-31 15:25:08', '2021-05-31 15:25:11');
```



工具类：

```java
package com.xiao.jdbc.util;


import java.io.IOException;
import java.io.InputStream;
import java.sql.*;
import java.util.Properties;


/**
 * JDBC工具类
 *
 * @author KongXiao
 * @date 2021/6/1
 */
public class JdbcUtils {

    private static String driver;
    private static String url;
    private static String username;
    private static String password;

    // 读取属性文件，获取jdbc信息
    static {
        try {
            readConfig();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    /**
     * 读取数据库配置文件
     *
     * @throws IOException
     */
    private static void readConfig() throws IOException {
        InputStream inputStream = JdbcUtils.class.getClassLoader().getResourceAsStream("db.properties");
        Properties properties = new Properties();
        properties.load(inputStream);
        driver = properties.getProperty("driver");
        url = properties.getProperty("url");
        username = properties.getProperty("username");
        password = properties.getProperty("password");
    }

    /**
     * 获取数据库连接对象
     *
     * @return 数据库连接对象Connection
     */
    public static Connection getConnection() {
        Connection connection = null;
        try {
            // 1. 注册JDBC驱动
            Class.forName(driver);
            // 2. 获取数据库连接
            connection = DriverManager.getConnection(url, username, password);
        } catch (ClassNotFoundException | SQLException e) {
            e.printStackTrace();
        }
        return connection;
    }

    /**
     * 关闭结果集、数据库操作对象、数据库连接
     *
     * @param connection        数据库连接对象
     * @param preparedStatement 数据库操作对象
     * @param resultSet         结果集
     */
    public static void release(Connection connection, PreparedStatement preparedStatement, ResultSet resultSet) {

        if (resultSet != null) {
            try {
                resultSet.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
        if (preparedStatement != null) {
            try {
                preparedStatement.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
        if (connection != null) {
            try {
                connection.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
    }
}
```

测试类：

```java
/**
 * JDBC工具类测试
 *
 * @author KongXiao
 * @date 2021/6/1
 */
public class JdbcUtilsTest {

    /**
     * 查询
     * @throws SQLException
     */
    @Test
    public void JDBCQueryTest() throws SQLException {
        Connection connection = JdbcUtils.getConnection();

        String sql = "select * from user_info where user_name = ?";
        PreparedStatement preparedStatement = connection.prepareStatement(sql);
        preparedStatement.setString(1, "张三");
        ResultSet resultSet = preparedStatement.executeQuery();
        while (resultSet.next()) {
            // 通过字段检索
            int id = resultSet.getInt("id");
            String userName = resultSet.getString("user_name");
            int age = resultSet.getInt("age");
            String createTime = resultSet.getString("create_time");

            // 输出数据
            System.out.print("ID: " + id);
            System.out.print(", 姓名: " + userName);
            System.out.print(", 年龄: " + age);
            System.out.print(", 创建时间: " + createTime);
            System.out.print("\n");
        }
        JdbcUtils.release(connection, preparedStatement, resultSet);
    }
}
```