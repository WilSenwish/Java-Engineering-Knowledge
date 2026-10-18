---
来源站点: javaguide.cn（JavaGuide）
原文链接: https://javaguide.cn/java/concurrent/java-concurrent-questions-02.html
抓取日期: 2026-10-18
所属知识域: 域1-Java核心与编码工程化
板块: Java 并发（中）
---

# Java并发常见面试题总结（中）

> 本篇覆盖：JMM、volatile、乐观锁/悲观锁、CAS、synchronized、ReentrantLock、ReentrantReadWriteLock、StampedLock、Atomic 原子类。

## ⭐️ JMM（Java 内存模型）

相关知识点单独成篇（JMM 详解）。

## ⭐️ volatile 关键字

### 可见性

对某个 volatile 变量的写入 happens-before 于后续对同一变量的读取，读线程能看到该写入及写入前按 happens-before 传递的结果。**这是 JMM 规定的语义，不等同于"每次访问都绕过 CPU 缓存直接读写主存"**。

### 禁止指令重排序

通过对 volatile 变量的读写插入**内存屏障**实现。`Unsafe` 提供 `loadFence()`/`storeFence()`/`fullFence()`。

**4 种内存屏障类型**（实现层模型，非 JLS 强制规定）：

| 屏障类型 | 指令示例 | 说明 |
|---------|---------|------|
| LoadLoad | `Load1; LoadLoad; Load2` | Load1 先于 Load2 及其后续读完成 |
| StoreStore | `Store1; StoreStore; Store2` | Store1 效果不晚于 Store2 对其他处理器可见 |
| LoadStore | `Load1; LoadStore; Store2` | Load1 先于 Store2 刷新到内存前完成 |
| StoreLoad | `Store1; StoreLoad; Load2` | Store1 对其他处理器可见先于 Load2。**开销最大，全能屏障（Full Barrier）** |

**volatile 写屏障插入策略（保守模型）**：写前插 StoreStore，写后插 StoreLoad。**volatile 读屏障插入策略**：读后插 LoadLoad + LoadStore。实际 JVM 会根据目标处理器内存模型选择、合并或省略。

**效果**：volatile 写-读组合建立类似**锁的释放-获取语义**——volatile 写前的所有操作结果，对后续该 volatile 读之后的所有操作可见。

### 双重校验锁（DCL）单例

```java
public class Singleton {
    private volatile static Singleton uniqueInstance;  // 必须 volatile
    private Singleton() {}
    public static Singleton getUniqueInstance() {
        if (uniqueInstance == null) {
            synchronized (Singleton.class) {
                if (uniqueInstance == null) {
                    uniqueInstance = new Singleton();
                }
            }
        }
        return uniqueInstance;
    }
}
```

`new Singleton()` 分三步：① 分配内存 ② 初始化 ③ 指向内存地址。**无 volatile 时可能重排为 1→3→2**，多线程下 T2 拿到未初始化实例。加 volatile 后：写前 StoreStore 禁止 2/3 重排，写后 StoreLoad 保证可见性。

### happens-before 与 volatile

- **volatile 变量规则**：volatile 写 happens-before 于后续对它的读；
- 配合**程序顺序规则**与**传递性**：写前的普通变量修改对读线程可见（经典 `flag` 状态标志位模式）：

```java
// 线程 A: a=1; b=2; flag=true(volatile写)
// 线程 B: if(flag){ x=a; y=b; }  // x 必为 1, y 必为 2
```

### volatile 不保证原子性

`inc++` 是复合操作（读→加→写），volatile 无法保证三步原子性。5 线程各 500 次自增结果 < 2500。解决：`synchronized` / `ReentrantLock` / `AtomicInteger.getAndIncrement()`。

## ⭐️ 乐观锁和悲观锁

### 悲观锁

总假设最坏情况，每次访问都加锁，其他线程阻塞直到释放。`synchronized`、`ReentrantLock`、`SELECT ... FOR UPDATE` 都是悲观锁。高并发下激烈锁竞争→阻塞→上下文切换→性能开销，且有死锁风险。

### 乐观锁

假设最好情况，不加锁直接执行，**提交修改时验证资源是否被修改**（版本号机制或 CAS）。`java.util.concurrent.atomic` 原子类即 CAS 实现。高并发下无锁竞争无阻塞无死锁，性能往往更好；**但冲突频繁（写多）时频繁失败重试，CPU 飙升**。`LongAdder` 以空间换时间（Cell 分段）解决频繁重试问题。

### 版本号机制

数据表加 `version` 字段，更新时校验版本相等才更新。操作员 A（v=1，扣$50）与 B（v=1，扣$20）并发：A 先提交成功 version→2；B 提交时版本不符被驳回，避免旧数据覆盖新结果。

### CAS 算法

- 三操作数：**V（当前值）、E（预期值）、N（新值）**。仅当 V=E 时原子更新为 N，否则放弃；
- 底层依赖 CPU 原子指令，通过 `Unsafe.compareAndSwapObject/Int/Long` 实现；
- `AtomicInteger` 核心：`value` 字段 `volatile` + `unsafe.compareAndSwapInt`；`getAndAddInt` = do-while 自旋重试（**自旋锁机制**）。

### CAS 的三个问题

