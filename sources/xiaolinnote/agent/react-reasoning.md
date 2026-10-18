# Agent 面试题 5：推理模式有哪些？ReAct 是啥？具体是怎么实现的？

> **来源站点**：小林面试笔记（xiaolinnote.com）
> **原文链接**：https://xiaolinnote.com/ai/agent/5_react.html
> **抓取日期**：2026-10-18
> **所属知识域**：域6 性能优化与架构演进（AI 前沿）
> **板块**：Agent 面试题 · 推理模式

---

👔面试官：你做过 Agent 项目，那你说说 ReAct 是什么，跟普通的 LLM 调用有什么区别？

🙋‍♂️我：ReAct 就是让模型一边思考一边行动，调工具拿结果，然后继续思考……就是一个循环。

👔面试官：好，那你说说这个循环是谁在驱动？是模型自己在转，还是有别的机制？

🙋‍♂️我：应该是……模型自己在循环？它判断完成了就停下来？

👔面试官：不对，模型每次只输出一段文本，它不会自己「循环」。那这个循环到底是怎么跑起来的？

🙋‍♂️我：那可能是框架帮它跑的？解析输出，执行工具，再把结果传回去？

👔面试官：对了，这才是关键。模型每次只做一件事：根据历史输出下一步的 Thought 和 Action。你的代码负责检测输出、执行工具、把 Observation 填回历史，再次调用模型，这才叫 ReAct 的真正实现方式。

理解了「模型负责推理、代码负责驱动」这个分工，ReAct 就不再神秘。

## 简要回答

Agent 的推理模式用过几种。最基础的是直接输出答案，没有中间推理；CoT 是让 LLM 先把推理过程写出来再给答案，准确率更高；ReAct 是在 CoT 基础上加了「行动」，让 LLM 交替输出思考和工具调用，每次行动后再根据结果继续思考，形成一个循环。

ReAct 是目前 Agent 用得最广的模式，因为它推理过程可见，又能动态利用外部工具。

## 详细解析

### 什么是推理模式？

LLM 的工作原理是根据输入一个 token 一个 token 地往后预测。问多步推导问题时，一口气预测答案中间步骤都是隐式的，误差会在中间某个暗处悄悄累积，最终暴露在答案里。

类比心算和笔算：心算 123 × 456 可能算错；把每一步写在纸上（123 × 6 = 738、123 × 50 = 6150……），算错概率大大降低。LLM 也一样，把推导过程写出来，下一步的预测就建立在一个已经写清楚的正确基础之上。

**推理模式存在的根本原因**：通过不同方式让 LLM 把隐式的思考过程显式化，减少多步推理中的累积误差。

### CoT 是什么？

**CoT**（Chain of Thought，思维链），Wei 等 2022 年提出。核心：prompt 里加一句「让我们一步步思考」，LLM 先把推理步骤写出来再给答案。

为什么有效：LLM 输出顺序生成，先写出的推理内容进入上下文成为后续生成的依据。

两种触发方式：

- **Zero-shot CoT**：直接加「让我们一步步思考」，零成本即插即用，但格式和深度不稳定；
- **Few-shot CoT**：给几个带完整推理过程的例子让 LLM 模仿，效果更稳定，适合格式固定场景，但需准备高质量示例且占 token。

**CoT 的根本局限**：纯文字推理，**没有办法和外部世界交互**——拿不到实时数据、不能执行计算、不能访问数据库。问「现在苹果公司市值多少」只能答训练数据里的旧知识。

### ReAct 是什么？

**ReAct**（Reasoning and Acting），Yao 等 2022 年提出。核心思路：在 CoT 推理链里插入真实的「行动」。LLM 按「思考 → 行动 → 观察」循环推进：先思考当前该怎么做，调用工具获取信息或执行操作，把结果作为「观察」接收回来，再进入下一轮思考，直到判断任务完成。

**三种方案对比**：

- 纯 CoT：只能脑子推理，拿不到真实数据，无外部事实校准易幻觉；
- 纯 Act-only：直接输出工具调用序列，无推理链条连接，HotpotQA 准确率明显低于 CoT 和 ReAct，动作序列脆弱、搜到不相关内容后面全跑偏；
- **ReAct**：Thought 分析局势决定下一步，Action 落地为真实操作，Observation 带回外部反馈——推理为行动提供方向，行动为推理提供事实。

**示例循环**（查 2024 年苹果和谷歌市值谁更高）：

