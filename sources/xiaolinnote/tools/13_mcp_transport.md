# LLM 工具调用面试题 13：MCP 的传输方式有哪些？stdio 和 HTTP 怎么选？

> **来源站点**：小林面试笔记（xiaolinnote.com）
> **原文链接**：https://xiaolinnote.com/ai/tools/13_mcp_transport.html
> **抓取日期**：2026-10-20
> **所属知识域**：域6 性能优化与架构演进（AI 前沿）
> **板块**：LLM 工具调用面试题（22 题）· MCP 传输

---

## 核心答案

MCP 有两种标准传输方式：

1. **stdio（标准输入输出）**：Host 把 MCP Server 当**子进程**拉起，通过管道收发 JSON-RPC 消息；
2. **Streamable HTTP**：Server 是独立 HTTP 服务，Client 走网络请求——短请求用普通 HTTP 响应，长请求/流式用 SSE 推送。

关键设计：**消息格式（JSON-RPC 2.0）与传输层解耦**——同一套消息能跑在任意传输上。

## stdio：本地子进程模式

```
Host 应用 ──stdin/stdout 管道──> MCP Server 子进程
```

- **优点**：零网络配置、低延迟、随 Host 启动消亡、天然隔离（子进程权限可控）；
- **代表场景**：Claude Desktop 本地工具（文件系统、本地脚本）；
- **硬约束**：**stdout 只能走协议消息，日志必须写 stderr**——一行日志混进 stdout 就会污染协议流导致解析崩溃；
- **局限**：只能本机，Server 无法被远程多方共享。

## Streamable HTTP：网络服务模式

```
Client ──HTTP POST──> Server（单端点 /mcp）
       <──HTTP 响应（短请求）/ SSE 流（长请求）──
```

- **优点**：Server 独立部署、多 Client 共享、可做鉴权/限流/审计、天然适配微服务和云原生；
- **代表场景**：远程工具服务、团队共享 MCP Server、跨组织协作。

## 传输层演进史（版本敏感点）

| 时间 | 形态 |
|------|------|
| 2024-11 | **HTTP + SSE 双端点**（POST 发消息 + GET SSE 收消息） |
| 2025-03 | **Streamable HTTP 单端点**，旧双端点方案 deprecated |
| 2026-07-28 | 无请求 `_meta` 字段带版本信息；移除初始化握手和 HTTP 会话要求 |

## 选型

| 维度 | stdio | Streamable HTTP |
|------|-------|-----------------|
| 部署位置 | 本机 | 远程/云端 |
| 共享性 | 一 Host 一 Server | 多 Client 共享 |
| 网络依赖 | 无 | 有 |
| 延迟 | 最低（管道） | 有网络开销 |
| 运维 | 零运维 | 需服务治理（鉴权/限流/监控） |
| 典型用户 | Claude Desktop | 生产级 Agent 平台 |

**选型一句话**：本地个人工具选 stdio（简单快）；要跨团队/跨服务复用、要统一管控，选 Streamable HTTP。同一份 Server 代码通常两种传输都支持，配置层切换。
