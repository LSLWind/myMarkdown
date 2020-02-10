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

**数据类型：**

* 所有数字都是64位浮点型数字，三个特殊数字Infinity,-Infinity与NaN分别表示正负无穷大与非数字特殊值，内置类Number提供一些可选常量，如Number.MAX_VALUE、Number.MIN_VALUE
* 字符串可直接表示（内置类），没有char，使用单引号或者双引号，转义字符使用\
* 