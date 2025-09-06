# DCL（Data Control Language）：数据控制语言
## 1.创建用户
CREATE USER '用户名@地址' IDENTIFIED BY '密码';
```sql
CREATE USER ‘user1’@localhost IDENTIFIED BY ‘123’;
CREATE USER ‘user2’@’%’ IDENTIFIED BY ‘123’;
```

## 2.给用户授权
GRANT 权限1，...,权限n  ON 数据库.*TO '用户名'@地址
```sql
GRANT CREATE,ALTER,DROP,INSERT,UPDATE,DELETE,SELECT ON mydb1.* TO 'user1'@localhost;

GRANT ALL ON mydb1.* TO user2@localhost;'
```

## 3.撤销授权
REVOKE 权限1,...,权限n ON 数据库.* FROM '用户名'@地址
```sql
REVOKE CREATE,ALTER,DROP ON mydb1.* FROM 'user1'@localhost;
```

## 4.查看用户权限
SHOW GRANTS FOR '用户名'@地址
```sql
SHOW GRANTS FOR 'user1'@localhost;
```

## 5.删除用户
DROP USER '用户名'@地址；
```sql
DROP USER ‘user1’@localhost;
```

## 6.修改用户密码（以root身份）
```sql
use mysql;
alter user '用户名'@localhost identified by '新密码';
```
