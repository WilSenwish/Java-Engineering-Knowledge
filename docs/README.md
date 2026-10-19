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
| 域1 | [01.3 Java 并发编程工程化](01-java-core/01.3-concurrency.md) | javaguide/java-concurrent 上+中+下 + javaup/synchronized锁升级 | ✅ |
| 域1 | [01.4 JVM 内核与线上排查](01-java-core/01.4-jvm.md) | javaguide/java-jvm | ✅ |
| 域1（计算机基础） | [01.5 计算机网络 · TCP/HTTP/I/O 多路复用/体系结构](01-java-core/01.5-network-tcp.md) | xiaolincoding/tcp + http + IO多路复用 + tcp可靠性 + 网络体系 | ✅ |
| 域1（计算机基础） | [01.6 操作系统基础与进程线程模型](01-java-core/01.6-os-fundamentals.md) | javaguide/os-questions-01 + 小林虚拟内存 | ✅ |
| 域1 | [01.7 设计模式与编码工程化](01-java-core/01.7-design-patterns.md) | javaup/design-pattern（总览原则 + 单例深度） | ✅（循环11） |
| 域2 后端技术组件工程化 | [02.1 MySQL 工程化](02-backend-components/02.1-mysql.md) | javaguide/mysql + 小林index/mvcc/索引失效/锁/日志/BufferPool/SQL执行链路 | ✅ |
| 域2 | [02.2 Redis 工程化](02-backend-components/02.2-redis.md) | javaguide/redis 上+下 + 小林AOF/RDB/缓存三难题/主从/一致性/哨兵/Redlock/大Key/Cluster/过期淘汰 | ✅ |
| 域2 | [02.3 Spring & MyBatis 工程化](02-backend-components/02.3-spring-mybatis.md) | javaguide/Spring+MyBatis + javaup/SpringBoot自动配置 | ✅ |
| 域3 分布式与高可用架构 | [03.1 分布式理论与核心方案](03-distributed/03.1-distributed-theory.md) | javaguide/distributed-system | ✅ |
| 域3 | [03.2 微服务架构工程化](03-distributed/03.2-microservices.md) | javaguide/microservices | ✅ |
| 域3 | [03.3 高并发方案设计精选](03-distributed/03.3-solution-design.md) | javaup/solution-design（秒杀库存 + 雪崩防护） | ✅（循环11） |
| 域6 性能优化与架构演进 | [06.1 AI 前沿：Agent/RAG/LLM/AI编程](06-performance-ai/06.1-ai-frontier.md) | xiaolinnote 89题（Agent 30题 + RAG 23题全解 + 工具调用16题）+ 图解Agent/RAG万字图解 + ReAct/Multi-Agent + ClaudeCode/Codex | ✅（循环21 推进中） |
| 域7 业务场景与实战案例 | [07.1 实战项目案例库](07-business-cases/07.1-projects.md) | nageoffer 5项目（Ragent/oneThread/oneCoupon/12306/shortlink + 12306技术文档3篇） | ✅（循环12 深挖） |
| 域4 工程交付与云原生 | [04.1 AI 原生研发与 Coding Agent 工程化](04-delivery-cloudnative/04.1-ai-native-delivery.md) | xiaolinnote 图解ClaudeCode/Codex + oneThread | ✅（首发） |
| 域5 生产运维与可观测性 | [05.1 生产运维与线上排查实战](05-ops-observability/05.1-troubleshooting.md) | javaguide 线上问题排查 + JDK工具总结 | ✅（首发） |
| 域5 | [05.2 可观测性三支柱深度](05-ops-observability/05.2-observability.md) | javaup 可观测性体系/SkyWalking 源码1-3/分布式日志/系统性能监控 | ✅（循环10） |

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
- **循环6（已完成）**：新增抓取 4 篇（哨兵 / Redlock 分布式锁 / 大 Key 对持久化影响 / Redis 面试题下；javaguide 设计模式篇为 PDF 推广页、javaup 设计模式 hub 页 403，已记录换源）→ 五段式增量并入：
  - 02.2 新增**模块十：哨兵与故障转移**（主客观下线/Leader 选举双条件/转移四步/quorum 规则）、**模块十一：分布式锁工业级方案**（七层加固/Redlock 与 fencing token 之争/Singleflight 与乐观锁替代）、**模块十二：大 Key 与热 Key 治理**（fork 两阶段阻塞/内存大页陷阱/UNLINK/发现手段矩阵）
  - 02.2 模块一补充批量操作三选型、Redis 阻塞原因清单
