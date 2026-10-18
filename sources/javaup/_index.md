# 码力全开（javaup.chat）站点目录

> **来源**：https://javaup.chat/
> **定位**：深挖技术 · 项目实战 · AI 进阶——Java 技术知识库、AI 大模型面试详解、高并发架构与生产级项目实战
> **抓取日期**：2026-10-18
> **知识库角色**：与 JavaGuide 互补去重，补深度原理（HashMap/ConcurrentHashMap 专题极深）+ 高并发项目实战
> **规模**：超级八股文约 250+ 篇（站点自述近 400 篇、100 万字），部分完整内容需知识星球解锁

## 板块总览与知识域映射

| 板块 | 站点路径 | 对应知识域 | 文章数 |
|------|----------|------------|--------|
| Java 基础/集合/IO/JVM/类加载/并发/设计模式/Stream/安全 | /java/... | 域1 | ~170 篇 |
| 开发工具（Maven/Git/Docker/日志） | /tools/... | 域4 | 13 篇 |
| 基础内功（OS/网络/数据结构） | /system/... | 域1 | 26 篇 |
| 数据库（MySQL/Redis/Oracle） | /database/... | 域2 | ~60 篇 |
| 框架中间件（Spring/MyBatis/Cloud/MQ/ES/Dubbo/Netty/XXL-JOB/Tomcat） | /framework/... | 域2 | ~75 篇 |
| 进阶设计与性能优化（微服务/分布式/分库分表/事务/DDD/AI/方案设计） | /architecture/... | 域3/6/7 | ~70 篇 |

---

## 1. Java（/java/）

### 面试技巧
- 面试准备全攻略 — /java/interview-skills/preparation-guide/
- 项目介绍与技术面试准备 — /java/interview-skills/project-and-tech-preparation/
- 面试与复盘技巧 — /java/interview-skills/mindset-and-review/

### 基础知识（/java/base/）
Java语言特性与核心概念、面向对象核心概念、异常处理与final-finally-finalize、基本类型与包装类、面向对象编程基础、接口与抽象类、String类详解、泛型详解、Java数值与金额精度、反射与动态代理、Java序列化详解、JDK新版本特性详解、static关键字与枚举详解、注解机制详解、日期时间处理与线程安全、UUID与唯一标识生成、字符编码详解、equals与hashCode方法详解、Arrays排序算法详解、ClassNotFoundException异常分析、Request的相关概念解释

### 集合（/java/collect/）
- Java集合框架详解 — /java/collect/introduce/
- Set集合与排序机制详解 — /java/collect/set-collection-and-sorting/
- 集合遍历修改与并发安全 — /java/collect/safe-traversal-and-modification/
- **HashMap 专题（7 篇）**：核心原理与数据结构 /java/collect/hashmap-core-principles/、哈希算法与索引计算 /hashmap-hash-and-index/、存取删除操作 /hashmap-operations/、扩容与容量优化 /hashmap-resize-and-capacity/、Hash冲突原理与解决方案 /hash-collision-solutions/、线程安全问题与解决方案 /hashmap-thread-safety/、遍历与操作最佳实践 /hashmap-traversal-best-practices/
- **ConcurrentHashMap 专题（3 篇）**：线程安全机制 /java/collect/concurrenthashmap-thread-safety/、架构演进与设计权衡 /concurrenthashmap-architecture-evolution/、高级特性与最佳实践 /concurrenthashmap-advanced-features/

### IO（/java/io/）
IO基础与流操作、IO设计模式详解、IO模型与并发处理策略、BIO/NIO/AIO核心原理、NIO核心组件与零拷贝、IO多路复用技术详解

### JVM（/java/jvm/）— 约 45 篇，分 5 组
- **内存分布**：运行时数据区概述、虚拟机栈与本地方法栈、Java堆内存详解、对象分代晋升规则、方法区与元空间、字符串常量池与直接内存、对象创建流程详解、对象内存布局详解、对象访问定位与堆栈区别、Java进程内存结构详解、对象创建与内存分配策略、对象存活判断与GC Roots、常量池机制与内存结构深度解析
- **内存回收**：垃圾回收机制深度解析、垃圾回收流程与触发机制、内存分配与回收原则
- **垃圾收集器**：垃圾收集器详解、G1原理与优化、ZGC低延迟收集器、Java版本GC演进对比、收集器对比与选型策略
- **诊断与调优**：JIT即时编译、AOT编译与混合编译、内存泄漏与内存溢出、JVM异常处理与退出机制、内存问题排查与恢复策略、堆内存与元空间配置、GC日志配置、JDK命令行诊断工具、可视化诊断工具、第三方诊断工具与实战

