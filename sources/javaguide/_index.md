# JavaGuide 站点目录（javaguide.cn）

> **来源**：https://javaguide.cn/
> **定位**：Java 全栈知识 + 面试体系（GitHub 156K+ Star，620+ 贡献者，持续维护近七年）
> **抓取日期**：2026-10-18
> **知识库角色**：**主干内容**（知识域 1/2/3 骨架）
> **关联项目**：AIGuide（https://github.com/Snailclimb/AIGuide）

## 板块总览与知识域映射

| 板块 | 站点路径 | 对应知识域 | 内容数 |
|------|----------|------------|--------|
| Java 基础 | /java/basis/ | 域1 Java核心 | 15 篇 |
| Java 集合 | /java/collection/ | 域1 Java核心 | 11 篇 |
| Java 并发 | /java/concurrent/ | 域1 Java核心 | 15 篇 |
| JVM | /java/jvm/ | 域1 Java核心 | 10 篇 |
| MySQL | /database/mysql/ | 域2 技术组件 | 13 篇 |
| Redis | /database/redis/ | 域2 技术组件 | 12 篇 |
| 分布式系统 | /distributed-system/ | 域3 架构设计 | 22 篇 |
| 高性能系统 | /high-performance/ | 域3/6 架构与优化 | 12 篇 |
| 高可用系统 | /high-availability/ | 域3 架构设计 | 8 篇 |
| 系统设计 | /system-design/ | 域1/2/7 跨域 | 20 篇 |
| 计算机基础 | /cs-basics/ | 域1 基础底座 | 30 篇 |
| AI 应用开发 | /ai/ | 域6 前沿 | 26 篇 |
| AI 编程 | /ai-coding/ | 域6 前沿 | 25 篇 |

---

## 1. Java 基础（/java/basis/）→ 域1

**学习主线**：Java 基础是集合、并发、JVM、Spring 和中间件的前置能力，重点理解对象模型、参数传递、泛型擦除、反射调用、动态代理、序列化边界和框架扩展机制。

### 基础面试题
- Java基础常见面试题总结(上) — /java/basis/java-basic-questions-01.html
- Java基础常见面试题总结(中) — /java/basis/java-basic-questions-02.html
- Java基础常见面试题总结(下) — /java/basis/java-basic-questions-03.html

### 语言机制
- Java 关键字总结 — /java/basis/java-keyword-summary.html
- Java 值传递详解 — /java/basis/why-there-only-value-passing-in-java.html
- 泛型&通配符详解 — /java/basis/generics-and-wildcards.html
- Java 语法糖详解 — /java/basis/syntactic-sugar.html

### 框架底层机制
- Java 反射机制详解 — /java/basis/reflection.html
- Java 代理模式详解 — /java/basis/proxy.html
- Java SPI 机制详解 — /java/basis/spi.html
- Java 序列化详解 — /java/basis/serialization.html

### 实用细节
- BigDecimal 详解 — /java/basis/bigdecimal.html
- Java 金额用 long 还是 BigDecimal？ — /java/basis/money-long-vs-bigdecimal.html
- Java 魔法类 Unsafe 详解 — /java/basis/unsafe.html

## 2. Java 集合（/java/collection/）→ 域1

**学习主线**：集合框架接口体系、底层数据结构与扩容机制、并发容器线程安全思路、源码分析四角度（数据结构/关键字段/核心方法/并发控制）。

### 集合面试与使用规范
- Java集合常见面试题总结(上) — /java/collection/java-collection-questions-01.html
- Java集合常见面试题总结(下) — /java/collection/java-collection-questions-02.html
- Java集合使用注意事项总结 — /java/collection/java-collection-precautions-for-use.html

### List 与 Map 源码
- ArrayList 源码分析 — /java/collection/arraylist-source-code.html
- LinkedList 源码分析 — /java/collection/linkedlist-source-code.html
- HashMap 源码分析 — /java/collection/hashmap-source-code.html
- LinkedHashMap 源码分析 — /java/collection/linkedhashmap-source-code.html

### 并发集合与队列
- ConcurrentHashMap 源码分析 — /java/collection/concurrent-hash-map-source-code.html
- CopyOnWriteArrayList 源码分析 — /java/collection/copyonwritearraylist-source-code.html
- ArrayBlockingQueue 源码分析 — /java/collection/arrayblockingqueue-source-code.html
- PriorityQueue 源码分析（付费） — /java/collection/priorityqueue-source-code.html
- DelayQueue 源码分析 — /java/collection/delayqueue-source-code.html