- **循环7（已完成）**：新增抓取 4 篇（Redis Cluster / OS 面试题上 / I/O 多路复用 / SQL 执行全链路；设计模式换源仍受阻：javaguide 为 PDF 推广页、javaup hub 页 403）→ 五段式增量并入：
  - 02.2 新增**模块十三：Redis Cluster 切片集群**（哈希槽 16384 推导/Smart Client/MOVED vs ASK/Gossip/pfail-fail 两阶段/HashTag/CROSSSLOT 规避/AP 定位）
  - 02.1 新增**模块十一：SQL 执行全链路**（连接器权限快照/查询缓存 8.0 移除真相/解析器与预处理器边界/优化器 CBO/执行器双指针模型/长连接内存治理/ICP 索引下推）
  - 01.5 新增**模块四：I/O 多路复用与高并发网络模型**（select/poll/epoll 演进/红黑树+就绪链表/LT vs ET/C10K/共享内存误区）
  - **新建 01.6 操作系统基础与进程线程模型**（用户态内核态三事件/系统调用 vs 上下文切换/进程线程 PCB-TCB/调度算法谱系/IPC/死锁四条件与检测）
- **循环8（已完成）**：新增抓取 6 篇（TCP 重传/滑动窗口/拥塞控制 / 虚拟内存 / 过期删除与内存淘汰 / 网络面试题上 / ReAct / Multi-Agent）→ 五段式增量并入：
  - 01.5 模块二深化为**可靠性机制全参数版**（RTO 公式/D-SACK 四判别/窗口三指针/四算法表/糊涂窗口/Nagle+延迟确认）；新增**模块五：网络体系结构与协议全景**（OSI vs TCP-IP/协议地图/输入网址七步/RSA vs ECDHE/HTTP vs RPC/Ping≠TCP 通）
  - 01.6 新增**模块三：内存管理（虚拟内存与分页）**（分段 vs 分页对比/多级页表/TLB/Linux 布局/COW 与 OOM Killer/JVM AlwaysPreTouch）
  - 02.2 新增**模块十四：过期删除与内存淘汰**（两机制分野表/惰性+定期实现细节/八策略/LRU vs LFU/logc 衰减增长）
  - 06.1 新增**模块六：Agent 推理与协作模式**（ReAct 代码驱动本质/漂移与错误传播/Plan-and-Execute 动态重规划/大小模型混合降本 70-90%/Multi-Agent 三模式/上下文隔离即质量）
- **循环9（已完成 · 收官）**：新增抓取 5 篇（Java 线上问题排查 / JDK 监控工具总结 / 图解 Claude Code 目录 / oneThread / 图解 Codex 目录）→ 五段式加工：
  - **新建 05.1 生产运维与线上排查实战**（域5 首发：现象→入口速查表/止血留证取舍/CPU·Load·内存·GC·线程池·连接池·Redis·MQ 全场景 SOP/jcmd-MAT-JFR-Arthas 工具矩阵）
  - **新建 04.1 AI 原生研发与 Coding Agent 工程化**（域4 首发：六件工程化设施/源码级设计决策 grep≠RAG/落地顺序/SDD 规约驱动/Starter 化交付范式）
  - 01.3 模块五新增「动态线程池工程化实战」附注（oneThread 四件套 + 设计模式映射）
  - 07.1 模块三补充 oneThread 项目结构与技术栈全貌
  - 06.1 模块四新增「Coding Agent 工程化能力栈」附注
- **循环10（已完成 · 任务1：域5 可观测性深度）**：突破 javaup hub 页 403（改用 sitemap.xml 拿全站 URL）→ 新增抓取 6 篇（微服务可观测性体系 / SkyWalking 源码 1-3 / 分布式日志架构 / 系统性能监控；Prometheus+Grafana 专篇被内容审批拦截未重试，其内容已被可观测性体系篇完整覆盖）→ 五段式加工：
  - **新建 05.2 可观测性三支柱深度**（模块一 体系与监控分层；模块二 Prometheus 工程化：Pull/TSDB/Micrometer/PromQL/AlertManager/inhibit；模块三 分布式日志：ELK 生产架构/Kafka 削峰/Loki/ClickHouse 表设计；模块四 SkyWalking 源码级：premain/AgentClassLoader/BootService SPI/Bootstrap 注入；模块五 系统性能监控：CPU 构成/负载排查六步/线程数公式）
  - 05.1 待补充清单联动更新
