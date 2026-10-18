# 事务隔离级别是怎么实现的？（MVCC）

> **来源**：小林coding（xiaolincoding.com）
> **原文链接**：https://xiaolincoding.com/mysql/transaction/mvcc.html
> **抓取日期**：2026-10-18
> **所属知识域**：域2 后端技术组件工程化
> **板块**：图解MySQL · 事务篇

---

## 一、事务四大特性与实现手段

| 特性 | 实现手段 |
|------|----------|
| 原子性 | undo log（回滚日志） |
| 持久性 | redo log（重做日志） |
| 隔离性 | **MVCC 或锁** |
| 一致性 | 持久性 + 原子性 + 隔离性 共同保证 |

## 二、三个并发问题与四种隔离级别

- **脏读**：读到其他事务**未提交**的修改；
- **不可重复读**：同一事务内多次读同一数据，前后不一致（侧重 update）；
- **幻读**：同一事务内多次范围查询，记录数量不一致（侧重 insert/delete）。

严重性：脏读 < 不可重复读 < 幻读。

| 隔离级别 | 脏读 | 不可重复读 | 幻读 |
|----------|------|-----------|------|
| 读未提交 | √ | √ | √ |
| 读提交 | × | √ | √ |
| 可重复读（InnoDB 默认） | × | × | √（很大程度上避免） |
| 串行化 | × | × | × |

- 实现方式：读未提交直接读最新版本；串行化加读写锁；**RC/RR 通过 Read View 实现，区别在 Read View 创建时机**；
- 事务启动时机：`begin/start transaction` 后**第一条 select 才是真正启动**；`start transaction with consistent snapshot` 立即启动。

## 三、RR 下幻读的避免

- **快照读**（普通 select）：MVCC 解决——看不到启动后其他事务插入的数据；
- **当前读**（select ... for update）：next-key lock（记录锁+间隙锁）解决——阻塞范围内插入；
- 不建议升到串行化：读写全加锁，并发性能大幅下降，易锁等待/死锁。

## 四、Read View 四字段

| 字段 | 含义 |
|------|------|
| m_ids | 创建 Read View 时**活跃事务 id 列表**（启动了未提交，不含创建者自身） |
| min_trx_id | m_ids 最小值；m_ids 为空时取 max_trx_id 值 |
| max_trx_id | **不是 m_ids 最大值**，而是创建时全局事务 id 计数器的当前值（下一个将分配的事务 id） |
| creator_trx_id | 创建该 Read View 的事务 id |

## 五、聚簇索引两个隐藏列

- `trx_id`：最近修改该记录的事务 id；
- `roll_pointer`：指向 undo log 中旧版本记录的指针，串成**版本链**。

## 六、可见性判断规则

trx_id 三种情况（自己的更新总是可见）：

1. `trx_id < min_trx_id` → 创建 Read View 前已提交 → **可见**；
2. `trx_id >= max_trx_id` → 创建 Read View 后才启动的事务 → **不可见**；
3. `min_trx_id <= trx_id < max_trx_id` → 看是否在 m_ids 中：在 → 创建时仍活跃 → 不可见；不在 → 已提交 → 可见。

沿 roll_pointer 向下找到第一个可见版本。

## 七、源码对应（MySQL 8.0 InnoDB `ReadView::changes_visible()`）

```cpp
bool changes_visible(trx_id_t id, const table_name_t &name) const {
    ut_ad(id > 0);
    // ① 小于低水位 或 是自己的事务 → 可见
    if (id < m_up_limit_id || id == m_creator_trx_id) {
        return (true);
    }
    check_trx_id_sanity(id, name);
    // ② 大于等于高水位 → 不可见
    if (id >= m_low_limit_id) {
        return (false);
    }
    // ③ 活跃事务列表为空 → 可见（防御性判断）
    else if (m_ids.empty()) {
        return (true);
    }
    // ④ 区间内二分查找：命中 → 不可见；未命中 → 可见
    const ids_t::value_type *p = m_ids.data();
    return (!std::binary_search(p, p + m_ids.size(), id));
}
```

**命名陷阱**：源码 `m_up_limit_id` = 概念上的 min_trx_id（低水位），`m_low_limit_id` = 概念上的 max_trx_id（高水位）——"up/low" 描述数轴上下边界，与数值大小直觉相反。

**性能要点**：m_ids 有序（`trx_sys->rw_trx_ids` 保证），`std::binary_search` O(log N)——成千上万活跃事务下单条记录可见性判断仍高效，这是 MVCC 高并发性能的关键设计之一。

## 八、RR 工作过程示例

事务 A（id=51）、B（id=52）先后启动，已提交事务 id=50 修改过记录（余额 100 万）：

- B 的 Read View：creator=52, m_ids=[51], min_trx_id=51, max_trx_id=53；
- B 第一次读：记录 trx_id=50 < 51 → 可见，读到 100 万；
- A 修改为 200 万（未提交），形成版本链（新记录 trx_id=51）；
- B 第二次读：trx_id=51 在 [51,53) 且在 m_ids=[51] → 不可见，沿版本链读到 trx_id=50 旧版本，仍为 100 万；
- A 提交；B 第三次读：**RR 全程复用启动时的 Read View**，m_ids 仍为 [51] → 依旧读到 100 万。

## 九、RC 工作过程对比

- **每次读都重新生成 Read View**；
- A 提交后，B 再读时新 Read View 的 m_ids 不再含 51 → trx_id=51 < 新 min_trx_id(53) → 可见，读到 200 万；
- 因此 RC 下同一事务多次读同一数据可能不一致（不可重复读的成因）。

## 十、总结

- RC/RR 都通过 Read View + 版本链比对实现，区别仅在 Read View 创建时机（每次读 vs 事务启动时一次）；
- RR 的普通 select 是**不加锁的快照读**；`select ... for update` 是**当前读**（读最新版本 + next-key lock）；
- undo log 的清理（purge）本文未展开，需专门资料补充。
