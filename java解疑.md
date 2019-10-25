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

#### 格式

public @interface 注解名称{
    属性列表;
}

#### 元注解

元注解，即注解的注解，定义一个注解实际上还是继承了接口Annotation,要想更方便使用注解必须约束注解的行为，即注解约束数据，元注解约束注解。

@Target() 用于约束注解的作用域，使用枚举ElementType选择,如果选择多个则使用{},如

    @Target({ElementType.METHOD,ElementType.FIELD})

@Retention()用于约束注解的保留时间段，有三个,使用RetentionPolicy选择，如果想通过反射获取注解信息则必须保留到运行时，即选择RUNTIME

     @Retention(RetentionPolicy.RUNTIME)

@Inherited() 约束注解是否可以被继承，默认不继承，即父类有注解，但子类没有，子类默认不继承注解

#### 本质

注解的本质是接口，所写的注解会被自动编译成接口interface并继承接口Annotation。

既然是接口，自然可以在内部声明抽象方法，但使用注解时可以给内部的方法名赋值，如

定义注解：

public @interface Lsl {
    int getAge();
}
使用时就可以直接这样写@Lsl(getAge = 18)，直接就对getAge进行赋值，因此getAge()看起来又是一个属性。
内部声明属性时可以设置默认值，如int getAge() default 18;

#### 使用

注解的核心在于注解所修饰的类/方法/字段可以通过反射读取其所绑定的注解信息。

因此注解的使用为:定义注解，使用注解，读取注解

读取注解就需要用到反射，通过反射获取注解信息，由注解信息知道约束条件从而可以进行动态检查

 Class<?>提供多种方法获取注解,getAnnotations()获取该Class对象上的所有注解，具体对象如Field也有getDeclaredAnnotations()获取注解,返回一个Annotation[]。如果是方法或者构造器因为参数也可以有注解，所以有方法getParameterAnnotations()获取参数的注解，返回一个Annotation[][],每一个参数对应一个Annotation[]。

读取注解之后因为注解携带信息，所以要把信息读取出来，因为注解的本质是接口，注解内部定义了方法，使用注解时直接复制，读取时直接调用方法即可，通过注解 注解对象 = 使用了该注解的XXX.getAnnotation(注解.class) 这样的形式返回注解对象，通过注解对象调用方法获取信息

#### 实例

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

#### 静态代理:

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

#### 动态代理:

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

#### cglib动态代理

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

#### JDK注解

#####   **@ConstructorProperties** 

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

因为运行时无法获得参数，通过该注解表明想，x,y可通过getX()与getY()获得

##### String不可变

实际内部封装字符数组private final char[] value，因此String不可变，截取substring()等方法返回的是新的String对象，使用+实际上扩充数组然后重新赋值

#### 常用方法

##### 拷贝数组

Arrays.copyOf(数组，长度)

Arrays.copyOf(数组，开始index，结束index)

返回对应数组][]

##### Set转List

①构造List时直接传参，参数类型为Collection

```java
List<String> list=new Arraylist<>(set);

```

②使用addAll()方法，参数类型为Collection

```java
list.addAll(set);

```

