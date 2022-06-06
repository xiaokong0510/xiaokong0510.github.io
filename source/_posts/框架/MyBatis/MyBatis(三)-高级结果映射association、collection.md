---
title: MyBatis (三) - 高级结果映射 association、collection
categories:
  - 框架
  - MyBatis
tags:
  - 关联查询
  - MyBatis
date: 2021-06-03
description: MyBatis 的多表关联查询，学会封装复杂结果集。
---

MyBatis 学习笔记 (三)，多表查询的高级结果映射，内容包括：

5. 多对一查询  关联 (association)
2. 一对多查询  集合 (collection)

学习视频：尚硅谷雷丰阳老师 MyBatis https://www.bilibili.com/video/BV1bb411A7bD


## 1. association--多对一查询

处理多表查询。

MyBatis 有两种不同的方式加载关联：

- 嵌套 Select 查询：通过执行另外一个 SQL 映射语句来加载期望的复杂类型。
- 嵌套结果映射：使用嵌套的结果映射来处理连接结果的重复子集。

### 1.1 测试环境

两张表，员工表 tbl_employee 和部门表 tbl_dept，员工表中设置外键部门 id ，指向部门表的主键

在之前的基础上新建了部门表，并在员工表中设置外键，插入数据：

```sql
# 员工表
CREATE TABLE tbl_employee(
id INT(10) PRIMARY KEY AUTO_INCREMENT,  
last_name VARCHAR(255),
gender CHAR(1),
email VARCHAR(255)
)ENGINE=INNODB DEFAULT CHARSET=utf8;

# 部门表
CREATE TABLE tbl_dept(
id INT(10) PRIMARY KEY AUTO_INCREMENT,
dept_name VARCHAR(255)
)ENGINE=INNODB DEFAULT CHARSET=UTF8;

INSERT INTO `mybatis`.`tbl_dept`(`id`, `dept_name`) VALUES (1, '开发部');
INSERT INTO `mybatis`.`tbl_dept`(`id`, `dept_name`) VALUES (2, '测试部');

# 员工表新增字段部门id，并设置外键
ALTER TABLE tbl_employee ADD COLUMN d_id INT(10);
ALTER TABLE tbl_employee ADD CONSTRAINT fk_emp_dept FOREIGN KEY(d_id) REFERENCES tbl_dept(id);

INSERT INTO `mybatis`.`tbl_employee`(`id`, `last_name`, `gender`, `email`, `d_id`) VALUES (1, 'lisi', '0', 'lisi@163.com', 1);
INSERT INTO `mybatis`.`tbl_employee`(`id`, `last_name`, `gender`, `email`, `d_id`) VALUES (2, 'wangwu', '1', 'wangwu@126.com', 1);
INSERT INTO `mybatis`.`tbl_employee`(`id`, `last_name`, `gender`, `email`, `d_id`) VALUES (3, '林青霞', '0', 'lingqingxia@163.com', 2);
INSERT INTO `mybatis`.`tbl_employee`(`id`, `last_name`, `gender`, `email`, `d_id`) VALUES (4, 'Tom', '0', '2', 2);
```

员工表：

