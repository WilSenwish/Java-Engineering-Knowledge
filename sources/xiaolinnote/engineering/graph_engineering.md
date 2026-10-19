# Graph Engineering 是什么？一文讲透多 Agent 图式编排

> **来源站点**：小林面试笔记（xiaolinnote.com）
> **原文链接**：https://xiaolinnote.com/agent/engineering/graph-engineering.html
> **抓取日期**：2026-10-20
> **所属知识域**：域6 性能优化与架构演进（AI 前沿）
> **板块**：图解 Agent 专栏 · Agent 工程方法论（万字长文，约 6121 字）

---

## 核心答案

**Graph Engineering 按照任务之间的依赖关系，用 graph（节点 + 边）来编排整个流程：该串行的串行，该并行的并行，每一块交给一个职责明确的节点去完成。** 它不是发明新玩法，是提醒你别再把本来可以并行、分支执行的任务，强行改成全程串行执行。

背景：OpenClaw 小龙虾创始人一句「我们还在聊 loop，还是已经切到 graph 了？」引爆讨论（距其带火 loop engineering 仅 40 天），有人跟帖喊「loop engineering 已死，graph engineering 永生」。

## 一、什么是 Graph Engineering？

想想公司怎么运转：不会让一个人从头到尾包办调研、开发、测试、上线——拆分职责，每个环节一个人负责，有依赖的等上一步，没依赖的同时推进，出问题能定位到负责人。复杂任务也一样：多个步骤，有的先后依赖，有的互不影响可并行。

**与 loop 的关系**（不是淘汰）：可以把一个 loop 理解成一张最小的 graph——只有一个节点、一条边，且这条边指回自己。**loop engineering 解决单个 agent 如何反复思考、调用工具、修正结果（管节点内部）；graph engineering 解决多个执行单元如何拆分、并行、汇合、返工（管节点之间）。**

**单个 agent 长对话串行执行的四个躲不掉的麻烦**：

1. **慢**——明明后端、前端、测试能三路一起跑，也得排队
2. **没存档**——全部家当只剩一长串聊天记录，执行到第 40 步中断怎么办？要么从头重跑，要么自己钻上下文里翻
3. **难暂停**——中间很难停下来等人工审批，隔天从断点接着干
4. **难表达复杂分工**——「一个负责规划的，带三个干活的，再来一个挑刺的」，串行流程擅长回答「下一步做什么」，不擅长同时表达「谁来做、哪几个人一起做、做完在哪汇合」

症结：**它缺的不是脑子，是任务调度和流程管理能力**。这两年模型完成单个步骤已越来越可靠，真正容易翻车的是「一百个步骤怎么协作起来」——graph engineering 补的就是这层。

## 二、工作原理：节点、边、状态

### 节点（Node）——干活的那个

可以是跑完整思考循环的 agent，也可以是一段固定规则的普通代码、一次数据库查询，**甚至人也行**（审批环节就是人类节点）。好节点的标准特别朴素：**只干一件事，单独就能测，想换掉它别的节点不受牵连**。一个节点身兼五职，那就不是节点，是又一个大 loop。

### 边（Edge）——决定下一步执行哪个节点

设计一条边是在回答两个问题：下一个节点由什么决定，以及是否根据结果选择不同分支。

- **确定性边（Deterministic Edge）**：提前写进代码的固定规则决定，如「测试全部通过就进入部署」，根本用不着模型
- **模型决策边（Model-decided Edge）**：交给模型判断，如工单该派给退款节点还是投诉节点
- **固定流程 vs 条件边（Conditional Edge）**：固定流程总进同一个节点；条件边根据上一步结果分叉（测试通过→下一节点，失败→返工节点）
- **注意**：条件边和确定性/模型决策边不是并列关系——一条条件边既可以由代码判断（「测试通过就审批，失败就返工」）也可以由模型判断（「根据工单内容分到退款或投诉节点」）

**设计功夫就在分清哪条边该写死、哪条该交给模型：能写死的尽量写死，把模型的判断能力留给少数确实需要理解语义的分支。**

### 状态（State）——字段结构固定的对象

任务按 graph 定义顺序经过各节点，执行到哪、每步产出什么、消耗多少 token，总得有地方存——这就是状态。一张字段固定的任务进度表 / 一份结构化 JSON：

