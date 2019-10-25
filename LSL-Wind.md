### 服务器基础安装配置

#### mysql

* 安装:sudo apt install mysql-server -y

版本跟镜像源有关，使用腾讯云服务器mysql安装版本为5.7

* 更改密码（安全模式安装）(**这一步不是必须的，安装完之后本地登录输入密码按回车即可**)

sudo mysql_secure_installation

会提示一系列选项，根据内容设置即可

也可进入mysql后更改root密码

5.7以前:UPDATE mysql.user SET Password = PASSWORD('password') WHERE User = 'root';

5.7以后:UPDATE mysql.user SET authentication_string = PASSWORD('password') WHERE User = 'root';

然后刷新指令：FLUSH PRIVILEGES;

* 登录 sudo mysql -u root -p

* 开启mysql服务

sudo systemctl start mysql，安装完的时候已经启动过了

* 重启后开启mysql服务

sudo systemctl enable mysql

* 远程访问

按理说应该先授权在访问，不过出现错误2003，cant connect to mysql

**参考：https://blog.csdn.net/qq_32144341/article/details/52403388**

1. 在控制台输入,进入mysql目录
   cd /etc/mysql

2. 打开my.cnf文件，找到 bind-address = 127.0.0.1 在前面加上#注释掉，如下：
   #bind-address = 127.0.0.1

3. 然后在添加如下代码

   skip-external-locking
   skip-name-resolve

若在my.cnf文件中找不到#bind-address = 127.0.0.1则在/etc/mysql/mysql.conf.d/ 文件夹中打开 mysqld.cnf文件修改即可。

4. 重启mysql服务

   service mysql restart

5. 本地登录mysql

   刷新权限 flush privileges; 

   6.登录局mysql，创建远程访问用户并进行授权

**GRANT ALL  PRIVILEGES ON \*.\* TO 'lsl'@'%'IDENTIFIED BY 'liang19425573436' WITH GRANT OPTION;**    

该语句将所有权限授予用户lsl并赋予密码liang19425573436，lsl可远程登录，本地登录仍使用root即可

该授权方式不限制ip，@后面跟的是ip，%即所有ip，也可以指定Ip（将指定ip替换为%）以保证安全性

* 远程连接：**mysql -h ip -P 3306(端口号) -u lsl -p （用户及密码)**

### 项目开发设置

#### idea+maven创建项目工程

1.打开IDEA,使用Maven,选择 create from archetype(使用原型创建项目),选择org.apache.maven.archetypes:maven-archtype-webapp，之后自动(import changes)导入资源。（注：必须使用企业版的IDEA，社区版不支持开发Web）

2.配置Tomcat服务器，选择Add Configuration,点击+ 往下滑，open more items 选择Tomcat Server-local，配置Server,先选择Deployment，点击+，选择artifact,选择 war exploded(exploded 展开,即直接将开发的包部署到服务器文件夹里，因此支持热部署，开发的时候这样更方便，但部署到远程服务器的时候仍然需要war包，即将项目压缩成war包后放到远程服务器对应文件下)，再在Server下(On 'Update' action和下面那个)选择Update classes and resources，即热部署方式。

3.启动Tomcat,访问index.jsp进行测试

#### 设置图床上传图片

本地编写markdown文档截图时换一个地方就会加载不出来图片，因此将截图上传到图床是一个很好地选择，同时如果图片很多，网站博客也可以远程引用图床中的图片而不放在本地，此处使用：