## 3. Java 并发（/java/concurrent/）→ 域1

**学习主线**：线程生命周期、锁机制、JMM、原子操作、线程池、并发工具类放在同一条主线；CompletableFuture、ThreadLocal、虚拟线程的工程实践与风险。

### 并发面试题
- Java并发常见面试题总结（上） — /java/concurrent/java-concurrent-questions-01.html
- Java并发常见面试题总结（中） — /java/concurrent/java-concurrent-questions-02.html
- Java并发常见面试题总结（下） — /java/concurrent/java-concurrent-questions-03.html

### 锁、内存模型与同步器
- Java 锁详解 — /java/concurrent/java-lock.html
- 乐观锁和悲观锁详解 — /java/concurrent/optimistic-lock-and-pessimistic-lock.html
- CAS 详解 — /java/concurrent/cas.html
- JMM（Java 内存模型）详解 — /java/concurrent/jmm.html
- AQS 详解 — /java/concurrent/aqs.html
- 从ReentrantLock的实现看AQS的原理及应用 — /java/concurrent/reentrantlock.html

### 并发工具与工程实践
- Java 线程池详解 — /java/concurrent/java-thread-pool-summary.html
- Java 线程池最佳实践 — /java/concurrent/java-thread-pool-best-practices.html
- Java 常见并发容器总结 — /java/concurrent/java-concurrent-collections.html
- Atomic 原子类总结 — /java/concurrent/atomic-classes.html
- ThreadLocal 详解 — /java/concurrent/threadlocal.html
- CompletableFuture 详解 — /java/concurrent/completablefuture-intro.html
- 虚拟线程常见问题总结 — /java/concurrent/virtual-thread.html

## 4. JVM（/java/jvm/）→ 域1

**学习主线**：对象创建与回收、类加载、GC 对应用的影响、参数配置、线上 OOM/频繁 GC/CPU 飙高排查。

### JVM 基础与内存
- JVM 常见面试题总结 — /java/jvm/jvm-interview-questions.html
- 大白话带你认识 JVM — /java/jvm/jvm-intro.html
- Java内存区域详解（重点） — /java/jvm/memory-area.html
- 类文件结构详解 — /java/jvm/class-file-structure.html

### 类加载
- 类加载过程详解 — /java/jvm/class-loading-process.html
- 类加载器详解（重点） — /java/jvm/classloader.html

### 垃圾回收与调优
- JVM垃圾回收详解（重点） — /java/jvm/jvm-garbage-collection.html
- 最重要的JVM参数总结 — /java/jvm/jvm-parameters-intro.html
- JDK监控和故障处理工具总结 — /java/jvm/jdk-monitoring-and-troubleshooting-tools.html
- Java 后端线上问题排查 — /java/jvm/jvm-in-action.html

## 5. MySQL（/database/mysql/）→ 域2

**学习主线**：索引怎么让查询变快、事务怎么保证一致性、日志怎么保证恢复复制、数据怎么同步到异构系统、执行计划怎么定位慢 SQL。

### 总览与规范
- MySQL 常见面试题总结 — /database/mysql/mysql-questions-01.html
- MySQL 高性能优化规范建议总结 — /database/mysql/mysql-high-performance-optimization-specification-recommendations.html
- 一千行 MySQL 学习笔记 — /database/mysql/a-thousand-lines-of-mysql-study-notes.html

### 索引与执行计划
- MySQL 索引详解 — /database/mysql/mysql-index.html
- MySQL 索引失效场景总结 — /database/mysql/mysql-index-invalidation.html
- MySQL 隐式转换造成索引失效 — /database/mysql/index-invalidation-caused-by-implicit-conversion.html
- MySQL 执行计划分析 — /database/mysql/mysql-query-execution-plan.html

### 事务、MVCC、日志与数据同步
- MySQL 事务隔离级别详解 — /database/mysql/transaction-isolation-level.html
- InnoDB 存储引擎对 MVCC 的实现 — /database/mysql/innodb-implementation-of-mvcc.html
- MySQL 三大日志详解 — /database/mysql/mysql-logs.html
- MySQL备份与恢复详解 — /database/mysql/mysql-backup-and-restore.html
- MySQL 数据同步到 Elasticsearch 详解 — /database/mysql/mysql-to-elasticsearch-sync.html