### 类加载（/java/class-loading/）
类的生命周期与加载时机、类加载详细流程、类加载器与双亲委派机制、双亲委派破坏与安全机制、Class文件结构深度解析

### 并发多线程（/java/concurrent-multithreading/）— 约 30 篇
- 基础：并发编程基础概念、线程调度与上下文切换、线程通信与协作机制、线程安全与并发控制、线程同步与数据一致性、线程异常处理机制、CAS机制深度剖析、虚拟线程与现代并发编程
- **内存模型**：JMM核心原理、Happens-Before原则、内存屏障与指令重排、Volatile关键字深入解析
- **锁**：锁机制分类与对比、锁的公平性与竞争机制、死锁与活锁问题分析、Synchronized并发特性保证、Synchronized实现原理、Synchronized锁升级与优化、Volatile与Synchronized对比
- **线程池**：核心原理与最佳实践、高级特性与调优策略
- **并发工具**：AQS抽象队列同步器、ThreadLocal、TransmittableThreadLocal线程池上下文传递、JUC并发工具包、Unsafe底层操作、并发集合深入剖析、异步编程与CompletableFuture、并发同步工具实战

### 设计模式（/java/design-pattern/）
设计模式概述与基本原则、单例模式深度解析、工厂模式体系详解、行为型设计模式实践、结构型设计模式应用、MVC架构模式与实战

### Stream（/java/stream/）
Lambda表达式与语法糖机制、Stream API核心操作、Stream并行流原理与性能优化

### 恶意攻击/安全（/java/security-attack/）
Web安全攻击与防护、网络层攻击与防护、权限控制与越权漏洞、密码学基础与加密技术

### Tomcat（/java/tomcat/）
核心架构与启动流程、请求处理机制、类加载器、IO模型

## 2. 开发工具（/tools/）

- **Maven**：项目管理工具详解、依赖管理与冲突解决、插件系统与开发实践
- **Git**：版本控制系统全面解析、代码回滚与分支合并策略、GitHub高效使用技巧
- **Docker**：容器技术核心概念、安装配置与实战操作、Dockerfile镜像构建与容器编排
- **日志**：日志框架与门面模式、日志性能优化实践、分布式日志系统架构

## 3. 基础内功（/system/）

- **操作系统**：进程线程与协程、进程通信与调度、内存管理与虚拟地址、CPU缓存与一致性协议、系统性能监控、IO优化与零拷贝、线程实现机制
- **网络**：网络基础与协议模型、TCP协议深入解析、HTTP协议演进与特性、网络工具与地址解析、网络架构与优化、网络诊断与监控
- **数据结构**：线性数据结构、树形结构基础、平衡树与搜索树、特殊树结构、图结构

## 4. 数据库（/database/）

### MySQL — 约 40 篇，分 6 组
- **锁**：锁机制概述、锁的粒度与层级、行级锁详解、乐观锁与悲观锁实战、死锁问题分析与解决
- **事务**：事务机制与ACID、事务执行流程与两阶段提交、事务隔离级别详解、隔离级别实现原理、幻读问题深度剖析、事务优化实战、隔离级别选择策略
- **索引**：索引基础与底层原理、索引类型与存储结构、联合索引与查询优化、索引失效问题排查、索引设计最佳实践
- **慢SQL调优**：执行计划深度解析、ORDER BY排序优化、JOIN查询优化、SQL性能调优全面指南、分页查询原理与深度优化
- **特征**：架构与执行流程、存储引擎与高级特性、主从复制与同步机制、版本特性与演进、查询缓存机制
- **使用优化**：字段类型与设计规范、主键设计与自增机制、性能优化与开发规范、数据存储优化策略、高级特性与最佳实践、热点数据更新优化、数据加密与安全
- **进阶实战**：大表在线DDL、千万级数据查询优化、热点数据更新与缓存、数据归档与平滑迁移、逻辑删除与唯一性约束

