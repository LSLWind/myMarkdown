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

#### 日期和时间类型

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

#### 字符串类型

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

### 基础语法

##### 增加外键

 Alter table 表名 add constraint FK_ID foreign key(外键字段名) REFERENCES 外表表名(主键字段名)； 

##### 插入数据

INSERT INTO table(column1,column2...) VALUES (value1,value2,...);

#### 更新数据

```
 ``UPDATE` `表名 ``SET
      ``字段1=值1, #注意语法，可以同时来修改多个值，用逗号分隔
      ``字段2=值2,
      ``WHERE` `CONDITION; #更改哪些数据，通过``where``条件来定位到符合条件的数据
```

#### 查

##### 去重distinct

```
select distinct id from xx where id=xx;
```



#### 更改表

##### 修改列名

alter table 表名 change 旧列名 新列名 新类型;

##### 插入新列

如果想在一个已经建好的表中添加一列，可以用诸如：

alter table TABLE_NAME add column NEW_COLUMN_NAME varchar(20) not null;

这条语句会向已有的表中加入新的一列，这一列在表的最后一列位置。如果我们希望添加在指定的一列，可以用：

alter table TABLE_NAME add column NEW_COLUMN_NAME varchar(20) not null after COLUMN_NAME;

注意，上面这个命令的意思是说添加新列到某一列后面。如果想添加到第一列的话，可以用：

alter table TABLE_NAME add column NEW_COLUMN_NAME varchar(20) not null first;

##### 删除表

drop table 表名

##### 删除列

alter table 表名 drop column 列名; 

##### 删除行

delete from 表名 where condition

#### group by 分组

使用分组的目的是为了分组聚集，分组后对该组进行聚集函数，分组命中的是一组，如果select了并非是组中相同的列那么就会报错（组中肯定会有某一列是不相同的信息）

```mysql
SELECT name, COUNT(*) FROM   employee_tbl GROUP BY name;
```

WITH ROLLUP 可以实现在分组统计数据基础上再进行相同的统计（SUM,AVG,COUNT…）。

例如我们将以上的数据表按名字进行分组，再统计每个人登录的次数：

```
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

```
select coalesce(a,b,c);
```

参数说明：如果a==null,则选择b；如果b==null,则选择c；如果a!=null,则选择a；如果a b c 都为null ，则返回为null（没意义）。

以下实例中如果名字为空我们使用总数代替：

```
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

#### 特性

#### auto_increment原理

自增字段必须建立索引，可以不是主键索引，通过内部维护一个整形保证并发情况下该字段的唯一性。

auto_increment有三种模式：

1. innodb_autoinc_lock_mode=0（traditional lock mode） 传统模式：

每次插入列时对整个表加上AUTO_INC锁（表级锁），正常情况下连续（事务rollback可能出现间隙）

2. innodb_autoinc_lock_mode=1（consecutive lock mode） 

针对bulk inserts才会采用AUTO-INC锁这种方式，而针对simple inserts，则采用了一种新的轻量级的互斥锁来分配auto_increment列的值。当然，如果其他事务已经持有了AUTO-INC锁，则simple inserts需要等待

simple inserts指的是那种能够事先确定插入行数的语句，比如INSERT/REPLACE INTO 等插入单行或者多行的语句，语句中不包括嵌套子查询。此外，INSERT INTO … ON DUPLICATE KEY UPDATE这类语句也要除外。

bulk inserts指的是事先无法确定插入行数的语句，比如INSERT/REPLACE INTO … SELECT, LOAD DATA等。

mixed-mode inserts指的是simple inserts类型中有些行指定了auto_increment列的值有些没有指定，比如：
INSERT INTO t1 (c1,c2) VALUES (1,’a’), (NULL,’b’), (5,’c’), (NULL,’d’);
另外一种mixed-mode inserts是 INSERT … ON DUPLICATE KEY UPDATE这种语句，可能导致分配的auto_increment值没有被使用。

3.innodb_autoinc_lock_mode=2（interleaved lock mode）

这种模式下任何类型的inserts都不会采用AUTO-INC锁，性能最好，但是在同一条语句内部产生auto_increment值间隙。此外，这种模式对statement-based replication也不安全。

https://blog.csdn.net/sgbfblog/article/details/44536941

### 基础

#### 服务器启动

**mysqld**

`mysqld`这个可执行文件就代表着`MySQL`服务器程序，运行这个可执行文件就可以直接启动一个服务器进程。但这个命令不常用，我们继续往下看更牛逼的启动命令。

**mysqld_safe**

