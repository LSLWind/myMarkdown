### 引入

在maven中引入：

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```

在html标签下引入：

```
<html xmlns:th="http://www.thymeleaf.org">
```

### 变量表达式${}

变量表达式即 OGNL 表达式或 Spring EL 表达式(在 Spring 术语中也叫 model attributes)。语法为： `${session.user.name}`

它们将以HTML标签的一个属性来表示：

```
<span th:text="${book.author.name}">  
<li th:each="book : ${books}">  
```

### 文字国际化表达式

文字国际化表达式允许我们从一个外部文件获取区域文字信息(.properties)，用 Key 索引 Value，还可以提供一组参数(可选).

```
#{main.title}  
#{message.entrycreated(${entryId})}  
```

可以在模板文件中找到这样的表达式代码：

```
<table>  
  ...  
  <th th:text="#{header.address.city}">...</th>  
  <th th:text="#{header.address.country}">...</th>  
  ...  
</table>  
```

### URL 表达式@{}

URL 表达式指的是把一个有用的上下文或回话信息添加到 URL，这个过程经常被叫做 URL 重写。加载图片时用到@{}，如 `@{/order/list}`

URL还可以设置参数：`@{/order/details(id=${orderId})}`

相对路径：`@{../documents/report}`

```
<form th:action="@{/createOrder}">  
<a href="main.html" th:href="@{/main}">
```

### 星号表达式*{}

星号语法评估在选定对象上表达，而不是整个上下文，选定对象就是父标签的值，如下：

```
<div th:object="${session.user}">
  <p>Name: <span th:text="*{firstName}">Sebastian</span>.</p>
  <p>Surname: <span th:text="*{lastName}">Pepper</span>.</p>
  <p>Nationality: <span th:text="*{nationality}">Saturn</span>.</p>
</div>
```

这是完全等价于：

```
<div th:object="${session.user}">
  <p>Name: <span th:text="${session.user.firstName}">Sebastian</span>.</p>
  <p>Surname: <span th:text="${session.user.lastName}">Pepper</span>.</p>
  <p>Nationality: <span th:text="${session.user.nationality}">Saturn</span>.</p>
</div>
```

## 常用th标签

### \<th:if>

用在标签内表示if 语句，如果表达式成立则渲染语句，否则不渲染，最常用的直接使用${}，如果{}中的语句成立则执行数据渲染，如：

```html
<li th:if="${customer!=null}">
    <a class="dropdown-button" href="#!" data-activates="dropdown1">
    <img class="responsive-img circle " src="/mall/image/风.jpg" style="width: 50px;height: 50px">
     <i class="material-icons right">arrow_drop_down</i></a>
</li>
<li th:if="${customer==null}">
      <a class="waves-effect waves-light btn">登录/注册</a>
</li>
```

如果想使用外部条件进行某些判断，可使用符号缩写，表达式中内置的值为：

```
gt：great than（大于）>
ge：great equal（大于等于）>=
eq：equal（等于）==
lt：less than（小于）<
le：less equal（小于等于）<=
ne：not equal（不等于）!=
```

示例：

```html
<span th:if="${index} gt '7'"></span>
```

```html
<a th:if="${order.state} eq '已发货'" class="btn">已发货</a>
<a th:if="${order.state} eq '未发货'" class="btn" th:onclick="updateOrder([[${order.id}]])">设置为已发货</a>
```

### \<th:each\>

用于迭代集合

```html
<table>
   <thead>
      <tr>
         <th>序号</th>
         <th>用户名</th>
         <th>密码</th>
         <th>用户昵称</th>
      </tr>
      <tr th:each="user:${userlist}">
         <td th:text="${user.id}"></td>
         <td th:text="${user.username}"></td>
         <td th:text="${user.password}"></td>
         <td th:text="${user.petname}"></td>
      </tr>
   </thead>
