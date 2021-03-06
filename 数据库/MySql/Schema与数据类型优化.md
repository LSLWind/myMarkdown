## 选择优化的数据类型

MySQL支持的数据类型非常多，选择正确的数据类型对于获得高性能至关重要。不管存储哪种类型的数据，下面几个简单的原则都有助于做出更好的选择。

**更小的通常更好**

一般情况下，应该尽量使用可以正确存储数据的最小数据类型。更小的数据类型通常更快，因为它们占用更少的磁盘、内存和CPU缓存，并且处理时需要的CPU周期也更少。

但是要确保没有低估需要存储的值的范围，因为在schema中的多个地方增加数据类型的范围是一个非常耗时和痛苦的操作。如果无法确定哪个数据类型是最好的，就选择你认为不会超过范围的最小类型。（如果系统不是很忙或者存储的数据量不多，或者是在可以轻易修改设计的早期阶段，那之后修改数据类型也比较容易）。

**简单就好**

简单数据类型的操作通常需要更少的CPU周期。例如，整型比字符操作代价更低，因为字符集和校对规则（排序规则）使字符比较比整型比较更复杂。==这里有两个例子：一个是应该使用MySQL内建的类型而不是字符串来存储日期和时间，另外一个是应该用整型存储IP地址。==

**尽量避免NULL**

很多表都包含可为NULL（空值）的列，即使应用程序并不需要保存NULL也是如此，这是因为可为NULL是列的默认属性。通常情况下最好指定列为NOT NULL，除非真的需要存储NULL值。

如果查询中包含可为NULL的列，对MySQL来说更难优化，因为可为NULL的列使得索引、索引统计和值比较都更复杂。可为NULL的列会使用更多的存储空间，在MySQL里也需要特殊处理。当可为NULL的列被索引时，每个索引记录需要一个额外的字节，在MyISAM里甚至还可能导致固定大小的索引（例如只有一个整数列的索引）变成可变大小的索引。

通常把可为NULL的列改为NOT NULL带来的性能提升比较小，所以（调优时）没有必要首先在现有schema中查找并修改掉这种情况，除非确定这会导致问题。但是，如果计划在列上建索引，就应该尽量避免设计成可为NULL的列。

当然也有例外，例如值得一提的是，InnoDB使用单独的位（bit）存储NULL值，所以对于稀疏数据有很好的空间效率。但这一点不适用于MyISAM。

在为列选择数据类型时，第一步需要确定合适的大类型：数字、字符串、时间等。

下一步是选择具体类型。很多MySQL的数据类型可以存储相同类型的数据，只是存储的长度和范围不一样、允许的精度不同，或者需要的物理空间（磁盘和内存空间）不同。相同大类型的不同子类型数据有时也有一些特殊的行为和属性。

例如，DATETIME和TIMESAMP列都可以存储相同类型的数据：时间和日期，精确到秒。

然而TIMESTAMP只使用DATETIME一半的存储空间，并且会根据时区变化，具有特殊的自动更新能力。另一方面，TIMESTAMP允许的时间范围要小得多，有时候它的特殊能力会成为障碍。

### 整数类型

有两种类型的数字：整数（whole number）和实数（real number）。如果存储整数，可以使用这几种整数类型：TINYINT，SMALLINT，MEDIUMINT，INT，BIGINT。分别使用8，16，24，32，64位存储空间。

整数类型有可选的UNSIGNED属性，表示不允许负值，这大致可以使正数的上限提高一倍。例如TINYINT UNSIGNED可以存储的范围是0～255，而TINYINT的存储范围是−128～127。

有符号和无符号类型使用相同的存储空间，并具有相同的性能，因此可以根据实际情况选择合适的类型。

你的选择决定MySQL是怎么在内存和磁盘中保存数据的。然而，整数计算一般使用64位的BIGINT整数，即使在32位环境也是如此

**MySQL可以为整数类型指定宽度，例如INT（11），对大多数应用这是没有意义的：它不会限制值的合法范围，只是规定了MySQL的一些交互工具（例如MySQL命令行客户端）用来显示字符的个数。对于存储和计算来说，INT（1）和INT（20）是相同的。****

### 实数类型

实数是带有小数部分的数字。然而，它们不只是为了存储小数部分；也可以使用DECIMAL存储比BIGINT还大的整数。MySQL既支持精确类型，也支持不精确类型。

