---
layout: post
title:  "Hibernate的基本实践"
date:   2018-04-27 09:50:30 +0200
categories: database jdbc
---

## Hibernate体系结构
<img src= "/assets/files/hibernate.png" alt="加载错误" title="hibernate体系结构" />

## 使用Hibernate
Hibernate Session可以实现CRUD功能
* 导入Hibernate

```java
<dependency>
    <groupId>org.hibernate</groupId>
    <artifactId>hibernate-core</artifactId>
    <version>4.3.2.Final</version>
</dependency>
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>5.1.40</version>
</dependency>
<dependency>
        <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.12</version>
</dependency>
```

* 配置主配置文件 hibernate.cfg.xml

```java
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE hibernate-configuration PUBLIC
    "-//Hibernate/Hibernate Configuration DTD 3.0//EN"
    "http://www.hibernate.org/dtd/hibernate-configuration-3.0.dtd">
<hibernate-configuration>
    <session-factory>
        <!-- 数据库连接参数  -->
        <property name="connection.driver_class">
            com.mysql.jdbc.Driver
        </property>
        <property name="connection.url">
            jdbc:mysql://localhost:3306/ssh         
        </property>
        <property name="connection.username">
            root
        </property>
        <property name="connection.password">
            root
        </property>
        <!-- 配置数据的方言(dialect) MySQL方言 -->
        <property name="dialect">
            org.hibernate.dialect.MySQLDialect
        </property>
        <!-- 配置调试参数, 显示生成的SQL -->
        <property name="show_sql">true</property>
        <property name="format_sql">true</property>
        <!-- 配置子配置文件(映射文件) -->
        <mapping resource="hbm/User.hbm.xml"/>
    </session-factory>
</hibernate-configuration>
```

* 声明实体类
无参构造，有参构造，getter/setter，hascode/equals
* 配置子配置文件(映射文件) hbm/User.hbm.xml

```java
<?xml version="1.0" encoding="UTF-8"?>
	<!DOCTYPE hibernate-mapping PUBLIC 
    	"-//Hibernate/Hibernate Mapping DTD 3.0//EN"
    	"http://www.hibernate.org/dtd/hibernate-mapping-3.0.dtd">
<hibernate-mapping>
    <!-- 定义了 类和表的对应关系 -->
    <class name="cn.tedu.entity.User" 
        table="t_user">
        <!-- 定义属性和列(字段)的对应关系 -->
        <id name="id" column="u_id"></id>
        <property name="name" column="u_name"/>
        <property name="password" column="u_password"/>
        <property name="age" column="u_age"/>
        <property name="salary" column="u_salary"/>
        <property name="hiredate" column="u_hiredate"/>
    </class>
</hibernate-mapping>
```

* 利用Hibernate操作数据库

## HQL
Hibernate为了消除SQL提供了替代的查询语言 HQL.
HQL的语法与SQL非常类似:
1. 将SQL中的表名替换为对应的实体类名
2. 将SQL中的列名(字段名)替换为对应的实体属性名
3. 使用Query接口执行HQL查询

## 整合SSH
1. 创建项目
    - 创建部署描述文件
    - 引入目标服务器运行环境
    - 如果创建不了项目, 请检测网络:是否能够访问Maven服务器.
2. 导入相关的包:
    - Struts2
    - Struts2-Spring-plugin
    - Struts2-json-plugin
    - Hibernate
    - MySql jdbc Drvier
    - dataSource
    - Spring-orm
    - Spring-jdbc
    - JUnit
3. 配置
    - web.xml
    - struts2
    - Spring + hibernate

4. 整合具体步骤:
* 创建项目导入包:

```java
<dependency>
    <groupId>org.apache.struts</groupId>
    <artifactId>struts2-core</artifactId>
    <version>2.5.1</version>
</dependency>
<dependency>
    <groupId>org.apache.struts</groupId>
    <artifactId>struts2-spring-plugin</artifactId>
    <version>2.5.1</version>
</dependency>
<dependency>
    <groupId>org.apache.struts</groupId>
    <artifactId>struts2-json-plugin</artifactId>
    <version>2.5.1</version>
</dependency>

<dependency>
    <groupId>org.hibernate</groupId>
    <artifactId>hibernate-core</artifactId>
    <version>4.3.2.Final</version>
</dependency>
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>5.1.40</version>
</dependency>

<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.12</version>
</dependency>

<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid</artifactId>
    <version>1.0.23</version>
</dependency>
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-orm</artifactId>
    <version>4.1.6.RELEASE</version>
</dependency>
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-jdbc</artifactId>
    <version>4.1.6.RELEASE</version>
</dependency>
```

* 配置web.xml

