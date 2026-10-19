# Harness Engineering 是什么？AI Agent 越用越聪明的秘密

> **来源站点**：小林面试笔记（xiaolinnote.com）
> **原文链接**：https://xiaolinnote.com/agent/engineering/harness-engineering.html
> **抓取日期**：2026-10-20
> **所属知识域**：域6 性能优化与架构演进（AI 前沿）
> **板块**：图解 Agent 专栏 · Agent 工程方法论（万字长文，约 16359 字）

---

## 核心答案

**Agent = Model + Harness**。在一个 AI Agent 系统里，除了模型本身之外，几乎所有决定它能不能稳定交付的东西，都属于 Harness（马具/驾驭工程）。Harness Engineering 解决的是「怎么让模型在真实执行里持续做对一连串的事」——前两代工程（Prompt/Context）都停留在输入侧，Harness 是对「整个运行系统」的工程化。

## 一、这个词从哪儿冒出来的？

### 1.1 溯源：Mitchell Hashimoto

2026 年 2 月 5 日，Mitchell Hashimoto（HashiCorp 联合创始人）博客《My AI Adoption Journey》正式提出 **Engineer the Harness**。其 6 步采纳 AI 的过程中三个关键动作：

1. **直接扔掉对话框**——聊天界面给人「我在和 AI 讨论」的幻觉，实际上啥也没干；要用就用能自主执行任务的 Agent，只有放开让它自己执行，才能真正感知能力边界
2. **强迫自己用 Agent 重做已手写过的代码**——过程 painful，但只有亲自用 AI 重做熟得不能再熟的任务，才能精准校准「它到底哪里行、哪里不行」
3. **给做法命名 Engineer the Harness**——核心定义：

> **每次当你发现 Agent 犯了一个错误，就花点时间去工程化一个解决方案，让它永远不会再犯同样的错误。**

修补必须沉淀到环境里（AGENTS.md 加规则、加 linter、补自动化测试、Git Hook），而不是留在人脑子里。威力在于**复利**：每次犯错环境变强一点 → 下次更少犯错 → 改进速度更快 → Harness 越来越坚固（复利飞轮）。

### 1.2 OpenAI 发文背书

一周后 OpenAI 官方博客《Harness engineering: leveraging Codex in an agent-first world》：内部 **3 人小团队**、从空仓库出发、**5 个月**、靠 Agent 写出 **100 万行代码**、合并 **1500 个 PR**、**全程没人手动写过一行代码**。走红路径：基础设施圈老法师先喊出来 → OpenAI 几天后发文背书 → 一周内 AI 圈刷屏。这种出身决定了它是在真实工程土壤里长出来的，不是炒一波就凉的概念。

## 二、从提示词到 Harness：三次重心迁移

| 阶段 | 解决的问题 |
|---|---|
| Prompt Engineering | 怎么让模型「**听懂**」你想干啥 |
| Context Engineering | 怎么让模型「**知道**」该用什么信息 |
| Harness Engineering | 怎么让模型在真实执行里「**持续做对**」一连串的事 |

### 2.1 Prompt Engineering：让模型「听懂」

- 大模型本质：**一个特别大的参数文件**，根据输入**预测下一个字**（是「预测」不是「思考」）；输入越模糊猜的范围越大，越具体越收敛
- 写提示词的本质不是「命令」模型，而是**塑造它的概率空间**（角色/示例/约束都在悄悄把输出往某个方向推）
- 提示词配方六要素：角色设定、背景信息、参考资料、明确任务、约束条件、输出格式
- **天花板**：提示词擅长「表达」，不擅长「信息」——没看过你的财报/规范/API，写得再漂亮也替代不了「事实」本身；也不擅长长链路任务维持稳定状态

### 2.2 Context Engineering：让模型「知道」

- Agent 从「回答问题」到「进真实环境干活」，系统面对的不再是「这一次回答对不对」，而是「整条链路能不能跑通」
- **Context（上下文）= 每一次发给大模型的所有信息加在一起**（用户输入、历史对话、检索资料、工具返回、任务状态、中间产物、系统规则、安全约束）
- 核心思想：**模型未必知道，所以系统必须在合适的时机，把正确的信息送进去**
- 两个现实约束：上下文窗口有限；且塞得太满会出现「**上下文腐化**」（Context Rot）——记不住前面、前后矛盾、忽略最初规则
- 三步实践：**召回**（RAG 检索）→ **压缩**（摘要化）→ **组装**（重要指令放靠后位置，模型对靠后信息更敏感）
- Anthropic Agent Skills 的「**渐进式披露**」（Progressive Disclosure）：先给目录，判断需要时才加载详细说明——上下文优化的本质不是「给得更多」，而是「按需给、分层给、在正确的时机给」

