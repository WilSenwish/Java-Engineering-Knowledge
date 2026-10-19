# Ragent 源码级走读（nageoffer/ragent）

> **来源站点**：GitHub 开源仓库 nageoffer/ragent（浅克隆 depth=1）
> **原文链接**：https://github.com/nageoffer/ragent
> **走读日期**：2026-10-19
> **所属知识域**：域6/域7（AI 应用 · 实战案例）
> **板块**：项目实战 · Ragent（企业级 Agentic RAG）· 源码级走读

---

## 走读范围与方法

- 仓库规模：**945 个 Java 文件、约 119030 行代码、7 个 Maven 模块**（agent / bootstrap / framework / infra-ai / rag / system / mcp-server）+ React 前端
- 技术栈：Java 17、Spring Boot 3.5.7、MyBatis-Plus、MySQL、Milvus 2.6、Redis + Redisson、RocketMQ 5.x、S3/RustFS、Sa-Token、AgentScope 2.0.2、OkHttp、Apache Tika 3.2、Langfuse（OTLP）
- 方法：按「入口链路 → 问题理解 → 多路检索 → 后处理 → 生成 → 记忆」主链 + 「模型工程化 / Agent 引擎 / 工程基建 / 入库 Pipeline」四条支线逐类走读，本文记录**类级别事实**（路径、接口签名、调用链、并发与降级细节）

---

## 一、总架构：一次提问穿过两个引擎

**引擎选择是部署级 Bean 装配开关，不是运行时策略**：`ragent.engine.type=workflow`（默认，v1）时 agent 模块所有组件不注册（`@ConditionalOnAgentEngine` 自定义条件注解），入口是 rag 侧 `RAGChatController` 的 `/rag/v3/chat`；`=agent`（v2）时注册 AgentScope ReAct 引擎，RAG 管线降级为一个工具（`search_knowledge`）。两套引擎共用 DB 与检索门面。

**v1 主链调用链**（包前缀 `com.nageoffer.ai.ragent.rag`）：

```
RAGChatController#chat（GET /rag/v3/chat，SseEmitter，@IdempotentSubmit 按 userId 防重复提交）
 → RAGChatServiceImpl#streamChat（雪花 conversationId/taskId）
   → ChatQueueLimiter#enqueue（未开限流直投 chatEntryExecutor；超时发 REJECT+FINISH+DONE 并落库拒绝消息）
     → StreamChatTraceRunner#run（ForwardingStreamCallback 装饰记录 TTFT/终态）
       → StreamChatPipeline#execute(ctx)
```

`StreamChatPipeline#execute` 阶段序（`StreamChatContext` 承载中间状态，私有 `handleXxx` 返回 true 短路）：

```
loadMemory → rewriteQuery → resolveIntents
 → handleGuidance（歧义澄清则短路输出引导 prompt）
 → handleSystemOnly（纯系统意图裸 LLM 应答）
 → retrieve → handleEmptyRetrieval（空则回复"未检索到…"）
 → streamRagResponse（sources 装配 → 引用编号注入 → grounding → streamLLMResponse）
```

**v2 主链**（包前缀 `com.nageoffer.ai.ragent.agent`）：

```
AgentChatServiceImpl#streamChat（agent/v1/chat）
 → ReActAgentProvider#buildAgent（AgentScope ReActAgent 单例，按「人设+工具目录指纹」懒重建）
   → agent.streamEvents(input, runtimeContext) → Flux<AgentEvent>
 → AgentStreamEventBridge（转 SSE + 落库）
```

ReActAgent 构建参数：`maxIters=10 / maxRetries=1 / enablePendingToolRecovery(true)（强制停止留悬空调用，下轮先补错误结果）/ stateStore(PgAgentStateStore) / middleware 洋葱链`。中间件链顺序（外→内）：`OtelTracing → AgentTraceEnrichment → userMemory → contextCompaction → confirmDenial → skillMasking → toolBatch(order=0 最内)`。

---

## 二、问题理解（rewrite / 映射 / 意图树）

