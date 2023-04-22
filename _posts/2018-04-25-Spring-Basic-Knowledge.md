---
layout: post
title:  "Spring框架的基本知识"
date:   2018-04-25 08:00:30 +0200
categories: java spring
---

Spring是开源的用来简化企业级应用开发的应用开发框架Spring对常用的一些api(比如jdbc)做了封装, 这样代码会大大简化, 而且代码质量也会提高(比如, 使用spring jdbc访问数据库, 就不用考虑获取连接与关闭连接)

## Spring简介
Spring是开源的用来简化企业级应用开发的应用开发框架Spring对常用的一些api(比如jdbc)做了封装, 这样代码会大大简化, 而且代码质量也会提高(比如, 使用spring jdbc访问数据库, 就不用考虑获取连接与关闭连接)

## Spring特点
* 解耦: spring可以帮助管理对象(帮助创建对象并管理对象之间依赖关系), 这样软件更容易维护
* 集成: spring可以集成其他的一些框架(比如集成任务调度框架Quartz等)

## 为什么使用spring
1. 轻量: Spring是轻量的,基本的版本大约2MB
2. IOC/DI: Spring通过IOC容器实现了Bean的生命周期的管理, 以及通过DI实现依赖注入, 从而实现了对象依赖的松耦合管理
3. 面向切面的编程(AOP): Spring支持面向切面的编程, 从而把应用业务逻辑和系统服务分开
4. MVC框架: Spring MVC提供了功能更加强大且更加灵活的Web框架支持
5. 事务管理: Spring通过AOP实现了事务的统一管理, 对应用开发中的事务处理提供了非常灵活的支持
6. 集成测试: 在开发环境即可生成测试
7. 生态庞大: 在业务开发领域,Spring生态几乎提供了非常完善的支持, 其社区的活跃度和技术的成熟度都非常高

## Spring容器
是Spring框架中一个核心模块, 用来管理对象(创建对象, 初始化、销毁以及管理对象之间的依赖关系)
1. 启动Spring容器
* 导包spring_webmvc；
* 添加Spring配置文件
* 启动容器
2. 创建对象
* 无参构造器: 
    * 为类添加无参构造器(缺省构造器)；
    * 在配置文件中添加bean元素；
    * 启动容器, 调用容器提供的getBean方法;
* 静态工厂方法
* 实例工厂方法
3. 生命周期
* 初始化方法
* 销毁方法
4. 作用域
* 默认情况下, 对于一个bean元素, 容器只会创建一个实例
* scope属性:指定作用域, 其缺省值是singleton(单例);如果将作用域设置为prototype, 则会创建多个实例,并且销毁方法会失效
5. 延迟加载
* 默认情况下, 容器启动后, 会将所有作用域为“singleton”的bean先实例化
* 容器启动后,对于作用于为“singleton”的bean不再实例化,直到调用getBean方法才会创建

## IOC和DI
### IOC(控制反转)
对象之间的依赖关系由容器来建立. 在传统的Java开发中, 只能通过new关键字来创建对象, 导致对象的依赖关系比较复杂, 耦合度较高. 把设计好的对象交给了IOC容器控制, 需要用对象时直接从容器中去获取
#### 工作流程
1. 第一阶段,IOC容器初始化
* 这个阶段主要是根据程序中不同的bean的声明方式, 把它们解析和加载后生成BeanDefinition对象, 然后把BeanDefinition注册到IOC容器
2. 第二阶段, 完成Bean初始化及依赖注入
* 通过反射针对没有设置lazy-init属性的单例bean进行初始化
* 完成Bean的依赖注入
3. 第三阶段, Bean的使用
通过@Autowired或者BeanFactory.getBean()从IOC容器中获取指定的bean实例

### DI(依赖注入)
容器调用set方法或者构造器来建立对象之间的依赖关系
### 依赖注入方式
1. set方法注入: 添加相应的set方法; 在配置文件当中, 使用`<property>`元素进行配置
2. 构造器注入: 添加相应的构造器; 在配置文件中, 使用`<constructor-arg>`元素进行配置
3. 自动装配: 容器依据某种规则, 自动建立对象之间的依赖关系. 提供了@Resource和@Autowired这两个注解,分别是根据bean的id和bean的类型来实现依赖注入
4. Spring表达式#{}用于访问其他bean的属性或者读取properties文件的内容

