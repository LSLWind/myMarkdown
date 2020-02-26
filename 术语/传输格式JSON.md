JSON 是存储和交换文本信息的语法。类似 XML,JSON 比 XML 更小、更快，更易解析。

### 语法

JSON 语法是 JavaScript 对象表示语法的子集。

- 数据在名称/值对中
- 数据由逗号分隔
- 大括号保存对象
- 中括号保存数组

JSON 数据的书写格式是：名称/值对。

名称/值对包括字段名称（在双引号中），后面写一个冒号，然后是值,

JSON 值可以是：

- 数字（整数或浮点数）
- 字符串（在双引号中）
- 逻辑值（true 或 false）
- 数组（在中括号中）
- 对象（在大括号中）
- null

JSON 数字可以是整型或者浮点型：

   例:{ "age":30 }

**JSON 对象**,**JSON 对象在大括号（{}）中书写：**

对象可以包含多个名称/值对：

{ "name":"xxx" , "age":13 }与这条 JavaScript 语句等价：name = "xxx" age=13

**JSON 数组在中括号中书写,数组可包含多个对象/字符串：**

{ "sites": [ { "name":"google" , "url":"www.google.com" }, { "name":"微博" , "url":"www.weibo.com" } ] }

在上面的例子中，对象 "sites" 数组。每个对象代表一条关于某个网站（name、url）的记录。

**JSON 布尔值可以是 true 或者 false：**

{ "flag":true }

**JSON 可以设置 null 值：**

{ "xx":null }

### JSON 对象

**使用点号（.）来访问对象的值：**

> var myObj = { "name":"xxx", "alexa":10000, "site":null };
>
> x = myObj.name;

**也可以使用中括号（[]）来访问对象的值：**

var myObj = { "name":"xxx", "alexa":10000, "site":null };

x = myObj["name"];

**可以使用 for-in 来循环对象的属性：**

> var myObj = { "name":"xxx", "alexa":10000, "site":null };
>
> for (x in myObj) {
>
>    document.getElementById("demo").innerHTML += x + "<br>";
>
> }

**在 for-in 循环对象的属性时，使用中括号（[]）来访问属性的值：**

> var myObj = { "name":"xxx", "alexa":10000, "site":null };
>
> for (x in myObj) {
>
>    document.getElementById("demo").innerHTML += myObj[x] + "<br>";
>
> }

**JSON 对象中可以包含另外一个 JSON 对象：**

> myObj = {
>
> "name":"xxx", "alexa":10000,
>
> "sites": { 
>
> ​        "site1":"www.xxxx.com",
>
> ​        "site2":"m.xxxx.com",
>
> ​        "site3":"c.xxxx.com"
>
> ​       }
>
> }

可以使用点号(.)或者中括号([])来访问嵌套的 JSON 对象。

> x = myObj.sites.site1; // 或者 x = myObj.sites["site1"];

**可以使用 delete 关键字来删除 JSON 对象的属性：**

> delete myObj.sites.site1;

**可以使用中括号([])来删除 JSON 对象的属性：**

> delete myObj.sites["site1"]

JSON是key:值得集合，而{}表示对象，所以不允许，多个{}出现在同一个{}中，如果有，使用数组[]，[]中可以加多个{}

{{id:1,name:"x"},{id:2,name:"y"}}}--------------错误

'{ sites: [{id:1,name:"x"},{id:2,name:"y"}] }'--------------正确

'{ [{id:1,name:"x"},{id:2,name:"y"}] }'--------------正确

### JSON.parse()

JSON 通常用于与服务端交换数据,在接收服务器数据时一般是字符串。

可以使用 JSON.parse() 方法将数据转换为 JavaScript 对象。

**JSON.parse(text[, reviver])**

**参数说明：**

- **text:**必需， 一个有效的 JSON 字符串。
- **reviver:** 可选，一个转换结果的函数， 将为对象的每个成员调用此函数。

解析完成后，我们就可以在网页上使用 JSON 数据了

**eval(string):函数可计算某个字符串，并执行其中的的 JavaScript 代码。**

> ```html
> eval("var a=1");     // 声明一个变量a并赋值1。
> eval("2+3");         // 执行加运算，并返回运算值。
> eval("mytest()");    // 执行mytest()函数。
> eval("{b:2}");       // 声明一个对象。
> ```
>
> ![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

### JSON.stringify()

JSON 通常用于与服务端交换数据,在向服务器发送数据时一般是字符串。

可以使用 JSON.stringify() 方法将 JavaScript 对象转换为字符串。

```html
JSON.stringify(value[, replacer[, space]])
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

**参数说明：**

- value:

  必需， 要转换的 JavaScript 值（通常为对象或数组）。

- replacer:

  可选。用于转换结果的函数或数组。

  如果 replacer 为函数，则 JSON.stringify 将调用该函数，并传入每个成员的键和值。使用返回值而不是原始值。如果此函数返回 undefined，则排除成员。根对象的键是一个空字符串：""。

  如果 replacer 是一个数组，则仅转换该数组中具有键值的成员。成员的转换顺序与键在数组中的顺序一样。当 value 参数也为数组时，将忽略 replacer 数组。

- space:

  可选，文本添加缩进、空格和换行符，如果 space 是一个数字，则返回值文本在每个级别缩进指定数目的空格，如果 space 大于 10，则文本缩进 10 个空格。space 也可以使用非数字，如：\t。

### eval()

由于 JSON 语法是 JavaScript 语法的子集，JavaScript 函数 eval() 可用于将 JSON 文本转换为 JavaScript 对象。

eval() 函数使用的是 JavaScript 编译器，可解析 JSON 文本，然后生成 JavaScript 对象。必须把文本包围在括号中，这样才能避免语法错误：

> var txt = '{
>
> ['{ "name":"google" , "url":"www.google.com" },' + '{ "name":"微博" , "url":"www.weibo.com" } ]
>
> }';
>
> var obj = eval ("(" + txt + ")");