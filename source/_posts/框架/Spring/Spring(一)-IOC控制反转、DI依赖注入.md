---
title: Spring (一) - IOC 控制反转、DI 依赖注入
categories:
  - 框架
  - Spring
tags:
  - Spring
  - IOC
  - Bean
date: 2021-06-10
description: Spring 入门，学会 Bean 的装配和属性注入，Spring 核心特性之 IOC。
---

Spring 基础知识学习笔记 (一)，内容包括：

1. Spring 入门案例
2. IOC 控制反转理解
3. 属性注入的不同方式
4. 注入不同类型的属性值
5. 自动装配与注解开发

参考视频：

B站 尚硅谷雷丰阳大神的 Spring、Spring MVC、MyBatis 课程 https://www.bilibili.com/video/BV1d4411g7tv

【狂神说Java】Spring5 最新完整教程 IDEA 版通俗易懂  https://www.bilibili.com/video/BV1WE411d7Dv

## 1. Spring 概述

- 开源的免费框架，是一个容器，可以管理所有的组件(类)；

- 轻量级的、非入侵的框架，不依赖于 Spring 的 API

- **控制反转(IOC)和面向切面编程(AOP)**

- 支持事务处理，支持对框架整合

- 组件化、一站式

  官网：https://spring.io/

  文档：https://docs.spring.io/spring/docs/5.2.5.RELEASE/spring-framework-reference/core.html#spring-core

【总结】：Spring 是一个轻量级的、控制反转和面向切面编程的框架

体系结构：

