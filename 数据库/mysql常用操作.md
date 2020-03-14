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

