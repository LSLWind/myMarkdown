#### Maven中导入

```xml
<dependency>
  <groupId>org.mybatis</groupId>
  <artifactId>mybatis</artifactId>
  <version>x.x.x</version>
</dependency>
```

同时必须导入数据库，以mysql为例

```xml
    <dependency>
      <groupId>mysql</groupId>
      <artifactId>mysql-connector-java</artifactId>
      <version>8.0.17</version>
    </dependency>
```

#### 基本概念

每个基于 MyBatis 的应用都是以一个 SqlSessionFactory 的实例为核心的。SqlSessionFactory 的实例可以通过 SqlSessionFactoryBuilder 获得。而 SqlSessionFactoryBuilder 则可以从 XML 配置文件或一个预先定制的 Configuration 的实例构建出 SqlSessionFactory 的实例。 

 既然有了 SqlSessionFactory，我们就可以从中获得 SqlSession 的实例了。SqlSession 完全包含了面向数据库执行 SQL 命令所需的所有方法。可以通过 SqlSession 实例来直接执行已映射的 SQL 语句。 

```java
try (SqlSession session = sqlSessionFactory.openSession()) {
  BlogMapper mapper = session.getMapper(BlogMapper.class);//通过Sesson获取映射
  Blog blog = mapper.selectBlog(101);//通过映射执行sql语句
}
```

上述代码对应的xml配置文件为：

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
  PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="org.mybatis.example.BlogMapper">
  <select id="selectBlog" resultType="Blog">
    select * from Blog where id = #{id}
  </select>
</mapper>
```

简单理解：每一个DAO都可以进行SQL操作（对应的就是数据库表操作），SQL语句通过映射方式，在XMl中声明SQL语句并传参，在代码中通过构建DAO映射对象（也就是映射器，一般为接口，如上述的BlogMapper）执行对应的SQL操作（标识符就是id），通过配置文件读取信息构建会话工厂获取会话实例并执行SQL

对于简单的语句来说，也可以通过注解实现，如下所示：

```java
package org.mybatis.example;
public interface BlogMapper {
  @Select("SELECT * FROM blog WHERE id = #{id}")
  Blog selectBlog(int id);
}
```

#### #与$的区别

1 #是将传入的值当做字符串的形式，eg:select id,name,age from student where id =#{id},当前端把id值1，传入到后台的时候，就相当于 select id,name,age from student where id ='1'.

 2 $是将传入的数据直接显示生成sql语句，eg:select id,name,age from student where id =${id},当前端把id值1，传入到后台的时候，就相当于 select id,name,age from student where id = 1.

 3 使用#可以很大程度上防止sql注入。(语句的拼接)

 4 但是如果使用在order by 中就需要使用 $.

 5 在大多数情况下还是经常使用#，但在不同情况下必须使用$. 

#与的区别最大在于：#{} 传入值时，sql解析时，参数是带引号的，而的区别最大在于：#{} 传入值时，sql解析时，参数是带引号的，而{}穿入值，sql解析时，参数是不带引号的。

#### 作用域

官网给出的推荐（最佳实践）为：

##### SqlSessionFactoryBuilder

这个类可以被实例化、使用和丢弃，一旦创建了 SqlSessionFactory，就不再需要它了。 因此 SqlSessionFactoryBuilder 实例的最佳作用域是方法作用域（也就是局部方法变量）。 可以重用 SqlSessionFactoryBuilder 来创建多个 SqlSessionFactory 实例，但是最好还是不要让其一直存在，以保证所有的 XML 解析资源可以被释放给更重要的事情。

##### SqlSessionFactory

SqlSessionFactory 一旦被创建就应该在应用的运行期间一直存在，没有任何理由丢弃它或重新创建另一个实例。 使用 SqlSessionFactory 的最佳实践是在应用运行期间不要重复创建多次，因此 SqlSessionFactory 的最佳作用域是应用作用域。 有很多方法可以做到，最简单的就是使用单例模式或者静态单例模式。

##### SqlSession

每个线程都应该有它自己的 SqlSession 实例。SqlSession 的实例不是线程安全的，因此是不能被共享的，所以它的最佳的作用域是请求或方法作用域。  换句话说，每次收到的 HTTP 请求，就可以打开一个 SqlSession，返回一个响应，就关闭它。 这个关闭操作是很重要的，你应该把这个关闭操作放到 finally 块中以确保每次都能执行关闭。

##### 映射器实例

映射器是一些由你创建的、绑定你映射的语句的接口。映射器接口的实例是从 SqlSession 中获得的。因此从技术层面讲，任何映射器实例的最大作用域是和请求它们的 SqlSession 相同的。尽管如此，映射器实例的最佳作用域是方法作用域。 也就是说，映射器实例应该在调用它们的方法中被请求，用过之后即可丢弃。

#### MyBatis XML配置

基本配置为

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="${driver}"/>
                <property name="url" value="${url}"/>
                <property name="username" value="${username}"/>
                <property name="password" value="${password}"/>
            </dataSource>
        </environment>
    </environments>
    <mappers>
        <mapper resource="org/mybatis/example/BlogMapper.xml"/>
    </mappers>
</configuration>
```

