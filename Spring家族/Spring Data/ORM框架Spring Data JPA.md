## 基础

### JPA

**Spring Data JPA** 可以理解为JPA规范的再次封装抽象，底层还是使用了Hibernate 的JPA技术实现，引用JPQL（Java Persistence Query Language）查询语言，属于Spring整个生态体系的一部分。随着SpringBoot和 Spring Cloud在市场上的流行，Spring Data JPA也逐渐进入大家的视野，它们组成有机的整体，使用起来比较方便，加快了开发的效 率，使开发者不需要关心和配置更多的东西，完全可以沉浸在Spring的完整生态标准实现下。JPA上手简单，开发效率高，对对象的支持比较好，又有很大的灵活性，市场的认可度越来越高。

JPA是Java Persistence API的简称，中文名为Java持久层API，是JDK 5.0注解或XML描述对象－关系表的映射关系，并将运行期的实体对象持久化到数据库中。JPA包括以下3方面的内容： 

* 一套API标准。在javax.persistence的包下面，用来操作实体对象，执行CRUD操作，框架在后台替代我们完成所有的事情，开 发者从烦琐的JDBC和SQL代码中解脱出来。 
* 面向对象的查询语言：Java Persistence Query Language（JPQL）。这是持久化操作中很重要的一个方面，通过面向 对象而非面向数据库的查询语言查询数据，避免程序的SQL语句紧密 耦合。
* ORM（object/relational metadata）元数据的映射。JPA 支持XML和JDK5.0注解两种元数据的形式，元数据描述对象和表之间的映射关系，框架据此将实体对象持久化到数据库表中。  

JPA的宗旨是为POJO提供持久化标准规范，由此可见，经过这几年的实践探索，能够脱离容器独立运行，方便开发和测试的理念已经深入人心了。Hibernate 3.2+、TopLink 10.1.3以及OpenJPA都提供了JPA的实现，以及最后的Spring的整合Spring Data JPA。

### Spring Data

Spring Data项目是从2010年发展起来的，从创立之初Spring Data就想提供一个大家熟悉的、一致的、基于Spring的数据访问编程模型，同时仍然保留底层数据存储的特殊特性。它可以轻松地让开发者使用数据访问技术，包括关系数据库、非关系数据库（NoSQL）和基于云的数据服务。Spring Data Common是Spring Data所有模块的公用部分，该项目提供跨Spring数据项目的共享基础设施。它包含了技术中立的库接口以及一个坚持java类的元数据模型。Spring Data不仅对传统的数据库访问技术JDBC、Hibernate、 JDO、TopLick、JPA、Mybitas做了很好的支持、扩展、抽象、提供方便的API，还对NoSQL等非关系数据做了很好的支持，包括MongoDB、 Redis、Apache Solr等

Spring Data包含多个子项目，主要子项目（Main modules）如下： 

* Spring Data Commons 
* Spring Data Gemfire 
* Spring Data JPA 
* Spring Data KeyValue 
* Spring Data LDAP 
* Spring Data MongoDB 
* Spring Data REST 
* Spring Data Redis 
* Spring Data for Apache Cassandra 
* Spring Data for Apache Solr 

Spring Data项目旨在为大家提供一种通用的编码模式。数据访问对象实现了对物理数据层的抽象，为编写查询方法提供了方便。通过对象映射，实现域对象和持续化存储之间的转换，而模板提供的是对底层存储实体的访问实现

## 基础接口

### Repository

Repository位于Spring Data Common的lib里面，是Spring Data里面做数据库操作的最底层的抽象接口、最顶级的父类，源码里面其实什么方法都没有，仅仅起到一个标识作用。管理域类以及域类的id类型作为类型参数，此接口主要作为标记接口捕获要使用的类型，并帮助你发现扩展此接口的接口。Spring底层做动态代理的时候发现只要是它的子类或者实现类，都代表储存库操作。Repository的源码为

