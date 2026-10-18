---
来源站点: javaup.chat（码力全开）
原文链接: https://javaup.chat/java/collect/concurrenthashmap-thread-safety/
抓取日期: 2026-10-18
所属知识域: 域1-Java核心与编码工程化
板块: Java 集合 - ConcurrentHashMap 线程安全
---

# ConcurrentHashMap 线程安全机制：CAS、synchronized、volatile 详解

## 线程安全实现概述

不同于 Hashtable 的粗粒度全表锁（所有公共方法 synchronized，put 时其他读写全等），ConcurrentHashMap 用细粒度锁策略，允许多线程同时访问不同数据段。

## JDK 1.7 的分段锁机制

### Segment 架构

整个哈希表划分为多个独立段，每段独立管理自己的锁。每个 Segment **继承 ReentrantLock**（本身即可重入锁）：

```java
static class Segment<K,V> extends ReentrantLock implements Serializable {
    transient volatile HashEntry<K,V>[] table;  // 段内哈希表
    transient int count;                        // 元素数量
    transient int modCount;                     // 修改次数
    transient int threshold;                    // 扩容阈值
    final float loadFactor;
}
```

- **默认并发度 16**：Segment 数组默认 16，支持 16 线程同时写不同段；
- 锁隔离：不同 Segment 操作互不干扰；
- 局部锁定：仅修改特定段时获取对应 Segment 锁。

## JDK 1.8 的节点锁优化

### 架构变革

摒弃 Segment，改用 **Node 数组 + CAS + synchronized** 组合。

### 核心数据结构

```java
static class Node<K,V> implements Map.Entry<K,V> {
    final int hash;
    final K key;
    volatile V val;          // volatile 保证可见性
    volatile Node<K,V> next;
}
```

TreeBin 红黑树包装器（root/first/waiter/lockState，WRITER/WAITER/READER 状态位）。

### 初始化并发控制（sizeCtl）

`initTable()`：sizeCtl < 0 表示其他线程正在初始化（`Thread.yield()` 让出 CPU）；否则 CAS 竞争初始化权限（SIZECTL 从 sc 改为 -1），成功则建表，`sizeCtl = n - (n >>> 2)`（0.75×容量）。

### putVal 并发控制（四种场景）

```java
// 场景1: 表未初始化 → initTable()
// 场景2: 目标桶为空 → CAS 无锁插入（casTabAt）
// 场景3: 桶首节点 hash == MOVED → helpTransfer 协助扩容
// 场景4: 桶已有数据 → synchronized (f) 锁桶首节点
//          - fh >= 0: 链表插入（尾插，遍历找相等 key 覆盖/挂尾）
//          - f instanceof TreeBin: putTreeVal 红黑树插入
//         binCount >= TREEIFY_THRESHOLD(8) → treeifyBin 树化
```

**关键设计：锁的是桶的首节点 f，而不是整张表**——不同桶之间完全无锁。

### 多线程协同扩容

- 每线程分配至少 16 个桶的迁移任务；
- `transferIndex` 记录待迁移桶索引，CAS 确保唯一分配；
- 已迁移桶用 **ForwardingNode** 标记指向新数组，其他线程遇到 MOVED 会协助迁移（helpTransfer）。

## 为何选择 synchronized 而非 ReentrantLock

1. **并发冲突概率低**：锁粒度细化到桶，多线程同桶概率极低，synchronized 大多停留在偏向锁/轻量级锁阶段（JDK15 前），损耗可忽略；
2. **JVM 优化支持**：锁消除（对象不逃逸时移除锁）、锁粗化（合并相邻同步块）、自适应自旋（失败先自旋不立即挂起，省上下文切换）；
3. **内存占用**：ReentrantLock 每个对象要维护锁状态/AQS 等待队列/持有线程/重入计数——成千上万节点继承 ReentrantLock 内存开销可观；synchronized 靠对象头 Mark Word，**无需额外对象**；
4. **编程简洁**：自动释放锁（异常也释放），无死锁风险；ReentrantLock 必须 try/finally 手动 unlock。

## 版本差异总结

| 维度 | JDK 1.7 | JDK 1.8 |
|------|---------|---------|
| 锁策略 | Segment 分段锁（ReentrantLock） | Node 节点锁（synchronized） |
| 锁粒度 | 段级别（默认 16 段） | 桶级别（数组长度） |
| 无锁化技术 | 较少 | 大量 CAS |
| 并发度 | 受限于 Segment 数量 | 理论等于数组长度 |
| 内存占用 | Segment 额外开销 | 更紧凑 |
| Hash 冲突 | 仅链表 | 链表+红黑树（≥8 转换） |

## 实战场景

- **高并发写入**（实时日志计数）：`computeIfAbsent(module, k -> new AtomicLong()).incrementAndGet()`，不同 key hash 到不同桶可并发；
- **读多写少**（配置中心缓存）：`get` 几乎无锁（volatile val 可见），`put` 仅锁目标桶。