| 关键类 | 职责 |
|---|---|
| `rag/core/rewrite/QueryRewriteService`（接口） | `rewrite` / `rewriteWithSplit`，default 逐级退化 |
| `rag/core/rewrite/MultiQuestionRewriteService` | LLM 改写+拆分（FAST 档，temperature 0.1）；历史裁剪：assistant 仅留最近 2 条且只收 NORMAL 状态（排除 INTERRUPTED/REJECTED 半截答案）；解析失败兜底归一化；`ruleBasedSplit` 按标点拆分兜底 |
| `rag/core/rewrite/QueryTermMappingService` | 术语归一化：Redis 缓存映射规则，priority+源词长度排序，防重复替换 |
| `rag/core/intent/IntentResolver` | 按子问题 `CompletableFuture` 并行分类（intentClassifyExecutor）；`INTENT_MIN_SCORE=0.35` 过滤、`MAX_INTENT_COUNT=3` 上限；超限「每子问题保底 1 个 + 余配额按分补」；分类异常降级空意图 |
| `rag/core/intent/DefaultIntentClassifier` | 唯一实现（串行）：意图树 Redis 优先、DB 回填（IntentTreeCacheManager 7 天）；单次 LLM 输出 `[{id,score}]`，容错 `{"results":[…]}` 包装；任何失败返回空列表 |
| `rag/core/guidance/IntentGuidanceService` + `AmbiguityLLMChecker` | 歧义澄清：按根到叶路径重名找分叉，LLM 二次确认（FAST 档）；**响应非法/调用失败一律返回 false（跳过澄清、放行检索）**，不让模型异常反复阻断用户 |

**意图置信度不足的处理**：低于 `INTENT_MIN_SCORE` 的意图不参与检索；`RetrievalScopeResolver` 再看 `scope.confidence-threshold(0.6)`，top1 不足则整体退化为全局检索。

---

## 三、多路检索与后处理流水线

**通道接口**（`rag/core/retrieval/channel/SearchChannel.java`）：

```java
String getName();
boolean isEnabled(SearchContext context);
SearchChannelResult search(SearchContext context);
SearchChannelType getType();
default SearchChannelResult emptyResult(long latencyMs);   // 统一空结果形状
```

| 实现 | 要点 |
|---|---|
| `VectorSearchChannel` | 定向/全局同一条通道（避免同一证据在 RRF 自我加权）；定向时主路+补充路**共用一次 embedding、同池并行**；补充路 `exceptionally` 隔离失败；`CollectionParallelRetriever` 逐库 fan-out（后端不支持跨库时），出口统一排序+按 budget 截断（防 PG `relaxed_order` 乱序） |
| `KeywordSearchChannel` | ES BM25，`@ConditionalOnProperty(rag.keyword.type=es)` |
| `GraphSearchChannel` | LightRAG，单实例单图，结果侧按 file_path 归属切分；定向下 top_k×3 上浮补召回 |
| `WebSearchChannel` | You.com；任何失败只 warn + 空结果；初始分 1/(rank+1) 中性分 |

**并行调度**（`MultiChannelRetrievalEngine`）：通道过滤后按 `SearchChannelType.ordinal()` 稳定排序（仅日志可复现，无语义优先级）；`CompletableFuture.supplyAsync(..., ragRetrievalExecutor)` 并行，`future.orTimeout(channelTimeoutMs)`（默认 15s）**超时按空结果降级、不中断任务**；`join()` 汇总后逐通道打日志。

**作用域与配额**：`RetrievalScopeResolver` 取 KB 意图最高分（按节点去重、需绑定 collection），`topScore ≥ 0.6` → 定向；否则全局。`ScopeQuota` 主路/补充路名额切分（`supplement-ratio=0.25`，夹在 `[1, budget-1]`）——补充路给固定名额而非自由竞争（意图判错时正确证据拼不过命中库的「表面相似」）。

**预算漏斗**（`RetrievalBudget`）：`recallBudget(20) → candidateLimit(40) → contextTopK(10)` 三段独立，`SearchChannelProperties#afterPropertiesSet` 启动校验漏斗单调。

**后处理责任链**（`rag/core/retrieval/postprocessor/SearchResultPostProcessor.java`，Spring 注入 List 按 order 排序，单处理器异常只跳过该处理器）：

