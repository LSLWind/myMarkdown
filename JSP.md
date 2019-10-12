#### 概述

Servlet作为中间件用于数据的交互处理，先描述一下处理过程

前端请求request->应用层协议http/https封装info->网络层/传输层->服务器网络层->http/https->info，服务器接收info，Tomcat Server封装成request，数据拿来，处理后封装成response进行响应，如何返回响应,response中只有通过流的方式进行数据传输

1. PrintWriter out=response.getWriter();
2. out.println()

为了在响应中动态加载并显示信息，出现了各种后台语言/技术

可以将jsp看成这样一种后台技术

request->容器识别jsp->加载jsp，转换成servlet后进行数据处理并返回页面（数据可以实时嵌入页面中）

作为后台动态语言，jsp自有其语法格式，但是没有实现前后端分离

### 基本语法

##### 内嵌java代码<% ... %>

```html
<% 代码片段 %>
```

##### 显示中文<%@ page  ... %>

```html
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
```

##### 声明变量<%! ... %>

```html
<%! int i = 0; %> 
<%! int a, b, c; %> 
<%! Circle a = new Circle(2.0); %> 
```

##### 表达式<%= ... %>

表达式的值会被转化成String，然后插入到表达式出现的地方。表达式不能使用分号来结束表达式。

```html
<%= 表达式 %>
例：
<p>
   今天的日期是: <%= (new java.util.Date()).toLocaleString()%>
</p>
```

##### 注释<%-- ... --%>

<%-- 注释 --%>

#### JSP指令<%@ ... %>

JSP指令用来设置与整个JSP页面相关的属性。

JSP指令语法格式：

```html
<%@ directive attribute="value" %>
```

这里有三种指令标签：

| **指令**           | **描述**                                                  |
| ------------------ | --------------------------------------------------------- |
| <%@ page ... %>    | 定义页面的依赖属性，比如脚本语言、error页面、缓存需求等等 |
| <%@ include ... %> | 包含其他文件                                              |
| <%@ taglib ... %>  | 引入标签库的定义，可以是自定义标签                        |

##### 指令@ page：定义整个页面的相关信息

| **属性**           | **描述**                                            |
| ------------------ | --------------------------------------------------- |
| buffer             | 指定out对象使用缓冲区的大小                         |
| autoFlush          | 控制out对象的 缓存区                                |
| contentType        | 指定当前JSP页面的MIME类型和字符编码                 |
| errorPage          | 指定当JSP页面发生异常时需要转向的错误处理页面       |
| isErrorPage        | 指定当前页面是否可以作为另一个JSP页面的错误处理页面 |
| extends            | 指定servlet从哪一个类继承                           |
| import             | 导入要使用的Java类                                  |
| info               | 定义JSP页面的描述信息                               |
| isThreadSafe       | 指定对JSP页面的访问是否为线程安全                   |
| language           | 定义JSP页面所用的脚本语言，默认是Java               |
| session            | 指定JSP页面是否使用session                          |
| isELIgnored        | 指定是否执行EL表达式                                |
| isScriptingEnabled | 确定脚本元素能否被使用                              |

例：

```html
<%@ page import="java.util.Date" %>
```

##### 指令@ include:包含其他文件，包含的文件就好像是该JSP文件的一部分，会被同时编译执行

```html
<%@ include file="文件相对 url 地址" %>
```

编译器默认在当前路径下寻找。

#### JSP行为<jsp: ... />

JSP行为标签使用XML语法结构来控制servlet引擎。它能够动态插入一个文件，重用JavaBean组件，引导用户去另一个页面，为Java插件产生相关的HTML等等。主要是可以用javaBean

```html
<jsp:action_name attribute="value" />
```

行为标签基本上是一些预先就定义好的函数，下表罗列出了一些可用的JSP行为标签：

| **语法**        | **描述**                                                   |
| --------------- | ---------------------------------------------------------- |
| jsp:include     | 用于在当前页面中包含静态或动态资源                         |
| jsp:useBean     | 寻找和初始化一个JavaBean组件                               |
| jsp:setProperty | 设置 JavaBean组件的值                                      |
| jsp:getProperty | 将 JavaBean组件的值插入到 output中                         |
| jsp:forward     | 从一个JSP文件向另一个文件传递一个包含用户请求的request对象 |
| jsp:plugin      | 用于在生成的HTML页面中包含Applet和JavaBean对象             |
| jsp:element     | 动态创建一个XML元素                                        |
| jsp:attribute   | 定义动态创建的XML元素的属性                                |
| jsp:body        | 定义动态创建的XML元素的主体                                |
| jsp:text        | 用于封装模板数据                                           |

**所有的动作要素都有两个属性：id属性和scope属性。**

- id属性：

  id属性是动作元素的唯一标识，可以在JSP页面中引用。动作元素创建的id值可以通过PageContext来调用。

- scope属性：

  该属性用于识别动作元素的生命周期。 id属性和scope属性有直接关系，scope属性定义了相关联id对象的寿命。 scope属性有四个可能的值： (a) page, (b)request, (c)session, 和 (d) application。

使用行为标签就可以动态加载页面，复用其它类

##### jsp:include把指定文件(jsp)插入正在生成的页面

```html
<jsp:include page="相对 URL 地址" flush="true" />
```