```json
{
  "backend": {"status": "done", "artifact": "api/export.js"},
  "frontend": {"status": "running"},
  "tests": {"status": "pending"},
  "tokens_used": 12000
}
```

谁要看进度读这个对象就行，不用钻聊天记录。工程实现时通常在一个节点执行完成后把最新状态保存下来（存一次档）。**注意：用 graph 编排不会自动存档**，还得靠运行框架或自己把状态写进数据库/文件。有了存档，中断叫「读档，重试那一个节点」——像打游戏，没有存档点死一次回出生点。

三样凑齐，问题各有解法：想并行 → 一个节点分出三条边启动三个节点（**分支并行 fan-out**），干完**汇合（fan-in）**；想暂停等审批 → 加一个等待人工确认的节点；想中断后续跑 → 读上一个存档点。状态结构化、程序能直接读，存档和返工才转得起来（条件判断读测试结果字段，知道该继续还是返工）。

## 三、上手实操

### 真的需要用 graph 编排吗？（先泼冷水）

**大部分任务按顺序执行就够了，犯不着专门用 graph。** 写脚本、修 bug、简单调研，步骤前后明确，顺序执行就行——一个人一下午能干完的任务，没必要开项目启动会。

适合 graph 编排的任务特征：
- 能拆成互相独立的子任务，且需要缩短总耗时，不能全串行
- 不同环节需要不同模型/工具/权限（干活节点可写、审查节点只读）
- 流程中间有必须人工确认的环节（上线前必须有人点头）
- 执行时间长、分支多，需要随时查看每个分支的进度、产物和 token 消耗

一条都不符合→继续串行；符合两三条→往下看。**前提：先确保每个节点都能稳定完成一件事，再谈怎么把多个节点编排到一起，顺序不能反。**

### 用 graph 编排一个开发需求

场景：给系统加数据导出功能——后端写导出接口、前端加导出按钮和进度条、测试补用例。约定好同一份 API 契约，三块任务各自开工，最后合到一起验收。串行方式：先写后端再写前端再补测试，中间哪步引入的问题继续影响后面，越到后面越乱。

Graph 编排：
1. **规划节点**（用模型）：拆成后端/前端/测试三份任务，确定三方共用的 API 契约，一起写进状态对象
2. **并行分支**：三个开发节点同时启动，各自在独立 `git worktree` 里干活，互不碰对方文件；每个节点内部就是一个标准编码 loop（领任务、写代码、自测、提交）
3. **汇合到集成节点**：合并三个分支，跑全量测试
4. **条件边（整个设计最妙的地方）**：测试全过→继续往下；失败→看哪个模块用例没过，把状态对象里对应任务标记返工，交回那个开发节点重做——**只有出问题的那一路返工，另外两路不重跑，直接保留已有结果**（因为每个节点完成后都保存过状态）
5. **人工确认节点**：不执行自动化任务，暂停流程等你本人确认——你看一眼 diff 点个头，graph 才走完最后一条边合入主干

整个过程状态对象持续记录三份任务、每路进度和产物路径、测试结果、token 消耗。任何时刻想知道进展看状态对象，不用翻聊天记录。

### 用 LangGraph 怎么写？

```python
from langgraph.graph import StateGraph, START, END

g = StateGraph(DevState)

# 注册节点
g.add_node("plan", plan_task)
g.add_node("backend", backend_dev)
g.add_node("frontend", frontend_dev)
g.add_node("tests", test_dev)
g.add_node("integrate", integrate_code)
g.add_node("rework", rework_code)
g.add_node("approval", human_approval)

# 从规划节点开始
g.add_edge(START, "plan")

# 规划完成后，三个节点并行执行
g.add_edge("plan", "backend")
g.add_edge("plan", "frontend")
g.add_edge("plan", "tests")

# 三个节点都完成后，再进入集成节点
g.add_edge(
    ["backend", "frontend", "tests"],
    "integrate"
)

# 根据测试结果决定下一步
g.add_conditional_edges(
    "integrate",
    check_tests,
    {
        "pass": "approval",
        "fail": "rework"
    }
)

# 返工完成后重新集成
g.add_edge("rework", "integrate")

# 人工确认后结束
g.add_edge("approval", END)

app = g.compile()
```

