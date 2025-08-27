---
title: Redis内存淘汰策略
updated: 2025-08-24T10:49:35
created: 2025-06-13T17:06:06
---

**Redis内存淘汰策略**
Redis是不断的删除一些过期数据，但是很多没有设置过期时间的数据也会越来越多，那么Redis内存不够用的时候是怎么处理的呢？答案就是淘汰策略。
当Redis的内存超过最大允许的内存之后，Redis会触发内存淘汰策略，删除一些不常用的数据，以保证Redis服务器的正常运行。
**Redisv4.0前提供 6种数据淘汰策略：**
- volatile-lru：利用**LRU算法**移除设置过过期时间的key (LRU:最近使用 Least Recently Used )
- allkeys-lru：当内存不足以容纳新写入数据时，在键空间中，**移除最近最少使用的key（这个是最常用的）**
- volatile-ttl：从已设置过期时间的数据集（server.db\[i\].expires）中挑选将要过期的数据淘汰
- volatile-random：从已设置过期时间的数据集（server.db\[i\].expires）中任意选择数据淘汰
- allkeys-random：从数据集（server.db\[i\].dict）中任意选择数据淘汰
- no-eviction：禁止驱逐数据，也就是说当内存不足以容纳新写入数据时，新写入操作会报错。这个应该没人使用吧！
**Redisv4.0后增加以下两种：**
- volatile-lfu：**从已设置过期时间的数据集(server.db\[i\].expires)中挑选最不经常使用的数据淘汰(LFU(Least Frequently Used)算法**，也就是最频繁被访问的数据将来最有可能被访问到)
- allkeys-lfu：当内存不足以容纳新写入数据时，在键空间中，移除最不经常使用的key。
内存淘汰策略可以通过配置文件来修改，Redis.conf对应的配置项是maxmemory-policy 修改对应的值就行，默认是noeviction。
