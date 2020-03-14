## 基础

MongoDB 是一款非关系型的数据库，非关系型就是把数据直接放进一个大仓库，不标号、不连线、单纯的堆起来

MongoDB 支持的数据结构非常松散，是类似 json 的 bson 格式，因此可以存储比较复杂的数据类型，如

```json
{
   title: '上周实验学习报告',
   body: '上周我们学习了 MongoDB 的知识',
   by: 'lsl',
}
```

如上是一个基本的文档形式，它存储了一封邮件的简易信息，从这里也可以看出 MongoDB 的‘松散’，它和关系型数据库不同，一个集合里面可以有不同格式的文档，而关系型数据库的每一条记录都拥有相同的字段。

**面向集合的存储**

在 MongoDB 中，一个数据库包含多个集合，类似于 MySQL 中一个数据库包含多个表；一个集合包含多个文档，类似于 MySQL 中一个表包含多条数据。可以把集合记为表，文档记为一条记录。

这样命名是有原因的，因为 MongoDB 没有行列统一的表格式排列，而是采用一个大仓库的形式将所有数据包纳其中。文档也一样，它是一段自由独立的数据，受外部限制少，所以区别于关系型数据库的记录。

### 数据库

- 一个 MongoDB 可以创建多个数据库
- 使用 `show dbs` 可以查看所有数据库的列表
- 执行 `db` 命令则可以查看当前数据库对象或者集合
- 运行 `use` 命令可以连接到指定的数据库

```shell
$ mongo    # 进入到mongo命令行
> use test    # 连接到test数据库
```

注意：数据库名可以是任何字符，但是不能有空格、点号和 $ 字符。

### 文档

文档是 MongoDB 的核心，类似于 SQLite 数据库（关系数据库）中的每一行数据。多个键及其关联的值放在一起就是文档。在 Mongodb 中使用一种类 json 的 bson 存储数据，bson 数据可以理解为在 json 的基础上添加了一些 json 中没有的数据类型。

例如：

```json
{"company":"Chenshi keji"}
```

### 文档的逻辑联系

假设有两个文档：

```json
# user文档
{
   "name": "Tom Hanks",
   "contact": "987654321",
   "dob": "01-01-1991"
}

# address文档
{
   "building": "22 A, Indiana Apt",
   "pincode": 123456,
   "city": "chengdu",
   "state": "sichuan"
}
```

关系 1：嵌入式关系：把 address 文档嵌入到 user 文档中

```json
# 这就是嵌入式的关系
{
   "name": "Tom Hanks",
   "contact": "987654321",
   "dob": "01-01-1991",
   "address":
   [{
   "building": "22 A, Indiana Apt",
   "pincode": 123456,
   "city": "chengdu",
   "state": "sichuan"
   },
   {
   "building": "170 A, Acropolis Apt",
   "pincode": 456789,
   "city": "beijing",
   "state": "beijing"
   }]
}
```

关系 2：引用式关系：将两个文档分开，通过引用文档的_id 字段来建立关系

```json
# 这就是引用式关系
{
   "name": "Tom Benzamin",
   "contact": "987654321",
   "dob": "01-01-1991",
   "address_ids": [
      ObjectId("52ffc4a5d85242602e000000")    #对应address文档的id字段
   ]
}
```

在实际应用的时候，嵌入式关系比较适合一对一的关系，引用式关系比较适合一对多或者多对多的情况。

### 集合

集合就是一组文档的组合，就相当于是**关系数据库中的表**，在 MongoDB 中可以存储不同的文档结构的文档。

例如：

```json
{"company":"Chenshi keji"} {"people":"man","name":"peter"}
```

上面两个文档就可以存储在同一个集合中，在关系型数据库中是很难实现上述数据结构的，要么需要定义大量的字段，对于一些字段名不确定的属性，关系型数据库会更加力不从心。

MongoDB 的做法也不是完美的。比如在存储用户信息的时候，用户名和密码分别用 username 和 password 字段表示。 关系型数据库只需要把字段名作为表结构的一部分保存起来就可以了，而 MongoDB 需要将这两个字段名存储多次，每一条记录都会存储一次字段名，在一些原本就比较碎片化的字段上，用于存储字段名所耗费的空间甚至会超过存储数值的空间，比较好的解决办法是采用尽可能短的字段名，不过这又涉及到了可读性的问题，需要大家在二者之间进行权衡。