要点：`StateGraph(DevState)` 创建 graph，每个节点可读可写状态；`add_edge(["backend", "frontend", "tests"], "integrate")` 表示必须等三个节点都完成才进集成（**fan-in**）；`add_conditional_edges` 读测试结果分流；`compile()` 组装成可执行的 graph。

### 用 Claude Code 怎么玩？

不想学 LangGraph 也能实操 graph engineering——Claude Code 提供 subagent、hook 和 workflow 组件：

- **一次 subagent 执行 = 一个节点**：`.claude/agents/` 目录下一个 md 文件定义一个专职 agent（写清职责、能用哪些工具、用哪个模型）——干活的给写权限，审查的只给只读
- **hook**：特定时机强制固定检查（如 agent 结束任务前自动跑测试，不过就继续改）
- **workflow（新版内置编排能力）**：一段负责拆任务、派 agent、等结果、处理分支的调度脚本——**脚本不用你动手**，对话框里把目标讲清楚即可：

> 用 workflow 帮我开发一个记账网站，拆成后端接口、前端页面、测试用例三块并行开发，做完合并跑全量测试，测试全部通过后叫我确认

实测流程：接到后先讲清打算怎么拆，主动写一份 API 契约让三路 agent 按同一份开发（免得合并时各说各话）→ 展示编排方案等确认（两阶段：三 agent 并行开发 → 集成测试）→ 后台运行，`/workflows` 随时看进度树（三路各自跑到哪一步、消耗多少 token）→ 42 个测试用例全过后喊验收 → 回复「没问题」交付完成（最后一步就是人类节点，确认后才走完最后一条边合入主干）。

`/deep-research` 本身就是已投入使用的 graph 工作流：拆解问题 → 并行搜索 → 抓原文 → 交叉验证 → 合成带引用报告。实测拆成五阶段，搜索那步又拆 5 个角度并行（国内头部、海外三巨头、其他海外玩家等），`/workflows` 里能亲眼看到一个节点分出多条边的样子。

两个例子的共同点：**主会话只负责拆解任务、启动 subagent、等待结果、最终汇总，不再亲自完成中间每一步**。具体工作在 subagent 的独立上下文进行，大量中间推理/工具输出/临时结果不需要塞回主会话，只返回后续流程真正用得上的结果。**graph 编排不仅让任务跑得更快，也把调度过程和具体执行过程分开了。**

## 四、Graph Engineering vs LangGraph

| | Graph Engineering | LangGraph |
|---|---|---|
| 层级 | 设计**方法** | 实现框架 |
| 关注 | 任务怎么拆分、哪些可并行、下一步由什么决定、状态怎么记录、失败怎么返工 | 提供现成的节点、边、状态和流程控制能力 |

做 graph engineering 不一定用 LangGraph——AutoGen、Google ADK、Claude Code 的 workflow、甚至自己写调度代码都行，只要在按节点/边/状态组织多个执行单元的协作，本质就是在做 graph engineering。**graph engineering 告诉你应该怎么设计，LangGraph 帮你实现出来。**（这套技术不是最近才出现，新的是名字和注意力转向；LangChain 官方还发博客调侃 graph engineering 是「X 的 AI 内容工厂」刚造的新词——他们用 LangGraph 做了三年了。）

## 五、总结

- graph engineering = 把复杂任务拆成多个职责明确的节点，按依赖关系安排串行、并行、条件分支和失败返工；**节点干活，边决定下一个执行谁，状态记录进度和结果**
- 不是所有任务都值得用 graph：任务小、按顺序能完成的，直接一个 agent 跑完；只有可拆并行分支、需要人工确认、中断后要继续、需长期追踪进度产物时，价值才真正体现
- 真正值得关注的不是新名字，而是背后的问题：**随着 agent 能完成的任务越来越复杂，要解决的不只是「这一步怎么做对」，而是「几十个步骤怎么稳定地协作起来」**
- 名字是新的，问题不是新的——就算过几个月又被新词替代，节点、边、状态和任务调度这些东西不会消失

## 参考资料

- We Are Entering the Graph Engineering Phase：https://www.drjoshcsimmons.com/writing/we-are-entering-the-graph-engineering-phase
- 3 Years of Graph Engineering with LangGraph（LangChain 官方）：https://www.langchain.com/blog/3-years-of-graph-engineering-with-langgraph
- Is Graph Engineering Real?（Turing Post）：https://www.turingpost.com/p/is-graph-engineering-real-why-everyone-is-talking-about-it
