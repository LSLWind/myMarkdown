## 配置Web Flow

Spring Web Flow是Spring MVC的扩展，它支持开发基于流程的应用程序。它将流程的定义与实现流程行为的类和视图分离开来。

Spring Web Flow是构建于Spring MVC基础之上的。这意味着所有的流程请求都需要首先经过Spring MVC的`DispatcherServlet`。我们需要在Spring应用上下文中配置一些bean来处理流程请求并执行流程。

整个流程的定义基于xml文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<flow xmlns="http://www.springframework.org/schema/webflow"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xsi:schemaLocation="http://www.springframework.org/schema/webflow
                          http://www.springframework.org/schema/webflow/spring-webflow.xsd">

</flow>
		
```

### 装配流程执行器

**流程执行器（flow executor）**驱动流程的执行。当用户进入一个流程时，流程执行器会为用户创建并启动一个流程执行实例。当流程暂停的时候（如为用户展示视图时），流程执行器会在用户执行操作后恢复流程。

在Spring中，`flow:flow-executor`元素会创建一个流程执行器：

```xml
<flow:flow-executor id="flowExecutor" />
```

尽管流程执行器负责创建和执行流程，但它并不负责加载流程定义。这个责任落在了流程注册表（flow registry）身上，接下来我们会创建它。

### 配置流程注册表

**流程注册表**（flow registry）的工作是加载流程定义并让流程执行器能够使用它们。我们可以在Spring中使用`flow:flow-registry`配置流程注册表，如下所示：

```xml
<flow:flow-registry id="flowRegistry" base-path="/WEB-INF/flows">
   <flow:flow-location-pattern value="*-flow.xml" />
</flow:flow-registry>
```

在这里的声明中，流程注册表会在“/WEB-INF/flows”目录下查找流程定义，这是通过`base-path`属性指明的。依据`flow:flow-location-pattern`元素的值，任何文件名以“-flow.xml”结尾的XML文件都将视为流程定义。

所有的流程都是通过其ID来进行引用的。这里我们使用了`flow:flow-location-pattern`元素，流程的ID就是相对于`base-path`的路径——或者双星号所代表的路径。

<img src="https://i.loli.net/2020/02/24/arvCK4FJ8wcOshN.png" alt="image.png" style="zoom:50%;" />

也可以显式声明流程

```xml
<flow:flow-registry id="flowRegistry">
   <flow:flow-location path="/WEB-INF/flows/springpizza.xml" />
</flow:flow-registry>
```

在这里，使用了`flow:flow-location`而不是`flow:flow-location-pattern`，`path`属性直接指明了“/WEB-INF/flows/springpizza.xml”作为流程

另外一种方式，可以给流程设置id

```xml
<flow:flow-registry id="flowRegistry">
   <flow:flow-location id="pizza"
         path="/WEB-INF/flows/springpizza.xml" />
</flow:flow-registry>
```

### 处理流程请求

`DispatcherServlet`一般将请求分发给控制器。但是对于流程而言，我们需要一个`FlowHandlerMapping`来帮助`DispatcherServlet`将流程请求发送给Spring Web Flow。在Spring应用上下文中，`FlowHandlerMapping`的配置如下：

```xml
<bean class=
        "org.springframework.webflow.mvc.servlet.FlowHandlerMapping">
  <property name="flowRegistry" ref="flowRegistry" />
</bean>
```

你可以看到，`FlowHandlerMapping`装配了流程注册表的引用，这样它就能知道如何将请求的URL匹配到流程上。例如，如果我们有一个ID为`pizza`的流程，`FlowHandlerMapping`就会知道如果请求的URL模式（相对于应用程序的上下文路径）是“/pizza”的话，就要将其匹配到这个流程上。

然而，`FlowHandlerMapping`的工作仅仅是将流程请求定向到Spring Web Flow上，响应请求的是`FlowHandlerAdapter`。`FlowHandlerAdapter`等同于Spring MVC的控制器，它会响应发送的流程请求并对其进行处理。`FlowHandlerAdapter`可以像下面这样装配成一个Spring bean，如下所示：

```xml
<bean class=
       "org.springframework.webflow.mvc.servlet.FlowHandlerAdapter">
  <property name="flowExecutor" ref="flowExecutor" />