`mysqld_safe`是一个启动脚本，它会间接的调用`mysqld`，而且还顺便启动了另外一个监控进程，这个监控进程在服务器进程挂了的时候，可以帮助重启它。另外，使用`mysqld_safe`启动服务器程序时，它会将服务器程序的出错信息和其他诊断信息重定向到某个文件中，产生出错日志，这样可以方便我们找出发生错误的原因。

**mysql.server**

`mysql.server`也是一个启动脚本，它会间接的调用`mysqld_safe`，在调用`mysql.server`时在后边指定`start`参数就可以启动服务器程序了，就像这样：

```
mysql.server start
```

**mysqld_multi**

其实我们一台计算机上也可以运行多个服务器实例，也就是运行多个`MySQL`服务器进程。`mysql_multi`可执行文件可以对每一个服务器进程的启动或停止进行监控。

#### 启动MySQL客户端程序

在我们成功启动`MySQL`服务器程序后，就可以接着启动客户端程序来连接到这个服务器喽，`bin`目录下有许多客户端程序，比方说`mysqladmin`、`mysqldump`、`mysqlcheck`等等等等。这里我们重点要关注的是可执行文件`mysql`，通过这个可执行文件可以让我们和服务器程序进程交互，也就是发送请求，接收服务器的处理结果。启动这个可执行文件时一般需要一些参数，格式如下：

```
mysql -h主机名  -u用户名 -p密码
```

各个参数的意义如下：

| 参数名 | 含义                                                         |
| :----: | :----------------------------------------------------------- |
|  `-h`  | 表示服务器进程所在计算机的域名或者IP地址，如果服务器进程就运行在本机的话，可以省略这个参数，或者填`localhost`或者`127.0.0.1`。也可以写作 `--host=主机名`的形式。 |
|  `-u`  | 表示用户名。也可以写作 `--user=用户名`的形式。               |
|  `-p`  | 表示密码。也可以写作 `--password=密码`的形式。               |

