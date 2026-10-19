# LLM 工具调用面试题 6：MCP 和 Function Calling 有什么区别？有没有实际跑过 MCP？

> **来源站点**：小林面试笔记（xiaolinnote.com）
> **原文链接**：https://xiaolinnote.com/ai/tools/6_mcp_vs_fc.html
> **抓取日期**：2026-10-20
> **所属知识域**：域6 性能优化与架构演进（AI 前沿）
> **板块**：LLM 工具调用面试题（22 题）· FC vs MCP

---

## 核心答案

**不是竞争关系，是不同层次**。FC 是「调用语言」——定义模型怎么表达「调哪个函数、参数是什么」；MCP 是「工具生态协议」——定义工具怎么标准化打包、注册和被客户端发现。类比：FC 像 HTTP 请求格式，MCP 像 REST 规范 + 服务注册发现。**MCP 底层依然靠 FC 驱动**：Client 连上 Server 后 `list_tools` 拉工具定义，转换成模型原生 FC 格式传给模型，模型输出 tool_calls，Client 路由到 Server 执行——**模型完全感知不到 MCP 的存在**，所有魔法（自动发现/schema 转换/调用路由）发生在宿主程序层。如果模型不支持 FC，MCP 完全没法用。

## FC 的痛点：每次都是一次性的

5 个应用 × 8 个工具 = 40 份工具对接代码在维护。GitHub API 字段变了要在 5 处同步改，漏一个凌晨报警；从 Claude 迁 GPT-4，40 份代码的 FC 格式全要重新适配。**同一个工具换个项目重新对接一遍**——FC 没有任何工具管理、发现、跨项目复用的概念。

## MCP 的改变

工具提供方实现独立 MCP Server 进程暴露标准接口，任何支持 MCP 的客户端连上自动发现使用，不用写对接代码——工具实现一次到处复用。

## 实操：接入 MCP Server

Claude Desktop 配置文件加几行：

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/Users/yourname/Documents"]
    }
  }
}
```

重启后自动启动 Server 子进程、自动发现工具，直接问「帮我读 Documents 里的 report.md」即可，零对接代码。

自己写 Server 核心三步（MCP Python SDK，约 30 行）：`@app.list_tools()` 声明工具及参数 schema → `@app.call_tool()` 实现执行逻辑返回 TextContent → `stdio_server()` 运行让 Client 通过管道通信。

## 选型

临时接一两个工具 → FC（简单直接不需额外进程协议）；工具多/跨项目复用/用社区现成 Server（GitHub、数据库、浏览器自动化）→ MCP；**做 Agent 系统更应考虑 MCP**——工具来源杂数量多，手写 FC 维护会让工具定义代码散落各处，MCP 的自动发现让架构干净，新增工具不改主程序逻辑。