![image-20200613222724177](http://image.kongxiao.top/image-20200613222724177.png)

- Test：Spring 的单元测试模块
- Core Container：核心容器 (IOC)，包括 4 部分： 
  - spring-core：提供了框架的基本组成部分，包括 IoC 和依赖注入功能。
  - spring-beans：提供 BeanFactory，
  - spring-context：模块建立在由 core 和 beans 模块的基础上建立起来的，它以一种类似于 JNDI 注册的方式访问对象。Context 模块继承自 Bean 模块，并且添加了国际化（比如，使用资源束）、事件传播、资源加载和透明地创建上下文（比如，通过 Servelet 容器）等功能
  - spring-expression：提供了强大的表达式语言，用于在运行时查询和操作对象图。它是 JSP2.1 规范中定义的统一表达式语言的扩展，支持set和get属性值、属性赋值、方法调用、访问数组集合及索引的内容、逻辑算术运算、命名变量、通过名字从Spring IoC容器检索对象，还支持列表的投影、选择以及聚合等

- AOP+Aspects：面向切面编程模块

- Data Access：数据访问模块
- Web：Spring开发Web引用模块

导入依赖：spring-webmvc 包含的最广泛

```xml
<!-- https://mvnrepository.com/artifact/org.springframework/spring-webmvc -->
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-webmvc</artifactId>
    <version>5.2.0.RELEASE</version>
</dependency>
```

## 2. HelloWorld 案例

### 2.1 IOC 和 DI

**Inversion of Control：控制反转。**

控制，即资源的获取方式，包括：

- 主动式：要什么资源自己创建，对于复杂对象的创建时比较庞大的工程

- 被动式：资源的获取不是我们自己创建，而是交给容器创建。

  所谓容器，是用来管理所有的组件的(即有功能的类)；容器可以自动探查出哪些组件需要用到另一些组件

**DI：Dependency Injection，依赖注入**，是 IOC 的一种实现形式。容器能知道哪个组件运行时需要另外一个类，容器通过反射的形式，将容器中准备好的对象注入。

### 2.2 入门案例

HelloWorld：所有的对象交给容器创建，给容器中注册组件

1. 新建一个 Person 类，**添加 set 方法**

   ```java
   @Data
   @AllArgsConstructor
   @NoArgsConstructor
   public class Person {
       private String lastName;
       private Integer age;
       private String gender;
       private  String email;
   }
   ```
   
2. 新建一个 Spring 配置文件 ApplicationContext.xml，注册 bean。

   使用`bean`标签注册一个 Person 对象，Spring会自动创建这个 Person 对象

   - class：写要注册的组件的全类名
   -  id：这个对象的唯一标识
   - 使用 `property` 标签为 Person 对象的属性值，name：指定属性名；value：指定属性值

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <beans xmlns="http://www.springframework.org/schema/beans"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://www.springframework.org/schema/beans
           https://www.springframework.org/schema/beans/spring-beans.xsd">
   
       <!--注册一个Person对象，Spring会自动创建这个Person对象
           class:写要注册的组件的全类名
           id:这个对象的唯一标识
       -->
       <bean id="person01" class="com.xiao.helloworld.bean.Person">
           <!--使用property标签为Person对象的属性赋值
               name:指定属性名
               value:指定属性值
            -->
           <property name="lastName" value="zhangsan"/>
           <property name="age" value="20"/>
           <property name="email" value="zhangsan@163.com"/>
           <property name="gender" value="0"/>
       </bean>
   </beans>
   ```

   3. 测试：

   ```java
   public class com.xiao.helloworld.IocTest {
       @Test
       public void test(){
           ApplicationContext ioc = new ClassPathXmlApplicationContext("ApplicationContext.xml");
           Person bean = (Person) ioc.getBean("person01");
           System.out.println(bean);
           Person bean2 = (Person) ioc.getBean("person01");
           System.out.println(bean == bean2);
           Person bean3 = ioc.getBean(Person.class);
           System.out.println(bean == bean3 );
       }
   }
   ```
   
   结果：
   
   ```java
   Person(lastName=zhangsan, age=20, gender=0, email=zhangsan@163.com)
   true
   true
   ```
   
   
   
   >  几个细节：
   
   1.  ApplicationContext：IOC 容器的接口
   3. **同一个组件在 IOC 容器中默认是单实例的**
   3.  **容器中的对象的创建在容器创建完成的时候就已经创建好了**
   4.  容器中如果没有这个组件，获取组件时会报异常 NoSuchBeanDefinitionException
   5.  IOC 容器用 `property` 标签创建这个组件对象的时候，会利用 setter 方法为其属性赋值，**注意属性名是set方法后的那串的首字母小写**

### 2.3 根据 bean 类型获取 bean 实例

ioc.getBean() 方法中可以传入 bean 的 id，也可以传入class对象，也可以同时传入。

如果一个类型指只注册了一个，则可以通过`ioc.getBean(....class)`获得该对象

```java
Person bean1 = ioc.getBean(Person.class);
```

但是如果 IOC 容器中这个类型的 bean 有多个，则会报异常 NoUniqueBeanDefinitionException

也可以同时传入 bean 的 id 和 class 对象：

```java
Person bean1 = ioc.getBean("person02",Person.class);
```

## 3. 属性的注入方式

- 依赖：bean 对象的创建依赖于容器

- 注入：bean 对象中所有的属性由容器来注入

### 3.1 setter 注入

**需要借助 set 方法**，使用`propetry`标签

```xml
 <property name="lastName" value="zhangsan"/>
```

### 3.2 通过构造器注入

使用`constructor-arg`标签，则调用构造器进行属性注入，**需要借助有参构造**

- **通过构造函数中的参数名称注入**

```xml
<bean id="person02" class="com.xiao.helloworld.bean.Person">
    <constructor-arg name="lastName" value="wangwu"/>
    <constructor-arg name="age" value="30"/>
    <constructor-arg name="gender" value="1"/>
    <constructor-arg name="email" value="wangwu@qq.com"/>
</bean>
```

- **只写 value 属性，会默认按顺序寻找构造方法进行匹配**

```xml
<bean id="person03" class="com.xiao.helloworld.bean.Person">
    <constructor-arg  value="wangwu"/>
    <constructor-arg  value="30"/>
    <constructor-arg  value="1"/>
    <constructor-arg value="wangwu@qq.com"/>
</bean>
```

- **通过构造函数参数类型**，默认按照顺序

```xml
<bean id="person04" class="com.xiao.helloworld.bean.Person">
    <constructor-arg type="java.lang.String" value="wangwu"/>
    <constructor-arg type="java.lang.Integer" value="30"/>
    <constructor-arg type="java.lang.String" value="1"/>
    <constructor-arg type="java.lang.String" value="wangwu@qq.com"/>
</bean>
```

- **通过构造函数参数索引**，如果有多个重载的构造函数时也可以配合 type 一起使用

```xml
<bean id="person05" class="com.xiao.helloworld.bean.Person">
    <constructor-arg index="0" value="wangwu"/>
    <constructor-arg index="1" value="30"/>
    <constructor-arg index="2" value="1"/>
    <constructor-arg index="3" value="wangwu@qq.com"/>
</bean>
```

### 3.3 p 名称空间注入

使用 p:propertyName 直接注入属性的值。本质上还是调用的 set 方法

导入头文件约束：

```xml
  xmlns:p="http://www.springframework.org/schema/p"
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:p="http://www.springframework.org/schema/p"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">
    
    <bean id="person06" class="com.xiao.helloworld.bean.Person"
        p:lastName="zhangsan" p:age="30" p:email="zhangsan@qq.com" p:gender="1">
    </bean>

</beans>
```

### 3.4 c 命名空间注入

c (构造: Constructor) 命名空间注入，使用 c:propertyName 注入属性值,**本质上使用的是构造器注入**

导入头文件约束：

```xml
 xmlns:c="http://www.springframework.org/schema/c"
```

```xml
  <bean id="person07" class="com.xiao.bean.Person"
          c:lastName="zhangsan" c:age="30" c:email="zhangsan@qq.com" c:gender="1">
    </bean>
```

## 4. 注入不同类型的属性值

新建了一个 Student 类和一个 Address 类，来测试不同类型的属性值注入：

Student 类：

```java
@Data
public class Student {
    private String name;
    private Address address;
    private String[] books;
    private List<String> hobbies;
    private Map card;
    private Set<String> games;
    private String wife;
    private Properties info;
}
```

Address 类：

```java
@Data
public class Address {
    private String name;
    private Integer num;
}
```

### 4.1 注入基本类型值

之前的例子都是注入基本类型的属性。如果不赋值的话，会使用属性的默认值

### 4.2 注入 null

如果有属性给了初始值，想注入为 null，则在 property 内部需要使用 `null` 标签：

```xml
    <bean id="student01" class="com.xiao.helloworld.bean.Student">
        <property name="name">
            <null/>
        </property>
    </bean>
```

**注意，使用 value="null" 是不对的：**

```xml
 <bean id="student01" class="com.xiao.helloworld.bean.Student">
        <property name="name" value="null"/>
  </bean>
```

上面的用法虽然对象的 name 属性打印出来是 null，但是 bean.getName()==null 是 false：

```java
Student bean = ioc.getBean("student01", Student.class);
System.out.println(bean);
System.out.println(bean.getName()==null);  //false
```

结果：

```java
Student(name=null, address=null, books=null, hobbies=null, card=null, games=null, wife=null, info=null)
true
```

### 4.3 注入 bean

可以使用` ref `引用外部的值：

```xml
<!--先注册一个Address对象-->
<bean id="address01" class="com.xiao.helloworld.bean.Address">
    <property name="name" value="beijing"/>
    <property name="num" value="001"/>
</bean>

<bean id="student02" class="com.xiao.helloworld.bean.Student">
    <!--通过id值引用-->
    <property name="address" ref="address01"/>
</bean>
```

要注意，**ref 是严格的引用**，通过容器拿到的 Address 实例就是 Student 实例中的 Address 属性

```java
Address address01 = ioc.getBean("address01", Address.class);
Student student02 = ioc.getBean("student02", Student.class);
System.out.println(student02);
System.out.println(student02.getAddress() == address01);  //true
```

也可以**引用内部 bean**，在`property`标签体中再定义bean，这个 Address 和外面的没有关系，**只能内部使用，外面获取不到**：

```xml
<bean id="student03" class="com.xiao.helloworld.bean.Student">
    <property name="address">
        <bean class="com.xiao.helloworld.bean.Address">
            <property name="name" value="tianijng"/>
            <property name="num" value="002"/>
        </bean>
    </property>
</bean>
```

### 4.3 集合类型赋值

#### 01 数组

`array`标签+`value`标签：

```xml
<property name="books">
    <array>
        <value>西游记</value>
        <value>红楼梦</value>
        <value>水浒传</value>
    </array>
</property>
```

#### 02 List

`list`标签+`value`标签：

```xml
<property name="hobbies">
    <list>
        <value>玩游戏</value>
        <value>看电影</value>
    </list>
</property>
```

#### 03 Map

`map`标签+`entry`标签，`entry`也可以使用ref引用：

```xml
<property name="card">
    <map>
        <entry key="中行" value="001"/>
        <entry key="邮政" value="002"/>
        <entry key-ref="..." value-ref="...."/>
    </map>
</property>
```

#### 04 Properties

`props`标签：

```xml
 <property name="info">
     <props>
         <prop key="学号">20190604</prop>
         <prop key="性别">男</prop>
         <prop key="姓名">小明</prop>
     </props>
 </property>
```

#### 05 util 名称空间

util 名称空间可以创建集合类型的 bean，以便别的地方引用。

头文件约束：

```xml
xmlns:util="http://www.springframework.org/schema/util"

xsi:schemaLocation= "http://www.springframework.org/schema/util
http://www.springframework.org/schema/util/spring-util-4.1.xsd"
```

```xml
<!--util名称空间 提取出通用的集合-->
    <util:list id="myList">
        <value>玩游戏</value>
        <value>看电影</value>
    </util:list>
<!--使用ref直接引用util提取出来的集合id即可-->
    <bean id="student05" class="com.xiao.helloworld.bean.Student">
        <property name="hobbies" ref="myList"/>
    </bean>
```

### 4.4 级联属性赋值

`propetry `标签中的 name 标签，可以使用级联属性，修改属性的属性，但是原来属性的值会被修改。

```xml
<bean id="address01" class="com.xiao.helloworld.bean.Address">
     <property name="name" value="beijing"/>
     <property name="num" value="001"/>
</bean>

<bean id="student02" class="com.xiao.helloworld.bean.Student">
    <property name="address" ref="address01"/>
    <!--将address01中的num属性进行了修改-->
    <property name="address.num" value="00005"/>
</bean>
```

### 4.5 继承实现配置信息重用

指定 parent 属性为要重用的 bean 的 id 值，不写的属性就沿用，也可以重写定义属性

```xml
<bean id="person01" class="com.xiao.helloworld.bean.Person">
    <property name="lastName" value="zhangsan"/>
    <property name="age" value="20"/>
    <property name="email" value="zhangsan@163.com"/>
    <property name="gender" value="0"/>
</bean>

<!--parent：要重用的配置信息 -->
<bean id="person001" class="com.xiao.helloworld.bean.Person" parent="person01">
    <!--单独修改name属性的值 -->
    <property name="lastName" value="zhang"/>
</bean>
```

还可以指定属性 abstract="true"，这样的 bean 只能被用来继承信息，不能获取实例。否则会报异常 BeanIsAbstractException

```xml
<bean id="person01" class="com.xiao.helloworld.bean.Person" abstract="true">
    <!--使用property标签为Person对象的属性赋值
        name:指定属性名
        value:指定属性值
     -->
    <property name="lastName" value="zhangsan"/>
    <property name="age" value="20"/>
    <property name="email" value="zhangsan@163.com"/>
    <property name="gender" value="0"/>
</bean>
```

## 5 bean 的一些性质

### 5.1 bean 之间依赖

多个 bean 的默认创建顺序，是按照配置顺序创建的。

```xml
<bean id="student" class="com.xiao.bean.Student"></bean>
<bean id="address" class="com.xiao.bean.Address"></bean>
<bean id="person" class="com.xiao.bean.Person"></bean>
```

```
Student创建了
Address创建了
Person创建了
```

可以用 depends-on 属性进行设置：

```xml
<bean id="student" class="com.xiao.bean.Student" depends-on="person,address"></bean>
<bean id="address" class="com.xiao.bean.Address"></bean>
<bean id="person" class="com.xiao.bean.Person"></bean>
```

```
Person创建了
Address创建了
Student创建了
```

### 5.2 bean 的作用域 scope

在 bean 配置中可以设置作用域属性 scope：

- single： 单例模式，是默认模式。**在容器启动完成之前就已经创建好对象保存在容器中了。**
- prototype ：原型模式，容器启动会不去创建，**每次从容器中 get 的时候才会产生一个新对象**

- request：在 web 环境下，同一次请求创建一个 bean 实例(没用)
- session：在 web 环境下，同一次会话创建一个 bean 实例(没用)

### 5.3 静态工厂与实例工厂

静态工厂：工厂本身不用创建对象，通过静态方法调用，对象 = 工厂类.工厂方法名( )

实例工厂：工厂本身需要创建对象，先创建工厂对象，再通过工厂对象创建所需对象

新建三个类 Air、AirStaticFactory和AirInstanceFactory：

```java
public class Air {
    private String name;
    private Double weight;
    private Double length;
    private Integer PersonNum;
    //get/set...
}
```

```java
public class AirStaticFactory {
    //提供一个静态方法获取Air对象
    public static Air getAir(String name){
        System.out.println("AirStaticFactory正在造飞机！");
        Air air = new Air();
        air.setName(name);
        air.setLength(100.0);
        air.setWeight(100.0);
        air.setPersonNum(200);
        return air;
    }
}
```

```java
public class AirInstanceFactory {

    //提供一个方法获取Air对象
    public Air getAir(String name){
        System.out.println("AirInstanceFactory正在造飞机！");
        Air air = new Air();
        air.setName(name);
        air.setLength(100.0);
        air.setWeight(100.0);
        air.setPersonNum(200);
        return air;
    }
}
```

**静态工厂：**不需要创建工厂本身，class 指定静态工厂的全类名，factory-method 指定工厂方法

```xml
<!--静态工厂，不需要创建工厂本身,class指定静态工厂的全类名-->
<bean id="air01" class="com.xiao.AirStaticFactory" factory-method="getAir">
    <constructor-arg name="name" value="林青霞"/>
</bean>
```

```java
//获取到的就是Air的实例
Air air01 = ioc.getBean("air01",Air.class);
```

**实例工厂：**先创建示例工厂本身，再创建对象，指定当前对象的创建需要哪个工厂 factory-bean 和哪个方法  factory-method

```xml
 <!--实例工厂，需要先创建示例工厂本身-->
<bean id="airInstanceFactory" class="com.xiao.AirInstanceFactory">
</bean>
<!--指定当前对象的创建需要哪个工厂和哪个方法，不需要指定class了-->
<bean id="air02" factory-bean="airInstanceFactory" factory-method="getAir">
    <constructor-arg name="name" value="张学友"/>
</bean>
```

```java
Air air02 = ioc.getBean("air02",Air.class);
```

### 5.4 自定义工厂

**实现了 FactoryBean 接口的类**，是 Spring 可以认识的工厂类，Spring 会自动调用工厂方法创建对象。

```java
public class MyFactoryBeanImpl implements FactoryBean<Air> {

    //工厂方法，Spring会自动调用这个方法来创建对象并返回
    @Override
    public Air getObject() throws Exception {
        Air air = new Air();
        air.setName("zhangsan");
        return air;
    }

    //返回对象的类型,Spring会自动调用这个方法来确认创建的对象是什么类型
    @Override
    public Class<?> getObjectType() {
        return null;
    }

    //是单例模式吗？
    @Override
    public boolean isSingleton() {
        return false;
    }
}
```

注册工厂对象，会自动调用工厂方法返回对象：

```xml
<!--注册工厂对象，会自动调用工厂方法返回对象-->
<bean id="air03" class="com.xiao.MyFactoryBeanImpl">
</bean>
```

```java
Object air03 = ioc.getBean("air03");
```

这种类型，IOC 容器启动时不会创建实例，使用 getBean 时才会创建

### 5.5 bean 的生命周期方法

可以为 bean 自定义一些生命周期方法，Spring在创建或销毁bean时调用。`init-method`，`destroy-method`，不能有参数。

IOC 容器中注册的 bean：

- 单实例 bean：容器启动的时候就会创建好，容器关闭也会销毁创建的bean

  (容器启动)构造器 ---> 初始化方法 ---> （容器关闭)销毁方法

- 多实例 bean：获取的时候才去创建

  (容器启动)构造器 ---> 初始化方法 ，容器关闭不会调用 bean 的销毁方法

在 Air 类中新增两个方法：

```java
public class Air {
    private String name;
    private Double weight;
    private Double length;
    private Integer PersonNum;

    public void destroy(){
        System.out.println("销毁方法被调用了！");
    }

    public void init(){
        System.out.println("初始方法被调用了");
    }
}
```

```xml
<bean id="air04" class="com.xiao.Air" init-method="init" destroy-method="destroy"> </bean>
```

### 5.6 bean 的后置处理器

定义一个类实现 BeanPostProcessor 接口，其中两个方法 `postProcessBeforeInitialization` 和`postProcessAfterInitialization `会在调用初始化方法前后调用。需要注册这个实现类

即使没有定义初始化方法，这两个方法也会被调用。

```java
public class MyBeanPostProcessor implements BeanPostProcessor {

    /**
     * 前置处理器，在初始化方法之前调用
     * @param bean 传递过来的，将要初始化的bean
     * @param beanName
     * @return 经该方法处理之后可以返回一个新的bean
     * @throws BeansException
     */
    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("【"+beanName+"】将要调用初始化方法了..BeforeInitialization..这个bean是这样的：+【"+bean+"】");
        return bean;
    }

    /**
     * 后置处理器，在初始化方法之后调用
     * @param bean
     * @param beanName
     * @return 经该方法处理后返回给IOC容器保存的bean
     * @throws BeansException
     */
    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("【"+beanName+"】初始化方法调用完了..AfterInitialization..这个bean是这样的：+【"+bean+"】");
        return bean;
    }
}
```

```xml
<bean id="air04" class="com.xiao.Air" init-method="init" destroy-method="destroy">
</bean>
<bean id="myBeanPostProcessor" class="com.xiao.MyBeanPostProcessor"/>
```

结果：

```
【air04】将要调用初始化方法了..BeforeInitialization..这个bean是这样的：+【Air{name='null', weight=null, length=null, PersonNum=null}】
初始方法被调用了
【air04】初始化方法调用完了..AfterInitialization..这个bean是这样的：+【Air{name='null', weight=null, length=null, PersonNum=null}】
```

## 6. bean 的装配

### 6.1 Spring 管理连接池

配置 C3P0 的数据库连接池，注册一 ComboPooledDataSource 对象即可

```xml
<bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
    <property name="user" value="root"/>
    <property name="password" value="root"/>
    <property name="jdbcUrl" value="jdbc:mysql://localhost:3306/mybatis"/>
    <property name="driverClass" value="com.mysql.jdbc.Driver"/>