| 顺序 | 类 | 行为 |
|---|---|---|
| 1 | `DeduplicationPostProcessor` | `RetrievedChunkKey` 去重，保留首次出现；不比较跨通道分数 |
| 5 | `FusionPostProcessor` | RRF：`score = Σ weight/(k+rank+1)`（rrfK=20；vector/keyword 权重 1.0、graph/webSearch 0.5）；名次取自不可变 results（去重不丢多路命中）；融合后按 rerankCandidateLimit 截断 |
| 10 | `RerankPostProcessor` | Cross-Encoder 按 contextTopK 精排；打精排分布与图谱存活率归因日志 |
| 15 | `EvidenceGatePostProcessor` | 证据闸门：整批最高精排分 < min-rerank-score(0.2) 整批丢弃；**无分可读（精排关闭/noop）一律放行** |
| 20 | `MetadataEnrichmentPostProcessor` | 链末回表富化 docId/chunkIndex/docName；只富化不重排 |

---

## 四、上下文组装、生成与会话记忆

- `core/source/SourcesAssembler`：按 docId 归并片段、按相关度赋**唯一来源编号**（SSE/落库/角标同源），url/feishu 补外链
- `core/source/CitationContextEnricher`：正则把上下文里内部 `data-ragent-doc-id` 替换为模型可见 `ref="N"`；**内部 docId 不暴露给模型**
- `core/prompt/RAGPromptService#buildStructuredMessages`：system（命中单意图的 promptTemplate 或 Agent 模板 KB_ANSWER + 引用规则）→ history（摘要作为 history[0] system）→ user（证据+问题合并一条）
- `core/prompt/DefaultContextFormatter`：三分支（单/多/无意图）；**刻意不注入文档标题**（防「出自《XX》」幻觉）
- `infra/chat/LLMService#streamChat(request, callback) → StreamCancellationHandle`；回调式（`onContent/onThinking/onSources/onGroundingChunks/onComplete/onError`），非 Flux
- `service/handler/StreamChatEventHandler`：按 messageChunkSize（默认 5 码位）分块发 message 事件；onComplete 组装落库后发 finish+done；取消时已累积内容以 INTERRUPTED 落库并补发 cancel
- SSE 事件类型：`meta / message / finish / done / cancel / reject`

**会话记忆**（`rag/core/memory`）：
- `DefaultConversationMemoryService`：`load` 用 memoryLoadExecutor **并行**加载摘要与历史，各自 try-catch 兜底；摘要包装成 system 消息置于 history[0]
- `JdbcConversationMemoryStore`：滑动窗口 `historyKeepTurns(8)×2` 条；`normalizeHistory` 剥离开头 ASSISTANT 保证 USER 打头；assistant 入库前 `CitationMarkup.strip` 去角标
- `JdbcConversationMemorySummaryService`：仅 ASSISTANT 触发、异步；Redisson 锁 `ragent:memory:summary:lock:{userId}:{conversationId}` 防并发压缩；`summaryStartTurns(9)` 触发；摘要窗口取最近 maxTurns 轮中点作 cutoff（**约覆盖一半原文窗口，重叠段滑出后才再生成**）；FAST 档 LLM 合并旧摘要+新段（≤200 字符、单行），失败保留旧摘要

---

## 五、模型工程化（infra-ai 模块）

**客户端抽象**（OkHttp + Gson + Hutool）：

```java
public interface ChatClient {
    String provider();
    String chat(ChatRequest request, ModelTarget target);
    StreamCancellationHandle streamChat(ChatRequest request, StreamCallback callback, ModelTarget target);
}
public interface EmbeddingClient { String provider(); List<Float> embed(...); List<List<Float>> embedBatch(...); }
public interface RerankClient { String provider(); List<RetrievedChunk> rerank(query, candidates, topN, target); }
```

