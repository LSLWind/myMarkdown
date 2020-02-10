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

#### like模糊查询

在 where like 的条件查询中，SQL 提供了四种匹配方式。

1. **%**：表示任意 0 个或多个字符。可匹配任意类型和长度的字符，有些情况下若是中文，请使用两个百分号（%%）表示。
2. **_**：表示任意单个字符。匹配单个任意字符，它常用来限制表达式的字符长度语句。
3. **[]**：表示括号内所列字符中的一个（类似正则表达式）。指定一个字符、字符串或范围，要求所匹配对象为它们中的任一个。
4. **[^]** ：表示不在括号所列之内的单个字符。其取值和 [] 相同，但它要求所匹配对象为指定字符以外的任一个字符。
5. 查询内容包含通配符时,由于通配符的缘故，导致我们查询特殊字符 “%”、“_”、“[” 的语句无法正常实现，而把特殊字符用 “[ ]” 括起便可正常查询。

#### 特性







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