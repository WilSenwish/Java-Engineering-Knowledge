# Java 后端开发 & 架构 知识库 · 总索引

> 依据《Java 后端开发 & 架构・复杂生产环境工程化落地全链路知识库构建规划.md》执行。
> 本文件是知识库总入口：定义内容来源、处理顺序、目录规范与沉淀标准。

## 一、内容来源与处理顺序

| 批次 | 站点 | 定位 | 处理顺序理由 | 对应知识域 |
|------|------|------|--------------|------------|
| 第一批 | [javaguide.cn](https://javaguide.cn/) | Java 全栈知识 + 面试体系（GitHub 156K+ Star） | **知识库主干**：Java 核心、数据库、分布式、高并发/高可用、系统设计体系最完整，作为主干骨架优先落地 | 域1/2/3 |
| 第一批 | [javaup.chat](https://javaup.chat/) | 码力全开：超级八股文（深度原理+代码+流程图）+ 生产级项目实战 | 与 JavaGuide 互补去重，补深度原理与高并发项目实战 | 域1/2/3/7 |
| 第二批 | [xiaolincoding.com](https://xiaolincoding.com/) | 小林coding：图解网络/系统/MySQL/Redis + 后端面试八股 | 计算机基础原理可视化补充，补网络/OS 短板 | 域1/2/3 |
| 第三批 | [nageoffer.com](https://nageoffer.com/) | 拿个offer：Ragent / oneThread / oneCoupon / 12306 / Link 五大项目实战 | 业务场景与实战案例库核心来源 | 域7 |
| 第四批 | [xiaolinnote.com](https://xiaolinnote.com/) | 小林面试笔记：Agent/RAG/LLM/AI编程面试题 + 图解Agent/Claude Code/Codex | AI 前沿内容，作为进阶提升知识域的前沿模块 | 域6 |

**处理原则**：先主干（javaguide/javaup）→ 后基础图解（xiaolincoding）→ 再实战案例（nageoffer）→ 最后前沿（xiaolinnote）。与规划文档"阶段一优先填充 Java核心 / Spring / MySQL / Redis 三大基础模块"一致。

## 二、目录结构

```
Java-Engineering-Knowledge/
├── README.md                              # 本文件：总索引
├── Java 后端开发 & 架构・...构建规划.md     # 原始规划文档
└── sources/                               # 原始收集层（按站点归档，保留原站结构）
    ├── javaguide/                         # 站点1：JavaGuide（主干）
    │   ├── _index.md                      # 站点完整目录
    │   ├── java-basis/                    # Java 基础
    │   ├── java-collection/               # Java 集合
    │   ├── java-concurrent/               # Java 并发
    │   ├── java-jvm/                      # JVM
    │   ├── database-mysql/                # MySQL
    │   ├── database-redis/                # Redis
    │   ├── distributed-system/            # 分布式
    │   ├── high-performance/              # 高性能
    │   ├── high-availability/             # 高可用
    │   ├── system-design/                 # 系统设计
    │   ├── cs-basics/                     # 计算机基础
    │   ├── ai/                            # AI 应用开发
    │   └── ai-coding/                     # AI 编程
    ├── javaup/                            # 站点2：码力全开
    ├── xiaolincoding/                     # 站点3：小林coding
    ├── nageoffer/                         # 站点4：拿个offer
    └── xiaolinnote/                       # 站点5：小林面试笔记
```

> 规划阶段只建"原始收集层"（sources/）。各知识域的 curated 文档（docs/01~07）在收集完成后按五段式标准二次加工，避免边收边改导致结构反复。

## 三、内容沉淀标准（五段式）

每个知识点统一采用五段式结构，确保工程化落地属性：

1. **核心原理**：精简讲清本质，避免冗余
2. **生产落地**：配置示例、实现步骤、集成方式
3. **踩坑规避**：生产环境常见问题、根因、解决方案
4. **优化进阶**：性能优化、高可用加固方案
5. **选型对比**：同类技术的适用场景、优劣势分析

收集层（sources/）的 md 文件统一带头部元信息：`来源站点 / 原文链接 / 抓取日期 / 所属知识域`，保证可溯源。

## 四、进度看板

| 站点 | 目录抓取 | 内容沉淀 | 状态 |
|------|----------|----------|------|
| javaguide.cn | ✅ 13/13 板块 | ✅ 17 篇（P0 8 + 并发中/下 + Spring + MyBatis + JVM垃圾回收 + Redis数据结构 + OS面试题上 + 网络面试题上 + 线上排查 + JDK工具）+ 五段式加工（01.1~01.6、02.1~02.3、03.1~03.2、05.1） | ✅ 循环1~9 完成 |
| javaup.chat | ✅ 6 大板块 ~250 篇 | ✅ 2 篇（HashMap 深度 + ConcurrentHashMap 线程安全）+ 五段式加工（01.2/01.3） | 进行中（hub 页 403，换入口待办） |
| xiaolincoding.com | ✅ 图解网络/系统/MySQL/Redis + 24套面试题 | ✅ 21 篇（+TCP可靠性/虚拟内存/过期淘汰）+ 五段式加工（01.5/01.6/02.1/02.2） | ✅ 循环1~9 完成 |
| nageoffer.com | ✅ 5 个项目卡片 | ✅ Ragent V2 + oneThread 项目介绍 + 五项目五段式加工（07.1） | ✅ 循环1~9 完成 |
| xiaolinnote.com | ✅ Agent/RAG/LLM/AI编程 89 题 | ✅ 89 题 + 图解Agent/RAG万字图解 + ReAct/Multi-Agent + ClaudeCode/Codex 目录，五段式加工（06.1/04.1） | ✅ 循环1~9 完成 |

### 已完成文件清单（sources/）

```
javaguide/
├── _index.md                                          # 站点完整目录（13 板块 200+ 篇文章）
├── java-basis/java-basic-questions-01.md              # Java基础面试题(上)
├── java-collection/java-collection-questions-01.md    # Java集合面试题(上)
├── java-concurrent/java-concurrent-questions-01.md    # Java并发面试题(上)
├── java-concurrent/java-concurrent-questions-02.md    # Java并发面试题(中)：JMM/volatile/锁/CAS
├── java-concurrent/java-concurrent-questions-03.md    # Java并发面试题(下)：ThreadLocal/线程池/Future/AQS
├── java-jvm/jvm-interview-questions.md                # JVM面试题总结
├── java-jvm/jvm-garbage-collection.md                 # JVM垃圾回收详解（算法/收集器/GC过程）
├── database-mysql/mysql-questions-01.md               # MySQL面试题总结
├── database-redis/redis-questions-01.md               # Redis面试题(上)
├── database-redis/redis-questions-02.md               # Redis面试题(下)：事务/性能优化/生产问题/规范
├── database-redis/redis-data-structures-01.md         # Redis 5种基本数据类型详解
├── distributed-system/distributed-system-interview-questions.md  # 分布式面试题总结
├── distributed-system/microservices-interview-questions.md       # 微服务面试题总结
├── system-design/spring-questions.md                  # Spring常见面试题（IoC/AOP/事务/MVC）
└── system-design/mybatis-questions.md                 # MyBatis常见面试题（执行流/缓存/插件）

javaup/
├── _index.md                                          # 站点完整目录（6 板块 ~250 篇 + 7 实战项目）
├── java-collection/hashmap-core-principles.md         # HashMap核心原理与数据结构
└── java-collection/concurrenthashmap-thread-safety.md # ConcurrentHashMap线程安全机制

xiaolincoding/
├── _index.md                                          # 站点完整目录（图解网络/系统/MySQL/Redis + 24套面试题）
├── network/tcp-interview.md                           # TCP三次握手与四次挥手面试题
├── network/http-interview.md                          # HTTP常见面试题
├── mysql/index-interview.md                           # 索引常见面试题
├── mysql/mvcc-transaction-isolation.md                # 事务隔离级别是怎么实现的（MVCC）
├── mysql/index-invalid.md                             # 索引失效有哪些（六大场景）
├── mysql/mysql-locks.md                               # MySQL有哪些锁（三级锁体系/死锁）
├── mysql/three-logs.md                                # 三大日志与两阶段提交
├── mysql/buffer-pool.md                               # 揭开BufferPool的面纱（三链表/young-old分区）
├── redis/aof-persistence.md                           # AOF持久化是怎么实现的
├── redis/rdb-snapshot.md                              # RDB快照是怎么实现的（COW/混合持久化）
├── redis/cache-problems.md                            # 缓存雪崩、击穿、穿透
├── redis/master-slave-replication.md                  # 主从复制是怎么实现的（全量/增量/脑裂防护）
├── redis/mysql-redis-consistency.md                   # 数据库和缓存如何保证一致性
├── redis/sentinel.md                                  # 为什么要有哨兵（主客观下线/选主/故障转移四步）
├── redis/redlock-distributed-lock.md                  # Redis分布式锁高可用高性能（七层加固/Redlock争议）
├── redis/bigkey-impact.md                             # 大Key对持久化的影响（fork两阶段阻塞/内存大页）
├── redis/cluster.md                                   # 为什么要有cluster集群（哈希槽/MOVED-ASK/Gossip/故障转移）
├── redis/expire-eviction.md                           # 过期删除策略和内存淘汰策略区别（八策略/LRU vs LFU）
├── mysql/sql-execution-flow.md                        # 执行一条select语句期间发生了什么（Server层全链路）
├── network/tcp-reliability.md                         # TCP重传/滑动窗口/流量控制/拥塞控制（全参数）
└── os/io-multiplexing.md                              # I/O多路复用select/poll/epoll（红黑树/就绪链表/LT-ET）
└── os/virtual-memory.md                               # 为什么要有虚拟内存（分段分页/多级页表/TLB/Linux布局）

nageoffer/
├── _index.md                                          # 5 个项目完整卡片（Ragent/oneThread/oneCoupon/12306/SaaS短链）
├── ragent-intro.md                                    # Ragent V2 项目介绍（7模块/生产级特性/设计模式）
└── onethread.md                                       # oneThread动态线程池（配置中心热更/拒绝代理/监控告警/项目结构）

xiaolinnote/
├── _index.md                                          # Agent/RAG/LLM/AI编程 89 题完整题目清单
├── agent-illustrated-index.md                         # 图解Agent专栏目录（三方向8篇）
├── rag-principles.md                                  # RAG检索增强万字图解（全链路/切割/Re-rank/多路召回/幻觉）
├── agent/react-reasoning.md                           # Agent面试题5：推理模式/ReAct/Plan-and-Execute
├── agent/multi-agent.md                               # Agent面试题10：Multi-Agent（context硬限制/三模式）
├── claudecode/index.md                                # 图解Claude Code专栏目录（五方向/源码解析/工程化六件套）
└── codex/index.md                                     # 图解Codex专栏目录（AGENTS.md/Skills/Harness架构）

javaguide/（cs-basics + java-jvm 子目录）
├── cs-basics/os-questions-01.md                       # 操作系统常见面试题(上)（内核态/进程线程/调度/死锁）
├── cs-basics/network-questions-01.md                  # 计算机网络常见面试题(上)（体系结构/协议/缓存/RPC）
├── java-jvm/jvm-troubleshooting.md                    # Java后端线上问题排查（CPU/内存/GC/线程池/DB/Redis/MQ全场景SOP）
└── java-jvm/jdk-monitoring-tools.md                   # JDK监控和故障处理工具总结（jps/jstat/jmap/jstack/jcmd/JFR/MAT）
```

### 二次加工层（docs/）—— 五段式加工成果

```
docs/
├── README.md                                  # 二次加工层索引 + 五段式模板 + 循环进度
├── 01-java-core/                              # 域1 Java核心与编码工程化
│   ├── 01.1-java-basics.md                    # Java基础与语法工程化（3模块×五段式）
│   ├── 01.2-collections.md                    # 集合工程化（5模块：框架/List/HashMap/Queue/ConcurrentHashMap）
│   ├── 01.3-concurrency.md                    # 并发编程（7模块：线程基础/协作/死锁/JMM与锁/线程池/ThreadLocal/AQS）
│   ├── 01.4-jvm.md                            # JVM内核与线上排查（5模块，含垃圾回收深度/收集器选型）
│   ├── 01.5-network-tcp.md                    # 计算机网络（5模块：TCP/可靠性全参数/TCP与UDP/HTTP·HTTPS/I/O多路复用/体系结构）
│   └── 01.6-os-fundamentals.md                # 操作系统基础（3模块：内核态与系统调用/进程线程模型/内存管理虚拟内存）
├── 02-backend-components/                     # 域2 后端技术组件工程化
│   ├── 02.1-mysql.md                          # MySQL工程化（11模块：…/锁体系/三大日志/BufferPool/SQL执行全链路）
│   ├── 02.2-redis.md                          # Redis工程化（14模块：…/哨兵/分布式锁/大Key治理/Redis Cluster/过期淘汰）
│   └── 02.3-spring-mybatis.md                 # Spring & MyBatis 工程化（5模块：IoC/AOP/事务/MVC/MyBatis）
├── 03-distributed/                            # 域3 分布式与高可用架构
│   ├── 03.1-distributed-theory.md             # 分布式理论与核心方案（CAP/BASE、RPC、ID锁事务）
│   └── 03.2-microservices.md                  # 微服务架构工程化（拆分/治理/一致性/稳定性）
├── 06-performance-ai/
│   └── 06.1-ai-frontier.md                    # AI前沿（6模块：Agent/RAG题库/工具调用/AI编程/RAG全链路/推理与协作模式）
├── 07-business-cases/
│   └── 07.1-projects.md                       # 实战项目案例库（Ragent V2深挖 + oneThread + 高并发三案例）
├── 04-delivery-cloudnative/
│   └── 04.1-ai-native-delivery.md             # AI原生研发与Coding Agent工程化（六件设施/源码级设计/Starter交付范式）
└── 05-ops-observability/
    └── 05.1-troubleshooting.md                # 生产运维与线上排查实战（现象速查表/止血留证/全场景SOP/工具矩阵）
```

### 状态：循环 1~9 全部完成 ✅

- **sources/** 收集层：46 个 md（41 篇内容 + 5 站点目录），全部带溯源元信息；
- **docs/** 加工层：14 个 md（13 篇知识域文档 + 索引），七大知识域全部有首发文档，统一五段式；
- **远程仓库**：https://github.com/WilSenwish/Java-Engineering-Knowledge （main 分支，本地提交 9 个循环提交；网络抖动时仅保本地）；
- **后续扩充方向**（非本轮范围）：传统云原生（Docker/K8s/CI-CD）、可观测性三支柱深度（Prometheus/OTel/ELK）、javaguide 分布式/微服务 P1 篇目、设计模式专题（换源受阻待解）。
