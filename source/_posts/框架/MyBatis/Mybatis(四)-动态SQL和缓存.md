---
title: MyBatis (四)-动态 SQL 和缓存
categories:
  - 框架
  - MyBatis
tags:
  - MyBatis
  - 动态 SQL
date: 2021-06-05
description: 基于 OGNL 的表达式 MyBatis 的动态 SQL 常用标签，根据不同的条件生成不同的 SQL 语句。
---



MyBatis 笔记 (四)，内容包括：

1. 动态SQL的几个标签
2. 缓存，包括一级缓存、二级缓存和第三方缓存

## 1. 动态SQL

动态 SQL 就是指根据不同的条件生成不同的 SQL 语句

MyBatis 采用功能强大的基于 OGNL 的表达式来淘汰其它大部分元素。

- `if`
- `choose (when, otherwise)`
- `trim (where, set)`
- `foreach`

数据库环境和实体类依然如下：

```sql
CREATE TABLE tbl_employee(
id INT(10) PRIMARY KEY AUTO_INCREMENT,  
last_name VARCHAR(255),
gender CHAR(1),
email VARCHAR(255)
)ENGINE=INNODB DEFAULT CHARSET=utf8;

INSERT INTO tbl_employee VALUES
(1,"zhansgan",0,"zhangsan@qq.com"),
(2,"lisi",0,"lisi@163.com"),
(3,"wangwu",1,"wangwu@126.com");
```

```java
public class Employee {
    private Integer id;
    private String lastName; 
    private String email;
    private String gender;
}
```

### 1.1 if

语法： **test**  属性可以拿到 `if `标签中的参数值

```xml
 <if test="...">
     ....
</if>
```

适用于多条件查询的场景，携带了哪个字段查询条件，就带上这个字段的值；即如果传入id，则根据id查询；如果同时传入 id 和 lastName ，则根据 id 和 lastName 查询。

接口文件中声明查询方法，传入的是一个对象作为条件：

```java
public interface EmployeeMapper {

    //定义一个查询方法，传入的参数是一个对象
    List<Employee> selectEmp(Employee employee);
    
}
```

sql映射文件写法：

```xml
<select id="selectEmp" resultType="com.xiao.mybatis.dynamic.entity.Employee">
    select *
    from tbl_employee
    where
    <if test="id!=null">
        id = #{id}
    </if>
    <if test="lastName!=null and lastName !='' ">
        and last_name = #{lastName}
    </if>
    <!--ognl会进行字符串与数字的转换判断-->
    <if test="gender==0 or gender == 1">
        and gender = #{gender}
    </if>
    <if test="email!=null">
        and email = #{email}
    </if>

</select>
```

测试：

```java
Employee employee = new Employee(1,null,null,"1");
List<Employee> list = mapper.selectEmp(employee);
```

执行的sql语句：

```java
==>  Preparing: select * from tbl_employee where id = ? and gender = ? 
==> Parameters: 1(Integer), 1(String)
```



### 1.2 where

上面的例子有一个弊端，就是如果传入的 id 值是 null ，而后续条件不为 null，则 sql 语句中会出现 where 后面直接跟一个 and 的情况。

解决方法：

1. 在查询语句中写 where 1=1，后面的每一个 `if` 标签的前面都加上 and
2. **将查询语句放入`where` 标签中**

`where` 标签只会在子元素返回任何内容的情况下才插入 “WHERE” 子句。

而且，若子句的开头为 “AND” 或 “OR”，`where` 标签也会将它们去除。

```xml
    <select id="selectEmp2" resultType="com.xiao.mybatis.dynamic.entity.Employee">
        select *
        from tbl_employee
        <where>
        <if test="id!=null">
            id = #{id}
        </if>
        <if test="lastName!=null and lastName !='' ">
            and last_name = #{lastName}
        </if>
        <!--ognl会进行字符串与数字的转换判断-->
        <if test="gender == 0 or gender == 1">
            and gender = #{gender}
        </if>
        <if test="email!=null">
            and email = #{email}
        </if>
        </where>
    </select>
```

### 1.3 trim

如果 `where` 标签与期望的不太一样，也可以通过自定义 `trim` 标签来定制 `where` 元素的功能。

比如，和 `where` 标签等价的自定义 trim 标签为：

属性：

