# Redis 5 种基本数据类型详解

> **来源**：JavaGuide（javaguide.cn）
> **原文链接**：https://javaguide.cn/database/redis/redis-data-structures-01.html
> **抓取日期**：2026-10-18
> **所属知识域**：域2 后端技术组件工程化
> **板块**：数据库 · Redis（数据类型）

---

## 一、类型与底层结构映射

5 种基本类型依赖 8 种底层数据结构：SDS、LinkedList、Dict、SkipList、Intset、ZipList、QuickList、ListPack（7.0 起取代 ZipList）。

| 类型 | 底层实现 |
|------|----------|
| String | SDS |
| List | LinkedList/ZipList（3.2 前）→ **QuickList**（3.2+）→ **ListPack**（7.0+） |
| Hash | Dict、ZipList |
| Set | Dict、Intset |
| ZSet | ZipList、SkipList |

## 二、String

- **SDS（简单动态字符串）**：二进制安全、**取长度 O(1)**（C 字符串 O(N)）、API 安全防缓冲区溢出；
- 命令：SET/GET/MSET/MGET/STRLEN/INCR/DECR/EXPIRE/SETEX；
- 场景：缓存 Session/Token/序列化对象（比 Hash 存对象更省内存）、计数器（限流计数、页面访问数）、分布式锁基础（SETNX，有缺陷）。

## 三、List

- **双向链表**，支持反向查找；LPUSH/RPUSH/LPOP/RPOP/LRANGE/LLEN；
- 队列：RPUSH + LPOP；栈：RPUSH + RPOP；
- 场景：信息流（最新文章/动态）、分页（LRANGE）；
- **不建议做消息队列**：功能简单、无 ACK、消息丢失与堆积难解决；Redis 5.0 Stream 稍好但仍简陋，专业场景用 MQ。

## 四、Hash

- String 型 field-value 映射表，适合存对象；类 JDK1.8 前 HashMap（数组+链表），Redis 做了优化；
- 命令：HSET/HMSET/HGET/HMGET/HGETALL/HEXISTS/HDEL/HLEN/HINCRBY；
- 场景：用户信息、商品信息、购物车（可单字段更新）。

## 五、Set

- 无序唯一集合，类 HashSet；SADD/SMEMBERS/SCARD/SISMEMBER；
- **核心能力：交并差**——SINTER/SUNION/SDIFF（+STORE 落库）；
- 场景：去重（点赞、UV——大数据量用 HyperLogLog）、共同关注/粉丝（交集）、好友推荐（差集）、抽奖（SPOP 不重复 / SRANDMEMBER 可重复）。

## 六、Sorted Set（ZSet）

- Set + score 权重，按 score 有序；类 HashMap + TreeSet；
- 命令：ZADD/ZCARD/ZSCORE/ZRANGE/ZREVRANGE/ZREVRANK、ZINTERSTORE/ZUNIONSTORE/ZDIFFSTORE（score 可 SUM/MIN/MAX 聚合）；
- 场景：排行榜（送礼榜/步数榜/热度榜）、优先级任务队列；
- 底层：ZipList（小数据）或 **SkipList（跳表）**——范围查询与排名 O(log N) 的结构取舍。

## 七、选型速查

| 需求 | 类型 |
|------|------|
| 简单 KV / 计数 / 锁 | String |
| 最新列表 / 简单队列 | List |
| 对象存储 / 单字段更新 | Hash |
| 去重 / 交并差 | Set |
| 排序 / 排名 / 延迟任务 | ZSet |
