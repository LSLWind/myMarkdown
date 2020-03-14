## Window对象

BOM 的核心对象是 window ，它表示浏览器的一个实例。在浏览器中， window 对象有双重角色,它既是通过 JavaScript 访问浏览器窗口的一个接口，又是 ECMAScript 规定的 Global 对象。这意味着在网页中定义的任何一个对象、变量和函数，都以 window 作为其 Global 对象，因此有权访问parseInt() 等方法。

### 全局作用域

由于 window 对象同时扮演着 ECMAScript中 Global 对象的角色，因此所有在全局作用域中声明的变量、函数都会变成 window 对象的属性和方法。

```js
var age = 29;
function sayAge(){
	alert(this.age);
}
alert(window.age); //29
sayAge(); //29
window.sayAge(); //29
```

我们在全局作用域中定义了一个变量 age 和一个函数 sayAge() ，它们被自动归在了 window 对象名下。于是，可以通过 window.age 访问变量 age ，可以通过 window.sayAge() 访问函数 sayAge() 。由于 sayAge() 存在于全局作用域中，因此 this.age 被映射到 window.age ，最终显示的仍然是正确的结果。

抛开全局变量会成为 window 对象的属性不谈，定义全局变量与在 window 对象上直接定义属性还是有一点差别：全局变量不能通过 delete 操作符删除，而直接在 window 对象上的定义的属性可以。

```js
var age = 29;
window.color = "red";
//在 IE < 9 时抛出错误，在其他所有浏览器中都返回 false
delete window.age;
//在 IE < 9 时抛出错误，在其他所有浏览器中都返回 true
delete window.color; //returns true
alert(window.age); //29
alert(window.color); //undefined
```

刚才使用 var 语句添加的 window 属性有一个名为 [[Configurable]] 的特性，这个特性的值被设置为 false ，因此这样定义的属性不可以通过 delete 操作符删除。尝试访问未声明的变量会抛出错误，但是通过查询 window 对象，可以知道某个可能未声明的变量是否存在。例如：

```js
//这里会抛出错误，因为 oldValue 未定义
var newValue = oldValue;
//这里不会抛出错误，因为这是一次属性查询
//newValue 的值是 undefined
var newValue = window.oldValue;
```

### 窗口关系及框架
如果页面中包含框架，则每个框架都拥有自己的 window 对象，并且保存在 frames 集合中。在 frames集合中，可以通过数值索引（从 0 开始，从左至右，从上到下）或者框架名称来访问相应的 window 对象。每个 window 对象都有一个 name 属性，其中包含框架的名称。

```html
<html>
    <head>
    	<title>Frameset Example</title>
    </head>
    <frameset rows="160,*">
    	<frame src="frame.htm" name="topFrame">
    <frameset cols="50%,50%">
   	 <frame src="anotherframe.htm" name="leftFrame">
   	 <frame src="yetanotherframe.htm" name="rightFrame">
    </frameset>
    </frameset>
</html>
```

以上代码创建了一个框架集，其中一个框架居上，两个框架居下。对这个例子而言，可以通过window.frames[0] 或者 window.frames["topFrame"] 来引用上方的框架。不过，恐怕你最好使用top 而非 window 来引用这些框架（例如，通过 top.frames[0] ）。我们知道， top 对象始终指向最高（最外）层的框架，也就是浏览器窗口。使用它可以确保在一个框架中正确地访问另一个框架。因为对于在一个框架中编写的任何代码来说，其中的 window 对象指向的都是那个框架的特定实例，而非最高层的框架。

### 导航和打开窗口

使用 window.open() 方法既可以导航到一个特定的 URL，也可以打开一个新的浏览器窗口。这个方法可以接收 4 个参数：要加载的 URL、窗口目标、一个特性字符串以及一个表示新页面是否取代浏览器历史记录中当前加载页面的布尔值。通常只须传递第一个参数，最后一个参数只在不打开新窗口的情况下使用。
如果为 window.open() 传递了第二个参数，而且该参数是已有窗口或框架的名称，那么就会在具有该名称的窗口或框架中加载第一个参数指定的 URL。

