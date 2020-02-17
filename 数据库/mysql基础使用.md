## 基础

### linux下安装/配置/卸载

**安装**

```
sudo apt-get install mysql-server
sudo apt-get install mysql-client
```

 **重启/打开/关闭 MySQL**  

```
sudo service mysql restart/start/stop
```

**初始进入**

初始登录时查看一个文件

```
sudo cat /etc/mysql/debian.cnf
```

在这个文件里面有着MySQL默认的用户名和用户密码， 最最重要的是：用户名默认的不是root，而是debian-sys-maint

 记下这里的 ***user*** 和 ***password***，然后到终端里输入 `mysql -u debian-sys-maint -p`，随即会让我们输入密码，此时输入我们刚才记下的密码即可进入 mysql 的shell环境了。 

**创建新角色**

```
CREATE USER 'username'@'host' IDENTIFIED BY 'password';
```

* username：你将创建的用户名

* host：指定该用户在哪个主机上可以登陆，如果是本地用户可用localhost，如果想让该用户可以**从任意远程主机登陆**，可以使用通配符`%`

* password：该用户的登陆密码，密码可以为空，如果为空则该用户可以不需要密码登陆服务器

**给角色授权**

```
GRANT privileges ON databasename.tablename TO 'username'@'host'
```

- privileges：用户的操作权限，如`SELECT`，`INSERT`，`UPDATE`等，如果要授予所的权限则使用`ALL`
- databasename：数据库名
- tablename：表名，如果要授予该用户对所有数据库和表的相应操作权限则可用`*`表示，如`*.*`



 https://blog.csdn.net/w3045872817/article/details/77334886 

 首先用dpkg --list|grep mysql查看自己的mysql有哪些依赖 

先卸载sudo apt-get remove mysql-common

然后：sudo apt-get autoremove --purge mysql-server-5.0 

再用dpkg --list|grep mysql查看，还剩什么就卸载什么

最后清楚残留数据就可以了：

dpkg -l |grep ^rc|awk '{print $2}' |sudo xargs dpkg -P




关闭Mysql  net stop mysql

### 数据类型

无符号在前面加 unsigned

| 类型         | 大小                                     | 范围（有符号）                                               | 范围（无符号）                                               | 用途            |
| :----------- | :--------------------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- | :-------------- |
| TINYINT      | 1 字节                                   | (-128，127)                                                  | (0，255)                                                     | 小整数值        |
| SMALLINT     | 2 字节                                   | (-32 768，32 767)                                            | (0，65 535)                                                  | 大整数值        |
| MEDIUMINT    | 3 字节                                   | (-8 388 608，8 388 607)                                      | (0，16 777 215)                                              | 大整数值        |
| INT或INTEGER | 4 字节                                   | (-2 147 483 648，2 147 483 647)                              | (0，4 294 967 295)                                           | 大整数值        |
| BIGINT       | 8 字节                                   | (-9,223,372,036,854,775,808，9 223 372 036 854 775 807)      | (0，18 446 744 073 709 551 615)                              | 极大整数值      |
| FLOAT        | 4 字节                                   | (-3.402 823 466 E+38，-1.175 494 351 E-38)，0，(1.175 494 351 E-38，3.402 823 466 351 E+38) | 0，(1.175 494 351 E-38，3.402 823 466 E+38)                  | 单精度 浮点数值 |
| DOUBLE       | 8 字节                                   | (-1.797 693 134 862 315 7 E+308，-2.225 073 858 507 201 4 E-308)，0，(2.225 073 858 507 201 4 E-308，1.797 693 134 862 315 7 E+308) | 0，(2.225 073 858 507 201 4 E-308，1.797 693 134 862 315 7 E+308) | 双精度 浮点数值 |
| DECIMAL      | 对DECIMAL(M,D) ，如果M>D，为M+2否则为D+2 | 依赖于M和D的值                                               | 依赖于M和D的值                                               | 小数值          |

