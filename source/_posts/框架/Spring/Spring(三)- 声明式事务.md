---
title: Spring (三) - 声明式事务
categories:
  - 框架
  - Spring
tags:
  - Spring
  - AOP
  - 动态代理
date: 2021-06-14
description: Spring 中事务的使用。
---

Spring 基础知识学习笔记 (三)，声明式事务，内容包括：：

1. 注解实现声明式事务
2. 事务的隔离级别
3. 事务的传播行为
4. 配置文件实现

参考视频：

B 站尚硅谷雷丰阳大神的 Spring、Spring MVC、MyBatis 课程 https://www.bilibili.com/video/BV1d4411g7tv

## 1. 环境搭建

### 1.1 数据库环境

三张表：账户表 account，书籍价格表 book，书籍库存表 book_stock

```sql
CREATE TABLE account (
username VARCHAR(50) PRIMARY KEY,
balance INT(10) NOT NULL
)ENGINE=INNODB DEFAULT CHARSET=utf8;


INSERT INTO account VALUES 
("Tom",1000),
("Jerry",1000);

CREATE TABLE book (
isbn VARCHAR(50) PRIMARY KEY,
book_name VARCHAR(50) NOT NULL,
price INT(10)
)ENGINE	= INNODB DEFAULT CHARSET=utf8;

INSERT INTO book VALUES	
("ISBN-001","book01",100),
("ISBN-002","book02",200),
("ISBN-003","book03",300),
("ISBN-004","book04",400),
("ISBN-005","book05",500);

CREATE TABLE book_stock (
isbn VARCHAR(50) PRIMARY KEY,
stock INT(10) NOT NULL
)ENGINE= INNODB DEFAULT CHARSET = utf8;

INSERT INTO book_stock	VALUES 
("ISBN-001",10),
("ISBN-002",10),
("ISBN-003",10),
("ISBN-004",10),
("ISBN-005",10);
```

### 1.2 减余额、减库存的方法

1. 新建一个 BookDao 类，用于操作数据库，包括减账户余额、修改图书价格、减图书库存方法等

```java
@Repository
public class BookDao {

    @Autowired
    private JdbcTemplate jdbcTemplate;

    /**
     * 减去某个用户的账户余额
     * @param userName 用户名
     * @param price 金额
     */
    public void updateBalance(String userName, int price) {
        String sql = "UPDATE account SET balance = balance - ? WHERE username = ?";
        this.jdbcTemplate.update(sql, price, userName);
    }

    /**
     * 按照图书的isbn获取图书的价格
     * @param isbn 编号
     * @return
     */
    public Integer getPrice(String isbn) {
        String sql = "SELECT price FROM book WHERE isbn = ?";
        return this.jdbcTemplate.queryForObject(sql, Integer.class, isbn);
    }

    /**
     * 减去图书的库存,每次减去1
     * @param isbn 编号
     */
    public void updateStock(String isbn) {
        String sql = "UPDATE book_stock SET stock = stock-1 WHERE isbn = ?";
        this.jdbcTemplate.update(sql, isbn);
    }

    /**
     * 修改图书价格
     * @param isbn 编号
     * @param price 要修改的价格
     */
    public void updatePrice(String isbn, Integer price) {
        String sql = "UPDATE book SET price=? where isbn =?";
        this.jdbcTemplate.update(sql, price, isbn);
    }
}
```

2. 新建一个 BookService 类，结账方法，调用减账户余额和减图书库存两个方法

```java
@Service
public class BookService {

    @Autowired
    private BookDao bookDao;

    // 结账方法，分为减库存，减余额两步操作
    public void checkOut(String username,String isbn) {
        // 1.减库存
        this.bookDao.updateStock(isbn);
        System.out.println("减库存完成！");
        // 故意引入异常
        int a = 1/0;
        // 根据isbn查询价格
        Integer price = this.bookDao.getPrice(isbn);
        // 2.减账户余额
        this.bookDao.updateBalance(username,price);
        System.out.println("结账完成!");

    }
}
```

3. 数据库配置文件 **db.properties** 和 Spring配置文件 **ApplicationContext.xml**：

`db.properties`

```properties
jdbc.driver=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/spring_learn?useSSL=false&useUnicode=true&characterEncoding=utf8
jdbc.username=root
jdbc.password=root
```

`ApplicationContext.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context
       https://www.springframework.org/schema/context/spring-context.xsd">

    <!--包扫描-->
    <context:component-scan base-package="com.xiao"/>

    <!--引入数据库配置文件-->
    <context:property-placeholder location="classpath:db.properties"/>

    <!--数据库连接信息-->
    <bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
        <property name="driverClass" value="${jdbc.driver}"/>
        <property name="jdbcUrl" value="${jdbc.url}"/>
        <property name="user" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
    </bean>

    <!--注册jdbcTemplate，传入一个数据源即可-->
    <bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
    <constructor-arg name="dataSource" ref="dataSource"/>
    </bean>
    
</beans>
```