- 模板方法基类：`AbstractOpenAIStyleChatClient`（钩子 `supportsEnableThinkingParam`/`requiresApiKey`/`customizeRequestBody`）、`AbstractOpenAIStyleEmbeddingClient`（钩子 maxBatchSize）
- 供应商薄子类：BaiLian / SiliconFlow（DashScope 私有 `enable_thinking`）/ Ollama（无 key）/ AIHubMix / DeepSeek（思考方言是 `thinking:{type:enabled}` 对象而非布尔）
- SSE 解析：`OpenAIStyleSseParser`（`data:` 前缀、`[DONE]`、delta/message 双形态、可选 `reasoning_content`）
- OkHttp 双客户端：streaming（read/callTimeout 0，流式不限）+ sync（45s 上限）

**模型档位**（`infra/model`）：`Tier{FAST,STANDARD,DEEP}`；`ModelSelector.resolveTierName` 顺序：`request.thinking=true → deepThinkingTier` → 显式 override → defaultTier。`buildTierTargets` 构建有序候选：preferredModelId 置队首 → 追加档位 candidates 去重 → 逐个过滤（未登记/enabled=false/思考请求但不支持/ provider 配置缺失/healthStore.isUnavailable）。`ChatTierConfigValidator` 启动 fail-fast：tiers 非空、default/deep-thinking 引用存在、timeout-ms 必填、候选 id 在注册表、每个 Tier key 有对应档位、deep 档至少一个支持思考的候选。`timeout-ms` 语义：同步=整段调用上限，流式=首包 TTFT 预算。

**首包探测与流式零脏数据切换**（`infra/chat`）：

- `ProbeStreamBridge`（缓冲装饰型 StreamCallback）：未提交前缓存全部事件；`onContent/onThinking` **非空白**才完成探测（纯空白流走 onComplete→noContent）；`awaitFirstPacket` 超时→TIMEOUT；SUCCESS 时锁内按序 flush 整个 buffer（含首包前空白/thinking）；**ERROR/NO_CONTENT 时 buffer 永不 flush**，随桥接器丢弃
- `RoutingLLMService.streamChat` 候选循环：`healthStore.allowCall` → 建 bridge → `client.streamChat` → `awaitFirstPacket(timeout)` → SUCCESS 则 commit 返回（后续事件直连下游）；失败则 markFailure + OkHttp `call.cancel()` → 下一候选（**用户端零脏数据**）；全部失败 `onError(RemoteException)`
- `LlmFirstPacketProbe` 拆独立 bean 正是为规避 Spring AOP 类内 self-call 不拦截

**三态熔断器**（`infra/model/ModelHealthStore`，进程内单例）：`ConcurrentHashMap<String, ModelHealth>` **每模型独立**；`State{CLOSED,OPEN,HALF_OPEN}`；全部迁移在 `compute()` 内原子完成：
- `allowCall`：OPEN 未过期 → 拒绝；OPEN 过期 → HALF_OPEN + `halfOpenInFlight=true` + 唯一探测名额（半开期间并发仅放行 1 个）；CLOSED → 放行
- `markSuccess`：→ CLOSED 计数清零（恢复后经 `isUnavailable=false` 自然重新入池）
- `markFailure`：HALF_OPEN 失败 → 立即回 OPEN 重置冷却；CLOSED 连续失败 ≥ threshold(2) → OPEN，`openUntil=now+30s`
- `releaseHalfOpenPermit`：仅当 halfOpenToken 匹配才归还名额（取消场景专用，不计失败）
- **取消甄别**：`TaskCancellation.isCancellation(e)` 为真时不降级、不累计失败，仅归还半开名额后原样抛出

**同步 fallback**（`ModelRoutingExecutor#executeWithFallback`）：chat/embedding/rerank 共用；provider→client 缺失 warn 跳过；熔断拒绝跳过；全失败抛 `RemoteException("All X model candidates failed")`。VLM 只取第一个候选**无 fallback**（入库期单次同步）；Embedding 指定模型时用单 target 不降级。

**Token 计算**（`infra/token/HeuristicTokenCounterService`）：不用 tokenizer 库，纯启发式——CJK 1 字符 = 1 token；ASCII 4 字符 ≈ 1 token；其他 2 字符 ≈ 1 token；空文本 0，结果下限 1。

---

## 六、Agent 引擎 V2（agent 模块）

