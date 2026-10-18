# 小林面试笔记（xiaolinnote.com）站点目录

> **来源**：https://xiaolinnote.com/
> **定位**：图解 Agent+RAG+LLM 大模型面试题，让一部分人先跑赢 AI Agent 开发面试
> **抓取日期**：2026-10-18
> **知识库角色**：**AI 前沿知识域（域6）核心来源**，与 javaguide/ai（应用开发）、javaup/ai（八股）形成"原理→题库→实战"闭环
> **关联站点**：与 xiaolincoding.com 同作者（小林），是其 AI 方向新站

## 板块总览与知识域映射

| 专栏 | 站点路径 | 形式 | 对应知识域 |
|------|----------|------|------------|
| Agent 面试题 | /ai/agent/ | 30 题面试翻车现场 | 域6 |
| RAG 面试题 | /ai/rag/ | 23 题 | 域6 |
| LLM 工具调用面试题 | /ai/tools/ | 22 题 | 域6 |
| 大模型工程面试题 | /ai/llm/ | 专题题库 | 域6 |
| LangChain 框架面试题 | /ai/langchain/ | 专题题库 | 域6 |
| AI 编程面试题 | /ai/coding/ | 14 题 | 域6/4 |
| 图解 Agent | /agent/ | 万字长文图解 | 域6 |
| 图解 Claude Code | /claudecode/ | 实战+源码 | 域6/4 |
| 图解 Codex | /codex/ | 工程化+源码 | 域6/4 |

## 1. Agent 面试题（/ai/agent/）— 30 题

> 结合真实面经整理，每道题用「面试翻车现场」方式写：开头模拟真实面试对话展示踩雷，再一步步把知识点从根上讲透。目标不是背答案，而是换个角度问也不慌。

**主题覆盖**：Agent 概念与架构、Workflow 与 Agent 区别、ReAct/Plan-and-Execute/Reflection 设计范式、任务拆分、记忆机制、上下文工程、会话状态、效果评测、路径稳定性、链路性能、Multi-Agent 容错、任务幻觉、数据库访问安全、Harness、自进化、人工确认、压缩阈值、执行结果差异。

**题目结构分组**：
- 第 1-3 题：**基础概念**（Agent 是什么/与大模型本质区别/核心组件、Workflow-Agent-Tools 三者区分）
- 第 4-7 题：**设计范式**（ReAct/Plan-and-Execute/Reflection 区别与选型、复杂任务拆分）——面试高频考点
- 第 8、9、12-15、17-21、23、24 题：**工程实践**（记忆模块设计、长短期记忆存储、上下文动态装配、多轮任务恢复、效果评测、死循环与延迟治理、任务完成判定、数据库权限边界、为什么有时手搓 Agent 不用框架）
- 第 10、11、16、22 题：**Multi-Agent**（何时用多 Agent、Single vs Multi 选型、协作与动态切换、子 Agent 失联与并发冲突）
- 第 25-30 题：**运行机制与控制策略**（Harness 职责、完整运行架构、经验验证后复用、人工确认时机、压缩阈值、相同输入不同结果排查）