4. 测试：

```java
public class TransactionTest {

    ApplicationContext ioc = new ClassPathXmlApplicationContext("applicationContext.xml");

    @Test
    public void test() {
        BookService bookService = ioc.getBean(BookService.class);
        System.out.println(bookService.getClass());
        System.out.println(ioc.getBean(BookDao.class).getClass());
        bookService.checkOut("Tom", "ISBN-001");

    }
}
```

如果在 **减图书库存** 和 **减账户余额** 之间故意插入异常，则只会执行 **减图书库存** ，**减账户余额** 未执行。

## 2. 声明式事务

### 2.1 事务的 ACID 原则

- **原子性（Atomicity）** ： 事务是最小的执行单位，不允许分割。事务的原子性确保动作要么全部完成，要么完全不起作用；
- **一致性（Consistency）** ：执行事务前后，数据保持一致，多个事务对同一个数据读取的结果是相同的；
- **隔离性（Isolation）**：并发访问数据库时，一个用户的事务不被其他事务所干扰，各并发事务之间数据库是独立的；
- **持久性（Durability）** ： 一个事务被提交之后。它对数据库中数据的改变是持久的，即使数据库发生故障也不应该对其有任何影响。

### 2.2 声明式事务配置

Spring 提供了事务管理器，就可以在目标方法运行前后进行事务控制(事务切面)。这里使用DataSourceTransaction。

步骤：

1. 配置事务管理器让其进行事务控制，传入要控制哪个数据源
2. 开启基于注解的事务控制，依赖于 tx 名称空间，指定事务管理器的id
3. 给事务方法加注解 `@Transactional`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context
       https://www.springframework.org/schema/context/spring-context.xsd
       http://www.springframework.org/schema/tx
       http://www.springframework.org/schema/tx/spring-tx.xsd">


    <!--包扫描-->
    <context:component-scan base-package="com.xiao"/>

    <!--引入数据库配置文件-->
    <context:property-placeholder location="classpath:db.properties"/>

    <!--数据库连接信息-->
    <bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
        <property name="driverClass" value="${jdbc.driver}"/>
        <property name="jdbcUrl" value="${jdbc.url}"/>
        <property name="user" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
    </bean>

    <!--注册jdbcTemplate，传入一个数据源即可-->
    <bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
    <constructor-arg name="dataSource" ref="dataSource"/>
    </bean>

    <!--配置事务管理器(切面)，DataSourceTransactionManager-->
    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <!--要控制哪个数据源-->
        <property name="dataSource" ref="dataSource"/>
    </bean>

    <!--开启基于注解的事务控制模式，指定事务管理器的id-->
    <tx:annotation-driven transaction-manager="transactionManager"/>