**日期和时间类型**

表示时间值的日期和时间类型为DATETIME、DATE、TIMESTAMP、TIME和YEAR。

每个时间类型有一个有效值范围和一个"零"值，当指定不合法的MySQL不能表示的值时使用"零"值。

TIMESTAMP类型有专有的自动更新特性，将在后面描述。

| 类型      | 大小 (字节) | 范围                                                         | 格式                | 用途                     |
| :-------- | :---------- | :----------------------------------------------------------- | :------------------ | :----------------------- |
| DATE      | 3           | 1000-01-01/9999-12-31                                        | YYYY-MM-DD          | 日期值                   |
| TIME      | 3           | '-838:59:59'/'838:59:59'                                     | HH:MM:SS            | 时间值或持续时间         |
| YEAR      | 1           | 1901/2155                                                    | YYYY                | 年份值                   |
| DATETIME  | 8           | 1000-01-01 00:00:00/9999-12-31 23:59:59                      | YYYY-MM-DD HH:MM:SS | 混合日期和时间值         |
| TIMESTAMP | 4           | 1970-01-01 00:00:00/2038结束时间是第 **2147483647** 秒，北京时间 **2038-1-19 11:14:07**，格林尼治时间 2038年1月19日 凌晨 03:14:07 | YYYYMMDD HHMMSS     | 混合日期和时间值，时间戳 |

**字符串类型**

字符串类型指CHAR、VARCHAR、BINARY、VARBINARY、BLOB、TEXT、ENUM和SET。该节描述了这些类型如何工作以及如何在查询中使用这些类型。5.x以后varchar()括号里的数字代表的就是字符数，无论是中文还是英文

| 类型       | 大小                | 用途                            |
| :--------- | :------------------ | :------------------------------ |
| CHAR       | 0-255字节           | 定长字符串                      |
| VARCHAR    | 0-65535 字节        | 变长字符串                      |
| TINYBLOB   | 0-255字节           | 不超过 255 个字符的二进制字符串 |
| TINYTEXT   | 0-255字节           | 短文本字符串                    |
| BLOB       | 0-65 535字节        | 二进制形式的长文本数据          |
| TEXT       | 0-65 535字节        | 长文本数据                      |
| MEDIUMBLOB | 0-16 777 215字节    | 二进制形式的中等长度文本数据    |
| MEDIUMTEXT | 0-16 777 215字节    | 中等长度文本数据                |
| LONGBLOB   | 0-4 294 967 295字节 | 二进制形式的极大文本数据        |
| LONGTEXT   | 0-4 294 967 295字节 | 极大文本数据                    |

### 权限管理

MySQL 的账户信息保存在 mysql 这个数据库中。

```
USE mysql;
SELECT user FROM user;
```

**创建账户**

新创建的账户没有任何权限。

```
CREATE USER myuser IDENTIFIED BY 'mypassword';
```

**修改账户名**

```
RENAME USER myuser TO newuser;
```

**删除账户**

```
DROP USER myuser;
```

**查看权限**

```
SHOW GRANTS FOR myuser;
```

**授予权限**

账户用 username@host 的形式定义，username@% 使用的是默认主机名。

```
GRANT SELECT, INSERT ON mydatabase.* TO myuser;
```

**删除权限**

GRANT 和 REVOKE 可在几个层次上控制访问权限：

- 整个服务器，使用 GRANT ALL 和 REVOKE ALL；
- 整个数据库，使用 ON database.*；
- 特定的表，使用 ON database.table；
- 特定的列；
- 特定的存储过程。

```
REVOKE SELECT, INSERT ON mydatabase.* FROM myuser;
```

**更改密码**

必须使用 Password() 函数进行加密。

```
SET PASSWROD FOR myuser = Password('new_password');
```

## 基础语法

### 创建database/模式

数据库创建与使用：

