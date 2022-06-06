---
title: SpringMVC (一) - SpringMVC 执行流程、注解快速开发接口
categories:
  - 框架
  - SpringMVC
tags:
  - SpringMVC
date: 2021-06-15
description: SpringMVC 入门，重点是理解 SpringMVC 执行流程。
---

内容包括：

1. SpringMVC 执行流程

2. 注解开发`@controller`，`@RequestMapping`

3. 请求参数处理

4. 数据输出处理

5. 视图解析


参考视频：

B站 尚硅谷雷丰阳大神的 Spring、Spring MVC、MyBatis 课程 https://www.bilibili.com/video/BV1d4411g7tv

## 1. SpringMVC 概述

MVC：

- **Model（模型）：**数据模型，提供要展示的数据，：Value Object（数据 Dao） 和 服务层（行为Service），提供数据和业务。
- **View（视图）：**负责进行模型的展示，即用户界面
- **Controller（控制器）：**调度员，接收用户请求，委托给模型进行处理（状态改变），处理完毕后把返回的模型数据返回给视图，由视图负责展示。

SpringMVC 的特点：

- Spring 为展现层提供的基于 MVC 设计理念的 Web 框架
- SpirngMVC 通过一套 MVC 注解，让 POJO 成为处理请求的控制器，而无须实现任何接口
- 支持 REST 风格的 URL 请求
- 采用了松散耦合可拔插组件结构，扩展性和灵活性

## 2. HelloWorld

**1) 导入依赖**

spring-webmvc 的 maven 依赖

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-webmvc</artifactId>
    <version>5.2.0.RELEASE</version>
</dependency>
```

**2)  配置 web.xml  ， 注册 DispatcherServlet**

`DispatcherServlet`：前端控制器，负责请求分发。

要绑定 Spring 的配置文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">

    <!--注册DispatcherServlet,请求分发器（前端控制器）-->
    <servlet>
        <servlet-name>springDispatcherServlet</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <!--绑定Spring配置文件-->
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:springmvc-config.xml</param-value>
        </init-param>
        <!--启动级别为1，即服务器启动后就启动-->
        <!--值越小优先级越高，越先创建对象-->
        <load-on-startup>1</load-on-startup>
    </servlet>

    <!-- /  拦截所有的请求；（不包括.jsp）-->
    <!-- /* 拦截所有的请求；（包括.jsp，一旦拦截jsp页面就不能显示了）-->
    <servlet-mapping>
        <servlet-name>springDispatcherServlet</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>

</web-app>
```

**3) Spring配置文件**

Spring 的配置文件 Springmvc-config.xml。

1. 开启了包扫描，让指定包下的注解生效，由 IOC 容器统一管理
2. 配置了视图解析器`InternalResourceViewResolver`，这里可以设置前缀和后缀，拼接视图名字

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context
       http://www.springframework.org/schema/context/spring-context.xsd">

    <!--开启包扫描,让指定包下的注解生效,由IOC容器统一管理-->
    <context:component-scan base-package="com.xiao.controller"/>

    <!--配置视图解析器，拼接视图名字，找到对应的视图-->
    <bean id="internalResourceViewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <!--前缀-->
        <property name="prefix" value="/WEB-INF/page/"/>
        <!--后缀-->
        <property name="suffix" value=".jsp"/>
    </bean>

</beans>
```

**4) 编写 controller 层**

HelloController 类：

1. @Controller：告诉 Spirng 这是一个控制器，交给 IOC 容器管理

2.  @RequestMapping("/hello01")：/ 表示项目地址，当请求项目中的 hello01 时，返回一个 /WEB-INF/page/success.jsp 页面给前端

```java
@Controller
public class HelloController {

    @RequestMapping("/hello01")
    public String toSuccess(){
        System.out.println("请求成功页面");
        return "success";
    }
    @RequestMapping("/hello02")
    public String toError() {
        System.out.println("请求错误页面");
        return "error";
    }
}
```

**5) 编写跳转的 jsp 页面**

项目首页 index.jsp，两个超链接，分别发出 hello01 和 hello02 的请求

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
  <head>
    <title>$Title$</title>
  </head>
  <body>

  <a href="hello01">点这里去成功页面</a>
  <a href="hello02">点这里去失败页面</a>

  </body>
</html>
```