DECIMAL类型用于存储精确的小数。浮点和DECIMAL类型都可以指定精度。对于DECIMAL列，可以指定小数点前后所允许的最大位数。这会影响列的空间消耗。

浮点类型在存储同样范围的值时，通常比DECIMAL使用更少的空间。FLOAT使用4个字节存储。DOUBLE占用8个字节，相比FLOAT有更高的精度和更大的范围。和整数类型一样，能选择的只是存储类型；MySQL使用DOUBLE作为内部浮点计算的类型。

因为需要额外的空间和计算开销，所以应该尽量只在对小数进行精确计算时才使用DECIMAL——例如存储财务数据。==但在数据量比较大的时候，可以考虑使用BIGINT代替DECIMAL，将需要存储的货币单位根据小数的位数乘以相应的倍数即可。==假设要存储财务数据精确到万分之一分，则可以把所有金额乘万，然后将结果存储在BIGINT里，这样可以同时避免浮点存储计算不精确和DECIMAL精确计算代价高的问题。

### VARCHAR类型

VARCHAR类型用于存储可变长字符串，是最常见的字符串数据类型。它比定长类型更节省空间，因为它仅使用必要的空间（例如，越短的字符串使用越少的空间）。有一种情况例外，如果MySQL表使用ROW_FORMAT=FIXED创建的话，每一行都会使用定长存储，这会很浪费空间。

VARCHAR需要使用1或2个额外字节记录字符串的长度：如果列的最大长度小于或等于255字节，则只使用1个字节表示，否则使用2个字节。假设采用latin1字符集，一个VARCHAR（10）的列需要11个字节的存储空间。VARCHAR（1000）的列则需要1002个字节，因为需要2个字节存储长度信息。

VARCHAR节省了存储空间，所以对性能也有帮助。但是，由于行是变长的，在UPDATE时可能使行变得比原来更长，这就导致需要做额外的工作。

### CHAR类型

CHAR类型是定长的：MySQL总是根据定义的字符串长度分配足够的空间。

CHAR适合存储很短的字符串，或者所有值都接近同一个长度。==例如，CHAR非常适合存储密码的MD5值，因为这是一个定长的值。==对于经常变更的数据，CHAR也比VARCHAR更好，因为定长的CHAR类型不容易产生碎片。对于非常短的列，CHAR比VARCHAR在存储空间上也更有效率。==例如用CHAR（1）来存储只有Y和N的值，如果采用单字节字符集只需要一个字节，但是VARCHAR（1）却需要两个字节，因为还有一个记录长度的额外字节==。

### BLOB和TEXT类型

BLOB和TEXT都是为存储很大的数据而设计的字符串数据类型，分别采用二进制和字符方式存储。

实际上，它们分别属于两组不同的数据类型家族：字符类型是TINYTEXT，SMALLTEXT，TEXT，MEDIUMTEXT，LONGTEXT；对应的二进制类型是TINYBLOB，SMALLBLOB，BLOB，MEDIUMBLOB，LONGBLOB。BLOB是SMALLBLOB的同义词，TEXT是SMALLTEXT的同义词。

与其他类型不同，MySQL把每个BLOB和TEXT值当作一个独立的对象处理。存储引擎在存储时通常会做特殊处理。当BLOB和TEXT值太大时，InnoDB会使用专门的“外部”存储区域来进行存储，此时每个值在行内需要1～4个字节存储一个指针，然后在外部存储区域存储实际的值。

### DATETIME类型

这个类型能保存大范围的值，从1001年到9999年，精度为秒。它把日期和时间封装到格式为YYYYMMDDHHMMSS的整数中，与时区无关。使用8个字节的存储空间。默认情况下，MySQL以一种可排序的、无歧义的格式显示DATETIME值，例如“2008-01-16 22:37:08”。这是ANSI标准定义的日期和时间表示方法。

### TIMESTAMP类型

就像它的名字一样，TIMETAMP类型保存了从1970年1月1日午夜（格林尼治标准时间）以来的秒数，它和UNIX时间戳相同。TIMESTAMP只使用4个字节的存储空间，因此它的范围比DATETIME小得多：只能表示从1970年到2038年。MySQL提供了FROM_UNIXTIME()函数把Unix时间戳转换为日期，并提供了UNIX_TIMESTAMP()函数把日期转换为Unix时间戳。

TIMESTAMP显示的值也依赖于时区。MySQL服务器、操作系统，以及客户端连接都有时区设置。