### 执行过程与工程细节
- SQL 语句在 MySQL 中的执行过程 — /database/mysql/how-sql-executed-in-mysql.html
- MySQL 查询缓存详解 — /database/mysql/mysql-query-cache.html
- MySQL 自增主键一定是连续的吗？ — /database/mysql/mysql-auto-increment-primary-key-continuous.html
- MySQL 日期类型选择建议 — /database/mysql/some-thoughts-on-database-storage-time.html

## 6. Redis（/database/redis/）→ 域2

**学习主线**：缓存读写策略、底层数据结构、持久化、阻塞原因、内存管理、复制与集群等工程问题。

### 缓存基础与读写策略
- 缓存基础常见面试题总结 — /database/redis/cache-basics.html
- 3 种常用的缓存读写策略详解 — /database/redis/3-commonly-used-cache-read-and-write-strategies.html
- Redis 常见面试题总结（上） — /database/redis/redis-questions-01.html
- Redis 常见面试题总结（下） — /database/redis/redis-questions-02.html

### 数据结构与典型应用
- Redis 5 种基本数据类型详解 — /database/redis/redis-data-structures-01.html
- Redis 3 种特殊数据类型详解 — /database/redis/redis-data-structures-02.html
- Redis 为什么用跳表实现有序集合 — /database/redis/redis-skiplist.html
- 如何基于 Redis 实现延时任务？ — /database/redis/redis-delayed-task.html
- 如何基于 Redis 实现消息队列？ — /database/redis/redis-stream-mq.html

### 持久化、内存与集群
- Redis 持久化机制详解 — /database/redis/redis-persistence.html
- Redis 内存碎片详解 — /database/redis/redis-memory-fragmentation.html
- Redis 常见阻塞原因总结 — /database/redis/redis-common-blocking-problems-summary.html
- Redis 集群详解 — /database/redis/redis-cluster.html

## 7. 分布式系统（/distributed-system/）→ 域3

**学习主线**：分布式入门 → 通信调用 → 服务治理 → 一致性与协调 → 工程实践。四条线：理论线（CAP/BASE/Paxos/Raft/ZAB/Gossip）、调用线（RPC/网关）、数据一致性线（ID/锁/事务）、协调组件线（配置中心/ZooKeeper）。

### 分布式基础与理论协议
- 分布式系统入门 — /distributed-system/distributed-system-intro.html
- 微服务面试题总结 — /distributed-system/microservices-interview-questions.html
- 分布式理论、算法与协议专题 — /distributed-system/protocol/
- 分布式协调详解 — /distributed-system/protocol/centralized-and-decentralized.html
- CAP 定理与 BASE 理论详解 — /distributed-system/protocol/cap-and-base-theorem.html
- 拜占庭将军问题详解 — /distributed-system/protocol/byzantine-generals-problem.html
- Raft 算法详解 — /distributed-system/protocol/raft-algorithm.html
- Paxos 算法详解 — /distributed-system/protocol/paxos-algorithm.html
- ZAB 协议详解 — /distributed-system/protocol/zab.html
- Gossip 协议详解 — /distributed-system/protocol/gossip-protocol.html
- 一致性哈希算法详解 — /distributed-system/protocol/consistent-hashing.html

### RPC 与服务调用
- RPC 专题 — /distributed-system/rpc/
- RPC 远程过程调用详解 — /distributed-system/rpc/rpc-intro.html
- Dubbo 面试题总结 — /distributed-system/rpc/dubbo.html
- 有了 HTTP 协议，为什么还要 RPC？ — /cs-basics/network/http-vs-rpc.html

### API 网关与流量入口
- API 网关详解 — /distributed-system/api-gateway.html
- Spring Cloud Gateway 面试题总结 — /distributed-system/spring-cloud-gateway-questions.html

### 分布式 ID、锁与事务
- 分布式 ID 生成方案详解 — /distributed-system/distributed-id.html
- 分布式 ID 设计实战 — /distributed-system/distributed-id-design.html
- 分布式锁入门 — /distributed-system/distributed-lock.html
- 分布式锁实现方案详解 — /distributed-system/distributed-lock-implementations.html
- 分布式事务解决方案详解 — /distributed-system/distributed-transaction.html