```js
//等同于< a href="http://www.wrox.com" target="topFrame"></a>
window.open("http://www.wrox.com/", "topFrame");
```

调用这行代码，就如同用户单击了 href 属性为 http://www.wrox.com/， target 属性为 "topFrame"的链接。如果有一个名叫 "topFrame" 的窗口或者框架，就会在该窗口或框架加载这个 URL；否则，就会创建一个新窗口并将其命名为 "topFrame" 。此外，第二个参数也可以是下列任何一个特殊的窗口名称： _self 、 _parent 、 _top 或 _blank 。

**弹出窗口**
如果给 window.open() 传递的第二个参数并不是一个已经存在的窗口或框架，那么该方法就会根据在第三个参数位置上传入的字符串创建一个新窗口或新标签页。如果没有传入第三个参数，那么就会打开一个带有全部默认设置（工具栏、地址栏和状态栏等）的新浏览器窗口（或者打开一个新标签页——根据浏览器设置）。在不打开新窗口的情况下，会忽略第三个参数。第三个参数是一个逗号分隔的设置字符串，表示在新窗口中都显示哪些特性。

* fullscreen： yes 或 no 表示浏览器窗口是否最大化。仅限IE
* height： 数值 表示新窗口的高度。不能小于100
* left ：数值 表示新窗口的左坐标。不能是负值 
* location： yes 或 no 表示是否在浏览器窗口中显示地址栏。不同浏览器的默认值不同。如果设置为no，地址栏可能会隐藏，也可能会被禁用（取决于浏览器） 
* menubar ：yes 或 no 表示是否在浏览器窗口中显示菜单栏。默认值为 no
* resizable ：yes 或 no 表示是否可以通过拖动浏览器窗口的边框改变其大小。默认值为 no
* scrollbars：yes 或 no 表示如果内容在视口中显示不下，是否允许滚动。默认值为 no
* status：yes 或 no 表示是否在浏览器窗口中显示状态栏。默认值为 no
* toolbar：yes 或 no 表示是否在浏览器窗口中显示工具栏。默认值为 no
* top：数值 表示新窗口的上坐标。不能是负值
* width：数值 表示新窗口的宽度。不能小于100

设置选项都可以通过逗号分隔的名值对列表来指定。其中，名值对以等号表示（注意，整个特性字符串中不允许出现空格）

```js
window.open("http://www.wrox.com/","wroxWindow",
"height=400,width=400,top=10,left=10,resizable=yes");
```

这行代码会打开一个新的可以调整大小的窗口，窗口初始大小为 400×400 像素，并且距屏幕上沿和左边各 10 像素。

### 间歇调用和超时调用

JavaScript 是单线程语言，但它允许通过设置超时值和间歇时间值来调度代码在特定的时刻执行。前者是在指定的时间过后执行代码，而后者则是每隔指定的时间就执行一次代码。

**超时调用**

超时调用需要使用 window 对象的 setTimeout() 方法，它接受两个参数：要执行的代码和以毫秒表示的时间（即在执行代码前需要等待多少毫秒）。其中，第一个参数可以是一个包含 JavaScript 代码的字符串（就和在 eval() 函数中使用的字符串一样），也可以是一个函数。例如，下面对 setTimeout()的两次调用都会在一秒钟后显示一个警告框。

```js
//不建议传递字符串！
setTimeout("alert('Hello world!') ", 1000);
//推荐的调用方式
setTimeout(function() {
	alert("Hello world!");
}, 1000);
```

虽然这两种调用方式都没有问题，但由于传递字符串可能导致性能损失，因此不建议以字符串作为第一个参数。第二个参数是一个表示等待多长时间的毫秒数，但经过该时间后指定的代码不一定会执行。