例：

date.jsp

```jsp
<%@ page import="java.util.Date" %>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<<p>
今天的日期是:<%=(new Date().toLocaleString())%>
</p>
```

动态引用date.jsp的hello.jsp

```jsp
<%@ page import="java.util.Date" %>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head><title>Hello World</title></head>
<body>
<jsp:include page="date.jsp" flush="true" />
</body>
</html>
```

##### jsp:useBean加载一个将在JSP页面中使用的JavaBean。

```html
<jsp:useBean id="name" class="package.class" />
```

在类载入后，可以通过 jsp:setProperty 和 jsp:getProperty 动作来修改和检索bean的属性。

##### jsp:setProperty设置已经实例化的Bean对象的属性

用法1：

```jsp
<jsp:useBean id="myName" ... />
...
<jsp:setProperty name="myName" property="someProperty" value="xxx".../>
```

用法2：

```jsp
<jsp:useBean id="myName" ... >
...
   <jsp:setProperty name="myName" property="someProperty" value="xxx".../>
</jsp:useBean>
```

即通过id检索javaBean，指定属性property（value可选，为属性设置值）

##### jsp:getProperty提取指定Bean属性的值，转换成字符串，然后输出

```jsp
<jsp:useBean id="myName" ... />
...
<jsp:getProperty name="myName" property="someProperty" .../>
```

即通过id检索javaBean，获取指定属性property

### JSP隐含对象

JSP支持九个自动定义的变量，江湖人称隐含对象，可以直接使用它们而不用显式声明

| **对象**    | **描述**                                                     |
| ----------- | ------------------------------------------------------------ |
| request     | **HttpServletRequest**类的实例                               |
| response    | **HttpServletResponse**类的实例                              |
| out         | **PrintWriter**类的实例，用于把结果输出至网页上              |
| session     | **HttpSession**类的实例                                      |
| application | **ServletContext**类的实例，与应用上下文有关                 |
| config      | **ServletConfig**类的实例                                    |
| pageContext | **PageContext**类的实例，提供对JSP页面所有对象以及命名空间的访问 |
| page        | 类似于Java类中的this关键字                                   |
| Exception   | **Exception**类的对象，代表发生错误的JSP页面中对应的异常对象 |

**request对象**

request对象是javax.servlet.http.HttpServletRequest 类的实例。每当客户端请求一个JSP页面时，JSP引擎就会制造一个新的request对象来代表这个请求。request对象提供了一系列方法来获取HTTP头信息，cookies，HTTP方法等等。

**response对象**

response对象是javax.servlet.http.HttpServletResponse类的实例。当服务器创建request对象时会同时创建用于响应这个客户端的response对象。response对象也定义了处理HTTP头模块的接口。通过这个对象，开发者们可以添加新的cookies，时间戳，HTTP状态码等等。

**out对象**

out对象是 javax.servlet.jsp.JspWriter 类的实例，用来在response对象中写入内容。

| **方法**                     | **描述**                 |
| ---------------------------- | ------------------------ |
| **out.print(dataType dt)**   | 输出Type类型的值         |
| **out.println(dataType dt)** | 输出Type类型的值然后换行 |
| **out.flush()**              | 刷新输出流               |

**session对象**

session对象是 javax.servlet.http.HttpSession 类的实例。和Java Servlets中的session对象有一样的行为。

session对象用来跟踪在各个客户端请求间的会话。

**application对象**

application对象直接包装了servlet的ServletContext类的对象，是javax.servlet.ServletContext 类的实例。

这个对象在JSP页面的整个生命周期中都代表着这个JSP页面。这个对象在JSP页面初始化时被创建，随着jspDestroy()方法的调用而被移除。通过向application中添加属性，则所有组成web应用的JSP文件都能访问到这些属性。

**config对象**

config对象是 javax.servlet.ServletConfig 类的实例，直接包装了servlet的ServletConfig类的对象。

这个对象允许开发者访问Servlet或者JSP引擎的初始化参数，比如文件路径等。

**pageContext 对象**

pageContext对象是javax.servlet.jsp.PageContext 类的实例，用来代表整个JSP页面。

这个对象主要用来访问页面信息，同时过滤掉大部分实现细节。

这个对象存储了request对象和response对象的引用。application对象，config对象，session对象，out对象可以通过访问这个对象的属性来导出。

pageContext对象也包含了传给JSP页面的指令信息，包括缓存信息，ErrorPage URL,页面scope等。

PageContext类定义了一些字段，包括PAGE_SCOPE，REQUEST_SCOPE，SESSION_SCOPE， APPLICATION_SCOPE。它也提供了40余种方法，有一半继承自javax.servlet.jsp.JspContext 类。

其中一个重要的方法就是removeArribute()，它可接受一个或两个参数。比如，pageContext.removeArribute("attrName")移除四个scope中相关属性，但是下面这种方法只移除特定scope中的相关属性：

```html
pageContext.removeAttribute("attrName", PAGE_SCOPE);
```

**page 对象**

这个对象就是页面实例的引用。它可以被看做是整个JSP页面的代表。

page 对象就是this对象的同义词。

**exception 对象**

exception 对象包装了从先前页面中抛出的异常信息。它通常被用来产生对出错条件的适当响应。