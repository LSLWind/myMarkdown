AJAX = 异步 JavaScript 和 XML。异步处理的数据传输/前端处理标准

AJAX 是一种用于创建快速动态网页的技术。

通过在后台与服务器进行少量数据交换，AJAX 可以使网页实现异步更新。这意味着可以在不重新加载整个网页的情况下，对网页的某部分进行更新。

 ![AJAX](https://www.runoob.com/wp-content/uploads/2013/09/ajax-yl.png) 

#### XMLHttpRequest发送异步请求

XMLHttpRequest 对象是 AJAX 的基础。所有现代浏览器均支持 XMLHttpRequest 对象。

XMLHttpRequest 用于在后台与服务器交换数据。这意味着可以在不重新加载整个网页的情况下，对网页的某部分进行更新。

```js
xmlhttp=new XMLHttpRequest();
```

如需将请求发送到服务器，使用 XMLHttpRequest 对象的 open() 和 send() 方法：

| 方法                         | 描述                                                         |
| :--------------------------- | :----------------------------------------------------------- |
| open(*method*,*url*,*async*) | 规定请求的类型、URL 以及是否异步处理请求。*method*：请求的类型；GET 或 POST，url：文件在服务器上的位置，async：true（异步）或 false（同步） |
| send(*string*)               | 将请求发送到服务器。*string*：仅用于 POST 请求               |

 当使用 async=true 时，规定在响应处于 onreadystatechange 事件中的就绪状态时执行的函数： 

```js
//数据传输完成后前端调用该方法更新数据，响应体数据都在xmlhttp中
xmlhttp.onreadystatechange=function()
{
    if (xmlhttp.readyState==4 && xmlhttp.status==200)
    {
        document.getElementById("myDiv").innerHTML=xmlhttp.responseText;
    }
}
xmlhttp.open("GET","/try/ajax/ajax_info.txt",true);//请求指定url处理后的response
xmlhttp.send();//参数string只对post有效，用于发送类表单数据

```

如果需要像 HTML 表单那样 POST 数据，使用 setRequestHeader() 来添加 HTTP 头。然后在 send() 方法中规定希望发送的数据：

| 方法                             | 描述                                                         |
| :------------------------------- | :----------------------------------------------------------- |
| setRequestHeader(*header,value*) | 向请求添加 HTTP 头。*header*: 规定头的名称,*value*: 规定头的值 |

```js
xmlhttp.open("POST","/try/ajax/demo_post2.php",true); 
mlhttp.setRequestHeader("Content-type","application/x-www-form-urlencoded"); xmlhttp.send("fname=Henry&lname=Ford");
```

#### XMLHttpRequest处理返回响应

使用 XMLHttpRequest 对象的 responseText 或 responseXML 获得服务器的响应，调用onreadystatechange异步更新前端，因为数据已经拿到了，在XMLHttpRequest对象中。

| 属性         | 描述                       |
| :----------- | :------------------------- |
| responseText | 获得字符串形式的响应数据。 |
| responseXML  | 获得 XML 形式的响应数据。  |

##### responseText 属性

如果来自服务器的响应并非 XML，使用 responseText 属性。responseText 属性返回字符串形式的响应

document.getElementById("myDiv").innerHTML=**xmlhttp.responseText**;

##### responseXML 属性

如果来自服务器的响应是 XML，需要作为 XML 对象进行解析，使用 responseXML 属性，通过getElementsByTagName获取标签值

```js
xmlDoc=xmlhttp.responseXML; 
x=xmlDoc.getElementsByTagName("ARTIST"); 
```

##### onreadystatechange 事件

当请求被发送到服务器时，需要执行一些基于响应的任务。每当 readyState 改变时，就会触发onreadystatechange 事件。

 XMLHttpRequest 的状态信息中的三个重要的属性：

| 属性               | 描述                                                         |
| :----------------- | :----------------------------------------------------------- |
| onreadystatechange | 存储函数（或函数名），每当 readyState 属性改变时，就会调用该函数。 |
| readyState         | 存有 XMLHttpRequest 的状态。从 0 到 4 发生变化。0: 请求未初始化1: 服务器连接已建立2: 请求已接收3: 请求处理中4: 请求已完成，且响应已就绪 |
| status             | 200: "OK" 404: 未找到页面                                    |

在 onreadystatechange 事件中，我们规定当服务器响应已做好被处理的准备时所执行的任务。

当 readyState 等于 4 且状态为 200 时，表示响应已就绪：

```js
xmlhttp.onreadystatechange=function()
{
    if (xmlhttp.readyState==4 && xmlhttp.status==200)
    {
        document.getElementById("myDiv").innerHTML=xmlhttp.responseText;
    }
}
```

#### JQuery使用ajax

通用的jquery的ajax写法为

```js
$.ajax({    
    url: "http://www.hzhuti.com",    //请求的url地址   
    dataType: "json",   //返回格式为json    
    async: true, //请求是否异步，默认为异步，这也是ajax重要特性    
    data: { "id": "value" },    //参数值    
    type: "GET",   //请求方式    
    beforeSend: function(request) {        
      //请求前的处理
      request.setRequestHeader("Content-type","application/json");
      request.setRequestHeader("Source","101");
      request.setRequestHeader("Token","aaw--wssw-ss...");
    },   
    success: function(data) {        
    //请求成功时处理    
    },   
    complete: function() {        
      //请求完成的处理    
    },    
    error: function() {        
      //请求出错处理    
    }
});
```

ajax的详细属性为

```js
url
String
(默认: 当前页地址) 发送请求的地址。

type
String
(默认: "GET") 请求方式 ("POST" 或 "GET")， 默认为 "GET"。
注意：其它 HTTP 请求方法，如 PUT 和 DELETE 也可以使用，但仅部分浏览器支持。

timeout
Number
设置请求超时时间（毫秒）。此设置将覆盖全局设置。

async
Boolean
(默认: true) 默认设置下，所有请求均为异步请求。如果需要发送同步请求，请将此选项设置为 false。注意，同步请求将锁住浏览器，用户其它操作必须等待请求完成才可以执行。

beforeSend
Function
发送请求前可修改 XMLHttpRequest 对象的函数，如添加自定义 HTTP 头。XMLHttpRequest 对象是唯一的参数。
function (XMLHttpRequest) { 
  this; 
}

cache
Boolean
(默认: true) jQuery 1.2 新功能，设置为 false 将不会从浏览器缓存中加载请求信息。

complete
Function
请求完成后回调函数 (请求成功或失败时均调用)。参数： XMLHttpRequest 对象，成功信息字符串。
function (XMLHttpRequest, textStatus) { 
}

contentType
String
(默认: "application/x-www-form-urlencoded") 发送信息至服务器时内容编码类型。默认值适合大多数应用场合。

data
Object,String
发送到服务器的数据。将自动转换为请求字符串格式。GET 请求中将附加在 URL 后。
查看 processData 选项说明以禁止此自动转换。必须为 Key/Value 格式。
如果为数组，jQuery 将自动为不同值对应同一个名称。
如 {foo:["bar1", "bar2"]} 转换为 '&foo=bar1&foo=bar2'。

dataType
String
预期服务器返回的数据类型。如果不指定，jQuery 将自动根据 HTTP 包 MIME 信息返回 responseXML 或 responseText，并作为回调函数参数传递，可用值:
"xml": 返回 XML 文档，可用 jQuery 处理。
"html": 返回纯文本 HTML 信息；包含 script 元素。
"script": 返回纯文本 JavaScript 代码。不会自动缓存结果。
"json": 返回 JSON 数据 。
"jsonp": JSONP 格式。使用 JSONP 形式调用函数时，
如 "myurl?callback=?" jQuery 将自动替换 ? 为正确的函数名，以执行回调函数。
 
error
Function
(默认: 自动判断 (xml 或 html)) 请求失败时将调用此方法。
这个方法有三个参数：XMLHttpRequest 对象，错误信息，（可能）捕获的错误对象。
function (XMLHttpRequest, textStatus, errorThrown) {
   // 通常情况下textStatus和errorThown只有其中一个有值  this; 
}

global
Boolean
(默认: true) 是否触发全局 AJAX 事件。
设置为 false 将不会触发全局 AJAX 事件，如 ajaxStart 或 ajaxStop 。
可用于控制不同的Ajax事件

ifModified
Boolean
(默认: false) 仅在服务器数据改变时获取新数据。使用 HTTP 包 Last-Modified 头信息判断。

processData
Boolean
(默认: true) 默认情况下，发送的数据将被转换为对象(技术上讲并非字符串) 以配合默认内容类型 "application/x-www-form-urlencoded"。
如果要发送 DOM 树信息或其它不希望转换的信息，请设置为 false。

success
Function
请求成功后回调函数。这个方法有两个参数：服务器返回数据，返回状态function (data, textStatus) { 
  // data could be xmlDoc, jsonObj, html, text, etc... 
}
```

