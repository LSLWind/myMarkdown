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

#### 配置元数据（基于XML）

IOC容器通过配置元数据读取要管理的bean信息，由元数据决定如何注入依赖

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

##### 配置bean与bean的实例化方式

IOC默认通过构造器实例化一个bean并返回（注入依赖同理），因此至少要提供一个默认的无参数构造器，如果不使用构造器而是工厂方法则应有对应的配置说明

**通过静态工厂方法返回实例:**

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

#### 依赖注入，配置bean的依赖

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

在PetStoreServiceImpl中一定有对应的setter方法才行，区别就是使用的标签不同。对于属性值，property可以简写为p:id="001"，前提是引入 xmlns:p="http://www.springframework.org/schema/p" 

##### 依赖注入过程

1. ApplicationContext被创建并且初始化，读取配置元数据
2. 对于每个bean，其依赖(dependencies)以属性、构造器参数、静态工厂方法的方式被描述，创建bean时注入依赖
3. bean的每个依赖属性都要被实际定义

##### 循环依赖问题

如果通过构造器注入可能导致循环依赖，类A需要类B作为构造器参数，类B需要类A作为构造器参数，因此循环构造依赖，解决办法就是使用setter方式注入依赖































###### 基于注解:

配置类由@Configuration标注，Bean由@Bean标注