JavaScript 是一个单线程序的解释器，因此一定时间内只能执行一段代码。为了控制要执行的代码，就有一个 JavaScript 任务队列。这些任务会按照将它们添加到队列的顺序执行。 setTimeout() 的第二个参数告诉 JavaScript 再过多长时间把当前任务添加到队列中。如果队列是空的，那么添加的代码会立即执行；如果队列不是空的，那么它就要等前面的代码执行完了以后再执行。

调用 setTimeout() 之后，该方法会返回一个数值 ID，表示超时调用。这个超时调用 ID 是计划执行代码的唯一标识符，可以通过它来取消超时调用。要取消尚未执行的超时调用计划，可以调用clearTimeout() 方法并将相应的超时调用 ID 作为参数传递给它，如下所示。

```js
//设置超时调用
var timeoutId = setTimeout(function() {
	alert("Hello world!");
}, 1000);
//注意：把它取消
clearTimeout(timeoutId);
```

只要是在指定的时间尚未过去之前调用 clearTimeout() ，就可以完全取消超时调用。前面的代码在设置超时调用之后马上又调用了 clearTimeout() ，结果就跟什么也没有发生一样。
超时调用的代码都是在全局作用域中执行的，因此函数中 this 的值在非严格模式下指向 window 对象，在严格模式下是 undefined 。

**间歇调用**

间歇调用与超时调用类似，只不过它会按照指定的时间间隔重复执行代码，直至间歇调用被取消或者页面被卸载。设置间歇调用的方法是 setInterval() ，它接受的参数与 setTimeout() 相同：要执行的代码（字符串或函数）和每次执行之前需要等待的毫秒数。

```js
//不建议传递字符串！
setInterval ("alert('Hello world!') ", 10000);
//推荐的调用方式
setInterval (function() {
	alert("Hello world!");
}, 10000);
```

调用 setInterval() 方法同样也会返回一个间歇调用 ID，该 ID 可用于在将来某个时刻取消间歇调用。要取消尚未执行的间歇调用，可以使用 clearInterval() 方法并传入相应的间歇调用 ID。取消间歇调用的重要性要远远高于取消超时调用，因为在不加干涉的情况下，间歇调用将会一直执行到页面卸载。以下是一个常见的使用间歇调用的例子。

```js
var num = 0;
var max = 10;
var intervalId = null;
function incrementNumber() {
	num++;
	//如果执行次数达到了 max 设定的值，则取消后续尚未执行的调用
	if (num == max) {
		clearInterval(intervalId);
		alert("Done");
	}
}
intervalId = setInterval(incrementNumber, 500);
```

在这个例子中，变量 num 每半秒钟递增一次，当递增到最大值时就会取消先前设定的间歇调用。这个模式也可以使用超时调用来实现，如下所示。

```js
var num = 0;
var max = 10;
function incrementNumber() {
    num++;
    // 如果执行次数未达到 max 设定的值，则设置另一次超时调用
    if (num < max) {
   	 	setTimeout(incrementNumber, 500);
    } else {
   	 	alert("Done");
    }
}
setTimeout(incrementNumber, 500);
```

可见，在使用超时调用时，没有必要跟踪超时调用 ID，因为每次执行代码之后，如果不再设置另一次超时调用，调用就会自行停止。一般认为，使用超时调用来模拟间歇调用的是一种最佳模式。在开发环境下，很少使用真正的间歇调用，原因是后一个间歇调用可能会在前一个间歇调用结束之前启动。而像前面示例中那样使用超时调用，则完全可以避免这一点。所以，最好不要使用间歇调用。

### 系统对话框

浏览器通过 alert() 、 confirm() 和 prompt() 方法可以调用系统对话框向用户显示消息。系统对话框与在浏览器中显示的网页没有关系，也不包含 HTML。它们的外观由操作系统及（或）浏览器设置决定，而不是由 CSS 决定.。此外，通过这几个方法打开的对话框都是同步和模态的。也就是说，显示这些对话框的时候代码会停止执行，而关掉这些对话框后代码又会恢复执行。

