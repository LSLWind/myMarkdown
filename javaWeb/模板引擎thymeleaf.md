### 基础

#### 引入

在maven中引入：

```xml
    <dependency>
        <groupId>org.thymeleaf</groupId>
        <artifactId>thymeleaf</artifactId>
        <version>2.1.4</version>
    </dependency>
```

在html标签下引入：

```
<html xmlns:th="http://www.thymeleaf.org">
```

#### 继承热部署

jsp引擎很方便热部署，thymeleaf默认不是，必须修改配置一下，添加依赖

```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <optional>true</optional>
            <scope>true</scope>
        </dependency>
```



#### 表达式

### 变量表达式

变量表达式即 OGNL 表达式或 Spring EL 表达式(在 Spring 术语中也叫 model attributes)。如下所示：
 `${session.user.name}`

它们将以HTML标签的一个属性来表示：

```
<span th:text="${book.author.name}">  
<li th:each="book : ${books}">  
```

### 选择(星号)表达式

选择表达式很像变量表达式，不过它们用一个预先选择的对象来代替上下文变量容器(map)来执行，如下：
 `*{customer.name}`

被指定的 object 由 th:object 属性定义：

```
<div th:object="${book}">  
  ...  
  <span th:text="*{title}">...</span>  
  ...  
</div>  
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

URL 表达式指的是把一个有用的上下文或回话信息添加到 URL，这个过程经常被叫做 URL 重写。加载图片时用到@{}
 `@{/order/list}`

URL还可以设置参数：
 `@{/order/details(id=${orderId})}`

相对路径：
 `@{../documents/report}`

让我们看这些表达式：

```
<form th:action="@{/createOrder}">  
<a href="main.html" th:href="@{/main}">
```

### 变量表达式和星号表达有什么区别吗？

如果不考虑上下文的情况下，两者没有区别；星号语法评估在选定对象上表达，而不是整个上下文
 什么是选定对象？就是父标签的值，如下：

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

当然，美元符号和星号语法可以混合使用：

```
  <div th:object="${session.user}">
      <p>Name: <span th:text="*{firstName}">Sebastian</span>.</p>
      <p>Surname: <span th:text="${session.user.lastName}">Pepper</span>.</p>
      <p>Nationality: <span th:text="*{nationality}">Saturn</span>.</p>
  </div>
```

## 常用th标签都有那些？

| 关键字      | 功能介绍                                     | 案例                                                         |
| ----------- | -------------------------------------------- | ------------------------------------------------------------ |
| th:id       | 替换id                                       | ``                                                           |
| th:text     | 文本替换                                     | `description`                                                |
| th:utext    | 支持html的文本替换                           | `conten`                                                     |
| th:object   | 替换对象                                     | ``                                                           |
| th:value    | 属性赋值                                     | ``                                                           |
| th:with     | 变量赋值运算                                 | ``                                                           |
| th:style    | 设置样式                                     | `th:style="'display:' + @{(${sitrue} ? 'none' : 'inline-block')} + ''"` |
| th:onclick  | 点击事件                                     | `th:onclick="'getCollect()'"`                                |
| th:each     | 属性赋值                                     | `tr th:each="user,userStat:${users}">`                       |
| th:if       | 判断条件                                     | ``                                                           |
| th:unless   | 和th:if判断相反                              | `Login`                                                      |
| th:href     | 链接地址                                     | `Login />`                                                   |
| th:switch   | 多路选择 配合th:case 使用                    | ``                                                           |
| th:case     | th:switch的一个分支                          | `User is an administrator`                                   |
| th:fragment | 布局标签，定义一个代码片段，方便其它地方引用 | ``                                                           |
| th:include  | 布局标签，替换内容到引入的文件               | ` />`                                                        |
| th:replace  | 布局标签，替换整个标签到引入的文件           | ``                                                           |
| th:selected | selected选择框 选中                          | `th:selected="(${xxx.id} == ${configObj.dd})"`               |
| th:src      | 图片类地址引入                               | ``                                                           |
| th:inline   | 定义js脚本可以使用变量                       | ``                                                           |
| th:action   | 表单提交的地址                               | ``                                                           |
| th:remove   | 删除某个属性                                 |                                                              |
| th:attr     | 设置标签属性，多个属性可以用逗号分隔         | 比如`th:attr="src=@{/image/aa.jpg},title=#{logo}"`，此标签不太优雅，一般用的比较少。 |

还有非常多的标签，这里只列出最常用的几个,由于一个标签内可以包含多个th:x属性，其生效的优先级顺序为:
 `include,each,if/unless/switch/case,with,attr/attrprepend/attrappend,value/href,src ,etc,text/utext,fragment,remove。`





#### \<th:if>

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

#### \<th:each\>

迭代集合

```
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