### 2.3 Harness Engineering：让模型「做对」一连串的事

- 长链路新问题：计划很好但执行跑偏；工具调对了但理解错返回；悄悄偏离初衷而系统没察觉；跑着跑着忘了最初要干啥
- **谁来监督它？谁来约束它？谁在它跑偏时把它拉回来？** ——前两代都在优化「输入侧」，这是全新问题
- **Harness 词源**：马具/缰绳。马本身有强大力量，但没有缰绳就是失控的；马具让这股力量为你所用
- **销售经理派新人**比喻：讲清楚流程 = Prompt；资料准备齐全 = Context； checklist + 实时汇报 + 录音复盘 + 跑偏电话纠正 + 按标准验收 = **Harness**
- **核心等式**：`Agent = Model + Harness`，反过来 `Harness = Agent − Model`
- **包含关系**（非替代）：Prompt 是对「指令」的工程化，Context 是对「输入环境」的工程化，Harness 是对「整个运行系统」的工程化。Prompt 是 Context 的一部分，Context 是 Harness 的一部分

## 三、Harness 六层结构

OpenAI、Anthropic、LangChain 产品形态各异，但 Harness 内部结构惊人相似——「让 Agent 在真实世界稳定工作」这个命题天然推着所有人收敛。**六层按三组划分**：

| 分组 | 目标 | 包含层 |
|---|---|---|
| 输入侧 | 让模型看到正确的东西 | 上下文精细化管理 + 记忆与状态管理 |
| 动作侧 | 让模型做出正确的事 | 工具系统 + 任务执行编排 |
| 校验侧 | 让模型知道做没做对 + 出错能爬起来 | 评估观测 + 约束恢复 |

对应工程师干活的三个必要条件：**看得准 → 做得对 → 错了能兜底**。

### 六问对应表

| 层 | 这一层在解决的一句话问题 |
|---|---|
| 上下文精细化 | 模型**这一轮**该看到什么？ |
| 工具系统 | 模型**用什么**动手？ |
| 执行编排 | 模型**下一步**该干啥？ |
| 记忆与状态 | 模型**跨轮**该记住什么？ |
| 评估与观测 | 模型**做得好不好**有没有尺子？ |
| 约束与恢复 | 模型**出错了**能不能爬起来？ |

### 贯穿例子：PR Review Agent（每天定时扫 GitHub 关注仓库的新 PR，挑值得关注的生成摘要点评，发到 Slack）

**第一层 上下文的精细化管理**：管「空间」（这一轮上下文长啥样），与第四层管「时间」区分——像相机的「取景框」vs「胶卷」。三个核心工作：① 把角色和目标钉死（大部分 Agent 跑偏根源就是这步没说清）② 动态筛选而不是一次塞满（Anthropic 解法 **just-in-time retrieval**，边干活边按需抓信息）③ 结构化组织（固定规则/动态证据/中间结论三者分开，否则模型「自我污染」——用错的中间结论影响后面判断）。**context rot**：塞越多无关信息注意力越散。

**第二层 工具系统的可控调用**：三个问题——给它哪些工具（OpenAI Codex 早期接一堆工具结果频繁用错，砍掉一大半效果反而上升）；什么时候用哪个（该查才查，diff 已在上下文里重复拉就是浪费）；**工具结果怎么喂回模型**（最易忽略——30 条匹配先提炼再喂，否则上下文又被污染）。MCP 本质就是工具层标准化。

**第三层 任务执行的全局编排**：**Agent 的本质就是一个 for 循环**：思考一步 → 行动一步 → 观察结果 → 再思考下一步，即 **ReAct**（Reasoning + Acting）。翻车场景：每步都会做，串起来就不会——第三层给模型一条明确的工作轨道。常见编排模式：Plan-and-Execute（长链路）、Reflexion（失败反思再重试）、Tree of Thoughts（多思路选优）。

**第四层 记忆与状态的分层管理**：核心洞察——**Agent 的状态不应该放在上下文窗口里，而应该外化到文件系统**。Anthropic《Effective harnesses for long-running agents》做法：维护 `claude-progress.txt` 日志 + `init.sh` 启动脚本 + 完整 git history 作为长期记忆介质。三类状态必须分层存：任务状态（`today-progress.json`，活到任务结束）、会话中间结果（随会话结束丢）、长期记忆与用户偏好（`user-preferences.md`，跨所有任务存在）。CLAUDE.md / .cursorrules 就是「长期记忆」的典型实现。