[![3VBel6.png](https://s2.ax1x.com/2020/02/19/3VBel6.png)](https://imgchr.com/i/3VBel6)

这个接口定义了所有Repostory操作的实体和ID两个泛型参数。我们不需要继承任何接口，只要继承这个接口，就可以使用Spring JPA里面提供的很多约定的方法查询和注解查询，同时可以添加自定义的查询方法

```java
@Repository
public interface UserDao extends JpaRepository<User,Long> {
    User findByPhoneNumberAndPassword(long phoneNumber,String password);
    User findByNameAndPassword(String name,String password);
}
```

只要@Repository声明接口就相当于定义了一个DAO，里面可以自定义方法并直接使用

### **CrudRepository**

CrudRepository接口提供了公共的通用的CRUD 方法，源码为

```java
package org.springframework.data.repository;
import java.util.Optional;
@NoRepositoryBean
public interface CrudRepository<T, ID> extends Repository<T, ID> {
    <S extends T> S save(S var1);
    <S extends T> Iterable<S> saveAll(Iterable<S> var1);
    Optional<T> findById(ID var1);
    boolean existsById(ID var1);
    Iterable<T> findAll();
    Iterable<T> findAllById(Iterable<ID> var1);
    long count();
    void deleteById(ID var1);
    void delete(T var1);
    void deleteAll(Iterable<? extends T> var1);
    void deleteAll();
}
```

SimpleJpaRepository是该接口的简单实现类

### PagingAndSortingRepository

PagingAndSortingRepository接口是CrudRepository的子接口，它增加了分页和排序等对查询结果进行限制的基本的、常用的、通用的一些分页方法。 

```java
package org.springframework.data.repository;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;
import org.springframework.data.domain.Sort;
@NoRepositoryBean
public interface PagingAndSortingRepository<T, ID> extends CrudRepository<T, ID> {
    Iterable<T> findAll(Sort var1);
    Page<T> findAll(Pageable var1);
}
```

（1）根据排序取所有对象的集合。 

（2）根据分页和排序进行查询，并用Page对象封装。Pageable对象包含分页和Sort对象。 

### JpaRepository

JpaRepository到这里可以进入分水岭了，上面的那些都是 Spring Data为了兼容NoSQL而进行的一些抽象封装，从 JpaRepository开始是对关系型数据库进行抽象封装。它继承PagingAndSortingRepository类，实现类也是SimpleJpaRepository。

```java
package org.springframework.data.jpa.repository;
import java.util.List;
import org.springframework.data.domain.Example;
import org.springframework.data.domain.Sort;
import org.springframework.data.repository.NoRepositoryBean;
import org.springframework.data.repository.PagingAndSortingRepository;
import org.springframework.data.repository.query.QueryByExampleExecutor;
@NoRepositoryBean
public interface JpaRepository<T, ID> extends PagingAndSortingRepository<T, ID>, QueryByExampleExecutor<T> {
    List<T> findAll();
    List<T> findAll(Sort var1);
    List<T> findAllById(Iterable<ID> var1);
    <S extends T> List<S> saveAll(Iterable<S> var1);
    void flush();
    <S extends T> S saveAndFlush(S var1);
    void deleteInBatch(Iterable<T> var1);
    void deleteAllInBatch();
    T getOne(ID var1);
    <S extends T> List<S> findAll(Example<S> var1);
    <S extends T> List<S> findAll(Example<S> var1, Sort var2);
}
```

SimpleJpaRepository是JPA整个关联数据库的所有Repository的 接口实现类。如果想进行扩展，可以继承此类。

### 简单使用

使用SpringBoot创建项目并进行配置

```properties
#通用数据源配置
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
spring.datasource.url=jdbc:mysql://10.110.2.56:3306/springboot_jpa?charset=utf8mb4&useSSL=false
spring.datasource.username=springboot
spring.datasource.password=springboot
# Hikari 数据源专用配置
spring.datasource.hikari.maximum-pool-size=20
spring.datasource.hikari.minimum-idle=5
# JPA 相关配置
spring.jpa.show-sql=true
```

 spring.jpa.show-sql=true 配置在日志中打印出执行的 SQL 语句信息。 

**创建Entity并声明注解**

创建一个实体类并声明注解信息

```java
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;
import org.springframework.lang.NonNull;

import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.Id;

@Data
@NoArgsConstructor
@AllArgsConstructor
@Entity
public class User {

    @Id
    private long id;

    @Column(length = 15,nullable = false)
    private String name;

    @Column(length = 15,nullable = false)
    private String password;

    @Column(nullable = false)
    private long phoneNumber;

    @Column(nullable = false)
    private int regionId;

    @Column(length = 50)
    private String address;

    private byte status;

}
```

@Entity 是一个必选的注解，声明这个类对应了一个数据库表。

@Table(name = "User") 是一个可选的注解。声明了数据库实体对应的表信息。包括表名称、索引信息等。这里声明这个实体类对应的表名是User。如果没有指定，则表名和实体的名称保持一致。

@Id 注解声明了实体唯一标识对应的属性。

@Column(length = 32) 用来声明实体属性的表字段的定义。默认的实体每个属性都对应了表的一个字段。字段的名称默认和属性名称保持一致（并不一定相等）。字段的类型根据实体属性类型自动推断。这里主要是声明了字符字段的长度。如果不这么声明，则系统会采用 255 作为该字段的长度

**定义数据访问接口**

只需要继承接口Repository即可，Repository及其子类在Spring Data JPA中定义了丰富的查询方法及实现

```java
import org.phoenix.aladdin.model.entity.User;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;

@Repository
public interface UserDao extends JpaRepository<User,Long> {
}
```

**使用数据访问接口**

直接注入接口UseDao(数据访问对象DAO)，会通过动态代理自动生成内部类并将引用赋给接口对象，通过接口内置的定义方法完成常见的CURD操作

```java
import org.junit.jupiter.api.Test;
import org.phoenix.aladdin.model.entity.Region;
import org.phoenix.aladdin.model.entity.User;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;

import static org.junit.jupiter.api.Assertions.*;

@SpringBootTest
class UserDaoTest {

    @Autowired
    UserDao userDao;

    @Test
    public void dbTest(){
        User user=new User();
        user.setId(1L);
        user.setName("lsl");
        user.setPassword("123456");
        user.setPhoneNumber(13849720276L);
        user.setRegionId(1);
        userDao.save(user);
        System.out.println(userDao.findAll().get(0).toString());
    }
}
```

## 基础方法定义查询

由于Spring JPA Repository的实现原理是采用动态代理的机制，所以有两种定义查询方法：从方法名称中可以指定特定用于存储的查询和更新，或通过使用@Query手动定义的查询，这个取决于实际存储操作。只需要实体Repository继承Spring Data Common里面的Repository接口即可。如果想有其他更多默认通用方法的实现，可以选择JpaRepository、PagingAndSortingRepository、CrudRepository等接口，也可以直接继承我们后面要讲的JpaSpecificationExecutor、QueryByExampleExecutor和自定义Response，都可以达到同样的效果。 

### 方法的查询策略设置

通过@EnableJpaRepositories(queryLookupStrategy= QueryLookupStrategy.Key.CREATE_IF_NOT_FOUND)可以配置方法的查询策略，其中QueryLookupStrategy.Key的值一共有三个。 

* CREATE：直接根据方法名进行创建。规则是根据方法名称的构造进行尝试，一般的方法是从方法名中删除给定的一组已知前缀，并解析该方法的其余部分。如果方法名不符合规则，启动的时候就会报异常。 
* USE_DECLARED_QUERY：声明方式创建，即注解方式。启动的时候会尝试找到一个声明的查询，如果没有找到就将抛出一个异常。查询可以由某处注释或其他方法声明。 
* CREATE_IF_NOT_FOUND：这个是默认的，以上两种方式的结合版。先用声明方式进行查找，如果没有找到与方法相匹配的查询，就用create的方法名创建规则创建一个查询。 

### 查询方法的创建

内部基础架构中有个根据方法名的查询生成器机制，对于在存储库的实体上构建约束查询很有用。该机制方法的前缀有find…By、read…By、query…By、count…By和get…By，从这些方法可以分析它的其余部分（实体里面的字段）。根据这种约定指定要查询的行以及字段完成指定的查询方法，如：

```java
@Repository
public interface UserDao extends JpaRepository<User,Long> {
    User findByPhoneNumberAndPassword(long phoneNumber,String password);
    User findByNameAndPassword(String name,String password);
}
```

基于这种方式相当于直接根据方法名来进行查询，对于普通查询很方便，对于复杂一点的，如查询是排序，也可以使用其约定，如可以通过OrderBy在引用属性和提供排序方向（Asc或Desc）的查询方法中附加一个子句来应用静态排序。显然，对于复杂查询，基于方法的查询并不太灵活。

<img src="https://i.loli.net/2020/02/24/Lu1wUIqTSRmhK6V.png" alt="image.png" style="zoom: 80%;" />

<img src="https://i.loli.net/2020/02/24/T3bKSBL4ioOeAdv.png" alt="image.png" style="zoom:80%;" />

前缀是必须的，前缀除find外，还有

* 查：find,read,get,query,stream
* 计数：count
* 存在：exists
* 删除：delete,remove

**delete方法没有返回值，如果删除失败则会抛出EmptyResultDataAccessException异常**

**对于delete 与 update 方法，要使用@Modifying进行修饰**

### 分页与排序

```text
@Repository
public interface EmployeeRepository extends PagingAndSortingRepository<Employee, Long> {
    //Page<Employee> findAll(Pageable pageable);这个已经定义了，不用再定义
    Page<Employee> findByFirstName(String firstName, Pageable pageable);
    Slice<Employee> findByFirstNameAndLastName(String firstName, String lastName, Pageable pageable);
}
```

`Pageable` 是一个的接口，`PageRequest`是其实现。、

```text
Pageable pageable = PageRequest.of(0, 10);
Page<Employee> page = employeeRepository.findAll(pageable);
```

在第一行中，我们创建了 `PageRequest`10名员工，并要求提供第一页（0）。页面从0开始。

如果我们想要访问下一组后续页面，我们可以每次都增加页码。

```text
PageRequest.of(1, 10);
PageRequest.of(2, 10);
PageRequest.of(3, 10);
...
```

Spring Data JPA提供了一个 `Sort` 对象以提供排序机制。

```text
employeeRepository.findAll(Sort.by("fistName"));
employeeRepository.findAll(Sort.by("fistName").ascending().and(Sort.by("lastName").descending());
```

显然，第一个按“firstName”排序，另一个按“firstName”升序和“lastName”降序排序。

```text
Pageable pageable = PageRequest.of(0, 20, Sort.by("firstName"));
Pageable pageable = PageRequest.of(0, 20, Sort.by("fistName").ascending().and(Sort.by("lastName").descending());
```

**切片与页**

 `Page` 是 `Slice`子接口，它们都用于保存和返回数据子集。

```text
List<T> getContent(); // get content of the slice
Pageable getPageable(); // get current pageable
boolean hasContent(); 
boolean isFirst();
boolean isLast();
Pageable nextPageable(); // pageable of the next slice
Pageable previousPageable(); // pageable of the previous slice
```

```text
static <T> Page<T> empty; //create an empty page
long getTotalElements(); // number of total elements in the table
int totalPages() // number of total pages in the table
```

### @Modifying

@Modifying 以通知 SpringData， 这是一个 UPDATE 或 DELETE 操作

### 插入

使用save()方法进行插入

## 基于注解的查询

### @Query

声明在DAO接口的方法上，传递一个SQL语句

```java
@Repository
public interface ExpressDao extends JpaRepository<Express,Long> {
    @Query("SELECT e.id,e.status from Express e where e.id = ?1")
    Express findById(long id);
}
```

<img src="https://i.loli.net/2020/02/24/yUYTlJhIHNAbk38.png" alt="image.png" style="zoom:80%;" />

<img src="https://i.loli.net/2020/02/24/sfJYBLGh7dapqi4.png" alt="image.png" style="zoom:80%;" />

想要排序的话就传递一个Sort

<img src="https://i.loli.net/2020/02/24/jmB2Ec7hzMPAIqg.png" alt="image.png" style="zoom:80%;" />

想要分页的话直接用Page对象接收接口，参数直接用Pageable的实现类即可。

<img src="https://i.loli.net/2020/02/24/ytD41I5RPcbuNUg.png" alt="image.png" style="zoom:80%;" />

**传入参数是对象**

```java
@Query("update HsAlbum album set album.albumName = :#{#hsAlbum.albumName} ,album.description = :#{#hsAlbum.description } ")
public int update(HsAlbum hsAlbum);
```

### @Param

默认情况下，参数是通过顺序绑定在查询语句上的。这使得查询方法对参数位置的重构容易出错。为了解决这个问题，可以使用@Param注解指定方法参数的具体名称，通过绑定的参数名字做查询条件。

<img src="https://i.loli.net/2020/02/24/QOFB6rJGvNcP7ns.png" alt="image.png" style="zoom:80%;" />

## 实体@Entity常见注解

@Entity定义对象将会成为被JPA管理的实体，将映射到指定的数据库表。默认就是声明的类的名字

### @Table

@Table指定数据库的表名。

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
public @interface Table {
    String name() default "";//表名，可选
    String catalog() default "";
    String schema() default "";//所在的schema，可选
    UniqueConstraint[] uniqueConstraints() default {};
    Index[] indexes() default {};
}
```

### @Id
@Id定义属性为数据库的主键，一个实体里面必须有一个。

### @GeneratedValue
@GeneratedValue为主键生成策略

```java
@Target({ElementType.METHOD, ElementType.FIELD})
@Retention(RetentionPolicy.RUNTIME)
public @interface GeneratedValue {
    GenerationType strategy() default GenerationType.AUTO;

    String generator() default "";
}
```

默认是自动选择最佳生成策略

**自增主键上要加上**

```java
@GeneratedValue(strategy= GenerationType.IDENTITY)
```

否则会报错

### @Basic
@Basic表示属性是到数据库表的字段的映射。如果实体的字段上没有任何注解，默认即为@Basic。

### @Column

@Column定义该属性对应数据库中的列名。

```java
@Target({ElementType.METHOD, ElementType.FIELD})
@Retention(RetentionPolicy.RUNTIME)
public @interface Column {
    String name() default "";//对应的列名
    boolean unique() default false;//是否唯一
    boolean nullable() default true;//是否允许为null
    boolean insertable() default true;
    boolean updatable() default true;
    String columnDefinition() default "";//字段在数据库中的实际类型
    String table() default "";
    int length() default 255;
    int precision() default 0;
    int scale() default 0;
}
```

### @Temporal

@Temporal用来设置Date类型的属性映射到对应精度的字段。
（1）@Temporal(TemporalType.DATE)映射为日期∥date（只有日期）。
（2）@Temporal(TemporalType.TIME)映射为日期∥time（只有时间）。
（3）@Temporal(TemporalType.TIMESTAMP)映射为日期∥datetime（日期+时间）。

### @Enumerated
@Enumerated很好用，直接映射enum枚举类型的字段。

### 实体关联注解

#### @JoinColumn

@JoinColumns定义多个字段的关联关系。@JoinColumn需要配合@OneToOne、@ManyToOne、@OneToMany一起使用，单独使用没有意义。

#### @OneToOne

@OneToOne需要配合@JoinColumn一起使用。注意：可以双向关联，也可以只配置一方，需要视实际需求而定。

【例】一个人对应一个地址

```java
@Entity
public class People {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "id", nullable = false)
    private Long id;//id
    
    @Column(name = "name", nullable = true, length = 20)
    private String name;//姓名
    
    @Column(name = "sex", nullable = true, length = 1)
    private String sex;//性别
    
    @Column(name = "birthday", nullable = true)
    private Timestamp birthday;//出生日期
    
    @OneToOne(cascade=CascadeType.ALL)//People是关系的维护端，当删除 people，会级联删除 address
    @JoinColumn(name = "address_id", referencedColumnName = "id")//people中的address_id字段参考address表中的id字段
    private Address address;//地址
}
```

```java
@Entity
public class Address {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "id", nullable = false)
    private Long id;//id
    
    @Column(name = "phone", nullable = true, length = 11)
    private String phone;//手机
    
    @Column(name = "zipcode", nullable = true, length = 6)
    private String zipcode;//邮政编码
    
    @Column(name = "address", nullable = true, length = 100)
    private String address;//地址
    
    //如果不需要根据Address级联查询People，可以注释掉
//    @OneToOne(mappedBy = "address", cascade = {CascadeType.MERGE, CascadeType.REFRESH}, optional = false)
//    private People people;
}
```

这样就可以通过关联表的方式来保存一对一的关系，而不用对关系再建立一张表

people 表（**id**，name，sex，birthday）

address 表 (**id**，phone，zipcode，address）

people_address (**people_id**，**address_id**)

只需要创建 People 和 Address 两个实体

#### @OneToMany 和 @ManyToOne

JPA使用@OneToMany和@ManyToOne来标识一对多的双向关联。一端(Author)使用@OneToMany,多端(Article)使用@ManyToOne。

【例】假设一个作者（Author）有多篇文章（Article）

在JPA规范中，一对多的双向关系由多端(Article)来维护。就是说多端(Article)为关系维护端，负责关系的增删改查。一端(Author)则为关系被维护端，不能维护关系。

一端(Author)使用@OneToMany注释的mappedBy="author"属性表明Author是关系被维护端。

多端(Article)使用@ManyToOne和@JoinColumn来注释属性 author,@ManyToOne表明Article是多端，@JoinColumn设置在article表中的关联字段(外键)。

```java
@Entity
public class Author {
    @Id // 主键
    @GeneratedValue(strategy = GenerationType.IDENTITY) // 自增长策略
    private Long id; //id
    