## 注解
1. 使用注解来简化配置文件
2. 组件扫描:  
* 容器启动之后, 会扫描指定的包及其子包下面所有的类, 如果该类前面有一些特定的注解(如@Component), 则容器会将该类纳入容器进行管理(相当于在配置文件里有一个bean)
* 该bean默认的id是首字母小写的类名, 若要是使用其他名可以Component("xxx")
* 编写步骤: a.在类前添加特定注解；b.在配置文件中, 配置组件扫描
3. 依赖注入相关的注解
* @Autowired/@Qualifier可以处理构造器注入和set方法注入
* @Resource 只能处理set方法注入  也可添加到属性前
4. @Value 注入基本类型的值, 注入spring表达式的值, 该注解也可以添加到set方法前

## Spring异步调用
1. 注解方式: 可以在配置类和方法上加特定注解. 在配置类加上@EnableAsync来启用, 使用@Async注解标记需要异步执行的方法
2. 内置线程池方式
* SimpleAsyncTaskExecutor: 它不会复用线程, 每次调用都是启动一个新线程
* ConcurrentTaskExecutor: 它是Java API中Executor实例的适配器
* ThreadPoolTaskExecutor: 最常用的. 它公开了用于配置的bean属性, 并将它包装在TaskExecutor中
* WorkManagerTaskExecutor: 基于CommonJ WorkManager来实现的, 并且是在Spring上下文中的WebLogic或WebSphere中设置CommonJ线程池的工具类
* DefaultManagedTaskExecutor: 主要用于支持JSR-236兼容的运行时环境, 它是使用JNDI获得
ManagedExecutorService, 作为CommonJ WorkManager的替代方案
3. 自定义线程池方式: 通过实现AsyncConfigurer接口或者直接继承AsyncConfigurerSupport类来自定义线程池

## Spring中的bean
构成应用程序主干并由IOC容器实例化、组装和管理的对象称为Bean
### 定义方式
1. XML
* 在同一个XML配置文件里面, 不能存在id相同的两个bean, 否则spring容器启动的时候会报错
* 因为id这个属性表示一个Bean的唯一标志符号, 所以Spring在启动的时候会去验证id的唯一性,一旦发现重复就会报错
* 这个错误发生Spring对XML文件进行解析转化为BeanDefinition的阶段
2. 声明式
Spring3.x以后, 提供@Configuration注解去声明一个配置类,然后使用@Bean注解实现Bean的声明, 取代了XML
* 如果我们在同一个配置类里面声明多个相同名字的bean,在Spring IOC容器中只会注册第一个声明的Bean的实例
* 假设使用@Bean定义两个名字相同的bean, Bean1的实例为User1, Bean2的实例是User2
    * 如果使用@Autowired注解根据类型实现依赖注入,因为IOC容器只有Bean1的实例User1,所以启动的时候会提示找不到Bean2的实例User2
    * 如果使用@Resource注解根据名词实现依赖注入,在IOC容器里面得到的实例对象是User1,于是Spring把User1这个实例赋值给User2,就会提示类型不匹配错误

### 配置属性
 <img src= "/assets/files/SpringBean配置属性.jpg" alt="加载错误" title="SpringBean配置属性"/>

### @Resource和@Autowired的区别
1. 内部定义的参数不同
* @Autowired只包含一个required参数,默认为true, 表示开启自动注入
* @Resource包含七个参数, 其中最重要的两个是name和type
2. 装配方式的默认值不同
* @Autowired默认按type自动装配
* @Resource默认按name自动装配
3. 注解应用的范围不同
* @Autowired能够用在构造方法、成员变量、方法参数以及注解上
* @Resource能用在类、成员变量和方法参数上, 这点从源码也能看得出来
4. 出处不同
* @Autowired是Spring定义的注解,只能用于Spring框架下
* @Resource是遵循JSR-250规范,定义在JDK中, 可以与其他框架一起用
5. 装载顺序不同
* @Autowired默认先按byType进行匹配, 如果发现找到多个bean, 则又按照byName方式进行匹配, 如果还有多个则报出异常
* @Resource的装载顺序分为四种情况
    * 如果同时指定了name和type, 则从Spring上下文中找到唯一匹配的bean进行装配, 找不到则抛出异常
    * 如果指定了name, 则从上下文中查找名称匹配的bean进行装配, 找不到则抛出异常
    * 如果指定了type, 则从上下文中找到类型匹配的唯一bean进行装配, 找不到或者找到多个,都会抛出异常
    * 如果既没有指定name和type, 则自动按byName方式进行装配. 如果没有匹配则回退为按照类型进行匹配