### 元数据

数据库的信息存储在集合中，他们统一使用系统的命名空间：`DBNAME.system.*`。

DBNAME 可用 db 或数据库名替代：

- DBNAME.system.namespaces ：列出所有名字空间
- DBNAME.system.indexs ：列出所有索引
- DBNAME.system.profile ：列出数据库概要信息
- DBNAME.system.users ：列出访问数据库的用户
- DBNAME.system.sources ：列出服务器信息

## 语法

### 创建/销毁数据库

启动服务后，进入 MongoDB 命令行操作界面：

```bash
mongo
```

使用 use 命令创建数据库：

```bash
use mydb
```

查看当前连接的数据库：

```bash
db
```

查看所有的数据库：

```bash
show dbs
```

列出的所有数据库中看不到 mydb 或者显示 mydb(empty) ，因为 mydb 为空，里面没有任何东西，MongoDB 不显示或显示 mydb(empty)。

**销毁数据库**

使用 db.dropDatabase() 销毁数据库：

```bash
> use local
switched to db local
> db.dropDatabase()
```

查看所有的数据库：

```bash
> show dbs
```

可以发现 local 数据库已经被删除了。

### 创建/删除集合

**语法：`db.createCollection(name,options)`**

参数描述：

- name：创建的集合名称
- options：文档初始化选项

```shell
> db.createCollection("x") #无参数
{ "ok" : 1 }
> show collectionsx
x
system.indexes
> db.createCollection("y", { capped : 1, autoIndexId : 1, size : 6142800, max : 10000 } ) #带参数
{ "ok ": 1 }
```

参数描述：

- `capped`：类型为 Boolean，如果为 true 则创建一个固定大小的集合，当其条目达到最大时可以自动覆盖以前的条目。在设置其为 true 时也要指定参数大小；
- `autoIndexId`：类型为 Boolean，默认为 false，如果设置为 true，则会在 _id 字段上自动创建索引；
- `size`：如果 capped 为 true 则需要指定，指定参数的最大值，单位为 byte；
- `max`：指定最大的文档数。

在 Mongodb 中也可以不用创建集合，因为在创建文档的时候也会自动的创建集合。

查看创建的集合：

```bash
> show collections
```

**删除集合**

删除集合的方法如下：（删除 users 集合）

```bash
> show collections
> db.users.drop()
```

查看是否删除成功：

```bash
> show collections
```

### 插入数据

**使用 insert()**

插入数据时，如果 users 集合没有创建会自动创建。

```bash
> use mydb
switched to db mydb
> db.users.insert([
... { name : "jam",
... email : "jam@qq.com"
... },
... { name : "tom",
... email : "tom@qq.com"
... }
... ])
```

**使用 save()**

插入数据时，如果 users 集合没有创建会自动创建。

```bash
> use mydb2
switched to db mydb2
> db.users.save([
... { name : "jam",
... email : "jam@qq.com"
... },
... { name : "tom",
... email : "tom@qq.com"
... }
... ])
```

insert 和 save 的区别：为了方便记忆，可以先从字面上进行理解，insert 是插入，侧重于新增一个记录的含义；save 是保存，可以保存一个新的记录，也可以保存对一个记录的修改。因此，insert 不能插入一条已经存在的记录，如果已经有了一条记录(以主键为准)，insert 操作会报错，而使用 save 指令则会更新原记录。

### 查询数据

**find() 语句**

find() 用法：`db.COLLECTION_NAME.find()`

```bash
> use post
> db.post.insert([
{
   title: 'MongoDB Overview',
   description: 'MongoDB is no sql database',
   likes: 100
},
{
   title: 'NoSQL Database',
   description: "NoSQL database doesn't have tables",
   likes: 20,
   comments: [
      {
         user:'user1',
         message: 'My first comment',
         dateCreated: new Date(2013,11,10,2,35),
         like: 0
      }
   ]
}
])
```

查询数据，不加任何参数默认返回所有数据记录：

```
> db.post.find()
```

这条语句会返回 post 集合中的所有文档，实际应用中不常见，因为这样会导致大量的数据传输，造成服务器响应迟缓甚至失去响应。

**pretty() 语句**

pretty() 可以使查询输出的结果更美观。

```shell
> db.post.find().pretty()
```

