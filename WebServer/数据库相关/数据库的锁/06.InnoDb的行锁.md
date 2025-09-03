---
title: InnoDb的行锁
updated: 2025-06-15T17:06:36
created: 2025-06-13T16:12:21
---

InnoDB 是 MySQL 默认的存储引擎，它支持高并发事务控制，**核心机制就是“行级锁”**。InnoDB 的行锁机制其实非常细致，主要分为以下几种模式：

### InnoDB 的行锁模式
#### 一、两种基本行锁类型
| **锁类型** | **简称** | **作用描述** | **示例** |
|----|----|----|----|
| **共享锁** | S锁 | 允许多个事务读取行，不能修改 | SELECT … LOCK IN SHARE MODE |
| **排他锁** | X锁 | 一个事务独占行，允许读写，其他事务不能读写该行 | SELECT … FOR UPDATE，DML操作 |

#### 二、InnoDB 实际上的三种“行锁”方式（细化）
InnoDB 实际不只有 S/X 锁，还使用以下三种方式加锁：
| **锁模式** | **含义** | **应用场景和说明** |
|----|----|----|
| **Record Lock** | 精确锁定某一行记录 | 如：SELECT \* FROM t WHERE id = 5 FOR UPDATE; |
| **Gap Lock** | 锁定“两个记录之间的间隙” | 防止插入新记录，常用于防止幻读 |
| **Next-Key Lock** | 锁定当前行 + 其后间隙（Record Lock + Gap Lock） | Repeatable Read 下的默认模式，用于防止幻读 |

说明：
- Gap Lock **不会锁住已有的行数据**，只锁间隙。
- Next-Key Lock 是 InnoDB 在默认隔离级别下最常用的组合锁。
#### 三、示例说明
假设表中有以下主键数据：
id: 10, 20, 30
```sql
SELECT * FROM t WHERE id = 20 FOR UPDATE;
```
- **Record Lock**: 锁住 id=20 的行
- **Gap Lock**: 锁住 (10, 20) 和 (20, 30) 的间隙（若是 Next-Key Lock）
- InnoDB 默认使用 **Next-Key Lock**（除非唯一索引等特定情况会退化为 Record Lock）
```sql
SELECT * FROM t WHERE id > 15 AND id < 25 FOR UPDATE;
```
- 锁住满足条件的记录（id=20）
- 同时锁住间隙 (15, 20)、(20, 25)，防止插入新数据导致“幻读”

#### 四、几点重要说明
1.  **InnoDB 的行锁是通过索引加的**：
    - 若没有用索引，会锁整张表（退化为表锁）。
    - 建议查询加锁操作一定要使用索引字段。
2.  **主键/唯一索引可触发 Record Lock（避免间隙锁）**：
    - 精确匹配唯一索引 + FOR UPDATE，InnoDB 会使用最小粒度锁。
3.  **在 Read Committed 隔离级别下，InnoDB 不使用间隙锁**，仅使用 Record Lock；
    - 但可能会出现幻读，需要开发者自己处理。