```text
Thought: 需要两家公司实时市值，先查苹果
Action: search
Action Input: 苹果公司 2024 年市值
Observation: 苹果约 3.5 万亿美元

Thought: 再查谷歌
Action: search
Action Input: 谷歌 2024 年市值
Observation: 谷歌约 2.1 万亿美元

Thought: 苹果 3.5 万亿 > 谷歌 2.1 万亿，差 1.4 万亿
Final Answer: 苹果约 3.5 万亿，谷歌约 2.1 万亿，苹果更高，差约 1.4 万亿
```

**实现原理**：通过 prompt 格式约束 LLM 输出结构，但**循环不是 LLM 自己在转，而是由代码驱动**。LLM 每次只输出下一步 Thought + Action，代码负责检测输出（有没有 Final Answer）、解析 Action、执行工具、把 Observation 填回历史、再次调用 LLM。

典型 ReAct prompt：

```text
你是一个 AI 助手，可以使用以下工具：
- search(query): 搜索互联网获取最新信息
- calculator(expr): 计算数学表达式

回答时请严格按照以下格式：
Thought: 你的思考过程（分析当前情况，决定下一步）
Action: 工具名称
Action Input: 工具的输入参数
Observation: （此行由系统填入工具返回的结果，你不用写）
... 以上可以重复多轮 ...
Final Answer: 当你确定可以回答时，在这里给出最终答案
```

驱动代码：

```python
def react_agent(question: str, tools: dict, max_steps: int = 10):
    prompt = build_react_prompt(question, tools)
    history = []

    for _ in range(max_steps):
        response = llm.generate(prompt + "\n".join(history))

        if "Final Answer:" in response:
            return response.split("Final Answer:")[-1].strip()

        action, action_input = parse_action(response)

        if action in tools:
            observation = tools[action](action_input)
        else:
            observation = f"工具 {action} 不存在，请选择可用工具"

        history.append(response)
        history.append(f"Observation: {observation}")

    return "超过最大步数，任务未完成"
```

整个 loop 里真正的「智能」全在 LLM 每次输出的 Thought 里。代码框架做的是：管理对话历史、执行工具、检测循环终止条件。

**现代演进**：GPT-4、Claude 3 之后原生支持 Function Calling / Tool Use，模型直接输出结构化 JSON 工具调用，不再靠解析 `Action: xxx` 文本。「思考 → 行动 → 观察」循环没变，只是「行动」从解析文本变成解析结构化 JSON，更可靠。

**ReAct 的两个坑**：

1. **循环漂移**：每一步都根据当前历史重新决策，没有全局计划约束。例：让查苹果三年营收趋势，第二步看到搜索结果提到苹果和三星竞争，第三步跑去搜三星，越走越远。步骤越多、历史越长，漂移概率越大；
2. **错误传播**：每步决策建立在前面所有步骤结果之上，某步拿到错误信息后面全被带跑，且没有内置「回头检查」机制，错误出现在早期则整条链全废。

根源相同：纯前向推理，无全局规划约束方向，无反思机制纠正错误。解法是 **Plan-and-Execute**。

### Plan-and-Execute：先规划再执行

类比「边走边问路」（ReAct）vs「先看地图再出发」（Plan-and-Execute）。

两阶段：

- **规划（Planner）**：LLM 只做规划不执行工具，输出结构化执行计划；
- **执行（Executor）**：按计划逐步执行，每步本身可用 ReAct 跑，但执行器始终知道自己在整体计划中的位置。

**动态重规划**：每执行完一步检查结果与预期是否一致，出乎意料则把已有结果和剩余步骤交回 Planner 调整后续计划——像导航遇封路自动重新规划。

**选型**：ReAct 灵活，适合任务边界不明确、需探索性获取信息的场景（开放式问答、信息搜索），代价是易漂移、token 随步骤线性增长；Plan-and-Execute 有全局视野，适合目标明确的多步复杂任务（深度研究、长文写作、多工具数据分析），代价是简单任务多一次规划调用。

**混合使用**：Plan-and-Execute 做全局规划 + ReAct 做每步执行；规划用大模型（GPT-4/Claude Opus）保质量，执行用小模型控成本——可降低 70%~90% LLM 调用成本。

## 面试总结

最容易踩的坑是以为模型自己在「循环」。核心两点：① ReAct 本质是「思考 → 行动 → 观察」循环，推理显式化 + 动态调外部工具，解决 CoT 只能纯文字推理的局限；② 循环由代码框架驱动，模型每次只输出 Thought + Action，代码负责解析、执行工具、填回 Observation、带完整历史进下一轮。

主动提两个实战局限（循环漂移、错误传播）+ Plan-and-Execute 的解法 + 大小模型混合架构，回答会很有深度。
