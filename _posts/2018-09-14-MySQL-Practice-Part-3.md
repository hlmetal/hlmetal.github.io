---
layout: post
title:  "MySQL实践 Part 3"
date:   2018-09-14 09:00:30 +0200
categories: mysql
---

## 创建和操作表
### 创建表
* 创建表时各字段必须给出数据类型
* 若字段不允许为空则必须指定NOT NULL,否则默认允许为空NULL
* 主键必须唯一，一般以单列作为主键， 若多个列组合则，则组合必须唯一
* AUTO_INCREMENT自增量，每个表只允许一个自增列，且必须被索引， 自增值可以MySQL默认，也可以设定特殊值，后续增量将从此特殊值开始。
* 创建表时各字段可以指定默认值, 只支持常量
* 创建表时可根据需要选择使用引擎
```
    CREATE TABLE users(
        user_id bigint  NOT NULL AUTO_INCREMENT,
        user_name varchar(50)   NOT NULL DEFAULT 'user000', 
        address varchar(50),
        PRIMARY KEY(user_id)
        )ENGINE=InnoDB;

```
### 更新表
在创建表时需要仔细设计表，以免插入数据后再对表做较大改动。、
```
    //添加列
    ALTER TABLE users
    ADD phone int DEFAULT 0;
    //删除列
    ALTER TABLE users
    DROP CLOUMN phone;
    //添加外键
    ALTER TABLE orders
    ADD CONSTRAINT fk_orders_users
    FOREIGN KEY(user_id)
    REFERENCES orders(user_id);
```
### 删除表
```
    DELETE TABLE users;
```
### 修改表名
```
    RENAME TABLE users TO users2;
```

## 视图
### 什么是视图
视图是虚拟表，与包含数据的表不同，它只包含使用时动态检索数据的查询。
### 为什么使用视图
* 重用SQL语句
* 简化复杂SQL操作
* 使用表的组成部分而不是整个表
* 保护数据，授权给用户部分表数据的访问权限而不是全部
* 更改数据格式和表示，视图可以返回与实体表的表示和格式不同的数据
### 使用规则
* 视图名必须唯一
* 视图可以嵌套
* 视图不能索引，也没有关联的触发器或默认值
* 视图可以和表一起用
* 视图可以使用ORDER BY
### 创建视图
```
    // 创建一个查询用户及其购买的产品的视图
    CREATE VIEW user_prods AS
    SELECT user_name, prod_name
    FROM users AS u, orders AS o, products AS p
    WHERE u.user_id = o.user_id AND o.prod_id = p.prod_id

    SELECT * FROM user_prods;

    //格式化检索出来的数据
    CREATE VIEW user_address AS
    SELECT CONCAT(TRIM(user_name), '(', CONCAT(TRIM(address)), ')')
    FROM users
    ORDER BY user_name;

    SELECT * FROM user_address;
```
### 更新视图
视图可以更新，但视图一般都应用与检索数据，而不用于更新(INSERT、UPDATE、DELETE)。

