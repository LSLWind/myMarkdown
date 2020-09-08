## Maven常用命令

1. mvn compile 编译源代码
2. mvn test-compile 编译测试代码
3. mvn test 运行测试
4. mvn package 打包，根据pom.xml打成war或jar

> 如果pom.xml中设置 war，则此命令相当于mvn war:war
>  如果pom.xml中设置 jar，则此命令相当于mvn jar:jar

1. mvn -Dtest package 打包但不测试。完整命令为：mvn -D maven.test.skip=true package
2. mvn install 在本地Repository中安装jar
3. mvn clean 清除产生的项目
4. mvn eclipse:eclipse 生成eclipse项目
5. mvn idea:idea 生成idea项目
6. mvn eclipse:clean 清除eclipse的一些系统设置
7. mvn dependency:sources 下载源码

#### depencyManagement

用于进行依赖版本管理，在我们项目顶层的POM文件中，我们会看到dependencyManagement元素。通过它来管理jar包的版本，让子项目中引用一个依赖而不用显示的列出版本号。Maven会沿着父子层次向上走，直到找到一个拥有dependencyManagement元素的项目，然后它就会使用在这个dependencyManagement元素中指定的版本号。
这样做的好处：统一管理项目的版本号，确保应用的各个项目的依赖和版本一致，才能保证测试的和发布的是相同的成果，因此，在顶层pom中定义共同的依赖关系。同时可以避免在每个使用的子项目中都声明一个版本号，这样想升级或者切换到另一个版本时，只需要在父类容器里更新，不需要任何一个子项目的修改；如果某个子项目需要另外一个版本号时，只需要在dependencies中声明一个版本号即可。子类就会使用子类声明的版本号，不继承于父类版本号。

  dependencyManagement里只是声明依赖，并不实现引入，因此子项目需要显示的声明需要用的依赖。如果不在子项目中声明依赖，是不会从父项目中继承下来的；只有在子项目中写了该依赖项，并且没有指定具体版本，才会从父项目中继承该项，并且version和scope都读取自父pom;另外如果子项目中指定了版本号，那么会使用子项目中指定的jar版本。

### Maven生命周期

Maven 有以下三个标准的生命周期：

- **clean**：项目清理的处理
- **default(或 build)**：项目部署的处理
- **site**：项目站点文档创建的处理

每个生命周期中都包含着一系列的阶段(phase)。这些 phase 就相当于 Maven 提供的统一的接口，然后这些 phase 的实现由 Maven 的插件来完成。

我们在输入 mvn 命令的时候 比如 **mvn clean**，clean 对应的就是 Clean 生命周期中的 clean 阶段。但是 clean 的具体操作是由 **maven-clean-plugin** 来实现的。

所以说 Maven 生命周期的每一个阶段的具体实现都是由 Maven 插件实现的。

Maven 实际上是一个依赖插件执行的框架，每个任务实际上是由插件完成。Maven 插件通常被用来：

- 创建 jar 文件
- 创建 war 文件
- 编译代码文件
- 代码单元测试
- 创建工程文档
- 创建工程报告

maven的生命周期，其实是对项目构建生命周期的抽象与统一。命令行的输入，就是对应了生命周期，例如mvn package就是执行默认生命周期阶段package。以下的模板方法可以体现出maven生命周期的概念：

```java
public abstract class AbstractBuild {
    public void build() {
        initialize();
        compile();
        test();
        package();
        integrationTest();
        deploy();
    }
    protected abstract void initialize();
    protected abstract void compile();
    protected abstract void test();
    protected abstract void package();
    protected abstract void integrationTest();
    protected abstract void deploy();
}
```

maven的生命周期不是一个整体，而是有三套相互独立的生命周期，分别是clean(清理项目)、default(构建项目)和site(建立项目站点)。而每个生命周期有包含了不同的阶段，下面来列出（阶段是按执行顺序来排列的）：

- clean：pre-clean/clean/post-clean
- default：validate/initialize/generate-sources/process-sources/generate-resources/process-resources/compile/process-classes/generate-test-sources/process-test-sources/generate-test-resources/process-test-resources/test-compile/process-test-classes/test/prepare-package/package/pre-integration-test/integration-test/post-integration-test/verify/install/deploy
- site：pre-site/site/post-site/site-deploy

maven的三个生命周期之间是独立的，但是一个生命周期的每个阶段是前后依赖的。例如，当执行mvn clean时，实际上执行的是pre-clean和clean两个阶段。当执行mvn install时，实际上执行了install前的所有阶段，但它不会执行clean生命周期。
再回到上面的模板方法，maven的生命周期其实是抽象的，本身不会做任何的工作。实际工作都是交给插件来完成的，如package阶段的任务会由maven-jar-plugin来完成。

## build

### 插件plugin

**插件目标**

对于插件而言，为了能够代码复用，往往都具有多个功能。比如maven-dependency-plugin，它能分析项目依赖，能列出项目额依赖树，还能够列出项目已解析的依赖等等。我们不能够为每个功能来编写一个插件，因为这样代码无法复用。所以把这些功能聚集在一个插件里，每个功能就是一个目标。

**插件绑定**

```xml
<plugin>
    <groupId>org.apache.avro</groupId>
    <artifactId>avro-maven-plugin</artifactId>
    <version>1.7.6</version>
    <dependencies>
        <!--插件运行所需要的依赖-->
    </dependencies>
    <!--插件运行所需要的配置-->
    <configuration>
        <sourceDirectory>${project.basedir}/source/</sourceDirectory>
        <outputDirectory>${project.basedir}/target/</outputDirectory>
    </configuration>
    <!--插件目标-->
    <executions>
        <execution>
            <phase>generate-sources</phase>
            <goals>
                <goal>schema</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

使用groupId、artifactId和version三个标签指定使用的maven插件，phase标签表示生命周期的某个阶段，goal标签设置插件的目标，configuration标签用来对插件进行配置(不同的插件有不同的配置项)。

**内置绑定**

使用maven时，几乎不用配置便可以构建项目。这是因为maven为一些重要的生命周期阶段预先绑定了很多插件的目标。例如clean生命周期的clean阶段，预先与maven-clean-plugin:clean绑定。default生命周期的compile阶段，预先与maven-compiler-plugin:compile阶段绑定。我们可以在pom重新配置内置插件。

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-compiler-plugin</artifactId>
    <version>3.2</version>
    <configuration>
        <source>1.8</source>
        <target>1.8</target>
    </configuration>
</plugin>
```

上面的插件设置了jdk版本为1.8（默认为1.5）。
运行maven，在控制台上可以看到生命周期阶段与插件目标的绑定关系。

所以使用插件根导入依赖一样，一般遵循插件开发者的意图使用即可（对应的可以配置各种文件供插件读取配置以完成使用者所需要的操作）



```
spring.config.additional-location=
```



## 常见问题

### omitted for xxx

代表依赖冲突，将pom.xml中原有的低版本依赖删除即可