```shell
> db.users.find()
{ "_id" : ObjectId("5c35657e6ee1e0307e215fc8"), "name" : "lsl", "age" : "21" }

> db.users.find().pretty()
{
    "_id" : ObjectId("5c35657e6ee1e0307e215fc8"),
    "name" : "lsl",
    "age" : "21"
}
```

如果你想让 mongo shell 始终以 pretty 的方式显示返回数据，可以通过下面的指令实现：

```shell
echo "DBQuery.prototype._prettyShell = true" >> ~/.mongorc.js
```

这样就把默认的显示方式设置为 pretty 了。

```shell
> db.users.find()
{
    "_id" : ObjectId("5c35657e6ee1e0307e215fc8"),
    "name" : "bage",
    "age" : "21"
}
```

**条件查询**

MongoDB 不需要类似于其他数据库的 AND 运算符，当 find() 中传入多个键值对时，MongoDB 就会将其作为 AND 查询处理。

用法：`db.mycol.find({ key1: value1, key2: value2 }).pretty()`

```shell
> db.post.find({"by":"小明","to": "小华"}).pretty()
```

如上语句就可以查找出 by 字段为 'lsl'，to 字段为 'chenshi' 的所有记录，意思是找出系统中由小明发送给 小华 的所有邮件。

它对应的关系型 SQL 语句为：

```sql
SELECT * FROM post WHERE by = '小明' AND to = '小华'
```

MongoDB 中，OR 查询语句以 `$or` 作为关键词，用法如下：

```bash
> db.post.find(
  {
    $or: [
      {key1: value1}, {key2:value2}
    ]
  }
).pretty()
```

操作示例：

```bash
> db.post.find({
    $or:[
        {"by":"小华"},
        {"title": "MongoDB Overview"}
    ]
}).pretty()
```

它对应的关系型 SQL 语句为：

```
SELECT * FROM post WHERE by = '小华' OR title = 'MongoDB Overview'
```

**条件运算符**

`{$gt:10}` 表示大于 10，另外，`$lt` 表示小于、`$gte` 表示大于等于、`$lte` 表示小于等于、`$ne` 表示不等于。

- `gt`：大于 greater than
- `lt`：小于 less than
- `gte`：大于或等于 greater than equal
- `lte`：小于或等于 less than equal

```shell
> db.post.find({
    "likes": {$gt:10},
    $or: [
        {"by": "小华"},
        {"title": "MongoDB Overview"}
    ]
}).pretty()
```

**模糊查询**

MongoDB 的模糊查询可以用正则匹配的方式实现

```shell
# 以 'start' 开头的匹配式：
{"name":/^start/}

# 以 'tail' 结尾的匹配式：
{"name":/tail$/}
```

### 高级查询

mongdb通过内置类型编号来进行数据类型的查询

语法：`$type:[key]`

可选的 key 值如下：

- 1: 双精度型(Double)
- 2: 字符串(String)
- 3: 对象(Object)
- 4: 数组(Array)
- 5: 二进制数据(Binary data)
- 7: 对象 ID(Object id)
- 8: 布尔类型(Boolean)
- 9: 日期(Date)
- 10: 空(Null)
- 11: 正则表达式(Regular Expression)
- 13: JS 代码(Javascript)
- 14: 符号(Symbol)
- 15: 有作用域的 JS 代码(JavaScript with scope)
- 16: 32 位整型数(32-bit integer)
- 17: 时间戳(Timestamp)
- 18: 64 位整型数(64-bit integer)
- -1: 最小值(Min key)
- 127: 最大值(Max key)

范例：

```shell
> db.lsl.find({"name":{$type:2}})
```

上面的命令是用于查找 name 是字符串的文档记录，它等同于下面的命令：

```shell
> db.lsl.find({"name":{$type:'string'}})
```

**limit() 与 skip()**

 `limit()`用于读取指定数量的数据记录

```bash
> db.lsl.find().limit(1)
```

读取一条记录，默认是排在最前面的那一条被读取。

 `skip()`用于读取时跳过指定数量的数据记录

```bash
> db.lsl.find().limit(1).skip(1)
```

**sort()**

升序用 1 表示，降序用 -1 表示。

**语法：`db.COLLECTION_NAME.find().sort({KEY:1|-1})`**

```bash
> db.lsl.find().sort({"time":1})
```

