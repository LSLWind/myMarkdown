#### HashCode

int类型，主要用于在集合Set中确定对象的存储地址，防止出现重复元素，本地方法(native)，在Set中如果使用equals()则消耗太高，因此使用hash算法先判断一次HashCode,如果HashCode相同则再使用equals()判断

#### ==与equals

对于基本类型==比较值，对于引用类型比较引用，equals默认比较引用，需要时重写比较值（如String的equals）

#### final

- final 修饰的类，该类不能被继承。
- final 修饰的方法不能被重写。
- final 修饰的变量叫常量，常量必须初始化，初始化之后值就不能被修改。

#### String/StringBuffer/StringBuilder

内部都是字符串数组，区别在于String不可变，StringBuffer与StringBuilder可变，StringBuffer线程安全，StringBuilder线程不安全（线程安全的一般效率较低）

#### String内存分配

使用=分配到常量池（字符串常量池）中，使用new分配到堆内存中



### 反射

 每个类都在内存中保留有相应信息，这些信息保存于泛型类Class<?\>中，每个实例或者静态类都可以动态获取Class<>对象，从而在运行时知道对象类的内部信息，称之为反射。

1. 如果知道类名，可以直接通过类名.class获取Class<?>

2. Object下有一个getClass()方法返回一个Class<?>

3. Class类有一个静态方法forName(String name)可以通过类的名称返回Class<?>

作用
运行时动态获取类的具体信息,使用Class.forName()还可以将某个类直接加载到内存，常用于驱动器的加载

### 注解

所谓注解，就是元数据(描述数据的数据)，与注释不同，注解可以联合反射对注解所声明的类、字段、方法进行动态检验，执行代码等一系列操作。

如@Override，表明该方法必须被重写，否则直接报错。

注解为java注入元编程的能力，注解大量应用于各种框架，用于简化配置，提高灵活性，进一步抽象代码。

**格式**

public @interface 注解名称{
    属性列表;
}

**元注解**

元注解，即注解的注解，定义一个注解实际上还是继承了接口Annotation,要想更方便使用注解必须约束注解的行为，即注解约束数据，元注解约束注解。

@Target() 用于约束注解的作用域，使用枚举ElementType选择,如果选择多个则使用{},如

    @Target({ElementType.METHOD,ElementType.FIELD})

@Retention()用于约束注解的保留时间段，有三个,使用RetentionPolicy选择，如果想通过反射获取注解信息则必须保留到运行时，即选择RUNTIME

     @Retention(RetentionPolicy.RUNTIME)

@Inherited() 约束注解是否可以被继承，默认不继承，即父类有注解，但子类没有，子类默认不继承注解

\**本质**

注解的本质是接口，所写的注解会被自动编译成接口interface并继承接口Annotation。

既然是接口，自然可以在内部声明抽象方法，但使用注解时可以给内部的方法名赋值，如

定义注解：

public @interface Lsl {
    int getAge();

}
使用时就可以直接这样写@Lsl(getAge = 18)，直接就对getAge进行赋值，因此getAge()看起来又是一个属性。
内部声明属性时可以设置默认值，如int getAge() default 18;

**使用**

注解的核心在于注解所修饰的类/方法/字段可以通过反射读取其所绑定的注解信息。

因此注解的使用为:定义注解，使用注解，读取注解

读取注解就需要用到反射，通过反射获取注解信息，由注解信息知道约束条件从而可以进行动态检查

 Class<?>提供多种方法获取注解,getAnnotations()获取该Class对象上的所有注解，具体对象如Field也有getDeclaredAnnotations()获取注解,返回一个Annotation[]。如果是方法或者构造器因为参数也可以有注解，所以有方法getParameterAnnotations()获取参数的注解，返回一个Annotation[][],每一个参数对应一个Annotation[]。

读取注解之后因为注解携带信息，所以要把信息读取出来，因为注解的本质是接口，注解内部定义了方法，使用注解时直接复制，读取时直接调用方法即可，通过注解 注解对象 = 使用了该注解的XXX.getAnnotation(注解.class) 这样的形式返回注解对象，通过注解对象调用方法获取信息

**实例**

定义注解

