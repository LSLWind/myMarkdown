## 基础

JavaScript是ECMAScript在浏览器上的扩展，ECMAScript定义了核心类，DOM类是针对HTML的扩展，BOM类是对浏览器的扩展。JavaScript广义上包含三个部分：ECMAScript（核心），DOM（文档对象模型），BOM（浏览器对象模型）。ECMAScript是JS语言的基础。

[<img src="https://s2.ax1x.com/2020/02/10/14DSx0.png" alt="14DSx0.png" style="zoom: 50%;" />](https://imgchr.com/i/14DSx0)

[<img src="https://s2.ax1x.com/2020/02/10/14sVjx.png" alt="14sVjx.png" style="zoom:67%;" />](https://imgchr.com/i/14sVjx)

**ECMAScript：**，由ECMA-262定义，提供核心语言功能。与浏览器、HTML文档没有依赖关系(唯一的联系是JS标准中规定，JS的全局对象Global object 一定是宿主对象window、fram)。ECMA-262定义的只是这门语言的基础。Web浏览器只是ECMAScrip实现的可能宿主。宿主环境不仅提供基本的ECMAScrip的实现，同时也会提供该语言的扩展，以便语言与环境之间的对接交互。而这些扩展---如DOM，则利用ECMAScrip的核心类型和语法提供更多更具体的功能，以便实现针对环境的操作。其它的宿主环境包括Node和Adobe Flash。

**DOM：**DOM提供访问和操作网页的方法和接口。

文档对象模型（DOM，Document Object Model）是针对XML但经过扩展用于HTML的应用程序编程接口。DOM把整个页面影射为一个多层节点结构。HTML或XML页面中的每个组成部分都是某种类型的节点，这些节点又包含着不同的数据。DOM已经成为JavaScript这门语言的一个重要组成部分。

DOM的级别由负责Web通信标准的W3C（World Wide Web Consortium 万维网联盟）制定，有DOM1级、DOM2级、DOM3级。Web浏览器对DOM的支持不同。

DOMAPI定义了XML或HTML中每种节点类型的声明，及其属性、方法和事件的声明；声明类之间的继承关系。DOMAPI只有类型及类型的属性、方法、事件的定义和描述，而没有具体实现，与平台无关。而DOM实现类是某浏览器对标准DOMAPI的实现，与平台有关。

[<img src="https://s2.ax1x.com/2020/02/10/14Dsij.md.png" alt="14Dsij.md.png" style="zoom:67%;" />](https://imgchr.com/i/14Dsij)

**BOM：**BOM(Browser Object Model 浏览器对象模型)提供访问与操作浏览器的方法和接口。从根本上讲，BOM只处理浏览器的窗口、标签窗口和框架（window、frame、iframe）；但人们习惯上也把所有针对浏览器的JavaScript在ECMAScript基础上的扩展算是BOM的一部分。开发人员使用BOM可以控制浏览器显示的页面以外的部分。

BOM真正的问题是它作为JavaScript的扩展实现的一部分但却没有相关的标准。这个问题在HTML5中得到了解决，HTMl5致力于把很多BOM功能写入正式规范。有了HTML5，BOM实现的细节有望朝着兼容性越来越高的方向发展。

[<img src="https://s2.ax1x.com/2020/02/10/14stDf.md.png" alt="14stDf.md.png" style="zoom:67%;" />](https://imgchr.com/i/14stDf)

JavaScript语言的解释器是纯解释器（没有编译），所以运行速度慢。但是由于JavaScript程序是在客户端运行，单线程、一般都比较短、而且重复率低，所以慢速的纯解释器还是可以接受的，解释器即解释一条语句即立刻执行一条语句（转换成机器指令执行），编译则先将整个程序编译成可直接执行的机器指令（汇编）然后再直接执行，速度更快。

**静态类型语言：**变量定义时有类型声明的语言。

1） 变量的类型在编译的时候确定

2） 变量的类型在运行时不能修改

这样编译器就可以确定运行时需要的内存总量。

**动态类型语言：**变量定义时无类型声明的语言。

1） 变量的类型在运行的时候确定

2） 变量的类型在运行可以修改

**强类型语言：**例如Java/C#语言是强类型语言，强类型定义语言是类型安全的语言，是由编译器以及编译器生成的中间代码来保证类型安全。

**弱类型语言：**C/C++/Javascript语言是弱类型语言，其类型安全由程序员来保证，Javascript语言的安全由程序员来保证。

## 基本概念

js使用Unicode字符集编码，大小写敏感，分号可选（尽量使用分号）

 JavaScript解释器有自己的内存管理机制，可以自动对内存进行垃圾回收（garbage collection）。 

### 严格模式

ECMAScript 5 引入了严格模式（strict mode）的概念。严格模式是为 JavaScript 定义了一种不同的解析与执行模型。在严格模式下，ECMAScript 3 中的一些不确定的行为将得到处理，而且对某些不安全的操作也会抛出错误。要在整个脚本中启用严格模式，可以在顶部添加如下代码：

```js
"use strict";
```

这行代码看起来像是字符串，而且也没有赋值给任何变量，但其实它是一个编译指示（pragma），用于告诉支持的 JavaScript 引擎切换到严格模式。这是为不破坏 ECMAScript 3 语法而特意选定的语法。

严格模式下，JavaScript 的执行结果会有很大不同。支持严格模式的浏览器包括 IE10+、Firefox 4+、Safari 5.1+、Opera 12+和 Chrome

### 基础数据类型

ECMAScript 中有 5 种简单数据类型（也称为基本数据类型）： Undefined 、 Null 、 Boolean 、 Number和 String 。还有 1种复杂数据类型—— Object ， Object 本质上是由一组无序的名值对组成的。ECMAScript不支持任何创建自定义类型的机制，而所有值最终都将是上述 6 种数据类型之一。

**typeof 操作符**

对一个值使用 typeof 操作符可能返回下列某个字符串：

* "undefined" ——如果这个值未定义；
*  "boolean" ——如果这个值是布尔值；
* "string" ——如果这个值是字符串；
*  "number" ——如果这个值是数值；
* "object" ——如果这个值是对象或 null ；
*  "function" ——如果这个值是函数。

调用 typeof null会返回 "object" ，因为特殊值 null 被认为是一个空的对象引用。Safari 5 及之前版本、Chrome 7 及之前版本在对正则表达式调用 typeof 操作符时会返回 "function" ，而其他浏览器在这种情况下会返回"object" 

**undefined**

Undefined 类型只有一个值，即特殊的 undefined 。在使用 var 声明变量但未对其加以初始化时，这个变量的值就是 undefined，相当于为变量赋予的初值。

令人困惑的是：对未初始化的变量执行 typeof 操作符会返回 undefined 值，而对未声明的变量执行 typeof 操作符同样也会返回 undefined 值。

**null**

null 值表示一个空对象指针，而这也正是使用 typeof 操作符检测 null 值时会返回 "object" 的原因

```js
var car = null;
alert(typeof car); // "object"
```

如果定义的变量准备在将来用于保存对象，那么最好将该变量初始化为 null 而不是其他值。

**Number**

所有数字都是64位浮点型数字，三个特殊数字Infinity,-Infinity与NaN分别表示正负无穷大与非数字特殊值，内置类Number提供一些可选常量，如Number.MAX_VALUE、Number.MIN_VALUE

NaN 本身有两个非同寻常的特点。首先，任何涉及 NaN 的操作（例如 NaN /10）都会返回 NaN ，这个特点在多步计算中有可能导致问题。其次， NaN 与任何值都不相等，包括 NaN 本身。

```js
alert(NaN == NaN); //false
```

针对 NaN 的这两个特点，ECMAScript 定义了 isNaN() 函数。这个函数接受一个参数，该参数可以是任何类型，而函数会帮我们确定这个参数是否“不是数值”。 isNaN() 在接收到一个值之后，会尝试将这个值转换为数值。某些不是数值的值会直接转换为数值，例如字符串 "10" 或 Boolean 值。而任何能被转换为数值的值都会导致这个函数返回 true 。

```js
alert(isNaN(NaN)); //true
alert(isNaN(10)); //false（10 是一个数值）
alert(isNaN("10")); //false（可以被转换成数值 10）
alert(isNaN("blue")); //true（不能转换成数值）
alert(isNaN(true)); //false（可以被转换成数值 1）
```

 parseFloat() 与 parseInt()函数用于将字符串转换为数值

**String**

字符串不可变，String 数据类型包含一些特殊的字符字面量，也叫转义序列

* \n 换行
* \t 制表
* \b 空格
* \r 回车
* \f 进纸
* \\ \斜杠
* \\' 单引号（ ' ），在用单引号表示的字符串中使用。例如： 'He said, \'hey.\''
* \\" 双引号（ " ），在用双引号表示的字符串中使用。例如： "He said, \"hey.\""
* \xnn 以十六进制代码 nn 表示的一个字符（其中 n 为0～F）。例如， \x41 表示 "A"
* \unnnn 以十六进制代码 nnnn 表示的一个Unicode字符（其中 n 为0～F）。例如， \u03a3 表示希腊字符Σ

任何字符串的长度都可以通过访问其 length 属性取得

```js
alert(text.length); // 输出 28
```

### 变量

直接声明的变量是全局变量

 var 操作符定义的变量将成为定义该变量的作用域中的局部变量。也就是说，如果在函数中使用 var 定义一个变量，那么这个变量在函数退出后就会被销毁，例如：

```js
function test(){
var message = "hi"; // 局部变量
}
test();
alert(message); // 错误！
```

### 执行环境与作用域

执行环境（execution context，为简单起见，有时也称为“环境”）是 JavaScript 中最为重要的一个概念。执行环境定义了变量或函数有权访问的其他数据，决定了它们各自的行为。每个执行环境都有一个与之关联的变量对象（variable object），环境中定义的所有变量和函数都保存在这个对象中。

全局执行环境是最外围的一个执行环境。根据 ECMAScript 实现所在的宿主环境不同，表示执行环境的对象也不一样。在 Web 浏览器中，全局执行环境被认为是 window 对象，因此所有全局变量和函数都是作为 window 对象的属性和方法创建的。某个执行环境中的所有代码执行完毕后，该环境被销毁，保存在其中的所有变量和函数定义也随之销毁（全局执行环境直到应用程序退出——例如关闭网页或浏览器——时才会被销毁）。

每个函数都有自己的执行环境。当执行流进入一个函数时，函数的环境就会被推入一个环境栈中。而在函数执行之后，栈将其环境弹出，把控制权返回给之前的执行环境。ECMAScript 程序中的执行流正是由这个方便的机制控制着。
当代码在一个环境中执行时，会创建变量对象的一个作用域链（scope chain）。作用域链的用途，是保证对执行环境有权访问的所有变量和函数的有序访问。作用域链的前端，始终都是当前执行的代码所在环境的变量对象。如果这个环境是函数，则将其活动对象（activation object）作为变量对象。活动对象在最开始时只包含一个变量，即 arguments 对象（这个对象在全局环境中是不存在的）。作用域链中的下一个变量对象来自包含（外部）环境，而再下一个变量对象则来自下一个包含环境。这样，一直延续到全局执行环境；全局执行环境的变量对象始终都是作用域链中的最后一个对象。

标识符解析是沿着作用域链一级一级地搜索标识符的过程。搜索过程始终从作用域链的前端开始，然后逐级地向后回溯，直至找到标识符为止（如果找不到标识符，通常会导致错误发生）。

```js
var color = "blue";
function changeColor(){
	if (color === "blue"){
		color = "red";
	} else {
		color = "blue";
	}
}
changeColor();
alert("Color is now " + color);
```

在这个简单的例子中，函数 changeColor() 的作用域链包含两个对象：它自己的变量对象（其中定义着 arguments 对象）和全局环境的变量对象。可以在函数内部访问变量 color ，就是因为可以在这个作用域链中找到它。

#### try-catch与with语句

这两个语句都会在作用域链的前端添加一个变量对象。对 with 语句来说，会将指定的对象添加到作用域链中。对 catch 语句来说，会创建一个新的变量对象，其中包含的是被抛出的错误对象的声明。

```js
function buildUrl() {
	var qs = "?debug=true";
	with(location){
		var url = href + qs;
	}
	return url;
}
```

在此， with 语句接收的是 location 对象，因此其变量对象中就包含了 location 对象的所有属性和方法，而这个变量对象被添加到了作用域链的前端。 buildUrl() 函数中定义了一个变量 qs 。当在with 语句中引用变量 href 时（实际引用的是 location.href ），可以在当前执行环境的变量对象中找到。当引用变量 qs 时，引用的则是在 buildUrl() 中定义的那个变量，而该变量位于函数环境的变量对象中。至于 with 语句内部，则定义了一个名为 url 的变量，因而 url 就成了函数执行环境的一部分，所以可以作为函数的值被返回。



### 对象Object

#### 创建对象

 可以通过对象直接量、关键字new和（ECMAScript 5中的）Object.create()函数来创建对象。 

**对象直接量**就是直接写一个js对象，如

```js
var empty={};//没有任何属性的对象
var point={x:0,y:0};//两个属性
var point2={x:point.x,y:point.y+1};//更复杂的值
```

 ECMAScript 5中，对象直接量中的最后一个属性后的逗号将忽略，且在ECMAScript 3的大部分实现中也可以忽略这个逗号，但在IE中则报错。

 **new运算符**创建并初始化一个新对象。关键字new后跟随一个函数调用。这里的函数称做构造函数（constructor），构造函数用以初始化一个新创建的对象。JavaScript语言核心中的原始类型都包含内置构造函数。如`var o=new Object();//创建一个空对象，和{}一样`

ECMAScript 5定义了一个名为**Object.create()**的方法，它创建一个新对象，其中第一个参数是这个对象的原型。Object.create()提供第二个可选参数，用以对对象的属性进行进一步描述。可以通过任意原型创建新对象 

```js
var o1=Object.create({x:1,y:2});//o1继承了属性x和y
```

#### 对象属性与方法

Object 的每个实例都具有下列属性和方法。

* constructor ：保存着用于创建当前对象的函数。
*  hasOwnProperty(propertyName) ：用于检查给定的属性在当前对象实例中（而不是在实例的原型中）是否存在。其中，作为参数的属性名（ propertyName ）必须以字符串形式指定（例如：o.hasOwnProperty("name") ）。
*  isPrototypeOf(object) ：用于检查传入的对象是否是传入对象的原型。
*  propertyIsEnumerable(propertyName) ：用于检查给定的属性是否能够使用 for-in 语句来枚举。与 hasOwnProperty() 方法一样，作为参数的属性名必须以字符串形式指定。
*  toLocaleString() ：返回对象的字符串表示，该字符串与执行环境的地区对应。
* toString() ：返回对象的字符串表示。
* valueOf() ：返回对象的字符串、数值或布尔值表示。通常与 toString() 方法的返回值
  相同。

由于在 ECMAScript 中 Object 是所有对象的基础，因此所有对象都具有这些基本的属性和方法。

**对象属性访问**

对象的属性有两种访问方式

```js
object.property
object["property"]
```

区别在于[]访问方式是通过字符串进行访问，更加灵活，下标可以动态变化，如

```js
var addr="";
for(i=0;i＜4;i++){
	addr+=customer["address"+i]+'\n';
}
```

 这段代码读取customer对象的address0、address1、address2和address3属性，并将它们连接起来。 

**继承**

假设要查询对象o的属性x，如果o中不存在x，那么将会继续在o的原型对象中查询属性x。如果原型对象中也没有x，但这个原型对象也有原型，那么继续在这个原型对象的原型上执行查询，直到找到x或者查找到一个原型是null的对象为止。可以看到，对象的原型属性构成了一个“链”，通过这个“链”可以实现属性的继承。

**属性访问错误**

 查询一个不存在的属性并不会报错，如果在对象o自身的属性或继承的属性中均未找到属性x，属性访问表达式o.x返回undefined。 但对象不存在则会报错，因为undefined并没有属性

**属性检测**

 JavaScript对象可以看做属性的集合，判断某个属性是否存在于某个对象中可以通过in运算符、hasOwnPreperty()和propertyIsEnumerable()方法来完成这个工作。

in运算符的左侧是属性名（字符串），右侧是对象。如果对象的自有属性或继承属性中包含这个属性则返回true：

```js
var o={x:1}
"x"in o;//true："x"是o的属性
"y"in o;//false："y"不是o的属性
"toString"in o;//true：o继承toString属性
```

 另一种更简便的方法是使用“!==”判断一个属性是否是undefined，如` o.x!==undefined;//true:o中有属性x `

(Object)对象的hasOwnProperty()方法用来检测给定的名字是否是对象的自有属性。对于继承属性它将返回false,如` o.hasOwnProperty("x");//true：o有一个自有属性x `， propertyIsEnumerable()是hasOwnProperty()的增强版，只有检测到是自有属性且这个属性的可枚举性为true时它才返回true。 

  **属性getter和setter**

在ECMAScript 5中，属性值可以用一个或两个方法替代，这两个方法就是getter和setter，使用关键字get与set 进行声明，方法名就是属性名，这样的属性称之为存储器属性，如：

```js
var o={
	get accessor_prop(){/*这里是函数体*/},
	set accessor_prop(value){/*这里是函数体*/}
};
```

```js
var random={
	get octet(){return Math.floor(Math.random()*256);},
	get int16(){return Math.floor(Math.random()*65536)-32768;}
};
```

**序列化对象**

 对象序列化（serialization）是指将对象的状态转换为字符串，也可将字符串还原为对象。ECMAScript 5提供了内置函数JSON.stringify()和JSON.parse()用来序列化和还原JavaScript对象。这些方法都使用JSON作为数据交换格式 

### 对象属性

 每一个对象都有与之相关的原型（prototype）、类（class）和可扩展性（extensible attribute） 

**原型prototype**

每一个JavaScript对象（null除外）都和另一个对象相关联。“另一个”对象就是我们熟知的原型，每一个对象都从原型继承属性。

所有通过对象直接量创建的对象都具有同一个原型对象，并可以通过`对象.prototype`获得对原型对象的引用。

没有原型的对象为数不多，Object.prototype就是其中之一。它不继承任何属性。其他原型对象都是普通对象，普通对象都具有原型。例如，Date.prototype的属性继承自Object.prototype，因此由new Date()创建的Date对象的属性同时继承自 Date.prototype和Object.prototype。这一系列链接的原型对象就是所谓的“原型链”（prototype chain）。(原型的引入为js提供了继承特性) 

 通过对象直接量创建的对象使用Object.prototype作为它们的原型。通过new创建的对象使用构造函数的prototype属性作为它们的原型。，如通过new Date()创建的对象的原型就是Date.prototype。通过Object.create()创建的对象使用第一个参数（也可以是null）作为它们的原型。 

要想检测一个对象是否是另一个对象的原型（或处于原型链中），请使用isPrototypeOf()方法。例如，可以通过p.isPrototypeOf(o)来检测p是否是o的原型：

```js
var p={x:1};//定义一个原型对象
var o=Object.create(p);//使用这个原型创建一个对象
p.isPrototypeOf(o)//=＞true:o继承自p
Object.prototype.isPrototypeOf(o)//=＞true:p继承自Object.prototype
```

### 对象方法

 所有的JavaScript对象都从Object.prototype继承属性（除了那些不通过原型显式创建的对象）。这些继承属性主要是方法。常用的定义在Object.prototype里的对象的主要方法有：

* toString()方法：toString()方法没有参数，它将返回一个表示调用这个方法的对象值的字符串。

```js
var s={x:1,y:1}.toString();
```

默认的toString()方法返回其类型，上述结果为 "[object Object]" 

* toLocaleString()方法：除了基本的toString()方法之外，对象都包含toLocaleString()方法，这个方法返回一个表示这个对象的本地化字符串。Object中默认的toLocaleString()方法并不做任何本地化自身的操作，它仅调用toString()方法并返回对应值。Date和Number类对toLocaleString()方法做了定制，可以用它对数字、日期和时间做本地化的转换。
*  toJSON()方法 ： Object.prototype实际上没有定义toJSON()方法，但对于需要执行序列化的对象来说，JSON.stringify()方法会调用toJSON()方法。如果在待序列化的对象中存在这个方法，则调用它，返回值即是序列化的结果，而不是原始的对象。 
* valueOf()方法：valueOf()方法和toString()方法非常类似，但往往当JavaScript需要将对象转换为某种原始值而非字符串的时候才会调用它，尤其是转换为数字的时候。

### 数组

数组是值的有序集合。每个值叫做一个元素，而每个元素在数组中有一个位置，以数字表示，称为索引。JavaScript数组是无类型的：数组元素可以是任意类型，并且同一个数组中的不同元素也可能有不同的类型。数组的元素甚至也可能是对象或其他数组，这允许创建复杂的数据结构，如对象的数组和数组的数组。

 JavaScript数组是动态的：根据需要它们会增长或缩减，并且在创建数组时无须声明一个固定的大小或者在数组大小变化时无须重新分配空间。  

 数组继承自Array.prototype中的属性，它定义了一套丰富的数组操作方法 

**创建数组**

 使用数组直接量是创建数组最简单的方法，在方括号中将数组元素用逗号隔开即可。 

```js
var empty=[];//没有元素的数组
var primes=[2,3,5,7,11];//有5个数值的数组
var misc=[1.1,true,"a",];//3个不同类型的元素和结尾的逗号
```

 调用构造函数Array()是创建数组的另一种方法。可以用三种方式调用构造函数。 

```js
var a=new Array();//该方法创建一个没有任何元素的空数组，等同于数组直接量[]。 
var a=new Array(10);//调用时有一个数值参数，它指定长度
var a=new Array(5,4,3,2,1,"testing,testing");//显式指定元素
```

**数组元素的读和写** 

```js
var a=["world"];//从一个元素的数组开始
var value=a[0];//读第0个元素
a[1]=3.14;//写第1个元素
```

 数组是对象的特殊形式。使用方括号访问数组元素就像用方括号访问对象的属性一样。JavaScript将指定的数字索引值转换成字符串——索引值1变成“1”——然后将其作为属性名来使用。 数组索引仅仅是对象属性名的一种特殊类型，这意味着JavaScript数组没有“越界”错误的概念。当试图查询任何对象中不存在的属性时，不会报错，只会得到undefined值。类似于对象，对于对象同样存在这种情况。  

也可以使用push()方法在数组末尾增加一个或多个元素：

```js
a=[];//开始是一个空数组
a.push("zero")//在末尾添加一个元素。a=["zero"]
```

 数组有pop()方法（它和push()一起使用），后者一次使减少长度1并返回被删除元素的值。还有一个shift()方法（它和unshift()一起使用），从数组头部删除一个元素。 

**多维数组**

JavaScript不支持真正的多维数组，但可以用数组的数组来近似。访问数组的数组中的元素，只要简单地使用两次[]操作符即可。

```js
//创建一个多维数组
var table=new Array(10);//表格有10行
for(var i=0;i＜table.length;i++)
table[i]=new Array(10);//每行有10列
//初始化数组
for(var row=0;row＜table.length;row++){
	for(col=0;col＜table[row].length;col++){
		table[row][col]=row*col;
	}
}
//使用多维数组来计算（查询）5*7
var product=table[5][7];//35
```

**数组常用方法**

* join()：Array.join()方法将数组中所有元素都转化为字符串并连接在一起，返回最后生成的字符串。可以指定一个可选的字符串在生成的字符串中来分隔数组的各个元素。如果不指定分隔符，默认使用逗号 

```js
var a=[1,2,3];//创建一个包含三个元素的数组
a.join();//=＞"1,2,3"
a.join("");//=＞"1 2 3"
```

* reverse()： Array.reverse()方法将数组中的元素颠倒顺序，返回逆序的数组。 

```js
var a=[1,2,3];
a.reverse().join()//=＞"3,2,1",并且现在的a是[3,2,1]
```

* sort()：Array.sort()方法将数组中的元素排序并返回排序后的数组。当不带参数调用sort()时，数组元素以字母表顺序排序。如果数组包含undefined元素，它们会被排到数组的尾部。为了按照其他方式而非字母表顺序进行数组排序，必须给sort()方法传递一个比较函数。该函数决定了它的两个参数在排好序的数组中的先后顺序。a<b则返回<0的值

```js
var a=[33,4,1111,222];
a.sort();//字母表顺序:1111,222,33,4
a.sort(function(a,b){//数值顺序:4,33,222,1111
	return a-b;//根据顺序，返回负数、0、正数
});
```

* concat()：Array.concat()方法创建并返回一个新数组，它的元素包括调用concat()的原始数组的元素和concat()的每个参数。

```js
var a=[1,2,3];
a.concat(4,5)//返回[1,2,3,4,5]
a.concat([4,5]);//返回[1,2,3,4,5]
a.concat([4,5],[6,7])//返回[1,2,3,4,5,6,7]
a.concat(4,[5,[6,7]])//返回[1,2,3,4,5,[6,7]]
```

*  slice() ： Array.slice()方法返回指定数组的一个片段或子数组。它的两个参数分别指定了片段的开始和结束的位置。返回的数组包含第一个参数指定的位置和所有到但不含第二个参数指定的位置之间的所有数组元素。如果只指定一个参数，返回的数组将包含从开始位置到数组结尾的所有元素。如参数中出现负数，它表示相对于数组中最后一个元素的位置。例如，参数-1指定了最后一个元素，而-3指定了倒数第三个元素。 

```js
var a=[1,2,3,4,5];
a.slice(0,3);//返回[1,2,3]
a.slice(3);//返回[4,5]
a.slice(1,-1);//返回[2,3,4]
a.slice(-3,-2);//返回[3]
```

* push()和pop()：push()和pop()方法允许将数组当做栈来使用。

* nshift()和shift()：unshift()和shift()方法的行为非常类似于push()和pop()，不一样的是前者是在数组的头部而非尾部进行元素的插入和删除操作。

**ECMAScript 5中的数组方法 **

 ECMAScript 5定义了9个新的数组方法来遍历、映射、过滤、检测、简化和搜索数组。ECMAScript 5中的数组方法大多数将函数作为参数，该函数可传递三个参数，分别代表数组元素、元素的索引和数组本身。 

*  forEach()：forEach()方法从头至尾遍历数组，为每个元素调用指定的函数。如上所述，传递的函数作为forEach()的第一个参数。然后forEach()使用三个参数调用该函数：数组元素、元素的索引和数组本身。 

```js
var data=[1,2,3,4,5];//要求和的数组
//计算数组元素的和值
var sum=0;//初始为0
data.forEach(function(value){sum+=value;});//将每个值累加到sum上
sum//=＞15//每个数组元素的值自加1
data.forEach(function(v,i,a){a[i]=v+1;});
data//=＞[2,3,4,5,6]
```

* map()：map()方法将调用的数组的每个元素传递给指定的函数，并返回一个数组，它包含该函数的返回值。

```js
a=[1,2,3];
b=a.map(function(x){return x*x;});//b是[1,4,9]
```

 map()返回的是新数组：它不修改调用的数组。 

* filter()：fliter()方法返回的数组元素是调用的数组的一个子集。传递的函数是用来逻辑判定的：该函数返回true或false，用于过滤元素

```js
a=[5,4,3,2,1];
smallvalues=a.filter(function(x){return x＜3});//[2,1]
everyother=a.filter(function(x,i){return i%2==0});//[5,3,1]
```

* every()和some()：every()和some()方法是数组的逻辑判定：它们对数组元素应用指定的函数进行判定，返回true或false。

every()方法就像数学中的“针对所有”的量词：当且仅当针对数组中的所有元素调用判定函数都返回true，它才返回true：

```js
a=[1,2,3,4,5];
a.every(function(x){return x＜10;})//=＞true:所有的值＜10
a.every(function(x){return x%2===0;})//=＞false:不是所有的值都是偶数
```

some()方法就像数学中的“存在”的量词：当数组中至少有一个元素调用判定函数返回true，它就返回true；并且当且仅当数值中的所有元素调用判定函数都返回false，它才返回false：

```js
a=[1,2,3,4,5];
a.some(function(x){return x%2===0;})//=＞true：a含有偶数值
a.some(isNaN)//=＞false：a不包含非数值元素
```

一旦every()和some()确认该返回什么值它们就会停止遍历数组元素。

* reduce()和reduceRight()：reduce()和reduceRight()方法使用指定的函数将数组元素进行组合，生成单个值。这在函数式编程中是常见的操作，也可以称为“注入”和“折叠”。举例说明它是如何工作的：

```js
var a=[1,2,3,4,5]
var sum=a.reduce(function(x,y){return x+y},0);//数组求和
var product=a.reduce(function(x,y){return x*y},1);//数组求积
var max=a.reduce(function(x,y){return(x＞y)?x:y;});//求最大值
```

reduce()需要两个参数。第一个是执行化简操作的函数。化简函数的任务就是用某种方法把两个值组合或化简为一个值，并返回化简后的值。在上述例子中，函数通过加法、乘法或取最大值的方法组合两个值。第二个（可选）的参数是一个传递给函数的初始值。

reduceRight()的工作原理和reduce()一样，不同的是它按照数组索引从高到低（从右到左）处理数组，而不是从低到高。如果化简操作的优先顺序是从右到左，你可能想使用它，例如：

```js
var a=[2,3,4]//计算2^(3^4)。乘方操作的优先顺序是从右到左
var big=a.reduceRight(function(accumulator,value){
return Math.pow(value,accumulator);
});
```

* indexOf()和lastIndexOf()：indexOf()和lastIndexOf()搜索整个数组中具有给定值的元素，返回找到的第一个元素的索引或者如果没有找到就返回-1。indexOf()从头至尾搜索，而lastIndexOf()则反向搜索。

```js
a=[0,1,2,1,0];
a.indexOf(1)//=＞1:a[1]是1
a.lastIndexOf(1)//=＞3:a[3]是1
a.indexOf(3)//=＞-1:没有值为3的元素
```

**判断数组**

在ECMAScript 5中，可以使用Array.isArray()函数来做这件事情：

```js
Array.isArray([])//=＞true
Array.isArray({})//=＞false
```

## 函数

 函数使用function关键字来定义。如果return语句没有一个与之相关的表达式，则它返回undefined值。如果一个函数不包含return语句，那它就只执行函数体中的每条语句，并返回undefined值给调用者。 

```js
//输出o的每个属性的名称和值，返回undefined
function printprops(o){
for(var p in o)
	console.log(p+":"+o[p]+"\n");
}
```

 如果函数或者方法调用之前带有关键字new，它就构成构造函数调用，也就是说，构造函数是与对象同名的函数 

**可选形参**

当调用函数的时候传入的实参比函数声明时指定的形参个数要少，剩下的形参都将设置为undefined值。因此在调用函数时形参是否可选以及是否可以省略应当保持较好的适应性。

```js
//将对象o中可枚举的属性名追加至数组a中，并返回这个数组a
//如果省略a，则创建一个新数组并返回这个新数组
function getPropertyNames(o,/*optional*/a){
	if(a===undefined)a=[];//如果未定义，则使用新数组
	for(var property in o)a.push(property);
	return a;
}
//这个函数调用可以传入1个或2个实参
var a=getPropertyNames(o);//将o的属性存储到一个新数组中
getPropertyNames(p,a);//将p的属性追加至数组a中
```

如果在第一行代码中不使用if语句，可以使用“||”运算符，这是一种习惯用法：

```js
a=a||[];
```

**可变长的实参列表：实参对象**

当调用函数的时候传入的实参个数超过函数定义时的形参个数时，没有办法直接获得未命名值的引用。参数对象解决了这个问题。在函数体内，标识符**arguments**是指向实参对象的引用，实参对象是一个类数组对象，这样可以通过数字下标就能访问传入函数的实参值，而不用非要通过名字来得到实参。

### 闭包

 JavaScript也采用词法作用域（lexical scoping），也就是说，函数的执行依赖于变量作用域，这个作用域是在函数定义时决定的，而不是函数调用时决定的。为了实现这种词法作用域，JavaScript函数对象的内部状态不仅包含函数的代码逻辑，还必须引用当前的作用域链。函数对象可以通过作用域链相互关联起来，函数体内部的变量都可以保存在函数作用域内，这种特性在计算机科学文献中称为“闭包” 。

 从技术的角度讲，所有的JavaScript函数都是闭包：它们都是对象，它们都关联到作用域链。 

```js
var scope="global scope";//全局变量
function checkscope(){
	var scope="local scope";//局部变量
	function f(){return scope;}//在作用域中返回这个值
	return f();
}
checkscope()//=＞"local scope"
```

这段代码很清楚，返回函数内局部变量，稍微改动一下

```js
var scope="global scope";//全局变量
function checkscope(){
    var scope="local scope";//局部变量
    function f(){return scope;}//在作用域中返回这个值
    return f;
}
checkscope()()//返回值是什么?
```

 在这段代码中，我们将函数内的一对圆括号移动到了checkscope()之后。checkscope()现在仅仅返回函数内嵌套的一个函数对象，而不是直接返回结果 

 JavaScript函数的执行用到了作用域链，这个作用域链是函数定义的时候创建的。嵌套的函数f()定义在这个作用域链里，其中的变量scope一定是局部变量，不管在何时何地执行函数f()，这种绑定在执行f()时依然有效。因此最后一行代码返回"local scope"，而不是"global scope"。简言之，闭包的这个特性强大到让人吃惊：它们可以捕捉到局部变量（和参数），并一直保存下来，看起来像这些变量绑定到了在其中定义它们的外部函数。 

也就是说，利用闭包特性，在函数内部使用嵌套函数，可以将函数内部的局部变量保存在函数内，调用时并不会消失，就相当于一个静态变量，但是这个变量只与其作用域有关，并不是全局唯一的静态变量

```js
var uniqueInteger=(function(){//定义函数并立即调用
    var counter=0;//函数的私有状态
    return function(){return counter++;};
}());
```

counter就相当于函数的静态变量，并且每使用uniqueInteger一次就会对counter+1，相当于这个变量实现了唯一自增。

```js
function counter(){
    var n=0;
    return{
    	count:function(){return n++;},
    	reset:function(){n=0;}
    };
}

var c=counter(),d=counter();//创建两个计数器
c.count()//=＞0
d.count()//=＞0:它们互不干扰

c.reset()//reset()和count()方法共享状态
c.count()//=＞0:因为我们重置了c
d.count()//=＞1:而没有重置d
```

 counter()函数返回了一个“计数器”对象，这个对象包含两个方法：count()返回下一个整数，reset()将计数器重置为内部状态。首先要理解，这两个方法都可以访问私有变量n。再者，**每次调用counter()都会创建一个新的作用域链和一个新的私有变量。因此，如果调用counter()两次，则会得到两个计数器对象，而且彼此包含不同的私有变量，调用其中一个计数器对象的count()或reset()不会影响到另外一个对象。**

也可以使用getter与setter

```js
unction counter(n){//函数参数n是一个私有变量
    return{//属性getter方法返回并给私有计数器var递增1
        get count(){return n++;},//属性setter不允许n递减
        set count(m){
            if(m＞=n)n=m;
            else throw Error("count can only be set to a larger value");
        }
    };
}
var c=counter(1000);
c.count//=＞1000
c.count//=＞1001
c.count=2000
c.count//=＞2000
c.count=2000//=＞Error!
```

```js
//这个函数给对象o增加了属性存取器方法
//方法名称为get＜name＞和set＜name＞。如果提供了一个判定函数
//setter方法就会用它来检测参数的合法性，然后在存储它
//如果判定函数返回false，setter方法抛出一个异常
//
//这个函数有一个非同寻常之处，就是getter和setter函数
//所操作的属性值并没有存储在对象o中
//相反，这个值仅仅是保存在函数中的局部变量中
//getter和setter方法同样是局部函数，因此可以访问这个局部变量
//也就是说，对于两个存取器方法来说这个变量是私有的
//没有办法绕过存取器方法来设置或修改这个值
function addPrivateProperty(o,name,predicate){
    var value;//这是一个属性值
    //getter方法简单地将其返回
    o["get"+name]=function(){return value;};//setter方法首先检查值是否合法，若不合法就抛出异常
    //否则就将其存储起来
    o["set"+name]=function(v){
        if(predicate＆＆!predicate(v))
        throw Error("set"+name+":invalid value"+v);
        else
        value=v;
    };
}
```

### 函数属性与常用方法



**call()与apply()**

**JavaScript中的函数也是对象，和其他JavaScript对象没什么两样，函数对象也可以包含方法。其中的两个方法call()和apply()可以用来间接地调用函数。两个方法都允许显式指定调用所需的this值，也就是说，任何函数可以作为任何对象的方法来调用，哪怕这个函数不是那个对象的方法。（类似于，所有函数都可以作为某个对象的静态方法来调用，函数本身就是对象）**

两个方法都可以指定调用的实参。ca ll()方法使用它自有的实参 