</table>
```

th:each属性用于迭代循环，语法：th:each="obj,iterStat:${objList}"

迭代对象可以是java.util.List,java.util.Map,数组等;

obj就是迭代对象

iterStat称作状态变量，属性有：
    index:当前迭代对象的index（从0开始计算）
    count: 当前迭代对象的index(从1开始计算)
    size:被迭代对象的大小
    current:当前迭代变量
    even/odd:布尔值，当前循环是否是偶数/奇数（从0开始计算）
    first:布尔值，当前循环是否是第一个
    last:布尔值，当前循环是否是最后一个

```html
<div th:each="depository:${depositoryMap}">
                <div class="section">
                    <div class="row" >
                        <div class="col s9">
                            <div class="row">
                                <div class="col s4">名称：<p th:text="${depository.value.name}"></p> </div>
                                <div class="col s4">库存量：<p th:text="${depository.key.count}"></p> </div>
                                <div class="col s4">现价：<p th:text="${depository.value.discountPrice}"></p></div>
```

### js中使用后台的传值

js通过model取值

```js
<script th:inline="javascript">
    var message = [[${message}]];
    console.log(message);
</script>
```

注：script标签中 th:inline 一定不能少，通常在取值的前后会加上不同的注释

### th:onclick向js函数中传递参数

使用th:onclick，进行字符串的拼接

```js
<a class="waves-effect waves-light btn col s4" th:onclick="'javascript:addToCart('+${book.id}+','+${book.name}+','+${book.discountPrice}+','+${book.imgs}+')' ">加入购物车</a>
```

报错：Only variable expressions returning numbers or booleans are allowed in this context, any other datatypes are not trusted in the context of this expression, including Strings or any other object that could be rendered as a text literal. A typical case is HTML attributes for event handlers (e.g. "onload"), in which textual data from variables should better be output to "data-*" attributes and then read from the event handl

只允许数字或者boolean

尝试第二种：自定义id进行拼接：

```html
<a class="waves-effect waves-light btn col s4" th:name-id="${book.name}" th:imgs-id="${book.imgs}" th:onclick="'javascript:addToCart('+${book.id}+','+this.getAttribute('name-id')+','+${book.discountPrice}+','+this.getAttribute('imgs-id')+')'">加入购物车</a>
```

还是无效，尝试第三种：[[]]包裹

```html
<a class="waves-effect waves-light btn col s4" th:onclick="addToCart([[${book.id}]],[[${book.name}]],[[${book.discountPrice}]],[[${book.imgs}]])">加入购物车</a>
```

这回可以正常解析

### 在js中赋值（null情况）

EL表达式相当于写代码，为了判断空可以使用判断运算符，否则引用null会解析出错

```
var customerId=[[${customer==null?0:customer.id}]];
```

## 常用的使用方法

### 1、赋值、字符串拼接

```
<p  th:text="${collect.description}">description</p>
<span th:text="'Welcome to our application, ' + ${user.name} + '!'">
```

字符串拼接还有另外一种简洁的写法

```
<span th:text="|Welcome to our application, ${user.name}!|">
```

### 2、for 循环

```html
<tr th:each="collect,iterStat : ${collects}"> 
   <th scope="row" th:text="${collect.id}">1</th>
   <td >
      <img th:src="${collect.webLogo}"/>
   </td>
   <td th:text="${collect.url}">Mark</td>
   <td th:text="${collect.title}">Otto</td>
   <td th:text="${collect.description}">@mdo</td>
   <td th:text="${terStat.index}">index</td>
</tr>
```

iterStat称作状态变量，属性同\<th:each>中解析的一样

### 3、内嵌变量

为了模板更加易用，Thymeleaf 还提供了一系列 Utility 对象（内置于 Context 中），可以通过 # 直接访问：

- dates ： *java.util.Date的功能方法类。*
- calendars : *类似#dates，面向java.util.Calendar*
- numbers : *格式化数字的功能方法类*
- strings : *字符串对象的功能类，contains,startWiths,prepending/appending等等。*
- objects: *对objects的功能类操作。*
- bools: *对布尔值求值的功能方法。*
- arrays：*对数组的功能类方法。*
- lists: *对lists功能类方法*
- sets
- maps
   ...

下面用一段代码来举例一些常用的方法：

#### dates

```
/*
 * Format date with the specified pattern
 * Also works with arrays, lists or sets
 */