1. **ABA 问题**：值 A→B→A，CAS 误认为未修改。解决：`AtomicStampedReference`（版本号/时间戳）；
2. **循环时间长开销大**：自旋重试耗 CPU。`pause` 指令优化（延迟流水线 + 避免内存顺序冲突）；
3. **只能保证一个共享变量**：多变量需封装为对象用 `AtomicReference`，或加锁。

## synchronized 关键字

### 使用方式

1. 修饰实例方法 → 锁**当前对象实例**；
2. 修饰静态方法 → 锁**当前 Class**（所有实例共享）；
3. 修饰代码块 → `synchronized(object)` 锁给定对象 / `synchronized(类.class)` 锁 Class。

**静态 synchronized 与非静态 synchronized 不互斥**（一个锁 Class，一个锁实例）。**构造方法不能 synchronized**（内部可用同步块）。

### 底层原理

- **同步代码块**：`monitorenter` + `monitorexit` 指令（两个 monitorexit 保证正常/异常都能释放）。执行 monitorenter 时尝试获取对象监视器 monitor 的持有权，锁计数器 0→1 可获取，`monitorexit` 后归 0；
- **同步方法**：`ACC_SYNCHRONIZED` 标识，JVM 据此辨别同步方法。实例方法尝试获取实例对象锁，静态方法获取 Class 锁；
- **本质**：都是对对象监视器 monitor 的获取。Monitor 由 C++ ObjectMonitor 实现，`wait/notify` 也依赖 monitor（不在同步块调用抛 `IllegalMonitorStateException`）。

### JDK1.6 后的优化与锁升级

自旋锁、适应性自旋、锁消除、锁粗化、偏向锁、轻量级锁。**四种状态：无锁 → 偏向锁 → 轻量级锁 → 重量级锁，只能升级不能降级**。

**偏向锁废弃**（JEP 374）：JDK15 默认关闭，JDK18 彻底废弃。原因：① 收益不明显（受益者多为 HashTable/Vector 等老集合，ConcurrentHashMap 时代收益微弱；多线程竞争时撤销偏向锁需等全局安全点，开销昂贵）；② 增加 JVM 复杂性、维护成本高。

### synchronized vs volatile

| 维度 | volatile | synchronized |
|------|----------|--------------|
| 实现 | 内存屏障，无线程阻塞 | 操作系统互斥锁，用户态/内核态切换 |
| 读开销 | 与普通变量几乎相同 | 获取 monitor 锁 |
| 写开销 | 插 StoreStore+StoreLoad 屏障 | 获取/释放 monitor，竞争时阻塞切换 |
| 功能 | 只修饰变量，可见性+有序性 | 方法/代码块，可见性+有序性+原子性 |

**选择**：只需可见性（状态标志、DCL）用 volatile；需复合操作原子性必须 synchronized/Lock/原子类。

## ReentrantLock

- 实现 `Lock` 接口，可重入独占锁。内部类 `Sync` 继承 AQS，`FairSync`/`NonfairSync` 两个子类。**默认非公平**，`new ReentrantLock(true)` 公平；
- **公平 vs 非公平**：公平锁竞争时优先等待最久的（但不保证 OS 调度严格按时间）；非公平性能更好但可能饥饿。无参 `tryLock()` 不遵守公平设置；
- **synchronized vs ReentrantLock**：都可重入；synchronized 依赖 JVM（优化不暴露），ReentrantLock 依赖 API（源码可读）；ReentrantLock 多三个高级功能：
  1. **等待可中断**：`lockInterruptibly()` 等待中响应 `interrupt()` 抛 `InterruptedException`；
  2. **公平策略可配置**；
  3. **多 Condition 选择性通知**：`synchronized` 相当于只有一个 Condition，`notify/notifyAll` 由 JVM 选择唤醒；ReentrantLock 可绑多个 Condition 分组唤醒；
  4. **支持超时**：`tryLock(timeout)` 防死锁、提高响应速度。

## ReentrantReadWriteLock

- 读写两把锁：读锁共享、写锁独占。规则：**读读不互斥、读写互斥、写写互斥**。适合读多写少；
- **持有读锁不能获取写锁**（多线程可能同时持读锁，都等写锁则永远不满足独占）；**持有写锁可以获取读锁**（写锁可降级为读锁）；
- 读锁不能升级为写锁：需先释放读锁再获取写锁，获取后重新检查共享状态。

## StampedLock（JDK1.8）

- 不可重入、不支持 Condition，基于 **CLH 锁**独立实现（不实现 Lock/ReadWriteLock 接口）；
- 三种模式：写锁（独占）、读锁（悲观读，共享）、**乐观读**（tryOptimisticRead 返回 stamp，读后 validate 校验，无锁）；
- 乐观读允许一个写线程获取写锁，减少线程饥饿 → **读多写少吞吐高于 ReentrantReadWriteLock**；
- **使用复杂、易出生产问题**：写锁不可重入（同线程再获取写锁会自我死锁）、中断支持不友好（使用不当 CPU 飙升）。用前必读官方文档案例。

## Atomic 原子类

单独成篇（Atomic 原子类总结）。核心：`AtomicInteger`/`LongAdder` 等基于 CAS + volatile。