上述示例即按时间进行升序排列

### 更新数据

**语法：`db.COLLECTION_NAME.update(SELECTION_CRITERIA,UPDATED_DATA)`**

```bash
> db.lsl.update({"user_id":2,"e-mail":"test@qq.com"},{$set:{"e-mail":"group@qq.com"}})
WriteResult({"nMatched":1,"nUpserted":1,"nModified":1})
> db.lsl.find()
```

- 将 user_id=2 的文档的 e-mail 改为 [group@qq.com](mailto:group@qq.com)
- 第一个大括号内容标示查找条件，第二个大括号内容则表示更新后的数据
- 默认的 update 函数只对一个文档更新，如果想作用所有文档，则需要加入 multi:true

```bash
db.lsl.update({"e-mail":"test@qq.com"},{$set:{"e-mail":"group@qq.com"}},{multi:true})
```

更新文档数据可以使用save()语句，相当于替换

**语法：`db.COLLECTION_NAME.save({_id:ObjectId(),NEW_DATA})`**

```bash
>db.lsl.save({"_id":ObjectId("53ea174ccb4c62646d9544f4"),"name":"Bob","position":"techer"})
WriteResult({"nMatched":1,"nUpserted":1,"nModified":1})
```

这里的 _id 对应的是要替换文档的 _id。跟 insert 差不多，但是 save 更好用。

### 删除数据

**语法：`db.COLLECTION_NAME.remove(DELECTION_CRITERIA)`**

```bash
> db.lsl.remove({"name":"Bob"})
WriteResult({"nRemoved":1})
```

其实 remove 函数的参数跟 update 函数的第一个参数一样，相当于查找条件。

索引通常能够极大的提高查询的效率，如果没有索引，MongoDB 在读取数据时必须扫描集合中的每个文件并选取那些符合查询条件的记录。这种扫描全集合的查询效率是非常低的，特别在处理大量的数据时，查询可能要花费几十秒甚至几分钟，这无疑对网站的性能是非常致命的。

索引是特殊的数据结构，索引存储在一个易于遍历读取的数据集合中，索引是对数据库集合中一个文档或多个文档的值进行排序的一种结构。

语法：`db.COLLECTION_NAME.ensureIndex({KEY:1|-1})`

同样 1 代表升序，-1 代表降序。

范例：

```bash
> db.lsl.ensureIndex({"name":1})
```

`ensureIndex()` 的可选参数：

| 参数               | 类型          | 描述                                           |
| ------------------ | ------------- | ---------------------------------------------- |
| background         | Boolean       | 建立索引要不要阻塞其他数据库操作，默认为 false |
| unique             | Boolean       | 建立的索引是否唯一，默认 false                 |
| name               | string        | 索引的名称，若未指定，系统自动生成             |
| dropDups           | Boolean       | 建立唯一索引时，是否删除重复记录，默认 flase   |
| sparse             | Boolean       | 对文档不存在的字段数据不启用索引，默认 false   |
| expireAfterSeconds | integer       | 设置集合的生存时间，单位为秒                   |
| v                  | index version | 索引的版本号                                   |
| weights            | document      | 索引权重值，范围为 1 到 99999                  |
| default-language   | string        | 默认为英语                                     |
| language_override  | string        | 默认值为 language                              |

范例：

```bash
> db.lsl.ensureIndex({"user_id":1,"name":1},{background:1})
```

### 索引

索引通常能够极大的提高查询的效率，如果没有索引，MongoDB 在读取数据时必须扫描集合中的每个文件并选取那些符合查询条件的记录。这种扫描全集合的查询效率是非常低的，特别在处理大量的数据时，查询可能要花费几十秒甚至几分钟，这无疑对网站的性能是非常致命的。

索引是特殊的数据结构，索引存储在一个易于遍历读取的数据集合中，索引是对数据库集合中一个文档或多个文档的值进行排序的一种结构。

**语法：`db.COLLECTION_NAME.ensureIndex({KEY:1|-1})`**

同样 1 代表升序，-1 代表降序。

```bash
> db.lsl.ensureIndex({"name":1})
```

`ensureIndex()` 的可选参数：

