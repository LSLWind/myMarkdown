### 基础操作

> * 变量被声明时内部赋初始值undefined，对undefined进行数操作结果为NaN，字符操作则仍为undefined
> * js变量遵循驼峰命名法
> * ' ' 与 " " 作用相同，一般' '写在外面
> * ==比较值,===比较值与类型，同理还有!=与!==

#### 严格模式

'use strict'

顶部加上这一句，规范js的写法，只允许出现在脚本或函数的开头

### 内置对象

#### 字符串对象

字符串不可变，只能改变引用

##### 属性

> * length:长度

#### 数组

数组可以存储任意对象，嵌入数组也可以

##### 常用方法

> * push():末尾加上一个对象
> * pop():删除末尾对象并返回
> * shift():删除第一个对象并返回
> * unshift():开头加上一个对象

### 函数

使用关键字function 声明，参数不要声明类型

函数外声明的变量都是全局变量，内部声明的变量是局部变量，如果同名局部变量优先于全局变量

## 前端相关

### DOM

dom即document Object Model文档对象模型

![DOM HTML tree](https://www.runoob.com/images/pic_htmltree.gif)

通过控制DOM结构来构建动态页面

js内置document表示整个DOM，而通过DOM控制返回的各元素也都是可控制的相应内置对象

#### document常用方法

* getElementById(""):通过id返回元素,id唯一
* getElementsByClassName(""):通过class返回元素,class不唯一
* getElementsByTagName(""):通过标签名(html标签)返回元素
* write(""):直接向html输出流写内容

#### 元素常用属性与事件

##### 属性

* innerHtml:该元素的html输出流内容

* 与具体元素绑定的各种属性都可以修改，例

* ```js
  document.getElementById("image").src="landscape.jpg";
  ```

* style:表示该元素的css样式，可修改各种css属性，例

* ```javascript
  document.getElementById("p2").style.fontSize="larger";
  ```

##### 事件

- onclick:点击时做出的反应，一般指定一个函数,例

- ```js
  document.getElementById("myBtn").onclick=function(){displayDate()};
  ```

- onload/onunload:进入/离开页面时触发的函数

- onmouseover/onmouseout:鼠标悬浮/移开时触发的函数

##### 增加事件句柄

类似于事件监听器，每个DOM对象都可以进行设置，语法为

*element*.addEventListener(*event, function, useCapture*);

event为一个string,如"click"，function为触发函数,useCapture为触发后使用方式，默认为false,冒泡式：内部元素事件先于外部元素事件触发，true为捕获式：外部元素事件先于内部元素事件被触发