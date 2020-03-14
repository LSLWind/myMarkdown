### 依赖

```xml
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>fastjson</artifactId>
    <version>1.2.46</version>
</dependency>
```

### 序列化与反序列化

fastjson入口类是com.alibaba.fastjson.JSON, 主要API是JSON.toJSONString和parseObject，使用fastjson要注意要转换的类必须有默认的无参构造方法。

**序列化**

```java
String jsonString = JSON.toJSONString(obj);
```

**反序列化**

```java
VO vo = JSON.parseObject("jsonString", VO.class);
```

**泛型反序列化**

```java
List<VO> list = JSON.parseObject("jsonString", new TypeReference<List<VO>>(){});
```

### 主要API:

```java
public static final Object parse(String text); // 把JSON文本parse为JSONObject或者JSONArray 
public static final JSONObject parseObject(String text)； // 把JSON文本parse成JSONObject    
public static final <T> T parseObject(String text, Class<T> clazz); // 把JSON文本parse为JavaBean 
public static final JSONArray parseArray(String text); // 把JSON文本parse成JSONArray 
public static final <T> List<T> parseArray(String text, Class<T> clazz); //把JSON文本parse成JavaBean集合 
public static final String toJSONString(Object object); // 将JavaBean序列化为JSON文本 
public static final String toJSONString(Object object, boolean prettyFormat); // 将JavaBean序列化为带格式的JSON文本 
public static final Object toJSON(Object javaObject); 将JavaBean转换为JSONObject或者JSONArray。
```

SerializeWriter：相当于StringBuffer
JSONArray：相当于List
JSONObject：相当于Map

JSON.DEFFAULT_DATE_FORMAT = "yyyy-MM-dd";  
JSON.toJSONString(obj, SerializerFeature.WriteDateUseDateFormat);

反序列化能够自动识别如下日期格式：

ISO-8601日期格式
yyyy-MM-dd
yyyy-MM-dd HH:mm:ss
yyyy-MM-dd HH:mm:ss.SSS
毫秒数字
毫秒数字字符串
NET JSON日期格式
new Date(198293238)

3 过滤属性

需要根据不同的环境返回定制化返回属性时，可以使用SimplePropertyPreFilter。

```java
SimplePropertyPreFilter的代码接口如下：
public class SimplePropertyPreFilter implements PropertyPreFilter {      
      public SimplePropertyPreFilter(String... properties){
          this(null, properties);
      }

      public SimplePropertyPreFilter(Class<?> clazz, String... properties){
          ... ...
      }
    
      public Class<?> getClazz() {
          return clazz;
      }
    
      public Set<String> getIncludes();
    
      public Set<String> getExcludes();

  ...
  }
```


你可以配置includes、excludes。当class不为null时，针对特定类型；当class为null时，针对所有类型。
当includes的size > 0时，属性必须在includes中才会被序列化，excludes优先于includes。

使用方法：
在1.1.23版本之后，JSON提供新的序列化接口toJSONString，如下：
String JSON.toJSONString(Object, SerializeFilter, SerializerFeature...);

用法如下：

```java
 User user = new User();
 user.setName("shamo");
 user.setPwd("123");
 user.setAge(25);

 SimplePropertyPreFilter filter = new SimplePropertyPreFilter(User.class, "pwd", "age");
 Set<String> includes = filter.getIncludes();
 Set<String> excludes = filter.getExcludes();
 includes.add("name");
 excludes.add("pwd");
 String json = JSON.toJSONString(user, filter);
 System.out.println(json);
输出:
```


{"age":25,"name":"shamo"}

### 修改属性

可以使用JSONObject对json形式的字符串进行任意的增删改
用法如下:

```java
String jsonString = JSON.toJSONString(user);
JSONObject jsonObject = JSON.parseObject(jsonString);
jsonObject.put("gender", "male");
jsonObject.put("pwd","456");
jsonObject.remove("name");
String json = JSON.toJSONString(jsonObject);
System.out.println(json);
```


输出：{"pwd":"456","age":25,"gender":"male"}