- **循环11（已完成 · 任务2：javaup 内容抓取）**：sitemap 定位 URL 后新增抓取 6 篇（设计模式总览与原则 / 单例模式深度 / Synchronized 锁升级 / SpringBoot 自动配置 / 秒杀库存方案 / 雪崩防护方案；后两篇页面为动态加载仅获静态部分，已标注并以站内文档交叉补全）→ 五段式加工：
  - **新建 01.7 设计模式与编码工程化**（SOLID 七原则裁决序 + 单例五写法对比表 + 反射/序列化双破坏防护）——**解决循环6 以来"设计模式换源受阻"历史遗留**
  - 01.3 新增**模块八：Synchronized 锁升级与 JIT 锁优化**（Mark Word 四态/单向升级/偏向锁 JDK15 废弃史/锁消除粗化/无降级真相）
  - 02.3 新增**模块六：SpringBoot 自动配置原理**（ImportSelector 链/imports 文件演变与云原生动机/条件装配/覆盖与排除/自定义 starter）——**解决循环4 以来"javaguide SpringBoot 付费页"历史遗留**
  - **新建 03.3 高并发方案设计精选**（秒杀五层过滤+热点三板斧+扣减方案矩阵；雪崩传导链+防护四层矩阵+Chaos Engineering）
- **循环12（已完成 · 任务3：nageoffer 项目深挖）**：新增抓取 6 篇（牛券 oneCoupon 介绍页含 34 小节完整课程结构 / 12306 介绍页 / SaaS 短链介绍页 / 12306 技术文档三篇：分布式 ID、线程池 Builder、责任链；GitHub 双开源仓库 README 佐证源码结构）→ 五段式加工：
  - 07.1 新增**模块五：牛券 oneCoupon 深挖**（十万 QPS 定位/最小必要技术理念/34 小节课程地图/防重三层与 Starter 抽象路径/兑换 vs 秒杀/分库分表 vs 分布式数据库三论点）
  - 07.1 新增**模块六：12306 深挖**（座位分配五步优先级阶梯/区间三段扣减模型/ES vs Redis 组装误区/读扩散治理/雪花算法 Redis+Lua 动态分配标识位/Builder 线程池/责任链两类型与 MyBatis Interceptor）
  - 07.1 新增**模块七：Link SaaS 短链深挖**（五步原理链/短码四策略对比/SaaS 五挑战架构应答表/冷热分层/多租户隔离/PTS 压测 12k TPS·56k QPS/301 vs 302 裁决）
  - 07.1 待补充清单四项全部清零（牛券课程拆解/12306 算法/Link 架构/12306 技术文档）
- **循环13（进行中 · 任务4：xiaolinnote 89 题逐题加工）**：Agent 面试题批次一 8 篇（题 1/2/3/4/6/7/8/9）→ 五段式加工：
  - 06.1 新增**模块七：Agent 面试题库精解（第一批：概念·范式·拆分·记忆）**——Q1 三局限三支撑与自主闭环 / Q2 四组件工程细节（System Prompt/模型选型三要素/工具设计四原则）/ Q3 三层决策维度表 + 停止条件四机制 + Anthropic 五编排模式 / Q4·Q6 三范式与 token 量化对比 / Q7 任务拆分方法论（DAG 并行降 40-60% 延迟/原子标准/自适应拆分/验证三标准）/ Q8·Q9 记忆四层与代码级实现（Mem0/Letta/Zep/知识图谱/记忆整合/读-用-写闭环）+ 翻车点汇总表
  - 06.1 待补充清单更新：89 题精解进度 10/89（5/10 在模块六 + 8 题在模块七）
- **循环14（已完成 · 任务4 批次二）**：Agent 面试题批次二 8 篇（题 11/13/15/17/18/19/20/21）→ 五段式加工：
  - 06.1 新增**模块八：Agent 面试题库精解（第二批：选型·框架·反思·上下文·状态·评测·稳定性·Trace）**——Q11 Single/Multi 选型三类场景 + Orchestrator 三变体 + 渐进演进；Q13 手搓 vs 框架三痛点 + 「不理解就依赖」判断信号 + 核心手写周边借用；Q15 反思闭环两关键设计 + 两粒度 + 互评去「自洽」偏见；Q17 上下文工程四层组成 + 装配流水线 + 注入防护 + Token 预算降载序 + 概念边界；Q18 状态四类信息 + 字段设计 + 状态机/模型分工 + checkpoint 恢复 + 幂等；Q19 四层评测体系 + 评测集四来源 + LLM-as-Judge 偏差治理 + 安全硬门禁 + Badcase 回流；Q20 无进展循环检测四信号 + 治理四步；Q21 TTFT/关键路径/六类瓶颈/优化验证护栏
  - 06.1 待补充清单更新：89 题进度 18/89
