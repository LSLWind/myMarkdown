### 

## 数据库设计

#### 数据库建表

顾客--customer

```mysql
create table customer(
	id int auto_increment,
	name varchar(10) not null,
	sex varchar(1) check(sex='男' or sex='女'),
	birth_date date,
    head_img varchar(200) comment "用户头像",
	province varchar(9) comment "省份",
	city varchar(10) comment "城市",
	address varchar(45) comment "具体地址",
	e_mail varchar(25),
	phone_number varchar(12) comment "绑定的手机号",
	password varchar(18) not null comment "密码",
	changes int comment "零钱",
    points int comment "积分点数",
    primary key (id)
)default charset=utf8;
```



售出商品--图书

```mysql
create table book(
	id bigint auto_increment,
    channel_id int not null,
	name varchar(30) not null,
	author varchar(20) not null,
	price int not null,
	price_discount int,
	content varchar(500),
	category varchar(2000),
	imgs varchar(400),
	press_date date,
	print_date date,
	press varchar(50),
    one_type varchar(30) comment"一级标签",
	two_type varchar(30) comment"二级标签",
    page_count int,
    words int,
    ISBN varchar(13),
    sale_count int,
	primary key(id),
    foreign key(channel_id) references channel(id) 
)default charset=utf8;
```



商场商品分类--图书频道

```mysql
create table channel(
	id int auto_increment,
	code int not null,
	name varchar(10) not null,
	book_count int,
	primary key(id)
)default charset=utf8;
```





购买图书--订单

```mysql
create table ord(
	id bigint auto_increment,
    customer_id int not null,
    book_id bigint not null,
    book_name varchar(30) not null,
    book_price int not null,
    count int not null,
    bought_date date,
    is_finish int,
    state varchar(10),
    shop_id int not null,
    primary key (id),
    foreign key(customer_id) references customer(id),
    foreign key(book_id) references book(id)
)default charset=utf8;
```



商品管理--店铺

```mysql
create table shop(
	id int auto_increment,
	name varchar(30) not null,
    head_img varchar(200),
	phone_number varchar(12),
	password varchar(20) not null comment"登录管理密码",
	address varchar(30) comment"店铺地址",
	primary key(id)
)default charset=utf8;
```



店铺上架商品--仓库

```mysql
create table depository(
	id bigint auto_increment,
	book_id bigint comment"存储的图书id",
	shop_id int comment"所属的店铺",
    shop_name varchar(30),
    shop_head_img varchar(200),
    depository_name varchar(30),
	count int comment"存货数量",
	create_date date,
	primary key(id),
	foreign key(book_id) references book(id),
	foreign key(shop_id) references shop(id)
)default charset=utf8;
```



商家与订单--处理

```mysql
create table handle(
	id long auto_increment,
	customer_id int not null
	order_id long not null,
	state varchar(20),
	is_pay tinyint,
	is_finish tinyint,
    primary key(id),
    foreign key(customer_id) references customer(id),
    foreign key(order_id) references order(id)
)default charset=utf8;
```





收藏--购物车

```mysql
create table cart(
	id int auto_increment,
	book_id bigint not null,
	customer_id int not null,
	book_name varchar(30) not null,
	book_price int not null,
    book_img varchar(200),
	count int not null,
	add_date date,
	primary key(id),
	foreign key(book_id) references book(id),
	foreign key(customer_id) references customer(id)
)default charset=utf8;
```



图书与客户--评论

```mysql
create table comment(
	id bigint auto_increment,
	book_id bigint not null,
	customer_id int not null,
    customer_name varchar(10) not null,
    customer_head_img varchar(200),
	comment_date date not null,
	content varchar(255) not null,
	primary key(id),
	foreign key(book_id) references book(id),
	foreign key(customer_id) references customer(id)	
)default charset=utf8;
```



店铺与客户--消息

```mysql
create table message(
	id bigint auto_increment,
	customer_id int not null,
	shop_id int not null,
	date datetime not null,
	content varchar(255) not null,
    state tinyint comment "状态标志，为1时表示是未读消息"
    is_to_shop tinyint comment"1表示是客户向商店发送消息，0表示是商店向客户发送消息"
	primary key(id),
	foreign key(customer_id) references customer(id),
	foreign key(shop_id) references shop(id)
)default charset=utf8;
```

## 前端设计

前端分两套显示系统，为加快响应速度，使用cdn进行静态资源引入，一是给用户的，命名为mall；二是给商家的，即后台订单管理，名为shop。

### 引入框架

#### 引入Bootstrap

