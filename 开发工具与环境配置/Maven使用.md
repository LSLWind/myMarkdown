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