</beans>
```

方法上加上注解`@Transactional`

```java
@Transactional(rollbackFor = Exception.class)
public void checkOut(String username,String isbn) {
//....
}
```

此时 **减图书库存** 和 **减账户余额** 之间发生异常时，事务会回滚。

## 3. 事务细节

- `Isolation isolation()`：事务的隔离级别，默认为 Isolation.DEFAULT
- `Propagation propagation()`：事务的传播行为，默认为 Propagation.REQUIRED
- `Class<? extends Throwable>[] rollbackFor() `：哪些异常事务需要回滚，让本来不回滚的异常进行回滚
- `String[] rollbackForClassName()`：
- `Class<? extends Throwable>[] noRollbackFor()`：哪些异常事务可以不回滚，让本来回滚的异常不回滚
- `String[] noRollbackForClassName()`
- `int timeout() `：事务超出指定执行时长后自动终止并回滚，单位为秒
- `boolean readOnly()`：设置事务为只读事务，加快查询速度，不用管事务那一堆操作。默认为false

### 3.1 超时/只读

```java
@Transactional(timeout = 3,readOnly = false)
```

### 3.2 rollbackFor/noRollbackFor

运行时异常（非检查异常）：可以不用处理，默认都回滚

编译时异常（检查异常）：要么 try-catch，要么在方法上声明 throws，默认不回滚

- `Class<? extends Throwable>[] rollbackFor() `：哪些异常事务需要回滚，让本来不回滚的异常进行回滚

- `Class<? extends Throwable>[] noRollbackFor()`：哪些异常事务可以不回滚，让本来回滚的异常不回滚

```java
@Transactional(noRollbackFor = {ArithmeticException.class})
```

算术运算异常，是运行时异常，本来默认回滚的，设置noRollbackFor属性后，就不回滚了。

### 3.3 事务的隔离级别

####  01 事务并发运行带来的问题

多个事务并发运行，经常会操作相同的数据来完成各自的任务，能会导致以下的问题：

- **脏读（Dirty read）:** 当一个事务正在访问数据并且对数据进行了修改，而这种修改还没有提交到数据库中，这时另外一个事务也访问了这个数据，然后使用了这个数据。因为这个数据是还没有提交的数据，那么另外一个事务读到的这个数据是“脏数据”，依据“脏数据”所做的操作可能是不正确的。
- **不可重复读（Unrepeatableread）:** 指在一个事务内多次读同一数据。在这个事务还没有结束时，另一个事务也访问该数据。那么，在第一个事务中的两次读数据之间，由于第二个事务的修改导致第一个事务两次读取的数据可能不太一样。这就发生了在一个事务内两次读到的数据是不一样的情况，因此称为不可重复读。
- **幻读（Phantom read）:** 幻读与不可重复读类似。它发生在一个事务（T1）读取了几行数据，接着另一个并发事务（T2）插入了一些数据时。在随后的查询中，第一个事务（T1）就会发现多了一些原本不存在的记录，就好像发生了幻觉一样，所以称为幻读。

#### 02 SQL的隔离级别

SQL 标准定义了四个隔离级别：

- **READ-UNCOMMITTED(读取未提交)：** 最低的隔离级别，允许读取尚未提交的数据变更，**可能会导致脏读、幻读或不可重复读**。
- **READ-COMMITTED(读取已提交)：** 允许读取并发事务已经提交的数据，**可以阻止脏读，但是幻读或不可重复读仍有可能发生**。
- **REPEATABLE-READ(可重复读)：** 对同一字段的多次读取结果都是一致的，除非数据是被本身事务自己所修改，**可以阻止脏读和不可重复读，但幻读仍有可能发生**。MySQL 的 InnoDB 存储引擎默认的。
- **SERIALIZABLE(可串行化)：** 最高的隔离级别，完全服从 ACID 的隔离级别。所有的事务依次逐个执行，这样事务之间就完全不可能产生干扰，也就是说，**该级别可以防止脏读、不可重复读以及幻读**。

#### 03 Spring事务的隔离级别

TransactionDefinition 接口中定义了五个表示隔离级别的常量：

- **TransactionDefinition.ISOLATION_DEFAULT:** 使用后端数据库默认的隔离级别，MySQL 默认采用的 REPEATABLE_READ 隔离级别 Oracle 默认采用的 READ_COMMITTED隔离级别.
- **TransactionDefinition.ISOLATION_READ_UNCOMMITTED:** 最低的隔离级别，允许读取尚未提交的数据变更，**可能会导致脏读、幻读或不可重复读**
- **TransactionDefinition.ISOLATION_READ_COMMITTED:** 允许读取并发事务已经提交的数据，**可以阻止脏读，但是幻读或不可重复读仍有可能发生**
- **TransactionDefinition.ISOLATION_REPEATABLE_READ:** 对同一字段的多次读取结果都是一致的，除非数据是被本身事务自己所修改，**可以阻止脏读和不可重复读，但幻读仍有可能发生。**
- **TransactionDefinition.ISOLATION_SERIALIZABLE:** 最高的隔离级别，完全服从ACID的隔离级别。所有的事务依次逐个执行，这样事务之间就完全不可能产生干扰，也就是说，**该级别可以防止脏读、不可重复读以及幻读**。但是这将严重影响程序的性能。通常情况下也不会用到该级别。

### 3.4 事务的传播行为

事务的传播行为，即如果有多个事务进行嵌套运行，子事务是否要和父事务公用一个事务。

当事务方法被另一个事务方法调用时，必须指定事务应该如何传播。

事务传播属性可以在 `@Transactional` 注解的 propagation 属性中定义。Spring 定义了 7 种传播行为。

**支持当前事务的情况：**

- **TransactionDefinition.PROPAGATION_REQUIRED：** 如果当前存在事务，则加入该事务；如果当前没有事务，则创建一个新的事务。
- **TransactionDefinition.PROPAGATION_SUPPORTS：** 如果当前存在事务，则加入该事务；如果当前没有事务，则以非事务的方式继续运行。
- **TransactionDefinition.PROPAGATION_MANDATORY：** 如果当前存在事务，则加入该事务；如果当前没有事务，则抛出异常。（mandatory：强制性）

**不支持当前事务的情况：**

- **TransactionDefinition.PROPAGATION_REQUIRES_NEW：** 当前事务总是创建一个新的事务，如果当前存在事务，则把当前事务挂起。
- **TransactionDefinition.PROPAGATION_NOT_SUPPORTED：** 以非事务方式运行，如果当前存在事务，则把当前事务挂起。
- **TransactionDefinition.PROPAGATION_NEVER：** 以非事务方式运行，如果当前存在事务，则抛出异常。

**其他情况：**

- **TransactionDefinition.PROPAGATION_NESTED：** 如果当前存在事务，则创建一个事务作为当前事务的嵌套事务来运行；如果当前没有事务，则该取值等价于TransactionDefinition.PROPAGATION_REQUIRED。

### 3.5 案例

REQUIRED：将之前事务用的 connection 传递给这个方法使用

REQUIRES_NEW：这个方法直接使用新的 connection

在 BookDao 类中再定义一个修改图书价格的方法：

```java
    //修改图书价格
    public void updatePrice(String isbn,Integer price) {
        String sql = "UPDATE book SET price=? where isbn =?";
        this.jdbcTemplate.update(sql,price,isbn);
    }
