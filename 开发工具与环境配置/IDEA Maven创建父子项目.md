通过 maven 可以创建父子-聚合项目。 所谓的父子项目，即有一个父项目，有多个子项目。
这些子项目，在业务逻辑上，都归纳在这个父项目下，并且一般来说，都会有重复的jar包共享。
所以常用的做法会把重复的 jar 包都放在父项目下进行依赖，那么子项目就无需再去依赖这些重复的 jar 包了。 

#### 使用IDEA创建maven项目

创建的maven项目就是父项目，一般只提供给子项目依赖并将子项目进行聚合，并不会实际的写代码，创建父项目时要创建空项目，父项目仅仅是根，不需要其他的模板框架（如创建web项目或者springboot）

创建之后要在pom.xml中添加

```xml
<!--包为pom，即父子聚合项目-->
<packaging>pom</packaging>				 
```

完整pom.xml为：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>lsl</groupId>
    <artifactId>maven-modulesProject</artifactId>
    <version>1.0-SNAPSHOT</version>

    <!--包为pom，即父子聚合项目-->
    <packaging>pom</packaging>

    <dependencies>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.11</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>cn.hutool</groupId>
            <artifactId>hutool-all</artifactId>
            <version>4.3.1</version>
        </dependency>
    </dependencies>

</project>
```

#### 创建子项目maven module

#### ![创建子项目](https://stepimagewm.how2j.cn/9294.png) 

子项目就是实际要写代码的项目，创建Maven项目时可使用快速原型 org.apache.maven.archetypes:maven-archetype-quickstart 进行创建，也可以手动更改父子项目的pom.xml设置项目间的父子依赖关系

 ![选择maven项目](https://stepimagewm.how2j.cn/9289.png) 

在子项目中进行一次测试：

```java
import cn.hutool.core.date.DateUtil;
import java.util.Date;
public class App 
{
    public static void main(String[] args) {
        String dateStr = "2012-12-12 12:12:12";
        Date date = DateUtil.parse(dateStr);
        System.out.println(date);
    }
}
```

在子项目的pom.xml中并没有导入cn.hutool依赖但仍然能够使用

#### 父子项目中的pom.xml

观察子项目中的pom.xml为：

```xml
<!--父类--> 
<parent>
        <artifactId>maven-modulesProject</artifactId>
        <groupId>lsl</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
<!--模块版本与模块名-->
    <modelVersion>4.0.0</modelVersion>

    <artifactId>childMavenProject</artifactId>
```

在看一下父项目的pom.xml：

```xml
    <groupId>lsl</groupId>
    <artifactId>maven-modulesProject</artifactId>
    <version>1.0-SNAPSHOT</version>
<!--所包含模块-->
    <modules>
        <module>childMavenProject</module>
    </modules>
```

子项目中多出来了标签<parent\>，父项目中多出了标签<modules\>，这两种标签即代表了父子项目之间的逻辑依赖关系，其它标签也相应改变，通过扫描父母录以及子目录生成逻辑关系并完成功能

无论用什么工具，使用maven最关键的还是在于配置文件pom.xml，创建父子项目的关键在于标签<parent\>、

<modules\>、<module\>、 <groupId\>、<artifactId\>、<version\>、<modelVersion\>，定义好了逻辑关系即可