| 参数               | 类型          | 描述                                           |
| ------------------ | ------------- | ---------------------------------------------- |
| background         | Boolean       | 建立索引要不要阻塞其他数据库操作，默认为 false |
| unique             | Boolean       | 建立的索引是否唯一，默认 false                 |
| name               | string        | 索引的名称，若未指定，系统自动生成             |
| dropDups           | Boolean       | 建立唯一索引时，是否删除重复记录，默认 flase   |
| sparse             | Boolean       | 对文档不存在的字段数据不启用索引，默认 false   |
| expireAfterSeconds | integer       | 设置集合的生存时间，单位为秒                   |
| v                  | index version | 索引的版本号                                   |
| weights            | document      | 索引权重值，范围为 1 到 99999                  |
| default-language   | string        | 默认为英语                                     |
| language_override  | string        | 默认值为 language                              |

```bash
> db.lsl.ensureIndex({"user_id":1,"name":1},{background:1})
```

### 聚合与管道

聚合即分组聚合，对于指定符合条件的集合中的所有数据进行一次聚合运算

```bash
db.COLLECTION_NAME.aggregate({
$match:{x:1},
{limit:NUM},
$group:{_id:$age}
})
```

这些参数都可选：

- `$match`：查询，跟 find 一样；
- `$limit`：限制显示结果数量；
- `$skip`：忽略结果数量；
- `$sort`：排序；
- `$group`：按照给定表达式组合结果。

```bash
> db.lsl.aggregate([{$group:{_id:"$name", user:{$sum:"$user_id"}}}])
```

`$name` 意为取得 name 的值。

可用的聚合表达式有：

| 名称      | 描述                                       |
| --------- | ------------------------------------------ |
| $sum      | 计算总和                                   |
| $avg      | 计算平均值                                 |
| min 和max | 计算最小值和最大值                         |
| $push     | 在结果文档中插入值到一个数组               |
| $addToSet | 在结果文档中插入值到一个数组，但不创建副本 |
| $first    | 根据资源文档的排序获取第一个文档数据       |
| $last     | 根据资源文档的排序获取最后一个文档数据     |

MongoDB 的聚合管道将 MongoDB 文档在一个管道处理完毕后将结果传递给下一个管道处理。管道操作是可以重复的。

表达式：处理输入文档并输出。表达式是无状态的，只能用于计算当前聚合管道的文档，不能处理其它的文档。

聚合框架中常用的几个操作：

- `$project`：修改输入文档的结构。可以用来重命名、增加或删除域，也可以用于创建计算结果以及嵌套文档。
- `$match`：用于过滤数据，只输出符合条件的文档。`$match` 使用 MongoDB 的标准查询操作。
- `$limit`：用来限制 MongoDB 聚合管道返回的文档数。
- `$skip`：在聚合管道中跳过指定数量的文档，并返回余下的文档。
- `$unwind`：将文档中的某一个数组类型字段拆分成多条，每条包含数组中的一个值。
- `$group`：将集合中的文档分组，可用于统计结果。
- `$sort`：将输入文档排序后输出。
- `$geoNear`：输出接近某一地理位置的有序文档。

```bash
> db.lsl.aggregate([{$match:{user_id:{$gt:0,$lte:2}}},{$group:{_id:"user",count:{$sum:1}}}])
{"_id":"user","count":2}
```

### 原子操作

常用原子操作命令：

$set：用来指定一个键并更新键值，若键不存在则创建。

```bash
{ $set : { field : value } }
```

$unset：用来删除一个键。

```bash
{ $unset : { field : 1} }
```

$inc：\$inc 可以对文档的某个值为数字型（只能为满足要求的数字）的键进行增减的操作。

```bash
{ $inc : { field : value } }
```

$push：把 value 追加到 field 里面去，field 一定要是数组类型才行，如果 field 不存在，会新增一个数组类型加进去。

```bash
{ $push : { field : value } }
```

$pushAll：同 \$push ，只是一次可以追加多个值到一个数组字段内。

```bash
{ $pushAll : { field : value_array } }
```

$pull：从数组 field 内删除一个等于 value 值。

```bash
{ $pull : { field : _value } }
```

$addToSet：增加一个值到数组内，而且只有当这个值不在数组内才增加。

$pop：删除数组的第一个或最后一个元素。

```bash
{ $pop : { field : 1 } }
```

$rename：修改字段名称:

```bash
{ $rename : { old_field_name : new_field_name } }
```

$bit：位操作，integer 类型

```bash
{$bit : { field : {and : 5}}}
```