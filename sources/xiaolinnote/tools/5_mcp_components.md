# LLM 工具调用面试题 5：MCP 由哪几部分组成？

> **来源站点**：小林面试笔记（xiaolinnote.com）
> **原文链接**：https://xiaolinnote.com/ai/tools/5_mcp_components.html
> **抓取日期**：2026-10-20
> **所属知识域**：域6 性能优化与架构演进（AI 前沿）
> **板块**：LLM 工具调用面试题（22 题）· MCP 协议

---

## 核心答案

三层组成：**角色层**（Host/Client/Server）+ **能力层**（Tools/Resources/Prompts）+ **协议层**（JSON-RPC 2.0 + stdio/Streamable HTTP）。三层各自独立互不耦合。最大误区是把 MCP 理解成「Client + Server」二元结构漏掉 Host，以及把 Server 暴露的全归为「工具」。

## 角色层：Host / Client / Server

- **Host**：宿主 AI 应用（Claude Desktop/Cursor/Windsurf），启动管理所有 Client，控制连哪些 Server、何时断开——MCP 系统调度中心；
- **Client**：Host 内部连接模块，**一个 Client 对应一个 Server 连接**，负责初始化连接、能力发现（查询有哪些工具/资源/模板）、转发调用请求和带回结果——「驻场联络员」；
- **Server**：工具提供方的独立进程，按协议响应请求，完全不关心哪个 Host 在用——**Server 写一次，任何支持 MCP 的 Host 都能直接用**（GitHub 官方 Server 不用为 Claude Desktop 和 Cursor 各写一份）。

## 能力层：Tools / Resources / Prompts

- **Tools**：可调用功能，**有副作用改变外部状态**（创建文件/提交代码/发 Slack/调 API），模型主动触发，通常需授权确认。对应 FC 里「函数」的概念，在 MCP 框架下被标准化打包；
- **Resources**：只读数据无副作用（读日志/查数据库/获取文档），URI 标识，宽松暴露不需谨慎授权——「工具的资料室」；
- **Prompts**：预定义提示词模板带参数占位符，解决「每次手写重复 prompt」——团队最佳实践模板化复用（代码审查标准封装成 review_code(language, code)）。

## 协议层：消息格式与传输解耦

消息格式 JSON-RPC 2.0（易读易调试任何语言可实现，Python/TS Server 消息格式一样）；传输两种：**stdio**（本地子进程，管道通信，无需网络延迟极低无端口问题——Claude Desktop 接本地 Server 走此方式）+ **Streamable HTTP**（远程独立部署，多 Client 共享——团队共用服务器上的数据库 Server）。

演进：2024-11-05 双端点 HTTP+SSE → 2025-03 改单端点 Streamable HTTP（老方案 deprecated 保留兼容）——**不是抛弃 SSE，是双端点合并成一个 /mcp**：POST 发请求，短请求回普通 JSON，长请求升级 SSE 流持续推中间结果，对负载均衡和 serverless 更友好。

**消息格式和传输方式解耦**：同一套 JSON-RPC 消息可跑在任意传输层上——这让 Server 既可轻量本地进程运行也可作正式微服务部署，协议层始终一致。