</bean>
```

### 6.2 引入外部配置文件

首先新建一个数据库连接池的配置文件 db.properties：

```properties
jdbc.driver=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/mybatis?useSSL=false&useUnicode=true&characterEncoding=utf8
jdbc.username=root
jdbc.password=root
```

需要用到 context 命名空间：

```xml
 xmlns:context="http://www.springframework.org/schema/context"
 xsi:schemaLocation="
       http://www.springframework.org/schema/context
       https://www.springframework.org/schema/context/spring-context.xsd"
```

使用`context:property-placeholder location=" ... "`标签导入数据库配置文件 db.properties，就可以用 $ 取出对应的属性了：

```xml
<bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
    <property name="driverClass" value="${jdbc.driver}"/>
    <property name="jdbcUrl" value="${jdbc.url}"/>
    <property name="user" value="${jdbc.username}"/>
    <property name="password" value="${jdbc.password}"/>
</bean>

<context:property-placeholder location="classpath:db.properties"/>
```

【一个小坑】：Spring 内部已经定义过一个 username 了，${username} 就是系统的用户名，所以这里定义的是 jdbc.username

### 6.3 基于 XML 的自动装配

自动装配是 Spring 满足 bean 依赖的一种方式。Spring 会在上下文中自动寻找，并给 bean 自动装配属性。

Spring 中的三种装配方式：

1. 在 xml 显示配置

2. 在 Java 中显示配置

3. 隐式的自动装配 bean

在`bean`标签中设置`autowire`属性：

```xml
<bean id="air05" class="com.xiao.Air" autowire="byName" ></bean>
```

- `autpwire="default/no"`：不自动装配

- `autpwire="byName"`：按照名字，以属性名作为id去容器中找到这个组件，为其赋值；如果找不到就装配 null

- `autpwire="byType"`：按照类型，以属性的类型作为查找依据去容器中找到这个组件，为其赋值，**该类型必须只有一个**，否则会报异常NoUniqueBeanDifinetionException；如果找不到就装配 null

- `autpwire="construction"`：按照构造器进行赋值：先按照有参构造器的参数类型进行装，如果没有就直接为组件装配null即可；如果按照类型有多个，就会把参数名作为id继续匹配，匹配到就自动装配，匹配不到就装配null。不会报错

## 7. 注解开发

### 7.1 不同层组件

1. 通过给 bean 上添加注解，可以快速的将 bean 加入到 IOC 容器中。创建 Dao、Service、Controller层所需要用到的注解：
   - `@Component`：组件，放在类上，将某个类注册到 Spring 中，**id 是类名首字母小写**。相当于：`<bean id=".." class="..">`
   - `@Repository`：Dao 持久化层
   - `@Servic`：Service 业务逻辑层
   - `@Controller`：Controller 控制器层。后面三个含义更清晰

2. 还需要告诉Spring，自动扫描加了注解的组件：**添加 context 名称空间，`<context:component-scan base-package="com.xiao"/>`**。还需要有 AOP 包的依赖。
3. **组件的 id 默认是类名首字母小写，作用于默认是单例**，可以修改。

```java
@Repository(value = "book")
@Scope(value = "prototype")
public class BookDao {

}
```

- `@Value`：注入值，注入基本数据类型和 String 类型数据

- ``@Scope``：标注作用域。singleton, prototype...

  细节：如果注解中有且只有一个属性要赋值时，且名称是 value，value 在赋值是可以不写。

### 7.2 context 扫描包的配置

指定要扫描的包：

```xml
<context:component-scan base-package="com.xiao"/>
```

指定扫描包时指定排除一些不要的组件：

```xml
<context:component-scan base-package="com.xiao">
    <!--指定排除不要的组件-->
    <context:exclude-filter type="..." expression="..."/>