```java
  <filter>
    <display-name>StrutsPrepareAndExecuteFilter</display-name>
    <filter-name>StrutsPrepareAndExecuteFilter</filter-name>
    <filter-class>org.apache.struts2.dispatcher.filter.StrutsPrepareAndExecuteFilter</filter-class>
  </filter>
  <filter-mapping>
    <filter-name>StrutsPrepareAndExecuteFilter</filter-name>
    <url-pattern>/StrutsPrepareAndExecuteFilter</url-pattern>
  </filter-mapping>
  <listener>
    <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
  </listener>
  <context-param>
      <param-name>contextConfigLocation</param-name>
      <param-value>classpath:conf/spring-*.xml</param-value>  
  </context-param> 
```

* 添加struts配置文件: struts.xml

```java
<?xml version="1.0" encoding="UTF-8"?>
<!-- 从 struts-2.5.dtd 文件中复制 DOCTYPE -->
<!DOCTYPE struts PUBLIC
    "-//Apache Software Foundation//DTD Struts Configuration 2.5//EN"
    "http://struts.apache.org/dtds/struts-2.5.dtd">
<struts>
</struts>
```

* 添加spring-struts配置文件: conf/spring-struts.xml

```java
<?xml version="1.0" encoding="UTF-8"?>
<!-- /** 配置文件描述: spring-mvc配置 */ -->
<beans default-lazy-init="true"
    xmlns="http://www.springframework.org/schema/beans" 
    xmlns:p="http://www.springframework.org/schema/p"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
    xmlns:context="http://www.springframework.org/schema/context"
    xmlns:tx="http://www.springframework.org/schema/tx"
    xmlns:aop="http://www.springframework.org/schema/aop" 
    xmlns:mvc="http://www.springframework.org/schema/mvc"
    xmlns:util="http://www.springframework.org/schema/util"
    xmlns:jpa="http://www.springframework.org/schema/data/jpa"
    xsi:schemaLocation="  
       http://www.springframework.org/schema/beans   
       http://www.springframework.org/schema/beans/spring-beans-4.1.xsd  
       http://www.springframework.org/schema/mvc   
       http://www.springframework.org/schema/mvc/spring-mvc-4.1.xsd   
       http://www.springframework.org/schema/tx   
       http://www.springframework.org/schema/tx/spring-tx-4.1.xsd   
       http://www.springframework.org/schema/aop 
       http://www.springframework.org/schema/aop/spring-aop-4.1.xsd
       http://www.springframework.org/schema/util 
       http://www.springframework.org/schema/util/spring-util-4.1.xsd
       http://www.springframework.org/schema/data/jpa 
       http://www.springframework.org/schema/data/jpa/spring-jpa-1.3.xsd
       http://www.springframework.org/schema/context
       http://www.springframework.org/schema/context/spring-context-4.1.xsd">

    <context:component-scan base-package="cn.tedu.ssh.action"/>
</beans> 
```

* 添加数据库连接参数文件 conf/conf.properties

```java
# conf.properties
driver=com.mysql.jdbc.Drvier
url=jdbc:mysql://localhost:3306/ssh
username=root
password=root
initialSize=5
maxActive=50
minIdle=0
maxWait=60000
timeBetweenLogStatsMillis=60000
```

* 利用Spring配置文件, 配置Hibernate: conf/spring-hibernate.xml

