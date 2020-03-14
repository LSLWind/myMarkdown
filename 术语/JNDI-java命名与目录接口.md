JNDI是 Java 命名与目录接口（Java Naming and Directory Interface），在J2EE规范中是重要的规范之一，说白了就是为了解耦，将配置与应用分开，在文件中统一配置便于修改，使用时直接读取文件信息。

<img src="https://upload-images.jianshu.io/upload_images/7650139-ee21f8c8ebf92e92.png?imageMogr2/auto-orient/strip|imageView2/2/w/785/format/webp" style="zoom:50%;" />

没有JNDI的做法：
程序员开发时，知道要开发访问MySQL数据库的应用，于是将一个对 MySQL JDBC 驱动程序类的引用进行了编码，并通过使用适当的 JDBC URL 连接到数据库。
就像以下代码这样：

```java
Connection conn=null; 
try { 
	Class.forName("com.mysql.jdbc.Driver", true, Thread.currentThread().getContextClassLoader()); 
	conn=DriverManager.getConnection("jdbc:mysql://MyDBServer?user=xxx&password=xxx"); 
	...... 
	conn.close(); 
} catch(Exception e) { 
	e.printStackTrace(); 
} finally { 
	if(conn!=null) { 
		try { 
			conn.close(); 
		} catch(SQLException e) {} 
	}
}
```

没有JNDI的做法存在的问题：
1、数据库服务器名称MyDBServer 、用户名和口令都可能需要改变，由此引发JDBC URL需要修改；
2、数据库可能改用别的产品，如改用DB2或者Oracle，引发JDBC驱动程序包和类名需要修改；
3、随着实际使用终端的增加，原配置的连接池参数可能需要调整；
4、......

**解决办法：**
程序员应该不需要关心“具体的数据库后台是什么？JDBC驱动程序是什么？JDBC URL格式是什么？访问数据库的用户名和口令是什么？”等等这些问题，程序员编写的程序应该没有对 JDBC 驱动程序的引用，没有服务器名称，没有用户名称或口令 —— 甚至没有数据库池或连接管理。而是把这些问题交给J2EE容器来配置和管理，程序员只需要对这些配置和管理进行引用即可。

由此，就有了JNDI。

**用了JNDI之后的做法：**
首先，在在J2EE容器中配置JNDI参数，定义一个数据源，也就是JDBC引用参数，给这个数据源设置一个名称；然后，在程序中，通过数据源名称引用数据源从而访问后台数据库。
具体操作如下（以JBoss为例）：
**1、配置数据源**
在JBoss 的 D:/jboss420GA/docs/examples/jca 文件夹下面，有很多不同数据库引用的数据源定义模板。将其中的 mysql-ds.xml 文件Copy到你使用的服务器下，如 D:/jboss420GA/server/default/deploy。
修改 mysql-ds.xml 文件的内容，使之能通过JDBC正确访问你的MySQL数据库，如下：

```html
<?xml version="1.0" encoding="UTF-8"?>
<datasources>
<local-tx-datasource>
  <jndi-name>MySqlDS</jndi-name>
  <connection-url>jdbc:mysql://localhost:3306/lw</connection-url>
  <driver-class>com.mysql.jdbc.Driver</driver-class>
  <user-name>root</user-name>
  <password>rootpassword</password>
<exception-sorter-class-name>org.jboss.resource.adapter.jdbc.vendor.MySQLExceptionSorter</exception-sorter-class-name>
  <metadata>
  <type-mapping>mySQL</type-mapping>
  </metadata>
</local-tx-datasource>
</datasources>
```

这里，定义了一个名为MySqlDS的数据源，其参数包括JDBC的URL，驱动类名，用户名及密码等。

**2、在程序中引用数据源：**

```java
Connection conn=null; 
try { 
	Context ctx = new InitialContext(); 
	Object datasourceRef = ctx.lookup("java:MySqlDS"); 
	//引用数据源 
	DataSource ds = (Datasource) datasourceRef; 
	conn = ds.getConnection(); 
	...... 
	c.close(); 
} catch(Exception e) { 
	e.printStackTrace(); 
} finally { 
	if(conn!=null) { 
		try { 
			conn.close(); 
		} catch(SQLException e) {} 
	} 
}
```

直接使用JDBC或者通过JNDI引用数据源的编程代码量相差无几，但是现在的程序可以不用关心具体JDBC参数了。
在系统部署后，如果数据库的相关参数变更，只需要重新配置 mysql-ds.xml 修改其中的JDBC参数，只要保证数据源的名称不变，那么程序源代码就无需修改。

由此可见，JNDI避免了程序与数据库之间的紧耦合，使应用更加易于配置、易于部署。