```mysql
CREATE DATABASE test;
USE test;
```

 模式定义了数据如何存储、存储什么样的数据以及数据如何分解等信息，数据库和表都有模式。 

### 创建表

```mysql
CREATE TABLE mytable (
  # int 类型，不为空，自增
  id INT NOT NULL AUTO_INCREMENT,
  # int 类型，不可为空，默认值为 1，不为空
  col1 INT NOT NULL DEFAULT 1,
  # 变长字符串类型，最长为 45 个字符，可以为空
  col2 VARCHAR(45) NULL,
  # 日期类型，可为空
  col3 DATE NULL,
  # 设置主键为 id
  PRIMARY KEY (`id`));
```

### 修改

**修改列名**

```mysql
alter table 表名 change 旧列名 新列名 新类型;
```

### 插入

**添加列**

```mysql
ALTER TABLE mytable
ADD col CHAR(20);
```

这条语句会向已有的表中加入新的一列，这一列在表的最后一列位置。如果我们希望添加在指定的一列，可以用：

alter table TABLE_NAME add column NEW_COLUMN_NAME varchar(20) not null after COLUMN_NAME;

注意，上面这个命令的意思是说添加新列到某一列后面。如果想添加到第一列的话，可以用：

alter table TABLE_NAME add column NEW_COLUMN_NAME varchar(20) not null first;

**普通插入**

```mysql
INSERT INTO mytable(col1, col2)
VALUES(val1, val2);
```

**插入检索出来的数据**

```mysql
INSERT INTO mytable1(col1, col2)
SELECT col1, col2
FROM mytable2;
```

**将一个表的内容插入到一个新表**

```mysql
CREATE TABLE newtable AS
SELECT * FROM mytable;
```

**增加外键**

```mysql
 Alter table 表名 
 add constraint FK_ID foreign key(外键字段名) REFERENCES 外表表名(主键字段名)； 
```

### 更新

```mysql
UPDATE mytable
SET col = val,col2=val,
WHERE id = 1;
```

### 删除

```mysql
DELETE FROM mytable
WHERE id = 1;
```

**TRUNCATE TABLE** 可以清空表，也就是删除所有行。

```mysql
TRUNCATE TABLE mytable;
```

使用更新和删除操作时一定要用 WHERE 子句，不然会把整张表的数据都破坏。可以先用 SELECT 语句进行测试，防止错误删除。

**删除列**

```mysql
ALTER TABLE mytable
DROP COLUMN col;
```

**删除表**

```mysql
DROP TABLE mytable;
```



### 查询

**去重distinct**

 相同值只会出现一次。它作用于所有列，也就是说所有列的值都相同才算相同。 

```mysql
select distinct id from xx where id=xx;
```

**limit**

限制返回的行数。可以有两个参数，第一个参数为起始行，从 0 开始；第二个参数为返回的总行数。

返回前 5 行：

```mysql
SELECT *
FROM mytable
LIMIT 5;
```

```mysql
SELECT *
FROM mytable
LIMIT 0, 5;
```

返回第 3 ~ 5 行：

```mysql
SELECT *
FROM mytable
LIMIT 2, 3;
```

### 排序order by

- **ASC** ：升序（默认）
- **DESC** ：降序

可以按多个列进行排序，并且为每个列指定不同的排序方式：

```
SELECT *
FROM mytable
ORDER BY col1 DESC, col2 ASC;
```

### where可用操作符

| 操作符  | 说明         |
| ------- | ------------ |
| =       | 等于         |
| <       | 小于         |
| >       | 大于         |
| <> !=   | 不等于       |
| <= !>   | 小于等于     |
| >= !<   | 大于等于     |
| BETWEEN | 在两个值之间 |
| IS NULL | 为 NULL 值   |

应该注意到，NULL 与 0、空字符串都不同。