- **循环15（已完成 · 任务4 批次三）**：Agent 面试题批次三 8 篇（题 12/14/16/22/23/24/25/26）→ 五段式加工：
  - 06.1 新增**模块九：Agent 面试题库精解（第三批：记忆压缩·规划·协作·容错·任务幻觉·数据库安全·Harness）**——Q12 压缩四法对比 + 阈值公式 + Prompt Caching 互补定位；Q14 CoT/ToT/GoT 演进与成本 + Plan-and-Execute/Re-planner；Q16 消息传递 vs 共享状态 + 静态/动态路由 + Handoff 防循环；Q22 通信层 vs 一致性层容错分层（fencing token 防僵尸/Integrator 最终提交权/Git 发现不了语义冲突）；Q23 三类幻觉区分 + 六态状态机 + 证据链 + 故障注入评测；Q24 租户隔离三层纵深 + AST Allowlist + 写操作两阶段审批 + 数据库文本即注入面；Q25 Harness 本质（运行停止≠任务完成/上下文 vs 任务状态分离）；Q26 五模块设计表 + 工具结果三态 + 对抗「改断言」
  - 06.1 待补充清单更新：89 题进度 26/89
- **循环16（已完成 · 任务4 批次四：Agent 题库收官）**：Agent 面试题最后 4 篇（题 27/28/29/30）→ 五段式加工：
  - 06.1 新增**模块十：Agent 面试题库精解（第四批：自进化·人工确认·压缩阈值·执行方差）**——Q27 自进化三层改进位置 + 四步闭环 + 验证三件套（新数据/表达变化/反例/回归）+ 经验结构两层次 + 可发布可撤回；Q28 澄清/审批/人工接管三形态 + 授权×后果判断 + 批准有边界许可 + 批准≠权限≠结果；Q29 阈值预算公式（窗口-输出-增长-余量）+ 增长预估 + 触发边界与压缩目标分离 + 缓存≠扩容；Q30 三层比较范围 + 五类差异源 + 最早可观察分歧排查 + 分层回放 + 稳定性分层定义
  - **Agent 30 题全部加工完成（30/30）**，待补充清单转向 RAG 23 题
  - 06.1 待补充清单更新：Agent 30/30 完成，下一批次 RAG
- **循环17（已完成 · 任务4 批次五：RAG 题库启动）**：RAG 面试题批次一 8 篇（题 1/2/3/4/5/6/7/8）→ 五段式加工：
  - 06.1 新增**模块十一：RAG 面试题库精解（第一批：概念·Chunking·Embedding·向量库）**——Q1 两阶段流程（离线建库/在线检索）+ 三大易混淆；Q2 三问题一根（时效/私有/幻觉）+ 幻觉是副产品；Q3 微调 vs RAG + 「检索层卡死上限」直觉 + 组合模式；Q4 存储三件套 + 六种切割策略矩阵（含 Late Chunking）；Q5 语义截断本质 + 六方案（重点 Contextual Retrieval 成本与效果）；Q6 Embedding 本质 + 余弦原理 + 选型三判断 + Hit@K 自测；Q7 三代演进 + SimCSE 各向异性 + 新趋势；Q8 ANN 原理 + HNSW/IVF + 生产三能力 + 五库选型对比
  - 06.1 待补充清单更新：RAG 进度 8/23
- **循环18（已完成 · 任务4 批次六：RAG 题库批次二）**：RAG 面试题批次二 8 篇（题 9/10/11/12/13/14/15/16）→ 五段式加工：
  - 06.1 新增**模块十二：RAG 面试题库精解（第二批：生产实战·在线流程·混合检索·查询优化·高级范式·图增强）**——Q9 Milvus 生产实战（150万向量/内存账/SQ8量化/Segment合并错峰/HNSW三参数）；Q10 在线六步流水线（同模型约束/HyDE/多路并行/Rerank两阶段/溯源）；Q11 BM25 vs 向量互补 + RRF 排名融合原理；Q12 Query Rewrite 四法对症；Q13 多路召回三路分工 + 调大 K 不是解法；Q14 四层优化框架（Small-to-Big/查询四法/三路召回/Cross-Encoder）+ 典型生产组合；Q15 三代演进 + Self-RAG/CRAG/GraphRAG/Agentic RAG 机制对比；Q16 图增强判据（多实体关系推理）+ 向量定位+图遍历组合
  - 06.1 待补充清单更新：RAG 进度 16/23
