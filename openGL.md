大学课程，新坑

### 基础

openGL安装glut/glew等

**https://www.jianshu.com/p/b49fd7fe8b5c**

glut太老--吐槽，网上教程都不使用glut了，教材还在使用，现在使用glut/freeglut/glew

freeglut安装与glew安装：https://blog.csdn.net/xuguangsoft/article/details/8002375

GLEW是一个跨平台的C++扩展库，基于OpenGL图形接口。GLEW能自动识别你的平台所支持的全部OpenGL高级扩展涵数。也就是说，只要包含一个glew.h头文件，你就能使用gl,glu,glext,wgl,glx的全部函数。GLEW支持目前流行的各种操作系统（including Windows, Linux, Mac OS X, FreeBSD, Irix, and Solaris）使用OpenGL的朋友都知道，window目前只支持OpenGL1.1的涵数，但 OpenGL现在都发展到2.0以上了，要使用这些OpenGL的高级特性，就必须下载最新的扩展，另外，不同的显卡公司，也会发布一些只有自家显卡才支 持的扩展函数，你要想用这数涵数，不得不去寻找最新的glext.h,有了GLEW扩展库，你就再也不用为找不到函数的接口而烦恼，因为GLEW能自动识 别你的平台所支持的全部OpenGL高级扩展h函数。也就是说，只要包含一个glew.h头文件，你就能使用gl,glu,glext,wgl,glx的全部函数。

装吧，我装了，一枪秒了，有什么好说的

OpenGL是一个状态机）。OpenGL使用状态方案的原因是渲染是一个非常复杂的任务，不能仅仅通过一个函数接受几个参数来完成（一个合理设计的函数是不会接受大量的参数的）。对于渲染效果的设置我们需要定义shader着色器，buffer缓冲还有各种各样的flag标志变量。另外，我们也经常想保存一些相同的配置在多个渲染操作中使用（比如：如果我们从来不需要禁掉深度检测depth test，我们没必要在每一个渲染回调中来明确定义它）。这也是为什么多数的渲染操作配置都是通过在OpenGL状态机中设置flag标志变量和值来完成，而且渲染回调本身通常也被局限于几个参数，参数解决需要绘制的定点数量和他们的偏移量。调用一个改变状态的函数后，具体的配置保持不变，直到下次再调用这个相同的函数再次改变状态和配置。上面的函数设置了当帧缓存（帧缓存后面还会介绍）清空后要使用的颜色值。颜色值有四个通道（RGBA）,使用单位化的值0.0-1.0来表示。

报错：找不到freeglut.lib/dll balabalabala......

将对应的编译过后的dll,lib等放到System32 /SysWow64/   VS的VC目录下。。。各种放

### glut

#### 创建窗口函数

* glutInit(&argc, argue);

调用这个函数来初始化GLUT.参数可以直接引用command line的，而且有一些有用的选项，比如：‘-sync’和‘-gldebug’，可以禁掉X窗口的异步特征并分别自动检查和显示GL错误。

* glClearColor(0.0f, 0.0f, 0.0f, 0.0f);   设置清屏颜色

* glClear(GL_COLOR_BUFFER_BIT); 刷新颜色缓冲区

* glFlush(); 刷新指令队列

* glutInitDisplayMode(GLUT_DOUBLE | GLUT_RGBA);

这里配置一些GLUT的选项设置。GLUT_DOUBLE在多数渲染结束后开启双缓冲机制（维护两个图像缓冲数据，屏幕显示一副图像时在后台同时绘制另一份图像缓冲数据，交替显示）和颜色缓冲。我们通常需要这两个设置，还有其他的选项设置后面会继续介绍。

* glutInitWindowSize(960, 640); // 窗口尺寸
* glutInitWindowPosition(200, 200); // 窗口位置
* glutCreateWindow("Tutorial 01"); // 创建窗口，设置窗口标题
* glutDisplayFunc(RenderScenceCB);  传递函数指针，回调函数执行
* glutMainLoop();  状态机循环，开启程序，用在结尾

