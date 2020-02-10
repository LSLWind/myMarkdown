```mysql
create table student(
	id varchar(15) not null,
	name varchar(20) not null,
    sex varchar(2) not null check(sex="男" or sex="女"),
    nation varchar(20) comment "民族",
    education varchar(6) not null comment "学历层次" check(education="本科生" or education="研究生") ,
    age tinyint not null,
    department_id tinyint not null comment "院系",
    major_id int not null comment "专业",
    year year not null comment "年级",
    class_id int not null comment "班级",
    #enrollment_year date not null comment "入学年份",
    #graduation_year date not null comment "毕业年份",
    header_img varchar(40) comment "头像",
    phone_number varchar(12) comment "手机号，联系方式",
    email varchar(25) comment "邮箱，联系方式",
    gmt_create datetime comment "插入时间",
    gmt_modified datetime comment "修改时间",
    primary key(id,department_id,major_id,year,class_id),
    foreign key(class_id) references class(id),
    foreign key(year) references year(year),
    foreign key(major_id) references major(id),
    foreign key(department_id) references department(id)
)default charset=utf8;
```

```mysql
create table class(
	id int not null,
    department_id tinyint not null,
    major_id int not null,
    year year not null,
    gmt_create datetime comment "插入时间",
    gmt_modified datetime comment "修改时间",
    primary key(id,department_id,major_id,year),
    foreign key(year) references year(year),
    foreign key(major_id) references major(id),
    foreign key(department_id) references department(id)
)default charset=utf8;
```



```mysql
create table year(
	year year not null,
	department_id tinyint not null,
    major_id int not null,
    gmt_create datetime comment "插入时间",
    gmt_modified datetime comment "修改时间",
    primary key(year,major_id,department_id),
    foreign key(major_id) references major(id),
    foreign key(department_id) references department(id)
)default charset=utf8;
```



```mysql
create table major(
	id int not null,
	name varchar(30) not null,
	department_id tinyint not null,
    gmt_create datetime comment "插入时间",
    gmt_modified datetime comment "修改时间",
    primary key(id,department_id),
    foreign key(department_id) references department(id)
)default charset=utf8;
```





```mysql
create table department(
	id tinyint not null,
	name varchar(30) not null,
	header_img varchar(40) comment "院系logo",
    gmt_create datetime comment "插入时间",
    gmt_modified datetime comment "修改时间",
    primary key(id)
)default charset=utf8；
```

```mysql
create table course(
	id int not null auto_increment,
	name varchar(30) not null,
	credits tinyint not null comment "学分",
    year year not null comment "开课年级",
	major_id int not null comment "开课专业",
    department_id int not null comment "开课院系",
    create_date date not null comment "开课日期",
    end_date date not null comment "结课日期",
    gmt_create datetime comment "插入时间",
    gmt_modified datetime comment "修改时间",
    primary key(id)
)default charset=utf8;
```

```mysql
create table takes(
	id bigint not null auto_increment,
	student_id varchar(15) not null,
	course_id int not null,
    score tinyint not null,
    gmt_create datetime comment "插入时间",
    gmt_modified datetime comment "修改时间",
    primary key(id,student_id,course_id),
    foreign key(student_id) refrences student(id),
    foreign key(course_id) refrences course(id)
)default charset=utf8;
```





管理员

```mysql
create table manager(
	id int not null auto_increment,
	name varchar(40),
	password varchar(20),
    employee_id varchar(15) not null comment "工号",
    phone_number varchar(12) comment "手机号，联系方式",
    email varchar(25) comment "邮箱，联系方式",
	gmt_create datetime comment "插入时间",
    gmt_modified datetime comment "修改时间",
    primary key(id)
)default charset=utf8;
```

### API

#### 登录

/account/login   post   登录，成功跳转，失败返回信息