**依赖**：`io.agentscope:agentscope-core 2.0.2` + `agentscope-extensions-model-openai`（OTel SDK/OTLP 自持）。

**工具系统**：
- `AgentToolCatalog`：固定注册 3 个内置工具（`KnowledgeSearchTool` = search_knowledge / `MemoryFlushTool` = flush_memory / `SkillLoadTool` = load_skill）+ 按意图树挂载 MCP 工具；产出 `ResolvedCatalog` 快照（含指纹，用于判重建）
- `AgentMcpClients`：`@PostConstruct` 遍历 `agent.mcp.servers`，`McpClientBuilder.create().streamableHttpTransport(url).buildSync()` 连接并 `listTools()` 发现；重名保留先连者
- `McpToolProxy extends McpTool`：`checkPermissions` 按 needsConfirm 返回 `PermissionDecision.ask/allow`；`callAsync` 透传 McpMeta（userId 经 `_meta`）；取消→INTERRUPTED、异常→固定文案（防泄漏内部信息）
- MCP 生效判定：意图树配置 ∩ AgentScope 实际发现取交集；`needsConfirm = 意图树勾选 || 服务端未声明 readOnlyHint=true`
- **结果回注**：工具返回 `ToolResultBlock` → 框架写回 `ToolCallState` 并作为 TOOL 角色消息并入上下文 → 下一轮推理
- `AgentToolExecutionFacts`：**一次 run 的执行事实唯一来源**（批次号、callIndex、单工具真起止、返回结果、中断时刻），trace/SSE/落库三处共用，杜绝双真相

**Skills 渐进式披露三层**：工具描述里的清单（L0）→ `load_skill` 取手册正文（L1）→ 手册解锁的工具随加载放行（L2）。`AgentSkillMaskingMiddleware` 在 `onReasoning` 计算遮蔽集 **G−L**（全部技能 toolIds − 已加载）从 `input.tools()` 过滤；`McpToolProxy` 执行前复查命中则短路；**手册被上下文压缩带走时遮蔽自动复位**（解锁只认上下文里的 load_skill 结果 metadata）。

**记忆三层**（`agent/memory`）：

| 层 | 实现 | 触发/阈值（相对 contextWindowChars=1,200,000） |
|---|---|---|
| 短期裁剪 | `AgentContextTrimmer` | 超 0.5×：老工具结果正文换占位（`[历史工具结果已省略，原长 N 字符，原入参 ...]`），保留调用/id 配对；保护本轮+未闭合循环+前 2 个已完成循环；只裁 evictableTools（默认 search_knowledge） |
| 压缩摘要 | `AgentContextCompactor` + `AgentConversationSummarizer` | 超 0.8× 且末条为 USER：`findSafeCutoff` 从尾部累计保留 ≥0.2× 后取第一个用户轮起点切点；摘要以 `__compaction_summary__`（USER 角色）替换早期消息，每代覆盖式重生成；summarize 同步阻塞首字，120s 超时放弃 |
| 长期跨会话 | `AgentMemoryPipeline` + Judge + Consolidator | 两入口（FLUSH 模型触发 / BACKGROUND 轮次后异步，门槛 ≥3 条待处理消息）；claim 抢锁 → Judge 一次模型调用产出 ADD/UPDATE/DELETE/CLEAR 决策 → 短事务双校验提交；容量上限 0.005×（夹 [1500,6000]），顶限先受限合并（压到 0.75× 停）再淘汰 |

**人工确认**（`agent/confirm`）：`McpToolProxy.checkPermissions` 返回 ask → 框架挂起整批 → `RequireUserConfirmEvent` → `AgentStreamEventBridge.onRequireUserConfirm` 建 confirm 块（内部工具过滤、字段按 schema 声明序排）→ 消息 `AWAITING_CONFIRM` 落库 + SSE confirm 事件（与 finish 互斥）。恢复：`AgentConversationServiceImpl.confirmPendingTool` → runGate.acquire → 从 agent state 取最后一条 assistant 上 ToolCallState.ASKING 的 ToolUseBlock（**入参从状态取，防前端篡改**）→ 空 UserMessage + metadata[CONFIRM_RESULTS] 驱动 streamEvents 续跑。状态机：卡片 `pending→approved|denied|expired`；消息 `AWAITING_CONFIRM→NORMAL`；`streamChat` 入口用 `hasPendingConfirm` 拦截并发提问。

