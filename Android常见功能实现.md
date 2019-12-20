吐槽：网上很多博客，很多功能实现都根本不对，只给代码却不说是怎么做的，或许在他的环境下能够运行，但是换一个环境就不行了，给的代码引用了其它代码却不告诉引用代码，好一点的好歹放个github地址，差一点的什么都没有，那就别放在网上，文抄公太多，关键是自己也没有试验过，所以还是要自己记录下来，避免踩坑。

### 点击跳转到搜索框并显示搜索历史

**页面间跳转**

Activity跳转使用intent，Fragment跳转使用Fragment管理器或者直接用导航Navigation，跳转后的页面是实际搜索页面，这里使用Activity+SQLiteOpenHelper

**历史数据保存**

历史数据直接保存在客户端本地的SQLite里，因此要先建立一个数据库，使用SQLiteOpenHelper

```java
public class RecordSQLiteOpenHelper extends SQLiteOpenHelper {

    private static String name = "temp.db";
    private static Integer version = 1;

    public RecordSQLiteOpenHelper(Context context) {
        super(context, name, null, version);
    }

    @Override
    public void onCreate(SQLiteDatabase db) {
        db.execSQL("create table records(id integer primary key autoincrement,name varchar(200))");
    }

    @Override
    public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion) {

    }

}
```

**搜索页面布局**

名为activity_search.xml

```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:focusableInTouchMode="true"
    android:orientation="vertical"
    tools:context=".ui.activity.SearchActivity"><!--注意这里，配置为自己的搜索页面的Activity-->

    <LinearLayout
        android:layout_width="fill_parent"
        android:layout_height="50dp"
        android:background="#E54141"
        android:orientation="horizontal"
        android:paddingRight="16dp">

        <ImageView
            android:layout_width="45dp"
            android:layout_height="45dp"
            android:layout_gravity="center_vertical"
            android:padding="10dp"
            android:src="@drawable/back" />

        <EditText
            android:id="@+id/et_search"
            android:layout_width="0dp"
            android:layout_height="match_parent"
            android:layout_weight="1"
            android:background="@null"
            android:drawableLeft="@drawable/search"
            android:drawablePadding="8dp"
            android:gravity="start|center_vertical"
            android:hint="输入查询的关键字"
            android:imeOptions="actionSearch"
            android:singleLine="true"
            android:textColor="@android:color/white"
            android:textSize="16sp" />

    </LinearLayout>
    <TextView
        android:id="@+id/tv_tip"
        android:layout_width="match_parent"
        android:layout_height="50dp"
        android:gravity="left|center_vertical"
        android:text="搜索历史" />

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="vertical"
        >

        <ListView
            android:id="@+id/listView"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:scrollbars="none"/>


        <Button
            android:id="@+id/tv_clear"
            android:layout_width="match_parent"
            android:layout_height="50dp"
            android:text="清除搜索历史"/>

    </LinearLayout>

</LinearLayout>

```

注意几个关键的id

**显示历史数据的listView的子项布局**

名为search_item.xml

```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

    <TextView
        android:id="@+id/history"
        android:layout_width="match_parent"
        android:layout_height="50dp" />

</LinearLayout>

```

高度可自定义，注意子项使用的便是该布局，适配器最后的数据会根据id渲染到View上，注意history这个id

**搜索页面Activity的关键函数**