```html
<head>
    <meta charset="UTF-8">
    <title>Title</title>

    <link href="https://cdn.bootcss.com/twitter-bootstrap/3.3.7/css/bootstrap.min.css" rel="stylesheet">
    <link href="https://cdn.bootcss.com/twitter-bootstrap/3.3.7/css/bootstrap-theme.min.css" rel="stylesheet">
    <script src="http://cdn.bootcss.com/jquery/1.11.1/jquery.min.js"></script>
    <script src="http://cdn.bootcss.com/bootstrap/3.3.7/js/bootstrap.min.js"></script>
    
</head>
<body>
```

#### 引入sweetalter

一个用于美化提示alter，如弹出错误框提示等的框架

```html
<link href="https://cdn.bootcss.com/sweetalert/1.1.2/sweetalert.css" rel="stylesheet">
```



```html
<script src="https://unpkg.com/sweetalert/dist/sweetalert.min.js"></script>
```

### Restful API 设计

/customer/loginPage  请求跳转到登录界面

/customer/login  ajax请求登录验证，返回响应json，成功设置token

/customer/{customerId}  参数customerId没有用，只不过用来区别表示，实际信息从token中取出来，进入用户个人中心

/customer/edit/{customerId}  参数customerId没有用，只不过用来区别表示，实际信息从token中取出来，进入个人信息编辑界面

/customer/update/{customerId} Ajax请求更新用户信息，提交表单

/customer/comment/add/{customerId} Ajax请求提交评论信息

/customer/order/stateUpdate/{orderId}  Ajax更新订单状态信息，确认订单已完成



/channel/{channelId}    根据channelId返回具体的频道的图书的数据的页面



/book/{bookId} 根据bookId返回具体图书信息的数据页面，包括具体信息与评论

/book/order/{customerId}/{bookId} 请求购买图书，进入订单页面

/book/buy   Ajax，请求购买图书，进行实际购买，传递json格式为订单order，实时生成，返回Result



/cart/add Ajax将指定用户所指定的书加入购物车，前台获取用户信息通过json传递，类型为Cart

/cart/{customerId}  指定用户id，进入购物车页面

/cart/remove/{cartId} Ajax指定cartId删除一条购物车记录，必须指定为用户删除，否则返回失败结果



/shop/login  Ajax用户请求登录

/shop/depository/{shopId}/{depositoryName}  进入具体仓库

/shop/editCount/{shopId}/{depositoryName}/{depositoryId}  进入仓库库存数据编辑页面

/shop/depositoryUpdate/{depositoryId}  Ajax请求更新仓库数据



/shop/{shopId}/{depositoryName}/depositoryBookAdd   进入上架图书页面，填写表单数据并发送

/shop/{shopId}/{depositoryName}/addBook/{count}   Ajax请求增加图书数据

/shop/{shopId}/orderManage  访问商店订单处理页面

/shop/order/stateUpdate/{orderId} Ajax更新订单状态信息，这里直接在sql中设置状态为已发货

### 后端

#### 数据库随机取数据

**先随机后条件**

```
    @Select("select * from book where id>=((select max(id) from book)-(select min(id) from book))*RAND()+(select min(id) from book) limit 10")
```

这里用到了数学知识，最小数+总数×随机数，得到一个随机记录。

建立索引之后，最大值与最小值的获取很快

**先条件后随机**

上面那个先随机在可以and查询，如果要先查询在随机则还是使用rand()，

```
select * from show_pictures where picture_type ='lunbotu' ORDER BY RAND() LIMIT 6;
```

#### 购买书籍/上架书籍时的并发处理

对于书籍数量的操作是并发操作

引入了仓库，仓库中存放了书籍的库存数量，用户进行购买时库存数量要更新，此时想到了两种解决方案：

1.对数据库中的仓库加上排它锁，先查询库存数量，如果有剩余则执行仓库更新操作，否则返回告知书籍卖完了。

2.服务器维护仓库列表，书籍有用户访问时就读取数据库中的仓库进入内存，维护仓库中各书籍的数量，用户购买时在内存中对书籍数量进行加锁，先查看数量，再执行库存更新操作，等到数量为0或要上架书籍时或每隔一段时间再向数据库中更新数据（缺点很明显，服务器一旦宕机，数据就会不一致，解决办法是另开一个线程实时写日志，根据日志可以恢复数据）。

## 补充

图书分类需要建表，数据为每个频道Channel，可更加细化





1.用户登录验证，token解析，点击购物车判断是否登录，未登录则跳转到登录界面，否则将bookId加入购物车