**第五层 独立的评估与观测体系**：最容易被跳过、跳过就进退两难的一层。两件事——① **Eval 集（灵魂）**：手写一批典型任务并标注正确答案，每次改 Harness（改 CLAUDE.md、加工具、调整流程）都重跑对比成功率；LangChain 靠 Eval 集 + trace 回放把 Terminal Bench 从 52.8 推到 66.5、榜单 30 开外冲到前 5；② **Trace + 日志 + 指标**（LangSmith/Langfuse）：看到每一步决策、调了哪个工具、拿到什么返回、花多少 token，定位失败那步发生了什么。没有 Eval 集，判断永远停留在「我感觉这次变好了」的玄学阶段。

**第六层 约束校验与失败恢复机制**：真实环境里**失败不是例外，是常态**。三件事：① **约束**——定义「什么事 Agent 不能做」（一次最多分析 20 个 PR / 不能对 closed PR 评论 / token 超 10 万立刻停），最好硬编码到代码或 linter，而不是写在提示词里靠 Agent 自己遵守；② **校验**——每步输出前后自动检查（格式校验、频道名白名单），**校验不是审美品味，是硬规则**；③ **恢复**——每种典型失败都有明确恢复路径（API 限流→等待重试；发送失败→本地队列；token 快耗光→保存进度下一轮继续）。

### 六层与 Mitchell 复利效应的对应

修复沉到哪儿？**答案就是六层其中之一**：漏上下文信息→第一层；用错工具→第二层；步骤乱→第三层；跨天记不住→第四层；无法判断好坏→第五层；一失败就崩溃→第六层。**六层不是一次搭完的任务清单，是路标——告诉你下一次 Agent 犯错时修复该落到哪里**。

## 四、五大难点与大厂反常识解法

### 难题一：Agent 跑久了为啥越走越偏？

**现象**：跑着跑着「忘」——忘目标、忘已做决定、重复劳动、偏离主线。Cognition 用 Claude Sonnet 4.5 重做 Devin 时观察到「**上下文焦虑**」（Context Anxiety）：模型觉得窗口快用完时，不仅丢细节，还会着急想收尾——突然简化方案、跳过验证步骤、急匆匆宣布「任务完成」；且模型对自己「还剩多少上下文」估计非常不准，经常以为快没空间了其实还剩一大半。

**常见解法的局限**：上下文压缩（Context Compaction）对，但**光压缩不够**——Anthropic 确认 Sonnet 4.5 的「累了」负担感只压缩历史不换窗口去不掉。

**反常识解法：Context Reset**——直接把旧上下文窗口整个丢掉，换干净的接手。落地 = **让 Agent 跨多轮接力跑，状态全部外化到文件系统**：整个系统只有一个 Agent（系统提示词、工具集、harness 全一样），变的只是每轮初始 user prompt——第一轮用「初始化」prompt 写 `claude-progress.txt`/`init.sh`/初始 git commit，后续每轮用「增量推进」prompt 做一点进展再写回文件。每轮开始 Agent 面对完全干净的窗口，靠读交接文档恢复「我在哪一步」。（initializer agent / coding agent 的命名只是方便讲解，其实是同一个 Agent。）像工程里遇到内存泄漏不会拼命优化内存，而是**重启进程、从磁盘恢复状态**。

> **原则一：重启胜过修补，状态沉到文件里，Agent 随时可以在一个干净的上下文窗口里接力继续。**

### 难题二：让 Agent 自己给自己打分，为啥总偏乐观？

**现象**：干完活让它自评，它永远觉得自己干得不错；没有标准答案的任务（设计 UI、写文案、评估可读性）偏差特别明显。类比：让一个人自己给自己打绩效，很难公平。

**Anthropic 解法：三角分工**——**Planner**（把模糊需求扩展成完整规格）/ **Generator**（一步步实现）/ **Evaluator**（像 QA 一样真实测试，必须真的操作页面、看交互、检查运行结果，验收要有「真实环境」托底）。三者足够独立，系统形成闭环：规划 → 生成 → 验收 → 修复 → 再验收。**不要让一个 Agent 既当运动员又当裁判**。

> **原则二：生产和验收必须分离，而且验收方必须能摸到真实世界。**

### 难题三：Agent 总是失败，工程师到底该干啥？

**OpenAI Codex 实践**：人类工程师几乎不写一行代码，全部由 Agent 写。工程师工作重心压在三件事：① 把产品目标拆解成 Agent 能力边界内的小任务；② **当 Agent 反复失败时，不催它「再努力一点」，而是看它「环境里缺了什么能力」，把那个能力补进环境里**；③ 建立反馈链路让 Agent 真正能看到自己工作的结果。

思维转变：以前加提示词「请仔细检查不要有 bug」是求模型发挥；现在给 Agent 接上 lint、单测、运行环境，让它自己写完自己跑、看见 bug 自己改，是改造环境——彻底决定下次会不会再犯。

> **原则三：Agent 反复失败的时候，别问模型能不能更努力，要问环境还缺什么。**