## 存储过程
### 什么是存储过程
存储过程简单来说就是为以后的使用而保存的多条SQL的集合，类似于批处理函数，但又不限于批处理。
### 为什么使用存储过程
* 简单-将处理步骤进行封装，简化复杂操作
* 安全-使用同一存储过程，不必反复建立一系列处理步骤，防止错误，保证了数据完整性
* 高性能-使用存储过程比使用单独SQL要快
### 创建并使用存储过程
```
    //创建一个简单存储过程
    CREATE PROCEDURE avgprice()
    BEGIN
        SELECT AVG(price) AS priceaverage
        FROM products;
    END;
    //使用存储过程
    CALL avgprice;
```
### 创建含参数的存储过程
```
    //OUT 表示的参数是从存储过程中传出的值, DECIMAL表示参数数据类型为十进制
    //IN 表示的参数是传入存储过程的值, 一般用于筛选条件
    CREATE PROCEDURE productpricing(
        IN pid INT,
        OUT pmin DECIMAL(8,2),
        OUT pmax DECIMAL(8,2),
        OUT pavg DECIMAL(8,2)
    )
    BEGIN
        SELECT MIN(price) INTO pmin FROM products WHERE prod_id = pid;
        SELECT MAX(price) INTO pmax FROM products WHERE prod_id = pid;
        SELECT AVG(price) INTO pavg FROM products WHERE prod_id = pid;
    END;

    //使用存储过程
    CALL productpricing(1, @pmin, @pmax, @pavg); //不显示任何数据,返回以后可以显示的变量
    SELECT @pmin, @pmax, @pavg; //显示变量值
```
### 创建智能存储过程
```
    //创建订单总价存储过程，并且有条件的把营业税加入总价
    CREATE PROCEDURE orderprice(
        IN oid INT,
        IN taxable BOOLEAN,
        OUT op DECIMAL(8,2)
    )
    BEGIN
        DECLARE total DECIMAL(8,2); // 局部变量
        DECLARE taxrate INT DEFAULT 5;

        SELECT SUM(price * quantity)
        FROM orders
        WHERE order_id = oid
        INTO total;

        IF taxable THEN
            SELECT total + (total/100*taxrate) INTO total;
        END IF;

        SELECT total INTO ototal;

    //使用
    CALL orderprice(100000111, 1, @ototal);//1为真,0为假
    SELECT @ototal;
```

## 游标
### 什么是游标
游标是存储在MySQL服务器上的数据库查询，它是SELECT检索出来的结果集。存储游标后，可以根据需要滚动或浏览其中数据。有时需要在检索出来的数据中前进或后退一行或多行时就要使用游标。
### 游标的使用
* MySQL的游标只能用于存储过程
* 使用前必须声明，声明后，必须打开游标以供使用，结束使用后，必须关闭游标
#### 创建游标
```
    CREATE PROCEDURE allorders()
    BEGIN
        DECLARE ordernumbers CURSOR
        FOR
        SELECT order_num FROM orders;
    END;
```
#### 打开和关闭游标
```
    OPEN ordernumbers;
    CLOSE ordernumbers;
```
#### 使用游标数据
```
    CREATE PROCEDURE allorders()
    BEGIN
        //定义变量
        DECLARE done BOOLEAN DEFAULT 0;
        DECLARE o INT;
        //定义游标
        DECLARE ordernumbers CURSOR
        FOR
        SELECT order_num FROM orders;
        //定义循环结束句柄-条件出现时被执行，即SQLSTATE '02000'出现时done设置为1
        //'02000'是一个未找到条件，在REPEAT没有行循环时出现
        DECLARE CONTINUE HANDLER FOR SQLSTATE '02000' SET done=1;

        OPEN ordernumbers;
        //循环检索数据
        REPEAT
            FETCH ordernumbers INTO o; //检索当前行的order_num列到变量o中
        UNTIL done END REPEAT;

        CLOSE ordernumbers;
    END;
```
**DECLARE语句次序：定义变量必须在定义游标前，定义句柄必须在定义游标后**