</bean>
```

这个处理适配器是`DispatcherServlet`和Spring Web Flow之间的桥梁。它会处理流程请求并管理基于这些请求的流程。在这里，它装配了流程执行器的引用，而后者是为所处理的请求执行流程的。

## 流程的组件

在Spring Web Flow中，流程是由三个主要元素定义的：状态、转移和流程数据。**状态（State）**是流程中事件发生的地点。如果你将流程想象成公路旅行，那状态就是路途上的城镇、路边饭店以及风景点。流程中的状态是业务逻辑执行、做出决策或将页面展现给用户的地方，而不是在公路旅行中买Doritos薯片和健怡可乐的所在。

如果流程状态就像公路旅行中停下来的地点，那**转移（transition）**就是连接这些点的公路。在流程中，你通过转移的方式从一个状态到另一个状态。

当你在城镇之间旅行的时候，你可能要买一些纪念品，留下一些记忆并在路上取一些空的零食袋。类似地，在流程处理中，它要收集一些**数据**：流程的当前状况。

### 状态

Spring Web Flow定义了五种不同类型的状态。通过选择Spring Web Flow的状态几乎可以把任意的安排功能构造成会话式的Web应用。Spring Web Flow可供选择的状态：

| 状 态 类 型       | 它是用来做什么的                                             |
| ----------------- | ------------------------------------------------------------ |
| 行为（Action）    | 行为状态是流程逻辑发生的地方                                 |
| 决策（Decision）  | 决策状态将流程分成两个方向，它会基于流程数据的评估结果确定流程方向 |
| 结束（End）       | 结束状态是流程的最后一站。一旦进入End状态，流程就会终止      |
| 子流程（Subflow） | 子流程状态会在当前正在运行的流程上下文中启动一个新的流程     |
| 视图（View）      | 视图状态会暂停流程并邀请用户参与流程                         |

**视图状态**

视图状态用于为用户展现信息并使用户在流程中发挥作用。实际的视图实现可以是Spring支持的任意视图类型，如jsp。在流程定义的XML文件中，`view-state`用于定义视图状态：

```xml
<view-state id="welcome" />
```

在这个简单的示例中，`id`属性有两个含义。它在流程内标示这个状态。除此以外，因为在这里没有在其他地方指定视图，所以它也指定了流程到达这个状态时要展现的逻辑视图名为`welcome`。

如果你愿意显式指定另外一个视图名，那可以使用`view`属性做到这一点：

```xml
<view-state id="welcome" view="greeting" />
```

如果流程为用户展现了一个表单，你可能希望指明表单所绑定的对象。为了做到这一点，可以设置`model`属性：

```xml
<view-state id="takePayment" model="flowScope.paymentDetails"/>
```

这里我们指定`takePayment`视图中的表单将绑定流程作用域内的`paymentDetails`对象。

**行为状态**

视图状态会涉及到流程应用程序的用户，而行为状态则是应用程序自身在执行任务。行为状态一般会触发Spring所管理bean的一些方法并根据方法调用的执行结果转移到另一个状态。

在流程定义XML中，行为状态使用`action-state`元素来声明。

```xml
<action-state id="saveOrder">
  <evaluate expression="pizzaFlowActions.saveOrder(order)" />
  <transition to="thankYou" />
</action-state>
```

尽管不是严格需要的，但是`action-state`元素一般都会有一个`evaluate`作为子元素。`evaluate`元素给出了行为状态要做的事情。`expression`属性指定了进入这个状态时要评估的表达式。在本示例中，给出的`expression`是SpEL表达式，它表明将会找到ID为`pizzaFlowActions`的bean并调用其`saveOrder()`方法。

**决策状态**

有可能流程会完全按照线性执行，从一个状态进入另一个状态，没有其他的替代路线。但是更常见的情况是流程在某一个点根据流程的当前情况进入不同的分支。

决策状态能够在流程执行时产生两个分支。决策状态将评估一个Boolean类型的表达式，然后在两个状态转移中选择一个，这要取决于表达式会计算出`true`还是`false`。在XML流程定义中，决策状态通过`decision-state`元素进行定义。典型的决策状态：

```xml
<decision-state id="checkDeliveryArea">
  <if test="pizzaFlowActions.checkDeliveryArea(customer.zipCode)"
      then="addCustomer"
      else="deliveryWarning" />