### 配置中心与 ZooKeeper
- 分布式配置中心详解 — /distributed-system/distributed-configuration-center.html
- ZooKeeper 专题 — /distributed-system/distributed-process-coordination/zookeeper/
- ZooKeeper 入门指南 — /distributed-system/distributed-process-coordination/zookeeper/zookeeper-intro.html
- ZooKeeper 进阶详解 — /distributed-system/distributed-process-coordination/zookeeper/zookeeper-plus.html
- ZooKeeper 实战教程 — /distributed-system/distributed-process-coordination/zookeeper/zookeeper-in-action.html

## 8. 高性能系统（/high-performance/）→ 域3/6

**学习主线**：请求链路分析（CDN → 负载均衡 → 应用 → 缓存 → 数据库 → MQ），瓶颈定位 → 方案选择 → 取舍分析 → 落地风险。

### 流量入口与请求分发
- CDN 工作原理详解 — /high-performance/cdn.html
- 负载均衡原理及算法详解 — /high-performance/load-balancing.html

### 数据库与数据访问优化
- 读写分离和分库分表详解 — /high-performance/read-and-write-separation-and-library-subtable.html
- 数据冷热分离详解 — /high-performance/data-cold-hot-separation.html
- 常见 SQL 优化手段总结 — /high-performance/sql-optimization.html
- 深度分页介绍及优化建议 — /high-performance/deep-pagination-optimization.html

### 消息队列与异步削峰
- 消息队列专题 — /high-performance/message-queue/
- 消息队列面试题总结 — /high-performance/message-queue/message-queue-interview-questions.html
- 消息队列基础知识总结 — /high-performance/message-queue/message-queue.html
- Kafka 常见问题总结 — /high-performance/message-queue/kafka-questions-01.html
- RocketMQ 常见问题总结 — /high-performance/message-queue/rocketmq-questions.html
- RabbitMQ 常见问题总结 — /high-performance/message-queue/rabbitmq-questions.html
- Disruptor 常见问题总结 — /high-performance/message-queue/disruptor-questions.html

## 9. 高可用系统（/high-availability/）→ 域3

**学习主线**：减少故障、隔离故障、快速恢复；SLA/多少个 9/RTO/RPO；限流、降级、熔断、超时重试、幂等、冗余容灾、压测。

### 总览与面试路线
- 高可用系统设计面试题总结 — /high-availability/high-availability-system-interview-questions.html
- 高可用系统设计详解 — /high-availability/high-availability-system-design.html

### 稳定性治理
- 服务限流详解 — /high-availability/limit-request.html
- 服务降级与熔断详解 — /high-availability/fallback-and-circuit-breaker.html
- 超时和重试机制详解 — /high-availability/timeout-and-retry.html

### 幂等、容灾与验证
- 接口幂等性设计详解 — /high-availability/idempotency.html
- 冗余设计详解 — /high-availability/redundancy.html
- 性能测试入门 — /high-availability/performance-test.html

## 10. 系统设计（/system-design/）→ 域1/2/7

**学习主线**：API 设计、代码质量、设计模式、框架原理、认证授权、数据安全、定时任务、实时消息推送。

### 系统设计与工程基础
- 系统设计常见面试题总结 — /system-design/system-design-questions.html
- 系统设计基础专题 — /system-design/basis/
- RestFul API 简明教程 — /system-design/basis/RESTfulAPI.html
- 代码重构指南 — /system-design/basis/refactoring.html
- 单元测试到底是什么？应该怎么做？ — /system-design/basis/unit-test.html
- 设计模式常见面试题总结 — /system-design/design-pattern.html

### 常用框架
- Spring & Spring Boot 专题 — /system-design/framework/spring/
- Spring常见面试题总结 — /system-design/framework/spring/spring-knowledge-and-questions-summary.html
- SpringBoot常见面试题总结 — /system-design/framework/spring/springboot-knowledge-and-questions-summary.html
- MyBatis常见面试题总结 — /system-design/framework/mybatis/mybatis-interview.html
- Netty常见面试题总结 — /system-design/framework/netty.html

