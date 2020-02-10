### NoSQL

not only SQL，一种**基于内存的数据库**，并且提供一定的持久化功能。

**Redis**和**MongoDB**是当前使用最广泛的NoSQL，就Redis技术而言，可以**支持每秒十几万此的读/写操作**，其性能远超数据库，并且还**支持集群、分布式、主从同步等**配置，原则上可以无限扩展，让更多的数据存储在内存中。

 主要有两个应用场景：

- 存储**缓存**用的数据；
- 需要高速读/写的场合**使用它快速读/写**；

简单的说， Redis 是一个基于内存的高性能key-value数据库。 

### Redis

#### Redis 安装使用

访问地址：[https://github.com/ServiceStack/redis-windows/tree/master/downloads](https://link.zhihu.com/?target=https%3A//github.com/ServiceStack/redis-windows/tree/master/downloads)

下载解压，几个重要的配置文件为：

- 服务端：src/redis-server
- 客户端：src/redis-cli
- 默认配置文件：redis.conf

在Windows下为了方便启动，可以在该目录下新建一个 startup.cmd 的文件，然后将以下内容写入文件：

```text
redis-server redis.windows.conf
```

这个命令其实就是在调用 redis-server.exe 命令来读取 redis.window.conf 的内容。

在linux下则可以直接将命令放入bin中：

```shell
cp redis-server /usr/local/bin/
cp redis-cli /usr/local/bin/
```

执行redis-server启动服务端，执行redis-cli启动客户端（用于连接服务器），成功则显示相应信息

如果出现中文乱码的情况，可以使用下列命令启动 redis 客户端：

```
redis-cli --raw
```

#### Redis操作语法

##### 数据类型

字符串 string 

字符串列表 lists

Hashes

集合 set

有序集合 ordered set

##### string：set/incr/incrby

set用于增加k-v，如set mykey myvalue即增加一个k-v，可以一次增加多个，如 `set a 1 b 2 c 3`，默认的值类型为string，incr/incrby用于增加值：

```
> set counter 100 # 初始化
> incr counter   # +1
> incr counter   # +1
> incrby counter 50 # +50 自定义计数
```

##### lists：rpush/lpush/lrange/lpop/rpop/blpop/brpop

用于向列表中添加元素。（list很适用于存储聊天记录等线性元素）

 lpush 命令插入一个新的元素到头部，而 rpush 命令插入一个新元素到尾部(left与right)。 可以以此添加多个元素

```
> rpush mylist A
> rpush mylist B
> lpush mylist first
> rpush mylist 1 2 3 4 5 "foo bar"
> lrange mylist 0 -1
```

lrange 返回列表范围内的元素，负数代表从后往前，正向索引从0开始

lpop与rpop用于弹出元素

blpop与brpop用于阻塞式弹出元素

##### Hashes：hmset/hincrby/hget/hgetall

向Hashes中添加元素，即用于存储多个字符串字段（一个字符串映射的多个k-y），第一个为关键字段，后面为附加字段

```
> hmset user:1000 username antirez birthyear 1977 verified 1
> hget user:1000 username
> hget user:1000 birthyear
> hgetall user:1000
```

查找Hashes，hget查找某个属性值，hgetall查找所有属性值

##### Set：sadd/smembers

用于向集合中添加元素

```
> sadd myset 1 2 3
> smembers myset
```

sadd 命令产生一个无序集合，返回集合的元素个数。smembers 用于查看集合。

sismember 用于查看集合是否存在，匹配项包括集合名和元素个数。匹配成功返回 1，匹配失败返回 0。

```
> sismember myset 3
> sismember myset 30
```

##### ZSet：zadd/zrange/zrevrange

zadd 与 sadd 类似，但是在元素之前多了一个参数hackers，这个参数便是用于排序的。按key排序。

```
> zadd hackers 1940 "Alan Kay"
> zadd hackers 1957 "Sophie Wilson"
> zadd hackers 1953 "Richard Stallman"
```

查看集合：zrange 是查看正序的集合，zrevrange 是查看反序的集合。0 表示集合第一个元素，-1 表示集合的倒数第一个元素。

```
> zrange hackers 0 -1
> zrevrange hackers 0 -1
```

使用 withscores 参数返回key值。

```
> zrange hackers 0 -1 withscores
```

#### Reids常用命令

`exists` key：判断一个 key 是否存在，存在返回 1，否则返回 0。

`del` key：删除某个 key，或是一系列 key，比如：del key1 key2 key3 key4。成功返回 1，失败返回 0（key 值不存在）。

```
> set mykey hello
> exists mykey
> del mykey
> exists mykey
```

`type` key：返回某个 key 元素的数据类型（none：不存在，string：字符，list：列表，set：元组，zset：有序集合，hash：哈希），key 不存在返回空。

`keys` key—pattern：返回匹配的 key 列表，比如：keys foo* 表示查找 foo 开头的 keys。

`randomkey`：随机获得一个已经存在的 key，如果当前数据库为空，则返回空字符串。

```
> randomkey
```

`flushdb`：清空当前数据库中的所有键。

`flushall`：清空所有数据库中的所有键。

#### 时间相关

 `expire`：设置某个 key 的过期时间（秒），比如：expire bruce 1000 表示设置 bruce 这个 key 1000 秒后系统自动删除，注意：如果在还没有过期的时候，对值进行了改变，那么那个值会被清除。 

限时操作可以在 set 命令中实现，并且可用 ttl 命令查询 key 剩余生存时间。

`ttl`：查找某个 key 还有多长时间过期，返回时间单位为秒。

#### 设置

Redis 有其配置文件，可以通过 client-command 窗口查看或者更改相关配置。下面介绍相关命令。

`config get`：用来读取运行 Redis 服务器的配置参数。

`config set`：用于更改运行 Redis 服务器的配置参数。

`auth`：认证密码。

下面针对 Redis 密码的示例：

```
> config get requirepass  # 查看密码
> config set requirepass test123  # 设置密码为 test123
> config get requirepass  # 报错，没有认证
> auth test123  # 认证密码
> config get requirepass
```

刚开始时 Reids 并未设置密码，密码查询结果为空。然后设置密码为 test123，再次查询报错。经过 auth 命令认证后，可正常查询。

可以通过修改 Redis 的配置文件 redis.conf 修改密码。

`config get` 命令是以 list 的 key-value 对显示的，如查询数据类型的最大条目：

```
> config get *max-*-entries*
```

`config resetstat`：重置数据统计报告，通常返回值为“OK”。

```
> CONFIG RESETSTAT
```

#### 查询info

`info [section]`：查询 Redis 相关信息。

info 命令可以查询 Redis 几乎所有的信息，其命令选项有如下：

- server: Redis server 的常规信息
- clients: Client 的连接选项
- memory: 存储占用相关信息
- persistence: RDB and AOF 相关信息
- stats: 常规统计
- replication: Master/Slave 请求信息
- cpu: CPU 占用信息统计
- cluster: Redis 集群信息
- keyspace: 数据库信息统计
- all: 返回所有信息
- default: 返回常规设置信息

若命令参数为空，info 命令返回所有信息。

```
> info keyspace
> info server
```

### Redis高级特性

#### 主从复制

为了分担服务器压力，会在特定情况下部署多台服务器分别用于缓存的读和写操作，用于写操作的服务器称为主服务器，用于读操作的服务器称为从服务器。

从服务器通过 psync 操作同步主服务器的写操作，并按照一定的时间间隔更新主服务器上新写入的内容。

Redis 主从复制的过程：

1. Slave 与 Master 建立连接，发送 psync 同步命令。
2. Master 会启动一个后台进程，将数据库快照保存到文件中，同时 Master 主进程会开始收集新的写命令并缓存。
3. 后台完成保存后，就将此文件发送给 Slave。
4. Slave 将此文件保存到磁盘上。

Redis 主从复制特点：

1. 可以拥有多个 Slave。
2. 多个 Slave 可以连接同一个 Master 外，还可以连接到其它的 Slave。（当 Master 宕机后，相连的 Slave 转变为 Master）。
3. 主从复制不会阻塞 Master，在同步数据时， Master 可以继续处理 Client 请求。
4. 提高了系统的可伸缩性。

从服务器的主要作用是响应客户端的数据请求，比如返回一篇博客信息。

上面说到了主从复制是不会阻塞 Master 的，就是说 Slave 在从 Master 复制数据时，Master 的删改插入等操作继续进行不受影响。

如果在同步过程中，主服务器修改了一篇博客，而同步到从服务器上的博客是修改前的。这时候就会出现时间差，即修改了博客过后，在访问网站的时候还是原来的数据，这是因为从服务器还未同步最新的更改，这也就意味着非阻塞式的同步只能应用于对读数据延迟接受度较高的场景。

要建立这样一个主从关系的缓存服务器，只需要在 Slave 端执行命令:

```
# SLAVEOF IPADDRESS:PORT
> SLAVEOF 127.0.0.1:6379
```

如果主服务器设置了连接密码，就需要在从服务器中事先设置好：

```
config set masterauth <password>
```

这样，当前服务器就作为 127.0.0.1:6379 下的一个从服务器，它将定期从该服务器复制数据到自身。

#### 事务处理

Redis 的事务处理比较简单。只能保证 client 发起的事务中的命令可以连续的执行，而且不会插入其它的 client 命令，当一个 client 在连接中发出 `multi` 命令时，这个连接就进入一个事务的上下文，该连接后续的命令不会执行，而是存放到一个队列中，当执行 `exec` 命令时，redis 会顺序的执行队列中的所有命令。

```
> multi
> set name a
> set name b
> exec
> get name
```

 需要注意的是，redis 对于事务的处理方式比较特殊，它不会在事务过程中出错时恢复到之前的状态，这在实际应用中导致我们不能依赖 redis 的事务来保证数据一致性。 

#### 持久化机制

Redis 是一个支持持久化的内存数据库，Redis 需要经常将内存中的数据同步到磁盘来保证持久化。

Redis 支持两种持久化方式：

1、`snapshotting`（快照）：将数据存放到文件里，默认方式。

是将内存中的数据以快照的方式写入到二进制文件中，默认文件 dump.rdb，可以通过配置设置自动做快照持久化的方式。可配置 Redis 在 n 秒内如果超过 m 个 key 被修改就自动保存快照。比如：

`save 900 1`：900 秒内如果超过 1 个 key 被修改，则发起快照保存。

`save 300 10`：300 秒内如果超过 10 个 key 被修改，则快照保存。

2、`Append-only file`（缩写为 aof）：将读写操作存放到文件中。

由于快照方式在一定间隔时间做一次，所以如果 Redis 意外 down 掉的话，就会丢失最后一次快照后的所有修改。

aof 比快照方式有更好的持久化性，使用 aof 时，redis 会将每一个收到的写命令都通过 write 函数写入到文件中，当 redis 启动时会通过重新执行文件中保存的写命令来在内存中重新建立整个数据库的内容。

由于 os 会在内核中缓存 write 做的修改，所以可能不是立即写到磁盘上，这样 aof 方式的持久化也还是有可能会丢失一部分数据。可以通过配置文件告诉 redis 我们想要通过 fsync 函数强制 os 写入到磁盘的时机。

配置文件中的可配置参数：

```
appendonly yes //启用 aof 持久化方式
# appendfsync always //收到写命令就立即写入磁盘，最慢，但是保证了数据的完整持久化
appendfsync everysec //每秒钟写入磁盘一次，在性能和持久化方面做了很好的折中
# appendfsync no //完全依赖 os，性能最好，持久化没有保证
```

在 redis-cli 的命令中，`save` 命令是将数据写入磁盘中。

```
> help save
> save
```

### 在 Java 中使用 Redis

导入依赖

```java
@Test
public void redisTester() {
	Jedis jedis = new Jedis("localhost", 6379, 100000);
	int i = 0;
	try {
		long start = System.currentTimeMillis();// 开始毫秒数
		while (true) {
			long end = System.currentTimeMillis();
			if (end - start >= 1000) {// 当大于等于1000毫秒（相当于1秒）时，结束操作
				break;
			}
			i++;
			jedis.set("test" + i, i + "");
		}
	} finally {// 关闭连接
		jedis.close();
	}
	// 打印1秒内对Redis的操作次数
	System.out.println("redis每秒操作：" + i + "次");
}
-----------测试结果-----------
redis每秒操作：10734次
```

据说 Redis 的性能能达到十万级别，我不敢相信我的台式机电脑只有十分之一不到的性能，虽然说这里不是流水线的操作，会造成一定的影响，但我还是不信邪，我查到了官方的性能测试方法：

#### 使用 Redis 连接池

跟数据库连接池相同，Java Redis也同样提供了类`redis.clients.jedis.JedisPool`来管理我们的Reids连接池对象，并且我们可以使用`redis.clients.jedis.JedisPoolConfig`来对连接池进行配置，代码如下：

```xml
JedisPoolConfig poolConfig = new JedisPoolConfig();
// 最大空闲数
poolConfig.setMaxIdle(50);
// 最大连接数
poolConfig.setMaxTotal(100);
// 最大等待毫秒数
poolConfig.setMaxWaitMillis(20000);
// 使用配置创建连接池
JedisPool pool = new JedisPool(poolConfig, "localhost");
// 从连接池中获取单个连接
Jedis jedis = pool.getResource();
// 如果需要密码
//jedis.auth("password");
```

Redis 只能支持六种数据类型（string/hash/list/set/zset/hyperloglog）的操作，但在 Java 中我们却通常以类对象为主，所以在需要 Redis 存储的五中数据类型与 Java 对象之间进行转换，如果自己编写一些工具类，比如一个角色对象的转换，还是比较容易的，但是涉及到许多对象的时候，这其中无论工作量还是工作难度都是很大的，所以总体来说，**就操作对象而言，使用 Redis 还是挺难的**，好在 Spring 对这些进行了封装和支持。

#### 在 Spring 中使用 Redis

上面说到了 Redis 无法操作对象的问题，无法在那些基础类型和 Java 对象之间方便的转换，但是在 Spring 中，这些问题都可以**通过使用RedisTemplate**得到解决！

想要达到这样的效果，除了 Jedis 包以外还需要在 Spring 引入 spring-data-redis 包：[https://mvnrepository.com/artifact/org.springframework.data/spring-data-redis](https://link.zhihu.com/?target=https%3A//mvnrepository.com/artifact/org.springframework.data/spring-data-redis)

**（1）第一步：使用Spring配置JedisPoolConfig对象**

大部分的情况下，我们还是会用到连接池的，于是先用 Spring 配置一个 JedisPoolConfig 对象：

```text
<bean id="poolConfig" class="redis.clients.jedis.JedisPoolConfig">
    <!--最大空闲数-->
    <property name="maxIdle" value="50"/>
    <!--最大连接数-->
    <property name="maxTotal" value="100"/>
    <!--最大等待时间-->
    <property name="maxWaitMillis" value="20000"/>
</bean>
```

**（2）第二步：为连接池配置工厂模型**

好了，我们现在配置好了连接池的相关属性，那么具体使用哪种工厂实现呢？在Spring Data Redis中有四种可供我们选择的工厂模型，它们分别是：

- JredisConnectionFactory
- JedisConnectionFactory
- LettuceConnectionFactory
- SrpConnectionFactory

我们这里就简单配置成JedisConnectionFactory：

```xml
<bean id="connectionFactory" class="org.springframework.data.redis.connection.jedis.JedisConnectionFactory">
    <!--Redis服务地址-->
    <property name="hostName" value="localhost"/>
    <!--端口号-->
    <property name="port" value="6379"/>
    <!--如果有密码则需要配置密码-->
    <!--<property name="password" value="password"/>-->
    <!--连接池配置-->
    <property name="poolConfig" ref="poolConfig"/>
</bean>
```

**（3）第三步：配置RedisTemplate**

普通的连接根本没有办法直接将对象直接存入 Redis 内存中，我们需要替代的方案：将对象序列化（可以简单的理解为继承Serializable接口）。我们可以把对象序列化之后存入Redis缓存中，然后在取出的时候又通过转换器，将序列化之后的对象反序列化回对象，这样就完成了我们的要求：



![img](https://pic3.zhimg.com/v2-7c367843671ba1684a1e06bfd5eb746a_b.jpg)



RedisTemplate可以帮助我们完成这份工作，它会找到对应的序列化器去转换Redis的键值：

```xml
<bean id="redisTemplate"
      class="org.springframework.data.redis.core.RedisTemplate"
      p:connection-factory-ref="connectionFactory"/>
```

> 笔者从《JavaEE互联网轻量级框架整合开发》中了解到，这一步需要配置单独的序列化器去支撑这一步的工作，但是自己在测试当中，发现只要我们的POJO类实现了Serializable接口，就不会出现问题…所以我直接省略掉了配置序列化器这一步…

**（4）第四步：编写测试**

首先编写好支持我们测试的POJO类：

```java
/**
 * @author: @我没有三颗心脏
 * @create: 2018-05-30-下午 22:31
 */
public class Student implements Serializable{

	private String name;
	private int age;

	/**
	 * 给该类一个服务类用于测试
	 */
	public void service() {
		System.out.println("学生名字为：" + name);
		System.out.println("学生年龄为：" + age);
	}

	public String getName() {
		return name;
	}

	public void setName(String name) {
		this.name = name;
	}

	public int getAge() {
		return age;
	}

	public void setAge(int age) {
		this.age = age;
	}
}
```

然后编写测试类：

```java
@Test
public void test() {
	ApplicationContext context =
			new ClassPathXmlApplicationContext("applicationContext.xml");
	RedisTemplate redisTemplate = context.getBean(RedisTemplate.class);
	Student student = new Student();
	student.setName("我没有三颗心脏");
	student.setAge(21);
	redisTemplate.opsForValue().set("student_1", student);
	Student student1 = (Student) redisTemplate.opsForValue().get("student_1");
	student1.service();
}
```

运行可以成功看到结果：



![img](https://pic1.zhimg.com/v2-3d3c0c4272b2bce302371c768b841ff8_b.jpg)

#### 在 SpringBoot 中使用 Redis

**（1）在SpringBoot中添加Redis依赖：**

```xml
<!-- Radis -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

**（2）添加配置文件：**

在SpringBoot中使用`.properties`或者`.yml`都可以，这里给出`.properties`的例子，因为自己的`.yml`文件看上去感觉乱糟糟的：

```properties
# REDIS (RedisProperties)
# Redis数据库索引（默认为0）
spring.redis.database=0
# Redis服务器地址
spring.redis.host=localhost
# Redis服务器连接端口
spring.redis.port=6379
# Redis服务器连接密码（默认为空）
spring.redis.password=
# 连接池最大连接数（使用负值表示没有限制）
spring.redis.pool.max-active=8
# 连接池最大阻塞等待时间（使用负值表示没有限制）
spring.redis.pool.max-wait=-1
# 连接池中的最大空闲连接
spring.redis.pool.max-idle=8
# 连接池中的最小空闲连接
spring.redis.pool.min-idle=0
# 连接超时时间（毫秒）
spring.redis.timeout=0
```

**（3）测试访问：**

```java
@RunWith(SpringJUnit4ClassRunner.class)
@SpringBootTest()
public class ApplicationTests {

	@Autowired
	private StringRedisTemplate stringRedisTemplate;

	@Test
	public void test() throws Exception {

		// 保存字符串
		stringRedisTemplate.opsForValue().set("aaa", "111");
		Assert.assertEquals("111", stringRedisTemplate.opsForValue().get("aaa"));

	}
}
```

通过上面这段极为简单的测试案例演示了如何通过自动配置的**StringRedisTemplate**对象进行Redis的读写操作，该对象从命名中就可注意到支持的是String类型。原本是RedisTemplate<K, V>接口，StringRedisTemplate就相当于RedisTemplate<String, String>的实现。

运行测试，如果一切成功则不会报错，如果我们没有拿到或者拿到的数不是我们想要的 “111” ，那么则会报错，这是使用Assert的好处（下面是我改成112之后运行报错的结果）：



![img](https://pic1.zhimg.com/v2-533247dc2ffc9efb184c9d92fcba6e38_b.jpg)



**（4）存储对象：**

这一步跟上面使用Spring一样，只需要将POJO类实现**Serializable接口**就可以了，我这里就贴一下测试代码：

```java
@RunWith(SpringJUnit4ClassRunner.class)
@SpringBootTest()
public class ApplicationTests {

	@Autowired
	private RedisTemplate redisTemplate;

	@Test
	public void test() throws Exception {

		User user = new User();
		user.setName("我没有三颗心脏");
		user.setAge(21);

		redisTemplate.opsForValue().set("user_1", user);
		User user1 = (User) redisTemplate.opsForValue().get("user_1");

		System.out.println(user1.getName());
	}
}
```

仍然没有任何问题：



![img](https://pic3.zhimg.com/v2-6550d649a402a673033459d3c882cb6a_b.jpg)



> 参考文章：
> 1.[https://www.cnblogs.com/ityouknow/p/5748830.html](https://link.zhihu.com/?target=https%3A//www.cnblogs.com/ityouknow/p/5748830.html)
> 2.[http://blog.didispace.com/springbootredis/](https://link.zhihu.com/?target=http%3A//blog.didispace.com/springbootredis/)

#### 在Redis中操作集合

> 引用文章：[https://www.jianshu.com/p/29aaac3172b5](https://link.zhihu.com/?target=https%3A//www.jianshu.com/p/29aaac3172b5)

直接黏上两段简单的示例代码：

#### 在Redis中操作List

```java
// list数据类型适合于消息队列的场景:比如12306并发量太高，而同一时间段内只能处理指定数量的数据！必须满足先进先出的原则，其余数据处于等待
@Test
public void listPushResitTest() {
	// leftPush依次由右边添加
	stringRedisTemplate.opsForList().rightPush("myList", "1");
	stringRedisTemplate.opsForList().rightPush("myList", "2");
	stringRedisTemplate.opsForList().rightPush("myList", "A");
	stringRedisTemplate.opsForList().rightPush("myList", "B");
	// leftPush依次由左边添加
	stringRedisTemplate.opsForList().leftPush("myList", "0");
}

@Test
public void listGetListResitTest() {
	// 查询类别所有元素
	List<String> listAll = stringRedisTemplate.opsForList().range("myList", 0, -1);
	logger.info("list all {}", listAll);
	// 查询前3个元素
	List<String> list = stringRedisTemplate.opsForList().range("myList", 0, 3);
	logger.info("list limit {}", list);
}

@Test
public void listRemoveOneResitTest() {
	// 删除先进入的B元素
	stringRedisTemplate.opsForList().remove("myList", 1, "B");
}

@Test
public void listRemoveAllResitTest() {
	// 删除所有A元素
	stringRedisTemplate.opsForList().remove("myList", 0, "A");
}
```

#### 在Redis中操作Hash

```java
@Test
public void hashPutResitTest() {
	// map的key值相同，后添加的覆盖原有的
	stringRedisTemplate.opsForHash().put("banks:12600000", "a", "b");
}

@Test
public void hashGetEntiresResitTest() {
	// 获取map对象
	Map<Object, Object> map = stringRedisTemplate.opsForHash().entries("banks:12600000");
	logger.info("objects:{}", map);
}

@Test
public void hashGeDeleteResitTest() {
	// 根据map的key删除这个元素
	stringRedisTemplate.opsForHash().delete("banks:12600000", "c");
}

@Test
public void hashGetKeysResitTest() {
	// 获得map的key集合
	Set<Object> objects = stringRedisTemplate.opsForHash().keys("banks:12600000");
	logger.info("objects:{}", objects);
}

@Test
public void hashGetValueListResitTest() {
	// 获得map的value列表
	List<Object> objects = stringRedisTemplate.opsForHash().values("banks:12600000");
	logger.info("objects:{}", objects);
}

@Test
public void hashSize() { // 获取map对象大小
	long size = stringRedisTemplate.opsForHash().size("banks:12600000");
	logger.info("size:{}", size);
}
```

