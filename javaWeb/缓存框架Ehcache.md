## ehcache介绍

在`java`中有很多技术都可以实现缓存功能，最简单直接就是使用`java`自带的`Map`容器，或者就是使用现有的缓存框架，例如`memcache`,`ehcache` ,以及非常热门的`redis`。这里介绍`ehcache`的主要是因为它真的很方便，而且`memcache`和`redis`都需要额外搭建服务，更适合分布式部署的项目以便于各个模块之间的使用共有的缓存内容。而`ehcache`主要是内存缓存，也可以缓存到磁盘中，速度快，效率高，功能也强大，适合我们一般的单个项目使用。

## spring boot 配置ehcache

**添加依赖**

在`pom.xml`中添加

```xml
<!--开启缓存-->
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-cache</artifactId>
</dependency>
<!-- EhCache -->
<dependency>
    <groupId>net.sf.ehcache</groupId>
    <artifactId>ehcache</artifactId>
</dependency>
```

**编写配置文件**

添加了依赖之后，`spring boot`会自动默认加载`src/mian/resources`目录下的`ehcache.xml`文件，所以我们需要在该目录下手动创建该文件，这里先给出一个样例：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<ehcache xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	 xsi:noNamespaceSchemaLocation="http://ehcache.org/ehcache.xsd">
	 
	 <!-- 磁盘缓存文件路径 -->
	 <diskStore path="java.io.tmpdir"/>
	 
	 <!-- 默认配置 -->
	 <defaultCache eternal="false"
	    maxElementsInMemory="1000"
	    overflowToDisk="false"
	    diskPersistent="false"
	    timeToIdleSeconds="0"
	    timeToLiveSeconds="600"
	    memoryStoreEvictionPolicy="LRU"/>
	    
	 <!-- 自定义配置 -->
	 <cache name="userCache"
	   eternal="false"
	   maxElementsInMemory="1000"
	   overflowToDisk="false"
	   diskPersistent="false"
	   timeToIdleSeconds="0"
	   timeToLiveSeconds="600"
	   memoryStoreEvictionPolicy="LRU"/>