### 认证授权与数据安全
- 认证授权与数据安全专题 — /system-design/security/
- 认证授权基础概念详解 — /system-design/security/basis-of-authority-certification.html
- JWT 基础概念详解 — /system-design/security/jwt-intro.html
- 权限系统设计详解 — /system-design/security/design-of-authority-system.html
- 常见加密算法总结 — /system-design/security/encryption-algorithms.html
- 数据脱敏方案总结 — /system-design/security/data-desensitization.html

### 任务调度与消息推送
- Java 定时任务详解 — /system-design/schedule-task.html
- Web 实时消息推送详解 — /system-design/web-real-time-message-push.html
- J2EE 基础知识 — /system-design/J2EE基础知识.html

## 11. 计算机基础（/cs-basics/）→ 域1

**学习主线**：计算机网络 → 操作系统 → 数据结构 → 算法；300+ 张技术配图。

### 计算机网络（专题 /cs-basics/network/）
- 计算机网络常见面试题总结（上） — /cs-basics/network/other-network-questions.html
- 计算机网络常见面试题总结（下） — /cs-basics/network/other-network-questions2.html
- OSI 七层模型与 TCP/IP 四层模型详解 — /cs-basics/network/osi-and-tcp-ip-model.html
- 从输入 URL 到页面展示到底发生了什么？ — /cs-basics/network/the-whole-process-of-accessing-web-pages.html
- HTTP vs HTTPS — /cs-basics/network/http-vs-https.html
- HTTP 1.0 vs HTTP 1.1 — /cs-basics/network/http1.0-vs-http1.1.html
- HTTP 常见状态码总结 — /cs-basics/network/http-status-codes.html
- TCP 三次握手和四次挥手 — /cs-basics/network/tcp-connection-and-disconnection.html
- TCP 传输可靠性保障 — /cs-basics/network/tcp-reliability-guarantee.html

### 操作系统（专题 /cs-basics/operating-system/）
- 操作系统常见面试题总结（上） — /cs-basics/operating-system/operating-system-basic-questions-01.html
- 操作系统常见面试题总结（下） — /cs-basics/operating-system/operating-system-basic-questions-02.html
- 进程与线程详解 — /cs-basics/operating-system/process-and-thread.html
- 进程间通信（IPC）详解 — /cs-basics/operating-system/ipc.html
- 操作系统锁与同步机制详解 — /cs-basics/operating-system/os-lock-and-sync.html
- 操作系统内存管理详解 — /cs-basics/operating-system/memory-management.html
- 虚拟内存详解 — /cs-basics/operating-system/virtual-memory.html
- 操作系统文件系统详解 — /cs-basics/operating-system/file-system.html
- I/O 多路复用详解 — /cs-basics/operating-system/io-multiplexing.html
- 零拷贝详解 — /cs-basics/operating-system/zero-copy.html
- Linux 基础知识总结 — /cs-basics/operating-system/linux-intro.html
- Shell 编程基础知识总结 — /cs-basics/operating-system/shell-intro.html

### 数据结构（专题 /cs-basics/data-structure/）
- 线性数据结构详解 — /cs-basics/data-structure/linear-data-structure.html
- 哈希表面试题总结 — /cs-basics/data-structure/hash-table.html
- 树结构详解 — /cs-basics/data-structure/tree.html
- 图详解 — /cs-basics/data-structure/graph.html
- 堆详解 — /cs-basics/data-structure/heap.html
- 红黑树详解 — /cs-basics/data-structure/red-black-tree.html
- 布隆过滤器详解 — /cs-basics/data-structure/bloom-filter.html
- Trie 前缀树面试题总结 — /cs-basics/data-structure/trie.html
- 并查集面试题总结 — /cs-basics/data-structure/union-find.html
- 跳表面试题总结 — /cs-basics/data-structure/skip-list.html
- LRU 缓存面试题总结 — /cs-basics/data-structure/lru-cache.html

