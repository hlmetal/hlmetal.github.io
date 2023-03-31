---
layout: post
title:  "Spring框架的基本知识"
date:   2018-04-25 08:00:30 +0200
categories: java spring
---

Spring是开源的用来简化企业级应用开发的应用开发框架。Spring对常用的一些api(比如jdbc)做了封装，这样代码会大大简化，而且代码质量也会提高(比如，使用spring jdbc访问数据库，就不用考虑获取连接与关闭连接)

## Spring简介
Spring是开源的用来简化企业级应用开发的应用开发框架。Spring对常用的一些api(比如jdbc)做了封装，这样代码会大大简化，而且代码质量也会提高(比如，使用spring jdbc访问数据库，就不用考虑获取连接与关闭连接)
## Spring生态初识
### Spring Framework
用于构建企业级应用的轻量级一站式解决方案
* 保持向后兼容性
* 专注API设计
* 追求严苛的代码质量
* 可选择性

### Spring Boot
快速构建基于Spring的应用程序
* 非常快
* 开箱即用，按需改动
* 提供各种非功能特性
* 不用生产代码，没有XML配置

### Spring Cloud
简化分布式系统的开发
* 配置管理
* 服务注册、服务发现、服务追踪
* 熔断
* ···

## Spring特点
* 解耦：spring可以帮助管理对象(帮助创建对象并管理对象之间依赖关系)，这样软件更容易维护
* 集成：spring可以集成其他的一些框架(比如集成任务调度框架Quartz等)

## Spring容器
是Spring框架中一个核心模块，用来管理对象(创建对象，初始化、销毁以及管理对象之间的依赖关系)
1. 启动Spring容器
* 导包spring_webmvc；
* 添加Spring配置文件
* 启动容器
2. 创建对象
* 无参构造器：
    * 为类添加无参构造器(缺省构造器)；
    * 在配置文件中添加bean元素；
    * 启动容器，调用容器提供的getBean方法;
* 静态工厂方法
* 实例工厂方法
3. 生命周期
* 初始化方法
* 销毁方法
4. 作用域
* 默认情况下，对于一个bean元素，容器只会创建一个实例。
* scope属性:指定作用域，其缺省值是singleton(单例);如果将作用域设置为prototype，则会创建多个实例,并且销毁方法会失效
5. 延迟加载
* 默认情况下，容器启动后，会将所有作用域为“singleton”的bean先实例化
* 容器启动后,对于作用于为“singleton”的bean不再实例化,直到调用getBean方法才会创建。

## IOC和DI
1. IOC(Inversion of Control 控制反转):对象之间的依赖关系由容器来建立
2. DI(dependency injection 依赖注入):容器调用set方法或者构造器来建立对象之间的依赖关系
* set方法注入: 添加相应的set方法; 在配置文件当中，使用`<property>`元素进行配置
* 构造器注入: 添加相应的构造器; 在配置文件中，使用`<constructor-arg>`元素进行配置
* 自动装配: 容器依据某种规则，自动建立对象之间的依赖关系
    * 容器默认不会自动装配,需设置autowire属性值：byName;byType;consturctor
* 注入基本类型的值: value属性
* 注入集合类型的值: List;Set;Map;Properties
3. Spring表达式#{}用于访问其他bean的属性或者读取properties文件的内容

## 注解
1. 使用注解来简化配置文件
2. 组件扫描： 
* 容器启动之后，会扫描指定的包及其子包下面所有的类，如果该类前面有一些特定的注解(如@Component)，则容器会将该类纳入容器进行管理(相当于在配置文件里有一个bean)。
* 该bean默认的id是首字母小写的类名，若要是使用其他名可以Component("xxx")
* 编写步骤：a.在类前添加特定注解；b.在配置文件中，配置组件扫描
3. 依赖注入相关的注解
* @Autowired/@Qualifier可以处理构造器注入和set方法注入
* @Resource 只能处理set方法注入  也可添加到属性前
4. @Value 注入基本类型的值，注入spring表达式的值，该注解也可以添加到set方法前

