# Agent 面试题 9：Agent 的长短期记忆系统怎么做的？记忆怎么存？粒度多少？怎么用？

> **来源站点**：小林面试笔记（xiaolinnote.com）
> **原文链接**：https://xiaolinnote.com/ai/agent/9_memory_storage.html
> **抓取日期**：2026-10-20
> **所属知识域**：域6 性能优化与架构演进（AI 前沿）
> **板块**：Agent 面试题（30 题）· 工程实践

---

## 核心答案

记忆分两层：**短期记忆**是 context window 里的对话历史，存当前任务中间状态，任务结束清掉；**长期记忆**用向量数据库存，embedding 后写入，语义检索拿回来注入 prompt。

**粒度**：按「一次完整交互」或「一个关键事件/独立知识点」为单位存——太细碎检索噪音大，太粗糙丢失细节。

## 短期记忆

每次调 LLM 时传进去的 messages 列表——LLM 当前的「工作台」：用户指令、LLM 自己的思考、工具调用结果、每步中间状态。LLM 靠这张桌面知道「我在做什么、做到哪了、前面几步发现了什么」。

```python
class ShortTermMemory:
    def __init__(self):
        self.messages = []   # role: user / assistant / tool

    def add(self, role: str, content: str):
        self.messages.append({"role": role, "content": content})

    def get_context(self):
        return self.messages   # 调 LLM 时把完整历史传进去

    def clear(self):
        self.messages = []    # 任务结束后清空
```

**关键点：每次调 LLM 传的是完整历史，而不只是最新一条消息**——把整个任务状态带在身上。代价：messages 越来越长，context window 装满后早期内容被截断，Agent 开始「遗忘」。任务结束清空，下次新任务桌面是空的。

**进阶：结构化工作记忆（Structured Working Memory）**：给工作台划几个固定区域——「当前任务目标」「已确认的中间结论」「待验证的假设」。每步执行后主动更新对应区域（过时结论替换掉、已验证假设移到确认区）。好处：对话很长时工作台始终结构清晰，不被杂乱历史淹没，LLM 每次读到的都是当前最准确的任务状态。

## 长期记忆

核心工具：**向量数据库 + Embedding**。

- **Embedding**：把文字转成一组数字（几百到几千维），捕捉语义。语义相近的文字向量在空间里靠得近（类比：颜色编码 RGB，红色 (255,0,0) 和橙色 (255,165,0) 距离近；实际文本 embedding 是高维空间，距离用余弦相似度算）；
- **向量数据库**：专门存数字向量，核心能力「相似度检索」——给查询向量找出距离最近的几条记录。类比图书馆索引卡：不用逐本翻阅，先查索引卡快速定位。HNSW/IVF 这类近似最近邻（ANN）索引结构加速——不用和库里每条都比，只在少量候选里精查。

**三个子类型**（按类型区分存储，检索时精准去对应库查）：
1. **语义记忆（Semantic）**：事实性知识（「用户是 Python 开发者」「项目预算上限 5 万」）——回答「是什么」；
2. **情节记忆（Episodic）**：具体事件经历（「上周二用户让我写爬虫，因反爬策略改了三次方案」）——回答「之前怎么处理过类似情况」；
3. **程序记忆（Procedural）**：方法论/SOP（「给这个用户写代码时，先确认风格偏好，再写主逻辑，最后加注释」）——回答「该按什么流程来」。

比一锅端混合库搜，召回质量高不少。

```python
def save_to_long_term(content: str, metadata: dict):
    embedding = client.embeddings.create(input=content, model="text-embedding-3-small").data[0].embedding
    collection.add(
        embeddings=[embedding],   # 索引：用于相似度检索
        documents=[content],      # 原文：检索命中后返回给 LLM 直接使用
        metadatas=[metadata],     # 标签：存入时间/任务类型/重要程度/记忆类型，检索时按标签过滤
        ids=[f"mem_{hash(content)}"]
    )

def retrieve_memory(query: str, top_k: int = 3) -> list[str]:
    query_embedding = client.embeddings.create(input=query, model="text-embedding-3-small").data[0].embedding
    results = collection.query(query_embeddings=[query_embedding], n_results=top_k)
    return results["documents"][0]
```