### 算法（专题 /cs-basics/algorithms/）
- 时间复杂度和空间复杂度面试指南 — /cs-basics/algorithms/complexity-analysis.html
- 二分查找面试题总结 — /cs-basics/algorithms/binary-search.html
- 双指针与滑动窗口面试题总结 — /cs-basics/algorithms/two-pointers-and-sliding-window.html
- DFS 与 BFS 面试题总结 — /cs-basics/algorithms/dfs-bfs.html
- 回溯算法面试题总结 — /cs-basics/algorithms/backtracking.html
- 动态规划面试题总结 — /cs-basics/algorithms/dynamic-programming.html
- 贪心算法面试题总结 — /cs-basics/algorithms/greedy.html
- Top K 问题面试题总结 — /cs-basics/algorithms/top-k.html
- 经典算法思想总结 — /cs-basics/algorithms/classical-algorithm-problems-recommendations.html
- 常见数据结构经典 LeetCode 题目推荐 — /cs-basics/algorithms/common-data-structures-leetcode-recommendations.html
- 几道常见的字符串算法题 — /cs-basics/algorithms/string-algorithm-problems.html
- 几道常见的链表算法题 — /cs-basics/algorithms/linkedlist-algorithm-problems.html
- 剑指 Offer 部分编程题 — /cs-basics/algorithms/the-sword-refers-to-offer.html
- 十大经典排序算法总结 — /cs-basics/algorithms/10-classical-sorting-algorithms.html

## 12. AI 应用开发（/ai/）→ 域6

**学习主线**：大模型基础、Agent、RAG、系统设计连起来理解；开源项目 AIGuide。

### 面试与复习路线
- Java/Go 开发者 AI 应用开发与 Agent 学习路线（2026） — /roadmap/java-to-ai-roadmap.html
- 后端开发者转型 AI Agent 学习建议（2026） — /roadmap/backend-to-ai-agent-roadmap.html
- AI 核心概念总览 — /ai/ai-core-concepts.html
- AI 应用开发面试题专题 — /ai/interview-questions/
- AI 应用开发面试指南 — /ai/interview-questions/ai-interview-guide.html
- 大模型基础面试题总结 — /ai/interview-questions/llm-interview-questions.html
- AI Agent 面试题总结 — /ai/interview-questions/agent-interview-questions.html
- Agent 项目面试怎么讲？ — /ai/interview-questions/agent-project-interview-guide.html
- RAG 面试题总结 — /ai/interview-questions/rag-interview-questions.html
- AI 系统设计面试题总结 — /ai/interview-questions/ai-system-design-interview-questions.html

### 大模型基础（/ai/llm-basis/）
- LLM 运行机制：Token、上下文窗口与采样参数 — /ai/llm-basis/llm-operation-mechanism.html
- 大模型 API 调用工程实践 — /ai/llm-basis/llm-api-engineering.html
- 大模型结构化输出详解 — /ai/llm-basis/structured-output-function-calling.html
- AI 应用评测体系 — /ai/llm-basis/llm-evaluation.html

### AI Agent（/ai/agent/）
- AI Agent 核心概念 — /ai/agent/agent-basis.html
- AI Agent 记忆系统 — /ai/agent/agent-memory.html
- 多 Agent 协作系统设计 — /ai/agent/multi-agent.html
- 大模型提示词工程 — /ai/agent/prompt-engineering.html
- 上下文工程 — /ai/agent/context-engineering.html
- 万字拆解 MCP 协议 — /ai/agent/mcp.html
- 万字详解 Agent Skills — /ai/agent/skills.html
- Harness Engineering：六层检查框架 — /ai/agent/harness-engineering.html
- AI 工作流中的 Workflow、Graph 与 Loop — /ai/agent/workflow-graph-loop.html
- Loop Engineering 是什么？ — /ai/agent/loop-engineering.html

### RAG 检索增强生成（/ai/rag/）
- RAG 基础概念 — /ai/rag/rag-basis.html
- RAG 文档处理与切分策略 — /ai/rag/rag-document-processing.html
- RAG 向量索引算法和向量数据库 — /ai/rag/rag-vector-store.html
- RAG 检索优化 — /ai/rag/rag-optimization.html
- GraphRAG — /ai/rag/graphrag.html
- RAG 知识库文档更新策略 — /ai/rag/rag-knowledge-update.html

### AI 系统设计（/ai/system-design/）
- AI 应用系统设计 — /ai/system-design/ai-application-architecture.html
- LLM/Agent 安全实战 — /ai/system-design/llm-security.html
- 大模型网关详解 — /ai/system-design/llm-gateway.html
- AI 语音技术详解 — /ai/system-design/ai-voice.html

## 13. AI 编程（/ai-coding/）→ 域6

**学习主线**：Claude Code、Cursor、Codex、Trae 实战与原理；上下文管理、CLAUDE.md、Skills、Hooks、多 Agent。