开启窗口：

```c++
#include<gl/glut.h>
void myDisplay(void) {
	glClearColor(0, 0, 0, 1);//清屏颜色，也就是设置背景
	glClear(GL_COLOR_BUFFER_BIT);//刷新颜色缓冲区
	glFlush();//刷新指令队列

}
int main(int argc,char *argv[]) {
	glutInit(&argc, argv);//初始化
	glutInitDisplayMode(GLUT_RGB | GLUT_SINGLE);//显示模式
	glutInitWindowPosition(100, 100);//位置
	glutInitWindowSize(400, 400);//大小
	glutCreateWindow("第一个OpenGL程序");//创建窗口，设置标题

	glutDisplayFunc(&myDisplay);//执行回调函数

	glutMainLoop();//指令队列
	return 0;
}
```

#### 图形绘制

图形绘制语句一般放在	

glClear(GL_COLOR_BUFFER_BIT);//刷新颜色缓冲区
glFlush();//刷新指令队列

之间，用于立马执行绘制

基本图元的绘制必须放在：

* glBegin(GLenum mode)：开始绘制图元，mode指明图形类型，标志点列表的开始
* glEnd()：标志点列表的结束

之间，中间指明点列表，会按点顺序连线

#### 窗口坐标系

开启的窗口，以中间为(0,0)点，x轴长度范围为[-1,1]，y轴长度范围为[-1,1]因此绘制点时的参数设置大小在[-1,1]之间。

 在OpenGL中，任何事物都在3D空间中，而屏幕和窗口却是2D像素数组，这导致OpenGL的大部分工作都是关于把3D坐标转变为适应你屏幕的2D像素。3D坐标转为2D坐标的处理过程是由OpenGL的图形渲染管线（Graphics  Pipeline，大多译为管线，实际上指的是一堆原始图形数据途经一个输送管道，期间经过各种变化处理最终出现在屏幕的过程）管理的。图形渲染管线可以被划分为两个主要部分：第一部分把你的3D坐标转换为2D坐标，第二部分是把2D坐标转变为实际的有颜色的像素。 

一旦你的顶点坐标已经在顶点着色器中处理过，它们就应该是**标准化设备坐标**了，标准化设备坐标是一个x、y和z值在-1.0到1.0的一小段空间。任何落在范围外的坐标都会被丢弃/裁剪，不会显示在你的屏幕上。下面你会看到我们定义的在标准化设备坐标中的三角形(忽略z轴)：

