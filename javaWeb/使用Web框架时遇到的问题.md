#### Spring中使用@Autowired注解静态实例对象

@Autowired是不能注解静态实例对象的，因为静态字段优先加载到方法区当中，实例对象仍然放在栈中，但是如如不初始化显然为null，不可能扫描到注解的静态字段，因此编译正常，但一运行就会报空指针错误（2小时找错，）

当然如果想要实例化，也有其他办法，不能直接给静态字段注入，但可以给构造器或者方法加上@Autowired在运行时注入实例并指向引用：

方式一
将@Autowired 注解到类的构造函数上。很好理解，Spring扫描到AutowiredTypeComponent的bean，然后赋给静态变量component。示例如下：

```java
@Component
public class TestClass {

    private static AutowiredTypeComponent component;
    
    @Autowired
    public TestClass(AutowiredTypeComponent component) {
        TestClass.component = component;
    }
    
    // 调用静态组件的方法
    public static void testMethod() {
        component.callTestMethod()；
    }

}
```

方式二
给静态组件加setter方法，并在这个方法上加上@Autowired。Spring能扫描到AutowiredTypeComponent的bean，然后通过setter方法注入。示例如下：

```java
@Component
public class TestClass {

    private static AutowiredTypeComponent component;
    
    @Autowired
    public void setComponent(AutowiredTypeComponent component){
        TestClass.component = component;
    }
    
    // 调用静态组件的方法
    public static void testMethod() {
        component.callTestMethod()；
    }

}
```

#### Ajax时获取不到Select表单的值

真的是一个奇怪的问题，最开始使用的是：

```js
var sex = $("#sex:selected").text();
```

总是获取不到值，打印时值的类型为undefined

换成

```js
var sex = $("#sex :selected").text();
```

就可以了，加了一个空格，无语了。

#### Ajax请求后总是自动刷新

问题在于按钮button的类型type="submit"总是会刷新请求后的页面，为避免刷新，改为type="button"即可。

#### Ajax进入不到success

加上async:false，本身就是异步请求，加上false要获取数据，同步进入success

```js
 $.ajax({
        type: "POST",//方法类型
        dataType: "json",//预期服务器返回的数据类型
        url: "/students/add",
        contentType: "application/json; charset=utf-8",
        data: JSON.stringify(data),
        async:false,
        success: function (result) {
            alert(result.message);
            showErrorInfo(result.message());
            return;
        },
        error: function () {
            showErrorInfo("接口异常，请联系管理员！");
            return;
        }
    });
```

#### js获取不到标签span的值

使用表单时常使用$("#id").val()这样的方式（这样好像是获取输入的值），而其他不输入的标签如<span\>应该使用$("#id").text()这样的获取标签包裹的值

#### @RequestMapping请求数据类型绑定错误

遵循格式，DispatcherHandlerServlet根据最短前缀匹配，如果在controller上声明一个

```
@RequestMapping("/students")
```

之后如果有请求映射

```
@RequestMapping("/{departmentId}/{majorId}")
```

这样写本身没问题，如果在加一个

```
@RequestMapping("/select/{studentId}")
```

就会出现数据转换错误，如果请求的是/select/{studentId}，那么DispatcherHandlerServlet会先找到@RequestMapping("/{departmentId}/{majorId}")，因此出现数据绑定错误，将第二个请求映射修改一下，使用固定前缀即可

```
@RequestMapping("/all/{departmentId}/{majorId}")
```