**AND 和 OR** 用于连接多个过滤条件。优先处理 AND，当一个过滤表达式涉及到多个 AND 和 OR 时，可以使用 () 来决定优先级，使得优先级关系更清晰。

**IN** 操作符用于匹配一组值，其后也可以接一个 SELECT 子句，从而匹配子查询得到的一组值。

**NOT** 操作符用于否定一个条件。

### like与通配符

通配符也是用在过滤语句中，但它只能用于文本字段。

- **%** 匹配 >=0 个任意字符；
- **_** 匹配 ==1 个任意字符；
- **[ ]** 可以匹配集合内的字符，例如 [ab] 将匹配字符 a 或者 b。用脱字符 ^ 可以对其进行否定，也就是不匹配集合内的字符。

使用 Like 来进行通配符匹配。

```
SELECT *
FROM mytable
WHERE col LIKE '[^AB]%'; -- 不以 A 和 B 开头的任意文本
```

不要滥用通配符，通配符位于开头处匹配会非常慢。

### group by 分组

使用分组的目的是为了分组聚集，分组后对该组进行聚集函数，分组命中的是一组，如果select了并非是组中相同的列那么就会报错（组中肯定会有某一列是不相同的信息）

```mysql
SELECT name, COUNT(*) FROM   employee_tbl GROUP BY name;
```

**having**用于过滤分组

```mysql
SELECT col, COUNT(*) AS num
FROM mytable
WHERE col > 2
GROUP BY col
HAVING num >= 2;
```

分组规定：

- GROUP BY 子句出现在 WHERE 子句之后，ORDER BY 子句之前；
- 除了汇总字段外，SELECT 语句中的每一字段都必须在 GROUP BY 子句中给出；
- NULL 的行会单独分为一组；
- 大多数 SQL 实现不支持 GROUP BY 列具有可变长度的数据类型。

**WITH ROLLUP**可以实现在分组统计数据基础上再进行相同的统计（SUM,AVG,COUNT…）。

例如我们将以上的数据表按名字进行分组，再统计每个人登录的次数：

```mysql
mysql> SELECT name, SUM(singin) as singin_count FROM  employee_tbl GROUP BY name WITH ROLLUP;
+--------+--------------+
| name   | singin_count |
+--------+--------------+
| 小丽 |            2 |
| 小明 |            7 |
| 小王 |            7 |
| NULL   |           16 |
+--------+--------------+
4 rows in set (0.00 sec)
```

其中记录 NULL 表示所有人的登录次数。

我们可以使用 coalesce 来设置一个可以取代 NUll 的名称，coalesce 语法：

```mysql
select coalesce(a,b,c);
```

参数说明：如果a==null,则选择b；如果b==null,则选择c；如果a!=null,则选择a；如果a b c 都为null ，则返回为null（没意义）。

以下实例中如果名字为空我们使用总数代替：

```mysql
mysql> SELECT coalesce(name, '总数'), SUM(singin) as singin_count FROM  employee_tbl GROUP BY name WITH ROLLUP;
+--------------------------+--------------+
| coalesce(name, '总数') | singin_count |
+--------------------------+--------------+
| 小丽                   |            2 |
| 小明                   |            7 |
| 小王                   |            7 |
| 总数                   |           16 |
+--------------------------+--------------+
4 rows in set (0.01 sec)
```

### 子查询

子查询中只能返回一个字段的数据。

可以将子查询的结果作为 WHRER 语句的过滤条件：

```mysql
SELECT *
FROM mytable1
WHERE col1 IN (SELECT col2
               FROM mytable2);
```

下面的语句可以检索出客户的订单数量，子查询语句会对第一个查询检索出的每个客户执行一次：

```mysql
SELECT cust_name, (SELECT COUNT(*)
                   FROM Orders
                   WHERE Orders.cust_id = Customers.cust_id)
                   AS orders_num
FROM Customers
ORDER BY cust_name;
```

### 连接