### Redis — 约 25 篇，分 7 组
- 基础与数据类型：数据类型详解与应用场景、核心数据类型与底层实现、线程模型与性能优化、高级数据结构实现原理
- 持久化与内存：持久化与数据安全、内存管理与优化策略
- 集群与高可用：分布式特性与对比、集群架构与高可用方案、Cluster操作限制与解决方案
- 事务与原子操作：事务与原子性操作、原子性操作与命令详解
- 消息队列：消息队列与通信模式
- **分布式锁（6 篇）**：分布式锁实战指南、乐观锁机制与并发控制、Redisson核心实现原理、Watchdog自动续期机制深度剖析、lock与tryLock对比、RedLock算法原理与实践
- 缓存一致性：与数据库一致性保障方案
- 性能优化与实战：性能问题诊断与优化、实战应用场景、操作最佳实践与遍历技巧
- 新版本：Redis 8.0 新特性

### Oracle（5 篇）
索引技术全面解析、核心语法与特性、事务隔离机制、与MySQL全面对比、视图与数据安全

## 5. 框架中间件（/framework/）

- **Spring（17 篇）**：核心概念、Bean生命周期与依赖注入、高级特性、事件机制、循环依赖解决方案、后置处理器；Spring事务（事务管理核心、传播机制与隔离级别、实现原理与注解、失效场景、高级应用）；SpringBoot（核心特性、AutoConfigureBefore、自动配置原理、启动与Web容器、自定义Starter、为什么要自动装配、多环境配置、条件化Bean、SpringBoot3/6新特性、Bean加载顺序控制、启动原理、SpringBoot4/7新特性）
- **MyBatis（11 篇）**：基础与核心概念、工作原理与核心组件、参数处理与SQL注入防护、动态SQL、结果映射与类型处理、分页实现、缓存机制、插件机制与拦截器、延迟加载与关联查询、连接池与数据源、MyBatis-Plus增强特性
- **SpringCloud（12 篇）**：核心概念与架构、API网关选型、Eureka、Hystrix熔断、Nacos（架构/服务发现/配置中心/一致性协议）、负载均衡、Feign（概念/配置超时/性能优化）
- **Zookeeper（5 篇）**：核心架构与数据模型、一致性保障、选举机制、监听机制、典型应用场景
- **Kafka（7 篇）**：基础架构、消息可靠性、发送与存储原理、消费机制、事务与一致性、重平衡机制、高级特性
- **RocketMQ（6 篇）**：架构设计与工作流程、消息可靠性、消费模式与分发、顺序消息与幂等、高级特性与事务消息、性能优化与故障处理
- **RabbitMQ（5 篇）**：架构与工作模式、消息可靠性、高级特性、消费者流控与幂等、集群与高可用
- **Elasticsearch（8 篇）**：基础与数据类型、倒排索引、集群架构、数据精度与并发控制、深度分页、索引生命周期管理、性能优化、数据同步与一致性
- **Dubbo（6 篇）**：RPC基础、核心架构与调用流程、序列化与通信协议、负载均衡与服务治理、SPI扩展机制、高级特性
- **Netty（6 篇）**：核心架构与线程模型、设计模式、零拷贝、ByteBuf内存管理、性能优化、序列化与粘包拆包
- **XXL-JOB（3 篇）**：调度机制与一致性、时间轮算法、分片任务与并行调度

## 6. 进阶设计与性能优化（/architecture/）

### 微服务（5 篇）
架构设计理念、拆分策略与实践、间通信技术、治理与稳定性保障、发布与部署策略

### 分布式理论（3 篇）
一致性理论、一致性哈希算法、共识算法与拜占庭问题

### 分布式ID（2 篇）
生成方案详解、MybatisPlus 在 k8s 环境 id 重复问题

### 限流与熔断（4 篇）
限流算法原理与实现、熔断降级机制、高级限流策略、高并发系统设计实践

### 缓存设计（4 篇）
淘汰策略、本地缓存实现、本地缓存一致性、多级缓存架构