```
<div th:each="depository:${depositoryMap}">
                <div class="section">
                    <div class="row" ><!--具体的仓库，由商店id与仓库名称标识，商品数据-->
                        <div class="col s9">
                            <div class="row">
                                <div class="col s4">名称：<p th:text="${depository.value.name}"></p> </div>
                                <div class="col s4">库存量：<p th:text="${depository.key.count}"></p> </div>
                                <div class="col s4">现价：<p th:text="${depository.value.discountPrice}"></p></div>
                            </div>
                        </div>
                        <div class="col s3">
                            <a class="btn">编辑详情</a>
                        </div>
                    </div>
                </div>
```



#### js中使用后台的传值

3.js通过model取值

    <script th:inline="javascript">
        var message = [[${message}]];
        console.log(message);
    </script>

注：script标签中 th:inline 一定不能少，通常在取值的前后会加上不同的注释

#### onclick向js函数中传递参数

th:onclick，进行字符串的拼接

```js
<a class="waves-effect waves-light btn col s4" th:onclick="'javascript:addToCart('+${book.id}+','+${book.name}+','+${book.discountPrice}+','+${book.imgs}+')' ">加入购物车</a>
```

报错：Only variable expressions returning numbers or booleans are allowed in this context, any other datatypes are not trusted in the context of this expression, including Strings or any other object that could be rendered as a text literal. A typical case is HTML attributes for event handlers (e.g. "onload"), in which textual data from variables should better be output to "data-*" attributes and then read from the event handl

只允许数字或者boolean

尝试第二种：自定义id进行拼接：

```
<a class="waves-effect waves-light btn col s4" th:name-id="${book.name}" th:imgs-id="${book.imgs}" th:onclick="'javascript:addToCart('+${book.id}+','+this.getAttribute('name-id')+','+${book.discountPrice}+','+this.getAttribute('imgs-id')+')'">加入购物车</a>
```

还是无效，尝试第三种：[[]]包裹

```
<a class="waves-effect waves-light btn col s4" th:onclick="addToCart([[${book.id}]],[[${book.name}]],[[${book.discountPrice}]],[[${book.imgs}]])">加入购物车</a>
```

这回可以正常解析

#### 在js中赋值（null情况）

EL表达式相当于写代码，为了判断空可以使用判断运算符，否则引用null会解析出错

```
var customerId=[[${customer==null?0:customer.id}]];
```

给js变量赋值

表达式同样可以在javascript中使用。先用属性声明一下：javascript ( th:inline=”javascript” )，然后我们开始在js中声明变量：（这样用可能出现问题，使用[[]]这样的方式加三目表达式更简单，直接赋值""

```
<script th:inline="javascript">
/*<![CDATA[*/
    ...
    var username = /*[[${session.user.name}]]*/ 'Sebastian';
    ...
/*]]>*/
</script>1234567
```

`/*[[…]]*/`表达式的理解如下：

1. `/*…*/`中的内容在用浏览器打开静态 网页时，会被忽略
2. ‘Sebastian’会在浏览器中显示。静态时。
3. Thymeleaf解析时，会解析`/*[[…]]*/`的内容，并把获得的值替换掉`/*[[…]]*/`后面的内容。 

## 几种常用的使用方法

### 1、赋值、字符串拼接

```
<p  th:text="${collect.description}">description</p>
<span th:text="'Welcome to our application, ' + ${user.name} + '!'">
```

字符串拼接还有另外一种简洁的写法

```
<span th:text="|Welcome to our application, ${user.name}!|">
```

### 2、条件判断 If/Unless

Thymeleaf中使用th:if和th:unless属性进行条件判断，下面的例子中，``标签只有在`th:if`中条件成立时才显示：

```
<a th:if="${myself=='yes'}" > </i> </a>
<a th:unless=${session.user != null} th:href="@{/login}" >Login</a>
```

`th:unless` 于 `th:if` 恰好相反，只有表达式中的条件不成立，才会显示其内容。

也可以使用 `(if) ? (then) : (else)`这种语法来判断显示的内容

### 3、for 循环

