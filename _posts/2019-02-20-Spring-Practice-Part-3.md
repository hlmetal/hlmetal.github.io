---
layout: post
title:  "Spring初识之Spring Boot"
date:   2019-02-20 09:00:30 +0200
categories: java spring web
---

Spring Boot基本知识、自动配置等各种机制, 以及运行中的Spring Boot的各种配置内容

## 重新认识[Spring Boot](https://www.tutorialspoint.com/spring_boot/spring_boot_introduction.htm)
Spring Boot是帮助开发者尽快能快的构建出一个可以运行的应用程序, 不需要太多配置就能运行, 提供了很多面向生产环境的非功能性需求。
* Spring Boot不是应用服务器, 如Tomcat等, 在SpringBoot应用程序中可以包含这些服务器，或者将应用程序打成war包，部署在服务器上
* Spring Boot不是JavaEE等规范, 它可以构建出符合JaveEE等规范的应用程序
* Spring Boot不是代码生成器
* Spring Boot不是Spring Framework的升级版, 但可以帮助开发者更好地使用Spring Framework

### Spring Boot的特性
1. 方便地创建可独立运行的spring应用程序
2. 直接内嵌Tomcat、Jetty或Undertow
3. 简化了项目构建配置(maven)
4. 为spring及第三方库提供自动配置
5. 提供生产级特性
6. 无需生成代码或进行XML配置

### Spring Boot的四大核心
1. 自动配置-Auto Configuration
2. 起步依赖-Starter Dependency
3. 命令行界面-Spring Boot CLI
4. Actuator

### 自动配置
基于添加的jar依赖自动对SpringBoot应用程序进行配置, 自动配置的代码都在Spring-boot-autoconfiguration中
1. 开启自动配置
* `@EnableAutoConfiguration`
* `@SpringBootApplication`中包含了`@EnableAutoConfiguration`
* 使用`exclude=Class<?>[]`参数可以排除特定的自动配置
2. @EnableAutoConfiguration
* AutoConfigurationImportSelector会加载spring.factories里面的特定属性autoconfigure.EnableAutoCOnfiguration
* 条件注解:
    * @Conditonal
    * @ConditionalOnClass-存在特定类时
    * @ConditionalOnMissingClass-不存在特定类时
    * @COnditionalOnBean-存在特定bean时
    * @ConditionalOnMissingBeam-不存在特定bean时
    * @COnditionalOnSingleCandidate-上下文只有一个候选bean时
    * @CondtionalOnProperty-属性等于特定值时
    * @CondtionalOnResource
    * @CondtionalOnWebApplication
    * @CondtionalOnNotWebAppliction
    * @CondtionalOnExpression
    * @ConditionalOnJava
    * @ConditionalOnJndi
3. 观察自动配置的结果
* 使用`--debug`命令启动ConditionEvaluationReportLoggingListener输出结果
* Positive matches-匹配上的
* Negative matches-没有匹配的
* Exclusions-排除掉的
* Unconditional classes-无条件配置的

#### 实现自定义自动配置
1. 编写Java Config,  在配置类添加`@Configuration`注解
2. 添加条件`@Conditional`
3. 指定自动配置的执行顺序`@AutoConfigureBerfore``@AutoConfigureAfter``@AutoConfigureOrder`
4. 定位自动配置`META-INF/spring.factories

```java
@Configuration
@ConditionalOnClass(GreetingApplicationRunner.class)
public class GreetingAutoConfiguration {
    @Bean
    @ConditionalOnMissingBean(GreetingApplicationRunner.class)
    @ConditionalOnProperty(name = "greeting.enabled", havingValue = "true", matchIfMissing = true)
    public GreetingApplicationRunner greetingApplicationRunner() {
        return new GreetingApplicationRunner();
    }
}

// spring.factories
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
com.spring.hualan.hualanspring.GreetingAutoConfiguration
```

#### 如何在低版本spring中实现自动配置
3.x的spring没有条件注解，且无法自动定位需要加载的自动配置
1. 核心思路
* 使用spring提供的BeanFactoryPostProcessor进行条件判断
* 配置加载: 编写config类, 引入配置类-通过`@component-scan`或xml文件import
2. spring提供的两个扩展点
* BeanPostProcessor-针对Bean实例, 在Bean创建后提供定制逻辑回调
* BeanFactoryPostProcessor-针对Bean定义, 在容器创建Bean前获取配置元数据
3. 关于Bean的一些定制
* Lifecycle Callback - InitializingBean\@PostConstruct\init-method; DisposableBean\@PreDestroy\destroy-method
* XxxAware接口 - ApplicationContextAware\BeanFactoryAware\BeanNameAware
4. 一些常用操作
* ClassUtils.isPresent()-判断类是否存在
* ListableBeanFactory.containBeanDefinition()-判断Bean是否已定义
* ListableBeanFactory.getBeanNamesForType()-判断Bean是否已定义
* BeanDefinitionRegistry.registerBeanDefinition()-注册Bean定义
* BeanFactory.registerSingleton()-注册Bean定义

```java
@Configuration
public class GreetingAutoConfiguration {
    @Bean
    public static GreetingBeanFactoryPostProcessor greetingBeanFactoryPostProcessor() {
        return new GreetingBeanFactoryPostProcessor();
    }
}

