### （一）Activity 布局

#### 组件常用属性

> * 属性值match_parent:组件填充以扩展父容器
> * 属性值wrap_content:组件内容扩展到整个组件
> * id:组件唯一标识
> * background:背景
> * gravity:组件对齐方式，比较多，有top,left,fill等，可多选

#### 1.线性布局LinearLayout

线性布局即组件垂直或水平排列

##### 基本属性

> * orientation: 组件排列方式，horizontal(水平)，vertical(垂直)
> * layout_width:宽度
> * layout_height:高度
> * weight:权重，组件占有父容器的相对比例，为int
> * divider:组件之间的分割线，该分割线是图片

**注意:**当 android:orientation="vertical" 时， 只有水平方向的设置才起作用，垂直方向的设置不起作用。 即：left，right，center_horizontal 是生效的。 当 android:orientation="horizontal" 时， 只有垂直方向的设置才起作用，水平方向的设置不起作用。 即：top，bottom，center_vertical 是生效的。**

#### 2.相对布局RelativeLayout

相对布局即组件根据属性自动调整组件间相对位置，同时可设置margin与padding细调组件位置

##### 基本属性

> * ignoreGravity:设置为true则不受gravity属性影响

##### 在该布局下的组件属性设置

> * layout_alignParentXXX:XXX可为left,right,top,bottom,centerHorizontal,centerVertical,centerInParent(中间位置)，表示根据父容器定位
> * layout_XXX:XXX可为toLeftOf,toRightOf,above,below,alignTop/Bottom/Left/Right，表示根据兄弟组件定位
> * layout_marginXXX:XXX可为Left,Right,Top,Bottom,表示组件与父容器边框之间的边距，也称偏移量
> * paddingXXX:XXX可为Left,Right,Top,Bottom,表示组件内部元素与组件边框之间的边距，也称填充量

父容器定位示意图:

![img](https://www.runoob.com/wp-content/uploads/2015/07/44967125.jpg)

兄弟组件定位示意图

![img](https://www.runoob.com/wp-content/uploads/2015/07/23853521.jpg)

#### 3.表格布局TableLayout

表格布局即组件按表格(行列)方式排版

**表格布局必须添加行组件TableRow，每个行组件再添加列组件,TableRow默认填充整个TableLayout**

##### 常用属性（针对所有TableRow):

> * collapseColumns:int,被隐藏的组件的列号，列号索引从0开始，下同
> * shrinkColumns:设置允许收缩的列号，收缩即自动收缩组件以适应父容器而不覆盖其它组件
> * stretchColumns:设置允许拉伸的列号，拉伸即自动拉伸以适应填充整个父容器
> * layout_span:设置合并单元格数量，即一个组件占据的单元格数量

#### 4.帧布局FrameLayout

帧布局即默认将组件放在容器左上角，没有任何定位方式

**前景图像:永远处于帧布局最上方不会被覆盖的图片**

##### 常用属性

> * foreground:设置前景图像
> * foregroundGravity:设置前景图像的显示位置

通过设置前景图像可以完成图片的叠加式显示，加上定时时间刷新信息可以控制图片的动态移动

#### 5.网格布局GridLayout

与表格布局类似，不过更加灵活好用

##### 常用属性

> * orientation:vertical或horizontal设置排列方式，垂直或水平
> * layout_gravity:设置对其方式
> * rowCount: int,设置行数
> * columnCount:设 int，设置列数

##### 该布局下的组件属性

> * layout_row:该组件所在的行
> * layout_column:该组件所在的列
> * layout_rowSpan:设置组件横跨几行
> * layout_columnSpan:设置组件横跨几列

**使用网格布局最好将其设置为根布局，否则可能不显示组件(迷惑)**

### （二）常用组件

#### 1.显示文本TextView

##### 常用属性

> * textXXX:XXX可为Color，Style，Size，设置文本
> * shadowColor:设置阴影颜色，需要与shadowRadius一起使用
> * shadowRadius:设置阴影模糊程度，0.1为字体颜色（重合），建议为3.0
> * shadowDx:阴影在水平方向上的偏移，即x轴偏移量
> * shadowDy:阴影在竖直方向上的偏移，即y轴偏移量

##### 常用技巧

###### ①带边框的TextView

设置背景为一个自己编写的ShapeDrawable即可，实际上是设置background为一个Drawable资源（如图片），因此设置合适的背景图片即可，自己画一个shape资源文件也可以

###### ②带图片的TextView

使用属性drawableXXX即可,XXX可为Top,Buttom,Left,Right,表示在文字上下左右显示指定图片,还可以使用drawablePadding指定文字与图片的边距

#### 2.输入文本EditText

##### 常用属性

> * hint:默认的提示文本
> * textColorHint:默认提示文本颜色
> * selectAllOnFocus:设置编辑文本时开始选取所有文本
> * inputType:限制输入的文本类型，如textEmailAddress，number等
> * singleLine:为true时不自动换行，默认为false
> * windowSoftInputMode:设置编辑文本时自动弹出键盘的模式，有多个值，可多选

#### 3.自动提示文本输入框AutoCompleteTextView

继承自EditText，输入内容时会自动显示相关文本以供选择

数据源来源于ArrayAdapter，泛型类

构造函数为public ArrayAdapter (Context context, int textViewResourceId, List\<T> objects)

context为上下文环境，通常为this

textViewResourceId为textView的id，即适配器绑定的textView，通常为指定的AutoCompleteTextView，类型为int，使用R.id.XXX即可

objects为绑定的数据源，匹配数据来源于objects

#### 4.消息提示框Toast(吐司)

在屏幕中显示一个消息提示框,没任何按钮,也不会获得焦点一段时间过后自动消失，用于消息提示，也可以用于调试程序

**Toast.makeText(MainActivity.this, "提示的内容", Toast.LENGTH_LONG);** 第一个是上下文对象，对二个是显示的内容，第三个是显示的时间，只有LONG和SHORT两种会生效

即绑定Activity，显示内容，显示时间，然后通过show()方法进行显示

可使用方法setGravity()定位toast的显示位置

#### 5.单选按钮RadioButton/多选框CheckBox

单选按钮RadioButton要放在RadioGroup中，使用属性orientation设置垂直或水平

复选框与按钮没啥区别，重点在于CheckBox是否被选中

 1.为每个CheckBox添加事件：setOnCheckedChangeListener 2.弄一个按钮，点击后，对每个checkbox进行判断:isChecked()；

#### 6.开关按钮/开关ToggleButton/Switch

##### 开关按钮ToggleButton

**常用属性**

> * disavledAlpha:按钮禁用时的透明度
> * textOff:按钮关闭时的显示文字
> * textOn:按钮打开时的显示文字

##### 开关Switch(滑块)

**常用属性**

> * showText:是否显示文字
> * switchMinWidth:开关最小宽度
> * switchPadding:滑块内文字间隔
> * track:底部图片
> * thumb:滑块图片

#### 7.滑动条SeekBar

##### 常用属性

> * max:滑动条的最大值
> * progress:滑动条的当前值
> * onProgressChanged:进度改变时触发的事件
> * onStartTrackingTouch:按住SeekBar时触发的事件
> * onStopTrackingTouch:放开SeekBar时触发的事件

#### 8.适配器Adapter

适配器用来向View填充数据

##### MVC模式

- **Model**：通常可以理解为数据,负责执行程序的核心运算与判断逻辑,,通过view获得用户 输入的数据,然后根据从数据库查询相关的信息,最后进行运算和判断,再将得到的结果交给view来显示
- **view**:用户的操作接口,说白了就是GUI,应该使用哪种接口组件,组件间的排列位置与顺序都需要设计
- **Controller**:控制器,作为model与view之间的枢纽,负责控制程序的执行流程以及对象之间的一个互动

Adapter是控制器的一部分

![img](https://www.runoob.com/wp-content/uploads/2015/09/77919389.jpg)

适配器初始化时（构造）要绑定上下文环境，View组件，数据，组件通过setAdapter()设置适配器

#### 9.列表ListView

ListView需要与适配器配合使用，列表项是适配器绑定的数据

列表项Item点击会触发ItemClick事件，由方法onItemClick()监听

列表项Item长按会触发ItemLongClick事件，由方法onItemLongClick监听

显示效果就是单纯的列表

#### 10.下拉列表选择框Spinner

同样需要与适配器配合使用，列表项为适配器绑定的数据

列表项是用来被选择的，点击时以浮动形式显示所有选项，点击选项时响应事件同时收回选项框

单击选项时触发onItemSelected()方法

```java
public void onItemSelected(AdapterView<?> parent, View view, int position, long id) 
```

parent为适配器，View 为被选择的选项,position为子项位置,id为子项id号

#### 对话框

##### 11.消息提示对话框AlterDialog

AlterDialog不能直接new对象，需要先构造内部Builder对象设置属性，然后使用create()创建对象

- **Step 1**：创建**AlertDialog.Builder**对象；
- **Step 2**：调用**setIcon()**设置图标，**setTitle()**或**setCustomTitle()**设置标题；
- **Step 3**：设置对话框的内容：**setMessage()**还有其他方法来指定显示的内容；
- **Step 4**：调用**setPositive/Negative/NeutralButton()**设置：确定，取消，中立按钮；
- **Step 5**：调用**create()**方法创建这个对象，再调用**show()**方法将对话框显示出来；

提示框有三个按钮，可为每个按钮设置监听，第一个参数为按钮名(String)，第二个参数为监听器对象onClickListener()，重写onClick()

```
setNegativeButton/setPositionButton/setNeutralButton
```

用户点击任意按钮会触发onClick()方法

`public void onClick(DialogInterface dialog, int which)`

dialog为触发事件的对话框，which为触发的按钮id

可通过setView()方法设置View对象为显示内容

##### 12.进度条对话框ProgressDialog

创建ProgressDialog对象,再设置对话框的参数,最后show()出来即可

**常用方法:**

```
setIndeterminate(false);//显示进度
setCancelable(false);//设置不可通过按取消按钮关闭进度条
setProgressStyle(ProgressDialog.STYLE_HORIZONTAL);//设置进度条样式,STYLE_SPINNER为圆形进度条
setProgress();//设置进度条进度
```

一般需要另开启一个线程处理数据然后通知给进度对话框

##### 13.日期选择对话框DatePickerDialog

直接构造对象，使用show即可

构造方法为` DatePickerDialog(Context con,OnDateSetListener call,int year,int month,int day)`

第一个为上下文环境，第二个为日期监听器，对话框确定按钮被按下时会调用下述重写方法

```
public void onDateSet(DatePicker view, int year, int monthOfYear,int dayOfMonth)
```

当点击日期表中的日期时调用DatePicker中的OnDateChanagedListener中的onDateChanaged()方法

##### 14.时间选择对话框TimePickerDialog

同DatePickerDialog一样，日期变为时间（年月日变为小时与分钟）

#### 菜单Menu

每个Activity默认都包含一个Menu对象，因此编写菜单只需要编写菜单项与对应的事件监听即可，有多中不同样式的菜单，加载菜单时重写相应方法即可

##### 15.选项菜单OptionMenu

**常用方法:**

- public boolean **onCreateOptionsMenu**(Menu menu)：调用OptionMenu，在这里完成菜单初始化
- public boolean **onOptionsItemSelected**(MenuItem item)：菜单项被选中时触发，这里完成事件处理

- public void **onOptionsMenuClosed**(Menu menu)：菜单关闭会调用该方法
- public boolean **onPrepareOptionsMenu**(Menu menu)：选项菜单显示前会调用该方法， 可在这里进行菜单的调整(动态加载菜单列表)
- public boolean **onMenuOpened**(int featureId, Menu menu)：选项菜单打开以后会调用这个方法

**加载菜单：**

1. 直接通过编写菜单XML文件，然后调用： **getMenuInflater().inflate(R.menu.menu_main, menu);**
2. 重写onCreateOptionsMenu，参数为menu，menu调用add方法添加 菜单，add(菜单项的组号，ID，排序号，标题)

**菜单项事件响应：**

重写 **onOptionsItemSelected**(MenuItem item)

item通过getItemId()获取id，与选项菜单绑定菜单项时对应

##### 16.上下文菜单ContextMenu

上下文菜单即长按时显示的菜单，可以注册到任意View当中，无法显示图标

- **Step 1**：重写onCreateContextMenu()方法
- **Step 2**：为view组件注册上下文菜单，使用registerForContextMenu()方法,参数是View
- **Step 3**：重写onContextItemSelected()方法为菜单项指定事件监听器

##### 17.子菜单SubMenu

Menu中的子项，可设置item，可调用addSubMenu()添加

##### 18.弹出式菜单PopupMenu

同样设置在xml中设置item，初始化构造加载即可，setOnMenuItemClickListener()设置监听器，参数为PopupMenu.OnMenuItemClickListener()(静态方法，返回监听器)，重写onMenuItemClick()方法

#### 19.状态通知栏Notification

通知栏的基本属性有：

- **con/Photo**：大图标
- **Title/Name**：标题
- **Message**：内容信息
- **Timestamp**：通知时间，默认是系统发出通知的时间，也可以通过setWhen()来设置
- **Secondary Icon**：小图标
- 内容文字，在小图标的左手边的一个文字

状态通知栏主要涉及到2个类：Notification 和NotificationManager

**Notification**：通知信息类，它里面对应了通知栏的各个属性

**NotificationManager**：是状态栏通知的管理类，负责发通知、清除通知等操作，通过系统服务获取。

使用的基本流程：

- **Step 1.** 获得NotificationManager对象： NotificationManager mNManager = (NotificationManager) getSystemService(NOTIFICATION_SERVICE);
- **Step 2.** 创建一个通知栏的Builder构造类： Notification.Builder mBuilder = new Notification.Builder(this);
- **Step 3.** 对Builder进行相关的设置，比如标题，内容，图标，动作等
- **Step 4.**调用Builder的build()方法为notification赋值，返回Notification对象
- **Step 5**.调用NotificationManager的notify()方法发送通知
- **PS:**另外我们还可以调用NotificationManager的cancel()方法取消通知

### （三）事件处理机制

#### 1.事件监听

![img](https://www.runoob.com/wp-content/uploads/2015/07/4109430.jpg)

对于组件来说，事件通过设置监听器进行响应如setOnClickListener(），通常传入一个事件监听接口重写内置方法为事件源注册监听器，例

```java
setOnClickListener(new OnClickListener() {    
            //重写点击事件的处理方法onClick()    
            @Override    
            public void onClick(View v) {    
                //显示Toast信息    
                Toast.makeText(getApplicationContext(), "你点击了按钮", Toast.LENGTH_SHORT).show();    
            }    
        });  
```

通常可以在xml中设置对应的监听器属性完成事件处理

#### 2.事件回调

##### 方法回调

一种将功能定义与功能实现相分开的解耦合思想，对外提供实现接口(方法)，客户可以根据自己的需要对具体问题实现自己的需要，即接口统一，实现不同

##### 使用场景

**1.自定义View**

即自定义组件，然后重写组件内置的相应回调方法，即相当于组件为自己实现监听器，当触发不同动作时执行对应的方法

```java
public class MyButton extends Button{  
    private static String TAG = "呵呵";  
    public MyButton(Context context, AttributeSet attrs) {  
        super(context, attrs);  
    }  
  
    //重写键盘按下触发的事件  
    @Override  
    public boolean onKeyDown(int keyCode, KeyEvent event) {  
        super.onKeyDown(keyCode,event);  
        Log.i(TAG, "onKeyDown方法被调用");  
        return true;  
    }  
  
    //重写弹起键盘触发的事件  
    @Override  
    public boolean onKeyUp(int keyCode, KeyEvent event) {  
        super.onKeyUp(keyCode,event);  
        Log.i(TAG,"onKeyUp方法被调用");  
        return true;  
    }  
  
    //组件被触摸了  
    @Override  
    public boolean onTouchEvent(MotionEvent event) {  
        super.onTouchEvent(event);  
        Log.i(TAG,"onTouchEvent方法被调用");  
        return true;  
    }  
} 
```

**2.实现事件传播**

所有回调方法的返回值类型都为boolean，返回为true时停止传播，为false时继续向外传播，传播范围为**监听器**&rightarrow;**组件内部回调方法**&rightarrow;**Activity的回调方法**

即对于组件来说，先执行监听器内的响应方法，根据返回值然后执行组件内置的回调方法，然后再根据返回值执行Activity内置的同名回调方法

常用方法：

*①在该组件上触发屏幕事件: boolean* **onTouchEvent***(MotionEvent event);
*②在该组件上按下某个按钮时: boolean* **onKeyDown***(int keyCode,KeyEvent event);
*③松开组件上的某个按钮时: boolean* **onKeyUp***(int keyCode,KeyEvent event);
*④长按组件某个按钮时: boolean* **onKeyLongPress***(int keyCode,KeyEvent event);
*⑤键盘快捷键事件发生: boolean* **onKeyShortcut***(int keyCode,KeyEvent event); 