**[https://sm.ms](https://sm.ms/)**

#### 设置SSM目录结构

![image.png](https://i.loli.net/2019/10/11/tr3acqOohjZklP7.png)

entity：放置javaBean（POJO），比如说用户user，customer等实体

mapper：接口，连接数据库与项目的中间层，同时对数据库的操作增查删改也写在里面

service：业务逻辑接口，用来对从数据库里拿出的数据进行进一步的操作

serviceImpl：业务逻辑具体实现

controller：规划访问路径，同时也用来接收从前台数据库传输的数据

resources：各种配置文件

数据库<----->mapper<------>service/serviceImpl<--------->controller

pages：客户端显示页面

static：前端引用资源

WEB-INF页面只对服务端开放，对客户端是不可见的。在web项目中，为了安全，可能需要把jsp文件放在WEB-INF目录下，这样如果我们的页面中出现超链接a标签或者js的location.href去直接转向到WEB-INF下的某一个jsp页面，那么就会引用不到，因为这样的请求方式是客户端的请求。
由于WEB-INF下对客户端是不可见的，所以相关的资源文件，如css，javascript和图片等资源文件不能放在WEB-INF下。

### 设计表结构

#### 创建数据库

````mysql
create database lslwind;
use lslwind;
````

#### 创建表用户User

```mysql
create table user(
id int(11) primary key not null auto_increment,
email varchar(30)not null,
password varchar(20) not null,
name varchar(20) not null,
sex varchar(2) check(sex='男' or sex='女'),
visits int(11),
head_img varchar(100),
airticle_count int,
airticle_types text
)ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

visits为总访问量，head_img为用户头像，在本地存储，数据库里放置引用路径，ariticle_count为文章总数，airticle_types为自定义文章分类，每个分类使用/隔开

#### 创建表文章Ariticle

```mysql
create table airticle(
id int(15) primary key not null auto_increment,
user_id int(11) not null,
title varchar(100) not null,
type varchar(30),
visits int(11),
update_time datetime not null,
content text,
foreign key(user_id) references user(id)
)ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

### api 文档





### 后端设计

先不管前端，前端样式处理可能需要耗费大量时间，先专注后台数据处理

#### 创建实体类（DAO）User/Article

根据设计的数据库表创建对应的DAO，放于包entity下，即创建数据库表对应的java Bean，略

#### 配置SpringMVC

##### pom.xml导入springMVC依赖

```xml
<dependency>
  <groupId>org.springframework</groupId>
  <artifactId>spring-web</artifactId>
  <version>4.3.1.RELEASE</version>
</dependency>
<dependency>
  <groupId>org.springframework</groupId>
  <artifactId>spring-webmvc</artifactId>
  <version>4.3.1.RELEASE</version>
</dependency>
```
##### web.xml中配置DispatcherServlet与映射

```xml
<web-app>
    <servlet>
        <servlet-name>home</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <load-on-startup>1</load-on-startup><!--启动时立即加载Servlet-->
    </servlet>

    <servlet-mapping>
        <servlet-name>home</servlet-name>
        <url-pattern>/home/*</url-pattern>
    </servlet-mapping>

</web-app>
```

所有/home下的url请求都由DispatcherServlet调度，配置其他路径同理，对应的要写相应的控制器并配置视图控制器

##### WEB-INF下配置home-servlet.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:context="http://www.springframework.org/schema/context"
    xmlns:mvc="http://www.springframework.org/schema/mvc"
    xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-4.2.xsd
        http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.2.xsd
        http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc-4.2.xsd">

    <!-- 开启注解 -->
    <mvc:annotation-driven />
    <!-- 配置自动扫描的包，完成 Bean 的创建和自动依赖注入的功能 -->
    <context:component-scan base-package="controller" />
    <!-- 默认静态资源处理 -->
    <mvc:default-servlet-handler/>
    <!-- 配置视图解析器 -->
    <bean id="viewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="prefix" value="/pages/"></property>
        <property name="suffix" value=".jsp"></property>
    </bean>
</beans>
```

开启注解，配置视图解析器，前缀为webapp下的/pages，后缀为jsp，使用jsp可在response时更新数据

处理过程为前端表单数据请求request->DispatcherServlet（绑定前缀路径-servlet.xml，控制器映射要省略该前缀）->调度Controller找寻RequestMapping处理，调度过程中DispatcherServlet绑定的url前缀是缺省的，不用再加->根据返回值查找视图解析器下的资源，还是由DispatcherServlet调度，如果返回的View名称与映射路径相同则会陷入死循环，DispatcherServlet优先分配到控制器，控制器没有映射返回View，但是前端仍然显示控制器映射的url，即后端View资源的url不会显示到->返回对应View

#### 用户注册

从前端读取表单数据，检验数据（邮箱）是否存在，将数据插入数据库中































### 前端设计

#### 编写登录界面，用户注册

使用前端模板

#### 编写博客，markdown编辑器嵌入

使用开源在线Markdown编辑器代码，参见**https://pandao.github.io/editor.md/**

使用代码为

```html
<link rel="stylesheet" href="editormd/css/editormd.css" />
<div id="test-editor">
    <textarea style="display:none;">
        此处是Markdown编辑地方
    </textarea>
</div>
<script src="https://cdn.bootcss.com/jquery/1.11.3/jquery.min.js"></script>
<script src="editormd/editormd.min.js"></script>
<script type="text/javascript">
    $(function() {
        var editor = editormd("test-editor", {
            //此处配置Markdown编辑器属性
            path   : "editormd/lib/"
        });
    });
</script>js
```

可以看到这段代码引用了一些资源，在官网点击下载，下载完之后，文件为editor.md-master

将css/editormd.css，css/editormd.min.css，src/editormd.js，editormd.min.js，examples/js/jquery.min.js，lib，fonts移动到本地项目

lib与fonts是关键目录，其它是关键样式，引入之后目录如下：

![image.png](https://i.loli.net/2019/10/11/nRFdlH9bCj4VvSN.png)

editArticle.html代码如下：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>写博客</title>
    <link rel="stylesheet" href="../static/css/style.css" />
    <link rel="stylesheet" href="../static/css/editormd.css" />
</head>
<body>
<div id="layout">
    <header>
        <h1>我于杀戮之中盛放，一如黎明前的花朵</h1>
    </header>
    <div id="editormd">
        <!--文本域-----数据读取-->
        <textarea style="display:none;"></textarea>
    </div>
</div>
<script src="../static/js/jquery.min.js"></script>
<script src="../static/js/editormd.min.js"></script>
<script type="text/javascript">
    let editor;
    //加载editor，对应上述div设置的id-editormd，通过path应用lib渲染
    $(function() {
        editor = editormd("editormd", {
            width   : "80%",
            height  : 640,
            syncScrolling : "single",
            path    : "../static/lib/"
        });

    });
</script>
</body>
</html>
```

代码分析：

<textarea\>是写文本的地方，放在一个div下，该div的id为editormd

之后使用脚本渲染该div（下的textarea），使用函数editormd("id",{});第一个参数就是textarea对应的id，第二个参数为属性配置，path最为关键，引用外部lib，否则加载不出来

当然也可以外部引用脚本

常用属性配置：

```js
$(function(){
editormd("test-editormd", {
        placeholder : "此处开始编写您要发布的内容...",
        width   : "100%",
        height  : 640,
        codeFold : true,//代码折叠
        syncScrolling : "single",//滚动同步
        imageUpload: true,//图片上传功能
        imageFormats : ["jpg", "jpeg", "gif", "png", "bmp", "webp"],
        imageUploadURL : "../article/uploadImage",//图片上传路径
        emoji: true,//加载表情
        taskList: true,//任务列表
        tex: true,                   // 开启科学公式TeX语言支持，默认关闭
        flowChart: true,             // 开启流程图支持，默认关闭
        sequenceDiagram: true,       // 开启时序/序列图支持，默认关闭,
        //这个配置在simple.html中并没有，但是为了能够提交表单，使用这个配置可以让构造出来的HTML代码直接在第二个隐藏的textarea域中，方便post提交表单。
        saveHTMLToTextarea : true,
           toolbarIcons : function() {//自定义工具栏,"|"为分隔符，默认显示所有工具
                return ["undo","redo","|","bold","italic","quote","uppercase","lowercase","|","h1","h2","h3","h4","|","list-ul","list-ol","hr","|","link","image","code","code-block","table","html-entities","|","watch","preview","fullscreen","clear","|","help"]
            },
});
```





### 可能问题

#### Computed property are not supported by current javascript version

引入editor.md时可能被idea报错，原因是idea设置的javascript版本过低，解决办法：

1. alt+ctrl+s打开设置

2. 选择"Languages and Frameworks"，选择“JavaScript”
3. 选择'ECMAScript 6'