连接用于连接多个表，使用 JOIN 关键字，并且条件语句使用 ON 而不是 WHERE。

连接可以替换子查询，并且比子查询的效率一般会更快。

可以用 AS 给列名、计算字段和表名取别名，给表名取别名是为了简化 SQL 语句以及连接相同表。

**内连接**

又称等值连接，使用 INNER JOIN 关键字，A表中的所有行与B表中的所有行做笛卡尔积。

```mysql
SELECT A.value, B.value
FROM tablea AS A INNER JOIN tableb AS B
ON A.key = B.key;
```

可以不明确使用 INNER JOIN，而使用普通查询并在 WHERE 中将两个表中要连接的列用等值方法连接起来。

```mysql
SELECT A.value, B.value
FROM tablea AS A, tableb AS B
WHERE A.key = B.key;
```

可以对同一张表做自连接

**自然连接**

自然连接是把同名列通过等值测试连接起来的，同名列可以有多个。

内连接和自然连接的区别：内连接提供连接的列，而自然连接自动连接所有同名列。

```mysql
SELECT A.value, B.value
FROM tablea AS A NATURAL JOIN tableb AS B;
```

**外连接**

外连接即在自然连接的基础上同时保留了没有关联的那些行。分为左外连接，右外连接以及全外连接，左外连接就是保留左表没有关联的行。

检索所有顾客的订单信息，包括还没有订单信息的顾客。

```mysql
SELECT Customers.cust_id, Orders.order_num
FROM Customers LEFT OUTER JOIN Orders
ON Customers.cust_id = Orders.cust_id;
```

customers 表：

| cust_id | cust_name |
| ------- | --------- |
| 1       | a         |
| 2       | b         |
| 3       | c         |

orders 表：

| order_id | cust_id |
| -------- | ------- |
| 1        | 1       |
| 2        | 1       |
| 3        | 3       |
| 4        | 3       |

结果：

| cust_id | cust_name | order_id |
| ------- | --------- | -------- |
| 1       | a         | 1        |
| 1       | a         | 2        |
| 3       | c         | 3        |
| 3       | c         | 4        |
| 2       | b         | Null     |

### 组合查询union

union代表集合中并的含义。使用 **UNION** 来组合两个查询，如果第一个查询返回 M 行，第二个查询返回 N 行，那么组合查询的结果一般为 M+N 行。

每个查询必须包含相同的列、表达式和聚集函数。

默认会去除相同行，如果需要保留相同行，使用 UNION ALL。

只能包含一个 ORDER BY 子句，并且必须位于语句的最后。

```mysql
SELECT col
FROM mytable
WHERE col = 1
UNION
SELECT col
FROM mytable
WHERE col =2;
```

### 视图

视图是虚拟的表，本身不包含数据，也就不能对其进行索引操作。

对视图的操作和对普通表的操作一样。

视图具有如下好处：

- 简化复杂的 SQL 操作，比如复杂的连接；
- 只使用实际表的一部分数据；
- 通过只给用户访问视图的权限，保证数据的安全性；
- 更改数据格式和表示。

```mysql
CREATE VIEW myview AS
SELECT Concat(col1, col2) AS concat_col, col3*col4 AS compute_col
FROM mytable
WHERE col5 = val;
```

### 触发器

触发器即由事件来触发某个操作，事件包括INSERT语句，UPDATE语句和DELETE语句；可以协助应用在数据库端确保数据的完整性。也就是说，触发器会在某个表执行：DELETE、INSERT、UPDATE语句时而自动执行。

**触发器尽量少的使用，因为不管如何，它还是很消耗资源，如果使用的话要谨慎的使用，确定它是非常高效的：触发器是针对每一行的；对增删改非常频繁的表上切记不要使用触发器，因为它会非常消耗资源**

触发器的语法为：

