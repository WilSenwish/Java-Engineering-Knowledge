# 二次加工层（Curated）· 知识域文档索引

> 本层基于 `sources/` 原始收集层，按《Java 后端开发 & 架构・复杂生产环境工程化落地全链路知识库构建规划.md》的**五段式标准**二次加工而成。
> 收集层保留原站结构与原文（可溯源）；本层做去重、提炼、工程化增强，形成统一知识体系。

## 一、五段式文档模板

每个知识模块统一采用五段式结构，确保工程化落地属性：

| 段 | 名称 | 写作要求 |
|----|------|----------|
| 1 | **核心原理** | 精简讲清本质，避免冗余；能用图和一句话结论优先 |
| 2 | **生产落地** | 配置示例、实现步骤、集成方式；代码遵循项目规范（Lombok、ApiResponse\<T\>、异常处理模式） |
| 3 | **踩坑规避** | 生产环境常见问题、根因、解决方案；每条给出现象→根因→对策 |
| 4 | **优化进阶** | 性能优化、高可用加固方案；给出量化收益或适用边界 |
| 5 | **选型对比** | 同类技术/方案的适用场景、优劣势分析；对比表呈现 |

## 二、知识域文档地图

| 知识域 | 文档 | 加工来源（sources/） | 状态 |
|--------|------|----------------------|------|
| 域1 Java核心与编码工程化 | [01.1 Java 基础与语法工程化](01-java-core/01.1-java-basics.md) | javaguide/java-basis | ✅ |
| 域1 | [01.2 Java 集合工程化（含 HashMap/ConcurrentHashMap 深度）](01-java-core/01.2-collections.md) | javaguide/java-collection + javaup/hashmap + javaup/concurrenthashmap | ✅ |
| 域1 | [01.3 Java 并发编程工程化](01-java-core/01.3-concurrency.md) | javaguide/java-concurrent 上+中+下 | ✅ |
| 域1 | [01.4 JVM 内核与线上排查](01-java-core/01.4-jvm.md) | javaguide/java-jvm | ✅ |
| 域1（计算机基础） | [01.5 计算机网络 · TCP/HTTP 深度](01-java-core/01.5-network-tcp.md) | xiaolincoding/tcp + http | ✅ |
| 域2 后端技术组件工程化 | [02.1 MySQL 工程化](02-backend-components/02.1-mysql.md) | javaguide/mysql + 小林index/mvcc/索引失效/锁/日志/BufferPool | ✅ |
| 域2 | [02.2 Redis 工程化](02-backend-components/02.2-redis.md) | javaguide/redis + 小林AOF/RDB/缓存三难题/主从/一致性 | ✅ |
| 域2 | [02.3 Spring & MyBatis 工程化](02-backend-components/02.3-spring-mybatis.md) | javaguide/system-design（Spring+MyBatis） | ✅ |
| 域3 分布式与高可用架构 | [03.1 分布式理论与核心方案](03-distributed/03.1-distributed-theory.md) | javaguide/distributed-system | ✅ |
| 域3 | [03.2 微服务架构工程化](03-distributed/03.2-microservices.md) | javaguide/microservices | ✅ |
| 域6 性能优化与架构演进 | [06.1 AI 前沿：Agent/RAG/LLM/AI编程](06-performance-ai/06.1-ai-frontier.md) | xiaolinnote 89题 + 图解Agent/RAG万字图解 | ✅ |
| 域7 业务场景与实战案例 | [07.1 实战项目案例库](07-business-cases/07.1-projects.md) | nageoffer 5项目 | ✅ |
| 域4 工程交付与云原生 | — | 待收集（规划：CI/CD、Docker、K8s） | ⏳ |
| 域5 生产运维与可观测性 | — | 待收集（规划：ELK、Prometheus、SkyWalking） | ⏳ |

## 三、与收集层的协作规则