### Spring如何解决循环依赖
#### 循环依赖
一个或多个Bean实例之间存在直接或间接的依赖关系,构成循环调用
1. 互相依赖: A依赖B,B依赖A
2. 间接依赖: A依赖B, B依赖C, C依赖A
3. 自我依赖

#### 解决办法
使用三级缓存, 用来存放不同类型的Bean. 核心思想就是把Bean的实例化和Bean里面的依赖注入进行分离. 采用一级缓存存储完整的Bean实例(成熟), 采用二级缓存来存储不完整的Bean实例(早期), 通过不完整的Bean实例作为突破口,解决循环依赖的问题.至于第三级缓存,主要是解决代理对象的循环依赖问题
1. 第一级缓存, 存放完全初始化好的Bean, 这个Bean可以直接使用
2. 第二级缓存, 存放原始的Bean对象, Bean里面的属性还没有进行赋值
3. 第三级缓存, 存放Bean工厂对象, 用来生成原始Bean对象并放入到二级缓存中

假设BeanA和BeanB存在循环依赖,那么在三级缓存的设计下:
 <img src= "/assets/files/Spring循环依赖解决.jpg" alt="加载错误" title="Spring循环依赖解决"/>

1. 初始化BeanA, 先把BeanA实例化, 然后把BeanA包装成ObjectFactory对象保存到第三级缓存中
2. 接着BeanA开始对属性BeanB进行依赖注入, 于是开始初始化BeanB(创建BeanB实例,并加入到第三级缓存)
3. BeanB也开始进行依赖注入, 在第三级缓存中找到了BeanA, 于是完成BeanA的依赖
4. 注入BeanB初始化成功以后保存到一级缓存, 于是BeanA可以成功拿到BeanB的实例,从而完成正常的依赖注入

#### 注意
Spring本身只能解决单实例存在的循环引用问题
1. 多例Bean通过setter注入的情况, 不能解决循环依赖问题
2. 构造器注入的Bean的情况, 不能解决循环依赖问题
3. 单例的代理Bean通过Setter注入的情况, 不能解决循环依赖问题
4. 设置了@DependsOn的Bean的情况, 不能解决循环依赖问题

### Spring中BeanFactory和FactoryBean的区别
Spring里面的核心功能是IOC容器, 其本质上是一个Bean的容器或是一个Bean的工厂. 它保存了所有需要对外提供的Bean的实例. 它能够根据xml里面声明的Bean配置进行bean的加载和初始化, 然后BeanFactory来生产我们需要的各种各样的Bean
#### BeanFactory
1. BeanFactory是所有Spring Bean容器的顶级接口, 它为Spring的容器定义了一套规范, 并提供像getBean这样的方法从容器中获取指定的Bean实例
2. BeanFactory在产生Bean的同时, 还提供了解决Bean之间的依赖注入的能力,也就是DI

#### FactoryBean
1. FactoryBean是一个工厂Bean, 它是一个接口, 主要的功能是动态生成某一个类型的Bean的实例. 可以自定义一个Bean并且加载到IOC容器里面
2. 和新方法getObject(), 这个方法里面就是用来实现动态构建Bean的过程
3. Spring中创建的AOP动态代理Bean一级Spring Cloud里面的OpenFeign组件等就是使用了FactoryBean来实现的

### Spring中Bean的作用域
1. singleton: 意味着在整个Spring容器中只会存在一个Bean实例
2. prototype: 意味着每次从IOC容器去获取指定Bean的时候,都会返回一个新的实例对象
3. 基于Spring框架下的Web应用里面,增加了一个会话纬度来控制Bean的生命周期
* request: 针对每一次http请求,都会创建一个新的Bean. 仅在当前Request中有效(请求——响应结果)
* session: 同一个session共享同一个Bean实例(首次访问至浏览器关闭)
* globalSession: 针对全局session纬度,共享同一个Bean实例

### Spring中Bean的线程安全问题
多例Bean每次都会创建新实例, 线程之间不存在Bean的共享问题, 而单例Bean因为是全局共享, 存在线程安全问题. 单例Bean又分为有状态和无状态两种
1. 无状态Bean: 在多线程操作中只会对Bean的成员变量进行查询操作,不会修改成员变量的值, 因此不存在线程安全问题
2. 有状态Bean: 在多线程操作中如果需要对Bean中的成员变量进行数据更新操作, 这时就存在线程安全问题

