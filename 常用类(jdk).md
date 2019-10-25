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