### 难题四：规范文件越写越长，为啥 Agent 反而更糊涂？

**OpenAI 自己踩的坑**：早期 Codex 搞了个特别大的 `AGENTS.md` 把所有规范塞进去，以为越全越好，结果 Agent 更糊涂——文件每次都注入，越长注意力越稀释（上下文腐化在规则文件场景的表现）。

**解法：从「百科全书」改成「目录页」**——主文件只保留 **100 行左右**的核心索引（OpenAI 原文明确数字），只告诉 Agent「你想看什么，去哪儿看」；详细内容拆到子文档（架构/设计原则/产品规格/执行计划/质量评分各一份）。就是**渐进式披露**，和软件设计的「按需加载/懒加载」一脉相承：**该给的时候给，不该给的时候藏起来**。写 CLAUDE.md / Cursor Rules 的同学应自查有没有「百科全书化」。

> **原则四：规则文件宁缺毋滥，给模型看的东西少即是多。**

### 难题五：Agent 写的代码越堆越烂，技术债怎么还？

**现象**：Agent 会疯狂模仿仓库里已有的代码模式——好的复制，坏的也复制；早期某段写歪了会被当成「惯例」越堆越歪，OpenAI 起名 **AI slop**（AI 代码泔水）。

**失败的人工方案**：每周五（20% 时间）让人类工程师手工打扫 AI slop——**失败了**，Agent 产出速度远超人类清理速度，周五清一天周一一早又堆满。

**Harness 解法两步**：① 把资深工程师「什么是好代码」的经验写成 **Golden Principles**（黄金原则）沉进仓库——如「优先用共享工具包而不是手写 helper」「不要瞎猜数据格式，必须校验边界或用带类型的 SDK」；② 让后台 Agent 按固定节奏自动跑，定期扫描仓库对比 Golden Principles，自动开修复 PR，大部分修复 PR **1 分钟内**被人类审完直接 auto merge。把「还技术债」从一周一次人工集中清扫变成每天持续自动偿还。

> 技术债就像一笔高利息贷款，几乎永远应该每天小额还一点点，而不是攒着等某一天集中还。

> **原则五：技术债不是攒一堆集中还，而是每天让后台 Agent 自动偿还一点。**

### 反直觉发现：Agent 用「老技术」反而更稳

OpenAI 实践：**Agent 对那些被人类称为「boring」的老技术掌握得最好**。三个原因：① 组合性好 ② API 稳定 ③ 训练数据里出现得多。老库十几年沉淀，代码示例/SO 问答/博客多如牛毛；新框架 AI 只看过零星文档容易张冠李戴。Codex 甚至**主动选择看起来很土的技术栈**；有时宁愿让 Agent 自己实现小工具函数，也不引入流行 npm 包——自己写的代码 Agent 能 100% 理解和控制，第三方包藏着 Agent 看不懂的黑盒行为。**做让 Agent 跑得稳的项目，选技术栈不要一味追新。**

### 五条原则口诀

> **重启胜过修补，生产验收分家，与其催模型不如改环境，规则宁缺毋滥，技术债天天还。**

共同点：都不是在调模型，而是在设计模型外面的那一整套环境——这就是 Harness Engineering 的灵魂。

## 五、总结

- Prompt Engineering 解决「怎么把任务讲清楚」（塑造概率空间，让模型听懂）
- Context Engineering 解决「怎么把信息送对」（动态管理上下文，让模型知道）
- Harness Engineering 解决「怎么让模型在真实执行中持续做对」（设计包裹模型的运行环境）
- 层层包含：Prompt 是 Context 的一部分，Context 是 Harness 的一部分
- 模型决定 Agent 的天花板，**Harness 决定它能不能落地、能不能稳定交付**
- 趋势：认真做 Harness 几个月，花在「亲手写代码」上的时间越来越少，花在「写规则、写流程、设计环境」上的时间越来越多——**设计能让 Agent 高质量产出的环境，是未来程序员真正的主战场**

## 参考资料

- Mitchell Hashimoto《My AI Adoption Journey》：https://mitchellh.com/writing/my-ai-adoption-journey
- OpenAI《Harness engineering: leveraging Codex in an agent-first world》：https://openai.com/index/harness-engineering/
- Anthropic《Effective harnesses for long-running agents》：https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents
- Anthropic《Harness design for long-running application development》：https://www.anthropic.com/engineering/harness-design-long-running-apps
- Anthropic《Effective context engineering for AI agents》：https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents
- LangChain《Improving Deep Agents with harness engineering》：https://blog.langchain.com/improving-deep-agents-with-harness-engineering/
- Cognition《Rebuilding Devin for Claude Sonnet 4.5: Lessons and Challenges》：https://cognition.ai/blog/devin-sonnet-4-5-lessons-and-challenges
