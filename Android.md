#### Activity

Activity：最基础Activity

FragmentActivity：间接继承自Activity,支持support Fragment嵌套

AppCompatActivity:继承自FragmentActivity，一般程序的Activity继承该类重写onCreate()即可

### （一）Activity 布局

#### 组件常用属性

> * match_parent:组件填充以扩展父容器
> * wrap_content:组件内容扩展到整个组件
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

#### 6.标签TabLayout

常与ViewPager联动作为页面滑动指示器

TabLayout下添加TabItem作为标签子选项

```java
setupWithViewPager（ViewPager）//TabLayout通过该方法与ViewPager绑定
```

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

通常可以在xml中设置对应的监听器属性完成事件处理，如设置属性onClick绑定某个函数，或设置某个监听器重写监听器内方法

Fragment通常用于在Activity中实现界面切换

#### 2.事件回调

##### 方法回调

一种将功能定义与功能实现相分开的解耦合思想，对外提供实现接口(方法)，客户可以根据自己的需要对具体问题实现自己的需要，即接口统一，实现不同。使用回调即重写方法过程，某些运行过程检测到某种变化后自动回调执行重写方法

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

### （四）Fragment

可以把Fragment看成一个子Activity，即Activity片段！使用Fragment 可以把屏幕划分成几块，然后进行分组，进行一个模块化的管理！从而可以更加方便的在 运行过程中动态地更新Activity的用户界面！Fragment并不能单独使用，他需要嵌套在Activity 中使用，Fragment 在应用中用作可重复使用的容器，可以在各种 Activity 和布局配置中呈现相同的界面布局。

![img](https://www.runoob.com/wp-content/uploads/2015/08/31722863.jpg)

#### 创建Fragment

如同创建Activity一样，使用Fragment必须继承Fragment或其它已有的Fragment子类，重写onCreateView()方法绘制Fragment

```java
public static class ExampleFragment extends Fragment {
    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container,
                             Bundle savedInstanceState) {
        // 将Fragment加载至Activity
        return inflater.inflate(R.layout.example_fragment, container, false);
    }
}
```

其中LayoutInflater对象用于布局加载，container为父级Activity

使用inflater的inflate方法将Fragment加载至Activity，第一个参数为组件布局文件，第二个为父级容器，第三个指示是否在扩展期间扩展布局至父级容器，一般为false

#### 在Activity中添加Fragment

每个Fragment都需要位于某个Activity中

1. 在Activity的xml布局文件中声明属性fragment并指定对应的Fragment类即可，每个片段都需要id以供Activity重启时恢复片段

2. 在代码中动态添加,获取FragmentManager后获取FragmentTransaction添加调用Fragment

   ```java
   FragmentManager fragmentManager = getSupportFragmentManager();
   FragmentTransaction fragmentTransaction = fragmentManager.beginTransaction();
   ```

   ```java
   ExampleFragment fragment = new ExampleFragment();
   fragmentTransaction.add(R.id.fragment_container, fragment);
   fragmentTransaction.commit();
   ```

   add()方法第一个参数为布局ViewGroup，第二个为要添加的片段，需要使用commit()提交生效

   即初始时Activity启动onCreate()加载xml，属性fragment中配置Fragment，加载Fragment启动onCreateView()，Fragment也有对应的xml，加载xml
   
   动态添加使用replace与add，第一个参数为Activity的某个Fragment的容器布局的id，第二个为某个Fragment对象，切换时使用hide/show方法

#### Fragment容器

通常在Activity中增加一个帧布局作为Fragment的容器

```

    <FrameLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:layout_below="@id/ly_top_bar"
        android:layout_above="@id/div_tab_bar"
        android:id="@+id/ly_content">
    </FrameLayout>
```

#### Fragment与Activity交互

既然Fragment可以看做子Activity被嵌入到任意Activity当中，那么交互就是一项必然的点

1. Fragment获得Activity中的组件: **getActivity().findViewById(R.id.list)；**
2. Activity获得Fragment中的组件(根据id和tag都可以)：**getFragmentManager.findFragmentByid(R.id.fragment1);**

3. Activit传递数据给Fragment：在Activity中创建Bundle数据包,调用Fragment实例的setArguments(bundle) 从而将Bundle数据包传给Fragment,然后Fragment中调用getArguments获得 Bundle对象,然后进行解析就可以了

4. **Fragment传递数据给Activity**

   使用回调机制，Fragment内部定义接口，Fragment实现一部分，调用接口方法，Activity传递接口，重写接口方法

   **Step 1:定义一个回调接口:(Fragment中)**

   ```java
   /*接口*/  
   public interface CallBack{  
       /*定义一个获取信息的方法*/  
       public void getResult(String result);  
   }  
   ```

   **Step 2：接口回调（Fragment中）**

   ```java
   /*接口回调*/  
   public void getData(CallBack callBack){  
       /*获取文本框的信息,当然也可以传其他类型的参数*/  
       String msg = editText.getText().toString();  
       callBack.getResult(msg);//调用接口方法  
   }  
   ```

   **Step 3:使用接口回调方法读数据(Activity中)**

   ```java
   /*耦合处 使用接口回调的方法获取数据 */  
   leftFragment.getData(new CallBack() {  
    @Override  
          public void getResult(String result) {              /*打印信息*/  
               Toast.makeText(MainActivity.this, "-->>" + result, 1).show();  
               }
       	}); 
   ```

Activity调用Fragment的接口回调，传递事件处理方法，参数即为Fragment要传达的信息，即不直接从Fragment中拿/处理数据，而是通过接口拿/处理

#### 实例：底部导航栏

对于Fragment来说，有多中模板可以快速构建常见的功能样式



### （五）Android线程间通信

 Android不允许在UI线程外操作UI ,更新UI只能在UI线程即主线程中执行，否则并发更新UI会导致线程安全问题，因此只能顺序更新UI（使用异步机制提高效率），为了使非UI线程可以更新UI，Android提供几种方法

```java
Activity.runOnUiThread(Runnnable)
View.post(Runnable)
View.postDelayed(Runnable,long)
```

UI会执行传入的线程，调用线程提供的方法更新UI

还有两种方式实现线程间通信，Handler与AsyncTask类

#### Handler

##### handler结构：hadler+looper+message->messageQueue->looper取信息并回调执行方法

![img](https://www.runoob.com/wp-content/uploads/2015/07/25345060.jpg) 

- **UI线程**:就是我们的主线程,系统在创建UI线程的时候会初始化一个Looper对象,同时也会创建一个与其关联的MessageQueue;
- **Handler**:作用就是发送与处理信息,如果希望Handler正常工作,在当前线程中要有一个Looper对象
- **Message**:Handler接收与处理的消息对象
- **MessageQueue**:消息队列,先进先出管理Message,在初始化Looper对象时会创建一个与之关联的MessageQueue;
- **Looper**:每个线程只能够有一个Looper,管理MessageQueue,不断地从中取出Message分发给对应的Handler处理

**当我们的子线程想修改Activity中的UI组件时,我们可以新建一个Handler对象,通过这个对象向主线程发送信息;而我们发送的信息会先到主线程的MessageQueue进行等待,由Looper按先入先出顺序取出,再根据message对象的what属性分发给对应的Handler进行处理**

MessageQueue位于UI线程，每个要更新UI的线程绑定Looper与Handler，发送Message到消息队列，UI线程使用Looper不断从消息队列中取消息并根据消息指定绑定的线程的Handler自动执行回调方法handlerMessage方法更新UI

更通用的，使用过该机制完成通用线程间的通信

1. **线程B直接调用Looper.prepare()方法即可为当前线程创建Looper对象,而它的构造器会创建配套的MessageQueue，调用Looper.loop()方法启动Looper，Looper维护消息队列，不断循环并处理消息。**

2. **线程A使用Looper创建Handler，重写Handler的handleMessage处理消息，Handler发送Message到消息队列，Looper取出消息，根据what调用对应的Handler的handlerMessage异步执行**

Handler中的常用方法

- **handleMessage**(Message msg):处理消息的方法,通常是用于被重写!
- **sendEmptyMessage**(int what):发送空消息
- **sendEmptyMessageDelayed**(int what,long delayMillis):指定延时多少毫秒后发送空信息
- **sendMessage**(Message msg):立即发送信息
- **sendMessageDelayed**(Message msg):指定延时多少毫秒后发送信息
- final boolean **hasMessage**(int what):检查消息队列中是否包含what属性为指定值的消息 如果是参数为(int what,Object object):除了判断what属性,还需要判断Object属性是否为指定对象的消息

##### Handler使用

 **1 )直接调用Looper.prepare()方法即可为当前线程创建Looper对象,而它的构造器会创建配套的MessageQueue;**

**2 )创建Handler对象,重写handleMessage( )方法就可以处理来自于其他线程的信息了!**

