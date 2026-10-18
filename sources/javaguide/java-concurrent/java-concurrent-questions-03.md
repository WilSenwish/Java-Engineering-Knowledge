# Java 并发常见面试题总结（下）

> **来源**：JavaGuide（javaguide.cn）
> **原文链接**：https://javaguide.cn/java/concurrent/java-concurrent-questions-03.html
> **抓取日期**：2026-10-18
> **所属知识域**：域1 Java核心与编码工程化
> **板块**：Java并发（ThreadLocal / 线程池 / Future / AQS）

---

## 一、ThreadLocal

### 1.1 原理

- 最终的变量存放在当前线程的 `ThreadLocalMap` 中，**不是存在 `ThreadLocal` 对象上**；
- 每个 `Thread` 内部持有 `threadLocals`（和 `inheritableThreadLocals`）两个 `ThreadLocalMap`，默认 null，首次 set/get 时创建；
- `ThreadLocalMap` 是定制化 HashMap：key 为 `ThreadLocal` 实例，value 为业务值；Entry 定义 `static class Entry extends WeakReference<ThreadLocal<?>>`——**key 弱引用、value 强引用**；
- 定位方式：`int i = key.threadLocalHashCode & (len-1)`（斐波那契哈希 + 线性探测）。

### 1.2 内存泄漏机理

- key 是弱引用：`ThreadLocal` 实例失去强引用后，下次 GC 即被回收，key 变 null；
- value 仍是强引用：Entry 强引用 value，**key 为 null 的 stale entry 无法被回收**；
- 泄漏需同时满足：① ThreadLocal 实例失去强引用；② 线程长期存活（线程池场景放大）；
- `ThreadLocalMap` 仅在 get/set/remove 时**被动清理** stale entry，不完全可靠；
- 弱引用设计是"兜底"：即使忘 remove，GC + 自清理仍有机会回收；若 key 为强引用则一旦忘记 remove 就完全无法补救。

### 1.3 使用规范（阿里手册强制）

```java
private static final ThreadLocal<UserContext> userContextHolder = new ThreadLocal<>();

public void processRequest(HttpServletRequest request) {
    try {
        userContextHolder.set(buildUserContext(request));
        doBusinessLogic();
    } finally {
        userContextHolder.remove();   // 必须清理
    }
}
```

要点：① 声明为 `static final`（避免重复创建实例留下过期条目）；② try-finally 保证 remove 一定执行；③ 用完立即清理而非依赖下次覆盖。

### 1.4 线程池场景双重风险

1. **内存累积**：复用线程的 ThreadLocalMap 中残留 value；
2. **数据污染（脏读）**：上一个任务的用户上下文被下一个任务读到——美团真实案例：用户 A 请求信息被用户 B 读到，造成数据串号。

### 1.5 跨线程传递

| 方案 | 原理 | 局限 |
|------|------|------|
| InheritableThreadLocal | `Thread.init()` 时把父线程 inheritableThreadLocals 复制给子线程 | **一次性**，线程池复用线程时拿到的是上一个任务的脏数据 |
| TransmittableThreadLocal（阿里 TTL） | 装饰器模式，把传递时机从"线程创建时"延迟到"任务提交与执行时" | 需引入依赖 |

**TTL 三阶段（CRR）**：
- **Capture**：`execute()` 提交任务瞬间 `TtlRunnable` 调用 `Transmitter.capture()` 抓取父线程所有活跃 TTL 值存入快照；
- **Replay**：工作线程 `run()` 前把快照 `set` 到当前线程，并备份旧值；
- **Restore**：任务结束后按备份恢复线程，防污染。

接入方式：① 显式包装 `TtlRunnable.get()` / `TtlExecutors.getTtlExecutorService()`；② Java Agent 字节码增强（默认修饰 ThreadPoolExecutor、ScheduledThreadPoolExecutor、ForkJoinTask 体系、TimerTask）。

```xml
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>transmittable-thread-local</artifactId>
    <version>2.14.4</version>
</dependency>
```

应用场景：压测流量标记、Trace ID 透传、用户上下文传递。

---

## 二、线程池

### 2.1 为什么用线程池

降低资源消耗（线程复用）、提高响应速度（核心线程常驻）、提高可管理性（统一参数/监控/限流）。阿里手册强制：线程资源必须通过线程池提供，不允许自行显式创建线程。