配置文件是 mybatis 用来建立 sessionFactory，里面主要包含了数据库连接相关内容

##### <properties\>

用于进行复用value，减少冗余，可引用外部文件，使用时通过${key}即可获取value

```xml
<properties resource="org/mybatis/example/config.properties">
  <property name="username" value="dev_user"/>
  <property name="password" value="F2Fa3!33TYyg"/>
</properties>
```

```xml
<dataSource type="POOLED">
  <property name="driver" value="${driver}"/>
  <property name="url" value="${url}"/>
  <property name="username" value="${username}"/>
  <property name="password" value="${password}"/>
</dataSource>
```

##### <settings\>

mybatis的全局配置，不配置则使用默认值

##### <typeAliases\>

类型别名，唯一目的仍然是减少冗余，有两种使用方式

1.某个具体类的别名，使用<typeAlias\>

```xml
<typeAliases>
  <typeAlias alias="Author" type="domain.blog.Author"/>
  <typeAlias alias="Blog" type="domain.blog.Blog"/>
</typeAliases>
```

2.直接指定某个包，默认包下的所有类的别名即为首字母小写的别名

```xml
<typeAliases>
  <package name="domain.blog"/>
</typeAliases>
```

然而在包下也可以通过注解指定其他别名（仍需上述配置以指定包）

```java
@Alias("author")
public class Author {
    ...
}
```

同时为了方便传参，mybatis对java内置的基类型与集合类型指定了别名（也是首字母小写，其余不变）

##### <environments\>

每个环境对应一个数据库，也对应一个SqlSessionFactory，一个环境即一个<environment\>，在环境下进行具体的数据库配置，在实例化SqlSessionFactory时通过构造方法传参指定环境，否则使用<environments\>指定的默认环境

```xml
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="${driver}"/>
                <property name="url" value="${url}"/>
                <property name="username" value="${username}"/>
                <property name="password" value="${password}"/>
            </dataSource>
        </environment>
    </environments>
```

#####  <transactionManager\>

一般为JDBC，如果使用Spring+Mybatis则不需要设置该项，Spring会覆盖

##### <dataSource\>

配置数据源，有三种类型，UNPOOLED,POOLED与JNDI

##### <mappers\>

用于指定映射器的位置，Mybatis会通过该路径查找映射器并读取Sql语句，有四种指定mapper的方式

**1.属性resources，在resources目录下寻找**

```xml
<!-- 使用相对于类路径的资源引用 -->
<mappers>
  <mapper resource="org/mybatis/builder/AuthorMapper.xml"/>
</mappers>
```

与主配置一样，会在resources目录下寻找指定文件

**2.属性class，在java目录下寻找，使用指定class**

```xml
<!-- 使用映射器接口实现类的完全限定类名 -->
<mappers>
  <mapper class="org.mybatis.builder.AuthorMapper"/>
</mappers>
```

**3.属性name，指定包，注册该包下的所有Mapper**

```xml
<!-- 将包内的映射器接口实现全部注册为映射器 -->
<mappers>
  <package name="org.mybatis.builder"/>
</mappers>
```

**4.属性file，绝对文件路径**

一般不用

```xml
<!-- 使用完全限定资源定位符（URL） -->
<mappers>
  <mapper url="file:///var/mappers/AuthorMapper.xml"/>
</mappers>
```



#### 映射器XML配置

定义一个DAO的SQL行为，每一个映射器需绑定一个对应接口，在配置文件中使用namespace进行声明，解析过程为SqlSessionFactory-SqlSession-Mapper（映射器接口，绑定SQL配置文件）-执行Mapper中的方法-MyBatis执行SQL，映射器配置文件一般都与接口映射器一起放在mapper下

##### <mapper\>

每个映射器xml文件使用<mapper\>作为顶部标签，需使用namespace指定接口映射器，接口映射器要与该xml文件同名，有几个注意事项

1. 映射器xml与映射器接口要同名
2. 映射器xml中的sql的id要与映射器接口声明的方法同名
3. 映射器xml要声明namespace，指定映射器接口

##### <select\>

执行select语句，参数通过#{}引用，因为一条语句必定与接口映射器当中的一个方法对应，传递参数即通过这种方式绑定

