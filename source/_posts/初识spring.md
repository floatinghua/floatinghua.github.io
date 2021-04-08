---
title: 初识spring
date: 2020-07-01 11:35:38
keyword: spring框架
description: 什么是框架?spring详解
cover: https://gitee.com/float97/wutang-imgs/raw/master/20200908141035.png
tags: 
- 框架
categories: 
- spring
---

## 什么是框架?

在特定的领域将总结的最佳实践编写成固定流程，帮助我们更加高效，更为健壮的编写。

  1. **总结的最佳实践** ，这就表明框架已经是完成了一些特定的功能，自己已经实现了一些功能，不同的框架完成了不同的功能；
  2. **固定流程**，说明框架本身是有一套流程，需要实现的功能按照其流程来，就能达到效果。
  3. **帮助我们**，框架本身是不会运行点，在框架的帮助下，配合其他的程序，框架就能有效的帮助我们省去很多步骤，直接调用其本身，来达到功能的实现。
     ![框架与程序员的关系](https://img-blog.csdnimg.cn/20191102104307835.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTQxNzIxMQ==,size_16,color_FFFFFF,t_70#pic_center)

## Spring概诉

Spring 是最受欢迎的企业级 Java 应用程序开发框架，数以百万的来自世界各地的开发人员使用 Spring 框架来创建性能好、易于测试、可重用的代码。

Spring 框架是一个开源的 Java 平台，它最初是由 Rod Johnson 编写的，并且于 2003 年 6 月首次在 Apache 2.0 许可下发布。

Spring 是轻量级的框架，其基础版本只有 2 MB 左右的大小。

Spring 框架的核心特性是可以用于开发任何 Java 应用程序，但是在 Java EE 平台上构建 web 应用程序是需要扩展的。 Spring 框架的目标是使 J2EE 开发变得更容易使用，通过启用基于 POJO 编程模型来促进良好的编程实践。

**Spring的核心是控制反转（IoC）和面向切面（AOP）。**
**简单来说，Spring是一个分层的JavaSE/EEfull-stack（一站式）轻量级开源框架。**

~~上面的干货是不是很枯燥，那我们看下Spring~~   ==集成的模块==

## Spring集成的模块

![Spring集成模块](https://img-blog.csdnimg.cn/20191102105609500.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTQxNzIxMQ==,size_16,color_FFFFFF,t_70)
1 **核心容器**

- Spring-beans:IoC的实现，配置文件的访问、创建和管理；
- Spring-core：核心工具包；
- Spring-context：封装IoC容器，提供扩展功能；
- SpEL:Spring的表达式支持

2.**基础服务**

- AOP：面向切面编程的支持；
- Aspect：模块提供了与 AspectJ 的集成，这是一个功能强大且成熟的面向切面编程（AOP）框架；
- Instrumentation：字节码操作；
- Messaging：对消息服务的支持

3.**功能**

- transactions:对事务的支持；
- JDBC:Spring对JDBC的封装；
- ORM：Spring对ORM的封装

4.**WEB**

- Web 模块提供面向web的基本功能和面向web的应用上下文，比如多部分（multipart）文件上传功能、使用Servlet监听器初始化IoC容器等。它还包括HTTP客户端以及Spring远程调用中与web相关的部分。。

- Web-MVC 模块为web应用提供了模型视图控制（MVC）和REST Web服务的实现。Spring的MVC框架可以使领域模型代码和web表单完全地分离，且可以与Spring框架的其它所有功能进行集成。

- Web-Socket 模块为 WebSocket-based 提供了支持，而且在 web 应用程序中提供了客户端和服务器端之间通信的两种方式。

- Web-Portlet 模块提供了用于Portlet环境的MVC实现，并反映了spring-webmvc模块的功能。

## Spring的核心理念——IoC容器

在讲解IoC的时候我们先来回想，之前写MVC三层的时候
![之前的MVC三层控制权](https://img-blog.csdnimg.cn/20191102113512328.png)
IoC:Inversion of Control 控制反转，通俗一点就是控制权由调用的类，转为Spring容器，由Spring容器来==创建实例以及依赖注入==。
![控制反转](https://img-blog.csdnimg.cn/20191103104511324.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTQxNzIxMQ==,size_16,color_FFFFFF,t_70)

### Bean的四种实例化

首先你需要在xml中来，配置bean。

1. 默认使用的bean就是用的无参构造
   其中id就表示这个bean的名字，class就是你要创建这个实例的class
   -- `<bean id="userDao" class="com.demo.dao.UserDaoImpl"/>`
2. 静态工厂方法实例化
   这就运用到一个类中的一个静态方法，这个静态方法get就是用来返回bean的实例
   -- `<bean id="userDao" factory-method="get"class="com.UserDaoFactory/>`
3. 工厂bean的实例化
   这个方法就用到了工厂模式，这个工厂的方法get用来生产这个bean的实例
   -- `<bean id="userDaoFactory" class="com.factory.user.UserDaoFactory"/>    <bean id="userDao" factory-bean="userDaoFactory" factory-method="get"/>`
4. Spring的factorybean接口
   使用这种方式就是要类已经实现了FactoryBean的接口，然后重写了方法
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20191103110313725.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTQxNzIxMQ==,size_16,color_FFFFFF,t_70)
   -- `<bean id="car" class="com.demo.dao.CarFactory"/>`

### 依赖注入

依赖注入Spring也给我提供了两种方式

1. setter注入
   -- 这个方式就是说，你要实例化的对象，必须要有setter方法来设置，然后才能才bean中进行依赖注入

```xml
<bean id="userService" class="com.demo.service.UserServiceImpl/>

<bean id="userController" class="com.demo.controller.UserController">
	//这里的name其实就是调用的setter方法
	<property name="userService" ref="userService"/>
</bean>
```

```xml
//当然你想注入其他类型的时候也是可以的
//value 注入基本类型	
<property name="" value="12"/></property>

//list 存放基本类型注入list 
<property name="" />
	<list>
		<value></value>
		<value></value>
   </list>
</property>

//map集合
<property name=""/>
	<map>
		<entry key="" value=""/>
		<entry key="" value=""/>
		<entry key="" value=""/>
</map>
</property>
```

2. 构造器注入
   ![构造器注入](https://img-blog.csdnimg.cn/20191103111839165.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTQxNzIxMQ==,size_16,color_FFFFFF,t_70)
   ==看完了这些实例化和依赖注入，再来看看怎么使用吧==
   ![容器的使用](https://img-blog.csdnimg.cn/201911031120165.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTQxNzIxMQ==,size_16,color_FFFFFF,t_70)
   **其中一般来说全局就一个spring容器**

### bean的作用域范围

prototype	原 型 => 每次创建一个实例
singleton 	单 例 =>一个bean的定义，只有一个实例，不是之前那种单例 一个类只有一个
request		一个请求一个实例
session		一个会话一个实例
websocket	一次websocket链接一个实例

### bean的生命周期

声明周期两种方式：==只有单例有效果==

1. 自己写开始结束的方法
   在bean的配置中，添加init-method=“初始化的方法”，destroy-method=“销毁的方法”

2. 实现开始结束的接口 重写方法 
   在类上实现InitializingBean,DisposableBean 方法

## IoC的配置

### 1. 通过注解来配置IoC

在XML中配置

```xml
<context:component-scan base-package="要扫描的注解的包路径“/>
```

==注解有==
@Controller
@Service
@Repository 这是dao的
@Autowired 自动依赖注入 就可以不用setter
@Component 不是上面三层的就统一用这个
@Value 注入基本数据类型 可以注入配置的值


作用域用：
@Scope（“propertype”）
生命周期 
@PostConstruct  初始化的方法上加
@PreDestroy	摧毁的方法上加

==**当你加了注解的时候，Spring会自动将类名第一个首字母小写作为bean的id，因为是加在类上，所以class也省略配置了**==

### 2.Java类配置IoC

*使用java类来代替XML*

**@Configuration 用于标记一个类为配置类
@Bean  标记某个方法返回值为spring的bean
@ComponentScan（要扫描的包） 打开扫描包的注解**

```java
@Configuration
public class AppConfig {
    
    @Bean
    public UserService userService(){
//        第一种推荐 @Bean就是来获得对象的 要注入信息就调用方法
        UserService userService = new UserService();
        userService.setUserDao(userDao());
        return userService;
    }
    
    @Bean
    public UserDao userDao(){
        return new UserDao();
    }
    
    @Bean
    public UserController userController(UserService userService){
//        第二种 通过传入参数 spring自己找对应的bean
        UserController userController=new UserController();
        userController.setUserService(userService);
        return userController;
    }
    
    //   获得DruidDataSource
    @Bean
    public DruidDataSource druidDataSource(){
        DruidDataSource dds = new DruidDataSource();
//        设置每个属性
        dds.setDriverClassName("com.mysql.jdbc.Driver");
        dds.setUsername("root");
        dds.setPassword("root123");
        dds.setUrl("jdbc:mysql://localhost:3306/demo);
        return  dds;
    }
    
    @Bean
    public QueryRunner queryRunner(){
//        指定druidDataSource当然也可以不指定 让spring自己去找
        QueryRunner qr = new QueryRunner(druidDataSource());
        return qr;
    }   
}
```

```java
@PropertySource("DataBase.properties")

@Value("${driverClassName}")
String driverClassName;

@Value("${url}")
String url;

@Value("${username}")
String username;

@Value("${password}")
String password;
```

**在使用的时候就得使用下面的进行调用了**

```java
AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(AppConfigScan.class);
```

@ComponentScan（要扫描的包） 打开扫描包的注解等价于
<context:component-scan base-package="xxx">

 这个时候就要用 @Controller @Service 直接生成bean 就不用自己@Bean来生成bean

### 多种的混用

在注解类上可以使用@Import（.class）导入Java配置  用@ImportResource(.xml)导入Xml的配置

==讲了这么久 讲讲IoC的优势==

### IoC优势

1. 解耦合 （一个类跟一个类的关系就是耦合度）  降低了类与类之间的耦合度
2. 提升了代码的灵活性，可维护性。因为一个类可能有多个实现，当不同需求的时候在xml中切换下就行

## AOP

OOP：这是刚开始学Java就学的面相对象编程
今天讨论的是AOP：Aspect Oritented Programming

AOP：面相切面编程——为了解决公共、系统的问题

**那么哪些是属于公共、系统的问题呢？**
	比如打印日志，打印参数调用，执行时间，事务管理，安全验证.......
	
了解了这些，我们来看下AOP中的名词

### 名词解释

**连接点**：	JoinPoint 	需要加入功能的位置(通常是方法)

**切入点**：	Pointcut 	真正执行加入功能的连接点，从连接点中选出需要加入功能的连接点

**通知**：		Advice		 需要实现的功能

**切面**：		Aspect 		Java语言中将切入点和通知组装在一起的代码单元

**目标对象**：	Target 		要操作的对象，方法（连接点）所在的对象

**织入**：		Weave		 将功能加入到切入点中的过程

 ### 步骤

 #### 配置方式1（较为麻烦）

  1.  编写service类
 2.  编写通知  ， 实现MethodBeforeAdvice接口（等下详细讲通知）
 3.  配置xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    <!--配置目标对象-->
    <bean id="userService" class="com.spring.aop.service.impl.UserServiceImpl"/>
    <!--配置通知: 实现了打日志的功能-->
    <bean id="beforeExecution" class="com.spring.aop.component.BeforeExecution"/>

    <!--切入点：写到需要增加功能的方法-->
    <bean id="pointCut" class="org.springframework.aop.support.JdkRegexpMethodPointcut">
        <property name="pattern" value="com.spring.aop.service.impl.UserServiceImpl.addUser"/>
    </bean>

    <!--切面：连接切入点和通知，让打日志功能在切入点的位置执行-->
    <bean id="aspect" class="org.springframework.aop.support.DefaultPointcutAdvisor">
        <property name="pointcut" ref="pointCut"/>
        <property name="advice" ref="beforeExecution"/>
    </bean>

    <!--包装userService-->
    <bean class="org.springframework.aop.framework.autoproxy.DefaultAdvisorAutoProxyCreator"/>


</beans>
```

#### 配置方式2 aop：config

非环绕式的通知

```xml
	<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    <!--配置通知-->
	<bean id="tx" class="com.demo.Trans"/>
	<!--配置切面-->
	<aop:config>
		<aop:aspect ref="tx">
			<!--配置切入点-->
			<aop:pointcut id="servicePointcut" expression="executon(* com.demo.aop.*.*(..)"/>
			<!--前置-->
			<aop:before method="begin" pointcut-ref="servicePointcut"/>
			<!--后-->
			<aop:after-returning method="begin" pointcut-ref="servicePointcut"/>
		</aop:aspect>
	</aop:config>
</beans>
```

expression语法：转载 [https://zhuanlan.zhihu.com/p/63001123](https://zhuanlan.zhihu.com/p/63001123) 
环绕式通知，就只需要一个就够了
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191103122312880.png)

#### 注解式配置

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191103122444263.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTQxNzIxMQ==,size_16,color_FFFFFF,t_70)
**注解式的配置切面就如下的配置**：
@Aspect  切面
@Pointcut	切入点
@Around 	环绕式
@Before 	前置
@After 	后置
@AfterThrowing 	抛出异常
@EnableAspectJAutoProxy  自动扫包等价于aop:aspectj-autoproxy<!-- toc -->



## 什么是框架?

在特定的领域将总结的最佳实践编写成固定流程，帮助我们更加高效，更为健壮的编写。

  1. **总结的最佳实践** ，这就表明框架已经是完成了一些特定的功能，自己已经实现了一些功能，不同的框架完成了不同的功能；
  2. **固定流程**，说明框架本身是有一套流程，需要实现的功能按照其流程来，就能达到效果。
  3. **帮助我们**，框架本身是不会运行点，在框架的帮助下，配合其他的程序，框架就能有效的帮助我们省去很多步骤，直接调用其本身，来达到功能的实现。
     ![框架与程序员的关系](https://img-blog.csdnimg.cn/20191102104307835.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTQxNzIxMQ==,size_16,color_FFFFFF,t_70#pic_center)

## Spring概诉

Spring 是最受欢迎的企业级 Java 应用程序开发框架，数以百万的来自世界各地的开发人员使用 Spring 框架来创建性能好、易于测试、可重用的代码。

Spring 框架是一个开源的 Java 平台，它最初是由 Rod Johnson 编写的，并且于 2003 年 6 月首次在 Apache 2.0 许可下发布。

Spring 是轻量级的框架，其基础版本只有 2 MB 左右的大小。

Spring 框架的核心特性是可以用于开发任何 Java 应用程序，但是在 Java EE 平台上构建 web 应用程序是需要扩展的。 Spring 框架的目标是使 J2EE 开发变得更容易使用，通过启用基于 POJO 编程模型来促进良好的编程实践。

**Spring的核心是控制反转（IoC）和面向切面（AOP）。**
**简单来说，Spring是一个分层的JavaSE/EEfull-stack（一站式）轻量级开源框架。**

~~上面的干货是不是很枯燥，那我们看下Spring~~   ==集成的模块==

## Spring集成的模块

![Spring集成模块](https://img-blog.csdnimg.cn/20191102105609500.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTQxNzIxMQ==,size_16,color_FFFFFF,t_70)
1 **核心容器**

- Spring-beans:IoC的实现，配置文件的访问、创建和管理；
- Spring-core：核心工具包；
- Spring-context：封装IoC容器，提供扩展功能；
- SpEL:Spring的表达式支持

2.**基础服务**

- AOP：面向切面编程的支持；
- Aspect：模块提供了与 AspectJ 的集成，这是一个功能强大且成熟的面向切面编程（AOP）框架；
- Instrumentation：字节码操作；
- Messaging：对消息服务的支持

3.**功能**

- transactions:对事务的支持；
- JDBC:Spring对JDBC的封装；
- ORM：Spring对ORM的封装

4.**WEB**

- Web 模块提供面向web的基本功能和面向web的应用上下文，比如多部分（multipart）文件上传功能、使用Servlet监听器初始化IoC容器等。它还包括HTTP客户端以及Spring远程调用中与web相关的部分。。

- Web-MVC 模块为web应用提供了模型视图控制（MVC）和REST Web服务的实现。Spring的MVC框架可以使领域模型代码和web表单完全地分离，且可以与Spring框架的其它所有功能进行集成。

- Web-Socket 模块为 WebSocket-based 提供了支持，而且在 web 应用程序中提供了客户端和服务器端之间通信的两种方式。

- Web-Portlet 模块提供了用于Portlet环境的MVC实现，并反映了spring-webmvc模块的功能。

## Spring的核心理念——IoC容器

在讲解IoC的时候我们先来回想，之前写MVC三层的时候
![之前的MVC三层控制权](https://img-blog.csdnimg.cn/20191102113512328.png)
IoC:Inversion of Control 控制反转，通俗一点就是控制权由调用的类，转为Spring容器，由Spring容器来==创建实例以及依赖注入==。
![控制反转](https://img-blog.csdnimg.cn/20191103104511324.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTQxNzIxMQ==,size_16,color_FFFFFF,t_70)

### Bean的四种实例化

首先你需要在xml中来，配置bean。

1. 默认使用的bean就是用的无参构造
   其中id就表示这个bean的名字，class就是你要创建这个实例的class
   -- `<bean id="userDao" class="com.demo.dao.UserDaoImpl"/>`
2. 静态工厂方法实例化
   这就运用到一个类中的一个静态方法，这个静态方法get就是用来返回bean的实例
   -- `<bean id="userDao" factory-method="get"class="com.UserDaoFactory/>`
3. 工厂bean的实例化
   这个方法就用到了工厂模式，这个工厂的方法get用来生产这个bean的实例
   -- `<bean id="userDaoFactory" class="com.factory.user.UserDaoFactory"/>    <bean id="userDao" factory-bean="userDaoFactory" factory-method="get"/>`
4. Spring的factorybean接口
   使用这种方式就是要类已经实现了FactoryBean的接口，然后重写了方法
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20191103110313725.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTQxNzIxMQ==,size_16,color_FFFFFF,t_70)
   -- `<bean id="car" class="com.demo.dao.CarFactory"/>`

### 依赖注入

依赖注入Spring也给我提供了两种方式

1. setter注入
   -- 这个方式就是说，你要实例化的对象，必须要有setter方法来设置，然后才能才bean中进行依赖注入

```xml
<bean id="userService" class="com.demo.service.UserServiceImpl/>

<bean id="userController" class="com.demo.controller.UserController">
	//这里的name其实就是调用的setter方法
	<property name="userService" ref="userService"/>
</bean>
```

```xml
//当然你想注入其他类型的时候也是可以的
//value 注入基本类型	
<property name="" value="12"/></property>

//list 存放基本类型注入list 
<property name="" />
	<list>
		<value></value>
		<value></value>
   </list>
</property>

//map集合
<property name=""/>
	<map>
		<entry key="" value=""/>
		<entry key="" value=""/>
		<entry key="" value=""/>
</map>
</property>
```

2. 构造器注入
   ![构造器注入](https://img-blog.csdnimg.cn/20191103111839165.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTQxNzIxMQ==,size_16,color_FFFFFF,t_70)
   ==看完了这些实例化和依赖注入，再来看看怎么使用吧==
   ![容器的使用](https://img-blog.csdnimg.cn/201911031120165.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTQxNzIxMQ==,size_16,color_FFFFFF,t_70)
   **其中一般来说全局就一个spring容器**

### bean的作用域范围

prototype	原 型 => 每次创建一个实例
singleton 	单 例 =>一个bean的定义，只有一个实例，不是之前那种单例 一个类只有一个
request		一个请求一个实例
session		一个会话一个实例
websocket	一次websocket链接一个实例

### bean的生命周期

声明周期两种方式：==只有单例有效果==

1. 自己写开始结束的方法
   在bean的配置中，添加init-method=“初始化的方法”，destroy-method=“销毁的方法”

2. 实现开始结束的接口 重写方法 
   在类上实现InitializingBean,DisposableBean 方法

## IoC的配置

### 1. 通过注解来配置IoC

在XML中配置

```xml
<context:component-scan base-package="要扫描的注解的包路径“/>
```

==注解有==
@Controller
@Service
@Repository 这是dao的
@Autowired 自动依赖注入 就可以不用setter
@Component 不是上面三层的就统一用这个
@Value 注入基本数据类型 可以注入配置的值


作用域用：
@Scope（“propertype”）
生命周期 
@PostConstruct  初始化的方法上加
@PreDestroy	摧毁的方法上加

==**当你加了注解的时候，Spring会自动将类名第一个首字母小写作为bean的id，因为是加在类上，所以class也省略配置了**==

### 2.Java类配置IoC

*使用java类来代替XML*

**@Configuration 用于标记一个类为配置类
@Bean  标记某个方法返回值为spring的bean
@ComponentScan（要扫描的包） 打开扫描包的注解**

```java
@Configuration
public class AppConfig {
    
    @Bean
    public UserService userService(){
//        第一种推荐 @Bean就是来获得对象的 要注入信息就调用方法
        UserService userService = new UserService();
        userService.setUserDao(userDao());
        return userService;
    }
    
    @Bean
    public UserDao userDao(){
        return new UserDao();
    }
    
    @Bean
    public UserController userController(UserService userService){
//        第二种 通过传入参数 spring自己找对应的bean
        UserController userController=new UserController();
        userController.setUserService(userService);
        return userController;
    }
    
    //   获得DruidDataSource
    @Bean
    public DruidDataSource druidDataSource(){
        DruidDataSource dds = new DruidDataSource();
//        设置每个属性
        dds.setDriverClassName("com.mysql.jdbc.Driver");
        dds.setUsername("root");
        dds.setPassword("root123");
        dds.setUrl("jdbc:mysql://localhost:3306/demo);
        return  dds;
    }
    
    @Bean
    public QueryRunner queryRunner(){
//        指定druidDataSource当然也可以不指定 让spring自己去找
        QueryRunner qr = new QueryRunner(druidDataSource());
        return qr;
    }   
}
```

```java
@PropertySource("DataBase.properties")

@Value("${driverClassName}")
String driverClassName;

@Value("${url}")
String url;

@Value("${username}")
String username;

@Value("${password}")
String password;
```

**在使用的时候就得使用下面的进行调用了**

```java
AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(AppConfigScan.class);
```

@ComponentScan（要扫描的包） 打开扫描包的注解等价于
<context:component-scan base-package="xxx">

 这个时候就要用 @Controller @Service 直接生成bean 就不用自己@Bean来生成bean

### 多种的混用

在注解类上可以使用@Import（.class）导入Java配置  用@ImportResource(.xml)导入Xml的配置

==讲了这么久 讲讲IoC的优势==

### IoC优势

1. 解耦合 （一个类跟一个类的关系就是耦合度）  降低了类与类之间的耦合度
2. 提升了代码的灵活性，可维护性。因为一个类可能有多个实现，当不同需求的时候在xml中切换下就行

## AOP

OOP：这是刚开始学Java就学的面相对象编程
今天讨论的是AOP：Aspect Oritented Programming

AOP：面相切面编程——为了解决公共、系统的问题

**那么哪些是属于公共、系统的问题呢？**
	比如打印日志，打印参数调用，执行时间，事务管理，安全验证.......
	
了解了这些，我们来看下AOP中的名词

### 名词解释

**连接点**：	JoinPoint 	需要加入功能的位置(通常是方法)

**切入点**：	Pointcut 	真正执行加入功能的连接点，从连接点中选出需要加入功能的连接点

**通知**：		Advice		 需要实现的功能

**切面**：		Aspect 		Java语言中将切入点和通知组装在一起的代码单元

**目标对象**：	Target 		要操作的对象，方法（连接点）所在的对象

**织入**：		Weave		 将功能加入到切入点中的过程

 ### 步骤

 #### 配置方式1（较为麻烦）

  1.  编写service类
 2.  编写通知  ， 实现MethodBeforeAdvice接口（等下详细讲通知）
 3.  配置xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    <!--配置目标对象-->
    <bean id="userService" class="com.spring.aop.service.impl.UserServiceImpl"/>
    <!--配置通知: 实现了打日志的功能-->
    <bean id="beforeExecution" class="com.spring.aop.component.BeforeExecution"/>

    <!--切入点：写到需要增加功能的方法-->
    <bean id="pointCut" class="org.springframework.aop.support.JdkRegexpMethodPointcut">
        <property name="pattern" value="com.spring.aop.service.impl.UserServiceImpl.addUser"/>
    </bean>

    <!--切面：连接切入点和通知，让打日志功能在切入点的位置执行-->
    <bean id="aspect" class="org.springframework.aop.support.DefaultPointcutAdvisor">
        <property name="pointcut" ref="pointCut"/>
        <property name="advice" ref="beforeExecution"/>
    </bean>

    <!--包装userService-->
    <bean class="org.springframework.aop.framework.autoproxy.DefaultAdvisorAutoProxyCreator"/>


</beans>
```

#### 配置方式2 aop：config

非环绕式的通知

```xml
	<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    <!--配置通知-->
	<bean id="tx" class="com.demo.Trans"/>
	<!--配置切面-->
	<aop:config>
		<aop:aspect ref="tx">
			<!--配置切入点-->
			<aop:pointcut id="servicePointcut" expression="executon(* com.demo.aop.*.*(..)"/>
			<!--前置-->
			<aop:before method="begin" pointcut-ref="servicePointcut"/>
			<!--后-->
			<aop:after-returning method="begin" pointcut-ref="servicePointcut"/>
		</aop:aspect>
	</aop:config>
</beans>
```

expression语法：转载 [https://zhuanlan.zhihu.com/p/63001123](https://zhuanlan.zhihu.com/p/63001123) 
环绕式通知，就只需要一个就够了
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191103122312880.png)

#### 注解式配置

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191103122444263.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTQxNzIxMQ==,size_16,color_FFFFFF,t_70)
**注解式的配置切面就如下的配置**：
@Aspect  切面
@Pointcut	切入点
@Around 	环绕式
@Before 	前置
@After 	后置
@AfterThrowing 	抛出异常
@EnableAspectJAutoProxy  自动扫包等价于aop:aspectj-autoproxy