# DQL（Data Query Language）：数据查询语言

数据库执行 DQL 语句不会对数据进行改变，而是让数据库发送结果集给客户端。

**语法**：

select 列名 ----> 要查询的列名称

from 表名 ----> 要查询的表名称

where 条件 ----> 行条件

group by 分组列 ----> 对结果分组

having 分组条件 ----> 分组后的行条件

order by 排序列 ----> 对结果分组

limit 起始行, 行数 ----> 结果限定

## 创建数据库：info，在info下创建表如下：
|字段名称|字段类型|说明|
|------|-------|------------|
|sid|char(6)|学生学号|
|sname|varchar(50)|学生姓名|
|age|int|学生年龄|
|gender|varchar(50)|学生性别|

```sql
CREATE TABLE stu (
	sid CHAR(6), 
	sname VARCHAR(50), 
	age INT, 
	gender VARCHAR(50)
);
INSERT INTO stu VALUES('S_1001', 'liuYi', 35, 'male');
INSERT INTO stu VALUES('S_1002', 'chenEr', 15, 'female');
INSERT INTO stu VALUES('S_1003', 'zhangSan', 95, 'male');
INSERT INTO stu VALUES('S_1004', 'liSi', 65, 'female');
INSERT INTO stu VALUES('S_1005', 'wangWu', 55, 'male');
INSERT INTO stu VALUES('S_1006', 'zhaoLiu', 75, 'female');
INSERT INTO stu VALUES('S_1007', 'sunQi', 25, 'male');
INSERT INTO stu VALUES('S_1008', 'zhouBa', 45, 'female');
INSERT INTO stu VALUES('S_1009', 'wuJiu', 85, 'male');
INSERT INTO stu VALUES('S_1010', 'zhengShi', 5, 'female');
INSERT INTO stu VALUES('S_1011', 'xxx', NULL, NULL);
```

## 查询基础
### 查询所有列
>SELECT * FROM 表名;
```sql
SELECT * FROM stu;
```
(*:通配符)

### 查询指定列
>SELECT 列名 1, 列名 2, …列名 n FROM 表名;
```sql
SELECT sid, sname, age FROM stu;
```

## 条件查询
### 条件查询介绍
条件查询就是在查询时给出 WHERE 子句，在 WHERE 子句中可以使用如下运算符及关键字：

- =、!=、<>、<、<=、>、>=；

- BETWEEN…AND；

- IN(set)；

- IS NULL；

- AND；

- OR；

- NOT；
### 举例说明
- 查询性别为女，并且年龄小于 50 的记录
    ```sql
    SELECT * FROM stu WHERE gender='female' AND age<50;
    ```
- 查询学号不是 S_1001，S_1002，S_1003 的记录
    ```sql
    SELECT * FROM stu
    WHERE sid NOT IN ('S_1001','S_1002','S_1003');
    ```

## 3.模糊查询
>SELECT 字段 FROM 表 WHERE 某字段 Like 条件

其中关于条件，SQL 提供了两种匹配模式：

- % ：表示任意 0 个或多个字符。可匹配任意类型和长度的字符，有些情
况下若是中文，请使用两个百分号（%%）表示。
- _ ： 表示任意单个字符。匹配单个任意字符，它常用来限制表达式的字 符长度语句。

### 举例说明
- 查询姓名由 5 个字母构成的学生记录
    ```sql
    SELECT * FROM stu WHERE sname LIKE '_ _ _ _ _';
    ```
- 查询姓名以“z”开头的学生记录
    ```sql
    SELECT * FROM stu WHERE sname LIKE 'z%';
    ```
    其中“%”匹配 0~n 个任何字母。

## 4.字段控制查询
### 去掉重复记录
去除重复记录（两行或两行以上记录中系列的上的数据都相同），例如 emp 表中 sal 字段就存在相同的记录。当只查询 emp 表的 sal 字段时，那么会出现重复记录，那么想去除重复记录，需要使用 `DISTINCT`：
```sql
SELECT DISTINCT sal FROM emp;
```