**状态与 Trace**：`PgAgentStateStore`（JSON 序列化 State upsert 到 PG）；`bindReleaseHooks` 保证出错/取消时补存盘；`AgentRunGate`（Redisson）会话互斥 + 单用户并发上限 5；`AgentRunHandle` 三条出口（complete/cancel/fail）只有一条收尾，优雅中断等 2s 让框架存盘。Langfuse 经 OTLP HTTP（`x-langfuse-ingestion-version: 4` 头）；span 树 `agent(根) → reasoning(轮次) ∥ tool_batch(批) → execute_tool(单工具)`，轮次与工具平级靠 `react_round_index` 归轮。

---

## 七、工程基建（framework 模块 + rag/config）

**8 个专用线程池**（`rag/config/ThreadPoolExecutorConfig`，全部 `TtlExecutors` 包装透传 ThreadLocal）：

| Bean | core/max | 队列 | 拒绝策略 | 用途 |
|---|---|---|---|---|
| ragContextExecutor | 4C/4C | SynchronousQueue | CallerRuns | 子问题并行检索 |
| ragRetrievalExecutor | 4C/4C | SynchronousQueue | CallerRuns | 通道级并行 |
| innerRetrievalExecutor | 2C/4C | LinkedBlocking(100) | CallerRuns | 内部检索 |
| intentClassifyExecutor | C/2C | SynchronousQueue | CallerRuns | 意图识别并行 |
| memorySummaryExecutor | 1/max(2,C/2) | LinkedBlocking(200) | CallerRuns | 记忆摘要 |
| modelStreamExecutor | max(2,C/2)/max(4,C) | LinkedBlocking(200) | Abort | 模型流式输出 |
| chatEntryExecutor | =globalMaxConcurrent(50) | SynchronousQueue + allowCoreThreadTimeOut | Abort | SSE 排队后入口 |
| knowledgeChunkExecutor | max(2,C/2)/max(4,C) | LinkedBlocking(200) | Abort | 文档分块 |

**上下文透传**：`UserContext`（`TransmittableThreadLocal<LoginUser>`）；`RagTraceContext` 3 个 TTL（TRACE_ID/TASK_ID/NODE_STACK），**NODE_STACK 重写 copy() 返回深拷贝**，避免父子线程共用 Deque 导致层级串挂。

**跨节点取消**（`framework/web/StreamTaskManager`）：Guava Cache（TTL 30min、max 10000）本地注册表 + Redisson 双 Bucket（`ragent:stream:cancel:{taskId}` 取消标记、`ragent:stream:owner:{taskId}` 属主）+ RTopic `ragent:stream:cancel` 广播。`cancelByUser` 校验 UserContext 属主（**taskId 是雪花可预测，不是凭证**）。`cancelLocal` 内 CAS 单次生效，先跑 cancelAction（中断上游流）再跑 finalizer（补发事件收尾）。

**SSE 封装**（`framework/web/SseEmitterSender`）：`AtomicBoolean closed` + CAS 保证 complete/completeWithError 幂等。

**幂等**（`framework/idempotent`）：`@IdempotentConsume`（防 MQ 重复消费）+ `@IdempotentSubmit`（防重复提交）；Aspect 用 Lua `SET key value NX GET PX` 三态机（CONSUMING→抛异常延迟重试 / CONSUMED→跳过 / 无值→放行）。

**MQ**（`framework/mq`）：`MessageQueueProducer#sendInTransaction`（RocketMQ 事务消息：half 消息→本地事务 Consumer→失败回滚）；`DelegatingTransactionListener` 按 txId 分发本地事务与回查。

**Trace**（`framework/trace` + `rag/aop`）：`@RagTraceNode(name,type)` + `RagTraceAspect`（`@Order(HIGHEST_PRECEDENCE+10)`，无 traceId 放行，startNode→push NODE_STACK→proceed→finishNode）；`RagStreamTraceSupport` 流式跨线程 span（beginStreamNode 在调用线程插行，detach 同步段弹出，onComplete/onError 在异步线程 CAS 收尾，`finishCancelledIfRunning` 防悬挂）。

