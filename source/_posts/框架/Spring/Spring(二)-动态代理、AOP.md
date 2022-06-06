---
title: Spring (二) - 动态代理、AOP
categories:
  - 框架
  - Spring
tags:
  - Spring
  - AOP
  - 动态代理
date: 2021-06-12
description: Spring 核心特性之 AOP，使用 AOP 切面实现业务增强。
---

Spring 基础知识学习笔记 (二)，内容包括：

1. 代理模式：静态代理和动态代理
2. AOP 实现：注解实现+配置文件实现
3. 切面、通知、切入点、切入点表达式
4. 环绕通知

几个概念：

`OOP` ：(Object Oriented Programming) 面向对象编程。

`AOP` ：(Aspect Oriented Programming) 面向切面编程，基于 OOP 基础之上的编程思想，在程序运行期间，将某段代码动态的切入到指定方法的指定位置进行运算。

应用场景：计算器运行计算方法的时候进行日志记录，不推荐直接在方法内部，修改维护麻烦。

希望：在核心功能运行期间，系统的辅助功能自己动态的加上。

参考视频：

B 站尚硅谷雷丰阳大神的 Spring、Spring MVC、MyBatis 课程 https://www.bilibili.com/video/BV1d4411g7tv

##  1. 代理模式

### 1.1 静态代理

**静态代理角色分析**：

- 抽象角色 : 一般使用接口或者抽象类来实现
- 真实角色 : 被代理的角色
- 代理角色 : 代理真实角色 ; 代理真实角色后 , 一般会做一些附属的操作 .
- 客户  :  使用代理角色来进行一些操作 

**案例：**  房东有房子，交给中介代理，客户直接找中介，中介在租房前后带客户看房子和收中介费。

租房接口 Renet：

```java
//抽象角色：租房接口
public interface Rent {
   public void rent();
}
```

真实角色房东：Host，实现了 Rent 接口，可以出租房子

```java
//真实角色: 房东，房东要出租房子
public class Host implements Rent{
   public void rent() {
       System.out.println("房屋出租!");
  }
}
```

代理角色 MyProxy：

```java
public class MyProxy implements Rent {
    
    private Host host;

    public MyProxy() {
    }

    public MyProxy(Host host) {
        this.host = host;
    }

    @Override
    public void rent() {
        //中介在出租房屋前带客户看房子
        seeHouse();
        this.host.rent();
        //中介在出租房屋后收中介费
        fare();
    }
    
    public void seeHouse() {
        System.out.println("带客户看房子");
    }
    
    public void fare() {
        System.out.println("收中介费");
    }
}
```

客户 Client，找中介租房：

```java
public class Client {

    public static void main(String[] args) {
        //房东
        Host host = new Host();
        //中介来代理房东
        MyProxy proxy = new MyProxy(host);
        //客户找中介，中介出租房屋
        proxy.rent();
    }
}
```

结果：

```java
带客户看房子
房屋出租！
收中介费
```

**静态代理的好处:**

- 使得真实角色更加纯粹，不再去关注一些公共的事情 
- 公共的业务由代理来完成，实现了业务的分工 
- 公共业务发生扩展时变得更加集中和方便 

**缺点 :**

- 类多了 , 多了代理类 , 工作量变大了 ，开发效率降低 

### 1.2 动态代理

动态代理的角色和静态代理的一样 ，区别是 **动态代理的代理类是动态生成的**  ，静态代理的代理类是提前写好的。

动态代理分为两类 : 一类是 **基于接口动态代理** , 一类是 **基于类的动态代理**

- 基于接口的动态代理--JDK动态代理，**代理对象和被代理对象唯一能产生的关联就是实现了同一个接口。如果目标对象没有实现任何接口，是无法为其创建代理对象的。**
- 基于类的动态代理--cglib

JDK动态代理需要两个核心类：`Proxy` 代理和 `InvocationHandler` 调用处理程序。

>  Proxy：

`Proxy.newProxyInstance()` 方法为目标对象创建代理对象，返回代理对象。三个参数：

- ClassLoader loader：和被代理对象使用相同的类加载器。 
- Class<?>[] interfaces：和被代理对象具有相同的行为。实现相同的接口。 
- InvocationHandler：如何代理，方法执行器。

>  InvocationHandler：

调用其`invoke()`方法，执行被代理对象的任何方法，都会经过该方法，三个参数：被代理对象、方法、参数