@Slf4j
public class GreetingBeanFactoryPostProcessor implements BeanFactoryPostProcessor {
    public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException {
        boolean hasClass = ClassUtils.isPresent("com.spring.hualan.hualanspring.GreetingApplicationRunner",
                GreetingBeanFactoryPostProcessor.class.getClassLoader());
        if (!hasClass) {
            log.info("GreetingApplicationRunner is NOT present in CLASSPATH.");
            return;
        }

        if (beanFactory.containsBeanDefinition("greetingApplicationRunner")) {
            log.info("We already have a greetingApplicationRunner bean registered.");
            return;
        }

        register(beanFactory);
    }

    private void register(ConfigurableListableBeanFactory beanFactory) {
        if (beanFactory instanceof BeanDefinitionRegistry) {
            GenericBeanDefinition beanDefinition = new GenericBeanDefinition();
            beanDefinition.setBeanClass(GreetingApplicationRunner.class);

            ((BeanDefinitionRegistry) beanFactory)
                    .registerBeanDefinition("greetingApplicationRunner",
                            beanDefinition);
        } else {
            beanFactory.registerSingleton("greetingApplicationRunner",
                    new GreetingApplicationRunner());
        }
    }
}
```

### [Starter Dependency](https://docs.spring.io/spring-boot/docs/3.0.5/reference/htmlsingle/#using.build-systems.starters)
直接面向功能，一站获得所有相关依赖，不再复制粘贴
#### Maven依赖管理
1. 查看依赖树：`mvn dependency:tree` 或插件IDEA Maven Helper
2. 排除特定依赖: `exclusion`
3. 统一管理依赖: `dependencyManagement`

#### 自定义Starter Dependency
1. 主要内容
* autoconfigure模块，包含自动配置代码(可选)
* starter模块，包含指向自动配置模块的依赖及其相关依赖
2. 命名方式
* xxx-spring-boot-autoconfigure
* xxx-spring-boot-starter
3. 注意事项
* 不要使用spring-boot作为依赖前缀
* 不要使用spring-boot的配置命名空间
* starter中仅添加必要依赖
* 声明对spring-boot-starter的依赖(可选)

### Spring Boot配置加载机制
#### 外化配置加载顺序
1. 开去DevTools时, ~/.spring-boot-devtools.properties
2. 测试类上的TestPropertySource注解
3. @SpringBootTest#properties属性
4. 命令行参数(--server.port=9000)
5. SPRING_APPLICATION_JSON中的属性
6. ServletConfig初始化参数
7. ServletContext初始化参数
8. java:comp/env中的JNDI属性
9. System.getPorperties()
10. 操作系统环境变量
11. random.*涉及到的RandomValuePropertySource
12. jar包外部的application-{properties}.properties或.yml
13. jar包内部的application-{properties}.properties或.yml
14. jar包外部的application.properties或.yml
15. jar包内部的application.properties或.yml
16. @Configuration类上的@PropertySource
17. SpringApplication.setDefaultPorperties()设置的默认属性

#### application.properties
1. 默认位置
* ./config
* ./
* CLASSPATH中的/config
* CLASSPATH中的/
2. 修改名字或路径
* spring.config.name
* spring.config.location
* spring.config.additional-location
3. Relaxed Binding支持的命名风格
* 短划线分隔(适用于properties/xml/系统属性)
* 驼峰式(适用于properties/xml/系统属性)
* 下划线分隔(适用于properties/xml/系统属性)
* 全大写，下划线分隔(用于环境变量)

#### PropertySource
1. 添加PropertySource
* `<context:property-placeholder>`
* `PropertySourcePlaceholderConfigurer`
* `@PropertySource`
* `@PorpertySources`
2. Spring Boot中的@ConfigurationProperties
* 可以将属性绑定到结构化对象上
* 支持Relaxed Binding
* 支持安全的类型转换
* @EnableConfigurationProperties
3. 定制PorpertySource
* 实现PropertySource类, 从Environment取得PropertySources, 将自定义的PorpertySource添加到合适的位置
* 切入位置: EnvironmentPostProcessor、BeanFactoryPostProcessor

## 运行中的SpringBoot
### Actuator Endpoint
Acuator的目的是监控并管理应用程序
#### 依赖: spring-boot-starter-actuator
#### 访问方式
1. HTTP访问`host:port/actuator/<id>`
2. JMX访问(如JConsole)

#### 配置项
1. 端口与路径
* management.server.address=
* management.server.port=
* management.endpoints.web.base-path=/actuator
* management.endpoints.web.path-mapping.&lt;id&gt;=路径
2. 开启或关闭Endpoint
*  managment.endpoint.&lt;id&gt;.enabled=true/false
* managment.endpoints.enabled-by-default=false
3. 暴露Endpoint
* management.endpoints.jmx.exposure.exclude=
* management.endpoints.jmx.exposure.include=
* management.endpoints.web.exposure.exclude=
* management.endpoints.web.exposure.include=

#### 一些常用的Endpoint
其中health和info默认HTTP访问,其他默认JMX访问, shutdowm默认关闭
* beans——显示容器中的bean列表
* caches——显示应用中的缓存
* conditions——显示配置条件的计算情况
* configprops——显示@ConfigurationProperties的信息
* env——显示ConfigurableEnvironment中的属性
* health——显示健康检查信息
* httprace——显示Http Trace信息
* info——显示设置好的应用信息
* loggers——显示并更新日志配置
* metrics——显示应用的度量信息
* mappings——显示所有@RequestMapping信息
* scheduledtasks——显示应用的调度任务信息
* shutdown——关闭应用程序
* threaddump——执行Thread Dump
* heapdump——返回Head Dump文件
* prometheus——返回可供Prometheus抓取的信息

### Spring Boot自带的Health Indicator
Health Indicator的目的是检查应用程序的运行状态
1. 状态
* DOWN-503
* OUT_OF_SERVICE-503
* UP-200
* UNKNOWN-200
2. 机制
* 通过HealthIndicatorRegistry收集信息
* HealthIndicator实现具体的检查逻辑
3. 配置项
* management.health.defaults.enabled=true/false
* management.health.&lt;id&gt;.enabled=true
* management.endpoint.health.show-details=never/when-authorized/always
4. 内置Health Indicator
* ElasticsearchHealthIndicator
* MongoHealthIndicator
* RedisHealthIndicator
* RabbitHealthIndicator
* DataSourceHealthIndicator
* MailHealthIndicator
* ......

### 自定义Health Indicator
1. 方法
* 实现HealthIndicator接口
* 根据自定义检查逻辑返回相应的Health状态

```java
//配置
management.endpoints.web.exposure.include=*
management.endpoint.health.show-details=always