${#dates.format(date, 'dd/MMM/yyyy HH:mm')}
${#dates.arrayFormat(datesArray, 'dd/MMM/yyyy HH:mm')}
${#dates.listFormat(datesList, 'dd/MMM/yyyy HH:mm')}
${#dates.setFormat(datesSet, 'dd/MMM/yyyy HH:mm')}

/*
 * Create a date (java.util.Date) object for the current date and time
 */
${#dates.createNow()}

/*
 * Create a date (java.util.Date) object for the current date (time set to 00:00)
 */
${#dates.createToday()}
```

#### strings

```
/*
 * Check whether a String is empty (or null). Performs a trim() operation before check
 * Also works with arrays, lists or sets
 */
${#strings.isEmpty(name)}
${#strings.arrayIsEmpty(nameArr)}
${#strings.listIsEmpty(nameList)}
${#strings.setIsEmpty(nameSet)}

/*
 * Check whether a String starts or ends with a fragment
 * Also works with arrays, lists or sets
 */
${#strings.startsWith(name,'Don')}                  // also array*, list* and set*
${#strings.endsWith(name,endingFragment)}           // also array*, list* and set*

/*
 * Compute length
 * Also works with arrays, lists or sets
 */
${#strings.length(str)}

/*
 * Null-safe comparison and concatenation
 */
${#strings.equals(str)}
${#strings.equalsIgnoreCase(str)}
${#strings.concat(str)}
${#strings.concatReplaceNulls(str)}

/*
 * Random
 */
${#strings.randomAlphanumeric(count)}
```

## 使用 Thymeleaf 布局

Spring Boot 2.0 将布局单独提取了出来，需要单独引入依赖：thymeleaf-layout-dialect。

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
<dependency>
    <groupId>nz.net.ultraq.thymeleaf</groupId>
    <artifactId>thymeleaf-layout-dialect</artifactId>
</dependency>
```

定义代码片段

```html
<footer th:fragment="copy"> 
&copy; 2019
</footer>
```

在页面任何地方引入：

```html
<body>
    <div th:insert="layout/copyright :: copyright"></div>
    <div th:replace="layout/copyright :: copyright"></div>
</body>
```

th:insert 和 th:replace 区别，insert 只是加载，replace 是替换。Thymeleaf 3.0 推荐使用 th:insert 替换 2.0 的 th:replace。

返回的 HTML 如下：

```html
<body> 
  <div> &copy; 2019 </div> 
  <footer>&copy; 2019 </footer> 
</body>
```

下面是一个常用的后台页面布局，将整个页面分为头部，尾部、菜单栏、隐藏栏，点击菜单只改变 content 区域的页面，content是可外部引用的布局

```html
<body class="layout-fixed">
  <div th:fragment="navbar"  class="wrapper"  role="navigation">
    <div th:replace="fragments/header :: header">Header</div>
    <div th:replace="fragments/left :: left">left</div>
    <div th:replace="fragments/sidebar :: sidebar">sidebar</div>
    <div layout:fragment="content" id="content" ></div>
    <div th:replace="fragments/footer :: footer">footer</div>
  </div>
</body>
```

任何页面想使用这样的布局只需要替换content 模块即可

```html
<html xmlns:th="http://www.thymeleaf.org" layout:decorator="layout">
 <body>
    <section layout:fragment="content">
  ...
```

也可以在引用模版的时候传参

```html
<head th:include="layout :: htmlhead" th:with="title='Hello'"></head>
```

layout 是文件地址，如果有文件夹可以这样写`fileName/layout:htmlhead`，htmlhead 是指定义的代码片段 如`th:fragment="copy"`

```java
<h4 class="media-heading" th:inline="text">[[${question.title}]]
    <a th:href="@{'/question/'+${question.id}}" th:text="${question.title}"></a>
 </h4>
```