**metadata 非常关键**：是记忆的「标签」，后续检索可按标签过滤（只查「coding 类型」、只查「最近 7 天」）。**存储和查询必须用同一个 embedding 模型**，语义距离才可比。

**记忆衰减**：给每条记忆加「新鲜度权重」，检索排序同时考虑语义相似度和时间新鲜度，越久远权重越低。或定期让 LLM 审查记忆库，把过时、矛盾的记忆标记失效或合并更新，保持记忆库「健康状态」。

## 粒度问题（面试易错点）

- **太细**：每句话存一条。「用户偏好 Python，不喜欢全局变量，风格追求简洁，注释要用英文」拆成四条 → 检索时只命中其中几条，记忆碎片化，LLM 拿到不完整偏好；
- **太粗**：一整次任务完整记录存一条。两千 token 里真正相关的可能只有一百 → LLM 在一堆无关内容里找有用信息，易被干扰；
- **合理粒度**：**「一次完整交互」（用户具体请求 + Agent 处理结果，信息完整性好）或「一个独立的知识点/事件」（如「用户偏好：Python，简洁风格，英文注释」打包成一条结构化记录，检索一次拿完整偏好）**。过程中的细小中间结果通常不存长期记忆，短期记忆够用。

## 两层记忆的配合

```python
def run_agent_with_memory(user_request: str, long_term_memory, short_term_memory):
    # 1. 任务开始前：用任务描述检索长期记忆，拿出相关历史
    relevant_memories = long_term_memory.retrieve(user_request, top_k=3)

    # 2. 把检索到的长期记忆注入 system prompt，LLM 当作背景知识
    system_prompt = f"""你是一个智能助手。
以下是用户的相关历史信息，请在处理任务时参考：
{chr(10).join(relevant_memories)}"""
    short_term_memory.add("system", system_prompt)
    short_term_memory.add("user", user_request)

    # 3. 任务执行中：靠短期记忆维持状态，每步中间结果追加进 messages
    result = execute_task_with_short_term_memory(short_term_memory)

    # 4. 任务完成后：重要结论写入长期记忆，沉淀下来下次用
    if result.is_important:
        long_term_memory.save(content=result.summary,
                              metadata={"task_type": "coding", "timestamp": now()})
    return result
```

场景：用户第一次说「帮我优化这段 Python 代码」→ 任务后把「该用户偏好 Python，代码风格简洁，变量命名用英文」存入长期记忆。两天后「帮我写个网页爬虫」→ 开始前先用「网页爬虫」检索长期记忆，拿到偏好注入 system prompt → 写出来的爬虫自然符合偏好。**用户感觉「这个 AI 真了解我」，实际是长期记忆在背后起作用。**

短期记忆是任务执行中时刻变化的「工作台」（保当前任务连贯）；长期记忆是任务前检索注入、任务后写入沉淀的「档案」（保跨任务积累）。

## 面试翻车点（三个雷）

1. **把长期记忆说成「存数据库靠关键词搜索」**（如 SQL LIKE）——暴露不了解向量检索。长期记忆核心是 Embedding + 向量数据库，**靠语义相似度而非字符串匹配**（用户问「代码习惯」，历史存「Python 风格偏好」，关键词不重叠，语义检索才能匹配）；
2. **以为粒度越细越好**——太细导致记忆碎片化，检索拿到不完整信息。合理粒度是「一次完整交互」或「一个独立知识点」；
3. **搞不清两层记忆的作用时机**——短期是任务执行中的工作台、结束清空；长期是任务前检索注入、任务后写入沉淀。
