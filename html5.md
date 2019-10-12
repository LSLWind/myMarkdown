#### 表单form

<form\>为用户输入创建表单，向服务器传输数据，需包含多个子选项如input

* action : 值为url ，向指定url发送数据
* method：发送方法, get或post
* target：规定在何处打开url,_blank为新窗口,_self为默认值，当前窗口 
* name：规定表单名，后台解析时使用

##### 示例

```html
<div >
    登录
    <form action="home.html" method="post" name="logIn">
        <div>
            <label for="email"></label><input type="text" id="email" placeholder="邮箱" >
        </div>
        <div>
            <input type="password" placeholder="密码">
        </div>
        <div>
            <input type="submit" value="登录">
        </div>
    </form>


</div>

<div>
    注册
    <form action="home.html" method="post" name="signUp">
        <div>
            <label for="register"></label><input type="text" id="register" placeholder="邮箱" formmethod="post" >
        </div>
        <div>
            <input type="password " placeholder="密码">
        </div>
        <div>
            <input type="password" placeholder="重复输入密码">
        </div>
        <div>
            <input type="submit" value="注册">
        </div>
    </form>

</div>
```

##### <input\>:输入框

type:**button** checkbox file hidden image **password** radio reset **submit** **text**

placeholder:初始提示字段

value：输入框的默认值，通常在type为submit时使用指定显示默认值

##### <label\>:与input绑定作为标记

for：绑定指定Input的id，当点击lable文本时将焦点移动到绑定输入框