**3 )调用Looper.loop()方法启动Looper**

#### AsyncTask

轻量级异步线程，不构造Handler进行异步操作

![img](https://www.runoob.com/wp-content/uploads/2015/07/27686655.jpg) 

AsyncTask是一个抽象类，定义了三种泛型，通常要写它的子类完成想要的任务，对象调用execute()开始该异步线程

 ![img](https://www.runoob.com/wp-content/uploads/2015/07/39584771.jpg) 

##### 使用

 ```java
public class MyAsyncTask extends AsyncTask<Integer,Integer,String>  
{  
    private TextView txt;  
    private ProgressBar pgbar;  
  
    //初始化传递相关组件
    public MyAsyncTask(TextView txt,ProgressBar pgbar)  
    {  
        super();  
        this.txt = txt;  
        this.pgbar = pgbar;  
    }  
    //1.初始化UI：该方法运行在UI线程中,UI初始化更新 
    @Override  
    protected void onPreExecute() {  
        txt.setText("开始执行异步线程~");  
    }  
    
    //2.异步执行：该方法不运行在UI线程中,主要用于异步操作,通过调用publishProgress()方法
    //触发onProgressUpdate对UI进行操作  
    @Override  
    protected String doInBackground(Integer... params) {   
        for (int i = 10;i <= 100;i+=10)  
        {  
            publishProgress(i);  
        }  
        return "100xxx";  
    }  
  
    //3.异步更新返回显示：在doInBackground方法中,每次调用publishProgress方法都会触发该方法  
    //运行在UI线程中,UI调用该方法
    @Override  
    protected void onProgressUpdate(Integer... values) {  
        pgbar.setProgress(value);  
    }  
    
    //4.异步方法执行后调用：UI在doInBackground执行完毕后执行该方法
    @Override
    protected void onPostExecute(String result){
        System.out.println("异步方法执行完毕");
    }
}

public class MyActivity extends ActionBarActivity {  
    private TextView txttitle;  
    private ProgressBar pgbar;  
    private Button btnupdate;  
 
    @Override  
    protected void onCreate(Bundle savedInstanceState) {  
        super.onCreate(savedInstanceState);  
        setContentView(R.layout.activity_main);  
        txttitle = (TextView)findViewById(R.id.txttitle);  
        pgbar = (ProgressBar)findViewById(R.id.pgbar);  
        btnupdate = (Button)findViewById(R.id.btnupdate);  
        btnupdate.setOnClickListener(new View.OnClickListener() {  
            @Override  
            public void onClick(View v) {
                //4.使用异步机制
                MyAsyncTask myTask = new MyAsyncTask(txttitle,pgbar);  
                myTask.execute(1000);  
            }  
        });  
    }  
} 
 ```

### 组件

#### 根布局

根布局作为View的容器，既可以是传统layout也可以是View，取决于

#### 菜单menu

每个Activity默认都包含一个Menu对象，因此编写菜单只需要编写菜单项与对应的事件监听即可，有多中不同样式的菜单，加载菜单时重写相应方法即可，菜单menu需要一个xml布局并在ActionBar/ToolBar上挂载

**1.创建menu菜单与menu xml布局文件**

选中res文件夹---->右键---->New----->Android resouce directory----->Resouce Type选下拉列表中的menu，点击ok，就在res文件夹下新建了menu文件夹

**2.在xml中添加菜单子项MenuItem**

例如

```xml
<?xml version="1.0" encoding="utf-8"?>
<menu xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:android="http://schemas.android.com/apk/res/android">

    <item
        android:id="@+id/home_menu_search"
        android:title="搜索"
          <!--指定一个View作为子项-->
        app:actionViewClass="androidx.appcompat.widget.SearchView" />
</menu>
```



#### RadioGroup+RadioButton实现底部导航

布局之后的事件监听：

```java
 radioGroup.setOnCheckedChangeListener(new RadioGroup.OnCheckedChangeListener() {
            @Override
            public void onCheckedChanged(RadioGroup radioGroup, int i) {
               //参数i为响应事件的RadioButtopn的id
            }
        });
```

设置RadioButton文字在下，图片在上

```xml
        <RadioButton
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:layout_weight="1"
            android:background="@android:color/transparent"//防止文字竖直拉长
            android:button="@null"//消除默认圆圈
            android:drawableTop="@mipmap/me"
            android:gravity="center"//设置图片居中
            android:text="@string/title_me"
            android:textColor="@color/colorPrimary"/>
```

#### 导航库Navigation：多Fragment跳转

**https://developer.android.com/reference/androidx/navigation/Navigation.html**

专为单Activity多Fragment设计，每个Activity都与一个navigation graph绑定并包含一个NavHostFragment

添加依赖：

```xml
implementation "androidx.navigation:navigation-fragment:$nav_version"
implementation "androidx.navigation:navigation-ui:$nav_version"
```

**1.导航放在专门的导航文件夹下,因此要新建文件夹navigation**

1. In the Project window, right-click on the `res` directory and select **New > Android Resource File**. The **New Resource File** dialog appears.
2. Type a name in the **File name** field, such as "nav_graph".
3. Select **Navigation** from the **Resource type** drop-down list, and then click **OK**.

**2.在主Activity下指定导航xml文件，指定NavHost**

NavHost：NavHost是一个空容器，相当于Fragment切换时当前Fragment显示的容器，在主Activity的xml布局下要指定一个导航xml文件并指定一个NavHost

```xml
    <fragment
        android:id="@+id/nav_host_fragment"
        android:name="androidx.navigation.fragment.NavHostFragment"<!--引入组件，必不可少-->
        android:layout_width="0dp"
        android:layout_height="0dp"
        app:layout_constraintLeft_toLeftOf="parent"
        app:layout_constraintRight_toRightOf="parent"
        app:layout_constraintTop_toTopOf="parent"
        app:layout_constraintBottom_toBottomOf="parent"

        app:defaultNavHost="true"<!--表示点击返回键返回上一个页面-->
        app:navGraph="@navigation/nav_graph" /><!--指定导航图xml文件-->
```

**3.在导航图xml文件配置要跳转的Fragment**

需设置一个开始Fragment，其余的点击拖拉即可，Fragment必须先设置好才行

**4.设置Fragment跳转**

代码中的Fragment跳转由NavController控制，每一个NavHost都有一个NavController用于管理Fragment的跳转，获取NavController有三种方法，适用于不同的场景

```java
NavHostFragment.findNavController(Fragment)
Navigation.findNavController(Activity, @IdRes int viewId)
Navigation.findNavController(View)
```

获取之后返回一个NavController对象，调用navigate(int id)即可实现Fragment的跳转



#### badge底部导航栏的消息提醒

#### 页面滑动切换ViewPager+FragmentPagerAdapter/FragmentStatePagerAdapter+Tablayout

ViewPager通过适配器PagerAdapter绑定多个View，可在View间左右滑动切换

要使用普通的ViewPager,必须要为Activity中findViewById找到的viewPager组件设置一个Adapter。可以定义一个类继承自PagerAdapter，这个类需要实现四个方法，分别为：

* instantiateItem(ViewGroup container, int position):添加到容器，初始化指定position为初始显示View

* destroyItem(ViewGroup container, int position,Object object):从容器移除指定position位置的View

* getCount()：返回视图个数

* isViewFromObject(View view,Object object)：是否匹配

```java
package com.example.allbooks.ui.home;

import android.view.View;
import android.view.ViewGroup;

import androidx.viewpager.widget.PagerAdapter;
import androidx.viewpager.widget.ViewPager;

import java.util.ArrayList;
import java.util.List;

public class HomeViewPagerAdapter extends PagerAdapter {
    private List<View> viewList;//视图列表
    public HomeViewPagerAdapter(){
        viewList=new ArrayList<>();
    }
    //添加初始化一个View并返回,position为列表View中的某个
    @Override
    public Object instantiateItem(ViewGroup container, int position){
       if(position<0||position>=viewList.size()){
           System.out.println("home/HomeViewPagerAdapter/add:超出ViewPager列表适配器的范围");
       }
       container.addView(viewList.get(position));
        return viewList.get(position);
    }
    //使从ViewGroup中移出当前View
    @Override
    public void destroyItem(ViewGroup container, int position, Object object) {
        if(position<0||position>=viewList.size()){
            System.out.println("home/HomeViewPagerAdapter/remove:超出ViewPager列表适配器的范围");
        }
        container.removeView(viewList.remove(position));
    }

    @Override
    //获取当前View数
    public int getCount() {
        if(viewList==null)viewList=new ArrayList<>();
        return viewList.size();
    }

    @Override
    //判断是否由对象生成界面
    public boolean isViewFromObject(View view, Object object) {
        return view==object;
    }
    
}

```

 Fragment与ViewPager的组合使用，Google提供了FragmentPagerAdapter与FragmentStatePagerAdapter两个类，其中第一个适用于fragment较少的，另一个适用较多的 。推荐使用FragmentPagerAdapter

对于Fragment内的页面滑动，要使用ViewPager与FragmentPagerAdapter

FragmentPagerAdapter也是抽象类，需要重写方法

* getItem(int)

* getCount()

同时绑定维护的是List<Fragment\>和Fragment对应的名称List<String\>而不再是List<View\>

```java
/**
 * 使用多Fragment+ViewPager实现滑动操作
 * 需要适配器FragmentPagerAdapter保存并提供Fragment信息以初始化或恢复
 */
public class HomeViewPagerAdapter extends FragmentPagerAdapter {
    private List<Fragment> frags;
    private List<String> titles;

    public HomeViewPagerAdapter(FragmentManager fm, List<Fragment> frags, List<String> titles) {
        //调用父类构造器，第二个参数即只显示当前Fragment,其余Fragment暂停而不是隐藏，隐藏选项已过时
        super(fm,FragmentPagerAdapter.BEHAVIOR_RESUME_ONLY_CURRENT_FRAGMENT);
        //构造方法初始化
        this.frags=frags;
        this.titles=titles;
    }

    @Override
    public Fragment getItem(int arg0) {
        //返回具体位置的Fragment
        return frags.get(arg0);
    }

    @Override
    public int getCount() {
        //Fragment列表大小
        return frags.size();
    }

    @Override
    public CharSequence getPageTitle(int position) {
        // 返回某个Fragment的标题
        return titles.get(position);
    }

}
```

更具体的步骤为

**1.在某个Fragment中使用ViewPager(布局文件中添加)**

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <androidx.viewpager.widget.ViewPager
        android:id="@+id/homeViewPager"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        app:layout_constraintTop_toTopOf="parent">

        <androidx.viewpager.widget.PagerTabStrip
            android:id="@+id/pagerTabStrip"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_gravity="top" />
    </androidx.viewpager.widget.ViewPager>

</androidx.constraintlayout.widget.ConstraintLayout>
```

**2.配置ViewPager对应的FragmentPagerAdapter**

代码如上

**3.配置FragmentPagerAdapter要使用的Fragment与其对应的xml文件**

**4.在主Fragment中初始化Fragment列表，传入适配器，让主Fragment设置适配器，主Fragment的ViewPager通过适配器管理Fragment列表**

```java
    /**
     * 创建顶部的滑动切换组件ViewPager
     */
    private ViewPager viewPager;//ViewPager，页面切换组件
    private List<Fragment> fragmentList;//要切换的Fragment队列
    private List<String> fragmentTitleList;//Fragment名称队列
    private PagerTabStrip pagerTabStrip;//滑动带


    private void scrollHorizonCreate(){
        viewPager = root.findViewById(R.id.homeViewPager);//获取ViewPager
        pagerTabStrip=root.findViewById(R.id.pagerTabStrip);

        fragmentTitleList=new ArrayList<>();
        fragmentTitleList.add("关注");
        fragmentTitleList.add("榜单");
        fragmentList=new ArrayList<>();
        fragmentList.add(new FocusFragment());
        fragmentList.add(new RankFragment());

        //绑定适配器
        viewPager.setAdapter(new HomeViewPagerAdapter(getChildFragmentManager(),fragmentList,fragmentTitleList));
        //设置viewPager的初始界面为第一个界面
        viewPager.setCurrentItem(0);
        //添加切换界面的监听器
        //viewPager.addOnPageChangeListener(new MyOnPageChangeListener());
    }

```

注意因为是Fragment嵌套Fragment，所以使用的是getChildFragmentManager()

 ![è¿éåå¾çæè¿°](https://img-blog.csdn.net/20170411111926684?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvTEFCTEVORVQ=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast) 

如果是Activtiy嵌套Fragment，则使用 getFragmentManager() 

 ![è¿éåå¾çæè¿°](https://img-blog.csdn.net/20170411105105796?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvTEFCTEVORVQ=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast) 

#### 顶部标题ToolBar

**参考: https://www.jianshu.com/p/e2ae6aaff696 **

ToolBar继承的是ViewGroup,也就是说Toolbar 是一个ViewGroup 容器,内部可以添加View

常位于顶部， 一个Toolbar 从左到右包括了 一个navigation button、一个logo、一个title和subtitle、一个或多个自定义的View和一个 action menu 这5部分 

 ![img](https://upload-images.jianshu.io/upload_images/3513995-d69b1fc00cd05569.png?imageMogr2/auto-orient/strip|imageView2/2/w/818/format/webp) 

自定义View只能在Text中手动添加或通过代码添加

action menu在res下的menu目录中写对应的menu文件，加载即可

#### 搜索框SerachView

SearchView属性

模式有三种：

* iconified(false);搜索框默认是开启的，左侧搜索图标在搜索框中，可以通过右侧叉叉关闭搜索框
* iconifiedByDefault(false); 右侧一开始没有叉叉，有输入内容后出现叉叉（常用）
* queryHint：提示文字

事件监听：

```java
    //搜索图标按钮的点击事件
    mSearchView.setOnSearchClickListener(new View.OnClickListener() {
        @Override
        public void onClick(View v) {
            Toast.makeText(MainActivity.this, "打开搜索框", Toast.LENGTH_SHORT).show();
        }
    });

    //搜索框内容变化监听
    mSearchView.setOnQueryTextListener(new SearchView.OnQueryTextListener() {
        @Override
        public boolean onQueryTextSubmit(String query) {//点击提交按钮时
            Toast.makeText(MainActivity.this, "Submit---提交", Toast.LENGTH_SHORT).show();
            return true;
        }

        @Override
        public boolean onQueryTextChange(String newText) {//搜索框内容变化时
            if (!TextUtils.isEmpty(newText)) {
//              mListView.setFilterText(newText);
                mAdapter.getFilter().filter(newText);
            } else {
                mListView.clearTextFilter();
            }
            return true;
        }
    });

    //搜索框展开时点击叉叉按钮关闭搜索框的点击事件
    mSearchView.setOnCloseListener(new SearchView.OnCloseListener() {
        @Override
        public boolean onClose() {
            Toast.makeText(MainActivity.this, "关闭搜索框", Toast.LENGTH_SHORT).show();
            return false;
        }
    });	
```

#### 列表ListView

**参考: https://hellosure.github.io/android/2015/06/02/android-viewholder **

ListView需要与适配器配合使用，列表项是适配器绑定的数据，用于大量数据的（通常是滑动）显示，现可使用RecyclerView替代

列表项Item点击会触发ItemClick事件，由方法onItemClick()监听

列表项Item长按会触发ItemLongClick事件，由方法onItemLongClick监听

ListView会调用适配器中的方法进行数据显示，适配器中的几个最基本方法有

```java
class MyAdapter extends BaseAdapter
    {
        private Context context;
        public MyAdapter(Context context)
        {
            this.context = context;
        }
        @Override
        public int getCount() {
            // How many items are in the data set represented by this Adapter.(在此适配器中所代表的数据集中的条目数)
            return 0;
        }
        @Override
        public Object getItem(int position) {
            // Get the data item associated with the specified position in the data set.(获取数据集中与指定索引对应的数据项)
            return null;
        }
        @Override
        public long getItemId(int position) {
            // Get the row id associated with the specified position in the list.(取在列表中与指定索引对应的行id)
            return 0;
        }
        @Override
        public View getView(int position, View convertView, ViewGroup parent) {
            // Get a View that displays the data at the specified position in the data set.
            return null;
        } 
    }
```

 ListView加载数据都是在适配器的public View getView(int position, View convertView, ViewGroup parent) {}方法中进行的,convertView是缓存View，避免每次重新从布局文件中加载xml，使用findViewById获取View并加载

 getView有几种使用方式：

第一种：没有任何处理，不建议这样写。如果数据量少看将就，但是如果列表项数据量很大的时候，会每次都重新创建View，设置资源，严重影响性能，所以从一开始就不要用这种方式 

```java
@Override
        public View getView(int position, View convertView, ViewGroup parent) {
            View item = mInflater.inflate(R.layout.list_item, null);
            ImageView img = (ImageView)item.findViewById(R.id.img)
            TextView title = (TextView)item.findViewById(R.id.title);
            TextView info = (TextView)item.findViewById(R.id.info);
            img.setImageResource(R.drawable.ic_launcher);
            title.setText("Hello");
            info.setText("world");
          
            return item;
        }
```

 第二种ListView优化：通过缓存convertView,这种利用缓存contentView的方式可以判断如果缓存中不存在View才创建View，如果已经存在可以利用缓存中的View，提升了性能 

```java
public View getView(int position, View convertView, ViewGroup parent) {
            if(convertView == null)
            {
                convertView = mInflater.inflate(R.layout.list_item, null);
            }
                                                                                         
            ImageView img = (ImageView)convertView.findViewById(R.id.img)
            TextView title = (TextView)convertView.findViewById(R.id.title);
            TextView info = (TextView)ConvertView.findViewById(R.id.info);
            img.setImageResource(R.drawable.ic_launcher);
            title.setText("Hello");
            info.setText("world");
                                                                                             
            return convertView;
        }
        
```

第三种ListView优化：通过convertView+ViewHolder来实现，ViewHolder就是一个静态类，使用 ViewHolder 的关键好处是缓存了显示数据的视图（View），加快了 UI 的响应速度。

当我们判断 convertView == null 的时候，如果为空，就会根据设计好的List的Item布局（XML），来为convertView赋值，并生成一个viewHolder来绑定converView里面的各个View控件（XML布局里面的那些控件）。再用convertView的setTag将viewHolder设置到Tag中，以便系统第二次绘制ListView时从Tag中取出。（看下面代码中）

如果convertView不为空的时候，就会直接用convertView的getTag()，来获得一个ViewHolder。

```java
//在外面先定义，ViewHolder静态类
static class ViewHolder
{
    public ImageView img;
    public TextView title;
    public TextView info;
}
//然后重写getView
@Override
public View getView(int position, View convertView, ViewGroup parent) {
    ViewHolder holder;
    //第一次加载View
    if(convertView == null)
    {
        holder = new ViewHolder();
        convertView = mInflater.inflate(R.layout.list_item, null);
        holder.img = (ImageView)item.findViewById(R.id.img)
        holder.title = (TextView)item.findViewById(R.id.title);
        holder.info = (TextView)item.findViewById(R.id.info);
        convertView.setTag(holder);//绑定一个View对象的引用
    }else
    {
        holder = (ViewHolder)convertView.getTag();
    }
        holder.img.setImageResource(R.drawable.ic_launcher);
        holder.title.setText("Hello");
        holder.info.setText("World");
    }
                                                                                         
    return convertView;
}
```

#### RecyclerView+RecyclerView.ViewHolder+RecyclerView.Adapter+CardView实现滑动列表

**参考： https://www.jianshu.com/p/4fc6164e4709 **

**https://developer.android.com/guide/topics/ui/layout/recyclerview **

**https://blog.csdn.net/xx326664162/article/details/61199895 **

**https://zhuanlan.zhihu.com/p/24807254 **

可使用ListView添加item实现列表滑动，数据源来源于Adapter， ViewHolder 通过内部维护View对象减少 findViewById() 的使用以及避免过多地 inflate view ，更改数据时直接更改以前的View数据即可

 RecyclerView用于在有限的窗口展现大量的数据 ，与ListView功能类似，但RecyclerView标准化了ViewHolder，而且异常的灵活，可以轻松实现ListView实现不了的样式和功能 

 在使用RecyclerView时候，必须指定一个适配器Adapter和一个布局管理器LayoutManager。适配器继承`RecyclerView.Adapter`类，具体实现类似ListView的适配器，取决于数据信息以及展示的UI，适配器内部使用`RecyclerView.ViewHolder`绑定View引用，加快速度。

```java
mRecyclerView = (RecyclerView) findViewById(R.id.my_recycler_view);
// 设置布局管理器
mRecyclerView.setLayoutManager(mLayoutManager);
// 设置adapter
mRecyclerView.setAdapter(mAdapter);
// 设置Item添加和移除的动画
mRecyclerView.setItemAnimator(new DefaultItemAnimator());
// 设置Item之间间隔样式
mRecyclerView.addItemDecoration(mDividerItemDecoration);
```

RecyclerView提供了三种布局管理器：

- LinerLayoutManager 以垂直或者水平列表方式展示Item
- GridLayoutManager 以网格方式展示Item
- StaggeredGridLayoutManager 以瀑布流方式展示Item

具体步骤为：

1. 在Fragment中添加RecyclerView

2. 写配套的Adapter并绑定ViewHolder，必须重写三个必要方法以供布局管理器回调

   初始化时对于先执行onCreateViewHolder，用于创建新的View并绑定ViewHolder

   然后执行onBindViewHolder，参数有一个holder，用于对holder绑定的View做数据填充（更改）

   ```java
   public class FocusRecyclerViewAdapter extends  RecyclerView.Adapter<FocusRecyclerViewAdapter.FocusRecyclerViewHolder> {
       private String[] contentCollection;//要变化的数据集
   
       //加快UI速度，内部维护ViewHolder，适配器维护CardView并进行加载
       public static class FocusRecyclerViewHolder extends RecyclerView.ViewHolder{
           public CardView cardView;
           public FocusRecyclerViewHolder(CardView cardView){
               super(cardView);//使用RecycleView内部的ViewHolder绑定一个View
               this.cardView=cardView;
           }
       }
       //构造器，提供数据集
       public FocusRecyclerViewAdapter(String[] dataSet){
           contentCollection=dataSet;
       }
   
       //创建一个新的View,即适配器数据子项，由layout manager进行回调，加快UI速度，内部绑定ViewHolder
       @Override
       public FocusRecyclerViewHolder onCreateViewHolder(ViewGroup parent,int viewType){
           //向父容器FocusFragment添加CardView
           CardView cardView= (CardView) LayoutInflater.from(parent.getContext()).inflate(R.layout.focus_recyclerview_cardview,parent,false);
           //绑定新建的CardView到ViewHolder
           FocusRecyclerViewHolder focusRecyclerViewHolder=new FocusRecyclerViewHolder(cardView);
           return focusRecyclerViewHolder;
       }
   
       //使用position位置处的内容替换指定view的内容，由layout manager进行回调，数据变换函数
       @Override
       public void onBindViewHolder(FocusRecyclerViewHolder holder,int position){
           TextView textView=holder.cardView.findViewById(R.id.cardViewTextView);
           textView.setText(contentCollection[position]);
       }
   
       //返回数据集长度，由layout manager回调
       @Override
       public int getItemCount() {
           return contentCollection == null ? 0 : contentCollection.length;
       }
   }
   ```

3. 在有RecyclerView的Fragment中加载RecyclerView并绑定布局管理器，适配器

```java
public class FocusFragment extends Fragment {
    private View view;
    private RecyclerView recyclerView;
    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
        view=inflater.inflate(R.layout.fragment_home_focus, container,false);//加载布局进入父容器HomeFragment

        recyclerView=view.findViewById(R.id.focusRecyclerView);//组件RecyclerView
        recyclerView.setLayoutManager(new StaggeredGridLayoutManager(3, OrientationHelper.VERTICAL));//设置布局管理器
        recyclerView.setAdapter(new FocusRecyclerViewAdapter(new String[]{"sad","qwe","pop","dsa","dsa","das","asd"}));//设置数据适配器，提供数据集

        return view;
    }
}
```

4. 在2步骤中适配器的 onCreateViewHolder方法中创建新的View并加载xml，因此一般使用CardView布局作为RecyclerView的子项进行加载，具体内容格式要在CardView中配置，RecyclerView只负责加载View，滑动时动态更改View。CardView可单独抽出作为xml被加载，可设置height与width

**细节**

RecyclerView对ViewHolder也进行了一定的封装，但是如果你仔细观察，你会发出一个疑问，ListView里面有个getView，它返回View为Item的布局，那么RecyclerView这个Item的样子在哪控制？

其实是这样的，我们创建的ViewHolder必须继承RecyclerView.ViewHolder，这个RecyclerView.ViewHolder构造时必须传入一个View，这个View相当于我们ListView getView中的convertView （即：inflate的item布局需要传入）。

还有一点，ListView中convertView是复用的，在RecyclerView中，是把ViewHolder类作为缓存的单位了，然后convertView作为ViewHolder的成员变量保持在ViewHolder中，也就是说，假设屏幕显示10个条目，则会创建10个ViewHolder缓存起来，每次复用的是ViewHolder，所以他把getView这个方法变为了onCreateViewHolder。

**分割线**

如果使用的是线性布局，则可以使用如下方法设置间隔，间隔样式来源于默认间隔样式

 mRecyclerView.addItemDecoration(new DividerItemDecoration(this,DividerItemDecoration.VERTICAL));  

##### 事件监听

每个View都有onClickListener，实现一个自己的事件监听，传递给Adapter，在绑定时传给ViewHolder，让ViewHolder实现点击监听的逻辑

#### SwipeRefreshLayout下拉刷新

SwipeRefreshLayout是一个下拉刷新布局，只能有一个孩子，常与RecyclerView配合使用

将RecyclerView放置于SwipeRefreshLayout下即可实现下拉刷新

 在SwipeRefleshLayout中的SwipeRefreshLayout.OnRefreshListener实现OnRefresh()方法即可在onReflesh()方法中，进行刷新数据操作，数据改变完毕后，使用适配器的notifyDataChanged()方法通知适配器数据集改变，调用 swipeRefreshView.setRefreshing(false); 终止刷新，

具体步骤为：

1. 在Activity或Fragment中加入SwipeRefreshLayout，添加能滑动的孩子，如ListView/RecyclerView

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.swiperefreshlayout.widget.SwipeRefreshLayout
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:id="@+id/swipeFocusRefreshLayout"
    xmlns:android="http://schemas.android.com/apk/res/android">

    <androidx.recyclerview.widget.RecyclerView
        android:id="@+id/focusRecyclerView"
        android:layout_width="match_parent"
        android:layout_height="match_parent">
    </androidx.recyclerview.widget.RecyclerView>
</androidx.swiperefreshlayout.widget.SwipeRefreshLayout>
```