### 工具选型与方法论（/ai-coding/practices/）
- AI 编程开放性面试题 — /ai-coding/practices/ai-ide.html
- AI 编程选 CLI 还是 IDE？ — /ai-coding/practices/cli-vs-ide.html
- Ghostty 安装、配置和常见技巧 — /ai-coding/practices/ghostty.html
- Spec Coding 规范驱动编程 — /ai-coding/practices/spec-coding.html
- Vibe Coding 实用技巧总结 — /ai-coding/practices/the-cool-tricks-for-vibe-coding.html

### Claude Code 与 Codex 实战
- Claude Code 使用指南 — /ai-coding/practices/claudecode-tips.html
- CLAUDE.md 最佳实践 — /ai-coding/practices/claude-md-best-practices.html
- Claude Code 核心命令详解 — /ai-coding/practices/claudecode-commands.html
- AI 编程必备 Skills 推荐 — /ai-coding/practices/programmer-essential-skills.html
- 强模型时代，AI 编程 Skills 还有必要装吗？ — /ai-coding/practices/skill-selection-and-pruning.html
- mattpocock/skills 深度使用指南 — /ai-coding/practices/mattpocock-skills.html
- drawio-chart Skill 开源 — /ai-coding/practices/drawio-chart-skill.html
- OpenAI Codex 最佳实践指南 — /ai-coding/practices/codex-best-practices.html
- 高颜值 Claude Code 替代 OMP — /ai-coding/practices/oh-my-pi.html
- Claude Code Agent View 多会话管理 — /ai-coding/practices/claudecode-agentview.html

### Claude Code 原理进阶（/ai-coding/principles/）
- Claude Code 上下文管理 — /ai-coding/principles/claude-code-context-management.html
- Claude Code 记忆系统 — /ai-coding/principles/claude-code-memory.html
- Claude Code Skills 原理 — /ai-coding/principles/claude-code-skills.html
- Claude Code Hooks 原理 — /ai-coding/principles/claude-code-hooks.html
- Claude Code 多 Agent 机制 — /ai-coding/principles/claude-code-multi-agent.html

### 真实项目案例（/ai-coding/cases/）
- IDEA 搭配 Qoder 插件实战 — /ai-coding/cases/idea-qoder-plugin.html
- Trae + MiniMax 多场景实战 — /ai-coding/cases/trae-m2.7.html
- Claude Code 接入第三方模型实战 — /ai-coding/cases/cc-glm5.1.html
- DeepSeek V4 + Claude Code 实战 — /ai-coding/cases/deepseek-v4-claude-code.html
- MiniMax M3 + Claude Code 实战 — /ai-coding/cases/cc-m3.html
- Kimi K3 多场景实战 — /ai-coding/cases/kimi-k3.html
- IDEA + CC GUI 插件实战 — /ai-coding/project/cc-guide.html

---

## 去重与互补策略

| 重叠主题 | JavaGuide 定位 | 其他站点处理 |
|----------|----------------|--------------|
| MySQL/Redis 原理 | 面试导向 + 工程实践 | xiaolincoding 图解系列做可视化补充，不重复全文收录 |
| Java 并发/JVM | 体系最全，作主干 | javaup 八股文只收录差异化解法（如线程池参数定制） |
| AI 应用开发 | 已有完整体系 | xiaolinnote 只收录面试题版本差异，javaup 收录 AI 大模型八股差异 |
| 计算机网络/OS | cs-basics 有覆盖 | xiaolincoding 图解系列作主来源（图更多），JavaGuide 作补充 |

## 抓取优先级（内容沉淀顺序）

1. **P0**：Java 基础面试题(上/中/下)、Java 集合面试题、Java 并发面试题、JVM 面试题、MySQL 面试题、Redis 面试题、分布式面试题、微服务面试题（面试主干，8 篇）
2. **P1**：MySQL 索引/事务/MVCC/日志、Redis 数据结构/持久化/集群、线程池、JMM/AQS、Spring/SpringBoot/MyBatis 面试题（核心原理，15 篇）
3. **P2**：分布式锁/事务/ID、限流熔断降级、幂等、消息队列四件套、系统设计面试题（架构落地，12 篇）
4. **P3**：计算机基础（网络/OS/数据结构/算法）、AI 应用开发、AI 编程（按需补充）
