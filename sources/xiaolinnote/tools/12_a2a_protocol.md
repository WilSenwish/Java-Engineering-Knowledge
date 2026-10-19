# LLM 工具调用面试题 12：什么是 A2A 协议（Agent2Agent）？

> **来源站点**：小林面试笔记（xiaolinnote.com）
> **原文链接**：https://xiaolinnote.com/ai/tools/12_a2a_protocol.html
> **抓取日期**：2026-10-20
> **所属知识域**：域6 性能优化与架构演进（AI 前沿）
> **板块**：LLM 工具调用面试题（22 题）· A2A

---

## 核心答案

**A2A（Agent2Agent）协议：让不同厂商、不同框架的 Agent 之间能互相通信和协作的开放协议**。由 Google 提出，后捐赠给 Linux 基金会。

与 MCP 的关系：**一纵一横，互补**——

| | MCP | A2A |
|--|-----|-----|
| 连接对象 | Agent ↔ 工具（纵向） | Agent ↔ Agent（横向） |
| 解决问题 | Agent 怎么用工具 | Agent 之间怎么分工协作 |
| 类比 | 微服务的 API 网关 | 微服务之间的服务调用 |

MCP 让单个 Agent 长出「手」，A2A 让多个 Agent 组成「团队」。

## 核心机制一：Agent Card（智能体名片）

每个 A2A Agent 在固定路径暴露一张名片：

```
https://agent-b.example.com/.well-known/agent-card.json
```

名片内容：

- **能力描述**：这个 Agent 会什么（技能列表、输入输出格式）；
- **服务端点**：任务提交地址；
- **认证方式**：怎么鉴权。

协作方先拉名片 → 发现「这个 Agent 有财报分析技能」→ 把对应子任务派过去。**本质是服务发现**——Agent 世界的 DNS + 服务注册中心。

## 核心机制二：Task（任务）

A2A 以 Task 为基本单位：

- **异步长任务**：提交任务后不用干等，对方后台执行，可随时查状态、可取消；
- **Task 状态机**：

```
submitted → working → completed
                ↘ failed / canceled
```

- **产物（Artifact）**：任务产出（文档、数据、图表）随状态回传；
- **消息（Message）**：协作过程中可追加上下文、追问澄清。

## 本质：Agent 微服务化

A2A 把 Agent 当作**微服务**来治理：

| 微服务概念 | A2A 对应 |
|-----------|---------|
| 服务注册与发现 | Agent Card |
| API 契约 | Skill 声明（输入/输出 schema） |
| 异步任务队列 | Task + 状态机 |
| 服务间调用 | message send |
| 服务治理（超时/重试/熔断） | 由协作方按 Task 状态实现 |

面试一句话：**「A2A 是 Agent 间的 HTTP，让 Agent 从单体走向分布式协作；配合 MCP 的纵向工具连接，构成完整的 Agent 互联网协议栈」**。