因此，存储值为0的TIMESTAMP在美国东部时区显示为“1969-12-31 19:00:00”，与格林尼治时间差5个小时。有必要强调一下这个区别：**如果在多个时区存储或访问数据，TIMESTAMP和DATETIME的行为将很不一样。前者提供的值与时区有关系，后者则保留文本表示的日期和时间。**

TIMESTAMP也有DATETIME没有的特殊属性。默认情况下，如果插入时没有指定第一个TIMESTAMP列的值，MySQL则设置这个列的值为当前时间。TIMESTAMP列默认为NOT NULL，这也和其他的数据类型不一样。

除了特殊行为之外，通常也应该尽量使用TIMESTAMP，因为它比DATETIME空间效率更高。有时候人们会将Unix时间截存储为整数值，但这不会带来任何收益。用整数保存时间截的格式通常不方便处理，所以我们不推荐这样做。

## MySQL schema设计中的陷阱

虽然有一些普遍的好或坏的设计原则，但也有一些问题是由MySQL的实现机制导致的，这意味着有可能犯一些只在MySQL下发生的特定错误。

**太多的列**

MySQL的存储引擎API工作时需要在服务器层和存储引擎层之间通过行缓冲格式拷贝数据，然后在服务器层将缓冲内容解码成各个列。从行缓冲中将编码过的列转换成行数据结构的操作代价是非常高的。MyISAM的定长行结构实际上与服务器层的行结构正好匹配，所以不需要转换。然而，MyISAM的变长行结构和InnoDB的行结构则总是需要转换。转换的代价依赖于列的数量。当我们研究一个CPU占用非常高的案例时，发现客户使用了非常宽的表（数千个字段），然而只有一小部分列会实际用到，这时转换的代价就非常高。如果计划使用数千个字段，必须意识到服务器的性能运行特征会有一些不同。

**太多的关联**

所谓的“实体-属性-值”（EAV）设计模式是一个常见的糟糕设计模式，尤其是在MySQL下不能靠谱地工作。MySQL限制了每个关联操作最多只能有61张表，但是EAV数据库需要许多自关联。我们见过不少EAV数据库最后超过了这个限制。事实上在许多关联少于61张表的情况下，解析和优化查询的代价也会成为MySQL的问题。一个粗略的经验法则，如果希望查询执行得快速且并发性好，单个查询最好在12个表以内做关联。

## 范式和反范式

对于任何给定的数据通常都有很多种表示方法，从完全的范式化到完全的反范式化，以及两者的折中。**在范式化的数据库中，每个事实数据会出现并且只出现一次。相反，在反范式化的数据库中，信息是冗余的，可能会存储在多个地方。**

以经典的“雇员，部门，部门领导”的例子开始：

| EMPLOYEE | DEPARTMENT  | HEAD  |
| -------- | ----------- | ----- |
| Jones    | Accounting  | Jones |
| Smith    | Engineering | Smith |
| Brown    | Accounting  | Jones |
| Green    | Engineering | Smith |

这个schema的问题是修改数据时可能发生不一致。假如Say Brown接任Accounting部门的领导，需要修改多行数据来反映这个变化，这是很痛苦的事并且容易引入错误。如果“Jones”这一行显示部门的领导跟“Brown”这一行的不一样，就没有办法知道哪个是对的。此外，这个设计在没有雇员信息的情况下就无法表示一个部门——如果我们删除了所有Accounting部门的雇员，我们就失去了关于这个部门本身的所有记录。要避免这个问题，我们需要对这个表进行范式化，方式是拆分雇员和部门项。拆分以后可以用下面两张表分别来存储雇员表：

| EMPLOYEE_NAME | DEPARTMENT  |
| ------------- | ----------- |
| Jones         | Accounting  |
| Smith         | Engineering |
| Brown         | Accounting  |
| Green         | Engineering |

和部门表：

| DEPARTMENT  | HEAD  |
| ----------- | ----- |
| Accounting  | Jones |
| Engineering | Smith |

这样设计的两张表符合第二范式，在很多情况下做到这一步已经足够好了。然而，第二范式只是许多可能的范式中的一种。

### 范式的优点和缺点

当为性能问题而寻求帮助时，经常会被建议对schema进行范式化设计，尤其是写密集的场景。这通常是个好建议。因为下面这些原因，范式化通常能够带来好处：

- 范式化的更新操作通常比反范式化要快。

