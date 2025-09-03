---
title: 什么是MySQL
updated: 2025-06-15T16:45:11
created: 2025-04-15T10:04:21
---

### 什么是MySQL
MySQL是一个<strong style="color:red">关系型数据库，它采用表的形式来存储数据。</strong>你可以理解成是Excel表格，既然是**表的形式存储数据，就有表结构（行和列）**。行代表每一行数据，列代表该行中的每个值。列上的值是有数据类型的，比如：整数、字符串、日期等等。

### MySQL中的行和列
在 MySQL（或任何关系型数据库）中，“**行（Row）**”和“**列（Column）**”是数据库表（table）最基本的组成单位，可以类比为 Excel 表格中的行和列：
**一张表的结构举例（User 表）**
| **id** | **username** | **email**      | **age** |
|--------|--------------|----------------|---------|
| 1      | alice        | alice@mail.com | 23      |
| 2      | bob          | bob@abc.com    | 30      |

#### 1. 列（Column）
- 是**表头**或字段定义；
- 每列代表一个属性或字段名；
- 每列都有**数据类型**（如 INT, VARCHAR, DATE）；
- 所有行的该列都遵循相同的数据类型。
**示例：**
```sql
CREATE TABLE User (  
id INT,  
username VARCHAR(100),  
email VARCHAR(100),  
age INT  
);
```
其中 id, username, email, age 就是列名（字段）。

#### 2. 行（Row）
- 是表中的一条**数据记录**；
- 每行代表一个实体（如一个用户、订单、商品）；
- 每一行中的值对应每一列。
**示例：**
```sql
INSERT INTO User VALUES (1, 'alice', 'alice@mail.com', 23);
```
这句插入了一行数据，对应一个用户的信息。
### 表结构设计
**表结构设计主要包括的内容：**
#### 1. 字段（列）的定义
- 每个字段的名称要明确、规范、有意义；
- 决定字段的数据类型（如 INT、VARCHAR、DATE 等）；
- 设置字段是否可以为 NULL、是否唯一（UNIQUE）等。
例：
```sql
CREATE TABLE User (  
id INT PRIMARY KEY AUTO_INCREMENT,  
username VARCHAR(100) NOT NULL UNIQUE,  
email VARCHAR(255),  
created_at DATETIME DEFAULT CURRENT_TIMESTAMP  
);
```
#### 2. 主键（Primary Key）设计
- 每张表通常需要一个主键，唯一标识每一行数据；
- 可以是一个自增的 id，也可以是自然键（如学号、身份证号）。

#### 3. 表之间的关系（外键设计）
- 表与表之间通过`外键（Foreign Key）`建立关联；
- 比如一个订单表可能有一个 user_id 字段，指向用户表的主键。
```sql
CREATE TABLE Order (  
id INT PRIMARY KEY,  
user_id INT,  
FOREIGN KEY (user_id) REFERENCES User(id)  
);
```
#### 4. 字段约束
- NOT NULL：不能为空
- UNIQUE：唯一
- DEFAULT：默认值
- CHECK：满足某种条件（MySQL 8.0+ 支持）
- FOREIGN KEY：外键约束

#### 5. 索引设计
- 为常用的查询条件字段创建索引，提高查询效率；
- <strong>主键默认有索引</strong>，其他如 email, username 可手动加索引。

