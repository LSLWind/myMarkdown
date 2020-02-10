泛型是一种设计与思维方法，在java语法层面上进行了实现，由编译器在编译期间对数据类型进行转换达到将数据结构/算法与数据类型分离的目的，使得同一套数据结构/算法能够应用于各种数据类型并且可以保证类型安全。

#### 类型参数

类型参数相当于运行时确定的数据类型，泛型将类型进行参数化，泛型类处理的数据类型不是固定的，而是将某一个数据类型作为参数传递进去进行处理。这是更高程度的一层抽象，将数据的通用部分抽象出来进行统一的泛化处理，针对抽象、接口、能力编程而不仅仅局限于某一种数据类型。关键点就在于类型参数化。

在java中，要对泛型类声明泛型的类型参数，内部则可以使用所声明的类型参数，可声明的泛型类型参数的通常约定为：

* E：element，集合中的元素，常用在集合数据结构中，如java的Collection
* K,V：key与value，表示map中的键值对
* N：Number，数字
* T：Type，表示类型，如String，Integer，指针类型等，常用于自定义的泛型类当中

当然，上述只是通常的约定，任意大写字母都可以充当类型参数，因为javac会将泛型类转中的类型参数进行填充变为强类型的内部类，并不是去识别这些大写字母。

泛型类的写法就是在声明类的时候使用<\>声明类型参数，多个类型参数之间使用逗号隔开，然后内部就可以使用这些声明的类型参数，编译器会自动将类型参数擦除替换为Object并进行必要的强制类型转换然后生成内部类在交给JVM去处理

#### 基础语法

基本语法为：使用<\>声明时声明类型参数，使用时传递具体强类型参数。

创建泛型类：

```java
class Pair<K,V>{
    private K key;
    private V value;
    public Pair(){
        
    }
    public Pair(K key,V value){
        this.key=key;
        this.value=value;
    }

    public K getKey() {
        return key;
    }
    public V getValue() {
        return value;
    }

    public void setKey(K key) {
        this.key = key;
    }

    public void setValue(V value) {
        this.value = value;
    }
}
```

使用泛型类：

```java
public class Test {
    public static void main(String[] args){
        Pair<Integer,String>pair=new Pair<>();
        pair.setKey(1);
        pair.setValue("x");
        System.out.println(pair.getValue());
    }
}
```

#### 内部原理

java泛型通过擦除实现，类定义中的类型参数会被编译器自动擦除转换成Object再插入必要的强制类型转换交给JVM去处理，程序运行过程中并不会对泛型中的类型参数校验而是在编译器校验，因此在运行时强制类型错误通过泛型在编译期间就可以发现，也就是类型安全。

当然，泛型最广泛的用途是用作容器类

#### 泛型方法

泛型方法即方法传递的参数可以是类型参数，泛型方法的类型参数放在返回值前面，如：

```java
public static <T> int indexOf(T[]arr,T elm){...}
```

方法是不是泛型与类是不是泛型无关，泛型方法传递类型参数时不必指定类型，javac可以自动推断。泛型方法的返回类型也可以是泛型类型。

#### 泛型接口

同泛型类一样，接口也可以是泛型的，implements接口是要指定具体的类型，如：

```java
public interface Comparable<T>{
	public int compareTo(T o);
}
public static CaseInsensitiveComparator implements Comparable<String>{...}
```

#### 类型参数限定

设计泛型时必须仔细考虑类型参数的问题，不能把所有可传递的参数都当做Object来处理，这样设计泛型时泛型类内部的处理就会变得很麻烦，比如要对某种数据类型进行排序，该数据类型则必须实现Comparable接口，因为泛型内部处理的时候需要这个接口定义的比较方法，如果传递的时候当成Object来处理，内部进行强制转换显然是不合适的。java中的泛型可以限定类型参数为某一种具体的类型。

**上界为某个具体类**

类型参数通过关键字extends指定传递的类型参数必须为某个类及其子类，如：

```java
class Pair<K extends Number,V extends Number>{...}
```

该泛型类限定传递的类型参数必须为Number或Number的子类，如果使用时不是，如：

```java
Pair<Integer,String>pair=new Pair<>();
```

则直接会在编译期报错。

**上界为某个接口**

同样使用关键字extends进行指定，如：

```java
public static <T extends Comparable<T>> T max(T[] arr){...}
```

从左至右解读，该泛型方法传递的类型参数为T，且T必须实现了Comparable接口并且该接口的传递参数也为T（也就是T能和同类型进行比较），返回类型为T。

#### 通配符

为了更方便的限定类型参数的传递，java在语法层面引入了通配符，通配符只用一个：?，**进行传递参数时的参数类型限制**，通常是用于泛型方法中传递参数时限制传递类型

**?**

单独的一个?代表无限定通配符，可以匹配任何数据类型，因此，下面两种写法是等价的：

```java
public static <T> void a(C<T> c){...}
public static void a(C<?> c){...}
```

**？extends E**

有限定通配符，表示匹配E及E的子类，如：

```java
class A{
    @Override
    public String toString(){
        return "a";
    }
}
class B extends A{
    @Override
    public String toString(){
        return "b";
    }
}
class C<E>{
    //E是类型参数，不是具体类型，使用new E[len]是行不通的，但是对于同样的泛型类来说，传递类型参数是可行的
    //因为都是javac进行强转，只要传递了就行
    private List<E> list=new ArrayList<>();
    public void add(E e){
        list.add(e);
    }
    public E get(int index){
        return list.get(index);
    }
    public int size(){
        return list.size();
    }
}
public class Test {
    public static void main(String[] args){
        C<B> one=new C<>();
        C<A> two=new C<>();

        one.add(new B());
        two.add(new A());

        addAll(one,two);

        for (int i=0;i<two.size();i++){
            System.out.println(two.get(i));
        }

    }
    //T是类型参数，?extendsT 表示from维护的类型必须是to维护的类型的子类
    public static <T> void addAll(C<? extends T> from,C<T> to){
        for(int i=0;i<from.size();i++){
            to.add(from.get(i));
        }
    }
}
```

java是类型安全的，因此上述两种使用？的写法只能读，不能写，否则会在编译期报错，因为类型参数传递的时候是未知的，假如允许写入，那么在调用方法时可以向泛型容器中写入任意类型数据，因此直接将这种会导致类型安全问题的写法禁止掉，如：

```java
C<? extends Number>b=new C<>();
b.add((Number)1);
```

这种写法直接报错，b不可以进行写入

**？super E**

超类型通配符，表示匹配E及E的父类，通常用于进行灵活的写入或比较