**完整题目清单**（30 题）：
1. 什么是 Agent？与大模型有什么本质不同？ — /ai/agent/1_whatisagent.html
2. Agent 的基本架构由哪些核心组件构成？ — /ai/agent/2_components.html
3. Workflow，Agent，Tools 这三个的概念和区别介绍一下？ — /ai/agent/3_workflow_tools.html
4. 了解哪些其他的 Agent 设计范式？Agent 和 Workflow 的区别是什么？ — /ai/agent/4_patterns.html
5. Agent 推理模式有哪些？ReAct 是啥？具体是怎么实现的？ — /ai/agent/5_react.html
6. ReAct、Plan-and-Execute、Reflection 三种范式有什么核心区别？实际项目中该如何选型？ — /ai/agent/6_three_patterns.html
7. 复杂任务怎么做的任务拆分？为什么要拆分？效果如何提升？ — /ai/agent/7_tasksplit.html
8. 请你介绍一下 AI Agent 的记忆机制，并说明在实际开发中应该如何设计记忆模块？ — /ai/agent/8_memory.html
9. Agent 的长短期记忆系统怎么做的？记忆是怎么存的？粒度是多少？怎么用的？ — /ai/agent/9_memory_storage.html
10. 什么是 Multi-Agent？ — /ai/agent/10_multiagent.html
11. 说说 Single-Agent 和 Multi-Agent 的设计方案？ — /ai/agent/11_single_multi.html
12. Agent 记忆压缩通常有哪些方法？ — /ai/agent/12_memcompress.html
13. 在工程实践中，为什么有时候选择「手搓」Agent，而不是直接用成熟框架？ — /ai/agent/13_handcode.html
14. 如何赋予 LLM 规划能力？ — /ai/agent/14_planning.html
15. 讲讲 Agent 的反思机制？为什么要用反思？具体怎么实现？ — /ai/agent/15_reflection.html
16. 如何设计多 Agent 的协作与动态切换机制？ — /ai/agent/16_collab.html
17. Agent 的上下文工程怎么设计？ — /ai/agent/17_context_engineering.html
18. Agent 的多轮对话状态如何管理？如何防止跑偏并支持中断恢复？ — /ai/agent/18_session_state.html
19. 如何评估一个 Agent 的效果？评测集和指标怎么设计？ — /ai/agent/19_evaluation.html
20. Agent 为什么会出现路径震荡、重复调用和死循环？怎么检测和治理？ — /ai/agent/20_loop_stability.html
21. 线上 Agent 延迟明显升高时，如何通过 Trace 定位和优化？ — /ai/agent/21_agent_latency_trace.html
22. Multi-Agent 系统如何处理子 Agent 超时、失联和并发修改冲突？ — /ai/agent/22_multi_agent_resilience.html
23. Agent 的「任务幻觉」是什么？如何避免没有执行工具却声称任务已经完成？ — /ai/agent/23_task_hallucination.html
24. 大模型或 Agent 连接数据库时，如何防止越权、敏感数据泄漏和查询幻觉？ — /ai/agent/24_database_security.html
25. Agent Harness 是什么？ — /ai/agent/25_harness.html
26. 如何设计一个完整的 Agent Harness？ — /ai/agent/26_harness_design.html
27. Agent 的自进化机制如何实现？ — /ai/agent/27_self_evolution.html
28. Agent 应该在什么情况下请求人工确认？ — /ai/agent/28_human_confirmation.html
29. Agent 的上下文压缩阈值应该如何确定？ — /ai/agent/29_context_compression_threshold.html
30. 相同输入为什么会得到不同的 Agent 执行结果？ — /ai/agent/30_execution_variance.html

## 2. RAG 面试题（/ai/rag/）— 23 题

> RAG 是 AI 应用标配，面试官喜欢从「RAG 是什么」一路追到「线上遇到过什么坑」。

**主题覆盖**：RAG 原理、文档切割、Embedding 选型、向量数据库、检索优化、Query 改写、多路召回、幻觉规避、知识冲突治理、长上下文与自主文件检索选型。

**题目结构分组**：
- 第 1-3 题：**基础概念**（RAG 定义与完整工作流程、解决什么问题、vs 微调优劣势）——面试开场必问
- 第 4-9 题：**索引构建**（文档存储与粒度、Chunking 策略、语义切割规避、Embedding 是什么/如何选型评估、Embedding 算法、向量数据库选型与实践）——深挖见真章
- 第 10-16 题：**检索优化**（在线工作流程、向量 vs 关键词检索、Query Rewrite、多路召回、检索优化策略、高级 RAG 范式 Self-RAG/Corrective RAG、图数据库增强场景）——最容易拉开差距
- 第 17-21 题：**生产落地**（幻觉规避、效果量化评估、知识库动态更新、最难的地方、知识冲突裁决）——认可"真跑过线上"的分水岭
- 第 22-23 题：**检索方案选型**（大窗口为什么还需要 RAG、RAG vs Agent 自主文件检索）