    @NotEmpty(message = "姓名不能为空")
    @Size(min=2, max=20)
    @Column(nullable = false, length = 20)
    private String name;//姓名
    
    @OneToMany(mappedBy = "author",cascade=CascadeType.ALL,fetch=FetchType.LAZY)
    //级联保存、更新、删除、刷新;延迟加载。当删除用户，会级联删除该用户的所有文章
    //拥有mappedBy注解的实体类为关系被维护端
     //mappedBy="author"中的author是Article中的author属性
    private List<Article> articleList;//文章列表
}
```

```java
@Entity
public class Article {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY) // 自增长策略
    @Column(name = "id", nullable = false)
    private Long id;
    
    @NotEmpty(message = "标题不能为空")
    @Size(min = 2, max = 50)
    @Column(nullable = false, length = 50) // 映射为字段，值不能为空
    private String title;
    
    @Lob  // 大对象，映射 MySQL 的 Long Text 类型
    @Basic(fetch = FetchType.LAZY) // 懒加载
    @NotEmpty(message = "内容不能为空")
    @Size(min = 2)
    @Column(nullable = false) // 映射为字段，值不能为空
    private String content;//文章全文内容
    
    @ManyToOne(cascade={CascadeType.MERGE,CascadeType.REFRESH},optional=false)//可选属性optional=false,表示author不能为空。删除文章，不影响用户
    @JoinColumn(name="author")//
    private Author author;//所属作者
}
git remote add origin "https://gitee.com/LSLWIND/aladdin-client"
```

最终生成的表结构

article 表(id，title，conten，**author_id**)

author 表(**id**，name)

==需要注意的是，上述写法一定会导致循环引用问题，最终导致StackOverflow，因为序列化对象时两者是相互关联的，A包含B，B包含A，最终导致循环，解决办法是序列化时忽略掉某些属性。==

**首先要禁用lombok的@Data，使用@Data注释后，默认会重写父类的toString()方法，hashcode()等方法，在往map里存的时候，会根据equals和hashcode方法，来计算下标，而如果@Data注释的类与其他类有关联的属性（如：@onetoone,@onetomany等）且关联的属性不为空时，会不断从关联方的属性进行查找，再从关联方查找该@Data注释的类，依次循环，造成堆栈溢出。**

@JsonIgnoreProperties 用在字段上，指定某些属性被忽略，如

```java
@OneToMany(mappedBy = "salesOrder", cascade = CascadeType.ALL, orphanRemoval = true)
@OrderBy(value = "order_item_id")
@Fetch(FetchMode.JOIN)
@JsonIgnoreProperties("salesOrder")
private List<SalesOrderItem> salesOrderItems;
```

```java
@ManyToOne
@JoinColumn(name = "order_id")
@JsonIgnoreProperties("salesOrderItems")
private SalesOrder salesOrder;
```

或者使用fastjson中的@JSONField

```java
@OneToMany(mappedBy = "express",cascade= CascadeType.ALL,fetch= FetchType.LAZY)
@OrderBy("time")
@JSONField(serialize = false)
private List<PackageHistory> packageHistoryList;
```

#### @OrderBy

@OrderBy关联查询时排序，一般和@OneToMany一起使用。

###  @ManyToMany

实体 User：用户。

实体 Authority：权限。

用户和权限是多对多的关系。一个用户可以有多个权限，一个权限也可以被很多用户拥有。

JPA中使用@ManyToMany来注解多对多的关系，由一个关联表来维护。这个关联表的表名默认是：主表名+下划线+从表名。(主表是指关系维护端对应的表,从表指关系被维护端对应的表)。这个关联表只有两个外键字段，分别指向主表ID和从表ID。字段的名称默认为：主表名+下划线+主表中的主键列名，从表名+下划线+从表中的主键列名。

需要注意的：

1、多对多关系中一般不设置级联保存、级联删除、级联更新等操作。

2、可以随意指定一方为关系维护端，在这个例子中，我指定 User 为关系维护端，所以生成的关联表名称为： user_authority，关联表的字段为：user_id 和 authority_id。

3、多对多关系的绑定由关系维护端来完成，即由 User.setAuthorities(authorities) 来绑定多对多的关系。关系被维护端不能绑定关系，即Game不能绑定关系。

4、多对多关系的解除由关系维护端来完成，即由Player.getGames().remove(game)来解除多对多的关系。关系被维护端不能解除关系，即Game不能解除关系。

5、如果 User 和 Authority 已经绑定了多对多的关系，那么不能直接删除 Authority，需要由 User 解除关系后，才能删除 Authority。但是可以直接删除 User，因为 User 是关系维护端，删除 User 时，会先解除 User 和 Authority 的关系，再删除 Authority。

```java
@Entity
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @NotEmpty(message = "账号不能为空")
    @Size(min=3, max=20)
    @Column(nullable = false, length = 20, unique = true)
    private String username; // 用户账号，用户登录时的唯一标识
    
    @NotEmpty(message = "密码不能为空")
    @Size(max=100)
    @Column(length = 100)
    private String password; // 登录时密码
    
    @ManyToMany
    @JoinTable(name = "user_authority",joinColumns = @JoinColumn(name = "user_id"),
    inverseJoinColumns = @JoinColumn(name = "authority_id"))
    //1、关系维护端，负责多对多关系的绑定和解除
    //2、@JoinTable注解的name属性指定关联表的名字，joinColumns指定外键的名字，关联到关系维护端(User)
    //3、inverseJoinColumns指定外键的名字，要关联的关系被维护端(Authority)
    //4、其实可以不使用@JoinTable注解，默认生成的关联表名称为主表表名+下划线+从表表名，
    //即表名为user_authority
    //关联到主表的外键名：主表名+下划线+主表中的主键列名,即user_id
    //关联到从表的外键名：主表中用于关联的属性名+下划线+从表的主键列名,即authority_id
    //主表就是关系维护端对应的表，从表就是关系被维护端对应的表
    private List<Authority> authorityList;
}
```

```java
@Entity
public class Authority {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Integer id;
    