#### 如何处理线程安全问题
1. 将Bean的作用域由单例改为多例
2. 在类中定义ThreadLocal的成员变量, 并将需要的可变成员变量保存在ThreadLocal中

### Spring中Bean的生命周期
 <img src= "/assets/files/SpringBean生命周期1.jpg" alt="加载错误" title="SpringBean生命周期1"/>

* 创建前准备阶段
这个阶段主要是Bean在开始加载之前, 需要从上下文和相关配置中解析并查找Bean有关的扩展实现. 比如`init-method`在初始化bean时调用的方法、`destory-method`在销毁bean时调用的方法、`BeanFactoryPostProcessor`这类的bean加载过程中的前置和后置处理

 <img src= "/assets/files/SpringBean生命周期2.jpg" alt="加载错误" title="SpringBean生命周期2"/>

* 创建实例阶段
这个阶段主要是通过反射来创建Bean的实例对象, 并且扫描和解析Bean声明的一些属性

 <img src= "/assets/files/SpringBean生命周期3.jpg" alt="加载错误" title="SpringBean生命周期3"/>

* 依赖注入阶段
如果被实例化的Bean存在依赖其他Bean对象的情况,则需要对这些依赖bean进行对象注入. 比如`@Autowired``@setter`等依赖注入的配置. 在这个阶段会触发一些扩展的调用, 比如`BeanPostProcessors`
(用来实现bean初始化前后的扩展回调)、`InitializingBean`中的afterPropertiesSet()给属性赋值)

 <img src= "/assets/files/SpringBean生命周期4.jpg" alt="加载错误" title="SpringBean生命周期4"/>

* 容器缓存阶段
这个阶段主要是把Bean保存到IoC容器中缓存起来, 这时Bean就可以被开发者使用了.`init-method`会在这个阶段调用

 <img src= "/assets/files/SpringBean生命周期5.jpg" alt="加载错误" title="SpringBean生命周期5"/>

* 销毁实例阶段
当Spring应用上下文关闭时, 该上下文中的所有bean都会被销毁. 如果存在Bean实现了`DisposableBean`接口或配置了`destory-method`属性,会在这个阶段被调用

 <img src= "/assets/files/SpringBean生命周期.jpg" alt="加载错误" title="SpringBean生命周期"/>

### Spring将Bean注入IOC容器的方式
1. 使用xml的方式来声明Bean的定义, Spring容器在启动的时候会加载并解析这个xml, 把bean装载到IOC容器中
2. 使用@CompontScan注解来扫描声明了@Controller、@Service、@Repository、@Component注解的类
3. 使用@Configuration注解声明配置类, 并使用@Bean注解实现Bean的定义(xml配置方式的一种演变,是Spring迈入到无配置化时代的里程碑)
4. 使用@Import注解,导入配置类或者普通的Bean
5. 使用FactoryBean工厂bean, 动态构建一个Bean实例(Spring Cloud OpenFeign里面的动态代理实例就是使用FactoryBean来实现的)
6. 实现ImportBeanDefinitionRegistrar接口, 可以动态注入Bean实例(SpringBoot的启动注解有用到)
7. 实现ImportSelector接口, 动态批量注入配置类或者Bean对象(Spring Boot的自动装配机制有用到)

## Spring Boot核心能力
1. 可以独立运行Spring项目
* Spring Boot可以以jar包的形式进行独立的运行. 使用: java -jar xx.jar就可以成功的运行项目
2. 内嵌的Servlet容器
* 内嵌容器使得我们可以执行运行项目的主程序main函数,实现项目的快速运行
3. 提供starter简化Manen依赖
* 一系列的starter pom用来简化Maven依赖
4. 自动配置Spring
* 根据项目中类路径的jar包/类, 为jar包的类进行自动配置Bean
5. 无代码生成、无XML配置
* 不是借助于代码生成来实现的,而是通过条件注解的方式来实现的

