# Spring框架笔记

## Spring

说明：Spring是一个基于IOC和AOP的结构J2EE系统的框架 

参考文档：https://www.cnblogs.com/wmyskxz/p/8820371.html

### 什么是Spring

1. Spring 是一个轻量级的 DI / IoC 和 AOP 容器的开源框架

2. Spring 提倡以“最少侵入”的方式来管理应用中的代码，这意味着我们可以随时安装或者卸载 

3. Spring适用范围：任何 Java 应用

4. Spring 的根本使命：简化 Java 开发

### Spring 中常用术语

1. 框架：是能完成一定功能的半成品。框架能够帮助我们完成的是：项目的整体框架、一些基础功能、规定了类和对象如何创建，如何协作等，当我们开发一个项目时，框架帮助我们完成了一部分功能，我们自己再完成一部分，那这个项目就完成了。

2. 非侵入式设计：从框架的角度可以理解为：无需继承框架提供的任何类这样我们在更换框架时，之前写过的代码几乎可以继续使用。

3. 轻量级和重量级：轻量级是相对于重量级而言的，轻量级一般就是非入侵性的、所依赖的东西非常少、资源占用非常少、部署简单等等，其实就是比较容易使用，而重量级正好相反。

4. JavaBean：即符合 JavaBean 规范的 Java 类POJO：即 Plain Old Java Objects，简单老式 Java 对象，它可以包含业务逻辑或持久化逻辑，但不担当任何特殊角色且不继承或不实现任何其它Java框架的类或接口。

5. 容器：在日常生活中容器就是一种盛放东西的器具，从程序设计角度看就是装对象的对象，因为存在放入、拿出等操作，所以容器还要管理对象的生命周期。一般使用ApplicationContext容器，ApplicationContext容器包含BeanFactory容器的所有功能。

 

### Spring 的优势

1. 低侵入 / 低耦合 （降低组件之间的耦合度，实现软件各层之间的解耦）

2. 声明式事务管理（基于切面和惯例）

3. 方便集成其他框架（如MyBatis、Hibernate）

4. 降低 Java 开发难度

5. Spring 框架中包括了 J2EE 三层的每一层的解决方案（一站式）

**J2EE开发三层架构扩展**：表现层（UI）、业务逻辑层（BLL）、数据访问层（DAL）

1、 表现层（UI）：通俗讲就是展现给用户的界面，即用户在使用一个系统的时候他的所见所得。表现层的主流框架有：struts1 ，struts2,，springMVC，webwork

 

2、 业务逻辑层（BLL）：针对具体问题的操作，也可以说是对数据层的操作，对数据业务逻辑处理。业务逻辑层的主流框架有：Spring

 

3、 数据访问层（DAL）：该层所做事务直接操作数据库，针对数据的增添、删除、修改、查找等。数据访问层测主流框架有：Hibernate，Ibatis,以及Ibatis的升级版Mybatis

 

### Spring 的框架结构

