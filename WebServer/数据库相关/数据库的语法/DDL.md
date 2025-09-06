# DDL（Data Definition Language）：数据定义语言

## 操作数据库
### 1.创建数据库
```sql
CREATE DATABASE 库名;
```

### 2.删除数据库
```sql
DROP DATABASE 库名; -- 如果库名不存在会报错
DROP DATABASE IF EXISTS 库名; -- 不存在也不报错
```
### 3.修改数据库编码
```sql
ALTER DATABASE mydb1 CHARACTER SET utf8;
```

## 数据类型
MySQL 与 Java、C 一样，也有数据类型MySQL 中数据类型主要应用在列上。

常用类型：

- **int**：整型
- **double**：浮点型，例如 double(5,2)表示最多 5 位，其中必须有 2 位小数，即最大值为 999.99；
- **decimal**：泛型型，在表单线方面使用该类型，因为不会出现精度缺失问题；
- **char**：固定长度字符串类型；(当输入的字符不够长度时会补空格)
- **varchar**：固定长度字符串类型；
- **text**：字符串类型；
- **blob**：字节类型；
- **date**：日期类型，格式为：yyyy-MM-dd；
- **time**：时间类型，格式为：hh:mm:ss
- **timestamp**：时间戳类型；

## 操作表
### 1.创建表
```sql
CREATE TABLE 表名(
	列名 列类型, 
	列名 列类型,
	...... 
);
```
### 2.查看表的结构
```sql
DESC 表名;
```

### 3.删除表
```sql
DROP TABLE 表名;
```

### 4.修改表
```sql
添加列：给 stu 表添加 classname 列
ALTER TABLE stu ADD (classname varchar(100));

修改列的数据类型：修改 stu 表的 gender 列类型为 CHAR(2)
ALTER TABLE stu MODIFY gender CHAR(2);

修改列名：修改 stu 表的 gender 列名为 sex
ALTER TABLE stu change gender sex CHAR(2);

删除列：删除 stu 表的 classname 列
ALTER TABLE stu DROP classname;

修改表名称：修改 stu 表名称为 student
ALTER TABLE stu RENAME TO student;
```
