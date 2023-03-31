---
layout: post
title:  "MySQL实践 Part 1"
date:   2018-09-11 21:58:30 +0200
categories: database mysql
---

MySQL 数据库基础学习之一，包括数据库配置、数据检索基础(SELECT语句、WHERE语句、通配符等)

## MySQL简介
MySQL是一种DBMS，即数据库管理软件(而不是数据库本身)。MySQL作为开源软件，其成本低、性能高，并且简单易用，大多数企业均使用MySQL处理其重要数据。

## MySQL使用
### 安装[MySQL](https://dev.mysql.com/downloads/mysql/)
### 配置MySQL-根目录下新建my.ini文件并编辑以下信息
```
    [mysql]
    //默认字符集
    default-character-set=utf8
    [mysqld]
    //默认端口
    port=3306
    //mysql目录
    basedir=
    //mysql数据存放目录
    datadir=
    //默认最大连接数
    max_connections=200
    //服务端默认字符集
    character-set-server=utf8
    //默认存储引擎
    default-storage-engine=INNODB
```

### 初始化

```
    //初始化
    mysqld --initialize --console
    //安装服务
    mysqld --install mysql
    //启动
    net start mysql
    //登录
    mysql -u root -p
    //输入初始密码
    //修改密码
    ALTER USER USER() IDENTIFIED BY 'newPaw';
```
### 基本命令
```
    //返回可用数据库列表(包括MySQL内部数据库)
    SHOW DATABASES;
    //使用数据库
    USE 'dataname';
    //返回当前数据库内可用表
    SHOW TABLES;
    //返回所选择表的字段信息
    SHOW COLUMNS FROM 'tablename';
    or
    DESCRIBE 'tablename';
```
## 检索数据
### 简单SELECT语句
```
    //返回表中所有数据
    SELECT * FROM 'tablename';
    //返回特定字段的所有数据
    SELECT 'id', 'username', ··· FROM 'tablename';
    //返回唯一不同数据，即去重
    SELECT DISTINCT 'name', 'address' FROM 'tablename'; //DISTINCT关键字应用于所选择的全部列
```
### LIMIT关键字(限定行数)
```
    //返回前n行
    SELECT * FROM 'tablename' LIMIT 3;
    //返回从第n行开始的n行
    SELECT * FROM 'tablename' LIMIT 3,3;
```
### ORDER BY子句(排序)
```
    //返回排序的数据
    SELECT * FROM 'tablename' ORDER BY 'id';  //默认升序 ASC
    //返回多列排序的数据
    SELECT * FROM 'tablename' ORDER BY 'id', 'username'; //先按id排序,再按username排序,若id都是唯一的,则不会再按后者排序
    //降序
    SELECT * FROM 'tablename' ORDER BY 'id' DESC, 'username'; //DESC只作用于其前面的列
    //返回按某字段排序后的最大或最小值
     SELECT * FROM 'tablename' ORDER BY 'id' LIMIT 1;
     SELECT * FROM 'tablename' ORDER BY 'id' DESC LIMIT 1;
```
### WHERE子句(过滤)
```
    //返回id=1的数据
     SELECT * FROM 'tablename' WHERE 'id' = 1; //操作符包括 =, <, >, <=, >=,  <>, !=, BETWEEN
    //返回范围数据
     SELECT * FROM 'tablename' WHERE 'id' BETWEEN 1 AND 5;
    //检查空值
     SELECT * FROM 'tablename' WHERE 'address' IS NULL; //空值与0, 空字符串, 空格不同
    // 逻辑操作符AND|OR, AND优先级更高，在多个AND或OR中需要用圆括号以保证计算次序，消除歧义
    SELECT * FROM 'tablename' WHERE 'user_name' = "" AND(OR) 'address' = "";
    //IN操作符,相当于OR,但比OR更灵活直观，并且可嵌套SELECT语句
    SELECT * FROM 'tablename' WHERE 'id' IN (1,2,3);
    //NOT操作符, 可以对IN|BETWEEN|EXISITS子句取反
    SELECT * FROM 'tablename' WHERE 'id' NOT IN (1,2,3);
```
#### LIKE操作符与通配符
``` 
    //LIKE指示MySQL WHERE子句后跟的搜索模式利用通配符匹配而不是直接相等匹配
    // % 表示任何字符出现任意次数, %可以在任意位置使用
    SELECT * FROM 'tablename' WHERE 'user_name' LIKE '%jack'; ('%jack%'|'jack%')
    // _ 表示任何字符出现1次， 即只匹配一个1个字符
    SELECT * FROM 'tablename' WHERE 'user_name' LIKE '_jack';
```
* 不要过度使用通配符
* 除非必要否则不要将通配符置于搜索条件开始处

#### 正则表达式
* LIKE和REGEXP区别: LIKE是匹配整个列值，REGEXP则是在列值内进行匹配，例如： *LIKE '1000'是匹配列值为1000的行, 而REGEXP '1000则是匹配列值内包含1000的行， c1000也可以匹配上'*
```
    //用 | 进行OR 匹配
    SELECT * FROM 'tablename' WHERE 'user_name' REGEXP 'jack|nico';
    //用 [] 进行特定字符集匹配
    SELECT * FROM 'tablename' WHERE 'user_name' REGEXP 'jack[123]'; //jack[123]= jack[1|2|3], 匹配jack1, jack2, jack3
    //字符集取反
    SELECT * FROM 'tablename' WHERE 'user_name' REGEXP 'jack[^123]';
    //匹配范围
    SELECT * FROM 'tablename' WHERE 'user_name' REGEXP 'jack[1-9]'; //[a-z]
    //匹配诸如 . | - [] \ 元字符等特殊字符，需要使用\\转义
    SELECT * FROM 'tablename' WHERE 'user_name' REGEXP 'jack\\.';
    //定位符使用
    // ^ 表示文本的开始; $ 表示文本的结尾; [[:<:]] 表示词的开始; [[:>:]] 表示词的结尾
    // 注意当 ^ 放在[]中时表示集合取反
    SELECT * FROM 'tablename' WHERE 'user_name' REGEXP '^[0-9\\.]'; //匹配0-9或.开头的值
```
* 预定义字符集
    + [:alnum:]     任意字母或数字      [a-zA-Z0-9]
    + [:alpha:]     任意字符            [a-zA-Z]
    + [:blank:]     空格和制表          [\\t] 
    + [:cntrl:]     ASCII控制字符       ASCII 0到31和127
    + [:digit:]     任意数字            [0-9]
    + [:graph:]     任意可打印字符, 空格除外
    + [:lower:]     任意小写字母        [a-z]
    + [:print:]     任意可打印字符
    + [:punct:]     既不在[:alnum:]也不在[:cntrl:]中的字符
    + [:space:]     任意空白字符,包括空格 [\\f\\n\\r\\t\\v]
    + [:upper:]     任意大写字母        [A-Z]
    + [:xdigit:]    任意16进制数字      [a-fA-F0-9]
* 重复元字符
    + \*            0或多个匹配          
    + \+            1或多个匹配
    + ?             0或1个匹配
    + {n}           指定数目匹配
    + {n,}          不少于指定数目的匹配
    + {n,m}         匹配数目的范围， mmm<=255

```
    //匹配连续4个数字
    SELECT * FROM 'tablename' WHERE 'user_name' REGEXP '[[:digit:]]{4}';
    //匹配jack 或jacks
     SELECT * FROM 'tablename' WHERE 'user_name' REGEXP 'jacks?'; //匹配?前s字符0或1次
```