- 当数据较好地范式化时，就只有很少或者没有重复数据，所以只需要修改更少的数据。
- 范式化的表通常更小，可以更好地放在内存里，所以执行操作会更快。
- 很少有多余的数据意味着检索列表数据时更少需要DISTINCT或者GROUP BY语句。还是前面的例子：在非范式化的结构中必须使用DISTINCT或者GROUP BY才能获得一份唯一的部门列表，但是如果部门（DEPARTMENT）是一张单独的表，则只需要简单的查询这张表就行了。

范式化设计的schema的缺点是通常需要关联。稍微复杂一些的查询语句在符合范式的schema上都可能需要至少一次关联，也许更多。这不但代价昂贵，也可能使一些索引策略无效。例如，范式化可能将列存放在不同的表中，而这些列如果在一个表中本可以属于同一个索引。

### 反范式的优点和缺点

反范式化的schema因为所有数据都在一张表中，可以很好地避免关联。

如果不需要关联表，则对大部分查询最差的情况——即使表没有使用索引——是全表扫描。当数据比内存大时这可能比关联要快得多，因为这样避免了随机I/O。

单独的表也能使用更有效的索引策略。假设有一个网站，允许用户发送消息，并且一些用户是付费用户。现在想查看付费用户最近的10条信息。如果是范式化的结构并且索引了发送日期字段published，这个查询也许看起来像这样：

```mysql
    mysql> SELECT message_text, user_name
        -> FROM message
        -> INNER JOIN user ON message.user_id=user.id
        -> WHERE user.account_type='premiumv'
        -> ORDER BY message.published DESC LIMIT 10;
```

要更有效地执行这个查询，MySQL需要扫描message表的published字段的索引。对于每一行找到的数据，将需要到user表里检查这个用户是不是付费用户。如果只有一小部分用户是付费账户，那么这是效率低下的做法。

主要问题是关联，使得需要在一个索引中又排序又过滤。如果采用反范式化组织数据，将两张表的字段合并一下，并且增加一个索引（account_type，published），就可以不通过关联写出这个查询。这将非常高效：

```mysql
mysql> SELECT message_text,user_name
        -> FROM user_messages
        -> WHERE account_type='premium'
        -> ORDER BY published DESC
        -> LIMIT 10;
```

### 混用范式化和反范式化

事实是，完全的范式化和完全的反范式化schema都是实验室里才有的东西：在真实世界中很少会这么极端地使用。在实际应用中经常需要混用，可能使用部分范式化的schema、缓存表，以及其他技巧。

## 缓存表和汇总表

有时提升性能最好的方法是在同一张表中保存衍生的冗余数据。然而，有时也需要创建一张完全独立的汇总表或缓存表（特别是为满足检索的需求时）。如果能容许少量的脏数据，这是非常好的方法，但是有时确实没有选择的余地（例如，需要避免复杂、昂贵的实时更新操作）。

术语“缓存表”和“汇总表”没有标准的含义。我们用术语“缓存表”来表示存储那些可以比较简单地从schema其他表获取（但是每次获取的速度比较慢）数据的表（例如，逻辑上冗余的数据）。而术语“汇总表”时，则保存的是使用GROUP BY语句聚合数据的表（例如，数据不是逻辑上冗余的）。也有人使用术语“累积表（Roll-Up Table）”称呼这些表。因为这些数据被“累积”了。

仍然以网站为例，假设需要计算之前24小时内发送的消息数。在一个很繁忙的网站不可能维护一个实时精确的计数器。作为替代方案，可以每小时生成一张汇总表。这样也许一条简单的查询就可以做到，并且比实时维护计数器要高效得多。缺点是计数器并不是100％精确。

如果必须获得过去24小时准确的消息发送数量（没有遗漏），有另外一种选择。以每小时汇总表为基础，把前23个完整的小时的统计表中的计数全部加起来，最后再加上开始阶段和结束阶段不完整的小时内的计数。假设统计表叫作msg_per_hr并且这样定义：

```mysql
    CREATE TABLE msg_per_hr (
       hr DATETIME NOT NULL,
       cnt INT UNSIGNED NOT NULL,
       PRIMARY KEY(hr)
    );
```

可以通过把下面的三个语句的结果加起来，得到过去24小时发送消息的总数。我们使用LEFT（NOW(),14）来获得当前的日期和时间最接近的小时：