//代码
@Component
public class UserIndicator implements HealthIndicator {
    @Autowired
    private UserService userService;
    @Override
    public Health health() {
        User user = userService.selectByPrimaryKey(1L);
        Health health;
        if (user != null) {
            health = Health.up()
                    .withDetail("message", "this user is exist")
                    .withDetail("user", user)
                    .build();
        } else {
            health = Health.down()
                    .withDetail("user", "could not found this user")
                    .build();
        }
        return health;
    }
}

//访问
http://localhost:8080/actuator/health
```

### 通过Micrometer收集度量指标
#### Micrometer
1. 特性
* 多维度度量, 支持Tag
* 预置大量探针: 缓存、类加载器、GC、CPU利用率、线程池等
* 与Spring深度结合
* 支持多种监控系统: Dimensional(AppOptics,Atlas,Azure Monitor, Influx...)/Hierarchical(Graphite,Ganglia,JMX,Etsy StatsD)
2. 核心接口
Meter
3. 内置实现
* Gauge, TimeGauge
* Timer, LongTaskTimer, FunctionTimer
* Counter, FunctionCounter
* DistributionSummary

#### Spring Boot中的[Micrometer](https://docs.spring.io/spring-boot/docs/3.0.5/reference/htmlsingle/#actuator.metrics)
1. 一些URL
* /acutator/metrics
* /acutator/prometheus
2. 一些配置项
```java
management.metrics.export.*
management.metrics.tags.*
management.metrics.enabled.*
management.metrics.distribution.*
management.metrics.web.server.auto-time-requests
```
3. 核心度量项
* JVM
* CPU
* 文件句柄数
* 日志
* 启动时间
4. 其他度量项
* Spring MVC、Spring WebFlux
* Tomcat、Jersey JAX-RS
* RestTemplate、WebClient
* 缓存、数据源
* Kafka、RabbtiMQ

#### 自定义度量指标
1. 通过MeterRegistry注册Meter
2. 提供MeterBuilder Bean让Spring Boot自动绑定
3. 通过MeterFilter进行定制

#### Spring Boot Admin
为Spring Boot应用程序提供一套管理界面
1. 主要功能
* 集中展示应用程序Actuator相关内容
* 变更通知
2. 服务端使用
* 引入依赖`de.codecentric:spring-boot-admin-starter-server:2.1.3`
* 开启注解`@EnableAdminServer`
3. 客户端使用
* 引入依赖`de.codecentric:spring-boot-admin-starter-client:2.1.3`
* 配置服务端及Endpoint
    * `spring.boot.admin.client.url=http://localhost:8080`
    * `management.endpoints.web.exposure.include=*`