### 如何理解Spring Boot中的Starter
Starter是Spring Boot的四大核心功能特性之一. 它还有自动装配、Actuator监控等特性. 这些特性,都是为了让开发者在开发基于Spring生态下的企业级应用时,只需要关心业务逻辑,减少对配置和外部环境的依赖
### Starter作用
Starter是启动依赖
1. Starter组件以功能为纬度,来维护对应的jar包的版本依赖,使得开发者可以不需要去关心这些版本冲突这种容易出错的细节
2. Starter组件会把对应功能的所有jar包依赖全部导入进来,避免了开发者自己去引入依赖带来的麻烦
3. Starter内部集成了自动装配的机制,也就说在程序中依赖对应的starter组件以后,这个组件自动会集成到Spring生态下,并且对于相关Bean的管理,是基于自动装配机制来完成
4. 依赖Starter组件后,这个组件对应的功能所需要维护的外部化配置,会自动集成到Spring Boot里面. 开发者只需要在yml或properties文件里面进行维护即可

### Spring Boot自动装配机制
自动把第三方组件的Bean装载到Spring IOC器里面,不需要开发人员再去写Bean的配置. 在Spring Boot应用里面,只需要在启动类加上`@SpringBootApplication`(复合注解,其中实际是`@EnableAutoConfiguration`实现自动装配)就可以实现自动装配.
1. 引入Starter启动依赖组件的时候, 这个组件里面必须要包含`@Configuration`配置类, 在这个配置类里面通过`@Bean`注解声明需要装配到IOC容器的Bean对象
2. 然后通过SpringBoot中的约定优于配置思想,把这个配置类的全路径放在`classpath:/META-INF/spring.factories`文件中,SpringBoot就可以知道第三方jar包里面的配置类的位置, 这个步骤主要是用到了Spring里面的`SpringFactoriesLoader`来完成的
3. SpringBoot拿到所第三方jar包里面声明的配置类以后, 再通过Spring提供的`ImportSelector`接口实现对这些配置类的动态加载

### Spring Boot的约定优于配置
一种软件设计的范式, 核心思想是减少开发人员对于配置项的维护, 从而让开发人员更加聚焦在业务逻辑上. Spring Boot就是该理念的实现, 类似于Spring框架下的一个脚手架, 通过Spring Boot, 快速开发基于Spring生态下的应用程序. 开发人员不必在关心依赖管理, tomcat容器等配置, Spring Boot Starter启动依赖能帮我们管理所有jar包版本; Spring Boot自动内置Tomcat容器来运行web应用, 不需要再去单独部署tomcat

## springMVC
### MVC(Model View Controller) 模型、视图、控制器
MVC是一种软件架构模式, 核心思想是将一个软件可以划分成模型, 视图和控制器三种不同类型的模块, 其中, 模型负责封装业务逻辑的处理, 视图负责提供界面(包括数据展现和用户操作界面), 控制器负责协调模型和视图(视图将请求先发送给控制器, 由控制器选择对应的模型来处理；模型将处理结果交给控制器, 由控制器选择对应的视图来展现数据)
### 使用MVC
 <img src= "/assets/files/MVC.jpg" alt="加载错误" title="MVC"/>

### MVC优点
1. 方便测试(如直接将业务逻辑写在servlet类中, 需要部署才能测试, 而写在java类中可以直接测试)
2. 方便维护: 修改视图不影响模型, 反之亦然
3. 方便分工协作

### SpringMVC
SpringMVC是用来简化基于MVC架构的web应用程序的开发,是Spring框架的一部分. 
1. 在Servlet基础上构建并且使用MVC模式设计的一个Web框架,目的是简化传统Servlet+JSP模式下的Web开发方式
2. 对传统MVC架构模式做了增强和扩展
* 把传统MVC框架里面的Controller控制器做了拆分, 分成了前端控制器DispatcherServlet和后端控制器Controller
* 把Model模型拆分成业务层Service和数据访问层Repository
* 在视图层, 可以支持不同的视图, 如Freemark、velocity、JSP等