![NDC](https://learnopengl-cn.github.io/img/01/04/ndc.png)

与通常的屏幕坐标不同，y轴正方向为向上，(0, 0)坐标是这个图像的中心，而不是左上角。最终你希望所有(变换过的)坐标都在这个坐标空间中，否则它们就不可见了。

你的标准化设备坐标接着会变换为屏幕空间坐标(Screen-space Coordinates)，这是使用你通过glViewport函数提供的数据，进行视口变换(Viewport Transform)完成的。所得的屏幕空间坐标又会被变换为片段输入到片段着色器中。

#### 点的绘制

* glPointSize(GLfloat size)：指明点的大小，放在glBegin()上面
* glVertex{234}{sifd}[V\](Type cords)：定义一个顶点
* glColor3f(GLfloat red,GLfloat green,GLfloat blue)：定义点的颜色，rgb

点的参数mode为GL_POINTS

```c++
	glClearColor(0, 0, 0, 1);
	glClear(GL_COLOR_BUFFER_BIT);

	glPointSize(2.0);
	glBegin(GL_POINTS);
		glColor3f(1.0, 0.0, 1.0);
		glVertex2f(0.5,0.5);
	glEnd();

	glFlush();
```

#### 直线绘制

**GL_LINES**：两点之间的直线

```c++
	glClearColor(0, 0, 0, 1);
	glClear(GL_COLOR_BUFFER_BIT);

	glPointSize(2.0);
	glBegin(GL_LINES);
		glColor3f(1.0, 0.0, 1.0);
		glVertex2f(0.5,0.5);
		glVertex2f(-0.5, 0.5);
	glEnd();

	glFlush();
```

**GL_LINE_STRIP**：点之间的折线段

```c++
	glClear(GL_COLOR_BUFFER_BIT);

	glPointSize(2.0);
	glBegin(GL_LINE_STRIP);
		glColor3f(1.0, 0.0, 1.0);
		glVertex2f(0.5,0.5);
		glVertex2f(-0.5, 0.5);
		glVertex2f(-0.5, 0.1);
	glEnd();

	glFlush();
```

**GL_LINE_LOOP**：封闭直线

```c++
	glClearColor(0, 0, 0, 1);
	glClear(GL_COLOR_BUFFER_BIT);

	glPointSize(2.0);
	glBegin(GL_LINE_LOOP);
		glColor3f(1.0, 0.0, 1.0);
		glVertex2f(0.5,0.5);
		glVertex2f(-0.5, 0.5);
		glVertex2f(-0.5, 0.1);
	glEnd();

	glFlush();
```

* glLineWidth(GLfloat width)：指定线宽大小

#### 直线的点划模式

需要启动：

```c
glEnable(GL_LINE_STIPPLE);
```

点画设置：

```c++
glLineStipple(factor,pattern);
```

factor为重复因子，表示画一条线段周期重复的数字

pattern为16位进制数，从低到高，设置线段绘制模式，1表示画像素，0表示不画像素

```
	glEnable(GL_LINE_STIPPLE);
	glLineStipple(3,0x00FF);

	glLineWidth(4.0);
	glBegin(GL_LINE_LOOP);
	glColor3f(1.0, 0.0, 1.0);
	glVertex2f(0.5, 0.5);
	glVertex2f(-0.5, 0.5);
	glVertex2f(-0.5, 0.1);
	glEnd();
```

#### 绘制多边形与填充图元

多边形是由点集来确定的，模式设为GL_POLYGON即可

```
glLineWidth(4.0);
	glBegin(GL_POLYGON);
	glColor3f(1.0, 0.0, 1.0);
	glVertex2f(0.5, 0.5);
	glVertex2f(-0.5, 0.5);
	glVertex2f(-0.5, -0.5);
	glVertex2f(0.5, -0.5);
	glEnd();
```

图元要先开启多边形模式：

* glPolygonMode（GLenum face，GLenum mode）：开启绘制多边形模式

face有三种，GL_FRONT、GL_BACK、GL_FRONT_AND_BACK，绘制前面、背面还是都绘制

mode有三种，GL_POINT、GL_LINE、GL_FILL，以点、线、填充模式绘制

#### 圆的绘制

没有绘制圆的函数，有绘制球的，可以替代

```
glutWireSphere(0.5, 30, 30);
```

第一个为半径，剩下是经纬度

#### 反走样/抗锯齿实现

开启抗锯齿

```
glEnable(GL_BLEND);
glBlendFunc(GL_SRC_ALPHA, GL_ONE_MINUS_SRC_ALPHA);
```

之后设置平滑曲线，对点、线、多边形进行平滑处理

```
glEnable(GL_POINT_SMOOTH);
glEnable(GL_LINE_SMOOTH);
glEnable(GL_POLYGON_SMOOTH);
```

### 动画效果

#### 单/双缓存

双缓存技术：将图形在内存中绘制出来，然后一次性赋值到屏幕上的画布中，双缓存中，前缓存：屏幕正在显示的图形，后缓存：内存中正在生成的图形，单双缓存交替可实现动画效果

双缓存设置：

在main中设置

```
glutInitDisplayMode(GLUT_RGB | GLUT_DOUBLE);
```

在绘图函数后加上

```
glutSwapBuffers();
```

即开启双缓存

### 鼠标键盘交互

#### 普通键盘响应

* glutKeyboardFunc(void *func(unsigned char key,int x,int y))：注册键盘响应函数，key为一个ASCII码，x,y为当前鼠标坐标，func是接收后所调用的处理函数，这个要自己写并传进去