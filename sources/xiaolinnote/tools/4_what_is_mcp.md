# LLM 工具调用面试题 4：什么是 MCP（模型上下文协议）？讲讲它的核心内容？

> **来源站点**：小林面试笔记（xiaolinnote.com）
> **原文链接**：https://xiaolinnote.com/ai/tools/4_what_is_mcp.html
> **抓取日期**：2026-10-20
> **所属知识域**：域6 性能优化与架构演进（AI 前沿）
> **板块**：LLM 工具调用面试题（22 题）· MCP 协议

---

## 核心答案

MCP（Model Context Protocol，Anthropic 2024 年 11 月发布）= 连接 AI 应用与外部工具/数据的**开放协议**。统一能力发现和调用方式，让兼容 AI 应用复用同一个 MCP Server——「M×N 变成 M+N」：M 个应用 N 个服务原需 M×N 套专用适配，统一协议后各实现协议支持即可复用（减少的是重复适配，不是连接数）。

## 三角色

- **Host**：AI 应用本体（Claude Desktop/Cursor），负责模型交互、上下文管理、权限控制；
- **Client**：Host 内部通信组件，**每个 Client 对接一个 Server**（Host 管多个 Client）；
- **Server**：工具提供方的独立进程，提供并执行能力，不关心上面是哪个 Host。

模型不直接建连接——模型输出调用意图，Host 根据配置和授权决定如何执行，Client 传递消息，Server 处理业务。

## 三类服务端能力

| 能力 | 用途 | 方法 | 例 |
|------|------|------|-----|
| Tools | 可调用的功能（**可只读也可写入，不能按副作用分类**——搜索代码也是只读 Tool） | tools/list、tools/call | search_code、create_issue |
| Resources | 上下文数据（URI 标识） | resources/list、resources/read | repo://demo/README.md |
| Prompts | 可复用消息模板 | prompts/list、prompts/get | review_code(language, code) |

记忆法：**Tools 改变世界，Resources 观察世界，Prompts 结构化表达**。同一份 README 既可作 Resource 也可用 read_file Tool 读——内容相同接口职责不同。

## 一次调用链路

Host 经 Client `tools/list` 拉工具定义 → 转成模型原生 Function Calling 格式 → 模型输出调用请求 → **Host 检查授权** → Client 发 `tools/call` → Server 调 GitHub API → 结果回 Host → 模型生成回答。**FC 表达调用意图，MCP 规范发现和调用过程，Host 负责两边格式衔接**——MCP 不规定模型如何推理，不保证选对工具。

## 消息与传输

JSON-RPC 2.0：请求（带 id）/ 响应（同 id 匹配）/ 通知（无 id）。失败分层：协议错误返回 JSON-RPC `error`；业务失败在 result 里 `isError: true`。**`tools/call` 是协议方法，`search_issues` 是具体工具**。

传输两种：**stdio**（本地子进程，stdin/stdout 传消息——stdout 留给协议消息，日志写 stderr 否则会被误当协议消息）；**Streamable HTTP**（Server 独立 HTTP 服务，POST 发请求，可返回单个 JSON 或 SSE 流；2025-03 规范把早期双端点 HTTP+SSE 改为单端点，老的标记 deprecated）。

版本演进：2024-11-05（双端点 HTTP+SSE）→ 2025-03-26~2025-11-25（Streamable HTTP + 初始化握手 + 协议级会话）→ **2026-07-28（每请求 `_meta` 带版本和能力，移除握手和 HTTP 会话，通知改 `subscriptions/listen` 的 SSE 流）**。

## 安全

授权围绕具体操作和数据设计，不能简单规定「Tools 都高风险、Resources 都开放」；**Server 返回的文档/工具说明/提示模板也可能含不可信内容**——Host 不能因为来自 MCP Server 就当成高优先级指令照办（提示词注入面）；支持 MCP 不代表支持全部能力；工具和资源多时要控制上下文开销按任务选择。
