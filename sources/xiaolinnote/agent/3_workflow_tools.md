# Agent 面试题 3：Workflow、Agent、Tools 三个的概念和区别

> **来源站点**：小林面试笔记（xiaolinnote.com）
> **原文链接**：https://xiaolinnote.com/ai/agent/3_workflow_tools.html
> **抓取日期**：2026-10-20
> **所属知识域**：域6 性能优化与架构演进（AI 前沿）
> **板块**：Agent 面试题（30 题）· 基础概念

---

## 核心答案：一个决策维度分清三层

三者**不是同一维度的东西，而是粒度不同、可以相互嵌套的三层结构**（不是三选一，实际项目通常同时存在）：

| 层 | 定位 | 决策能力 | 执行方式 | 确定性 | 调试难度 |
|----|------|---------|---------|--------|---------|
| **Tools** | 最小能力单元（按特定格式暴露给 LLM 的函数） | **无**（只执行不决策） | 被动等待被调用 | 高 | 容易 |
| **Agent** | 完整决策系统（LLM 做大脑） | **有**（LLM 自主动态决策） | 主动循环直到完成 | 低（同输入可能不同路径） | 难 |
| **Workflow** | 上层编排框架 | **无**（开发者提前写死） | 按定义顺序执行 | 高 | 容易 |

**一句话：Tools 不做决策只执行，Agent 自己做决策，Workflow 是开发者替所有节点把决策提前写好。**

## 第一层：Tools

本质上是一个「按特定格式暴露给 LLM 的函数」——普通函数给程序员调用，Tool 给 LLM 调用，必须配 LLM 看得懂的 schema（名字、描述、参数类型）。除 schema 包装外与普通函数无本质区别。

**关键设计：工具本身没有任何决策能力，它甚至不知道自己「应该」在什么时候被使用**——这是故意的，Tools 的使命就是把具体能力封装好随时待命。比喻：瑞士军刀的刀片不会说「现在该把我翻出来」，决定拿哪个刀片的是拿着刀的那只手（Agent）。

**好的工具设计四原则**：
1. **职责单一**：一个工具只做一件事（不要把「查天气+发邮件」混在一个工具里，模型难以精确判断调用时机）；
2. **描述要精确**：「查询数据」vs「查询公司内部销售数据库，支持按日期、产品类别筛选，返回销售额和订单数」；
3. **错误信息要清晰**：「参数 city 不能为空」比「Error code 400」好得多——前者能帮助 LLM 自己修正参数重试；
4. **参数设计要简洁**：能少传就少传、有默认值就给默认值——LLM 填的参数越多出错概率越大。

## 第二层：Agent

「拿着工具、自己决定用哪个」的角色。运行方式是反复循环：**想清楚（Thought）→ 行动（Action）→ 看结果（Observation）→ 再想清楚**……直到 LLM 判断任务完成。

```python
def run_agent(user_goal: str):
    messages = [{"role": "user", "content": user_goal}]
    while True:
        response = client.messages.create(model="...", tools=tools, messages=messages)
        if response.stop_reason == "end_turn":
            return response.content[0].text   # LLM 判断任务完成
        tool_use = next(b for b in response.content if b.type == "tool_use")
        tool_result = execute_tool(tool_use.name, tool_use.input)   # 代码真正执行
        messages.append({"role": "assistant", "content": response.content})
        messages.append({"role": "user", "content": [{"type": "tool_result", "tool_use_id": tool_use.id, "content": tool_result}]})
        # 回到循环顶部，LLM 再看一遍现在的状态，做下一步决策
```

**关键点**：`while True` 会跑几次，开发者完全不知道也不需要知道——这正是 Agent 和普通代码最不一样的地方。

**停止条件（Stop Condition，生产必须）**：① LLM 主动判断完成（最理想）；② 最大循环次数（如 15 轮强制停）；③ 总 token 预算上限（防成本失控）；④ 超时机制（如 60 秒）。实际工程几种同时存在，哪个先触发用哪个。

**副作用：行为不确定**。LLM 是概率模型，同样任务两次跑可能走不同路径。灵活性和不确定性是孪生兄弟。生产中某次结果出错很难复现执行路径 → 需要详细的执行日志（每步思考过程和工具调用结果）。

## 第三层：Workflow

把整个执行流程的「骨架」写在代码里，LLM/Agent/Tools 都只是「节点」，整体走哪条路由开发者代码决定。

```python
def run_customer_service_workflow(user_query: str) -> str:
    intent = classify_intent_with_llm(user_query)  # LLM 当分类器用
    if intent == "product":                        # 分支判断是开发者写的 Python，不是 LLM 决策
        docs = search_knowledge_base(user_query)
        return generate_answer_with_llm(user_query, docs)
    elif intent == "refund":
        ...
```

Workflow 和 Agent 最核心的区别：**谁做「下一步去哪」的决策？Agent 是 LLM 自己决定，Workflow 是开发者在代码里写死。** 优点：可预测、可控、好调试（打断点逐步追，精确定位哪个节点故障）——确定性在线上系统非常珍贵。

## Agentic Workflow：生产主流

- **完全靠 Agent 自主决策**的系统很少出现在生产：行为难控制、难排查、成本易失控；
- **完全靠 Workflow 写死**又太脆：没法穷举所有情况，遇到预料外输入容易失败；
- **主流模式**：**用 Workflow 固定主流程骨架，在需要灵活判断的节点嵌入 Agent**——骨架确定（可控可调试），关键节点灵活（应对复杂）。

**Anthropic 总结的常见编排模式**：
1. **Prompt Chaining（提示链）**：大任务拆多步，前步输出作后步输入，流水线；
2. **Routing（路由）**：先 LLM 分类，再分发到不同处理分支；
3. **Parallelization（并行化）**：无依赖子任务并行执行后汇总（多维度分析场景）；
4. **Orchestrator-Workers（编排者-工人）**：中央编排者分配任务，多 Worker 各自完成独立子任务；
5. **Evaluator-Optimizer（评估者-优化者）**：一个 LLM 生成、另一个评估，不通过则反馈改进重试——把「人类审稿-修改」自动化。**注意：评估标准必须在代码里定义清楚（打分函数），不能让评估者自由发挥**。

**性能成本视角**：纯 Agent 复杂任务可能十几轮循环、每轮带完整上下文，token 线性增长；Workflow 可精确控制每节点 token 预算、不需要的上下文不传、该并行并行。**很多团队从原型（纯 Agent 快速验证）到生产都会重构成 Agentic Workflow。**

## 面试翻车点

最典型误区：把 Workflow 理解成「多个 Agent 串联」——**不对，Workflow 的节点可以是任意 LLM 调用、Tools 或 Agent，关键不是节点类型，而是控制流由谁掌握**。答题抓「谁做决策」：Tools 无决策、Agent 运行时动态决策、Workflow 决策提前写死。补一句：生产主流不是纯 Agent 而是 Agentic Workflow。
