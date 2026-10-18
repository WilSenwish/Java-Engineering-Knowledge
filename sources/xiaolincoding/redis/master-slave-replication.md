# 主从复制是怎么实现的？

> **来源站点**：小林coding（xiaolincoding.com）
> **原文链接**：https://xiaolincoding.com/redis/cluster/master_slave_replication.html
> **抓取日期**：2026-10-18
> **所属知识域**：域2/域3（Redis · 高可用）
> **板块**：图解 Redis · 集群篇

---

## 第一次同步（全量复制）

`replicaof <master-ip> <port>`（Redis 5.0 前为 `slaveof`）建立主从关系。第一次同步三阶段：

**阶段一：建立链接、协商同步**
- 从服务器发 `psync ? -1`（runID=?、offset=-1）；
- 主服务器响应 `FULLRESYNC <runID> <offset>`，告知采用全量复制。

**阶段二：主服务器同步数据**
- 主服务器 `bgsave` 生成 RDB（子进程异步，**不阻塞主线程**）发送给从服务器；
- 从服务器清空旧数据、载入 RDB；
- 三个时间间隙（生成 RDB 期间、发送 RDB 期间、从库加载 RDB 期间）的写命令写入 **replication buffer**，保证数据一致。

**阶段三：命令传播**
- 从库加载完 RDB 回复确认，主库将 replication buffer 中的写命令发给从库执行，主从一致。

## 命令传播（长连接）

- 第一次同步后维护 **TCP 长连接**（避免频繁连接开销），主库收到写命令即通过该连接传播给从库；
- **读写分离**：主库读写，从库只读并执行主库同步的命令。

## 分摊主库压力：一主多从的级联复制

多从库全量同步的两个问题：fork() 创建子进程阻塞主线程；传输 RDB 占用带宽。

方案：**从服务器也可以有自己的从服务器**（"经理"角色），分摊 RDB 生成与传输压力。

## 增量复制（Redis 2.8+）

网络断开恢复后不重新全量同步：

1. 从库发 `psync <runID> <offset>`（offset 非 -1）；
2. 主库响应 `CONTINUE`，采用增量复制；
3. 主库补发断线期间的写命令。

**两个关键结构**：
- **repl_backlog_buffer**：环形缓冲区（默认 1MB），保存最近传播的写命令；写满覆盖旧数据；
- **replication offset**：`master_repl_offset`（主库写位置）与 `slave_repl_offset`（从库读位置）的差值决定增量 or 全量。

**容量估算公式**：`repl_backlog_size ≥ second × write_size_per_second`（断线重连平均秒数 × 每秒写命令数据量），建议按 2 倍设置（`repl-backlog-size 10mb`）。缓冲被覆盖 → 退化为全量同步。

## 面试题精要

### 两个 Buffer 的区别（replication buffer vs repl_backlog_buffer）

| 维度 | replication buffer | repl_backlog buffer |
|------|-------------------|---------------------|
| 出现阶段 | 全量复制 + 增量复制 | 仅增量复制 |
| 分配方式 | 主库**每个从库连接一个** | 主库全局**只有一个** |
| 满的行为 | 连接断开，重新全量复制 | 环形覆盖旧数据 |

### 主从数据不一致应对

原因：命令复制是**异步**的，无法强一致。

1. 保证主从网络良好（避免跨机房）；
2. 外部监控程序：`INFO replication` 读 `master_repl_offset - slave_repl_offset`，差值超阈值时摘除该从库读流量（阈值设大些，避免摘完全部从库）。

### 主从切换减少数据丢失

**异步复制丢失**：主库返回 ok 后未同步即宕机。方案：`min-slaves-max-lag 10s`（从库延迟超 10s 主库拒写）+ 客户端降级写本地/磁盘/Kafka，恢复后回放。

**脑裂丢失**：主库网络分区但客户端可写，哨兵误判下线选新主，网络恢复后旧主降级为从，**全量同步清空本地数据 → 脑裂期间写入丢失**。

方案：`min-slaves-to-write 1` + `min-slaves-max-lag 12s`，配合 `down-after-milliseconds 10s`：原主卡住 15s 时无从库能在 12s 内 ACK，原主自动拒写，脑裂期间不产生新数据。

### 其他要点

- 主库每 10s 发 ping（`repl-ping-slave-period`）；从库每 1s 发 `replconf ack{offset}` 上报偏移量；
- 过期 key：主库模拟 `del` 命令发给从库执行删除（**从库不自主过期**）；
- 主库收到写命令先写内部缓冲区，**异步**发送给从库。
