## 基础

### IOC/DI

IDC/DI是一种仅通过构造器参数、工厂方法参数或由构造器/工厂方法产生的实例的属性定义依赖关系，由容器在创造bean的过程中注入依赖的过程。

 **所谓依赖注入，就是把底层类作为参数传入上层类，实现上层类对下层类的“控制**” 

 ![img](https://pic2.zhimg.com/80/v2-c920a0540ce0651003a5326f6ef9891d_hd.png) 

 ![img](https://pic3.zhimg.com/80/v2-c845802f9187953ed576e0555f76da42_hd.png) 

 ![img](https://pic4.zhimg.com/80/v2-555b2be7d76e78511a6d6fed3304927f_hd.png) 

   ![img](https://pic3.zhimg.com/80/v2-24a96669241e81439c636e83976ba152_hd.png) 

 ![img](https://pic2.zhimg.com/80/v2-5ca61395f37cef73c7bbe7808f9ea219_hd.png) 

DI实现了解耦，解决了类之间的依赖问题，由ioc容器动态注入对象的依赖，管理bean生命周期，而不用手动创建依赖对象并管理。 

### IOC容器与应用上下文

接口BeanFactory提供管理object的配置机制， ApplicationContext（应用上下文）是其子接口，提供更多特性。BeanFactory提供基本功能与配置框架，ApplocationContext提供更多企业级功能。

接口org.springframework.context.ApplicationContext象征着IOC容器，负责beans的实例化，配置与管理。容器通过读取配置元数据(configuration metadata)获取bean的构造信息，配置元数据可以是XML，注解或者java代码。

- `AnnotationConfigApplicationContext：`从一个或多个基于Java的配置类中加载Spring应用上下文。
- `AnnotationConfigWebApplicationContext：`从一个或多个基于Java的配置类中加载Spring Web应用上下文。
- `ClassPathXmlApplicationContext：`从类路径下的一个或多个XML配置文件中加载上下文定义，把应用上下文的定义文件作为类资源（相对路径）。
- `FileSystemXmlapplicationcontext：`从文件系统下的一个或多个XML配置文件中加载上下文定义（绝对路径）。
- `XmlWebApplicationContext：`从Web应用下的一个或多个XML配置文件中加载上下文定义。

### Spring模块

[<img src="https://s2.ax1x.com/2020/02/14/1jgP61.png" alt="1jgP61.png" style="zoom:50%;" />](https://imgchr.com/i/1jgP61)

* Spring核心容器：容器是Spring框架最核心的部分，它管理着Spring应用中bean的创建、配置和管理。在该模块中，包括了Spring bean工厂，它为Spring提供了DI的功能。
* Spring的AOP模块：在AOP模块中，Spring对面向切面编程提供了丰富的支持。这个模块是Spring应用系统中开发切面的基础。与DI一样，AOP可以帮助应用对象解耦。
* 数据访问与集成：使用JDBC编写代码通常会导致大量的样板式代码，例如获得数据库连接、创建语句、处理结果集到最后关闭数据库连接。Spring的ORM模块建立在对DAO的支持之上，并为多个ORM框架提供了一种构建DAO的简便方式。Spring没有尝试去创建自己的ORM解决方案，而是对许多流行的ORM框架进行了集成，包括Hibernate、Java Persisternce API、Java Data Object和iBATIS SQL Maps。Spring的事务管理支持所有的ORM框架以及JDBC。
* Web与远程调用：Spring远程调用功能集成了RMI（Remote Method Invocation）、Hessian、Burlap、JAX-WS，同时Spring还自带了一个远程调用框架：HTTP invoker。
* Spring Web Flow：Spring Web Flow建立于Spring MVC框架之上，它为基于流程的会话式Web应用（可以想一下购物车或者向导功能）提供了支持。

## Bean的装配

创建应用对象之间协作关系的行为通常称为装配（wiring），这也是依赖注入（DI）的本质。

作为开发人员，你需要告诉Spring要创建哪些bean并且如何将其装配在一起。当描述bean如何进行装配时，Spring具有非常大的灵活性，它提供了三种主要的装配机制：

- 在XML中进行显式配置。
- 在Java中进行显式配置。
- 隐式的bean发现机制和自动装配。

### 自动装配Bean

Spring从两个角度来实现自动化装配：

- 组件扫描（component scanning）：Spring会自动发现应用上下文中所创建的bean。
- 自动装配（autowiring）：Spring自动满足bean之间的依赖。

#### @Configuration+@ComponentScan

@Configuration 用在类上，声明一个配置类，等价于配置文件中的标签<beans\>，@ComponentScan表明启用注解扫描，会自动扫描与该类在同一个包下的类。

可以指定扫描的包，如@ComponentScan("sounds")或@ComponentScan(basePackages="sound")就表明扫描包sounds下的类；可以指定扫描多个包，如@ComponentScan(basePackages={"soundsystem", "video"})，也可以使用某个包下面的一个类的方式，如@ComponentScan(basePackageClasses={CDPlayer.class, DVDPlayer.class})表明这些类所在的包将会作为组件扫描的基础包。

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

可以指定该Bean的id，如@Component("lsl")表明其id为lsl，默认的id是将该类名首字母小写

#### @Autowired

用在字段上，表示自动注入该字段,使用时自动扫描容器中的所有bean，按类型(默认byType)进行注入

也可以用在构造器或方法上（常用在setter上），Spring会尽可能将参数中的Bean注入进去，如

```java
@Autowired
public void insertDisc(CompactDisc cd) {
  this.cd = cd;
}
```

如果没有匹配的bean，那么在应用上下文创建的时候，Spring会抛出一个异常。为了避免异常的出现，可以将`@Autowired`的`required`属性设置为`false`：

```java
@Autowired(required=false)
public CDPlayer(CompactDisc cd) {
  this.cd = cd;
}
```

将`required`属性设置为`false`时，Spring会尝试执行自动装配，但是如果没有匹配的bean的话，Spring将会让这个bean处于未装配的状态。但是，把`required`属性设置为`false`时，你需要谨慎对待。如果在你的代码中没有进行null检查的话，这个处于未装配状态的属性有可能会出现`NullPointerException`。

如果有多个bean都能满足依赖关系的话，Spring将会抛出一个异常，表明没有明确指定要选择哪个bean进行自动装配。

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
}
```

#### @ContextConfiguration

用在类上，指定一个配置类，表明该类中需要使用容器（载入配置类）

```java
@ContextConfiguration(classes=CDPlayerConfig.class)
```

表明该类需要载入配置类CDPlayerConfig，同时能够自动注入容器中的bean

### 使用javaConfig配置bean

#### @Bean

用在方法上，声明一个bean，等价于标签`<bean>`，要与@Configuration配合使用，相当于直接在方法中自己实例化一个类，设置属性并返回，交给容器去管理

bean的id就是方法名，可指定name，如`@Bean(name="lonelyHeartsClubBand")`

带有`@Bean`注解的方法可以采用任何必要的Java功能来产生bean实例，因为Spring会按Bean的生命周期进行管理，如：

```java
@Bean
public CDPlayer cdPlayer() {
  return new CDPlayer(sgtPeppers());
}
@Bean
public CDPlayer anotherCDPlayer() {
  return new CDPlayer(sgtPeppers());
}
```

这两个方法返回的Bean其实是一样的，Spring会拦截对`sgtPeppers()`的调用并确保返回的是Spring所创建的bean，也就是Spring本身在调用`sgtPeppers()`时所创建的`CompactDisc`bean。因此，两个`CDPlayer` bean会得到相同的`SgtPeppers`实例。

### 通过XML装配Bean

在使用XML为Spring装配bean之前，需要创建一个新的配置规范，需要在配置文件的顶部声明多个XML模式（XSD）文件，这些文件定义了配置Spring的XML元素。如：

```xml
?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans.xsd
    http://www.springframework.org/schema/context">

  <!-- configuration details go here -->

</beans>
```

用来装配bean的最基本的XML元素包含在`spring-beans`模式之中，在上面这个XML文件中，它被定义为根命名空间。`<beans>`是该模式中的一个元素，它是所有Spring配置文件的根元素。

<bean\>等价于@Bean，用于在配置文件中声明一个Bean，如

```xml
<bean class="soundsystem.SgtPeppers" />
```

这里声明了一个很简单的bean，创建这个bean的类通过class属性来指定的，并且要使用全限定的类名。

因为没有明确给定ID，所以这个bean将会根据全限定类名来进行命名。在本例中，bean的ID将会是“`soundsystem.SgtPeppers#0`”。其中，“`#0`”是一个计数的形式，用来区分相同类型的其他bean。如果你声明了另外一个`SgtPeppers`，并且没有明确进行标识，那么它自动得到的ID将是“`soundsystem.SgtPeppers#1`”。

可以直接指定id：

```xml
<bean id="compactDisc" class="soundsystem.SgtPeppers" />
```

稍后将这个bean装配到`CDPlayer` bean之中的时候，你会用到这个具体的名字。

在XML配置中，bean的创建显得更加被动，不过，它并没有JavaConfig那样强大，在JavaConfig配置方式中，你可以通过任何可以想象到的方法来创建bean实例。

声明一个bean之后就需要配置DI注入方式，有多种方式可选：

#### 构造器注入bean引用

```xml
<bean id="cdPlayer" class="soundsystem.CDPlayer">
  <constructor-arg ref="compactDisc" />
</bean>
```

当Spring遇到这个元素时，它会创建一个`CDPlayer`实例。`<constructor-arg>`元素会告知Spring要将一个ID为`compactDisc`的bean引用传递到`CDPlayer`的构造器中。

也可以使用Spring的c-命名空间。c-命名空间是在Spring 3.0中引入的，它是在XML中更为简洁地描述构造器参数的方式。要使用它的话，必须要在XML的顶部声明其模式，如下所示：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:c="http://www.springframework.org/schema/c"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.springframework.org/schema/beans
  http://www.springframework.org/schema/beans/spring-beans.xsd">
 ...
</beans>
```

在c-命名空间和模式声明之后，我们就可以使用它来声明构造器参数了，如下所示：

```xml
<bean id="cdPlayer" class="soundsystem.CDPlayer"
      c:cd-ref="compactDisc" />
```

[<img src="https://s2.ax1x.com/2020/02/15/1vOO4H.png" alt="1vOO4H.png" style="zoom:50%;" />](https://imgchr.com/i/1vOO4H)

冒号后面跟着的是构造器参数名，也可以直接将参数名改为参数索引，效果是一样的，如：

```xml
<bean id="cdPlayer" class="soundsystem.CDPlayer"
      c:_0-ref="compactDisc" />
```

这个c-命名空间属性看起来似乎比上一种方法更加怪异。我将参数的名称替换成了“0”，也就是参数的索引。因为在XML中不允许数字作为属性的第一个字符，因此必须要添加一个下画线作为前缀。

#### 构造器注入bean字面量

```java
package soundsystem;
public class BlankDisc implements CompactDisc {
  private String title;
  private String artist;

  public BlankDisc(String title, String artist) {
    this.title = title;
    this.artist = artist;
  }
}
```

构造器不是引用，而是字面量，也可注入：

```xml
<bean id="compactDisc"
      class="soundsystem.BlankDisc">
  <constructor-arg value="Sgt. Pepper's Lonely Hearts Club Band" />
  <constructor-arg value="The Beatles" />
</bean>
```

使用了`value`属性，通过该属性表明给定的值要以字面量的形式注入到构造器之中，也可使用c-命名空间

```xml
<bean id="compactDisc"
      class="soundsystem.BlankDisc"
      c:_title="Sgt. Pepper's Lonely Hearts Club Band"
      c:_artist="The Beatles" /><!--可直接使用索引，如c:_0="xxx"-->
```

#### 构造器装配集合

`<list>`元素是`<constructor-arg>`的子元素，这表明一个包含值的列表将会传递到构造器中。其中，`<value>`元素用来指定列表中的每个元素。

```xml
<bean id="compactDisc" class="soundsystem.BlankDisc">
  <constructor-arg value="Sgt. Pepper's Lonely Hearts Club Band" />
  <constructor-arg value="The Beatles" />
  <constructor-arg>
    <list>
      <value>Sgt. Pepper's Lonely Hearts Club Band</value>
      <value>With a Little Help from My Friends</value>
      <!-- ...other tracks omitted for brevity... -->
    </list>
  </constructor-arg>
</bean>
```

与之类似，我们也可以使用`<ref>`元素替代`<value>`，实现bean引用列表的装配。

`<set>`与·`<list>`使用方法相同，不过其创建的对象是java.util.Set

#### setter注入bean

```java
public class CDPlayer implements MediaPlayer {
  private CompactDisc compactDisc;

  @Autowired
  public void setCompactDisc(CompactDisc compactDisc) {
    this.compactDisc = compactDisc;
  }
  public void play() {
    compactDisc.play();
  }
}
```

```xml
<bean id="cdPlayer"
      class="soundsystem.CDPlayer">
  <property name="compactDisc" ref="compactDisc" />
</bean>
```

`<property>`元素为属性的Setter方法所提供Bean的注入功能，在本例中，它引用了ID为`compactDisc`的bean（通过`ref`属性），并将其注入到`compactDisc`属性中（通过`setCompactDisc()`方法）。

Spring为`<construct-arg>`元素提供了c-命名空间作为替代方案，与之类似，Spring提供了更加简洁的p-命名空间，为了启用p-命名空间，必须要在XML文件中与其他的命名空间一起对其进行声明：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:p="http://www.springframework.org/schema/p"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans.xsd">
  ...
</bean>
```

```xml
<bean id="cdPlayer"
      class="soundsystem.CDPlayer"
      p:compactDisc-ref="compactDisc" />
```

[<img src="https://s2.ax1x.com/2020/02/15/1vvIvn.md.png" alt="1vvIvn.md.png" style="zoom:50%;" />](https://imgchr.com/i/1vvIvn)

#### setter注入字面量

`ref`变为`name`与`value`即可

```xml
bean id="compactDisc"
      class="soundsystem.BlankDisc">
  <property name="title"
               value="Sgt. Pepper's Lonely Hearts Club Band" />
  <property name="artist" value="The Beatles" />
  <property name="tracks">
    <list>
      <value>Sgt. Pepper's Lonely Hearts Club Band</value>
      <value>With a Little Help from My Friends</value>
      <!-- ...other tracks omitted for brevity... -->
    </list>
  </property>
</bean>
```

除了使用`value`属性来设置`title`和`artist`，我们还使用了内嵌的`<list>`元素来设置`tracks`属性，这与之前通过`<construct-arg>`装配`tracks`是完全一样的。

### 导入与混合配置

#### @Import

与@Configuration混合使用，用于导入另一个配置类的配置到当前配置中，用于解耦

```java
@Configuration
public class CDConfig {
  @Bean
  public CompactDisc compactDisc() {
    return new SgtPeppers();
  }
}
```

```java
@Configuration
@Import(CDConfig.class)
public class CDPlayerConfig {
  @Bean
  public CDPlayer cdPlayer(CompactDisc compactDisc) {
    return new CDPlayer(compactDisc);
  }
}
```

一个更好的办法就是不在`CDPlayerConfig`中使用`@Import`，而是创建一个更高级别的`SoundSystemConfig`，在这个类中使用`@Import`将两个配置类组合在一起：

```java
@Configuration
@Import({CDPlayerConfig.class, CDConfig.class})
public class SoundSystemConfig {
}
```

#### @ImportResource

@ImportResource用于加载一个xml配置文件进入当前配置类，将其一块注入Spring容器中

```java
@Configuration
@Import(CDPlayerConfig.class)
@ImportResource("classpath:cd-config.xml")
public class SoundSystemConfig {
}
```

#### xml配置文件导入<import\>

`<import>`元素可以导入另一个xml配置文件进入当前文件，同样用于解耦

```xml
<import resource="cd-config.xml" />
```

也可以将JavaConfig类导入到XML配置中

```xml
<bean class="soundsystem.CDConfig" />
```

## 高级装配

### 条件装配bean-@Conditional

假设你希望一个或多个bean只有在应用的类路径下包含特定的库时才创建。或者我们希望某个bean只有当另外某个特定的bean也声明了之后才会创建。我们还可能要求只有某个特定的环境变量设置之后，才会创建某个bean。Spring 4引入了一个新的`@Conditional`注解，它可以用到带有`@Bean`注解的方法上。如果给定的条件计算结果为`true`，就会创建这个bean，否则的话，这个bean会被忽略。

例如，假设有一个名为`MagicBean`的类，我们希望只有设置了`magic`环境属性的时候，Spring才会实例化这个类。如果环境中没有这个属性，那么`MagicBean`将会被忽略。

[![1xPZRS.png](https://s2.ax1x.com/2020/02/15/1xPZRS.png)](https://imgchr.com/i/1xPZRS)

可以看到，`@Conditional`中给定了一个`Class`，它指明了条件——在本例中，也就是`MagicExistsCondition`，@Conditional括号内任意一个实现了`Condition`接口的类，该接口的定义为

```java
public interface Condition {
    boolean matches(ConditionContext ctxt,
                  AnnotatedTypeMetadata metadata);
}
```

@Conditional会检查该类中的matches方法，只有返回真时才创建Bean，matches中的两个参数为条件创建Bean提供了交互，这两个参数为当前类提供@Conditional所注解的类的信息。

`ConditionContext`是一个接口，大致如下所示：

```java
public interface ConditionContext {
  BeanDefinitionRegistry getRegistry();
  ConfigurableListableBeanFactory getBeanFactory();
  Environment getEnvironment();
  ResourceLoader getResourceLoader();
  ClassLoader getClassLoader();
}
```

通过`ConditionContext`，我们可以做到如下几点：

* 借助`getRegistry()`返回的`BeanDefinitionRegistry`检查bean定义；

- 借助`getBeanFactory()`返回的`ConfigurableListableBeanFactory`检查bean是否存在，甚至探查bean的属性；
- 借助`getEnvironment()`返回的`Environment`检查环境变量是否存在以及它的值是什么；
- 读取并探查`getResourceLoader()`返回的`ResourceLoade`r所加载的资源；
- 借助`getClassLoader()`返回的`ClassLoader`加载并检查类是否存在。

`AnnotatedTypeMetadata`则能够让我们检查带有`@Bean`注解的方法上还有什么其他的注解。像`ConditionContext`一样，`AnnotatedTypeMetadata`也是一个接口。它如下所示：

```java
public interface AnnotatedTypeMetadata {
  boolean isAnnotated(String annotationType);
  Map<String, Object> getAnnotationAttributes(String annotationType);
  Map<String, Object> getAnnotationAttributes(
          String annotationType, boolean classValuesAsString);
  MultiValueMap<String, Object> getAllAnnotationAttributes(
          String annotationType);
  MultiValueMap<String, Object> getAllAnnotationAttributes(
          String annotationType, boolean classValuesAsString);
}
```

借助`isAnnotated()`方法，我们能够判断带有`@Bean`注解的方法是不是还有其他特定的注解。借助其他的那些方法，我们能够检查`@Bean`注解的方法上其他注解的属性。

### 处理自动装配的歧义性@Qualifier 

因为多态的性质，有时候容器可能发现多个符合条件的Bean可以注入从而导致混乱，如

```java
@Autowired
public void setDessert(Dessert dessert) {
    this.dessert = dessert;
}
```

Dessert是一个接口，Spring容器中有多个Dessert对象

```java
@Component
public class Cake implements Dessert { ... }
@Component
public class Cookies implements Dessert { ... }
@Component
public class IceCream implements Dessert { ... }
```

当Spring试图自动装配`setDessert()`中的`Dessert`参数时，它并没有唯一、无歧义的可选值，因此会抛出`NoUniqueBeanDefinitionException`异常

注解@Qualifier 可以用于限定注入的Bean的类型，可以与`@Autowired`和`@Inject`协同使用，在注入的时候指定想要注入进去的是哪个bean。例如，我们想要确保要将`IceCream`注入到`setDessert()`之中：

```java
@Autowired
@Qualifier("iceCream")
public void setDessert(Dessert dessert) {
    this.dessert = dessert;
}
```

为`@Qualifier`注解所设置的参数就是想要注入的bean的ID。实际上，`@Qualifier`的参数是限定符，只不过Spring默认的Bean的限定符就是Bean的Id，`@Qualifier`可以和`@Bean`或者`@Component`配合使用指定一个Bean的限定符

```java
@Bean
@Qualifier("cold")
public Dessert iceCream() {
    return new IceCream();
}
@Component
@Qualifier("cold")
public class IceCream implements Dessert { ... }
```

此时`@Qualifier`配合`@Autowired`使用时就更加解耦

```java
@Autowired
@Qualifier("cold")
public void setDessert(Dessert dessert) {
    this.dessert = dessert;
}
```

### Bean作用域@Scope

在默认情况下，Spring应用上下文中所有bean都是作为以单例（singleton）的形式创建的。也就是说，不管给定的一个bean被注入到其他bean多少次，每次所注入的都是同一个实例。Spring中Bean的作用域为

- 单例（Singleton）：在整个应用中，只创建bean的一个实例。
- 原型（Prototype）：每次注入或者通过Spring应用上下文获取的时候，都会创建一个新的bean实例。
- 会话（Session）：在Web应用中，为每个会话创建一个bean实例。
- 请求（Rquest）：在Web应用中，为每个请求创建一个bean实例。

如果选择其他的作用域，要使用`@Scope`注解，它可以与`@Component`或`@Bean`一起使用，如

```java
@Component
@Scope(ConfigurableBeanFactory.SCOPE_PROTOTYPE)
public class Notepad { ... }
```

等价于属性`scope`，如

```java
<bean id="notepad"
      class="com.myapp.Notepad"
      scope="prototype" />
```

### 运行时注入字面量

即在程序运行过程中注入字面量的值，有两种方式可以运行时注入字面量

#### @PropertySource

在Spring中，处理外部值的最简单方式就是声明属性源并通过Spring的`Environment`来检索属性，属性源通常都是properties文件

[![1zY1UI.md.png](https://s2.ax1x.com/2020/02/15/1zY1UI.md.png)](https://imgchr.com/i/1zY1UI)

在本例中，`@PropertySource`引用了类路径中一个名为app.properties的文件。它大致会如下所示：

```properties
disc.title=Sgt. Peppers Lonely Hearts Club Band
disc.artist=The Beatles
```

这个属性文件会加载到Spring的`Environment`中，稍后可以从这里检索属性。同时，在`disc()`方法中，会创建一个新的`BlankDisc`，它的构造器参数是从属性文件中获取的，而这是通过调用`getProperty()`实现的。

`getProperty()`方法有四个重载的变种形式：

- String getProperty(String key)
- String getProperty(String key, String defaultValue)：如果没有检索到就使用默认值
- T getProperty(String key, Class<T\> type)：指定返回类型，如env.getProperty("db.connection.count", Integer.class);
- T getProperty(String key, Class<T\> type, T defaultValue)

#### 使用SpEL动态装配

Spring 3引入了Spring表达式语言（Spring Expression Language，SpEL），它能够以一种强大和简洁的方式将值装配到bean属性和构造器参数中，在这个过程中所使用的表达式会在运行时计算得到值，SpEL表达式要放到“`#{ ... }`”之中，这与属性占位符有些类似，属性占位符需要放到“`${ ... }`”之中

基本的SpEL表达式形如

```
#{T(System).currentTimeMillis()}
```

它的最终结果是计算表达式的那一刻当前时间的毫秒数。`T()`表达式会将`java.lang.System`视为Java中对应的类型，因此可以调用其`static`修饰的`currentTimeMillis()`方法。

SpEL表达式也可以引用其他的bean或其他bean的属性。例如，如下的表达式会计算得到ID为`sgtPeppers`的bean的`artist`属性：

```
#{sgtPeppers.artist}
```

我们还可以通过`systemProperties`对象引用系统属性：

```
#{systemProperties['disc.title']}
```

在XML配置中，可以将SpEL表达式传入`value`属性中，或者将其作为p-命名空间或c-命名空间条目的值。如：

```xml
<bean id="sgtPeppers"
      class="soundsystem.BlankDisc"
      c:_title="#{systemProperties['disc.title']}"
      c:_artist="#{systemProperties['disc.artist']}" />
```

**?. 避免空指针**

可以使用?.运算符确保引用的对象不为null，避免出现空指针问题，如

```
#{artistSelector.selectArtist()?.toUpperCase()}
```

与之前只是使用点号（.）不同，现在我们使用了“?.”运算符。这个运算符能够在访问它右边的内容之前，确保它所对应的元素不是`null`,如果为null则不会调用后续方法

**T()访问静态方法与常量**

```
T(java.lang.Math)
```

这里所示的`T()`运算符的结果会是一个`Class`对象，代表了`java.lang.Math`。如果需要的话，我们甚至可以将其装配到一个`Class`类型的bean属性中。但是`T()`运算符的真正价值在于它能够访问目标类型的静态方法和常量，如`T(java.lang.Math).PI`

SpEL支持的运算有：

| 运算符类型 | 运　算　符                                                   |
| ---------- | ------------------------------------------------------------ |
| 算术运算   | `+`、`-`、 `*` 、`/`、`%`、`^`                               |
| 比较运算   | `<` 、 `>` 、 `==` 、 `<=` 、 `>=` 、 `lt` 、 `gt` 、 `eq` 、 `le` 、 `ge` |
| 逻辑运算   | `and` 、 `or` 、 `not` 、`│`                                 |
| 条件运算   | `?: (ternary)` 、 `?: (Elvis)`                               |
| 正则表达式 | `matches`                                                    |

**.?[]过滤集合**

如`#{jukebox.songs.?[artist eq 'Aerosmith']}`[]中是过滤规则

**.![]投影属性**

它会从集合的每个成员中选择特定的属性放到另外一个集合中，如`#{jukebox.songs.![title]}`

### @Value

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

### 

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

#### @Required

 应用于 bean 属性的 setter 方法，表明这个属性必须被注入，否则抛出异常