</decision-state>
```

你可以看到，`decision-state`并不是独立完成工作的。`if`元素是决策状态的核心。这是表达式进行评估的地方，如果表达式结果为`true`，流程将转移到`then`属性指定的状态中，如果结果为`false`，流程将会转移到`else`属性指定的状态中。

**子流程状态**

你可能不会将应用程序的所有逻辑写在一个方法中，而是将其分散到多个类、方法以及其他结构中。

同样，将流程分成独立的部分是个不错的主意。`subflow-state`允许在一个正在执行的流程中调用另一个流程。这类似于在一个方法中调用另一个方法。

```xml
<subflow-state id="order" subflow="pizza/order">
  <input name="order" value="order"/>
  <transition on="orderCreated" to="payment" />
</subflow-state>
```

在这里，`<input>`元素用于传递订单对象作为子流程的输入。如果子流程结束的`<end-state>`状态ID为`orderCreated`，那么流程将会转移到名为`payment`的状态。

**结束状态**

最后，所有的流程都要结束。这就是当流程转移到结束状态时所做的。`end-stat`元素指定了流程的结束

```xml
<end-state id="customerReady" />
```

当到达`end-stat`状态，流程会结束。接下来会发生什么取决于几个因素：

- 如果结束的流程是一个子流程，那调用它的流程将会从`subflow-state`处继续执行。`end-stat`的ID将会用作事件触发从`subflow-state`开始的转移。
- 如果``end-stat``设置了`view`属性，指定的视图将会被渲染。视图可以是相对于流程路径的视图模板，如果添加“`externalRedirect:`”前缀的话，将会重定向到流程外部的页面，如果添加“`flowRedirect:`”将重定向到另一个流程中。
- 如果结束的流程不是子流程，也没有指定`view`属性，那这个流程只是会结束而已。浏览器最后将会加载流程的基本URL地址，当前已没有活动的流程，所以会开始一个新的流程实例。

需要意识到流程可能会有不止一个结束状态。子流程的结束状态ID确定了激活的事件，所以你可能会希望通过多种结束状态来结束子流程，从而能够在调用流程中触发不同的事件。即使不是在子流程中，也有可能在结束流程后，根据流程的执行情况有多个显示页面供选择。

### 转移

转移连接了流程中的状态。流程中除结束状态之外的每个状态，至少都需要一个转移，这样就能够知道一旦这个状态完成时流程要去向哪里。状态可以有多个转移，分别对应于当前状态结束时可以执行的不同的路径。

转移使用`transition`元素来进行定义，它会作为各种状态元素（`<action-state>`、`<view-state`、`<subflow-state`）的子元素。最简单的形式就是`<transition>`元素在流程中指定下一个状态：

```xml
<transition to="customerReady" />
```

属性`to`用于指定流程的下一个状态。如果`<transition>`只使用了`to`属性，那这个转移就会是当前状态的默认转移选项，如果没有其他可用转移的话，就会使用它。

更常见的转移定义是基于事件的触发来进行的。在视图状态，事件通常会是用户采取的动作。在行为状态，事件是评估表达式得到的结果。而在子流程状态，事件取决于子流程结束状态的ID。在任意的事件中（这里没有任何歧义），你可以使用`on`属性来指定触发转移的事件：

```xml
<transition on="phoneEntered" to="lookupCustomer"/>
```

在本例中，如果触发了`phoneEntered`事件，流程将会进入`lookupCustomer`状态。

在抛出异常时，流程也可以进入另一个状态。例如，如果顾客的记录没有找到，你可能希望流程转移到一个展现注册表单的视图状态。以下的代码片段显示了这种类型的转移：

```xml
<transition
    on-exception=
           "com.springinaction.pizza.service.CustomerNotFoundException"
    to="registrationForm" />
```

属性`on-exception`类似于`on`属性，只不过它指定了要发生转移的异常而不是一个事件。在本示例中，`CustomerNotFoundException`异常将导致流程转移到`registrationForm`状态。

**全局转移**

在创建完流程之后，你可能会发现有一些状态使用了一些通用的转移。例如，如果在整个流程中到处都有如下`transition`：

```xml
<transition on="cancel" to="endState" />
```

与其在多个状态中都重复通用的转移，我们可以将`transition`元素作为`<global-transitions>`的子元素，把它们定义为全局转移。例如：

```xml
<global-transitions>
  <transition on="cancel" to="endState" />
