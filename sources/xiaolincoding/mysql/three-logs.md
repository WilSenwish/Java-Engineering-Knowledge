# undo log、redo log、binlog 有什么用？

> **来源**：小林coding（xiaolincoding.com）
> **原文链接**：https://xiaolincoding.com/mysql/log/how_update.html
> **抓取日期**：2026-10-18
> **所属知识域**：域2 后端技术组件工程化
> **板块**：图解MySQL · 日志篇

---

## 一、三大日志总览

| 日志 | 层级 | 特性 | 用途 |
|------|------|------|------|
| **undo log** | InnoDB 引擎层 | 记录**修改前**的数据 | 事务回滚（原子性）+ MVCC 版本链 |
| **redo log** | InnoDB 引擎层 | **物理日志**（某页某偏移量做了什么修改） | 崩溃恢复（持久性），crash-safe |
| **binlog** | Server 层（所有引擎可用） | 逻辑/行格式，**追加写全量** | 备份恢复、主从复制 |

## 二、Buffer Pool 与 WAL

- 数据以 **16KB 页**为单位进出 Buffer Pool；修改在内存页上进行并标记**脏页**，后台线程择机刷盘；
- **WAL（Write-Ahead Logging）**：先写 redo log（更新即完成），脏页稍后刷盘；
- redo log 是**顺序追加写**，数据页是随机写——磁盘顺序写远快于随机写，这是 redo log 提升写性能的根本原因；
- redo log buffer 默认 16MB（`innodb_log_buffer_size`），大事务场景可调大减少刷盘；
- redo log 文件组**循环写**（ib_logfile0/1，write pos 与 checkpoint 之间可写）；**write pos 追上 checkpoint → 更新操作阻塞**，强制刷脏页推进 checkpoint。

## 三、redo log 刷盘参数

`innodb_flush_log_at_trx_commit`：

| 值 | 提交时行为 | 安全性 | 性能 |
|----|-----------|--------|------|
| 1（默认） | redo log buffer 直接 fsync 落盘 | 最高 | 最低 |
| 2 | write 到 OS page cache（未真正落盘） | OS 不挂不丢数据 | 中 |
| 0 | 留 buffer，后台线程每秒 write+fsync | MySQL 进程崩即丢 1 秒 | 最高 |

## 四、binlog

- 三种格式：**STATEMENT**（记录 SQL，动态函数 `now()/uuid()` 主从不一致）、**ROW**（记录行变更，无动态函数问题但批量更新体积大）、**MIXED**（自动选择）；
- binlog cache → 事务提交时整笔 write 到文件（**一个事务的 binlog 不可拆分**，否则从库分段执行破坏原子性）；
- `sync_binlog`：0=只 write 不 fsync（默认）；1=每次提交都 fsync（最安全）；N=累积 N 个事务 fsync（生产常设 100~1000 平衡性能）；
- **恢复被删的整库只能用 binlog**——redo log 循环写会覆盖，binlog 全量保留。

## 五、主从复制

- 三阶段：主库写 binlog → I/O 线程拉取写入 relay log → 回放线程执行；
- 三种模型：**同步**（等所有从库，性能可用性双差，基本不用）、**异步**（默认，主库宕机丢数据）、**半同步**（5.7+，等任一从库 ACK 即返回，兼顾）；
- 从库不是越多越好：主库需为每个从库建 log dump 线程，一般 1 主 2~3 从。

## 六、两阶段提交（2PC）

**问题**：redo log 与 binlog 是两个独立逻辑，任一先刷盘后崩溃 → 两份日志不一致 → 主从数据不一致。

**方案**（内部 XA：binlog 为协调者，InnoDB 为参与者）：

1. **prepare 阶段**：redo log 写 XID + 状态置 prepare，刷盘；
2. **commit 阶段**：binlog 写 XID 并刷盘 → 调引擎接口把 redo log 状态置 commit（**只需 write 到 page cache，不必 fsync**——binlog 写盘成功即事务成功的标志）。

**崩溃恢复判定**：重启扫描 redo log，遇 prepare 状态拿 XID 去 binlog 查——**有则提交，无则回滚**。

注意：redo log 可能在事务提交前就被后台线程刷盘（未提交没事，因为 binlog 未写，重启照样回滚）；**binlog 必须在事务提交后才刷盘**。

## 七、2PC 的性能问题与组提交

- **双 1 配置**（sync_binlog=1 + innodb_flush_log_at_trx_commit=1）每事务 2 次 fsync；
- **组提交**：commit 阶段拆为 flush（多事务 binlog 一起 write 到文件）/ sync（合并一次 fsync）/ commit（顺序调引擎提交）三阶段，每阶段队列+锁，leader 带队；
- 调优参数：`binlog_group_commit_sync_delay`（等 N 微秒攒更多事务）、`binlog_group_commit_no_delay_count`（攒够 N 个立即刷盘）；
- MySQL 5.7 起 redo log 也支持组提交（prepare 推迟到 flush 阶段一次刷盘）。

## 八、update 语句完整链路（以 `UPDATE t_user SET name='xiaolin' WHERE id=1` 为例）

1. 执行器经聚簇索引取记录（Buffer Pool 没有则读盘）；
2. 比较更新前后，不同则传给 InnoDB；
3. 开启事务，先写 **undo log**（旧值）到 Undo 页，改 Undo 页记 **redo log**；
4. 更新内存页标脏页，写 **redo log**（更新即完成，WAL）；
5. 写 **binlog** 到 binlog cache（未刷盘）；
6. 提交：redo log prepare 刷盘 → binlog 刷盘 → redo log 置 commit；
7. 后台线程择机刷脏页；checkpoint 推进。