### 其他专题
- 布隆过滤器（1 篇）
- 延迟队列（2 篇）：高效延迟队列设计、Redisson 分布式延迟队列原理
- 问题故障解决（4 篇）：JVM诊断工具、堆Dump分析、字节码插桩、Linux日志分析
- **分库分表（7 篇）**：核心概念、分表字段选择与数据倾斜、分片算法与ShardingJDBC、容量规划与扩容、分布式ID、跨库查询优化、分布式数据库选型
- **分布式事务（5 篇）**：基础概念、2PC/3PC、TCC、消息方案、本地消息表
- **Seata（3 篇）**：核心原理、事务模式选型、AT模式与隔离性
- **DDD（4 篇）**：核心思想、分层架构、领域模型要素、领域事件与落地
- **AI（5 篇）**：大模型核心概念、DeepSeek架构、RAG、AI Agent与工具调用协议、Spring AI
- **方案设计（16 篇）**：秒杀系统架构与库存扣减、MQ设计与消息乱序、高并发计数、缓存设计与预热、订单支付并发冲突、敏感词过滤、短链服务、高并发流量防护、接口性能优化、认证授权与会话管理、大规模消息推送、信息流Feed架构、网站流量统计、RPC框架设计、MQ设计要点、分库分表二次扩容

## 实战项目（站点首页展示）

| 项目 | 亮点 | 链接 |
|------|------|------|
| Nexus Agent 智能体 | 前置编排 + ReAct 执行器 + 五路检索融合精排 + 多级知识图谱 + Skills/Harness 治理 | /super-agent/overview/project-intro/ |
| 智能体认知引擎 | 跨客户端共享记忆、L0–L3 分层演化、MCP 六大记忆工具、权限治理 | /agent-up-memory/overview/project-intro/ |
| 黑马点评 Plus | 令牌前置授权+令牌桶限流、多层缓存+布隆过滤器、分布式锁与幂等、Kafka可靠消息、分库分表、可观测 | /hmdp-plus/overview/project-intro/ |
| 大麦网高并发 | 分库分表、缓存策略、限流降级、分布式锁、性能监控 | /damai/overview/project-intro/ |
| 大麦AI智能 | SpringAI + Ollama/OpenAI/DeepSeek/通义千问多模型、RAG、Function Calling | /damai-ai/overview/project-intro/ |
| 流量灵活切换 | 蓝绿发布、灰度发布、流量切换、动态配置、组件化集成 | /link-flow/business-intro/getting-started-overview/ |
| 数据中台 | 规则化指标计算引擎、动态数据源治理、MQ驱动增量汇总、对账闭环 | /dock-data-center/business-intro/info/ |

## 去重与互补策略

| 重叠主题 | javaup 定位 | 与 JavaGuide 的关系 |
|----------|-------------|---------------------|
| Java 基础 | 逐篇深度专题（如 String、泛型、注解各一篇） | JavaGuide 作面试主干；javaup 作深度补充，按需收录差异点 |
| HashMap/ConcurrentHashMap | **7+3 篇专题，全中文互联网最深的免费专题之一** | 直接收录为核心资产 |
| JVM | 45 篇细分到每个知识点（如对象内存布局、GC Roots） | 比 JavaGuide 更细，作查漏工具书 |
| MySQL/Redis | 与 JavaGuide 大量重叠，但有独有专题（如 k8s id 重复、MySQL 8.0、热点数据更新） | 只收录独有专题 |
| 分库分表/分布式事务/Seata/DDD | 体系完整且有实战深度 | 与 JavaGuide 互补，作域3核心来源之一 |
| 方案设计 16 篇 | 场景题极全（秒杀/短链/Feed/敏感词/RPC设计） | 域7 核心来源 |

## 抓取优先级

1. **P0**：HashMap 7 篇、ConcurrentHashMap 3 篇、线程池 2 篇、AQS、Synchronized 锁升级（JavaGuide 覆盖浅的深度专题）
2. **P1**：分库分表 7 篇、分布式事务 5 篇、Seata 3 篇、Redisson 锁 6 篇、方案设计 16 篇
3. **P2**：MySQL 独有专题（大表DDL、热点更新、千万级优化）、Kafka/RocketMQ/RabbitMQ 三套（JavaGuide 是面试题形式，javaup 是专题形式）
4. **P3**：基础内功（OS/网络/数据结构）、工具链（Maven/Git/Docker）、AI 5 篇