**完整题目清单**（23 题）：
1. 什么是 RAG？详细描述一个完整 RAG 系统的详细工作流程？ — /ai/rag/1_whatisrag.html
2. 大模型的 RAG 主要用来解决什么问题？ — /ai/rag/2_rag_problems.html
3. 相比直接微调 LLM，RAG 解决了什么问题？微调和 RAG 各自的优劣势是什么？ — /ai/rag/3_rag_vs_finetune.html
4. RAG 中的文档是怎么存的？粒度是多大？详细说说文档切割（Chunking）策略？ — /ai/rag/4_chunking.html
5. 怎么规避语义被切割掉的问题？ — /ai/rag/5_semantic_cuts.html
6. 在 RAG 中 Embedding 究竟是什么？如何选择和评估一个 Embedding 模型？ — /ai/rag/6_embedding.html
7. Embedding 有哪几种算法你了解过吗？ — /ai/rag/7_embedding_algos.html
8. 什么是向量数据库？有没有做过向量数据库的对比选型？ — /ai/rag/8_vectordb.html
9. 讲讲你用的向量数据库？数据量级是多大？性能如何？遇到过性能瓶颈吗？ — /ai/rag/9_vectordb_practice.html
10. 你使用 RAG 给大模型一个输入，系统是怎样的工作流程？ — /ai/rag/10_online_workflow.html
11. 请你介绍一下向量检索和关键词检索的区别？ — /ai/rag/11_retrieval_types.html
12. 如何润色用户的 Query（Query Rewrite）？目的是什么？ — /ai/rag/12_query_rewrite.html
13. 什么是多路召回？具体怎么做？ — /ai/rag/13_multi_retrieval.html
14. RAG 检索优化策略有哪些？ — /ai/rag/14_retrieval_opt.html
15. 了解哪些更复杂的 RAG 范式？ — /ai/rag/15_advanced_paradigms.html
16. 在什么场景下，你会选择使用图数据库来增强传统的向量检索？ — /ai/rag/16_graph_db.html
17. 如何规避 RAG 系统中大模型的幻觉？ — /ai/rag/17_hallucination.html
18. 怎么量化你的 RAG 效果？ — /ai/rag/18_evaluation.html
19. RAG 知识库如何实现动态与持续更新？ — /ai/rag/19_dynamic_update.html
20. 在实际落地中，你觉得 RAG 最难的地方是哪里？ — /ai/rag/20_hardest_parts.html
21. 不同来源的文档发生知识冲突时，RAG 应该信谁？ — /ai/rag/21_knowledge_conflict.html
22. 上下文窗口很大时为什么还需要 RAG？ — /ai/rag/22_rag_long_context.html
23. RAG 与 Agent 自主文件检索应该如何选型？ — /ai/rag/23_rag_vs_file_search.html

## 3. LLM 工具调用面试题（/ai/tools/）— 22 题

> 从 Function Calling 到 MCP 再到 A2A 完整捋清楚。理解三层架构：**FC 是语言、MCP 是工具箱、Skill 是操作手册**。

**主题覆盖**：Function Calling 原理与训练、MCP 协议架构与通信、SSE/WebSocket/WebRTC 通信协议对比、A2A 协议、Skill 概念、LLM 网关、Tool Routing、工具调用容错、CLI 封装与选型、代码执行沙箱、Skill 加载。

**题目结构分组**：
- 第 1-3 题：**Function Calling 基础**（是什么/原理、LLM 如何学会调工具、FC 能力怎么训练）——整个工具调用体系的地基
- 第 4-5 题：**MCP 协议**（是什么/核心内容、由哪几部分组成）
- 第 6-8 题：**FC 与 MCP 对比选型**（区别、场景选择、为什么有些推理模型不支持 MCP）——最容易拉开差距
- 第 9-11 题：**Agent Skill**（Skill 是什么、Skill vs MCP、FC/Skill/MCP 三者关系）。Skill 是 Anthropic 2025 年 10 月推出、12 月开源为跨平台规范的概念，把使用工具完成任务的知识和流程打包成可复用模块，与 MCP 的工具能力互补
- 第 12 题：**A2A 协议**（Google 2025 年 4 月推出后捐给 Linux 基金会，解决"Agent 连 Agent"，与 MCP 的"Agent 连工具"不同）
- 第 13-15 题：**通信协议**（MCP 通信方式、SSE vs WebSocket、WebRTC vs WebSocket 在 AI 对话场景差异）
- 第 16-18 题：**生产实践**（LLM 网关统一模型入口、Tool Routing 减少 Token 避免选错、容错：格式错误/超时/重试/熔断/副作用）
- 第 19-22 题：**工具接入与执行环境**（为什么封装成 CLI、CLI vs MCP 选择、代码执行沙箱设计、Skill 从目录发现到按需加载）

**完整题目清单**（22 题）：/ai/tools/1_function_calling.html ～ 22_skill_loading.html（题目清单见源码目录，涵盖上述分组）

## 4. 大模型工程面试题（/ai/llm/）

涵盖 Transformer 架构、Attention 优化、训练与对齐、LoRA 微调、KV Cache、MoE、Prompt 等。信息页：/ai/llm/llm_info.html

## 5. LangChain 框架面试题（/ai/langchain/）

涵盖 Chain、Tools、Memory、LangGraph、LlamaIndex、框架选型、DeepResearch 等。信息页：/ai/langchain/langchain_info.html

## 6. AI 编程面试题（/ai/coding/）— 14 题

> 简历写"熟练使用 AI 编程工具"前，先回答：**AI 写完代码以后，你怎么判断这次修改可以交付？** 测试预期从哪里来？有没有检查原来的功能？如果实现和测试都是 AI 写的，它们一起理解错了需求怎么办？

**主题覆盖**：日常让 AI 参与开发的通用方法——用 AI 写接口、改页面、修 Bug、补测试；Coding Agent 怎样搜索代码、利用执行反馈、开发者还需要核对什么。

