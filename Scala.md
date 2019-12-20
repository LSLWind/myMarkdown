Scala 是一门多范式（multi-paradigm）的编程语言，设计初衷是要集成面向对象编程和函数式编程的各种特性。Scala 运行在Java虚拟机上，并兼容现有的Java程序。Scala 源代码被编译成Java字节码，所以它可以运行于JVM之上，并可以调用现有的Java类库。可以看做是OOP+FP

目前的教程都基于Scala2,Scala3马上就要出了，有时间更新

#### HelloWorld

```scala
object HelloWorld {
    def main(args: Array[String]): Unit = {
        println("Hello, world!")
    }
}
```

编译成字节码(javac)，然后交给JVM解释执行(java)

```shell
$ scalac HelloWorld.scala  // 把源码编译为字节码
$ scala HelloWorld  // 把字节码放到虚拟机中解释运行
```

#### ScalaWeb框架

- [Lift 框架](http://liftweb.net/)
- [Play 框架](http://www.playframework.org/)

有时间再看吧

#### Scala安装

 官网：http://www.scala-lang.org/downloads  

官网给出了两种安装方式，一种是通过IDEA，一种是下载安装，配环境变量

这里使用IDEA进行安装：

* 在plugins中搜索Scala进行下载安装
* 重启IDEA，选择new→Project→Scala即创建Scala Project
* 选择src，右键点击新建File，自己随便输入scala文件，如test.scala
* IDEA提示No Scala SDK in module，点击右上角提示的Setup Scala SDK
* 弹出框左下角选择Download，即自动下载Scala SDK，目前的版本为2.13.1

下载安装可参考： https://www.runoob.com/scala/scala-install.html 

#### 教程

官网文档，很详细： https://docs.scala-lang.org/tour/basics.html 

### Scala语法特性

#### 换行符

Scala是面向行的语言，语句可以用分号（;）或换行符结束。分号可选，一行只有一个语句则不加，但是如果一行里写多个语句那么分号是需要的：

```scala
val s = "lsl"; println(s)
```

#### 变量与数据类型

Scala的基本数据类型大致与java的相同，但Scala不提供基类型，不提供自动拆箱与装箱，在Scala中，任何值都必须有类型（即强类型语言）

![](https://docs.scala-lang.org/resources/images/tour/unified-types-diagram.svg)

Any是任何类型的基类，定义通用方法，如equals

AnyVal是所有值类型的基类，AnyRef是所有引用类型的基类（类似于java中的Object），所有没有值的类型（如Function）的基类，即任何类型都源于Any

```scala
val list: List[Any] = List(
  "a string",
  732,  // an integer
  'c',  // a character
  true, // a boolean value
  () => "an anonymous function returning a string"
)

list.foreach(element => println(element))
结果为：
a string
732
c
true
<function>
```

##### 类型转换

![](https://docs.scala-lang.org/resources/images/tour/type-casting-diagram.svg)

不用显示声明，直接转

```scala
val face: Char = '☺'
val number: Int = face  // 9786
```

多行字符可以使用三个"表示：

```scala
val foo = """l
s
l"""
```

##### 声明变量

关键词 **"var"** 声明变量，所有普通变量

关键词 **"val"** 声明常量，可看做用final修饰的变量

Scala 可以根据赋值的内容推算出变量的类型，即“ type inference ”（类型推断）：

```scala
scala> val msg="Hello,World"
msg: String = Hello,World
```

当然也可以采用和 Java 一样的方法，明确指定变量的类型，语法为 **val/var 变量名:类型：值**：

```scala
scala> val msg2:String ="Hello again,world"
msg2: String = Hello again,world
```

#### 类Classes

使用关键字**class** 声明一个类，与java不同的是，类本身可以传参，就是其构造器，使用关键字**new**声明一个类，**abstract**声明一个抽象类，**extends**继承一个类，scala只允许单继承

```scala
class Greeter(prefix: String, suffix: String) {
  def greet(name: String): Unit =
    println(prefix + name + suffix)
}
val greeter = new Greeter("Hello, ", "!")
greeter.greet("Scala developer") // Hello, Scala developer
```

对于构造器来说，参数时是可选的，按顺序赋值，当然也可以指定参数名来指定赋值

```scala
class Point(var x: Int = 0, var y: Int = 0)
val point2 = new Point(y=2)
println(point2.y)  // prints 2
```

无参构造器就是直接不带()声明：

```scala
class User
val user1 = new User
```

##### private与getter/setter

private用于声明私有变量，赋值的时候使用getter/setter，但语法非常奇怪，getter使用def声明一个方法返回，setter使用def还要在字段后加上_=，说不明白，看个例子：

```scala
class Point {
  private var _x = 0
  private var _y = 0
  private val bound = 100

  def x = _x
  def x_= (newValue: Int): Unit = {
    if (newValue < bound) _x = newValue else printWarning
  }

  def y = _y
  def y_= (newValue: Int): Unit = {
    if (newValue < bound) _y = newValue else printWarning
  }

  private def printWarning = println("WARNING: Out of bounds")
}

val point1 = new Point
point1.x = 99
point1.y = 101 // prints the warning

```

getter就是def x与def y，setter就是def x_=与 def y_=，很奇怪的语法，给<img src="file:///C:\Users\lsl\AppData\Local\Temp\SGPicFaceTpBq\147160\211A1134.png" alt="img" style="zoom:67%;" />整懵了，如果在class中不使用val或者var声明的，默认就是private字段，需注意

#### 特质--Trait

Traits用于在类之间共享接口与字段，类似于java 8的接口，内部定义功能方法，使用关键字**trait**进行声明，内部定义的方法可以有默认实现，使用关键字**extends**进行继承，使用关键字**override**声明trait中重写或实现的方法

```scala
trait Greeter {
  def greet(name: String): Unit =
    println("Hello, " + name + "!")
}
class DefaultGreeter extends Greeter

class CustomizableGreeter(prefix: String, postfix: String) extends Greeter {
  override def greet(name: String): Unit = {
    println(prefix + name + postfix)
  }
}

val greeter = new DefaultGreeter()
greeter.greet("Scala developer") // Hello, Scala developer!

val customGreeter = new CustomizableGreeter("How are you, ", "?")
customGreeter.greet("Scala developer") // How are you, Scala developer?
```

Trait支持泛型，可以带泛型参数

```scala
trait Iterator[A] {
  def hasNext: Boolean
  def next(): A
}
```

Trait内部可以有字段，子类通过构造器为该字段赋值

```scala
import scala.collection.mutable.ArrayBuffer

trait Pet {
  val name: String
}

class Cat(val name: String) extends Pet
class Dog(val name: String) extends Pet

val dog = new Dog("Harry")
val cat = new Cat("Sally")

val animals = ArrayBuffer.empty[Pet]
animals.append(dog)
animals.append(cat)
animals.foreach(pet => println(pet.name))  // Prints Harry Sally
```

#### 元组--Tuples

和 `List` 不同的是，`Tuples` 可以包含不同类型的数据，而 `List`只能包含同类型的数据。`Tuples` 在方法需要返回多个结果时非常有用。（`Tuple` 对应到数学的 `矢量` 的概念）。 

 一旦定义了一个元组，可以使用 **_索引**的方式来访问元组的元素（矢量的分量，注意和数组不同的是，元组的索引从1开始），如_1,_2。 

```scala
val pair=(99,"Luftballons")
println(pair._1)
println(pair._2)
```

 元组的实际类型取决于它的分量的类型，比如上面 `pair` 的类型实际为 `Tuple2[Int,String]` ， 如果是含三个元素的元组则是Tuple3，目前 Scala 支持的元组的最大长度为 22（官网说的）

##### 模式匹配

在元组中声明的元素外部可以直接访问到（表面上是这样）：

```scala
val ingredient = ("Sugar" , 25)
val (name, quantity) = ingredient
println(name) // Sugar
println(quantity) // 25
```

#### 混合模式Mixins

类似于java中的extends...implements，在这里的语法为extends...with，即继承一个class实现多个trait

```scala
abstract class A {
  val message: String
}
class B extends A {
  val message = "I'm an instance of class B"
}
trait C extends A {
  def loudMessage = message.toUpperCase()
}
class D extends B with C

val d = new D
println(d.message)  // I'm an instance of class B
println(d.loudMessage)  // I'M AN INSTANCE OF CLASS B
```

#### 函数--Functions

使用函数进行函数式编程，函数越简单越好，**Scala中函数就是带参数的表达式(当然也可以不带参数)**，左边是参数，右边是表达式，可以赋值给变量：

```scala
val addOne = (x: Int) => x + 1
println(addOne(1)) // 2
val add = (x: Int, y: Int) => x + y
println(add(1, 2)) // 3
val getTheAnswer = () => 42
println(getTheAnswer()) // 42
```

#### 方法--Methods

 方法与函数很相似，但还是有点差别，Scala将其分开，用关键字**def**定义的函数就叫方法。

与传统的**类型 变量名**这样的声明方式不同，Scala的类型总是放在后面**变量名 : 类型**，这在很多其他语言中也有体现，如TypeScript。 这样做的一个好处是便于“ type inference ” ， 同样如果函数需要返回值，它的类型也是定义在参数的后面（实际上每个Scala函数都有返回值，只是有些返回值类型为 `Unit` ，类似于 `void` 类型）。 

使用关键字**def**定义一个方法

```scala
scala> def max(x:Int,y:Int) : Int ={
     | if (x >y) x
     | else
     | y
     | }
max: (x: Int, y: Int)Int
```

 每个 Scala 表达式都有返回结果（这一点和 Java，C# 等语言不同），比如 Scala 的 `if else` 语句也是有返回值的，因此函数返回结果无需使用 `return` 语句。实际上在Scala代码中应当尽量避免使用 `return` 语句。函数的最后一个表达式的值就可以作为函数的结果进行返回。 

方法也可以看做是一个表达式，赋值给变量名（注意与java中的方法不同的是要加个=，有多行语句则使用块{}）

```scala
def add(x: Int, y: Int): Int = x + y
println(add(1, 2)) // 3

def addThenMultiply(x: Int, y: Int)(multiplier: Int): Int = (x + y) * multiplier
println(addThenMultiply(1, 2)(3)) // 9

def name: String = System.getProperty("user.name")
println("Hello, " + name + "!")

def getSquareString(input: Double): String = {
  val square = input * input
  square.toString
}
println(getSquareString(2.5)) // 6.25
```

#### 高阶函数higher-order functions

和js中的高阶函数有的一拼，高阶函数作为函数/方法参数或返回结果来使用，Scala面向函数，高阶函数自然必不可少。

**作为参数来使用**

和java8中的流式思想stream很像，直接把函数作为参数用于对函数内的数据进行处理

```scala
val salaries = Seq(20000, 70000, 40000)
val doubleSalary = (x: Int) => x * 2
val newSalaries = salaries.map(doubleSalary) // List(40000, 140000, 80000)
```

更简化一点，直接传参时写函数：

```scala
val salaries = Seq(20000, 70000, 40000)
val newSalaries = salaries.map(x => x * 2) // List(40000, 140000, 80000)
```

还能更简化，假如只是对内部数据进行处理，可以直接省略参数，用_指代内部数据，进行迭代数据处理：

```scala
val salaries = Seq(20000, 70000, 40000)
val newSalaries = salaries.map(_ * 2)
```

使用**def**声明的方法同样可以直接这样使用,scalac会在编译的时候自动把方法转换为函数

```scala
case class WeeklyWeatherForecast(temperatures: Seq[Double]) {

  private def convertCtoF(temp: Double) = temp * 1.8 + 32

  def forecastInFahrenheit: Seq[Double] = temperatures.map(convertCtoF) // <-- passing the method convertCtoF
}
```

**作为返回值来使用**

这个就复杂一点，直接返回函数

```scala
def urlBuilder(ssl: Boolean, domainName: String): (String, String) => String = {
  val schema = if (ssl) "https://" else "http://"
  (endpoint: String, query: String) => s"$schema$domainName/$endpoint?$query"
}

val domainName = "www.example.com"
def getURL = urlBuilder(ssl=true, domainName)
val endpoint = "users"
val query = "id=1"
val url = getURL(endpoint, query) // "https://www.example.com/users?id=1": String
```

在:声明返回类型时，返回的是(String, String) => String，表名该方法返回一个函数，返回值为(endpoint: String, query: String) => s"$schema$domainName/$endpoint?$query"，使用的时候就是直接返回一个函数，$schema表示直接引用参数

#### 块--Blocks

可以嵌入的表达式，使用{}引入，也就是先执行{}中的指令，最后一行的结果就是表达式的返回值

```scala
println({
  val x = 1 + 1
  x + 1
}) // 3
```

#### ++i与i++?

  **Scala 不支持 `++i` 和 `i++` 运算符，因此需要使用 `i += 1` 来加一 ，需要注意**

#### 访问数组

数组直接使用Array()声明，内部可直接赋初始值

```scala
val args = Array("I","like","scala")
var i=0
while (i < args.length) {
  println (args(i))
  i+=1
}
```

 **Scala 访问数组的语法是使用 `()` 而非 `[]` ，需要注意**

数组也是一个对象， 在 Scala 中，数组和其它普遍的类定义一样，没有什么特别之处，当你在某个值后面使用 `()` 时，Scala 将其翻译成对应对象的 `apply` 方法 

可以指定数组的类型，语法为：**val 变量名=new Array[类型\](容量\)**

```scala
val greetStrings = new Array[String](3)
```

#### 函数式特点--List

Scala 也是一个面向函数的编程语言，面向函数的编程语言的一个特点是，调用某个方法不应该有任何副作用，参数一定，调用该方法后，返回一定的结果，而不会去修改程序的其它状态（副作用）。这样做的一个好处是方法和方法之间关联性较小，从而方法变得更可靠和重用性高。使用这个原则也就意味着需要把变量设成不可修改的，这也就避免了多线程访问的互锁问题。

数组的元素是可以被修改的。如果需要使用不可以修改的序列，Scala 中提供了 `Lists` 类。和 Java 的 `List` 不同，Scala 的 `Lists` 对象是不可修改的。它被设计用来满足函数编程风格的代码。它有点像 Java 的 `String`，`String` 也是不可以修改的，如果需要可以修改的 `String` 对像，可以使用 `StringBuilder` 类。

Scala中为List提供两个方法::与:::（看起来就很奇怪），可以将其看做为操作符

 `:::` 方法将两个列表连接起来 ，因为List不可修改，所以连接起来返回的是新的List

```scala
val oneTwo = List(1,2)
val threeFour = List(3,4)
val oneTwoThreeFour=oneTwo ::: threeFour//(1,2,3,4)
```

  `::` 方法用来向 `List` 中添加一个元素 , `::` 方法（操作符）是右操作符，也就是使用 `::` 右边的对象来调用它的 `::` 方法，Scala 中规定所有以 `：` 开头的操作符都是右操作符 

```scala
val oneTwoThree = 1 :: 2 ::3 :: Nil
println(oneTwoThree)//(1,2,3)
```



#### 集合--Sets/Maps

Scala 语言的一个设计目标是让程序员可以同时利用面向对象和面向函数的方法编写代码，因此它提供的集合类分成了可以修改的集合类和不可以修改的集合类两大类型。比如 `Array`总是可以修改内容的，而 `List` 总是不可以修改内容的。类似的情况，`Scala` 也提供了两种 `Sets` 和 `Maps` 集合类。

比如 Scala API 定义了 `Set` 的 `基Trait` 类型 `Set` （`Trait` 的概念类似于Java中的 `Interface`，所不同的是Scala中的 `Trait` 可以有方法的实现），分两个包定义 `Mutable` （可变）和 `Immutable` （不可变），使用同样名称的子 `Trait` 。下图为 `Trait` 和类的基础关系：

 ![2-3.9-1](https://doc.shiyanlou.com/document-uid162034labid1679timestamp1453867993618.png/wm) 

 缺省情况 `Set` 为 `Immutable Set`，如果你需要使用可修改的集合类（ `Set` 类型），你可以使用全路径来指明 `Set` ，比如 `scala.collection.mutable.Set`  ，Map同理

```
val romanNumeral = Map ( 1 -> "I" , 2 -> "II",
  3 -> "III", 4 -> "IV", 5 -> "V")
println (romanNumeral(4))
```



#### 循环

只有while与foreach循环，foreach便于使用函数式编程，类似于java的lambad表达式，只不过->变为了=>

```scala
val args = Array("I","like","scala")
args.foreach(arg => println(arg))
```

有内味了，更简单一点， 利用 Scala 支持的缩写形式，如果一个函数只有一个参数并且只包含一个表达式，那么你无需明确指明参数。因此上面的代码可以写成： 

```scala
args.foreach( println)
```

#### case Classes

使用关键字**case**声明一个类意味着该类不可变并且可以互相比较

```scala
case class Point(x: Int, y: Int)
val point = Point(1, 2)
val anotherPoint = Point(1, 2)
val yetAnotherPoint = Point(2, 2)
if (point == anotherPoint) {
  println(point + " and " + anotherPoint + " are the same.")
} else {
  println(point + " and " + anotherPoint + " are different.")
} // Point(1,2) and Point(1,2) are the same.

if (point == yetAnotherPoint) {
  println(point + " and " + yetAnotherPoint + " are the same.")
} else {
  println(point + " and " + yetAnotherPoint + " are different.")
} // Point(1,2) and Point(2,2) are different.

```

#### Objects

使用关键字**object**声明一个单例类，唯一声明的一个对象

```scala
object IdFactory {
  private var counter = 0
  def create(): Int = {
    counter += 1
    counter
  }
}
val newId: Int = IdFactory.create()
println(newId) // 1
val newerId: Int = IdFactory.create()
println(newerId) // 2
```

#### 程序入口Main

jvm会寻找到名为Main的object并从main(args:Array[String]):Unit方法开始执行程序