**公平排队限流**（`rag/service/ratelimit/FairDistributedRateLimiter`）：
- 四件套：`{name}:semaphore`（RPermitExpirableSemaphore）+ `{name}:queue`（ZSET，score=队列序号）+ `{name}:queue:seq`（RAtomicLong 发号）+ `{name}:queue:notify`（RTopic 唤醒）；`{name}:entry:{requestId}` 存活标记（TTL=剩余等待+5s）
- `lua/queue_claim_atomic.lua` 原子 claim：`ZRANGE 0, maxRank+slack-1`（slack=16），**僵尸（entry 过期）ZREM 清理**；liveRank < maxRank 才 ZREM+DEL entry 返回
- Ticket 状态机 `PENDING→GRANTED/TIMED_OUT/CANCELLED`（CAS 终态互斥）；grant 先 set permitRef 再 CAS，竞争失败方负责释放；grant 把 onAcquired 包 try/finally 必释放；cleanup 幂等且 **GRANTED 态不释放 permit**（防让出使用中的 slot）；「队头无 permit」按原 score 重入队保公平；PollNotifier 用 firing CAS + 计数合并通知风暴
- 默认参数（`rag.rate-limit.global.*`）：enabled / max-concurrent=50 / max-wait-seconds=20 / lease-seconds=600 / poll-interval-ms=200

**另两处并发控制**：文档上传 `UploadRateLimitFilter`（OncePerRequestFilter，multipart 解析前拦截，tryAcquire 30s 拿不到直接 429，SemaphoreInitializer setPermits(10)）；MinerU 解析并发（RPermitExpirableSemaphore 压跨实例任务数）。

**入库 Pipeline**（`rag/ingestion`）：
- `IngestionEngine`：构造时按 getNodeType() 收注册表；execute 流程 = 建 nodeConfigMap → 环检测（沿 nextNodeId 走查遇重复抛错）→ findStartNodes（不被引用者，多于 1 个报错）→ executeChain 顺序执行；`NodeResult.fail` 置 FAILED 并 break；**无重试机制**（失败即终态，重跑靠重新提交）
- 节点接口：`IngestionNode#execute(IngestionContext, NodeConfig)`；`NodeResult{success, shouldContinue, message, error}` 工厂 `ok()/skip()/terminate()/fail()`
- `ConditionEvaluator`：JSON 条件树（all/any/not/field + eq/ne/in/contains/regex/gt/... /exists）+ 文本 SpEL（root=ctx）；先 isStructurallyValid 再求值，异常一律 false
- 节点实现：FetcherNode（SourceType 策略路由）/ ParserNode（`ParserRegistry` MIME×档位双表，Tika 自检 22 扩展名启动失败即报错；Tika/MinerU/VLM/Excel/Csv/Markdown 六个解析器）/ EnhancerNode（文档级 LLM 增强）/ ChunkerNode（chunk + embed）/ EnricherNode（分块级 LLM 加工）/ IndexerNode（写 Milvus，skipIndexerWrite 时由调用方事务统一写）
- `IngestionTaskServiceImpl`：`@Transactional` + @LogRecord 审计；每节点一行 `t_ingestion_task_node`（nodeOrder/durationMs/status，outputJson 超 1MB 截断）；终态回写
- `NodeOutputExtractor`：只产输出摘要（rawBytesLength 只记长度、chunk 只记 chunkCount/totalChars/embeddingDim）

**mcp-server 模块**（独立部署，零内部依赖）：`McpServerConfig`（HttpServletStreamableServerTransportProvider + `/mcp`，**启动校验所有工具必须声明 readOnlyHint**）；工具 = `@Component` 内 `@Bean McpServerFeatures.SyncToolSpecification`；工具有 weather_query / sales_query / ticket_query / youcom_search（YDC_API_KEY 无 Key 不注册）/ 当前日期 / OA 类 / 15 个电商写操作（MyBatis）。

---