```mysql
    mysql> SELECT SUM(cnt) FROM msg_per_hr
        -> WHERE hr BETWEEN
        ->   CONCAT(LEFT(NOW(), 14), '00:00') - INTERVAL 23 HOUR
        ->   AND CONCAT(LEFT(NOW(), 14), '00:00') - INTERVAL 1 HOUR;
    mysql> SELECT COUNT(*) FROM message
        -> WHERE posted >= NOW() - INTERVAL 24 HOUR
        ->   AND posted < CONCAT(LEFT(NOW(), 14), '00:00') - INTERVAL 23 HOUR;
    mysql> SELECT COUNT(*) FROM message
        -> WHERE posted >= CONCAT(LEFT(NOW(), 14), '00:00');
```

不管是哪种方法——不严格的计数或通过小范围查询填满间隙的严格计数——都比计算message表的所有行要有效得多。这是建立汇总表的最关键原因。实时计算统计值是很昂贵的操作，因为要么需要扫描表中的大部分数据，要么查询语句只能在某些特定的索引上才能有效运行，而这类特定索引一般会对UPDATE操作有影响，所以一般不希望创建这样的索引。计算最活跃的用户或者最常见的“标签”是这种操作的典型例子。

### 物化视图

许多数据库管理系统（例如Oracle或者微软SQL Server）都提供了一个被称作物化视图的功能。物化视图实际上是预先计算并且存储在磁盘上的表，可以通过各种各样的策略刷新和更新。MySQL并不原生支持物化视图。然而，使用Justin Swanhart的开源工具Flexviews（http://code.google.com/p/flexviews/），也可以自己实现物化视图。Flexviews比完全自己实现的解决方案要更精细，并且提供了很多不错的功能使得可以更简单地创建和维护物化视图。它由下面这些部分组成：

- 变更数据抓取（Change Data Capture，CDC）功能，可以读取服务器的二进制日志并且解析相关行的变更。
- 一系列可以帮助创建和管理视图的定义的存储过程。
- 一些可以应用变更到数据库中的物化视图的工具。

对比传统的维护汇总表和缓存表的方法，Flexviews通过提取对源表的更改，可以增量地重新计算物化视图的内容。这意味着不需要通过查询原始数据来更新视图。例如，如果创建了一张汇总表用于计算每个分组的行数，此后增加了一行数据到源表中，Flexviews简单地给相应的组的行数加一即可。同样的技术对其他的聚合函数也有效，例如SUM()和AVG()。这实际上是有好处的，基于行的二进制日志包含行更新前后的镜像，所以Flexviews不仅仅可以获得每行的新值，还可以不需要查找源表就能知道每行数据的旧版本。计算增量数据比从源表中读取数据的效率要高得多。

先写出一个SELECT语句描述想从已经存在的数据库中得到的数据。这可能包含关联和聚合（GROUP BY）。Flexviews中有一个辅助工具可以转换SQL语句到Flexviews的API调用。Flexviews会做完所有的脏活、累活：监控数据库的变更并且转换后用于更新存储物化视图的表。现在应用可以简单地查询物化视图来替代查询需要检索的表。

Flexviews有不错的SQL覆盖范围，包括一些棘手的表达式，你可能没有料到一个工具可以在MySQL服务器之外处理这些工作。这一点对创建基于复杂SQL表达式的视图很有用，可以用基于物化视图的简单、快速的查询替换原来复杂的查询。

### 计数器表

如果应用在表中保存计数器，则在更新计数器时可能碰到并发问题。计数器表在Web应用中很常见。可以用这种表缓存一个用户的朋友数、文件下载次数等。创建一张独立的表存储计数器通常是个好主意，这样可使计数器表小且快。使用独立的表可以帮助避免查询缓存失效，并且可以使用本节展示的一些更高级的技巧。

应该让事情变得尽可能简单，假设有一个计数器表，只有一行数据，记录网站的点击次数：

```mysql
    mysql> CREATE TABLE hit_counter (
        ->   cnt int unsigned not null
        -> ) ENGINE=InnoDB;
```

网站的每次点击都会导致对计数器进行更新：

```mysql
    mysql> UPDATE hit_counter SET cnt = cnt + 1;
```

问题在于，对于任何想要更新这一行的事务来说，这条记录上都有一个全局的互斥锁（mutex）。这会使得这些事务只能串行执行。要获得更高的并发更新性能，也可以将计数器保存在多行中，每次随机选择一行进行更新。这样做需要对计数器表进行如下修改：

```mysql
    mysql> CREATE TABLE hit_counter (
        ->   slot tinyint unsigned not null primary key,
        ->   cnt int unsigned not null
        -> ) ENGINE=InnoDB;
```