### 2.2 为什么不推荐 Executors 内置工厂

| 工厂 | 队列/线程数 | 风险 |
|------|------------|------|
| FixedThreadPool / SingleThreadExecutor | LinkedBlockingQueue 默认 Integer.MAX_VALUE（无界） | 任务堆积 OOM |
| CachedThreadPool | SynchronousQueue + maximumPoolSize=Integer.MAX_VALUE | 线程数爆炸 OOM |
| ScheduledThreadPool | DelayedWorkQueue 无界 | 任务堆积 OOM |

### 2.3 七参数

```java
public ThreadPoolExecutor(int corePoolSize, int maximumPoolSize,
    long keepAliveTime, TimeUnit unit,
    BlockingQueue<Runnable> workQueue,
    ThreadFactory threadFactory,
    RejectedExecutionHandler handler)
```

- 最重要三参数：`corePoolSize` / `maximumPoolSize` / `workQueue`；
- 工作线程 < corePoolSize 时新任务优先触发创建线程；达到 corePoolSize 后入队；入队失败且 < maximumPoolSize 才继续创建线程。

### 2.4 执行流程（4 步）

1. 运行线程数 < corePoolSize → 新建核心线程执行；
2. ≥ corePoolSize → 任务入 workQueue；
3. 入队失败（队列满）且 < maximumPoolSize → 新建非核心线程执行；
4. ≥ maximumPoolSize → 触发拒绝策略 `RejectedExecutionHandler.rejectedExecution()`。

预热：`prestartCoreThread()` / `prestartAllCoreThreads()`。

### 2.5 核心线程回收与状态

- 默认 `allowCoreThreadTimeOut=false`，核心线程永不回收；
- 置 true 后空闲核心线程按 keepAliveTime 回收（`interruptIdleWorkers()`）；
- `getTask()` 中 `timed = allowCoreThreadTimeOut || wc > corePoolSize`：timed=true 用 `poll(timeout)`（超时退出，TERMINATED）；timed=false 用 `take()`（永久阻塞 WAITING）。

### 2.6 拒绝策略（4 种）

| 策略 | 行为 | 适用 |
|------|------|------|
| AbortPolicy（默认） | 抛 RejectedExecutionException | 关键任务不可丢失 |
| CallerRunsPolicy | 调用 execute() 的线程自己执行（isShutdown 后丢弃） | 反压、尽量不丢任务 |
| DiscardPolicy | 直接丢弃 | 允许丢失 |
| DiscardOldestPolicy | 丢弃队列中最老任务再试入队 | 喜新厌旧 |

**CallerRunsPolicy 风险**：耗时任务回退到主线程执行会阻塞主线程，恶性循环甚至 OOM。解法：增大队列/最大线程数、**任务持久化**（自定义 RejectedExecutionHandler 落库 + 混合 BlockingQueue 优先取库中任务）。框架实践：Netty NewThreadRunsPolicy（临时线程）、ActiveMQ（限时 offer 60s 争取入队）。

### 2.7 阻塞队列选型

| 队列 | 容量 | 使用方 |
|------|------|--------|
| LinkedBlockingQueue | 默认无界 | Fixed/SingleThreadExecutor（maximumPoolSize 形同虚设） |
| SynchronousQueue | 0 容量，直接移交 | CachedThreadPool |
| DelayedWorkQueue | 无界，按延迟时间堆排序 | ScheduledThreadPool |
| ArrayBlockingQueue | 有界，数组 | 生产推荐 |
| PriorityBlockingQueue | 无界，二叉堆 | 优先级任务线程池 |

**优先级线程池**：任务实现 Comparable 或构造时传 Comparator（推荐）。风险：无界 OOM、低优先级饥饿（老化提升优先级）、排序开销。OOM 解法：继承重写 offer，超限返回 false。

### 2.8 线程异常后销毁还是复用

- `execute()`：未捕获异常 → 线程终止，线程池新建线程替补（保持配置线程数）；
- `submit()`：异常封装进 Future，调用 `Future.get()` 抛 ExecutionException，**线程继续复用**。

### 2.9 线程命名

