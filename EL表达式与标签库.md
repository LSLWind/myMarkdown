### 基础语法

el表达式可直接引用bean属性，最基本的即${表达式}，将整个文本作为字符流，只要找到${}就由el引擎进行解析替换。

el的目的是为了避免在jsp中使用java，提供一种创建JSP的工具。

el表达式不能添加到任何指令当中，如<% %\> <%= %\>，el表达式在渲染jsp稍后执行，el表达式最常用于添加到输出屏幕当中，即内嵌到文本上：

```elm
<a class="collapsible-header">${department.getName()}<i class="fa fa-angle-double-down"></i></a>
```

也可以直接写在html标签或者jsp标签当中：

```elm
<li><a href="/students/${department.getId()}/${major.getId()}">${major.getName()}</a></li>
```

#### 使用操作符

除了可以在el中引用bean属性，也可以使用数学运算符，如%,/,==,>等等，也可以使用&& || !

### 标签库

可直接在jsp中使用，相当于封装了的某些具体的语法

#### 迭代遍历\<c:forEach>

可迭代集合对象，用法为：

```java
<c:forEach items="${map1}" var="map">
    <h5>key: ${map.key} value：${map.value}</h5>
</c:forEach>
```

var:迭代时的具体的元素

items:要迭代的集合对象