```java
 @Override
    public void initView() {
        // 初始化控件
        et_search = (EditText) findViewById(R.id.et_search);
        tv_tip = (TextView) findViewById(R.id.tv_tip);
        listView = (ListView) findViewById(R.id.listView);
        tv_clear = (TextView) findViewById(R.id.tv_clear);

        // 调整EditText左边的搜索按钮的大小
        Drawable drawable = getResources().getDrawable(R.drawable.search);
        drawable.setBounds(0, 0, 60, 60);// 第一0是距左边距离，第二0是距上边距离，60分别是长宽
        et_search.setCompoundDrawables(drawable, null, null, null);// 只放左边
        et_search.setLongClickable(false);

        //listView.addFooterView(tv_clear);
    }


    @Override
    public void initListener(){
        // 清空搜索历史
        tv_clear.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                deleteData();
                queryData("");
            }
        });

        // 搜索框的键盘搜索键点击回调
        et_search.setOnKeyListener(new View.OnKeyListener() {// 输入完后按键盘上的搜索键

            public boolean onKey(View v, int keyCode, KeyEvent event) {
                if (keyCode == KeyEvent.KEYCODE_ENTER && event.getAction() == KeyEvent.ACTION_DOWN) {// 修改回车键功能
                    // 先隐藏键盘
                    ((InputMethodManager) getSystemService(Context.INPUT_METHOD_SERVICE)).hideSoftInputFromWindow(
                            getCurrentFocus().getWindowToken(), InputMethodManager.HIDE_NOT_ALWAYS);
                    // 按完搜索键后将当前查询的关键字保存起来,如果该关键字已经存在就不执行保存
                    boolean hasData = hasData(et_search.getText().toString().trim());
                    if (!hasData) {
                        insertData(et_search.getText().toString().trim());
                        queryData("");
                    }
                    // TODO 根据输入的内容模糊查询商品，并跳转到另一个界面，由你自己去实现
                    Toast.makeText(SearchActivity.this, "clicked!", Toast.LENGTH_SHORT).show();

                }
                return false;
            }
        });

        // 搜索框的文本变化实时监听
        et_search.addTextChangedListener(new TextWatcher() {
            @Override
            public void beforeTextChanged(CharSequence s, int start, int count, int after) {

            }

            @Override
            public void onTextChanged(CharSequence s, int start, int before, int count) {

            }

            @Override
            public void afterTextChanged(Editable s) {
                if (s.toString().trim().length() == 0) {
                    tv_tip.setText("搜索历史");
                } else {
                    tv_tip.setText("搜索结果");
                }
                String tempName = et_search.getText().toString();
                // 根据tempName去模糊查询数据库中有没有数据
                queryData(tempName);
            }
        });

        listView.setOnItemClickListener(new AdapterView.OnItemClickListener() {
            @Override
            public void onItemClick(AdapterView<?> parent, View view, int position, long id) {
                TextView textView = (TextView) view.findViewById(android.R.id.text1);
                String name = textView.getText().toString();
                et_search.setText(name);
                Toast.makeText(SearchActivity.this, name, Toast.LENGTH_SHORT).show();
                // TODO 获取到item上面的文字，根据该关键字跳转到另一个页面查询，由你自己去实现
            }
        });
    }

    @Override
    public void initData() {
        // 插入数据，便于测试，否则第一次进入没有数据怎么测试呀？
        Date date = new Date();
        long time = date.getTime();
        insertData("Leo" + time);

        // 第一次进入查询所有的历史记录
        queryData("");
    }

    /**
     * 插入数据
     */
    private void insertData(String tempName) {
        db = helper.getWritableDatabase();
        db.execSQL("insert into records(name) values('" + tempName + "')");
        db.close();
    }

    /**
     * 模糊查询数据
     */
    private void queryData(String tempName) {
        Cursor cursor = helper.getReadableDatabase().rawQuery(
                "select id as _id,name from records where name like '%" + tempName + "%' order by id desc ", null);


        // 创建adapter适配器对象
        adapter = new SimpleCursorAdapter(this, R.layout.search_item, cursor, new String[] { "name" },
                new int[] { R.id.history }, CursorAdapter.FLAG_REGISTER_CONTENT_OBSERVER);
        // 设置适配器

        listView.setAdapter(adapter);
        adapter.notifyDataSetChanged();
    }
    /**
     * 检查数据库中是否已经有该条记录
     */
    private boolean hasData(String tempName) {
        Cursor cursor = helper.getReadableDatabase().rawQuery(
                "select id as _id,name from records where name =?", new String[]{tempName});
        //判断是否有下一个
        return cursor.moveToNext();
    }

    /**
     * 清空数据
     */
    private void deleteData() {
        db = helper.getWritableDatabase();
        db.execSQL("delete from records");
        db.close();
    }
```

只给出关键函数，初始化调用即可

注意关键代码

```java
        // 创建adapter适配器对象
        adapter = new SimpleCursorAdapter(this, R.layout.search_item, cursor, new String[] { "name" },
                new int[] { R.id.history }, CursorAdapter.FLAG_REGISTER_CONTENT_OBSERVER);
        // 设置适配器

        listView.setAdapter(adapter);
        adapter.notifyDataSetChanged();
```

 SimpleCursorAdapter传参为ListView子项布局文件，第4个String[]为数据库列名，第5个为子项布局文件中渲染数据的View的id