---
layout: post
title:  "JDBC连接数据库"
date:   2018-03-25 08:27:30 +0200
categories: database jdbc
---

JDBC——Java数据库连接。JDBC是一种标准Java应用编程接口(JAVA API), 用来连接Java编程语言和广泛的数据库。可以通过JDBC代码实现对数据库的操作,包括DDL，DML，DQL等。

## JDBC 接口
1. DriverManager 驱动管理
2. Connection    连接接口
3. Statement     语句对象接口
4. ResultSet     结果集接口
## 连接步骤
1. 加载类库（驱动jar包）到JVM
```
Class.forName("com.mysql.jdbc.Driver");
```
2. 通过DriverManager建立连接，这里会加载jar包中JDBC实现类与数据库建立连接
```
Connection conn = DriverManager.getConnection(url, username, password);
```
3. 通过Connection创建Statement对象
```
Statement state=conn.createStatement();
```
4. 通过Statement执行SQL语句
```
String sql="";
state.execute(sql);
```
5. 若执行的是DQL语句，则会得到查询结果集Result,遍历该结果集得到查询内容 
6. 关闭连接
```
state.close();
connection.close();
```
## DriverManager 类
DriverManager类下面提供的全是静态方法, 只需要用类名点方法就可以调用,不需要实例化.
1. 注册驱动： registerDriver();
2. 获取链接： getConnection(url, username, password);
## Connection 类
connection类主要有两个作用：**1.可以获取执行sql的对象，2.管理事务的功能**。数据库里面的数据一旦改变就是永久的，所以需要一个管理事务功能验证sql语句的正确性。在mysql中事务管理通过begin开启事务，rollback回滚事务，commit提交事务。在jdbc里面通过connection类的特定方法也可以实现这三个功能。
## Statement 类
* Statement用来执行SQL语句，但只适合执行静态sql语句。 即SQL语句中不含有拼接动态数据的地方。因为拼接SQL语句会导致SQL语句的复杂度提高，并且可能出现SQL注入攻击。
* PreparedStatement是Statement的子类，专门用来解决上述问题，PreparedStatement适合执行含有动态信息的SQL语句其执行的是预编译SQL语句。
1. int executeUpdate(String sql)专门用来执行DML语句的方法，返回值是一个整数用来表示执行了DML语句后影响了表中多少条数据
2. ResultSet executeQuery(String sql)专门用来指定DQL语句的方法，返回值为查询结果集
3. booleam execute(String sql) 什么SQL语句都可以指定行，但由于DML,DQL有专门方法，所以一般用来执行DDL语句，返回值为true时说明执行后有返回值。
## ResultSet 类
对DQL查询语句进行封装，返回一个resultset对象，并且可以通过特定方法获取查询结果。并用特定方法获取查询结果内容。
