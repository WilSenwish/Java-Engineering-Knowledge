# LLM 工具调用面试题 10：MCP 和 Skill 有什么区别？怎么配合？

> **来源站点**：小林面试笔记（xiaolinnote.com）
> **原文链接**：https://xiaolinnote.com/ai/tools/10_mcp_vs_skill.html
> **抓取日期**：2026-10-20
> **所属知识域**：域6 性能优化与架构演进（AI 前沿）
> **板块**：LLM 工具调用面试题（22 题）· MCP vs Skill

---

## 核心答案

一句话区分：

- **MCP 是「手」**——提供工具和数据访问能力，是**原子操作粒度**的接口协议；
- **Skill 是「操作手册」**——沉淀「做某类事的正确方法和流程」，是**工作流粒度**的知识包。

MCP 解决「Agent 能碰到什么」，Skill 解决「Agent 会怎么干」。两者正交，互补而不竞争。

## 详细对比

| 维度 | MCP | Skill |
|------|-----|-------|
| 本质 | 工具/数据访问协议 | 能力包（指令+脚本+模板） |
| 粒度 | 原子操作（读文件、查数据库） | 工作流（code-review 全流程） |
| 回答的问题 | 能做什么 | 该怎么做 |
| 内容形态 | JSON-RPC 工具定义 | SKILL.md 流程指令 |
| 加载方式 | list_tools 全量/按需暴露 | 渐进式三层（目录→正文→资源） |
| 复用单位 | MCP Server（一次实现到处接） | Skill 文件夹（一次编写到处用） |
| 提供方 | 工具/平台方封装 | 团队/个人沉淀 |

## 配合场景走查：代码审查 Skill

一个「code-review Skill」的完整工作流，展示两者如何交织：

```
1. 加载 Skill：Agent 匹配到 code-review 技能，读 SKILL.md 指令
   （Skill 层：知道审查流程 = 静态检查 → 人工规则 → 安全扫描 → 汇总）
2. 拉取代码：调用 MCP git server 的 get_diff 工具
   （MCP 层：原子能力「获取 diff」）
3. 读规范：Skill 指令说「先读 references/team-conventions.md」
   （Skill 层：参考资料定义团队规范）
4. 逐文件审查：调用 MCP filesystem server 读文件
   （MCP 层：原子能力「读文件」）
5. 条件分支：发现 SQL 变更 → 按 Skill 指令触发安全扫描脚本
   （Skill 层：流程编排 + 脚本执行）
6. 汇总：调用 MCP github server 提交 review 评论
   （MCP 层：原子能力「提交评论」）
```

关键观察：**流程判断（先审什么、什么情况触发什么、用什么规范）全在 Skill；具体动手（读、写、提交）全在 MCP 工具**。

## 选型判断

- 只有 MCP 没有 Skill：Agent 有手但没脑子，每次从零摸索流程，行为不稳定；
- 只有 Skill 没有 MCP：有手册但没工具，指令落不了地（除非手册自带脚本）；
- 生产形态：**MCP 提供能力底座 + Skill 沉淀流程知识**，两者按「工具生态」和「方法论沉淀」两条线分别建设。