![https://upload-images.jianshu.io/upload_images/7896890-a7c003d175bd41af.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240](https://cdn.jsdelivr.net/gh/kender1314/NotePicture/20200722234226.png)

 

1. Data Access/Integration层包含有JDBC、ORM、OXM、JMS和Transaction模块。

2. Web层包含了Web、Struts、Web-Servlet、WebSocket、Web-Porlet模块。

3. AOP模块提供了一个符合AOP联盟标准的面向切面编程的实现。

4. Core Container(核心容器)：包含有Beans、Core、Context和SpEL模块。

5. Test模块支持使用JUnit和TestNG对Spring组件进行测试。

 

### Spring IoC 和 DI 简介

#### Java创建对象有哪些方式

1. 构造方法，new
2. 反射
3. 序列化
4. 克隆
5. Ioc：容器创建对象
6. 动态代理

#### **IoC：Inverse of Control（控制反转）**

定义：读作“反转控制”，更好理解，不是什么技术，而是一种设计思想，就是将原本在程序中手动创建对象的控制权，交由容器（Spring框架）来管理。

1. 控制：创建对象，对象的属性赋值，对象之间的关系管理。

2. 正转：若要使用某个对象，需要自己去负责对象的创建。

3. 反转：若要使用某个对象，只需要从 Spring 容器中获取需要使用的对象，不关心对象的创建过程，也就是把创建对象的控制权反转给了容器（Spring框架）。

   

总结：

1. 传统的方式：通过new 关键字主动创建一个对象

2. IOC方式：对象的生命周期由Spring来管理，直接从Spring那里去获取一个对象。 IOC是反转控制 (Inversion Of Control)的缩写，就像控制权从本来在自己手里，交给了Spring。

 

![https://upload-images.jianshu.io/upload_images/7896890-bb752724e10e0df2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240](https://cdn.jsdelivr.net/gh/kender1314/NotePicture/20200722234300.png)

 

**DI：Dependency Injection（依赖注入）**

<!--DI是IOC技术实现的一种方式。-->

指 Spring 创建对象的过程中，将对象依赖属性（简单值，集合，对象）通过配置设值给该对象

自我描述：使用DI之后，就不用再new类对象了，一切交给容器来创建

![img](https://cdn.jsdelivr.net/gh/kender1314/NotePicture/20200722234306.jpg)

 

总结：Spring是使用DI实现了IOC的功能，Spring底层创建对象，是使用的反射机制。

#### 注解

##### 使用注解的步骤

1. 使用注解，必须将spring-aop的包加入项目中（当项目中加入spring-context的时候，就会间接地加入spring-aop依赖）
2. 在类中加入spring的注解
3. 在spring的配置文件中，加入一个组件扫描器的标签，说明注解在你项目中的位置

##### 注解

1. @Component

2. @Respotory（放在持久层：dao的实现类上面，访问数据库）

3. @Service（业务层：Service可以做业务处理，可以有事务等功能）

4. @Controller（控制层：能接收用户提交的参数，实现请求的处理结果）

   以上四个基本功能相同，都是创建对象，但是下面三个有额外功能。

5. @Value（为指定字段注入值）

6. @Autowired

7. @Resource（来源于JDK的注解）

### Spring AOP

#### Spring AOP 简介

（AOP全名Aspect Oriented Program）

首先，在面向切面编程的思想里面，把功能分为核心业务功能，和周边功能。

1. 所谓的核心业务，比如登陆，增加数据，删除数据都叫核心业务

2. 所谓的周边功能，比如性能统计，日志，事务管理等等

周边功能在 Spring 的面向切面编程AOP思想里，即被定义为切面

在面向切面编程AOP的思想里面，核心业务功能和切面功能分别独立进行开发，然后把切面功能和核心业务功能 "编织" 在一起，这就叫AOP

 

#### **AOP 的目的**

AOP能够将那些与业务无关，却为业务模块所共同调用的逻辑或责任（例如事务处理、日志管理、权限控制等）封装起来，便于减少系统的重复代码，降低模块间的耦合度，并有利于未来的可拓展性和可维护性。

 

#### **AOP 当中的概念**

1. 切入点（Pointcut）：在哪些类，哪些方法上切入（where）
2. 通知（Advice）：在方法执行的什么实际（when:方法前/方法后/方法前后）做什么（what:增强的功能）
3. 切面（Aspect）：表示增强功能，就是一堆代码，完成某个功能。非业务功能，常见的切面有事务，统计信息，参数检查参数检查，权限认证等。
4. 织入（Weaving）：把切面加入到对象，并创建出代理对象的过程。（由 Spring 来完成）

#### AOP的实现

aop是一种规范，是动态的一个规范，一个标准。

aop 的技术实现框架：

1. spring：spring在内部实现了aop的规范，能做aop的工作。spring主要在事务处理时使用aop。我们在项目开发中很少使用spring的aop实现，因为spring的aop比较笨重。

2. aspectJ：一个开源的专门做aop的框架。spring框架中集成了aspectJ框架，通过spring就可以使用aspectJ的功能。

   使用aspectJ框架实现aop有两种方式：

   1. 使用xml的配置文件：配置全局事务。
   2. 使用注解

#### AspectJ框架的使用

##### 切面的执行时间

@Before

@AfterReturning

@Around

@AfterThrowing

@After

@Pointcut

##### AspectJ切入点表达式

execution(...)



####  动态代理

在不改变目标对象方法的情况下对方法进行增强，这就是动态代理。比如，我们希望对方法的调用增加日志记录，或者对方法的调用进行拦截，等等...

##### JDK动态代理

![image-20200820224627810](https://cdn.jsdelivr.net/gh/kender1314/NotePicture/20200820224629.png)

##### CGLIB动态代理

![image-20200820224652060](https://cdn.jsdelivr.net/gh/kender1314/NotePicture/20200820224653.png)



## Spring MVC

参考文档：https://www.cnblogs.com/wmyskxz/p/8848461.html#4183609

### MVC 设计概述

在早期 Java Web 的开发中，统一把显示层、控制层、数据层的操作全部交给 JSP 或者 JavaBean 来进行处理

 

Model1：

![img](https://cdn.jsdelivr.net/gh/kender1314/NotePicture/20200722234314.jpg)

 

出现的弊端：

-  JSP 和 Java Bean 之间严重耦合，Java 代码和 HTML 代码也耦合在了一起
-  要求开发者不仅要掌握 Java ，还要有高超的前端水平
-  前端和后端相互依赖，前端需要等待后端完成，后端也依赖前端完成，才能进行有效的测试代码难以复用

正因为上面的种种弊端，所以很快这种方式就被 Servlet + JSP + Java Bean 所替代了，早期的 MVC 模型（Model2）就像下图这样

 

![img](https://cdn.jsdelivr.net/gh/kender1314/NotePicture/20200722234318.png)

 

首先用户的请求会到达 Servlet，然后根据请求调用相应的 Java Bean，并把所有的显示结果交给 JSP 去完成，这样的模式我们就称为 MVC 模式。

 

M 代表 模型（Model）

模型是什么呢？ 模型就是数据，就是 dao,bean

V 代表 视图（View）

视图是什么呢？ 就是网页, JSP，用来展示模型中的数据

C 代表 控制器（controller)

控制器是什么？ 控制器的作用就是把不同的数据(Model)，显示在不同的视图(View)上，Servlet 扮演的就是这样的角色。

 

### Spring MVC 的架构

为解决持久层中一直未处理好的数据库事务的编程，又为了迎合 NoSQL 的强势崛起，Spring MVC 给出了方案：

![img](https://cdn.jsdelivr.net/gh/kender1314/NotePicture/20200722234528.jpg)

传统的模型层被拆分为了业务层(Service)和数据访问层（DAO,Data Access Object）。在 Service 下可以通过 Spring 的声明式事务操作数据访问层，而在业务层上还允许我们访问 NoSQL ，这样就能够满足异军突起的 NoSQL 的使用了，它可以大大提高互联网系统的性能。

 

特点：

- 结构松散，几乎可以在 Spring MVC 中使用各类视图
- 松耦合，各个模块分离
- 与 Spring 无缝集成

 

### 跟踪 Spring MVC 的请求

每当用户在 Web 浏览器中点击链接或者提交表单的时候，请求就开始工作了，像是邮递员一样，从离开浏览器开始到获取响应返回，它会经历很多站点，在每一个站点都会留下一些信息同时也会带上其他信息，下图为 Spring MVC 的请求流程：

 

![img](https://cdn.jsdelivr.net/gh/kender1314/NotePicture/20200722234524.png)

 

### 使用注解配置 Spring MVC

在 dispatcher-servlet.xml 文件中，注释掉之前的配置，然后增加一句组件扫描：

```
<!-- 扫描controller下的组件 --> 

<context:component-scan base-package="controller"/>
```

 

在类中：

@Controller 注解：

很明显，这个注解是用来声明控制器的，但实际上这个注解对 Spring MVC 本身的影响并不大。

@RequestMapping 注解：

很显然，这就表示路径 /hello 会映射到该方法上

###  domain的概念

domain的概念，通常会分很多层，比如经典的三层架构，控制层、业务层、数据访问层（DAO），此外，还有一个层，就是domain层

domain层，通常就是用于放置这个系统中，与数据库中的表，一一对应起来的JavaBean的



model层：和domain区别；可能都是javaBean，

这个区别是用途不同，domain通常就代表了与数据库表--一一对应的javaBean,

model通常代表了不与数据库一一对应的javaBean，但是封装的数据是前端的JS脚本，需要使用的数据



### 配置视图解析器

还记得我们 Spring MVC 的请求流程吗，视图解析器负责定位视图，它接受一个由 DispaterServlet 传递过来的逻辑视图名来匹配一个特定的视图。

 

**需求：** 

有一些页面我们不希望用户用户直接访问到，例如有重要数据的页面，例如有模型数据支撑的页面。

**造成的问题：**

我们可以在【web】根目录下放置一个【test.jsp】模拟一个重要数据的页面，我们什么都不用做，重新启动服务器，网页中输入 localhost/test.jsp 就能够直接访问到了，这会造成数据泄露...

另外我们可以直接输入 localhost/index.jsp 试试，根据我们上面的程序，这会是一个空白的页面，因为并没有获取到 ${message} 参数就直接访问了，这会影响用户体验

**解决方案：**

我们将我们的 JSP 文件配置在【WEB-INF】文件夹中的【page】文件夹下，【WEB-INF】是 Java Web 中默认的安全目录，是不允许用户直接访问的（也就是你说你通过 localhost/WEB-INF/ 这样的方式是永远访问不到的）

 

但是我们需要将这告诉给视图解析器，我们在 dispatcher-servlet.xml 文件中做如下配置：

![img](https://cdn.jsdelivr.net/gh/kender1314/NotePicture/20200722234355.jpg)

代码：

```
<bean id="viewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver"> 
<property name="prefix" value="/WEB-INF/page/" /> 
<property name="suffix" value=".jsp" /> 
</bean>
```

 

这里配置了一个 Spring MVC 内置的一个视图解析器，该解析器是遵循着一种约定：会在视图名上添加前缀和后缀，进而确定一个 Web 应用中视图资源的物理路径的。

 

 

### 文件上传

在 Spring MVC 中如何实现文件的上传和下载

 

注意： 需要先导入 commons-io-1.3.2.jar 和 commons-fileupload-1.2.1.jar 两个包

第一步：配置上传解析器

在 dispatcher-servlet.xml 中新增一句：

 

```
<bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver"/>
```

开启对上传功能的支持

 

第二步：编写 JSP

文件名为 upload.jsp，仍创建在【page】下：

 

```
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
  <title>测试文件上传</title>
</head>
<body>
<form action="/upload" method="post" enctype="multipart/form-data">
  <input type="file" name="picture">
  <input type="submit" value="上 传">
</form>
</body>
</html>
```

 

第三步：编写控制器

在 Package【controller】下新建【UploadController】类：

 

```
package controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.multipart.MultipartFile;
import org.springframework.web.servlet.ModelAndView;

 
@Controller
public class UploadController {

  @RequestMapping("/upload")
  public void upload(@RequestParam("picture") MultipartFile picture) throws Exception {
​    System.out.println(picture.getOriginalFilename());
  }

  @RequestMapping("/test2")

  public ModelAndView upload() {
​    return new ModelAndView("upload");
  }
}
```

 

 

第四步：测试

在浏览器地址栏中输入：localhost/test2 ，选择文件点击上传，测试成功：

 

 

 

 

 

 

## Mybatis

 

 

## mybatis的逆向工程

 

 

 

##  事务

 

###  什么是事务？

事务是指是程序中一系列严密的逻辑操作，而且所有操作必须全部成功完成，否则在每个操作中所作的所有更改都会被撤消。

 

### Spring中，事务用在什么地方？

在Spring中，事务一般是用在服务层的业务方法上，因为业务方法会调用多个到dao方法，执行多个sql语句



### JDBC和Mybatis访问数据库怎么处理事务？

jdbc访问数据库，处理事务  Connection conn;  conn.commit(); conn.rollback();

mybatis访问数据，处理事务 SqlSession.commit(); SqlSession.rollback();



缺点：虽然底层封装的都是一样的，但是如果使用的话，还是需要了解各种范文数据库的使用方法，麻烦。

### Spring处理事务

spring提供了一种处理事务的统一模型，能使用统一的步骤，方式完成多种不同数据库访问技术的事务处理。



也就是说，可以使用spring的事务处理机制，可以完成mybatis，JPA等访问数据库的事务处理



###  spring支持编程式事务管理和声明式事务管理两种方式。

1. 编程式事务使用TransactionTemplate或者直接使用底层的PlatformTransactionManager。对于编程式事务管理，spring推荐使用TransactionTemplate。
2. 声明式事务是建立在AOP之上的。其本质是对方法前后进行拦截，然后在目标方法开始之前创建或者加入一个事务，在执行完目标方法之后根据执行情况提交或者回滚事务。声明式事务最大的优点就是不需要通过编程的方式管理事务，这样就不需要在业务逻辑代码中掺杂事务管理的代码，只需在配置文件中做相关的事务规则声明(或通过基于@Transactional注解的方式)，便可以将事务规则应用到业务逻辑中。

 

###  事务的类型

#### 事务的隔离级别：有4个值。

1. DEFAULT：采用 DB 默认的事务隔离级别。MySql 的默认为 REPEATABLE_READ； Oracle默认为 READ_COMMITTED。
2. READ_UNCOMMITTED：读未提交。未解决任何并发问题。
3. READ_COMMITTED：读已提交。解决脏读，存在不可重复读与幻读。
4. REPEATABLE_READ：可重复读。解决脏读、不可重复读，存在幻读
5. SERIALIZABLE：串行化。不存在并发问题。

#### 事务的超时时间

表示一个方法最长的执行时间，如果方法执行时超过了时间，事务就回滚。

单位是秒， 整数值， 默认是 -1. 

#### 事务的传播行为

控制业务方法是不是有事务的， 是什么样的事务的。

7个传播行为，表示你的业务方法调用时，事务在方法之间是如果使用的。

1. PROPAGATION_REQUIRED（支持当前事务，如果当前没有事务，就新建一个事务。这是最常见的选择。 ）

2. PROPAGATION_REQUIRES_NEW（新建事务，如果当前存在事务，把当前事务挂起，直到新事务执行完毕，再执行存在的事务。 ）

3. PROPAGATION_SUPPORTS（支持当前事务，如果当前没有事务，就以非事务方式执行。 ）

   以上三个需要掌握的

4. PROPAGATION_MANDATORY（支持当前事务，如果当前没有事务，就抛出异常。 ）

5. PROPAGATION_NESTED

6. PROPAGATION_NEVER

7. PROPAGATION_NOT_SUPPORTED

##### PROPAGATION_REQUIRED

假如当前正要执行的事务不在另外一个事务里，那么就起一个新的事务 
比如说，ServiceB.methodB的事务级别定义为PROPAGATION_REQUIRED, 那么由于执行ServiceA.methodA的时候

 1、如果ServiceA.methodA已经起了事务，这时调用ServiceB.methodB，ServiceB.methodB看到自己已经运行在ServiceA.methodA的事务内部，就不再起新的事务。这时只有外部事务并且他们是共用的，所以这时ServiceA.methodA或者ServiceB.methodB无论哪个发生异常methodA和methodB作为一个整体都将一起回滚。

 2、如果ServiceA.methodA没有事务，ServiceB.methodB就会为自己分配一个事务。这样，在ServiceA.methodA中是没有事务控制的。只是在ServiceB.methodB内的任何地方出现异常，ServiceB.methodB将会被回滚，不会引起ServiceA.methodA的回滚





####    事务提交事务，回滚事务的时机

1. 当你的业务方法，执行成功，没有异常抛出，当方法执行完毕，spring在方法执行后提交事务。事务管理器commit

2. 当你的业务方法抛出运行时异常或ERROR， spring执行回滚，调用事务管理器的rollback

   运行时异常的定义： RuntimeException  和他的子类都是运行时异常， 例如NullPointException , NumberFormatException

3. 当你的业务方法抛出非运行时异常， 主要是受查异常时，提交事务

   受查异常：在你写代码中，必须处理的异常。例如IOException, SQLException

 

 

 

###