2. 在Activity或Fragment中加载SwipeRefreshLayout，设置监听器，在onRefresh中设置数据处理方法

```java
        focusRecyclerViewAdapter=new FocusRecyclerViewAdapter(dataSet);//初始化适配器，绑定数据集
        //下滑刷新布局组件
        final SwipeRefreshLayout swipeRefreshLayout=(SwipeRefreshLayout)view.findViewById(R.id.swipeFocusRefreshLayout);
        //下滑刷新监听器
        swipeRefreshLayout.setOnRefreshListener(new SwipeRefreshLayout.OnRefreshListener() {
            @Override
            public void onRefresh() {
                dataSet.add("d"+test);
                focusRecyclerViewAdapter.notifyDataSetChanged();//适配器通知数据集改变，回调onBindViewHolder，根据状态决定是否加载onCreteViewHolder

                swipeRefreshLayout.setRefreshing(false);//数据处理完毕，终止刷新状态
            }
        });
```

#### RecyclerView的上拉加载/加载多种子项视图

上拉加载基本思想就是原本的数据（如CardView）View队列最末尾加上一个新的View(如ProgressBar)，用于上拉加载时的显示，在Fragment中设置滑动监听器，当滑动到最末尾时更新数据并设置上面的ProgressBar隐藏，通知UI线程更新