#### 查询过程

 ![image_1c8d26fmg1af0ms81cpc7gm8lv39.png-97.9kB](https://user-gold-cdn.xitu.io/2018/12/28/167f4c7b99f87e1c?imageView2/0/w/1280/h/960/format/webp/ignore-error/1) 

查询缓存就不说了，缓存未命中时进行实际查询

**语法解析**

如果查询缓存没有命中，接下来就需要进入正式的查询阶段了。因为客户端程序发送过来的请求只是一段文本而已，所以`MySQL`服务器程序首先要对这段文本做分析，判断请求的语法是否正确，然后从文本中将要查询的表、各种查询条件都提取出来放到`MySQL`服务器内部使用的一些数据结构上来。

#### 查询优化

语法解析之后，服务器程序获得到了需要的信息，比如要查询的列是哪些，表是哪个，搜索条件是什么等等，但光有这些是不够的，因为我们写的`MySQL`语句执行起来效率可能并不是很高，`MySQL`的优化程序会对我们的语句做一些优化，如外连接转换为内连接、表达式简化、子查询转为连接吧啦吧啦的一堆东西。优化的结果就是生成一个执行计划，这个执行计划表明了应该使用哪些索引进行查询，表之间的连接顺序是啥样的。我们可以使用`EXPLAIN`语句来查看某个语句的执行计划

### 存储引擎

截止到服务器程序完成了查询优化为止，还没有真正的去访问真实的数据表，`MySQL`服务器把数据的存储和提取操作都封装到了一个叫`存储引擎`的模块里。我们知道`表`是由一行一行的记录组成的，但这只是一个逻辑上的概念，物理上如何表示记录，怎么从表中读取数据，怎么把数据写入具体的物理存储器上，这都是`存储引擎`负责的事情。为了实现不同的功能，`MySQL`提供了各式各样的`存储引擎`，不同`存储引擎`管理的表具体的存储结构可能不同，采用的存取算法也可能不同。

为了管理方便，人们把`连接管理`、`查询缓存`、`语法解析`、`查询优化`这些并不涉及真实数据存储的功能划分为`MySQL server`的功能，把真实存取数据的功能划分为`存储引擎`的功能。各种不同的存储引擎向上边的`MySQL server`层提供统一的调用接口（也就是存储引擎API），包含了几十个底层函数，像"读取索引第一条内容"、"读取索引下一条内容"、"插入记录"等等。

所以在`MySQL server`完成了查询优化后，只需按照生成的执行计划调用底层存储引擎提供的API，获取到数据后返回给客户端就好了。

### 设置表的存储引擎

我们前边说过，存储引擎是负责对表中的数据进行提取和写入工作的，我们可以为不同的表设置不同的存储引擎，也就是说不同的表可以有不同的物理存储结构，不同的提取和写入方式。

#### 创建表时指定存储引擎

我们之前创建表的语句都没有指定表的存储引擎，那就会使用默认的存储引擎`InnoDB`（当然这个默认的存储引擎也是可以修改的，我们在后边的章节中再说怎么改）。如果我们想显式的指定一下表的存储引擎，那可以这么写：

```
CREATE TABLE 表名(
    建表语句;
) ENGINE = 存储引擎名称;
```

比如我们想创建一个存储引擎为`MyISAM`的表可以这么写：

```
mysql> CREATE TABLE engine_demo_table(
    ->     i int
    -> ) ENGINE = MyISAM;
Query OK, 0 rows affected (0.02 sec)

mysql>
```

#### 修改表的存储引擎

如果表已经建好了，我们也可以使用下边这个语句来修改表的存储引擎：

```
ALTER TABLE 表名 ENGINE = 存储引擎名称;
```

### 字符集

`utf8`字符集表示一个字符需要使用1～4个字节，但是我们常用的一些字符使用1～3个字节就可以表示了。而在`MySQL`中字符集表示一个字符所用最大字节长度在某些方面会影响系统的存储和性能，所以设计`MySQL`的大叔偷偷的定义了两个概念：

- `utf8mb3`：阉割过的`utf8`字符集，只使用1～3个字节表示字符。

- `utf8mb4`：正宗的`utf8`字符集，使用1～4个字节表示字符。

有一点需要十分的注意，在`MySQL`中`utf8`是`utf8mb3`的别名，所以之后在`MySQL`中提到`utf8`就意味着使用1~3个字节来表示一个字符，如果大家有使用4字节编码一个字符的情况，比如存储一些emoji表情啥的，那请使用`utf8mb4`。

### 并发处理

#### select 显式加锁

```sql
SELECT ... LOCK IN SHARE MODE       #共享锁，其它事务可读，不可更新
SELECT ... FOR UPDATE               #排它锁，其它事务不可读写复制代码
```

#### update加锁机制

MySQL是支持给数据行加锁（InnoDB）的，并且在UPDATE/DELETE等操作时确实会自动加上排它锁。只是**并非只要有UPDATE关键字就会全程加锁**，针对上面的MySQL语句而言，其实并不只是一条UPDATE语句，而应该类似于两条SQL语句（伪代码）：

```sql
a = SELECT * FROM table1 WHERE id=1;
UPDATE table1 SET num = a.num + 1 WHERE id=1; 复制代码
```

其中执行SELECT语句时没有加锁，只有在执行UPDATE时才进行加锁的。所以才会出现并发操作时的更新数据不一致。



### 常见错误

#### [系统错误 5 拒绝访问](https://www.cnblogs.com/youyouxiaosheng-lh/p/8343408.html)

权限太低，以管理员方式启动

#### MySQL 服务无法启动。

目前原因是因为端口号3306被占用，换一个端口号，

#### Can't connect toMySQL server on 'localhost'(10061)

找到【服务】中的MySQL，启动该服务

####  Error querying database. Cause:  com.mysql.cj.jdbc.exceptions.PacketTooBigException: Packet for query is  too large (1,063 > 1,024). You can change this value on the server by setting the 'max_allowed_packet' variable. 

 Error querying database. Cause:  com.mysql.cj.jdbc.exceptions.PacketTooBigException: Packet for query is  too large (1,063 > 1,024). You can change this value on the server by setting the 'max_allowed_packet' variable.

MYSQL会根据配置文件会限制server接受的数据包大小。![img](http://forums.mysql.com/common/logos/logo_mysql_sun.gif)

有时候在大的插入和更新会被max_allowed_packet参数限制掉，导致失败

 

执行SQL语句，查看大小：

show variables like '%max_allowed_packet%';

 

更改大小方法：

set global max_allowed_packet = 2*1024*1024*10;（20M）

 

经测试根据sql设置最大为4M

需要在/etc/my.cnf 设置

max_allowed_packet = 20M

重启mysql后生效

#### net start mysql 发生系统错误2 系统找不到指定的文件]

以管理员身份运行，在命令行输入cd+mySQL的bin目录的安装路径

C:\Windows\system32>cd C:\Program Files\MySQL\MySQL Server5.6\bin

C:\Program Files\MySQL\MySQL Server5.6\bin>mysqld --remove

Service successfully removed.

C:\Program Files\MySQL\MySQL Server5.6\bin>mysqld --install

Service successfully installed.

C:\Program Files\MySQL\MySQL Server5.6\bin>net start mysql

MySQL 服务正在启动 .

MySQL 服务已经启动成功。