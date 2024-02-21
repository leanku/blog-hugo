---
title: "SQL语法速成"
date: 2023-09-01T20:46:01+08:00
draft: false
categories: ["SQL语法速成"]
tags: ["MySQL"]
keywords: ["SQL语法速成"]
---


# SQL语法速成

## SQL 语法结构
* 子句 - 是语句和查询的组成成分。（在某些情况下，这些都是可选的。）
* 表达式 - 可以产生任何标量值，或由列和行的数据库表
* 谓词 - 给需要评估的 SQL 三值逻辑（3VL）（true/false/unknown）或布尔真值指定条件，并限制语句和查询的效果，或改变程序流程。
* 查询 - 基于特定条件检索数据。这是 SQL 的一个重要组成部分。
* 语句 - 可以持久地影响纲要和数据，也可以控制数据库事务、程序流程、连接、会话或诊断。

## SQL 语法要点
* SQL 语句不区分大小写，但是数据库表名、列名和值是否区分，依赖于具体的 DBMS 以及配置。
    例如：SELECT 与 select 、Select 是相同的。
* 多条 SQL 语句必须以分号 ; 分隔。
* 处理 SQL 语句时，所有空格都被忽略。SQL 语句可以写成一行，也可以分写为多行。
* SQL 支持三种注释
   ``` mysql
   ## 注释1
   -- 注释2
   /* 注释3 */
   ```

## 增删改查，又称为 CRUD，数据库基本操作中的基本操作。

### 插入数据
**INSERT INTO**  语句用于向表中插入新记录。

插入完整的行
```mysql
INSERT INTO user VALUES (10, 'root', 'root', 'xxxx@163.com');
```

插入行的一部分
```mysql
INSERT INTO user(username, password, email)VALUES ('admin', 'admin', 'xxxx@163.com');
```

插入查询出来的数据

```mysql
INSERT INTO user(username) SELECT nameFROM account;
```

### 更新数据
**UPDATE** 语句用于更新表中的记录。
```mysql
UPDATE userSET username='robot', password='robot'WHERE username = 'root';
```

### 删除数据
***DELETE*** 语句用于删除表中的记录。

***TRUNCATE TABLE*** 可以清空表，也就是删除所有行。

删除表中的指定数据
```mysql
DELETE FROM userWHERE username = 'robot';
```

清空表中的数据
```mysql
TRUNCATE TABLE user;
```

## 查询数据

***SELECT*** 语句用于从数据库中查询数据。

***DISTINCT*** 用于返回唯一不同的值。它作用于所有列，也就是说所有列的值都相同才算相同。

***LIMIT*** 限制返回的行数。可以有两个参数，第一个参数为起始行，从 0 开始；第二个参数为返回的总行数。

***ASC*** ：升序（默认）

***DESC*** ：降序

查询指定列
```mysql
SELECT prod_id, prod_name, prod_price FROM products;

# *表示查询所有字段
SELECT * FROM products;
```

查询不同的值
```mysql
SELECT DISTINCTvend_id FROM products;
```

限制查询结果
```mysql
-- 返回前 5 行
SELECT * FROM mytable LIMIT 5;
SELECT * FROM mytable LIMIT 0, 5;

-- 返回第 3 ~ 5 行
SELECT * FROM mytable LIMIT 2, 3;
```

## 子查询
子查询是嵌套在较大查询中的 SQL 查询。子查询也称为内部查询或内部选择，而包含子查询的语句也称为外部查询或外部选择。

* 子查询可以嵌套在 SELECT，INSERT，UPDATE 或 DELETE 语句内或另一个子查询中。
* 子查询通常会在另一个 SELECT 语句的 WHERE 子句中添加。
* 可以使用比较运算符，如 >，<，或 =。比较运算符也可以是多行运算符，如 IN，ANY 或 ALL。
* 子查询必须被圆括号 () 括起来。
* 内部查询首先在其父查询之前执行，以便可以将内部查询的结果传递给外部查询。
  
子查询的子查询
```mysql
SELECT cust_name, cust_contactFROM customersWHERE cust_id IN (
   SELECT cust_id FROM orders WHERE order_num IN (
      SELECT order_num FROM orderitems WHERE prod_id = 'RGAN01'
   )
);
```

### WHERE
* WHERE 子句用于过滤记录，即缩小访问数据的范围。
* WHERE 后跟一个返回 true 或 false 的条件。
* WHERE 可以与 SELECT，UPDATE 和 DELETE 一起使用。
* 可以在 WHERE 子句中使用的操作符

|运算符|描述|
|:--|:--|
|= |等于|
|<>|不等于。注释：在 SQL 的一些版本中，该操作符可被写成！=|
|>|大于|
|< |小于|
|>= |大于等于|
|<=|小于等于|
|LIKE |搜索某种模式|
|IN |指定针对某个列的多个可能值|

SELECT 语句中的 WHERE 子句

```mysql 
SELECT * FROM CustomersWHERE cust_name = 'Kids Place';
```

UPDATE 语句中的 WHERE 子句

```mysql
UPDATE CustomersSET cust_name = 'Jack Jones'WHERE cust_name = 'Kids Place';
```
DELETE 语句中的 WHERE 子句

```mysql
DELETE FROM CustomersWHERE cust_name = 'Kids Place';
```
IN 和 BETWEEN

   * IN 操作符在 WHERE 子句中使用，作用是在指定的几个特定值中任选一个值。

   * BETWEEN 操作符在 WHERE 子句中使用，作用是选取介于某个范围内的值。
  
      IN 示例
      ```mysql
      SELECT *FROM productsWHERE vend_id IN ('DLL01', 'BRS01');
      ```

      BETWEEN 示例
      ```mysql
      SELECT *FROM productsWHERE prod_price BETWEEN 3 AND 5;
      ```
AND、OR、NOT
   * AND、OR、NOT 是用于对过滤条件的逻辑处理指令。
   * AND 优先级高于 OR，为了明确处理顺序，可以使用 ()。
   * AND 操作符表示左右条件都要满足。
   * OR 操作符表示左右条件满足任意一个即可。
   * NOT 操作符用于否定一个条件。

      AND 示例
      ```mysql
      SELECT prod_id, prod_name, prod_priceFROM productsWHERE vend_id = 'DLL01' AND prod_price <= 4;
      ```

      OR 示例
      ```mysql
      SELECT prod_id, prod_name, prod_priceFROM productsWHERE vend_id = 'DLL01' OR vend_id = 'BRS01';
      ```

      NOT 示例
      ```mysql
      SELECT *FROM productsWHERE prod_price NOT BETWEEN 3 AND 5;
      ```