</global-transitions>
```

定义完这个全局转移后，流程中的所有状态都会默认拥有这个`cancel`转移。

### 流程数据

如果你曾经玩过那种老式的基于文字的冒险游戏的话，那么当从一个地方转移到另一个地方时，你会偶尔发现散布在周围的一些东西，你可以把它们捡起来并带上。有时候，你会马上需要一件东西。其他的时候，你会在整个游戏过程中带着这些东西而不知道它们是做什么用的——直到你到达游戏结束的时候才会发现它是真正有用的。

在很多方面，流程与这些冒险游戏是很类似的。当流程从一个状态进行到另一个状态时，它会带走一些数据。有时候，这些数据只需要很短的时间（可能只要展现页面给用户）。有时候，这些数据会在整个流程中传递并在流程结束的时候使用。

**声明变量**

流程数据保存在变量中，而变量可以在流程的各个地方进行引用。它能够以多种方式创建。在流程中创建变量的最简单形式是使用`<var>`元素：

```xml
<var name="customer" class="com.springinaction.pizza.domain.Customer"/>
```

这里，创建了一个新的`Customer`实例并将其放在名为`customer`的变量中。这个变量可以在流程的任意状态进行访问。

作为行为状态的一部分或者作为视图状态的入口，你有可能会使用`<evaluate>`元素来创建变量。例如：

```xml
<evaluate result="viewScope.toppingsList"
   expression="T(com.springinaction.pizza.domain.Topping).asList()" />
```

在本例中，`<evaluate>`元素计算了一个表达式（SpEL表达式）并将结果放到了名为`toppingsList`的变量中，这个变量是视图作用域的。

类似地，`<set>`元素也可以设置变量的值：

```xml
<set name="flowScope.pizza"
   value="new com.springinaction.pizza.domain.Pizza()" />
```

`<set`元素与`<evaluate>`元素很类似，都是将变量设置为表达式计算的结果。这里，我们设置了一个流程作用域内的`pizza`变量，它的值是`Pizza`对象的新实例。

### 定义流程数据的作用域

流程中携带的数据会拥有不同的生命作用域和可见性，这取决于保存数据的变量本身的作用域。Spring Web Flow定义了五种不同作用域，Spring Web Flow的作用域

| 范　　围     | 生命作用域和可见性                                           |
| ------------ | ------------------------------------------------------------ |
| Conversation | 最高层级的流程开始时创建，在最高层级的流程结束时销毁。被最高层级的流程和其所有的子流程所共享 |
| Flow         | 当流程开始时创建，在流程结束时销毁。只有在创建它的流程中是可见的 |
| Request      | 当一个请求进入流程时创建，在流程返回时销毁                   |
| Flash        | 当流程开始时创建，在流程结束时销毁。在视图状态渲染后，它也会被清除 |
| View         | 当进入视图状态时创建，当这个状态退出时销毁。只在视图状态内是可见的 |

当使用`<var>`元素声明变量时，变量始终是流程作用域的，也就是在定义变量的流程内有效。当使用`<set>`或`<esaluate>`的时候，作用域通过`name`或`result`属性的前缀指定。例如，将一个值赋给流程作用域的`theAnswer`变量：

```xml
<set name="flowScope.theAnswer" value="42"/>
```

## 基础示例：披萨流程

订购披萨的过程可以很好地定义在一个流程中。我们首先从构建一个高层次的流程开始，它定义了订购披萨的整体过程。接下来，我们会将这个流程拆分成子流程，这些子流程在较低的层次定义了细节。

### 定义基本流程

一个新的披萨连锁店Spizza决定允许用户在线订购以减轻店面电话的压力。当顾客访问Spizza站点时，他们需要进行用户识别，选择一个或更多披萨添加到订单中，提供支付信息然后提交订单并等待热乎又新鲜的披萨送过来。

<img src="https://i.loli.net/2020/02/24/fZq8NRH3ACuGaXb.png" alt="image.png" style="zoom: 67%;" />

图中的方框代表了状态而箭头代表了转移。你可以看到，订购披萨的整个流程很简单且是线性的。在Spring Web Flow中，表示这个流程是很容易的。使这个过程变得更有意思的就是前三个流程会比图中的简单方框更复杂。