然后预先在这张表增加100行数据。现在选择一个随机的槽（slot）进行更新：

```mysql
    mysql> UPDATE hit_counter SET cnt = cnt + 1 WHERE slot = RAND() * 100;
```

要获得统计结果，需要使用下面这样的聚合查询：

```mysql
    mysql> SELECT SUM(cnt) FROM hit_counter;
```

一个常见的需求是每隔一段时间开始一个新的计数器（例如，每天一个）。如果需要这么做，则可以再简单地修改一下表设计：

```mysql
    mysql> CREATE TABLE daily_hit_counter (
        ->   day date not null,
        ->   slot tinyint unsigned not null,
        ->   cnt int unsigned not null,
        ->   primary key(day, slot)
        -> ) ENGINE=InnoDB;
```

在这个场景中，可以不用像前面的例子那样预先生成行，而用ON DUPLICATE KEY UPDATE代替：

```mysql
mysql> INSERT INTO daily_hit_counter(day, slot, cnt)
    ->   VALUES(CURRENT_DATE, RAND() * 100, 1)
    ->   ON DUPLICATE KEY UPDATE cnt = cnt + 1;
```

如果希望减少表的行数，以避免表变得太大，可以写一个周期执行的任务，合并所有结果到0号槽，并且删除所有其他的槽：

```mysql
    mysql> UPDATE daily_hit_counter as c
        ->   INNER JOIN (
        ->     SELECT day, SUM(cnt) AS cnt, MIN(slot) AS mslot
        ->     FROM daily_hit_counter
        ->     GROUP BY day
        ->    ) AS x USING(day)
        -> SET c.cnt = IF(c.slot = x.mslot, x.cnt, 0),
        ->    c.slot = IF(c.slot = x.mslot, 0, c.slot);
    mysql> DELETE FROM daily_hit_counter WHERE slot <> 0 AND cnt = 0;
```

## 加快ALTER TABLE操作的速度

**MySQL的ALTER TABLE操作的性能对大表来说是个大问题。MySQL执行大部分修改表结构操作的方法是用新的结构创建一个空表，从旧表中查出所有数据插入新表，然后删除旧表。这样操作可能需要花费很长时间，如果内存不足而表又很大，而且还有很多索引的情况下尤其如此。**许多人都有这样的经验，ALTER TABLE操作需要花费数个小时甚至数天才能完成。

一般而言，大部分ALTER TABLE操作将导致MySQL服务中断。我们会展示一些在DDL操作时有用的技巧，但这是针对一些特殊的场景而言的。对常见的场景，能使用的技巧只有两种：一种是先在一台不提供服务的机器上执行ALTER TABLE操作，然后和提供服务的主库进行切换；另外一种技巧是“影子拷贝”。**影子拷贝的技巧是用要求的表结构创建一张和源表无关的新表，然后通过重命名和删表操作交换两张表。**也有一些工具可以帮助完成影子拷贝工作：例如，Facebook数据库运维团队（https://launchpad.net/mysqlatfacebook）的“online schema change”工具、Shlomi Noach的openark toolkit（http://code.openark.org/），以及Percona Toolkit（http://www.percona.com/software/）。

不是所有的ALTER TABLE操作都会引起表重建。例如，有两种方法可以改变或者删除一个列的默认值（一种方法很快，另外一种则很慢）。假如要修改电影的默认租赁期限，从三天改到五天。下面是很慢的方式：

```mysql
    mysql> ALTER TABLE sakila.film
        -> MODIFY COLUMN rental_duration TINYINT(3) NOT NULL DEFAULT 5;
```

SHOW STATUS显示这个语句做了1000次读和1000次插入操作。换句话说，它拷贝了整张表到一张新表，甚至列的类型、大小和可否为NULL属性都没改变。

理论上，MySQL可以跳过创建新表的步骤。列的默认值实际上存在表的.frm文件中，所以可以直接修改这个文件而不需要改动表本身。然而MySQL还没有采用这种优化的方法，所有的MODIFY COLUMN操作都将导致表重建。

另外一种方法是通过ALTER COLUMN操作来改变列的默认值：

```mysql
    mysql> ALTER TABLE sakila.film
        -> ALTER COLUMN rental_duration SET DEFAULT 5;
```

这个语句会直接修改.frm文件而不涉及表数据。所以，这个操作是非常快的。