</context:component-scan>
```

- `type="annotation"`：按照注解进行排除，`expression`属性中指定要排除的注解的全类名
- `type="assignable"`：按照类名进行排除，`expression`属性中指定要排除的类的全类名

只扫描进入指定的组件，默认都是全部扫描进来，`use-default-filters`需要设置为false：

```xml
<context:component-scan base-package="com.xiao" use-default-filters="false">
    <context:include-filter type="..." expression="..."/>
</context:component-scan>
```

### 7.3 Autowired自动装配

#### 01 基本使用

直接在成员上添加` @Autowired`完成自动装配。

Dao 层：

```java
@Repository
public class BookDao {
	//声明一个方法
    public void readBook() {
        System.out.println("读了一本书！");
    }
}
```

Service 层，使用注解`@Autowired`完成成员 BookDao 的自动装配，调用 dao 层的方法：

```java
@Service
public class BookService {
	//使用@Autowired完成成员BookDao的自动装配
    @Autowired
    private BookDao bookDao;

    public void read() {
        this.bookDao.readBook();
    }
}
```

Controller 层，使用注解`@Autowired`完成成员 BookService 的自动装配，调用 service 层的方法：：

```java
@Controller
public class BookController {

    @Autowired
    private BookService bookService;

    public void one() {
        this.bookService.read();
    }
}
```

#### 02 Autowired 的执行流程

`@Autowired`可以直接用在属性上，执行流程：

1. 首先按照类型去容器中找对应的组件，如果找到一个就赋值，找不到就抛异常；

2. 如果有多个类型匹配时，会使用要注入的对象变量名称作为 bean 的 id，在 spring 容器查找，找到了也可以注入成功，找不到就报错。

3. 结合注解` @Qualifer`，指定一个 id：在自动按照类型注入的基础之上，再按照指定的 bean 的 id 去查找。它在给字段注入时不能独立使用，必须和`@Autowired`一起使用；但是给方法参数注入时，可以独立使用。

   

`@Autowired`标注的属性如果找不到就会报错，可以指定 required 属性，找不到就自动装配 null

```java
 @Autowired(required = false )