- Object proxy：被代理的对象
- Method method：方法
- Object[] args：执行方法的参数

代码实现：

1. 定义一个出租房子的接口Rent

2. 房东类实现Rent，具有出租房子的功能
3. 定义一个类实现InvocationHandler接口，来创建动态代理对象，增强功能
4. 动态代理对象调用方法

```java
/**
@Description: 定义一个类实现InvocationHandler接口，来创建动态代理对象
 */
public class ProxyInvocationHandler implements InvocationHandler {
    private Rent rent;
    
	// 设置要代理的接口
    public void setRent(Rent rent) {
        this.rent = rent;
    }

    // 声明一个生成代理类的方法
    public Object getProxy() {
        // Proxy.newProxyInstance()传入三个参数：类加载器，类实现的接口，InvocationHandler对象
        return Proxy.newProxyInstance(this.getClass().getClassLoader(), rent.getClass().getInterfaces(), this);
    }

    // 处理实例，并返回结果
    @Override
    public Object invoke 
		  // 先看房
        seeHouse();
        // 使用反射机制invoke方法，传入被代理的接口和参数。使用真实对象的方法
        Object result = method.invoke(proxy, args);
        fare();
        return result;
    }

    public void seeHouse() {
        System.out.println("中介带看房子");
    }
    public void fare(){
        System.out.println("中介收费");
    }
}
```

```java
/**
 * @Description:测试类
 */
public class ProxyTest {
    public static void main(String[] args) {
        // 真实角色
        Host host = new Host();
        ProxyInvocationHandler proxyInvocationHandler = new ProxyInvocationHandler();

        // 传入要代理的接口
        proxyInvocationHandler.setRent(host);

        // 获得代理对象
        Rent proxy = (Rent) proxyInvocationHandler.getProxy();
        // 代理对象使用真实对象的方法，方法被增强了
        proxy.rent();
    }
}

```

### 1.3 动态代理实现日志功能

1. 定义一个 `Calculator` 接口，声明加减乘除方法
2. 定义一个 `MyCalculator` 类实现 `Calculator` 接口，完成方法体
3. 定义一个生成代理对象的类 `CalculatorProxy`，获取代理对象
4. 重写`InvocationHandler` 的 `invoke()`方法，在执行目标方法前后，添加相应的日志输出，也可以处理异常信息

Calculator 接口：

```java
public interface Calculator {

    //加减乘除方法
    public int add(int i, int j);
    public int subtract(int i, int j);
    public int multiply(int i, int j);
    public int divide(int i, int j);

}
```

MyCalculator 类：

```java
public class MyCalculator implements Calculator {
    @Override
    public int add(int i, int j) {
        return i + j;
    }

    @Override
    public int subtract(int i, int j) {
        return i - j;
    }

    @Override
    public int multiply(int i, int j) {
        return i * j;
    }

    @Override
    public int divide(int i, int j) {
        return i / j;
    }
}
```

日志工具类 LogUtils：

```java
public class LogUtils {
    // 执行前
    public static void before(Method method,Object... args) {
        System.out.println("【"+method.getName()+"】方法开始执行了，用的参数列表是【"+ Arrays.asList(args)+"】");
    }
    // 执行后
    public static void after(Method method,Object result) {
        System.out.println("【"+method.getName()+"】方法执行完成了，计算结果是【"+ result+"】");
    }
    // 出现异常
    public static void exception(Method method,Exception e) {
        System.out.println("【"+method.getName()+"】方法出现异常了,异常信息是："+e.getCause());
    }
    // 方法结束
    public  static void end(Method method) {
        System.out.println("【"+method.getName()+"】方法最终结束了");
    }

}
```

生成代理对象的类 CalculatorProxy：