### 查看雇员的月薪与佣金之和
因为 sal 和 comm 两列的类型都是数值类型，所以可以做加运算。如果 sal 或 comm 中有一个字段不是数值类型，那么会出错。
```sql
SELECT *,sal+comm FROM emp;
```
comm 列有很多记录的值为 NULL，因为任何东西与 NULL 相加结果还是 NULL，所以结算结果可能会出现 NULL。下面使用了把 NULL 转换成数值 0 的函数 IFNULL：
```sql
SELECT *, sal+IFNULL(comm,0) FROM emp;
```

### 给列名添加别名
在上面查询中出现列名为 sal+IFNULL(comm,0)，这很不美观，现在我们给这一列给出一个别名，为 total：
```sql
SELECT *, sal+IFNULL(comm,0) AS total FROM emp;
```
给列起别名时，是可以省略 AS 关键字的：
```sql
SELECT *, sal+IFNULL(comm,0) total FROM emp;
```

## 5.排序
### 查询所有学生记录，按年龄升序排序
```sql
SELECT * FROM stu
ORDER BY sage ASC;
或者
SELECT * FROM stu ORDER BY sage;
```

### 查询所有学生记录，按年龄降序排序
```sql
SELECT * FROM stu
ORDER BY age DESC;
```

### 查询所有雇员，按月薪降序排序，如果月薪相同时，按编号升序排序
```sql
SELECT * FROM emp
ORDER BY sal DESC ,empno ASC;
```

## 聚合函数
聚合函数是用来做纵向运算的函数：

- `COUNT()`：统计指定列不为 NULL 的记录行数；
- `MAX()`：计算指定列的最大值，如果指定列是字符串类型，那么使用字符串排序运算；
- `MIN()`：计算指定列的最小值，如果指定列是字符串类型，那么使用字符串排序运算；
- `SUM()`：计算指定列的数值和，如果指定列类型不是数值类型，那么计算结果为 0；
- `AVG()`：计算指定列的平均值，如果指定列类型不是数值类型，那么计算结果为 0；
### 查询 emp 表中月薪大于 2500 的人数：
```sql
SELECT COUNT(*) FROM emp WHERE sal > 2500;
```
## 分组查询
当需要分组查询时需要使用 GROUP BY 子句，例如查询每个部门的工资和，这说明要使用部分来分组。
```sql
SELECT deptno, SUM(sal)
FROM emp
GROUP BY deptno;
```

### HAVING子句
查询工资总和大于 9000 的部门编号以及工资和：
```sql
SELECT deptno, SUM(sal)
FROM emp
GROUP BY deptno
HAVING SUM(sal) > 9000;
```
<b>注意，WHERE 是对分组前记录的条件，如果某行记录没有满足 WHERE 子句的条件，那
么这行记录不会参加分组；而 HAVING 是对分组后数据的约束。</b>

## 8.LIMIT：用来限定查询结果的起始行，以及总行数。
### 查询 5 行记录，起始行从 0 开始
```sql
SELECT * FROM emp LIMIT 0, 5;
```
<b>注意，起始行从 0 开始，即第一行开始！</b>


### 查询 10 行记录，起始行从 3 开始
```sql
SELECT * FROM emp LIMIT 3, 10;
```

## SELECT的返回值
SELECT 语句<b>返回的是“行”（Row），更准确地说，是返回一个由行组成的结果集（Result Set）。</b>

### 1. 核心概念：结果集（Result Set）
当你执行一条 SELECT 语句时，MySQL 服务器会进行以下操作：

- 解析查询。

- 根据 WHERE、JOIN 等条件在表中查找数据。

- 将找到的所有符合条件的数据**临时组装成一个表格**。

- 将这个临时的表格返回给客户端（如命令行、PHP、Java 程序等）。

<b>这个返回的临时表格就叫做 结果集（Result Set）。</b>

