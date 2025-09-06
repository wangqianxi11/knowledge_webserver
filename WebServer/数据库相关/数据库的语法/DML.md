# DML（Data Manipulation Language）：数据操作语言
## 1.插入数据
### 语法一：
INSERT INTO 表名(列名1，列名2，...);
```sql
INSERT INTO stu(sid, sname,age,gender) VALUES('s_1001', 'zhangSan', 23, 'male');
```
### 语法二：
INSERT INTO 表名 VALUES(值1，值2，...);

没有指定插入的列，按表创建时的列的顺序插入
```sql
INSERT INTO stu VALUES('s_1002', 'liSi', 32, 'female');
```

<b><font style="color:red">注意：所有字符串数据必须使用单引用！</font></b>

## 2.修改数据
### 语法：
UPDATE 表名 SET 列名1=值1,...,列名n=值n WHERE 条件;
```sql
UPDATE stu SET sname=’zhangSanSan’, age=’32’, gender=’female’ WHERE sid=’s_1001’;

UPDATE stu SET sname=’liSi’, age=’20’WHERE age>50 AND gender=’male’;

UPDATE stu SET sname=’wangWu’, age=’30’WHERE age>60 OR gender=’female’;

UPDATE stu SET gender=’female’WHERE gender IS NULL;

UPDATE stu SET age=age+1 WHERE sname=’zhaoLiu’;
```

## 3.删除数据
### 语法一：
DELETE FROM 表名 [WHERE 条件];
```sql
DELETE FROM stu WHERE sid=’s_1001’003B;
DELETE FROM stu WHERE sname=’chenQi’ OR age > 30;
DELETE FROM stu;
```
### 语法二：
TRUNCATE TABLE 表名;
```sql
TRUNCATE TABLE stu;
```
### 区别
虽然 TRUNCATE 和 DELETE 都可以删除表的所有记录，但有原理不同。

<b>DELETE的效率没有 TRUNCATE 高！</b>

TRUNCATE 其实属性 DDL 语句，因为它是先 DROP TABLE，再 CREATE TABLE。

`而且TRUNCATE删除的记录是无法回滚的，但DELETE删除的记录是可以回滚的（回滚是事务的知识！）。`