- prefix：前缀，给trim标签体中整个拼接好的字符串加一个前缀
- prefixOverrides：前缀覆盖，去掉整个字符串前面多余的字符
- suffix：后缀，给trim标签体中整个拼接好的字符串加一个后缀
- suffixOverrides：后缀覆盖，去掉整个字符串后面多余的字符

```xml
<trim prefix="WHERE" prefixOverrides="AND |OR ">
  ...
</trim>
```

### 1.4 choose (when, otherwise)

`choose ` 标签，从多个条件中选择一个使用，类似于 Java 中的 switch 语句。

带了id条件就用id查询，带了 lastName 条件就用 lastName 查，否则就查 gender==0 的

```xml
    <select id="selectEmp3" resultType="com.xiao.mybatis.dynamic.entity.Employee">
        select *
        from tbl_employee
        <where>
            <choose>
                <when test="id!=null">
                    id = #{id}
                </when>
                <when test="lastName!=null and lastName !='' ">
                    and last_name = #{lastName}
                </when>
                <otherwise>
                    and gender=0
                </otherwise>
            </choose>
        </where>
    </select>
```

###  1.5 set

与if结合使用，用于动态包含需要更新的列，忽略其它不更新的列，会动态地在行首插入 SET 关键字，并会删掉额外的逗号。

```xml
<update id="updateEmp">
    update tbl_employee
    <set>
        <if test="lastName!=null">
            last_name = #{lastName},
        </if>
        <if test="gender!=null">
            gender = #{gender},
        </if>
        <if test="email!=null">
            email = #{email}
        </if>
    </set>
    where id = #{id}
</update>
```

### 1.6 foreach

`foreach` 标签，指定一个集合，声明可以在元素体内使用的集合项（item）和索引（index）变量。也可以指定开头与结尾的字符串以及集合项迭代之间的分隔符（separator）。

- 可以将任何可迭代对象（如 List、Set 等）、Map 对象或者数组对象作为集合参数传递给`foreach`。
- 当使用可迭代对象或者数组时，index 是当前迭代的序号，item 的值是本次迭代获取到的元素。
- 当使用 Map 对象（或者 Map.Entry 对象的集合）时，index 是键，item 是值。

#### 01 查询条件是集合

如果要执行 `select  * from tbl_employee where id in (1,2,3) `语句，则可以写为：

接口文件：

```java
//根据传入的id集合查询
List<Employee> selectEmpByList(@Param("ids") List<Integer> ids);
```

sql映射文件：


```xml
<select id="selectEmpByList" resultType="com.xiao.mybatis.dynamic.entity.Employee">
    select  * from tbl_employee
    <foreach collection="ids" item="item_id" separator="," open="where id in (" close=")">
        #{item_id}
    </foreach>
</select>
```

属性说明：

- `collection`：要遍历的集合
- `item`：将当前遍历出的元素赋值给指定的变量
- `separator`：每个元素之间的分隔符
- `open`：遍历出所有的结果拼接一个开始的字符
- `close`：遍历出所有的结果拼接一个结束的字符

测试：

```java
EmployeeMapper mapper = sqlSession.getMapper(EmployeeMapper.class);
mapper.selectEmpByList(Arrays.asList(1, 2, 3));
```

执行的SQL语句：

```java
==>  Preparing: select * from tbl_employee where id in ( ? , ? , ? ) 
==> Parameters: 1(Integer), 2(Integer), 3(Integer)
```



#### 02 批量插入

批量插入即执行：insert into tbl_employee(last_name, gender, email) values (...),(...),(...)

接口文件中声明一个批量插入方法：

```java
//批量插入
void insertEmp(@Param("emps") List<Employee> employees);
```

sql映射文件：

```xml
<insert id="insertEmp">
    insert into tbl_employee(last_name, gender, email) values
    <foreach collection="emps" item="emp" separator=",">
        (#{emp.lastName},#{emp.gender},#{emp.email})
    </foreach>
</insert>
```

**也可以使用执行多次sql语句的方式：**

```xml
<insert id="insertEmp">
    <foreach collection="emps" item="emp" separator=";">
        insert into tbl_employee(last_name, gender, email)
        values (#{emp.lastName},#{emp.gender},#{emp.email})
    </foreach>
</insert>
```

