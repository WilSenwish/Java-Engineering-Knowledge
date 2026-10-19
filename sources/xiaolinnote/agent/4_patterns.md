# Agent 面试题 4：了解哪些其他的 Agent 设计范式？Agent 和 Workflow 的区别是什么？

> **来源站点**：小林面试笔记（xiaolinnote.com）
> **原文链接**：https://xiaolinnote.com/ai/agent/4_patterns.html
> **抓取日期**：2026-10-20
> **所属知识域**：域6 性能优化与架构演进（AI 前沿）
> **板块**：Agent 面试题（30 题）· 设计范式

---

## 核心答案

**Agent 和 Workflow 最核心的区别是「谁来决定下一步」**：Workflow 提前把流程写死，确定性高、好控制；Agent 让 LLM 自己决定下一步，灵活但不可控。

常见设计范式除纯 Agent 外：**ReAct、Plan-and-Execute、Reflection**。工程里用得最多的是把两者混用——固定流程用 Workflow，需要灵活决策的节点嵌入 Agent。

## Workflow vs Agent（代码结构对比）

```python
# Workflow 风格：流程固定，每步确定，LLM 只是工具
def workflow_answer_question(user_query: str):
    docs = vector_db.search(user_query, top_k=5)   # 固定向量检索
    reranked = reranker.rank(user_query, docs)      # 固定 rerank
    answer = llm.generate(user_query, context=reranked)  # 固定生成
    return answer

# Agent 风格：流程不固定，LLM 运行时动态决定每一步
def agent_answer_question(user_query: str):
    while True:
        action = llm.decide(user_query, history=memory)
        if action.type == "search":
            result = vector_db.search(action.query); memory.append(result)
        elif action.type == "calculate":
            result = calculator.run(action.expr); memory.append(result)
        elif action.type == "final_answer":
            return action.content
```

Workflow 每一行都是明确指令，控制流完全由代码决定；Agent 的 loop 里只有 `llm.decide()`，所有路径都是 LLM 运行时选的。**Workflow 是「开发者在驾驶」，Agent 是「LLM 在驾驶，开发者在副驾驶设安全限制」。**

## 三种设计范式

### ReAct（Reasoning + Acting）

推理与行动交替：**Thought → Action → Observation** 循环。
- Thought 阶段先把当前情况分析、推理过程写出来（「用户想查竞品信息，我应该先搜竞品 A 最新动态」）；
- Action 阶段决定调哪个工具、传什么参数；
- Observation 阶段工具结果反馈给 LLM，进入下一轮 Thought。
- **为什么显式分三步**：直接输出行动容易「冲动决策」（没搞清楚用户要什么就急着调工具）；Thought 让模型先理清问题，决策质量更稳，且思考内容可见、便于调试（出了问题直接看哪一步想歪了）。
- **短板**：走一步看一步，每步局部最优，处理需要全局规划的复杂任务容易中间迷失方向（十几步任务做到第五步忘了最初目标，或在几个工具间打转）。

### Plan-and-Execute

针对 ReAct「长任务跑偏」的优化：把规划和执行彻底分开——先让一个 LLM 专门规划、输出完整步骤列表，再由另一个 LLM（或同模型不同角色）逐步执行。

- 好处：复杂任务整体结构清晰，**执行前可人工审核计划**；
- **关键机制：动态重规划**。成熟实现里每执行完一步把结果反馈给规划器，判断「当前结果和预期一致吗？后续计划还适用吗？」。如原计划「搜竞品 A → 搜竞品 B → 对比分析」，执行第一步发现竞品 A 刚发布重大更新，规划器可能动态插入「搜竞品 A 最新更新详情」。**计划是活的、根据执行结果动态调整**；
- 缺点：多规划和重规划调用，延迟成本增加；初始规划方向错了后面难挽回。

### Reflection（反思）

在前两种范式基础上加「质量保障」：完成一步或整个任务后，让 LLM 判断做得好不好、是否符合预期，不通过则重试或换策略。

- 适合代码生成、文案写作这类「质量要求高但一次做对难」的场景；
- **变体 Reflexion**：不只是「结果不好重做一遍」，而是生成具体「反思总结」记录失败原因和改进建议，作为额外上下文传给下一次尝试——「写错题本」。**效果数据：HumanEval 代码基准上 GPT-4 直接做约 80%，加 Reflexion 后 91%**；
- 代价：每加一轮反思多一次甚至多次 LLM 调用，token 和延迟增加；无轮次限制易陷「为了改而改」死循环。

**三范式定位（面试核心）**：三者解决不同层次问题——ReAct 解决单步灵活性，Plan-and-Execute 解决长任务跑偏，Reflection 解决输出质量。**Reflection 不是独立流程，是给前两者加的「检查 buff」，本身不能单独成立。**

## 生产环境首选：Agentic Workflow

**Anthropic 原则：能用 Workflow 解决的问题，就不要用 Agent。** 生产环境可控性比灵活性更重要——Workflow 行为确定、问题能精确定位到节点；Agent 行为概率性、测试覆盖率天然低。**从最简单 Workflow 开始，发现某节点确实需要灵活决策时才升级成 Agent**——「从简单到复杂、按需升级」。

## 面试翻车点

1. **设计范式不熟**：ReAct 之外必须能说出 Plan-and-Execute（规划执行解耦）和 Reflection（执行后自我评估）；
2. **把 Reflection 当调试手段**：它是正式运行时机制，内嵌执行流程，代价是 token 和延迟；
3. **以为 Agent 是生产首选**：纯 Agent 生产里很少用（不确定、难调试、成本易失控）——工程答案是 Agentic Workflow。能主动说出「为什么纯 Agent 在生产里有局限」是拿高分关键。