1. **单向引用**：本层文档头部注明加工来源；sources/ 原文不改动；
2. **增量补充**：新抓取内容先落 sources/，再增量并入本层对应文档（不推倒重写）；
3. **交叉引用**：文档间用相对路径互链（如集合篇 → 并发篇的 ConcurrentHashMap 章节）；
4. **五段式缺项处理**：来源未覆盖的段落，标注「待补充」并列入抓取计划，不用臆造填充。

## 四、当前加工进度

- **循环1（已完成）**：sources/ 存量 10 篇内容 + 5 站点目录 → 本层 11 篇知识域文档；
- **循环2（已完成）**：新增抓取 4 篇（HTTP 面试题 / 索引面试题 / 并发面试题中 / ConcurrentHashMap 线程安全）→ 五段式增量并入：
  - 01.2 新增**模块五：ConcurrentHashMap 深度**（分段锁→节点锁、CAS+synchronized、协同扩容）
  - 01.3 新增**模块四：JMM/volatile/锁机制**（内存屏障、DCL、CAS 三问题、锁升级废弃史）
  - 01.5 新增**模块三：HTTP/HTTPS/演进**（缓存两级、HTTP2/3、TLS 握手、证书链）
  - 02.1 新增**模块五：索引深度补充**（范围查询匹配规则、key_len、前缀索引、NOT NULL）
- **循环3（已完成）**：新增抓取 6 篇（并发面试题下 / MVCC / AOF 持久化 / 缓存三难题 / Spring / MyBatis）→ 五段式增量并入：
  - 01.3 新增**模块五：线程池工程化**（七参数/执行流程/拒绝策略/动态调参）、**模块六：ThreadLocal 与上下文传递**（泄漏机理/TTL CRR）、**模块七：AQS 与 JUC 工具**
  - 02.1 新增**模块六：事务隔离与 MVCC 深度**（ReadView 四字段/可见性规则/RC vs RR 时机/源码对应）
  - 02.2 新增**模块五：AOF 持久化深度**（三策略/重写/COW/MP-AOF）、**模块六：缓存异常三难题**（雪崩/击穿/穿透 + 布隆过滤器）
  - **新建 02.3 Spring & MyBatis 工程化**（IoC/AOP/事务/MVC/MyBatis 五模块）
- **循环4（已完成）**：新增抓取 6 篇（索引失效/MySQL锁/三大日志/RDB/Redis数据结构/JVM垃圾回收；javaguide SpringBoot 面试题为付费页无正文，已换源）→ 五段式增量并入：
  - 01.4 新增**模块五：垃圾回收深度**（可达性分析/分代算法/收集器对比/G1 与 ZGC 选型决策）
  - 02.1 新增**模块七：索引失效案例集与 SQL 编写规约**（六大失效场景速查表）、**模块八：锁体系与死锁治理**（三级锁/MDL 事故链/AUTO-INC 与主从一致）、**模块九：三大日志与两阶段提交**（WAL/双1配置/组提交/2PC 判定）
  - 02.2 模块一补充底层结构版本演进；新增**模块七：RDB 快照与混合持久化**（COW/混合持久化/生产推荐组合）
- **循环5（已完成）**：新增抓取 6 篇（BufferPool / 主从复制 / 缓存一致性 / Ragent V2 / 图解Agent目录 / RAG 万字图解）→ 五段式增量并入：
  - 02.1 新增**模块十：Buffer Pool 深度**（三链表/young-old 分区/时间门槛/刷盘抖动）
  - 02.2 新增**模块八：主从复制与高可用**（全量三阶段/增量复制/双 Buffer/脑裂防护）、**模块九：数据库与缓存一致性工程方案**（四种顺序分析/Canal+MQ/选型矩阵）
  - 06.1 新增**模块五：RAG 全链路原理深度**（切割五级/两阶段 Re-rank/多路召回 RRF/Embedding 四代/向量库选型/幻觉四层防线/RAGAS 量化）
  - 07.1 新增**Ragent V2 架构深挖**（7 模块分层/v1-v2 演进/生产级特性/设计模式实战/扩展点）
- **循环6 计划**：小林哨兵/Cluster/RedLock/大Key热Key；javaup 框架板块；javaguide Redis 面试题(下)/设计模式专题。
