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

