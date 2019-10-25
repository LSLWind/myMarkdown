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