### 流程
浏览器的请求首先会经过SpringMVC里面的核心控制器DispatcherServlet, 它负责对请求进行分发到对应的Controller, Controller里面处理完业务逻辑之后.返回ModeAndView, 然后DispatcherServlet寻找一个或多个ViewResolver视图解析器, 找到ModeAndView指定的视图, 并把数据显示到客户端
1. 配置阶段, 主要是完成对xml配置和注解配置
* 从web.xml开始, 配置DispatcherServlet的url匹配规则和Spring主配置文件的加载路径
* 配置注解, 比如@Controller、@Service、@Autowrited、@RequestMapping等
2. 初始化阶段, 主要是加载并解析配置信息以及IoC容器、DI操作和HandlerMapping的初始化
* Web容器启动以后,自动调用DispatcherServlet的init()方法
* 在init()方法, 会初始化IoC容器
* 根据配置好的扫描包路径,扫描出相关的类,并利用反射进行实例化,存放到IoC容器中
* 缓存之后, Spring将再次迭代扫描IoC容器中的实例, 给需要自动赋值的属性自动赋值
* 读取@RequestMapping注解, 获得请求url, 建立url和method的映射关系并缓存起来
3. 运行阶段, 在Spring启动以后, 等待用户请求, 完成内部调度并返回响应结果
* 用户在浏览器输入url, Web容器会接收到用户请求. Web容器会自动调用doGet()或者doPost()方法,从doGet()或doPost()方法中可以获得request和response. 通过request可以获得用户请求带过来的信息,通过response可以往浏览器端输出响应结果
* 根据request中获得的请求url,可以从HandlerMapping中找到对应Method
* 利用反射调用方法, 获得方法调用的返回结果
* 将返回结果通过response输出到浏览器, 用户就可以看到响应结果

#### 五大组件
1. DispatcherServlet    前端控制器
2. HandlerMapping   映射处理器
3. Controller   处理器
4. ModelAndView 用于封装处理结果
5. ViewResolver 视图解析器
 <img src= "/assets/files/SpringMVC.jpg" alt="加载错误" title="SpringMVC"/> 

#### 编程步骤
1. 导包: spring-webmvc
2. 添加spring配置文件
3. 配置DispatcherServlet   
4. Controller类
5. jsp
6. 配置HandlerMapping和ViewResolver
 <img src= "/assets/files/SpringMVC_process.jpg" alt="加载错误" title="SpringMVC处理过程"/> 

#### 基于注解的springMVC应用
1. 导包: spring-webmvc
2. 添加spring配置文件
3. 配置DispatcherServlet   
4. Controller类
5. jsp
6. 配置文件中添加组件扫描, mvc注解扫描, 视图解析器

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
    * 表示层(UI层):    数据展现和操作界面以及请求分发
    * 业务层(服务层): 封装业务处理逻辑
    * 持久层(数据访问层): 负责处理数据访问逻辑
*注: a.上一层通过调用接口调用下一层的服务, 当下一层的实现发生改变, 不影响上一层*

#### 处理表单中文参数值乱码问题
可以配置springMVC提供的过滤器, 表单提交方式必须设置为post, 客户端的编码必须与过滤器的编码一致

#### 拦截器
1. DispatcherServlet在收到请求之后, 如果有拦截器, 会先调用拦截器,再调用处理器(Controller)
    * 过滤器属于Servlet规范, 而拦截器属于Spring框架
2. 编程步骤: 
    * 写一个java类实现HanlerInterceptor接口
    * 在拦截器接口方法中实现拦截处理逻辑
    * 配置拦截器
    * 如果多个拦截器都满足要求, 则依据配置的先后顺序执行
 <img src= "/assets/files/Interceptor.jpg" alt="加载错误" title="拦截器"/>

#### 异常处理
1. 配置简单异常处理器, 只适合对异常进行简单处理
    * 在配置文件中配置SimpleMappingExceptionResolver 
    * 添加相应的异常处理页面
2. 若要对异常做复杂处理, 则使用@ExceptionHandler
    * 在处理类中添加一个异常处理方法
    * 添加相应异常处理页面

#### SpringJDBC
Spring对jdbc的简单封装
1. 编程步骤
    * 导包, spring-webmvc, spring-jdbc, ojdbc,dbcp,junit
    * 添加spring配置文件
    * 配置JdbcTemplate模板
    * 调用JdbcTemplate提供的方法来访问数据库；通常将JdbcTemplate注入到DAO

## spring事务与分布式事务
1. spring事务本质上对数据库事务管理的封装. 通过声明式事务配置, 使得开发人员从一些复杂的事务处理中得到解脱,不再需要关心连接的获取、连接的关闭、事务提交、事务回滚这些操作,更加聚焦在业务开发. 这种事务管理主要是针对单数据库操作
2. 分布式事务是解决多数据的实务操作的数据一致性, 关系型数据库不支持跨库事务的操作, 所以需要引入分布式事务的解决方案