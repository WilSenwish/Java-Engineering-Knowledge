# Redis 常见面试题总结（下）

> **来源站点**：JavaGuide（javaguide.cn）
> **原文链接**：https://javaguide.cn/database/redis/redis-questions-02.html
> **抓取日期**：2026-10-18
> **所属知识域**：域2（Redis）
> **板块**：面试题 · Redis（下）
> **备注**：本篇实际内容为 Redis 事务、性能优化、生产问题与使用规范（集群/IO 模型等内容以问题清单形式出现）

---

## 一、Redis 事务

`MULTI` 入队 → `EXEC` 执行 → `DISCARD` 取消 → `WATCH` 监视 key（EXEC 前被**其他客户端**修改则事务不执行；同 session 内修改不影响）。

**定位：功能鸡肋，日常开发不建议使用**：
- 不满足原子性（执行出错不回滚，错误命令外其他仍执行——Redis 设计哲学认为错误应在开发阶段发现）；
- 不满足持久性（always 可基本满足但性能极差）；
- 每条命令单独网络交互，浪费资源。

**替代：Lua 脚本**（2.6+）：批量执行减少网络开销，执行期间不插入其他命令（类似原子但不完全——中途出错无法撤销）；Redis 7.0 新增 functions 更强大。

## 二、性能优化

### 批量操作三选型

| 方案 | 原子性 | 混搭不同命令 | 实现层 | Cluster 限制 |
|------|--------|-------------|--------|--------------|
| 原生批量命令（MGET/HMGET） | ✅ | ❌ | 服务端 | key 可能不在同 slot |
| pipeline | ❌ | ✅ | 服务端+客户端 | 需自行维护 key-slot 关系 |
| Lua 脚本 | ✅（不被打断） | ✅ 支持逻辑 | 服务端 | Cluster 下无法保证原子性 |

命令间有依赖 → 用 Lua；无依赖求吞吐 → pipeline；同 key 批量 → 原生命令。

### 大量 key 集中过期

定期删除遇大量过期 key 时清理任务在主线程执行阻塞请求。**解决**：随机过期时间；开启 lazy-free（4.0+ 异步释放）。

### bigkey / hotkey

- **bigkey**：String > 1MB 或复合类型元素 > 5000。危害：客户端超时阻塞、网络阻塞（1MB×1000QPS=1GB/s）、del 阻塞工作线程、集群数据倾斜；
- **hotkey**：访问显著多于其他 key（秒杀商品、热点新闻）。发现依赖 `maxmemory-policy` 为 `allkeys-lfu`/`volatile-lfu` 时 `redis-cli --hotkeys`、MONITOR（临时，影响大）、业务预估。解决：从节点读、Cluster 分散、本地二级缓存（Caffeine）。

### 慢查询命令

`KEYS *`、`HGETALL`、`LRANGE`、`SMEMBERS`、`SINTER/SUNION/SDIFF` 均 O(n)；遍历用 `HSCAN/SSCAN/ZSCAN` 替代。慢查询日志：`slowlog-log-slower-than` 默认 10ms、`slowlog-max-len` 默认 128 条。

## 三、生产问题三兄弟

| 问题 | 定义 | 解决方案 |
|------|------|----------|
| **穿透** | 缓存和数据库都无此 key | 参数校验、缓存无效 key（短 TTL）、**布隆过滤器**、接口限流 |
| **击穿** | 热点 key 缓存过期（DB 有） | 提前预热（推荐）、互斥锁回源、永不过期（不推荐） |
| **雪崩** | 大量缓存同时失效或 Redis 宕机 | 随机失效时间、预热（xxl-job 定时/Kafka 异步）、集群+多级缓存 |

**一致性**：Cache Aside——读：先读缓存，未命中读 DB 回填；写：**先更新数据库再删除缓存**。删缓存失败 → 缩短 TTL 或**MQ 异步重试删除**。

### Redis 阻塞原因清单

O(n) 命令 / `SAVE`（同步 RDB）/ AOF 写日志与刷盘（always/everysec 磁盘压力）/ AOF 重写缓冲区追加 / 大 Key / `flushdb`/`flushall` / 集群迁移同步阻塞 / 内存不足 Swap / CPU 竞争与网络。

## 四、使用规范

1. 使用连接池；2. 慎用 O(n) 命令；3. 批量操作；4. 不用事务用 Lua；5. 禁止长时间 MONITOR；6. 控制 key 生命周期防冷数据驻留。
