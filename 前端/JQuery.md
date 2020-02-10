### JQuery

轻量级js框架，便于扩展

#### 使用

使用CDN （内容分发网络） 远程引用JQuery

```html
<head>
<script src="https://apps.bdimg.com/libs/jquery/2.1.4/jquery.min.js"> </script> 
</head>
```

#### 语法

 通过 jQuery，可以选取（查询，query） HTML 元素，并对它们执行"操作"（actions）。 

  为了防止文档在完全加载（就绪）之前运行 jQuery 代码 ，可以使用如下方式，否则可能加载失败

```js
$(function(){
   // 开始写 jQuery 代码...
});
```

![img](https://www.runoob.com/wp-content/uploads/2019/05/20171231003829544.jpeg)

基础语法： **$(selector).action()** 

美元符号中包裹选择器选择页面中的元素

* 元素选择器：所有标签元素，如 $("p") 
* #id选择器：通过id选择元素，如$("#test")，加#
* .class选择器：通过class选择元素，如$(".test")，加.

选取某个元素后，就要对该元素进行操作，常用函数有：

##### 事件

*  click()：当按钮点击事件被触发时会调用一个函数，参数为函数。 
*  dblclick ()： 当双击元素时，会发生 dblclick 事件，参数为函数。 
*  mouseenter()：当鼠标指针穿过元素时，会发生 mouseenter 事件，参数为函数。
*  mouseleave()/mousedown()/mouseup()：鼠标离开，移动到上方并点击鼠标，移动到上方松开鼠标发生
*  focus()：当元素获得焦点时，发生 focus 事件。当通过鼠标点击选中元素或通过 tab 键定位到元素时，该元素就会获得焦点。
*  blur()：当元素失去焦点时，发生 blur 事件。

##### 显示

*  hide()/show()：hide() 和 show() 方法来隐藏和显示 HTML 元素： 
*  toggle()：您可以使用 toggle() 方法来切换 hide() 和 show() 方法。 

#### 特效

##### 滑动

slideDown()/slideUp()向上/下滑动/slideToggle()来回切换，一般与click配合使用实现点击滑动

```html
<script> 
$(document).ready(function(){
  $("#flip").click(function(){
    $("#panel").slideToggle("slow");
  });
});
</script>
 
<style type="text/css"> 
#panel,#flip
{
	padding:5px;
	text-align:center;
	background-color:#e5eecc;
	border:solid 1px #c3c3c3;
}
#panel
{
	padding:50px;
	display:none;
}
</style>
</head>
<body>
<div id="flip">点我，显示或隐藏面板。</div>
<div id="panel">Hello world!</div>
</body>
```

#### 加载页面时自动调用js

最简单的调用方式，直接写到html的body标签里面：

```html
<html> <body onload="func1();func2();func3();"> </body> </html>
```

#### 清空input(其它标签)中的内容

直接调用val方法设置其默认值为""

```js
$("#message").val("");//清空input中的内容
```

#### 向div（其它标签）动态变更/追加内容

可以使用直接DOM文档的这种方式

```js
doucment.getElementById("").innerHtml+=xxx;
```

也可以使用方法html()进行变更：

```js
$("#history").html("xxx");
```

#### 监听键盘事件

通过函数keydown()进行监听

```js
$("body").keydown(function() {
             if (event.keyCode == "13") {//keyCode=13是回车键
                 $('#btnSumit').click();
             }
         });
```