1.RecyclerView的适配器内部维护自定义状态量，根据状态加载不同布局并绑定ViewHolder，ViewHolder内部维护不同的View组件，在OnCreateViewHolder中动态绑定View并进行组件加载，因此RecyclerView可以适配多种不同的View，只要管好适配器就行。

```java
public class FocusRecyclerViewAdapter extends  RecyclerView.Adapter<RecyclerView.ViewHolder> {
    private List<JSONObject> contentCollection;//数据集
    private int count=0;//本地测试数据循环使用下标

    private int normalType=0;//正常加载的cardView
    private int footType=1;//底部加载的FooterView，实际上就是一个ProgressBar
    private boolean loading=true;//判断是否正在加载，用于控制加载条ProgressBar的显示

    /**
     * 内部ViewHolder，加快View加载速度，绑定CardView，用于数据显示
     */
    public static class FocusRecyclerViewHolder extends RecyclerView.ViewHolder{
        public CardView cardView;
        public FocusRecyclerViewHolder(CardView cardView){
            super(cardView);//使用RecycleView内部的ViewHolder绑定一个View
            this.cardView=cardView;
        }
    }

    /**
     * 内部FooterViewHolder，绑定一个ProgressBar，更好的用户体验
     */
    public static class FooterViewHolder extends RecyclerView.ViewHolder{
        private ProgressBar loadMore;
        public FooterViewHolder(View view){
            super(view);
            this.loadMore=((LinearLayout)view).findViewById(R.id.load_more);
        }
    }

    /**
     * 构造器，提供数据集
     * @param dataSet 适配器需要的数据集
     */
    public FocusRecyclerViewAdapter(List<JSONObject> dataSet){
        contentCollection=dataSet;
    }

    /**
     * 由于RecyclerView不提供上拉加载功能，所以自定义重写返回子项Item的方法，要判断返回的子项是正常的CardView还是
     * 底部加载条progressBar，该方法在使用onCreateViewHolder时作为参数viewType回调
     * @param position 数据集项目位置
     * @return View类型，CardView由normalType代表，ProgressBar由footType代表
     */
    @Override
    public int getItemViewType(int position){
        //正好到了底部，需要加载更多,比如有15条数据,索引从0开始，第15条是ProgressBar,getItemCount()返回16
        System.out.println(position);
        if(position==getItemCount()-1){
            return footType;
        }else {
            return normalType;
        }
    }

    /**
     * 创建一个新的View,即适配器数据子项，由layout manager进行回调，加快UI速度，内部绑定ViewHolder
     * 根据viewType的不同加载不同的布局文件
     * @param parent 父容器，也就是RecyclerView
     * @param viewType 要加载的View类型，如果子类不指定，父类永远为0
     * @return ViewHolder，绑定一个View
     */
    @Override
    public RecyclerView.ViewHolder onCreateViewHolder(ViewGroup parent, int viewType){
        RecyclerView.ViewHolder holder;
        if(viewType==normalType){
            //加载布局文件，向父容器FocusFragment添加CardView
            CardView cardView= (CardView) LayoutInflater.from(parent.getContext()).inflate(R.layout.focus_recyclerview_cardview,parent,false);
            holder=new FocusRecyclerViewHolder(cardView);
        }else{
            View linearLayout =LayoutInflater.from(parent.getContext()).inflate(R.layout.progress_bar,parent,false);
            holder=new FooterViewHolder(linearLayout);
        }
        return holder;
    }

    /**
     * 使用position位置处的内容替换指定view的内容，由layout manager进行回调，用于给ViewHolder中的数据做加载(或变换数据)
     * 分类型绑定数据为CardView还是ProgressBar
     * @param holder holder，可能为CardView的holder，也可能为ProgressBar的holder，分别处理
     * @param position 子项的位置
     */
    @Override
    public void onBindViewHolder(RecyclerView.ViewHolder holder,int position){
        if(holder instanceof FocusRecyclerViewHolder){
            CardView cardView=((FocusRecyclerViewHolder)holder).cardView;//获取ViewHolder绑定的数据
            // 对CardView中的数据做实时绑定，使用findViewById获取内部组件
            JSONObject article=contentCollection.get(count);
            TextView title=cardView.findViewById(R.id.articleTitle);
            TextView introduce=cardView.findViewById(R.id.articleIntroduce);
            TextView author=cardView.findViewById(R.id.articleAuthor);;
            try{
                title.setText(article.getString("title"));
                introduce.setText(article.getString("content"));
                author.setText(article.getString("author")+" "+article.getString("visits")+" "+article.getString("thumbs"));
            }catch (Exception e){
                e.printStackTrace();
            }
            count=(count+1)%contentCollection.size();//取模循环
        }else{
            ProgressBar progressBar=((FooterViewHolder)holder).loadMore;
            if(!loading){
                progressBar.setVisibility(View.GONE);
            }
        }

        //设置item-CardView事件监听
    }

    /**
     * 返回数据集长度，由layout manager回调
     * @return 返回数据集长度，不为null加1是因为要加载底部footerView
     */
    @Override
    public int getItemCount() {
        return contentCollection == null ? 0 : contentCollection.size()+1;
    }

    /**
     * 由Recycler调用，当加载完数据时，调用该方法设置ProgressBar显示状态并通知数据集改变
     * @param state ProgressBar的状态，true时显示，false时不显示
     */
    public void setLoadState(boolean state){
        this.loading=state;
        notifyDataSetChanged();
        this.loading=true;//通知完数据集改变之后，将上一步progressBar隐藏，之后新的progressBar仍然显示
    }
}

```