</ehcache>
```

下面介绍样例中出现的三个节点：

- <diskStore\>：这个节点是非必须的，只有在使用了磁盘存储的情况下才需要配置，表示缓存文件在磁盘中保存的路径，该路径通过path属性来指定，磁盘缓存使用的文件后缀名是*.data和\*.index,主要有以下几个值：
  1. `user.home`:用户主目录
  2. `user.dir`:用户当前的工作目录
  3. `java.io.tmpdir`:默认临时路径
  4. `ehcache.disk.store.dir`:cache的配置目录
  5. 自定义绝对路径

如果对于这几个目录不熟悉，可以在`java`中获取，如下：

```java
public static void main(String[] args) {
	System.out.println(System.getProperty("user.home"));
	System.out.println(System.getProperty("user.dir"));
	System.out.println(System.getProperty("java.io.tmpdir"));
}
```

下面是我本机打印出来的路径，仅做参考：

```
C:\Users\Administrator
D:\Program Data\eclipse-workspace\springboot-ehcache
C:\Users\Administrator\AppData\Local\Temp\
```

> 这里需要注意一点，要想某个对象被缓存到磁盘中，需要该对象实现序列化接口。

- `<ehcache>`：自定义缓存区，可以有零个或者多个，重要属性如下：
  - `name`：缓存区名字，必须属性，用来区分缓存区的唯一标识。
  - `eternal`:设置缓存区中的内容是否永久有效，可选值`true`或`false`，如果选择`true`那么设置的`timeToIdleSeconds`以及`timeToLiveSeconds`将失效。
  - `maxElementsInMemory`:该缓存区中最多可以存放的对象数量，超过这个数量时，会根据`overflowToDisk`属性的值有不同的操作。
  - `overflowToDisk`:缓存对象超出最大数量时是否启用磁盘保存，可选值`true`或`false`，值为`true`时，会将超出的内容缓存到磁盘中，为`false`时则会根据`memoryStoreEvictionPolicy`属性配置的策略替换掉原来的内容。
  - `diskPersistent`:磁盘存储是否在虚拟机重启后持续存在，默认是`false`,如果为`true`系统在初始化时会将磁盘中的内容加载到缓存。
  - `timeToIdleSeconds`:设置一个元素在过期前的空闲时间(单位:秒)，即访问该元素的最大间隔时间，超过这个时间该元素就会被清除,默认值为`0`，表示一个元素可以无限的空闲。
  - `timeToLiveSeconds`:设置一个元素在缓存区中的生存时间(单位:秒)，即从创建到清除的时间，超过这个时间，该元素就会被清除，默认值为`0`，表示一个元素可以无限的保存。
  - `memoryStoreEvictionPolicy`:缓存存储与清除策略。即达到`maxElementsInMemory`限制并且`overflowToDisk`值为`false`时`ehcache`就会根据这个属性的值执行相应的清空策略，该属性有以下三个值分别代表`ehcache`的三种缓存清理策略，默认值为`LRU`：
    1. `FIFO`:先进先出策略`(First In First Out)`。
    2. `LFU`:最少被使用`(Less Frequently Used)`，所有的缓存元素都会有一个属性记录该元素被使用的次数，清理元素时最小的那个将会被清除。
    3. `LRU`:最近最少使用`(Least Resently Used)`,所有缓存的元素都会有一个属性记录最后一次使用的时间，清理元素时时间最早的那个元素将会被清除。
  - `diskExpiryThreadIntervalSeconds`:磁盘缓存的清理线程运行间隔,默认是120秒。
  - `diskSpoolBufferSizeMB`:设置磁盘缓存区的大小，默认为30MB。
  - `maxEntriesLocalDisk`:设置磁盘缓存区最多能存放元素的数量。
- `<defaultCache>`:默认缓存区，即是一个`name`属性为`default`的``节点，属性和``节点都一样，一个`ehcache.xml`文件中只能有一个`<defaultCache>`节点，当我们没有自定义的`<ehcache>`时，默认使用该缓存区。

> 对于defaultCache这里有需要注意的地方，因为他是一个特殊的，所以我们在自定义缓存区的时候不能再定义名为default的,并且在使用的时候也不能通过value=default来指定默认的缓存区。

这里补充一点，项目中如果不想使用默认的路径以及名字我们也可以自定义`ehcache`配置文件的名字以及路径，在`application.properties`配置文件中配置如下内容：

```
#后边的路径可以自己指定
spring.cache.ehcache.config=classpath:ehcache.xml
```

## 开启缓存

`spring boot`中开启缓存非常简单，只需要在在启动类上添加一个`@EnableCaching`注解即可。

## 使用注解

`spring boot`中使用`ehcache`缓存主要是通过注解来使用，而且我们一般在`service`实现层使用缓存功能，常用的注解如下：

### @Cacheable

该注解主要用在方法上边，每当程序进入被该注解标记的方法时，系统会首先判断缓存中是否存在相同`key`的元素，如果存在就直接返回缓存区中存放的值，**并且不会执行方法的内容**，如果不存在就执行该方法，并且判断是否需要将返回值添加到缓存区中，常用属性：

- value:指定使用哪个缓存区，就是我们在配置文件里边配置的<ehcache\>节点的name属性对应的值，可以指定多个值。

  ```java
  //指定一个
  @Cacheable(value="userCache")
  //指定多个
  @Cacheable(value={"userCache","userCache2"})
  ```

- key:缓存元素的key，需要按照SpEL表达式编写，这个我们一般按照指定方法的参数来确定。

  ```java
  //#p0表示将第一个参数当成key,也可以直接写参数名字例如：#id,两者表达意思一样
  @Cacheable(value="userCache",key="#p0")
  public SysUser getById(Integer id){//内容省略...};
  ```

- condition:添加缓存的条件，需要按照SpEL表达式编写，仅当该属性返回true时才添加缓存。

  ```java
   //仅当id>10时才缓存
   @Cacheable(value="userCache",key="#p0",condition="#p0>10")
   public SysUser getById(Integer id){//内容省略...};
  ```

### @CachePut

该注解主要用在方法上边，能够根据方法的参数以及返回值以及自定义的条件判断是否添加缓存，该注解标记的方法一定会执行，其属性与`@Cacheable`一致。

```java
@CachePut(value="userCache",key="#entity.id")
public SysUser insertSysuser(SysUser entity) {
	// TODO Auto-generated method stub		
    //省略内容
}
```

### @CacheEvict

该注解主要用在方法上边，能根据条件对缓存进行清空，常用属性如下：

- `value`:同上
- `key`：同上
- `condition`：同上
- `allEntries`:是否清空所有缓存内容，默认为`false`，如果设置为`true`,那么在方法执行完成之后并且满足`condition`条件时会清空该缓存区的所有内容。
- `beforeInvocation`：清除内容操作是否发生在方法执行之前，默认为`false`，表示清除操作在方法执行完之后再进行，如果方法执行过程中抛出异常，那么清除操作就不执行，如果为`true`，则表示在方法执行之前执行清除操作。

```java
@CacheEvict(value="userCache",key="#p0",allEntries=false, beforeInvocation=true)
public int deleteByPrimarykey(Integer key) {
	// TODO Auto-generated method stub
    //省略内容
}
```

## 效果测试

上边介绍了`spring boot`配置`ehcache`的步骤，接下来测试缓存效果，本项目在整合了`Mybatis`以及日志框架的前提下进行，基本的代码就不贴出来了，直接给出最关键的`service`实现层以及`controller`的代码：

**SysuserServiceImpl.java**

```
package com.web.springbootehcache.service.impl;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.cache.annotation.CacheEvict;
import org.springframework.cache.annotation.CachePut;
import org.springframework.cache.annotation.Cacheable;
import org.springframework.stereotype.Service;

import com.web.springbootehcache.dao.SysUserMapper;
import com.web.springbootehcache.entity.SysUser;
import com.web.springbootehcache.service.IsysUserService;

/**
* @author Promise
* @createTime 2019年3月19日 
* @description
*/
@Service("sysuserService")
public class SysUserServiceImpl implements IsysUserService{
	
	private final static Logger log = LoggerFactory.getLogger(SysUserServiceImpl.class);
	
	@Autowired
	private SysUserMapper sysuserMapper;

	@Override
	@Cacheable(value="userCache",key="#p0")
	public SysUser fingByPrimarykey(Integer key) {
		// TODO Auto-generated method stub
		log.debug("去数据库查询了数据！");
		return sysuserMapper.selectByPrimaryKey(key);
	}

	@Override
	@CachePut(value="userCache",key="#p0.id")
	public SysUser updateSysuser(SysUser entity) {
		// TODO Auto-generated method stub
		log.debug("更新了数据库数据！");
		int res = sysuserMapper.updateByPrimaryKey(entity);
		if(res >0)
			return entity;
		else
			return null;
	}

	@Override
	@CachePut(value="userCache",key="#entity.id")
	public SysUser insertSysuser(SysUser entity) {
		// TODO Auto-generated method stub		
		int res = sysuserMapper.insert(entity);
		log.debug("新增了数据！id为：{}",entity.getId());
		if(res >0)
			return entity;
		else
			return null;
	}

	@Override
	@CacheEvict(value="userCache",key="#p0",beforeInvocation=true)
	public int deleteByPrimarykey(Integer key) {
		// TODO Auto-generated method stub
		log.debug("删除了数据！");
		return sysuserMapper.deleteByPrimaryKey(key);
	}

}

```

该类中给出了基本的`CRUD`操作对应的缓存操作，当然不是绝对的，实际使用中根据自己需要改动。

**IndexController.java**

```
package com.web.springbootehcache.controller;

import java.util.HashMap;
import java.util.Map;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import com.web.springbootehcache.entity.SysUser;
import com.web.springbootehcache.service.IsysUserService;

/**
* @author Promise
* @createTime 2019年3月19日
* @description
*/
@RestController
public class IndexController {
	
	private final static Logger log = LoggerFactory.getLogger(IndexController.class);
	
	@Autowired
	private IsysUserService sysuserService;

	@RequestMapping(value="/select/{id}")
	public Object index(@PathVariable Integer id) {
		Map<String, Object> map = new HashMap<>();
		SysUser sysuser = sysuserService.fingByPrimarykey(id);
		log.debug("查询了id为:{}的用户信息！",sysuser.getId());
		SysUser sysuser2 = sysuserService.fingByPrimarykey(id);
		log.debug("查询了id为:{}的用户信息！",sysuser2.getId());
		map.put("res", sysuser);
		return map;
	}
	
	@RequestMapping(value="/update")
	public Object update() {
		Map<String, Object> map = new HashMap<>();
		//第一次修改
		SysUser sysuser = new SysUser(1, "eran", "eran1", 20, "M");
		sysuserService.updateSysuser(sysuser);
		//第一次查询
		sysuser = sysuserService.fingByPrimarykey(1);
		log.debug("查询了id为:{}的用户信息！",sysuser.getId());
		//第2次修改
		sysuser = new SysUser(1, "eran", "eran2", 20, "M");
		sysuserService.updateSysuser(sysuser);
		//第2次查询
		sysuser = sysuserService.fingByPrimarykey(1);
		log.debug("查询了id为:{}的用户信息！",sysuser.getId());
 		map.put("res", sysuser);
		return map;
	}
	
	@RequestMapping(value="/insert")
	public Object insert() {
		Map<String, Object> map = new HashMap<>();
		SysUser sysuser = new SysUser();
		sysuser.setName("admin");
		sysuser.setAge(22);
		sysuser.setPass("admin");
		sysuser.setSex("M");
		sysuserService.insertSysuser(sysuser);
		//查询
		sysuser = sysuserService.fingByPrimarykey(sysuser.getId());
		map.put("res", sysuser);
		return map;
	}
	
	@RequestMapping(value="/delete/{id}")
	public Object delete(@PathVariable Integer id) {
		Map<String, Object> map = new HashMap<>();
		sysuserService.deleteByPrimarykey(id);
		//查询
		SysUser sysuser = sysuserService.fingByPrimarykey(id);
		map.put("res", sysuser);
		return map;
	}
}

```

**数据库测试数据**



![在这里插入图片描述](https://user-gold-cdn.xitu.io/2019/3/20/1699a9a3832f1ad5?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



启动项目，访问`localhost:1188/select/1`,控制台日志如下：

预期效果：执行查询操作两次，访问数据库一次。

```
[default]2019-03-20 17:35:05,287 [http-nio-1188-exec-2 32] DEBUG >> 去数据库查询了数据！ >> c.w.s.s.i.SysUserServiceImpl
[default]2019-03-20 17:35:05,327 [http-nio-1188-exec-2 110] INFO >> HikariPool-1 - Starting... >> c.z.h.HikariDataSource
[default]2019-03-20 17:35:05,332 [http-nio-1188-exec-2 68] WARN >> Registered driver with driverClassName=com.mysql.jdbc.Driver was not found, trying direct instantiation. >> c.z.h.u.DriverDataSource
[default]2019-03-20 17:35:06,106 [http-nio-1188-exec-2 123] INFO >> HikariPool-1 - Start completed. >> c.z.h.HikariDataSource
[default]2019-03-20 17:35:06,113 [http-nio-1188-exec-2 159] DEBUG >> ==>  Preparing: select id, `name`, pass, sex, age from sys_user where id = ?  >> c.w.s.d.S.selectByPrimaryKey
[default]2019-03-20 17:35:06,141 [http-nio-1188-exec-2 159] DEBUG >> ==> Parameters: 1(Integer) >> c.w.s.d.S.selectByPrimaryKey
[default]2019-03-20 17:35:06,176 [http-nio-1188-exec-2 159] DEBUG >> <==      Total: 1 >> c.w.s.d.S.selectByPrimaryKey
[default]2019-03-20 17:35:06,185 [http-nio-1188-exec-2 33] DEBUG >> 查询了id为:1的用户信息！ >> c.w.s.c.IndexController
[default]2019-03-20 17:35:06,186 [http-nio-1188-exec-2 35] DEBUG >> 查询了id为:1的用户信息！ >> c.w.s.c.IndexController
复制代码
```

可以很直白的看出，我们执行了两次查询操作，但是从数据库中取数据的操作就执行了一次，可见还有一次直接从缓存中取数据，达到了我们预期的效果。

访问`localhost:1188/update`，代码中我们对`id`为`2`的数据做了两次修改以及两次查询操作，并且在执行修改操作时缓存了数据，执行该方法之前，`id`为`2`的数据还不在缓存中。

预期效果：执行两次修改操作，访问两次数据库，两次查询操作不访问数据库。

```
[default]2019-03-20 17:47:37,254 [http-nio-1188-exec-1 40] DEBUG >> 更新了数据库数据！ >> c.w.s.s.i.SysUserServiceImpl
[default]2019-03-20 17:47:37,291 [http-nio-1188-exec-1 110] INFO >> HikariPool-1 - Starting... >> c.z.h.HikariDataSource
[default]2019-03-20 17:47:37,299 [http-nio-1188-exec-1 68] WARN >> Registered driver with driverClassName=com.mysql.jdbc.Driver was not found, trying direct instantiation. >> c.z.h.u.DriverDataSource
[default]2019-03-20 17:47:37,953 [http-nio-1188-exec-1 123] INFO >> HikariPool-1 - Start completed. >> c.z.h.HikariDataSource
[default]2019-03-20 17:47:37,964 [http-nio-1188-exec-1 159] DEBUG >> ==>  Preparing: update sys_user set `name` = ?, pass = ?, sex = ?, age = ? where id = ?  >> c.w.s.d.S.updateByPrimaryKey
[default]2019-03-20 17:47:38,006 [http-nio-1188-exec-1 159] DEBUG >> ==> Parameters: eran(String), eran1(String), M(String), 20(Integer), 2(Integer) >> c.w.s.d.S.updateByPrimaryKey
[default]2019-03-20 17:47:38,104 [http-nio-1188-exec-1 159] DEBUG >> <==    Updates: 1 >> c.w.s.d.S.updateByPrimaryKey
[default]2019-03-20 17:47:38,237 [http-nio-1188-exec-1 48] DEBUG >> 查询了id为:2的用户信息！ >> c.w.s.c.IndexController
[default]2019-03-20 17:47:38,239 [http-nio-1188-exec-1 40] DEBUG >> 更新了数据库数据！ >> c.w.s.s.i.SysUserServiceImpl
[default]2019-03-20 17:47:38,239 [http-nio-1188-exec-1 159] DEBUG >> ==>  Preparing: update sys_user set `name` = ?, pass = ?, sex = ?, age = ? where id = ?  >> c.w.s.d.S.updateByPrimaryKey
[default]2019-03-20 17:47:38,243 [http-nio-1188-exec-1 159] DEBUG >> ==> Parameters: eran(String), eran2(String), M(String), 20(Integer), 2(Integer) >> c.w.s.d.S.updateByPrimaryKey
[default]2019-03-20 17:47:38,286 [http-nio-1188-exec-1 159] DEBUG >> <==    Updates: 1 >> c.w.s.d.S.updateByPrimaryKey
[default]2019-03-20 17:47:38,287 [http-nio-1188-exec-1 54] DEBUG >> 查询了id为:2的用户信息！ >> c.w.s.c.IndexController
复制代码
```

结果符合我们预期。

新增操作和更新操作原理一样都是使用`@CachePut`注解，这里就不重复演示，直接测试删除数据清除相应缓存功能，访问`localhost:1188/delete/2`，此时缓存区中有`id`为`1`,`2`的两条数据，我们删除`id`为`2`的数据，再做查询操作。

预期效果：删除数据访问数据库一次，并清除缓存区中那个相应的数据，因为清除了缓存区的内容所以查询数据会访问数据库一次，但是数据库中相应的内容也已经被删除，所以查询不到任何数据。

```
[default]2019-03-20 17:57:28,337 [http-nio-1188-exec-4 64] DEBUG >> 删除了数据！ >> c.w.s.s.i.SysUserServiceImpl
[default]2019-03-20 17:57:28,341 [http-nio-1188-exec-4 159] DEBUG >> ==>  Preparing: delete from sys_user where id = ?  >> c.w.s.d.S.deleteByPrimaryKey
[default]2019-03-20 17:57:28,342 [http-nio-1188-exec-4 159] DEBUG >> ==> Parameters: 2(Integer) >> c.w.s.d.S.deleteByPrimaryKey
[default]2019-03-20 17:57:28,463 [http-nio-1188-exec-4 159] DEBUG >> <==    Updates: 1 >> c.w.s.d.S.deleteByPrimaryKey
[default]2019-03-20 17:57:28,464 [http-nio-1188-exec-4 32] DEBUG >> 去数据库查询了数据！ >> c.w.s.s.i.SysUserServiceImpl
[default]2019-03-20 17:57:28,467 [http-nio-1188-exec-4 159] DEBUG >> ==>  Preparing: select id, `name`, pass, sex, age from sys_user where id = ?  >> c.w.s.d.S.selectByPrimaryKey
[default]2019-03-20 17:57:28,468 [http-nio-1188-exec-4 159] DEBUG >> ==> Parameters: 2(Integer) >> c.w.s.d.S.selectByPrimaryKey
[default]2019-03-20 17:57:28,494 [http-nio-1188-exec-4 159] DEBUG >> <==      Total: 0 >> c.w.s.d.S.selectByPrimaryKey
复制代码
```

日志输出的内容符合我们预期。


