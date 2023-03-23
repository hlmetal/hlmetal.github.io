---
layout: post
title:  "MyBatis的基本实践"
date:   2018-04-30 08:20:30 +0200
categories: database jdbc
---

MyBatis是一个基于Java的持久层框架。支持定制化 SQL、存储过程以及高级映射。MyBatis 避免了几乎所有的JDBC代码和手动设置参数以及获取结果集。MyBatis可以使用简单的XML或注解来配置和映射原生信息，将接口和Java的POJOs映射成数据库中的记录。

## MyBatis发展
apache iBatis -> google MyBatis -> Github

## MyBatis、jdbc、Hibernate区别
* jdbc——速度快，易掌握，要写sql，代码繁琐
* mybatis——速度适中，易掌握，要写sql，代码简洁
* hibernate——速度慢，比较难掌握，不用写sql，代码简洁；业务复杂时，经常需要优化sql

## 编程步骤
1. 导包：mybatis,ojdbc,junit
2. 添加mybatis配置文件
3. 实体类：属性名要与表的字段名一样（大小写可忽略）
4. 添加映射文件，写sql
5. 修改配置文件，指定映射文件的位置
7. 调用MyBatis的api来访问数据库

## 原理
<img src= "/assets/files/mybatis_principle.png" alt="加载错误" title="MyBatis原理" />

## Mapper映射器
1. Mapper映射器是一个符合映射文件要求的接口,MyBatis会生成一个符合该接口要求的对象
2. 步骤:
    * 写一个接口,其方法名要与映射文件当中的sqlId一样,参数类型要与映射文件当中parameterType一样,返回类型要与映射文件当中的resultType一样.
    * 修改映射文件,将namespace设置为接口名
    * 调用SqlSession对象的方法getMapper来获得映射器的实现
3. 获得Map类型的结果: MyBatis在查询时,会将记录中的数据先存放到一个Map对象上面(以字段名作为key，以字段值作为value),然后再将Map对象中的数据添加到实体对象上。获得Map类型的结果,指的是获得这个Map对象。实际使用中建议还是获得实体对象,这样获得数据更方便
4. 解决表字段名与实体属性名不一致的情况: 使用别名,将别名设置成与属性名一致或者使用resultMap

## Spring集成MyBatis
1. 集成方式一(使用Mapper映射器)
    * 导包：srping-webmvc，mybatis，mybatis-spring，ojdbc，dbcp，spring-jdbc，junit
    * 添加spring配置文件, 不在需要mybatis的配置文件，mybatis相关的配置用一个bean代替
    * 配置sqlSessionFactoryBean
    * 实体类
    * 映射文件
    * Mapper映射器(接口)
    * 配置MapperScannerConfigurer: 会扫描指定包及其子包下面的所有Mapper映射器，然后调用SqlSession的getMapper方法(该方法会返回符合Mapper映射器要求的对象),并将这些对象添加到spring容器中(默认id是首字母小写后的接口名)
2. 集成方式二(不使用Mapper映射器)
    * 导包：srping-webmvc，mybatis，mybatis-spring，ojdbc，dbcp，spring-jdbc，junit
    * 添加spring配置文件,不在需要mybatis的配置文件，mybatis相关的配置用一个bean代替
    * 配置sqlSessionFactoryBean
    * 实体类
    * 映射文件：namespace不要求与映射器名字一样
    * DAO接口：不要求与映射文件一致
    * 写DAO实现类：注入SqlSessionTemplate，它封装了对SqlSession的操作