```java
public class CalculatorProxy {

    /**
     * Proxy.newProxyInstance()
     * 为传入的参数对象创建一个动态代理对象
     * @param calculator 被代理的对象
     * @return
     */
    public static Calculator getProxy(Calculator calculator) {

        Object proxyInstance = Proxy.newProxyInstance(calculator.getClass().getClassLoader(), calculator.getClass().getInterfaces(),
                new InvocationHandler() {
                    /**
                     * @param proxy 代理对象，给JDK使用的
                     * @param method 当前将要执行的目标对象的方法
                     * @param args 参数
                     * @return
                     * @throws Throwable
                     */
                    @Override
                    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                        Object result = null;

                        try {
                            //目标方法执行前
                            LogUtils.before(method,args);
             
                            System.out.println("动态代理要帮你执行方法！");
                            //利用反射执行目标方法
                            result = method.invoke(calculator, args);

                            //目标方法执行后
                            LogUtils.after(method,result);
                        } catch (Exception e) {
                           //目标方法出现异常
                            LogUtils.exception(method,e);
                        } finally {
                            //目标方法结束后
                            LogUtils.end(method);
                        }
                        //返回值必须返回出去，外界才能拿到真正执行后的返回值
                        return result;
                    }
                });

        //返回代理对象
        return (Calculator) proxyInstance;
    }
}
```

测试：

```java
public class CalculatorTest {

    @Test
    public void test(){
        Calculator calculator = new MyCalculator();
        Calculator proxy = CalculatorProxy.getProxy(calculator);
        proxy.add(1,2);
        proxy.divide(2,0);
    }
}
```

结果：

```java
【add】方法开始执行了，用的参数列表是【[1, 2]】
动态代理要帮你执行方法！
【add】方法执行完成了，计算结果是【3】
【add】方法最终结束了
【divide】方法开始执行了，用的参数列表是【[2, 0]】
动态代理要帮你执行方法！
【divide】方法出现异常了,异常信息是：java.lang.ArithmeticException: / by zero
【divide】方法最终结束了
```

## 2. AOP

`AOP` ：(Aspect Oriented Programming) 面向切面编程，将某段代码动态的切入到指定方法的指定位置(方法的开始、结束、异常...)。

使用场景：

- 加日志保存到数据库
- 做权限验证
- 做安全检查
- 做事务控制

### 2.1 几个专业术语