4. 安全控制
* 引入依赖`spring-boot-starter-security`
* 服务端配置: 
    * `spring.security.user.name`
    * `spring.security.user.password`
* 客户端配置: 
    * `spring.boot.admin.client.username`
    * `pring.boot.admin.client.password`
    * `spring.boot.admin.client.instance.metadata.user.name`
    * `spring.boot.admin.client.instance.metadata.user.password`

### 定制Web容器的运行参数
#### 内嵌Web容器
* tomcat
* jetty
* undertow
* reactor-netty


#### 端口
* `server.prot`
* `server.address`

#### 压缩配置

```java
//开启压缩
server.compression.enabled
//压缩最小大小,默认2k
server.compression.min-response-size
//压缩类型
server.compression.mime-types
```

#### Tomcat特定配置

```java
server.tomcat.max-connections=1000
server.tomcat.max-http-post-size=2MB
server.tomcat.max-swallow-size=2MB
server.tomcat.max-threads=200
server.tomcat.min-spare-threads=10
```

#### 错误处理

```java
server.error.path=/error
server.error.include-exception=false
server.error.include-stacktrace=never
server.error.whitelable.enabled=true
```

#### 其他

```java
server.use-forward-headers
server.servlet.session.timeout
```


#### 编程方式修改配置
实现`WebServerFactoryCustomizer<T>`接口

```java
@Override
	public void customize(TomcatServletWebServerFactory factory) {
		Compression compression = new Compression();
		compression.setEnabled(true);
		compression.setMinResponseSize(DataSize.ofBytes(512));
		factory.setCompression(compression);
	}
```

#### 配置HTTPS支持
##### 服务端配置
1. 通过参数进行配置
```java
server.port=8443
server.ssl.key-store= classpath:hualanspring.p12
server.ssl.key-store-type=PKCS12
server.ssl.key-store-password=hualanspring
```
2. 生成证书文件
* keytool -genkey -alias 别名
* keytool -storetype 仓库类型(JKS/JCEKS/PKCS12) -keyalg 算法(RSA/DSA) -keysize 长度
* keytool -keystore 文件名 -validity 有效期

##### 客户端配置
1. 配置HttpClient
* SSLContextBuilder构造SSLContext
* setSSLHostnameVerifier(new NoopHostnameVerifier())
2. 配置RequestFactory
* HttpComponentsClientHttpRequestFactory.setHttpClient()

```java
hualanspring.service.url=https://localhost:8443
security.key-store=classpath:hualanspring.p12
security.key-pass=spring

@Value("${security.key-store}")
private Resource keyStore;
@Value("${security.key-pass}")
private String keyPass;

SSLContext sslContext = null;
try {
    sslContext = SSLContextBuilder.create()
            // 会校验证书
            .loadTrustMaterial(keyStore.getURL(), keyPass.toCharArray())
            // 放过所有证书校验
//			  .loadTrustMaterial(null, (certificate, authType) -> true)
            .build();
} catch(Exception e) {
    log.error("Exception occurred while creating SSLContext.", e);
}

CloseableHttpClient httpClient = HttpClients.custom()
        .evictIdleConnections(30, TimeUnit.SECONDS)
        .setMaxConnTotal(200)
        .setMaxConnPerRoute(20)
        .disableAutomaticRetries()
        .setKeepAliveStrategy(new CustomConnectionKeepAliveStrategy())
        .setSSLContext(sslContext)
        .setSSLHostnameVerifier(NoopHostnameVerifier.INSTANCE)
        .build();

HttpComponentsClientHttpRequestFactory requestFactory =
        new HttpComponentsClientHttpRequestFactory(httpClient);

return requestFactory;
```

#### 配置HTTP2支持
##### 前提条件
1. Java >=JDK9
2. Tomcat >=9.0.0
3. Spring Boot不支持h2c, 需要先配置SSL

