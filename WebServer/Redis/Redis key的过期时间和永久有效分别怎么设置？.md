---
title: Redis key的过期时间和永久有效分别怎么设置？
updated: 2025-06-13T17:06:01
created: 2025-06-13T17:05:31
---

## Redis key的过期时间和永久有效分别怎么设置？
通过expire或pexpire命令，客户端可以以秒或毫秒的精度为数据库中的某个键设置生存时间。
与expire和pexpire命令类似，客户端可以通过expireat和pexpireat命令，以秒或毫秒精度给数据库中的某个键设置过期时间，可以理解为：让某个键在某个时间点过期。