![image-20200615132452367](http://image.kongxiao.top/image-20200615132452367.png)

- **横切关注点**：与业务逻辑无关的，但是需要关注的部分，就是横切关注点，方法的开始、返回、异常、结束等。

- **切面（ASPECT）类**：在上面例子中相当于自己定义的一个日志工具类。
- **通知（Advice）**：切面必须要完成的工作，是类中的一个方法。

- **目标（Target）**：被通知对象。
- **代理（Proxy）**：向目标对象应用通知之后创建的对象。
- **连接点（JointPoint）**：每一个方法的每一个位置都是一个连接点
- **切入点（PointCut）**：切面通知执行的 “地点”，即真正需要执行日志记录的地方
- **切入点表达式**：在众多连接点中选出我们感兴趣的地方

### 2.2 注解实现步骤

需要 AOP 织入，要导入依赖：

```xml
<dependency>
    <groupId>org.aspectj</groupId>
    <artifactId>aspectjweaver</artifactId>
    <version>1.9.4</version>
</dependency>
```

步骤：

1. 将目标类和切面类(封装了通知方法的类)加入到IOC容器中，注解`@Component`，配置文件开启context:component-scan 包扫描

2. 告诉Spring到底哪个是切面类，在类上注解`@Aspect`
3. 告诉Spring切面中的方都是何时何地运行，方法上注解
   - @Before：在目标方法之前运行；前置通知
   - @After：在目标方法之后运行；后置通知
   - @AfterReturning：在目标方法正常返回之后；返回通知
   - @AfterThrowing：在目标方法抛出异常之后；异常通知
   - @Around：环绕通知

4. 在注解中写切入点表达式：`execution(访问权限符 返回值类型 方法全类名(参数表))`

5. 配置文件中开启基于注解的 AOP 功能

   AOP 名称空间头文件

   ```xml
   xmlns:aop="http://www.springframework.org/schema/aop"
   
   xsi:schemaLocation="http://www.springframework.org/schema/aop https://www.springframework.org/schema/aop/spring-aop.xsd"
   ```

   ```xml
   <!--开启注解支持-->
   <aop:aspectj-autoproxy/>
   ```

代码实现：

目标类：

```java
@Component
public class MyCalculator implements Calculator {
...
}
```

切面类：

```java
@Aspect
@Component
public class LogUtils {
    //执行前
    @Before("execution(public int com.xiao.MyProxy02.MyCalculator.*(int,int))")
    public static void before() {
        System.out.println("方法开始执行了");
    }
    //执行后
    @AfterReturning("execution(public int com.xiao.MyProxy02.MyCalculator.*(int,int))")
    public static void after() {
        System.out.println("方法执行完成了");
    }
    //出现异常
    @AfterThrowing("execution(public int com.xiao.MyProxy02.MyCalculator.*(int,int))")
    public static void exception() {
        System.out.println("方法出现异常了");
    }
    //方法结束
    @After("execution(public int com.xiao.MyProxy02.MyCalculator.*(int,int))")
    public  static void end() {
        System.out.println("方法最终结束了");
    }
}
```

测试，获取到目标对象的 bean，执行方法

```java
public class AopTest {

    ApplicationContext ioc = new ClassPathXmlApplicationContext("ApplicationContext.xml");

    @Test
    public void test() {
        //注意这里是根据接口类型获取的
        Calculator bean = ioc.getBean(Calculator.class);
        System.out.println(bean);//com.xiao.aop.MyCalculator@3c9bfddc
        System.out.println(bean.getClass());//class com.sun.proxy.$Proxy22
        bean.add(1,2);
    }

}
```

配置文件：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context
       https://www.springframework.org/schema/context/spring-context.xsd
    http://www.springframework.org/schema/aop https://www.springframework.org/schema/aop/spring-aop.xsd">
    
 	<!--包扫描-->
    <context:component-scan base-package="com.xiao.aop"/>

    <!--开启注解支持-->
    <aop:aspectj-autoproxy/>

</beans>
```

### 2.3 注解实现的几个细节

#### 01 获取组件

IOC 容器中保存的是组件的代理对象。ioc.getBean() 中使用的接口类型，也可以用id名

```java
Calculator bean = ioc.getBean(Calculator.class);
System.out.println(bean);// com.xiao.MyProxy02.MyCalculator@3c9bfddc
System.out.println(bean.getClass());// class com.sun.proxy.$Proxy22
```
#### 02 cglib

`<aop:aspectj-autoproxy />`有一个 proxy-target-class 属性，默认为 false，表示使用 JDK 动态代理织入增强。

当配为`<aop:aspectj-autoproxy  poxy-target-class="true"/>`时，表示使用 CGLib 动态代理技术织入增强。不过即使 proxy-target-class 设置为 false，如果目标类没有声明接口，则 spring 将自动使用 CGLib 动态代理。

CGLib 可以为没有实现接口的组件创建代理对象，通过本类类型或者 id 名获取到：

```java
class com.xiao.MyProxy02.MyCalculator$$EnhancerBySpringCGLIB$$5ef61d8e
```

#### 03 切入点表达式的写法

固定格式：**execution(访问权限符 返回值类型 方法全类名(参数表))**，表达式中支持 && 、||、 ！

`"execution(* *.*(..))"`：表示任意返回值类型，任意包下的任意类的任意方法，任意参个数

通配符：

- `*` 可以匹配一个或多个字符；匹配一个参数；匹配一层路径；权限位置不写就行
- `..` 匹配任意多个参数，任意类型参数，任意多层路径

#### 04 通知方法的执行顺序

正常执行：Before →方法执行 →After → AfterReturning (正常返回)

出现异常：Before →方法执行 →After → AfterThrowing

#### 05 拿到目标方法的详细信息

从 JoinPoint 对象中可以拿到方法的详细信息，`joinPoint.getArgs()`，`joinPoint.getSignature()`

也可以接收异常和返回值，需要自己传入对应的参数Object result、Exception exception，并且要告诉Spring指定返回值**returning** ，指定异常**throwing**

```java
    //执行前
    @Before("execution(public int com.xiao.aop.MyCalculator.*(int,int))")
    public static void before(JoinPoint joinPoint) {
        System.out.println("【"+ joinPoint.getSignature().getName()+"】方法开始执行了，用的参数列表是【"+ Arrays.asList(joinPoint.getArgs())+"】");

    }
    //执行后
    @AfterReturning(value = "execution(public int com.xiao.aop.MyCalculator.*(int,int))",returning = "result")
    public static void after(JoinPoint joinPoint,Object result) {
        System.out.println("【"+ joinPoint.getSignature().getName()+"】方法执行完成了，执行结果是【"+ result +"】");
    }
    //出现异常
    @AfterThrowing(value = "execution(public int com.xiao.aop.MyCalculator.*(int,int))",throwing = "exception")
    public static void exception(JoinPoint joinPoint,Exception exception) {
        System.out.println("【"+joinPoint.getSignature().getName()+"】方法出现异常了,异常信息是："+exception);
    }
    //方法结束
    @After("execution(public int com.xiao.aop.MyCalculator.*(int,int))")
    public  static void end(JoinPoint joinPoint) {
        System.out.println("【"+joinPoint.getSignature().getName()+"】方法最终结束了");
    }
```

#### 06 抽取可重用的切入点表达式

自定义一个没有返回值和参数的方法，加上`@Pointcut`注解，声明切入点表达式，别的地方可以直接使用其方法名进行引用

```java
@Pointcut("execution(public int com.xiao.aop.MyCalculator.*(int,int))")
public static void myPoint(){

}

//执行前
@Before("myPoint()")
public static void before(JoinPoint joinPoint) {
...
}
```

#### 07 环绕通知

`@Around`：就是利用反射调用目标方法，可以在其中定义环绕前置、环绕返回、环绕异常和环绕后置通知。**环绕通知是优先于普通通知执行的。**

环绕通知只作用在自己的切面内。

```java
@Around("myPoint()")
public Object myAround(ProceedingJoinPoint point) throws Throwable {
    //获取参数
    Object[] args = point.getArgs();
    //获取方法名
    String name = point.getSignature().getName();
    Object proceed = null;

    try {
        // @Before
        System.out.println("【环绕前置通知】..【" + name + "】方法开始，用的参数列表是" + Arrays.asList(args));
        //就是利用反射调用目标方法，类似于method.invoke(obj,args)
        proceed = point.proceed(args);
        // @AfterReturning
        System.out.println("【环绕返回通知】..【" + name + "】方法返回，返回值是" + proceed);
    } catch (Exception e) {
        // @AfterThrowing
        System.out.println("【环绕异常通知】..【" + name + "】方法出现异常，异常信息是" + e);
        //为了让外界知道这个异常，将其抛出
         throw new RuntimeException(e);
    } finally {
        // @After
        System.out.println("【环绕后置通知】..【" + name + "】方法结束");
    }
    //反射调用后的返回值也一定返回出去
    return proceed;
}
```

结果：

```java
【环绕前置通知】..【add】方法开始，用的参数列表是[1, 2]
【add】方法开始执行了，用的参数列表是【[1, 2]】
【环绕返回通知】..【add】方法返回，返回值是3
【环绕后置通知】..【add】方法结束
【add】方法最终结束了
【add】方法执行完成了，执行结果是【3】
```

> 执行顺序：
>
> (环绕前置 ---> 普通前置) ---> 目标方法执行  ---> 环绕正常返回/出现异常  --->  环绕后置 --->  普通后置  ---> 普通返回或者异常



#### 08 多切面情况

执行顺序按照类名顺序，前置1-->前置2-->目标方法 -->后置2-->后置1

![image-20200615165029298](http://image.kongxiao.top/image-20200615165029298.png)

在切面上使用`@Order`注解，给一个int值，值越小，优先级越高

### 2.4 配置文件实现

在容器中注册 bean，相当于`@component`

`<aop:config>`：进行配置。

`<aop:aspect ref="...">：`指定谁是切面类，相当于`@Aspet`

`<aop:pointcutid="..." expression="..."`：指定切入点和切入表达式

`<aop:before method="..." pointcut-ref="..." >`：指定怎么切入，切在哪里，相当于`@Before`等，该标签中也可以指定返回值、异常等信息。

```xml
<!--注册bean-->
<bean id="logUtils" class="com.xiao.aop.LogUtils"/>
<bean id="myCalculator" class="com.xiao.aop.MyCalculator"/>

<aop:config>
    <!--自定义切面aspect，ref:要引用的类-->
  <aop:aspect ref="logUtils">
      <!--切入点-->
      <aop:pointcut id="point" expression="execution(public int com.xiao.aop.MyCalculator.*(int,int))"/>
      <!--前置-->
      <aop:before method="before" pointcut-ref="point"/>
      <!--返回-->
      <aop:after-returning method="after" pointcut-ref="point" returning="result" />
      <!--异常-->
      <aop:after-throwing method="exception" pointcut-ref="point" throwing="exception"/>
      <!--后置-->
      <aop:after method="end" pointcut-ref="point"/>
  </aop:aspect>
</aop:config>
```