```
<tr  th:each="collect,iterStat : ${collects}"> 
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

iterStat称作状态变量，属性有：

- index:当前迭代对象的 index（从0开始计算）
- count: 当前迭代对象的 index(从1开始计算)
- size:被迭代对象的大小
- current:当前迭代变量
- even/odd:布尔值，当前循环是否是偶数/奇数（从0开始计算）
- first:布尔值，当前循环是否是第一个
- last:布尔值，当前循环是否是最后一个

### 4、URL

URL 在 Web 应用模板中占据着十分重要的地位，需要特别注意的是 Thymeleaf 对于 URL 的处理是通过语法 `@{...}` 来处理的。
 如果需要 Thymeleaf 对 URL 进行渲染，那么务必使用 `th:href`，`th:src` 等属性，下面是一个例子

```
<!-- Will produce 'http://localhost:8080/standard/unread' (plus rewriting) -->
 <a  th:href="@{/standard/{type}(type=${type})}">view</a>

<!-- Will produce '/gtvg/order/3/details' (plus rewriting) -->
<a href="details.html" th:href="@{/order/{orderId}/details(orderId=${o.id})}">view</a>
```

设置背景

```
<div th:style="'background:url(' + @{/<path-to-image>} + ');'"></div>
```

根据属性值改变背景

```
 <div class="media-object resource-card-image"  th:style="'background:url(' + @{(${collect.webLogo}=='' ? 'img/favicon.png' : ${collect.webLogo})} + ')'" ></div>
```

几点说明：

- 上例中 URL 最后的`(orderId=${o.id})`表示将括号内的内容作为 URL 参数处理，该语法避免使用字符串拼接，大大提高了可读性
- `@{...}`表达式中可以通过`{orderId}`访问 Context 中的 orderId 变量
- `@{/order}`是 Context 相关的相对路径，在渲染时会自动添加上当前 Web 应用的 Context 名字，假设 context 名字为 app，那么结果应该是 `/app/order`

### 5、内联 js

内联文本：[[...]] 内联文本的表示方式，使用时，必须先用`th:inline="text/javascript/none"`激活，`th:inline`可以在父级标签内使用，甚至作为 body 的标签。内联文本尽管比`th:text`的代码少，不利于原型显示。

```
<script th:inline="javascript">
/*<![CDATA[*/
...
var username = /*[[${sesion.user.name}]]*/ 'Sebastian';
var size = /*[[${size}]]*/ 0;
...
/*]]>*/
</script>
```

js 附加代码：

```
/*[+
var msg = 'This is a working application';
+]*/
```

js 移除代码：

```
/*[- */
var msg = 'This is a non-working template';
/* -]*/
```

### 6、内嵌变量

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

```
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

```
<footer th:fragment="copy"> 
&copy; 2019
</footer>
```

在页面任何地方引入：

```
<body>
    <div th:insert="layout/copyright :: copyright"></div>
    <div th:replace="layout/copyright :: copyright"></div>
</body>
```

th:insert 和 th:replace 区别，insert 只是加载，replace 是替换。Thymeleaf 3.0 推荐使用 th:insert 替换 2.0 的 th:replace。

返回的 HTML 如下：

```
<body> 
   <div> &copy; 2019 </div> 
  <footer>&copy; 2019 </footer> 
</body>
```

下面是一个常用的后台页面布局，将整个页面分为头部，尾部、菜单栏、隐藏栏，点击菜单只改变 content 区域的页面

```
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

任何页面想使用这样的布局值只需要替换中见的 content 模块即可

```
<html xmlns:th="http://www.thymeleaf.org" layout:decorator="layout">
 <body>
    <section layout:fragment="content">
  ...
```

也可以在引用模版的时候传参

```
<head th:include="layout :: htmlhead" th:with="title='Hello'"></head>
```

layout 是文件地址，如果有文件夹可以这样写`fileName/layout:htmlhead`，htmlhead 是指定义的代码片段 如`th:fragment="copy"`

# thymeleaf中th:text覆盖问题

​                              

在使用thymeleaf模板编写页面时发现父元素与子元素都用了th:text标签时，子元素的内容会被父元素覆盖。

```java
<h4 class="media-heading" th:text="${question.title}">
   <a th:href="@{'/question/'+${question.id}}" th:text="${question.title}"></a>
 </h4>
123
```

解决方法：
 1、删除父元素中的th:text
 2、

```java
<h4 class="media-heading" th:inline="text">[[${question.title}]]
    <a th:href="@{'/question/'+${question.id}}" th:text="${question.title}"></a>
 </h4>
```