这种方式需要开启MySQL的allowMultiQueries：允许一条语句中使用分号来分隔各语句

```properties
url=jdbc:mysql://localhost:3306/mybatis?useSSL=false&useUnicode=true&characterEncoding=UTF-8&allowMultiQueries=true
```

### 1.7 bind 绑定

可以将ognl表达式的值绑定到一个变量中，方便后面来引用这个值。

模糊查询中，只传入了lastName值，用bind标签在两端拼接了%，然后name属性为_lastName，给sql语句引用

```xml
<select id="selectEmp" resultType="com.xiao.mybatis.dynamic.entity.Employee">
  <bind name="_lastName" value="'%' + lastName + '%'" />
	select * from tbl_employee where last_name like #{_lastName}
</select>
```

### 1.8 sql 抽取可重用片段

1. 将经常要使用的片段用 sql 标签抽取出来，sql 标签中指定 id
2. 增删改标签里用 include 标签来引用，指定 refid 即可
3. include 标签内部还可以自定义 property 标签，name 属性指定列名，value 属性为值，sql 标签内部就能使用自定义的属性，用 $ 来使用

```xml
<sql id = "insert">
id,last_name,email
</sql>

<select>
   select
	<include refid = "insert"></include>
   from  tbl_employee
</select>



<sql id="userColumns"> ${alias}.id,${alias}.username,${alias}.password </sql>

<select id="selectUsers" resultType="map">
	select
		<include refid="userColumns"><property name="alias" value="t1"/></include>,
		<include refid="userColumns"><property name="alias" value="t2"/></include>
	from tbl_employee t1
		left join tbl_employee t2
</select>
```





### 1.9 两个内置参数

MyBatis 中有两个内置参数可以直接使用：

- _parameter：代表整个参数；如果是单个参数，那么它就是这个参数；如果是多个参数，则会被封装为一个map，那么它代表这个 map

- _databaseId：如果配置了 databaseIdProvider 标签，则代表的就是当前数据库的别名

## 2. 缓存机制

MyBatis 系统中默认定义了两级缓存：一级缓存和二级缓存。

- **默认情况下，只有一级缓存（SqlSession级别的缓存，也称为本地缓存）开启。**
- **二级缓存需要手动开启和配置，是基于 namespace 级别的缓存。**
- 为了提高扩展性。MyBatis 定义了缓存接口 Cache。可以通过实现 Cache 接口来自定义二级缓存

### 2.1 一级缓存

一级缓存 (local cache)，即本地缓存, 作用域默认为 SqlSession ，即一次数据库连接，是一直开启的。

同一次会话期间只要查询过的数据都会保存在当前 SqlSession 的一个Map中：key--hashCode+查询的SqlId+编写的 sql 查询语句+参数

```java
Employee employee1 = mapper.selectEmpById(1);
System.out.println(employee1);
Employee employee2 = mapper.selectEmpById(1);
System.out.println(employee2);
System.out.println(employee1==employee2); //true
```

**一级缓存失效的情况：**

1. 不同的 SqlSession 对应不同的一级缓存
2. 同一个 SqlSession 但是查询条件不同
3. 同一个 SqlSession 两次查询期间执行了任何一次增删改操作
4. 同一个 SqlSession 两次查询期间手动清空了缓存，调用 sqlSession.clearCache() 方法

全局配置文件中：**localCacheScope** 属性即 MyBatis 的本地缓存机制，默认值为 SESSION，会缓存一个会话中执行的所有查询。 若设置值为 STATEMENT，本地缓存将仅用于执行语句，对相同 SqlSession 的不同查询将不会进行缓存，即禁用了一级缓存。

### 2.2 二级缓存

#### 01 工作机制

一级缓存的作用域比较小，需要使用到二级缓存 (second level cache)，即全局作用域缓存，**是基于namespace级别的缓存**

- 二级缓存默认不开启，需要手动配置
- MyBatis 提供二级缓存的接口以及实现，缓存实现要求 POJO 实现 Serializable 接口
- 二级缓存在 SqlSession 关闭或提交之后才会生效

工作机制：

1. 一个会话查询一条语句，这个数据就会被放在当前会话的一级缓存中；

2. **如果会话关闭，一级缓存中的数据才会被保存到二级缓存中**，新的会话信息就可以参照二级缓存中的内容

