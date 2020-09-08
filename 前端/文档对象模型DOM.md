## 节点层次

DOM 可以将任何 HTML 或 XML 文档描绘成一个由多层节点构成的结构。节点分为几种不同的类型，每种类型分别表示文档中不同的信息及（或）标记。每个节点都拥有各自的特点、数据和方法，另外也与其他节点存在某种关系。节点之间的关系构成了层次，而所有页面标记则表现为一个以特定节点为根节点的树形结构。以下面的 HTML 为例：

```html
<html>
	<head>
		<title>Sample Page</title>
	</head>
	<body>
		<p>Hello World!</p>
	</body>
</html>
```

可以将这个简单的 HTML 文档表示为一个层次结构。文档节点是每个文档的根节点。在这个例子中，文档节点只有一个子节点，即 <html\> 元素，我们称之为文档元素。文档元素是文档的最外层元素，文档中的其他所有元素都包含在文档元素中。每个文档只能有一个文档元素。在 HTML 页面中，文档元素始终都是 <html\> 元素。在 XML 中，没有预定义的元素，因此任何元素都可能成为文档元素。

每一段标记都可以通过树中的一个节点来表示：HTML 元素通过元素节点表示，特性（attribute）通过特性节点表示，文档类型通过文档类型节点表示，而注释则通过注释节点表示。总共有 12 种节点类型，这些类型都继承自一个基类型。

<img src="https://i.loli.net/2020/03/11/JMTVEo9RnyK8swH.png" alt="image.png" style="zoom: 50%;" />

### Node 类型

DOM1 级定义了一个 Node 接口，该接口将由 DOM 中的所有节点类型实现。这个 Node 接口在JavaScript 中是作为 Node 类型实现的；除了 IE 之外，在其他所有浏览器中都可以访问到这个类型。JavaScript 中的所有节点类型都继承自 Node 类型，因此所有节点类型都共享着相同的基本属性和方法。每个节点都有一个 nodeType 属性，用于表明节点的类型。节点类型由在 Node 类型中定义的下列
12 个数值常量来表示，任何节点类型必居其一：

* Node.ELEMENT_NODE (1)；
* Node.ATTRIBUTE_NODE (2)；
* Node.TEXT_NODE (3)；
* Node.CDATA_SECTION_NODE (4)；
* Node.ENTITY_REFERENCE_NODE (5)；
* Node.ENTITY_NODE (6)；
* Node.PROCESSING_INSTRUCTION_NODE (7)；
* Node.COMMENT_NODE (8)；
*  Node.DOCUMENT_NODE (9)；
* Node.DOCUMENT_TYPE_NODE (10)；
* Node.DOCUMENT_FRAGMENT_NODE (11)；
* Node.NOTATION_NODE (12)。

通过比较上面这些常量，可以很容易地确定节点的类型，例如：

```js
if (someNode.nodeType == Node.ELEMENT_NODE){ //在 IE 中无效
	alert("Node is an element.");
}
```

这个例子比较了 someNode.nodeType 与 Node.ELEMENT_NODE 常量。如果二者相等，则意味着someNode 确实是一个元素。然而，由于 IE 没有公开 Node 类型的构造函数，因此上面的代码在 IE 中会导致错误。为了确保跨浏览器兼容，最好还是将 nodeType 属性与数字值进行比较，如下所示：

```js
if (someNode.nodeType == 1){ // 适用于所有浏览器
	alert("Node is an element.");
}
```

并不是所有节点类型都受到 Web 浏览器的支持。开发人员最常用的就是元素和文本节点。

在 DOM 中，每种成分都是节点。元素节点没有文本值。

元素节点的文本存储在子节点中。该节点称为文本节点。

改变元素文本的方法，就是改变这个子节点（文本节点）的值。

### 节点关系

文档中所有的节点之间都存在这样或那样的关系。节点间的各种关系可以用传统的家族关系来描述，相当于把文档树比喻成家谱。在 HTML 中，可以将 <body\> 元素看成是 <html\> 元素的子元素；相应地，也就可以将 <html\> 元素看成是 <body\> 元素的父元素。而 <head\> 元素，则可以看成是 <body\> 元素的同胞元素，因为它们都是同一个父元素 <html\> 的直接子元素。

每个节点都有一个 childNodes 属性，其中保存着一个 NodeList 对象。 NodeList 是一种类数组对象，用于保存一组有序的节点，可以通过位置来访问这些节点。请注意，虽然可以通过方括号语法来访问 NodeList 的值，而且这个对象也有 length 属性，但它并不是 Array 的实例。 NodeList 对象的独特之处在于，它实际上是基于 DOM 结构动态执行查询的结果，因此 DOM 结构的变化能够自动反映在 NodeList 对象中。我们常说， NodeList 是有生命、有呼吸的对象，而不是在我们第一次访问它们的某个瞬间拍摄下来的一张快照。