### 2. 结果集的构成
这个结果集由两个基本单位构成：

- 行（Row）： 也常被称为“记录”（Record）。每一行代表一条符合查询条件的完整数据。

- 列（Column）： 也常被称为“字段”（Field）。每一列代表一个特定的数据属性，其名称由 SELECT 后面指定的列名或表达式决定。

### 举例
假设我们有一个 `users` 表：

|id|name|age|	city|
|-----|------|-----|----|
|1	|张三	|25|	北京|
|2|	李四|	30|	上海|
|3|	王五|	28|	广州|

```sql
SELECT name, city FROM users WHERE age > 26;
```

返回的结果集将是：

|name|	city|
|-----|------|
|李四|	上海|
|王五|	广州|

这个结果集包含了 <b>2 行 数据（李四和王五的记录）。</b>

每一行有 <b>2 列（name 和 city）。</b>


## JOIN条件
<b>JOIN 条件是在 SQL 查询中，用于指定两个或多个表之间如何建立关联（如何匹配行）的规则。 </b>它通常位于 ON 或 USING 子句中，是 JOIN 语句的核心部分。

### 为什么需要 JOIN 条件？
数据库设计时，为了减少数据冗余和保持一致性，通常会将数据分散到多个相关的表中（这称为“规范化”）。JOIN 条件就是当你需要从这些关联表中组合数据时，告诉<b>数据库引擎根据什么规则来匹配这些表中的行。</b>

如果没有 JOIN 条件，或者条件写错，会产生笛卡尔积（Cartesian Product），即一个表的每一行都与另一个表的每一行组合，导致结果集异常庞大且毫无意义。

### JOIN 条件的语法和位置
JOIN 条件最常出现在 ON 子句中，紧跟在 JOIN 关键字之后。
```sql
SELECT ...
FROM table1
[INNER | LEFT | RIGHT | FULL] JOIN table2
    ON table1.column_name = table2.column_name; -- 这就是 JOIN 条件
WHERE ...
```

### JOIN 条件 与 WHERE 条件的区别
这是一个关键点：

- JOIN 条件 (ON): 决定**如何将表连接起来**，指定表之间的匹配规则。它在连接发生时起作用。

- WHERE 条件: 在连接完成之后，对**已经连接好的结果集进行过滤**。它决定最终结果集中包含哪些行。

对于 INNER JOIN，将条件放在 ON 子句和 WHERE 子句中，结果通常是相同的。但对于 OUTER JOIN (LEFT/RIGHT JOIN)，区别巨大！

### OUTER JOIN 的三种类型
主要有三种外连接，它们决定要保留哪个表的全部记录。

#### 1. LEFT (OUTER) JOIN - 左外连接
<b>规则：返回左表 (FROM 后的表) 的所有行，即使右表中没有匹配的行。如果右表没有匹配，则结果集中右表的部分全部为 NULL。</b>
```sql
SELECT 列名
FROM 左表
LEFT [OUTER] JOIN 右表
    ON 连接条件;
```
#### 2. RIGHT (OUTER) JOIN - 右外连接
<b>规则：与左连接相反。返回右表的所有行，即使左表中没有匹配的行。如果左表没有匹配，则结果集中左表的部分全部为 NULL。</b>
```sql
SELECT 列名
FROM 左表
RIGHT [OUTER] JOIN 右表
    ON 连接条件;
```
#### 3. FULL (OUTER) JOIN - 全外连接
<b>规则：结合了左连接和右连接的效果。只要左表或右表中有一个表存在匹配的行，就返回一行。它会返回两个表的所有行，并在缺少匹配的地方用 NULL 填充。</b>
```sql
SELECT 列名
FROM 左表
FULL [OUTER] JOIN 右表
    ON 连接条件;
```
<b> 注意：MySQL 不直接支持 FULL OUTER JOIN，但可以通过组合 LEFT JOIN 和 RIGHT JOIN 并使用 UNION 来模拟实现。</b>