3. 不同 namespace 查出的数据会被放在自己对应的缓存中 (map)，EmployeeMapper ==> Employee；DepartmentMapper ==> Department

#### 02 使用步骤

1. 全局配置文件中开启二级缓存

   ```xml
   <settings>
       <!--开启二级缓存-->
       <setting name="cacheEnabled" value="true"/>
   </settings>
   ```

2. 需要使用二级缓存的 sql 映射文件中使用 cache 标签配置缓存
```xml
   <cache eviction="FIFO" flushInterval="60000" readOnly="false" size="1024">
   
   </cache>
```

3. 注意：POJO 需要实现 Serializable 接口

4. 开启两个 sqlSession 进行测试：

   ```java
         SqlSession sqlSession = sqlSessionFactory.openSession();
         EmployeeMapper mapper = sqlSession.getMapper(EmployeeMapper.class);
   	  Employee employee1 = mapper.selectEmpById(1);
         System.out.println(employee1);
         sqlSession.close();
   
         SqlSession sqlSession2 = sqlSessionFactory.openSession();
         EmployeeMapper mapper2 = sqlSession2.getMapper(EmployeeMapper.class);
         Employee employee2 = mapper2.selectEmpById(1);
         System.out.println(employee2);
         System.out.println(employee1 == employee2); //false
         sqlSession2.close();
   ```

   结果：

   ```java
   Setting autocommit to false on JDBC Connection [com.mysql.jdbc.JDBC4Connection@140e5a13]
   ==>  Preparing: select * from tbl_employee where id = ? 
   ==> Parameters: 1(Integer)
   <==    Columns: id, last_name, gender, email, d_id
   <==        Row: 1, lisi, 0, lisi@163.com, 1
   <==      Total: 1
   Employee(id=1, lastName=lisi, email=lisi@163.com, gender=0)
   Resetting autocommit to true on JDBC Connection [com.mysql.jdbc.JDBC4Connection@140e5a13]
   Closing JDBC Connection [com.mysql.jdbc.JDBC4Connection@140e5a13]
   Returned connection 336484883 to pool.
   Cache Hit Ratio [com.xiao.mybatis.dynamic.mapper.EmployeeMapper]: 0.5
   Employee(id=1, lastName=lisi, email=lisi@163.com, gender=0)
   false
   ```
   
   注意：**第一个sqlSession.close()后，才会把一级缓存存入二级缓存**，第二次缓存命中 Cache Hit Ratio 为0.5，即第二次查缓存，命中了一次，没有执行sql

#### 03 参数说明

**eviction：缓存回收策略 **

- LRU –– 最近最少使用的：移除最长时间不被使用的对象。

- FIFO –– 先进先出：按对象进入缓存的顺序来移除它们。

- SOFT –– 软引用：移除基于垃圾回收器状态和软引用规则的对象。

- WEAK –– 弱引用：更积极地移除基于垃圾收集器状态和弱引用规则的对象。

  默认的是 LRU。

**flushInterval：刷新间隔，单位毫秒**

- 默认情况是不设置，也就是没有刷新间隔，缓存仅仅调用语句时刷新


**size：引用数目，正整数**

  - 代表缓存最多可以存储多少个对象，太大容易导致内存溢出

**readOnly：只读，true/false**

  - true：只读缓存；直接将数据在缓存中的引用交给用户。加快了获取速度，不安全
  - false：读写缓存；会返回缓存对象的拷贝（通过序列化）。这会慢一些，但是安全，因此默认是 false。

type：可以指定自定义缓存的全类名，实现 cache 接口

#### 04 缓存有关设置小结

1. **全 局setting 的 cacheEnable**：配置二级缓存的开关，置为 alse，关闭的是二级缓存，一级缓存一直是打开的
2. **select 标签的useCache属性**：配置这个 select 是否使用二级缓存。一级缓存一直是使用的
3. **sql标签的flushCache属性**：增删改标签中默认 flushCache=true，即清除缓存，**sql 执行以后，会同时清空一级和二级缓存**。查询标签默认 flushCache=false
4. **sqlSession.clearCache()**：只是用来清除一级缓存。
5. 当在某一个作用域(一级缓存 Session/二级缓存 Namespaces) 进行了 C/U/D 操作后，默认该作用域下所有select 中的缓存将被 clear。

