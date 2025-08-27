---
title: Redis事务的三个阶段
updated: 2025-06-14T14:13:25
created: 2025-06-14T14:13:00
---

**Redis事务的三个阶段**
1.  multi 开启事务
2.  大量指令入队
3.  exec执行事务块内命令，截止此处一个事务已经结束。
4.  discard 取消事务
5.  watch 监视一个或多个key，如果事务执行前key被改动，事务将打断。unwatch 取消监视。
事务执行过程中，如果服务端收到有EXEC、DISCARD、WATCH、MULTI之外的请求，将会把请求放入队列中排队.