成功页面 success.jsp 和失败页面 error.jsp，要注意文件的路径 /WEB-INF/page/..jsp，与上面的保持一致

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>成功页面</title>
</head>

<body>

<h1>这里是成功页面</h1>
</body>
</html>
```

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>错误页面</title>
</head>

<h1>这里是错误页面</h1>
<body>

</body>
</html>
```

**6) 访问**

启动项目：

![image-20200616123925928](http://image.kongxiao.top/20220531231459.png)

点击去成功页面，可以看到发出了 /hello01 请求，页面转发到 /WEB-INF/page/success.jsp，控制台输出了请求成功页面。

![image-20200616123951757](http://image.kongxiao.top/20220531231504.png)

## 3. 实现细节

### 3.1 运行流程

1. 客户端点击链接发送请求：http://localhost:8080/hello01；

2. 来到 tomcat 服务器；
3. SpringMVC 的前端控制器收到所有请求；
4. 看请求地址和 @RequestMapping 标注的哪个匹配，来找到底使用哪个类的哪个方法来处理；
5. 前端控制器找到目标处理器类和目标方法，直接利用反射执行目标方法；
6. 方法执行完后有一个返回值，SpringMVC 认为这个返回值就是要去的页面地址；
7. 拿到方法返回值后，视图解析器进行拼串得到完整的页面地址
8. 得到页面地址，前端控制器帮我们转发到页面

### 3.2 @RequestMapping

#### 01 标注在方法上

告诉 SpringMVC 这个方法用来处理什么请求。

`@RequestMapping("/hello01")`中的 `/`可以省略，就是默认从当前项目下开始。

#### 02 标注在类上

表示为当前类中的所有方法的请求地址，指定一个基准路径。toSuccess() 方法处理的请求路径是`/haha/hello01`。

```java
@Controller
@RequestMapping("/haha")
public class HelloController {

    @RequestMapping(value = "/hello01")
    public String toSuccess(){
        System.out.println("请求成功页面");
        return "success";
    }
}
```

#### 03 规定请求方式

method 属性规定请求方式，默认是所求请求方式都行。method = RequestMethod.GET，method = RequestMethod.POST。

如果方法不匹配会报：**HTTP Status 405 错误 – 方法不被允许**

```java
@RequestMapping(value = "/hello01",method = RequestMethod.GET)
public String toSuccess(){
    System.out.println("请求成功页面");
    return "success";
}
```

**组合用法**

- @GetMapping  等价于 @RequestMapping(method =RequestMethod.GET) 
- @PostMapping
- @PutMapping
- @DeleteMapping
- @PatchMapping

#### 04 规定请求参数

params 属性规定请求参数。会造成错误：**HTTP Status 400 – 错误的请求**

不携带该参数，表示参数值为null；携带了不给值表示参数值是空串

```java
//必须携带username参数
@RequestMapping(value = "/hello03",params ={"username"})
//必须不携带username参数
@RequestMapping(value = "/hello03",params ={"！username"})
//必须携带username参数，且值必须为123
@RequestMapping(value = "/hello03",params ={"username=123"})
//username参数值必须不为123，不携带或者携带了不是123都行
@RequestMapping(value = "/hello03",params ={"username=！123"})
```

#### 05 规定请求头

headers 属性规定请求头。其中 User-Agent：浏览器信息

谷歌浏览器：User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/83.0.4103.97 Safari/537.3

#### 06 Ant 风格URL

URL 地址可以写模糊的通配符，模糊和精确多个匹配情况下精确优先。

？：**替代任意一个字符**

```java
@RequestMapping( "/hello0?") /
```

*：**替代任意多个字符或一层路径**

```java
@RequestMapping( "/hello0*")   //任意多个字符
@RequestMapping( "/a/*/hello01")  //一层路径
```

**：替代任意多层路径

```java
@RequestMapping( "/a/*/*/hello01")  //任意多层路径
```

### 3.3 Spring 配置文件的默认位置

默认位置是 /WEB-INF/xxx-servlet.xml，其中 xxx 是自己在 web.xml 文件中配置的 servlet-name 属性。

当然也可以手动指定文件位置。

### 3.4 url-pattern

/  拦截所有的请求，不拦截 jsp

/* **拦截所有的请求**，包括 *.jsp，一旦拦截 jsp 页面就不能显示了。. jsp 是 tomcat 处理的事情

看 Tomcat 的配置文件 web.xml中，有 DefaultServlet和JspServlet，

- DefaultServlet 是 Tomcat中处理静态资源的，Tomcat 会在服务器下找到这个资源并返回。如果我们自己配置` url-pattern=/`，相当于禁用了 Tomcat 服务器中的 DefaultServlet，这样如果请求静态资源，就会去找前端控制器找 @RequestMapping，**这样静态资源就不能访问了**。解决办法：

  ```xml
  <!-- 告诉Spring MVC自己映射的请求就自己处理，不能处理的请求直接交给tomcat -->
  <mvc:default-servlet-handler />
  <!--开启MVC注解驱动模式，保证动态请求和静态请求都能访问-->
  <mvc:annotation-driven/>
  ```

- JspServlet，保证了 jsp 可以正常访问

```xml
<servlet>
    <servlet-name>default</servlet-name>
    <servlet-class>org.apache.catalina.servlets.DefaultServlet</servlet-class>
    <init-param>
        <param-name>debug</param-name>
        <param-value>0</param-value>
    </init-param>
    <init-param>
        <param-name>listings</param-name>
        <param-value>false</param-value>
    </init-param>
    <load-on-startup>1</load-on-startup>
</servlet>

    <servlet-mapping>
        <servlet-name>default</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>


<servlet>
     <servlet-name>jsp</servlet-name>
     <servlet-class>org.apache.jasper.servlet.JspServlet</servlet-class>
     <init-param>
         <param-name>fork</param-name>
         <param-value>false</param-value>
      </init-param>
      <init-param>
          <param-name>xpoweredBy</param-name>
          <param-value>false</param-value>
      </init-param>
      <load-on-startup>3</load-on-startup>
</servlet>

    <servlet-mapping>
        <servlet-name>jsp</servlet-name>
        <url-pattern>*.jsp</url-pattern>
        <url-pattern>*.jspx</url-pattern>
    </servlet-mapping>
```
## 4. REST风格

### 4.1 概述

REST 就是一个资源定位及资源操作的风格。不是标准也不是协议，只是一种风格。基于这个风格设计的软件可以更简洁，更有层次，更易于实现缓存等机制。其强调HTTP应当以资源为中心，并且规范了 URI 的风格；规范了HTTP 请求动作（GET/PUT/POST/DELETE/HEAD/OPTIONS）的使用，具有对应的语义。

- 资源（Resource）：网络上的一个实体，每种资源对应一个特定的URI，即URI为每个资源的独一无二的识别符；
- 表现层（Representation）：把资源具体呈现出来的形式，叫做它的表现层。比如 txt、HTML、XML、JSON格式等；
- 状态转化（State Transfer）：每发出一个请求，就代表一次客户端和服务器的一次交互过程。GET 用来获取资源，POST 用来新建资源，PUT 用来更新资源，DELETE 用来删除资源。

在参数上使用  @PathVariable 注解，可以获取到请求路径上的值，也可以写多个

```java
    @RequestMapping(value = "/hello04/username/{id}")
    public String test2(@PathVariable("id") int id){
        System.out.println(id);
        return "success";
    }
```

### 4.2 页面上发出 PUT 请求

**对一个资源的增删改查用请求方式来区分：**

- /book/1 GET：查询1号图书

- /book/1 DELETE：删除1号图书

- /book/1 PUT：修改1号图书

- /book  POST：新增图书

页面上只能发出 GET 请求和 POST 请求。将 POST 请求转化为 put 或者 delete 请求的步骤：

1. 把前端发送方式改为 post 。

2. 在 web.xml 中配置一个filter：HiddenHttpMethodFilter 过滤器

3. 必须携带一个键值对，key=_method, value=put 或者 delete

```xml
<!--这个过滤器的作用 ：就是将post请求转化为put或者delete请求-->
<filter>
    <filter-name>HiddenHttpMethodFilter</filter-name>
    <filter-class>org.springframework.web.filter.HiddenHttpMethodFilter</filter-class>
</filter>
<filter-mapping>
    <filter-name>HiddenHttpMethodFilter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```

```jsp
<form action="hello03" method="post">
  <input type="hidden" name="_method" value="delete">
  <input type="submit" name="提交">
</form>
```

高版本 Tomcat 会出现问题：JSPs only permit GET POST or HEAD，在页面上加上异常处理即可

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java"  isErrorPage="true" %>
```

## 5 请求参数处理

### 5.1 传入参数

**1. 如果提交的参数名称和处理方法的参数名一致，则无需处理，直接使用**

提交数据 : http://localhost:8080/hello05?username=zhangsan，控制台会输出 zhangsan

```java
@RequestMapping("/hello05")
public String test03(String username) {
    System.out.println(username);
    return "success";
}
```

**2. 提交的参数名称和处理方法的参数名不一致，使用 @RequestParam 注解**

注解`@RequestParam`可以获取请求参数，默认必须携带该参数，也可以指定`required=false`，和没携带情况下的默认值`defaultValue`

```java
@RequestMapping("/hello05")
public String test03(@RequestParam(value = "username",required = false, defaultValue ="hehe" ) String name) {
    System.out.println(name);
    return "success";
}
```

还有另外两个注解：

- `@RequestHeader `：获取请求头中的信息，比如 User-Agent：浏览器信息

  ```java
  @RequestMapping("/hello05")
  public String test03(@RequestHeader("User-Agent" ) String name) {
      System.out.println(name);
      return "success";
  }
  ```

- `@CookieValue`：获取某个 cookie 的值

  ```java
  @RequestMapping("/hello05")
  public String test03(@CookieValue("JSESSIONID" ) String name) {
       System.out.println(name);
       return "success";
  }
  ```

### 5.2 传入一个对象

传 入POJO，SpringMVC 会自动封装，**提交的表单域参数必须和对象的属性名一致，否则就 是null，请求没有携带的字段，值也会是 null。**同时也还可以级联封装。

新建两个对象User和Address：

```java
public class User {
    private String username;
    private Integer age;
    private Address address;
    //....
}
```

```java
public class Address {
    private String name;
    private Integer num;
        //....
}
```

前端请求：

```jsp
<form action="hello06" method="post">
    姓名： <input type="text" name="username"> <br>
    年龄： <input type="text" name="age"><br>
    地址名：<input type="text" name="address.name"><br>
    地址编号：<input type="text" name="address.num"><br>
    <input type="submit" name="提交">
</form>
```

后端通过对象名也能拿到对象的值，没有对应的值则为 null

```java
@RequestMapping("/hello06")
public String test03(User user) {
    System.out.println(user);
    return "success";
}
```

### 5.3 乱码问题

一定要放在在其他 Filter 前面。

```xml
<filter>
   <filter-name>encoding</filter-name>
   <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
   <init-param>
       <param-name>encoding</param-name>
       <param-value>utf-8</param-value>
   </init-param>
</filter>
<filter-mapping>
   <filter-name>encoding</filter-name>
   <url-pattern>/*</url-pattern>
</filter-mapping>
```

### 5.4 传入原生 ServletAPI

处理方法还可以传入原生的 ServletAPI：

```java
@RequestMapping("/hello07")
public String test04(HttpServletRequest request, HttpSession session) {
    session.setAttribute("sessionParam","我是session域中的值");
    request.setAttribute("reqParam","我是request域中的值");
    return "success";
}
```

通过EL表达式获取到值，`${requestScope.reqParam}`：

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java"  isErrorPage="true" %>
<html>
<head>
    <title>成功页面</title>
</head>

<body>

<h1>这里是成功页面</h1>
${requestScope.reqParam}
${sessionScope.sessionParam}
</body>
</html>
```

## 6. 数据输出

### 6.1 Map、Model、ModerMap

实际上都是调用的 BindingAwareModelMap (隐含模型)，将数据放在**请求域 (requestScope) 中**进行转发，用EL表达式可以取出对应的值。

```java
@RequestMapping("/hello01")
public String test01 (Map<String,Object> map){
    map.put("msg","HelloWorld!");
    return "success";
}
```

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
</head>
<body>

pageScope:  ${pageScope.msg}

requestScope :   ${requestScope.msg}

sessionScope:     ${sessionScope.msg}

applicationScope:   ${applicationScope.msg}

</body>
</html>
```

**【补充】jsp 的4个作用域 pageScope、requestScope、sessionScope、applicationScope 的区别：**

- **page **指当前页面有效。在一个jsp页面里有效

- **request** 指在一次请求的全过程中有效，即从 http 请求到服务器处理结束，返回响应的整个过程，存放在**HttpServletRequest **对象中。在这个过程中可以使用 forward 方式跳转多个 jsp。在这些页面里都可以使用这个变量。

- **Session** 是用户全局变量，在整个会话期间都有效。只要页面不关闭就一直有效（或者直到用户一直未活动导致会话过期，默认 session 过期时间为 30 分钟，或调用 HttpSession的invalidate() 方法。存放在HttpSession 对象中 

- **application **是程序全局变量，对每个用户每个页面都有效。存放在 ServletContext 对象中。它的存活时间是最长的，如果不进行手工删除，它们就一直可以使用 

### 6.2 ModerAndView

返回一个模型视图对象 ModerAndView， 既包含视图信息(页面地址)，也包含模型数据(给页面带的数据)

```java
@RequestMapping("/hello04")
public ModelAndView test04 (){
   //新建一个模型视图对象，也可以直接传入名字
   ModelAndView mv = new ModelAndView();
   //封装要显示到视图中的数据
   //相当于req.setAttribute("msg",HelloWorld!);
   mv.addObject("msg","HelloWorld!");
   //设置视图的名字，相当于之前的return "success";
   mv.setViewName("success");
   return mv;
}
```

### 6.3 @SessionAttributes

给 Session 域中携带数据使用注解` @SessionAttributes`，只能标在类上，value 属性指定 key，type 可以指定保存类型。这个注解会引发异常**一般不用，就用原生API**

`@SessionAttributes(value = "msg")`：表示给 BindingAwareModelMap 中保存 key 为 msg 的数据时，在session 中也保存一份；

`@SessionAttributes(types = {String.class})`：表示只要保存 String 类型的数据时，给 session 中也放一份。

```java
//表示给BindingAwareModelMap中保存key为msg的数据时，在session中也保存一份
@SessionAttributes(value = "msg")
@Controller
public class outputController {
    @RequestMapping("/hello01")
    public String test01 (Map<String,Object> map){
        map.put("msg","HelloWorld!");
        return "success";
    }
}
```

### 6.4  @ModelAttribute 

方法入参标注该注解后，入参的对象就会放到数据模型中，会提前于控制方法先执行，并发方法允许的结果放在隐含模型中。

处理这样的场景：

前端传来数据，SpringMVC 自动封装成对象，实际上是创建了一个对象，每个属性都有默认值，然后将请求参数中对应是属性设置过来，但是如果没有的值将会是 null，如果拿着这个数据去更新数据库，会造成其他字段也变为 null。因此希望使用`@ModelAttribute `，会在目标方法执行前先做一些处理

```java
@ModelAttribute
public void  myModelAttribute(ModelMap modelMap){
    System.out.println("modelAttribute方法执行了");
    //提前做一些处理
    User user = new User("zhangsan",20);
    //保存一个数据到BindingAwareModelMap中，目标方法可以从中取出来
    modelMap.addAttribute("user",user);
}

@RequestMapping("/hello05")
public void  test05(@ModelAttribute("user") User user){
    System.out.println("目标方法执行了");
    //在参数上加上@ModelAttribute注解，可以拿到提前存入的数据
    System.out.println(user);

}
```

### 6.5 @ResponseBody

在控制器类中，在方法上使用 **@ResponseBody** 注解可以不走视图解析器，如果返回值是字符串，那么直接将字符串写到客户端；如果是一个对象，会将对象转化为 JSON 串，然后写到客户端。

或者在类上加  **@RestController **注解，可以让类中的所有方法都不走视图解析器，直接返回 JSON 字符串

## 7. 再谈 SpringMVC 执行流程

### 7.1 前端控制器DisatcherServlet

![image-20200616233004463](http://image.kongxiao.top/20220531231516.png)

### 7.2 SpringMVC 执行流程

1. 用户发出请求，DispatcherServlet 接收请求并拦截请求。

2. 调用 doDispatch() 方法进行处理：

   1）getHandler()：根据当前请求地址，在 HandlerMapping (处理器映射器)中找到能处理这个请求的目标处理器类(处理器)；

   2）getHandlerAdapter()：根据当前处理器类找到当前类的 HandlerAdapter(处理器适配器)

   3）使用刚才获取到的适配器 (AnnotationMethodHandlerAdapter) 执行目标方法；

   4）目标方法执行后，会返回一个 ModerAndView 对象

   5）根据 ModerAndView的信息转发到具体页面，并可以在请求域中取出 ModerAndView 中的模型数据

HandlerMapping 为处理器映射器，保存了每一个处理器能处理哪些请求的映射信息，handlerMap

HandlerAdapter 为处理器适配器，能解析注解方法的适配器，其按照特定的规则去执行 Handler


![image-20200616233302627](http://image.kongxiao.top/20220531231521.png)

### 7.3 SpringMVC 的九大组件

-  multipartResolver：文件上传解析器
- localeResolver：区域信息解析器，和国际化有关
- themeResolver：主题解析器
- handlerMappings：handler 的映射器
- handlerAdapters：handler 的适配器
- handlerExceptionResolvers：异常解析功能
- viewNameTranslator：请求到视图名的转换器
- flashMapManager：SpringMVC 中允许重定向携带数据的功能
- viewResolvers：视图解析器

```java
    @Nullable
    private MultipartResolver multipartResolver;
    @Nullable
    private LocaleResolver localeResolver;
    @Nullable
    private ThemeResolver themeResolver;
    @Nullable
    private List<HandlerMapping> handlerMappings;
    @Nullable
    private List<HandlerAdapter> handlerAdapters;
    @Nullable
    private List<HandlerExceptionResolver> handlerExceptionResolvers;
    @Nullable
    private RequestToViewNameTranslator viewNameTranslator;
    @Nullable
    private FlashMapManager flashMapManager;
    @Nullable
    private List<ViewResolver> viewResolvers;
```

## 8. 视图解析

通过 SpringMVC 来实现转发和重定向。

- 直接 return "success"，会走视图解析器进行拼串
- 转发：return "forward:/succes.jsp"；直接写绝对路径，/表示当前项目下，不走视图解析器
- 重定向：return "redirect:/success.jsp"；不走视图解析器

```java
@Controller
public class ResultSpringMVC {
   @RequestMapping("/hello01")
   public String test1(){
       //转发
       //会走视图解析器
       return "success";
  }

   @RequestMapping("/hello02")
   public String test2(){
       //转发二
       //不走视图解析器
       return "forward:/success.jsp";
  }

   @RequestMapping("/hello03")
   public String test3(){
       //重定向
       //不走视图解析器
       return "redirect:/success.jsp";
  }
}
```

使用原生的 ServletAPI 时要注意，**/路径需要加上项目名才能成功**

```java
   @RequestMapping("/result/t2")
   public void test2(HttpServletRequest req, HttpServletResponse resp) throwsIOException {	
       //重定向
       resp.sendRedirect("/index.jsp");
  }

   @RequestMapping("/result/t3")
   public void test3(HttpServletRequest req, HttpServletResponse resp) throwsException {
       //转发
       req.setAttribute("msg","/result/t3");
       req.getRequestDispatcher("/WEB-INF/jsp/test.jsp").forward(req,resp);
  }
```



`mvc:view-controller`：直接将请求映射到某个页面，不需要写方法了：

```xml
<mvc:view-controller path="/toLogin" view-name="login"/>
<!--开启MVC注解驱动模式-->
<mvc:annotation-driven/>
```

