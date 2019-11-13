 https://www.zhihu.com/question/23277575/answer/169698662 

## IOC/DI

一种仅通过构造器参数、工厂方法参数或由构造器/工厂方法产生的实例的属性定义依赖关系，由容器在创造bean的过程中注入依赖的过程。

 **所谓依赖注入，就是把底层类作为参数传入上层类，实现上层类对下层类的“控制**” 

 ![img](https://pic2.zhimg.com/80/v2-c920a0540ce0651003a5326f6ef9891d_hd.png) 

 ![img](https://pic3.zhimg.com/80/v2-c845802f9187953ed576e0555f76da42_hd.png) 

 ![img](https://pic4.zhimg.com/80/v2-555b2be7d76e78511a6d6fed3304927f_hd.png) 

   ![img](https://pic3.zhimg.com/80/v2-24a96669241e81439c636e83976ba152_hd.png) 

 ![img](https://pic2.zhimg.com/80/v2-5ca61395f37cef73c7bbe7808f9ea219_hd.png) 

`org.springframework.beans` and `org.springframework.context`  是IoC容器的基础，

 [`BeanFactory`](https://docs.spring.io/spring-framework/docs/5.2.0.RELEASE/javadoc-api/org/springframework/beans/factory/BeanFactory.html) 接口提供管理object的配置机制，  [`ApplicationContext`](https://docs.spring.io/spring-framework/docs/5.2.0.RELEASE/javadoc-api/org/springframework/context/ApplicationContext.html) 是其子类，提供更多特性。换句话说，BeanFactory提供基本功能与配置框架，ApplocationContext是其超集。

### 功能

实现了解耦，解决了bean的依赖问题

由ioc容器动态注入对象的依赖，管理bean生命周期，而不手动创建依赖对象并管理。

 ![img](https://pic4.zhimg.com/80/v2-dc8c6fc99a74e95cf6ff8066f421596f_hd.jpg) 

### IOC容器

  `org.springframework.context.ApplicationContext` 象征着IOC容器，负责beans的实例化，配置与管理。容器通过读取配置元数据(configuration metadata)获取bean的构造信息，配置元数据可以是XML，注解或者java代码。

 ![container magic](https://docs.spring.io/spring/docs/current/spring-framework-reference/images/container-magic.png) 



 [`ClassPathXmlApplicationContext`](https://docs.spring.io/spring-framework/docs/5.2.0.RELEASE/javadoc-api/org/springframework/context/support/ClassPathXmlApplicationContext.html) or [`FileSystemXmlApplicationContext`](https://docs.spring.io/spring-framework/docs/5.2.0.RELEASE/javadoc-api/org/springframework/context/support/FileSystemXmlApplicationContext.html) 是Spring提供的两个Application实例

即使用方式为：在resources下配置数据源xml文件→使用时读取配置文件，获取IOC容器，通过IOC容器与反射机制根据配置文件注入依赖并获取bean

#### 配置bean

IOC容器通过配置元数据读取要管理的bean信息，由元数据决定如何注入依赖，如果使用注解则需要在配置文件中加上

```xml
<context:annotation-config/>
```

使用标签<bean/\>与<beans\>定义bean

使用：

```java
// 创建IOC
ApplicationContext context = new ClassPathXmlApplicationContext("services.xml", "daos.xml");
// 获取bean
PetStoreService service = context.getBean("petStore", PetStoreService.class);
// 获取元数据
List<String> userList = service.getUsernameList();
```

##### **使用静态工厂**

```java
public class DefaultServiceLocator {

    private static ClientService clientService = new ClientServiceImpl();

    private static AccountService accountService = new AccountServiceImpl();

    public ClientService createClientServiceInstance() {
        return clientService;
    }

    public AccountService createAccountServiceInstance() {
        return accountService;
    }
}
```

对应的xml为：

```xml
<bean id="serviceLocator" class="examples.DefaultServiceLocator">
    <!-- inject any dependencies required by this locator bean -->
</bean>

<bean id="clientService"
    factory-bean="serviceLocator"<!--通过id引用配置依赖关系-->
    factory-method="createClientServiceInstance"/>

<bean id="accountService"
    factory-bean="serviceLocator"
    factory-method="createAccountServiceInstance"/>
```

##### 基于构造器

默认按照构造器中参数的顺序依次为bean注入属性，参数中的bean由ref指示，按照顺序注入依赖，使用标签< constructor-arg \>配置

```java
package x.y;

public class ThingOne {
    public ThingOne(ThingTwo thingTwo, ThingThree thingThree) {
        // ...
    }
}
```

```xml
<beans>
    <bean id="beanOne" class="x.y.ThingOne">
        <constructor-arg ref="beanTwo"/>
        <constructor-arg ref="beanThree"/>
    </bean>

    <bean id="beanTwo" class="x.y.ThingTwo"/>

    <bean id="beanThree" class="x.y.ThingThree"/>
</beans>
```

当构造器中的参数包含其它基类型时，使用value注入，或者直接根据index注入

```java
package examples;

public class ExampleBean {

    // Number of years to calculate the Ultimate Answer
    private int years;

    // The Answer to Life, the Universe, and Everything
    private String ultimateAnswer;

    public ExampleBean(int years, String ultimateAnswer) {
        this.years = years;
        this.ultimateAnswer = ultimateAnswer;
    }
}
```

```xml
<bean id="exampleBean" class="examples.ExampleBean">
    <constructor-arg type="int" value="7500000"/><!--通过指示type注入value，默认还是按照顺序的-->
    <constructor-arg type="java.lang.String" value="42"/>
</bean>
```

更通用的，可以使用index，index从0开始

```xml
<bean id="exampleBean" class="examples.ExampleBean">
    <constructor-arg index="0" value="7500000"/>
    <constructor-arg index="1" value="42"/>
</bean>
```

也可以通过参数名注入属性，要使用name则必须启动debug模式，或者使用注解 **@ConstructorProperties** 声明构造器参数，否则运行时无法确定构造器参数名

```xml
<bean id="exampleBean" class="examples.ExampleBean">
    <constructor-arg name="years" value="7500000"/>
    <constructor-arg name="ultimateAnswer" value="42"/>
</bean>
```

```java
    @ConstructorProperties({"years", "ultimateAnswer"})
    public ExampleBean(int years, String ultimateAnswer) {
        this.years = years;
        this.ultimateAnswer = ultimateAnswer;
    }
```

注意，基类型使用value注入，bean使用ref注入

也可以使用c-scheam简化，引入 xmlns:c="http://www.springframework.org/schema/c" 

参数属性值可以直接写，引用bean则在名后加-ref即可，如下所示：

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:c="http://www.springframework.org/schema/c"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="beanTwo" class="x.y.ThingTwo"/>
    <bean id="beanThree" class="x.y.ThingThree"/>

    <!-- traditional declaration with optional argument names -->
    <bean id="beanOne" class="x.y.ThingOne">
        <constructor-arg name="thingTwo" ref="beanTwo"/>
        <constructor-arg name="thingThree" ref="beanThree"/>
        <constructor-arg name="email" value="something@somewhere.com"/>
    </bean>

    <!-- c-namespace declaration with argument names -->
    <bean id="beanOne" class="x.y.ThingOne" c:thingTwo-ref="beanTwo"
        c:thingThree-ref="beanThree" c:email="something@somewhere.com"/>

</beans>
```



##### 基于setter

通过setter方法注入依赖，使用标签<property\>

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <!-- services -->
    <!--服务层实现依赖于DAO层对象-->
    <bean id="petStore" class="org.springframework.samples.jpetstore.services.PetStoreServiceImpl">
        <property name="accountDao" ref="accountDao"/>
        <property name="itemDao" ref="itemDao"/>
        <property name="id" value="001"/><!--Spring提供类型自动转换服务-->
    </bean>

    <bean id="accountDao"
          class="org.springframework.samples.jpetstore.dao.jpa.JpaAccountDao">
    </bean>

    <bean id="itemDao" class="org.springframework.samples.jpetstore.dao.jpa.JpaItemDao">
    </bean>

</beans>
```

在PetStoreServiceImpl中一定有对应的setter方法才行，区别就是使用的标签不同。对于属性值，property可以简写为p:id="001"，前提是引入 xmlns:p="http://www.springframework.org/schema/p" ，对于这种写法来说，没有模式定义，也就是直接使用p:bean属性名=value这种形式，对于引用属性来说，直接属性名-ref即可，如下所示：

```xml
 <bean name="john-modern"
        class="com.example.Person"
        p:name="John Doe"
        p:spouse-ref="jane"/><!--spouse为属性名-->

    <bean name="jane" class="com.example.Person">
        <property name="name" value="Jane Doe"/>
    </bean>
```

##### 依赖注入过程

1. ApplicationContext被创建并且初始化，读取配置元数据
2. 对于每个bean，其依赖(dependencies)以属性、构造器参数、静态工厂方法的方式被描述，创建bean时注入依赖
3. bean的每个依赖属性都要被实际定义

**循环依赖问题**

如果通过构造器注入可能导致循环依赖，类A需要类B作为构造器参数，类B需要类A作为构造器参数，因此循环构造依赖，解决办法就是使用setter方式注入依赖

##### 内联bean/inner Beans

既可以通过ref引用一个bean实例，也可以通过内联bean的方式。默认情况下，xml中配置的bean都是单例模式，即每个bean都引用同一个对象，inner Beans相当于配置匿名内部类

```xml
<bean id="outer" class="...">
    <!-- instead of using a reference to a target bean, simply define the target bean inline -->
    <property name="target">
        <bean class="com.example.Person"> <!-- this is the inner bean -->
            <property name="name" value="Fiona Apple"/>
            <property name="age" value="25"/>
        </bean>
    </property>
</bean>
```

##### 集合类型Collections

集合元素属性 list，set，map，properties分别用标签<list/\>,<set/\>,<map/\>,<pros/\>表示，集合中如果要使用bean则使用带ref的引用标签，下面给出部分示例：

```xml
<bean id="moreComplexObject" class="example.ComplexObject">
    <!-- results in a setAdminEmails(java.util.Properties) call -->
    <property name="adminEmails">
        <props>
            <prop key="administrator">administrator@example.org</prop>
            <prop key="support">support@example.org</prop>
            <prop key="development">development@example.org</prop>
        </props>
    </property>
    <!-- results in a setSomeList(java.util.List) call -->
    <property name="someList">
        <list>
            <value>a list element followed by a reference</value>
            <ref bean="myDataSource" />
        </list>
    </property>
    <!-- results in a setSomeMap(java.util.Map) call -->
    <property name="someMap">
        <map>
            <entry key="an entry" value="just some string"/>
            <entry key ="a ref" value-ref="myDataSource"/>
        </map>
    </property>
    <!-- results in a setSomeSet(java.util.Set) call -->
    <property name="someSet">
        <set>
            <value>just some string</value>
            <ref bean="myDataSource" />
        </set>
    </property>
</bean>
```

##### 集合合并 Collection Merging

对于子类来说，可与父类的集合合并，相同元素子类会覆盖父类

```xml
<beans>
    <bean id="parent" abstract="true" class="example.ComplexObject">
        <property name="adminEmails">
            <props>
                <prop key="administrator">administrator@example.com</prop>
                <prop key="support">support@example.com</prop>
            </props>
        </property>
    </bean>
    <bean id="child" parent="parent">
        <property name="adminEmails">
            <!-- the merge is specified on the child collection definition -->
            <props merge="true">
                <prop key="sales">sales@example.com</prop>
                <prop key="support">support@example.co.uk</prop>
            </props>
        </property>
    </bean>
<beans>
```

子类bean指定parent同时启动merge=true，无法合并不同类型（抛出异常），Spring会自动检查语法，因此基类型如int会自动转换而不会当做String类型处理，对于null来说使用标签<null/\>

#### 依赖相关

bean通过ref属性引用另一个bean的id表示依赖注入，但有时候依赖关系可能比较低，即出现间接依赖情况

##### depends-on/@DependsOn

用于控制bean的实例化顺序。 spring容器载入bean顺序是不确定的，spring框架没有约定特定顺序逻辑规范。但spring保证如果A依赖B(如beanA中有@Autowired B的变量)，那么B将先于A被加载。但如果beanA不直接依赖B，但在使用时要确保B已经被实例化，即依赖于B的某些实现，典型的如观察者模式，全局缓存访问等，可以通过depends-on或@DependsOn 约定B必须先于A实例化，以避免某些不好的影响。

假设bean C中有一个静态字段为全局参数

```java
public class SystemSettings{  
           //缓存更新时间  
           public static int REFRESH_CYCLE = 60;  
           ......  
}  
```

bean B用于在系统启动时更新C的值

```java
public class SystemInit{  
          public SystemInit(){  
                     //模拟从数据库中加载的系统参数配置值  
                     SystemSettings.REFRESH_CYCLE=100;  
                     ......  
          }  
}  
```

bean A直接使用C的值

```java
public class CacheManager{  
          public CacheManager(){                     
                     //缓存刷新定时处理  
                    t.schedule(new CacheTask() ,0,SystemSettings.REFRESH_CYCLE);  
          }  
          ......  
}  
```

逻辑上应该希望B先于A实例化，但spring实例化bean的顺序不确定，假如A先于B实例化，那么可能就访问到了不希望的值，出现某些意想不到的错误，但实际上还是要由业务功能确定，注意depends-on/@DependsOn用于控制bean的初始化顺序即可。

在xml中使用depends-on：

```xml
<bean id="beanOne" class="ExampleBean" depends-on="manager,accountDao">
    <property name="manager" ref="manager" />
</bean>

<bean id="manager" class="ManagerBean" />
<bean id="accountDao" class="x.y.jdbc.JdbcAccountDao" />
```

或在代码中使用@DependsOn：

**1：直接标注在带有@Component注解的类上面;**

**2：直接标注在带有@Bean 注解的方法上面;**

##### lazy-init/default-lazy-init

即设置bean为懒汉模式，因为默认情况下设置的bean都为单例模式，所以有两种初始化bean的方式，一种是先实例化，使用的时候直接给就行了，另一种就是先不实例化，等到要使用的时候才进行实例化操作，默认情况下是先实例化，lazy-init即开启第二种情况，如

```xml
<bean id="lazy" class="com.something.ExpensiveToCreateBean" lazy-init="true"/>
<bean name="not.lazy" class="com.something.AnotherBean"/>
```

同时，对于标签<beans\>，也提供default-lazy-init，表示所有bean使用懒汉模式

```xml
<beans default-lazy-init="true">
    <!-- no beans will be pre-instantiated... -->
</beans>
```

#### bean作用域（bean Scopes）

使用标签<scope\>设置

##### 单例模式

默认模式，返回唯一的bean实例

![prototype](https://docs.spring.io/spring/docs/current/spring-framework-reference/images/prototype.png) 

##### 原型模式

每次都返回一个新的bean实例

```xml
<bean id="accountService" class="com.something.DefaultAccountService" scope="prototype"/>
```

![prototype](https://docs.spring.io/spring/docs/current/spring-framework-reference/images/prototype.png) 

#### 自动装配Bean(Autowire)

自动装配即自动将一个bean依赖装配到bean中而不必使用ref手动引用，自动装配基于setter，即自动将所有setter装配为依赖，默认情况下关闭

##### byName

 根据 Property 的 Name 自动装配，如果一个 bean 的id和另一个 bean 中的 Property 的 name 相同，则自动装配这个 bean 到 Property 中。 

```xml
 <bean id="customerService" class="spring.services.CustomerService" autowire="byName">
 </bean>
 <bean id="customerDAO" class="spring.dao.CustomerDAO" />
```

根据setter名字自动装配，CustomerService中有属性  CustomerDAO customerDAO和setCustomerDAO()方法，id和属性名相同，根据名称及setter自动装配

##### byType

 根据属性 Property 的数据类型自动装配，这种情况，CustomerService 设置了 `autowire="byType"` ，Spring 会自动寻找与属性类型相同的 bean ，找到后，通过调用 setCustomerDAO(CustomerDAO customerDAO) 将其注入。 

```xml
 <bean id="customerService" class="spring.services.CustomerService" autowire="byType">
 </bean>
 <bean id="customerDAO" class="spring.dao.CustomerDAO" />
```

即根据bean的属性类型注入，如果有两个一样的类型的bean在xml中被声明，那么直接抛出异常

##### constructor

根据bean的构造器参数的类型注入，同样不能有多个相同类型

```xml
 <bean id="customerService" class="spring.services.CustomerService" autowire="constructor">
 </bean>
 <bean id="customerDAO" class="spring.dao.CustomerDAO" />
```

### 常用注解

使用注解必须在配置文件中加上

```xml
<context:annotation-config/>
```

#### @Required

 应用于 bean 属性的 setter 方法，表明这个属性必须被注入，否则抛出异常

#### @Autowired 

用在字段上，表示自动注入，可省略getter与setter，使用时自动扫描容器中的所有bean，按类型(默认byType)进行注入，在配置文件中加入：

```xml
    <bean class="org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor"/>
```

#### @Qualifier 

要与@Autowired 配合使用，因为多态的性质，有时候容器可能发现多个符合条件的Bean可以注入从而导致混乱，引入@Qualifier(“ ”)表明注入哪一个bean，该bean是配置文件中的一个bean的id

#### @Configuration 

用在类上，声明一个配置类，等价于配置文件中的beans，要使用该配置类则加载的时候可使用 AnnotationConfigApplicationContext()获取容器

#### @Bean

用在方法上，声明一个bean，等价于配置文件中的bean，方法名就是bean的id，可指定name，要与@Configuration配合使用，相当于直接在方法中自己实例化一个类，设置属性并返回，交给容器去管理

```java
public class C {
    public void c(){
        System.out.println("cccccccccccccccccccccccccccc");
    }
}
public class D {
    public void d(){
        System.out.println("dddddddddddddddddddddddddd");
    }
}
@Configuration
public class TestConfigure {

    @Bean(name = "c")
    public C getC(){
        return new C();
    }

    @Bean(name = "d")
    public D getD(){
        return new D();
    }
}
    public static void main(String[] args){
        ApplicationContext context=new AnnotationConfigApplicationContext(TestConfigure.class);
        context.getBean(C.class).c();
        context.getBean(D.class).d();

    }
```

#### @Component

 用在class上，开启自动扫描的时候会自动将该bean加入容器当中。在配置文件中加入

```xml
    <context:component-scan base-package="springTest"/>
    <context:annotation-config/>
```

指定扫描的包并开启自动扫描

除此之外，Spring 有三个与 @Component 等效的注解：

1. @Controller:对应表现层的 Bean，也就是 Action 。
2. @Service:对应的是业务层 Bean 。
3. @Repository:对应数据访问层 Bean 。

配置文件：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd">

    <context:component-scan base-package="springTest"/>
    <context:annotation-config/>
    <bean class="org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor"/>
</beans>
```

```java
@Component("a")
public class A {
    private int id;

    @Autowired
    private B b;

    public void setId(int id) {
        this.id = id;
    }

    public int getId() {
        return id;
    }

    public A(){

    }

    @Override
    public String toString() {
        return "A"+id+b.getId();
    }
}
@Component("b")
public class B {
    private int id;

    public void setId(int id) {
        this.id = id;
    }

    public int getId() {
        return id;
    }

    public B(){

    }

    @Override
    public String toString() {
        return "B" +getId();
    }
}
public class BTest {
    ApplicationContext context=new ClassPathXmlApplicationContext("/spring/test.xml");
    B b=context.getBean(B.class);//通过反射返回实例
    A a=context.getBean(A.class);//A B都通过自动扫描注入
    @Test
    public void test(){
        System.out.println(b.toString());
        System.out.println(a.toString());
    }

}
```

### AOP

 https://zhuanlan.zhihu.com/p/25892085 

 Spring AOP**默认是使用JDK动态代理**，如果代理的类**没有接口则会使用CGLib代理** 

引入AOP要在配置文件中加入

```xml
 <aop:aspectj-autoproxy/>
 //或者
 <bean class="org.springframework.aop.aspectj.annotation.AnnotationAwareAspectJAutoProxyCreator"/>
```

术语：

**1：连接点(Joinpoint)**

程序执行的某个特定位置：如类开始初始化前，类初始化后，类某个方法调用前。一个类或一段代码拥有一些边界性质的特定点，这些代码中的特定点就被称为“连接点”。**Spring仅支持方法的连接点**，既仅能在方法调用前，方法调用后，方法抛出异常时等这些程序执行点进行织入增强。

连接点由两个信息确定：第一是用方法表示的程序执行点；第二是用相对点表示的方位。如在Test.foo()方法执行前的连接点，执行点为Test.foo()，方位为该方法执行前的位置。Spring使用切点对执行点进行定位，而方位则在增强类型中定义。

**2：切点（Pointcut）**

每个程序类都拥有多个连结点，如一个拥有两个方法的类，这两个方法都是连接点，既连接点是程序中客观存在的事务。当某个连接点满足指定要求时，就可以被添加Advice，成为切点，比如：

pointcut xxxPorintcut()

:execution(void H*.say*())

每个方法被调用，都只是连接点，如果某个方法属于H开头的类，而且方法一say开头，则该方法就变成了切入点。如何使用表达式定义切入点，是AOP的核心，Spring默认使用**AspectJ**切入点语法。

**3：增强（Advice）**

连接上一条，你可以在切点这里做一些事情，有“around”，“before”，“after”等类型。

**4：目标对象（Target）**

被AOP框架进行增强处理的对象。如果是动态AOP的框架，则对象是一个被代理的对象。

**5：引介(Introduction)：**

引介是一种特殊的增强，它为类添加一些属性和方法。这样，即使一个业务类原本没有实现某个接口，通过AOP的引介功能，我们可以动态的为该事务添加接口的实现逻辑，让业务类成为这个接口的实现类。

**6：织入（Weaving）：**

织入是将增强添加对目标类具体连接点上的过程，AOP象一台织布机，将目标类增强或引介AOP这台织布机天衣无缝的编织在一起。

**7：代理（Proxy）**

一个类被AOP织入增强后，就产生了一个结果类，它是融合了原类和增前逻辑的代理类。根据不同的代理方式，代理类及可能是和原类具有相同的接口的类，也可能是原类的子类，所以我们可以采用调用原类得相同方式调用代理类。

**8：切面（Aspect）**

切面由切点和增强（引介）组成，它既包括了横切逻辑的定义，也包括了连接点的定义，Spring AOP就是负责实施切面的框架，它将切面所定义的横切逻辑织入到切面所指定的链接点中。

AspectJ静态织入，在编译时生成.class插入，Spring动态织入，纯 Java。

 AOP编程其实是很简单的事情，纵观AOP编程，程序员只需要参与三个部分：

1、定义普通业务组件

2、定义切入点，一个切入点可能横切多个业务组件

3、定义增强处理，增强处理就是在AOP框架为普通业务组件织入的处理动作

所以进行AOP编程的关键就是定义切入点和定义增强处理，一旦定义了合适的切入点和增强处理，AOP框架将自动生成AOP代理，即：**代理对象的方法=增强处理+被代理对象**的方法。

不使用Spring则可以自己使用JDK代理与Cglib，或者Aspectj，Aspectj专为AOP提供了表达式语法，使用Spring则直接通过读取配置文件拿到增强处理，拿到代理对象，织入增强方法即可

#### 简单使用