## 八、设计模式落地总表

| 模式 | 落地类 |
|---|---|
| 策略 | SearchChannel（四通道 + @ConditionalOnProperty）、IntentClassifier、QueryRewriteService、ContextFormatter、VectorRetrieverService（Pg/Milvus 双实现 + supportsGlobalRetrieval 能力查询）、RerankService、ObjectStorageClient（S3/OSS）、ChatClient/EmbeddingClient/RerankClient 家族、ModelCaller 函数式 |
| 模板方法 | AbstractOpenAIStyleChatClient/EmbeddingClient、IngestionNode.execute 固定骨架 |
| 责任链 | SearchResultPostProcessor（order 链 + 异常隔离）、入库 nextNodeId 链 + 条件门、Agent middleware 洋葱链 |
| 流水线 | StreamChatPipeline（私有方法 + boolean 短路） |
| 工厂 | StreamCallbackFactory |
| 装饰器 | ForwardingStreamCallback、ProbeStreamBridge（缓冲型）、GraphSyncingVectorStoreService/KeywordSyncingVectorStoreService（写入同步图谱/ES） |
| 门面 | KnowledgeSearchFacade（Agent 模式检索窄口：无历史、不走来源装配）、RoutingLLMService/RoutingEmbeddingService/RoutingRerankService（@Primary） |
| 注册表 | ParserRegistry（MIME×档位）、IngestionEngine（nodeType）、AgentToolCatalog（ResolvedCatalog + 指纹）、DefaultIntentClassifier（兼 IntentNodeRegistry）、DelegatingTransactionListener（topic→TransactionChecker） |
| 状态机 | ModelHealthStore（CLOSED/OPEN/HALF_OPEN）、FairDistributedRateLimiter.Ticket（PENDING/GRANTED/TIMED_OUT/CANCELLED CAS 终态互斥）、confirm 卡片、文档定时刷新 |
| 代理 | McpToolProxy（权限检查 + _meta 透传 + 异常固定文案） |
| 快照+指纹 | ResolvedCatalog（人设+工具目录指纹判重建）、ReActAgentProvider 懒重建 |
| 单一事实源 | AgentToolExecutionFacts（trace/SSE/落库共用） |
| 观察者 | RTopic（跨节点取消广播、poller 唤醒）、StreamCallback 流式事件 |
| 空对象 | RagStreamTraceSupport.NOOP_SPAN |
| AOP | @RagTraceNode / @IdempotentConsume / @IdempotentSubmit / @LogRecord / @ChatRateLimit |

---

## 九、值得传承的工程判断

1. **引擎选择用 Bean 装配而非运行时分支**：workflow/agent 两套引擎零互相干扰，部署级决定，代码零 if-else
2. **首包探测缓冲 + 切换丢弃**：模型切换对用户零脏数据，靠的是「未提交前 buffer 住、失败永不 flush」
3. **熔断每模型独立 + 半开单名额**：`halfOpenToken` 全局序号防错释；取消不计失败（TaskCancellation 甄别）
4. **限流 Lua 原子 claim + 僵尸清理**：entry TTL 判活，队头无 permit 原位重入保公平；GRANTED 不释放 permit 防让出使用中 slot
5. **引用编号一次生成四处共用**：检索完成后一次性赋号，Prompt 角标/SSE 面板/落库/前端同源，内部 docId 不暴露给模型
6. **意图补充路固定名额**：意图判错时正确证据拼不过命中库的「表面相似」，用 ScopeQuota 配额对抗
7. **上下文刻意不注入文档标题**：防「出自《XX》」幻觉
8. **改写历史裁剪**：assistant 仅留最近 2 条 NORMAL，防实体串台
9. **摘要窗口中点 cutoff**：约覆盖一半原文窗口，重叠段滑出后再生成，避免反复重摘要
10. **确认入参从状态取**：防前端篡改工具参数
11. **fail-fast 文化**：档位配置、检索漏斗、证据闸门联动、ParserRegistry 自检全部启动期校验，不静默降质
12. **NODE_STACK 深拷贝**：父子线程共用 Deque 会层级串挂，TTL copy() 必须深拷贝