    @Column(nullable = false)
    private String name; //权限名
    
    @ManyToMany(mappedBy = "authorityList")
    private List<User> userList;
}
```

## 高级查询

从JpaRepository开始的子类，都是Spring Data项目对JPA实现的封装与扩展。JpaRepository本身继承
PagingAndSortingRepository接口，是针对JPA技术的接口，提供flush()、saveAndFlush()、deleteInBatch()、deleteAllInBatch()等方法。它的实现类和子类，又提供了一些非常优雅的方法

### QueryByExampleExecutor

```java
public interface QueryByExampleExecutor<T> {
    <S extends T> Optional<S> findOne(Example<S> var1);

    <S extends T> Iterable<S> findAll(Example<S> var1);

    <S extends T> Iterable<S> findAll(Example<S> var1, Sort var2);

    <S extends T> Page<S> findAll(Example<S> var1, Pageable var2);

    <S extends T> long count(Example<S> var1);

    <S extends T> boolean exists(Example<S> var1);
}
```





 ## 常见问题

**'Sort(org.springframework.data.domain.Sort.Direction, java.util.List<java.lang.String>)' has private access in 'org.springframework.data.domain.Sort'****

构造方法已经变成了私有的，因此不能直接构造Sort了，可以使用Sort.by

```java
@GetMapping("/test2")
    public String find3() {
        List<Book> book = bookDao.findAll(Sort.by(Sort.Direction.DESC, "bookId"));
        for (Book book1 : book) {
            System.out.println(book1);
        }
        return "success";
    }
```



**type类型错误**

对于已经继承的接口的方法不能重复定义，典型的如findAll

### 不可以生成ids

常见原因是使用了自增主键，在主键字段上加上注解即可

```java
@GeneratedValue(strategy=GenerationType.IDENTITY)
```



**Can not set int field org.entity.contracts.contract_owner_id to null value**

原始数据类型不能为null，因此如果有映射为空的字段应该将其设置为对应的包装类，如int换位Integer

 

**Can't resolve column 'xxx'**

在使用@OneToMany时出现了该问题，因为JPA的解决方案中，使用外键关联实体关系，因此要检测配置是否正确必须指定已经配置好的数据源，然后才能检测是否正确。

进入Assign Data Sources弹出框中，如果已经配置了数据源，点击DataSource选择一个即可，否则要配置一个数据库连接（IDEA）

**could not initialize proxy - no Session**

经常在测试时出现，在进行数据库访问之时，当前针对数据库的访问与操作session已经关闭且释放了，故提示no Session可用

在测试类上加上@Transactional解决



**org.hibernate.exception.SQLGrammarException: could not extract ResultSet**

SQL语句写错了，一般会报错误提示，需要注意的是类名与表名的映射，数据库中对应的单词间要加上_，如PackageHistory默认映射的表是package_history