![image-20210618153546857](http://image.kongxiao.top/image-20210618153546857.png)

部门表：

![image-20210618153603192](http://image.kongxiao.top/image-20210618153603192.png)

Employee实体类：

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
public class Employee {
    private Integer id;
    private String lastName;
    private String email;
    private String gender;
    private Dept dept;

}
```

Dept实体类：

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
public class Dept {
    private Integer id;
    private String departmentName;
}
```

### 1.2 级联属性封装结果

在接口文件中声明一个根据员工 id 查询员工信息的方法，因为员工信息中一个成员是部门对象，按照之前的映射关系，无法查询部门信息。

```java
public interface EmployeeMapper {
    /**
     * 根据id查询员工的信息及其部门信息
     * @param id 员工id
     * @return
     */
    Employee getEmpAndDept(Integer id);
}
```

**采用级联属性来封装结果：**

employee 中有一个成员是 dept，由 dept.id 和 dept.departmentName 则可以获取到部门的 id 和部门名称

```xml
<!--联合查询，级联属性封装结果集-->
<resultMap id="MyDifEmp" type="com.xiao.mybatis.entity.Employee">
    <id column="id" property="id"/>
    <result column="last_name" property="lastName"/>
    <result column="gender" property="gender"/>
    <result column="email" property="email"/>
    <result column="did" property="dept.id"/>
    <result column="dept_name" property="dept.departmentName"/>
</resultMap>

<select id="getEmpAndDept" resultMap="MyDifEmp">
       select emp.id        id,
            emp.last_name last_name,
            emp.gender    gender,
            emp.email     email,
            dep.id        did,
            dep.dept_name dept_name
        from tbl_employee emp,
             tbl_dept dep
        where emp.d_id = dep.id
          and emp.id = #{id}
</select>
```

测试：

```java
Employee employee = mapper.getEmpAndDept(1);
System.out.println(employee);
```

结果：

```java
Setting autocommit to false on JDBC Connection [com.mysql.jdbc.JDBC4Connection@8b87145]
==>  Preparing: select emp.id id, emp.last_name last_name, emp.gender gender, emp.email email, dep.id did, dep.dept_name dept_name from tbl_employee emp, tbl_dept dep where emp.d_id = dep.id and emp.id = ? 
==> Parameters: 1(Integer)
<==    Columns: id, last_name, gender, email, did, dept_name
<==        Row: 1, lisi, 0, lisi@163.com, 1, 开发部
<==      Total: 1
Employee(id=1, lastName=lisi, email=lisi@163.com, gender=0, dept=Dept(id=1, departmentName=开发部))
```

### 1.3 association嵌套结果集

关联（association）元素可以处理多表查询。

MyBatis 有两种不同的方式加载关联：

- 嵌套结果映射：使用嵌套的结果映射来处理连接结果的重复子集。
- 嵌套 Select 查询：即分步查询，通过执行另外一个 SQL 映射语句来加载期望的复杂类型。

查询结果本质上是 Employee 类，其中的成员 Dept 是对象，需要进行关联 association，用 javaType 获取属性的类型。两种方式结果相同。

```xml
 <!--查询结果本质上是Employee类-->
<resultMap id="EmpAndDep" type="com.xiao.mybatis.entity.Employee">
     <!--对查询结果进行映射-->
    <!--数据库中的字段取了别名后用别名-->
    <id column="id" property="id"/>
    <result column="last_name" property="lastName"/>
    <result column="gender" property="gender"/>
    <result column="email" property="email"/>

    <!-- dept成员是一个对象，需要用association指定联合的JavaBean对象-->
    <!-- property="dept":指定哪个属性是联合的对象-->
    <!-- javaType:指定这个属性对象的类型，不能省略-->
    <association property="dept" javaType="com.xiao.mybatis.entity.Dept">
        <id column="did" property="id"/>
        <id column="dept_name" property="departmentName"/>
    </association>
</resultMap>

<select id="getEmpAndDept"  resultMap="EmpAndDep">
       select emp.id        id,
            emp.last_name last_name,
            emp.gender    gender,
            emp.email     email,
            dep.id        did,
            dep.dept_name dept_name
        from tbl_employee emp,
             tbl_dept dep
        where emp.d_id = dep.id
          and emp.id = #{id}
</select>
```

结果：

```java
Setting autocommit to false on JDBC Connection [com.mysql.jdbc.JDBC4Connection@8b87145]
==>  Preparing: select emp.id id, emp.last_name last_name, emp.gender gender, emp.email email, dep.id did, dep.dept_name dept_name from tbl_employee emp, tbl_dept dep where emp.d_id = dep.id and emp.id = ? 
==> Parameters: 1(Integer)
<==    Columns: id, last_name, gender, email, did, dept_name
<==        Row: 1, lisi, 0, lisi@163.com, 1, 开发部
<==      Total: 1
Employee(id=1, lastName=lisi, email=lisi@163.com, gender=0, dept=Dept(id=1, departmentName=开发部))
```

### 1.4 association分步查询

思路：

1. 先根据输入的员工 id 查询员工信息
2. 再根据查询到的员工信息中的 d_id，查询对应的部门信息
3. 将部门信息设置给员工

```xml
 <!--association分步查询-->
 <!--第一步,先查询员工信息-->
 <!--第二步,根据查询到的员工信息的did，寻找对应的部门信息-->
 <select id="getEmpAndDept" resultMap="EmpAndDep2">
     <!--第一步,先查询员工信息-->
     select id, last_name, gender,email,d_id from tbl_employee  where id = #{id}
 </select>
 <resultMap id="EmpAndDep2" type="com.xiao.mybatis.entity.Employee">
     <!--简单的属性直接写-->
     <id column="id" property="id"/>
     <result column="last_name" property="lastName"/>
     <result column="gender" property="gender"/>
     <result column="email" property="email"/>
     <!--复杂的属性，需要单独处理-->
     <!--select：调用指定的方法查出结果-->
     <!--column：将哪一列的值传给这个方法-->
     <!--property：查出的结果封装给property指定的属性-->
    <association property="dept" column="d_id" javaType="com.xiao.mybatis.entity.Dept" select="getDept"/>

</resultMap>
<!--第二步,根据查询到的员工信息中的d_id，查询对应的部门信息-->
<select id="getDept" resultType="com.xiao.mybatis.entity.Dept">
    select id,dept_name departmentName from tbl_dept where id = #{d_id}
</select>
```

 association标签中的属性：

- `select`：调用指定的方法查出结果
- `column`：将哪一列的值传给这个方法
- `property`：查出的结果封装给property指定的属性

**执行流程：使用 select 指定的方法，用传入的 column 指定的这列参数的值，查出对象，并封装给 property属性**

结果：有两条 sql 语句

```java
Setting autocommit to false on JDBC Connection [com.mysql.jdbc.JDBC4Connection@8b87145]
==>  Preparing: select id, last_name, gender,email,d_id from tbl_employee where id = ? 
==> Parameters: 1(Integer)
<==    Columns: id, last_name, gender, email, d_id
<==        Row: 1, lisi, 0, lisi@163.com, 1
<==      Total: 1
==>  Preparing: select id, dept_name departmentName from tbl_dept where id = ? 
==> Parameters: 1(Integer)
<==    Columns: id, departmentName
<==        Row: 1, 开发部
<==      Total: 1
Employee(id=1, lastName=lisi, email=lisi@163.com, gender=0, dept=Dept(id=1, departmentName=开发部))
```



> 分步查询可以使用延迟加载

延迟加载(懒加载、按需加载)，需要在全局配置文件中开启

- lazyLoadingEnabled：延迟加载的全局开关。当开启时，所有关联对象都会延迟加载。 特定关联关系中可通过设置 `fetchType` 属性来覆盖该项的开关状态。默认是 false
- aggressiveLazyLoading：开启时，任一方法的调用都会加载该对象的所有延迟加载属性。 否则，每个延迟加载属性会按需加载。（在 3.4.1 及之前的版本中默认为 true，之后默认为 false）

```xml
<settings>
    <!--开启懒加载-->
    <setting name="lazyLoadingEnabled" value="true"/>
    <setting name="aggressiveLazyLoading" value="false"/>  
</settings>
```

测试中只使用查询到的员工信息，不使用部门信息，则只会发出一条 sql 语句，如果不开启延迟加载，则总是会发出两条 sql 语句： 

```java
Employee employee = mapper.getEmpAndDept(1);
System.out.println(employee.getEmail());
```

结果，只执行了第一步的 sql 语句：

```java
Setting autocommit to false on JDBC Connection [com.mysql.jdbc.JDBC4Connection@8b87145]
==>  Preparing: select id, last_name, gender,email,d_id from tbl_employee where id = ? 
==> Parameters: 1(Integer)
<==    Columns: id, last_name, gender, email, d_id
<==        Row: 1, lisi, 0, lisi@163.com, 1
<==      Total: 1
lisi@163.com
```

关联关系中设置 `fetchType="eager"` ，则会忽略全局配置参数 `lazyLoadingEnabled`，使用改属性的值，执行了两条 sql 语句。

```xml
<association property="dept" column="d_id" javaType="com.xiao.mybatis.entity.Dept" select="getDept"
                     fetchType="eager"/>
```

## 2 collection--一对多查询

查询部门，并将部门对应的所有员工信息查询出来

Employee 实体类：

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
public class Employee {
    private Integer id;
    private String lastName;
    private String email;
    private String gender;
}
```

Dept 实体类：

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
public class Dept {
    private Integer id;
    private String departmentName;
    private List<Employee> emps;
}
```

DeptMapper 接口文件：

```java
public interface DeptMapper {
    Dept selectDeptById(Integer id);
}
```

### 2.1 嵌套结果集

使用关联查询，根据 Dept 的 id 查询 Dept 信息，其中 Dept 中有一个成员是 emps，即员工的集合，需要用到**collection 标签**进行结果映射。

集合标签：`collection`

- `property`：指定哪个属性是集合
- `ofType`：获取集合中的泛型信息

sql映射文件：

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.xiao.dao.DeptMapper">

    <!--嵌套结果查询-->
    <select id="selectDeptById" resultMap="MyDept">
        select d.id did, d.dept_name dept_name, e.id eid, e.last_name last_name, e.email email, e.gender gender
        from tbl_dept d
        left join tbl_employee e
        on d.id = e.d_id
        where d.id = #{id}
    </select>

    <resultMap id="MyDept" type="com.xiao.mybatis.entity.Dept">
        <id column="did" property="id"/>
        <result column="dept_name" property="departmentName"/>
        <!--复杂的属性，单独处理，集合：collection，property指定哪个属性是集合，用ofType获取集合中的泛型信息-->
        <collection property="emps" ofType="com.xiao.mybatis.entity.Employee">
            <result column="eid" property="id"/>
            <result column="last_name" property="lastName"/>
            <result column="email" property="email"/>
            <result column="gender" property="gender"/>
        </collection>
    </resultMap>

</mapper>
```

测试：

```java
Dept dept = mapper.selectDeptById(1);
System.out.println(dept);
```

查询结果，一个部门中有两个员工信息：

```java
==>  Preparing: select d.id did, d.dept_name dept_name, e.id eid, e.last_name last_name, e.email email, e.gender gender from tbl_dept d left join tbl_employee e on d.id = e.d_id where d.id = ? 
==> Parameters: 1(Integer)
<==    Columns: did, dept_name, eid, last_name, email, gender
<==        Row: 1, 开发部, 1, lisi, lisi@163.com, 0
<==        Row: 1, 开发部, 2, wangwu, wangwu@126.com, 1
<==      Total: 2
Dept(id=1, departmentName=开发部, emps=[Employee(id=1, lastName=lisi, email=lisi@163.com, gender=0), Employee(id=2, lastName=wangwu, email=wangwu@126.com, gender=1)])
```

### 2.2 分步查询

思路与多对一中的思路类型：

1. 先根据输入的部门 id 查询部门信息
2. 再根据查询到的部门信息中的部门 id，查询对应的员工信息
3. 将员工信息设置给部门

```xml
<!--分步查询-->
<select id="selectDeptById" resultMap="MyDept">
    select id, dept_name
    from tbl_dept
    where id = #{id}
</select>

<resultMap id="MyDept" type="com.xiao.mybatis.entity.Dept">
    <id column="id" property="id"/>
    <result column="dept_name" property="departmentName"/>
    <collection property="emps" ofType="com.xiao.mybatis.entity.Employee" select="getEmp" column="id"/>
</resultMap>

<select id="getEmp" resultType="com.xiao.mybatis.entity.Employee">
    select id,last_name lastName,email,gender from tbl_employee where d_id = #{id}
</select>
```

查询结果，执行了两条 SQL 语句：

```java
==>  Preparing: select id, dept_name from tbl_dept where id = ? 
==> Parameters: 1(Integer)
<==    Columns: id, dept_name
<==        Row: 1, 开发部
<==      Total: 1
==>  Preparing: select id,last_name lastName,email,gender from tbl_employee where d_id = ? 
==> Parameters: 1(Integer)
<==    Columns: id, lastName, email, gender
<==        Row: 1, lisi, lisi@163.com, 0
<==        Row: 2, wangwu, wangwu@126.com, 1
<==      Total: 2
Dept(id=1, departmentName=开发部, emps=[Employee(id=1, lastName=lisi, email=lisi@163.com, gender=0), Employee(id=2, lastName=wangwu, email=wangwu@126.com, gender=1)])
```



### 2.3 补充几个小点

1. 分步查询如何传递多列的值？

   将多列的值封装成 map 传递：column={key1=column1,key2=column2...}，使用时则用 #{key} 获取

2. 开启了全局懒加载后，特定关联关系中可通过设置 `fetchType` 属性来覆盖该项的开关状态

- fetchType="lazy"：懒加载

- fetchType="eager"：立即加载

3. **鉴别器 discriminator**

   类似于 Java 语言中的 switch 语句，可以根据不同的条件封装不同的结果集。需要指定 column 和 javaType 属性。

   - javaType：列值对应的java类型
   - column：指定用于判定的列名
   - value：进行条件的判断，满足该条件，则选择该结果封装规则

```xml
<discriminator javaType="string" column="gender">
  <case value="0" resultType="com.xiao.mybatis.entity.Employee">
      ....
  </case>
  <case value="1" resultType="com.xiao.mybatis.entity.Employee">
      ....
  </case>
</discriminator>
```