2. 对RecyclerView设置事件监听

   ```java
           //为recyclerView设置滑动监听器，加载更多数据
           recyclerView.addOnScrollListener(new RecyclerOnScrollListener() {
               @Override
               public void onLoadMore() {//这里模拟网络数据传输
                   new Timer().schedule(new TimerTask() {
                       @Override
                       public void run() {
                           view.post(new Runnable() {
                               @Override
                               public void run() {
                                   initData();
                                   focusRecyclerViewAdapter.setLoadState(false);
                               }
                           });
                       }
                   },1000);
   
               }
           });
           
               /**
        * RecyclerView的滑动监视器，RecyclerView必须使用线性布局管理器，因为onScrollStateChanged(RecyclerView recyclerView, int newState)
        * 方法中要使用
        */
       abstract class RecyclerOnScrollListener extends RecyclerView.OnScrollListener {
           private boolean isSlidingUpward = false;//用来标记是否正在向上滑动
   
           /**
            * 当滑动状态改变时调用该方法，如停止滑动
            * @param recyclerView 调用该监视器的RecyclerView
            * @param newState 表示滑动状态
            */
           @Override
           public void onScrollStateChanged(RecyclerView recyclerView, int newState) {
               super.onScrollStateChanged(recyclerView, newState);
               LinearLayoutManager manager = (LinearLayoutManager) recyclerView.getLayoutManager();
               // 当不滑动时
               if (newState == RecyclerView.SCROLL_STATE_IDLE) {
                   //获取最后一个完全显示的itemPosition
                   int lastItemPosition = manager.findLastCompletelyVisibleItemPosition();
                   int itemCount = manager.getItemCount();
   
                   // 判断是否滑动到了最后一个item，并且是向上滑动
                   if (lastItemPosition == (itemCount - 1) && isSlidingUpward) {
                       //加载更多
                       onLoadMore();
                   }
               }
           }
   
           /**
            * 正在滑动的过程中执行该方法，内部维护的变量isSlidingUpward表示是否正在向上滑动
            * @param recyclerView 调用该监视器的RecyclerView
            * @param dx 向左滑或向右滑
            * @param dy 向上滑或向下滑
            */
           @Override
           public void onScrolled(RecyclerView recyclerView, int dx, int dy) {
               super.onScrolled(recyclerView, dx, dy);
               // 大于0表示正在向上滑动，小于等于0表示停止或向下滑动
               isSlidingUpward = dy > 0;
           }
   
           /**
            * 加载更多回调，由子类RecyclerView来实现，如向适配器中载入新的数据等
            */
           public abstract void onLoadMore();
       }
   ```

   加载多项子视图同理，适配器内部维护子项类型，根据getItemViewType(int position)方法可根据数据集中数据的不同返回不同的类型，在onCreateViewHolder中通过返回类型绑定不同的子项布局文件，也就是说一个holder必定绑定一种具体的子项类型并对应数据集中的特定数据子集，数据集中的数据必定是某种类型中的一种，在更新数据时onBindViewHolder有一个position参数，根据holder instanceof那种适配器判断属于那种类型，根据position直接获取数据集中的指定数据。

