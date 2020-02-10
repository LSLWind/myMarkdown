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

### 