```mysql
CREATE
    [DEFINER = { user | CURRENT_USER }]
TRIGGER trigger_name
trigger_time trigger_event
ON tbl_name FOR EACH ROW
　　[trigger_order]
trigger_body
```

其中，

* trigger_time: { BEFORE | AFTER } BEFORE或AFTER参数指定了触发执行的时间，在事件之前或是之后。 

* trigger_event: { INSERT | UPDATE | DELETE }：①INSERT型触发器：插入某一行时激活触发器，可能通过INSERT、LOAD DATA、REPLACE 语句触发(LOAD DAT语句用于将一个文件装入到一个数据表中，相当与一系列的INSERT操作)；②UPDATE型触发器：更改某一行时激活触发器，可能通过UPDATE语句触发；③DELETE型触发器：删除某一行时激活触发器，可能通过DELETE、REPLACE语句触发。
* trigger_order: { FOLLOWS | PRECEDES } other_trigger_name：MySQL5.7之后的一个功能，用于定义多个触发器，使用follows(尾随)或precedes(在…之先)来选择触发器执行的先后顺序。  
* trigger_body：触发器执行的语句，可使用begin...end来包括多行语句

 FOR EACH ROW表示任何一条记录上的操作满足触发事件都会触发该触发器，也就是说触发器的触发频率是针对每一行数据触发一次。 

**例1：创建了一个名为trig1的触发器，一旦在work表中有插入动作，就会自动往time表里插入当前时间**

```mysql
mysql> CREATE TRIGGER trig1 AFTER INSERT
    -> ON work FOR EACH ROW
    -> INSERT INTO time VALUES(NOW());
```

**例2：定义一个触发器，一旦有满足条件的删除操作，就会执行BEGIN和END中的语句**

```mysql
mysql> DELIMITER ||
mysql> CREATE TRIGGER trig2 BEFORE DELETE
    -> ON work FOR EACH ROW
    -> BEGIN
    -> 　　INSERT INTO time VALUES(NOW());
    -> 　　INSERT INTO time VALUES(NOW());
    -> END||
mysql> DELIMITER ;
```

**NEW与OLD**

MySQL 中定义了 NEW 和 OLD，用来表示触发器的所在表中，触发了触发器的那一行数据，来引用触发器中发生变化的记录内容，具体地：

　　①在INSERT型触发器中，NEW用来表示将要（BEFORE）或已经（AFTER）插入的新数据；

　　②在UPDATE型触发器中，OLD用来表示将要或已经被修改的原数据，NEW用来表示将要或已经修改为的新数据；

　　③在DELETE型触发器中，OLD用来表示将要或已经被删除的原数据；

使用时直接使用NEW.columnName （columnName为相应数据表某一列名）就代表那一列的符合触发器条件的那一行数据，OLD是只读的 。

**例**

INSERT 触发器包含一个名为 NEW 的虚拟表。

```mysql
CREATE TRIGGER mytrigger AFTER INSERT ON mytable
FOR EACH ROW SELECT NEW.col into @result;

SELECT @result; -- 获取结果
```

DELETE 触发器包含一个名为 OLD 的虚拟表，并且是只读的。

UPDATE 触发器包含一个名为 NEW 和一个名为 OLD 的虚拟表，其中 NEW 是可以被修改的，而 OLD 是只读的。

MySQL 不允许在触发器中使用 CALL 语句，也就是不能调用存储过程。

### 事务管理

基本术语：

- 事务（transaction）指一组 SQL 语句；
- 回退（rollback）指撤销指定 SQL 语句的过程；
- 提交（commit）指将未存储的 SQL 语句结果写入数据库表；
- 保留点（savepoint）指事务处理中设置的临时占位符（placeholder），你可以对它发布回退（与回退整个事务处理不同）。

不能回退 SELECT 语句，回退 SELECT 语句也没意义；也不能回退 CREATE 和 DROP 语句。

