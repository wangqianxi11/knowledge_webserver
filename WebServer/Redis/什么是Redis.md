---
title: 什么是Redis
updated: 2025-08-24T10:41:58
created: 2025-06-13T16:34:21
---

### Redis是什么？简述它的优缺点？
Redis本质上是一个**Key-Value类型**的内存数据库，很像[Memcached](https://zhida.zhihu.com/search?content_id=178187577&content_type=Article&match_order=1&q=Memcached&zhida_source=entity)，整个数据库加载在内存当中操作，定期通过异步操作把数据库中的数据flush到硬盘上进行保存。

﻿因为是**纯内存操作**，Redis的性能非常出色，每秒可以处理超过 10万次读写操作，是已知性能最快的Key-Value 数据库。
### 优点：
- 读写性能极高， Redis能读的速度是110000次/s，写的速度是81000次/s。
- 支持数据持久化，支持[AOF](https://zhida.zhihu.com/search?content_id=178187577&content_type=Article&match_order=1&q=AOF&zhida_source=entity)和[RDB](https://zhida.zhihu.com/search?content_id=178187577&content_type=Article&match_order=1&q=RDB&zhida_source=entity)两种持久化方式。
- 支持事务， **Redis的所有操作都是原子性的**，意思就是要么成功执行要么失败完全不执行。单个操作是原子性的。多个操作也支持事务，即原子性，通过MULTI和EXEC指令包起来。
- 数据结构丰富，除了支持string类型的value外，还支持hash、set、zset、list等数据结构。
- 支持[主从复制](https://zhida.zhihu.com/search?content_id=178187577&content_type=Article&match_order=1&q=%E4%B8%BB%E4%BB%8E%E5%A4%8D%E5%88%B6&zhida_source=entity)，主机会自动将数据同步到从机，可以进行读写分离。
- 丰富的特性 – Redis还支持 publish/subscribe， 通知， key 过期等特性。
### 缺点：
- 数据库容量受到物理内存的限制，不能用作海量数据的高性能读写，因此Redis适合的场景主要局限在较小数据量的高性能操作和运算上。
- 主机宕机，宕机前有部分数据未能及时同步到从机，切换IP后还会引入数据不一致的问题，降低了系统的可用性。