```java
<?xml version="1.0" encoding="UTF-8"?>
<!-- /** 配置文件描述: spring-mvc配置 */ -->
<beans default-lazy-init="true"
    xmlns="http://www.springframework.org/schema/beans" 
    xmlns:p="http://www.springframework.org/schema/p"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
    xmlns:context="http://www.springframework.org/schema/context"
    xmlns:tx="http://www.springframework.org/schema/tx"
    xmlns:aop="http://www.springframework.org/schema/aop" 
    xmlns:mvc="http://www.springframework.org/schema/mvc"
    xmlns:util="http://www.springframework.org/schema/util"
    xmlns:jpa="http://www.springframework.org/schema/data/jpa"
    xsi:schemaLocation="  
       http://www.springframework.org/schema/beans   
       http://www.springframework.org/schema/beans/spring-beans-4.1.xsd  
       http://www.springframework.org/schema/mvc   
       http://www.springframework.org/schema/mvc/spring-mvc-4.1.xsd   
       http://www.springframework.org/schema/tx   
       http://www.springframework.org/schema/tx/spring-tx-4.1.xsd   
       http://www.springframework.org/schema/aop 
       http://www.springframework.org/schema/aop/spring-aop-4.1.xsd
       http://www.springframework.org/schema/util 
       http://www.springframework.org/schema/util/spring-util-4.1.xsd
       http://www.springframework.org/schema/data/jpa 
       http://www.springframework.org/schema/data/jpa/spring-jpa-1.3.xsd
       http://www.springframework.org/schema/context
       http://www.springframework.org/schema/context/spring-context-4.1.xsd">

    <!-- spring-hibernate.xml  -->
    <!-- 配置数据源, 连接到数据库, 替换Hibernate的主配置文件 hibernate.cfg.xml -->
    <!-- 读取properties -->
    <util:properties id="conf" location="classpath:conf/conf.properties"/>
    <!-- 配置数据源 -->
    <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource"
init-method="init" destroy-method="close">
<property name="driverClassName" value="#{conf.driver}"/> 
<property name="url" value="#{conf.url}"/>
<property name="username" value="#{conf.username}"/>
<property name="password" value="#{conf.password}"/>
<property name="initialSize" value="#{conf.initialSize}"/>
<property name="maxActive" value="#{conf.maxActive}"/>
<property name="minIdle" value="#{conf.minIdle}"/>
<property name="maxWait" value="#{conf.maxWait}"/>
<property name="timeBetweenLogStatsMillis" 
            value="#{conf.timeBetweenLogStatsMillis}">
</bean>
    
<!-- 利用spring-orm 提供的工厂Bean创建SessionFactory 对象 -->
<bean id="sessionFactory" 
    class="org.springframework.orm.hibernate4.LocalSessionFactoryBean">
<!-- 配置数据源, 使Hibernate能够连接到数据库 -->
<property name="dataSource"ref="dataSource"/>
<!-- 配置Hibernate的工作参数, 包括"SQL方言" -->
<property name="hibernateProperties">
<props>
    <prop key="hibernate.dialect">
    org.hibernate.dialect.MySQLDialect
    </prop>
    <prop key="hibernate.show_sql">
        true
    </prop>
    <prop key="hibernate.format_sql">
        true
    </prop>
</props>
</property>
<!-- 配置"子映射"文件的存储位置 -->
<property name="mappingLocations">
    <value>classpath:hbm/*.xml</value>
</property>
    </bean>
</beans> 
```

* 部署测试

## 整合Spring-Hibernate续
Spring 提供了 HibernateTemplate类 用于封装Session接口, 在Session接口基础之上提供了更加简洁高效的API方法. 使用HibernateTemplate可以替代Sesison接口. 并且更加简洁方便

*案例*
* 在spring-hibernate.xml 中配置HibernateTemplate

```java
<bean id="hibernateTemplate" 
        class="org.springframework.orm.hibernate4.HibernateTemplate">
    <property name="sessionFactory" ref="sessionFactory"/>
</bean>
```

* 添加实体类 User
* 添加映射文件 hbm/User.hbm.xml
* 测试
*使用HibernateTemplate实现UserDao*
* 配置hibernate.xml

```java
<!-- 配置Hibernate事务管理器 -->
<bean id="txMgr" 
    class="org.springframework.orm.hibernate4.HibernateTransactionManager">
    <property name="sessionFactory"
        ref="sessionFactory"/>
</bean>
<!-- 开启基于注解的声明式事务管理 -->
<tx:annotation-driven transaction-manager="txMgr"/>
<context:component-scan base-package="com.dmetal.ssh.dao"/>
```

* 编写userDAO的接口
* 实现UserDAO的接口
* 测试

## 持久对象生存周期管理
Hibernate 为了自动化的处理ORM, 设计了对象持久状态管理。
1. 临时状态: 刚刚创建的新实体对象, 还没有保存到数据库时候, 这是对象是临时状态. 
    - 从Hibernate中删除的对象也是临时状态.
2. 持久状态: 已经保存到数据库的对象, 并且缓存到了session中, 持久状态对象有个非常重要的特点, 在更改属性时候会自动的更新的到数据库中.
    - session.get, session.save, session.update 以后的对象是持久状态的.
    - Hibernate4 需要利用 session.flush 手动执行更新功能
3. 游离状态: 是指持久状态的对象, 被从session缓存中清除, 这时候更新对象的属性不再影响数据库, 可以利用 session.update方法使对象返回到持久状态.
    - session.evict() session.clear() 可将对象从session清除, 使对象变成游离状态.
<img src= "/assets/files/hbm.png" alt="加载错误" title="hibernate生命周期" />

## ValueStack
Struts2 用于共享数据的存储结构. 在整个Struts请求期间共享数据.包含两个区域:
1. 内容区域: 主要共享数据的区域, 控制器Bean就保存在这个区域.
    - 在JSP中, 使用 Struts 标签+OGNL表达式可以读取
    - Struts2 接管了JSTL和EL的底层, JSTL/EL就可以访问这个区域
2. 上下文环境区域: 用于访问request,session和application范围
    - 使用 Struts 标签+OGNL表达式可以读取, 读取时候需要#
    - #request.message  #session.message  #application.message
    - 接管了JSTL和EL的底层, ${requestScope.message}