MySQL 的事务提交默认是隐式提交，每执行一条语句就把这条语句当成一个事务然后进行提交。当出现 START TRANSACTION 语句时，会关闭隐式提交；当 COMMIT 或 ROLLBACK 语句执行后，事务会自动关闭，重新恢复隐式提交。

设置 autocommit 为 0 可以取消自动提交；autocommit 标记是针对每个连接而不是针对服务器的。

如果没有设置保留点，ROLLBACK 会回退到 START TRANSACTION 语句处；如果设置了保留点，并且在 ROLLBACK 中指定该保留点，则会回退到该保留点。

```mysql
START TRANSACTION
// ...
SAVEPOINT delete1
// ...
ROLLBACK TO delete1
// ...
COMMIT
```

## 常用函数

### avg()/count()/max()/min()/sum()

| 函 数   | 说 明            |
| ------- | ---------------- |
| AVG()   | 返回某列的平均值 |
| COUNT() | 返回某列的行数   |
| MAX()   | 返回某列的最大值 |
| MIN()   | 返回某列的最小值 |
| SUM()   | 返回某列值之和   |

AVG() 会忽略 NULL 行。

使用 DISTINCT 可以汇总不同的值。

```mysql
SELECT AVG(DISTINCT col1) AS avg_col
FROM mytable;
```

### 文本处理函数

| RIGHT()  | 说明           |
| -------- | -------------- |
| LOWER()  | 转换为小写字符 |
| UPPER()  | 转换为大写字符 |
| LENGTH() | 长度           |

### 数值处理函数

| 函数   | 说明   |
| ------ | ------ |
| SIN()  | 正弦   |
| COS()  | 余弦   |
| TAN()  | 正切   |
| ABS()  | 绝对值 |
| SQRT() | 平方根 |
| MOD()  | 余数   |
| EXP()  | 指数   |
| PI()   | 圆周率 |
| RAND() | 随机数 |

### concat()连接字段

**CONCAT()** 用于连接两个字段。许多数据库会使用空格把一个值填充为列宽，因此连接的结果会出现一些不必要的空格，使用 **TRIM()** 可以去除首尾空格。

```mysql
SELECT CONCAT(TRIM(col1), '(', TRIM(col2), ')') AS concat_col
FROM mytable;
```

## 常见错误

#### 系统错误 5 拒绝访问

权限太低，以管理员方式启动

#### MySQL 服务无法启动。

目前原因是因为端口号3306被占用，换一个端口号，

#### 找不到localhost(10061)

找到【服务】中的MySQL，启动该服务

#### 发送数据包过大

 Error querying database. Cause:  com.mysql.cj.jdbc.exceptions.PacketTooBigException: Packet for query is  too large (1,063 > 1,024). You can change this value on the server by setting the 'max_allowed_packet' variable.

MYSQL会根据配置文件会限制server接受的数据包大小。![img](http://forums.mysql.com/common/logos/logo_mysql_sun.gif)

有时候在大的插入和更新会被max_allowed_packet参数限制掉，导致失败

执行SQL语句，查看大小：

```mysql
show variables like '%max_allowed_packet%';
```

更改大小方法：

```mysql
set global max_allowed_packet = 2*1024*1024*10;（20M）
```

经测试根据sql设置最大为4M

需要在/etc/my.cnf 设置

max_allowed_packet = 20M

重启mysql后生效

#### 发生系统错误2 系统找不到指定的文件

以管理员身份运行，在命令行输入cd+mySQL的bin目录的安装路径

C:\Windows\system32>cd C:\Program Files\MySQL\MySQL Server5.6\bin

C:\Program Files\MySQL\MySQL Server5.6\bin>mysqld --remove

Service successfully removed.

C:\Program Files\MySQL\MySQL Server5.6\bin>mysqld --install

Service successfully installed.

C:\Program Files\MySQL\MySQL Server5.6\bin>net start mysql

MySQL 服务正在启动 .

MySQL 服务已经启动成功。