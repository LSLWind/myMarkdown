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

## 语法

js使用Unicode字符集编码，大小写敏感，分号可选（尽量使用分号）

### 基础数据类型

* 所有数字都是64位浮点型数字，三个特殊数字Infinity,-Infinity与NaN分别表示正负无穷大与非数字特殊值，内置类Number提供一些可选常量，如Number.MAX_VALUE、Number.MIN_VALUE
* 字符串可直接表示（内置类），没有char，使用单引号或者双引号，转义字符使用\
* 函数：js中的函数是一种数据类型，也就是函数可以被作为参数来使用，也可以被赋值或存放在数组数组、对象等中
* lambda表达式：将函数直接赋值给变量，如var square=function(x){return x*x;}
* 对象：已命名的数据的集合
* 数组：可以存放任意数据类型，可通过函数Array()创建，如var a=new Array();，可传递一个参数，为数组长度
* null/undefined：null就代表空，undefined代表未声明，已声明或未赋值

 JavaScript解释器有自己的内存管理机制，可以自动对内存进行垃圾回收（garbage collection）。 

### 对象

**创建对象**

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

**类class**