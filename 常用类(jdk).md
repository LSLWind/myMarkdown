### java.util.Properties加载配置文件( key-value类型和xml类型 )

 https://www.cnblogs.com/lingiu/p/3468464.html 

- [java.lang.Object](../../java/lang/Object.html)  
- - [java.util.Dictionary](../../java/util/Dictionary.html)<K,V>  
  - - [java.util.Hashtable](../../java/util/Hashtable.html)<[Object](../../java/lang/Object.html),[Object](../../java/lang/Object.html)>  
      - java.util.Properties 

可以看到，继承于Hashtable，因此内部使用map进行key-value映射，通过方法load与loadFromXML加载文件(InputStream)，解析文件，内部保存为key-value形式，通过getProperty()获取属性，也可以自己通过setProperty()设置属性

```java
public class LoadSample {  
 2     public static void main(String args[]) throws Exception {  
 3       Properties prop = new Properties();  
 4       FileInputStream fis =   
 5         new FileInputStream("sample.properties");  
 6       prop.load(fis);  
 7       prop.list(System.out);  
 8       System.out.println("\nThe foo property: " +  
 9           prop.getProperty("foo"));  
10     }  
11 }  
```

### org.apache.commons.lang.StringUtils 字符串常用类

* public static boolean contains(String str,char searchChar)
*  public static boolean contains(String str,str searchChar) 

搜索字符/字符串是否存在

* public static boolean containsIgnoreCase(String str,String searchStr)

搜索字符串，忽略大小写

* public static boolean isBlank(String str)
* public static boolean isEmpty(String str)

检查字符是否为“ ” 或者null

* public static String replace(String text,String searchString,String replacement)

 用一个字符串替换另一个字符串中出现的指定字符串。 

* public static String join(Object[] array,String separator);

 用指定分隔符连接数组各个元素。 

* public static boolean endsWith(String str, String suffix)

是否以指定字符串结尾