![](http://image.kongxiao.top/image-20200613174757061.png)

### 2.3 第三方缓存

EhCache 是一个纯 Java 的进程内缓存框架，具有快速、精干等特点，是 Hibernate 中默认的 CacheProvider。

MyBatis 定义了 Cache 接口方便我们进行自定义扩展。

1. 添加依赖

```xml
<!-- https://mvnrepository.com/artifact/org.mybatis.caches/mybatis-ehcache -->
<dependency>
    <groupId>org.mybatis.caches</groupId>
    <artifactId>mybatis-ehcache</artifactId>
    <version>1.1.0</version>
</dependency>

```

2. 在 sql 映射文件中配置 cache 标签

```xml
	 <!--加入使用缓存-->
    <cache type="org.mybatis.caches.ehcache.EhcacheCache">
        <!--缓存自创建日期起至失效时的间隔时间一个小时-->
        <property name="timeToIdleSeconds" value="3600"/>
        <!--缓存创建以后，最后一次访问缓存的日期至失效之时的时间间隔一个小时-->
        <property name="timeToLiveSeconds" value="3600"/>
        <!--设置在缓存中保存的对象的最大的个数，这个按照业务进行配置-->
        <property name="maxEntriesLocalHeap" value="1000"/>

        <!--设置在磁盘中最大实体对象的个数-->
        <property name="maxEntriesLocalDisk" value="10000000"/>
        <!--缓存淘汰算法-->
        <property name="memoryStoreEvictionPolicy" value="LRU"/>
    </cache>
```

或者单独建立一个 ehcache.xml 配置文件：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<ehcache xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
 xsi:noNamespaceSchemaLocation="../config/ehcache.xsd">
 <!-- 磁盘保存路径 -->
 <diskStore path="D:\44\ehcache" />
 
 <defaultCache 
   maxElementsInMemory="10000" 
   maxElementsOnDisk="10000000"
   eternal="false" 
   overflowToDisk="true" 
   timeToIdleSeconds="120"
   timeToLiveSeconds="120" 
   diskExpiryThreadIntervalSeconds="120"
   memoryStoreEvictionPolicy="LRU">
 </defaultCache>
</ehcache>
 
<!-- 
属性说明：
diskStore：指定数据在磁盘中的存储位置。
defaultCache：当借助CacheManager.add("demoCache")创建Cache时，EhCache便会采用<defalutCache/>指定的的管理策略
 
以下属性是必须的：
maxElementsInMemory - 在内存中缓存的element的最大数目 
maxElementsOnDisk - 在磁盘上缓存的element的最大数目，若是0表示无穷大
eternal - 设定缓存的elements是否永远不过期。如果为true，则缓存的数据始终有效，如果为false那么还要根据timeToIdleSeconds，timeToLiveSeconds判断
overflowToDisk - 设定当内存缓存溢出的时候是否将过期的element缓存到磁盘上
 
以下属性是可选的：
timeToIdleSeconds - 当缓存在EhCache中的数据前后两次访问的时间超过timeToIdleSeconds的属性取值时，这些数据便会删除，默认值是0,也就是可闲置时间无穷大
timeToLiveSeconds - 缓存element的有效生命期，默认是0.,也就是element存活时间无穷大
diskSpoolBufferSizeMB 这个参数设置DiskStore(磁盘缓存)的缓存区大小.默认是30MB.每个Cache都应该有自己的一个缓冲区.
diskPersistent - 在VM重启的时候是否启用磁盘保存EhCache中的数据，默认是false。
diskExpiryThreadIntervalSeconds - 磁盘缓存的清理线程运行间隔，默认是120秒。每个120s，相应的线程会进行一次EhCache中数据的清理工作
memoryStoreEvictionPolicy - 当内存缓存达到最大，有新的element加入的时候， 移除缓存中element的策略。默认是LRU（最近最少使用），可选的有LFU（最不常使用）和FIFO（先进先出）
 -->
```

参照缓存：若想在命名空间中共享相同的缓存配置和实例。可以使用 cache-ref 标签来引用另外一个缓存，指定namespace 即可

```xml
 <cache-ref namespace="..."/>
```

执行流程：

![](http://image.kongxiao.top/image-20200613180156203.png)