```

BookService 类：

```java
//改价格方法
@Transactional(rollbackFor = Exception.class)
public void updatePrice(String isbn,Integer price){
    this.bookDao.updatePrice(isbn, price);
}
```

新建一个 MulService 类，其中有一个声明了事务的方法，同时调用结账方法和修改价格的方法

```java
@Service
public class MulService {

    @Autowired
    private BookService bookService;

    @Transactional(rollbackFor = Exception.class)
    public void mulTx() {
        //结账 
        this.bookService.checkOut("Tom", "ISBN-001");
        //修改价格
        this.bookService.updatePrice("ISBN-002", 998);
    
    }
}

```

#### 01 情况一

checkOut() 方法和 updatePrice() 方法默认传播行为是 REQUIRED，因为 mulTx() 方法存在事务，所以就加入它，**所以如果一个方法崩了，则整体都会回滚。**

```java
    //结账 REQUIRED
    this.bookService.checkOut("Tom", "ISBN-001");
    //修改价格 REQUIRED
    this.bookService.updatePrice("ISBN-002", 998);
```
大家都在一车上，一个翻车全翻车。

#### 02 情况二

checkOut() 方法传播行为设置为 REQUIRES_NEW，即自己去开个新事务，在updatePrice() 方法中引入异常。

相当于第一个方法开了新车，第二个方法跟主方法在一车上，翻车了并不影响第一个方法，因此结账执行完成了，修改价格会回滚。

```java
//结账 REQUIRES_NEW
this.bookService.checkOut("Tom", "ISBN-001");
//修改价格 REQUIRED
this.bookService.updatePrice("ISBN-002", 998);
```
## 4. 配置文件实现声明式事务

`aop:config`：告诉 Spring 哪些方法是事务方法：事务切面按照外面的切入点表达式去切入事务方法

`tx:advice`：配置事务建议，切入上面的切入点

```xml
 <?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context
       https://www.springframework.org/schema/context/spring-context.xsd
       http://www.springframework.org/schema/tx
       http://www.springframework.org/schema/tx/spring-tx.xsd
       http://www.springframework.org/schema/aop
       http://www.springframework.org/schema/aop/spring-aop.xsd">

    <!--包扫描-->
    <context:component-scan base-package="com.xiao"/>

    <!--引入数据库配置文件-->
    <context:property-placeholder location="classpath:db.properties"/>

    <!--数据库连接信息-->
    <bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
        <property name="driverClass" value="${jdbc.driver}"/>
        <property name="jdbcUrl" value="${jdbc.url}"/>
        <property name="user" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
    </bean>

    <!--注册jdbcTemplate，传入一个数据源即可-->
    <bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
        <constructor-arg name="dataSource" ref="dataSource"/>
    </bean>

    <!--配置事务管理器(切面)，DataSourceTransactionManager-->
    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <!--要控制哪个数据源-->
        <property name="dataSource" ref="dataSource"/>
    </bean>

    <!--结合AOP实现事务的织入-->
    <!--配置事务通知-->
    <tx:advice id="myAdvice" transaction-manager="transactionManager">
        <!--配置事务属性，传播特性、超时时间等-->
        <tx:attributes>
            <!--指明哪些方法是事务方法-->
            <tx:method name="*" propagation="REQUIRED"/>
        </tx:attributes>
    </tx:advice>

    <!--配置事务切入，告诉Spring哪些方法是事务方法-->
    <aop:config>
        <!--配置切入点，com.xiao.service包下的所有类的所有方法，只是说事务管理器要切入这些方法-->
        <aop:pointcut id="myPoint" expression="execution(* com.xiao.service.*.*(..))"/>
        <!--事务建议，让事务管理器切面来切入这个切入点表达式-->
        <aop:advisor advice-ref="myAdvice" pointcut-ref="myPoint"/>
    </aop:config>

</beans>
```

