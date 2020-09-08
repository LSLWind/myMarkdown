## 建表后修改

### 修改主键自增

```mysql
alter table express change id  id bigint auto_increment;
```

如果该主键是其他表的外键，那么直接更改会受到DBMS外键约束，报错不让修改，可以先关闭外键约束，然后在启用

```mysql
SET FOREIGN_KEY_CHECKS = 0; #关闭外键约束
alter table express change id  id bigint auto_increment;
SET FOREIGN_KEY_CHECKS = 1; #启用
```

## 数据库备份

mysql提供mysqldump（位于bin下）生成sql文件

```mysql
mysqldump -u <用户名> -p <密码> --databases <要备份的数据库> > ~/<备份路径>/xx.sql
```

### 执行sql文件

在linux下可以使用管道操作符

```mysql
mysql -u <用户名> -p <密码> < ~/<sql文件路径>
```

