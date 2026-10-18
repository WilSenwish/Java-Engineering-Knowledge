# MySQL 有哪些锁？

> **来源**：小林coding（xiaolincoding.com）
> **原文链接**：https://xiaolincoding.com/mysql/lock/mysql_lock.html
> **抓取日期**：2026-10-18
> **所属知识域**：域2 后端技术组件工程化
> **板块**：图解MySQL · 锁篇

---

## 一、全局锁

```sql
flush tables with read lock;   -- 整个库只读，阻塞增删改与 DDL
unlock tables;                 -- 释放（会话断开也自动释放）
```

- **用途**：全库逻辑备份，避免备份期间数据变更导致备份文件数据不一致（如用户表与商品表备份间隙发生购买 → 恢复后"钱没扣、库存少了"）；
- **替代方案**：`mysqldump --single-transaction`——备份前开启事务，RR 隔离级别 + MVCC 下业务可继续更新，备份数据是开启事务时的快照。**仅适用于支持事务的引擎（InnoDB）**；MyISAM 只能全局锁。

## 二、表级锁

| 锁 | 说明 |
|----|------|
| **表锁** | `lock tables t read`（共享）/ `write`（独占）。**限制其他线程也限制本线程**（加了 read 锁本线程连写都不行，且不能访问其他表）。InnoDB 应避免，粒度太粗 |
| **MDL（元数据锁）** | 自动加：CRUD → MDL 读锁；DDL → MDL 写锁。**事务提交后才释放** |
| **意向锁** | 加行锁前先加表级意向锁（IS/IX），目的是**快速判断表里是否有记录被加锁**，避免加独占表锁时遍历全表 |
| **AUTO-INC 锁** | 自增赋值用的特殊表锁，插入语句执行完即释放 |

### MDL 长事务事故链

线程 A 长事务未提交（持 MDL 读锁）→ 线程 C DDL 申请 MDL 写锁被阻塞 → **写锁等待优先级高于读锁**，后续所有该表 CRUD 全被阻塞 → 线程池打满。**DDL 前先查长事务**（`information_schema.innodb_trx`），必要时 kill。

### AUTO-INC 锁与 innodb_autoinc_lock_mode

- `=0`：语句结束才释放；`=2`：申请自增后立即释放（性能最高）；`=1`：普通 insert 立即释放，`insert...select` 等批量语句结束才释放；
- **`=2` + statement 格式 binlog → 主从不一致**（从库顺序执行导致 id 连续、主库并发分配导致 id 不连续）。**解法：`=2` 必须搭配 binlog_format=row**。

## 三、行级锁（InnoDB）

普通 select 是快照读不加锁；锁定读需显式：

```sql
select ... lock in share mode;   -- S 锁
select ... for update;           -- X 锁
```

**必须在事务中**（事务提交即释放）。

| 类型 | 范围 | 作用 |
|------|------|------|
| **Record Lock 记录锁** | 一条记录 | S/X 有兼容矩阵；唯一索引等值查询退化为记录锁 |
| **Gap Lock 间隙锁** | 开区间，不含记录 | 只存在于 RR/串行化；**间隙锁之间互相兼容**（都是防插入） |
| **Next-Key Lock** | Record + Gap，如 (3,5] | RR 默认，防幻读 |
| **插入意向锁** | 一个"点"，特殊间隙锁 | 插入时发现目标区间有间隙锁 → 生成插入意向锁并等待；**插入意向锁与区间上的间隙锁互斥** |

## 四、死锁

- 成因：两个事务互相等待对方持有的锁（InnoDB `innodb_lock_wait_timeout` 默认 50s 后自动回滚发起方）；
- 排查：`SHOW ENGINE INNODB STATUS` 的 LATEST DETECTED DEADLOCK；
- 规避：① 相同顺序访问表和行；② 大事务拆小，缩短持锁时间；③ 避免间隙锁范围过大（范围查询条件精确）；④ 加锁/更新走合适索引（无索引会锁全表，见 update 锁全表专题）；⑤ 设置合理锁等待超时。
