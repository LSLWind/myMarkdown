#### 父类Throwable

Throwable是所有异常的父类，Throwable维护两个主要信息：

* message：表示异常消息
* cause：表示触发该异常的其它异常，通过该参数可以形成一条异常链

直接来看源码，Throwable内部维护的信息为：

```java
//追踪异常
private transient Object backtrace;
//该异常维护的异常信息
private String detailMessage;
//触发该异常的其它异常
private Throwable cause = this;
```

围绕这两个参数，Throwable提供了四个构造方法

```java
    public Throwable() {
        fillInStackTrace();
    }
    public Throwable(String message) {
        fillInStackTrace();
        detailMessage = message;
    }
    public Throwable(String message, Throwable cause) {
        fillInStackTrace();
        detailMessage = message;
        this.cause = cause;
    }
    public Throwable(Throwable cause) {
        fillInStackTrace();
        detailMessage = (cause==null ? null : cause.toString());
        this.cause = cause;
    }
```

fillInStackTrace()方法的定义为：

```java
    public synchronized Throwable fillInStackTrace() {
        if (stackTrace != null ||
                backtrace != null ) {
            fillInStackTrace(0);
            stackTrace = UNASSIGNED_STACK;
        }
        return this;
    }
private native Throwable fillInStackTrace(int dummy);
```

看到native就知道，交给底层去实现了，用于初始化栈追踪信息，java对整个异常体系提供了语法级的支持，Throwable内部维护的许多东西都与底层相关，主要的核心目的就是：

1. 配合throws,thorw,try..catch...finally语法能在代码解决不了问题时抛出异常与捕获异常，保证程序代码的健壮性
2. 抛出/捕获异常时能提供/显示相关异常信息，主要方法就是常用的printStackTrace

#### 异常类体系

Throwable有两个子类：

* Error：表示系统错误或资源耗尽，由java系统自己使用，应用程序不应使用
* Exception：表示应用程序错误，可通过继承Exception或其子类自定义异常

##### Error

表示系统错误或资源耗尽,直接子类有：

* VirtualMacheError：虚拟机出现错误

* OutOfMemoryError：大名鼎鼎的OOM，内存溢出，栈、堆、方法区都有可能出现OOM
* StackOverflowError：同样出名的栈溢出（实际上还是内存溢出了），经常碰到的方法无限递归时就会出现

##### Exception

表示应用程序错误，有很多子类，三个直接子类有：

* IOException：出现I/O异常，也就是向OS请求调度I/O的时候或者OS自己调度I/O的时候出错，这个很大情况下编写应用程序时无法处理，只能抛出
* RuntimeException：运行时异常，实际上表示的是未受检异常的意思，未受检就是不强制要求程序员处理异常，受检则强制要求处理，否则会编译错误，Error及其子类，RuntimeException及其子类都是未受检异常
* SQLException：数据库SQL异常，也经常出现

##### RuntimeException

提供了大量子类，主要是为了名字不同，便于识别与处理

* NullPointerException：耳熟能详的空指针异常
* NumberFormatException：数字格式错误异常，最经典的除于0时就会出现
* IndexOutOfBoundsException：索引越界的异常，想要访问不该访问的内存，刷题的时候经常出现
* IllegalStateException：非法状态异常，官方文档给出的说法是 在非法或不适当的时间调用方法时产生的信号。换句话说，即 Java 环境或 Java 应用程序没有处于请求操作所要求的适当状态下。也即当前请求非法。 
* ClassCastException：非法强制类型转换

异常中最关键的处理方法都定义在Throwable中，自定义异常一般也仅仅是在super()的基础上增加某些信息，来看一下RuntimeException的源码：

```java
public class RuntimeException extends Exception {
    static final long serialVersionUID = -7034897190745766939L;
    public RuntimeException() {
        super();
    }
    public RuntimeException(String message) {
        super(message);
    }
    public RuntimeException(String message, Throwable cause) {
        super(message, cause);
    }
    public RuntimeException(Throwable cause) {
        super(cause);
    }
    protected RuntimeException(String message, Throwable cause,
                               boolean enableSuppression,
                               boolean writableStackTrace) {
        super(message, cause, enableSuppression, writableStackTrace);
    }
}
```

RuntimeException总共也就这么点代码，同级的IOException也一样，子类也几乎差不多，看一下NullPointerException的源码：

```java
public class NullPointerException extends RuntimeException {
    private static final long serialVersionUID = 5162710183389028792L;

    public NullPointerException() {
        super();
    }

    public NullPointerException(String s) {
        super(s);
    }
}
```

自定义异常一般也同样，仅仅是为了标识或者增加业务逻辑异常信息而丰富自己的异常体系，使整个系统更加健壮，重点在于异常的使用--什么时候抛出异常，捕获异常，对于不同的异常该怎么处理，这都是根具体的业务逻辑相关了。

#### 异常处理

使用try..catch..final捕获，使用throw在程序代码内抛出，使用throws使一个方法声明可能抛出的异常

多个异常之间可以使用|操作符

```java
catch(ExceptionA | ExceptionB)
```

一定要先抛出异常才能捕获异常，针对异常体系可以自定义自己的异常体系（实际上还是继承RuntimeException来丰富自己的异常类集合），灵活使用才是王道

**final**

一般用于释放资源，有无异常都会执行，需要注意的是如果try..catch中有return语句则return会在final后才返回（返回值先保存在一个临时变量中），**如果final中有return则会覆盖try..catch中的异常以及返回值**，如果final中抛出异常（一般不能这么干）则覆盖原有异常

**throws**

声明一个方法可能抛出的异常，未受检异常不强制，受检异常强制，多个异常用逗号分开

#### try-with-resources

针对使用资源的场景而被特意加上去的，毕竟使用资源的场景太多了，该语法与接口java.lang.AutoCloseable配套使用，该接口内部只有一个close方法用于关闭资源，JDK许多内置的资源获取类（如FileInputStream）已经实现了该接口，如果不使用则是常见的try..final语法，使用之后为：

```java
try(AutoCloseable r=new FileInputStream("hello")){//创建资源
//使用资源r即可，在执行完try之后会自动调用r的close方法关闭资源
}
```

