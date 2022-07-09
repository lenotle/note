# MySQL 

## mysql8 设置密码模式

```mysql
update user set plugin='mysql_native_password' where user='root';
ALTER user 'root'@'%' IDENTIFIED WITH mysql_native_password BY '123456';
flush privileges;
```

## 关于不允许远程登录

设置 user表中host字段为通配符 '%'

```mysql
# 查看 host字段是否为 %
select host from user where user='root';
# 不是则更新
update user set host='%' where user='root';
flush privileges;
```

