IDEA这个工具非常强大，有很多非常强大的功能，在这里记录一下

#### 根据代码生成类图

参考：https://blog.csdn.net/hy_coming/article/details/80741717

1.打开设置 File->Setting或windows下按Ctrl+Alt+S
2.选择Tools→Diagram

 ![img](https://img-blog.csdn.net/20180620093420200?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2h5X2NvbWluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70) 

如上所示，我们主要关心的只有Java Class Diagrams下面的几个单选框，分别对应红字部分，一般的UML类图只需要知道成员变量、构造器和方法（前面三个），其他的随意。

3.选择需要的类文件或包
4.按Ctrl + Shift + Alt + U或Ctrl + Alt + U或右键选择，生成类Uml关联图，如下图：

 ![img](https://img-blog.csdn.net/2018062009435730?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2h5X2NvbWluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70) 

局限性：虽然这个很是方便，但是也有他自己的局限性，首先这个功能只能是根据类来自动生成的，所以对于设计类的时候就不行了，还是需要正规的UML图软件。

#### 查看源码

 Ctrl + Shift+i ，此时出现一个小窗口，使用Ctrl + Ente查看完整源码

### 分析包

先查找到任意一个接口，单击Navigate→Type Hierarchy。会出现该类的实现类型层次结构

### 连接数据库

选中view—>点击Tool Windows—>选择Database

点击+号—>选择你需要连接的数据库类型—>Data Source

![](https://img-blog.csdn.net/20180309133254547?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzgzNzcxOTA=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

### 编辑数据库

**编辑表**

右键单击表-->Modify table