---
layout: post
title:  "MySQL实践 Part 2"
date:   2018-09-13 21:58:30 +0200
categories: mysql
---

MySQL 数据库基础学习之二，包括检索进阶(数据处理函数、子查询、联结表等), 插入、删除、更新。

## 检索数据进阶
### 计算字段
当表中存储数据不是应用所需要的，就需要直接从数据库中检索出转换、计算或格式化过的数据。
计算字段不存在于数据库表中，而是在**SELECT**语句内创建的。
```
    // as: 给计算字段起别名
    SELECT prod_id, prod_name, price, quantity, (price * quantity) as total_price
    FROM product;
```

### 数据处理函数
#### 文本处理函数
1. TRIM(), RTRIM(), LTRIM() 去除列值左右空格
```
    SELECT TRIM(prod_name) FROM product;
```
2. UPPER(), LOWER()将文本转为大写或小写
```
    SELECT UPPER(prod_name) FROM product;
```
3. LENGTH() 返回字符串长度
4. LEFT(), RIGHT() 返回字符串左边或右边字符
```
    SELECT LEFT(address, 4) FROM test_01 where id = 6; // 4表示返回左边4个字符
```
5. LOCATE() 返回字符串中的子串位置
```
    SELECT LOCATE('en',address) FROM test_01 where id = 6;
```
6. SUBSTRING() 返回字符串中字串字符
```
    SELECT SUBSTRING('en',1) FROM test_01 where id = 6;
```
#### 日期时间处理函数
1. YEAR(), MONTH(), DAY(), DAYOFWEEK(), HOUR(), MINUTE(), SECOND()
```
    //返回日期的年份
    SELECT YEAR(produce_date) as produce_year FROM product where prod_id = 1;
```
2. DATE(), TIME(), NOW() 返回日期、时间、当前时间
```
    SELECT * FROM product where DATE(produce_date) = '2018-09-13';
```
3. DATEDIFF(), DATE_FORMAT()
```
    //返回两个日期之差
    SELECT DATEDIFF(update_time,create_time) FROM product; //返回的是整数天数
    //格式化日期
    SELECT DATE_FORMAT(NOW(),"%Y-%M-%D")FROM test_01;
```
4. ADDDATE(), ADDTIME() 增加日期或时间
```
    SELECT DATE_ADD(NOW(),INTERVAL 1 MONTH);
```
5. CURDATE(), CURTIME() 返回当前日期或时间
```
    SELECT CURDATE();
```
#### 数值处理函数
1. ABS() 返回绝对值
2. SIN(), COS(), TAN() 返回角度正弦、余弦、正切
3. EXP() 返回 e 的指定数值的次方
```
    SELECT EXP(1); //返回2.718281828459045
```
4. MOD() 返回除操作余数
```
    SELECT MOD(3,2); //返回1
```
5. PI() 返回圆周率
6. RAND() 返回随机数
7. SQRT() 返回平方根

#### 聚集函数
1. AVG() 返回某列平均值
```
    SELECT AVG(price) as avg_price FROM product;
```
2. COUNT() 返回某列行数, 指定列名是，该列值为空自动忽略统计
```
    SELECT COUNT(*) FROM product;
```
3. MAX(), MIN() 返回某列最大值或最小值
```
    SELECT MAX(price) as max_price FROM product;
```
4. SUM() 返回某列之和
```
    SELECT SUM(price * quantity) as total_price FROM product;
```

### 数据分组(GROUP BY)
#### 使用说明
* GROUP BY子句必须在WHERE子句后, ORDER BY子句前
* GROUP BY子句可以再嵌套GROUP BY子句
* GROUP BY子句中的列必须在SELECT语句中给出
* GROUP BY子句会将NULL分为一组
* GROUP BY子句中的列不能是聚集函数, 必须是检索列或有效表达式
#### WITH ROLLUP 得到分组汇总值
```  
    SELECT user_name, count(*) FROM test_01 GROUP BY user_name WITH ROLLUP;
```
#### HAVING子句 过滤分组
 * HAVING 与 WHERE 可替换使用，但唯一差别是前者过滤分组，后者过滤行
```
    SELECT user_name, count(*) FROM test_01 GROUP BY user_name HAVING count(*) > 2;
```
#### 分组并排序
```
    SELECT user_name, count(*) FROM test_01 GROUP BY user_name ORDER BY user_name;
```

### 子查询
* 子查询通常与IN连用
```
    SELECT user_id FROM orders where order_id in (
        SELECT order_id FROM order_items where item_name = ''
    );
```
* 作为计算字段
```
    SELECT user_name, (SELECT COUNT(*) FROM orders where users.user_id = orders.user_id) as total_orders
    FROM users;
```

### 联结表
**联结是一种机制，用来在一条SELECT语句中关联表**
1. 利用WHERE建立简单联结
```
    SELECT user_id, user_name, prod_name
    FROM users, product
    WHERE users.user_id = product.user_id;
```
2. 内联结
上述联结即内联结，也叫等值联结，为了更清楚地表示是那种联结方式，可以使用特定关键字
```
    SELECT user_id, user_name, prod_name
    FROM users INNER JOIN product
    ON users.user_id = product.user_id;
```
3. 自联结
```
    SELECT u1.user_id, u1.user_name
    FROM users AS u1, users AS u2
    WHERE u1.address = u2.address;
```
4. 自然联结
排除多次出现，每个列只返回一次。基本上每一个内部联结都是自然联结
5. 外部联结
联结中包含了那些在相关表中没有关联行的行，分为左外和右外，区别在于关联表的顺序不同
```
    //返回所有购买产品了的用户信息，包括没有购买的
    SELECT user_id, user_name, prod_name
    FROM users LEFT OUT JOIN product
    ON users.user_id = product.user_id;
```
6. 联结表使用别名
联结多个表时，为简化SQL语句，方便阅读，可以给表加上别名。
```
    SELECT user_id, user_name, prod_name
    FROM users AS u INNER JOIN product AS p
    ON u.user_id = p.user_id;
```
7. 带聚合函数的联结
```
    SELECT user_id, user_name, count(product.numb) as num_prod
    FROM users INNER JOIN product
    ON users.user_id = product.user_id;
```