####  CardView

**参考： https://developer.android.com/guide/topics/ui/layout/cardview **.

通常配合RecyclerView实现滑动列表，将CardView视为根视图

组件监听为：cardView.setOnClickListener()

#### 滑动显示ScrollView

 `ScrollView`的直接子View只能有一个。 因此只能绑定一个子布局以实现复杂布局。

 `ScrollView`只支持竖直滑动，水平滑动使用`HorizontalScrollView`。

一般直接将ScrollView直接作为根视图 

### 技巧

#### Fragment双击返回(定义接口，事件回调)

Fragment并不提供返回事件的监听器设置，只有Activity实现了。

同时，有的Fragment需要双击退出，有的Fragment并不需要双击退出，因此通过指定监听器接口来实现，Activty一直监听键盘事件，当处于某个Fragment时，当前Fragment为Activity设置具体的监听器实现，由Activity监控实现即可

**1.定义返回监听接口**

```java
public interface FragmentBackListener {
    void onBackForward();
}
```

**2.在Activity中引入监听器，提供设置监听器方法，在监听键盘的方法中调用监听器(回调)方法**

```java
    private FragmentBackListener backListener;//Fragment返回监听拦截
    public void setBackListener(FragmentBackListener backListener){
        this.backListener=backListener;
    }
```