```

#### 03 注解加在方法上

` @Autowired`：也可以使用在 set 方法上，执行流程跟上面一样；

` @Qualifer`：还可以用在方法的参数，指定按照哪个 id 去装配。

`` @Nullable``：标记的属性可以 null

```xml
@Service
public class BookService {


    private BookDao bookDao;

    @Autowired
    private void setBookDao(BookDao bookDao) {
        this.bookDao = bookDao;
    }

    public void read() {
        this.bookDao.readBook();
    }
}
```

#### 04 @Resource

` @Resource：`直接按照 Bean 的 id 注入，是 Java 自带的注解。执行流程：

　　1. 如果同时指定了 name 和 type，则从 Spring 上下文中找到唯一匹配的 bean 进行装配，找不到则抛出异常
　　2. 如果指定了 name，则从上下文中查找id匹配的 bean 进行装配，找不到则抛出异常
　　3. 如果指定了 type，则从上下文中找到类型匹配的唯一 bean 进行装配，找不到或者找到多个，都会抛出异常
　　4. 如果既没有指定 name，又没有指定 type，则自动按照 byName 方式进行装配；如果没有匹配，则回退为一个原始类型进行匹配，如果匹配则自动装配；

### 7.4 Spring的单元测试

使用 Spring 的单元测试，不需要用 ioc.getBean() 来获取组件了，直接 Autowired 组件，Spring 自动装配

导入依赖：

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-test</artifactId>
    <version>5.2.0.RELEASE</version>
    <scope>test</scope>
</dependency>

<!--Junit-->
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.12</version>
</dependency>
```

添加注解：

- @ContextConfiguration：指定 Spring 配置文件的位置
- @RunWith：指定用哪种驱动进行单元测试，默认是 junit，这里指定用 Spring 的单元测试模块来执行标了@Test 注解的测试方法

```java
/*
 *@ContextConfiguration:指定Spring配置文件的位置
 *@RunWith：指定用哪种驱动进行单元测试，默认是junit,这里指定用Spring的单元测试模块来执行标了@Test注解的测试方法
 *
 */
@ContextConfiguration(locations = "classpath:ApplicationContext.xml")
@RunWith(SpringJUnit4ClassRunner.class)
public class Test02 {
    @Autowired
    private BookController bookController;

    @Test
    public void test01() {
        this.bookController.one();
    }
}
```