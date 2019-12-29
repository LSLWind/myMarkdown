Android的东西实在太多了，必须把UI与数据访问分开

### SQLite

 SQLite支持**五种数据类型**:NULL,INTEGER,REAL(浮点数),TEXT(字符串文本)和BLOB(二进制对象) 虽然只有五种,但是对于varchar,char等其他数据类型都是可以保存的;因为SQLite有个最大的特点: **你可以各种数据类型的数据保存到任何字段中而不用关心字段声明的数据类型**是什么,比如你 可以在Integer类型的字段中存放字符串,当然**除了声明为主键INTEGER PRIMARY KEY的字段只能够存储64位整数** 。

#### 内置对象

- **SQLiteOpenHelper**：抽象类，我们通过继承该类，然后重写数据库创建以及更新的方法， 我们还可以通过该类的对象获得数据库实例，或者关闭数据库！
- **SQLiteDatabase**：数据库访问类：我们可以通过该类的对象来对数据库做一些增删改查的操作
- **Cursor**：游标，有点类似于JDBC里的resultset，结果集！可以简单理解为指向数据库中某 一个记录的指针！

#### SQLiteOpenHelper

> - **onCreate(database)**:首次使用软件时生成数据库表
> - **onUpgrade(database,oldVersion,newVersion)**:在数据库的版本发生变化时会被调用， 一般在软件升级时才需改变版本号，而数据库的版本是由程序员控制的，假设数据库现在的 版本是1，由于业务的变更，修改了数据库表结构，这时候就需要升级软件，升级软件时希望 更新用户手机里的数据库表结构，为了实现这一目的，可以把原来的数据库版本设置为2 或者其他与旧版本号不同的数字即可！

> - **Step 1**：自定义一个类继承SQLiteOpenHelper类
> - **Step 2**：在该类的构造方法的super中设置好要创建的数据库名,版本号
> - **Step 3**：重写onCreate( )方法创建表结构
> - **Step 4**：重写onUpgrade( )方法定义版本号发生改变后执行的操作

```java
public class MyDBOpenHelper extends SQLiteOpenHelper {
    public MyDBOpenHelper(Context context, String name, CursorFactory factory,
            int version) {super(context, "my.db", null, 1); }
    @Override
    //数据库第一次创建时被调用
    public void onCreate(SQLiteDatabase db) {
        db.execSQL("CREATE TABLE person(personid INTEGER PRIMARY KEY AUTOINCREMENT,name VARCHAR(20))");
        
    }
    //软件版本号发生改变时调用
    @Override
    public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion) {
        db.execSQL("ALTER TABLE person ADD phone VARCHAR(12) NULL");
    }
}
```

#### 使用SQL操控数据库

- **execSQL**(SQL,Object[]):使用带占位符的SQL语句,这个是执行修改数据库内容的sql语句用的
- **rawQuery**(SQL,Object[]):使用带占位符的SQL查询操作 另外前面忘了介绍下Curosr这个东西以及相关属性，这里补充下： ——**Cursor**对象有点类似于JDBC中的ResultSet,结果集!使用差不多,提供一下方法移动查询结果的记录指针:
- **move**(offset):指定向上或者向下移动的行数,整数表示向下移动;负数表示向上移动！
- **moveToFirst**():指针移动到第一行,成功返回true,也说明有数据
- **moveToLast**():指针移动到最后一样,成功返回true;
- **moveToNext**():指针移动到下一行,成功返回true,表明还有元素！
- **moveToPrevious**():移动到上一条记录
- **getCount**( )获得总得数据条数
- **isFirst**():是否为第一条记录
- **isLast**():是否为最后一项
- **moveToPosition**(int):移动到指定行

1.插入数据：

```java
public void save(Person p)
{
    SQLiteDatabase db = dbOpenHelper.getWritableDatabase();
    db.execSQL("INSERT INTO person(name,phone) values(?,?)",
                new String[]{p.getName(),p.getPhone()});
}
```

2.删除数据：

```java
public void delete(Integer id)
{
    SQLiteDatabase db = dbOpenHelper.getWritableDatabase();
    db.execSQL("DELETE FROM person WHERE personid = ?",
                new String[]{id});
}
```

3.修改数据：

```java
public void update(Person p)
{
    SQLiteDatabase db = dbOpenHelper.getWritableDatabase();
    db.execSQL("UPDATE person SET name = ?,phone = ? WHERE personid = ?",
        new String[]{p.getName(),p.getPhone(),p.getId()});
}
```

4.查询数据：

```java
public Person find(Integer id)
{
    SQLiteDatabase db = dbOpenHelper.getReadableDatabase();
    Cursor cursor =  db.rawQuery("SELECT * FROM person WHERE personid = ?",
            new String[]{id.toString()});
    //存在数据才返回true
    if(cursor.moveToFirst())
    {
        int personid = cursor.getInt(cursor.getColumnIndex("personid"));
        String name = cursor.getString(cursor.getColumnIndex("name"));
        String phone = cursor.getString(cursor.getColumnIndex("phone"));
        return new Person(personid,name,phone);
    }
    cursor.close();
    return null;
}
```

5.数据分页：

```java
public List<Person> getScrollData(int offset,int maxResult)
{
    List<Person> person = new ArrayList<Person>();
    SQLiteDatabase db = dbOpenHelper.getReadableDatabase();
    Cursor cursor =  db.rawQuery("SELECT * FROM person ORDER BY personid ASC LIMIT= ?,?",
        new String[]{String.valueOf(offset),String.valueOf(maxResult)});
    while(cursor.moveToNext())
    {
        int personid = cursor.getInt(cursor.getColumnIndex("personid"));
        String name = cursor.getString(cursor.getColumnIndex("name"));
        String phone = cursor.getString(cursor.getColumnIndex("phone"));
        person.add(new Person(personid,name,phone)) ;
    }
    cursor.close();
    return person;
}
```

6.查询记录数：

```java
public long getCount()
{
    SQLiteDatabase db = dbOpenHelper.getReadableDatabase();
    Cursor cursor =  db.rawQuery("SELECT COUNT (*) FROM person",null);
    cursor.moveToFirst();
    long result = cursor.getLong(0);
    cursor.close();
    return result;      
}   
```