具体来说，调用alert() 方法的结果就是向用户显示一个系统对话框，其中包含指定的文本和一个 OK（“确定”）按钮。

第二种对话框是调用 confirm() 方法生成的。从向用户显示消息的方面来看，这种“确认”对话框很像是一个“警告”对话框。但二者的主要区别在于“确认”对话框除了显示 OK 按钮外，还会显示一个 Cancel（“取消”）按钮，两个按钮可以让用户决定是否执行给定的操作，返回一个boolean值。

最后一种对话框是通过调用 prompt() 方法生成的，这是一个“提示”框，用于提示用户输入一些文本。提示框中除了显示 OK 和 Cancel 按钮之外，还会显示一个文本输入域，以供用户在其中输入内容。prompt() 方法接受两个参数：要显示给用户的文本提示和文本输入域的默认值（可以是一个空字符串），返回输入内容，没有则放回null。

## location 对象

location 是最有用的 BOM对象之一，它提供了与当前窗口中加载的文档有关的信息，还提供了一些导航功能。事实上， location 对象是很特别的一个对象，因为它既是 window 对象的属性，也是document 对象的属性；换句话说， window.location 和 document.location 引用的是同一个对象。location 对象的用处不只表现在它保存着当前文档的信息，还表现在它将 URL 解析为独立的片段，让开发人员可以通过不同的属性访问这些片段。

location的主要属性包括：

* hash ：返回URL中的hash（#号后跟零或多个字符），如果URL中不包含散列，则返回空字符串
* host ：返回服务器名称和端口号（如果有）
* hostname：返回不带端口号的服务器名称
* href ：返回当前加载页面的完整URL。而location对象的toString()方法也返回这个值
* pathname：返回URL中的目录和（或）文件名
* port ：返回URL中指定的端口号。如果URL中不包含端口号，则这个属性返回空字符串
* protocol：返回页面使用的协议。通常是http:或https:
* search：返回URL的查询字符串。这个字符串以问号开头

### 查询字符串参数

虽然通过上面的属性可以访问到 location 对象的大多数信息，但其中访问 URL 包含的查询字符串的属性并不方便。尽管 location.search 返回从问号到 URL 末尾的所有内容，但却没有办法逐个访问其中的每个查询字符串参数。为此，可以像下面这样创建一个函数，用以解析查询字符串，然后返回包含所有参数的一个对象：

```js
function getQueryStringArgs(){
	//取得查询字符串并去掉开头的问号
    var qs = (location.search.length > 0 ? location.search.substring(1) : ""),
    //保存数据的对象
    args = {},
    //取得每一项
    items = qs.length ? qs.split("&") : [],
    item = null,
    name = null,
    value = null,
    //在 for 循环中使用
    i = 0,
    len = items.length;
    //逐个将每一项添加到 args 对象中
    for (i=0; i < len; i++){
   		item = items[i].split("=");
    	name = decodeURIComponent(item[0]);
    	value = decodeURIComponent(item[1]);
    	if (name.length) {
    		args[name] = value;
    	}
    }
    return args;
}
```

### 位置操作

使用 location 对象可以通过很多方式来改变浏览器的位置。首先，也是最常用的方式，就是使用assign() 方法并为其传递一个 URL，如`location.assign("http://www.wrox.com");`
这样，就可以立即打开新 URL 并在浏览器的历史记录中生成一条记录。如果是将 location.href或 window.location 设置为一个 URL 值，也会以该值调用 assign() 方法。例如，下列两行代码与显式调用 assign() 方法的效果完全一样。

```js
window.location = "http://www.wrox.com";
location.href = "http://www.wrox.com";
```

在这些改变浏览器位置的方法中，最常用的是设置 location.href 属性。