```js
var firstChild = someNode.childNodes[0];
var secondChild = someNode.childNodes.item(1);
var count = someNode.childNodes.length;
```

无论使用方括号还是使用 item() 方法都没有问题，但使用方括号语法看起来与访问数组相似，因此颇受一些开发人员的青睐。另外，要注意 length 属性表示的是访问 NodeList 的那一刻，其中包含的节点数量。

每个节点都有一个 parentNode 属性，该属性指向文档树中的父节点。

包含在childNodes 列表中的每个节点相互之间都是同胞节点。通过使用列表中每个节点的 previousSibling和 nextSibling 属性，可以访问同一列表中的其他节点。列表中第一个节点的 previousSibling 属性值为 null ，而列表中最后一个节点的 nextSibling 属性的值同样也为 null 

父节点与其第一个和最后一个子节点之间也存在特殊关系。父节点的 firstChild 和 lastChild属性分别指向其 childNodes 列表中的第一个和最后一个节点。

<img src="https://i.loli.net/2020/03/11/NcoSt58jfkMW3gr.png" alt="image.png" style="zoom:50%;" />

### 操作节点

因为关系指针都是只读的，所以 DOM 提供了一些操作节点的方法。其中，最常用的方法是appendChild() ，用于向 childNodes 列表的末尾添加一个节点。添加节点后， childNodes 的新增节点、父节点及以前的最后一个子节点的关系指针都会相应地得到更新。更新完成后， appendChild()返回新增的节点。

```js
var returnedNode = someNode.appendChild(newNode);
alert(returnedNode == newNode); //true
alert(someNode.lastChild == newNode); //true
```

如果传入到 appendChild() 中的节点已经是文档的一部分了，那结果就是将该节点从原来的位置转移到新位置。即使可以将 DOM 树看成是由一系列指针连接起来的，但任何 DOM 节点也不能同时出现在文档中的多个位置上。因此，如果在调用 appendChild() 时传入了父节点的第一个子节点，那么该节点就会成为父节点的最后一个子节点

```js
//someNode 有多个子节点
var returnedNode = someNode.appendChild(someNode.firstChild);
alert(returnedNode == someNode.firstChild); //false
alert(returnedNode == someNode.lastChild); //true
```

有两个方法是所有类型的节点都有的。第一个就是 cloneNode() ，用于创建调用这个方法的节点的一个完全相同的副本。 cloneNode() 方法接受一个布尔值参数，表示是否执行深复制。在参数为 true的情况下，执行深复制。

我们要介绍的最后一个方法是 normalize() ，这个方法唯一的作用就是处理文档树中的文本节点。由于解析器的实现或 DOM 操作等原因，可能会出现文本节点不包含文本，或者接连出现两个文本节点的情况。当在某个节点上调用这个方法时，就会在该节点的后代节点中查找上述两种情况。如果找到了空文本节点，则删除它；如果找到相邻的文本节点，则将它们合并为一个文本节点。

### Document 类型
JavaScript 通过 Document 类型表示文档。在浏览器中， document 对象是 HTMLDocument （继承自 Document 类型）的一个实例，表示整个 HTML 页面。而且， document 对象是 window 对象的一个属性，因此可以将其作为全局对象来访问。 

## 常用方法

### 元素节点方法

**删除元素节点**

DOM 需要清楚您需要删除的元素，以及它的父元素。常用的解决方案：找到您希望删除的子元素，然后使用其 parentNode 属性来找到父元素：

```js
var child=document.getElementById("p1");
child.parentNode.removeChild(child);
```

**插入元素节点**

```js
node.parentNode.insertBefore(新添加节点，参照节点);//node.parentNode是节点的父节点
```

**更新元素节点内容**

直接变更html文本

```js
element.innerHTML='xxx';
```

## 常用操作

### 操控输入框

**获取输入框的值**

==document.getElementById("account").value==

```js
<h1>登录</h1>
<small>本网页展示了使用HTML DOM来访问HTML元素最常用的方法</small>
<hr/>
 
Account &nbsp;<input type="text" id="account" value="15515"><br/>
Password <input type="password" id="pwd" value="123456"><br/>
<input type="button" value="Show Values" onclick="getValue()">
<hr/>
<p id="show"></p>
 
<script>
    var a = document.getElementById("account").value;
    var p = document.getElementById("pwd").value;
    function getValue() {
        document.getElementById("show").innerHTML = "Account: " + a + "<br/>" + "Password: " + p;
    }
```



