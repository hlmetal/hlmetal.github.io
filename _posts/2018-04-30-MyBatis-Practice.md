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


## 数据分页
1. 逻辑分页: 先查询出所有的数据缓存到内存, 再根据业务相关需求,从内存数据中筛选出合适的数据
进行分页
2. 物理分页: 直接利用数据库支持的分页语法来实现,比如Mysql里面提供了分页关键词Limit

### Mybatis分页
1. 在Mybatis Mapper配置文件里面直接写分页SQL,这种方式比较灵活,实现简单
2. RowBounds实现逻辑分页,一次性加载所有符合查询条件的目标数据,根据分页参数值在内存中实现分页
3. Interceptor拦截器实现, 通过拦截需要分页的select语句, 然后在这个sql语句里面动态拼接分页关
键字,从而实现分页查询
4. 插件(PageHelper)等, 本质上也是使用Mybatis的拦截器来实现的

## ${}与#{}区别
都是实现动态SQL的方式, 通过这两种方式把参数传递到XML之后,在执行操作之前,Mybatis会对这两种占位符进行动态解析
1. $号相当于直接把参数拼接到了原始的SQL里面, MyBatis不会对它进行特殊处理, 即动态参数, 不能防止SQL注入
2. #号等同于JDBC里面的?号, 即占位符. 相当于向PreparedStatement预处理语句中设置参数, SQL是预编译的, 如果在设置的参数包含特殊字符,就会自动进行转义. 所以#号占位符可以防止SQL注入

## 二级缓存
提升数据的检索效率, 避免每次数据的访问都需要去查询数据库
1. 一级缓存: 它是SqlSession级别的缓存, 也叫本地缓存(SqlSession中的Executor对象有一个LocalCache)
* 每个用户在执行查询的时候都需要使用SqlSession来执行
* 为了避免每次都去查数据库, MyBatis把查询出来的数据保存到SqlSession的本地缓存中
* 后续的SQL如果命中缓存, 就可以直接从本地缓存读取
2. 二级缓存: 跨SqlSession级别的缓存, 是个全局缓存(对Executor进行了装饰CachingExecutor)
* 多个用户在查询数据的时候, 只要有任何一个SqlSession拿到了数据就会放入到二级缓存里面, 其他的
SqlSession就可以从二级缓存加载数据
* 在进入一级缓存之前, 会先通过CachingExecutor进行二级缓存的查询
* 二级缓存缓存没有再查一级缓存. 最后则是数据库


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