## navigator 对象

最早由 Netscape Navigator 2.0引入的 navigator 对象，现在已经成为识别客户端浏览器的事实标准。虽然其他浏览器也通过其他方式提供了相同或相似的信息（例如，IE 中的 window.clientInfor-mation 和 Opera 中的 window.opera ），但 navigator 对象却是所有支持 JavaScript 的浏览器所共有的。与其他 BOM 对象的情况一样，每个浏览器中的 navigator 对象也都有一套自己的属性。下表列出了存在于所有浏览器中的属性和方法，以及支持它们的浏览器版本。

![image.png](https://i.loli.net/2020/03/11/F9OcxvpMI3WPVat.png)

### 检测插件

检测浏览器中是否安装了特定的插件是一种最常见的检测例程。对于非 IE 浏览器，可以使用plugins 数组来达到这个目的。该数组中的每一项都包含下列属性。

* name ：插件的名字。
*  description ：插件的描述。
*  filename ：插件的文件名。
*  length ：插件所处理的 MIME 类型数量。

一般来说， name 属性中会包含检测插件必需的所有信息，但有时候也不完全如此。在检测插件时，需要像下面这样循环迭代每个插件并将插件的 name 与给定的名字进行比较。

```js
//检测插件（在 IE 中无效）
function hasPlugin(name){
    name = name.toLowerCase();
    for (var i=0; i < navigator.plugins.length; i++){
    	if (navigator. plugins [i].name.toLowerCase().indexOf(name) > -1){
    		return true;
    	}
    }
    return false;
}
//检测 Flash
alert(hasPlugin("Flash"));
//检测 QuickTime
alert(hasPlugin("QuickTime"));
```

检测 IE 中的插件比较麻烦，因为 IE 不支持 Netscape 式的插件。在 IE 中检测插件的唯一方式就是使用专有的 ActiveXObject 类型，并尝试创建一个特定插件的实例。IE 是以 COM对象的方式实现插件的，而 COM对象使用唯一标识符来标识。因此，要想检查特定的插件，就必须知道其 COM 标识符。例如，Flash 的标识符是 ShockwaveFlash.ShockwaveFlash 。知道唯一标识符之后，就可以编写类似下面的函数来检测 IE 中是否安装相应插件了。

```js
//检测 IE 中的插件
function hasIEPlugin(name){
	try {
		new ActiveXObject(name);
		return true;
	} catch (ex){
		return false;
	}
}
//检测 Flash
alert(hasIEPlugin("ShockwaveFlash.ShockwaveFlash"));
//检测 QuickTime
alert(hasIEPlugin("QuickTime.QuickTime"));
```

在这个例子中，函数 hasIEPlugin() 只接收一个 COM 标识符作为参数。在函数内部，首先会尝试创建一个 COM对象的实例。之所以要在 try-catch 语句中进行实例化，是因为创建未知 COM对象会导致抛出错误。这样，如果实例化成功，则函数返回 true ；否则，如果抛出了错误，则执行 catch块，结果就会返回 false 。例子最后检测 IE 中是否安装了 Flash 和 QuickTime 插件。

鉴于检测这两种插件的方法差别太大，因此典型的做法是针对每个插件分别创建检测函数，而不是使用前面介绍的通用检测方法。来看下面的例子。

```js
//检测所有浏览器中的 Flash
function hasFlash(){
	var result = hasPlugin("Flash");
	if (!result){
		result = hasIEPlugin("ShockwaveFlash.ShockwaveFlash");
    }
}
```

## screen 对象

JavaScript 中有几个对象在编程中用处不大，而 screen 对象就是其中之一。 screen 对象基本上只用来表明客户端的能力，其中包括浏览器窗口外部的显示器的信息，如像素宽度和高度等。每个浏览器中的 screen 对象都包含着各不相同的属性



来源：**JavaScript高级程序设计（第三版）**