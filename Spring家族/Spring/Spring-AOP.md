## AOP

软件开发中，散布于应用中多处的功能被称为横切关注点（cross-cutting concern）。通常来讲，这些横切关注点从概念上是与应用的业务逻辑相分离的（但是往往会直接嵌入到应用的业务逻辑之中）。把这些横切关注点与业务逻辑相分离正是面向切面编程（AOP）所要解决的问题。DI有助于应用对象之间的解耦，而AOP可以实现横切关注点与它们所影响的对象之间的解耦。

<img src="https://s2.ax1x.com/2020/02/17/3CAQZ8.png" alt="3CAQZ8.png" style="zoom:67%;" />

在使用面向切面编程时，我们仍然在一个地方定义通用功能，但是可以通过声明的方式定义这个功能要以何种方式在何处应用，而无需修改受影响的类。横切关注点可以被模块化为特殊的类，这些类被称为切面（aspect）。这样做有两个好处：首先，现在每个关注点都集中于一个地方，而不是分散到多处代码中；其次，服务模块更简洁，因为它们只包含主要关注点（或核心功能）的代码，而次要关注点的代码被转移到切面中了。

[<img src="https://s2.ax1x.com/2020/02/17/3CAJRs.png" alt="3CAJRs.png" style="zoom: 67%;" />](https://imgchr.com/i/3CAJRs)

通过在代理类中包裹切面，Spring在运行期把切面织入到Spring管理的bean中。代理类封装了目标类，并拦截被通知方法的调用，再把调用转发给真正的目标bean。当代理拦截到方法调用时，在调用目标bean方法之前，会执行切面逻辑。

[<img src="https://s2.ax1x.com/2020/02/17/3CVmHf.png" alt="3CVmHf.png" style="zoom: 67%;" />](https://imgchr.com/i/3CVmHf)

Spring基于动态代理，所以Spring只支持方法连接点。这与一些其他的AOP框架是不同的，例如AspectJ和JBoss，除了方法切点，它们还提供了字段和构造器接入点。Spring缺少对字段连接点的支持，无法让我们创建细粒度的通知，例如拦截对象字段的修改。而且它不支持构造器连接点，我们就无法在bean创建时应用通知。

但是方法拦截可以满足绝大部分的需求。如果需要方法拦截之外的连接点拦截功能，那么我们可以利用Aspect来补充Spring AOP的功能。

### 术语

**通知（Advice）**

在AOP术语中，切面的工作被称为通知。

通知定义了切面是什么以及何时使用。除了描述切面要完成的工作，通知还解决了何时执行这个工作的问题。它应该应用在某个方法被调用之前？之后？之前和之后都调用？还是只在方法抛出异常时调用？

Spring切面可以应用5种类型的通知：

- 前置通知（Before）：在目标方法被调用之前调用通知功能；
- 后置通知（After）：在目标方法完成之后调用通知，此时不会关心方法的输出是什么；
- 返回通知（After-returning）：在目标方法成功执行之后调用通知；
- 异常通知（After-throwing）：在目标方法抛出异常后调用通知；
- 环绕通知（Around）：通知包裹了被通知的方法，在被通知的方法调用之前和调用之后执行自定义的行为。

**连接点（Join point）**

我们的应用可能也有数以千计的时机应用通知。这些时机被称为连接点。连接点是在应用执行过程中能够插入切面的一个点。这个点可以是调用方法时、抛出异常时、甚至修改一个字段时。切面代码可以利用这些点插入到应用的正常流程之中，并添加新的行为。

**切点（Poincut）**

一个切面并不需要通知应用的所有连接点。切点有助于缩小切面所通知的连接点的范围。

如果说通知定义了切面的“什么”和“何时”的话，那么切点就定义了“何处”。切点的定义会匹配通知所要织入的一个或多个连接点。我们通常使用明确的类和方法名称，或是利用正则表达式定义所匹配的类和方法名称来指定这些切点。有些AOP框架允许我们创建动态的切点，可以根据运行时的决策（比如方法的参数值）来决定是否应用通知。

**切面（Aspect）**

切面是通知和切点的结合。通知和切点共同定义了切面的全部内容——它是什么，在何时和何处完成其功能。

**引入（Introduction）**

引入允许我们向现有的类添加新方法或属性。例如，我们可以创建一个`Auditable`通知类，该类记录了对象最后一次修改时的状态。这很简单，只需一个方法，`setLastModified(Date)`，和一个实例变量来保存这个状态。然后，这个新方法和实例变量就可以被引入到现有的类中，从而可以在无需修改这些现有的类的情况下，让它们具有新的行为和状态。

**织入（Weaving）**

织入是把切面应用到目标对象并创建新的代理对象的过程。切面在指定的连接点被织入到目标对象中。在目标对象的生命周期里有多个点可以进行织入：

- 编译期：切面在目标类编译时被织入。这种方式需要特殊的编译器。AspectJ的织入编译器就是以这种方式织入切面的。

- 类加载期：切面在目标类加载到JVM时被织入。这种方式需要特殊的类加载器（`ClassLoader`），它可以在目标类被引入应用之前增强该目标类的字节码。AspectJ 5的加载时织入（load-time weaving，LTW）就支持以这种方式织入切面。
- 运行期：切面在应用运行的某个时刻被织入。一般情况下，在织入切面时，AOP容器会为目标对象动态地创建一个代理对象。Spring AOP就是以这种方式织入切面的。

通知包含了需要用于多个应用对象的横切行为；连接点是程序执行过程中能够应用通知的所有点；切点定义了通知被应用的具体位置（在哪些连接点）。其中关键的概念是切点定义了哪些连接点会得到通知。

### 编写切点

切点用于准确定位应该在什么地方应用切面的通知，在Spring AOP中，要使用AspectJ的切点表达式语言来定义切点(切点实际上就由切点表达式定义），因此要使用AspectJ特有的表达式语言来进行AOP。为了阐述Spring中的切面，我们需要有个主题来定义切面的切点。为此，我们定义一个`Performance`接口：

```java
package concert;
public interface Performance {
  public void perform();
}
```

**指示器execcution**

假设我们想编写`Performance`的`perform()`方法触发的通知。`execution`用于匹配切点，这个表达式能够设置当perform()方法执行时触发通知的调用。

[<img src="https://s2.ax1x.com/2020/02/17/3CZKqx.md.png" alt="3CZKqx.md.png" style="zoom:67%;" />](https://imgchr.com/i/3CZKqx)

我们使用`execution()`指示器选择`Performance`的`perform()`方法。方法表达式以“`*`”号开始，表明了我们不关心方法返回值的类型。然后，我们指定了全限定类名和方法名。对于方法参数列表，我们使用两个点号（`..`）表明切点要选择任意的`perform()`方法，无论该方法的入参是什么。

**指示器within**

现在假设我们需要配置的切点仅匹配`concert`包。在此场景下，可以使用`within()`指示器来限制匹配

<img src="https://s2.ax1x.com/2020/02/17/3Cei6A.png" alt="3Cei6A.png" style="zoom:67%;" />

使用“`&&`”操作符把`execution()`和`within()`指示器连接在一起形成与（and）关系（切点必须匹配所有的指示器）。类似地，我们可以使用“`||`”操作符来标识或（or）关系，而使用“`!`”操作符来标识非（not）操作。

因为“`&`”在XML中有特殊含义，所以在Spring的XML配置里面描述切点时，我们可以使用`and`来代替“`&&`”。同样，`or`和`not`可以分别用来代替“`||`”和“`!`”。

**指示器bean**

`bean()`指示器允许我们在切点表达式中使用bean的ID来标识bean。`bean()`使用bean ID或bean名称作为参数来限制切点只匹配特定的bean。

```
execution(* concert.Performance.perform())
        and bean('woodstock')
```

在这里，我们希望在执行`Performance`的`perform()`方法时应用通知，但限定bean的ID为`woodstock`。

在某些场景下，限定切点为指定的bean或许很有意义，但我们还可以使用非操作为除了特定ID以外的其他bean应用通知：

```
execution(* concert.Performance.perform())
        and !bean('woodstock')
```

在此场景下，切面的通知会被编织到所有ID不为`woodstock`的bean中。

### 编写切面

使用AspecJ注解来定义切面。

[<img src="https://s2.ax1x.com/2020/02/17/3CmTM9.png" alt="3CmTM9.png" style="zoom:67%;" />](https://imgchr.com/i/3CmTM9)

`Audience`类使用`@AspectJ`注解进行了标注。该注解表明`Audience`不仅仅是一个POJO，还是一个切面。`Audience`类中的方法都使用注解来定义切面的具体行为。

`Audience`有四个方法，定义了一个观众在观看演出时可能会做的事情。在演出之前，观众要就坐（`takeSeats()`）并将手机调至静音状态（`silenceCellPhones()`）。如果演出很精彩的话，观众应该会鼓掌喝彩（`applause()`）。不过，如果演出没有达到观众预期的话，观众会要求退款（`demandRefund()`）。

可以看到，这些方法都使用了通知注解来表明它们应该在什么时候调用。AspectJ提供了五个注解来定义通知

| 注　　解          | 通　　知                                 |
| ----------------- | ---------------------------------------- |
| `@After`          | 通知方法会在目标方法返回或抛出异常后调用 |
| `@AfterReturning` | 通知方法会在目标方法返回后调用           |
| `@AfterThrowing`  | 通知方法会在目标方法抛出异常后调用       |
| `@Around`         | 通知方法会将目标方法封装起来             |
| `@Before`         | 通知方法会在目标方法调用之前执行         |

**@Pointcut复用切点表达式**

对于重复的切点表达式可以使用注解@Pointcut进行复用

[<img src="https://s2.ax1x.com/2020/02/17/3CMfm9.png" alt="3CMfm9.png" style="zoom:67%;" />](https://imgchr.com/i/3CMfm9)

`performance()`方法的实际内容并不重要，在这里它实际上应该是空的。其实该方法本身只是一个标识，供`@Pointcut`注解依附。

需要注意的是，除了注解和没有实际操作的`performance()`方法，`Audience`类依然是一个POJO。我们能够像使用其他的Java类那样调用它的方法，它的方法也能够独立地进行单元测试，这与其他的Java类并没有什么区别。`Audience`只是一个Java类，只不过它通过注解表明会作为切面使用而已，像其他的Java类一样，它可以装配为Spring中的bean：

```java
@Bean
public Audience audience() {
    return new Audience();
}
```

如果你就此止步的话，`Audience`只会是Spring容器中的一个bean。即便使用了AspectJ注解，但它并不会被视为切面，这些注解不会解析，也不会创建将其转换为切面的代理。

**声明开启AOP**

使用JavaConfig的话，可以在配置类的类级别上通过使用`EnableAspectJ-AutoProxy`注解启用自动代理功能。

[<img src="https://s2.ax1x.com/2020/02/17/3CMblD.png" alt="3CMblD.png" style="zoom:80%;" />](https://imgchr.com/i/3CMblD)

使用XML来装配bean的话，那么需要使用Spring `aop`命名空间中的`<aop:aspectj-autoproxy>`元素。

[![3Cl0P0.png](https://s2.ax1x.com/2020/02/17/3Cl0P0.png)](https://imgchr.com/i/3Cl0P0)

不管是使用JavaConfig还是XML，AspectJ自动代理都会为使用`@Aspect`注解的bean创建一个代理，这个代理会围绕着所有该切面的切点所匹配的bean。需要记住的是，Spring的AspectJ自动代理仅仅使用`@AspectJ`作为创建切面的指导，切面依然是基于代理的。在本质上，它依然是Spring基于代理的切面。这一点非常重要，因为这意味着尽管使用的是`@AspectJ`注解，但我们仍然限于代理方法的调用。如果想利用AspectJ的所有能力，我们必须在运行时使用AspectJ并且不依赖Spring来创建基于代理的切面。

**使用环绕通知**

环绕通知是最为强大的通知类型。它能够让你所编写的逻辑将被通知的目标方法完全包装起来。实际上就像在一个通知方法中同时编写前置通知和后置通知。

<img src="https://s2.ax1x.com/2020/02/17/3Cl6r4.png" alt="3Cl6r4.png" style="zoom:67%;" />

这里，`@Around`注解表明`watchPerformance()`方法会作为`performance()`切点的环绕通知。在这个通知中，观众在演出之前会将手机调至静音并就坐，演出结束后会鼓掌喝彩。像前面一样，如果演出失败的话，观众会要求退款。可以看到，这个通知所达到的效果与之前的前置通知和后置通知是一样的。但是，现在它们位于同一个方法中，不像之前那样分散在四个不同的通知方法里面。

使用环绕通知可以有返回值，用于进行灵活的切面

```java
    /**
      * 环绕通知需要携带ProceedingJoinPoint类型的参数
     * 环绕通知类似于动态代理的全过程：ProceedingJoinPoint类型的参数可以决定是否执行目标方法。
     * 而且环绕通知必须有返回值，返回值即为目标方法的返回值
      */
    @Around("execution(public int com.yl.spring.aop.ArithmeticCalculator.*(..))")
     public Object aroundMethod(ProceedingJoinPoint pjd) {
        Object result = null;
         String methodName = pjd.getSignature().getName();
        //执行目标方法
         try {
            //前置通知
            System.out.println("The method " + methodName + " begins with " + Arrays.asList(pjd.getArgs()));
           result = pjd.proceed();
           //返回通知
            System.out.println("The method " + methodName + " ends with " + Arrays.asList(pjd.getArgs()));
        } catch (Throwable e) {
            //异常通知
           System.out.println("The method " + methodName + " occurs expection : " + e);
            throw new RuntimeException(e);
         }
         //后置通知
         System.out.println("The method " + methodName + " ends");
         return result;
     }
```

### 处理通知中的参数

切面能访问和使用传递给被通知方法的参数吗，看一个样例。`play()`方法会循环所有的磁道并调用`playTrack()`方法。但是，我们也可以通过`playTrack()`方法直接播放某一个磁道中的歌曲。

假设你想记录每个磁道被播放的次数。一种方法就是修改`playTrack()`方法，直接在每次调用的时候记录这个数量。但是，记录磁道的播放次数与播放本身是不同的关注点，因此不应该属于`playTrack()`方法。看起来，这应该是切面要完成的任务。

为了记录每个磁道所播放的次数，我们创建了`TrackCounter`类，它是通知`playTrack()`方法的一个切面。

<img src="https://s2.ax1x.com/2020/02/17/3C1ROg.png" alt="3C1ROg.png" style="zoom: 80%;" />

**通过"argNames"属性指定参数名。**

```java
@Before(value="args(param)", argNames="param") //明确指定了 
public void beforeTest(String param) { 
    System.out.println("param:" + param); 
}
```

**指示器args**

需要关注的是切点表达式中的`args(trackNumber)`限定符。它表明传递给`playTrack()`方法的`int`类型参数也会传递到通知中去。参数的名称`trackNumber`也与切点方法签名中的参数相匹配。

这个参数会传递到通知方法中，这个通知方法是通过`@Before`注解和命名切点`trackPlayed(trackNumber)`定义的。切点定义中的参数与切点方法中的参数名称是一样的，这样就完成了从命名切点到通知方法的参数转移。

### 通过JoinPoint获取参数

Spring AOP提供使用org.aspectj.lang.JoinPoint类型获取连接点数据，任何通知方法的第一个参数都可以是JoinPoint(环绕通知是ProceedingJoinPoint，JoinPoint子类)，当然第一个参数位置也可以是JoinPoint.StaticPart类型，这个只返回连接点的静态部分。

1. `JoinPoint`：提供访问当前被通知方法的目标对象、代理对象、方法参数等数据：

   

   ```java
   public interface JoinPoint {
   
       String toString();                  //连接点所在位置的相关信息  
       String toShortString();             //连接点所在位置的简短相关信息  
       String toLongString();              //连接点所在位置的全部相关信息
       Object getThis();                   //返回AOP代理对象,如果想要使用这个方法的话，最好使用this连接点，这样可以获得最佳的性能。  
       Object getTarget();                 //返回目标对象,如果想要使用这个方法的话，最好使用target连接点，这样可以获得最佳的性能。
       Object[] getArgs();                 //返回被通知方法参数列表  
       Signature getSignature();           //返回当前连接点签名  
       SourceLocation getSourceLocation(); //返回连接点方法所在类文件中的位置  
       String getKind();                   //连接点类型  
       StaticPart getStaticPart();         //返回连接点静态部分  
   
       // getKind 方法的返回值
       static String METHOD_EXECUTION = "method-execution";
       static String METHOD_CALL = "method-call";
       static String CONSTRUCTOR_EXECUTION = "constructor-execution";
       static String CONSTRUCTOR_CALL = "constructor-call";
       static String FIELD_GET = "field-get";
       static String FIELD_SET = "field-set";
       static String STATICINITIALIZATION = "staticinitialization";
       static String PREINITIALIZATION = "preinitialization";
       static String INITIALIZATION = "initialization";
       static String EXCEPTION_HANDLER = "exception-handler";
       static String SYNCHRONIZATION_LOCK = "lock";
       static String SYNCHRONIZATION_UNLOCK = "unlock";
       static String ADVICE_EXECUTION = "adviceexecution"; 
   }
   ```

2. `ProceedingJoinPoint`：用于环绕通知，使用`proceed()`方法来执行目标方法：



```java
public interface ProceedingJoinPoint extends JoinPoint {  
    void set$AroundClosure(AroundClosure arc);              // 这是个内部方法，不应该直接调用
    public Object proceed() throws Throwable;               // 执行目标函数，以默认的参数执行
    public Object proceed(Object[] args) throws Throwable;  // 执行目标函数，并传入所需的参数
}  
```

1. `JoinPoint.StaticPart`：提供访问连接点的静态部分，如被通知方法签名、连接点类型等：



```java
public interface StaticPart {
    Signature getSignature();           // 返回当前连接点签名
    SourceLocation getSourceLocation(); // 返回连接点所在资源路径
    String getKind();                   // 连接点类型
    int getId();                        // 唯一标识 
    String toString();                  // 连接点所在位置的相关信息
    String toShortString();             // 连接点所在位置的简短相关信息
    String toLongString();              // 连接点所在位置的全部相关信息
}
```

如果使用该种方式声明参数，必须放到方法第一个位置上，如



```java
@Aspect
@Component
public class JoinPointAop {

    // 切点范围
    @Pointcut("execution(* com.learn.service..IJoinPointService+.*(..))")
    public void pointcut(){ }

    @Before("pointcut()")
    public void before1(JoinPoint jp) {
        System.out.println("---------------@Before1----------------");
        System.out.println(Arrays.toString(jp.getArgs()));
    }

    @Before("pointcut()")
    public void before2(JoinPoint.StaticPart jp) {
        System.out.println("---------------@Before2----------------");
        System.out.println(jp.getSignature());
    }

    @Around("pointcut()")
    public Object around(ProceedingJoinPoint pjp) throws Throwable {
        System.out.println("---------------@Around----------------");
        return pjp.proceed();
    }

}
```



### 使用XML声明AOP

在Spring的`aop`命名空间中，提供了多个元素用来在XML中声明切面

<img src="https://s2.ax1x.com/2020/02/17/3C8ZKU.png" alt="3C8ZKU.png" style="zoom:67%;" />

`aop`命名空间的其他元素能够让我们直接在Spring配置中声明切面，而不需要使用注解。例如，我们重新看一下`Audience`类，这一次我们将它所有的AspectJ注解全部移除掉：

```java
package concert;
public class Audience {
  public void silenceCellPhones() {
    System.out.println("Silencing cell phones");
  }
  public void takeSeats() {
    System.out.println("Taking seats");
  }
  public void applause() {
    System.out.println("CLAP CLAP CLAP!!!");
  }
  public void demandRefund() {
    System.out.println("Demanding a refund");
  }
}
```

正如你所看到的，`Audience`类并没有任何特别之处，它就是有几个方法的简单Java类。我们可以像其他类一样把它注册为Spring应用上下文中的bean。尽管看起来并没有什么差别，但`Audience`已经具备了成为AOP通知的所有条件。我们再稍微帮助它一把，它就能够成为预期的通知了。

<img src="https://s2.ax1x.com/2020/02/17/3C8DRP.png" alt="3C8DRP.png" style="zoom:67%;" />





在`<aop:config>`元素内，我们可以声明一个或多个通知器、切面或者切点。使用`<aop:aspect>`元素声明了一个简单的切面。`ref`元素引用了一个POJO bean，该bean实现了切面的功能——在这里就是`audience`。`ref`元素所引用的bean提供了在切面中通知所调用的方法。

使用`<aop:pointcut>`可以复用切点表达式

<img src="https://s2.ax1x.com/2020/02/17/3C8LdJ.png" alt="3C8LdJ.png" style="zoom:67%;" />

### SpringBoot引入AOP

**引入依赖**

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```

**是否需要在程序主类中增加@EnableAspectJAutoProxy 注解**
答案是否。只要引入了AOP依赖后，默认已经增加了@EnableAspectJAutoProxy。

**实例**

```java
@Aspect
@Component
public class WebLogAspect {

    private Logger logger = Logger.getLogger(getClass());
     
    @Pointcut("execution(public * com.didispace.web..*.*(..))")
    public void webLog(){}
     
    @Before("webLog()")
    public void doBefore(JoinPoint joinPoint) throws Throwable {
        // 接收到请求，记录请求内容
        ServletRequestAttributes attributes = (ServletRequestAttributes) RequestContextHolder.getRequestAttributes();
        HttpServletRequest request = attributes.getRequest();
     
        // 记录下请求内容
        logger.info("URL : " + request.getRequestURL().toString());
        logger.info("HTTP_METHOD : " + request.getMethod());
        logger.info("IP : " + request.getRemoteAddr());
        logger.info("CLASS_METHOD : " + joinPoint.getSignature().getDeclaringTypeName() + "." + joinPoint.getSignature().getName());
        logger.info("ARGS : " + Arrays.toString(joinPoint.getArgs()));
     
    }
     
    @AfterReturning(returning = "ret", pointcut = "webLog()")
    public void doAfterReturning(Object ret) throws Throwable {
        // 处理完请求，返回内容
        logger.info("RESPONSE : " + ret);
    }

}
```

注：doBefore也可以这样写，doBefore(ProceedingJoinPoint point)，ProceedingJoinPoint 是JoinPoint实现子类且他俩都是接口。ProceedingJoinPoint是在JoinPoint的基础上暴露出 proceed 这个方法。proceed很重要，这个是aop代理链执行的方法。暴露出这个方法，就能支持 aop:around 这种切面（而其他的几种切面只需要用到JoinPoint，这跟切面类型有关）， 能决定是否走代理链还是走自己拦截的其他逻辑。


**防止二次代理**

5 @Pointcut 组合使用，切入点之间可以通过 && || ! 逻辑表达式进入组合使用。 

@Pointcut("execution(public * com.didispace.web..*.*(..))")
    public void a1(){}
 @Pointcut("execution(public * com.didispace.web..*.*(..))")
    public void a2(){}
 @Before("a1||a2")
 public void doBefore(JoinPoint joinPoint) throws Throwable {。。。。。}
拓展：

指定类上包含注解的作为切点，然后可以在切入点开始处(如@Before @after)做具体的处理，如判断是否标记了@service注解

@Pointcut("@annotation(com.aa.abc................)") 

包含指定注解的作为切点：所有标记了@ResponseBody 注解的作为切点

@Pointcut(value = "@annotation(org.springframework.web.bind.annotation.ResponseBody)")
public void responseBodyPointcut() {}

组合使用：指定类上，标注了指定注解的作为切点

6 指定代理方式jdk/cglib 在yml文件配置

spring:
  aop:
    proxy-target-class: false #jdk

## 常见错误

### 不可以创建Bean

最常见的错误就是切面的执行表达式写错了，代理时不能解析。



默认使用的是jdk代理，在SpringBoot中可以设置为使用cglib代理，在主Application中添加注解

```java
@EnableAspectJAutoProxy(proxyTargetClass = true)
```