**题目结构分组**：
- 第 1-4 题：补全、对话生成和 Agent 编程的分工；怎样描述需求、提供上下文、找到相关实现（你希望 AI 完成什么，它有没有拿到足够信息）
- 第 5-6 题：需求复杂时的推进方式——什么时候先探索效果、什么时候把规则写成 Spec、任务怎样拆到可以逐步检查
- 第 7-10 题：代码质量、实际改动审查、报错排查、业务测试覆盖（代码看着顺不代表行为正确；测试全绿也要看验证了什么）——建议结合小项目过一遍
- 第 11-14 题：长任务的推进与边界——上下文约束保留、文档同步源码、跨会话接续、AI 可操作范围（能改文件跑命令后要知道可以操作到哪里）

## 7. 图解 Agent（/agent/）— 万字长文系列

> 与「大模型面试题」逐题拆解风格不同，本专栏每篇都是一个主题的万字长文，建立体系化认知。

### Agent 概念与生态（/agent/concept/）
- AI Agent 是什么？Agent 面试题万字图解 — /agent/concept/agent.html
- OpenClaw 是什么？OpenClaw 面试题万字图解 — /agent/concept/openclaw.html

### RAG 检索增强（/agent/rag/）
- RAG 是什么？RAG 检索增强面试题万字图解 — /agent/rag/rag.html
- GraphRAG 和 LightRAG 详解：原理、对比与选型 — /agent/rag/graphrag-lightrag.html

### Agent 工程方法论（/agent/engineering/）
- Harness Engineering 是什么？AI Agent 越用越聪明的秘密 — /agent/engineering/harness-engineering.html
- Loop Engineering 是什么？AI 编程从 Prompt 到 Loop 的范式转变 — /agent/engineering/loop-engineering.html
- Loop Engineering 怎么落地？一条从 0 到 1 的上手路径 — /agent/engineering/loop_engineering_handbook.html
- Graph Engineering 是什么？一文讲透多 Agent 图式编排 — /agent/engineering/graph-engineering.html

## 8. 图解 Claude Code（/claudecode/）

万字长文图解 Claude Code 实战与源码：主循环、代码检索、上下文压缩、记忆机制、多 Agent 等。目录：/claudecode/

## 9. 图解 Codex（/codex/）

图解 Codex 工程化、使用技巧与源码架构：AGENTS.md、Skills、工具调用、上下文管理、长期记忆等。目录：/codex/

## 面试项目（与 xiaolincoding 共享）

| 项目 | 方向 | 规模 |
|------|------|------|
| DevFlow AI 项目 | 多 Agent 协作 | 25 万字文档教程+简历写法+面试题 |
| MewHelp 智能客服 | AI 应用开发 | 25 万字 |
| MewCode Agent | Agent 开发 | 40 万字 |
| 智能 OnCall Agent | AI 应用开发 | 10 万字 |

## 与其他站点 AI 内容的互补关系

| 主题 | xiaolinnote | javaguide/ai | javaup/ai |
|------|-------------|--------------|-----------|
| Agent | 30 题面试题库（翻车现场式） | 体系化教程（概念/Memory/Multi-Agent/MCP/Skills/Harness） | 5 篇八股速览 |
| RAG | 23 题面试题库 | 体系化教程（文档处理/向量库/检索优化/GraphRAG） | 1 篇速览 |
| 工具调用/MCP | 22 题（FC/MCP/A2A/Skill 三层架构） | MCP/Skills 万字详解 | Agent 与工具调用协议 |
| 大模型工程 | 大模型工程面试题专题 | LLM 运行机制/API 工程/评测 | 大模型核心概念/DeepSeek |
| AI 编程 | 14 题 | AI 编程实战指南（Claude Code 全套） | — |
| 实战项目 | 4 个 AI 面试项目 | AI 智能面试辅助平台 | Nexus Agent 等 7 个 |

**结论**：xiaolinnote 是**面试题库层**，javaguide/ai 是**原理教程层**，javaup 是**速览层**，nageoffer/ragent 是**实战案例层**。四层互补，覆盖"学原理→刷题库→看实战"完整链路。

## 抓取优先级

1. **P0**：Agent 面试题 30 题信息页（已收录）、RAG 面试题 23 题信息页（已收录）、图解 Agent 专栏目录（已收录）
2. **P1**：LLM 工具调用 22 题、AI 编程 14 题、图解 Claude Code/Codex 目录
3. **P2**：大模型工程、LangChain 面试题信息页
4. **P3**：具体题目内容（与 javaguide/ai 教程互补收录，避免重复全文）