```xml
<select id="selectPerson" parameterType="int" resultType="hashmap">
  SELECT * FROM PERSON WHERE ID = #{id}
</select>
```

类似对应与下面的JDBC代码

```java
String selectPerson = "SELECT * FROM PERSON WHERE ID=?";
PreparedStatement ps = conn.prepareStatement(selectPerson);
ps.setInt(1,id);
```

##### <insert\>

插入语句大致类似，有几点必须特别注意

1. 如果传入参数有多个，那么接口映射器参数必须使用@Param()传值绑定，否则发生绑定错误

```java
void insertUser(@Param("email") String email,@Param("password") String password, @Param("name") String name);
```

```xml
    <insert id="insertUser" useGeneratedKeys="true"  keyColumn="id">
        insert into user(email,password,name) values(#{email},#{password},#{name});
    </insert>
```

2. 主键自增，绑定数据库只需开启useGeneratedKeys="true"  keyColumn="id"，keyColumn表示数据库中的自增主键列，如果绑定对应DAO的主键，必须指定参数类型 parameterType 为哪一个DAO，并使用keyProperty制定该DAO的主键字段，否则仍会发生绑定错误

##### <resultMap\>

表示映射器返回的字段的映射，指定column与property

### 返回类型

#### select返回类型

默认就是List，resultType=对象，返回的就是list，只有一个则是对象

#### insert返回类型

插入单个数据返回成功插入数据的条数，成功为1，失败直接返回sql的异常

#### delete返回类型

删除一个不存在的值返回0

正常删除返回1

删除存在外键约束的报异常

### 注解配置

常与SpringBoot配合使用

#### @Mapper

@Mapper，用于接口上，表示这是一个接口映射器

#### @Select

查询语句

#### @Results/@Result

表示resultMap，表与bean的字段映射，内部一般为@Result，与其配合使用

```java
    //根据院系id获取专业列表
    @Select("select * from major where department_id=#{department_id}")
    @Results({
            @Result(column = "id", property = "id"),
            @Result(column = "name", property = "name"),
            @Result(column = "department_id",property = "departmentId")
    }
    )
    List<Major> getMajorListByDepartmentId(int departmentId);
```

#### @ResultMap

可引用@Results的结果避免重复映射：

```java
.......省略一万行sql
@Results(id = "accResultMap",value = {
            @Result(property = "accId",column = "acc_id"),
            @Result(property = "accRole",column = "acc_login"),
            @Result(property = "accName",column = "acc_name"),
            @Result(property = "accPwd",column = "acc_pass"),
    })
Account queryAccById(Integer accid);
 
.......省略一万行sql
@ResultMap("accResultMap")
Account queryByName();


@Result是@Results的儿子。@Results可以算作是@ResultMap的干儿子！
```

#### @Options 

配置可选项，与其他SQL注解语句混用，如获取自增主键，源码为：

```java
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.METHOD})
public @interface Options {
    boolean useCache() default true;

    boolean flushCache() default false;

    ResultSetType resultSetType() default ResultSetType.FORWARD_ONLY;

    StatementType statementType() default StatementType.PREPARED;

    int fetchSize() default -1;

    int timeout() default -1;

    boolean useGeneratedKeys() default false;

    String keyProperty() default "id";

    String keyColumn() default "";
}
```
##### 插入数据后获取自增主键值

```java
@Insert("INSERT INTO `wx_act` (`name`, `modelId`, `image`) VALUES (#{name}, #{modelId}, #{image})")
@Options(useGeneratedKeys = true, keyProperty = "id", keyColumn = "id")
int saveMoonCoke(MidAutumn midAutumn);
```
配置选项useGeneratedKeys，keyProperty，keyColumn即可

#### 常见错误

##### invalid bound statement (not found)

显然映射器xml没能与映射器接口绑定，此时检查映射器接口与映射器xml是否完全符合绑定规则。如果无误，可能是idea没能把包java下的xml文件进行编译，在pom.xml下的<build\>处增加：

```xml
    <!--指定扫描java包下的xml文件-->
    <resources>
      <resource>
        <directory>src/main/java</directory>
        <includes>
          <include>**/*.xml</include>
        </includes>
      </resource>
    </resources>
```

如果仍错误，注意如果是result语句，且开启自增那么参数要特别注意，见上述<insert\>

#####  Cannot find class:com.mysql.jdbc.Drive** 

两种可能，1是数据库源的配置写错了，要检查。2是可能没引入数据库包，在pom.xml导入依赖即可

注意mysql5.x是com.mysql.jdbc.Drive，mysql6.x及以上是com.mysql.cj.jdbc.Drive