##### 服务端配置项
1. server.http2.enabled

##### 客户端配置
1. HTTP库选择: OkHttp
2. RestTemplate配置: OkHttp3ClientHttpRequestFactory

```java
@Value("${security.key-store}")
private Resource keyStore;
@Value("${security.key-pass}")
private String keyPass;

@Bean
public ClientHttpRequestFactory requestFactory() {
    OkHttpClient okHttpClient = null;
    try {
        KeyStore keyStore = KeyStore.getInstance(KeyStore.getDefaultType());
        keyStore.load(this.keyStore.getInputStream(), keyPass.toCharArray());
        TrustManagerFactory tmf = TrustManagerFactory.getInstance(TrustManagerFactory.getDefaultAlgorithm());
        tmf.init(keyStore);
        SSLContext sslContext = SSLContext.getInstance("TLS");
        sslContext.init(null, tmf.getTrustManagers(), null);

        okHttpClient = new OkHttpClient.Builder()
                .sslSocketFactory(sslContext.getSocketFactory(), (X509TrustManager) tmf.getTrustManagers()[0])
                .hostnameVerifier((hostname, session) -> true)
                .build();
    } catch (Exception e) {
        log.error("Exception occurred!", e);
    }
    return new OkHttp3ClientHttpRequestFactory(okHttpClient);
}
```

#### 关闭容器(编写命令行程序)
##### 关闭容器方式
1. 控制依赖: 不添加web相关依赖(若使用RestTemplate等相关web内容, 就不能排除此依赖)
2. 配置方式: spring.main.web-application-type=none
3. 编程方式: 
```java
public static void main(String[] args) {
    //new SpringApplication().setWebApplicationType(WebApplicationType.NONE);
    new SpringApplicationBuilder()
            .sources(CustomerServiceApplication.class)
            .bannerMode(Banner.Mode.OFF)
            //必须在run()方法之前设置
            .web(WebApplicationType.NONE)
            .run(args);
}
```

##### 常用工具类
1. ApplicationRunner(ApplicationArguments)
2. CommandLineRunner(String[])
3. 可以通过注解@Order指定各个Runner的顺序

### 认识可执行Jar
#### Jar包中的内容
1. Jar描述, META-INF/MANIFEST.MF
2. Spring Boot Loader, org/springframework/boot/loader
3. 项目内容, BOOT-INF/classes
4. 项目依赖, BOOT-INF/lib

#### Jar的启动类
MANIFEST.MF中的Main-Class: org.springframework.boot.loader.JarLauncher会找带有@SpringApplication注解的类去执行

#### 可直接执行的jar
打包后的jar可直接运行,无需java命令,在pom文件中配置`<configuration><executable>true</executable></configuration>`即可:
```java
<plugins>
    <plugin>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-maven-plugin</artifactId>
        
    </plugin>
</plugins>
```

### 如何将Spring Boot应用程序打包成Docker镜像
#### 什么是Docker镜像
* 镜像是静态的只读模板
* 镜像中包含构建Docker容器的指令
* 镜像是分层的
* 通过Dockerfile来创建镜像

#### Dockerfile
* FROM  基于哪个镜像
* LABEL 设置标签
* RUN   运行安装命令
* CMD   容器启动时的命令
* ENTRYPOINT    容器启动后的命令
* VOLUME    挂载目录
* EXPOSE    容器要监听的端口
* ENV   设置环境变量
* ADD   添加文件
* WORKDIR   设置运行的工作目录
* USER  设置运行的用户

#### 通过Maven构建Docker镜像
* 提供一个Dockerfile

```java
FROM java:8
EXPOSE 8080
ARG JAR_FILE
ADD target/${JAR_FILE} /hualan-spring.jar
ENTRYPOINT ["java", "-jar","/hualan-spring.jar"]
```

* 配置dockerfile-maven-plugin插件

```java
<plugin>
    <groupId>com.spotify</groupId>
    <artifactId>dockerfile-maven-plugin</artifactId>
    <version>1.4.10</version>
    <executions>
        <execution>
            <id>default</id>
            <goals>
                <goal>build</goal>
                <goal>push</goal>
            </goals>
        </execution>
    </executions>
    <configuration>
        <repository>${project.artifactId}</repository>
        <tag>${project.version}</tag>
        <buildArgs>
            <JAR_FILE>${project.build.finalName}.jar</JAR_FILE>
        </buildArgs>
    </configuration>
</plugin>
```

* 执行mvn package 或者mvn dockerfile:build
* 检查结果 docker images
* 运行`docker run --name hualan-spring -d -p 8080:8080 hualan-spring:0.0.1-SNAPSHOT`



