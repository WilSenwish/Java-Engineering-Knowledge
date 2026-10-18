# 揭开 Buffer Pool 的面纱

> **来源站点**：小林coding（xiaolincoding.com）
> **原文链接**：https://xiaolincoding.com/mysql/buffer_pool/buffer_pool.html
> **抓取日期**：2026-10-18
> **所属知识域**：域2 后端技术组件工程化（MySQL）
> **板块**：图解 MySQL · Buffer Pool

---

## 为什么要有 Buffer Pool？

MySQL 数据存储在磁盘，但每次从磁盘读取性能极差。InnoDB 设计**缓冲池（Buffer Pool）**提高读写性能：

- **读**：数据存在于 Buffer Pool 则直接读，否则读磁盘；
- **写**：先修改 Buffer Pool 中的页并标记为脏页，后台线程异步刷盘。

### Buffer Pool 有多大？

启动时向 OS 申请连续内存空间，默认 `128MB`。生产建议：`innodb_buffer_pool_size` 设为可用物理内存的 **60%~80%**。

### Buffer Pool 缓存什么？

- 以「页」为磁盘与内存交互单位，默认 **16KB/页**；Buffer Pool 中对应**缓存页**；
- 除索引页、数据页外，还缓存 **undo 页、插入缓存、自适应哈希索引、锁信息**等；
- 每个缓存页配一个**控制块**（表空间、页号、缓存页地址、链表节点），控制块位于 Buffer Pool 最前面，与缓存页之间可能有**碎片空间**。

> 查询一条记录时，InnoDB 会把**整个页**加载进 Buffer Pool（索引只能定位到页，不能定位到记录），再通过页目录定位具体记录。

## 如何管理 Buffer Pool？

### 三链表管理

| 链表 | 作用 |
|------|------|
| **Free 链表** | 管理空闲缓存页，节点为控制块；加载磁盘页时从 Free 链表取 |
| **Flush 链表** | 管理脏页，后台线程遍历刷盘 |
| **LRU 链表** | 管理脏页+干净页，淘汰最久未使用页 |

三种页状态：Free Page（空闲，Free 链表）、Clean Page（干净，LRU 链表）、Dirty Page（脏页，**同时在 LRU 和 Flush 链表**；刷盘后变干净页）。

### 简单 LRU 的两个问题

**1. 预读失效**：MySQL 预读（空间局部性，提前加载相邻页）会把不会被访问的页插入 LRU 头部，挤占热数据位置，甚至导致淘汰热页。

**2. Buffer Pool 污染**：单条 SQL 扫描大量数据（如 `like "%xxx%"` 全表扫描，即使结果集很小），大量只访问一次的页涌入 young 区，热数据被淘汰，缓存未命中 → 大量磁盘 IO → 性能急剧下降。

### InnoDB 的 LRU 改进

**改进一：young/old 分区**

- young 区在前，old 区在后，比例由 `innodb_old_blocks_pct` 控制（默认 37，即 young:old = 63:37）；
- 新读入的页（含预读页）只进 **old 区头部**；页被真正访问才进 young 区头部 → 预读失效的页很快从 old 区淘汰，不污染 young 区。

**改进二：old 区停留时间门槛**

- 首次访问 old 区页时记录访问时间，**只有「被访问」且「old 区停留超过 `innodb_old_blocks_time`（默认 1000ms）」**，才插入 young 区头部；
- 解决全表扫描等批量一次访问把热数据挤出的问题。

**改进三（young 区优化）**：young 区前 1/4 被访问不移动位置，仅后 3/4 被访问才移到头部，减少链表调整开销。

## 脏页什么时候刷入磁盘？

更新操作采用 **WAL（Write Ahead Log）**：先写 redo log，再写磁盘，保证崩溃恢复。以下时机触发刷脏：

1. redo log 满 → 主动刷脏；
2. Buffer Pool 空间不足，淘汰的页是脏页 → 先刷盘；
3. MySQL 空闲时后台线程定期刷适量脏页；
4. MySQL 正常关闭前全部刷盘。

> **生产信号**：开启慢 SQL 监控后，如果**偶尔**出现耗时稍长的 SQL，可能是脏页刷盘导致性能抖动；频繁出现 → 调大 Buffer Pool 或 redo log 大小。

## 总结

- Buffer Pool 默认 128MB，按 16KB 页缓存，通过 **Free / Flush / LRU 三链表**管理；
- LRU 改进：young/old 分区解决预读失效；`innodb_old_blocks_time` 时间门槛解决缓存污染；young 区前 1/4 不移动；
- 关键参数：`innodb_buffer_pool_size`（内存 60-80%）、`innodb_old_blocks_pct`（默认37）、`innodb_old_blocks_time`（默认1000ms）。