### 组合查询(UNION)
执行多个SELECT语句并将结果作为单个查询结果集返回, 其使用场景如下:
+ 在单个查询中从不同表返回类似结构的数据;
+ 对单个表执行多个查询，按单个查询返回数据;
```
    SELECT user_id, user_name, address
    FROM users
    WHERE user_name = 'nico'
    UNION
    SELECT user_id, user_name, address
    WHERE address in("Shanghai", "Shenzhen");
```
#### UNION注意事项
* UNION中的SELECT语句必须具有相同的列、表达式、聚集函数, 并且列值类型必须兼容
* UNION会自动排除SELECT语句中的重复行,若要全部返回则使用UNION ALL
* 排序时只能有一条OBDER BY子句且在最后一条SELECT语句之后

## 全文本搜索
全文本搜索并非所有数据库引擎都支持，前述的检索都是基于InnoDB引擎，而全文本搜索需要使用MyISAM引擎。
### 全文本搜索与通配符、正则表达式区别
* 性能-通配符和正则往往是匹配所有行,但这些搜索很少使用索引, 随着行数增加, 搜索时间也会变长
* 明确控制-通配符和正则很难明确控制匹配什么而不匹配什么
* 智能化的结果通配符和正则虽然灵活，但不能提供智能化的选择结果的方法
### 使用全文本搜索
在创建表时使用FULLTEXT关键字给指定列添加全文本搜索, 也可以在之后修改表时指定。在定义之后，MySQL会自动维护该索引
```
    SELECT prod_desc
    FROM product
    WHERE MATCH(pro_desc) AGAINST('detail'); //MATCH指定搜索列, AGAINST指定搜索表达式
```
MATCH()可指定多列，必须与FULLTEXT中定义的相同
### 注意事项
* 全文本搜索返回的数据顺序是根据其计算出的等级制降序排列，等级制为0的表示不符合条件不会返回。
* 全文本搜索默认自动忽略长度在3及以下的词
* MySQL默认的非用词会自动忽略
* 50%规则，如果一个词出现在50%以上的行中，则会作为非用词被忽略
* 若表中行数少于3行，不会返回结果
### 查询扩展(WITH QUERY EXPANSION)
即放宽搜索结果，MySQL首先进行全文本搜索与搜索条件匹配的行，然后再从这些匹配行中选择有用词再次进行全文本搜索与搜索条件和有用词匹配的行。因此返回结果被放宽，所有相关结果都会返回。
```
    SELECT prod_desc
    FROM product
    WHERE MATCH(pro_desc) AGAINST('detail' WITH QUERY EXPANSION);
```
### 布尔搜索(IN BOOLEAN MODE)
布尔搜索可以指定要匹配的词，要排斥的词，排列，表达式分组等。布尔搜索即使在没有启用FULLTEXT时也可以使用。
#### 布尔操作符
* \+    包含，词必须存在
* \-    排除，词必须不出现
* \>    包含，且增加等级值
* <     包含，且减少等级制
* ()    把词组成一个子表达式，允许子表达式作为一个组被包含、排除等
* ~     取消一个词的排序值
* \*    词尾通配符
* ""    定义一个短语，匹配包含或排除整个短语
```
    //排除以sort开头词
    SELECT prod_desc
    FROM product
    WHERE MATCH(pro_desc) AGAINST('detail -sort*' IN BOOLEAN MODE);

    //必须匹配2个词，并降低后者等级
    SELECT prod_desc
    FROM product
    WHERE MATCH(pro_desc) AGAINST('+detail +(<sort)'IN BOOLEAN MODE);
```

## 插入数据
### 插入完整行
```
    INSERT INTO users(user_id,user_name,address)
    VALUES(123451, 'nico', 'Shenzhen');
```
### 插入多行
```
    INSERTINSERT INTO users(user_id,user_name,address)
    VALUES(123451, 'nico', 'Shenzhen');
    INSERT INTO users(user_id,user_name,address)
    VALUES(123452, 'lucy', 'Shanghai');
    //或者
    INSERT INTO users(user_id,user_name,address)
    VALUES(123456, 'nico', 'Shenzhen'),(123452, 'lucy', 'Shanghai');
```
### 插入检索出的数据
```
     INSERT INTO users(user_id,user_name,address)
     SELECT user_id,user_name,address
     FROM customs;
```
## 更新数据
```
    UPDATE users
    SET user_name = 'jack', address = 'Shanghai'
    WHERE user_id = 123451;
```
## 删除数据
```
    DELETE FROM users
    WHERE user_id = 123451;
```
* 若要删除表中所有数据，可以使用TRUNCATE TABLE, 速度更快，其实际是删除原来的表并新建表。
* UPDATE、DELETE使用时千万注意WHERE条件是否正确，可先使用SELECT语句进行测试。