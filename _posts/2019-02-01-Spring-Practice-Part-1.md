---
layout: post
title:  "Spring初识之数据访问"
date:   2019-02-01 08:00:30 +0200
categories: java spring database
---

Spring入门,及搭建Springboot项目并连接各种数据源，进行数据访问和存储操作(MySQL,NoSQL,Reactor)

## 初识Spring
### Spring家族
#### Spring Framework
用于构建企业级应用的轻量级一站式解决方案
* 保持向后兼容性
* 专注API设计
* 追求严苛的代码质量
* 可选择性

#### Spring Boot
快速构建基于Spring的应用程序
* 非常快
* 开箱即用，按需改动
* 提供各种非功能特性
* 不用生产代码，没有XML配置

#### Spring Cloud
微服务解决方案, 简化分布式系统的开发
* 配置管理(Nacos)
* 服务注册(Eureka)、服务发现、服务追踪
* 负载均衡(Ribbon)
* 熔断(Resilience4j)
* 网关(Zuul)

### 编写Spring程序
#### Spring initializr
使用Spring官方提供的[项目初始化工具](https://start.spring.io/)来生成项目基本骨架
1. Project选择Maven
2. Language选择Java
3. Spring Boot版本根据需要选择(**本文基于v2.2.4.RELEASE**)
4. Project Metadata
* Group: com.example
* Artifact: demo
* Name: demo
* Description: A demo project
* Package name: com.example.demo
* Packaging: jar
* java: 根据需要选择版本
5. Dependencies根据需要引入

## Spring JDBC实践
### 配置单数据源
#### 引入相关依赖
1. 引入数据库驱动——MySQL
2. 引入JDBC依赖

#### 配置数据源
1. 配置文件
```yml
datasource:
    url: jdbc:mysql://localhost:3306/test_1?useUnicode=true&characterEncoding=utf8&serverTimezone=GMT
    username: root
    password: root
    driver-class-name: com.mysql.cj.jdbc.Driver
```

2. Spring Boot自动配置
* DataSourceAutoConfiguration
* DataSourceTransactionManagerAutoConfiguration
* JdbcTemplateAutoConfiguration等
3. 也可手动配置, 需要在SpringBootApplication中排除上述内容, 并新建DataSourceConfig.java

* 数据源
```java
    @Bean
    @ConfigurationProperties(prefix="datasource") //从配置文件读取数据源配置
    public DataSourceProperties dataSourceProperties() {
        return new DataSourceProperties();
    }

    @Bean
    public DataSource DataSource() {
        DataSourceProperties dataSourceProperties = dataSourceProperties();
        log.info("datasource: {}", dataSourceProperties.getUrl());
        return dataSourceProperties.initializeDataSourceBuilder().build();
    }
```

* 事务相关
```java
    @Bean
    @Resource
    public PlatformTransactionManager TxManager(DataSource DataSource) {
        return new DataSourceTransactionManager(DataSource);
    }
```

* 操作相关
```java
   @Bean(name = "sqlSessionFactory")
   public SqlSessionFactory sqlSessionFactory(@Qualifier("dataSource") final DataSource dataSource) throws Exception {

       final SqlSessionFactoryBean sessionFactory = new SqlSessionFactoryBean();
       sessionFactory.setDataSource(dataSource);
       sessionFactory.setConfiguration(mybatisProperties.getConfiguration());
       PathMatchingResourcePatternResolver resolver = new PathMatchingResourcePatternResolver();
       sessionFactory.setMapperLocations(resolver.getResources("classpath:mapper/*.xml"));
       return sessionFactory.getObject();
   }
```

* 其他


### 配置多数据源
不同数据源的配置要分开, 关注每次使用的数据源。配置时可以加@Primary注解将某一数据源作为主要数据源。配置方式如上述单数据源手动配置类似，要注意ConfigurationProperties配置的prefix。

### 连接池
#### HikariCP-A high-performance JDBC connection pool
##### Hikari为什么快([Down the Rabbit Hole](https://github.com/brettwooldridge/HikariCP/wiki/Down-the-Rabbit-Hole))
1. 字节码级别的优化-尽量的利用JIT的内联手段
2. 字节码级别的优化-利用更容易被JVM优化的指令
* `invokevirtual`替换成`invokestatic`，更加容易被JVM优化
* 获取Connection,Statement,ResultSet的代理对象的单例工厂方法替换成了具有static方法的final类(由JavaAssist在编译时动态生成)
3. 代码级别的优化-利用改造后的`FastList`代替ArrayList
4. 代码级别的优化-利用无锁的`ConcurrentBag`

##### Spring Boot 2.x默认使用HikariCP, 需配置`spring.datasource.hikari.*`配置项
##### Spring Boot 1.x默认使用Tomcat连接池, 需排除tomcat-jdbc依赖, 并引入HikariCP依赖, 再配置`spring.datasouce.type=com.zaxxer.hikari.HikariDataSource`

##### HikariCP常用配置
1. mmaximumPoolSize=10
2. minimumIdle=10
3. idleTimeOut=600000
4. connectionTimeOut=300000
5. maxLifetime=1800000

#### Alibaba Druid
Druid为监控而生, 内置强大的监控功能, 监控特性不影响性能, 功能强大, 能防SQL注入, 内置Logging能诊断Hack行为
##### 实用功能
1. 详细的监控(可访问监控页面)
2. ExceptionSorter, 针对主流数据库的返回码都支持
3. SQL防注入
4. 内置加密配置
5. 可扩展, 方便定制整个操作的各个环节(例如: 执行SQL前或后加入其他需求等)

##### [如何使用](https://github.com/alibaba/druid/wiki/%E5%B8%B8%E8%A7%81%E9%97%AE%E9%A2%98)
1. 引入 druid-spring-boot-starter依赖, 并排除HikariCP依赖
2. 配置spring.datasource.*
```yml
    type: com.alibaba.druid.pool.DruidDataSource
    url: jdbc:mysql://localhost:3306/test_1?useUnicode=true&characterEncoding=utf8&serverTimezone=GMT
    username: root
    password: root
    druid:
        driver-class-name: com.mysql.cj.jdbc.Driver
        initial-size: 5
        max-active: 5
        min-idle: 5
        # 获取连接时检查
        test-on-borrow: true
        # 归还连接时检查
        test-on-return: true
        # idle时检查
        test-while-idle: true
```

##### Filter配置
* `druid.filters: stat,config,wall,log4j`

1. 密码加密
```yml
password: <加密密码>
druid.filter.config.enabled: true
druid.connection-properties: config.decrypt=true;config.decrypt.key=${public-key}
```

2. SQL防注入
```yml
druid.filter.wall.enabled: true
druid.filter.wall.db-type: mysql
druid.filter.wall.delete-allow: false
druid.ilter.wall.config.drop-table-allow: false
```
3. 慢SQL日志

* 系统属性配置
    * druid.stat.logSlowSql=true
    * druid.stat.slowSqlMillis=3000
* Spring boot配置
    * spring.datasouce.druid.filter.stat.enable=true
    * spring.datasouce.druid.filter.stat.log-slow-sql=true
    * spring.datasouce.druid.filter.stat.slow-sql-millis=3000

##### 定制连接池操作的各种环节
1. 继承FilterEventAdapter
2. 修改druid-filter.properties增加Filter配置

#### 连接池选择
##### 考量点
1. 可靠性
2. 性能
3. 功能
4. 可运维性
5. 可扩展性
6. 其他

### SpringJDBC访问数据库
#### spring-jdbc包
* core, jdbcTemplate等相关核心接口和类
* datasource数据源相关辅助类
* object 将基本jdbc操作封装成对象
* support 错误等其他辅助工具

#### 通过注解定义Bean
* @Component 通用
* @Repository 与数据库操作相关
* @Service 与业务服务相关
* @Controller 与SpringMVC相关
* @RestController 与Restful Web Service相关

#### JdbcTemplate

* query
```java
 jdbcTemplate.query("SELECT * FROM test_01", new RowMapper<Foo>() {
            @Override
            public Test mapRow(ResultSet rs, int rowNum) throws SQLException {
                return Test.builder()
                        .id(rs.getLong(1))
                        .name(rs.getString(2))
                        .build();
            }
        });
```

* queryForObject
`jdbcTemplate.queryForObject("SELECT COUNT(*) FROM test_01", Long.class);`
* queryForList
* queryForMap
* update
`jdbcTemplate.update("INSERT INTO test_01 (name) VALUES (?)", name);`
* execute

* JdbcTemplate
    * batchUpdate
    * batchPreparedStatementSetter
```java
jdbcTemplate.batchUpdate("INSERT INTO test_01 (name) VALUES (?)",
                new BatchPreparedStatementSetter() {
                    @Override
                    public void setValues(PreparedStatement ps, int i) throws SQLException {
                        ps.setString(1, "jack-" + i);
                    }

                    @Override
                    public int getBatchSize() {
                        return 2;
                    }
                });
```

* NamedParameterJdbcTemplate
    * batchUpdate
    * SqlParamenterSourceUtils.createBatch
```java
    List<Foo> list = new ArrayList<>();
    list.add(Test.builder().id(100L).name("jack-100").build());
    list.add(Test.builder().id(101L).name("jack-101").build());
    namedParameterJdbcTemplate
            .batchUpdate("INSERT INTO test_01 (ID, NAME) VALUES (:id, :name)",
                    SqlParameterSourceUtils.createBatch(list));
```

### Spring的事务抽象
#### 核心接口
PlatformTransactionManager
* 方法
    * void commit
    * void rollback
    * TransactionStatus getTransaction

* 实现
    * DataSourceTransactionManager
    * HibernateTransactionManager
    * JtaTransactionManager
    * ...

TransactionDefinition
* Propagation 传播特性
* Isolation 隔离性
* TimeOut 超时
* Read-only status 只读

#### 事务传播特性
1. int PROPAGATION_REQUIRED = 0;    当前有事务就用当前的，没有就用新的(默认)
2. int PROPAGATION_SUPPORTS = 1;    事务可有可无  
3. int PROPAGATION_MANDATORY = 2;   当前一定要有事务，否则抛异常
4. int PROPAGATION_REQUIRES_NEW = 3;    无论是否有事务，都起一个新事务
5. int PROPAGATION_NOT_SUPPORTED = 4;   不支持事务，按非事务方式运行
6. int PROPAGATION_NEVER = 5;   不支持事务，有则抛异常
7. int PROPAGATION_NESTED = 6; 内嵌事务，当前有事务就在当前事务内再起一个事务，内部事务有自己的状态属性等，其回滚等操作不影响外部事务，但外部事务回滚，内部事务也会回滚

#### 事务隔离特性
1. int ISOLATION_DEFAULT = -1; 
2. int ISOLATION_READ_UNCOMMITTED = 1;  允许脏读不可重复读、幻读
3. int ISOLATION_READ_COMMITTED = 2;    无脏读, 不可重复读、幻读
4. int ISOLATION_REPEATABLE_READ = 4;   无脏读, 可重复读、幻读
5. int ISOLATION_SERIALIZABLE = 8;      无脏读, 可重复读、无幻读

#### 编程式事务
1. TransactionTemplate
* TransactionCallback(有返回值)
* TransactionCallbackWithoutResult(无返回值)
```java
	log.info("COUNT BEFORE TRANSACTION: {}", getCount()); // 0
	transactionTemplate.execute(new TransactionCallbackWithoutResult() {
		@Override
		protected void doInTransactionWithoutResult(TransactionStatus transactionStatus) {
			jdbcTemplate.execute("INSERT INTO test_01 (ID, NAME) VALUES (10, 'nico')");
			log.info("COUNT IN TRANSACTION: {}", getCount()); // 1
			transactionStatus.setRollbackOnly();
		}
	});
	log.info("COUNT AFTER TRANSACTION: {}", getCount()); // 0
```

2. PlatformTransactionManager
* 可以传入TransactionDefinition进行定义

#### 声明式事务(AOP机制)
  <img src= "/assets/files/Spring声明式事务.jpg" alt="加载错误" title="Spring声明式事务"/>

##### 开启事务的方式
1. 基于注解的配置方式
* `@EnableTransactionManagement`

2. 基于xml的配置方式
* `<tx:annotation-driven/>`
    * proxyTargetClass= true/false
    * mode= AdviceMode.ASPECTJ/PROXY
    * order 拦截顺序

##### @Transactional
1. 在需要的类或方法上添加此注解
2. 此注解配置项有

* transactionManger
* propagation
* isolation
* timeout
* readOnly
* callback

##### Spring声明式事务何时失效
1. 方法访问权限必须是public,  private等权限,事务失效
2. 方法被定义成了final,会导致事务失效
3. 在同一个类中的方法直接内部调用, 会导致事务失效
4. 一个方法如果没交给spring管理, 就不会生成 spring事务
5. 多线程调用, 两个方法不在同一个线程中, 获取到的数据库连接不一样
6. 数据库存储引擎不支持事务
7. 如果自己try...catch误吞了异常,事务失效
8. rollbackFor参数设置错误

### Spring的JDBC异常抽象
Spring会将数据操作的异常转换为DataAccessException,无论使用何种数据访问方式,都能使用一样的异常
1. Spring通过SQLErrorCodeSQLExceptionTranslator类解析错误码
2. ErrorCode都放在spring-jdbc包下的support/sql-error-codes.xml下,包含了不同数据库的错误码

### 扩展
#### 一些常用注解
1. Java Config相关注解
* @Configuration 表示当前java类是一个配置类
* @ImportResource
* @CpomponentScan
* @Bean
* @ConfigurationProperties

2. 定义相关注解
* @Component/@Repository/@Service
* @Controller/@RestController
* @RequestMapping

3. 注入相关注解
* @Autowired/@Qualifier/@Resource
* @Value

## O/R Mapping实践
### Spring Data JPA(Java Persistence API)
1. JPA为对象关系映射提供了一种基于POJO的持久化模型
* 简化了数据持久化代码的开发
* 为java社区屏蔽不同持久化API的差异
2. Spring Data在保留底层存储特性的同时，提供了相对一致的，基于Spring的编程模型
* 主要模块： Spring Data Commons/JDBC/JPA/Redis/...

#### 常用JPA注解
实体
* @Entity
* @MappedSuperclass
* @Table(name)
主键
* @Id
    * @GeneratedValue(strategy, generator)
    * @SequenceGenerator(name, sequenceName)

```java
@Entity(name="Product")
public static class Product {
    @Id
    @GeneratedValue(strategy=GenerationType.SEQUENCE, generator="sequence-generator")
    @SequenceGenerator(name="sequence-generator", sequenceNam="product_sequence")
    private Long id;

    @Column(name="product_name")
    private String name;
}
```

映射
* @Column(name,nullable,length,insertable,updatable)
* @JoinTable(name)\@JoinColumn(name)

关系
* @OneToOne\@OneToMany\@ManyToOne\@ManyToMany
* @Orderby

#### 操作数据库
1. Repository
* @EnableJpaRepositories 自动发现对应的Repository
* Repository接口    泛型中指定实体对象
    * CrudCrudRepository
    * PagingAndSortingRepository
    * JpaRepository
2. 定义查询
* 根据方法名定义查询
    * find...By.../read...By.../query...By.../get...By...
    * count...By...
    * ...OrderBy...ASC/DESC
    * And/Or/IgnoreCase
    * Top/Fisrt/Distinct

* 分页查询接口
    * PagingAndSortingRepository
    * Pageable/Sort
    * Slice/Page

#### Spring Data JPA的Repository如何从接口变成Bean的 
Repository Bean是如何创建的
* JpaRepositoriesRegistrar
    * 激活了@EnableJpaRepositories
    * 返回了JpaRepositoryConfigExtension
* RepositoryBeanDefinitionRegistrarSupport.registerBeanDefinitions
    * 注册Repository Bean(类型是JpaRepositoryFactoryBean)
* RepositoryConfigurationExtensionSupport.getRepositoryConfigurations
    * 取得Repository配置
* JpaRepositoryFactory.getTargetRepository
    * 创建了Repository

接口中自定义的方法是如何被解释的
* RepositoryFactorySupport.getRepository添加了Advice
    * DefaultMethodInvokingMethodInterceptor
    * QueryExcutorMethodInterceptor

* AbstractJpaQuery.execute 执行具体的查询

* 语法解析在spring-data-commons包repository中的的Part中

### MyBatis
一款优秀的持久层框架、支持定制化SQL、存储过程、高级映射
#### Spring中使用MyBatis
1. MyBatis Spring Adapter
2. MyBatis Sprin-Boot-Starter

#### 简单配置
1. mybatis.mapper-locations= classpath:mapper/*.xml
2. mybatis.configuration.map-underscore-to-camel-case= true
3. mybatis.type-aliases-package= 类型别名的包名
4. mybtis.type-handler-package= TypeHandler扫描包名

#### Mapper的定义与扫描
1. @MapperScan配置扫描位置
2. @Mapper 定义接口
3. XML与注解 定义映射

* 注解方式

```java
@Insert("insert into customers (name, address, create_time, update_time)"
        + "values (#{name}, #{address}, now(), now())")
@Options(useGeneratedKeys = true)
int save(Customer customer);

@Select("select * from customers where id = #{id}")
@Results({
        @Result(id = true, column = "id", property = "id"),
        @Result(column = "create_time", property = "createTime"),

})
Customer findById(@Param("id") Long id);
```

* XML方式

#### Mybatis实用工具
##### [Mybatis Generator](https://mybatis.org/generator/)
它是Mybatis官方提供的代码生成器,根据数据库表生成相关代码(POJO、Mapper接口、SQL Map XML),运行Mybatis Generator方式如下,
* 命令行：java -jar mybatis-generator-core-x.x.x -configfile generatorConfig.xml
* Maven Plugin(mybatis-generator-maven-plugin)
    * mvn mybatis-generator:generate
    * ${basedir}/src/main/resource/generatorConfig.xml **url中的&放在xml中需要转义成&amp;**
* Java 程序

```java
    List<String> warnings = new ArrayList<>();
    ConfigurationParser cp = new ConfigurationParser(warnings);
    Configuration config = cp.parseConfiguration(
            this.getClass().getResourceAsStream("/generatorConfig.xml"));
    DefaultShellCallback callback = new DefaultShellCallback(true);
    MyBatisGenerator myBatisGenerator = new MyBatisGenerator(config, callback, warnings);
    myBatisGenerator.generate(null);
```

**配置Mybatis Generator**
* generatorConfiguration
* context
    * jdbcConnection
    * javaModleGenerator
    * sqlMapGenerator
    * javaClientGenerator
    * table

**内置插件(mybatis-generator-plugins)**
* FluentBuilderMethodsPlugin
* ToStringPlugin
* SerializablePlugin
* RowBoundsPlugin
* ...

```java
<generatorConfiguration>
    <context id="MysqlTables" targetRuntime="MyBatis3">
        <plugin type="org.mybatis.generator.plugins.FluentBuilderMethodsPlugin" />
        <plugin type="org.mybatis.generator.plugins.ToStringPlugin" />
        <plugin type="org.mybatis.generator.plugins.SerializablePlugin" />
        <plugin type="org.mybatis.generator.plugins.RowBoundsPlugin" />

        <jdbcConnection driverClass="com.mysql.cj.jdbc.Driver"
                        connectionURL="jdbc:mysql://localhost:3306/test_1?serverTimezone=GMT&amp;useUnicode=true&amp;characterEncoding=utf8"
                        userId="dmetal"
                        password="dmetal1207">
        </jdbcConnection>

        <javaModelGenerator targetPackage="com.spring.hualan.hualanspring.gmodel"
                            targetProject="./src/main/java">
            <property name="enableSubPackages" value="true" />
            <property name="trimStrings" value="true" />
        </javaModelGenerator>

        <sqlMapGenerator targetPackage="com.spring.hualan.hualanspring.gmapper"
                         targetProject="./src/main/resources/mapper">
            <property name="enableSubPackages" value="true" />
        </sqlMapGenerator>

        <javaClientGenerator type="MIXEDMAPPER"
                             targetPackage="com.spring.hualan.hualanspring.gmapper"
                             targetProject="./src/main/java">
            <property name="enableSubPackages" value="true" />
        </javaClientGenerator>

        <table tableName="test_01" domainObjectName="User">
            <generatedKey column="id" sqlStatement="LAST_INSERT_ID()" identity="true" />
            <columnOverride column="user_name" javaType="java.lang.String" jdbcType="VARCHAR"/>
            <columnOverride column="addresss" javaType="java.lang.String" jdbcType="VARCHAR"/>
            <columnOverride column="phone" javaType="java.lang.Integer" jdbcType="INTEGER"/>
        </table>
    </context>
</generatorConfiguration>
```

**使用生成的对象**
* 简单操作直接使用生成的xxxMapper的方法
* 复杂操作使用生成的xxxExample对象

```java
void playWithArtifacts() {
		User user = new User()
				.withUserName("nico")
				.withAddress("Beijing")
				.withPhone(324234)
				.withCreateTime(new Date())
                .withUpdateTime(new Date());
		int id = userMapper.insert(user);
		log.info("id {}", id);
		User s = userMapper.selectByPrimaryKey(1L);
		log.info("User {}", s);
        //复杂查询
		UserExample example = new UserExample();
		example.createCriteria().andUserNameEqualTo("nico");
		List<User> list = userMapper.selectByExample(example);
		list.forEach(e -> log.info("selectByExample: {}", e));
	}
```

#### [Mybatis PageHelper](https://github.com/pagehelper/Mybatis-PageHelper)
1. 支持多种数据库、多种分页方式、支持spring-boot集成[pagehelper-spring-boot-starter](https://github.com/pagehelper/pagehelper-spring-boot)
2. 配置PageHelper

* 引入依赖

```java
<dependency>
    <groupId>com.github.pagehelper</groupId>
    <artifactId>pagehelper-spring-boot-starter</artifactId>
    <version>1.2.4</version>
</dependency>
```

* 配置参数

```yml
pagehelper:
  offset-as-page-num: true #offset作为分页使用
  reasonable: true
  page-size-zero: true # pageSize=0时返回所有数据
  support-methods-arguments: true
```

* java代码

```java
UserExample example = new UserExample();
RowBounds rowBounds = new RowBounds(1, 3);
userMapper.selectByExampleWithRowbounds(example, rowBounds)
            .forEach(c -> log.info("Page(1) Coffee {}", c));
List<User> list = userMapper.selectByExampleWithRowbounds(example, rowBounds);
PageInfo page = new PageInfo(list);
log.info("PageInfo: {}", page);
```

## NoSQL初识及简单实践
### 前言-利用Docker辅助开发
#### 什么是[容器](https://cloud.google.com/learn/what-are-containers?hl=zh-cn)
它是一种应用层的抽象，是一个标准化的单元，容器跟虚拟机不同，容器不包含操作系统的内容，容器内的应用进程直接运行于宿主的内核，容器内没有自己的内核，而且也没有进行硬件虚拟。因此容器要比虚拟机更轻量，启动和部署更快
#### 什么是[Docker](https://yeasy.gitbook.io/docker_practice/)
Docker在容器的基础上，进行了进一步的封装，从文件系统、网络互联到进程隔离等等，极大的简化了容器的创建和维护。使得Docker技术比虚拟机技术更为轻便、快捷。对于开发简化了重复搭建开发环境的工作，对于运维交付系统更为流畅、伸缩性更好

#### Docker安装
1. [官方网站](https://www.docker.com/)
2. Windows下安装默认在C盘, 更改盘符可使用`mklink /j "C:\Program Files\Docker" "D:\Program Files\Docker"`建立软链接，把默认安装位置映射到自定义位置。
3. 修改Docker镜像存储路径
* 停止docker
* 查看应用是否停止 `wsl --list -v`
* 导出docker镜像文件 `wsl --export docker-desktop-data "D:develop\docker\docker-desktop-data.tar"`, `wsl --export docker-desktop "E:develop\docker\docker-desktop.tar"`
* 注销docker-desktop-data、docker-desktop
* 导入自定义文件夹
    `wsl --import docker-desktop-data "D:develop\docker\data" "D:develop\docker\docker-desktop-data.tar" --version 2` 
    `wsl --import docker-desktop "D:develop\docker\desktop" "D:develop\docker\docker-desktop.tar" --version 2`
* 重启Docker

#### Docker使用
##### Docker常用命令
**镜像相关**
1. 下载镜像 `docker pull <image>`
2. 搜索镜像 `docker search <image>`

**容器相关**
1. 运行镜像 `docker run`
    * -d 后台运行容器
    * -e 设置环境变量
    * --expose/-p 宿主端口:容器端口
    * --name 指定容器名
    * --link 链接不同容器
    * -v 宿主目录: 容器目录，挂载磁盘卷
2. 启动/停止容器 `docker start/stop <dockername>`
3. `docker ps <dockername>`
4. 查看日志 `docker logs <dockername>`

##### Docker启动报错问题
1. 错误信息
`Hardware assisted virtualization and data execution protection must enabled BIOS`
2. 错误原因
Hyper-V已禁用或Hypervisor代理未运行。docker desktop基于windows hyper-v，必须确保hyper-v组件已经开启
3. 解决办法
* 启用Hyper-V `dism.exe /Online /Enable-Feature:Microsoft-Hyper-V /All`
* 如果Hyper-V 功能已启用但不起作用，需确保其守护进程已自动运行，若未运行则`bcdedit /set hypervisorlaunchtype auto`
* 上述命令在管理员身份下CMD运行完毕后，重启电脑即可

### NoSQL适用场景
1. 数据模型比较简单
2. 需要灵活性更强的数据库
3. 对数据库性能要求较高
4. 不需要高度的数据一致性
5. 对于给定Key比较容易映射复杂值的环境

### 分类
1. K-V键值数据库: 主要应用于缓存、处理大量数据的高负载访问、记录系统日志. 如Redis、Mecached
2. 列存储数据库: 主要应用于分布式数据的存储和管理. 如HBase, HadoopDB
3. 文档数据库: 主要应用于管理半结构化数据或面向文档的数据. 如MongoDB、ES

### 主流NoSQL数据库对比
1. 如果对数据的读写要求极高, 并且数据规模不大, 也不需要长期存储, 那就选Redis
2. 如果数据规模较大, 对数据的读性能要求很高, 数据表的结构需要经常变, 有时还需要做一些聚合查询, 那就选
MongoDB
3. 如果要构造一个搜索引擎或者要完成一个高大上的数据可视化平台, 并且数据本身也具有分析价值, 就选ES
4. 如果你要存储海量数据, 而且还不能预估数据规模将来会增长多么大, 那么选HBase

### ES
一个建立在全文搜索引擎库Lucene基础上的开源搜索和分析引擎. ES本身具有分布式存储、检索速度快的特性.通常会用来实现全文检索的功能. Elastic Stack主要包括ElasticSearch、Logstash、Kibana. 这三个经典组合称之为ELK. ElasticSearch主要用来做数据存储、Logstash主要用来做数据采集、Kibana主要用来做数据可视化展示
#### ES为什么快
1. ES是基于Lucene开发的一个全文搜索引擎. 一方面Lucene是擅长管理大量的索引数据;另外一方面,它会对数据进行分词以后再保存索引, 能够去提升数据的检索效率
2. ES采用倒排索引. 所谓倒排索引就是通过属性值来确定数据记录位置的索引. 从而避免全表扫描的问题
3. ES存储数据采用分片机制
4. ES扩展性好, 支持通过水平扩展的方式来动态增加节点. 从而提升ES的处理性能, 能够支持上百台服务器节点的扩展, 并且支持TB级别的结构化数据和非结构化数据
5. ES内部提供的数据汇总和索引生命周期管理的功能, 更加便于高效地存储和检索数据

### MongoDB
#### 什么是[MongDB](https://www.mongodb.com/)
MongoDB 是一个基于分布式文件存储的数据库。由 C++ 语言编写。旨在为WEB应用提供可扩展的高性能数据存储解决方案。
MongoDB 是一个介于关系数据库和非关系数据库之间的产品，是非关系数据库当中功能最丰富，最像关系数据库的。
MongoDB 将数据存储为一个文档，数据结构由键值(key=>value)对组成。MongoDB 文档类似于 JSON 对象。字段值可以包含其他文档，数组及文档数组。
#### 主要特点
1. MongoDB 是一个面向文档存储的数据库，操作起来比较简单和容易。
2. Mongo支持丰富的查询表达式。查询指令使用JSON形式的标记，可轻易查询文档中内嵌的对象及数组
3. MongoDB允许在服务端执行脚本，可以用Javascript编写某个函数，直接在服务端执行，也可以把函数的定义存储在服务端，下次直接调用即可
4. MongoDB支持各种编程语言:RUBY，PYTHON，JAVA，C++，PHP，C#等多种语言

#### [MongoDB入门](https://www.runoob.com/mongodb/mongodb-intro.html)
##### MongoDB数据库角色说明
* 数据库用户角色：read、readWrite；
* 数据库管理角色：dbAdmin、dbOwner、userAdmin;
* 集群管理角色：clusterAdmin、clusterManager、4. clusterMonitor、hostManage；
* 备份恢复角色：backup、restore；
* 所有数据库角色：readAnyDatabase、readWriteAnyDatabase、userAdminAnyDatabase、dbAdminAnyDatabase
* 超级用户角色：root
* 内部角色：__system

##### MongoDB数据库角色详解
* Read：允许用户读取指定数据库
* readWrite：允许用户读写指定数据库
* dbAdmin：允许用户在指定数据库中执行管理函数，如索引创建、删除，查看统计或访问system.profile
* userAdmin：允许用户向system.users集合写入，可以在指定数据库里创建、删除和管理用户
* clusterAdmin：只在admin数据库中可用，赋予用户所有分片和复制集相关函数的管理权限
* readAnyDatabase：只在admin数据库中可用，赋予用户所有数据库的读权限
* readWriteAnyDatabase：只在admin数据库中可用，赋予用户所有数据库的读写权限
* userAdminAnyDatabase：只在admin数据库中可用，赋予用户所有数据库的userAdmin权限
* dbAdminAnyDatabase：只在admin数据库中可用，赋予用户所有数据库的dbAdmin权限
* root：只在admin数据库中可用。超级账号，超级权限

##### MongoDB基本命令
1. 连接 mongosh
2. 创建/切换数据库 `use dbname`
3. 查看数据 `show dbs`
4. 创建集合 `db.createCollection(name, options)`
`db.createCollection('collection1', {capped:true,size:1024,max:10000})`
* capped如果为true则创建固定集合（有着固定大小的集合）；
* size为固定集合指定一个最大值，如果capped为true需要指定该字段；
* max 指定固定集合中包含文档的最大数量
5. 查看集合 `show collections`
6. 插入文档 `db.collectionname.insert(document)`
* 单条数据 `db.collectionname.insert({'name':'zzz', 'age': 11})`
* 多条数据 `db.collectionname.insert([{'name':'zzz', 'age': 11}, {'name':'zzz', 'age': 11}])`
7. 查询文档`db.collectionname.find(query, projection)`
* query 可选，使用查询操作符指定查询条件
* projection    可选，使用投影操作符指定返回的键。查询时返回文档中所有键值，只需省略该参数即可（默认省略）
* find().pretty() 格式化方法显示文档
8. 更新文档 `db.collection.update(<query>,<update>,{upsert: <boolean>,multi: <boolean>,writeConcern:<document>}`
* query : update的查询条件，类似sql update查询内where后面的。
* update : update的对象和一些更新的操作符（如$,$inc...）等，也可以理解为sql update查询内set后面的
* upsert : 可选，这个参数的意思是，如果不存在update的记录，是否插入objNew,true为插入，默认是false，不插入。
* multi : 可选，mongodb 默认是false,只更新找到的第一条记录，如果这个参数为true,就把按条件查出来多条记录全部更新。
* writeConcern :可选，抛出异常的级别。

##### MongoDB操作符
1. 大于（>）: $gt
2. 大于等于（>=）： $gte
3. 小于（<）：$lt
4. 小于等于（<=）：$lte

```java
db.collectionname.find({age : {$gt : 30, $lt : 60,}})
```

##### Navicat连接mongoDB并创建集合和添加数据
1. 连接参数介绍
* stand alone：独立的
* shard cluster：分片集群
* replica set：复制集
* SRV record：SRV记录是DNS服务器的数据库中支持的一种资源记录的类型，它记录了哪台计算机提供了哪个服务这么一个简单的信息。
一般情况下，只需要连接主库查数据，选择独立的连接方式即可，填写好常规参数可以点击测试连接是否正常，即可连接。

#### 通过Docker启动MongoDB
1. 下载镜像 `docker pull mongo`
2. 运行容器`docker run -itd --name mongo -p 27017:27017 mongosh --auth`
* -itd  i是交互式操作，t是一个终端，d指的是在后台运行
* --name    容器名称
* -p 27017:27017    映射容器服务的27017端口到宿主机的27017端口, 外部可以直接通过宿主机ip:27017访问到mongo的服务
* --auth    指需要密码才能访问容器服务
3. 登录创建的mongoDB容器`docker exec -it  mongo mongosh admin`
4. 创建用户 `db.createUser({ user:'root',pwd:'d1027',roles:[ { role:'root', db: 'admin'}]});`
* user  用户名
* pwd   用户密码
* cusomData 任意内容，例如可以为用户全名介绍
* roles 指定用户的角色，可以用一个空数组给新用户设定空角色。在roles字段,可以指定内置角色和用户定义的角色
* db    是指定数据库的名字，admin是管理数据库
5 删除用户: `db.system.users.remove({user:"用户名"})`
6. 登录`db.auth('root', 'd1207')`

#### Spring对MongoDB的支持
##### Spring Data MongoDB
提供类似jdbcTemplate的MongoTemplate对数据做各种增删改查操作，提供了类似JPA Repository的Repository 
##### 基本用法
**项目配置**
`spring.data.mongodb=uri: mongodb://test:test@localhost:27017/test`
**实体**
* @Document 类似entity
* @ID

```java
@Document
@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class Product {

    @Id
    private String id;
    private String name;
    private Money price;
    private Date createTime;
    private Date updateTime;
}
```

**MongoTemplate基本用法**
* save/remove
* Criteria/Query/Update

```java
//插入
Product product = Product.builder()
        .name("apple")
        .price(Money.of(CurrencyUnit.of("CNY"), 20.0))
        .createTime(new Date())
        .updateTime(new Date()).build();
Product saved = mongoTemplate.save(product);
log.info("Product {}", saved);

//查询
List<Product> list = mongoTemplate.find(
        Query.query(Criteria.where("name").is("apple")), Product.class);
log.info("Find {} Product", list.size());

//更新
UpdateResult result = mongoTemplate.updateFirst(query(where("name").is("apple")),
            new Update().set("price", Money.ofMajor(CurrencyUnit.of("CNY"), 30))
                    .currentDate("updateTime"),
            Product.class);

//删除
Product pr = mongoTemplate.findById(saved.getId(), Product.class);
mongoTemplate.remove(pr);
```

**Repository基本用法**

```java
@EnableMongoRepositories
public class test {

    @Autowired
    private ProductRepository productRepository;

    public void testRepository() {
        Product banana = Product.builder()
                .name("banana")
                .price(Money.of(CurrencyUnit.of("CNY"), 10.0))
                .createTime(new Date())
                .updateTime(new Date()).build();
        Product peach = Product.builder()
                .name("peach")
                .price(Money.of(CurrencyUnit.of("CNY"), 20.0))
                .createTime(new Date())
                .updateTime(new Date()).build();

        productRepository.insert(Arrays.asList(banana, peach));
        productRepository.findAll(Sort.by("name"))
                .forEach(c -> log.info("Saved Product {}", c));

        Thread.sleep(1000);
        peach.setPrice(Money.of(CurrencyUnit.of("CNY"), 35.0));
        peach.setUpdateTime(new Date());
        productRepository.save(peach);
        productRepository.findByName("peach")
                .forEach(c -> log.info("Product {}", c));

        //productRepository.deleteAll();
    }
}
```

### [Redis](https://redis.io/)
#### Redis简介
redis是一个开源的、使用C语言编写的、支持网络交互的、可基于内存也可持久化的Key-Value数据库。Redis的数据是存在内存中的。它的读写速度非常快，每秒可以处理超过10万次读写操作。因此redis被广泛应用于缓存，另外，Redis也经常用来做分布式锁。除此之外，Redis支持事务、持久化、LUA 脚本、LRU 驱动事件、多种集群方案。
#### Redis数据类型
1. 五种基本数据类型及应用场景
* String（字符串）， 共享session、分布式锁，计数器、限流等
* Hash（哈希），缓存用户信息等
* List（列表），消息队列，文章列表等
* Set（集合）， 用户标签,生成随机数抽奖等
* zset（有序集合）排行榜，社交需求（如用户点赞）等
2. 三种特殊数据结构
* Geospatial 地理位置定位，用于存储地理位置信息，并对存储的信息进行操作
* Hyperloglog 用来做基数统计算法的数据结构，如统计网站的UV
* Bitmap 用一个比特位来映射某个元素的状态，在Redis中，它的底层是基于字符串类型实现的，可以把bitmaps成作一个以比特位为单位的数组

#### Redis的哨兵和集群模式
##### Redis Sentinel——Redis的一种高可用方案
##### Redis Cluster

#### 通过Docker启动Redis
1. 获取镜像 `docker pull redis`
2. 启动Redis `docker run --name redis -d -p 6379:6379 redis`
3. 登录 `docker exec it redis redis-cli`

#### Redis基本命令
1. `keys *` 查询key
2. `hgetall key` 查询key下所有value
3. `flushall` 清空缓存

#### Spring对Redis的支持
##### Spring Data Redis
1. 支持的客户端Jedis/Lettuce， 提供RedisTemplate、Repository
2. Jedis客户端简单使用
* Jedis不是线程安全的，不能在多个线程间共享同一个Jedis实例
* 通过JedisPool获取Jedis实例
* 直接使用Jedis中的方法

##### Spring的缓存抽象-基于AOP
为不同的缓存提供一层抽象,为java方法增加缓存，缓存执行结果,支持ConcurrentMap、EhCache、Caffeine、JCache,主要接口Cache、CacheManager
**基于注解的缓存**

* @EnableCaching——开启缓存
    * @Cachable——执行方法的结果在缓存中直接在缓存中取,不在则将执行结果放入缓存
    * @CacheEvict——缓存清理
    * @CachePut——不管方法执行情况直接做缓存设置
    * Caching
    * @CacheConfig 设置缓存，例如缓存名

**通过SpringBoot配置Redis缓存**
* 配置依赖
spring-boot-starter-cache、spring-boot-starter-data-redis
* 配置缓存

```java
spring.cache.type=redis
spring.cache.cache-names=coffee
spring.cache.redis.time-to-live=5000
spring.cache.redis.cache-null-values=false

spring.redis.host=localhost
```

* 开启缓存@EnableCaching

##### 使用RedisTemplate、Repository
**与Redis建立连接**

* 配置连接工厂 LettuceConnectionFactory与JedisConnectionFactory
    * RedisStandaloneConfiguration
    * RedisSentinelConfiguration
    * RedisClusterConfiguration
**Lettuce内置支持读写分离LettuceClientConfigurationBuilderCustomizer**

```java
@Bean
public LettuceClientConfigurationBuilderCustomizer customizer() {
    return builder -> builder.readFrom(ReadFrom.MASTER_PREFERRED);
}
```

**RedisTemplate、StringRedisTemplate**

```java
@Bean
public RedisTemplate<String, Coffee> redisTemplate(RedisConnectionFactory redisConnectionFactory) {
    RedisTemplate<String, Coffee> template = new RedisTemplate<>();
    template.setConnectionFactory(redisConnectionFactory);
    return template;
}

HashOperations<String, String, Coffee> hashOperations = redisTemplate.opsForHash();
if (redisTemplate.hasKey(CACHE) && hashOperations.hasKey(CACHE, name)) {
    log.info("Get coffee {} from Redis.", name);
    return Optional.of(hashOperations.get(CACHE, name));
}
```

### HBase
### ES


## 数据访问进阶
### [Project Reactor](https://projectreactor.io/)
#### 什么是Reactive Programming(反应式编程)
反应式编程(响应式编程)是一种面向数据流和变化传播的声明式编程范式.可以在编程语言中很方便地表达静态或动态的数据流，而相关的计算模型会自动将变化的值通过数据流进行传播.反应(响应)即在某个事件发生时作出回应。在响应式编程中，通常是采用异步回调的方式，回调方法的调用和控制则会由响应式框架来完成，对于应用开发来说只需要关注回调方法的实现就可以了。
#### Project Reacctor介绍
Reactor是基于JVM的非阻塞API，他直接跟JDK8中的API相结合，比如：CompletableFuture，Stream和Duration等。它提供了两个非常有用的异步序列API：Flux和Mono，并且实现了Reactive Streams的标准。
#### 核心概念
1. Operators-Publisher/Subscriber——Nothing Happens Until You Subscribe()
* Flux\[0...N\](一个表示包含0-n个元素的异步序列),在该序列中可以包含三种不同类型的消息通知：正常的包含元素的消息、序列结束的消息和序列出错的消息。当消息通知产生时，订阅者中对应的方法onNext(), onComplete()和onError()会被调用
* Mono\[0...1\](一个表示包含1个元素或者没有元素的异步结果)-同样包含与Flux相同的三种类型的消息通知
2. Backpressure
背压也叫反压，指的是下游系统处理过慢，导致上游系统阻塞的现象
* Subscribption
* onRequest()、onCancel()、onDispose()
3. Schedulers线程调度
* immediate()、single()、newSingle()
* elastic()、parallel()、newParallel()
4. 错误处理
* onError、onErrorReturn、onErrorResume
* doOnError、doFinally

#### 示例
1. 引入reactor依赖
2. 简单应用

```java
Flux.range(1, 6)
    //每次执行Request的时候打印这次请求了多少个数
    .doOnRequest(n -> log.info("Request {} number", n)) // 注意顺序造成的区别， publish会覆盖的subscribe的4个元素的设置
//				.publishOn(Schedulers.elastic()) //此处之后的代码都会执行在elastic线程池里
    .doOnComplete(() -> log.info("Publisher COMPLETE 1")) //publish 1-6完成后打印
    .map(i -> {
        log.info("Publish {}, {}", Thread.currentThread(), i); //map里可以实现元素转换，此处用于演示map执行在哪个线程
//                    return 10 / (i - 3);
        return i;
    })
    .doOnComplete(() -> log.info("Publisher COMPLETE 2")) //再执行一个onComplete，看两次都执行在哪个线程上
//				.subscribeOn(Schedulers.single()) //启动一个单独线程进行订阅
//				.onErrorResume(e -> { //发生错误时进行处理
//					log.error("Exception {}", e.toString());
//					return Mono.just(-1);
//				})
//				.onErrorReturn(-1) //发生错误时返回默认值
    .subscribe(i -> log.info("Subscribe {}: {}", Thread.currentThread(), i), //对每一个元素如何消费
            e -> log.error("error {}", e.toString()), //对错误如何消费
            () -> log.info("Subscriber COMPLETE") //运行完成后如何处理
//						s -> s.request(4) //backpressure，每次只请求4个元素, 不设置默认Long的最大值，不管生产多少都消费
    );
```

### Reactive访问Redis
Spring Data Redis中的主要支持

* ReactiveRedisConnection 建立Reactive连接
* ReactiveRedisConnectionFactory
* ReactiveRedisTemplate
    * opsForXxx()

```java
ReactiveHashOperations<String, String, String> hashOps = reactiveStringRedisTemplate.opsForHash();
Flux.fromIterable(list) //list是从数据库中查询出来的
    .publishOn(Schedulers.single()) //以单线程的方式将list全部publish
    .doOnComplete(() -> log.info("list ok")) //publish完成后打印
    .flatMap(c -> {
        log.info("try to put {},{}", c.getUserName(), c.getPhone());
        return hashOps.put("users", c.getUserName(), c.getPhone().toString()); //将list的每个元素放入redis的Hash中
    })
    .doOnComplete(() -> log.info("set ok")) //全部放入后打印
    .concatWith(reactiveStringRedisTemplate.expire("users", Duration.ofMinutes(1))) //设置缓存有效期1分钟
    .doOnComplete(() -> log.info("expire ok")) //设置完成后打印
    .onErrorResume(e -> {
        log.error("exception {}", e.getMessage());
        return Mono.just(false);
    })
    .subscribe(b -> log.info("Boolean: {}", b), //对每一个Flux Boolean的subscribe打印一个布尔值
            e -> log.error("Exception {}", e.getMessage()));
```

### 
1. MongoDB官方提供了支持Reactive的驱动: mongodb-driver-reactivestreams
2. Spring Data MongoDB中的主要支持
* ReactiveMongoClientFactoryBean
* ReactiveMongoDatabaseFactory
* ReactiveMongoTemplate
```java
 @Resource
private ReactiveMongoTemplate reactiveMongoTemplate;
// 插入数据
reactiveMongoTemplate.insertAll(initProduct()) //initProduct()方法构建两个Product对象
                .publishOn(Schedulers.elastic())
                .doOnNext(c -> log.info("Next: {}", c))
                .doOnComplete(runnable)
                .doFinally(s -> {
                    log.info("Finally 1, {}", s);
                })
                .count()
                .subscribe(c -> log.info("Insert {} records", c));
//更新数据
reactiveMongoTemplate.updateMulti(query(where("price").gte(3000L)),
        new Update().inc("price", -500L)
                .currentDate("updateTime"), Product.class)
.doFinally(s -> {
    log.info("Finally 2, {}", s);
})
.subscribe(r -> log.info("Result is {}", r));
```

### Reactive访问RDBMS
#### Spring Data R2DBC
1. 一些主要类
* ConnectionFactory
* DatabaseClient
* R2dbcExceptionTranslator

### 通过AOP打印数据访问层摘要
#### Spring AOP
#### 核心概念
1. Aspect   切面
2. Join Point   连接点，SpringAOP中总是代表一次方法执行
3. Advice   通知，在连接点上执行的动作
4. Pointcut 切点，说明如何匹配连接点
5. Introduction 引入，为现有类型声明额外的方法和属性
6. Target object    目标对象
7. AOP Proxy    AOP代理对象，可以是JDK的动态代理，也可以是cglib代理
8. Weaving  织入， 连接切面和目标对象或类型创建代理的过程

#### 常用注解
1. @EnableAspectJAutoProxy
2. @ Aspect 声明当前类是一个切面
3. @Pointcut
4. @Before 指定Advice是在方法执行前执行的
5. @After/@AfterReturning/@AfterThrowing
6. @Around
7. @Order 
```java
@Aspect
@Component
@Slf4j
public class PerformanceAspect {
    // 拦截gmapper下的所有方法执行，在方法执行前后切入
    @Around("execution(* com.spring.hualan.hualanspring.gmapper..*(..))")
    public Object logPerformance(ProceedingJoinPoint pjp) throws Throwable {
        long startTime = System.currentTimeMillis();
        String name = "-";
        String result = "Y";
        try {
            name = pjp.getSignature().toShortString();
            return pjp.proceed();
        } catch (Throwable t) {
            result = "N";
            throw t;
        } finally {
            long endTime = System.currentTimeMillis();
            log.info("{};{};{}ms", name, result, endTime - startTime);
        }
    }
}
```

#### 输出SQL日志到控制台的简单配置
##### HikariCP
可以使用p6spy打印SQL
**引入依赖**

```yml
<dependency>
    <groupId>p6spy</groupId>
    <artifactId>p6spy</artifactId>
    <version>3.8.1</version>
</dependency>
```

**修改datasource**

```yml
spring.datasource.driver-class-name=com.p6spy.engine.spy.P6SpyDriver
spring.datasource.url=jdbc:p6spy:mysql://localhost:3306/
```

**添加spy.properties**

```yml
//单行日志
logMessageFormat=com.p6spy.engine.spy.appender.SingleLineFormat
//使用Slf4J记录sql
appender=com.p6spy.engine.spy.appender.Slf4JLogger
//是否开启慢SQL记录
outagedetection=true
//慢SQL记录标准，单位秒
outagedetectioninterval=2
```

##### Alibaba Druid
Druid可以使用p6spy打印日志, 也可以使用自身配置
1. 修改datesouce,添加配置
```yml
spring.datasource.slf4j.enabled=true
spring.datasource.slf4j.statement-create-after-log-enabled=false
spring.datasource.slf4j.statement-close-after-log-enabled=false
spring.datasource.slf4j.result-set-open-after-log-enabled=false
spring.datasource.slf4j.result-set-close-after-log-enabled=false
```
2. 指定druid的日志登记为debug才能显示
```yml
logging.level.druid.sql.Statement=debug
```