默认 `pool-1-thread-n` 无业务含义。两种命名方式：Guava `ThreadFactoryBuilder().setNameFormat(prefix + "-%d")`；自实现 `ThreadFactory`（AtomicInteger 计数）。

### 2.10 线程数设定

- CPU 密集型：N+1（多 1 个防偶发缺页中断/暂停）；
- I/O 密集型：2N；
- 严谨公式：**最佳线程数 = N × (1 + WT/ST)**，WT=等待时间、ST=计算时间，WT/ST 可用 VisualVM 观测；
- 过小：任务排队/OOM/CPU 闲置；过大：上下文切换开销激增（上下文切换是 OS 中最耗 CPU 的操作之一）。

### 2.11 动态调参

`setCorePoolSize()` / `setMaximumPoolSize()` / `setKeepAliveTime()`。注意：运行期调小 corePoolSize，多余线程下次空闲时终止；调大时按需补启。队列长度不可直接改——美团方案：自定义 `ResizableCapacityLinkedBlockIngQueue`（去掉 capacity 的 final）。开源方案：**Hippo4j**、**Dynamic TP（dynamic-tp）**。

---

## 三、Future / CompletableFuture

### 3.1 Future 接口

5 个方法：`cancel(boolean)` / `isCancelled()` / `isDone()` / `get()` / `get(timeout, unit)`（JDK 19 起增加 `resultNow()`、`exceptionNow()`、`state()`）。FutureTask 同时实现 Future 和 Runnable，submit() 返回的就是 FutureTask。

### 3.2 CompletableFuture

- 实现 `Future` + `CompletionStage`，支持函数式编排与链式组合；
- **任务依赖**：`allOf(f1, f2).thenRunAsync(...)`；
- **异常处理四件套**：`whenComplete`（观察，保留原结果/异常）、`exceptionally`（异常时恢复为正常结果）、`handle`（无论成败都生成新结果）、`allOf`（一个异常则整体异常，需分别检查各阶段）；
- **必须自定义线程池**：不传 Executor 的异步方法默认用 `ForkJoinPool.commonPool()`——全局共享，大量任务时资源竞争与线程饥饿。自定义线程池带来：隔离性、资源控制、异常处理可控。

---

## 四、AQS

### 4.1 定位

JDK 1.5 引入的并发核心组件，为同步器（ReentrantLock/Semaphore/CountDownLatch…）提供通用框架：定义资源获取/释放的通用流程（模板方法），具体获取逻辑由子类重写 `tryAcquire`/`tryAcquireShared`/`tryRelease`/`tryReleaseShared` 实现。基于 **CLH 锁的变体队列**。

### 4.2 CLH 变体队列

- CLH 锁：基于单链表的自旋锁，线程自旋访问前驱节点状态；
- AQS 变体两点优化：① **自旋 → 自旋+阻塞**（先自旋尝试，失败再 `LockSupport.park`）；② **单向 → 双向队列**（释放锁时需唤醒后继，增加 next 指针）；
- 节点保存：thread 引用、waitStatus、prev、next；
- `state`（volatile int）表示同步状态，提供 final 的 `getState/setState/compareAndSetState`（CAS）。

### 4.3 语义示例

- ReentrantLock：state=0 无锁；lock() CAS state+1；重入累加；unlock 减到 0 才真正释放——**可重入**；
- CountDownLatch：state 初始化为 N，`countDown()` 减 1，归零时唤醒 await() 线程。

### 4.4 Semaphore

- 共享锁：state=permits，`acquire()` CAS 减 1（<0 则入队挂起），`release()` 加 1 并唤醒队首；
- 公平/非公平（默认非公平）；**无所有权约束**（获取与释放线程可不同）；
- 单机限流场景；生产推荐 Redis+Lua。

### 4.5 CountDownLatch vs CyclicBarrier

| | CountDownLatch | CyclicBarrier |
|---|---|---|
| 实现 | AQS 共享模式 | ReentrantLock + Condition |
| 计数 | 一次性，不能重置 | 可循环（Generation） |
| 角色 | 等待方 + 倒数方 | 一组线程互等 |

CyclicBarrier：`count` 减到 0 时执行 barrierCommand，`nextGeneration()` 重置 count 并唤醒全部；`BrokenBarrierException` 表示屏障破损（中断/超时）。
