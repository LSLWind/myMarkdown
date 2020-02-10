## IOC/DI

一种仅通过构造器参数、工厂方法参数或由构造器/工厂方法产生的实例的属性定义依赖关系，由容器在创造bean的过程中注入依赖的过程。

 **所谓依赖注入，就是把底层类作为参数传入上层类，实现上层类对下层类的“控制**” 

 ![img](https://pic2.zhimg.com/80/v2-c920a0540ce0651003a5326f6ef9891d_hd.png) 

 ![img](https://pic3.zhimg.com/80/v2-c845802f9187953ed576e0555f76da42_hd.png) 

 ![img](https://pic4.zhimg.com/80/v2-555b2be7d76e78511a6d6fed3304927f_hd.png) 

   ![img](https://pic3.zhimg.com/80/v2-24a96669241e81439c636e83976ba152_hd.png) 

 ![img](https://pic2.zhimg.com/80/v2-5ca61395f37cef73c7bbe7808f9ea219_hd.png) 

***(来源： https://www.zhihu.com/question/23277575/answer/169698662)***

接口`BeanFactory`提供管理object的配置机制， `ApplicationContext`]是其子接口，提供更多特性。BeanFactory提供基本功能与配置框架，ApplocationContext提供更多企业级功能。

### 功能

实现了解耦，解决了类之间的依赖问题，由ioc容器动态注入对象的依赖，管理bean生命周期，而不用手动创建依赖对象并管理。 

### IOC容器

接口`org.springframework.context.ApplicationContext` （应用上下文）象征着IOC容器，负责beans的实例化，配置与管理。容器通过读取配置元数据(configuration metadata)获取bean的构造信息，配置元数据可以是XML，注解或者java代码。

 ![container magic](https://docs.spring.io/spring/docs/current/spring-framework-reference/images/container-magic.png) 



`ClassPathXmlApplicationContext` 类与 `FileSystemXmlApplicationContext`是Spring提供的两个ApplicationContext的具体实现，可以通过读取xml获取元数据，为指定对象注入依赖（改配置文件而不改代码），使用方式为：在resources下配置数据源xml文件→使用时读取配置文件，获取IOC容器，通过IOC容器与反射机制根据配置文件注入依赖并获取想要的实例对象

#### 配置bean

IOC容器通过配置元数据读取要管理的bean信息，由元数据决定如何注入依赖，如果使用注解则需要在配置文件中加上

```xml
<context:annotation-config/>
```

使用标签<bean/\>与<beans\>定义bean

使用IOC容器：

```java
// 创建IOC
ApplicationContext context = new ClassPathXmlApplicationContext("services.xml", "daos.xml");
// 获取bean
PetStoreService service = context.getBean("petStore", PetStoreService.class);
// 获取元数据
List<String> userList = service.getUsernameList();
```

##### **使用静态工厂生成bean**

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

##### 基于构造器生成bean

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

##### 基于setter注入依赖

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

使用@Autowired不推荐注解静态字段，因为加载机制的问题，扫描时不会注入，因此运行时有空指针异常，@Autowired可用在构造器与方法上，扫描时可通过方法或构造器注入

```java
@Component
public class TestClass {

    private static AutowiredTypeComponent component;
    //静态字段的注入
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

#### @Value

@Value的作用是通过注解将常量、配置文件中的值、其他bean的属性值注入到变量中，作为变量的初始值。

**常量注入**

```java
@Value("normal")
private String normal; // 注入普通字符串

@Value("classpath:com/hry/spring/configinject/config.txt")
private Resource resourceFile; // 注入文件资源

@Value("http://www.baidu.com")
private Resource testUrl; // 注入URL资源
```

**bean属性、系统属性、表达式注入@Value(#{}")**

bean属性注入需要注入者和被注入者属于同一个IOC容器，或者父子IOC容器关系，在同一个作用域内。

 ```java
@Value("#{beanInject.another}")
private String fromAnotherBean; // 注入其他Bean属性：注入beanInject对象的属性another

@Value("#{systemProperties['os.name']}")
private String systemPropertiesName; // 注入系统属性

@Value("#{ T(java.lang.Math).random() * 100.0 }")
private double randomNumber; //注入表达式结果
 ```

**配置文件属性注入@Value("${}")**

@Value("#{}")读取配置文件中的值，注入到变量中去。配置文件分为默认配置文件application.properties和自定义配置文件

* application.properties：application.properties在spring boot启动时默认加载此文件

* 自定义属性文件：自定义属性文件通过@PropertySource加载。@PropertySource可以同时加载多个文件，也可以加载单个文件。如果相同第一个属性文件和第二属性文件存在相同key，则最后一个属性文件里的key起作用。加载文件的路径也可以配置变量，如下文的${anotherfile.configinject}，此值定义在第一个属性文件config.properties

```java
@Component
// 引入自定义配置文件。
@PropertySource({"classpath:com/hry/spring/configinject/config.properties",
  "classpath:com/hry/spring/configinject/config_**${anotherfile.configinject}**.properties"})
public class ConfigurationFileInject{
  @Value("${app.name}")//缺省的配置文件路径就是src/main/application.properties
  private String appName; //这里的值来自application.properties，spring boot启动时默认加载此文件
    
  @Value("${book.name}")
  private String bookName;//上述已声明配置文件，所以会查找声明的配置文件的属性

  @Value("${book.name.placeholder}")
  private String bookNamePlaceholder;
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

每个程序类都拥有多个连结点，如一个拥有两个方法的类，这两个方法都是连接点，既连接点是程序中客观存在的事务。当某个连接点满足指定要求时，就可以被添加Advice，成为切点.

如何使用表达式定义切入点，是AOP的核心，Spring默认使用**AspectJ**切入点语法。

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

不使用Spring则可以自己使用JDK代理与Cglib，或者Aspectj，Aspectj专为AOP提供了表达式语法，使用Spring则直接通过读取配置文件拿到增强处理，拿到代理对象，织入增强方法即可。

AOP能够将那些与业务无关，**却为业务模块所共同调用的逻辑或责任（例如事务处理、日志管理、权限控制等）封装起来**，便于**减少系统的重复代码**，**降低模块间的耦合度**，并**有利于未来的可拓展性和可维护性**。 

 ![img](https://upload-images.jianshu.io/upload_images/7896890-8225b1537175bd8b.png?imageMogr2/auto-orient/strip|imageView2/2/w/440/format/webp) 

 https://www.jianshu.com/p/994027425b44 

#### 通知类型

- 前置通知（Before Advice）:在切入点选择的连接点处的方法之前执行的通知，该通知不影响正常程序执行流程（除非该通知抛出异常，该异常将中断当前方法链的执行而返回）。
- **后置通知（After Advice）:**在切入点选择的连接点处的方法之后执行的通知，包括如下类型的后置通知：
  - **后置返回通知（After returning Advice）:**在切入点选择的连接点处的方法正常执行完毕时执行的通知，必须是连接点处的方法没抛出任何异常正常返回时才调用后置通知。
  - **后置异常通知（After throwing Advice）:** 在切入点选择的连接点处的方法抛出异常返回时执行的通知，必须是连接点处的方法抛出任何异常返回时才调用异常通知。
  - **后置最终通知（After finally Advice）:** 在切入点选择的连接点处的方法返回时执行的通知，不管抛没抛出异常都执行，类似于Java中的finally块。
- **环绕通知（Around Advices）：**环绕着在切入点选择的连接点处的方法所执行的通知，环绕通知可以在方法调用之前和之后自定义任何行为，并且可以决定是否执行连接点处的方法、替换返回值、抛出异常等等。

使用框架完成AOP也是基于配置，同时遵循表达式语法规则（如AspectJ），总而言之，能让目标读取并识别元信息，织入代码

#### 常规使用

1.定义目标类

 在日常开发中最后将业务逻辑定义在一个专门的service包下，而实现定义在service包下的impl包中，服务接口以IXXXService形式，而服务实现就是XXXService，规约设计，见名知义。 

```java
public interface IHelloWorldService {
	public void sayHello();
}
```

```java
public class HelloWorldService implements IHelloWorldService {
    @Override
    public void sayHello() {
    	System.out.println("============Hello World!");
    }
}
```

2.定义切面类

```java
public class HelloWorldAspect {
    //前置通知
    public void beforeAdvice() {
        System.out.println("===========before advice");
    }
    //后置最终通知
    public void afterFinallyAdvice() {
        System.out.println("===========after finally advice");
    }
}
```

3.配置切面

```xml
<bean id="aspect" class="cn.javass.spring.chapter6.aop.HelloWorldAspect"/>
<aop:config><!--AOP配置切面-->
    <!--配置切入点-->
    <aop:pointcut id="pointcut" expression="execution(* cn.javass...(..))"/>
    <!--配置切入类（切面）-->
    <aop:aspect ref="aspect">
    <!--配置前置通知-->
    <aop:before pointcut-ref="pointcut" method="beforeAdvice"/>
    <!--配置后置通知-->    
    <aop:after pointcut="execution(* cn.javass...(..))" method="afterFinallyAdvice"/>
    </aop:aspect>
</aop:config>
```

切入点使用\<aop:config>标签下的\<aop:pointcut>配置，expression属性用于定义切入点模式，默认是AspectJ语法，“execution(* cn.javass..*.*(..))”表示匹配cn.javass包及子包下的任何方法执行。

切面使用\<aop:config>标签下的\<aop:aspect>标签配置，其中“ref”用来引用切面支持类的方法。

前置通知使用\<aop:before>标签来定义，pointcut-ref属性用于引用切入点Bean，而method用来引用切面通知实现类中的方法，该方法就是通知实现，即在目标类方法执行之前调用的方法。

最终通知使用<aop:after >标签来定义，切入点除了使用pointcut-ref属性来引用已经存在的切入点，也可以使用pointcut属性来定义，如pointcut="execution(* cn.javass..*.*(..))"，method属性同样是指定通知实现，即在目标类方法执行之后调用的方法。

#### AspectJ注解使用

Spring默认不支持@AspectJ风格的切面声明，为了支持需要使用如下配置：

```
<aop:aspectj-autoproxy/>
```

**定义切面使用@Aspect()**

```java
@Aspect()
Public class Aspect{
……
}
```

使用的时候直接使用：

```xml
<bean id="aspect" class="……Aspect"/>
```

**切面中声明通知**

```
@Before(value = "切入点表达式或命名切入点", argNames = "参数列表参数名")
```

```
@AfterReturning(
value="切入点表达式或命名切入点",
pointcut="切入点表达式或命名切入点",
argNames="参数列表参数名",
returning="返回值对应参数名")
```

```
@AfterThrowing (
value="切入点表达式或命名切入点",
pointcut="切入点表达式或命名切入点",
argNames="参数列表参数名",
throwing="异常对应参数名")
```

```
@After (
value="切入点表达式或命名切入点",
argNames="参数列表参数名")
```

```
@Around (
value="切入点表达式或命名切入点",
argNames="参数列表参数名")
```

**在目标中定义切入点**

```
@Pointcut(value="切入点表达式", argNames = "参数名列表")
public void pointcutName(……) {}
```

**value：**指定切入点表达式；

**argNames：**指定命名切入点方法参数列表参数名字，可以有多个用“，”分隔，这些参数将传递给通知方法同名的参数，同时比如切入点表达式“args(param)”将匹配参数类型为命名切入点方法同名参数指定的参数类型。

**pointcutName：**切入点名字，可以使用该名字进行引用该切入点表达式。

```
@Pointcut(value="execution(* cn.javass..*.sayAdvisorBefore(..)) && args(param)", argNames = "param")
public void beforePointcut(String param) {}
```

定义了一个切入点，名字为“beforePointcut”，该切入点将匹配目标方法的第一个参数类型为通知方法实现中参数名为“param”的参数类型。

过程为：执行方法，遇到@Pointcut→找寻@Aspect()→根据value与argNames匹配增强的方法→织入

AspectJ语法： https://www.kancloud.cn/wizardforcel/gen-wo-xue-spring/153110 

### 资源Resource

在日常程序开发中，处理外部资源是很繁琐的事情，我们可能需要处理URL资源、File资源资源、ClassPath相关资源、服务器相关资源等等很多资源。因此处理这些资源需要使用不同的接口，这就增加了我们系统的复杂性；而且处理这些资源步骤都是类似的（打开资源、读取资源、关闭资源），因此如果能抽象出一个统一的接口来对这些底层资源进行统一访问那就很简洁方便了。

Spring 提供一个Resource接口来统一这些底层资源一致的访问，而且提供了一些便利的接口，从而能提供我们的生产力。

```java
public interface InputStreamSource {
    InputStream getInputStream() throws IOException;
}
```

```java
public interface Resource extends InputStreamSource {
       boolean exists();
       boolean isReadable();
       boolean isOpen();
       URL getURL() throws IOException;
       URI getURI() throws IOException;
       File getFile() throws IOException;
       long contentLength() throws IOException;
       long lastModified() throws IOException;
       Resource createRelative(String relativePath) throws IOException;
       String getFilename();
       String getDescription();
}
```

InputStreamSource就是抽象资源接口，定义获取资源流（字节流）的方法。接口Resource则进一步封装，提供对资源的操作定义。

 #### 接口Resource实现

##### ByteArrayResource

 ByteArrayResource代表byte[]数组资源，对于“getInputStream”操作将返回一个ByteArrayInputStream。 

```java
@Test
public void testByteArrayResource() {
Resource resource = new ByteArrayResource("Hello World!".getBytes());
        if(resource.exists()) {//提供外部资源流
            dumpStream(resource);
        }
}
private void dumpStream(Resource resource) {
        InputStream is = null;
        try {
            //1.获取文件资源，返回一个ByteArrayInputStream。 
            is = resource.getInputStream();
            //2.读取资源
            byte[] descBytes = new byte[is.available()];
            is.read(descBytes);
            System.out.println(new String(descBytes));
        } catch (IOException e) {
            e.printStackTrace();
        }
        finally {
            try {
                //3.关闭资源
                is.close();
            } catch (IOException e) {
            }
        }
    }
```

#####  InputStreamResource

 InputStreamResource代表java.io.InputStream字节流，对于“getInputStream ”操作将直接返回该字节流，因此只能读取一次该字节流，即“isOpen”永远返回true。 

#####  FileSystemResource

 FileSystemResource代表java.io.File资源，对于“getInputStream ”操作将返回底层文件的字节流，“isOpen”将永远返回false，从而表示可多次读取底层文件的字节流。 

##### ClassPathResource

ClassPathResource代表classpath路径的资源，将使用ClassLoader进行加载资源。classpath 资源存在于类路径中的文件系统中或jar包里，且“isOpen”永远返回false，表示可多次读取资源。

ClassPathResource加载资源替代了Class类和ClassLoader类的“( name)”和“( name)”两个加载类路径资源方法，提供一致的访问方式。

ClassPathResource提供了三个构造器：

**public ClassPathResource(String path)**：使用默认的ClassLoader加载“path”类路径资源；

**public ClassPathResource(String path, ClassLoader classLoader)：**使用指定的ClassLoader加载“path”类路径资源；

**public ClassPathResource(String path, Class<?> clazz)：**使用指定的类加载“path”类路径资源，将加载相对于当前类的路径的资源；

#####  UrlResource

UrlResource代表URL资源，用于简化URL资源访问。“isOpen”永远返回false，表示可多次读取资源。

UrlResource一般支持如下资源访问：

**http：**通过标准的http协议访问web资源，如new UrlResource(“[http://地址](http://xn--ces6a/)”)；

**ftp：**通过ftp协议访问资源，如new UrlResource(“[ftp://地址](ftp://xn--ces6a/)”)；

**file：**通过file协议访问本地文件系统资源，如new UrlResource(“file:d:/test.txt”)；

##### ServletContextResource

ServletContextResource代表web应用资源，用于简化servlet容器的ServletContext接口的getResource操作和getResourceAsStream操作

#### 资源加载器ResourceLoader

ResourceLoader接口用于返回Resource对象；定义获取资源功能，实现该接口即可认为是资源提供对象。

```java
public interface ResourceLoader {
       Resource getResource(String location);
       ClassLoader getClassLoader();
}
```

getResource用于根据提供的location参数返回相应的Resource对象；而getClassLoader则返回加载这些Resource的ClassLoader。

Spring提供了一个适用于所有环境的DefaultResourceLoader实现，可以返回ClassPathResource、UrlResource；还提供一个用于web环境的ServletContextResourceLoader，它继承了DefaultResourceLoader的所有功能，又额外提供了获取ServletContextResource的支持。

ResourceLoader在进行加载资源时需要使用前缀来指定需要加载：“classpath:path”表示返回ClasspathResource，“[http://path](http://path/)”和“file:path”表示返回UrlResource资源，如果不加前缀则需要根据当前上下文来决定，DefaultResourceLoader默认实现可以加载classpath资源

```java
@Test
public void testResourceLoad() {
    ResourceLoader loader = new DefaultResourceLoader();
    Resource resource = loader.getResource("classpath:cn/javass/spring/chapter4/test1.txt");
    //验证返回的是ClassPathResource
    Assert.assertEquals(ClassPathResource.class, resource.getClass());
    Resource resource2 = loader.getResource("file:cn/javass/spring/chapter4/test1.txt");
    //验证返回的是ClassPathResource
    Assert.assertEquals(UrlResource.class, resource2.getClass());
    Resource resource3 = loader.getResource("cn/javass/spring/chapter4/test1.txt");
    //验证返默认可以加载ClasspathResource
    Assert.assertTrue(resource3 instanceof ClassPathResource);
}
```

通过ResourceLoader可用来加载器资源并读取，提供返回其它资源的方法。目前所有ApplicationContext都实现了ResourceLoader，因此可以使用其来加载资源。

**ClassPathXmlApplicationContext：**不指定前缀将返回默认的ClassPathResource资源，否则将根据前缀来加载资源；

**FileSystemXmlApplicationContext：**不指定前缀将返回FileSystemResource，否则将根据前缀来加载资源；

**WebApplicationContext：**不指定前缀将返回ServletContextResource，否则将根据前缀来加载资源；

**其他：**不指定前缀根据当前上下文返回Resource实现，否则将根据前缀来加载资源。

### 事务Transaction

指的就是对数据库的事务管理，Spring提供几种事务管理接口完成事务操作：

- **PlatformTransactionManager：** （平台）事务管理器
- **TransactionDefinition：** 事务定义信息(事务隔离级别、传播行为、超时、只读、回滚规则)
- **TransactionStatus：** 事务运行状态

**Spring并不直接管理事务，而是提供了多种事务管理器** ，他们将事务管理的职责委托给Hibernate或者JTA等持久化机制所提供的相关平台框架的事务来实现。 Spring事务管理器的接口是： **org.springframework.transaction.PlatformTransactionManager** ，通过这个接口，Spring为各个平台如JDBC、Hibernate等都提供了对应的事务管理器，但是具体的实现就是各个平台自己的事情了。

#### 事务管理接口

PlatformTransactionManager接口定义如下：

```
public interface PlatformTransactionManager {
       TransactionStatus getTransaction(TransactionDefinition definition) throws TransactionException;
       void commit(TransactionStatus status) throws TransactionException;
       void rollback(TransactionStatus status) throws TransactionException;
}
```

- **getTransaction()：**返回一个已经激活的事务或创建一个新的事务（根据给定的TransactionDefinition类型参数定义的事务属性），返回的是TransactionStatus对象代表了当前事务的状态，其中该方法抛出TransactionException（未检查异常）表示事务由于某种原因失败。
- **commit()：**用于提交TransactionStatus参数代表的事务
- **rollback()：**用于回滚TransactionStatus参数代表的事务

**TransactionDefinition接口定义如下：**

```
public interface TransactionDefinition {
       int getPropagationBehavior();
       int getIsolationLevel();
       int getTimeout();
       boolean isReadOnly();
       String getName();
}
```

- **getPropagationBehavior()：**返回定义的事务传播行为；
- **getIsolationLevel()：**返回定义的事务隔离级别；
- **getTimeout()：**返回定义的事务超时时间；
- **isReadOnly()：**返回定义的事务是否是只读的；
- **getName()：**返回定义的事务名字。

**TransactionStatus接口定义如下：**

```
public interface TransactionStatus extends SavepointManager {
       boolean isNewTransaction();
       boolean hasSavepoint();
       void setRollbackOnly();
       boolean isRollbackOnly();
       void flush();
       boolean isCompleted();
}
```

- **isNewTransaction()：返回**当前事务状态是否是新事务**；**
- **hasSavepoint()：返回当前事务是否有保存点**；
- **setRollbackOnly()：**设置当前事务应该回滚；
- **isRollbackOnly(()：**返回当前事务是否应该回滚；
- **flush()：**用于刷新底层会话中的修改到数据库，一般用于刷新如Hibernate/JPA的会话，可能对如JDBC类型的事务无任何影响；
- **isCompleted():**当前事务否已经完成。

#### 内置事务管理器实现

Spring提供了许多内置事务管理器实现：

- **DataSourceTransactionManager：**位于org.springframework.jdbc.datasource包中，数据源事务管理器，提供对单个javax.sql.DataSource事务管理，用于Spring JDBC抽象框架、iBATIS或MyBatis框架的事务管理；
- **JdoTransactionManager：**位于org.springframework.orm.jdo包中，提供对单个javax.jdo.PersistenceManagerFactory事务管理，用于集成JDO框架时的事务管理；
- **JpaTransactionManager：**位于org.springframework.orm.jpa包中，提供对单个javax.persistence.EntityManagerFactory事务支持，用于集成JPA实现框架时的事务管理；
- **HibernateTransactionManager：**位于org.springframework.orm.hibernate3包中，提供对单个org.hibernate.SessionFactory事务支持，用于集成Hibernate框架时的事务管理；该事务管理器只支持Hibernate3+版本，且Spring3.0+版本只支持Hibernate 3.2+版本；
- **JtaTransactionManager：**位于org.springframework.transaction.jta包中，提供对分布式事务管理的支持，并将事务管理委托给Java EE应用服务器事务管理器；
- **OC4JjtaTransactionManager：**位于org.springframework.transaction.jta包中，Spring提供的对OC4J10.1.3+应用服务器事务管理器的适配器，此适配器用于对应用服务器提供的高级事务的支持；
- **WebSphereUowTransactionManager：**位于org.springframework.transaction.jta包中，Spring提供的对WebSphere 6.0+应用服务器事务管理器的适配器，此适配器用于对应用服务器提供的高级事务的支持；
- **WebLogicJtaTransactionManager：**位于org.springframework.transaction.jta包中，Spring提供的对WebLogic 8.1+应用服务器事务管理器的适配器，此适配器用于对应用服务器提供的高级事务的支持。

Spring不仅提供这些事务管理器，还提供对如JMS事务管理的管理器等，Spring提供一致的事务抽象如图9-1所示。

![img](https://box.kancloud.cn/2016-05-13_5735471fa8e43.jpg)

#### 事务传播行为（为了解决业务层方法之间互相调用的事务问题）：

当事务方法被另一个事务方法调用时，必须指定事务应该如何传播。例如：方法可能继续在现有事务中运行，也可能开启一个新事务，并在自己的事务中运行。在TransactionDefinition定义中包括了如下几个表示传播行为的常量：

**支持当前事务的情况：**

- **TransactionDefinition.PROPAGATION_REQUIRED：** 如果当前存在事务，则加入该事务；如果当前没有事务，则创建一个新的事务。
- **TransactionDefinition.PROPAGATION_SUPPORTS：** 如果当前存在事务，则加入该事务；如果当前没有事务，则以非事务的方式继续运行。
- **TransactionDefinition.PROPAGATION_MANDATORY：** 如果当前存在事务，则加入该事务；如果当前没有事务，则抛出异常。（mandatory：强制性）

**不支持当前事务的情况：**

- **TransactionDefinition.PROPAGATION_REQUIRES_NEW：** 创建一个新的事务，如果当前存在事务，则把当前事务挂起。
- **TransactionDefinition.PROPAGATION_NOT_SUPPORTED：** 以非事务方式运行，如果当前存在事务，则把当前事务挂起。
- **TransactionDefinition.PROPAGATION_NEVER：** 以非事务方式运行，如果当前存在事务，则抛出异常。

**其他情况：**

- **TransactionDefinition.PROPAGATION_NESTED：** 如果当前存在事务，则创建一个事务作为当前事务的嵌套事务来运行；如果当前没有事务，则该取值等价于TransactionDefinition.PROPAGATION_REQUIRED。