```java
    /**
     * 双击返回键返回，事件已被委托
     */
    @Override
    public boolean onKeyDown(int keyCode, KeyEvent event) {
        if (keyCode == KeyEvent.KEYCODE_BACK) {
            if (backListener != null) {
                backListener.onBackForward();//由监听器实现具体的操作，如果为null则不操作
                return false;
            }
        }
        return super.onKeyDown(keyCode, event);
    }
```

**3.在对应的Fragment中设置监听器，需要返回的实现，不需要返回的直接设置为null即可**

```java
        ((MainActivity)getActivity()).setBackListener(new FragmentBackListener() {
            //返回键点击间隔时间计算
            private long exitTime = 0;
            //捕捉返回键点击动作
            @Override
            public void onBackForward() {
                //和上次点击返回键的时间间隔
                long intervalTime = System.currentTimeMillis() - exitTime;
                if (intervalTime > 2000) {
                    Toast.makeText(getActivity(), "双击退出", Toast.LENGTH_SHORT).show();
                    exitTime = System.currentTimeMillis();
                }else {
                    getActivity().finish();//否则退出
                }
            }
        });
```

如果不需要，传入null即可

```java
((MainActivity)getActivity()).setBackListener(null)
```

### 注解框架butterknife（黄油刀）

### 引入

 在Android Studio中可以，很快直接引入，我们可以，选择项目->右键->open modules setting，然后选择Dependencies，选择绿色的Add按钮，输入com.jakewharton:butterknife:7.0.1或者com.jakewharton:butterknife:6.1.0等等，引入框架

#### @InjectView()

注入View，等价于findViewById(),例：

```java
@InjectView(R.id.listview)

ListView listview;
```

### 杂记

#### Fragment中发现View

Activity中使用findViewById()，Fragment中使用getView().findViewById()，getView()返回当前视图的根视图

如果在onCreateView中使用inflater加载布局则不能这样用，应该

```java
    public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
        View view = inflater.inflate(R.layout.fragment, container, false);
        Button btn = (Button) view.findViewById(R.id.btn);
    }

```

#### RadioButton美化

```xml
        <RadioButton
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:layout_weight="1"
            android:background="@android:color/transparent"//背景
            android:button="@null"//这样设置可以去掉RadioButton的默认样式
            android:drawableTop="@mipmap/creation"
            android:gravity="center"//显示内容居中"
            android:text="@string/title_creation"
            android:textColor="@color/colorPrimary"></RadioButton>
```



#### 去除顶部默认的标题栏

res/values/styles将AppTheme的值改为Theme.AppCompat.Light.NoActionBar

#### 常见错误

#####  Didn't find class "android.support.v4.view.ViewPager"

改为androidx.viewpager.widget.ViewPager