## 触发器
触发器是响应INSERT、UPDATE、DELETE语句而自动执行的一条语句或位于BEGIN/END之间的一组语句。
### 创建触发器
* 触发器仅支持表，不支持视图
* 唯一的触发器名(最好保持数据库范围内唯一)
* 触发器关联的表
* 触发器响应的活动(INSERT、UPDATE、DELETE)
* 触发器何时执行(处理前还是处理后)
* 每个表最多6个触发器，即INSERT、UPDATE、DELETE之前或之后
```
    CREATE TRIGGER newproduct AFTER INSERT ON products //AFTER INSERT指插入成功后触发
    FOR EACH ROW SELECT 'product added'; //FOR EACH ROW表示对每行插入执行
```
### 使用触发器
#### INSERT触发器
* INSERT触发器内，可以引用一个名为NEW的虚拟表，以访问被插入的行
* BEFORE INSERT中，NEW中的值也可以被更新
* 对于AUTO_INCREMENT列，在INSERT前，NEW中值为0，INSERT后为自动生成值
```
    CREATE TRIGGER newproduct AFTER INSERT ON products
    FOR EACH ROW SELECT NEW.prod_id;
```
#### UPDATE触发器
* UPDATE触发器内，可以引用一个名为OLD的虚拟表访问UPDATE前的值，引用一个名为NEW的虚拟表访问UPDATE后的值
* BEFORE UPDATE中，NEW中的值也可以被更新
* OLD中的值只能读，不能更新
```
    //保证更新的产品名为大写
    CREATE TRIGGER updateproduct BEFORE UPDATE ON products
    FOR EACH ROW SET NEW.prod_name = UPPER(NEW.prod_name);
```
#### DELETE触发器
* DELETE触发器内，可以引用一个名为OLD的虚拟表访问DELETE的行
* OLD中的值只能读，不能更新
```
    //在删除前将删除数据归档
    CREATE TRIGGER deleteproduct BEFORE DELETE ON products
    FOR EACH ROW 
    BEGIN
        INSERT INTO archive_products(prod_id,prod_name)
        VALUES(OLD.prod_id, OLD.prod_name);
    END;
```

## 事务处理
事务处理是一种机制，保证一组操作不会中途停止，即保证必须成批SQL语句要么完全执行，要么完全不执行，以维护数据库的完整性。
### 事务处理关键术语
* 事务(transaction) 指一组SQL语句
* 回退(rollback) 指撤销指定SQL语句的过程
* 提交(commit)  指将未存储的SQL语句结果写入数据库表
* 保留点(savepoint) 指事务处理中设置的临时占位符，以对其发布回退操作
### 启用事务处理
```
    //事务开始
    START TRANSACTION;
```
### 使用ROLLBACK
```
    SELECT * FROM products;
    START TRANSACTION;
    DELETE FROM products;
    SELECT * FROM products;
    ROLLBACK; //回退START TRANSACTION后的所有SQL语句
    SELECT * FROM products;
```
### 使用commit
```
    START TRANSACTION;
    DELETE FROM products WHERE order_id = 10011;
    DELETE FROM orders WHERE order_id = 10011;
    COMMIT;//任一DELETE语句失败，都不会提交
```
### 使用保留点
```
    START TRANSACTION;
    INSERT INTO products(prod_id, prod_name) VALUES(10011, 'rr');
    SAVEPOINT delete1; //保留点在事务完成后自动释放，也可用RELEASE明确释放
    DELETE FROM products WHERE order_id = 10011;
    ROLLBACK delete1;
```

## 安全管理
### 管理用户
#### 查看用户
```
    USE mysql;
    SELECT user FROM user;
```
#### 创建用户
```
    CREATE USER 'username' IDENTIFIED BY 'password';
    // 重命名
    RENAME USER 'username' TO 'newusername';
```
#### 更改口令
```
    SET PASSWORD FOR 'username' = PASSWORD('newpassword');
```
#### 删除用户及权限
```
    DROP USER 'username';
```
#### 设置访问权限
GRANT和REVOKE控制权限层次:
* 整个服务器, 使用 GRANT ALL和REVOKE ALL
* 整个数据库, 使用 ON database.*
* 特定表, 使用 ON database.table
* 特定列、特定存储过程
```
    //查看用户权限
    SHOW GRANTS FOR 'username';
    //授予指定数据库下所有表的访问,修改权限
    GRANT SELECT, INSERT, UPDATE, DELETE ON databasename.* TO 'username@locakhost';
    //撤销权限
    REVOKE INSERT, UPDATE, DELETE ON databasename.* FROM'username@localhost';

    // 更新root用户主机名， %表示可以访问所有主机
    UPDATE user SET host="%" WHERE user = 'root';
    // 授予所有权限
    GRANT ALL PRIVILEGES ON *.* TO root@'%';
    //刷新权限
    FLUSH PRIVILEGES;
```