## springMVC
### MVC(Model View Controller) 模型、视图、控制器
MVC是一种软件架构模式，核心思想是将一个软件可以划分成模型，视图和控制器三种不同类型的模块，其中，模型负责封装业务逻辑的处理，视图负责提供界面(包括数据展现和用户操作界面)，控制器负责协调模型和视图(视图将请求先发送给控制器，由控制器选择对应的模型来处理；模型将处理结果交给控制器，由控制器选择对应的视图来展现数据)
### 使用MVC
 <img src= "/assets/files/MVC.png" alt="加载错误" title="MVC"/>

### MVC优点
1. 方便测试(如直接将业务逻辑写在servlet类中，需要部署才能测试，而写在java类中可以直接测试)
2. 方便维护：修改视图不影响模型，反之亦然。
3. 方便分工协作

### SpringMVC
SpringMVC是用来简化基于MVC架构的web应用程序的开发,是Spring框架的一部分
#### 五大组件
1. DispatcherServlet    前端控制器
2. HandlerMapping   映射处理器
3. Controller   处理器
4. ModelAndView 用于封装处理结果
5. ViewResolver 视图解析器
 <img src= "/assets/files/SpringMVC.png" alt="加载错误" title="SpringMVC"/> 

#### 编程步骤
1. 导包：spring-webmvc
2. 添加spring配置文件
3. 配置DispatcherServlet   
4. Controller类
5. jsp
6. 配置HandlerMapping和ViewResolver
 <img src= "/assets/files/SpringMVC_process.png" alt="加载错误" title="SpringMVC处理过程"/> 

#### 基于注解的springMVC应用
1. 导包：spring-webmvc
2. 添加spring配置文件
3. 配置DispatcherServlet   
4. Controller类
5. jsp
6. 配置文件中添加组件扫描，mvc注解扫描，视图解析器

#### 读取请求参数值
1. 通过request对象
2. 通过@RequestParam注解
3. 通过javabean

#### 向页面传值
1. 绑定数据到request对象
2. 返回ModelAndView对象
3. 将数据添加到ModelMap对象
4. 将数据绑定到session对象

#### 重定向
1. 如果方法的返回值是String  则  `return "redirect:toIndex.do";`
2. 如果方法的返回值是ModelAndView 则
```java
    RedirectView rv=new RedirectView("toIndex.do");
    ModelAndView mav=new ModelAndView(rv);
```
#### 扩展:系统分层
1. 如何分层
    * 表示层(UI层)：   数据展现和操作界面以及请求分发
    * 业务层(服务层)：封装业务处理逻辑
    * 持久层(数据访问层)：负责处理数据访问逻辑
*注：a.上一层通过调用接口调用下一层的服务，当下一层的实现发生改变，不影响上一层*

#### 处理表单中文参数值乱码问题
可以配置springMVC提供的过滤器, 表单提交方式必须设置为post, 客户端的编码必须与过滤器的编码一致

#### 拦截器
1. DispatcherServlet在收到请求之后，如果有拦截器，会先调用拦截器,再调用处理器(Controller)
    * 过滤器属于Servlet规范，而拦截器属于Spring框架
2. 编程步骤：
    * 写一个java类实现HanlerInterceptor接口
    * 在拦截器接口方法中实现拦截处理逻辑
    * 配置拦截器
    * 如果多个拦截器都满足要求，则依据配置的先后顺序执行
 <img src= "/assets/files/Interceptor.png" alt="加载错误" title="拦截器"/>

#### 异常处理
1. 配置简单异常处理器, 只适合对异常进行简单处理
    * 在配置文件中配置SimpleMappingExceptionResolver 
    * 添加相应的异常处理页面
2. 若要对异常做复杂处理，则使用@ExceptionHandler
    * 在处理类中添加一个异常处理方法
    * 添加相应异常处理页面

#### SpringJDBC
Spring对jdbc的简单封装
1. 编程步骤
    * 导包，spring-webmvc，spring-jdbc，ojdbc,dbcp,junit
    * 添加spring配置文件
    * 配置JdbcTemplate模板
    * 调用JdbcTemplate提供的方法来访问数据库；通常将JdbcTemplate注入到DAO