@Target({ElementType.METHOD,ElementType.FIELD,ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
public @interface Lsl {
    int age();
}
使用注解

@Deprecated
@Lsl(age = 12)
public class TestUse {
}
读取注解(读取信息)

```java
public class TestDemo {
    public static void main(String[] args){
        Class<?> cls=TestUse.class;
        Annotation[] annotations=cls.getAnnotations();
        for(Annotation annotation:annotations){
            System.out.println(annotation.annotationType());
        }

        System.out.println("--------------------------------");
        Lsl myAnnotation=cls.getAnnotation(Lsl.class);//获取注解信息，直接传入注解class返回注解(接口)对象
        System.out.println(myAnnotation.age());
    }

}
```

结果为:

@java.lang.Deprecated()

@mapper.Lsl(age=12)

12

作用
        使用注解可以轻松的描述数据的信息，假如A想知道B的信息，直接通过反射读取B的注解，轻松获取信息，用于各种框架当中，简化了繁琐的xml配置。

### 动态代理

动态代理为java注入运行时动态更改对象代码的能力，常用于实现AOP。既然有动态代理，那么必定也有静态代理。

静态代理实际上就是一种设计模式的应用，即代理模式。

**静态代理:**

       假如有一个类A,A中有一个方法a(),对于a()来说，不同的地方创建的A的实例调用a()方法时需要加上不同的东西，如日志记录等，即有时需要在a()上添加不同的东西，如何添加? 好办，直接将该方法抽象出来放在一个接口里,A继承该接口,然后实现，然后有一个代理类Proxy，该代理类内部维护A,重写a()按需添加即可。即对A进行包装，对于可能变化的方法重写覆盖即可。
    
        使用时，即可以使用原有的A,也可以使用经代理强化过的Proxy。

```java
public class TestDemo {
    interface IService{
        void say();
    }
    static class A implements IService{
        public void say(){
            System.out.println("I am A");
        }
    }
    static class Proxy implements IService{
        private A a;
        public Proxy(A a){
            this.a=a;
        }

        public void say(){
            System.out.println("我是代理,我先加点东西");
            a.say();
            System.out.println("我是代理，方法调用结束后加点东西");
        }
    }
     
    public static void main(String[] args){
        System.out.println("正常实现A");
        A a=new A();
        a.say();
        System.out.println("代理强化A");
        Proxy aProxy=new Proxy(a);
        aProxy.say();
    }

}
```

执行结果为:
正常实现A
I am A
代理强化A
我是代理,我先加点东西
I am A
我是代理，方法调用结束后加点东西

**动态代理:**

        说实话，静态代理的功能一点也不强，如果A,B,C...有许多类都需要在方法前加一点东西，不可能为每一个类都写一个对应的代理进行强化，如果能动态生成就好了。有需求必然有变化，java为应对需求推出了类java.lang.reflect.Proxy同时对动态代理设置了一定的规范。
    
          首先，将需要切入的方法抽取出来放在接口中，A照样实现接口从而实现方法。
    
          然后，写一个通用代理类，代理类Proxy内部不在维护A，而是维护一个Object,该代理类要实现通用的代理接口InvocationHandler,该接口只有一个方法invoke()。Proxy内部维护Object,这个Object就可以代表任何上述接口,想要执行接口方法JVM就会自动调用内部invoke()方法。

invoke有三个参数：Object proxy,JVM自动生成的对应接口对象,一般不用管它、Method method：要切入的方法、Object args：切入的方法的参数

invoke()返回一个Object,在invoke()内部要使用method.invoke(obj,args)调用内部维护的Object的原始方法，然后返回

        最后想用代理时创建代理即可,要创建代理，简单，调用(XXX)Proxy.newProxyInstance()即可，直接创建代理并进行强制类型转换

该方法也有三个参数:ClassLoader loader:任意上述接口对象的类加载器、Class<?>[] interfaces:任意上述接口对象的数组、InvocationHandler h:代理

        简而言之，动态代理就是在运行时动态生成指定接口的代理用于强化接口方法，就是动态生成静态代理，在内部使用时，每次通过Proxy.newProxyInstance()指定接口与通用代理类，通用代理类的构造器传入具体的接口实现类作为参数。JVM内部自动生成一个与指定接口绑定的代理类，名为ProxyX(X是自增数字),该代理类被传入通用代理类的invoke方法中，同时将绑定到的实现接口的具体类的方法method与方法参数args一并传入invoke,然后执行invoke()中的代码，要调用原方法，须使用method.invoke(obj,args),返回一个Object作为结果，返回该结果即可

实例：

```java
public class TestDemo {
    interface IA{
        void sayA();
    }
    interface IB{
        void sayB();
    }
    static class A implements IA{
        public void sayA(){
            System.out.println("I am A");
        }
    }
    static class B implements IB{
        public void sayB(){
            System.out.println("I am B");
        }
    }
    static class MyProxy implements InvocationHandler {
        private Object obj;
        public MyProxy(Object obj){
            this.obj=obj;
        }

        public Object invoke(Object proxy, Method method,Object[] args){
            System.out.println("我是自动创建的代理"+proxy.getClass().getName());
            System.out.println("我是代理，我需要处理方法"+method.getName());
     
            if(args!=null)System.out.println("我是代理，该方法有参数"+args.length+"个");
            try{
                Object result=method.invoke(obj,args);//执行原始object方法
                return result;
            }catch (Exception e){
                e.printStackTrace();
            }
            return null;
        }
    }
     
    public static void main(String[] args){
        IA a=(IA) Proxy.newProxyInstance(IA.class.getClassLoader(),new Class<?>[]{IA.class},new MyProxy(new A()));
        a.sayA();
        IB b=(IB)Proxy.newProxyInstance(IB.class.getClassLoader(),new Class<?>[]{IB.class},new MyProxy(new B()));
        b.sayB();
    }

}
```

结果为：
我是自动创建的代理mapper.$Proxy0
我是代理，我需要处理方法sayA
I am A
我是自动创建的代理mapper.$Proxy1
我是代理，我需要处理方法sayB
I am B

**cglib动态代理**

​        java sdk创建动态代理的要求必须是先创建接口，在动态生成代理接口的代理类，在转回接口，实际上这也有点麻烦，因为对每一个需要强化的方法都要有配套的接口，有没有更简单的方法?有，cglib类库，一个第三方类库，该类库采用继承的思想动态生成代理类，该代理类是原类的子类，对所有public非final进行重写，如果要加强方法，也只需在通用代理类中进行操作即可，省去了接口这一"中间件"

        首先，通用代理类要实现MethodInterceptor接口，该接口有方法intercept(),是主要的实现方法。

该方法有四个参数,Object object:内部自动生成的被代理类的子类   Method method:被代理类的方法  Object[] args:被代理类的方法的参数  MethodProxy proxy:自动生成的方法的代理类，调用原方法就是

proxy.invokeSuper(object,args)
返回一个Onject

然后使用自动创建的代理类，类似于:

```java
Enhancer enhancer=new Enhancer();
enhancer.setSuperclass(I.class);//绑定原类，内部自动生成该类的代理子类
enhancer.setCallback(new CommonProxy());//绑定通用代理类，内部自动生成被代理类的方法的代理类
I lsl=(I) enhancer.create();//返回强化后的原类的对象

```

cglib实例：

```java
public class TestDemo {
    static class I{
        public void say(){
            System.out.println("I am I");
        }
    }

    static class CommonProxy implements MethodInterceptor {
        public Object intercept(Object object, Method method, Object[] args, MethodProxy proxy){
            System.out.println(object.getClass().toString());
            System.out.println(method.getName());
            System.out.println(proxy.toString());
            try {
                return proxy.invokeSuper(object,args);
            }catch (Throwable e){
                e.printStackTrace();
            }
            return null;
        }
    }
    public static void main(String[] args){
        Enhancer enhancer=new Enhancer();
        enhancer.setSuperclass(I.class);
        enhancer.setCallback(new CommonProxy());
        I lsl=(I) enhancer.create();
        lsl.say();
    }

}

```

结果为：
class mapper.TestDemo$I$$EnhancerByCGLIB$$16a044b1
say
net.sf.cglib.proxy.MethodProxy@12410ac
I am I
        省略设置接口这一步，cgling使用继承思想，直接动态创建被代理类的子类，对所有public非final方法创建方法代理类，执行intercept()方法，比java sdk更灵活方便强大。



### 常用注解

**@ConstructorProperties** 

 https://docs.oracle.com/javase/8/docs/api/java/beans/ConstructorProperties.html 

用在构造器上，表明构造器中的哪些参数可以通过getter获得

```java
 public class Point {
       @ConstructorProperties({"x", "y"})
       public Point(int x, int y) {
           this.x = x;
           this.y = y;
       }

       public int getX() {
           return x;
       }

       public int getY() {
           return y;
       }

       private final int x, y;
   }
```

因为运行时无法获得参数，通过该注解表明，x,y可通过getX()与getY()获得

### NIO

IO分为文件IO与网络IO等

可简单认为：**IO是面向流的处理，同步阻塞IO，NIO是面向块(缓冲区)的处理，同步非阻塞IO**

NIO主要有**三个核心部分组成**：

- **buffer缓冲区**
- **Channel通道**
- **Selector选择器**

**Channel**

 数据可以从Channel读到Buffer中，也可以从Buffer 写到Channel中 。Channel通道用于传输数据，常用实现有：

- FileChannel   文件通道
- DatagramChannel  UDP通道
- SocketChannel      TCP通道
- ServerSocketChannel  监听TCP连接，每个先来的连接都会创建一个SocketChannel

Channel既可以从通道中读取数据，又可以写数据到通道。但流的读写通常是单向的。通道可以异步地读写。通道中的数据总是要先读到一个Buffer，或者总是要从一个Buffer中写入。即数据从通道读入缓冲区，从缓冲区写入通道

开启通道有两种方式

```java
//本地IO
FileInputStream fileInputStream = new FileInputStream("F:\\x");
FileChannel inchannel = fileInputStream.getChannel();// 得到文件的输入通道
// jdk1.7后通过静态方法.open()获取通道
FileChannel.open(Paths.get("F:\\x"), StandardOpenOption.WRITE);
```

通道之间可使用transferFrom（FileChannel到FileChannel）或transferTo（FileChannel到其它Channel）传递数据

```java
RandomAccessFile fromFile = new RandomAccessFile("fromFile.txt", "rw");
FileChannel      fromChannel = fromFile.getChannel();
RandomAccessFile toFile = new RandomAccessFile("toFile.txt", "rw");
FileChannel      toChannel = toFile.getChannel();
long position = 0;
long count = fromChannel.size();
toChannel.transferFrom(position, count, fromChannel);
```

 方法的输入参数position表示从position处开始向目标文件写入数据，count表示最多传输的字节数。如果源通道的剩余空间小于 count 个字节，则所传输的字节数要小于请求的字节数。 

**Buffer**

Buffer缓冲区用于实际存储数据，可看做一块存储数据的内存，常用实现有：

- ByteBuffer
- CharBuffer
- DoubleBuffer
- FloatBuffer
- IntBuffer
- LongBuffer
- ShortBuffer

使用Buffer读写数据一般遵循以下四个步骤：

1. 写入数据到Buffer
2. 调用`flip()`方法
3. 从Buffer中读取数据
4. 调用`clear()`方法或者`compact()`方法

Buffer类维护了4个核心变量属性来提供**关于其所包含的数组的信息**。它们是：

- 容量Capacity：缓冲区能够容纳的数据元素的最大数量。容量在缓冲区创建时被设定，并且永远不能被改变。(不能被改变的原因也很简单，底层是数组嘛)
- 上界Limit：缓冲区里的数据的总数，代表了当前缓冲区中一共有多少数据。
- 位置Position：下一个要被读或写的元素的位置。Position会自动由相应的 `get( )`和 `put( )`函数更新。
- 标记Mark：一个备忘位置。**用于记录上一次读写的位置**。

向缓冲区写数据，直接实例化对象并put即可，从缓冲区读数据需要先使用flip()切换到读模式然后使用get()读取缓冲区数据，flip的作用为将position切换到缓冲区开始位置，将limit切换为实际数据大小，读模式中读取完数据之后想要重写进入写模式必须调用clear()声明清空缓冲区（可看做进入写模式），更具体的：

**1.分配缓冲区**

```java
ByteBuffer buf = ByteBuffer.allocate(28);//以ByteBuffer为例子
```

**2.向缓冲区当中写数据**

```java
int bytesRead = inChannel.read(buf); //从通道当中读取数据到缓冲区
buf.put(127);//直接写入
```

**3.从缓冲区当中读取数据**

读取前先调用flip()切换到读模式

```java
int bytesWritten = inChannel.write(buf);//从缓冲区中读取数据到通道
byte aByte = buf.get();//直接读取
```

读取之后要调用以下几种方法：

* rewind()：设置position为0，可以从头开始读
* clear()：Buffer被清空，但数据并未清除
* compact()： 将所有未读的数据拷贝到Buffer起始处。然后将position设到最后一个未读元素正后面。limit属性依然像clear()方法一样，设置成capacity。现在Buffer准备好写数据了，但是不会覆盖未读的数据。 
* mqrk()/reset()： 通过调用Buffer.mark()方法，可以标记Buffer中的一个特定position。之后可以通过调用Buffer.reset()方法恢复到这个position 

**Selector**

允许单线程处理多个Channel，要使用Selector，得向Selector注册Channel，然后调用它的select()方法。

 Selector（选择器）是Java NIO中能够检测一到多个NIO通道，并能够知晓通道是否为诸如读写事件做好准备的组件。这样，一个单独的线程可以管理多个channel，从而管理多个网络连接 

**1.创建一个selector**

```java
Selector selector = Selector.open();
```

**2.向Selector注册通道**

```java
channel.configureBlocking(false);
SelectionKey key = channel.register(selector,Selectionkey.OP_READ);
```

与Selector一起使用时，Channel必须处于非阻塞模式下。这意味着不能将FileChannel与Selector一起使用，因为FileChannel不能切换到非阻塞模式。而套接字通道都可以。

注意register()方法的第二个参数。这是一个“interest集合”，意思是在通过Selector监听Channel时对什么事件感兴趣。通道触发了一个事件意思是该事件已经就绪。

这四种事件用SelectionKey的四个常量来表示：

1. SelectionKey.OP_CONNECT  某个channel成功连接到另一个服务器称为“连接就绪”。
2. SelectionKey.OP_ACCEPT  一个server socket channel准备好接收新进入的连接称为“接收就绪”。
3. SelectionKey.OP_READ    一个有数据可读的通道可以说是“读就绪”。
4. SelectionKey.OP_WRITE 等待写数据的通道可以说是“写就绪”。

如果你对不止一种事件感兴趣，那么可以用“位或”操作符将常量连接起来，如下：

```java
int interestSet = SelectionKey.OP_READ | SelectionKey.OP_WRITE;`
```

 register()方法会返回一个SelectionKey对象。这个对象包含了一些属性： 

* interest集合：通过该集合可确定某个事件是否在集合中

```java
	boolean isInterestedInConnect = interestSet & SelectionKey.OP_CONNECT == SelectionKey.OP_ACCEPT;
```

* ready集合：判断通道是否已经准备就绪

```java
selectionKey.isAcceptable();
selectionKey.isConnectable();
selectionKey.isReadable();
selectionKey.isWritable();
```

* 通过SelectionKey访问Channel与Selector

```java
Channel  channel  = selectionKey.channel();
Selector selector = selectionKey.selector();
```

* 可附加额外的对象

```java
selectionKey.attach(theObject);
Object attachedObj = selectionKey.attachment();
//还可以在用register()方法向Selector注册Channel的时候附加对象。如：
SelectionKey key = channel.register(selector, SelectionKey.OP_READ, theObject);
```

**3.通过Selector选择通道**

一旦向Selector注册了一或多个通道，就可以调用几个重载的select()方法。这些方法返回你所感兴趣的事件（如连接、接受、读或写）已经准备就绪的那些通道。换句话说，如果你对“读就绪”的通道感兴趣，select()方法会返回读事件已经就绪的那些通道。

下面是select()方法：

- int select() `select()`阻塞到至少有一个通道在你注册的事件上就绪了。
- int select(long timeout) `select(long timeout)`和select()一样，除了最长会阻塞timeout毫秒(参数)。
- int selectNow() `selectNow()`不会阻塞，不管什么通道就绪都立刻返回

select()方法返回的int值表示有多少通道已经就绪。亦即，自上次调用select()方法后有多少通道变成就绪状态。即并不返回所有就绪通道数量，而返回自上次调用select()后已就绪的通道数量

* selectedKeys()

一旦调用了select()方法，并且返回值表明有一个或更多个通道就绪了，然后可以通过调用selector的selectedKeys()方法，访问“已选择键集（selected key set）”中的就绪通道。如下所示：

```java
Set selectedKeys = selector.selectedKeys();
```

当像Selector注册Channel时，Channel.register()方法会返回一个SelectionKey 对象。这个对象代表了注册到该Selector的通道。可以通过SelectionKey的selectedKeySet()方法访问这些对象。

可以遍历这个已选择的键集合来访问就绪的通道。如下：

```java
Set selectedKeys = selector.selectedKeys();
Iterator keyIterator = selectedKeys.iterator();
while(keyIterator.hasNext()) {
	SelectionKey key = keyIterator.next();
	if(key.isAcceptable()) {
	// a connection was accepted by a ServerSocketChannel      
    } else if (key.isConnectable()) { 
		// a connection was established with a remote server.      
    } else if (key.isReadable()) {
		// a channel is ready for reading
	} else if (key.isWritable()) { 
		// a channel is ready for writing
	}
	keyIterator.remove();
}
```

这个循环遍历已选择键集中的每个键，并检测各个键所对应的通道的就绪事件。

注意每次迭代末尾的keyIterator.remove()调用。Selector不会自己从已选择键集中移除SelectionKey实例。必须在处理完通道时自己移除。下次该通道变成就绪时，Selector会再次将其放入已选择键集中。

SelectionKey.channel()方法返回的通道需要转型成你要处理的类型，如ServerSocketChannel或SocketChannel等。

**wakeUp()**

某个线程调用select()方法后阻塞了，即使没有通道已经就绪，也有办法让其从select()方法返回。只要让其它线程在第一个线程调用select()方法的那个对象上调用Selector.wakeup()方法即可。阻塞在select()方法上的线程会立马返回。

如果有其它线程调用了wakeup()方法，但当前没有线程阻塞在select()方法上，下个调用select()方法的线程会立即“醒来（wake up）”。

**close()**

用完Selector后调用其close()方法会关闭该Selector，且使注册到该Selector上的所有SelectionKey实例无效。通道本身并不会关闭。

**Path**

nio提供Path,表示文件系统，很多情况下，path可用于替代原来的File，另有一个Paths提供了操控Path的许多静态方法，打开文件为：

```java
Path path = Paths.get("c:\\data\\myfile.txt");
```

在path中可以使用. 与 ..

### NIO应用

在网络IO方面，传统的IO为同步阻塞IO模型，主线程阻塞等待socket连接请求，新的连接分配线程并直接交给线程池

```java
{
 ExecutorService executor = Excutors.newFixedThreadPollExecutor(100);//线程池

 ServerSocket serverSocket = new ServerSocket();
 serverSocket.bind(8088);
 while(!Thread.currentThread.isInturrupted()){//主线程死循环等待新连接到来
 Socket socket = serverSocket.accept();
 executor.submit(new ConnectIOnHandler(socket));//为新的连接创建新的线程
}

class ConnectIOnHandler extends Thread{
    private Socket socket;
    public ConnectIOnHandler(Socket socket){
       this.socket = socket;
    }
    public void run(){
      while(!Thread.currentThread.isInturrupted()&&!socket.isClosed()){死循环处理读写事件
          String someThing = socket.read()....//读取数据
          if(someThing!=null){
             ......//处理数据
             socket.write()....//写数据
          }

      }
    }
}
```

这就是移动端/客户端的传统BIO模型，但每一个socket都分配一个线程，资源消耗很高

NIO即同步非阻塞模型，单线程轮询IO事件，有则执行事件。 即NIO由原来的阻塞读写（占用线程）变成了单线程轮询事件，找到可以进行读写的网络描述符进行读写。除了事件的轮询是阻塞的（没有可干的事情必须要阻塞），剩余的I/O操作都是纯CPU操作，没有必要开启多线程。 

```java
 interface ChannelHandler{
      void channelReadable(Channel channel);
      void channelWritable(Channel channel);
   }
   class Channel{
     Socket socket;
     Event event;//读，写或者连接
   }

   //IO线程主循环:
   class IoThread extends Thread{
   public void run(){
   Channel channel;
   while(channel=Selector.select()){//选择就绪的事件和对应的连接
      if(channel.event==accept){
         registerNewChannelHandler(channel);//如果是新连接，则注册一个新的读写处理器
      }
      if(channel.event==write){
         getChannelHandler(channel).channelWritable(channel);//如果可以写，则执行写事件
      }
      if(channel.event==read){
          getChannelHandler(channel).channelReadable(channel);//如果可以读，则执行读事件
      }
    }
   }
   Map<Channel，ChannelHandler> handlerMap;//所有channel的对应事件处理器
  }
```

 ![img](https://pic2.zhimg.com/v2-90aab8868e33f22bc95f0d2706324cad_b.jpg) 

### AIO

异步非阻塞模型，jdk7引入，基于事件监听与回调

### 类加载机制与ClassLoader

类加载器ClassLoader即用于加载其它类的类，将字节码加载进内存，创建Class对象，输入完全限定的类名，输出Class对象。

ClassLoader分三类：

* 启动类加载器Bootstrap ClassLoader：Java虚拟机的一部分，使用C++实现，负责加载java的基础类，主要是<Java_Home\>/lib/rt.jar
* 扩展类加载器Extension ClassLoader（java9已删除，新增Platform Class Loader）：负责加载java的扩展类，主要是<Java_Home\>/lib/ext里的jar包
* 应用程序类加载器Application ClassLoader：负责加载应用程序类。

#### 双亲委派模型

Bootstrap ClassLoader→Extension ClassLoader→Application ClassLoader，后面的类有一个指针parent指向前面的类，ClassLoader在加载一个类时，基本步骤为：

1. 判断该类是否被加载过，有则直接返回Class对象，没有则被ClassLoader加载一次
2. 加载时，先让指针parent指向的“父”ClassLoader进行加载，加载成功返回Class对象
3. 加载不成功，自己尝试加载

该步骤所描述的模型即双亲委派模型，优先让父ClassLoader去加载。

这样做的原因是可以避免java类库被覆盖的问题，优先让父ClassLoader加载，避免自定义的类覆盖java类库，确保java安全机制，即使自定义ClassLoader可以不遵从双亲委派模型，但java安全机制保证以java开头的类不被其它类加载器加载。

#### ClassLoader

ClassLoader是一个抽象类，提供将字节码（class文件）加载至内存成为Class的基本方法，每一个Class都有一个基本方法 ：

* ClassLoader getClassLoader() ：获取该Class的实际类加载器

默认情况下，上述三类类加载器都有默认实现，Bootstrap ClassLoader用C++实现，Extension ClassLoader与

Application ClassLoader的实现位于sun.misc.Launcher中。

ClassLoader的基本方法为：

- ClassLoader getParent()：获取父ClassLoader，如果是Bootstrap ClassLoader则返回null
- static ClassLoader getSystemClassLoader()：获取系统默认ClassLoader
- Class<?\> loadClass(String name)：给定类的完全限定名，解析字节码，将类加载进内存并返回Class

#### Class.forName()

Class的静态方法forName()同样用于加载类进入内存并返回Class对象，区别在于Class.forName加载时可执行static静态代码块，而ClassLoader的loadClass不会执行，Class.forName方法可指定参数boolean initialize决定是否执行，ClassLoader的loadClass内部调用方法loadClass(name,false)，这个方法内部调用findClass(name)，是真正的类加载方法，第二个参数名为resolve，为true时才链接去执行static语句块。由于双亲委派模型，即使设置为true，父类传递的仍然是false。

#### ClassLoader应用

##### 读取文件加载类

广泛应用于各种框架中，使用ClassLoader，可通过读取配置文件直接加载Class进入内存，这样在面向接口编程中不用使用new 声明具体的实现，通过加载配置文件加载指定的类并返回实例，不动代码动配置，可以帮助完成依赖注入这个抽象概念的一部分实现。

1. 自定义一个类（或许是一个bean）

```java
public class HelloWorld {
    public static void say(){
        System.out.println("HelloWorld");
    }
}
```

2. 定义配置问件test.properties

```properties
#注意路径要正确，当前包下，也可以是其它包
word=com.lsl.kennen.common.HelloWorld 
```

3. 使用ClassLoader

```java
    public static void main(String[] args) {
        try {
            //读取配置文件，注意文件名路径要正确
            Properties properties=new Properties();
            String fileName=new File("").getAbsolutePath()+ "/kennen/src/main/java/com/lsl/kennen/common/test.properties";
            properties.load(new FileInputStream(fileName));
            //从配置文件中获取源数据
            String className =properties.getProperty("word");
            //根据信息加载类进入内存返回Class
            Class<?> cls=Class.forName(className);
            //获取实例
            HelloWorld helloWorld=(HelloWorld) cls.newInstance();
            helloWorld.say();
        }catch (Exception e){
            e.printStackTrace();
        }
    }
//结果为HelloWorld
```

使用ClassLoader加载类，读取配置文件动态将Class加载进内存

##### 自定义ClassLoader

自定义ClassLoader，利用defineClass方法直接返回一个Class<?>，可以远程调用Class的bytes读取Class文件将其加载进入本地内存并执行，这种功能就不是本地类加载器能办到的（没办法new，没办法forName，loadClass，因为读取的都是本地类的路径名加载本地bytes）。同时，面向接口编程时，可以动态加载calss,两个class，使用不同的ClassLoader加载，JVM就认为他们加载的对象是不同的，可以实现隔离与热部署。

自定义ClassLoader：继承ClassLoader，重写findClass，返回一个Class<?>

需要注意的是：

**方法defineClass()用于实际读取bytes()返回Class<?>，然而指定的类名必须符合规范，如com.lsl.xxx这样的，内部类则是com.lsl.xxx$Iservice这样的**

**因为是通过反射创建实例，所以对象的构造器不能设置为private，否则抛出异常。**

首先，先定义一个Class:

```java
public class HelloWorld{
    public void say(){
        System.out.println("HelloWorld");
    }
}
```

然后重写类加载器：

```java
public class Main {

    public static void main(String[] args) {
        //注意路径
        String className=new File("").getAbsolutePath()+ "/kennen/target/classes/com/lsl/kennen/common/HelloWorld.class";
        ClassLoader classLoader=new MyClassLoader();
        try {
            Class<?> c=classLoader.loadClass(className);
            ((HelloWorld) c.newInstance()).say();
        }catch (Exception e){
            e.printStackTrace();
        }
    }
    //重写ClassLoader
    static class MyClassLoader extends ClassLoader{
        @Override
        protected Class<?> findClass(String name) throws ClassNotFoundException{
            //本地测试，指定从本地读取class
            FileInputStream fileInputStream;//指定源
            ByteArrayOutputStream outputStream=new ByteArrayOutputStream();//指定目的
            try{
                fileInputStream=new FileInputStream(name);
                byte[] buf=new byte[1024];
                int ite=0;
                while((ite=fileInputStream.read(buf))!=-1){//从文件源中读取数据写入到buf
                    outputStream.write(buf,0,ite);//从buf中读取数据写入到outputStream
                }
            }catch (Exception e){
                e.printStackTrace();
            }
            return defineClass("com.lsl.kennen.common.HelloWorld",outputStream.toByteArray(),0,outputStream.toByteArray().length);//加载class文件的byte数组(bytes)，返回Class<?>
        }
    }
}
```

然而直接这样写会抛出异常，重写的ClassLoader将指定的class文件加载为Class，加载至内存，返回Class<?>，使用反射强制类型转换时如果直接强转为一个类会抛出异常，因为本地类没有没加载，就算加载了，使用的不是一个类加载器，加载出来的对象JVM是不认为相同的，这个时候就只能使用接口，可以强转为接口，只要加载的类实现了接口那么就可以强转为接口，调用接口方法。

定义接口：

```java
public interface World {
    public void say();
}
```

定义实现：

```java
public class HelloWorld implements World{
    @Override
    public void say(){
        System.out.println("HelloWorld");
    }
}
```

将 ((HelloWorld) c.newInstance()).say();改为 ((World) c.newInstance()).say();

运转正常，World恢复。

##### 热部署

热部署即在不停止程序运行的情况下动态更改Class，只要修改了Class就能立刻看出变化。

使用自定义ClassLoader，因为不同的类加载器可以加载出不同的Class，因此利用这一特性即可以实现模块隔离，也可以实现热部署。

定义服务实现：

```java
public class HelloWorld implements World{
    private static volatile World helloWorld;

    public static World getHelloWorld(){
        if(helloWorld==null){
            synchronized (HelloWorld.class){
                helloWorld=createHelloWorld();
            }
        }
        return helloWorld;
    }

    public static World createHelloWorld(){
        try{
            MyClassLoader myClassLoader=new MyClassLoader();
            String className=new File("").getAbsolutePath()+ "/kennen/target/classes/com/lsl/kennen/common/HelloWorld.class";
            Class<?> c=myClassLoader.loadClass(className);
            return ((World)c.newInstance());
        }catch (Exception e){
            e.printStackTrace();
        }
       return null;
    }

    public static void update(){
        helloWorld=createHelloWorld();
    }

    @Override
    public void say(){
        System.out.println("HelloWorld");
    }
}
```

服务内部实现内部维护一个接口World，提供服务功能，实际上是通过自定义的ClassLoader加载的Class返回的实例

类加载器为：

```java
public class MyClassLoader extends ClassLoader{

    @Override
    protected Class<?> findClass(String name) throws ClassNotFoundException{
        //本地测试，指定从本地读取class
        FileInputStream fileInputStream;//指定源
        ByteArrayOutputStream outputStream=new ByteArrayOutputStream();//指定目的
        try{
            fileInputStream=new FileInputStream(name);
            byte[] buf=new byte[1024];
            int ite=0;
            while((ite=fileInputStream.read(buf))!=-1){//从文件源中读取数据写入到buf
                outputStream.write(buf,0,ite);//从buf中读取数据写入到outputStream
            }
        }catch (Exception e){
            e.printStackTrace();
        }
        return defineClass("com.lsl.kennen.common.HelloWorld",outputStream.toByteArray(),0,outputStream.toByteArray().length);//加载class文件的byte数组(bytes)，返回Class<?>
    }

}
```

模拟热部署，定义两个线程，一个不断访问服务，一个不断检测字节码问件是否改变以确定是否更新服务

```java
public class Main {

    public static void main(String[] args) {

        Thread thread=new Thread(new Runnable() {
            @Override
            public void run() {
                while (true){
                    World world=HelloWorld.getHelloWorld();
                    world.say();
                    try {
                        Thread.sleep(1000);
                    }catch (Exception e){
                        e.printStackTrace();
                    }
                }
            }
        });

        Thread thread1=new Thread(new Runnable() {
            private long lastModilied=new File(new File("").getAbsolutePath()+ "/kennen/target/classes/com/lsl/kennen/common/HelloWorld.class").lastModified();
            @Override
            public void run() {
                while (true){
                    try{
                        Thread.sleep(100);
                        //根据时间差判断文件是否被修改
                        long now=new File(new File("").getAbsolutePath()+ "/kennen/target/classes/com/lsl/kennen/common/HelloWorld.class").lastModified();
                        if(now!=lastModilied){
                            //由时间差更新内部服务，完成热部署，动态变更执行的Class
                            lastModilied=now;
                            HelloWorld.update();
                        }
                    }catch (Exception e){
                        e.printStackTrace();
                    }
                }
            }
        });
        thread1.setDaemon(true);

        thread.start();
        thread1.start();
    }

}
```

当然直接运行时，没办法直接改Class文件，预先编译好Class，在运行时进行替换即可看到变化

#### URLClassLoader

对于远程加载Class，可以直接通过URLClassLoader加载，位于java.net包下，同样的，因为基于ClassLoader，所以一定要面向接口编程，具体的类强转是行不通的。参见： [locez.com/JAVA/urlcla…](https://locez.com/JAVA/urlclassloader-class/)

### 集合Collection

#### Collection与Collections

Collection是接口，接口Set与List都继承自该接口

Collections是java.util下的一个专用静态类，包含各种有关集合操作的静态方法，如排序搜索等

与之类似关系的有Array与Arrays

#### HashMap1.7/1.8

底部使用hash算法，维护hash表（数组/hash桶数组），使用链地址法解决冲突，内部维护变量

```java
 int threshold;             // 所能容纳的key-value对极限 
 final float loadFactor;    // 负载因子
 int modCount;  
 int size;
```

threshold=length*loadFactory，length为hash数组长度，也就是键值对数目由hash数组长度与负载因子决定,loadFactor常取0.75。即使用数值+单向链表

 在HashMap中，哈希桶数组table的长度length大小必须为2的n次方(一定是合数)，这是一种非常规的设计，常规的设计是把桶的大小设计为素数。相对来说素数导致冲突的概率要小于合数 ，因为分布情况由因数决定，冲突时进行取模运算，合数的因子比素数的因子多，因此对素数取模能使分布更加均匀。

put时取hashCode进行hash运算确定hash桶位置，有key则覆盖，没有则插入，插入成功判断容量，容量满则扩容，get同理，扩容时使用新数组覆盖旧数组与所有元素

jdk1.8进行了优化，当链表节点数量超过8是则使用红黑树进行替代（总数大于64），1.7在链表头部插入，1.8在尾部插入。线程不安全

#### HashSet

HashSet底层基于HashMap实现，HashSet的值存放于HashMap的key上，value为Parent（new Object()）



























 Buffer是缓冲区的抽象类 ，put()向缓冲区写数据，get()从缓冲区读数据







```java
        ByteBuffer buffer=ByteBuffer.allocate(1024);
        System.out.println("初始capacity:"+buffer.capacity());
        System.out.println("初始limit:"+buffer.limit());
        System.out.println("初始position:"+buffer.position());

        buffer.put("lslwind".getBytes());//向缓冲区加入byte数组
        System.out.println("put后capacity:"+buffer.capacity());
        System.out.println("put后limit:"+buffer.limit());
        System.out.println("put后position:"+buffer.position());

        buffer.put("疾风剑豪".getBytes());
        System.out.println("再次put后capacity:"+buffer.capacity());
        System.out.println("再次put后limit:"+buffer.limit());
        System.out.println("再次put后position:"+buffer.position());

        buffer.flip();//切换为读模式
        System.out.println("flip后capacity:"+buffer.capacity());
        System.out.println("flip后limit:"+buffer.limit());
        System.out.println("flip后position:"+buffer.position());

        byte[] bytes=new byte[buffer.limit()];
        buffer.get(bytes);
        System.out.println("读取数据"+new String(bytes));//从缓冲区读数据
        System.out.println("get后capacity:"+buffer.capacity());
        System.out.println("get后limit:"+buffer.limit());
        System.out.println("get后position:"+buffer.position());
```

结果为:

```
初始capacity:1024
初始limit:1024
初始position:0
put后capacity:1024
put后limit:1024
put后position:7
再次put后capacity:1024
再次put后limit:1024
再次put后position:19
flip后capacity:1024 
flip后limit:19
flip后position:0
读取数据lslwind疾风剑豪
get后capacity:1024
get后limit:19
get后position:19
```

 Channel通道**只负责传输数据、不直接操作数据**。操作数据都是通过Buffer缓冲区来进行操作，因此通道必须有数据源与目的地