- **循环19（已完成 · 任务4 批次七：RAG 题库收官）**：RAG 面试题最后 7 篇（题 17/18/19/20/21/22/23）→ 五段式加工：
  - 06.1 新增**模块十三：RAG 面试题库精解（第三批：幻觉治理·评估·更新·知识冲突·长上下文·Agent选型）**——Q17 幻觉两来源 + 四方案分级；Q18 Hit@K/MRR + RAGAs 四指标 + 线上五指标闭环；Q19 先删后增 + hash 检测 + 轮询/事件驱动 + 灰度回滚；Q20 三大难分层；Q21 冲突裁决顺序（先适用再权威）+ 元数据身份证 + 冲突检测两阶段 + 治理闭环；Q22 容量≠有效利用 + 检索缩范围+大窗口完整阅读；Q23 搜索方式 vs 取资料流程两维度 + 索引定位+Agent 探索组合
  - **RAG 23 题全部加工完成（23/23）**，待补充清单转向 LLM 工具调用 22 题
  - 06.1 待补充清单更新：RAG 23/23 完成，下一批次 LLM 工具调用
- **循环20（已完成 · 任务4 批次八：LLM 工具调用题库启动）**：LLM 工具调用面试题批次一 8 篇（题 1/2/3/4/5/6/7/8）→ 五段式加工：
  - 06.1 新增**模块十四：LLM 工具调用面试题库精解（第一批：Function Calling 原理与训练 · MCP 协议与选型）**——Q1 FC 本质（模型决策/代码执行/两轮对话/并行调用）；Q2/Q3 SFT+RLHF 两阶段 + 五场景数据覆盖 + PPO/KL 约束 + RLAIF；Q4 MCP 本质（三角色/三类能力/JSON-RPC+双传输/调用链路/安全）；Q5 三层组成 + Streamable HTTP 版本演进；Q6 FC vs MCP 层次关系 + MCP 底层靠 FC 驱动 + 实操接入；Q7 选型五步流程 + 内嵌 vs 独立；Q8 推理模型不支持 MCP 的范式冲突 + KV Cache/一致性/训练目标三层原因 + 思考后调用与 interleaved thinking 解法
  - 06.1 待补充清单更新：LLM 工具调用进度 8/22
- **循环21（已完成 · 任务4 批次九：LLM 工具调用题库批次二）**：LLM 工具调用面试题批次二 8 篇（题 9/10/11/12/13/14/15/16）→ 五段式加工：
  - 06.1 新增**模块十五：LLM 工具调用面试题库精解（第二批：Skill · A2A · 通信协议 · LLM 网关）**——Q9 Skill 技能包结构 + 渐进式三层加载（token 经济学）；Q10 MCP「手」vs Skill「操作手册」+ code-review 配合走查；Q11 三层架构总括（FC 语言/MCP 工具箱/Skill 操作手册）+ 依赖链 + 做菜类比 + 数据分析场景串联；Q12 A2A 协议（Agent Card 服务发现 + Task 异步状态机 + Agent 微服务化）；Q13 stdio vs Streamable HTTP 传输选型 + JSON-RPC 与传输解耦；Q14 SSE vs WebSocket（文字流式行业标准 + 双向场景局限）；Q15 WebRTC vs WebSocket（UDP 低延迟根因 + NAT 穿透三层降级 + WS 只做信令 + OpenAI Realtime 选型）；Q16 LLM 网关六大能力（语义缓存杀手级）+ 五产品选型
  - 06.1 待补充清单更新：LLM 工具调用进度 16/22

## 五、全量完成度总览

| 知识域 | 文档数 | 状态 |
|--------|--------|------|
| 域1 Java核心与编码工程化 | 7（01.1~01.7） | ✅ |
| 域2 后端技术组件工程化 | 3（02.1~02.3） | ✅ |
| 域3 分布式与高可用架构 | 3（03.1~03.3） | ✅ || 域4 工程交付与云原生 | 1（04.1，AI 原生方向） | ✅ 首发；传统 CI/CD·Docker·K8s 待扩充 |
| 域5 生产运维与可观测性 | 2（05.1 + 05.2） | ✅ 监控三支柱已补全 |
| 域6 性能优化与架构演进 | 1（06.1，十五模块） | ✅ Agent 30 题 + RAG 23 题全解 + 工具调用 16/22；编程题待加工 |
| 域7 业务场景与实战案例 | 1（07.1，七模块） | ✅ 三大业务项目深挖完成 |

**后续扩充方向**（非本轮范围）：传统云原生（Docker/K8s/CI-CD）、Prometheus 多集群长期存储（Thanos/Cortex）与 SLO 错误预算、OpenTelemetry 规范、javaguide 分布式/微服务 P1 篇目、设计模式专题其余篇目（javaup sitemap 已定位 URL，待抓取）、nageoffer 牛券/12306/Link GitHub 源码级走读与 Ragent 源码级拆解、xiaolinnote 89 题逐题精解。
