

### 模型Model

#### 实体DAO entity

##### 频道 Channel

实现第三方接口MultiItemEntity,表示不同的新闻内容，不代表新闻类型，只代表显示方式

信息：类型 ，标题，频道吗

##### 评论 **Comment**

### UI

#### 架构

##### 主页

MainActivity+四个Fragment

MainActivity使用ViewPager与底部TabLayout，ViewPager使用第三方NoScrollViewPager锁死Fragment，不设置滑动，仅用来作为Fragment容器，适配器MainTabAdapter维护四个Fragment列表

底部TabLayout使用第三方BottomBarLayout，继承于LineraLayout，同理子项BottomBarItem继承于LineraLayout，显示图片与点击事件

**首页Fragment**

顶部搜索框（固定死一个TextView-----------搜索需修改）

之后一个TabLayout，引用第三方ColorTrackTabLayout，随选项滑动颜色渐变的TabLayout，TabLayout最右侧一个TextView，加号，点击选择频道

TavLayout配合ViewPager，使用适配器ChannelPagerAdapter，自定义，内部维护列表List<NewsListFragment\> mFragments;//每个频道的新闻列表Fragment

List<Channel\> mChannels;//频道实体

实现了随顶部TabLayout频道选择滑动

**视频Fragment**

### 第三方

#### com.chad.library.adapter.base.entity.MultiItemEntity;

 [https://github.com/CymChad/BaseRecyclerViewAdapterHelper/wiki/%E8%87%AA%E5%AE%9A%E4%B9%89%E4%B8%8D%E5%90%8C%E7%9A%84item%E7%B1%BB%E5%9E%8B](https://github.com/CymChad/BaseRecyclerViewAdapterHelper/wiki/自定义不同的item类型) 

接口 MultiItemEntity，只有一个方法

```java
public int getItemType()
```

用于返回项目类型，实现该接口意味着这个类可以有多种不同的展示形式，通过返回一个int变量标识应该使用哪种展示形式

#### Activity滑动com.github.anzewei.parallaxbacklayout.ParallaxHelper;

 https://github.com/anzewei/ParallaxBackLayout 

实现Activity的任意方向滑动

 https://blog.csdn.net/caifengyao/article/details/79471596 

#### 日志com.socks.library.KLog

 https://github.com/ZhaoKaiQiang/KLog 

全局日志打印工具

 https://blog.csdn.net/zhangli_/article/details/51685939 

#### 网络框架okhttp3

 https://square.github.io/okhttp/ 

#### 视屏播放jzvd

cn.jzvd.Jzvd

#### ColorTrackTabLayout 

 https://github.com/yewei02538/ColorTrackTabLayout 

一个自适应Tab宽度，可以滑动文字逐渐变色的TabLayout 